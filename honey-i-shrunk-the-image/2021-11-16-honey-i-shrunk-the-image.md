---
title: Honey, I Shrunk the Docker Image!
published: true
categories: [docker]
tags: [docker]
date: 2021-11-16
---

# Hunting For The Smallest Possible Docker Image

I came across this challenge reading through an [engineering blog](https://idbs-engineering.com/) 
from a company I had originally applied to do do a placement at during my degree. 
(Unfortunately our good friend covid-19 cancelled that).

## [Container Competition](https://idbs-engineering.com/docker/containers/2021/01/28/container-competition.html)

An internal competition they had, where the challenge was:
> To build and publish a Docker image that prints the phrase “hello IDBS engineering” 
> (or similar) when run. At a minimum, you will need to build a Docker image, 
> run that image locally to test and publish to a Docker registry.  

> We'll keep it simple and the following dimensions will be taken into account:
> The image runs and prints to console. Image size - the smaller, the better.

I got about half way through this article before I decided that it sounded like
a fun challenge to spend an afternoon on, so cracked on with trying it myself
before I spoiled too much of their solution.

So here's some of my thought process and journey towards creating the smallest
possible "Hello World" docker image.

## docker run hello-world

I figured the best way to start would be to find a baseline image to try and
beat or improve from. Luckily Docker themselves have a 'hello-world' image.

```
❯ docker run hello-world
...
Hello from Docker!
...
```

```
❯ docker images
REPOSITORY     SIZE
hello-world    13.3kB
```

So our score to beat is 13.3kB, surprisingly small actually, so the alternative
baseline would be:

### ubuntu

Use the ubuntu base image, and simply echo "Hello world!":
```Docker
FROM ubuntu
CMD ["echo", "Hello World!"]
```
```shell
❯ docker build . -t ubuntu-hello

❯ docker run ubuntu-hello:latest
Hello There!

❯ docker images
REPOSITORY     SIZE
ubuntu-hello   72.8MB
hello-world    13.3kB
```

## Smaller base image options

The base ubuntu image is pretty gigantic, so what happens when you look for a
smaller base image to run?

### [alpine](https://hub.docker.com/_/alpine)

Alpine is a "great image base for utilities and even production applications"
and based around [musl-libc](https://musl.libc.org/) and BusyBox.

```Docker
FROM alpine

CMD ["echo","Hello There!"]
```
```
❯ docker build . -t alpine-hello

❯ docker run alpine-hello:latest
Hello There!

❯ docker images
REPOSITORY     SIZE
alpine-hello   5.61MB
ubuntu-hello   72.8MB
hello-world    13.3kB
```

Down to 6.51MB is definitely an improvement on the ubuntu base, but alpine is
based on an image of busybox, and contains some 'optional' extras we don't need.
So how small is a plain busybox version?

### [BusyBox](https://hub.docker.com/_/busybox)

BusyBox "provides a fairly complete environment for any small or embedded system". 
It only comes with the very bare necessities, this is fine however as in our case 
we only need one of them: "echo"

```Docker
FROM busybox

CMD ["echo","Hello There!"]
```

```shell
❯ docker build . -t busybox-hello

❯ docker run busybox-hello:latest
Hello There!

❯ docker images 
REPOSITORY      SIZE
busybox-hello   1.24MB
alpine-hello    5.61MB
ubuntu-hello    72.8MB
hello-world     13.3kB
```

An improvement! Just over 1MB is not bad, although the image still contains a
bunch of utilities that we don't need..

### scratch

> 0 BYTES
The 'SCRATCH' docker base image contains nothing. It contains 0 dependencies, 0
utilities, and is exactly what we're looking for. By removing all of the
overhead, all we need to do now is make an executable, copy it to the image, and
BAM we have a tiny hello world image.

## c

### gcc

### tcc

## asm

## cheating

# Conclusion


# Resources
https://idbs-engineering.com/docker/containers/2021/01/28/container-competition.html
inspo - held of reading the second half until i'd had aproper go

http://www.muppetlabs.com/~breadbox/software/tiny/teensy.html
smallest executable hello world

https://registry.hub.docker.com/_/busybox/
busybox embedded based image

https://devopsdirective.com/posts/2021/04/tiny-container-image/
6kb containerized http server,

https://codegolf.stackexchange.com/a/55479
codegolf hello world
