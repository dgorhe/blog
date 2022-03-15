---
layout: post
title:  "Beyond Bitcoin: How to Implement Decentralized Societies"
date:   2022-01-07 09:10:00 -0800
categories: decentralization
---

## Preamble
I'm writing this out of self-interest as well as a desire to push technological progressivism. As [Balaji Srinivasan put it in 2013][purpose-of-technology]:

> We need to consciously build a parallel tech-driven decentralized media ecosystem, and we need it to become the first point of call for anyone seeking to learn about technology.

While I disagree with some of Balaji's [opinions on the physical manifestations of social networks][software-is-reorganizing-the-world], it's obvious that software is subsuming peoples' time and attention. Balaji has a particular idea that I want to focus on, that of [starting a new non-contiguous country][how-to-start-a-new-country]. He leaves the implementation details to the readers' imagination so in this post we'll expound upon this idea with a concrete implementation. To the chagrin of many tech evangelists, we cannot wholly extricate our physical existence to the cloud no matter how cheap the EC2 instances become. Nonetheless, we can exercise agency in how and where technology will take our time, resources, and agency.

Specifically we're going to examine how to keep record of the physical citizens of *any* polity, digital or physical. There are a plethora of fascinating topics to discuss for decentralized societies such as food supply, delineation of trade routes, addressing natural disasters, etc. I'm choosing this topic because it's by definition the entry point of every human into any society. Those other topics may be the focus of future blog posts.


## Background - Conceptualizing DAOs
> A blockchain is a growing list of records, called blocks, that are linked together using cryptography.

> A decentralized autonomous organization (DAO)... is an organization represented by rules encoded as a computer program that is transparent, controlled by the organization members and not influenced by a central government.

The title mentions the term DAO, but let's relate DAOs to existing terminology to form a better intuition. We can view DAOs as an anarchic polity or a direct democracy. DAOs can be considered anarchic because there is no centralized control and no individual actually has to participate in any DAO - the null is zero participation. However, whenever people do partcipate everyone, by design, has an equal say at the start. As such it can be viewed as a direct democracy. A more colloquial example to conceptualize DAOs would be any sort of online group that exerts some authority. A recent example would be influence of the [WallStreetBets subreddit][wallstreet-bets] on the price of GameStop stock in 2021. They were a loosely organized coalition that had some rules, albeit not transparent, encoded in the subreddit. Ultimately, the users decided how they wanted to exercise their stock trades and call options for GameStop, but the group itself had some concerted, measureable effect on the market. These analogies will always be inexact, but the point is to help triangulate the term DAO within our existing lexicon.


## Thesis
Ok so I've been rambling about *why* I'm writing this but *what* is this about? It's simply this idea:

> What if we had a system of governance where each birth and death was put on a blockchain and people could join DAOs of various scales based on the institutions in which they want to have a say?


## Why Should We Care?
We should care about this because many aspects of society require identification and there is often too much friction in the existing centralized infrastructure. Certain data should be anonymized wherever we deem necessary, but we shouldn't throw the baby out with the bath water. Complete anonymity of every citizen would make society untenable. Driving on roads, flying on a plane, starting a business all require us to prove that we are who we say we are - and that should continue to be the case. Government identification works fine, but there are flaws. A core issue is that governments want to create identification that is physically mass producible but difficult to forge. The more complicated and expensive your passport becomes, the less accessible it becomes to the populous (subsidized or not). We would like to have an opt-in system that addresses this core issue such that citizens would feel compelled to opt-in to the new system or at worst, both the old and new system. This is where blockchains come into the picture.

Let's say you're a new parent and your home town has a blockchain of birth records. Your child's birth would be stored on their immutably. You don't have to worry if the hospital closes or you lose the birth certificate, the record of your child always exists. This birth record would enable you as parents to automatically be given access to votes related to child safety, education, etc. At larger scales, your vote gets correspondingly diluted, but the consequences of your votes become more impactful. To extend the education analogy, parents might have a more diluted influence in the education policy of North America but the scope of influence would be beyond that of the home town DAO.

We could even extend this to global governance. Imagine if we could vote on climate change issues globally? Stupidity is high-entropy when things are left to an open choice. "What should we do about climate change?" is different than, "Should we do X or Y about climate change?". In the former, people proactively propose solutions whereas the latter is a false choice often promulgated by governments. Imagine policy debates that reinforce the minimization of posturing. Citizens could waste time posturing to appear to do something, or reach out to other factions and reach a consensus. A DAO based system would incentivize the latter but not necessarily prevent the former. Factions that are apathetic about climate change *would have to actively organize to convince other voters to abstain from voting*. Similarly, DAO members could spend their time convincing members to leave to increase the voting power of a single vote, but that costs time and energy. These are all downstream effects of things built on top of this birth record system, but could we actually put *all* births on a blockchain? There's a lot of babies born so this is a non-trivial question.


