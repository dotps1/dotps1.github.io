---
layout: post
section-type: post
title: "HP Servers, EMC shelf, and Storage Pools"
category: server
---

Recently at work I was asked to build a PoC for [Storage Spaces Direct](https://docs.microsoft.com/en-us/windows-server/storage/storage-spaces/storage-spaces-direct-overview).  The premiss is actually pretty interesting.  Basically, you take two servers, pack them full of local storage, set up a storage pool between them and then set up an HA CSV between them and then run VM's on top of it.  The cool part is for the HA, all of the IO writes are synchronous.  This means, if one of the hosts where to die, your VM would be _exactly_ how it was on the dead host, on the still functioning one.  This could be important for applications that even a few seconds of latency would cause a bunch of errors in the event the server VM needed to fail over to the other host.  Especially if the VM is running on the host that dies, because you would still lose CPU and Memory from the host.  So the VM would actually have to be restarted, but there _should_ be no data discrepancy between the hosts and the VM's vhdx file.
 
 Well, gather around children, I am going to take you on a journey of learning, and frustration as I attempted to setup a Storage Space Direct PoC.

I began my journey with the guidance of this [article](https://docs.microsoft.com/en-us/windows-server/storage/storage-spaces/hyper-converged-solution-using-storage-spaces-direct).  I grabbed two HP DL380 G7s, that had previously been used, and had two 300 GB HDDs in a RAID1 via the onboard HP 410i Smart Array controller (the model is important).  These where just hypervisors that had Windows installed to that LUN, nothing special.  So I tossed in a 1TB HDD, and configured a RAID0, I figured we'd use these for the storage pool.  I also added a dual port 10GB Chelsio NIC to them.  The HP NICs that where in the systems don't support iWARP, and I'm not even sure if they support RoCEv2, which you will want so you can use RDMA and Jumbo Packets for the synchronous read/writes to the CSV.  So, I installed Windows Server 2016 Core Standard (keep in mind that last bit for later), updated the OS, and started to follow along the blog and configuring the NICs.  Now, one difference is that I did not have a _TOR_ switch (Top of Rack Switch, love that), so I just hooked the Chelsio cards together and gave them each static IPs on there own little network.  So, everything was going great until I got to the `Validate-Cluster` command.  The report (which is created in `$env:temp` by the way, the FileInfo object returned does not mention that) kept saying there where no supported drives for S2D (S2D is the acronym for Storage Spaces Direct).  And after further review, it was correct.  You cannot use drives with a RAID BUS type.  That 1TB drive was a solo drive in a RAID0.  Well great, now what?

 _Googles 'How to enable pass thru on hp smart array 410i'_...... long story short, you can't, removing the RAID functionality and setting the disks to pass thru are supported with the 420i and newer controllers, but not the 410i, great, just great.  So I went to see what else I could find for hardware to make this work.  I found we had two HP DL360p G8s, checked the Smart Array model, and luckily, they where 420i's so they _should_ work, awesome, moved the Chelsio cards to them, I booted to the latest SPP, made sure the Smart Array Controller was at the latest firmware, and from there you can also configure it, so I changed it to use Pass Thru mode.  I got Server 2016 core installed again, updated, configured all the networking, ran the validation tests again, S2D passed, but this time I got another failure, `OS Edition not supported`!  Are you kidding me!?  Sure enough, S2D is only supported on Datacenter edition.  So, Microsoft's attempt at helping you save money by creating a Hyper-Converged solution by reusing hardware is immediately shutdown by making you have to by an OS edition so expensive, its not even worth it.  What a shock.  I'm going to finish this part up quick so I can get to the actual solution we are going to use.  I went ahead upgraded the edition to Datacenter just to finish this test and see if it actually works, and it does, its pretty cool, its really cool if you have some kind of deal from Microsoft where you can just use Datacenter licenses willy nilly.

 So, onto my next idea, two servers, external LSI cards, and an old EMC shelf we have laying around.  Build a storage pool on the shelf, add a CSV, and use that, basically the same build, minus the local storage.  If only it where so easy...........

So, time to get back to where I started, I moved the Chelsio cards back into the G7s, as those are the servers we are going to use for this project.  I also added an LSI SAS card to each server to interface with the shelf.  We have a few old EMC shelves laying around with a buttload of storage in them (yes, buttload is an actual unit of measurement, [buttload](https://www.quora.com/If-someone-says-to-%E2%80%9Cbring-a-buttload-of-the-stuff-%E2%80%9D-how-much-should-you-bring-How-much-is-a-buttload-precisely-in-terms-of-a-numerical-measurement)), so I was hoping to find a use for them.  I re installed Sever 2016 Core Standard, _not Datacenter_, because I already know how to make it work with that edition.  Updated the OS, and then proceeded to prep the drives for Storage Spaces following steps from this [TechNet article](https://technet.microsoft.com/en-us/library/jj822937(v=ws.11).aspx).  And this is where things get a little interesting.  I started by making sure all the disk _CanPool_.  This is done with the `Get-PhysicalDisk` cmdlet.  Well, when I ran that, I noticed something else.  The EMC shelf had 24 HDDs, and I was only seeing about seven drives in the output from `Get-PhysicalDisk`, and all of them had a value of `False` for _CanPool_.  After some troubleshooting I found some, well, bad news.  EMC formats the HDDs with a 520 byte block sector, rather then the typical 512 byte.  I found a few blog posts out there explaining ways to rewrite the block size with [sg3_utils](http://sg.danny.cz/sg/sg3_utils.html).  But it sounded very time consuming, and I hate UNIX utilities, my experiences with them never go smooth.  And, this time was no different.  I downloaded the [System Rescue CD](http://www.system-rescue-cd.org/Download/) because it had the sg3_utils already included.  And after a lot of trial and error, I finally found the command would format and rewrite the block size:

```
# /dev/sg<ID> can be found with sg_scan -i
sg_format --format --size=512 --six -v /dev/sg<ID>
```

The EMC shelf had 24 900GB drives, and to format one, took about 4 hours, so yeah, I should be able to do the rest over the course of a week!  _Ain't nobody got time for that!_  I said to myself, 'if only this was a Windows utility, I could run this with PowerShell Jobs and do the remaining 23 in parallel'.  Finally, some good news, if you scroll all they way past all the .deb downloads on the download page for sg3_utils, there is a version for Windows, the link just does not stick out at all.

> The sg3_utils-1.42exe.zip file is a zip archive of Windows 32 bit executables made in a MinGW environment. 

So, after downloading and extracting that archive, I was able to find the drives with the sg3_utils executables.  But the syntax is a little different:

```
# command to identify the drives.
.\sg_scan -s
```

That will find the drives and give the disk number in the format of `PD<Int>`, IE `PD1`.  So, with this information we can using PowerShell Jobs and format and rewrite the block size of all the drives at the same time.

```
# Create an array of the PD numbers and pipe them to the Start-Job command.
(2..24) | %{ Start-Job -ScriptBlock { C:\sg_format.exe --format --fmtpinfo=0 --pfu=0 --size=512 --six -v "PD$($args[0])" } -ArgumentList $_ }
```

The `--fmtpinfo=0 --pfu=0` removes the drive protection if there is any.  Now, one downfall to using PSJobs, is that you cannot see the status of the format, but you can retrieve some data in two ways

1. `Get-Job -Status Running`.  This will tell you how many drives are still formating, but not give you the percentage.
2. `.\sg_format -v PD<Int>`.  Run the format executable with no arguments but `-v` which is for verbose output.

So, after a few hours, after all the drives had completed, I was able to run `Get-PhysicalDisk` and actually see all the drives.  However, they where still all `CanPool: False`.  I dug in a little bit deeper, `Get-PhysicalDisk | ? BusType -eq "SAS" | select -First 1 *` (I used the bus type for the filter because my boot drive was a RAID).  The output will have a `CannotPoolReason` property, and its value was _Insufficient Capacity_.  What, its 900GB, the minimum requirement for a drive to be part of a storage pool, is 4GB.  After a quick search, it appears that if the disk has any RAID/Storage Pool/MBR/etc info on it, this error will occur.  But there is a cmdlet to help fix it, `Reset-PhysicalDisk`.  So I ran the following: `Get-PhysicalDisk | ? BusType -eq "SAS" | Reset-PhysicalDisk`.  Took a few minutes, but after it completed, I ran `Get-PhysicalDisk` and all the drives could be pooled!  Success!

After all of this headache and frustration, I was able to finish the steps, create the cluster, create the storage pool, create the CSV and run a few VMs off of it.  I had no idea there would this much that would go wrong, but hey it was one hell of a learning experience.  Hopefully this post will help someone, somewhere.  
