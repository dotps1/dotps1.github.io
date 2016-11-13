---
layout: post
title: Deploying Dynamic Server Nano Images with SCCM
date: 2016-11-12 17:30:00 -0400
categories: SCCM, Server, Nano, WinPE
---

So, its been a while since my last blog post. 
My short lived carreer in Consulting has already come to an end and I am back where I belong, in Enterprise IT. 
That being said, this week I took the plunge into deploying Windows Server 2016 Nano, and this is my tale...

---

I have heard about Server Nano for sometime now, but have never really took a look at it. 
When we decided we where going to deploy Nano, I started doing some research, and found the typical blogs about deploying the image, using the `NanoServerImageGenerator` PowerShell Module. 
But, everything was so static, either build a Hyper-V host, or build a File Server, or DNS Server, spit out a WIM, then deploy it. 
Who wants to do that? 
It seemed like a huge step back from the typical methodolgy of SCCM, booting to WinPE, laying an OS, laying apps and customizing, and doing it all at build time. 
I mean could you imaging have to create static images for all your Windows 10 clients, with like Outlook Packages or Adobe Packages enbedded in them, and then neededing go maintaing those....? 


So, as an example, I was going to build some Hyper-V Hosts, so, per the examples i found online it would be something like this:

```
#requires -Modules NanoServerImageGenerator
# https://technet.microsoft.com/en-us/windows-server-docs/get-started/nano-server-quick-start
# https://www.microsoft.com/en-us/download/details.aspx?id=54065
New-NanoServerImage -Edition Standard -DeploymentType Host -MediaPath <path to root of media> -BasePath .\Base -TargetPath .\NanoServerPhysical\NanoServer.wim -ComputerName HyperVHost -OEMDrivers -Compute -Clustering
```

Where: 

* -MediaPath specifies a path to the root of the contents of the Windows Server 2016 ISO. For example if you have copied the contents of the ISO to d:\TP5ISO you would use that path. 
* -BasePath specifies a folder that will be created to copy the Nano Server WIM and packages to. (This parameter is optional.) 
* -TargetPath specifies a path, including the filename and extension, where the resulting WIM will be created. 
* -ComputerName is the computer name for the Nano Server you are creating.

Now, as you may can see right away with this method, there is more then a few problems.

* You can see that Edition is static in the out WIM file, "Standard", what if I wanted some Datacenter too, That would already be two WIMs to maintain.
* The DeploymentType, The options are Host,Guest, now with nested virtualization, I would most likely want this option dynamic, either Host or Guest, so this would mean two more WIMs to keep well 4 now.
* The OEMDrivers switch, this adds the basic OEM Drivers that is included with Server 2016 core, this is really only useful if the DeploymentType is type Host, so if this is a VM, then I don't really want this package added.

I think you can already see where this is going and why a more dynamic build setup is needed if your plan to deploy more then one configuration of Nano, else your going to have to maintain many WIM files (more then 1 is too many in my opinion). 
And how ever many you think you will need, it will be doubled if you need a Standard and Datacenter edition of each.

And there are many many different packages you can install and im sure more will come:

---

| Role or feature | PowerShell Parameter Option |
|:----------------|----------------------------:|
| Hyper-V role (including NetQoS) | -Compute |
| Failover Clustering | -Clusting |
| Basic drivers for a variety of network adapters and storage controllers.<br>This is the same set of drivers included in a Server Core installation of Windows Server 2016. | -OEMDrivers |
| File Server role and other storage components | -Storage |
| Windows Defender Antimalware, including a default signature file | -Defender |
| DNS Server role | -Package Microsoft-NanoServer-DNS-Package |
| Desired State Configuration (DSC) | -Package Microsoft-NanoServer-DSC-Package |
| Internet Information Server (IIS) | -Package Microsoft-NanoServer-IIS-Package |
| Host support for Windows Containers | -Containers |
| System Center Virtual Machine Manager agent | -Package Microsoft-NanoServer-SCVMM-Package<br>-Package Microsoft-NanoServer-SCVMM-Compute-Package |
| Network Performance Diagnostics Service (NPDS)<br>(Note: Requires Windows Defender Anti-Malware package, which you should install before installing NPDS) | -Package Microsoft-NanoServer-NPDS-Package |
| Data Center Bridging (including DCBQoS) | -Package Microsoft-NanoServer-DCB-Package |
| Deploying on a virtual machine | -DeploymentType Guest |
| Deploying on a physical machine | -DeploymentType Host |
| Secure Startup | -Package Microsoft-NanoServer-SecureStartup-Package |
| Sheilded VM<br>(Note: This package is only available for the Datacenter edition of Nano Server) | -Package Microsoft-NanoServer-ShieldedVM-Package |

---

And that is just the Features, there are many other arguments you can pass when building the WIM, such as copying files to the image and first run setup scripts. 
Anyway, lets dive into my experiense with this.

