# Install to AWS ECS

:warning: **SignalFx Smart Agent is deprecated. For details, see the [Deprecation Notice](./smartagent-deprecation-notice.md)** :warning:

Deploy the SignalFx Smart Agent to an AWS ECS instance using a SignalFx
configuration script, and run the Smart Agent as a Daemon service in an
EC2 ECS cluster.

## Prerequisites

* Access to the AWS ECS web console or the AWS CLI.
  To learn more, refer to the AWS ECS documentation.

* SignalFx access token. See [Smart Agent Access Token](../../../_sidebars-and-includes/smart-agent-access-token.html).
* Your SignalFx realm. See [Realms](../../../_sidebars-and-includes/smart-agent-realm-note.html).

## Configure the Smart Agent for AWS ECS

Configure the Smart Agent for AWS ECS by following these steps:

1. [Edit the main configuration file](#edit-the-main-configuration-file)
2. Optional: [Edit additional options](#edit-additional-options)
   Create a Smart Agent task definition for AWS ECS.
3. Download the [signalfx-agent-task.json](https://github.com/signalfx/signalfx-agent/tree/main/deployments/ecs/signalfx-agent-task.json) file.

### Edit signalfx-agent-task.json

Edit the signalfx-agent-task.json file and make these replacements:

| Text                    | Replacement                                                                                                                                                                                                                                                     |
|:------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `MY_ACCESS_TOKEN`       | Your SignalFx access token. See [Prerequisites](#prerequisites).                                                                                                                                                                                                |
| `MY_INGEST_URL`         | `https://ingest.<REALM>.signalfx.com`. Replace `<REALM>` with your realm. See [Prerequisites](#prerequisites)                                                                                                                                                   |
| `MY_API_URL`            | `https://api.<REALM>.signalfx.com`. Replace `<REALM>` with your realm. See [Prerequisites](#prerequisites)                                                                                                                                                      |
| `MY_TRACE_ENDPOINT_URL` | `null`. The Smart Agent only uses the property for Microservices APM. To learn more about the Smart Agent and Microservice APM, see [Deploy a SignalFx Smart Agent for µAPM](https://docs.splunk.com/observability/gdi/smart-agent/smart-agent-resources.html). |

### Edit additional options

By default, the main configuration in signalfx-task-agent.json uses additional options in the
agent.yaml file by pulling them from GitHub using `curl`. These options control how the Smart Agent
interacts with ECS. For example, the `observer` option specifies which features the Smart Agent
uses to discover running services.

To change additional configuration options, follow these steps:

1. Download the [agent.yaml](https://github.com/signalfx/signalfx-agent/blob/main/deployments/ecs/agent.yaml) file.
2. Copy agent.yaml to a new .yaml file with a custom name.
3. In the signalfx-agent-task.json file, change the environment variable `CONFIG_URL` to the URL of your
   custom version of agent.yaml. The URL must be accessible from your ECS cluster.
4. Deploy the custom .yaml file to your ECS cluster.

To learn more, see [agent.yaml](https://github.com/signalfx/signalfx-agent/blob/main/deployments/ecs/agent.yaml).

## Deploy the Smart Agent task definition to ECS

After you finish editing the configuration files, continue with these steps:

* If you already use the AWS ECS web console, use it to create the task definition
* If you're not using the web console, use the command-line interface to create the task definition

### AWS ECS web console

1. Start the web console and navigate to the **Task Definitions** tab.
2. Click **Create new Task Definition**.
3. Click **EC2**, then click **Next step**.
4. Click **Configure via JSON**.
5. Open the signalfx-agent-task.json file, copy the contents, paste the contents into the text box, and click **Save**.
6. Click **Update** and then **Create**.

### AWS command-line interface

Create the agent task definition using the AWS command-line interface tool by entering the following command:

```
aws ecs register-task-definition --cli-input-json file:///<path_to_signalfx-agent-task.json>
```

## Installation

Run the Smart Agent as a Daemon service in an ECS cluster.

To create this service in the ECS web admin console:

1. In the console, go to your cluster.
2. Click the **Services** tab.
3. Click **Create**.
4. Select the following options:
   - Launch Type: EC2
   - Task Definition (Family): signalfx-agent
   - Task Definition (Revision): <latest_revision>

     If you haven't created a definition before, set this option to **1**.

   - Service Name: signalfx-agent
   - Service type: DAEMON
   - Use the defaults for the other options
5. Click **Next step**.
6. Use the defaults for all options and click **Next step**.
7. Use the defaults for all options and click **Next step**.
8. Click **Create Service**. AWS deploys the Smart Agent to each node in  the ECS cluster.

## Verify the Smart Agent

* For infrastructure monitoring, perform these steps:
  1. In SignalFx, open the **Infrastructure** built-in dashboard
  2. In the override bar at the top of the back, select **Choose a
     host**. Select one of your nodes from the dropdown list.

  The charts display metrics from the infrastructure for that node.

  To learn more, see [Built-In Dashboards and Charts](https://docs.splunk.com/observability/data-visualization/dashboards/built-in-dashboards.html).

* For Kubernetes monitoring, perform these steps:
  1. In SignalFx, from the main menu select **Infrastructure** > **Kubernetes Navigator** > **Cluster map**.
  2. The map displays all the clusters running the Smart Agent
  3. Click the magnification icon to view the nodes in a cluster.

  The detail pane on the right hand side of the page displays details of that cluster and nodes.

  To learn more, see [Getting Around the Kubernetes Navigator](https://docs.splunk.com/observability/infrastructure/monitor/k8s.html).

* For APM monitoring, learn how to install, configure, and verify the Smart Agent for Microservices APM (**µAPM**). See
[Get started with SignalFx µAPM](https://docs.splunk.com/observability/apm/intro-to-apm.html#nav-Introduction-to-Splunk-APM).

### Uninstall the Smart Agent

- To deregister the `signalfx-agent` task definitions, see [Deregistering task definition revisions](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deregister-task-definition.html).

- To delete the `signalfx-agent` service, see [Deleting a service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/delete-service.html).
