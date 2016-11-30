---
layout: post
title: "Use Travis CI for building docker images"
comments: true
tags: 
  - docker
  - travis
---

I couldn't find a good `.travis.yml` reference for building docker images without a lot of additional config for running tests.  Here's what I'm using to just simply building a docker image, running it, verifying it's running and pushing it to Docker Hub.
<!--more-->

This assumes you already have Travis linked to your GitHub account and that the repo has been "turned on" in Travis.

For the template below, you also need to add three environment variables to the repo properties in Travis: `DOCKER_EMAIL`, `DOCKER_USERNAME` and `DOCKER_PASSWORD`.  (You could also [encrypt](https://docs.travis-ci.com/user/encryption-keys/) the sensitive data.  If you have a lot of repos to set up, this might save some time in the long run.)

Here's the template:

~~~ yaml
# needed for docker service
sudo: required

# add the docker service
services:
  - docker

# some env variables that we'll use in various places
# there's a gotcha (https://github.com/yarpc/crossdock/issues/31) 
# with using $TRAVIS_BRANCH and docker build tags.  
# if this bites me, i'll look at the workaround
env:
  global:
    - COMMIT=${TRAVIS_COMMIT::8}
    - REPO=b225ccc/docker-sickrage

# if nothing needs to be installed, just set to true
# otherwise build will fail
install: true

# build the image
before_script:
  - export TAG=`if [ "$TRAVIS_BRANCH" == "master" ]; then echo "latest"; else echo $TRAVIS_BRANCH ; fi`
  - docker build -f Dockerfile -t $REPO:$COMMIT .

# docker run the image and verify it's in the process list
script:
  - docker run -d -p 127.0.0.1:8081:8081 --name sickrage $REPO:$COMMIT
  - docker ps | grep -q sickrage

# if the script was successful, add some tags to the image, login to docker and then push it
after_success:
  - docker tag $REPO:$COMMIT $REPO:$TAG
  - docker tag $REPO:$COMMIT $REPO:travis-$TRAVIS_BUILD_NUMBER
  - docker login -e "$DOCKER_EMAIL" -u "$DOCKER_USERNAME" -p "$DOCKER_PASSWORD"
  - docker push $REPO
~~~

Each successful build, will result in three new tags in Docker hub:

1. "travis-$BUILD_NUMBER", `travis-8`
2. "latest", `latest`
3. "$GIT\_COMMIT_TAG", `f6debe03`

Example:

  ![docker hub screenshot](/_posts/_img/2016-06-14-2.png)

---

Reference(s):

1. [https://blog.travis-ci.com/2015-08-19-using-docker-on-travis-ci/](https://blog.travis-ci.com/2015-08-19-using-docker-on-travis-ci/)
2. [https://sebest.github.io/post/using-travis-ci-to-build-docker-images/](https://sebest.github.io/post/using-travis-ci-to-build-docker-images/)

