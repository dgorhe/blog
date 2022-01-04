---
layout: post
title:  "DAOs All The Way Down"
date:   2022-01-04 04:44:00 -0500
categories: decentralization
---

## Preamble
> A blockchain is a growing list of records, called blocks, that are linked together using cryptography.

> A decentralized autonomous organization (DAO)... is an organization represented by rules encoded as a computer program that is transparent, controlled by the organization members and not influenced by a central government.

Anarchy is underrated as a possible future for governance but it is understandably hard to envision. In particular, anarchy and total chaos are often equivocated. Most people imagine war-torn or countries when picturing anarchic states (as oxymoronic as that phrase sounds). Another common conception is feudalism which isn't far off but the difference is that there won't necessarily be geographically defined fiefdoms. 

There are obvious flaws with anarchy such as the inherent risk in the transition from a nation-state with a monopoly on violence to a stateless society. Preventing violence against children is also a serious issue with anarchy since there is no state to represent the interests of children before the children can reasonably represent themselves. More broadly speaking, we have a hard time imagining changes to the minutiae of our lives. All these pontifications are great but what would life look like for an average person? A lot would probably stay the same, but there would obviously be key differences.

Ok so I've been rambling about *why* I'm writing this but *what* is this about? It is simply this idea:

> What if we had a system of governance where each birth and death was put on a blockchain and people could join DAOs of various scales based on the institutions in which they want to have a say?

For example, your home town could be a DAO and you would have a large say in your town's governance. At larger scales, your vote gets correspondingly diluted, but the consequences of your votes become more impactful. You might have less of a say in how North America deals with shale oil reserves, but there's no way your vote in Wichita, Kansas could ever do anything about shale oil reserves in the first place. 

We could even extend this to global governance. Imagine if we could vote on climate change issues as a globe and allocate resources via these tokens? Stupidity is high-entropy when things are left to an open choice. "What should we do about climate change?" is different than, "Should we do X or Y about climate change?". In the former, people proactively propose a solution whereas the latter could be a false choice. Those factions that don't want to do anything about climate change *would have to actively organize to convince other token holders to abstain from voting*. Similarly, DAO members could spend their time getting members to leave to increase the voting power of the DAO, but that costs time and energy. 


## Technicalities
Let's talk about the technicalities of this. Could we even put all births on a blockchain? How can we determine this? How exactly does this relate to DAOs, tokens, and voting?


### Feasibility in a Proof-of-Work System
Around [385,000 babies are born per day][babies-born-per-day]. If we assume there rate is constant this means approximately 2,674 babies are born every 10 minutes. I'm using 10 minutes because by default that's how often a block is written for [bitcoin][bitcoin]. Each block has a theoretical [4mb limit][bitcoin]. Let's imagine the following very basic example mocked up in TypeScript.

{% highlight typescript %}
const block = {
    id: uuid;
    name: string;
    lat: BigInt;
    lon: BigInt;
}
{% endhighlight %}

Typically, a `uuid` is represented by 32 hexadecimal characters and 4 hyphens. Let's assume 4 bits per character so 144 bits total so far. For `name`, let's be liberal and use `UTF-32` which has a fixed length and takes 32 bits per character (how else are we going to store Elon's son's name, X Æ A-Xii). We'll arbitrarily limit names to 50 characters so 1600 bits. `BigInt` can store up to 64 bits so `lat` and `lon` together take up 128 bits. All together we have 144 + 1600 + 128 = 1,872 bits. This is the equivalent of 234 bytes. 234 bytes times 2,674 = 625,716 bytes = 0.625 mb. Okay, so we know there is no fundamental limitation to this approach even using something as intensive as proof-of-work as the basis for our blockchain. 

We could add more required or optional items to each block since we have some headroom. We might want more attributes so it's easier for a person to rediscover their id. Some kind of biometric hash would probably be useful. The main `id` field should probably not biometrically linked for privacy but that's also an option. 

As long as there is no consistent backlog of births, spikes in birth rate are fine (we just postpone writing them to a block for a few block durations). A proof-of-stake model could be even more amenable to this task although I'm less familiar with the inner workings of proof-of-stake systems. If someone has an idea on this, I'd gladly amend this post and credit you.


## Consequences
### Immutable Store of People
We have all these blocks, so what? Now we have an immutable store of people born. We could similarly create a blockchain for deaths based on death certificates. The difference between the set of birth ids and the set of ids in the death blockchain is the number of people alive. This forms the basis of token dissemination for the aforementioned organizations. A town could decide that anyone born at a certain group of hospitals is automatically added to the town's DAO. Citizenship would be optional but it could remain exactly as it is now. Being a part of a nation-state is a choice not a prerequisite. You could avoid joining any DAO but that would be a choice you have to make. 

### Built-in Redundancy
Blockchains are relatively safe from fraud by design. Similarly, natural disasters pose less risk to blockchains since they are distributed. Municipalities could also have redundant chains on the cloud to provide additional security. The cost of fabricating a block for a DAO would be too demanding, even for a cloud provider. 

### Built-in Diversity, Equity, and Inclusion
There is no information about one's race, sex, gender, etc. in the block. If your id exists and you are not dead, your vote counts. To be fair, a real issue would be if people are denied a birth block on the basis of these characteristics. What if Uighurs in China are denied person-hood from birth on the basis of their family's religion? Then again, keeping track of people in a separate database uses a lot of time and energy. At that point people have to wonder, what's in it for me?

## Voting
Leaving a DAO would increase the voting power of each share but not by much. Similarly, joining a DAO does not significantly dilute each member's voting power. A one token one vote method could be used but there are also other methods like [quadratic payments][quadratic-payments]. Entities could enact whatever voting rules they want but they would have to balance the risks/benefits of losing voters versus gaining voters for their cause. Perhaps smaller DAOs could have deals with larger DAOs to make changes but those changes are contingent upon having a minimum number of voters. Again, this *may* be linked to geography but it does not *have to* be linked to geography.


## Conclusion
I've kept everything at a high-level just to keep the conversation terse. Decentralization is more than just bitcoin. I'm so sick of hearing the same buzzwords and explanations over and over again. "Bitcoin is like digital gold", "It's a ledger that governments can't control" and on and on. I want real products and services. I want somebody to make a cryptocurrency based payroll so I can get paid on a daily or even hourly basis. I want logistics to be a series of smart contracts where changes in warehouse capacity automatically triggers a change in purchase orders. Like, my god people, be a little creative.


[bitcoin]: https://github.com/bitcoin/bitcoin
[babies-born-per-day]: https://www.theworldcounts.com/stories/How-Many-Babies-Are-Born-Each-Day
[quadratic-payments]: https://vitalik.ca/general/2019/12/07/quadratic.html