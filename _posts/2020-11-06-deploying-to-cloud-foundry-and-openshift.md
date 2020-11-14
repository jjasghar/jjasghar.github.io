---
layout: post
title: "Deploying to Cloud Foundry and OpenShift"
date: 2020-11-06 16:26:25
categories: openshift linux
---

## Scope

These are some notes/tutorial on what you'd do to deploy a simple
app first to Cloud Foundry on the IBM Cloud, then the equivalent to
OpenShift on the IBM Cloud. Hopefully this could be a "rosetta stone"
if you're coming from CF to OS.

> This will have a YouTube video [HERE][video] soon, but these are the notes.

## Cloud Foundry

We'll start with Cloud Foundry first, we'll clone down a simple python
app I've written then deploy it. Lets `git` clone this working repo
down:

```bash
git clone git@github.com:jjasghar/cloud-native-python-example-app.git
cd cloud-native-python-example-app
```

As you can see it's a very straight forward app, nothing too crazy `python`
flask app. Go ahead and open up the main `app.py`.

```bash
vim app.py
```

Next we should verify that it works, this way we know our code works as
we expect, and when we push it up to Cloud Foundry, everything should just
fall into place.

Go ahead and create a `virtualenv` so you don't "muddy up" your system
python, and also verify you are running `python3`.

```bash
virtualenv venv
source ./venv/bin/active
python -v
Python 3.6.8 (default, Apr  2 2020, 13:34:55)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-39)] on linux
```

Now install the dependancies, and verify it works as expected.

```bash
./venv/bin/pip install -r requirements.txt
./venv/bin/python app.py
# in another terminal
curl localhost:8080
curl localhost:8080/healthz
curl localhost:8080/healthzfoobar
```

Awesome, so we now see our output, lets take the next step to
deploy it to Cloud Foundry. Open up the `manifest.yaml` to see
the configuration.

```yaml
---
applications:
  - name: hellonerds
    memory: 256M
    instances: 1
```

As you can see, it's very straightforward, if you're actually going
to deploy this you'll need to change the name. ;).

Next, open up the `Procfile` which is what tells Cloud Foundry how to
run your application. Just like our local env, we just run the following:

```bash
web: python app.py
```

Next we need to log into the IBM Cloud via the CLI, and then push it
up, luckily it's pretty straight forward too.

```bash
ibmcloud login
ibmcloud cf install # if you haven't installed the cf plugin
ibmcloud target --cf
ibmcloud cf push # this will push the app into CF, and you should have your URL at the end
```

If you need to debug, use the following commands to check the logs, notice
the `--recent` if you want to take the _most_ recent logs.

```bash
ibmcloud cf logs hellonerds --recent
```

Awesome, and congrats, we have successfully ran our code locally and
deployed our code to CF. Now lets look at the same code on OpenShift.

## OpenShift

Now lets take the same code, and deploy it onto OpenShift. OpenShift
under the covers is Kubernetes, so we'll need to build a container
and deploy it. If you notice, I already have a `Dockerfile` in the
directory. If you want, you can build it locally, but as you can see
it's very sparse, and does what you'd expect such what a simple app
would require.

The first step is you'll need to login you OpenShift cluster, there's
a handful of ways to do this, I use the "Copy Login Command" from the
main console, and get something like:

```bash
oc login --token=LARGETOKENHERE --server=https://c107-e.us-south.containers.cloud.ibm.com:31937
```

The next step is just good hygiene and practice with OpenShift, you need
to create a new project.

```bash
oc new-project cfpython
```

You can name your project more or less anything, `cfpython` just seemed
right at the time I'm writing this.

Now that you've created this, lets push the app into OpenShift, we'll create
a new app,

```bash
cd cloud-native-python-example-app # if you aren't there
oc new-app . # you can also do the following if you want to point it to a remote --> oc new-app https://github.com/jjasghar/cloud-native-python-example-app.git
```

You'll notice that it'll find the `Dockerfile` almost instantly, and even
identify that it's an `python:3.7-alpine`. Pretty neat eh?

It'll kick off the build, and you can track it like it says:

```bash
oc logs -f bc/cloud-native-python-example-app
[-- snip --]
Successfully pushed image-registry.openshift-image-registry.svc:5000/cfpython/cloud-native-python-example-app@sha256:88209ad85f84a1094e1e2da1e1ed9fa05414022b64b7ee22000b2ac46f2bf544
Push successful
```

Next, we need to expose the app, thats with the following command:

```bash
oc expose svc/cloud-native-python-example-app
oc status # will give you the URL :)
```

Awesome! You've successfully deployed the same app into OpenShift now.
The next step is to use Web-hooks with OpenShift and GitHub, so you can
just commit to your repository, and kick off builds automatically. If you'd
like to see that demo, I'll have a YouTube link [here][video] walking through the beginning of this to the auto deploy.

[video]: https://youtu.be/vHVtKMhB9cU