## Feasibility in a Proof-of-Work System
Let's talk about the technicalities of this. How can we determine if all births can be put on a blockchain? How do we get from birth records to DAOs and global governance?

### Block Design
Around [385,000 babies are born per day][babies-born-per-day]. If we assume there rate is constant this means approximately 2,674 babies are born every 10 minutes. I'm using 10 minutes because by default that's how often a block is written for [bitcoin][bitcoin]. Each block has a theoretical [4mb limit][bitcoin]. Let's imagine the following very basic example mocked up in TypeScript.

{% highlight typescript %}
const block = {
    id: uuid;
    name: string;
    parents: uuid[];
    custody: LinkedList<uuid>;
    lat: BigInt;
    lon: BigInt;
}
{% endhighlight %}

We'll discuss optional parameters in the following section.

### Block Size
Typically, a `uuid` is represented by 32 hexadecimal characters and 4 hyphens. Let's assume 4 bits per character so 144 bits total so far. For `name`, let's be liberal and use `UTF-32` which has a fixed length and takes 32 bits per character (how else are we going to store Elon's son's name, X Æ A-Xii). We'll arbitrarily limit names to 50 characters so 1600 bits. It's important to differentiate custody and parenthood since they can often be decoupled. `parents` only includes biological parents (2 uuids so 288 bits). With `custody` we have an issue. We want to accomodate as many entries as necessary but we can't accomodate an infinite number of custody changes. Putting an upper bound might be harmful to children if the terminal custodian is abusive or absent. We could use foster care data to determine the maximum number of custodial changes or the 99.9th percentile of custodial changes. For now, we'll arbitrarily limit this to 18 custodians which I base on having 2 new custodians every 2 years until the child is 18 years old. This amounts to 18 uuids so 2,592 bits. `BigInt` can store up to 64 bits so `lat` and `lon` together take up 128 bits. All together we have 144 + 288 + 2,592 + 1600 + 128 = 4,752 bits. This is the equivalent of 594 bytes. 594 bytes times 2,674 = 1,588,356 bytes = 1.588 mb. Okay, so we know there is no fundamental limitation to this approach even using something as intensive as proof-of-work as the basis for our blockchain.

We could add more required or optional items to each block since we have headroom. We might want more attributes so it's easier for a person to rediscover their id. Specifically with the `id` attribute, I believe the most sustainable way would be to create a hash based on the most variable parts of human DNA in addition to a corneal scan or fingerprint (to account for identical twins). A newborn's data would be collected, the hash is generated and if someone needs to rediscover their block, they can resequence their DNA and take a new corneal scan or fingerprint scan. Other redundant biological hashes would need to be developed for people with birth defects. 

As long as there is no consistent backlog of births, spikes in birth rate are fine (we just postpone writing them to a block for a few block durations). A proof-of-stake model could be even more amenable to this task although I'm less familiar with the inner workings of proof-of-stake systems.

### Caveats
I mentioned the custody caveat and biological hash, but there are some other caveats to keep in mind. Namely, we assumed birth rates are consistent and will remain consistent. We have a ton of headroom so I don't see this as a real issue. If anything the size of the block should increase to enable more features. If we wanted a more accurate determination, we could look at a histogram of births every 10 minutes across a wide variety of hospitals and areas. From this we could simply use the sum of largest values plus some buffer or some other liberal estimate. I wasn't able to find a dataset this granular, but if someone knows of such a dataset, please let me know. Hard forks would be another issue since it's quite likely people will want to change the design of blockchains as time goes on. We could also imagine some fields I listed being empty and thus optional. What if you don't know who the parents are? We could amend biological hashes such that your hash will always equal the hash of your parents and thus we could reconstruct a family tree. The ethical issues of that decision should be left to individual DAOs. Competing hash functions would likely arise based on peoples' privacy tolerance. There is also the issue that a child's inclusion in at least 1 DAO is entirely decided by their parents or initial custodians. I could go on and on, but I don't think any of these issues are a dealbreaker. The nature of a DAO based system would offer competing solutions and tradeoffs. DAOs are not a panacea.

## Transitioning to DAO Based Society
Beyond the technical challenges facing DAO creation, there is an obvious social component to this transition. What is non-technical core product value of these systems for new users? What incentive is there for an existing person to onboard to this new system? There are advantageous side-effects of creating a biological hash for older citizens. It's a good way to collect data for personalized medicine. Older people have more complicated ailments and subsidizing genome sequencing could help bolster a database (blockchain or otherwise) of genomes to better research and address these ailments. Moreover, when people die this would be an immutable record of their identity which loved ones could use to resolve legal disputes. Families with children on the way could opt-in to the system knowing that their parenthood and custody is being recorded immutably. Other authentication technologies would be built on top of this system which might end up displacing passports and identification cards for people of all ages. The case to opt-in is weakest for young adults with no families. The most important demographics to focus on would be new parents and immigrants (at least for legal immigration). The best case scenario is that we onboard a majority of new parents within 1 generation. Realistically we would not expect Facebook or Twitter levels of growth, nor should that be the goal. However, we should imbibe the platform lessons from Facebook and Twitter - deliver core product value and everything else will fall into place.


