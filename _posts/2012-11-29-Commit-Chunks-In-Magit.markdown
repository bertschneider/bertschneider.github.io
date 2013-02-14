---
layout: post
title: Commit Chunks in Magit
tags: git magit emacs chunks commit
---

# Commit Chunks in Magit

A few days back I found a nice little feature in the 
[Emacs](http://www.gnu.org/software/emacs/)
[Magit](http://www.emacswiki.org/emacs/Magit) Mode.
 
If you work on one of your versioned files you might end up with
something like this:

![Chunks in th Magit Status View](/images/magit-chunks.png)

There are changes in different places in the file but to create a
cleaner commit you just want to commit some of them. 

To do so one could use [`git commit
-p`](http://stackoverflow.com/questions/1085162/how-can-i-commit-only-part-of-a-file-in-git).
As many posts state this is a pretty complicated command so you
probably don't want to do that if not really necessary.

Magit to the rescue!

Just select the stuff you want to stage in the Magit diff and press
`s` like you would do in the normal stage process. That's all. Easy,
right?

Here is a look after a chunk is staged. 

![Staged chunk in th Magit Status View](/images/magit-chunks-staged.png)

