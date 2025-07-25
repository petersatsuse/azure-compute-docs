---
title: Azure Service Fabric probes
description: How to model a liveness and readiness probe in Azure Service Fabric by using application and service manifest files.
ms.topic: concept-article
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 07/11/2022
# Customer intent: As a cloud application developer, I want to configure liveness and readiness probes in Service Fabric manifest files, so that I can ensure my application instances are properly monitored and can handle traffic efficiently.
---

# Service Fabric Probes
Before you proceed with this article, become familiar with the [Service Fabric application model][application-model-link] and the [Service Fabric hosting model][hosting-model-link]. This article provides an overview of how to define a liveness and readiness probe by using manifest files.

## Liveness probe
Starting with version 7.1, Azure Service Fabric supports a liveness probe mechanism for containerize and non containerized applications. A liveness probe helps to report the liveness of a code package, which will restart if it doesn't respond quickly.

## Readiness probe
Starting with 8.2, readiness probe is also supported. A readiness probe is used to decide whether a code package is ready to accept traffic. For example if your container is taking a long time to process request or if the request queue is full, then your code package cannot accept anymore traffic and hence the endpoints to reach the code package will be removed. 

The behavior of the Readiness Probe is:
1.	The container/code package instance starts
2.	Endpoints are published immediately
3.	Readiness Probe starts running
4.	Readiness Probe eventually reaches failure threshold, and the endpoint is removed making it unavailable
5.	Instance eventually becomes ready
6.	Readiness Probe notices the instance is ready and publishes endpoint again
7.	Requests are routed again and succeed since it was ready to serve requests

> [!NOTE] 
> For Readiness probe, the code package is not restarted, just the endpoints are unpublished so the replica/partition set in not impacted.
>

## Semantics
You can specify only one liveness and one readiness probe per code package and can control its behavior by using these fields:

* `type`: Used to specify whether the probe type is Liveness or Readiness. Supported values are **Liveness** or **Readiness**

* `initialDelaySeconds`: The initial delay in seconds to start executing the probe after the container has started. The supported value is **int**. The default is 0 and the minimum is 0.

* `timeoutSeconds`: The period in seconds after which we consider the probe as failed, if it hasn't finished successfully. The supported value is **int**. The default is 1 and the minimum is 1.

* `periodSeconds`: The period in seconds to specify the frequency of the probe. The supported value is **int**. The default is 10 and the minimum is 1.

* `failureThreshold`: When we hit this value, the container will restart. The supported value is **int**. The default is 3 and the minimum is 1.

* `successThreshold`: On failure, for the probe to be considered successful, it has to run successfully for this value. The supported value is **int**. The default is 1 and the minimum is 1.

There can be, at most, one probe to one container at any moment. If the probe doesn't finish in the time set in **timeoutSeconds**, wait and count the time toward the **failureThreshold**. 

Additionally, Service Fabric will raise the following probe [health reports][health-introduction-link] on **DeployedServicePackage**:

* `OK`: The probe succeeds for the value set in **successThreshold**.

* `Error`: The probe **failureCount** ==  **failureThreshold**, before the container restarts.

* `Warning`: 
    * The probe fails and **failureCount** < **failureThreshold**. This health report stays until **failureCount** reaches the value set in **failureThreshold** or **successThreshold**.
    * On success after failure, the warning remains but with updated consecutive successes.

## Specifying a probe

You can specify a probe in the ApplicationManifest.xml file under **ServiceManifestImport**.

The probe can be for any of the following:

* HTTP
* TCP
* Exec 

### HTTP probe

For an HTTP probe, Service Fabric will send an HTTP request to the port and path that you specify. A return code that is greater than or equal to 200, and less than 400, indicates success.

Here is an example of how to specify an HTTP Liveness probe:

```xml
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateless1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <CodePackagePolicy CodePackageRef="Code">
        <Probes>
          <Probe Type="Liveness" FailureThreshold="5" SuccessThreshold="2" InitialDelaySeconds="10" PeriodSeconds="30" TimeoutSeconds="20">
            <HttpGet Path="/" Port="8081" Scheme="http">
              <HttpHeader Name="Foo" Value="Val"/>
              <HttpHeader Name="Bar" Value="val1"/>
            </HttpGet>
          </Probe>
        </Probes>
      </CodePackagePolicy>
    </Policies>
  </ServiceManifestImport>
```

The HTTP probe has additional properties that you can set:

* `path`: The path to use in the HTTP request.

* `port`: The port to use for probes. This property is mandatory. The range is 1 to 65535.

* `scheme`: The scheme to use for connecting to the code package. If this property is set to HTTPS, the certificate verification is skipped. The default setting is HTTP.

* `httpHeader`: The headers to set in the request. You can specify multiple headers.

* `host`: The host IP address to connect to.

> [!NOTE]
> Port and scheme is not supported for non-containerized applications. For this scenario please use **EndpointRef="EndpointName"** attribute. Replace 'EndpointName' with the name from the Endpoint defined in ServiceManifest.xml.
>

### TCP probe

For a TCP probe, Service Fabric will try to open a socket on the container by using the specified port. If it can establish a connection, the probe is considered successful. Here's an example of how to specify a probe that uses a TCP socket:

```xml
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateless1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <CodePackagePolicy CodePackageRef="Code">
        <Probes>
          <Probe Type="Liveness" FailureThreshold="5" SuccessThreshold="2" InitialDelaySeconds="10" PeriodSeconds="30" TimeoutSeconds="20">
            <TcpSocket Port="8081"/>
          </Probe>
        </Probes>
      </CodePackagePolicy>
    </Policies>
  </ServiceManifestImport>
```

### Exec probe

This probe will issue an **exec** command into the container and wait for the command to finish.

> [!NOTE]
> **Exec** command takes a comma separated string. The command in the following example will work for a Linux container.
> If you're trying to probe a Windows container, use **cmd**.

```xml
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateless1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <CodePackagePolicy CodePackageRef="Code">
        <Probes>
          <Probe Type="Liveness" FailureThreshold="5" SuccessThreshold="2" InitialDelaySeconds="10" PeriodSeconds="30" TimeoutSeconds="20">
            <Exec>
              <Command>ping,-c,2,localhost</Command>
            </Exec>
          </Probe>        
       </Probes>
      </CodePackagePolicy>
    </Policies>
  </ServiceManifestImport>
```

## Next steps
See the following article for related information:
* [Service Fabric and containers][containers-introduction-link]

<!-- Links -->
[containers-introduction-link]: service-fabric-containers-overview.md
[health-introduction-link]: service-fabric-health-introduction.md
[application-model-link]: service-fabric-application-model.md
[hosting-model-link]: service-fabric-hosting-model.md

