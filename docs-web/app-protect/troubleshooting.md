# Troubleshooting

This document describes how to troubleshoot problems with the Ingress Controller with the [App Protect](/nginx-app-protect/) module enabled.

For general troubleshooting of the Ingress Controller, check the general [troubleshooting](/nginx-ingress-controller/troubleshooting/) documentation.

For additional troubleshooting of the App Protect module itself, check the [troubleshooting](/nginx-app-protect/troubleshooting/) guide in the App Protect module documentation. 

## Potential Problems

The table below categorizes some potential problems with the App Protect integration with the Ingress Controller you may encounter and suggests how to troubleshoot those problems using one or more methods from the next section.

```eval_rst
.. list-table::
   :header-rows: 1

   * - Problem area
     - Symptom
     - Troubleshooting method
     - Common cause
   * - Start.
     - The Ingress Controller fails to start.
     - Check the logs.
     - Misconfigured APLogConf or APPolicy.
   * - APLogConf, APPolicy or Ingress Resource.
     - The configuration is not applied.
     - Check the events of the APLogConf, APPolicy and Ingress Resource, check the logs.
     - APLogConf or APPolicy is invalid.
   * - NGINX.
     - The Ingress Controller NGINX verification timeouts while starting for the first time or while reloading after a change.
     - Check the logs for ``Unable to fetch version: X`` message.
     - Too many Ingress Resources with App Protect enabled. Check the `NGINX fails to start/reload section <#nginx-fails-to-start-or-reload>`_ of the Known Issues. 
```

## Troubleshooting Methods

### Checking the Ingress Controller and App Protect logs

App Protect logs are part of the Ingress Controller logs when the module is enabled. To check the Ingress Controller logs, follow the steps of [Checking the Ingress Controller Logs](/nginx-ingress-controller/troubleshooting/#checking-the-ingress-controller-logs) of the Troubleshooting guide.

### Checking the Events of an Ingress Resource

Follow the steps of [Checking the Events of an Ingress Resource](/troubleshooting/#checking-the-events-of-an-ingress-resource).

### Checking events of APLogConf

After you create or update an APLogConf, you can immediately check if the NGINX configuration was successfully applied by NGINX:
```
$ kubectl describe aplogconf logconf
Name:         logconf
Namespace:    default
. . . 
Events:
  Type    Reason          Age   From                      Message
  ----    ------          ----  ----                      -------
  Normal  AddedOrUpdated  11s   nginx-ingress-controller  AppProtectLogConfig  default/logconf was added or updated
```
Note that in the events section, we have a `Normal` event with the `AddedOrUpdated` reason, which informs us that the configuration was successfully applied.

### Checking events of APPolicy

After you create or update an APPolicy, you can immediately check if the NGINX configuration was successfully applied by NGINX:
```
$ kubectl describe appolicy dataguard-alarm
Name:         dataguard-alarm
Namespace:    default
. . . 
Events:
  Type    Reason          Age    From                      Message
  ----    ------          ----   ----                      -------
  Normal  AddedOrUpdated  2m25s  nginx-ingress-controller  AppProtectPolicy default/dataguard-alarm was added or updated
```
Note that in the events section, we have a `Normal` event with the `AddedOrUpdated` reason, which informs us that the configuration was successfully applied.
 
## Running App Protect in Debug Mode

Running the Ingress Controller in debug mode will also run the App Protect module in debug mode. Follow the steps of [Running NGINX in the Debug Mode](/nginx-ingress-controller/troubleshooting/#running-nginx-in-the-debug-mode).

## Known Issues

When using the Ingress Controller with the App Protect module, the following issues might happen depending on the number of Ingress Resources with App Protect enabled in your cluster:

### NGINX reload times increase

The Ingress Controller reloads NGINX whenever there is a change that requires NGINX to reload to apply a new configuration. Without the App Protect module enabled, usual reload times are around 150ms. If App Protect module is enabled and is being used by any number of Ingress Resources, these reloads might take a few seconds instead. If you are running more than one instance of the Ingress Controller, because there is no order on how the Ingress Controller processes the Kubernete Reources, it might occur that different instances have different NGINX configurations until all of them have finished with the reloads.

In order to reduce these inconsistencies, we recommend not to apply changes to resources handled by the Ingress Controller at the same time. 

### NGINX fails to start or reload

The first time the Ingress Controller starts or whenever there is a change that requires reloading NGINX, the Ingress Controller will verify if the reload was successful. There is a timeout for this verification. When App Protect is enabled, this timeout is 20 seconds (instead of 4 seconds without the module). 

This timeout should be more than enough to verify configurations. However, when numerous Ingress Resources with App Protect enabled are handled by the Ingress Controller at the same time (for instance, if you apply a large amount of Ingress Resources at once, or when the Ingress Controller runs for the first time a cluster where the Ingress Resources with App Protect enabled are already present), this timeout might not be enough.

You can increase this timeout by setting the `nginx-reload-timeout` [cli-argument](/nginx-ingress-controller/configuration/global-configuration/command-line-arguments/#cmdoption-nginx-reload-timeout).