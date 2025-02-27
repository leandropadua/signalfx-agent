# Install using kubectl

:warning: **SignalFx Smart Agent is deprecated. For details, see the [Deprecation Notice](./smartagent-deprecation-notice.md)** :warning:

Use kubectl to install the Smart Agent to Kubernetes environments.

If you have Helm installed already, you can also use Helm
to install the Smart Agent. To learn more, see
[Install the Smart Agent using Helm](agent-k8s-install-helm.md).

## Prerequisites

* Linux kernel version 3.2 or higher
* cap_dac_read_search and cap_sys_ptrace capabilities
* Terminal or a similar command-line interface application
* SignalFx access token. See [Smart Agent Access Token](../../../_sidebars-and-includes/smart-agent-access-token.html).
* Your SignalFx realm. See [Realms](../../../_sidebars-and-includes/smart-agent-realm-note.html).

## Configure kubectl for the Smart Agent

### Create a Kubernetes secret

To install the Smart Agent using kubectl, you need to create a
Kubernetes secret for your access token and update settings in the Smart Agent's configuration files.

1. To create the Kubernetes secret, log in to the host from which you run kubectl.
2. Run the following command to create the Kubernetes secret called signalfx-agent, substituting `<access_token>` with
   your SignalFx access token:

   ```
   kubectl create secret generic --from-literal access-token=<access_token> signalfx-agent
   ```

### Download configuration files

On the same host, download the following configuration files from the
[Kubernetes Deployments](https://github.com/signalfx/signalfx-agent/tree/main/deployments/k8s) area in GitHub:

| File                      | Description                                                                                    |
|:--------------------------|:-----------------------------------------------------------------------------------------------|
| `clusterrole.yaml`        | Configuration settings for the ClusterRole cluster-admin. This file doesn't require an update. |
| `clusterrolebinding.yaml` | Configuration settings for Kubernetes role-based access control (**RBAC**)                     |
| `configmap.yaml`          | Cluster configuration settings                                                                 |
| `daemonset.yaml`          | Daemonset configuration settings                                                               |
| `serviceaccount.yaml`     | Configuration settings for Kubernetes Service Accounts. This file doesn't require an update.   |

### Update .yaml files

1. Update `configmap.yaml`:
   - Cluster name: For each of your Kubernetes clusters, replace `MY-CLUSTER` with a unique cluster name.
   - Realm: Update the value of `signalFxRealm` with the name of your SignalFx realm.
   - To avoid sending docker and cadvisor metrics from some containers,
     update the `datapointsToExclude` property. To learn more, see [Filtering](https://docs.splunk.com/observability/gdi/smart-agent/smart-agent-resources.html#filtering-data-using-the-smart-agent).
2. In the clusterrolebinding.yaml file, update `MY_AGENT_NAMESPACE` or the service account token reference with the Smart
   Agent namespace in which you're deploying the agent.

3. Update the daemonset.yaml file:

   - For RBAC-enabled clusters, add the permissions required for the Smart Agent.

   - For Rancher nodes, ensure that the Docker engine proxy is configured so that it can pull the `signalfx-agent` Docker image from `quay.io`.
     To learn more, see the Rancher v1.6 or Rancher v2.x documentation regarding proxy configuration.

   - Update the **cAdvisor** monitor configuration to use port 9344:

     ```
     monitors:
       - type: cadvisor
       - cadvisorURL: http://localhost:9344
     ```

4. If you're using OpenShift and you can't use the default namespace, modify each
   namespace occurrence and then ask your cluster administrator to run the following commands:

   ```
   oc create serviceaccount signalfx-agent
   oc adm policy add-cluster-role-to-user anyuid system:serviceaccount:default:signalfx-agent
   oc edit scc privileged
   ```

   Make the following changes to the scc file:

   ```
   users:
    - system:serviceaccount:default:signalfx-agent
    - serviceAccountName: signalfx-agent
   ```

## Install the Smart Agent

1. Remove collector services such as `collectd`.

2. Run the following command to update `kubectl` with the configuration files you've just modified:

   ```
   cat *.yaml | kubectl apply -f -
   ```

## Verify the Smart Agent

After you install the Smart Agent, it starts sending data from your clusters to SignalFx.

To see the services the Smart Agent has discovered, perform the following tasks:

1. Navigate to the directory where you installed `kubectl`.

2. Run the following command:

```
while read -r line; do kubectl exec --namespace `echo $line` signalfx-agent status; done <<< `kubectl get pods -l app=signalfx-agent --all-namespaces --no-headers | tr -s " " | cut -d " " -f 1,2`
```

In addition, you can do the following in SignalFx:

* For infrastructure monitoring:
  1. In SignalFx, open the **Infrastructure** built-in dashboard.
  2. In the override bar, select **Choose a host**. Select one of your nodes from the dropdown.

  The charts display metrics from the infrastructure for that node.

  To learn more, see [Built-In Dashboards and Charts](https://docs.splunk.com/observability/data-visualization/dashboards/built-in-dashboards.html).

* For Kubernetes monitoring:
  1. In SignalFx, from the main menu select **Infrastructure** > **Kubernetes Navigator** > **Cluster map**.
  2. The map displays all the clusters running the Smart Agent.
  3. Click the magnification icon to view the nodes in a cluster.

  The detail pane displays details of that cluster and nodes.

  To learn more, see [Getting Around the Kubernetes Navigator](https://docs.splunk.com/observability/infrastructure/monitor/k8s.html).

* For APM monitoring, learn how to install, configure, and verify the Smart Agent for Microservices APM (**µAPM**). See
[Get started with SignalFx µAPM](https://docs.splunk.com/observability/apm/intro-to-apm.html#nav-Introduction-to-Splunk-APM).

### Uninstall the Smart Agent

To delete all Smart Agent resources, run the following command in the directory
containing the `.yaml` configuration files:

```
cat *.yaml | kubectl delete -f -
```

See the [Kubectl Reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete) for more details.
