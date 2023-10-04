# Подключение виртуальных машин {{ compute-name }} к {{ backup-name }}

Если вы хотите создавать резервные копии [ВМ](../../compute/concepts/vm.md) [{{ compute-full-name }}](../../compute/) в сервисе {{ backup-name }}, ее нужно подключить к сервису и корректно настроить.

Чтобы ВМ можно было подключить к {{ backup-name }}, на ней должна быть установлена одна из [поддерживаемых операционных систем](#os): Linux (CentOS, Ubuntu) или Windows Server. Вы можете подключить существующие ВМ на Linux и Windows Server, а также создать ВМ Linux с подключением к {{ backup-name }}. Подробнее о подключении см. в [инструкциях](../operations/index.md#connect-vm).

Чтобы подключение работало корректно, к ВМ нужно привязать [сервисный аккаунт](#sa) с ролью `backup.editor`, [публичный IP-адрес](#public-ip-address) и [группу безопасности](#security-groups) с определенными правилами (если эта функциональность включена в вашем [облаке](../../resource-manager/concepts/resources-hierarchy.md#cloud)).

После подключения к {{ backup-name }} и настройки ВМ нужно привязать к [политике резервного копирования](policy.md).

## Поддерживаемые операционные системы {#os}

{{ backup-name }} поддерживает резервное копирование ВМ со следующими операционными системами:
* Ubuntu версии 20.04 и ниже.
* CentOS 7.
* Windows Server 2019 и 2022.

Ubuntu или CentOS должна быть установлена из публичного образа (продукта {{ marketplace-full-name }}). При создании ВМ можно выбрать ОС напрямую либо использовать [образ](../../compute/concepts/image.md) или [снимок диска](../../compute/concepts/snapshot.md) с другой ВМ, если ОС на нее тоже устанавливалась из публичного образа.

Позже в {{ backup-name }} будет поддержана Ubuntu 22.04.

## Сервисный аккаунт {#sa}

[Сервисный аккаунт](../../iam/concepts/users/service-accounts.md) — специальный аккаунт, от имени которого копии ВМ создаются и загружаются в хранилище {{ backup-name }}.

Когда вы создаете ВМ, для которой хотите настроить резервное копирование в {{ backup-name }}, к ВМ нужно привязать сервисный аккаунт с [ролью `backup.editor`](../security/index.md).

Вы можете [назначить роль](../../iam/operations/sa/assign-role-for-sa.md) существующему сервисному аккаунту или [создать](../../iam/operations/sa/create.md) сервисный аккаунт с нужными ролями.

## Публичный IP-адрес {#public-ip-address}

ВМ, подключенная к {{ backup-name }}, должна обмениваться данными с серверами [провайдера резервного копирования](index.md#providers). Для этого при создании ВМ к ней нужно привязать [публичный IP-адрес](../../vpc/concepts/address.md#public-addresses) [{{ vpc-full-name }}](../../vpc/).

Использование публичного IP-адреса тарифицируется. Подробнее см. в разделе [{#T}](../../vpc/pricing.md).

## Группы безопасности {#security-groups} 

Группы безопасности позволяют управлять доступом ВМ к ресурсам {{ yandex-cloud }} или ресурсам в интернете.

Чтобы ВМ могла обмениваться данными с серверами провайдера резервного копирования, при ее создании и подключении к {{ backup-name }} выберите группу безопасности со следующими (или более широкими) правилами:

{% list tabs %}

- Исходящий трафик

  {{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-port-range }} | {{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-protocol }} | {{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-destination }} | {{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-cidr-blocks }}
  --- | --- | --- | ---
  `80` | `{{ ui-key.yacloud.common.label_tcp }}` | `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}` | `213.180.193.0/24`
  `80` | `{{ ui-key.yacloud.common.label_tcp }}` | `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}` | `213.180.204.0/24`
  `443` | `{{ ui-key.yacloud.common.label_tcp }}` | `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}` | `84.47.172.0/24`
  `443` | `{{ ui-key.yacloud.common.label_tcp }}` | `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}` | `84.201.181.0/24`
  `443` | `{{ ui-key.yacloud.common.label_tcp }}` | `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}` | `178.176.128.0/24`
  `443` | `{{ ui-key.yacloud.common.label_tcp }}` | `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}` | `193.32.199.0/24`
  `443` | `{{ ui-key.yacloud.common.label_tcp }}` | `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}` | `213.180.193.0/24`
  `443` | `{{ ui-key.yacloud.common.label_tcp }}` | `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}` | `213.180.204.0/24`
  `7770-7800` | `{{ ui-key.yacloud.common.label_tcp }}` | `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}` | `84.47.172.0/24`
  `8443` | `{{ ui-key.yacloud.common.label_tcp }}` | `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}` | `84.47.172.0/24`
  `44445` | `{{ ui-key.yacloud.common.label_tcp }}` | `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}` | `51.250.1.0/24`

{% endlist %}

Вы можете [добавить правила](../../vpc/operations/security-group-add-rule.md) в существующую группу безопасности или [создать](../../vpc/operations/security-group-create.md) группу с нужными правилами.