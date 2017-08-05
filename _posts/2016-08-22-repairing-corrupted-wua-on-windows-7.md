---
layout: post
section-type: post
title: Repairing Corrupted WUA on Windows 7
date: 2016-08-22 09:30:00 -0400
category: powershell
---

This article is an expansion on from my last blog post: [LogMeIn Breaks Patching With LabTech](http://dotps1.github.io/logmein-breaks-patching-with-labtech.html).  After
addressing the LogMeIn Patch Management issue with the Windows Update Agent, I experienced a slew of other issues that where hindering my abilities to successfully patch Windows 7 
clients.

Here is a general overview of what I was attempting to overcome:

1. Several different environments, mostly comprised of Windows 7 machines.
2. These machines had WUA v7.6.7601.19161 or less.
3. Almost all machines are failing to update with error code 0x8024001E.

Now, from my last blog we know that LogMeIn likes to put its grubby paws into the WUA to look for updates, but that has been disabled (see post for script).  So, 
why is this error still happening?  Well, I have a few different answers to that question:

First, I have come to learn that the WUA seems to watch the HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate registry key.  And if any changes are written to the Key or its Property
values, the wuauserv service will restart itself.  Awesome right?  So, what this says to me is a GPO is configuring something Windows Update related.  This turned out to be the case.  But 
I had already disabled the GPO titled _Windows Update Settings_ so wtf?  Guess what, there was a setting in the Default Domain Policy that was configuring Windows Updates.  So, every 90
minutes (plus or minus 30 minutes) the GPO processed, and when it did this during patching, well, it broke, throwing error code 0x8024001E.  So now what, I have to dig through every 
setting in every GPO in every domain and make sure there isn't a Windows Update setting in them?  The answer is yes, but there is no way im doing that manually.  Luckily with the 
GroupPolicy PowerShell Module its pretty easy to find which GPOs have settings defined for Windows Updates:

```powershell
#reqiures -Modules GroupPolicy

foreach ($gpo in (Get-GPO -All)) {
    $xml = [Xml](Get-GPOReport -Guid $gpo.Id -ReportType Xml)
    if (($xml.GPO.SelectNodes("//*[local-name() = 'Name']")."#text" | Where-Object { $_ -eq "Configure Automatic Updates" }).Count -gt 0) {
        Write-Output -InputObject "The following GPO is configuring Windows Update Settings: '$($xml.GPO.Name)'."
    }
}
```

Now, after finally making sure that both LogMeIn and Group Policy are done tampering with our WUA, we can finally get to addressing the big issue: getting these clients up to date.  Microsoft
released an Update Rollup in June, [KB3161608](https://support.microsoft.com/en-us/kb/3161608).  One of the biggest issues with WUA v7.6.7601.19161 (which is the March update) is that 
it is dog slow, I mean ridiculous slow.  So, getting these boxes to the latest WUA is crucial, because they have not been updated in quite some time, so I need them not only to update, but 
update fast.  But wait, in July Microsoft released another Update Rollup that supersedes KB3161608, [KB3172605](https://support.microsoft.com/en-us/kb/3172605).  This update has the 
WUA version we want, 7.6.7601.23453.  This update agent is working 10x better then its predecessor.

But, it cannot be that easy, just apply KB3172605 and then updates will just install like magic?  Of course not, I tried to push that update to a few Windows 7 clients, and I kept 
getting _This update is not applicable to the computer_.  Come on!  Just install!  After digging in, again, I found at the bottom of the page for KB3172605, there is a prerequisite 
the April 2015 servicing stack update for Windows 7 and Windows Server 2008 R2 is required [KB3020369](https://support.microsoft.com/en-us/kb/3020369).  Well that's great, the clients 
havent been patching , so of course most of them do not have this update installed.  And, updates are not installing so now what?  Well the first thing I did was get on the machine and 
try running the KB3020369.msu, and it the dialog comes up says _Scanning this computer for installed updates_.  And I wait, and I wait, and I wait.  I left the computer scanning and 
went home.  Came back in the next morning, and there was an error from scanning for updates!  BAH!

Lets dig deeper down the rabbit hole!  I ran a Microsoft FixIt on the computer, which basically reset the entire Windows Update Agent ([link](https://support.microsoft.com/en-us/kb/971058)).  Then 
I ran the KB3020369.msu and it worked!  And fast, didn't even have to reboot!  I went ahead and ran the KB3172605 right after, and it worked as well.  That update did require a reboot, 
after it came back up, I pushed updates to it, and like magic, its updating, and fast!  pushed 90+ updates in under an hour.

So to break it all down, here is the order to fix everything:

1. Disable the LogMeIn Patch Management (if applicable).
2. Make sure there are no GPOs messing with your Windows Update settings (if applicable).
3. Reset the WUA (_See update below_).
4. Install KB3020369.
5. Install KB3172605.

And lastly, running the Microsoft FixIt on these machines manually is not very ideal, so, here is a PowerShell script to do it for you, it can be found [here](https://gist.github.com/dotps1/8abb564c6dbfb1b768354b39ede033da).

---

**Update (2016/09/07)**

After some more troubleshooting with this, I found that reseting the WUA didn't _always_ work, it did most of the time, but not _all_ the time. 
So after some more digging, I found that applying the updates with DISM seem to do the trick.
Here is what you need to pull it off

1. First download the update, KB3020369 or KB3172605.
2. Expand the update to get the .cab file out of it, `expand -f:* <PathToKBmsu> <PathToExtractTo>`
3. Install the .cab with DISM: `DISM.exe /Online /Add-Package /PackagePath:<PathToExtractedCab> /Quiet /NoRestart`

And lastly, if you are using LabTech, here is a [Zip Package](https://onedrive.live.com/download?cid=E508E6CF25AFDF2F&resid=E508E6CF25AFDF2F%2113032&authkey=AO8zK2whHWr1MTY) with all the files you need to pull this all off!

Hope this helps, and good luck!
