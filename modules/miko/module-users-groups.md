# Группы пользователей

{% hint style="info" %}
Модуль возможно использовать начиная с **MikoPBX 2019.04.134.**
{% endhint %}

## Назначение <a href="#osnovnye_zadachi_reshaemye_modulem" id="osnovnye_zadachi_reshaemye_modulem"></a>

* Разграничение прав доступа к исходящим маршрутам
* Установка caller id для исходящего вызова

## Настройка модуля <a href="#nastrojka_modulja" id="nastrojka_modulja"></a>

1\. Выполните установку модуля в разделе [Управление модулями](../../manual/modules/pbx-extension-modules/).

2\. Включите модуль и зайдите в его настройки.

![](../../.gitbook/assets/mod\_grup\_polz\_0.gif)

3\. На основной странице модуля отображается список существующих групп. Сейчас он пустой.

![](../../.gitbook/assets/mod\_grup\_polz\_0.png)

3\. На вкладке **Cотрудники** отображается список всех сотрудников и то, к какой группе они принадлежат. Сейчас они сотрудники не принадлежат ни к какой группе, потому что самих групп еще не создано.

![](../../.gitbook/assets/mod\_grup\_polz\_1.png)

4\. Для добавления новой группы нажмите **Создать группу сотрудников.**

![](../../.gitbook/assets/mod\_grup\_polz\_2.png)

5\. На вкладке **Настройки группы** укажите ее имя и описание**.**  Затем нажмите **Сохранить.**

![](../../.gitbook/assets/mod\_grup\_polz\_3.png)

6\. Вы перейдете на вкладку **Сотрудники группы.** Добавьте в группу необходимых сотрудников.

![](../../.gitbook/assets/mod\_grup\_polz\_1.gif)

7\. Перейдите на вкладку **Правила исходящей маршрутизации** и активируйте разрешенные маршруты.

![](../../.gitbook/assets/mod\_grup\_polz\_2.gif)

{% hint style="warning" %}
Если все маршруты будут запрещены - то будут позволены только внутренние вызовы.
{% endhint %}

## Возможные сценарии использования <a href="#vozmozhnye_scenarii_ispolzovanija" id="vozmozhnye_scenarii_ispolzovanija"></a>

### Разрешить сотруднику только внутренние вызовы <a href="#razreshit_sotrudniku_tolko_vnutrennie_vyzovy" id="razreshit_sotrudniku_tolko_vnutrennie_vyzovy"></a>

1\. Создайте новую группу **Только внутренние** (название группы может быть любым).

![](../../.gitbook/assets/mod\_grup\_polz\_4.png)

2\. На вкладке **Сотрудники группы** заполните список сотрудников.

![](../../.gitbook/assets/mod\_grup\_polz\_5.png)

3\. На вкладке **Правила исходящей маршрутизации** отключите все маршруты.

![](../../.gitbook/assets/mod\_grup\_polz\_6.png)

### Запретить сотруднику международные направления <a href="#zapretit_sotrudniku_mezhdunarodnye_napravlenija" id="zapretit_sotrudniku_mezhdunarodnye_napravlenija"></a>

Настройка выполняется аналогично примеру [Разрешить сотруднику только внутренние вызовы](module-users-groups.md#razreshit\_sotrudniku\_tolko\_vnutrennie\_vyzovy) с тем лишь отличием, что следует запретить только международные маршруты.

### Персональный маршрут для каждого сотрудника <a href="#personalnyj_marshrut_dlja_kazhdogo_sotrudnika" id="personalnyj_marshrut_dlja_kazhdogo_sotrudnika"></a>

Для каждого сотрудника создайте отдельную группу. Укажите разрешенный маршрут.

### Свой caller id для каждого маршрута <a href="#svoj_caller_id_dlja_kazhdogo_marshruta" id="svoj_caller_id_dlja_kazhdogo_marshruta"></a>

На вкладке **Правила исходящей маршрутизации** укажите CallerID для каждого маршрута.

![](../../.gitbook/assets/mod\_grup\_polz\_7.png)

{% hint style="danger" %}
Не каждый провайдер позволяет подменить CallerID. Обычно позволяют использовать только тот номер, который принадлежит организации.
{% endhint %}

{% hint style="warning" %}
Если необходимо использовать этот функционал, то в настройках провайдера потребуется **отключить** использование поля **fromuser**.
{% endhint %}

![](../../.gitbook/assets/mod\_grup\_polz\_8.png)

