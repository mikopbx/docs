# Selectel

В данной инструкции мы пошагово произведем установку MikoPBX с помощью облачной платформы Selectel.&#x20;

Перед началом вам необходимо скачать актуальный образ MikoPBX с расширением .raw. Сделать это можно по [ссылке](https://github.com/mikopbx/Core/releases).

## Загрузка образа в Selectel

1. Перейдите в раздел **Облачная платформа** -> **Образы**.

<figure><img src="../../.gitbook/assets/imagesSectionSelectel.jpg" alt=""><figcaption><p>Раздел "Образы"</p></figcaption></figure>

2. Нажмите "**Создать образ**".

<figure><img src="../../.gitbook/assets/createNewImageButton.jpg" alt=""><figcaption><p>Кнопка "Создать образ"</p></figcaption></figure>

3. Укажите:

* **Имя образа** - любое желаемое название для вашего образа.
* **ОС** - Linux
* **Источник** - Файл
* **Файл** - выберите раннее загруженный файл с расширением .iso

Все остальное - **по умолчанию**.

Нажмите создать и дождитесь окончания процесса.

<figure><img src="../../.gitbook/assets/menuOfCreatingImageSelectel.jpg" alt=""><figcaption><p>Параметры загружаемого образа диска</p></figcaption></figure>

## Создание сервера в Selectel

1. Перейдите в раздел **Облачная платформа** -> **Серверы**

<figure><img src="../../.gitbook/assets/serverSectionSelctel.jpg" alt=""><figcaption><p>Раздел "Серверы"</p></figcaption></figure>

2. Нажмите "**Создать сервер**":

<figure><img src="../../.gitbook/assets/newServer.jpg" alt=""><figcaption><p>"Создать сервер"</p></figcaption></figure>

3. В конфигурации вашей ВМ укажите:

* Имя - произвольное название.
* Пул - такой же, как у раннее созданного образа.
* Источник - выберите раннее загруженный образ.
* Конфигурация - желаемое "железо" исходя из ваших потребностей.

<figure><img src="../../.gitbook/assets/confSection1.jpg" alt=""><figcaption></figcaption></figure>

* Диски: Здесь вам необходимо указать размер для первого диска (он же - системный диск) - 5Гб (минимально возможный в Selectel). А так
*
