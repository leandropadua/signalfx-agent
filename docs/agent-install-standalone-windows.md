# Install to Windows using a ZIP file

:warning: **SignalFx Smart Agent is deprecated. For details, see the [Deprecation Notice](./smartagent-deprecation-notice.md)** :warning:

Install the SignalFx Smart Agent to a Windows host using a standalone package in
a ZIP file.

## Prerequisites

* Windows Server 2012 or higher
* Windows PowerShell access
* Windows decompression application, such as WinZip or the native Windows
  decompression feature.
* .NET Framework 3.5 or higher
* TLS 1.0 or 1.2
* Administrator account in which to run the Smart Agent

If you want to invoke a Python script for non-default monitors such as `exec`,
you must have Python 3 installed. The ZIP file includes the Python 3.8.0
runtime.

## Install using a ZIP file

1. Remove collector services such as `collectd`

2. To get the latest Windows standalone installation package, navigate to
   [Smart Agent releases](https://github.com/signalfx/signalfx-agent/releases)
   and download the following file:

   ```
   signalfx-agent-<latest_version>-win64.zip
   ```

   For example, if the latest version is **5.1.6**, follow these steps:

   1. In the **releases** section, find the section called **v5.1.6**.
   2. In the **Assets** section, click `signalfx-agent-5.1.6-win64.zip`
   3. The ZIP file starts downloading.

3. Uncompress the ZIP file. The
   package contents expand into the `signalfx-agent` directory.

## Configure the ZIP file installation

Navigate to the `signalfx-agent` directory, then create a configuration
file for the agent:

1. In a text editor, create a new file called signalfx-agent\agent-config.yml.
2. In the file, add your hostname and port number:

   ```
   internalStatusHost: <local_hostname>
   internalStatusPort: <local_port>
   ```

3. Save the file.

The Smart Agent collects metrics based on the settings in
`agent-config.yml`. The `internalStatusHost` and `internalStatusPort`
properties specify the host and port number of the host that's running the Smart Agent.

### Configure user privileges

To run the Smart Agent in non-Administrator mode, grant specific permissions to the user. 

By default, only members of the Administrators group can start, stop, pause, resume, or restart a service. In this case, to use some monitors and observers, you need to grant the following required permissions:

- Full access of `signalfx-agent` and Windows Management Instrumentation (`Winmgmt`) for the user. See [Method 3: Use Subinacl.exe](https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/grant-users-rights-manage-services#method-3-use-subinaclexe) on the Microsoft documentation site for information on granting user rights.
- Full access rights `SC_MANAGER_ALL_ACCESS (0xF003F)` for the Security Compliance Manager (SCM). Use the following command to grant rights to the user:
```bash
sc.exe sdset SCMANAGER "D:(A;;0xF003F;;;<SID of user>)(all other existing rights)"
```

### Start the Smart Agent

To run the Smart Agent as a Windows program, run the following command in a console window:

  ```
  SignalFxAgent\bin\signalfx-agent.exe -config SignalFxAgent\agent-config.yml > <log_file>`
  ```

> The default log output for Smart Agent goes to `STDOUT` and `STDERR`.
> To persist log output, direct the log output to `<log_file>`.

To install the Smart Agent as a Windows service, run the following command in a console window:

  ```
  SignalFxAgent\bin\signalfx-agent.exe -service "install" -logEvents -config SignalFxAgent\agent-config.yml
  ```

To start the Smart Agent as a Windows service, run the following command in a console window:

  ```
  SignalFxAgent\bin\signalfx-agent.exe -service "start"
  ```

To learn about other Windows service options, see
[Service Configuration](https://docs.splunk.com/observability/gdi/smart-agent/smart-agent-resources.html#install-the-smart-agent).

### Verify the Smart Agent

To verify your installation and configuration, perform these steps:

* For infrastructure monitoring, perform these steps:
  1. In SignalFx, open the **Infrastructure** built-in dashboard
  2. In the override bar, select **Choose a host**. Select one of your hosts from the dropdown list.

  The charts display metrics from your infrastructure.

  To learn more, see [Built-In Dashboards and Charts](https://docs.splunk.com/observability/data-visualization/dashboards/built-in-dashboards.html).

* For Kubernetes monitoring, perform these steps:
  1. In SignalFx, from the main menu select **Infrastructure** > **Kubernetes Navigator** > **Cluster map**.
  2. In the cluster display, find the cluster you installed.
  3. Click the magnification icon to view the nodes in the cluster.

  The detail pane on the right hand side of the page displays details of your cluster and nodes.

  To learn more, see [Getting Around the Kubernetes Navigator](https://docs.splunk.com/observability/infrastructure/monitor/k8s.html)

* For APM monitoring, learn how to install, configure, and verify the Smart Agent for Microservices APM (**µAPM**). See
  [Get started with SignalFx µAPM](https://docs.splunk.com/observability/apm/intro-to-apm.html#nav-Introduction-to-Splunk-APM).

### Uninstall the Smart Agent

1. If the `signalfx-agent` service was installed, run the following PowerShell
   commands to stop and uninstall the service:
   ```powershell
   SignalFxAgent\bin\signalfx-agent.exe -service "stop"
   SignalFxAgent\bin\signalfx-agent.exe -service "uninstall"
   ```

1. Back up any files as necessary and delete the `SignalFxAgent` folder.
