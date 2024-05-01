---
title: "Zeropoint Security: DevOps for Pentesters Course Review"
layout: post
---

### What is it

[https://training.zeropointsecurity.co.uk/courses/devops-for-pentesters](https://training.zeropointsecurity.co.uk/courses/devops-for-pentesters)

This is a really bite-sized course which really focuses on a high-level introduction to what DevOps is for pentesting, some basic git usage, best practices, along with using TeamCity to automatically obfuscate and test builds.

At the time of writing this is the cheapest course Zero Point Security offers at **£26.15** ($32 USD or $45 CAD for me). It has 30 lessons and several have a video associated with them. 

You'll ideally have a development VM already setup with various tools to use along with the course material.

I had one Windows 10 VM with the following installed:

- [Visual Studio 2022](https://visualstudio.microsoft.com/downloads/) (Not required, VS2019 will do fine if you already have it)

- [Visual Studio 2019](https://learn.microsoft.com/en-us/visualstudio/releases/2019/history#release-dates-and-build-numbers) (make sure to install .NET 4.0 and .NET 4.5 targeting packs too)

- [Visual Studio Code](https://code.visualstudio.com/)

- [Git](https://git-scm.com/downloads)

- [PowerShell Core](https://github.com/PowerShell/PowerShell)

- Plus the tools the course walks you through

### Who is it for

This course really just introduces you to these topics for you to explore and expand on yourself. It doesn't really cover extended obfuscation techniques, or go too deeply on avoiding IOCs on public tools. That's completely understandable, but it's something I wanted to point out.

I am by no means a DevOps pro, but I knew many of these topics decently enough, but I did pick up some new tidbits here and there. So as far as who it's for, I'd say pick it up if you want to learn more about DevOps at a higher-level. The course will point you in a direction to expand and create an entire library of obfuscated artifacts for you and your team to use.

### Overview

Overall I thought it was good. 

Some of the lessons lack a bit in content in my opinion, and act more as a tutorial on how to install tools or edit build steps which leaves a bit to be desired (but keep the price in mind).

If the course was a bit more expensive, with more material and examples from Rasta's own workflows that'd be even better. However at this price I think the content makes sense. It is well structured and easy to follow along. It's a quick course to fly through on a weekend and take some knowledge from.

I am currently taking Red Team Ops II, so this course was a quick break from that, and they are both complimentary to each other.

That is really it, not much else to say! I will definitely be spending more time getting all of my favorite tools integrated into my CI/CD pipeline.

Thanks for reading, hopefully this helped you make your decision.
