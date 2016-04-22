---
layout: post
title: "Getting Jenkins and Jenkins Job Builder running"
date: 2016-01-03 15:51:33 -0600
comments: true
categories: linux sysadmin chef jenkins
---

After [Nick Silkey][nick] mentioned about the [Jenkins Job builder][jjb] on the
[Chef-Openstack Foodfight show][foodfight], or the `jjb` for short, I started to
look around for a how-to on starting from scratch. The [typical documentation][google]
is out there, but nothing to say, "Hey I want to use the `jjb` and
[Jenkins][jenkins] and I have neither."

This how-to is my steps on getting this build to work. (Granted, I should probably
write a wrapper cookbook for this, but that's a long term project and it
shouldn't be too hard.)

OK, so you have a machine you want to get Jenkins up and running on. First thing
first, get Jenkins. I work for [Chef][chef], so naturally I gravitate towards using the
[jenkins-cookbook][jenkinscookbook]. There is a lot you can do with it, but the
simplest installation is the following.

```bash
~$ git clone https://github.com/chef-cookbooks/jenkins
~$ chef exec knife cookbook upload jenkins
~$ chef exec knife bootstrap <machine> -r recipe[jenkins::master]
```

I should say that the `<machine>` is a node that you'd like to check in to your
chef server and adds the `jenkins::master` recipe to the `run_list` and converges.

You after this completes you should be able to go to: `http://<machine>:8080` and see
a new Jenkins build.

Ok, 1/2 way there.

Now, you have a couple ways to do this, I suggest starting on the box that has Jenkins
built, so you take out a possible variable. *NOTE:* There is no reason though you can't run
the following commands on your local machine and do it all remotely. I should also
mention that I built the following on Ubuntu 14.04.

First you need to checkout the `jjb`.

```bash
~$ git clone https://github.com/openstack-infra/jenkins-job-builder.git
~$ cd jenkins-job-builder
```

The base install of Ubuntu 14.04 is pretty sparse, so you need add some pip's for
python so `jjb` can compile correctly.

```bash
~/jenkins-job-builder$ sudo apt-get install python-pip
~/jenkins-job-builder$ sudo pip install pyyaml
~/jenkins-job-builder$ sudo pip install pbr
~/jenkins-job-builder$ sudo pip install python-jenkins
~/jenkins-job-builder$ sudo pip install setuptools
~/jenkins-job-builder$ sudo pip install ordereddict
```

After you have these things installed, you can run the following to build the `jjb`.

```bash
~/jenkins-job-builder$ sudo python setup.py install
```

Now you should be able to run the following to test out your build of the `jjb`.

```bash
~/jenkins-job-builder$ jenkins-jobs test tests/yamlparser/fixtures/templates002.yaml
INFO:root:Will use anonymous access to Jenkins if needed.
INFO:jenkins_jobs.builder:Number of jobs generated:  2
INFO:jenkins_jobs.builder:Job name:  project-name-26
<?xml version="1.0" encoding="utf-8"?>
<project>
  <actions/>
  <description>&lt;!-- Managed by Jenkins Job Builder --&gt;</description>
  <keepDependencies>false</keepDependencies>
  <blockBuildWhenDownstreamBuilding>false</blockBuildWhenDownstreamBuilding>
  <blockBuildWhenUpstreamBuilding>false</blockBuildWhenUpstreamBuilding>
  <concurrentBuild>false</concurrentBuild>
  <canRoam>true</canRoam>
  <properties/>
  <scm class="hudson.scm.NullSCM"/>
  <builders>
    <hudson.tasks.Shell>
      <command>git co old_branch</command>
    </hudson.tasks.Shell>
  </builders>
  <publishers/>
  <buildWrappers/>
</project>
INFO:jenkins_jobs.builder:Job name:  project-name-27
<?xml version="1.0" encoding="utf-8"?>
<project>
  <actions/>
  <description>&lt;!-- Managed by Jenkins Job Builder --&gt;</description>
  <keepDependencies>false</keepDependencies>
  <blockBuildWhenDownstreamBuilding>false</blockBuildWhenDownstreamBuilding>
  <blockBuildWhenUpstreamBuilding>false</blockBuildWhenUpstreamBuilding>
  <concurrentBuild>false</concurrentBuild>
  <canRoam>true</canRoam>
  <properties/>
  <scm class="hudson.scm.NullSCM"/>
  <builders>
    <hudson.tasks.Shell>
      <command>git co new_branch</command>
    </hudson.tasks.Shell>
  </builders>
  <publishers/>
  <buildWrappers/>
</project>
INFO:jenkins_jobs.builder:Cache saved
```

I ran into this problem, so here's a heads up. If you do `sudo pip install jenkins`
which seems logical, you'll see this when attempting to run `jjb`.

```bash
~/jenkins-job-builder$ jenkins-jobs test tests/yamlparser/fixtures/templates002.yaml
Traceback (most recent call last):
  File "/usr/local/bin/jenkins-jobs", line 6, in <module>
    from jenkins_jobs.cmd import main
  File "/usr/local/lib/python2.7/dist-packages/jenkins_jobs/cmd.py", line 27, in <module>
    from jenkins_jobs.builder import Builder
  File "/usr/local/lib/python2.7/dist-packages/jenkins_jobs/builder.py", line 25, in <module>
    import jenkins
  File "/usr/local/lib/python2.7/dist-packages/jenkins.py", line 9, in <module>
    lookup3 = cdll.LoadLibrary(os.path.join(get_python_lib(), "lookup3.so"))
  File "/usr/lib/python2.7/ctypes/__init__.py", line 443, in LoadLibrary
    return self._dlltype(name)
  File "/usr/lib/python2.7/ctypes/__init__.py", line 365, in __init__
    self._handle = _dlopen(self._name, mode)
OSError: /usr/lib/python2.7/dist-packages/lookup3.so: cannot open shared object file: No such file or directory
```

Now you need to add a user, I suggest the `jenkins` with the password of `jenkins`.
You can do this either by the `data_bag` if you want, or if you're lazy like me
you can use the GUI. After you create the user, take note of the API key, you'll
need to know it for the next step.

After this, you need to create a `etc/jenkins_jobs.ini` file, here's a very basic one:

```
[job_builder]
ignore_cache=True
keep_descriptions=False
include_path=.:scripts:~/git/
recursive=False
exclude=.*:manual:./development
allow_duplicates=False

[jenkins]
user=jenkins
password=0b7326314fca6256a25513c3d0f0e13f # this is the API key of the user above
url=http://127.0.0.1:8080
query_plugins_info=False
##### This is deprecated, use job_builder section instead
#ignore_cache=True
```

Now you should be able to:

```bash
~/jenkins-jobs-builder$ jenkins-jobs --conf etc/jenkins_jobs.ini update tests/yamlparser/fixtures/templates002.yaml
```

And you should have two jobs on your Jenkins server.

Congrats, now you have a jenkins+`jjb` build. I suggest you should create another
directory, call it `jobs/` or something like it, do a `git init` in it. You may
want to move the `jenkins_jobs.ini` to that directory, and create a simple `yaml`
and commit them. The following is an example of a simple `yaml`.

```yml
- job:
    name: Basic example
    project-type: freestyle
    description: 'A basic example of a `jjb` job that echos "Hello world and cats out /etc/password"'
    builders:
        - shell: |
            echo 'Hello world!'
            cat /etc/passwd
```

Now you can either, create a file per job, or have everything under one file. It
depends on how you think you'll grow your jobs.

Then now you can run the following whenever you need to update your jobs:

```bash
~/jobs$ jenkins-jobs --conf jenkins_jobs.ini update ./
```

Oh! Those demo jobs don't do anything and honestly, are ugly. If you want to delete
them, you can run the following:

```bash
~$ jenkins-jobs --conf etc/jenkins_jobs.ini delete project-name-27
~$ jenkins-jobs --conf etc/jenkins_jobs.ini delete project-name-26
```

[nick]: https://twitter.com/filler
[jjb]: http://docs.openstack.org/infra/jenkins-job-builder/
[chef]: http://chef.io
[jenkinscookbook]: https://github.com/chef-cookbooks/jenkins
[foodfight]: http://foodfightshow.org/2015/12/chef-and-openstack.html
[jenkins]: http://jenkins-ci.org/
[google]: https://www.google.com/webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8#q=jenkins%20job%20builder%20tutorial
