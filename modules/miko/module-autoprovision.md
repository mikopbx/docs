---
description: >-
  Это статья, описывающая процесс быстрой настройки телефонов в MikoPBX. В ней
  объясняется, как автоматически конфигурировать устройства для обеспечения их
  готовности к использованию в системе.
---

# Автоматическая настройка телефонов

**Autoprovisioning Plug & Play (PnP)**, эту технологию поддерживают многие производители — **Yealink**, **Snom**. Телефоны этих производителей могут быть настроен текущей версией модуля.

Основные достоинства автоматической настройки телефонов:

* **Облегчает первичную настройку** — не требуется заходить в web интерфейс каждого устройства. Достаточно на сервере автонастройки указать соответствие MAC адреса устройства и аккаунта.
* **Упрощает поддержку** — действительно становится легче при необходимости изменить настройки устройства. Управляем настройками опять же на сервере
* **Возможно свести настройку к набору star-кода:** «\*911\*\<SIP\_ACC>» — в ряде случаев этой функции просто цены нет. Не каждый офисный работник сможет настроить IP телефон, а вот набрать комбинацию цифр задача простая.

### Системные требования <a href="#sistemnye_trebovanija" id="sistemnye_trebovanija"></a>

* Модуль может работать **только** в локальной сети
* В сети должны быть разрешены **multicast запросы на IP 224.0.1.75**
* На текущий момент в качестве адреса регистрации можно задать только одно общее значение для всех устройств
* На АТС должны быть открыты порты web интерфейса (**HTTP**) и **SIP** - 80 и 5060
* Работа по **HTTPS пока не поддерживается**
* В сети не должно быть запущено других PnP серверов. Устройство будет получать настройки от первого ответившего сервера

### Поддерживаемые телефоны <a href="#podderzhivaemye_telefony" id="podderzhivaemye_telefony"></a>

#### Yealink

* Yealink T19(P)
* Yealink T28(P)
* Yealink W52
* Yealink WP530

#### Snom

* Snom D120
* Snom D785
* Snom D735
* Snom D715
* Snom D385
* Snom D335

#### Fanvil

* Fanvil X5U
* Fanvil X3SP
* Fanvil X1SP

{% hint style="success" %}
Мы обязательно будем расширять линейку поддерживаемых телефонов.
{% endhint %}

## Настройка модуля <a href="#nastrojka_modulja" id="nastrojka_modulja"></a>

1. Перейдите в интерфейс «**Модули**» -> «**Маркетплейс модулей**».
2. Установите модуль «**Модуль автоматической настройки телефонов**».

{% hint style="danger" %}
Запускайте модуль только после завершения его настройки.
{% endhint %}

3. Перейдите к его интерфейсу:

<figure><img src="../../.gitbook/assets/image (37).png" alt=""><figcaption><p>Интерфейс настройки модуля "Модуль автоматической настройки телефонов"</p></figcaption></figure>

* **Шаблон внутреннего номера** - укажите добавочный номер для возможности настройки телефона star-кодом.
* **Адрес сервера для регистрации телефонов** - ip или имя сервера MikoPBX. По этому адресу будут подключаться телефоны к АТС.
* **Черный список MAC адресов телефонов** - перечислите MAC телефонов, которые НЕ требуется настраивать Это список описывает исключения. Черный список имеет более высокий приоритет, чем белый.
* **Белый список MAC адресов телефонов** - ограничьте настраиваемые телефоны только перечисленными.

{% hint style="warning" %}
Если **Черный** и **Белый** списки не настроены, то модуль будет пытаться настроить все телефоны.
{% endhint %}

## Дополнительные параметры конфигурации телефонов <a href="#dopolnitelnye_parametry_konfiguracii_telefonov" id="dopolnitelnye_parametry_konfiguracii_telefonov"></a>

В поле «**Дополнительные параметры**» допускается описать произвольные настройки для конфигурационных файлов телефонов.

### Yealink <a href="#yealink" id="yealink"></a>

Сервер по умолчанию генерирует следующий конфигурационный файл:

