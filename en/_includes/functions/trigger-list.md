{% list tabs %}

- Management console

   1. In the [management console]({{ link-console-main }}), go to the folder where you want to view a list of triggers.
   1. Select **{{ ui-key.yacloud.iam.folder.dashboard.label_serverless-functions }}**.
   1. In the left-hand panel, select ![image](../../_assets/functions/triggers.svg) **{{ ui-key.yacloud.serverless-functions.switch_list-triggers }}**.

- CLI

   {% include [cli-install](../cli-install.md) %}

   {% include [default-catalogue](../default-catalogue.md) %}

   Get a list of triggers:

   ```bash
   yc serverless trigger list
   ```

   Result:

   ```text
   +----------------------+------------+----------------------+
   |          ID          |    NAME    |      FOLDER ID       |
   +----------------------+------------+----------------------+
   | dd0gj5tsj2********** | my-trigger | aoek49ghmk********** |
   +----------------------+------------+----------------------+
   ```

- API

   To get a list of triggers, use the [list](../../functions/triggers/api-ref/Trigger/list.md) REST API method for the [Trigger](../../functions/triggers/api-ref/Trigger/index.md) resource or the [TriggerService/List](../../functions/triggers/api-ref/grpc/trigger_service.md#List) gRPC API call.

{% endlist %}
