---
layout: post
title: "How to share your cookbook generator"
date: 2017-08-14 10:59:11
categories: chef sysadmin
---

Great news! You've gotten your team to start leveraging cookbook generator(s) to start
standardizing some basic cookbook development. (Maybe even inspired from my [blog post][myblogpost]
if I'm lucky.) The challenge now is sharing your generator. Here's one example workflow that
might inspire you for your company.

I'll assume you've already got a first iteration of your cookbook generator. I'll tell this story
by introducing 2 people, first, Betty, who created the generator, and wants to share it with her
team and maybe one day her whole company. The second person is Billy, he has been tasked with
creating a pilot cookbook ensure that [vim][vim] is installed on every machine.

So sitting at her desk, Betty has ready to ship her standardized cookbook, she created the
Github repository with the fun green button.

![](../../../../../pics/github_new_repo.png)

She sat back, and ran the commands to upload her cookbook to her internal Github.

```bash
cd ~/chef/acme-cookbook-generator
~/chef/acme-cookbook-generator $ git add .
~/chef/acme-cookbook-generator $ git commit -m "first commit"
~/chef/acme-cookbook-generator $ git remote add origin https://github.acme.com/chef/acme-cookbook-generator
~/chef/acme-cookbook-generator $ git commit -m "first commit" git push -u origin master
```

She quickly shot off an email to her team, saying that the first release was out.

Billy saw the note come in and realized he could start working on his cookbook. He opened up a terminal
and typed in the following commands:

```bash
cd ~/chef/
~/chef $ git clone https://github.acme.com/chef/acme-cookbook-generator
~/chef $ chef generate cookbook -g ~/chef/acme-cookbook-generator/cookbooks acme-vim
~/chef $ cd acme-vim
~/chef/acme-vim $ vi README.md
```

As a good chef practitioner he started with updating the `README.md`, filling out the TODOs he saw,
and started on his main basic recipe. All in all pretty straight forward, and got he job done.

As he was developing it he noticed that he could extend the generator a bit, so he wanted to help
out. He ran the following commands:

```bash
cd ~/chef/acme-cookbook-generator
~/chef/acme-cookbook-generator $ cd cookbooks/code_generator
~/chef/acme-cookbook-generator $ git checkout -b remove_license
~/chef/acme-cookbook-generator/cookbooks/code_generator $ rm templates/default/LICENSE.erb
~/chef/acme-cookbook-generator/cookbooks/code_generator $ vi recipes/cookbook.rb # and removed the reference to the LICENSE.erb
~/chef/acme-cookbook-generator/cookbooks/code_generator $ cd ../..
~/chef/acme-cookbook-generator $ git add .
~/chef/acme-cookbook-generator $ git commit -m "Removed the Apache2 license reference because that's not what we use here"
~/chef/acme-cookbook-generator $ git push origin remove_license
```

Billy opened up his browser and got to push a green button too! He created a pull request against
the repo with his `remove_license` branch.

![](../../../../../pics/github_new_pull_request.png)

Betty got the announcement via Github, commented and merged!

Billy got the notice that his PR has been merged, and needed to update his generator. He quickly realized
that every time the generator changed he'd need to pull down the newest changes, but there was no way
to update pre-created cookbooks. This wasn't ideal, but at least every new cookbook would have a running
"best practices." He opened up his terminal and did the following:

```bash
cd ~/chef/acme-cookbook-generator
~/chef/acme-cookbook-generator $ git pull
Updating f8a8b39..444a6f1
Fast-forward
 recipes/cookbook.rb                                                     |   8 +--
 template/default/LICENSE.erb                                            | 202 -----------------------
~/chef/acme-cookbook-generator $
```

This is a pretty typical Github PR based workflow, and there is no real reason you can't do something
like this. I hope this helps and gets people who normally wouldn't know how to leverage the generator
leverage it and even commit back changes.

[myblogpost]: http://jjasghar.github.io/blog/2017/08/08/using-the-cookbook-generator-as-soon-as-possible/
[vim]: http://www.vim.org/
