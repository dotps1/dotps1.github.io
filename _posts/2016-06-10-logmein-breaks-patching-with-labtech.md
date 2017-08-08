---
layout: post
section-type: post
title: LogMeIn Breaks Patching With LabTech
category: labtech
---

I recently accepted and moved in to a new role, and one of my first tasks was to figure out why patching was failing on clients.
The updates where being applied using Ignite VIA LabTech.
The updates continuously failed with the following error:

```
Error Occurred Durring Install: Exception from HRESULT: 0x8024001E
```

So I did all the normal Windows Updating troubleshooting steps.

* Run the Windows Repair Tool
* Update the Client
* Reset and re-register Windows Update Components
* Rename the SofwareDistribution Folder
* Run the Readiness Checker

And about a million other things, and nothing was working.  I reached out to LabTech Support, and he pointed me to this [article](https://docs.labtechsoftware.com/knowledgebase/article/12175).
I was reading it through, it has a lot of info, then, one of the last things I read was this (despite it saying 'This is the most common HRESULT'):

```
HRESULT 0x8024001E

This is the most common HRESULT. In most all cases this error indicates the Patching process on the agent was interrupted by another patching process or by a restart of the WUAUSERV. The WUAUSERV process is a single user application. It cannot operate multiple calls from different sources. Please follow the below:

Verify that under the Admin >  Schedules > Desktops | Laptops | Servers, the update Configuration is not occurring during a patching cycle. The Update Config performs a WUAUSERV restart and aborts the patching process.
https://docs.labtechsoftware.com/knowledgebase/article/7374
Check for other 3rd party Patch analysis applications such as LogMeIn, and even some full AV software that is checking for Patch Health. When possible re-schedule those applications to keep them away from your LabTech patching schedule, or disable those function in those 3rd Party Apps totally. To disable LogMeIn patching:
    Go into the registery and navigate to HKLM\SOFTWARE\LogMeIn\V5\PatchMgmt
    Enter a Dword in the field: PatchMgmtEnabled
    Set the value to 0
    Disable WUAUpdate = dword:00000001
    WUAStatus = dword:00000000
    MicrosoftUpdate = dword00000000
    Reboot the computer
```

And it hit me!  I was seeing this in the WindowsUpdate.log, but never thought anything of it:

```
2016-06-10	00:48:00:161	1084	204c	AU	Successfully wrote event for AU health state:0
2016-06-10	00:48:00:162	1084	204c	AU	Successfully wrote event for AU health state:0
2016-06-10	00:48:00:162	1084	204c	AU	AU finished delayed initialization
2016-06-10	01:48:00:408	2560	cb0	COMAPI	-------------
2016-06-10	01:48:00:408	2560	cb0	COMAPI	-- START --  COMAPI: Search [ClientId = LogMeIn]
2016-06-10	01:48:00:408	2560	cb0	COMAPI	---------
2016-06-10	01:48:00:432	2560	cb0	COMAPI	<<-- SUBMITTED -- COMAPI: Search [ClientId = LogMeIn]
2016-06-10	01:48:00:432	1084	1e38	Agent	*************
2016-06-10	01:48:00:432	1084	1e38	Agent	** START **  Agent: Finding updates [CallerId = LogMeIn]
```

But sure enough, every time the Patch Cycles where running, flipping LogMeIn would stick its nose in there just to see what was up, and break everything.

I wrote a PowerShell script to correctly set the Registry Values to disable the LogMeIn Patch Management 'Feature', and restart the LogMeIn services to correct this issue.  
I felt it was more important for LabTech to actually Patch the clients, rather then LogMeIn tell me they are not patched.  Patching is working great after applying this fix.

<script src="https://gist.github.com/dotps1/e3b5bf37aea3a84b0a40c20d405cdc97.js"></script>
