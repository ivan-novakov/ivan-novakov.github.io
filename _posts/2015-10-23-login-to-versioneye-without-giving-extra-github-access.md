---
layout: post
title: "Login to VersionEye without giving extra GitHub access"
description: ""
category: Devel
tags: [GitHub,OAuth]
---

I've been using [VersionEye](https://www.versioneye.com/) here and there to track the dependencies for some of my projects. I haven't logged in for some time. Recently, I wanted to check something and tried to log in via GitHub as usual. For my surprise, GitHub notified me, that VersionEye requests additional permissions to access my account. In fact, it wants to get read/write access to all my repositories in all organizations. And GitHub actually offered me just one option - *Authorize application*. Which I didn't want to...

I did a quick Google search and found out, that other users were complaining about that. The official statement by VersionEye was, that there is no other way, how you could log in with GitHub. For those, who didn't want to give those permissions it was suggested to use the username and password login option.

But... There is actually a way, how to log in with the old scope of permissions. And it's very easy. For a person with some knowledge of OAuth2 it's fairly easy to spot the solution.

On the VersionEye login page when you click on the *Login with GitHub* button, you are redirected to GitHub, where you need to log in and then the consent page with the permissions request shows up. Have a look at the URL:

```
https://github.com/login/oauth/authorize?client_id=50fb47103b8a3f03b2cd&scope=repo,user:email
```

The `scope` query parameter contains comma delimited values representing the permissions requested by VersionEye. In this case they are `repo` and `user:email`. Now, it's pretty straightforward how it works. Just delete the `repo` value and reload the page with this URL:

```
https://github.com/login/oauth/authorize?client_id=50fb47103b8a3f03b2cd&scope=user:email
```

You will be logged in without having to grant those additional permissions.
