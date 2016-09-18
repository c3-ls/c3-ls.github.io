---
title: Introducing our first open-source contribution "ServiceFabric-HttpServiceGateway"
---

We're happy to announce our first open-source library called "ServiceFabric-HttpServiceGateway"!

We use Azure Service Fabric for our internal services. Most of these services are built with ASP.NET WebAPIs 
and therefore offer a HTTP endpoint. To call one of these services you have to resolve the address by asking 
the Service Fabric naming service for the actual endpoint uri.

If you want to access your cluster from a machine that is not part of the cluster, 
resolving this address becomes complicated. We also see our cluster as a security border 
and therefore run the cluster in a separate virtual network. We therefore don't allow 
direct access into services from outside the cluster.

Instead, we are using an Azure Load Balancer which forwards all requests to one gateway service 
(an ASP.NET 5 project hosted with Kestrel). This gateway service is running on every machine and 
is responsible for forwarding requests to the actual internal services.

One component of this gateway is the open-sourced library "C3.ServiceFabric.HttpServiceGateway". 
It is an ASP.NET 5 middleware that does the actual work:

* It resolves the public endpoint (like "/myservice") to an internal address by calling the Service Fabric naming service
* It forwards the HTTP request
* It appends some proxy headers to give the actual service information about the caller and the proxy
* If the call fails, the middleware validates the error and retries the call if it might be a recoverable error.

You can find more information about this component in the [project wiki](https://github.com/c3-ls/ServiceFabric-HttpServiceGateway/wiki).

You would use the library in your Startup.cs and it would look like this:

```csharp
app.Map("/service", appBuilder =>
{
    appBuilder.RunHttpServiceGateway(new HttpServiceGatewayOptions
    {
        ServiceName = new Uri("fabric:/GatewaySample/HttpServiceService")
    });
});
```

If you use the gateway service exclusively for HTTP based services, it would be a very simple application and you 
can find an example for this in the samples-folder.

Please feel free to post any questions, issues and feedback as an issue [in the GitHub project](https://github.com/c3-ls/ServiceFabric-Http).

