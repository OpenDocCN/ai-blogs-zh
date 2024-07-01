<!--yml
category: 未分类
date: 2024-07-01 18:18:06
-->

# The HTML purification manifesto : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/10/the-html-purification-manifesto/](http://blog.ezyang.com/2010/10/the-html-purification-manifesto/)

I recently sent Greg Weber an email about his [xss-sanitize](http://github.com/gregwebs/haskell-xss-sanitize) package, cautioning about his reuse of the pandoc sanitization algorithm for his own package. He responded (with good justification) that a mere caution was not very constructive! So here is my response, the “HTML purification manifesto,” which HTML Purifier follows and which I think is a prerequisite for any industrial grade HTML sanitization library. I will admit it’s a tough manifesto to follow, and I’ll talk about when you can get away with not following it to the line.

*The manifesto.*

Use semantic data structures.

Whitelist and validate *everything*.

Output only what *all* browsers understand.

* * *

*Use semantic data structures.* Many filters attempt to never build up a DOM or tokenized representation of their HTML during the processing stage for performance reasons. It turns out this makes securing your filter notoriously difficult. Consider the [XSS cheatsheet](http://ha.ckers.org/xss.html): see how many of the vulnerabilities involve non-well-formed HTML. If you require HTML to be converted into a DOM, and then serialize it out again, you have a pretty good guarantee that the result will be well-formed, and you eliminate all of those vulnerabilities.

This must be applied to all sublanguages in HTML, and not just HTML itself. You also have to be careful that your serializer produces standards compliant output. For example, a [vulnerability in HTML Purifier itself](http://htmlpurifier.org/security/2008/http-protocol-removal) was caused by a lax adherence to standards. As it states:

> HTML Purifier's fix also percent-encodes any other reserved character in each segment of a URI. This was actually a previously identified section of relaxed standards compliance, and strictly enforcing the rules eliminated the vulnerability.

Semantic data structures are also useful for implementing extra features—for example extracting content from the document or converting all URIs into absolute form—and makes validation and manipulation tremendously easier. It will also be critical for the next step.

* * *

*Whitelist and validate everything.* Whitelisting is a well accepted practice, and I will not harp on it here. However, there are subtleties in its application. In order to understand the first part of the manifesto, you need to understand what *everything* means. At first glance, the obvious things that you might have to whitelist are elements and attributes. But if you decide to allow the `href` attribute, you need to make sure you whitelist URI schemes too—three things to whitelist. And heaven forbid you decide to allow `style` attributes!

It’s far better to take a different approach to whitelisting: every time you encounter an attribute, figure out what its sensible values are, and validate it to a whitelist. Think you don’t need to validate that `height`? Consider the imagecrash attack, which can bluescreen an unwitting Windows user simply by setting the width and height of an image to 999999\. Complicated attribute? It may have further structure, so expand it into the appropriate semantic data structure! (The most obvious candidates are URIs and CSS strings.) You don’t necessarily *have* to create these structures in memory (indeed, an optimized implementation would try to avoid it), but it certainly makes coding the algorithms a lot easier. Also, it is much easier to manipulate the resulting semantic structure than it is to manipulate a bunch of strings.

This is a rather large problem to tackle, because there are so many elements and attributes in HTML and so many sublanguages embedded within those. However, you can dramatically simplify matters by creating a domain specific language to help you declaratively specify what the semantics are: though I didn’t know it at the time, this was the approach that HTML Purifier took with its `HTMLModule` system.

* * *

*Output only what all browsers understand.* I used to believe that standards compliance was enough to make sure all browsers understood your syntax (the primary target of most exploits.) Sometimes, however, browsers just don’t parse by the standards at all. Usually, the wild and woolly fields of ambiguous behavior lie outside the dictates of standards, but sometimes the standard says X should happen and Internet Explorer [goes and does Y](http://htmlpurifier.org/security/2010/css-quoting). It is tempting to throw your hands up in the air and just deal with the vulnerabilities as they come in.

But because this is an incompatibility with the standards, there are many web developers have discovered that what should work doesn’t! For CSS, even the most obscure parsing bug in Internet Explorer has been tripped over by some web developer and been documented in a wiki or encoded into a CSS hack. This knowledge, while seemingly not related to the questions of security, is critical for someone writing an HTML filter! It tells you, in particular, what the *subset* of standards-compliant HTML that is understood by all browsers is.

* * *

*A more practical note.* Whenever I see a new HTML filter pop up on the web, I apply these three litmus tests to the source code (I don’t even bother trying the demo):

1.  Does it use an HTML parser or attempt to do a bunch of string transformations?
2.  How much does it whitelist? Just elements and attributes? What contents of attributes does it treat?
3.  Does the filter appear to have accreted those battle scars that come from dealing with the real world?

I apply the second and third criterion less strongly depending on what the filter claims to support: for example, a filter with no support for CSS needs not worry about its attendant difficulties. In the extreme case, if you’re writing a filter for just `<b>` and `<i>`, you can get away with not following any of these recommendations and be fine. But I suspect users always want more features, and you will inevitably start adding new elements and attributes. So I also judge whether or not the source seems to have thought about extending itself in these directions.

*A more linguistic note.* I use the word “purification” rather than “sanitization”, because sanitization implies that the pathogens have merely been rendered inert, whereas purification implies that they have actually been removed altogether. I think that the latter philosophy is a much safer stance!

*Conclusion.* Back in the day, I managed to write highly inflammatory comments about other HTML filters in an overzealous attempt to promote my own. I avoid doing direct comparisons these days; instead, I hope this manifesto is helpful to people interested in writing or improving their own filters.