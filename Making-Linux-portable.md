Making Linux portable
=====================

I've been interested in Linux for many years but every time I try to use it
for real work I find it handicapped. When I speak of Linux I am using
generalization. I don't mean just the kernel itself, nor a particular distro,
but in the Linux world the things from distro to distro doesn't really differ.
Maybe they differ from the point of view of someone who forked a distro and
added a bit of his own flavour, but from the point of view of an outsider it
is all the same - same filesystem, same software, same issues copied
everywhere. Just the skins are different. I don't mean to disrespect all the
work of all people who are doing it for free and sharing it. I recognize
distros are constantly evolving and there is a lot of high quality work and in
some areas Linux excels over proprietary OS-es. Nevertheless in many areas
Linux lacks a lot and it doesn't need to be like this. Why do I say I find it
handicapped. Because it is "do it yourself", which is fine for a hobby, but
even if you are into this kind of stuff when you want to get some other work
done you want to deal with the work, not with the tools. And Linux is not the
tools to get the work done because the tool itself require more work and
effort than buying a commercial OS and finishing your work (and profiting from
it). But I still keep an eye on Linux and hope things will improve (and they
do of course, just not with the pace I hope for).

This time I decided to dig a little deeper and see what I can do myself. I
started building a Linux from scratch and see what I can do about the problems
I find with Linux. I've also spent a lot of time with FUSE trying to utilize
it to find workarounds and abstract some of the problems temporarily until I
can get something workable in the shape I want. But to no avail, it is all so
entangled that to make even the highest level stuff work right you meet so
many show stoppers with anything that you may depend on, that the work to
disentangle everything amounts to (and actually surpasses) writing an OS from
scratch. This is neither a one man job nor my goal.

### The file system is awful
The biggest concern for me always was the file system, or rather its
structure. I could never imagine how people find it perfectly normal to group
your programs in the same directory. This is source of many problems. The
overall dependency handling is major issue in the Linux world, this is not a
secret to anyone. For me it is obvious that putting separate programs in the
same directory can only lead to trouble. Maybe this was fine in the early days
when there wasn't much software but nowadays it is different. Besides all of
this, the file system is full of things like "/dev", "/proc", "/etc" some of
which are not even real files but some kernel stuff. This is fine if you are
only using Linux to develop Linux itself. But from a user perspective my OS is
a tool to get my work done. By "user" I don't mean just someone who uses
primarily a word processor. I deal with servers, development things like that,
but I'm still a user.  It is not my job to care for the OS, to fix the OS, to
etc the OS. Sometimes this is fine when I want to do it, but I shouldn't be
forced to do it all the time. My hard drive is to store my own stuff, it is
not meant so that the OS will spill its guts all over it. Not only it is ugly
and dirty, it is very dangerous. I don't mind if the OS has its own corner as
it is doing great job of running my hardware, but with Linux it is the other
way around - the user has his/her corner in the midst of the kernel stuff on
his/her own storage device. Last day I was recompiling the kernel to do some
experimentation and did "make O=/some/target/dir install" as it says in the
docs, it worked great and the compiled kernel image went to the output
directory but everything else (kernel modules and firmware) went to my root
filesystem and my system would no longer boot. It is impossible to tell which
file belongs to which application or to the OS itself and everything is messed
up. And actually instead of improving it, someone (I presume in the early
years of computing) turned this mess into a "standard", like it was the end of
computing and it has no need to change or it was so great, and now people use
this as an excuse not to change because it will deviate from the "standard".
Standard says who? Come on, everyone can write one.

### Everything is terribly unportable
All applications, even the kernel assume a predefined file structure and
applications. For example to compile the kernel you need to to have
"/bin/bash", it doesn't matter if you have bash installed in your path, it
won't work if you don't have it in "/bin". To compile glibc you need to have
"/bin/pwd" and bunch of other stuff in "/bin". To compile man-pages you need
to have "/tmp", it doesn't matter if you have $TMPDIR defined because it has
hardcoded "/tmp" path. And the problem is that first of all I don't have the
flexibility to shape my own system. Since everything in this [Linux] realm is
open source, software somehow loses its identity and few people care to polish
their software as distinct product. It seems like the authors of "man-pages"
for example doesn't view their stuff as distinct product that should work
independently of hard coded file system, but rather a product for OS that
follows a certain file system hierarchy standard. How comes the practices used
in some open operating systems become so entangled with the notion of openness?
What is open about it if to use some "open" software I have to adhere to the
filesystem rules of another system? Is it really so hard to write $TMPDIR in
your build script rather than /tmp? This contradicts all the freedom stuff
that is so popular in the Linux realm. Distinct doesn't have to mean "paid" or
"not free", quite the opposite. Freedom is also the freedom to use the
product. If you care about freedom and open source you would want people to be
free to use your product in any environment, not to have to read books and
become developers to be able to use it. Not to mention lot of this software is
not of highest quality. Even within Linux, lets say I want to "chroot" to some
other directory. To be able to do so I need to duplicate the whole "/bin /tmp
/etc /lib" structure within the chrooted folder or nothing will work. And this
is not all. Even the dynamically linked executables has hardcoded paths in
them that can not be changed without patching the binary - that is the path to
the ELF interpreter. There is no option to use different interpreter or at
different location/file name. This means I can not freely distribute my
application in compiled form because the user need to have the same file in
the same location as me. Not very "dynamic" behaviour I would say. This (among
other things) leads to the result that every distro needs to maintain its own
repository for all the available software so its users can actually use it.
Although the software from one distro is perfectly binary compatible to every
other distro it is still not very compatible. This simply doesn't make sense.
It is not the job of the distro to know about and maintain compatibility with
all available software. It is job of the developers to compile version of
their software for a given OS (and CPU architecture) and distribute the same
version to all users (and in this case to all Linux distros compiled for the
same CPU). Just imagine if Microsoft had to build version of Google's (and
everyone else's) software for every version of Windows. This is simply
ridiculous. Just because Linux is mostly open source and it is possible to
take such approach it doesn't mean it is not completely wrong. If I want to
have my own distro I want to work to perfect the overall feeling of my distro,
no to care about every possible piece of software in the world. I know it is
free, everyone can do it himself, blah blah. But even if I *can* do it myself
it doesn't mean I *want* to or that I *should need* to. The developers of the
software already did the work of compiling and testing it... and shared their
work. So when I want to use their work to do my work, this is what I want to
do, not to repeat their work of compiling and testing. Not to mention I don't
even need to know about this stuff, maybe all I need to know is how to use a
word processor. But since Linux is mostly used by developers or hobbyist who
already know how to deal with this stuff and they probably like it, they don't
seem to notice.