So, I already knew I did not want to use the `New-NanoServerImage` cmdlet, I was pretty sure that this was just going to use DISM and inject the packages and commit, I can do that myself, so thats what I did.
I Copied the `NanoServer.wim` file from the Server 2016 media and added as an Operating System Image in SCCM. 
I then created a Task Sequence that foramted and partioned the disk, and then layed down this WIM. 
I did a quick test build on a VM and did a quick test build, it worked just fine, but a base Nano image with no remoting capabilities and no features is pretty usless.
Next I took the entire `Package` directory from the Server 2016 media, and just packaged it right up in SCCM, its only like 100 MB, so why not!
I then added a task sequence step using that package to add the Compute package (Hyper-V):
```
cmd.exe /C Dism.exe /image:C:\ /Add-Package /PackagePath:Microsoft-NanoServer-Compute-Package.cab /PackagePath:en-US\Microsoft-NanoServer-Compute-Package_en-US.cab
```
So, there is the Package: `Microsoft-NanoServer-Compute-Package.cab`, but there is also the Lang package in the `en-US` folder: `en-US\Microsoft-NanoServer-Compute-Package_en-US.cab`, not sure if both is needed, but I add both.
did another build, and then when I signed into the Recovery Console I could see the VM section, works awesome.

Now, at this point, I went ahead and added steps to the task sequence to add all the packages, the installation of them is controlled VIA a variable on the computer. 
I am not going to go into how that is created as its not directly part of this blog post, but the gist of it is when the variable `pkg_Compute` exits on the computer running this task sequnce, the Hyper-V nano package will be added. 
The edition is controlled the same way, via a Variable, there are 2 image indexes in the NanoServer.wim, so there are two `Apply Operating System` steps, only one executes via the existance of an `opt_Datacenter` Variable. 
At this point, I thought I was good to go, I could dynamically build any combo of Nano features to edition at runtime in WinPE with SCCM, or so I thought.


After building two production boxes, I went to go access one with PowerShell, using `Enter-PSSession -ComputerName MyNanoServer`, yeah well, guess what, it didn't work. 
I couldn't remotly access the machines, and a machine with no console that cannot be remotley administered, is well, completly useless. 
When I accessed the Recover Console, I went into the Firewall section, and nothing, not a single firewall exception, great. 
After about 5 seconds of research, I found out there is a switch for the `New-NanoServerImage` cmdlet, `-EnableRemoteManagmentPort`. 
Basically, this enables WinRM and opens the firewall for remote manamgment, this is a completely headless server, how is this not a default option.....? 
Ok, well, I'm building with SCCM, basically using DISM for everything, how am I going to run `winrm /qc` on an unmounted wim.......? 
Right, I'm not. 
Great, well, if that cmdlet can do it, there has to be a way to do it right? 
Well, the `NanoServerImageGenerator` PowerShell module is a .psm1, so I opened it up and started digging, and to simplify it all down, it is adding a batch script that runs at first start up that is setting up the Firewall for WinRM:

```batch
netsh advfirewall firewall add rule name="WinRM 5985" protocol=TCP dir=in localport=5985 profile=any action=allow
netsh advfirewall firewall set rule group="@FirewallAPI.dll,-29252" new enable=Yes
```

There is a lot of other stuff going on at this point, like if you add startup commands, it's adding to this batch file, but if you are just using the `-EnableRemoteManagmentPort`, then this is what its doing. 
After the batch file is complete, it is adding it to `%WinDir%\Setup\Scripts\SetupComplete.cmd`, so I just did the same thing, I packaged up .cmd files with those commands, and I copy it to that location and it runs on first boot. 


Whew, good to go right, I can now lay any combo of Nano features/roles, and I can remotely manage it....almost, forgot one important thing, joining the domain. 


After a lot of digging, there is not native way to join the domain with Server Nano, yeah, you read that right, you cannot join the domain. 
Most of the blogs I found talk about using djoin.exe and doing an offline join, sounds gross, especially at scale, I didn't even want to remotly look at using that method in this build. 
Luckly I stumbled accross this [blog](http://www.ephingadmin.com/deploy-nano-server-sccm-task-sequence/) with a way better way of doing it. 
It uses this [PowerShell Script](https://gist.github.com/Ryan2065/79838b78643d2311d60cb6147e3b87bf) (huge thanks to [@Ryan2065](https://github.com/Ryan2065)) to do the domain join with the following syntax:

```
WinPENanoDomainJoin.ps1 "ComputerName" "Domain.org" "DomainJoinAccount" "DomainJoinAccountPassword" "SMSTSOSDriveVariable"

# example
# WinPENanoDomainJoin.ps1 "%_SMSTSMachineName%" "mydomain.com" "computerjoiner" "lkajsdoifjeoijuasdf" "%OSDisk%"
```

I basically just packaged this script and added this step to the end of my task sequence, good to go!

---

Wow, I know this has been a lot, and I generally don't blog this long, and this is a 4 days or so, complied into a one post. 
The bottom line is that there should be a way better way to deploy Server 2016 Nano at scale, but for now this is the best I have come up with.
Anyway, hopefully this helps someone, somewhere, sometime, in some way.

Keep automating!
