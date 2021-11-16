---
title: World Wide Web Winners
published: false
categories: [resources]
tags: [resources]
date: 2021-11-12
---


## Fast Inverse Square Root

From the source of 'Quake 3', a black magic approximation of an inverse square
root function. 

```c
float Q_rsqrt( float number )
{
    long i;
    float x2, y;
    const float threehalfs = 1.5F;

    x2 = number * 0.5F;
    y  = number;
    i  = * ( long * ) &y;    // evil floating point bit level hacking
    i  = 0x5f3759df - ( i >> 1 );               // what the fuck? 
    y  = * ( float * ) &i;
    y  = y * ( threehalfs - ( x2 * y * y ) );   // 1st iteration
//  y  = y * ( threehalfs - ( x2 * y * y ) );   // 2nd iteration,
                                              // this can be removed

    return y;
}

```

A mesmerizing piece of code so inspiring I can't help but return to bask in it's
glow every few months. No other lines of code have ever made me want to sit down
and write code more than this has.

- Great YouTube video explaining the algorithm:
	- [Fast Inverse Square Root by Nemean](https://youtu.be/p8u_k2LIZyo "Nemean on
  Youtube")
- Medium post with some more of the history behind it:
  - [The Legendary Fast Inverse Square Root](https://medium.com/hard-mode/the-legendary-fast-inverse-square-root-e51fee3b49d9)


## IOCCC - Yusuke Endoh's ASCII Fluid Simulation
IOCCC, or the International Obfuscated C Code Contest is an annual "celebration
of C's syntactical opaqueness" that first started in 1984.

This particular 2012 submission by Yusuke Endoh (Also a developer of Ruby) is an
ASCII fluid simulation, that can use it's own source code as a starting
configuration:

**endoh1.c**
```c
#  include<stdio.h>//  .IOCCC                                         Fluid-  #
#  include <unistd.h>  //2012                                         _Sim!_  #
#  include<complex.h>  //||||                     ,____.              IOCCC-  #
#  define              h for(                     x=011;              2012/*  #
#  */-1>x              ++;)b[                     x]//-'              winner  #
#  define              f(p,e)                                         for(/*  #
#  */p=a;              e,p<r;                                        p+=5)//  #
#  define              z(e,i)                                        f(p,p/*  #
## */[i]=e)f(q,w=cabs  (d=*p-  *q)/2-     1)if(0  <(x=1-      w))p[i]+=w*/// ##
   double complex a [  97687]  ,*p,*q     ,*r=a,  w=0,d;    int x,y;char b/* ##
## */[6856]="\x1b[2J"  "\x1b"  "[1;1H     ", *o=  b, *t;   int main   (){/** ##
## */for(              ;0<(x=  getc (     stdin)  );)w=x  >10?32<     x?4[/* ##
## */*r++              =w,r]=  w+1,*r     =r[5]=  x==35,  r+=9:0      ,w-I/* ##
## */:(x=              w+2);;  for(;;     puts(o  ),o=b+  4){z(p      [1]*/* ##
## */9,2)              w;z(G,  3)(d*(     3-p[2]  -q[2])  *P+p[4      ]*V-/* ##
## */q[4]              *V)/p[  2];h=0     ;f(p,(  t=b+10  +(x=*p      *I)+/* ##
## */80*(              y=*p/2  ),*p+=p    [4]+=p  [3]/10  *!p[1])     )x=0/* ##
## */ <=x              &&x<79   &&0<=y&&y<23?1[1  [*t|=8   ,t]|=4,t+=80]=1/* ##
## */, *t              |=2:0;    h=" '`-.|//,\\"  "|\\_"    "\\/\x23\n"[x/** ##
## */%80-              9?x[b]      :16];;usleep(  12321)      ;}return 0;}/* ##
####                                                                       ####
###############################################################################
**###########################################################################*/
```

Self-documenting code in the worst way.. 

- Here is the IOCCC hint page for this submission:
  - [Most Complex ASCII Fluid](https://www.ioccc.org/2012/endoh1/hint.html)
- And a YouTube video of the simulation in action:
  - [ASCII fluid dynamics by Yusuke Endoh](https://youtu.be/QMYfkOtYYlg)


## "How I cut GTA Online loading times by 70%"
Fixing an JSON parsing bug in an 8 year old game that's made over $6.4 billion
since launch.
An incredible story of profiling, decompilation and reverse-engineering to fix a
6 minute load time that dropped down to under 2 minutes. 

- His blog post detailing his journey & the fix:
  - [How I cut GTA Online loading times by 70%](https://nee.lv/2021/02/28/How-I-cut-GTA-Online-loading-times-by-70/)



## Effective Java by Joshua Bloch
Not exactly an internet gem, but a book that I really enjoyed reading, with some
serious great advice and guidelines on writing better Java code. A must read in
my opinion for someone at an _intermediate_ or so level of Java.

Personally it really helped me overcome some imposter syndrome type feelings,
and something I like to come back to every now and then to refresh my memory.

- [Effective Java (Third Edition)](https://www.oreilly.com/library/view/effective-java/9780134686097/)


## The case of the 500-mile email

A classic funny tech support story from the mid 90s, a university professor that
"can't send an email more than 500 miles". A great story if you've never read it
before, and certainly one of my favourites out there.

- [The case of the 500-mile email, Trey Harris](https://www.ibiblio.org/harris/500milemail.html)


## The Hutter Prize
A compression challenge to compress 1GB of text from Wikipedia. Marcus Hutter
launched the challenge in 2006 with a €50,000 prize, and has recently (2020)
been upped to a €500,000 prize. 

The motivation behind the contest is the idea that general compression is
closely related to intelligence, and whilst intelligence is a difficult concept
to quantify, file sizes give a hard quantitative score. The contest has been
largely dominated by Alexander Rhatushnyak, although the most recent winner is
Artemiy Margaritov in May 2021 who compressed the 1GB file down to just 115MB.

The compression techniques used are pretty magical, and the contest as a whole
is a great rabbit hole to get lost in for an afternoon.

- [Compressing Human Knowledge](http://prize.hutter1.net/)


## Ben Eater

Hours and hours of detailed video content building breadboard computers.
Fantastic for learning how computers work, both the 8-bit and 6502 computers
from scratch series are a great intro into that space. Covering from the very
bottom with logic gates, all the way to functional breadboard computers and ASM.

- 8-bit computer from scratch:
  - [Website Page](https://eater.net/8bit)
  - [Youtube Playlist](https://youtube.com/playlist?list=PLowKtXNTBypGqImE405J2565dvjafglHU)
- 6502 breadboard computer:
  - [Website Page](https://eater.net/6502)
  - [Youtube Playlist](https://youtube.com/playlist?list=PLowKtXNTBypFbtuVMUVXNR0z1mu7dp7eH)


## 
 
