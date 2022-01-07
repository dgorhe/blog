---
layout: post
title:  "DAOs All The Way Down"
date:   2022-01-07 09:10:00 -0800
categories: decentralization
---

## Preamble
I am writing this out of self-interest as well as a desire to push technological progressivism. As [Balaji Srinivasan put it in 2013][purpose-of-technology]:

> We need to consciously build a parallel tech-driven decentralized media ecosystem, and we need it to become the first point of call for anyone seeking to learn about technology.

While I disagree with some of Balaji's [opinions on the physical manifestations of social networks][software-is-reorganizing-the-world], it is obvious that software is subsuming peoples' time and attention. An idea from Balaji more relevant to this discussion is the idea of [starting a new non-contiguous country][how-to-start-a-new-country]. However, he leaves the implementation deatils to the readers' imagination. To the chagrin of many tech evangelists, we cannot wholly extricate our physical existence to the cloud no matter how cheap the EC2 instances become. 

Instead we're going to address one small portion of implementing decentralized societies. Namely we'll look at how to keep record of the physical citizens of *any* polity, cloud-native or otherwise. There are other issues for decentralized societies such as food supply, delineation of trade routes, addressing natural disasters, etc. I'm choosing this topic because it is by definition the entry point of every human into any society.


## Background - Conceptualizing DAOs
> A blockchain is a growing list of records, called blocks, that are linked together using cryptography.

> A decentralized autonomous organization (DAO)... is an organization represented by rules encoded as a computer program that is transparent, controlled by the organization members and not influenced by a central government.

Let's try to relate DAOs to existing terminology so we have a better footing of what we're discussing. It is possible to view DAOs as an anarchic polity or a direct democracy. It can be viewed as anarchic because there is no centralized control and no individual actually has to participate in any DAO - the null is zero participation. However, whenever people do partcipate everyone, by design, has an equal say at the start. As such it can be viewed as a direct democracy. A more colloquial example to conceptualize DAOs would be any sort of online group that exerts some authority. A recent example would be influence of the [WallStreetBets subreddit][wallstreet-bets] on the price of GameStop stock in 2021. They were a loosely organized coalition that had some rules, albeit not transparent, encoded in the subreddit. These analogies will always be inexact, but the point is to help triangulate the term DAO within our existing lexicon.


## Thesis
Ok so I've been rambling about *why* I'm writing this but *what* is this about? It is simply this idea:

> What if we had a system of governance where each birth and death was put on a blockchain and people could join DAOs of various scales based on the institutions in which they want to have a say?


## Why Should We Care?
We should care about this because many aspects of society require some identification and the centralized infrastructure of current systems has lots of friction. Certain data should be anonymized wherever we deem necessary, but we shouldn't throw the baby out with the bathwater. Complete anonymity of every citizen would make society untenable. Driving on roads, flying on a plane, starting a business all require us to prove that we are who we say we are. I hope it's not a controversial opinion to say that that's a good thing? Government identification works fine, but there are flaws. Physical identification is fungible, verification is often limited by the speed of interpersonal communication, identification can be stolen and difficult to replace, and it costs lots of money to keep these systems operational. We would like to have an opt-in system that addresses these major shortcomings such that new parents would feel compelled to opt-in to the new system or at worst, both the old and new system.

Let's say you're a new parent and your home town has a DAO. The birth record for your child could automatically give you access to school and child safety related votes for your home town. At larger scales, your vote gets correspondingly diluted, but the consequences of your votes become more impactful. You might have less of a say in how North America deals with shale oil reserves, but there's no way your vote in Wichita, Kansas could ever do anything about shale oil reserves in the first place. 

We could even extend this to global governance. Imagine if we could vote on climate change issues as a globe? Stupidity is high-entropy when things are left to an open choice. "What should we do about climate change?" is different than, "Should we do X or Y about climate change?". In the former, people proactively propose solutions whereas the latter is a false choice often promulgated by governments. Imagine policy debates that reinforce the minimization of posturing. Factions that are apathetic about climate change *would have to actively organize to convince other voters to abstain from voting*. Similarly, DAO members could spend their time convincing members to leave to increase the voting power of a single vote, but that costs time and energy.


## Technicalities
Let's talk about the technicalities of this. Could we even put all births on a blockchain? How can we determine this? How exactly does this relate to DAOs and voting?


### Feasibility in a Proof-of-Work System
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

Typically, a `uuid` is represented by 32 hexadecimal characters and 4 hyphens. Let's assume 4 bits per character so 144 bits total so far. For `name`, let's be liberal and use `UTF-32` which has a fixed length and takes 32 bits per character (how else are we going to store Elon's son's name, X Æ A-Xii). We'll arbitrarily limit names to 50 characters so 1600 bits. It is important to differentiate custody and parenthood since they can often be decoupled. `parents` only includes biological parents (2 uuids so 288 bits). With `custody` we have an issue. We want to accomodate as many entries as necessary but we can't accomodate an infinite number of custody changes. Putting an upper bound might be harmful to children if the terminal custodian is abusive or absent. We could use foster care data to determine the maximum number of custodial changes or the 99.9th percentile of custodial changes. For now, we'll arbitrarily limit this to 18 custodians which I base on having 2 new custodians every 2 years until the child is 18 years old. This amounts to 18 uuids so 2,592 bits. `BigInt` can store up to 64 bits so `lat` and `lon` together take up 128 bits. All together we have 144 + 288 + 2,592 + 1600 + 128 = 4,752 bits. This is the equivalent of 594 bytes. 594 bytes times 2,674 = 1,588,356 bytes = 1.588 mb. Okay, so we know there is no fundamental limitation to this approach even using something as intensive as proof-of-work as the basis for our blockchain.

