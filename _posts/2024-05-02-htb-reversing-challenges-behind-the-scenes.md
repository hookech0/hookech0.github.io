---
title: "HTB Reversing Challenges Behind the Scenes"
layout: post
---

### Intro

I am brand new to reverse engineering but it has always been a topic I've had an interest in. I decided to play around with some of the HTB reversing challenges as they are very accessible, and seemed to be a good option to dip my toes into. I started with the easiest challenge, "Behind the Scenes" and documented the work below. I am hoping to continue this and complete all of the currently active challenges.

### Functionality

The binary looks like ELF executable which is expecting a password when ran:

![Running the binary](/assets/screenshots/behindthescenes/runningthebin.png)

### Solving Decompliation Problems

For this first challenge I opted to use Ghidra. There are so many different option but Ghidra seemed accessible and made sense for my limited light usage.   

To start I imported the `behindthescenes` binary into Ghidra and began analysis which only took a few seconds.

I clicked through the Symbol Tree to get to the `main` function. Here I found an issue:

`invalidInstructionException();`

![Running the binary](/assets/screenshots/behindthescenes/invalidinstruction.png)

Following this into the Listing window showed a `UD2` instruction which I was not familiar with.

![UD2](/assets/screenshots/behindthescenes/ud2)

Researching this instruction along with Ghidra led me to a [git issue]([https://github.com/NationalSecurityAgency/ghidra/issues/4113]) discussing the topic.

Based on that thread, it looks like it is working as intended, but as a workaround they suggest to swap the `UD2` instructions with a `NOP`.

UD2 (or Undefined Instruction 2) is typically used to intentionally cause an exception for debugging. So it makes sense that swapping to `NOP` instructions would help Ghidra complete the decompilation.

So I did just that, clicking on each `UD2` instruction and hitting `Ctrl-Shift-G` I manually patched each `UD2` instruction to a `NOP`.

![Patching UD2 to NOP](/assets/screenshots/behindthescenes/patchtonop.png)

![More invalidInstructions popping up](/assets/screenshots/behindthescenes/moreinvalidinstructions.png)

As shown above, we get another `invalidInstructionException` in the Decompile window.

I then selected all of the addresses which failed to decompile following the first `UD2` instruction in the Listing window and hit `d` to continue decompilation and find the rest of the `UD2` instructions.

![Decompiling after the first NOP patch](/assets/screenshots/behindthescenes/highlighttodecompile.png)

After this, we get more `UD2` instructions we have to patch out.

![More UD2 instructions revealed](/assets/screenshots/behindthescenes/moreud2s.png)

Continuing this process, more and more of the code is decompiled, which starts to resemble something interesting:

![Parts of the password string revealed](/assets/screenshots/behindthescenes/uncoveringthestring.png)

Finally, what looks like the password for the binary; `Itz_0nLy_UD2` is returned.

![Binary password](/assets/screenshots/behindthescenes/fullstring.png)

Supplying that to the binary returns the flag for the challenge:

![Obtaining the flag](/assets/screenshots/behindthescenes/flag.png)

### Conclusion

This was a great little challenge to sink my teeth into and get more familiar with Ghidra and some very simple reversing concepts.

### Tools Used

Ghidra | https://github.com/NationalSecurityAgency/ghidra/