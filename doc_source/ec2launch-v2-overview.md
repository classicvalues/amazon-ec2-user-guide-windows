# EC2Launch v2 overview<a name="ec2launch-v2-overview"></a>

EC2Launch v2 is a service that performs tasks during instance startup and runs if an instance is stopped and later started, or restarted\. 

**Note**  
In order to use EC2Launch with IMDSv2, the version must be 1\.3\.2002730 or later\.

**Topics**
+ [Compare Amazon EC2 launch services](#ec2launch-v2-agent-compare)
+ [EC2Launch v2 concepts](#ec2launch-v2-concepts)
+ [EC2Launch v2 tasks](#ec2launch-v2-tasks)
+ [Telemetry](#ec2launch-v2-telemetry)

## Compare Amazon EC2 launch services<a name="ec2launch-v2-agent-compare"></a>

The following table shows the major functional differences between EC2Config, EC2Launch v1, and EC2Launch v2\.


| Feature | EC2Config | EC2Launch v1 | EC2Launch v2 | 
| --- | --- | --- | --- | 
| Executed as | Windows Service |  PowerShell Scripts  | Windows Service | 
| Supports |  Windows 2012 Windows 2012 R2  |  Windows 2016 Windows 2019 \(LTSC and SAC\)  |  Windows 2012 Windows 2012 R2 Windows 2016 Windows 2019 \(LTSC and SAC\) Windows 2022  | 
|  Configuration file  | XML | XML |  YAML  | 
|  Set Administrator username  | No | No |  Yes  | 
|  User data size  | 16 KB | 16 KB |  60 KB \(compressed\)  | 
|  Local user data baked on AMI  | No | No | Yes, configurable | 
| Task configuration in user data | No | No | Yes | 
|  Configurable wallpaper  | No | No |  Yes  | 
|  Customize task execution order  | No | No |  Yes  | 
|  Configurable tasks  | 15 |  9  |  20 at launch  | 
|  Supports Windows Event Viewer  | Yes |  No  | Yes | 
|  Number of Event Viewer event types  | 2 |  0  |  30  | 

## EC2Launch v2 concepts<a name="ec2launch-v2-concepts"></a>

The following concepts are useful to understand when considering EC2Launch v2\.

**Task**  
A task can be invoked to perform an action on an instance\. For a list of available tasks for EC2Launch v2, see [EC2Launch v2 tasks](#ec2launch-v2-tasks)\. For task configuration schema and details, see [EC2Launch v2 task configuration](ec2launch-v2-settings.md#ec2launch-v2-task-configuration)\. Tasks can be configured in the `agent-config.yml` file or through user data\.

**Stage**  
A stage is a logical grouping of tasks that are run by the service\. Some tasks can run only in a specific stage\. Others can run in multiple stages\. When using `agent-config.yml`, you must specify a list of stages, and a list of tasks within each stage\.

The service runs the stages in the following order:

1. Boot

1. Network

1. PreReady

1. PostReady

After the PreReady stage completes, the service sends the `Windows is ready` message to the Amazon EC2 console\. Then, the PostReady stage will run\. User data runs after the PostReady stage completes\. For example stages and tasks, see [Example: `agent-config.yml`](ec2launch-v2-settings.md#ec2launch-v2-example-agent-config)\.

When you use user data, you must specify a list of tasks\. The stage is implied\. For example tasks, see [Example: user data](ec2launch-v2-settings.md#ec2launch-v2-example-user-data)\.

The service runs the list of tasks in the order that you specify in `agent-config.yml` and in user data\. Stages run sequentially\. The next stage starts after the previous stage completes\. Tasks are also run sequentially\. 

**Frequency**  
Task frequency determines when tasks should run, depending on the boot context\. Most tasks have only one allowed frequency\. You can specify a frequency for `executeScript` tasks\.

You will see the following frequencies in the [EC2Launch v2 task configuration](ec2launch-v2-settings.md#ec2launch-v2-task-configuration)\.
+ Once — The task runs once, when the AMI has booted for the first time \(finished Sysprep\)\.
+ Always — The task runs every time that the launch agent runs\. The launch agent runs when:
  + an instance starts or restarts
  + the EC2Launch service runs
  + `EC2Launch.exe run` is invoked

**agent\-config**  
`agent-config` is a file that is located in the configuration folder for EC2Launch v2\. It includes configuration for the boot, network, preready, and postready stages\. This file is used to specify the configuration for an instance for tasks that should run when the AMI is either booted for the first time or for subsequent times\.

By default, the EC2Launch v2 installation installs an `agent-config` file that includes recommended configurations that are used in standard Amazon Windows AMIs\. You can update the configuration file to alter the default boot experience for your AMI that EC2Launch v2 specifies\.

**User data**  
User data is data that is configurable when you launch an instance\. You can update user data to dynamically change how custom AMIs or quickstart AMIs are configured\. EC2Launch v2 supports 60 kB user data input length\. User data includes only the UserData stage, and therefore runs after the `agent-config` file\. You can enter user data when you launch an instance using the launch instance wizard, or you can modify user data from the EC2 console\. For more information about working with user data, see [Run commands on your Windows instance at launch](ec2-windows-user-data.md)\.

## EC2Launch v2 tasks<a name="ec2launch-v2-tasks"></a>

EC2Launch v2 can perform the following tasks at each boot:
+ Set up new and optionally customized wallpaper that renders information about the instance\.
+ Set the attributes for the administrator account that is created on the local machine\.
+ Add DNS suffixes to the list of search suffixes\. Only suffixes that do not already exist are added to the list\.
+ Set drive letters for any additional volumes and extend them to use available space\.
+ Write files to the disk, either from the internet or from the configuration\. If the content is in the configuration, it can be base64 decoded or encoded\. If the content is from the internet, it can be unzipped\.
+ Execute scripts either from the internet or from the configuration\. If the script is from the configuration, it can be base64 decoded\. If the script is from the internet, it can be unzipped\.
+ Execute a program with given arguments\.
+ Set the computer name\.
+ Send instance information to the Amazon EC2 console\.
+ Send the RDP certificate thumbprint to the EC2 console\.
+ Dynamically extend the operating system partition to include any unpartitioned space\.
+ Execute user data\. For more information about specifying user data, see [EC2Launch v2 task configuration](ec2launch-v2-settings.md#ec2launch-v2-task-configuration)\.
+ Set non\-persistent static routes to reach the metadata service and AWS KMS servers\.
+ Set non\-boot partitions to MBR or GPT\.
+ Start the Systems Manager \(SSM\) service following Sysprep\.
+ Optimize ENA settings\.
+ Enable OpenSSH for later Windows versions\.
+ Enable Jumbo Frames\.
+ Set Sysprep to run with EC2Launch v2\.
+ Publish Windows event logs\.

## Telemetry<a name="ec2launch-v2-telemetry"></a>

Telemetry is additional information that helps AWS to better understand your requirements, diagnose issues, and deliver features to improve your experience with AWS services\.

EC2Launch v2 version `2.0.592` and later collect telemetry, such as usage metrics and errors\. This data is collected from the Amazon EC2 instance on which EC2Launch v2 runs\. This includes all Windows AMIs owned by AWS\.

The following types of telemetry are collected by EC2Launch v2:
+ **Usage information** — agent commands, install method, and scheduled run frequency\.
+ **Errors and diagnostic information** — agent installation and run error codes\.

Examples of collected data:

```
2021/07/15 21:44:12Z: EC2LaunchTelemetry: IsAgentScheduledPerBoot=true
2021/07/15 21:44:12Z: EC2LaunchTelemetry: IsUserDataScheduledPerBoot=true
2021/07/15 21:44:12Z: EC2LaunchTelemetry: AgentCommandCode=1
2021/07/15 21:44:12Z: EC2LaunchTelemetry: AgentCommandErrorCode=5
2021/07/15 21:44:12Z: EC2LaunchTelemetry: AgentInstallCode=2
2021/07/15 21:44:12Z: EC2LaunchTelemetry: AgentInstallErrorCode=0
```

Telemetry is enabled by default\. You can disable telemetry collection at any time\. If telemetry is enabled, EC2Launch v2 sends telemetry data without additional customer notifications\.

**Telemetry visibility**  
When telemetry is enabled, it appears in the Amazon EC2 console output as follows:

```
2021/07/15 21:44:12Z: Telemetry: <Data>
```

**Disable telemetry on an instance**  
To disable telemetry for a single instance, you can either set a system environment variable, or use the MSI to modify the installation\.

To disable telemetry by setting a system environment variable, run the following command as an administrator:

```
setx /M EC2LAUNCH_TELEMETRY 0
```

To disable telemetry using the MSI, run the following command after you [download the MSI](ec2launch-v2-install.md): 

```
msiexec /i ".\AmazonEC2Launch.msi" Remove="Telemetry" /q
```