### Hygiene
This is very much related to my points about the filesystem, but applications
(i.e. their developers) need to learn some hygiene. Just as it makes no sense
to put all applications in the same folder it makes no sense to put all
configuration of all apps directly in the user folder. It becomes a mess where
you don't know which file belongs to which app, and which file breaks what. It
is ugly and it invades user's space. Not only the kernel invaded all my hard
drive and stuck me in the corner of /home/me, but now all apps take this
corner from me and I need to make /home/me/workspace to have a sane, clean
space for my own files. What's exactly the problem of having
/appdata/user/bash/bash_history instead of /home/user/.bash_history ? Or even
/home/user/appdata/bash/.bash_history if you want to copy all your user's
stuff at once, but in a sane, clean structure. And of course it shouldn't be
hard coded into apps. They simply need to use getenv("APPDATA"), or something
similar, not even longer to write than hardcoding a path!

### Related
There is an excellent article from the author (or someone of them, I don't
know) of GoboLinux called [I am not clueless](http://www.gobolinux.org/index.php?page=doc/articles/clueless). It
is very related to this post and explained by someone far more experienced
with Linux than myself.

### Other problems
There are others that really bother me but they are not subject of this
article at this point. Although I'm tempted to discuss the kernel design and
drivers, it wouldn't really differ much from the rest of this article. It has
the same fundamental design problems as the rest of the ecosystem.
Unfortunately it is not restricted to the file system or anything particular
but just all-encompassing entanglement, lack of structure and lack of
discipline and love on every level.

### Conclusion
Although my article is critical, I like open source in general and Linux in
particular. I've published most of my own work as open source. I want to
evolve Linux so it has the build quality, usability, performance and eye candy
of OSX and the portability of Windows, but remaining open and free. In my
point of view "disentangling Linux" is the first and most important step to
making it portable, and making it portable is the most important step to
evolving it and creating a solid ground and future for open software. By being
portable and flexible people will not duplicate all the effort and deal with
the same problems with every distro. They can put their energy to making their
distro distinct in the way they want. Distro maintainers woudn't have to
compile version of everything for their own distro, wouldn't have to invent
package manager for their own distro. Yet I understand there is no one person
or organization standing behind and funding Linux so it can develop in a
single direction. Much of the effort is done by individuals and they do it in
the direction that will get their own needs fulfilled quickly and easily, then
because others don't want to duplicate the work or simply can't because they
are not developers, but want to use the software, they adopt the individual
mini changes of everyone and it becomes a mess. I see some vision in the
community. Many distros recognize some of the problems (and I don't mean only
the ones I expose here), the guys behind Ubuntu in particular seem to have
greater vision, not to say I like Ubuntu but no wonder it became so popular.
And many other distro specialize on certain functionality to help actually get
work done and not deal with the tools for the job. But for me it is
unfortunate that influential people in the community doesn't see the need for
portability and modularity and want to keep things the way they were in 1979.
Just because everything is free doesn't mean you have to depend on everything
or you have to deal with the source of everything in every scenario. Things
being modular and portable doesn't mean they shouldn't be free or open or GPL.
They just should be sane. I read an article recently titled in the lines of
"ten things I hate about git", and one of the excellent points made there,
which I believe applies to much of the Linux software and the kernel itself,
is that it is hard because the authors didn't care about making it easier, not
because it has to be hard to be powerful. "It is hard, deal with it.". It is
very unfortunate that influential people that guide the community have this
attitude, and in a sense all developers also guide the community. This is not
the attitude of being open, this is rather selfish and closed attitude. Yes
sure, someone made some effort and is giving it for free. He/she is not
obliged to make any additional effort. But the thing is it is not really an
additional effort, it is just a slight attitude change and change of habits
dating back to the early days of computing. Particularly for people who are so
"concerned" about openness and freedom and make almost their \[life\] goal to
preserve it.

If you like this reading or feel in similar way about your Linux experience
you may also like [NixOS](http://nixos.org) and [GoboLinux](http://www.gobolinux.org).

Authors
-------
Borislav Peev (borislav.asdf at gmail dot com)