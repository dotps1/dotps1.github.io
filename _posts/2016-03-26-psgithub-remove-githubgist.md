---
layout: post
title: "Added Remove-GitHubGist Cmdlet to PSGitHub PowerShell Module"
date: 2016-03-26 01:00:00 -0400
categories: PowerShell
comments: true
---

Added some more functionality to the Gist cmdlets in **[PSGitHub PowerShell Module](http://pcgeek86.github.io/PSGitHub/)**.  Currently the following Gist cmdlets exist in the module with much more functionality to come:

- Get-GitHubGist
- New-GitHubGist
- Save-GitHubGist
- Set-GitHubGist
- Remove-GitHubGist

Today I added the `Remove-GitHubGist` cmdlet to the module.  This cmdlet is a little invasive with its default param set, it will remove the entire Gist, commits, comments, files, everything.
However, you can also use the `-FileName` parameter to specify one or more files to delete from a specified Gist.

Here is some examples of using these cmdlets.  (The `Set-GitHubToken` cmdlet needs to be run first before any other cmdlet in this module can be used).

```powershell
# Example 1
# Create and new Testing Gist with the contents from Test-File.ps1.
$gist = New-GitHubGist -File .\Test-File.ps1 -Description 'Testing Gist' -Public
# Remove the entire Gist.
Remove-GitHubGist -Id $gist.Id -Confirm:$false

# Example 2
# Remove the entire Gist.
Get-GitHubGist -Id 123456abcdef | Remove-GitHubGist -Confirm:$false

# Example 3
# Remove File2.ps1 and File3.ps1 from the Gist. 
Remove-GitHubGist -Id 123456abcdef -FileName File2.ps1, File3.ps1 -Confirm:$false
```

This module is coming along nicely.  Thanks to **[Trevor Sullivin @pcgeek86](https://trevorsullivan.net)** for allowing me to be a part of it.  Really enjoying the collaboration.