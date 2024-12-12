# Перенос с помощью rsync (в разработке)

{% hint style="danger" %}
Пока страница в разработке - пользуйтесь первым скриптом, если это необходимо!
{% endhint %}

### Вариант №3 <a href="#variant_3" id="variant_3"></a>

Скрипт для переноса данных вручную.

```php
#
# MikoPBX - free phone system for small business
# Copyright © 2017-2024 Alexey Portnov and Nikolay Beketov
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License along with this program.
# If not, see <>.
#

SSH_PORT=22;
PBX_HOST='root@PBX-ADDRESS-OR-IP';
CONF_DB_FILE='/cf/conf/mikopbx.db';
STORAGE_PBX_DIR='/storage/usbdisk1/mikopbx';
SYNC_APP='rsync';
type "$SYNC_APP" > /dev/null 2> /dev/null
if [ "$?" = '1' ];then
  SYNC_APP='scp';
fi;

# TODO
# Копирование SSH ключей, чтобы не вводить многократно пароль.
#rsaPubKey="$HOME/.ssh/id_rsa.pub";
#if [ ! -f "$rsaPubKey" ];then
#  dropbearkey -y -f /etc/dropbear/dropbear_rsa_host_key | grep "^ssh-rsa" > "$rsaPubKey";
#fi;
#ssh "$PBX_HOST" -p "$SSH_PORT" "echo >> /root/.ssh/authorized_keys && echo '$(cat "$rsaPubKey")' >> /root/.ssh/authorized_keys && nohup sh -c 'killall dropbear && /usr/sbin/dropbear -p $SSH_PORT' 2>&1 &";
#ssh "$PBX_HOST" -p "$SSH_PORT" 'sqlite3 /cf/conf/mikopbx.db .tables';

# Дамп.
ssh "$PBX_HOST" -p "$SSH_PORT" "sqlite3 $CONF_DB_FILE .dump > $STORAGE_PBX_DIR/tmp/mikopbx.db.dmp";
# Копируем дамп на локальную машину.
"$SYNC_APP" "$PBX_HOST":"$STORAGE_PBX_DIR/tmp/mikopbx.db.dmp" "$STORAGE_PBX_DIR/tmp/mikopbx.db.dmp"
# Восстанавливаем базу данных из дампа.
sqlite3 "$STORAGE_PBX_DIR/tmp/mikopbx.db" < "$STORAGE_PBX_DIR/tmp/mikopbx.db.dmp";
# Перемещаем восстановленную базу данных.
mv "$STORAGE_PBX_DIR/tmp/mikopbx.db" "$CONF_DB_FILE";
# Чистим временные файлы.
rm -rf "$STORAGE_PBX_DIR/tmp/mikopbx.db";
ssh "$PBX_HOST" -p "$SSH_PORT" "rm -rf $STORAGE_PBX_DIR/tmp/mikopbx.db";

# Отключаем провайдеров на локальной машине.
sqlite3 "$CONF_DB_FILE" "UPDATE m_Sip SET disabled='1' WHERE type='friend'"

# Копируем медиа файлы с удаленной машины.
"$SYNC_APP" -r "$PBX_HOST":"$STORAGE_PBX_DIR"/media/* "$STORAGE_PBX_DIR"/media
# Копируем дополнительные модули с удаленной машины.
"$SYNC_APP" -r "$PBX_HOST":"$STORAGE_PBX_DIR"/custom_modules/* "$STORAGE_PBX_DIR"/custom_modules

# Копируем историю звонков.
# Дамп.
ssh "$PBX_HOST" -p "$SSH_PORT" "sqlite3 $STORAGE_PBX_DIR/astlogs/asterisk/cdr.db .dump > $STORAGE_PBX_DIR/astlogs/asterisk/cdr.db.dmp";
# Копируем дамп на локальную машину.
"$SYNC_APP" -r "$PBX_HOST":"$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db.dmp "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db.dmp;
# Восстанавливаем базу данных из дампа.
sqlite3 "$STORAGE_PBX_DIR"/astlogs/asterisk/cdrdb.tmp < "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db.dmp;
# Удаляем текущую базу данных истории звонков.
rm -rf "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db*;
# Перемещаем восстановленную базу данных.
mv "$STORAGE_PBX_DIR"/astlogs/asterisk/cdrdb.tmp "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db;

# Чистим временные файлы.
rm -rf "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db.dmp;
ssh "$PBX_HOST" -p "$SSH_PORT" "rm -rf $STORAGE_PBX_DIR/astlogs/asterisk/cdr.db.dmp";

# Обновление структуры баз данных sqlite3.
php -r 'require_once "Globals.php"; use MikoPBX\Core\System\Upgrade\UpdateDatabase; $dbUpdater = new UpdateDatabase(); $dbUpdater->updateDatabaseStructure();'

# Копирование записей разговоров.
"$SYNC_APP" -r "$PBX_HOST":"$STORAGE_PBX_DIR"/astspool/monitor "$STORAGE_PBX_DIR"/astspool/monitor
```

**Шаг 1: Подготовка сервера**

На новом сервере нужно подготовить директорию для передачи данных и создать скрипт для синхронизации данных.