## Consequences
### Immutable Store of People
We have all these blocks, so what? Now we have an immutable store of people born. We could similarly create a blockchain for deaths based on death certificates. The difference between the set of birth ids and the set of ids in the death blockchain is the number of people alive. This forms the basis of token dissemination for the aforementioned organizations. A town could decide that anyone born at a certain group of hospitals is automatically added to the town's DAO. Citizenship would be optional but it could remain exactly as it is now. Being a part of a nation-state is a choice not a prerequisite. You could avoid joining any DAO but that would be a choice you have to make. 

### Built-in Redundancy
Blockchains are relatively safe from fraud by design. Similarly, natural disasters pose less risk to blockchains since they are distributed. Municipalities could also have redundant chains on the cloud to provide additional security. The cost of fabricating a block for a DAO would be too demanding, even for a cloud provider. 

### Built-in Diversity, Equity, and Inclusion
There is no information about one's race, sex, gender, etc. in the block. If your id exists and you are not dead, your vote counts. To be fair, a real issue would be if people are denied a birth block on the basis of these characteristics. What if Uighurs in China are denied person-hood from birth on the basis of their family's religion? Then again, keeping track of people in a separate database uses a lot of time and energy. At that point people have to wonder, what's in it for me? Unlike a lot of bitcoin maximalists and decentralization diehards, I do not think blockchains are the solution to these fundamental moral quandries. It's totally possible for a totalitarian state to suppress DAOs permanently. DAOs are simply organizations with different trade-offs, ones that I think are more amenable to a highly automated society but prone to human folly just like everything else.

## Voting
Leaving a DAO would increase the voting power of each share but not by much. Similarly, joining a DAO does not significantly dilute each member's voting power. A one token one vote method could be used but there are also other methods like [quadratic payments][quadratic-payments]. Entities could enact whatever voting rules they want but they would have to balance the risks/benefits of losing voters versus gaining voters for their cause. Perhaps smaller DAOs could have deals with larger DAOs to make changes but those changes are contingent upon having a minimum number of voters. Again, this *may* be linked to geography but it does not *have to* be linked to geography.


## Conclusion
Decentralization is more than just bitcoin. Aren't you so sick of hearing the same buzzwords and explanations over and over again? "Bitcoin is like digital gold", "It's a ledger that governments can't control" and on and on. We should demand some tangible core product value from blockchains beyond new toys for financial arbitrage. We should be asking for things like cryptocurrency based payroll so we can get paid on a daily or even hourly basis. We should ask for logistics to be a series of smart contracts where changes in warehouse capacity automatically triggers a change in purchase orders. For every dotcom era company we remember today there were dozens that were total nonsense and rightfully called out for being so. Bitcoin might Java Applets and JavaScript might be right under our noses.


## Addendum
I want to address probably the best set of criticisms against crypto-adjacent topics. There was a video by Folding Ideas titled Line Goes Up - The Problem With NFTs where he lambasts the idea of crypto currency. He cites the speculative nature of crypto as a continuation of the post 2008 financial crisis and the case is very compelling. Relevant to this posts, he unequivocally rejects the idea of crypto token based governance. 

> "... while the claim is that these machines will democratize the internet, the technical complexity they add, the new speciailized programming expertise that they require, concentrates a lot of power in the hands of people who can build the templates that in turn enable non-programmers to actually use it."

What I’m proposing here is different because I don’t think I'm proposing a template for DAOs themselves. I’m suggesting that digitizing identities in a robust, seamless way will allow variegated DAOs to emerge. Identification is also a very well understood interface between the analog and digital world so I think average people could wrap their heads around this aspect in particular. Perhaps I’m buying into the decadent ideas put forth by cryto-evangelists but I don’t think I am. Please correct me if I’m wrong. I don't want to get too far into this criticism, but I can dive deeper if anyone wants.

[bitcoin]: https://github.com/bitcoin/bitcoin
[babies-born-per-day]: https://www.theworldcounts.com/stories/How-Many-Babies-Are-Born-Each-Day
[quadratic-payments]: https://vitalik.ca/general/2019/12/07/quadratic.html
[wallstreet-bets]: https://reddit.com/r/wallstreetbets

<!-- Balaji -->
[purpose-of-technology]: https://balajis.com/the-purpose-of-technology/
[software-is-reorganizing-the-world]: https://www.wired.com/2013/11/software-is-reorganizing-the-world-and-cloud-formations-could-lead-to-physical-nations/
[how-to-start-a-new-country]: https://1729.com/how-to-start-a-new-country