---
title:  "A tale of debugging ASP.Net 5 and HttpPlatformHandler"
date:   2015-11-03 12:03:00
categories: c# aspnetcore httpplatformhandler
---

Had an interesting problem at work this morning. We were ready to deploy a new version of our ASP.Net 5 website to our staging environment and everything was looking fine both locally and on our development server. We merged it to our staging branch and pushed it to our buildserver and everything was looking fine until we tried to browse to the site and got this:

![Oops.](/assets/vnext-error.png)

Not much to go on there, something obviously crashed but that's it. So how do we proceed here?

First of, the Event Viewer in Windows is always a good start when you want to know what happened in IIS so lets start there. If we look in the "Application log" we could see something like this.

> The description for Event ID 1001 from source HttpPlatformHandler cannot be found. Either the component that raises this event is not installed on your local computer or the installation is corrupted. You can install or repair the component on the local computer.
> If the event originated on another computer, the display information had to be saved with the event.
> The following information was included with the event: 
> Process '5624' started successfully and is listening on port '5225'.

Well that's swell, it seems like the platform handler managed to start the dnx process and was happy with it, no other information was available though. Seems like the Event Viewer might no the be "go to place" it used to be when running via a platform handler.

After this I started checking all other parts of our system to see if there was something wrong with the API since we fetch some settings from it during startup which might end badly if the API wasn't responding. This showed up empty though, everything else was running fine so the culprit was somewhere at our ASP.Net 5 site after all.

What I then found was that the platform handler could log the stdout of the started process. Open the `web.config` located in wwwroot of your deployed site and change `stdoutLogEnabled` to true.

{% highlight xml %}
<configuration>
  <system.webServer>
    <handlers>
      <add name="httpplatformhandler" path="*" verb="*" modules="httpPlatformHandler" resourceType="Unspecified" />
    </handlers>
    <httpPlatform processPath="..\approot\web.cmd" arguments="" stdoutLogEnabled="true" stdoutLogFile="..\logs\httpplatform-stdout" startupTimeLimit="3600"></httpPlatform>
  </system.webServer>
</configuration>
{% endhighlight %}

When I then tried to browse to the site again I got this in my logfile.

    error   : [Microsoft.AspNet.Hosting.Internal.HostingEngine] Application startup exception
    System.Reflection.TargetInvocationException: Exception has been thrown by the target of an invocation. ---> System.ArgumentException: 
    Parameter name: value
       at Microsoft.AspNet.Http.PathString..ctor(String value)
       at Microsoft.AspNet.Builder.ExceptionHandlerExtensions.UseExceptionHandler(IApplicationBuilder app, String errorHandlingPath)
       at MorbidFox.Web.Startup.ConfigureStaging(IApplicationBuilder app, ILoggerFactory loggerFactory)
       --- End of inner exception stack trace ---
       at System.RuntimeMethodHandle.InvokeMethod(Object target, Object[] arguments, Signature sig, Boolean constructor)
       at System.Reflection.RuntimeMethodInfo.UnsafeInvokeInternal(Object obj, Object[] parameters, Object[] arguments)
       at System.Reflection.RuntimeMethodInfo.Invoke(Object obj, BindingFlags invokeAttr, Binder binder, Object[] parameters, CultureInfo culture)
       at Microsoft.AspNet.Hosting.Startup.ConfigureBuilder.Invoke(Object instance, IApplicationBuilder builder)
       at Microsoft.AspNet.Hosting.Internal.HostingEngine.BuildApplication()

It's not much, but at least we got a stracktrace pointing us to `Startup.ConfigureStaging` and the `UseExceptionHandler` call.

Here are our `ConfigureStaging` and `ConfigureProduction` methods from Startup.cs.

{% highlight c# %}
//This method is invoked when ASPNET_ENV is 'Staging'
public void ConfigureStaging(IApplicationBuilder app, ILoggerFactory loggerFactory) {
    // StatusCode pages to gracefully handle status codes 400-599.
    app.UseStatusCodePagesWithReExecute("~/System/Status/{0}");
    app.UseExceptionHandler("System/Error");

    Configure(app);
}

//This method is invoked when ASPNET_ENV is 'Production'
public void ConfigureProduction(IApplicationBuilder app, ILoggerFactory loggerFactory) {
    // StatusCode pages to gracefully handle status codes 400-599.
    app.UseStatusCodePagesWithReExecute("~/System/Status/{0}");
    app.UseExceptionHandler("/System/Error");

    Configure(app);
}
{% endhighlight %}

So what caused this hard to track down error? Apparently we were missing a `/` at the start of the url to `app.UseExceptionHandler` in our setup for the staging environment, and that caused it all to crash and burn. Since it's internally converted to a `PathString` it has to start with a `/` or it's an `ArgumentException` with the message "Parameter name: value". I added that missing `/`, tested it locally, pushed it to the staging branch and everything was up and running again. 
