---
title: Honey, I Shrunk the Image! (Hunting a Tiny Docker Image)
published: true
categories: [docker]
tags: [docker]
date: 2021-11-16
---

# Hunting For The Smallest Possible Docker Image

I came across this challenge reading through an [engineering blog](https://idbs-engineering.com/) 
from a company I had originally applied to do a placement at during my degree. 

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

So our score to beat is 13.3kB, smaller than I was expecting (or hoping), this
may take some work to beat. The alternative baseline is an ubuntu base image: 

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

The base ubuntu image is pretty gigantic, the actual Hello World is negligible
in size compared to the overhead of the ubuntu base. So what other options are
there for base images to use?

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

An improvement! But still an order of magnitude larger than the hello-world example
image. Just over 1MB is not bad, although the image still contains a
bunch of utilities that we don't need..

### scratch

> 0 BYTES


The 'SCRATCH' docker base image contains nothing. It contains 0 dependencies, 0
utilities, and is exactly what we're looking for. By removing all of the
overhead, the minimum size is now just the smallest self contained executable we
can build. 

## c
My first thought here was to start with a simple c program. 

### gcc

As a baseline, I started with the classic printf hello world.

```c
// hellogcc.c
#include <stdio.h>
int main()
{
    printf("Hello World\n");
    return 0;
}
```

And then compile with gcc:

```shell
gcc hellogcc.c -static -o hello1
```

Compiling with gcc, (and the -static flag to include dependencies), results in a
massive 892KB executable. Luckily we can quickly shrink this with some compiler
flags.

```shell
gcc hellogcc.c -Os -s -static -o hello2
```

Telling the compiler to optimise for size using "-Os", and "-s" to remove linker
table info from the executable. This results in a 793KB executable, better - but
still not good enough.


A bit more searching around brought me to [this stackoverflow answer](https://stackoverflow.com/a/33630488)
Which outlines how even though nothing from the c libraries is used in the
actual program, the C runtime (CRT) startfiles that call the main function do
call on some libraries.

The solution to this is to specify startfiles, then choose to not include them
in compilation.

The new code:
```c
#include <unistd.h>
void _start(void) {
  write(1, "Hello world!", 12);
  _exit(0);
}
```

```shell
gcc hello_write.c -s -static --nostartfiles -o hello3
```

File sizes so far:
```shell
❯ ls -lh | awk '{print $9 " : " $5}'
 : 
hello1 : 869K
hello2 : 793K
hello3 : 8.9K
```


This reduces the executable size down to just 8.9KB, which is finally smaller
than the example hello-world docker image. But I think we can still go smaller..

### tcc

[Tiny C Compiler](https://bellard.org/tcc/) (TCC) is a tiny but hyper fast C
compiler. Luckily for us, it also generally creates much smaller executables.

```c
int main() {
  write(1, "Hello World\n", 12);
  return 0;
}
```
Compile with tcc:
```shell
tcc hello.c -o hellotcc
```
```
ls -lh | awk '{print $9 " : " $5}'
hello.c : 136
hellotcc : 3.0K
```

Down to just 3KB just by swapping compiler, not bad!


*Results so far:*

```shell
❯ docker images --format "table {{.Repository}}\t{{.Size}}"
REPOSITORY      SIZE
tcc-hello       2.99kB
gcc-hello       9.06kB
busybox-hello   1.24MB
alpine-hello    5.61MB
ubuntu-hello    72.8MB

hello-world     13.3kB

```


## asm

The next logical step to try and go smaller than compiled c is to just write
straight assembly. Now, whilst I have some experience with ASM - the majority
of that is for GameBoy or N64. Whilst I could sit down and learn my way around,
this challenge was meant to be about Docker - so why reinvent the wheel when I
can just Google it..

My first thought was to check the [code golf](codegolf.stackexchange.com/)
stackexchange for a solution. I found one [here](https://codegolf.stackexchange.com/a/55479),
however right after that I found what is arguably the holy grail:

### [ELF Executable Magic](http://www.muppetlabs.com/~breadbox/software/tiny)

In an example of pure wizardry by Brian Raiter, an ELF executable "Hello, World"
program in just 62 Bytes. 

Every ELF executable has a 52 Byte header, containing data on: if it's 32/64
bit, little or big-endian, the target architecture etc. By placing the
instructions to write "hello, world" in the header, (and some other tricks
absolutely violating every inch of the ELF file standard), he produces a 62 Byte
executable.

So a simple Dockerfile that just copies this executable to the image:

```Docker
FROM scratch
COPY hello hello
CMD ["/hello"]
````
```shell
REPOSITORY      SIZE
asm-hello       62B
tcc-hello       2.99kB
gcc-hello       9.06kB
busybox-hello   1.24MB
alpine-hello    5.61MB
ubuntu-hello    72.8MB
hello-world     13.3kB
```

## Cheating

Unless I can break some laws of physics, it seems like 62 Bytes is the smallest
possible size for a Docker image with an executable in it.

But can we cheat by not having to include an executable at all?

Yes.. _kind of_


```Docker
FROM scratch
CMD ["dir/hello"]
```
```shell
❯ docker build . -t cheat-hello
```

This docker image simply runs an executable called 'hello' in the folder 'dir'.
This folder and executable don't exist when the image is created, so it gives an
error if ran normally, but this does mean that the actual Docker image is
exactly 0 Bytes.

```shell
REPOSITORY         SIZE
cheat-hello        0B
asm-hello          62B
tcc-hello          2.99kB
gcc-hello          9.06kB
busybox-hello      1.24MB
alpine-hello       5.61MB
ubuntu-hello       72.8MB
hello-world        13.3kB

```

The image is ran using a directory on the host machine linked as a volume in the
container. If the directory linked contains an executable called 'hello', then
the container will run the executable and output "hello world".


```shell
❯ docker run -v ~/path/to/folder/containg/executable/:/dir cheat-hello
hello, world
```


Is this cheating? Yes. The original competition did specify that it should be
able to be published to a Docker registry. I did try for a while to link to the
host machine's `/bin/` to grab `echo` from there, however grabbing dependencies
from the host machine is quite literally the antithesis of the role of Docker..

# Conclusion

Whilst the challenge quite quickly strayed away from Docker, learning to
dismantle and violate it's principles has been a great learning experience. As
has diving into the ELF file format and some c compiler options.


Having now read the rest of the original challenge post - we both ended up at
the same final answer. (Theirs is slightly longer only due to a longer message).
Once you get to zero Docker overhead, there's only so far you can go. 


It was an absolute blast for an afternoon - and I'd love to come back and try a
slightly modified challenge: perhaps a Docker quine (image that recreates
itself) or the smallest possible Docker web server etc.


# Resources
https://idbs-engineering.com/docker/containers/2021/01/28/container-competition.html

http://www.muppetlabs.com/~breadbox/software/tiny/teensy.html

http://timelessname.com/elfbin/

https://registry.hub.docker.com/_/busybox/

https://devopsdirective.com/posts/2021/04/tiny-container-image/

https://codegolf.stackexchange.com/a/55479

https://docs.docker.com/engine/reference/commandline/run/

