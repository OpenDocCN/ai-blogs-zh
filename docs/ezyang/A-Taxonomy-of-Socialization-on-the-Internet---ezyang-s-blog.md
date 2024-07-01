<!--yml
category: 未分类
date: 2024-07-01 18:17:45
-->

# A Taxonomy of Socialization on the Internet : ezyang’s blog

> 来源：[http://blog.ezyang.com/2011/06/taxonomy-of-socialization-on-the-internet/](http://blog.ezyang.com/2011/06/taxonomy-of-socialization-on-the-internet/)

## A Taxonomy of Socialization on the Internet

There’s *networking*, and then there’s *socialization.* Networking is establishing the link, knowing how to find and talk to the person if the need arises. Socialization is communication for the sake of communication; it strengthens networks and keeps people in touch. In many ways, it is the utility a social networking site provides. Trouble is, there are so many different ways to communicate on the Internet these days. In this blog post, I try to explain the essential differences of socialization over the Internet. I believe what is called *one-to-many socialization* is the fundamental characteristic of the newest social patterns (facilitated by Facebook and Twitter), and describe some of the challenges in designing methods for consuming and aggregating this information.

There are a few obvious ways to slice up communication methods. Here are a few:

*   *Media.* Most current applications are primarily text-based, but people have also tried audio, picture and video based modes with varying degrees of success.
*   *Length.* How long is the average message in this medium?
*   *Persistence.* If I ignore all activity older than an hour, how much do I miss? Are messages expected to be ephemeral or persistent? Is it even possible to send messages at all times?
*   *Distribution.* Is it one-to-one, one-to-many or many-to-many communication? (For the purposes of this discussion, we’ll define on-to-many when there is an asymmetric flow of information: so a “comment” system does not turn a conversation into many-to-many... unless it manages to capture a conversation of its own.) When there are many listeners, how do we select who is listening?
*   *Consumption.* How do we receive updates about communication?

We can look at a few existing networks and see how they answer these questions:

*   *Instant messaging* (IM). Text, short length, persistent but not always available, one-to-one, consumed via a chat client. Note that with the existence of multi-protocol chat clients, it’s not too difficult to navigate the multitude or protocols employed by people all over the world.
*   *Internet Relay Chat* (IRC) and *Jabber*. Text, short length, ephemeral, many-to-many, consumed via an IRC/Jabber client. There is some degree of integration between these protocols and instant messengers. Anyone can participate, but people can be kicked or banned from channels.
*   *Personal email.* Text, medium length, persistent, one-to-one, consumed via a webmail interface or a mail client.
*   *Mailing lists.* Text, medium length, persistent, many-to-many, consumed via webmail, mail client, or newsreader. For public mailing lists, anyone can participate.
*   *Forum.* Text, medium length, persistent, many-to-many, consumed via web browser.
*   *Blog.* HTML, medium to long length, persistent, one-to-many, consumed via web browser, possibly with an RSS reader. Anyone can subscribe to a public blog they are interested to.
*   *Twitter.* Text, short length, ephemeral or persistent, one-to-many (though one-to-one communication is possible with at-replies, and in even more rare cases, many-to-many with multiple at-recipients), consumed via web browser or a Twitter client. Anyone can follow a public Twitter user.
*   *Facebook Wall.* Text and images, short to medium length, ephemeral or persistent, one-to-many (but many-to-many with comments), consumed via web browser. Only friends can receive updates.
*   *Zephyr.* (Sorry, had to sneak in an MIT example, since I use this protocol *a lot*.) Text, short to medium length, ephemeral or persistent, many-to-many (but with personal classes, which let people initiate conversations in a one-to-many fashion), consumed via zephyr client. Effectively limited to members of a university only.
*   *Social question websites.* (e.g. Ask Reddit, Ask Hacker News, Stack Overflow.) Text, medium to long length, persistent, one-to-many (discussion itself can range from one-to-one to many-to-many), consumed via web browser. Updates are seen by subscribers of a given topic.
*   *Skype Voice/Video.* Audio/video, short length, ephemeral, one-to-one, done via the Skype client.

There are a few interesting features about this current landscape that I can observe, once we have distilled these protocols down to these levels.

*   Upstart companies attempting to create the next big social networks will frequently be innovating specifically in a few of these dimensions. Google Wave tried to innovate on medium, by introducing an extremely rich method of communication. Color is trying to innovate on medium and distribution.
*   Communication protocols that lie within the same taxonomy (for example, IM clients) have seen a proliferation of clients that can interoperate before all of them. This has not, however, generally been the case for cross-taxonomy jumps: have you seen a single client which integrates participation with forums, email, Facebook and instant messaging? And if you have, did it work well at all? The method of consumption for one type of social information does not necessarily work well for other types.
*   Similarly, it is not too difficult to bootstrap persistent communication protocols on other communication protocols. For example, a reply to a forum post is relatively persistent and not too much is lost if you send an email update whenever a new reply is found, and if you can’t rely on a user regularly checking your website for updates, this can be critical. However, sending instant messages to your email account makes no sense! There is something important here, which is that it does not make sense to mix ephemeral and persistent means of communication.
*   Early social networking tools focused on one-to-one and many-to-many interactions, since these closely model how we socialize in real life. However, the advent of Twitter and Facebook has demonstrated that normal people can use a one-to-many medium not just for publishing (as was the case for blogs) but for socialization. People did not pin up sheets of gossip on noticeboards, they disseminated it via one-on-one conversations. But this is exactly what the new Internet is all about. The new social Internet involves exploring between the lines of publishing and private communication. Understanding what this variation is, is in many ways the key to understanding how to use the new social medium correctly.

As an end-user, I’m interested in mechanisms of unifying all social interaction methods which have *the same consumption patterns*—mostly because I don’t have a long enough attention span to deal with so many different website I have to check. And no, this does *not* mean just providing a tabbed interface which lets you switch from one network to another. Email and RSS work reasonably well for me for the persistent methods, but I have always found there to be a big void when it comes the new one-to-many styles of ephemeral (yet persistent) communication. Part of the reason is that we’re still trying to figure out what an optimal way of approaching this is, even when only one communication protocol is involved. (Remember that Facebook’s news feed was much loathed when it was originally released—but this was the key feature that moved Facebook from being a social networking site to a social communication site.) The situation becomes much more complex when multiple, subtly incompatible modes of communication are thrown into the mix.

[Kyle Cordes](http://kylecordes.com/2010/better-social-media-client) writes about this topic from the perspective of a power user. I agree with him that it’s unclear whether or not there would actually be a market for such a unified social media client outside of power users. But it’s unclear to me if the open source community will step up to the bat and create such a client. I hope to be proven wrong.