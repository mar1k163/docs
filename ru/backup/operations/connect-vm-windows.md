# Подключить виртуальную машину Windows Server к {{ backup-name }}

Сервис {{ backup-name }} поддерживает резервное копирование [виртуальных машин {{ compute-name }}](../../compute/concepts/vm.md) с операционными системами Windows Server 2019 и 2022. Подробнее см. в разделе [{#T}](../concepts/vm-connection.md#os).

{% include [vm-prereqs-note](../../_includes/backup/vm-prereqs-note.md) %}

Чтобы подключить ВМ Windows к {{ backup-name }}:

1. [Создайте](../../iam/operations/sa/create.md) сервисный аккаунт с [ролью](../../iam/concepts/access-control/roles.md#backup-editor) `backup.editor`.
1. [Подключите](../../compute/operations/vm-control/vm-update.md) к ВМ сервисный аккаунт, созданный ранее.
1. Если у ВМ нет публичного IP-адреса, [подключите](../../compute/operations/vm-control/vm-attach-public-ip.md) его.
1. [Настройте](../../vpc/operations/security-group-add-rule.md) в группе безопасности [правила для работы с {{ backup-name }}](../concepts/vm-connection.md#security-groups).
1. [Подключитесь к ВМ по RDP](../../compute/operations/vm-connect/rdp.md).
1. Запустите Windows PowerShell.
1. Выполните следующую команду:

   ```powershell
   . { iwr -useb https://{{ s3-storage-host }}/backup-distributions/agent_installer.ps1 } | iex
   ```
   
   Результат:

   ```text
   ...
   Backup agent installation complete after 190 s!
   ```

После этого ВМ можно привязать к [политике резервного копирования](../concepts/policy.md).


#### См. также {#see-also}

* [{#T}](create-vm.md)
* [Привязать виртуальную машину к политике резервного копирования](./policy-vm/update.md#update-vm-list)
* [{#T}](./backup-vm/recover.md)
* [{#T}](./backup-vm/delete.md)
* [{#T}](./policy-vm/create.md)