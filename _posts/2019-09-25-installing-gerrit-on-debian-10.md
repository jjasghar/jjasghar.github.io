---
layout: post
title: "Installing gerrit on Debian 10"
date: 2019-09-25 12:32:20
categories: sysadmin debian gerrit
---


Believe it or not, but ever since my days in the OpenStack community I've always
been a fan of [gerrit](https://www.gerritcodereview.com). Most hate it, but for
whatever reason as soon as I got my head wrapped around it I fell in love.

Here are my notes on building a new gerrit instance on Debian 10.

First thing first, make sure your machine is up-to-date.

```bash
$ sudo apt-get update && sudo apt-get upgrade -y
```

Next, we need to install Java 1.8 and make sure git is there. You need to go [here](http://www.oracle.com/technetwork/java/javase/downloads/index.html) to pull down the correct version of Java. Extract it
and make sure that the `$JAVA_PATH` is correct.

```bash
$ sudo apt-get install git -y
```

Go ahead and `wget` down the `.war` file, from the releases. It should look something like:

```bash
$ wget https://gerrit-releases.storage.googleapis.com/gerrit-3.0.2.war
```

Now create a directory where you want to store your configuration, server's SSH Keys, and
the managed Git repositories.

```bash
$ mkdir $HOME/gerrit_directory
```

Create a `gerrit` user, and make sure they can read/write the application directory you
made above.

```bash
$ sudo adduser gerrit
Adding user `gerrit' ...
Adding new group `gerrit' (1002) ...
Adding new user `gerrit' (1002) with group `gerrit' ...
Creating home directory `/home/gerrit' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for gerrit
Enter the new value, or press ENTER for the default
	Full Name []:
	Room Number []:
	Work Phone []:
	Home Phone []:
	Other []:
Is the information correct? [Y/n] Y
$ chmod 775 gerrit_directory/
$ sudo su gerrit
$ cd gerrit_directory && touch blah # sanity check to make sure permissions works
$ cd ..
```

Now we need to initialize the directory and gerrit, use the `gerrit` user like above.

```bash
$ java -jar gerrit-3.0.2.war init -d gerrit_directory
```

There will be a bunch of questions, for now, go with all the defaults.
After they are completed, you can run the following to start your gerrit instance!

```bash
$ ./gerrit_directory/bin/gerrit.sh start
Starting Gerrit Code Review: WARNING: Could not adjust Gerrit's process for the kernel's out-of-memory killer.
         This may be caused by ./gerrit_directory/bin/gerrit.sh not being run as root.
         Consider changing the OOM score adjustment manually for Gerrit's PID=1946 with e.g.:
         echo '-1000' | sudo tee /proc/1946/oom_score_adj
OK
```

I should mention also, if you are planning on putting this behind a proxy, you'll need
to make some changes to it for forward the requests along. If you are using nginx,
something like this is good:

```nginx
server {
  server_name gerrit.YOURDOMAIN;
  location / {
    proxy_pass http://YOURIP:8080;
    proxy_set_header Host $host;
  }
}


server {
  listen   29418;
  server_name gerrit.YOURDOMAIN;
  location / {
    proxy_pass http://YOURIP:29418;
  }
}
```

Next, you need to create your repository when it's up. If you are the first person to login,
you are automatically given Administrator rights. Keep this in mind for later on. Go ahead
and `Browse -> Repositories` then look under your name on the right and click `CREATE NEW`.

You can also create a repository via ssh:

```bash
$ ssh -p 29418 ADMIN_USERNAME@gerrit.DOMAINNAME gerrit create-project demo/gerrittest
```

The above will create the repository `demo/gerrittest` in the browse GUI.


I went ahead and use the name `testing` for the following examples. After that, click `create`
and congratulations you have a repo created on your gerrit instance.

Next, go to your terminal, and we need to clone it down. You should have your public ssh-key and
a username configured under your account before cloning anything down with `ssh://`.

```bash
$ git clone ssh://gerrit.DOMAINNAME:29418/testing.git
Cloning into 'testing'...
remote: Counting objects: 2, done
remote: Finding sources: 100% (2/2)
remote: Total 2 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (2/2), done.
```

Congratulations, you now have the code on your machine, lets create a review.

Do something like the following to create a `README.md`.

```bash
$ cd testing/
~/testing(master) $ git checkout -b real_testing
Switched to a new branch 'real_testing'
~/repo/testing(real_testing) $ vim README.md
```

After that, save and commit:

```bash
~/testing(real_testing) $ git add .
~/testing(real_testing) $ git commit -m "inital readme"
[real_testing b889213] inital readme
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
~/testing(real_testing) $ git review -s
```

If you see this following error, you need to create the `.gitreview`.

```
No '.gitreview' file found in this repository. We don't know where
your gerrit is.
Please manually create a remote named "gerrit" and try again.
```
To create a `.gitreview` file, here's an example for you:

```ini
[gerrit]
host=gerrit.DOMAINNAME
port=29418
project=testing
defaultbranch=master
```

If everything is setup correctly now, you should be able to do:

```bash
~/testing(real_testing) $ git review
```

If this doesn't work, you can try the following.

```bash
$ git config remote.origin.push refs/heads/*:refs/for/*
~/testing(4th_test)$ git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 396 bytes | 396.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
remote: Processing changes: refs: 2, done
To ssh://gerrit.asgharlabs.io:29418/testing.git
 ! [remote rejected] 4th_test -> refs/for/4th_test (branch 4th_test not found)
 ! [remote rejected] master -> refs/for/master (duplicate request)
error: failed to push some refs to 'ssh://gerrit.DOMAINNAME:29418/testing.git'
```

I haven't been able to figure out how to get `branches` made via the push, but if
you log into the web GUI and then go to the repository, you can create the `branch`
via the right hand `CREATE NEW` button. In the above case I would do `4th_test`.

```bash
~/repo/testing(4th_test) $ git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 396 bytes | 396.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
remote: Processing changes: refs: 2, new: 1, done
remote:
remote: SUCCESS
remote:
remote:   http://gerrit.DOMAINNAME/c/testing/+/27 This is a better commit message. [WIP] [NEW]
remote:
To ssh://gerrit.DOMAINNAME:29418/testing.git
 * [new branch]      4th_test -> refs/for/4th_test
 ! [remote rejected] master -> refs/for/master (duplicate request)
error: failed to push some refs to 'ssh://gerrit.DOMAINNAME:29418/testing.git'
~/testing(4th_test) $
```

You can now go through your typical gerrit workflow, and when you merge your patch-set, you're almost done.
You'll need to delete the branch like you created it, right now I only know how to do this through the GUI.

```bash
~/testing(4th_test) $ git pull
From ssh://gerrit.DOMAINNAME:29418/testing
 - [deleted]         (none)     -> origin/4th_test
Already up to date.
~/repo/testing(master) >
```
