---
layout: post
title: "The art of Vim"
published: true
---


Here is how a beginner, an expert and a Vim master would modify the following text, replacing `nice` by `text`:

    Vim is more than just a nice editor. 
    It's a long path through concepts and technics to handle nice. 


## Beginner

> The beginner knows nothing.

People who start learning Vim use it like some standard text editor with strange key bindings.
In order to delete a region of a sentence, here is what they would do:

1. Take the mouse
2. Left-click and visually select the word `nice`
3. Press `c` 
4. Type `text`

Repeat the same 4 steps for the second occurrence of `nice`


## Expert

> The expert thinks he know everything.

People with more experience know how to move fast without the mouse. They know
that moving through text is fundamental and so they became ninjas. 

1. Search `nice`: `/nice`
2. Replace `nice` by `text`: `Rtext`
3. Go to next occurrence: `n`
4. Repeat 2: `.`


## Master

> The master knows.

This category of people know how to act without moving. No matter how fast the
expert can move through text, the master just acts.


