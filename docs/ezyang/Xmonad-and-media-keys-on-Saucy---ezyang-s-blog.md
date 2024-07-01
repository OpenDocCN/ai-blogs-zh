<!--yml
category: 未分类
date: 2024-07-01 18:17:16
-->

# Xmonad and media keys on Saucy : ezyang’s blog

> 来源：[http://blog.ezyang.com/2013/10/xmonad-and-media-keys-on-saucy/](http://blog.ezyang.com/2013/10/xmonad-and-media-keys-on-saucy/)

Ubuntu continues on its rampage of breaking perfectly good software, and on my most recent upgrade to Saucy Salamander, I discovered to my dismay that my media keys (e.g. volume keys, fn (function) keys, suspend button, etc) had stopped working. Of course, it worked fine if I logged into my user using Unity, but who wants to use a silly window manager like that...

The root problem, according to [these Arch Linux forum posts](https://bbs.archlinux.org/viewtopic.php?pid=1262471) is that Gnome has moved media-key support out of `gnome-settings-daemon` (which any self-respecting Xmonad user is sure to spawn) and into their window manager proper. Which, of course, is no good because I don’t want to use their window manager!

For now, it seems the simplest method of bringing back this functionality is to run a 3.6 version of gnome-settings-daemon. Fortunately, at least for Saucy, there are a few builds of 3.6 available before they upgraded to 3.8\. So, all you need to do is grab these two deb files appropriate for your architecture (you need gnome-control-center too, because it has a dependency on gnome-settings-daemon):

Once you've downloaded the appropriate deb files, a `dpkg -i $DEBFILE` and then `apt-mark hold gnome-control-center gnome-settings-daemon` should do the trick. You should run an `aptitude upgrade` to make sure you haven't broken any other dependencies (for example, `gnome-shell`). (Power-users can add the debs to a local repo and then downgrade explicitly from `apt-get`.)

Moving forward, we will probably be forced to reimplement media key bindings in some other software package, and it would be nice if this could be standardized in some way. Linux Mint has already forked gnome-settings-daemon, with their [cinnamon-settings-daemon](https://github.com/linuxmint/cinnamon-settings-daemon), but I've not tried it, and have no idea how well it works.

**Update.** Trusty has an updated version of this package which restores support, so I am providing backports [via my PPA.](https://launchpad.net/~ezyang/+archive/ppa)