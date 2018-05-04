---
layout: post
title: Multiple Github accounts & SSH keys on the same machine
date: 2016-10-06 19:31:23.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Environments &amp; Tools
- Random
tags:
- git
- github
- key
- keygen
- ssh
meta:
  _edit_last: '1'
  _jetpack_related_posts_cache: a:1:{s:32:"8f6677c9d6b0f903e98ad32ec61f8deb";a:2:{s:7:"expires";i:1525303672;s:7:"payload";a:0:{}}}
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p>If like me you have 2 or more, different Github accounts on the go, then accessing and committing as both on the same machine can be a challenge.<br />
In my case, I have 2 accounts, <a href="https://github.com/eoincgreenfinch">one for work</a> associated with my company email and a <a href="https://github.com/eoincampbell">second for my own personal code</a>.</p>
<p>If you'd like to be able to checkout, code and commit against different repo's across different github accounts on the same machine, then you can do so by setting up multiple ssh keys, and having hostname aliases configured in your .ssh config file.</p>
<p>First off all, you'll need to generate your SSH Keys. If you haven't done this already, you can use the following commands to generate your keys.</p>

```
$ ssh-keygen -t rsa -C "eoin@work.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/eoin/.ssh/id_rsa): id_rsa_eoin_at_work

$ ssh-keygen -t rsa -C "eoin@home.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/eoin/.ssh/id_rsa): id_rsa_eoin_at_home

```

<p>Once you've created your 2 files, you'll see 2 key pair files (the file you specified and a .pub) in your ~/.ssh directory. You can go ahead and add the respective key files each of your Github accounts. It's in the <a href="https://github.com/settings/keys">Github &gt; Settings &lt; SSH and GPG Keys</a> section of your settings. You'll also need to add these files to ssh.</p>
<p>Next you'll want to create an ssh config file in your ~/.ssh directory. You can see mine below.</p>

```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_eoin_at_work

Host personal.github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_eoin_at_home
```

<p>Here's the trick, when you execute a <code>git clone</code> command to clone a repo, the host in that command is not a real DNS hostname. It is the host entry specified on the first line of each section in the above files. So you can very easily change that. Now, if I want to check out work related projects from my work account, I can use.</p>

```
git clone git@github.com:eoincgreenfinch/heartbeat.git

# don't forget to set your git config to use your work meta data.
git config user.name "eoincgreenfinch"
git config user.email "eoin@work.com" 

```

<p>But if I want to check out code from my personal account, I can easily modify the clone URI with the following.</p>

```
git clone git@personal.github.com:eoincampbell/combinatorics.git

# don't forget to set your git config to use your work meta data.
git config user.name "eoincampbell"
git config user.email "eoin@home.com" 
```

<p><i>~Eoin Campbell</i></p>
