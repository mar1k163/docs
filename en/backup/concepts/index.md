# {{ backup-full-name }} overview

{{ backup-full-name }} is a service for creating backups and restoring {{ yandex-cloud }} resources and their data. You can back up and recover [{{ compute-name }} VMs](../../compute/concepts/vm.md) running Ubuntu, CentOS, and Windows Server.

VM backups are crash-consistent: they save both data stored on disks and intermediate data of running applications (in-memory data and active reads and writes to disks). This approach allows you to resume applications that were running at the time of backup creation directly after restoring a VM. This is important for VMs that are part of data storage systems, such as when a DBMS is running on the VM.

{{ backup-name }} can create [full and incremental backups](backup.md#types). A full backup stores complete data of a VM. This is reliable, but full backups take a lot of space and restoring VMs from them is more time-consuming. Incremental backups only contain data that differs from that of previous VM backups. This allows you to save space and time for VM recovery. However, if any backup in the chain is corrupt, you will not be able to recover your VM.

With {{ backup-name }}, you can also restore individual files and directories from a backup to any VM connected to the service. For more information, see [File-by-file recovery](backup.md#file-by-file).

To enable backups for your VM, you need to connect it to the service and configure it. For more information, see [{#T}](vm-connection.md).

Once you have connected your instance to {{ backup-name }} and set it up, you need to link it to a [backup policy](policy.md) that is, to a {{ backup-name }} resource. The policy includes settings for creating and storing backups. You can link your instances to default policies or create custom policies.

The service has two types of restrictions: [quotas and limits](limits.md).

## Comparison with disk snapshots in {{ compute-name }} {#snapshot-comparison}

| | Disk snapshots in {{ compute-name }} | {{ backup-name }} |
| --- | --- | --- |
| **Granularity** | Disk | VMs with all disks.<br>We may also add support for VM application backups. |
| **Disk types** | Network SSD and HDD | Any:<br>Network SSD and HDD,<br>non-replicated disks,<br>and local disks on [dedicated hosts](../../compute/concepts/dedicated-host.md) |
| **Application awareness** | No | To be implemented |
| **Pricing** | Per disk | Per VM |


## Service activation and backup provider {#providers}

{{ backup-name }} is enabled by a _backup provider_. Currently, the service only has one provider available: Cyberprotect.

If you got access to {{ backup-name }}, to get started, you need to activate the service, that is, to connect to a backup provider. You can activate the service directly on its page in the management console (see the [getting started tutorial](../quickstart.md)) or [connect](vm-connection.md) your first VM to {{ backup-name }}: the service will activate automatically in this case.

The minimum folder role required for activating the service is `backup.editor` (see the [role description](../security/index.md#backup-editor)).
