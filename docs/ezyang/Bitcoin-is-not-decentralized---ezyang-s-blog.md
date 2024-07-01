<!--yml
category: 未分类
date: 2024-07-01 18:17:46
-->

# Bitcoin is not decentralized : ezyang’s blog

> 来源：[http://blog.ezyang.com/2011/06/bitcoin-is-not-decentralized/](http://blog.ezyang.com/2011/06/bitcoin-is-not-decentralized/)

Bitcoin was designed by Satoshi Nakamoto, and the primary client is developed by a bunch of folks at [bitcoin.org](http://bitcoin.org/). Do you care who these people are? In theory, you shouldn’t: all they do is develop an open source client for an open source protocol. Anyone else can develop their own client (and some people have) and no one, save the agreement of everyone in the Bitcoin network, can change the protocol. This is because the Bitcoin network is designed to be decentralized.

If you believe in the long term viability of Bitcoin, you should care who these people are. While Bitcoin itself is decentralized, the transition from Bitcoin to a new currency cannot be. This transition is guaranteed by the fact that all cryptosystems eventually become obsolete. Who will decide how this new currency is structured? Likely the original creators of Bitcoin, and if you have significant holdings in Bitcoin, you should care who these people are.

The following essay will flesh out this argument more carefully, as follows:

1.  Cryptosystems, including cryptographic hashes, must be used with the understanding that they must eventually be replaced. One might argue that “If Bitcoin’s cryptography is broken, the rest of the financial industry is in trouble too”—we explain why this is irrelevant for Bitcoin. We also see why it’s reasonable to expect Bitcoin, if it becomes a serious currency, to stick around a long enough timespan for this obsolescence to occur.
2.  There are several rough transition plans circulating the Bitcoin community. We describe the most common decentralized and the most common centralized variant, and explain why the decentralized variant cannot work in a non-disruptive manner, appealing both to economics and existing markets which have similar properties.
3.  We more carefully examine the implications of these decentralized and centralized transitions, and assess the risk of the transition, in comparison to the other risks facing Bitcoin as a fledgling currency. We suggest that, while the transition of Bitcoin is not a central concern, the idea of naive decentralization is a myth that needs to be dispelled.

I’ve divided the essay into sections so that readers who are interested in specific sections of the argument. Feel free to skip around.

### The cryptosystem time bomb

“All cryptosystems eventually become obsolete.” Compared to currency, cryptographic hashes are a relatively recent invention, dating only as far back as the 1970s. MD5 was invented in 1991, and it only took about a decade and a half to [thoroughly break it](http://valerieaurora.org/hash.html). For computer programmers, the shifting landscape of cryptography is a given, and systems are designed with this in mind. Consider, for example, SSL certificates, which are used to secure many transactions on the Internet, including financial transactions. These need to be renewed every few years, and as new certificates are issued, their level of protection can be increased, to use newer ciphers or longer key sizes. Most current uses of cryptography follow this pattern: the ciphers and keys can be replaced with relative ease.

Bitcoin, however, is special. The way it achieves decentralization is by embedding all of its relevant technical details in the protocol. Among these is [the hashing algorithm, SHA-256](https://en.bitcoin.it/wiki/Protocol_specification). It is literally impossible to “change” the hashing algorithm in Bitcoin; any change would constitute a change in the protocol, and thus result in a completely new currency. Don’t believe anyone who tells you otherwise. The argument “If Bitcoin’s cryptography is broken, the rest of the financial industry is in trouble too” is irrelevant, because other financial institutions have central control of the ciphers they use and can easily change them: Bitcoin cannot. And due to the possibility of weaknesses in SHA-1 spilling into the SHA-2 family (among which SHA-256 is a member), a competition for SHA-3 is already being held.

Will Bitcoin last long enough for fraudulent transactions to become practical? It may not (after all, there are many other possible problems with the currency that may kill it off before it ever gets to this stage.) However, if it does become established, you can expect it to be a hardy little bastard. Currencies stick around for a long time.

### Decentralized and centralized currency transition

The Bitcoin community has realized the fact that a transition will become necessary, and though the general sense is that of, “We’ll figure it out when we get there,” there have been some vague proposals floated around. At the risk of constructing strawmen, I would like to now present my perception of the two most popularly voiced plans. First, the decentralized plan:

> Because cryptosystems don’t break overnight, once the concern about SHA-256 becomes sufficiently high we will create a new version of Bitcoin that uses a stronger cryptographic hash. We will then let the market decide an exchange rate between these two currencies, and let people move from one to the other.

This is decentralized because anyone can propose a new currency: the market will decide which one will win out in the end. It also cannot possibly work in a nondisruptive manner, for the simple reason that anyone seeking to exchange the old Bitcoin for the new one will have to find a willing buyer, and at some point, hyperinflation will ensure that there are no willing buyers. All existing Bitcoins will then be worthless.

At this point, we’ll take a short detour into the [mooncake black market](http://marketplace.publicradio.org/display/web/2010/09/21/a-black-market-for-mooncakes-in-china/), a fascinating “currency” in China that has many similar properties to an obsolescing Bitcoin. The premise behind this market is that, while giving cash bribes are illegal, giving moon cake vouchers are not. Thus, someone looking to bribe someone can simply “gift” them a moon cake voucher, which is then sold on the black market to be converted back into cash.

Those partaking in the moon cake black market must be careful, because once the Autumn Festival arrives, all of these vouchers must be exchanged for moon cakes or become worthless. As the date arrives, you see an increasingly frenzied game of hot potato for the increasingly devalued vouchers. The losers? They end up with lots of moon cakes. There is of course one critical difference, which is that the losers of the Bitcoin game are left with nothing at all.

Is this a transition? Yes. Is it disruptive? Definitely yes. It is certainly not what you want a currency you’re using for every day transactions to be doing. Of course, this may be acceptable risk for some industries, and we’ll analyze this more in the last section.

Here is the centralized plan:

> Once the concern for the hashing algorithm is high enough, we will create a new Bitcoin protocol. This protocol will not only include a new hashing algorithm, but also be based off of the value of the old Bitcoin economy at some date: at that point, all newer transactions are invalid in the new Bitcoin scheme, and that snapshot is used to determine the amount of Bitcoins everyone has.

There is a variant, which deals with the case when active attacks are being carried out against the hashing algorithm before they have managed to switch, which involves marking specific block chains as known good, and zeroing out suspected fraudulent transactions.

Is this plan really centralized? Yes: someone needs to design the new protocol, to convince all the clients to buy into it, and to uniformly switch over to the new economy when the day arrives. The fragmentation of the Bitcoin economy would be extremely disruptive and not in the best interests of any of the main players. Any other changes to the Bitcoin protocol (and at this point, there probably would be many proposals) could have massive implications for the Bitcoin economy.

### Implications and risk

Here, we assess the question, “Do I really care?” In the short term, no. Bitcoin has [many, many weaknesses](http://www.quora.com/Is-the-cryptocurrency-Bitcoin-a-good-idea?srid=pxt) that it will be tested against. Though I personally hope it will succeed (it is certainly a grand experiment that has never been carried out before), my assessment is that its chances are not good. Worrying excessively about the transition is not a good use of time.

However, this does not mean that it is not an important fact to remember. The future of Bitcoin depends on those who will design its successor. If you are investing substantially in Bitcoin, you should at the very least be thinking about who has the keys to the next kingdom. A more immediate issue are the implications of a [Bitcoin client monoculture](http://timothyblee.com/2011/04/19/bitcoins-collusion-problem) (one could push out an update that tweaks the protocol for nefarious purposes). Those using Bitcoin should diversify their clients as soon as possible. You should be extremely skeptical of updates which give other people the ability to flip your client from one version of the protocol to another. Preserve the immutability of the protocol as much as possible, for without it, Bitcoin is not decentralized at all.

Thanks to Nelson Elhage, Kevin Riggle, Shae Erisson and Russell O’Connor for reading and commenting on drafts of this essay.

*Update.* Off-topic comments will be ruthlessly moderated. You have been warned.

*Update two.* One possible third succession plan that has surfaced over discussion at Hacker News and Reddit is the decentralized bootstrapped currency. Essentially, multiple currencies compete for buy-in and adoption, but unlike the case of two completely separate currencies separated only by an exchange rate, these currencies are somehow pegged to the old Bitcoin currency (perhaps they reject all Bitcoin transactions after some date, or they require some destructive operation in order to convert an old Bitcoin into a new one—the latter may have security vulnerabilities.) I have not analyzed the economic situation in such a case, and I encourage someone else to take it up. My hunch is that it will still be disruptive; perhaps even more so, due to the artificial pegging of the currency.