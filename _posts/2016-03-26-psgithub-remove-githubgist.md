---
layout: post
title:  "Added Remove-GitHubGist Cmdlet to PSGitHub PowerShell Module."
date:   2016-03-26 01:00:00 -0400
categories: PowerShell
tags:
- Blog
---

Added some more functionality to the Gist cmdlets in [PSGitHub PowerShell Module](http://pcgeek86.github.io/PSGitHub/).  Currently the following Gist cmdlets exist in the module:
- Get-GitHubGist
- New-GitHubGist
- Save-GitHubGist
- Remove-GitHubGist

Today I added the `Remove-GitHubGist` cmdlet.  This cmdlet is a little invasive with its default param set, it will remove the entire Gist, commits, comments, files, everything.
My thoughts are to add other param sets, allow to remove files only, commits to possibly.

Here is some examples of using these cmdlets.  (The `Set-GitHubToken` cmdlet needs to be run first before any other cmdlet in this module can be used).

```powershell
# Example 1
$gist = New-GitHubGist -File .\Test-File.ps1 -Description 'Testing Gist' -Public

Remove-GitHubGist -Id $gist.Id -Confirm:$false

# Example 2
Get-GitHubGist -Id 123456abcdef | Remove-GitHubGist -Confirm:$false

# Example 3
Remove-GitHubGist -Id 123456abcdef -FileName File2.ps1, File3.ps1 -Confirm:$false
```

This module is coming along nicely.  Thanks to [Trevor Sullivin @pcgeek86](https://trevorsullivan.net) for allowing me to be a part of it.  Really enjoying the collaboration.