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

```php
#!/bin/bash
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

# Configuration
SSH_PORT=22
PBX_HOST='root@192.168.0.201'
CONF_DB_FILE='/cf/conf/mikopbx.db'
STORAGE_PBX_DIR='/storage/usbdisk1/mikopbx'
SSH_KEY_PATH="$HOME/.ssh/id_ed25519"
SYNC_APP='rsync'

# Check if rsync is available, otherwise use scp
type "$SYNC_APP" > /dev/null 2> /dev/null
if [ "$?" = '1' ]; then
    SYNC_APP='scp'
    echo "rsync not found, using scp instead"
fi

# Function to perform data transfer
perform_data_transfer() {
    # Create necessary directories
    mkdir -p "$STORAGE_PBX_DIR/tmp"
    mkdir -p "$STORAGE_PBX_DIR/astlogs/asterisk"
    mkdir -p "$STORAGE_PBX_DIR/media"
    mkdir -p "$STORAGE_PBX_DIR/custom_modules"
    mkdir -p "$STORAGE_PBX_DIR/astspool/monitor"
    
    echo -e "\nStarting data transfer process...\n"
    
    # Main database transfer
    echo "1. Transferring main database..."
    ssh -p "$SSH_PORT" "$PBX_HOST" "sqlite3 $CONF_DB_FILE .dump > $STORAGE_PBX_DIR/tmp/mikopbx.db.dmp"
    "$SYNC_APP" "$PBX_HOST":"$STORAGE_PBX_DIR/tmp/mikopbx.db.dmp" "$STORAGE_PBX_DIR/tmp/mikopbx.db.dmp"
    sqlite3 "$STORAGE_PBX_DIR/tmp/mikopbx.db" < "$STORAGE_PBX_DIR/tmp/mikopbx.db.dmp"
    mv "$STORAGE_PBX_DIR/tmp/mikopbx.db" "$CONF_DB_FILE"
    rm -f "$STORAGE_PBX_DIR/tmp/mikopbx.db.dmp"
    ssh -p "$SSH_PORT" "$PBX_HOST" "rm -f $STORAGE_PBX_DIR/tmp/mikopbx.db.dmp"
    
    # Disable providers
    echo "2. Disabling providers..."
    sqlite3 "$CONF_DB_FILE" "UPDATE m_Sip SET disabled='1' WHERE type='friend'"
    
    # Copy media files
    echo "3. Copying media files..."
    "$SYNC_APP" -r "$PBX_HOST":"$STORAGE_PBX_DIR"/media/* "$STORAGE_PBX_DIR"/media/
    
    # Copy custom modules
    echo "4. Copying custom modules..."
    "$SYNC_APP" -r "$PBX_HOST":"$STORAGE_PBX_DIR"/custom_modules/* "$STORAGE_PBX_DIR"/custom_modules/
    
    # CDR database transfer
    echo "5. Transferring call history..."
    ssh -p "$SSH_PORT" "$PBX_HOST" "sqlite3 $STORAGE_PBX_DIR/astlogs/asterisk/cdr.db .dump > $STORAGE_PBX_DIR/astlogs/asterisk/cdr.db.dmp"
    "$SYNC_APP" -r "$PBX_HOST":"$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db.dmp "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db.dmp
    sqlite3 "$STORAGE_PBX_DIR"/astlogs/asterisk/cdrdb.tmp < "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db.dmp
    rm -f "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db*
    mv "$STORAGE_PBX_DIR"/astlogs/asterisk/cdrdb.tmp "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db
    rm -f "$STORAGE_PBX_DIR"/astlogs/asterisk/cdr.db.dmp
    ssh -p "$SSH_PORT" "$PBX_HOST" "rm -f $STORAGE_PBX_DIR/astlogs/asterisk/cdr.db.dmp"
    
    # Update database structure
    echo "6. Updating database structure..."
    php -r 'require_once "Globals.php"; use MikoPBX\Core\System\Upgrade\UpdateDatabase; $dbUpdater = new UpdateDatabase(); $dbUpdater->updateDatabaseStructure();'
    
    # Copy call recordings
    echo "7. Copying call recordings..."
    "$SYNC_APP" -r "$PBX_HOST":"$STORAGE_PBX_DIR"/astspool/monitor "$STORAGE_PBX_DIR"/astspool/monitor/
    
    echo -e "\nData transfer completed successfully!"
}

# Function to setup SSH environment
setup_ssh_environment() {
    mkdir -p "$HOME/.ssh"
    chmod 700 "$HOME/.ssh"
}

# Function to generate SSH key using dropbearkey and convert to OpenSSH format
generate_ssh_key() {
    echo "Generating new ED25519 SSH key..."
    
    setup_ssh_environment
    
    # Remove existing keys if they exist
    rm -f "$SSH_KEY_PATH" "${SSH_KEY_PATH}.pub"
    
    # Generate temporary keys using dropbearkey
    local temp_key="/tmp/dropbear_temp"
    rm -f "$temp_key" "${temp_key}.pub"
    
    # Generate key using dropbearkey
    dropbearkey -t ed25519 -f "$temp_key" 2>/dev/null
    if [ $? -ne 0 ]; then
        echo "Failed to generate private key"
        exit 1
    fi
    
    # Generate public key without comment
    dropbearkey -y -f "$temp_key" | grep "^ssh-ed25519" | cut -d' ' -f1,2 > "${SSH_KEY_PATH}.pub"
    if [ ! -s "${SSH_KEY_PATH}.pub" ]; then
        echo "Failed to extract public key"
        rm -f "$temp_key" "${temp_key}.pub" "$SSH_KEY_PATH" "${SSH_KEY_PATH}.pub"
        exit 1
    fi
    
    # Convert private key to OpenSSH format
    dropbearconvert dropbear openssh "$temp_key" "$SSH_KEY_PATH"
    if [ $? -ne 0 ]; then
        echo "Failed to convert private key format"
        rm -f "$temp_key" "${temp_key}.pub" "$SSH_KEY_PATH" "${SSH_KEY_PATH}.pub"
        exit 1
    fi
    
    # Clean up temporary files
    rm -f "$temp_key" "${temp_key}.pub"
    
    # Set correct permissions
    chmod 600 "$SSH_KEY_PATH"
    chmod 644 "${SSH_KEY_PATH}.pub"
    
    echo -e "\nHere's your public SSH key. Please copy it and add it to /root/.ssh/authorized_keys on the old PBX:\n"
    cat "${SSH_KEY_PATH}.pub"
    
    echo -e "\nPress any key after you've added the key to continue..."
    read -n 1 -s
    
    # Test connection immediately after key is added
    test_ssh_connection
    if [ $? -eq 0 ]; then
        perform_data_transfer
    else
        echo "SSH connection failed. Please check your configuration and try again."
        exit 1
    fi
}

# Function to check for existing SSH keys
check_existing_keys() {
    for keytype in ed25519 rsa ecdsa; do
        if [ -f "$HOME/.ssh/id_$keytype" ]; then
            echo "Found existing $keytype key at $HOME/.ssh/id_$keytype"
            echo "Would you like to use this key? (Y/n)"
            read -r use_existing
            if [[ "$use_existing" =~ ^([nN][oO]|[nN])+$ ]]; then
                continue
            else
                SSH_KEY_PATH="$HOME/.ssh/id_$keytype"
                echo -e "\nHere's your public SSH key. If you haven't added it yet to the old PBX, please copy it:"
                cat "${SSH_KEY_PATH}.pub"
                echo -e "\nPress any key after you've verified the key is added..."
                read -n 1 -s
                
                # Test connection immediately after key is confirmed
                test_ssh_connection
                if [ $? -eq 0 ]; then
                    perform_data_transfer
                    return 0
                else
                    echo "SSH connection failed with existing key."
                    return 1
                fi
            fi
        fi
    done
    return 1
}

# Function to test SSH connection
test_ssh_connection() {
    echo "Testing SSH connection..."
    ssh -o StrictHostKeyChecking=accept-new -o PasswordAuthentication=no -o BatchMode=yes -p "$SSH_PORT" "$PBX_HOST" exit 2>/dev/null
    return $?
}

# Main script starts here
echo "MikoPBX Data Transfer Script"
echo "=========================="

echo "Do you want to generate a new SSH key? (y/N)"
read -r response
if [[ "$response" =~ ^([yY][eE][sS]|[yY])+$ ]]; then
    generate_ssh_key
else
    if ! check_existing_keys; then
        echo "No existing SSH keys found or connection failed. Generating new key..."
        generate_ssh_key
    fi
fi
```

***
