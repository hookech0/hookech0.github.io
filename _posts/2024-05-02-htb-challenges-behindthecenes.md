---
title: "HTB Reversing Challenges: Behind the Scenes"
layout: post
---

### Intro

I am brand new to reverse engineering but it has always been a topic I've had an interest in. I decided to play around with some of the HTB reversing challenges as they are very accessible, and seemed to be a good option to dip my toes into. I started with the easiest challenge, "Behind the Scenes" and documented the work below.

### Functionality

The binary looks like ELF execuatble which is expecting a password when ran:

![Running the binary](/assets/screenshots/behindthescenes/runningthebin.png)

### Solving Decompliation Problems

For this first challenge I opted to use Ghidra. There's so many different option but Ghidra seemed accessible and made sense for my limited light usage.  

To start I imported the `behindthescenes` binary into Ghdira and began analysis which only took a few seconds.

I clicked through the Sumbol Tree to get to the `main` function. Here I found an issue:

`invalidInstructionException();`

![Running the binary](/assets/screenshots/behindthescenes/invalidinstruction.png)

Following this in the Listing showed a `UD2` instruction which I was not familiar with. Researching this instruction along with Ghidra led me to a git issue discussing the topic: https://github.com/NationalSecurityAgency/ghidra/issues/4113

![UD2](/assets/screenshots/behindthescenes/ud2)

Based on that thread, it looks like it is working as intended, but as a workaround they suggest to swap the `UD2` instructions with a `NOP`.

UD2 (or Undefinied Instruction 2) is typically used to intentionally cause an exception for debugging. So it makes sense that swapping to `NOP` instructions would help Ghidra complete the decompilation.

https://www.felixcloutier.com/x86/ud

So I did just that, using `Ctrl-Shift-G` I manually patched each `UD2` instruction to a `NOP`

![Patching UD2 to NOP](/assets/screenshots/behindthescenes/patchtonop.png)

![More invalidinstructions popping up](/assets/screenshots/behindthescenes/moreinvalidinstructions.png)

As shown above, we get another `invalidInstructionException` in the decompile window.

I then selected all of the addresses which failed to decompile under the first `UD2` instruction in the Listing window and hit `d` to find the rest.

![Decompiling after the first NOP patch](/assets/screenshots/behindthescenes/highlighttodecompile.png)

After this, we get more `UD2` instructions we have to patch out.

![More UD2 instructions revealed](/assets/screenshots/behindthescenes/moreud2s.png)

Continuing this process, more and more of the code is decompiled, which starts to resemble something interesting:

![Parts of the password string revealed](/assets/screenshots/behindthescenes/uncoveringthestring.png)

Finally, Ghidra provides what looks like the password for the binary, `Itz_0nLy_UD2`.

![Binary password](/assets/screenshots/behindthescenes/fullstring.png)

Supplying that to the binary returns the flag for the challenge:

![Obtaining the flag](/assets/screenshots/behindthescenes/flag.png)

### Conclusion

This was a great little challenege to sink my teeth into and get more familiar with Ghidra and some very simple reversing concepts.

### Tools Used

- Ghidra | https://github.com/NationalSecurityAgency/ghidra/