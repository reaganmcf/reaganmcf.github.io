+++
title="Just install Linux for a bit. You won't regret it"
description="As a person who is a tinkerer and loves learning about tech, switching to Linux for a few months was one of the best decisions I have ever made. Here is why you should too"
date=2021-07-16

[taxonomies]
tags = ["linux", "os"]
categories = ["programming"]
+++

# Just install Linux

Last semester I took my university's Operating Systems class, and to really dive fully into the material I decided to challenge myself to use Linux as my daily OS during the course. Now, keep in mind my Linux experience is very shallow. I know the basics from my Systems Programming class and `ssh` to servers in AWS, but that is about it. I had no idea how to use Linux for my everyday workloads.
This post describes my journey through various distros and applications that not only made me a smarter programmer, but introduced me up to a world I never knew about.

## KUbuntu

I had no idea which distro of Linux I wanted at first, but I did know that I wanted something Debian based as I was most familiar with the `apt` package manager. This was because most of my AWS work in the past was on Ubuntu LTS systems, and I also heard that Debian based systems (Ubuntu specifically) have really great driver and hardware support.

I'm usually pretty picky about how my things look, and I wanted something that looked great out of the box without having to tinker too much (mostly because I was too scared I was going to break stuff). This search led me into the world of desktop environments such as `GNOME`, `KDE`, `MATE`, etc.

I eventually decided to go with `KUbuntu` as KDE looked very appealing and I had _some_ experience with Ubuntu so I wouldn't be completely lost.

#### Pros

Using KUbuntu was a lot of fun, and I remember spending a good 2-3 weeks tinkering with _everything_ on my computer. Being able to change the entire theme of my Operating System was beyond liberating, especially coming from an unactivated version of Windows 10.
I also found myself, unlike before, using the terminal _much_ more. I was always much more comfortable with POSIX compliant terminals, and having one always by my side was a lot of fun. I started customizing my terminal, trying out new shells, tinkering with my dotfiles, etc.

It is around this time I really started getting into Open source software, and contributing to projects that I was using personally. It really taught me a lot about how git works, how to merge, make PRs, etc.

#### Cons

I had various audio issues throughout my time using `KUbuntu`, but it wasn't too bad.

The idea that _everything is customizable_ was a game changer for me, and I took it to the full extent. But, you definitely hit a wall with `KUbuntu` in terms of how far you can customize. As a result, I decided it was time to switch to something with a little more flexibility.

## Manjaro

The classic transition for any linux user is Ubuntu -> Arch (Manjaro is Arch based), and there is a reason for that. Once you get a taste of being able to make _everything_ how _you_ want it to be, you are always wanting to change something.
Manjaro provided much broader capabilities in terms of customization. However, there was one **major** drawback to keep in mind: you as a user are usually all on your own. Very few (to almost zero) companies are developing software with Arch linux users. As a result, you are often left searching for OSS solutions to your problem, or writing the OSS solution that solves your problem.

For some people like myself, this is great! While I have to work a little harder, I get to expose myself to even more technologies and learn more about Linux. However, for some this is (understandably) tremendously annoying and is a deal breaker for them.

#### Pros

I really liked how _lean_ I could get my daily setup to be. I had already switched from VSCode to just a terminal with all the `vim` plugins I needed, but I wanted to get even leaner of a setup. 
As a result to get rid of _everything_ I don't need, I decided to ditch my `Gnome` desktop environment for the `XMonad` window manager. If you have never used a window manager, it is a complete game changer for someone outside the linux world. 
I can never go back to not using one, and even use [Amethyst on macOS](https://github.com/ianyh/Amethyst) (which I love, you should check it out).

I ended up finding lots of software that I now have used everyday since, such as
- Amethyst
- Nvim
- Alacritty
- Tmux

If it wasn't for Arch, I wouldn't have had to search for software and instead would have just used whatever system default that existed.

#### Cons
It is around this time I started to learn that for _day to day_ use, Linux can be fairly frustrating. The more and more you customize, the less and less stable your system becomes. 

I ran into the following issues that almost made me revert back to Windows:
- While Zoom has a desktop application for linux, I was having to constantly restart my audio drivers in order to get my headphones/speakers to work correctly. This was fairly unprofessional in some settings.
- Cisco WebEx has absolutely 0 linux support, requiring you to use the buggy web based application which also never recognized my speakers and/or microphone.
- I was never able to get my Airpods to successfully work after many hours of attempts across both Manjaro and KUbuntu

## Why I switched back

While I really did like Linux, and I sometimes miss my Manjaro setup, I ended up selling my PC in order to buy the M1 macbook pro. macOS allows enough customization for the stuff I need (especially with Amethyst), and I can get my entire vim setup while also being able to fall back on the system reliability of macOS. 

For example, all of my internships have used Virtual Desktop environments and I doubt they are optimized to run on Linux. Also, having a broken Zoom during the pandemic was really frustrating and wasn't reliable for long term.

## What I learned

I learned so much about software, open source, and how Operating Systems actually work. Being able to make things for your Operating System when you run into an issue is very liberating, and tinkering with the source code of the applications you use is something you practically never experience on macOS or Windows.
The culture surrounding the entire space is very open and full of people wanting to learn more about technology. 

## Give it a shot
I hear so many Linux users telling people to switch to Linux and ditch Windows or macOS, which is a very dumb idea as it doesn't take into account what role a computer plays for most users. 
A vast majority of users need system stability as a priority. Not being able to get Zoom to work without restarting your audio drivers is simply a **non-negotiable deal breaker for almost everyone**.

And that's why I'm saying you should give it a shot for just a few months, and then switch back. You end up taking for granted the complexity of most of the systems we use today, as it is almost all hidden away.
There is a lot that goes into making an Operating System reliable, and until you get your hands dirty for a few months you won't be able to understand.

I don't think I would've gotten an A in my Operating Systems class if it wasn't for going down the rabbit hole myself. It has been the best learning experience for me.
