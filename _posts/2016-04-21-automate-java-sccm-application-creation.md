---
layout: post
title: "Automate Java SCCM Application Creation"
date: 2016-04-21 10:00:00 -0400
categories: SCCM
comments: true
---

**Overview**

---

As a Configuration Manager Administrator, one thing I always have to deal with is [Java](java.com).  I Created a new application ten days ago, 8 Update 77, and today, its up to 8 Update 91.
I really am not sure how that is even possible.  But none the less.  Creating an application is SCCM isn't particularly hard, but it does take a little bit of time, and when you need to create 4 new Java packages a day, it can take a lot of time.
Just kidding of course, I hope you are not deploying every version of Java.  But I thought why not automate the application creation with PowerShell.

It has taken a few revisions, but it is now good to go, and I use it regularly.  Here is the breakdown:

This task, I accomplished in three files, mainly because I am a huge advocate of re-usability so, two of these files (the helper functions) I use in my personal PowerShell module, for more then just this task.

* Controller Script: New-SccmJavaApplication.ps1
* Helper Function Script: Get-MsiProdcutCode.ps1
* Helper Function Script: Invoke-JavaDownloadAndMsiExtraction.ps1

**What is happening?**

---

1. Download both x86 and x64 versions of the Java Offline Installer.
2. Run the x86 exe which extracts the msi to the current users Application Data. (This will show the Java installer windows, DO NOT CLOSE THIS WINDOW! The script will do so at the appropriate time.)
3. After the msi exists, terminate all the running java process.
4. Run the x64 exe which extracts the msi to the current users Application Data. (This will show the Java installer windows, DO NOT CLOSE THIS WINDOW! The script will do so at the appropriate time.)
5. After the msi exists, terminate all the running java process.
6. Create the package directory specified in the SoucesPath.
7. Copy both msi's to that path.
8. Extract the Installer Script to the package directory.
9. Sign the installer script, (if a certificate was supplied).
10. Extract the java installer properties file to the package directory.
11. Import in the SCCM PowerShell Module using implicit remoting.
12. Create the Application Object.
13. Create the Deployment Type Object.
14. Add the Deployment Type Object to the Application Management Object.
15. Move the application to the correct folder, (if a folder was supplied, else it will exist in the root application directory).

* *The application will NOT be distributed auto-magically.*
* *The application will NOT be deployed auto-magically.  (I'm to scared to do this)*


**Customize**

---

In the Controller Script, lines 42-77 is the installer script that get generated for the application to install, this can be customized.  I use this script to actually install the application.
This will remove all versions of Java that match the current major version that is being deployed, IE: 8 Update 91 will remove all previous version of Java 8.
But this script is extracted on the fly, so it can be customized.

Also in the Controller Script, lines 80-84, are the customizations for the Java install itself, things like disabling auto update, and not adding a start menu shortcut go here.  This is also extracted on the fly.
Options for using this file can be found [Here](http://docs.oracle.com/javase/8/docs/technotes/guides/install/config.html).


**Tips**

---

I recommend using the -Verbose switch for insight into what is happening if you use this technique.  I added a lot of verbose comments for this.


**Example Usage**

---

```
# SourcePackageCreationPath: is the path to where the application source files will be stored.
# ComputerName: is the name of the Site Server.
# Credential: context with rights to do create applications.
# SiteCode: the site code.
# ApplicationFolder: the folder in SCCM where to put the application, defaults to the root of the applications workspace.
PS C:\> .\New-SccmJavaApplication.ps1 -SourcePackageCreationPath '\\mysccmserver\sources\applications\java' -ComputerName mysccmserver -Credential (Get-Credential) -SiteCode 1AB -ApplicationFolder 'Java' -Verbose
```

**Disclaimer**

---

*Use at your own risk (Obviously)*

*Can be found as a Gist on [GitHub](https://gist.github.com/dotps1/492023ebd737f9cc46aa)*

---

<script src="https://gist.github.com/dotps1/492023ebd737f9cc46aa.js"></script>