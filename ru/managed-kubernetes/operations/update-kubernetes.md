# Обновление {{ k8s }}

Для {{ managed-k8s-name }} доступно как автоматическое, так и ручное обновление [кластера](../concepts/index.md#kubernetes-cluster) и [группы узлов](../concepts/index.md#node-group). Вы можете в любое время запросить обновление кластера или его узлов вручную до последней поддерживаемой версии. Ручные обновления обходят любые настроенные окна обслуживания и исключения обслуживания.

При обновлении мажорной версии {{ k8s }} сначала обновите кластер, потом его группу узлов.

{% note info %}

Вы можете изменить политику обновления [кластера](#cluster-auto-upgrade) или [группы узлов](#node-group-auto-upgrade) в любое время.

{% endnote %}

Подробнее см. в разделе [{#T}](../concepts/release-channels-and-updates.md).

## Список доступных версий {{ k8s }} {#versions-list}

{% list tabs %}

- Консоль управления

  Чтобы узнать список доступных версий для кластера:
  1. Перейдите на страницу каталога и выберите сервис **{{ managed-k8s-name }}**.
  1. Нажмите на имя нужного кластера {{ k8s }}.
  1. Нажмите кнопку **Редактировать** в правом верхнем углу.
  1. Получите список доступных версий в поле **Версия {{ k8s }}** блока **Конфигурация мастера**.

  Чтобы узнать список доступных версий для группы узлов:
  1. Перейдите на страницу каталога и выберите сервис **{{ managed-k8s-name }}**.
  1. Нажмите на имя нужного кластера {{ k8s }} и перейдите на вкладку **Управление узлами**.
  1. Выберите нужную группу узлов в списке и нажмите кнопку **Редактировать** в правом верхнем углу.
  1. Получите список доступных версий в поле **Версия {{ k8s }}**.

- CLI

  {% include [cli-install](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  Чтобы получить список доступных версий, выполните команду:

  ```bash
  yc managed-kubernetes list-versions
  ```

- API

  Чтобы получить список доступных версий, воспользуйтесь методом [list](../../managed-kubernetes/api-ref/Version/list.md).

{% endlist %}

## Обновление кластера {#cluster-upgrade}

### Настройка автоматического обновления при создании или изменении кластера {#cluster-auto-upgrade}

Выберите режим автоматического обновления кластера и задайте нужный график обновления:

{% list tabs %}

- Консоль управления

  Настройки обновлений можно указать при [создании кластера](../../managed-kubernetes/operations/kubernetes-cluster/kubernetes-cluster-create.md) или [изменении его настроек](../../managed-kubernetes/operations/kubernetes-cluster/kubernetes-cluster-update.md).

  В поле **Частота обновлений / Отключение** выберите политику обновления кластера:
  * **Отключено** — выберите эту опцию, чтобы не использовать автоматические обновления.
  * **В любое время** — выберите эту опцию, чтобы {{ managed-k8s-name }} управлял графиком установки обновлений.
  * **Ежедневно** — укажите время начала и продолжительность обновления.
  * **В выбранные дни** — укажите день, время начала и продолжительность обновления. При необходимости выберите несколько вариантов с помощью кнопки **Добавить день и время**.

- CLI

  Укажите параметры автоматического обновления при [создании](../../managed-kubernetes/operations/kubernetes-cluster/kubernetes-cluster-create.md) или [изменении](../../managed-kubernetes/operations/kubernetes-cluster/kubernetes-cluster-update.md) кластера:

  ```bash
  {{ yc-k8s }} cluster <create или update> <идентификатор или имя кластера> \
  ...
    --auto-upgrade <true или false> \
    --anytime-maintenance-window \
    --daily-maintenance-window <значение> \
    --weekly-maintenance-window <значение>
  ```

  Где:
  * `--auto-upgrade` — выбор режима автоматического обновления кластера. Значение по умолчанию — `true` (автоматическое обновление включено).
  * `--anytime-maintenance-window` — выбор произвольного времени обновления кластера.
  * `--daily-maintenance-window` — режим обновления **Ежедневно**.

    Пример ежедневного обновления кластера в 22:00 UTC длительностью не более 10 часов:

    ```bash
    --daily-maintenance-window 'start=22:00,duration=10h'
    ```

  * `--weekly-maintenance-window` — автоматическое обновление в указанные дни.

    Пример обновления кластера по понедельникам и вторникам с 22:00 UTC, длительностью не более 10 часов:

    ```bash
    --weekly-maintenance-window 'days=[monday,tuesday],start=22:00,duration=10h'
    ```

    Чтобы указать несколько периодов обслуживания, передайте настройки каждого из них в отдельном аргументе `--weekly-maintenance-window`.

    {% note info %}

    Аргументы `--daily-maintenance-window` и `--weekly-maintenance-window` требуют аргумента `--auto-upgrade` со значением `true`. Расписание обновления не будет создано, если установить `--auto-upgrade=false`.

    {% endnote %}

  Идентификатор и имя кластера можно получить [со списком кластеров в каталоге](./kubernetes-cluster/kubernetes-cluster-list.md).

- {{ TF }}

  1. Откройте актуальный конфигурационный файл с описанием кластера.

     Как создать такой файл, см. в разделе [{#T}](../../managed-kubernetes/operations/kubernetes-cluster/kubernetes-cluster-create.md).

  1. Измените параметры автоматического обновления в описании кластера {{ k8s }}.

     {% note info %}

     Вы можете выбрать только один из режимов обновления — ежедневный или в выбранные дни. Одновременное использование режимов не допускается.

     {% endnote %}

     * Чтобы включить режим ежедневного обновления:

       ```hcl
       resource "yandex_kubernetes_cluster" "<имя кластера>" {
         name = <имя кластера>
         ...
         maintenance_policy {
           auto_upgrade = true
           maintenance_window {
             start_time = "<время начала обновления, UTC>"
             duration   = "<длительность обновления>"
           }
         }
       }
       ```

     * Чтобы включить режим обновления в выбранные дни (можно указать несколько периодов):

       ```hcl
       resource "yandex_kubernetes_cluster" "<имя кластера>" {
         name = <имя кластера>
         ...
         maintenance_policy {
           auto_upgrade = true
           maintenance_window {
             day        = "<день начала обновления, например monday>"
             start_time = "<время начала обновления, UTC>"
             duration   = "<длительность обновления>"
           }
           maintenance_window {
             day        = "<день начала обновления, например monday>"
             start_time = "<время начала обновления, UTC>"
             duration   = "<длительность обновления>"
           }
         }
       }
       ```

     * Чтобы включить режим произвольного времени обновления, не добавляйте блок параметров `maintenance_policy` в описание кластера. Если в описании кластера не указаны настройки автоматического обновления, оно будет производиться в произвольное время.

     * Чтобы отключить автоматическое обновление:

       ```hcl
       resource "yandex_kubernetes_cluster" "<имя кластера>" {
         name = "<имя кластера>"
         ...
         maintenance_policy {
           auto_upgrade = false
         }
       }
       ```

  1. Проверьте корректность конфигурационных файлов.

     {% include [terraform-validate](../../_includes/mdb/terraform/validate.md) %}

  1. Подтвердите изменение ресурсов.

     {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

  Подробнее см. в [документации провайдера {{ TF }}]({{ tf-provider-k8s-cluster }}).

- API

  Настройки автоматического обновления задаются в блоке `masterSpec.maintenancePolicy` при [создании кластера](../../managed-kubernetes/api-ref/Cluster/create.md) или [изменении его настроек](../../managed-kubernetes/api-ref/Cluster/update.md).

  Воспользуйтесь методом [update](../../managed-kubernetes/api-ref/Cluster/update.md) и передайте в запросе:
  * Идентификатор кластера в параметре `clusterId`. Чтобы узнать идентификатор кластера, [получите список кластеров в каталоге](./kubernetes-cluster/kubernetes-cluster-list.md).
  * Настройки автоматического обновления в параметре `masterSpec.maintenancePolicy`.
  * Список изменяемых настроек в параметре `updateMask`.

  {% include [updateMask warning](../../_includes/mdb/warning-default-settings.md) %}

  Чтобы отключить автоматическое обновление, передайте значение `false` в параметре `masterSpec.maintenancePolicy.autoUpgrade`.

  Для включения и настройки окна обновлений передайте одно из допустимых значений параметра `maintenanceWindow`:
  * Чтобы кластер обновлялся в произвольное время, передайте значение `"anytime": {}`.
  * Чтобы настроить ежедневные обновления, добавьте блок `dailyMaintenanceWindow`:

    ```json
    "dailyMaintenanceWindow": {
      "startTime": {
        "hours": "<час начала обновления, UTC>",
        "minutes": "<минута начала обновления>",
        "seconds": "<секунда начала обновления>",
        "nanos": "<тысячная доля секунды начала обновления>"
      },
      "duration": "<длительность периода обновления, в часах>"
    }
    ```

  * Чтобы настроить обновление в выбранные дни, добавьте блок `weeklyMaintenanceWindow`:

    ```json
    "weeklyMaintenanceWindow": {
      "daysOfWeek": [
        {
          "days": [
            "<список дней, например: monday, tuesday>"
          ],
          "startTime": {
            "hours": "<час начала обновления, UTC>",
            "minutes": "<минута начала обновления>",
            "seconds": "<секунда начала обновления>",
            "nanos": "<тысячная доля секунды начала обновления>"
          },
          "duration": "<длительность периода обновления, в часах>"
        }
      ]
    }
    ```

{% endlist %}

### Ручное обновление версии кластера {#cluster-manual-upgrade}

При необходимости обновите версию кластера {{ k8s }} вручную. Для обновления доступна только следующая мажорная версия относительно текущей. Обновление до более новых версий следует производить в несколько этапов, например: 1.17 → 1.18 → 1.19.

{% list tabs %}

- Консоль управления

  1. Перейдите на страницу каталога и выберите сервис **{{ managed-k8s-name }}**.
  1. Нажмите на имя нужного кластера {{ k8s }}.
  1. Нажмите кнопку **Редактировать** в правом верхнем углу.
  1. В поле **Версия {{ k8s }}** выберите вариант **Обновить до версии <номер версии>**.
  1. Нажмите кнопку **Сохранить изменения**.

- CLI

  Укажите новую версию {{ k8s }} в значении аргумента `--version`:

  ```bash
  {{ yc-k8s }} cluster update <идентификатор или имя кластера> \
    --version <новая версия>
  ```

  Идентификатор и имя кластера можно [получить со списком кластеров в каталоге](./kubernetes-cluster/kubernetes-cluster-list.md).

- {{ TF }}

  1. Откройте актуальный конфигурационный файл с описанием кластера.

     О том, как создать такой файл, см. в разделе [{#T}](../../managed-kubernetes/operations/kubernetes-cluster/kubernetes-cluster-create.md).

  1. Измените версию в описании кластера {{ k8s }}:

     ```hcl
     resource "yandex_kubernetes_cluster" "<имя кластера>" {
       name = <имя кластера>
       ...
       version = "<новая версия>"
     }
     ```

  1. Проверьте корректность конфигурационных файлов.

     {% include [terraform-validate](../../_includes/mdb/terraform/validate.md) %}

  1. Подтвердите изменение ресурсов.

     {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

  Подробнее см. в [документации провайдера {{ TF }}]({{ tf-provider-k8s-cluster}}).

- API

  Воспользуйтесь методом [update](../../managed-kubernetes/api-ref/Cluster/update.md) и передайте в запросе:
  * Идентификатор кластера в параметре `clusterId`. Чтобы узнать идентификатор кластера, [получите список кластеров в каталоге](./kubernetes-cluster/kubernetes-cluster-list.md).
  * Нужную версию {{ k8s }} в параметре `masterSpec.version.version`.
  * Список изменяемых настроек в параметре `updateMask`.

  {% include [updateMask warning](../../_includes/mdb/warning-default-settings.md) %}

{% endlist %}

## Обновление группы узлов {#node-group-upgrade}

### Настройка автоматического обновления группы узлов {#node-group-auto-upgrade}

Выберите режим автоматического обновления группы узлов и задайте нужный график обновления:

{% list tabs %}

- Консоль управления

  Настройки обновлений можно указать при [создании группы узлов](../../managed-kubernetes/operations/node-group/node-group-create.md) или [изменении ее настроек](../../managed-kubernetes/operations/node-group/node-group-update.md).

  В поле **В процессе создания и обновления разрешено** укажите настройки масштабирования группы узлов:
  * **Расширение размера группы, макс.** — задайте максимальное количество виртуальных машин, на которое можно превысить размер группы при ее обновлении.
  * **Уменьшение размера группы, макс.** — задайте максимальное количество ВМ, на которое можно уменьшить размер группы при ее обновлении.

  В поле **Частота обновлений / Отключение** выберите политику обновления группы узлов:
  * **Отключено** — выберите эту опцию, чтобы не использовать автоматические обновления.
  * **В любое время** — выберите эту опцию, чтобы {{ managed-k8s-name }} управлял графиком установки обновлений.
  * **Ежедневно** — укажите время начала и продолжительность обновления.
  * **В выбранные дни** — укажите день, время начала и продолжительность обновления. При необходимости выберите несколько вариантов с помощью кнопки **Добавить день и время**.

- CLI

  Укажите параметры автоматического обновления при [создании](../../managed-kubernetes/operations/node-group/node-group-create.md) или [изменении](../../managed-kubernetes/operations/node-group/node-group-update.md) группы узлов:

  ```bash
  {{ yc-k8s }} node-group <create или update> <идентификатор или имя группы узлов> \
  ...
    --max-expansion <значение> \
    --max-unavailable <значение> \
    --auto-upgrade <true или false> \
    --auto-repair <true или false> \
    --anytime-maintenance-window \
    --daily-maintenance-window <значение> \
    --weekly-maintenance-window <значение>
  ```

  Где:
  * `--max-expansion` — максимальное количество ВМ, на которое можно превысить размер группы при ее обновлении.
  * `--max-unavailable` — максимальное количество ВМ, на которое можно уменьшить размер группы при ее обновлении.

    {% note info %}

    Параметры `--max-expansion` и `--max-unavailable` следует использовать совместно.

    {% endnote %}

  * `--auto-upgrade` — выбор режима автоматического обновления группы узлов. Значение по умолчанию — `true` (автоматическое обновление включено).
  * `--auto-repair` — выбор режима пересоздания сбойных узлов.

    Режим `--auto-repair` находится на стадии [Preview](../../overview/concepts/launch-stages.md).
  * `--anytime-maintenance-window` — выбор произвольного времени обновления группы узлов.
  * `--daily-maintenance-window` — режим обновления **Ежедневно**.

    Пример ежедневного обновления группы узлов в 22:00 UTC длительностью не более 10 часов:

    ```bash
    --daily-maintenance-window 'start=22:00,duration=10h'
    ```

  * `--weekly-maintenance-window` — автоматическое обновление в указанные дни.

    Пример обновления группы узлов по понедельникам и вторникам с 22:00 UTC, длительностью 10 часов:

    ```bash
    --weekly-maintenance-window 'days=[monday,tuesday],start=22:00,duration=10h'
    ```

    Чтобы указать несколько периодов обслуживания, передайте настройки каждого из них в отдельном аргументе `--weekly-maintenance-window`.

    {% note info %}

    Аргументы `--daily-maintenance-window` и `--weekly-maintenance-window` требуют аргумента `--auto-upgrade` со значением `true`.
        Расписание обновления не будет создано, если установить `--auto-upgrade=false`.

    {% endnote %}

  Идентификатор и имя группы узлов можно [получить со списком групп в кластере](./node-group/node-group-list.md).

- {{ TF }}

  1. Откройте актуальный конфигурационный файл с описанием группы узлов.

     О том, как создать такой файл, см. в разделе [{#T}](../../managed-kubernetes/operations/node-group/node-group-create.md).

  1. Измените параметры автоматического обновления в описании группы узлов.

     {% note info %}

     Вы можете выбрать только один из режимов обновления — ежедневный или в выбранные дни. Одновременное использование режимов не допускается.

     {% endnote %}

     * Чтобы включить режим ежедневного обновления:

       ```hcl
       resource "yandex_kubernetes_node_group" "<имя группы узлов>" {
         name = <имя группы узлов>
         ...
         maintenance_policy {
           auto_upgrade = true
           maintenance_window {
             start_time = "<время начала обновления, UTC>"
             duration   = "<длительность обновления>"
           }
         }
       }
       ```

     * Чтобы включить режим обновления в выбранные дни (можно указать несколько периодов):

       ```hcl
       resource "yandex_kubernetes_node_group" "<имя группы узлов>" {
         name = <имя группы узлов>
         ...
         maintenance_policy {
           auto_upgrade = true
           maintenance_window {
             day        = "<день начала обновления, например monday>"
             start_time = "<время начала обновления, UTC>"
             duration   = "<длительность обновления>"
           }
           maintenance_window {
             day        = "<день начала обновления, например monday>"
             start_time = "<время начала обновления, UTC>"
             duration   = "<длительность обновления>"
           }
         }
       }
       ```

     * Чтобы включить режим произвольного времени обновления, не добавляйте блок параметров `maintenance_policy` в описание группы узлов. Если в описании группы узлов не указаны настройки автоматического обновления, оно будет производиться в произвольное время.

     * Чтобы задать настройки масштабирования группы узлов при обновлении:

       ```hcl
       resource "yandex_kubernetes_node_group" "<имя группы узлов>" {
         name = <имя группы узлов>
         ...
         deploy_policy {
           max_expansion   = <максимальное количество ВМ, на которое можно превысить размер группы>
           max_unavailable = <максимальное количество ВМ, на которое можно уменьшить размер группы>
         }
       }
       ```

       {% note info %}

       Параметры `max_expansion` и `max_unavailable` следует использовать совместно.

       {% endnote %}

     * Чтобы отключить автоматическое обновление:

       ```hcl
       resource "yandex_kubernetes_node_group" "<имя группы узлов>" {
         name = "<имя группы узлов>"
         ...
         maintenance_policy {
           auto_upgrade = false
         }
       }
       ```

  1. Проверьте корректность конфигурационных файлов.

     {% include [terraform-validate](../../_includes/mdb/terraform/validate.md) %}

  1. Подтвердите изменение ресурсов.

     {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

  Подробнее см. в [документации провайдера {{ TF }}]({{ tf-provider-k8s-node-group }}).

- API

  Настройки автоматического обновления задаются в блоке `maintenancePolicy` при [создании группы узлов](../../managed-kubernetes/api-ref/NodeGroup/create.md) или [изменении ее настроек](../../managed-kubernetes/api-ref/NodeGroup/update.md).

  Воспользуйтесь методом [update](../../managed-kubernetes/api-ref/NodeGroup/update.md) и передайте в запросе:
  * Идентификатор группы узлов в параметре `nodeGroupId`. Чтобы узнать идентификатор группы, [получите список групп в кластере](./node-group/node-group-list.md).
  * Настройки автоматического обновления в параметре `maintenancePolicy`.
  * Список изменяемых настроек в параметре `updateMask`.

  {% include [updateMask warning](../../_includes/mdb/warning-default-settings.md) %}

  Чтобы отключить автоматическое обновление, передайте значение `false` в параметре `maintenancePolicy.autoUpgrade`.

  Для включения и настройки окна обновлений передайте одно из допустимых значений параметра `maintenanceWindow`:
  * Чтобы группа узлов обновлялась в произвольное время, передайте значение `"anytime": {}`.
  * Чтобы настроить ежедневные обновления, добавьте блок `dailyMaintenanceWindow`:

    ```json
    "dailyMaintenanceWindow": {
      "startTime": {
        "hours": "<час начала обновления, UTC>",
        "minutes": "<минута начала обновления>",
        "seconds": "<секунда начала обновления>",
        "nanos": "<тысячная доля секунды начала обновления>"
      },
      "duration": "<длительность периода обновления, в часах>"
    }
    ```

  * Чтобы настроить обновление в выбранные дни, добавьте блок `weeklyMaintenanceWindow`:

    ```json
    "weeklyMaintenanceWindow": {
      "daysOfWeek": [
        {
          "days": [
            "<список дней, например: monday, tuesday>"
          ],
          "startTime": {
            "hours": "<час начала обновления, UTC>",
            "minutes": "<минута начала обновления>",
            "seconds": "<секунда начала обновления>",
            "nanos": "<тысячная доля секунды начала обновления>"
          },
          "duration": "<длительность периода обновления, в часах>"
        }
      ]
    }
    ```

  Для настройки масштабирования группы узлов добавьте блок `deployPolicy`:

  ```json
  "deployPolicy": {
    "maxUnavailable": "<максимальное количество ВМ, на которое можно превысить размер группы>",
    "maxExpansion": "<максимальное количество ВМ, на которое можно уменьшить размер группы>"
  }
  ```

{% endlist %}

### Ручное обновление версии группы узлов {#node-group-manual-upgrade}

При необходимости обновите версию группы узлов {{ k8s }} вручную. Для обновления доступна только следующая мажорная версия относительно текущей. Обновление до более новых версий следует производить в несколько этапов, например: 1.17 → 1.18 → 1.19.

{% note warning %}

Обновите версию кластера перед обновлением группы узлов.

{% endnote %}

{% list tabs %}

- Консоль управления

  1. Перейдите на страницу каталога и выберите сервис **{{ managed-k8s-name }}**.
  1. Нажмите на имя нужного кластера {{ k8s }}.
  1. Перейдите на вкладку **Управление узлами**.
  1. Выберите нужную группу узлов в списке.
  1. Нажмите кнопку **Редактировать** в правом верхнем углу.
  1. В поле **Версия {{ k8s }}** выберите вариант **Обновить до версии <номер версии>**.
  1. Нажмите кнопку **Сохранить изменения**.

- CLI

  Укажите параметры автоматического обновления:

  ```bash
  {{ yc-k8s }} node-group update <идентификатор или имя группы узлов> \
  ...
    --version <новая версия>
  ```

  Идентификатор и имя группы узлов можно [получить со списком групп в кластере](./node-group/node-group-list.md).

- {{ TF }}

  1. Откройте актуальный конфигурационный файл с описанием группы узлов.

     О том, как создать такой файл, см. в разделе [{#T}](../../managed-kubernetes/operations/node-group/node-group-create.md).

  1. Измените версию в описании группы узлов:

     ```hcl
     resource "yandex_kubernetes_node_group" "<имя группы узлов>" {
       name = <имя группы узлов>
       ...
       version = "<новая версия>"
     }
     ```

  1. Проверьте корректность конфигурационных файлов.

     {% include [terraform-validate](../../_includes/mdb/terraform/validate.md) %}

  1. Подтвердите изменение ресурсов.

     {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

  Подробнее см. в [документации провайдера {{ TF }}]({{ tf-provider-k8s-node-group }}).

- API

  Воспользуйтесь методом [update](../../managed-kubernetes/api-ref/NodeGroup/update.md) и передайте в запросе:
  * Идентификатор группы узлов в параметре `nodeGroupId`. Чтобы узнать идентификатор группы, [получите список групп в кластере](./node-group/node-group-list.md).
  * Нужную версию {{ k8s }} в параметре `version.version`.
  * Список изменяемых настроек в параметре `updateMask`.

  {% include [updateMask warning](../../_includes/mdb/warning-default-settings.md) %}

{% endlist %}

## Обновление компонентов {{ k8s }} без изменения версии {#latest-revision}

Для кластера {{ managed-k8s-name }} и группы узлов доступно обновление в рамках одной версии {{ k8s }}. При установке обновления мажорная версия {{ k8s }} не меняется.

При таком обновлении возможна:
* установка новых пакетов;
* обновление образа {{ k8s }};
* обновление минорной версии {{ k8s }}.

Кластер и группы узлов будут обновлены, если в их настройках включен любой из вариантов автоматического обновления.

### Обновление для кластера с отключенным автоматическим обновлением {#cluster-latest-revision}

{% list tabs %}

- Консоль управления

  1. Перейдите на страницу каталога и выберите сервис **{{ managed-k8s-name }}**.
  1. Нажмите на имя нужного кластера {{ k8s }}.
  1. Нажмите кнопку **Редактировать** в правом верхнем углу.
  1. В поле **Версия Kubernetes** выберите **Получить последние улучшения и исправления для версии...**.
  1. Нажмите кнопку **Сохранить изменения**.

- CLI

  Выполните обновление кластера:

  ```bash
  {{ yc-k8s }} cluster update <идентификатор или имя кластера> \
    --latest-revision
  ```

  Идентификатор и имя кластера можно [получить со списком кластеров в каталоге](./kubernetes-cluster/kubernetes-cluster-list.md).

- API

  Воспользуйтесь методом [update](../../managed-kubernetes/api-ref/Cluster/update.md) и передайте в запросе:
  * Идентификатор кластера в параметре `clusterId`. Чтобы узнать идентификатор кластера, [получите список кластеров в каталоге](./kubernetes-cluster/kubernetes-cluster-list.md).
  * Значение `true` в параметре `masterSpec.version.version`.
  * Список изменяемых настроек в параметре `updateMask`.

  {% include [updateMask warning](../../_includes/mdb/warning-default-settings.md) %}

{% endlist %}

### Обновление для группы узлов с отключенным автоматическим обновлением {#node-group-latest-revision}

{% list tabs %}

- Консоль управления

  1. Перейдите на страницу каталога и выберите сервис **{{ managed-k8s-name }}**.
  1. Нажмите на имя нужного кластера {{ k8s }}.
  1. Перейдите на вкладку **Управление узлами**.
  1. Выберите нужную группу узлов в списке.
  1. Нажмите кнопку **Редактировать** в правом верхнем углу.
  1. В поле **Версия Kubernetes** выберите **Получить последние улучшения и исправления для версии...**.
  1. Нажмите кнопку **Сохранить изменения**.

- CLI

  Выполните обновление группы узлов:

  ```bash
  {{ yc-k8s }} node-group update <идентификатор или имя группы узлов> \
    --latest-revision
  ```

  Идентификатор и имя группы узлов можно [получить со списком групп в кластере](./node-group/node-group-list.md).

- API

  Воспользуйтесь методом [update](../../managed-kubernetes/api-ref/NodeGroup/update.md) и передайте в запросе:
  * Идентификатор группы узлов в параметре `nodeGroupId`. Чтобы узнать идентификатор группы, [получите список групп в кластере](./node-group/node-group-list.md).
  * Значение `true` в параметре `version.latestRevision`.
  * Список изменяемых настроек в параметре `updateMask`.

  {% include [updateMask warning](../../_includes/mdb/warning-default-settings.md) %}

{% endlist %}