---
layout: post
title: "Using Travis CI to release a docker container to the public Docker registry"
date: 2018-11-14 12:31:21
categories: linux sysadmin docker
---


I have a few [docker containers][ibmdocker] that I push to the public Docker
hub. I was going to set up a Jenkins job to do it for me, but thought instead
I could use Travis CI. These are the steps I've taken from two posts, [ops.tips][tips]
and [mobileforgood][good] to make work in my IBM container Repositories.

*Note*: The following examples are taken from [this][github] repository.

Thanks to these two for the direction, and hopefully this will help someone in the future.

First thing first, I needed to install the `travis` gem.

```shell
gem install travis
```

After that, go ahead and do a login using the following command in the repository you want to
publish:

```shell
~/repo/ibm-cloud-cli on master± travis login --auto
Successfully logged in as jjasghar!
```

Then if you don't already have a `.travis.yml` do a `travis init`:

```shell
~/repo/ibm-cloud-cli on master± travis init
Detected repository as jjasghar/ibm-cloud-cli, is this correct? |yes|
Main programming language used: |Ruby|
.travis.yml file created!
jjasghar/ibm-cloud-cli: enabled :)
```

The most important line is the `enabled :)` this is one step you don't have to click inside Github,
and it just does it for you. If you have a `.travis.yml` you can skip this. It seems it defaults to `Ruby`
here I opened it up immediately and removed everything in the file, and pasted in the following:

*Note*: it is inspired from the <https://ops.tips>:

```yaml
sudo: 'required'

services:
  - 'docker'

before_install:
  - './.travis/main.sh'

script:
  - 'make test'
  - 'make image'

# To have `DOCKER_USERNAME` and `DOCKER_PASSWORD`
# use `travis env set DOCKER_USERNAME ...`
# use `travis env set DOCKER_PASSWORD ...`
deploy:
  provider: script
  script: docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD && make push-image
  on:
    branch: master
```

Then created a directory called `.travis` and made a file called `main.sh` and `chmod +x` the
file with the following script:

```bash
#!/bin/bash

set -o errexit

main() {
  setup_dependencies
  update_docker_configuration

  echo "SUCCESS:
  Done! Finished setting up Travis machine.
  "
}

setup_dependencies() {
  echo "INFO:
  Setting up dependencies.
  "

  sudo apt update -y
  sudo apt install --only-upgrade docker-ce -y
  docker info
}

update_docker_configuration() {
  echo "INFO:
  Updating docker configuration
  "

  echo '{
  "experimental": true,
  "storage-driver": "overlay2",
  "max-concurrent-downloads": 50,
  "max-concurrent-uploads": 50
}' | sudo tee /etc/docker/daemon.json
  sudo service docker restart
}

main
```

And finally created a `Makefile` with the following:

```bash
IMAGE := jjasghar/ibm-cloud-cli
VERSION:= $(shell grep IBM_CLOUD_CLI Dockerfile | awk '{print $2}' | cut -d '=' -f 2)

test:
	true

image:
	docker build -t ${IMAGE}:${VERSION} .
	docker tag ${IMAGE}:${VERSION} ${IMAGE}:latest

push-image:
	docker push ${IMAGE}:${VERSION}
	docker push ${IMAGE}:latest


.PHONY: image push-image test
```

As you can see it's pretty straight forward. I pull the version from the `Dockerfile`
and create two tags and push them to the hub if needed.

From now on I'll have to update the `VERSION` in the `Dockerfile` but that's ok, it's a
good practice to know what your versions are. It will only push when you merge the PR
due to the `deploy` line, which is the power of this whole setup.


[ibmdocker]: http://jjasghar.github.io/ibm-docker/
[tips]: https://ops.tips/blog/travis-ci-push-docker-image/
[good]: https://medium.com/mobileforgood/patterns-for-continuous-integration-with-docker-on-travis-ci-71857fff14c5
[github]: https://github.com/jjasghar/ibm-cloud-cli/
