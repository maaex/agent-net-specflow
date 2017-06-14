[![Build status](https://ci.appveyor.com/api/projects/status/k9gnrmlt3yo5gl4g?svg=true)](https://ci.appveyor.com/project/nvborisenko/agent-net-specflow)

# Installation
Install **ReportPortal.SpecFlow** NuGet package into your project with scenarios.

[![NuGet version](https://badge.fury.io/nu/reportportal.specflow.svg)](https://badge.fury.io/nu/reportportal.specflow)

> PS> Install-Package ReportPortal.SpecFlow

After installing NuGet package your App.config is modified. Report Portal plugin and step assembly will be registered in the specFlow section.
```xml
<specFlow>
  ...
  <plugins>
    <add name="ReportPortal" type="Runtime" />
  </plugins>
  ...
  <stepAssemblies>
    <stepAssembly assembly="ReportPortal.SpecFlowPlugin" />
  </stepAssemblies>
  ...
</specFlow>
```

# Configuration
All settings are stored in *ReportPortal.SpecFlowPlugin.dll.config* file which was added into your project by nuget installation.

Example of config file:
```xml
<configuration>
  <configSections>
    <section name="reportPortal" type="ReportPortal.SpecFlowPlugin.ReportPortalSection, ReportPortal.SpecFlowPlugin, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  </configSections>
  <reportPortal enabled="true">
    <server url="https://rp.epam.com/api/v1/" project="default_project">
      <authentication username="default" password="aa19555c-c9ce-42eb-bb11-87757225d535" />
    </server>
    <launch name="SpecFlow Demo Launch" debugMode="true" tags="t1,t2" />
  </reportPortal>
</configuration>
```

Alternatively,you could configure the plugin via app.config of your application, instead of *ReportPortal.SpecFlowPlugin.dll.config*.Just add above mentioned config 
options to app.config and remove *ReportPortal.SpecFlowPlugin.dll.config*.


## Combine several execution in one launch

How it can be done:
1. CI server creates a RP launch and saves the launch id to `app.config` of test binaries.
2. Test binaries are copied to VMs and run there.
3. The tests start and see that there is launch id in `app.config` and don't create a new launch - they re-use the existing one. Also they don't close it once they are done.
```c#
[BeforeTestRun(Order = -30000)]
public static void BeforeTestRunPart()
{
	ReportPortalAddin.BeforeRunStarted += ReportPortalAddin_BeforeRunStarted;
	ReportPortalAddin.BeforeRunFinished += ReportPortalAddin_BeforeRunFinished;
}

public static void ReportPortalAddin_BeforeRunStarted(object sender, RunStartedEventArgs e)
{
	var launchId = System.Configuration.ConfigurationManager.AppSettings["ReportPortalLaunchId"];
	if (launchId.IsNullOrEmpty() == false)
	{
		e.Canceled = true;
		Bridge.Context.LaunchId = launchId;
	}
}

public static void ReportPortalAddin_BeforeRunFinished(object sender, RunFinishedEventArgs e)
{
	var launchId = System.Configuration.ConfigurationManager.AppSettings["ReportPortalLaunchId"];
	if (launchId.IsNullOrEmpty() == false)
	{
		e.Canceled = true;
	}
}
```
4. When all tests run, CI server closes the RP launch.
