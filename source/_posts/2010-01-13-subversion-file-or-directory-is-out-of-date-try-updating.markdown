---
layout: post
title: "Subversion: File or directory is out of date; try updating"
date: 2010-01-1 18:36
comments: true
categories: 
 - Subversion
 - Eclipse
---
I recently started encountering a strange problem with Subversion on one of my machines. In larger commits, I would suddenly get an error:

{% codeblock lang:bash %}
$ svn ci -m "My commit message."
Sending        src/Foo.java
svn: Commit failed (details follow):
svn: File or directory 'src/Foo.java' is out of date; try updating
svn: resource out of date; try updating
{% endcodeblock %}

Although I was positive the file(s) were up to date, I ran an svn up just to be safe:

{% codeblock lang:bash %}
$ svn up
At revision 10803.
{% endcodeblock %}

Nothing changed. If I tried to commit again, I would get the same out of date error. Eventually I figured out that if I delete the folder containing the problematic file, ran svn up to restore the working copy and re-edited (or restored from a backup) the modified file, the problem went away. This suggested an issue with the Subversion metadata, but this [blog post from Pageworthy](http://pageworthy.com/blog/2009/apr/14/subversion-says-your-file-or-directory-probably-ou/) describes a much easier work around:

{% blockquote Pageworthy http://pageworthy.com/blog/2009/apr/14/subversion-says-your-file-or-directory-probably-ou/ %}
Delete the all-wcprops file from the .svn metadata folder for the offending file.
{% endblockquote %}

In other words, if Subversion complains about `src/com/mycompany/Foo.java`, then delete `src/com/mycompany/.svn/all-wcprops`.

* OS: Mac 10.6
* IDE: Eclipse 20090920-1017
* Subversive Team Provider: 0.7.8
* Subversive Connectors: 2.2.1
* SVNKit 1.3.0: 2.2.1
* Subversion: 1.6.6