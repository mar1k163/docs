# Creating a timer that invokes a container

Create a [timer](../concepts/trigger/timer.md) that invokes a {{ serverless-containers-name }} [container](../concepts/container.md) every minute.

## Getting started {#before-you-begin}

1. [Create a container](../operations/index.md#create-container) to be invoked by your timer. [Application and Dockerfile examples](container.md#examples).
1. [Create a service account](../../iam/operations/sa/create.md) that will be used to invoke the container and assign it the `serverless-containers.containerInvoker` role.

## Create a timer {#timer-create}

{% include [trigger-time](../../_includes/functions/trigger-time.md) %}

{% list tabs %}

- Management console

   1. In the [management console]({{ link-console-main }}), select the folder where you want to create a timer.

   1. Select **{{ ui-key.yacloud.iam.folder.dashboard.label_serverless-containers }}**.

   1. In the left-hand panel, select ![image](../../_assets/functions/triggers.svg) **{{ ui-key.yacloud.serverless-functions.switch_list-triggers }}**.

   1. Click **{{ ui-key.yacloud.serverless-functions.triggers.list.button_create }}**.

   1. Under **{{ ui-key.yacloud.serverless-functions.triggers.form.section_base }}**:

      * Enter the trigger name: `timer`.
      * In the **{{ ui-key.yacloud.serverless-functions.triggers.form.field_type }}** field, select `{{ ui-key.yacloud.serverless-functions.triggers.form.label_timer }}`.
      * In the **{{ ui-key.yacloud.serverless-functions.triggers.form.field_invoke }}** field, select `{{ ui-key.yacloud.serverless-functions.triggers.form.label_container }}`.

   1. Under **{{ ui-key.yacloud.serverless-functions.triggers.form.section_timer }}**, enter `* * ? * * *` or select `{{ ui-key.yacloud.common.button_cron-1min }}`.

   1. {% include [container-settings](../../_includes/serverless-containers/container-settings.md) %}

   1. Click **{{ ui-key.yacloud.serverless-functions.triggers.form.button_create-trigger }}**.

- CLI

   {% include [cli-install](../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../_includes/default-catalogue.md) %}

   To create a trigger that invokes a container every minute, run this command:

   ```bash
   yc serverless trigger create timer \
     --name timer \
     --cron-expression '* * ? * * *' \
     --invoke-container-id <container_ID> \
     --invoke-container-service-account-id <service_account_ID>
   ```

   Where:

   * `--name`: Timer name.
   * `--cron-expression`: Container invocation schedule in [cron expression](../concepts/trigger/timer.md#cron-expression) format.
   * `--invoke-container-id`: Container ID.
   * `--invoke-container-service-account-id`: ID of the service account with rights to invoke the container.

   Result:

   ```text
   id: a1sp9tj1jhar********
   folder_id: b1g4j6o69kqj********
   created_at: "2023-03-16T17:45:23.145213360Z"
   name: timer
   rule:
     timer:
       cron_expression: '* * ? * * *'
       invoke_container_with_retry:
         container_id: bbavvhra8ekc********
         service_account_id: aje1ki4ae68u********
   status: ACTIVE
   ```

- API

   You can create a timer using the [create](../triggers/api-ref/Trigger/create.md) API method.

{% endlist %}

## Check the result {#check-result}

To make sure the timer is running properly, view the container logs. They should show that the container is invoked every minute.

1. In the [management console]({{ link-console-main }}), select the folder with your container.
1. Select **{{ ui-key.yacloud.iam.folder.dashboard.label_serverless-containers }}**.
1. Click the container to view its invocation log.
1. In the window that opens, go to **{{ ui-key.yacloud.common.logs }}** and specify the period to view them for. The default period is 1 hour.

## What's next {#what-is-next}

* [Check out the concepts](../concepts/trigger/index.md).
* [Learn how to create other triggers](../operations/index.md#create-trigger).