We could add more required or optional items to each block since we have some headroom. We might want more attributes so it's easier for a person to rediscover their id. Specifically with the `id` attribute, I believe the most sustainable way would be to create a hash based on the most variable parts of human DNA in addition to a corneal scan or fingerprint (to account for identical twins). A newborn's data would be collected, the hash is generated and if someone's identity gets deleted, they can resequence their DNA, take a new corneal scan or fingerprint and reinstate their identification.

As long as there is no consistent backlog of births, spikes in birth rate are fine (we just postpone writing them to a block for a few block durations). A proof-of-stake model could be even more amenable to this task although I'm less familiar with the inner workings of proof-of-stake systems.

## Transitioning to DAO Based Society
Beyond the technical challenges facing DAO creation, there is an obvious social component to this transition. What is non-technical core product value of these systems for new users? What incentive is there for an existing person to onboard to this new system? There are advantageous side-effects of creating a biological hash for older citizens. It's a good way to collect data for personalized medicine. Older people have more complicated ailments and subsidizing genome sequencing could help bolster a database (blockchain or otherwise) of genomes to better research and address these ailments. Moreover, when people die this would be an immutable record of their identity which loved ones could use to resolve legal disputes. Families with children on the way could opt-in to the system knowing that their parenthood and custody is being recorded immutably. Other authentication technologies would be built on top of this system which might end up displacing passports and identification cards for people of all ages. The case to opt-in is weakest for young adults with no families. The most important area would be to onboard new parents. Ideally we could have a majority of people onboarded within 1 generation. Realistically we would not expect Facebook or Twitter levels of growth, nor should that be the goal. However, we should learn the platform lessons from Facebook and Twitter - deliver core product value and everything else will fall into place.


## Consequences
### Immutable Store of People
We have all these blocks, so what? Now we have an immutable store of people born. We could similarly create a blockchain for deaths based on death certificates. The difference between the set of birth ids and the set of ids in the death blockchain is the number of people alive. This forms the basis of token dissemination for the aforementioned organizations. A town could decide that anyone born at a certain group of hospitals is automatically added to the town's DAO. Citizenship would be optional but it could remain exactly as it is now. Being a part of a nation-state is a choice not a prerequisite. You could avoid joining any DAO but that would be a choice you have to make. 

### Built-in Redundancy
Blockchains are relatively safe from fraud by design. Similarly, natural disasters pose less risk to blockchains since they are distributed. Municipalities could also have redundant chains on the cloud to provide additional security. The cost of fabricating a block for a DAO would be too demanding, even for a cloud provider. 

### Built-in Diversity, Equity, and Inclusion
There is no information about one's race, sex, gender, etc. in the block. If your id exists and you are not dead, your vote counts. To be fair, a real issue would be if people are denied a birth block on the basis of these characteristics. What if Uighurs in China are denied person-hood from birth on the basis of their family's religion? Then again, keeping track of people in a separate database uses a lot of time and energy. At that point people have to wonder, what's in it for me? Unlike a lot of bitcoin maximalists and decentralization diehards, I do not think blockchains are the solution to these fundamental moral quandries. It is totally possible for a totalitarian state to suppress DAOs permanently. DAOs are simply organizations with different trade-offs, ones that I think are more amenable to a highly automated society but prone to human folly just like everything else.

## Voting
Leaving a DAO would increase the voting power of each share but not by much. Similarly, joining a DAO does not significantly dilute each member's voting power. A one token one vote method could be used but there are also other methods like [quadratic payments][quadratic-payments]. Entities could enact whatever voting rules they want but they would have to balance the risks/benefits of losing voters versus gaining voters for their cause. Perhaps smaller DAOs could have deals with larger DAOs to make changes but those changes are contingent upon having a minimum number of voters. Again, this *may* be linked to geography but it does not *have to* be linked to geography.


## Conclusion
Decentralization is more than just bitcoin. Aren't you so sick of hearing the same buzzwords and explanations over and over again? "Bitcoin is like digital gold", "It's a ledger that governments can't control" and on and on. We should demand some tangible core product value from blockchains beyond new toys for financial arbitrage. We should be asking for things like cryptocurrency based payroll so we can get paid on a daily or even hourly basis. We should ask for logistics to be a series of smart contracts where changes in warehouse capacity automatically triggers a change in purchase orders. For every dotcom era company we remember today there were dozens that were total nonsense and rightfully called out for being so. Bitcoin might Java Applets and JavaScript might be right under our noses.


[bitcoin]: https://github.com/bitcoin/bitcoin
[babies-born-per-day]: https://www.theworldcounts.com/stories/How-Many-Babies-Are-Born-Each-Day
[quadratic-payments]: https://vitalik.ca/general/2019/12/07/quadratic.html
[wallstreet-bets]: https://reddit.com/r/wallstreetbets

<!-- Balaji -->
[purpose-of-technology]: https://balajis.com/the-purpose-of-technology/
[software-is-reorganizing-the-world]: https://www.wired.com/2013/11/software-is-reorganizing-the-world-and-cloud-formations-could-lead-to-physical-nations/
[how-to-start-a-new-country]: https://1729.com/how-to-start-a-new-country