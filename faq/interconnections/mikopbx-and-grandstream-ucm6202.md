# Объединение MIKOPBX и Grandstream UCM6202



В компании два отдела, один (Администрация) должен работать в рамках АТС Grandstream, другой «Отдел продаж» должен работать с MikoPBX и иметь возможность интеграции с 1С. В рамках данной статьи опишем пример объединения станций:

1. **Администрация**- имеет внутренний номерной план **1XX**
2. **Отдел продаж** - имеет внутренний номерной план **3XX**
3. К **Grandstream** подключена внешняя линия для звонков в город
4. **MikoPBX** должна использовать внешнюю линию Grandstream для звонков в город
5. Абоненты **1XX** должны иметь возможность позвонить абонентам **3XX**
6. Абоненты **3XX** должны иметь возможность позвонить абонентам **1XX**

## Grandstream UCM6202 <a href="#grandstream_ucm6202" id="grandstream_ucm6202"></a>

### Trunk <a href="#trunk" id="trunk"></a>

1. Раздел «**Extensions / Trunk**» -«**VoIP Trunk**», выполним «**Add SIP Trunk**»
2. «**Provider name**» - укажите произвольное имя провайдера, к примеру **MikoPBX**
3. «**Host Name**» - укажите адрес MikoPBX
4. «**Transport**» - укажем **UDP**
5. Установите флаг «**Keep Original CID**»
6. Нажмите кнопку «**Save**»

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

Список Trunks:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### Исходящие на \_2XX <a href="#isxodjaschie_na_2xx" id="isxodjaschie_na_2xx"></a>

Это правило позволит абонентам Grandstream (**2XX**) звонить на внутренние номера MikoPBX **2XX**

1. Раздел «**Extensions / Trunk**» - «**Outbound Routes**»
2. Добавьте новый маршрут
3. Установите «**Trunk**» в значение «**SIP Trunks – MikoPBX**»
4. «**Calling Rule Name**» в значение «**MikoPBX**» (произвольное, понятное вам значение)
5. «**Pattern**» в значение

```
_2XX
_90000099
```

тут **90000099** - это номер очереди, которую мы позже определим на MikoPBX

6. **Privilege Level**» - в данном случае можно установить **internal**, на MikoPBX нет выхода на город / межгород

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### Входящие на \_1XX <a href="#vxodjaschie_na_1xx" id="vxodjaschie_na_1xx"></a>

Это правило позволит абонентам MikoPBX (**2XX**) звонить на внутренние номера **1XX**

1. Раздел «**Extensions / Trunk**» - «**Inbound Routes**»
2. Выберите trunk «**SIP Trunks – MikoPBX**»
3. Добавьте маршрут с **Pattern** = **\_1XX**
4. Установите «**Allowed DID Destination**» в значение «**Extension**»
5. Установите «**Default detination**» в значение «**By DID**»
6. Установите «**Privilage Level**» в значение «**internal**»

#### Исходящие на \_\[78]XX... <a href="#isxodjaschie_na_78_xx" id="isxodjaschie_na_78_xx"></a>

Это правило позволит абонентам MikoPBX (2XX) звонить на внешние номера РФ

1. Раздел «**Extensions / Trunk**» - «**Inbound Routes**»
2. Выберите trunk «**SIP Trunks – MikoPBX**»
3. Добавьте маршрут с **Pattern** = **\_\[78]XXXXXXXXXX**
4. Установите флаг «**Dial trunk**»
5. Установите «**Allowed DID Destination**» в значение «**Extension**»
6. Установите «**Default detination**» в значение «**By DID**»
7. Установите «**Privilage Level**» в значение «**National**»

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

Входящие маршруты «**SIP Trunks – MikoPBX**»:

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

### Extensions <a href="#extensions" id="extensions"></a>

Чтобы на стороне MikoPBX корректно отображался CID в карточке **Extension** следует прописать **CallerID Number**:

* Раздел «**Extensions / Trunk**» - «**Extension**»

<figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

### IVR <a href="#ivr" id="ivr"></a>

1. Добавим возможность в IVR звонить на номера MikoPBX (**\_2XX**)
2. Раздел «**Call Features**» - «**IVR**», создадим / откроем на редактирование IVR

<figure><img src="../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

3. Установите «**Dial Other Extensions**» в значение **Yes**
4. Установите «**Dial Trunk**» в значение **Yes**
5. На вкладке «**Key Pressing Events**» настройте переадресацию на **External number**:

<figure><img src="../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

**90000099** - это номер очереди, которую мы позже определим на MikoPBX

## MikoPBX

### Провайдер <a href="#provajder" id="provajder"></a>

1. Раздел «**Маршрутизация**» - «**Провайдеры телефонии**» добавьте нового провайдера
2. «**Название провайдера**» - произвольное, понятное имя, к примеру **Grandstream**
3. «**Тип учетной записи**» - Аутентификация по IP адресу, без пароля
4. «**Хост или IP адрес**» - адрес АТС **Grandstream**
5. Сохраните изменения.

<figure><img src="../../.gitbook/assets/newProvider.jpg" alt=""><figcaption></figcaption></figure>

### Входящая маршрутизация <a href="#vxodjaschaja_marshrutizacija" id="vxodjaschaja_marshrutizacija"></a>

Это правило позволит абонентам Grandstream (**2XX**) звонить на внутренние номера MikoPBX **2XX**

1. Раздел «**Маршрутизация**» - «**Входящие маршруты**»
2. Добавим новое правило:&#x20;

* **Провайдер**» - выберите «**Grandstream**»
* «**DID**» - укажите шаблон «**2XX**»
* «**Телефонный номер**» - «**Направить на сотрудника (сопоставить по DID)**»
* «**Время в секундах…**» укажите значение **300**

Сохраните изменения.

<figure><img src="../../.gitbook/assets/incomingRouting.jpg" alt=""><figcaption></figcaption></figure>

### Исходящие на 1XX <a href="#isxodjaschie_na_1xx" id="isxodjaschie_na_1xx"></a>

Это правило позволит абонентам MikoPBX (**2XX**) звонить на внутренние номера **1XX**

1. Раздел «**Маршрутизация**» - «**Исходящие маршруты**»
2. Добавим новое правило:

* Номер начинается с **«1».**
* Остальная часть номера состоит из **«2».**
* Направить звонок через провайдера «Grandstream».

<figure><img src="../../.gitbook/assets/outgoingRouting.jpg" alt=""><figcaption></figcaption></figure>

### Исходящие на Городские <a href="#isxodjaschie_na_gorodskie" id="isxodjaschie_na_gorodskie"></a>

1. Раздел «**Маршрутизация**» - «**Исходящие маршруты**»
2. Добавим новое правило:

* Номер начинается с **(7|8).**
* Остальная часть номера состоит из **10.**
* Направить звонок через провайдера «**Grandstream**».

<figure><img src="../../.gitbook/assets/outgoingRouting2.jpg" alt=""><figcaption></figcaption></figure>

### Входящие на группу <a href="#vxodjaschie_na_gruppu" id="vxodjaschie_na_gruppu"></a>

1. Раздел «**Телефония**» - «**Очереди вызовов**»
2. Добавьте очередь с внутренним номером **90000099**
3. Внутренний номер можно исправить при редактировании очереди в разделе **Расширенные настройки**«

<figure><img src="../../.gitbook/assets/numberOfQueue.jpg" alt=""><figcaption></figcaption></figure>

Направьте маршрут по умолчанию на очередь.&#x20;

<figure><img src="../../.gitbook/assets/кщгеу.jpg" alt=""><figcaption></figcaption></figure>
