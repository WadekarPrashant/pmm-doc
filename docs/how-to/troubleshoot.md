# Resolve issues

This section describes solutions to common problems and scenarios you might encounter while using PMM.


## Troubleshooting checklist

The following questions might help you identify the origin of the problem while using Percona Monitoring and Management:

1. Are you using the latest PMM version?
2. Did you check the known issues section in the Release Notes for that particular PMM release?
3. Are you receiving any error messages?
4. Do the logs contain any messages about the problem? See Message logs and Trace logs for more information.
5. Does the problem occur while configuring PMM, such as:
     - Does the problem occur while you configure a specific function?
     - Does the problem occur when you perform a particular task?
6. Are you using the recommended [authentication](../details/api.md#authenticate) method?
7. Does your system’s firewall allow TCP traffic on the [ports](../setting-up/server/network.md#ports) used by PMM?
8. Have you allocated enough [disk space](https://www.percona.com/blog/2017/05/04/how-much-disk-space-should-i-allocate-for-percona-monitoring-and-management/) for installing PMM? If not, check the disk allocation space.
9. Are you using a Technical Preview feature? Technical Preview features are not production-ready and should only be used in testing environments. For more information, see the relevant Release Notes.
10. For installing the PMM client, are you using a package other than a binary package without root permissions?
11. Is your [PMM Server](../setting-up/server/index.md) installed and running with a known IP address accessible from the client node?
12. Is the [PMM Client](../setting-up/client/index.md) installed, and is the node [registered with PMM Server](../setting-up/client/index.md#register)?
13. Is PMM-client configured correctly and has access to the config file?
14. For monitoring MongoDB, do you have adminUserAnyDatabase or superuser role privilege to any database servers you want to monitor?
15. For monitoring Amazon RDS using PMM, is there too much latency between PMM Server and the Amazon RDS instance?
16. Have you upgraded the PMM Server before you upgraded the PMM Client? If yes, there might be configuration issues, thus leading to failure in the client-server communication, as PMM Server might not be able to identify all the parameters in the configuration.
17. Is the PMM Server version higher than or equal to the PMM Client version? Otherwise, there might be configuration issues, thus leading to failure in the client-server communication, as PMM Server might not be able to identify all the parameters in the configuration.

## Troubleshooting areas

### Upgrade issues

#### PMM server not updating correctly


If the PMM server wasn't updated correctly, or if you have concerns about the release, you can force the update process in 2 ways:

1. From the UI - Home panel: click the Alt key on the reload icon in the Update panel to make the Update Button visible even if you are on the same version as available for update. Pressing this button will force the system to rerun the update so that any broken or not installed components can be installed. In this case, you'll go through the usual update process with update logs and successful messages at the end.

2. By API call (if UI not available): You can call the Update API directly with:

    ```sh
    curl --user admin:admin --request POST 'http://PMM_SERVER/v1/Updates/Start'
    ```

    Replace `admin:admin` with your username/password, and replace `PMM_SERVER` with your server address.

    !!! note alert alert-primary ""
        You will not see the logs using this method.

    Refresh The Home page in 2-5 minutes, and you should see that PMM was updated.

3. Upgrade PMM server using [Docker](../setting-up/server/docker.md#upgrade).

### Configuration issues

This section focuses on configuration issues, such as PMM-agent connection, adding and removing services for monitoring, and so on.

#### Client-server connections

There are many causes of broken network connectivity.

The container is constrained by the host-level routing and firewall rules when using [using Docker](../setting-up/server/docker.md). For example, your hosting provider might have default `iptables` rules on their hosts that block communication between PMM Server and PMM Client, resulting in *DOWN* targets in VictoriaMetrics. If this happens, check the firewall and routing settings on the Docker host.

PMM can also generate diagnostics data that can be examined and/or shared with our support team to help solve an issue. You can get collected logs from PMM Client using the pmm-admin summary command.

Logs obtained in this way include PMM Client logs and logs received from the PMM Server, and stored separately in the `client` and `server` folders. The `server` folder also contains its `client` subfolder with the self-monitoring client information collected on the PMM Server.

Beginning with [PMM 2.4.0](../release-notes/2.4.0.md), there is a flag that enables the fetching of [`pprof`](https://github.com/google/pprof) debug profiles and adds them to the diagnostics data. To enable, run `pmm-admin summary --pprof`.

You can get PMM Server logs with either of these methods:

**Direct download**

In a browser, visit `https://<address-of-your-pmm-server>/logs.zip`.

**From Help menu**

1. Select <i class="uil uil-question-circle"></i> *Help* → <i class="uil uil-download-alt"></i> *PMM Logs*.

2. Click *PMM Logs* to retrieve PMM diagnostics data which can be examined and shared with our support team should you need help.

#### Connection difficulties

**Passwords**

When adding a service, the host might not be detected if the password contains special symbols (e.g., `@`, `%`, etc.).

In such cases, you should convert any password, replacing special characters with their escape sequence equivalents.

One way to do this is to use the [`encodeURIComponent`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent) JavaScript function in your browser's web console (commonly found under a *Development Tools* menu). Run the function with your password as the parameter. For example:

```js
> encodeURIComponent("s3cR#tpa$$worD")
```

will give:

```txt
"s3cR%23tpa%24%24worD"
```

**Password change**

When adding clients to the PMM server, you use the `admin` user. However, if you change the password for the admin user from the PMM UI, then the clients will not be able to access PMM due to authentication issues. Also, Grafana will lock out the admin user due to multiple unsuccessful login attempts.

In such a scenario, use [API key](../../details/api.md#api-keys-and-authentication) for authentication. You can use API keys as a replacement for basic authentication.

## Percona Alerting

### No Alert rule templates tab on the Alerting page
Percona Alerting option isn't active.

### Integrated Alerting issues

#### No <i class="uil uil-bell"></i> Integrated Alerting icon

You are not logged in as a privileged user. You need either Admin or Editor roles to work with Integrated Alerting.

** <i class="uil uil-bell"></i> Integrated Alerting icon but no submenu**

#### Integrated Alerting isn't activated.

1. Go to <i class="uil uil-cog"></i> *Configuration* → <i class="uil uil-setting"></i> *Settings* → *Advanced Settings*.
2. Enable *Alerting*.

### Custom alert rule templates not migrated to Percona Alerting
If you have used Integrated Alerting in previous PMM versions, and had custom templates under ``/srv/ia/templates``, make sure to transfer them to ``/srv/alerting/templates``. 
PMM is no longer sourcing templates from the ``ia`` folder, since we have deprecated Integrated Alerting with the 2.31 release. 

#### Unreachable external IP addresses

If you get an email or page from your system that the IP is not reachable from outside my organization, do the following:

To configure your PMM Server’s Public Address, select <i class="uil uil-cog"></i> *Configuration* → <i class="uil uil-setting"></i> *Settings* → *Advanced Settings*, and supply an address to use in your alert notifications.


#### Notification channels not working

**I tried to set up a Slack/Email channel but nothing happened.**

Before you can use a notification channel you must provide your connection details.

1. Go to *PMM* → *PMM Settings* → *Communication*.
2. Define your SMTP server or Slack incoming webhook URL.

For PagerDuty you can configure it in the notification channel tab of Integrated Alerting by supplying your server/routing key.


#### Alert Rule Templates are disabled

Built-In alerts are not editable, but you can copy them and edit the copies. (In [PMM 2.14.0](../release-notes/2.14.0.md) and above).

If you create a custom alert rule template, you will have access to edit.


### QAN issues

This section focuses on problems with QAN, such as queries not being retrieved so on.

#### Missing data

**Why don't I see any query-related information?**

There might be multiple places where the problem might come from:

- Connection problem between pmm-agent and pmm-managed
- PMM-agent cannot connect to the database.
- Data source is not properly configured.


**Why don't I see the whole query?**

Long query examples and fingerprints can be truncated to 1024 symbols to reduce space usage. In this case, the query explains section will not work.