```php
#!version:1.0.0.1
account.1.enable = 1
account.1.label = Askozia (204)
account.1.display_name = 204
account.1.auth_name = 204
account.1.user_name = 204
account.1.password = 1c9709222690713dd
account.1.sip_server_host = 172.16.156.223
account.1.sip_server_port = 5060
account.1.transport = 0
account.1.codec.1.enable = 1
account.1.codec.1.payload_type = PCMU
account.1.codec.1.priority = 1
account.1.codec.1.rtpmap = 0
account.1.cid_source = 4
voice_mail.number.1 = *001
phone_setting.lcd_logo.mode=0
auto_provision.dhcp_option.enable = 0
features.intercom.allow = 1
features.intercom.mute = 0
features.intercom.tone = 1
features.intercom.barge = 1
features.dtmf.transfer = ##
features.dtmf.replace_tran = 1
features.headset_prior = 1
features.intercom.allow = 1
```

К нему можно добавить в конец произвольный набор параметров. Для этого необходимо в поле «**Дополнительные параметры**» описать секцию «**\[yealink]**». Пример:

```php
[yealink]
features.headset_prior = 1
features.intercom.allow = 1
```

Каждый новый параметр выделяется отдельной строкой.

Ссылка на [сайт поддержки Yealink](http://support.yealink.com/documentFront/forwardToDocumentDetailPage?documentId=78#userdocument)

### Snom <a href="#snom" id="snom"></a>

Пример файла конфигурации:

```xml
<?xml version="1.0" encoding="utf-8"?>
<settings>
<time_24_format perm="R">off</time_24_format>
    <phone-settings>
        <user_pname idx="1" perm="RW">203</user_pname>
        <user_name idx="1" perm="RW">203</user_name>
        <user_realname idx="1" perm="RW">Smirnova Irina Aleksandrovna</user_realname>
        <user_pass idx="1" perm="RW">3256157a99f176eb959ef9c1fdd947f0</user_pass>
        <user_host idx="1" perm="RW">172.16.32.225</user_host>
        <user_srtp idx="1" perm="RW">off</user_srtp>
        <user_mailbox idx="1" perm="RW">*001</user_mailbox>
        <user_dp_str idx="1" perm="RW">!([^#]%2b)#!sip:\1@\d!d</user_dp_str>
        <contact_source_sip_priority idx="INDEX" perm="PERMISSIONFLAG">PAI RPID FROM</contact_source_sip_priority>
        <answer_after_policy perm="RW">idle</answer_after_policy>

    </phone-settings>
</settings>
```

Файл имеет более сложную структуру, чем у Yealink.

Для добавления данных в узел «**\<settings>**» следует описать секцию «**\[snom]**»:

```xml
[snom]

 <tbook>
  <item context="line1" type="none" index="0">
   <name>Adrian</name>
   <number>42965</number>
  </item>
  <item context="active" type="colleagues" index="1">
   <name>Roland</name>
   <number>16424</number>
  </item>
 </tbook>

 <dialplan>
  <template match="0" timeout="4" scheme="sip" user="phone" rewrite="" />
  <template match="00" timeout="0" scheme="sip" user="phone" rewrite="" />
  <template match="011............" timeout="4" scheme="sip" user="phone" rewrite="" />
  <template match="*79" timeout="0" scheme="sip" user="phone" rewrite="" />
  <template match="*234" timeout="0" scheme="sip" user="phone" rewrite="" />
 </dialplan>
```

Для добавления данных в узел «**\<phone-settings>**» следует описать секцию «**\[snom-phone-settings]**»:

```xml
[snom-phone-settings]
  <firmware_status perm="R"></<firmware_status>
  <update_policy perm="R">auto_update</update_policy>
  <firmware_interval perm="R">2880 </firmware_interval>
```

Документация доступна на сайте [wiki.snom.com](http://wiki.snom.com/Features/Auto\_Provisioning/Configuration\_Files/XML)

### Fanvil <a href="#fanvil" id="fanvil"></a>

Пример файла конфигурации:

```xml
<<VOIP CONFIG FILE>>Version:2.0002
PNP Enable         :0

<SIP CONFIG MODULE>
--SIP Line List--  :
SIP1 Enable Reg    :1
SIP1 Phone Number  :203
SIP1 Display Name  :Smirnova Irina Aleksandrovna
SIP1 Sip Name      :203
SIP1 Register Addr :172.16.156.223
SIP1 Register Port :5060
SIP1 Register User :203
SIP1 Register Pswd :3256157a99f176eb959ef9c1fdd947f0
SIP1 MWI Num       :*001
SIP1 Proxy User    :
SIP1 Proxy Pswd    :
SIP1 Proxy Addr    :


<TELE CONFIG MODULE>
SIP1 Caller Id Type:4
P1 Enable Intercom :1
P1 Intercom Mute   :0
P1 Intercom Tone   :1
P1 Intercom Barge  :1


<AUTOUPDATE CONFIG MODULE>
PNP Enable         :1
PNP IP             :224.0.1.75
PNP Port           :5060
PNP Transport      :0
PNP Interval       :1


<<END OF FILE>>
```

Принцип кастомизации схож. В поле «Дополнительные параметры» есть возможность описать следующие секции:

* **\[fanvil]** - добавляет конфигурацию в начало файла
* **\[fanvil-sip]** - добавляет строки конфигурации в конец раздела «**\<SIP CONFIG MODULE>**»
* **\[fanvil-tele]** - добавляет строки конфигурации в конец раздела «**\<TELE CONFIG MODULE>**»
* **\[fanvil-autoupdate]** - добавляет строки конфигурации в «**\<TELE CONFIG MODULE>**»

## Анализ проблем <a href="#analiz_problem_s_yealink" id="analiz_problem_s_yealink"></a>

### Анализ проблем с Yealink

{% hint style="danger" %}
Первым делом, убедитесь, что Вы используете актуальную версию прошивки телефона.

[http://support.yealink.com](http://support.yealink.com/)
{% endhint %}

1. Перейдите в web интерефс устройства
2. Перейдите в меню «**Настройки**» - «**Конфигурация**»:

* Включите уровень журнала на максимальное **значение 6.**
* **Перезагрузите** устройство.
* Выполните действие «**Экспорт**».

<figure><img src="../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

В скачанном логе следует обратить внимание на строки вида:

```xml
LIBD[528]: HTTP<5+notice> URL : 
LIBD[528]: HTTP<3+error > Connect Error
AUTP[528]: AUTP<3+error > http to file failed, code = -3, msg = Connect Failed, cout = 0
```

Видно, что телефон попытался скачать конфигурационный файл с 172.16.32.105:56080. В моем случае это был сервер со старой Askozia 4.

Корректный ответ должен выглядеть следующим образом:

```xml
Oct 17 11:26:58 LIBD[548]: HTTP<5+notice> URL : ...
Oct 17 11:26:58 LIBD[548]: DCMN<6+info  > Connecting 172.16.32.225:80
Oct 17 11:26:58 LIBD[548]: DCMN<6+info  > Connecting IP = 172.16.32.225, Port = 80
Oct 17 11:26:58 LIBD[548]: HTTP<6+info  > Request Line: GET /pbxcore/api/modules/ModuleAutoprovision/...
Oct 17 11:26:58 LIBD[548]: HTTP<6+info  > Host: 172.16.32.225
Oct 17 11:26:58 LIBD[548]: HTTP<6+info  > User-Agent: Yealink SIP-T28P 2.72.14.2 00:15:65:18:72:eb
Oct 17 11:26:58 LIBD[548]: HTTP<6+info  > process response
Oct 17 11:26:58 LIBD[548]: HTTP<5+notice> response code: 200
Oct 17 11:26:58 LIBD[548]: HTTP<6+info  > Content-Length: 961
Oct 17 11:26:58 LIBD[548]: HTTP<5+notice> response process finish!
Oct 17 11:26:58 LIBD[548]: HTTP<5+notice> recv : 961 bytes
Oct 17 11:26:58 AUTP[548]: AUTP<6+info  > download file success!!
```