1. **Создайте директорию для передачи данных:**
   *   На сервере выполните команду для создания папки, в которой будут храниться файлы для переноса:

       ```bash
       mkdir -p /storage/usbdisk1/transfer
       ```
2. **Перейдите в эту папку:**
   *   Перейдите в директорию, где будет храниться скрипт:

       ```bash
       cd /storage/usbdisk1/transfer
       ```

**Шаг 2: Создание скрипта**

Теперь нужно создать скрипт для передачи данных с удалённого сервера на новый сервер.

1. **Создайте файл скрипта:**
   *   Используйте текстовый редактор `vim` (или другой редактор по вашему выбору) для создания и редактирования файла `transfer-rsync.sh`:

       ```bash
       touch transfer-rsync.sh
       vi transfer-rsync.sh
       ```
2.  **Заполните файл содержимым:** Вставьте в файл скрипта следующее содержимое:

    ```bash
    #!/bin/bash

    # MikoPBX - transfer script for data migration
    SSH_PORT=22;                           # Порт SSH для подключения
    PBX_HOST='root@OLD-PBX-ADDRESS';       # Замените на IP или адрес старого сервера
    CONF_DB_FILE='/cf/conf/mikopbx.db';    # Путь к конфигурационному файлу базы данных
    STORAGE_PBX_DIR='/storage/usbdisk1/mikopbx';  # Путь для хранения данных
    SYNC_APP='rsync';                      # Утилита для синхронизации
    type "$SYNC_APP" > /dev/null 2> /dev/null
    if [ "$?" = '1' ]; then
      SYNC_APP='scp';
    fi;

    # Копирование SSH-ключа на сервер, чтобы не вводить пароль
    rsaPubKey="$HOME/.ssh/id_rsa.pub";
    if [ ! -f "$rsaPubKey" ]; then
      dropbearkey -y -f /etc/dropbear/dropbear_rsa_host_key | grep "^ssh-rsa" > "$rsaPubKey";
    fi;

    # Добавление публичного ключа в authorized_keys на сервере
    ssh "$PBX_HOST" -p "$SSH_PORT" "echo >> /root/.ssh/authorized_keys && echo '$(cat "$rsaPubKey")' >> /root/.ssh/authorized_keys && nohup sh -c 'killall dropbear && /usr/sbin/dropbear -p $SSH_PORT' 2>&1 &";

    # Копирование медиа файлов, модулей и записей разговоров
    "$SYNC_APP" -r "$PBX_HOST":"$STORAGE_PBX_DIR"/media/* "$STORAGE_PBX_DIR"/media;
    "$SYNC_APP" -r "$PBX_HOST":"$STORAGE_PBX_DIR"/custom_modules/* "$STORAGE_PBX_DIR"/custom_modules;
    "$SYNC_APP" -r "$PBX_HOST":"$STORAGE_PBX_DIR"/astspool/monitor "$STORAGE_PBX_DIR"/astspool/monitor;

    # Дамп базы данных конфигурации MikoPBX
    ssh "$PBX_HOST" -p "$SSH_PORT" "sqlite3 $CONF_DB_FILE .dump > $STORAGE_PBX_DIR/tmp/mikopbx.db.dmp";
    "$SYNC_APP" "$PBX_HOST":"$STORAGE_PBX_DIR/tmp/mikopbx.db.dmp" "$STORAGE_PBX_DIR/tmp/mikopbx.db.dmp";
    sqlite3 "$STORAGE_PBX_DIR/tmp/mikopbx.db" < "$STORAGE_PBX_DIR/tmp/mikopbx.db.dmp";
    mv "$STORAGE_PBX_DIR/tmp/mikopbx.db" "$CONF_DB_FILE";  # Восстановление базы данных
    rm -rf "$STORAGE_PBX_DIR/tmp/mikopbx.db";  # Удаление временных файлов
    ```

    **Обратите внимание:**

    * Замените `OLD-PBX-ADDRESS` на IP-адрес или доменное имя старого сервера.
    * Убедитесь, что путь к файлу базы данных и медиафайлам верный для вашей конфигурации.
3. **Сохраните и выйдите:**
   * После того как файл будет отредактирован, сохраните изменения и выйдите из редактора.
     * Нажмите `Esc` в редакторе `vi`, затем введите `:wq` и нажмите `Enter` для сохранения и выхода.

**Шаг 3: Сделать скрипт исполняемым**

Чтобы скрипт можно было выполнить, нужно дать ему права на выполнение.

1.  **Сделайте файл исполняемым:**

    ```bash
    chmod +x transfer-rsync.sh
    ```

**Шаг 4: Запуск скрипта**

Теперь вы готовы запустить скрипт и начать процесс переноса данных.

1.  **Запустите скрипт:**

    *   Для выполнения скрипта просто используйте команду:

        ```bash
        ./transfer-rsync.sh
        ```

    Этот процесс начнёт синхронизацию данных между старым и новым сервером, включая копирование базы данных, медиафайлов и пользовательских модулей.

**Шаг 5: Проверка результата**

После завершения работы скрипта выполните проверку, чтобы убедиться, что все данные были перенесены корректно.

* Проверьте наличие базы данных и файлов в директории `/storage/usbdisk1/mikopbx`.
* Убедитесь, что на новом сервере работает MikoPBX и все функции (например, звонки и медиафайлы) функционируют нормально.

***
