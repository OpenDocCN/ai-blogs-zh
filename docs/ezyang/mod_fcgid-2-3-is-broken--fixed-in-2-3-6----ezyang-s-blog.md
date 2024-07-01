<!--yml
category: 未分类
date: 2024-07-01 18:18:03
-->

# mod_fcgid 2.3 is broken (fixed in 2.3.6) : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/11/mod_fcgid-is-broke/](http://blog.ezyang.com/2010/11/mod_fcgid-is-broke/)

## mod_fcgid 2.3 is broken (fixed in 2.3.6)

This is a post to get some Google juice for a problem that basically prevented [Scripts](http://scripts.mit.edu) from being able to cut over from Fedora 11 to Fedora 13\. The cluster of new machines kept falling over from load, and we kept scratching our heads, wondering, “Why?”

Turns out, the [following commit](http://svn.apache.org/viewvc?view=revision&revision=753578) broke mod_fcgid in a pretty terrifying way: essentially, mod_fcgid is unable to manage the pools of running FastCGI processes, so it keeps spawning new ones until the system runs out of memory. This is especially obvious in systems with large amounts of generated virtual hosts, i.e. people using mod_vhost_ldap. It got fixed in mod_fcgid 2.3.6, which was released last weekend.

*Unrelatedly.* I’ve been sort of turning around in my head a series of *Philosophy of Computer Science* posts, where I try to identify interesting philosophical questions in our field beyond the usual topics of discussion (cough AI cough.) The hope is to draw in a number of topics traditionally associated with philosophy of science, of mathematics, of biology, etc. and maybe pose a few questions of my own. One of the favorite pastimes of philosophers is to propose plausible sounding theories and then come up with perplexing examples which seem to break them down, and it sounds like this in itself could generate some Zen koans as well, which are always fun.