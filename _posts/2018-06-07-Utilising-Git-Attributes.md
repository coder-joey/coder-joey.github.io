---
layout: post
title: Utilising Git Attributes to Exclude Items from Releases
---

I'm thinking I may be a little behind the times on this one... 
After a little while of releasing my WordPress plugin with multiple "developer-only" directories I stumbled upon the wonder that is .gitattributes

You can read all the amazing things you can do with this file [here](https://git-scm.com/docs/gitattributes) but to sum up this is a file just like .gitignore, added to the root of your repository that allows you to apply attributes to files/folders within your project.

There is a tonne of information to digest in that link but or me the eye opener here was learning of the ability to create an archive (a new release in my case) that excludes chosen files/folders.
For more information about what you can do when creating an archive check out [this page](https://git-scm.com/docs/git-archive)

So to the .gitattributes file itself. For my project I was only interested in this one function: "when creating an archive exclude these files", welcome `export-ignore`.
Here's an example .gitattributes file to show just how easy it is:
<pre>
.gitattributes export-ignore
.gitignore export-ignore
docs export-ignore
docs/** export-ignore
tests export-ignore
tests/** export-ignore
</pre>

One important thing to note is that when excluding an entire directory just the directory name plus a forward slash e.g. *tests/* doesn't cut it, you instead need to use *tests/\*\**

And that's it! Clean up those releases!
