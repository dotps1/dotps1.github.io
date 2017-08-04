---
layout: post
section-type: post
title: "DSC Pull Server Creating Excessive edb Logs"
date: 2017-04-21 19:30:00 -0400
categories: PowerShell, DSC
---

### The prodigal son returns...

So, its been a while since my last post, things have been busy, new job, baby on the way, remodeling, the list goes on and on.  But I'm back baby, and today I wanted to talk about a recent experience I had while using DSC and a Pull Server.  It left me banging my head against the wall and super frustrated, but I finally figured it out, and I'm not exactly sure why what happened, happened.

---

### In a galaxy far far away...

First, let me break down the situation.  I have been tasked with setting up "PAWs" for my team at work.  If you don't know what a _PAW_ is, it's a [Privileged Access Workstation](http://aka.ms/cyberpaw).  In a nutshell, it's a locked down, secure workstation, that you run your administraive tasks from.  And as such, limit the software that is on it.  Now, I still had some software and configurations I need to have on these machines, and I would like to beable to audit/report on the compliance of them.  I did not want to install any configuration clients, IE ConfigMgr, so I thought this would be a perfect time try out DSC for something other then lab environment shenanigans.  Now, I'm going to spare you the details of setting up the DSC Pull Server, Certificates, LCM configs, and the actual PAW Config, because even thought that did indeed take a little time, it did go pretty smoothly.  After just a few hours I had clients registered and pulling configs from the pull server, and I could see the reporting data.  However, even with only four or five clients registered, I noticed 5-15 edbXXXXX.log files (The 'XXXXX' seemed like random characters) where being created almost every minute in the directory where my Devices.edb existed (the file location vaires based on your Pull Server config). So, after a day or so in that directory, there would be 1200+ log files.  And after about a week, there was 120K+ log files.  They where not large in size (128k) but none the less, this is not going to fly.  And, the fix (if you can call it that) in my case eneded up being pretty elusive.

---

### Stay a while, and listen...

After some chatting on the PowerShell Slack channel and a small amount of google-fu, based on this [MSDN](https://msdn.microsoft.com/en-us/powershell/wmf/5.1/dsc-improvements) Artical, the fix looks pretty easy, just add an entry to the web.config, restart the IIS AppPool and I should be good to go!

>In eariler version of DSC-PullServer, the ESENT database log files were filling up the disk space of the pullserver becouse the database instance was being created without circular logging. In this release, customer will have the option to control the circular logging behavior of the instance using the web.config of the pullserver. By default CircularLogging will be set to TRUE.

```xml
<appSettings>
  <add key="dbprovider" value="ESENT" />
  <add key="dbconnectionstr" value="C:\Program Files\WindowsPowerShell\DscService\Devices.edb" />
  <add key="CheckpointDepthMaxKB" value="512" />
  <add key="UseCircularESENTLogs" value="TRUE" />
</appSettings>
```

After checking my web.config, sure enough, that key didn't exist, so I added it, set it to true and restarted the AppPool, deleted all the existing logs, and moved on to something else.

That night I was online for something else, and just happened to check on it, and really _not_ to any surprise, there where about 700+ log files.  Well, what the hell!  So I did what anyone would do, went back to the MSDN article, re opened my web.config to check for typos, I know sometimes when you copy-pasta from the inter-webs the quotes get all funky, but that was not the case.  So, maybe just reseting the AppPool isn't enough?  Just to be sure I rebooted the server.

After the Server came back online, almost instantaneously there where 10+ logs, and 50+ within 10 minutes while I did more research.  I came across a GitHub Article (cannot find link at the time of this writing) that mentioned creating a scheduled task to take care of this.  Now, if you know me at all, I cannot stand arbitrary work-arounds like this.  I mean, when do I schedule it to run?  Every minute, 5 minutes, 25 minutes, 1 hour and 6.5 minutes?  So, If I was going to do something like this, then I was going to turn auditing on for file creation on that folder, and tie the task to the file creation event, so at least if the task did run, it was running with some sort of _reason_.  Anyway, halfway through configuring this, I stopped, and said to myself, "This is not a fix!  Stop implementing this half-ass hack!".

After stumbling across a [GitHub Article](https://github.com/PowerShell/xPSDesiredStateConfiguration/issues/296) that was somewhat close to my issue, in this case the Devices.edb was just growing rediculious large, I left a comment of what I was seeing, and I opened up my own [GitHub Issue](https://github.com/PowerShell/DscResources/issues/265) as well, but on a different PowerShell DSC Repo, just so I could hopefully get some answers.

The next day, a fellow GitHub-er did indeed respond (thank the dark sith lord), he pointed out that I needed to have an updated dll for the `UseCircularESENTLogs` setting to take effect.  I mean that does make sense, however I hadn't seen anything in that MSDN blog about that, I had guessed it was a setting that was left out of the default web.config.  Luckly for me, he left me what needed to be updated, and thankfully, what version the dlls _should_ be!

>Install all OS patches and do this:
Simply stop the Website and the App Pool and then replace the Microsoft.Powershell.DesiredStateConfiguration.Service.dll and Microsoft.Powershell.DesiredStateConfiguration.Service.Resources.dll with the ones from C:\Windows\System32\WindowsPowerShell\v1.0\Modules\PSDesiredStateConfiguration\PullServer (Version 10.0.14393.479)

Awesome, so I hop on my Pull Server, navigate to `C:\inetpub\wwwroot\DSCPullServer\bin` check the properties of the `Microsoft.PowerShell.DesiredStateConfiguration.Service.dll` and sure enough, its `10.0.14393.0` (Server 2016 core mind you, so did this all with PowerShell of corse).  So far so good.  Next I run `cmd`, then `sconfig` hit the windows update section, check for update, all updates, and nothing!  Ok, so now what, how do I update these dlls?  Is it WMF 5.1 patch?  Is it a Windows Server patch?  Is there stand-alone DSC Pull Server updates?  Yeah, I didn't know either.  

After some more interigating _the google_, I decided, it had to be a Windows Update of some kind.  I tried running `sconfig` once more (no idea why, I think I was falling off the deep end by this point, after nothing but deadends), to no suprise, no updates.

Now, I remember a couple of months past, I had built some Windows Server 2016 boxes and I had updated them with `sconfig`, all updates.  And then right after handing them off, I recall someone installing PSWindowsUpdate and then checking for updates, and finding more.  Mainly drivers.  Anyway, I said to myself, what the hell, might as well give it a try.  So I installed PSWindowsUpdate, `Install-Module -Name PSWindowsUpdate -Repository PSGallery -Confirm:$false`.  After that completed, I tried looking for updates, `Get-WindowsUpdate` and to no shock really, nothing returned.  Now, I have used PSWindowsUpdate once or twice, but I don't know it very well, so I wanted to see what options `Get-WindowsUpdate` had, you know, maybe like a `-UpdateDscPullServerDll` switch or something.  So I typed `Get-WindowsUpdate -` and then hit it with some PSReadLine Badass-ery, `ctrl+space` to reveal all the parameters, and I noticed the `-MicrosoftUpdate` switch and the `-WindowsUpdate` switch.  What are those for?  I don't typically read the Help (I know, shame on me), I more like to trial and error, especially stuff like this.  I don't experiment with production `Set-ADUser` or anthing like that, but as for this, who cares, its working like garbage anyway!  So I typed it out and smashed the enter key, `Get-WindowsUpdate -MicrosoftUpdate`. _Insert huge suspensful pause here_...

>WARNING: Can't find registered service Microsoft Update. Use Get-WUServiceManager to get registered service.

Ok, ok, so I guess I'll abide this _Warning_, `Get-WUServiceManager`

```
ServiceID                            IsManaged IsDefault Name
---------                            --------- --------- ----
855e8a7c-ecb4-4ca3-b045-1dfa50104289 False     False     Windows Store (DCat Prod)
Redacted                             True      True      Windows Server Update Service
9482f4b4-e343-43b6-b170-9a65bc822c77 False     False     Windows Update
```

Ok, looks like I have a few `WUServiceManager`, you know, things.  But where is the Microsoft Update one?  After running `Invoke-GoogleQuery -Query "PSWindowsUpdate MicrosoftUpdate WUServiceManager"` (not a real cmd, but I think you get the idea), I grabbed one of the top results ([link](https://www.windowsbbs.com/threads/powershell-for-automatize-windows-update-and-program-installation.110303/)).  Just have to Register the Microsoft Update service (why does this not exist Out Of Box?  Because, _because_ is the best answer I have for that).  `Add-WUServiceManager -ServiceID 7971f918-a847-4430-9279-4a52d1efe18d -Confirm:$false`.  That actually registers a `WUServiceManager` with a name of like `com__object` which worried me a little, but re running the `Get-WUServiceManager` cmdlet will show you what you'd actually expect.

```
ServiceID                            IsManaged IsDefault Name
---------                            --------- --------- ----
7971f918-a847-4430-9279-4a52d1efe18d False     False     Microsoft Update
855e8a7c-ecb4-4ca3-b045-1dfa50104289 False     False     Windows Store (DCat Prod)
Redacted                             True      True      Windows Server Update Service
9482f4b4-e343-43b6-b170-9a65bc822c77 False     False     Windows Update
```

Ok, awesome sauce, lets try it again!  `Get-WindowsUpdate -MicrosoftUpdate`.  Sure enough, finds 4 updates, and one of which looked pretty important, [KB4015217 - Cumulitive Update for Windows Server 2016](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217).  So, I went ahead and installed it, `Install-WindowsUpdate -KBArticleID KB4015217 -MicrosoftUpdate -Confirm:$false`.  Now, it was 1GB and did take a little bit, but after a reboot, I navigated to `C:\Windows\System32\WindowsPowerShell\v1.0\Modules\PSDesiredStateConfiguration\PullServer` and checked out the properties of `Microsoft.PowerShell.DesiredStateConfiguration.Service.dll` and sure enough, it was the right version, `10.0.14393.479`.  _Maybe_ I'm finally on the right track!

I grabbed those two dlls, and dropped them in my virtual directory, and restarted the server, not even going to trust just restarting the AppPool to take care of it.  When the server came back up, I logged in, wacked all half a million temp log files, went to my GitHub issue, updated the details and closed it.  Came back to the server, to log off, hit one more `Get-ChildItem` just to make sure no temp logs where created, results came back, seven flipping log files!  *ARE YOU KIDDING ME!*

At this point I had to just walk away for a few moments.  When I came back, I re-opened my GitHub Issue with a big fat _NVM_, and then just went to work on some other stuff (other stuff being trying to move the Devices.edb to an .mdb database, but we will save that headache for another blog post, I was trying to do that, because I figured, no .edb, no edb log files, maybe...).

After about an hour of messing with the _other stuff_, I ran a `Get-ChildItem` again, and the results, for once, _were_ surprising.  There where only about eight log files, and all of them had been modified in the last few minutes.  Huh, strange.  So I continued to monitor for about a half hour.  As it turns out, I guess this _is_ expected behavior.  There will still be temp log files, however, they will be managed by the DscService.  So there is nothing to worry about if there are a _few_ of them (I would say, less then 50).  I assure you, I did not find that documented anywhere, and I mean _anywhere_.

In conclusion, I really have no idea why Windows Update via `sconfig` did not see these updates, I also have no idea why the documentation on how a DSC Pull Server is _supposed_ to work is so limited.  It seems that DSC is still very much immuature so the knoweledge out there and blog posts are still very limited (hence this _extenisive_ writing).  

May my new accumulation of gray hairs, hopefully spare you a few.

Party on Wayne.

---

### Lets break it down...

Question: "How do I fix my Windows Server 2016 DSC Pull Server from generating 100 million edbTEMP0.log files?"

Well........

* RDP to the PullServer.
* `Get-Item  -Path C:\Windows\System32\WindowsPowerShell\v1.0\Modules\PSDesiredStateConfiguration\PullServer\Microsoft.Powershell.DesiredStateConfiguration.Service.dll | Select-Object -ExpandProperty VersionInfo | Select-Object -ExpandProperty ProductVersion`
* Output: 10.0.14393.0
* Run `sconfig`.
* Check for updates.
* Use option 'A' to check for "all updates".
* No results.
* Install PSWindowsUpdate Module from the PSGallery `Install-Module -Name PSWindowsUpdate`.
* Check for updates with PSWindowsUpdate `Get-WindowsUpdate`.
* No results.
* Started looking at parameters of `Get-WindowsUpdate`, noticed the `-MicrosoftUpdate` switch.
* Run command `Get-WindowsUpdate -MicrosoftUpdate`.
* Abide warning: `WARNING: Can't find registered service Microsoft Update. Use Get-WUServiceManager to get registered service.`
* Google why.
* Found this [blog](https://www.windowsbbs.com/threads/powershell-for-automatize-windows-update-and-program-installation.110303/).
* Register the Microsoft Update Service Manager: `Add-WUServiceManager -ServiceID 7971f918-a847-4430-9279-4a52d1efe18d -Confirm:$false`
* Check for updates: `Get-WindowsUpdate -MicrosoftUpdate`
* Found four updates, including `KB4015217` which is a cumulative update for Server 2016.
* Install update `Install-WindowsUpdate -KBArticleID KB4015217 -MicrosoftUpdate`
* Reboot to finish.
* `Get-Item  -Path C:\Windows\System32\WindowsPowerShell\v1.0\Modules\PSDesiredStateConfiguration\PullServer\Microsoft.Powershell.DesiredStateConfiguration.Service.dll | Select-Object -ExpandProperty VersionInfo | Select-Object -ExpandProperty ProductVersion`
* 10.0.14393.479
* Update dlls in virtual directory.
* Reboot the machine (probably just need to restart the AppPool).
* Accept there will still be edb temp logs, but they will be managed.
* Open and consume well deserved beer.
* Repeat previous step.