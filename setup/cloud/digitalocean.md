---
description: Установка MikoPBX в сервисе DigitalOcean
---

# DigitalOcean (В следующем релизе)

В данной инструкции мы пошагово произведем установку MikoPBX с помощью облачной платформы DigitalOcean.

&#x20;Перед началом Вам необходимо скопировать ссылку на актуальный образ MikoPBX с расширением .raw. Сделать это можно на [github MikoPBX](https://github.com/mikopbx/core/releases).

## Загрузка образа в DigitalOcean

1. Перейдите в "**Manage**" -> "**Backups & Snapshots**"

<figure><img src="../../.gitbook/assets/backupsAndSnapshots.png" alt=""><figcaption><p>Раздел "<strong>Backups &#x26; Snapshots"</strong></p></figcaption></figure>

2. Перейдите в "**Custom Images**" -> "**Import via URL**":

<figure><img src="../../.gitbook/assets/customImagesImportViaURL.png" alt=""><figcaption><p>"Import via URL" </p></figcaption></figure>

3. Вставьте ссылку на файл образа диска с расширением .raw, скопированную ранее.&#x20;
4. Введите имя для образа, выберите регион для его загрузки (должен совпадать с будущим регионом виртуальной машины). В качестве операционной системы образа выберите "**Unknown**"

Нажмите "**Upload image**"

<figure><img src="../../.gitbook/assets/imageParameters.png" alt=""><figcaption><p>Параметры образа</p></figcaption></figure>

Дождитесь загрузки образа.

## Создание виртуальной машины в облаке

1. Перейдите на главную страницу DigitalOcean.

<figure><img src="../../.gitbook/assets/mainPageDO.png" alt=""><figcaption><p>Главная страница DigitalOcean</p></figcaption></figure>

2. Для перехода к создаю виртуальной машины, перейдите в "**Create**" -> "**Droplets**":

<figure><img src="../../.gitbook/assets/buttonForCreatingANewDroplet.png" alt=""><figcaption><p>Создание нового "Droplet"</p></figcaption></figure>

3. Выберите регион и датацентр для Вашей виртуальной машины.

<figure><img src="../../.gitbook/assets/regionAndDatacenter.png" alt=""><figcaption><p>Параметры ВМ #1</p></figcaption></figure>

4. Далее выберите ранее загруженный образ и конфигурацию для вашей виртуальной машины:

<figure><img src="../../.gitbook/assets/ImageAndConfig.png" alt=""><figcaption><p>Параметры ВМ #2</p></figcaption></figure>

5. Перейдите во вкладку "Additional Storage". Здесь необходимо добавить второй диск, который будет использоваться для хранения записей разговоров. Для этого нажмите "Add volume" и укажите параметры для нового диска.

{% hint style="info" %}
Рекомендуемый размер диска для хранения записей разговоров  - от 50ГБ.
{% endhint %}

<figure><img src="../../.gitbook/assets/additionalStorage.png" alt=""><figcaption><p>Раздел "Additional Storage"</p></figcaption></figure>

6. Перейдите в раздел "**Choose authentication method**". Здесь необходимо выбрать "SSH Key" и добавить связку ключей для SSH подключения. Подробнее прочитать про их создание Вы можете в следующих статьях:

* [WIndows](../../faq/troubleshooting/connecting-to-a-pbx-using-ssh/powershell.md)
* [MacOS/Linux](../../faq/troubleshooting/connecting-to-a-pbx-using-ssh/terminal.md)

<figure><img src="../../.gitbook/assets/sshKey.png" alt=""><figcaption><p>Методы авторизации</p></figcaption></figure>

7. Нажмите "Create Droplet"

## Подключение к консоли и первый вход в WEB-Интерфейс

1. Перейдите в меню созданной машины. Дождитесь ее запуска. Далее подключитесь к ней по SSH ([интрукция](../../faq/troubleshooting/connecting-to-a-pbx-using-ssh/)), либо с помощью встроенной консоли в DigitalOcean.

{% hint style="warning" %}
Стандартный логин для авторизации по SSH для ВМ в DigitalOcean - do-user
{% endhint %}

<figure><img src="../../.gitbook/assets/console.png" alt=""><figcaption><p>Консоль в Digital Ocean</p></figcaption></figure>

2. После загрузки системы, подключитесь через web-интерфейс, используя внешний IP-адрес.

<figure><img src="../../.gitbook/assets/externalIPAddress.png" alt=""><figcaption><p>external ip-адрес</p></figcaption></figure>

3. После перехода на страницу авторизации в MikoPBX, используйте следующие данные для входа:

* Логин - admin
* Пароль - id Виртуальной машины, найти который Вы можете в адресной строке:

<figure><img src="../../.gitbook/assets/MachineID.png" alt=""><figcaption><p>ID виртуальной машины</p></figcaption></figure>
