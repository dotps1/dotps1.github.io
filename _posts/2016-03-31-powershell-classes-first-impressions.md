---
layout: post
section-type: post
title: "My first impressions of Classes in PowerShell v5"
date: 2016-03-31 17:00:00 -0400
---

So, last night I began working on the Gist Object Model in the [PSGitHub](http://pcgeek86.github.io/PSGitHub) PowerShell Module, and this means, working with classes.  
I really don't have any experience with working with classes, in PowerShell or any other language.  
I did mess with them a bit when working through the Visual C# training from Channel 9 on the Microsoft Virtual Academy.  
But I have never actually used them in production.

So, after [Trevor](http://trevorsullivan.net) give me quick demo VIA Skype, I took off on my own and decided to give it a try.  
To put it simply, I loved working with the classes.  
Before I knew it I was creating classes that existed in other classes, and classes derived from other classes.  
Adding class methods, so on and so forth.

After reviewing some of the documentation of the [GitHub Gist](https://developer.github.com/v3/gists) objects returned from the API, when getting a single Gist, or single revision of a Gist you get the most in-depth response.  
Here is the response from getting one Gist:

```json
{
  "url": "https://api.github.com/gists/aa5a315d61ae9438b18d",
  "forks_url": "https://api.github.com/gists/aa5a315d61ae9438b18d/forks",
  "commits_url": "https://api.github.com/gists/aa5a315d61ae9438b18d/commits",
  "id": "aa5a315d61ae9438b18d",
  "description": "description of gist",
  "public": true,
  "owner": {
    "login": "octocat",
    "id": 1,
    "avatar_url": "https://github.com/images/error/octocat_happy.gif",
    "gravatar_id": "",
    "url": "https://api.github.com/users/octocat",
    "html_url": "https://github.com/octocat",
    "followers_url": "https://api.github.com/users/octocat/followers",
    "following_url": "https://api.github.com/users/octocat/following{/other_user}",
    "gists_url": "https://api.github.com/users/octocat/gists{/gist_id}",
    "starred_url": "https://api.github.com/users/octocat/starred{/owner}{/repo}",
    "subscriptions_url": "https://api.github.com/users/octocat/subscriptions",
    "organizations_url": "https://api.github.com/users/octocat/orgs",
    "repos_url": "https://api.github.com/users/octocat/repos",
    "events_url": "https://api.github.com/users/octocat/events{/privacy}",
    "received_events_url": "https://api.github.com/users/octocat/received_events",
    "type": "User",
    "site_admin": false
  },
  "user": null,
  "files": {
    "ring.erl": {
      "size": 932,
      "raw_url": "https://gist.githubusercontent.com/raw/365370/8c4d2d43d178df44f4c03a7f2ac0ff512853564e/ring.erl",
      "type": "text/plain",
      "language": "Erlang",
      "truncated": false,
      "content": "contents of gist"
    }
  },
  "truncated": false,
  "comments": 0,
  "comments_url": "https://api.github.com/gists/aa5a315d61ae9438b18d/comments/",
  "html_url": "https://gist.github.com/aa5a315d61ae9438b18d",
  "git_pull_url": "https://gist.github.com/aa5a315d61ae9438b18d.git",
  "git_push_url": "https://gist.github.com/aa5a315d61ae9438b18d.git",
  "created_at": "2010-04-14T02:15:15Z",
  "updated_at": "2011-06-20T11:34:15Z",
  "forks": [
    {
      "user": {
        "login": "octocat",
        "id": 1,
        "avatar_url": "https://github.com/images/error/octocat_happy.gif",
        "gravatar_id": "",
        "url": "https://api.github.com/users/octocat",
        "html_url": "https://github.com/octocat",
        "followers_url": "https://api.github.com/users/octocat/followers",
        "following_url": "https://api.github.com/users/octocat/following{/other_user}",
        "gists_url": "https://api.github.com/users/octocat/gists{/gist_id}",
        "starred_url": "https://api.github.com/users/octocat/starred{/owner}{/repo}",
        "subscriptions_url": "https://api.github.com/users/octocat/subscriptions",
        "organizations_url": "https://api.github.com/users/octocat/orgs",
        "repos_url": "https://api.github.com/users/octocat/repos",
        "events_url": "https://api.github.com/users/octocat/events{/privacy}",
        "received_events_url": "https://api.github.com/users/octocat/received_events",
        "type": "User",
        "site_admin": false
      },
      "url": "https://api.github.com/gists/dee9c42e4998ce2ea439",
      "id": "dee9c42e4998ce2ea439",
      "created_at": "2011-04-14T16:00:49Z",
      "updated_at": "2011-04-14T16:00:49Z"
    }
  ],
  "history": [
    {
      "url": "https://api.github.com/gists/aa5a315d61ae9438b18d/57a7f021a713b1c5a6a199b54cc514735d2d462f",
      "version": "57a7f021a713b1c5a6a199b54cc514735d2d462f",
      "user": {
        "login": "octocat",
        "id": 1,
        "avatar_url": "https://github.com/images/error/octocat_happy.gif",
        "gravatar_id": "",
        "url": "https://api.github.com/users/octocat",
        "html_url": "https://github.com/octocat",
        "followers_url": "https://api.github.com/users/octocat/followers",
        "following_url": "https://api.github.com/users/octocat/following{/other_user}",
        "gists_url": "https://api.github.com/users/octocat/gists{/gist_id}",
        "starred_url": "https://api.github.com/users/octocat/starred{/owner}{/repo}",
        "subscriptions_url": "https://api.github.com/users/octocat/subscriptions",
        "organizations_url": "https://api.github.com/users/octocat/orgs",
        "repos_url": "https://api.github.com/users/octocat/repos",
        "events_url": "https://api.github.com/users/octocat/events{/privacy}",
        "received_events_url": "https://api.github.com/users/octocat/received_events",
        "type": "User",
        "site_admin": false
      },
      "change_status": {
        "deletions": 0,
        "additions": 180,
        "total": 180
      },
      "committed_at": "2010-04-14T02:15:15Z"
    }
  ]
}

```

Based on this response, I ended up with these classes:

* GitHubUser
* GitHubGistHistory
* GitHubGistChangeStatus
* GitHubGistForks
* GitHubGistFile
* GitHubGist

Now, it would take a lot of space to show all the classes, but I am going to try to give a small example of how this works.  
Remember my response object follows the json from above.

```powershell
Class GitHubUser {
    # Define the class properties.
    [String]$Login
    [String]$Id
    [Uri]$AvatarUrl
    # And all the remaing properties from "owner: { }" here.... (or "user: { }" they are the same object).
    
    # Define a constructor for the object.
    # Using the object for and input object allows us to pass in <GistResponse>.owner and it will auto map the properties.
    # Else it would be GitHubUser([String]$Login, [String]$Id..........
    GitHubUser([Object]$object) {
        $this.Login = $object.login
        $this.Id = $object.id
        $this.AvatarUrl = $object.avatar_url
        # All the rest of the mappings here.....
    }
}
```

So now we have a `GitHubUser` class, now we can add that to the `GitHubGist` class:

```powershell
Class GitHubGist {
    # Define the class properties.
    [Uri]$Url
    [Bool]$Public
    [GitHubUser]$Owner
    # And all the rest of the root properties returned to us from the API.
    
    # Define a constructor for the object.
    # Using the object for and input object allows us to pass in <GistResponse> and it will auto map the properties.
    GitHubGist([Object]$object) {
        $this.Url = $object.url
        $this.Public = $object.public
        $this.Owner = $object.owner # This will pass the owner to our class above.
    }
}
```

Now we can put it all together:

```powershell
# Build our new GitHubGist object.
$gist = [GitHubGist]::new((<#Method for getting the response for the API.#>))

# We can access properties of the gist owner now.
$gist.Owner
$gist.Owner.AvatarUrl
```

This all had to be done with the files, history, forks and so forth.  
You may ask, why would we do this and not just use the json object?  
Well, using this we can now have cmdlet parameter sets based on input object type.  
And this will help with formatting in ps1xml, else it would be hard to differentiate multipule `System.Object`'s.

It was very fun, I still have more to do, but I wanted to share my inital usage of classes.

You can find all my progress on this project on [GitHub](https://github.com/dotps1/PSGitHub).