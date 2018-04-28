---
title: "Pass Data From Hosting Environment to Owin Startup Class"
date: 2018-04-27T20:18:28-07:00
draft: false
---

The diagram bellow, from MSDN Magazine, seems to suggest that there are ways to exchange data between Server and Owin Application. However, I have not found any official documentation/tutorials on how that can be accomplished.
![msdn magazine](http://i.imgur.com/3R4ms0i.png)

I found [this discussion](http://stackoverflow.com/questions/15511651/pass-a-parameter-to-owin-host) while searching for this topic on [stackoverflow](http://stackoverflow.com). The answer given there is only for self-hosted scenario, where you can call `WebApp.Start` method to manually load the Startup class and provide custom data. For ASP.NET hosted scenario, I haven't found a way to accomplish the same thing. The Startup class seems to be required to have a parameter-less constructor, and Katana will invoke that constructor to instantiate the class, then execute the Configuration method.

Ultimately, what I want to achieve in my projects is to have an implementation of Owin Application that is truly portable.It should work without any modification no matter where it's hosted, be it in ASP.NET or in a console process.For that, I think data that is server-dependent should be instantiated on the server side, then pass to the Startup class where the Owin pipeline is configured.

<!-- more -->

Currently, in projects I'm working on, I create a static holder class to store data that are initialized when server starts. When Startup.Configuration is invoked, I will read data from that static holder. I don't know if it's a recommended way, but it's not too bad an approach. Because there should be only once instance of the server running, so server-dependent data could be considered singletons. These data could be instantiated once in `Global.asax > Application_Start`, and be disposed, if needed, in `Global.asax > Application_End`.

I've created a sample app to demonstrate this approach. It this app, just for the sake of an example, I need to register an server-dependent Logger with my custom DI container, and then in my Startup class, I will get the container, and resolves that instance of the Logger to use through out the pipeline.

* The static holder class

```csharp
namespace OwinApplication
{
  public static class HostingEnvironment
  {
	public static string Name { get; set; }
	public static IUnityContainer DependencyResolver { get; set; }
  }
}
```

* Global.asax

```csharp
protected void Application_Start(object sender, EventArgs e)
{
  HostingEnvironment.Name = "AspNet";
  var container = new UnityContainer();
  container.RegisterType<ILogger, Logger>(new ContainerControlledLifetimeManager());
  HostingEnvironment.DependencyResolver = container;
}
```

* Startup class

```csharp
public class Startup
{
  public void Configuration(IAppBuilder app)
  {
    var dependencyResolver = HostingEnvironment.DependencyResolver;
    var logger = dependencyResolver.Resolve<Common.ILogger>();
    app.Run(context =>
      {
        var response = string.Format(
                          CultureInfo.InvariantCulture,
                          "Current hosting environment is {0}",
                          HostingEnvironment.Name);
        logger.Information(response);
        context.Response.ContentType = "text/plain";
        return context.Response.WriteAsync(response);
      });
  }
}
```
