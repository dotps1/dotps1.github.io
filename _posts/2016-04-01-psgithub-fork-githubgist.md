---
layout: post
section-type: post
title: "PSGitHub Fork-GitHubGist"
date: 2016-04-01 09:00:00 -0400
---

Last night I committed the [PSGitHub](http://github.com/dotps1/PSGitHub) repo, the `Fork-GitHubGist` cmdlet.  I know what your thinking, `Fork` is not an [approved verb](https://msdn.microsoft.com/en-us/library/ms714428(v=vs.85).aspx),
so the cmdlet name is actually `Copy-GitHubGist`, but I have also added an alias called `Fork-GitHubGist`.

It is pretty straight forward to use:

```powershell
# Import the PSGitHub Module.
PS C:\> Import-Module -Name PSGitHub

# If you have not done so, set your GitHub token.
# This only needs to be done once, not everytime you use the module.
PS C:\> Set-GitHubToken

# Get the Gist you would like to fork.
# I'm just going to use Trevor Sullivans latest updated Gist.
PS C:\> Get-GitHubGist -Owner pcgeek86 | Select-Object -First 1


Comments    : 0
CommentsUrl : https://api.github.com/gists/23ad223dba5c36041e21/comments
CommitsUrl  : https://api.github.com/gists/23ad223dba5c36041e21/commits
CreatedAt   : 7/9/2015 12:34:44 PM
Description : This Gist provides a suggested pattern for serializing / deserializing PowerShell v5 class instances
Files       : {PowerShell v5 Class Serialization Pattern.ps1}
Forks       : 
ForksUrl    : https://api.github.com/gists/23ad223dba5c36041e21/forks
History     : 
HtmlUrl     : https://gist.github.com/23ad223dba5c36041e21
Id          : 23ad223dba5c36041e21
Owner       : GitHubUser
Public      : True
PullUrl     : https://gist.github.com/23ad223dba5c36041e21.git
PushUrl     : https://gist.github.com/23ad223dba5c36041e21.git
Truncated   : False
UpdatedAt   : 3/31/2016 11:41:57 PM
Url         : https://api.github.com/gists/23ad223dba5c36041e21


# Looks good, now, we can either pass this object directly in Fork-GitHubGist VIA the pipe line, or use the Id property.
# This returns a new Gist object, mine, as 'Forking' creates a new instance of the Gist, under my users context.
PS C:\> Fork-GitHubGist -Id 23ad223dba5c36041e21


Comments    : 0
CommentsUrl : https://api.github.com/gists/f71b4ab6522af7cec700eb13248b82b2/comments
CommitsUrl  : https://api.github.com/gists/f71b4ab6522af7cec700eb13248b82b2/commits
CreatedAt   : 3/31/2016 11:41:57 PM
Description : This Gist provides a suggested pattern for serializing / deserializing PowerShell v5 class instances
Files       : {PowerShell v5 Class Serialization Pattern.ps1}
Forks       : 
ForksUrl    : https://api.github.com/gists/f71b4ab6522af7cec700eb13248b82b2/forks
History     : 
HtmlUrl     : https://gist.github.com/f71b4ab6522af7cec700eb13248b82b2
Id          : f71b4ab6522af7cec700eb13248b82b2
Owner       : GitHubUser
Public      : True
PullUrl     : https://gist.github.com/f71b4ab6522af7cec700eb13248b82b2.git
PushUrl     : https://gist.github.com/f71b4ab6522af7cec700eb13248b82b2.git
Truncated   : False
UpdatedAt   : 3/31/2016 11:41:57 PM
Url         : https://api.github.com/gists/f71b4ab6522af7cec700eb13248b82b2


# Now, you can see there is a [GitHubGistFork] object that now exists under Trevor's Gist.
PS C:\> (Get-GitHubGist -Id 23ad223dba5c36041e21).Forks

CreatedAt : 3/31/2016 11:41:57 PM
Id        : f71b4ab6522af7cec700eb13248b82b2
UpdatedAt : 3/31/2016 11:41:57 PM
User      : GitHubUser
Url       : https://api.github.com/gists/f71b4ab6522af7cec700eb13248b82b2


# And, if you dig into the User property of [GitHubGistFork] you can see it was me (dotps1) that forked it, and to when/where:
PS C:\> (Get-GitHubGist -Id 23ad223dba5c36041e21).Forks.User


AvatarUrl         : https://avatars.githubusercontent.com/u/1016996?v=3
EventsUrl         : https://api.github.com/users/dotps1/events{/privacy}
FollowersUrl      : https://api.github.com/users/dotps1/followers
FollowingUrl      : https://api.github.com/users/dotps1/following{/other_user}
GistsUrl          : https://api.github.com/users/dotps1/gists{/gist_id}
GravatarId        : 
HtmlUrl           : https://github.com/dotps1
Id                : 1016996
Login             : dotps1
OrganizationsUrl  : https://api.github.com/users/dotps1/orgs
ReceivedEventsUrl : https://api.github.com/users/dotps1/received_events
ReposUrl          : https://api.github.com/users/dotps1/repos
SiteAdmin         : False
StarredUrl        : https://api.github.com/users/dotps1/starred{/owner}{/repo}
SubscriptionsUrl  : https://api.github.com/users/dotps1/subscriptions
Type              : User
Url               : https://api.github.com/users/dotps1
```

Pretty cool right?

The Gist functionality for this module is coming along nicely, the [Object Model](https://github.com/dotps1/PSGitHub/blob/GitHubGist/Classes/PSGitHub.Classes.ps1) for Gists is complete.  
Hope to have the Gist functionality completed within the next week.