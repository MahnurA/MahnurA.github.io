---
layout: post
title:  "C: For loops and signal handling. Why I couldn't get both to work?"
date:   2019-03-14
categories: [Miscellaneous]
---

# Background

Right around exactly one year from now I was first introduced to C through the Systems Programming course I took. I was both terrified and beyond excited. 

Fast forward two months, and I was making a server-client model, [https://github.com/MahnurA/Systems-Programming](https://github.com/MahnurA/Systems-Programming), with the server emulating a handful of Linux's shell behavior. Somewhere in the middle of it's implementation I ran into a very weird issue.

# The problem

The signal handler wasn't working, at all. It would work the first time, which is akin to catching a signal without a handler and when I sent a signal in a loop, or repeatedly from another terminal the signal wouldn't be caught except just the one time. The signal handler was defaulting to normal behaviour after the first time and not remaining persistent. 

My code essentially at this point was handling signals as shown below:

<pre><code class="c">
void sig_handler(int signo){
  if (signo == SIGTERM){
    write(STDOUT_FILENO, "signal caught\n", strlen("signal caught\n"));
  }
}
int main(){
  signal(SIGTERM, sig_handler);
  while (1) ; 
  return 0; 
} 
</code></pre>

This would print “signal caught” when I sent a **SIGTERM** signal to the process running this code the first time, and “Terminated” the second time. The process was terminating without any errors. 

**First thoughts**
1. there is something wrong with the code
2. there is something wrong with my understanding of how signal handlers work (less likely)
3. ???

The first two were checked easily, between comparing code from examples and going over my professor's explanations I was confident these weren't the causes. However, some googling also showed me that I should consider using **sigaction()** instead of **signal()**. **Sigaction()** returned a multitude of errors and warnings most of which I didn't want to venture into solving as I hadn't yet studied **sigaction()** properly to be sure I understood it completely and secondly more importantly I needed **signal()** to work as per the assignment I was given.

Finally I gave my code to a friend and asked her to run it, believing there might be some error somewhere otherwise in my code that was interfering with my signal handling somehow. Loose thoughts, I know. The code ran, flawlessly. 

Now, I was stuck. This made no sense. We ran the code side by side, her on her Red Hat and while I ran it on Linux Mint, and I couldn't quite believe my eyes, when my code wouldn't run on my system but it did on Red Hat. 

**Next conclusion:** The code was fine, maybe slight kernel differences and how inherently signals are received by the two different systems were the culprit? My professor had warned that Mac OS might behave differently than Linux, so I tried to extrapolate that argument here. Some digging though showed I was definitely not on the right path, and Linux Mint shouldn't be producing any unexpected behavior like this.

I was stumped. 

Then, one day, my professor ran an example code on my computer and I noticed that the signal handler worked! Thinking back, I realized he had compiled the code without the **-std=c99** flag I was adding by default. 

The reason I was compiling with that additional flag was because right at the start of my introduction to C, I was hit with my first error.
If I wrote the for loop as below:

<pre><code class="c">
int main(){

    for(int i=0; i<2; i++){
        printf("%d\n", i);
	}
} 
</code></pre>

My compiler was giving the following error if compiled without any flags: 

![My helpful screenshot](/assets/cerror.png)

I had two options according to some online searching  and the compiler warning, either declare the variable used in the for loop outside the for condition bracket,

<pre><code class="c">
int main(){

    int i;
    for(i=0; i<2; i++){
        printf("%d\n", i);
        }
} 
</code></pre>

Or just compile with the **-std=gnu99** flag or **-std=c99** flag. I went with the option of adding this since I really didn't want to change the way I wrote my for loops. And since my compiler seemed to be emphasizing it was a c99 incompatibility error, and most online searching pointed to the latter flag as a way of resolving the for loop error, I chose to add the **-std=c99** flag. I didn't think it'd be a problem. Until I started trying to handle signals, that is.   


# **The solution**

My first instinct after the revelation of **-std=c99** being the culprit was first to just drop the flag, that took me back to my loop problem. Next, I tried adding the **-std=gnu99** and checked. Did both my signal handler and loop work as they should? Yep, they did :D

# Why does the signal handler not work when C code is compiled with the **-std=c99** flag?

When I was adding the **-std=c99** flag, I was explicitly directing that the code be compiled by the C standard C99 which should have allowed signals to work normally as according to the signals() man page [https://linux.die.net/man/2/signal](https://linux.die.net/man/2/signal),  **signal()** conforms to C89, C99 and POSIX.1-2001. 

However, its portability varies greatly, *“In the original UNIX systems, when a handler that was established using **signal()** was invoked by the delivery of a signal, the disposition of the signal would be reset to **SIG_DFL**, and the system did not block delivery of further instances of the signal,”* and that System V also provided similar semantics. 

The situation on Linux is as follows: The kernel's **signal()** system call provides System V semantics. 

This seemed to cement the fact that signals shouldn't work correctly for me since I have Linux, but the man page also goes on to say that *“by default in glibc 2 and later, the **signal()** wrapper function does not invoke the kernel system call. Instead, it calls **sigaction(2)** using flags that supply the BSD semantics.”* 
The BSD semantics are equivalent to calling **sigaction(2)** with the following flags: 
sa.sa_flags = **SA_RESTART**;     

In short, what I needed was BSD semantics and not System V's, so **signal()** could call **sigaction()**.

I have GLIBC version 2.19. You can check your version by typing the following command in the terminal:
**ldd --version**

My version should have been causing my compilation of code to adhere to BSD semantics but since I was invoking gcc with the **-std=c99** option I was getting System V again. 

*“On glibc 2 and later, if the **_BSD_SOURCE** feature test macro is not defined, then **signal()** provides **System V** semantics. (The default implicit definition of **_BSD_SOURCE** is not provided if one invokes gcc(1) in one of its standard modes (-std=xxx or -ansi) or defines various other feature test macros such as **_POSIX_SOURCE, _XOPEN_SOURCE,** or **_SVID_SOURCE**”*

However when I put **-std=gnu99** instead of **-std=c99** it worked? Here, what I conclude is when I added the latter flag, gcc would revert to System V semantics as my flag would explicitly direct C standard, while gcc's version would be set to BSD semantics by default, so there'd be a conflict. And in case of conflicts, BSD semantics are disfavored. 
[https://linux.die.net/man/7/feature_test_macros](https://linux.die.net/man/7/feature_test_macros)

With the former, gcc by default follows BSD semantics. Gnu99 allows C99 functions along with other functions like **sigaction()**, part of POSIX, to also be available for usage. So by adding the **-std=gnu99** I was getting both, which allowed my signal handling to work properly.  
Going back to the for loop problem, loops with the variable used, declared within the condition bracket, are only supported in versions equal to, or after, C99. 

Since I was using gcc, by default, my code is compiled with the gnu89 standard. This does not support loops where the variable is declared within the condition bracket, but does support signal handlers. By default, gcc does not explicitly follow any ANSI/ISO C standards. For versions below 5.1.0, the default version is roughly equivalent to **-std=gnu90**, which is the 1989/1990 C standard with GNU-specific extensions. For version 5.1.0 the default is **-std-gnu11**.

I have gcc version 4.8.4, which resulted in loops not working the way I wanted them to. 

Signals work as they should with all gnu versions -std=gnuxx, and not with any version of strictly -std=cxx

**P.S:** During my search into this rabbit hole, some information points to gcc being very strict on Linux but fairly flexible on FreeBSD  so adding the **-std=c99** flag should still allow gnu extensions to remain enabled on BSD. 




