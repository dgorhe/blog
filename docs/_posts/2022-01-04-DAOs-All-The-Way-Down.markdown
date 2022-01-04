---
layout: post
title:  "DAOs All The Way Down"
date:   2022-01-04 04:44:00 -0500
categories: jekyll update
---

## Preamble
Most of the blog post ideas I've had have been too expansive for one neat post. However, I've stumbled upon an idea as I was thinking about governance. First, we should clarify some terminology because they are often overused:

> A blockchain is a growing list of records, called blocks, that are linked together using cryptography.

> A decentralized autonomous organization (DAO)... is an organization represented by rules encoded as a computer program that is transparent, controlled by the organization members and not influenced by a central government.

I believe anarchy is underrated as a possible future for governance (or lack thereof). Understandably, this is hard to envision. In particular, there is a common equivocation of anarchy with total chaos. Most people imagine war-torn or countries or those that lack a rule of law. Another common conception is feudalism which isn't far off but the difference is that there won't necessarily be geographically defined fiefdoms. There are obvious flaws with anarchy such as the transition to a stateless society from a society in which nation-states have a monopoly on violence. I think we have a particularly hard time imagining how the minutiae of life will change under each of these regimes. All of these pontifications are great but what would life look like for an average person? A lot would probably stay the same, but there would obviously be key differences.

Ok so I've been rambling about *why* I'm writing this but *what* is this about? It is simply this idea:

> What if we had a system of governance where each birth was put on a blockchain and people could join DAOs of various scales based on the institutions in which they want to have a say? 

For example, you could be in the DAO of your hometown. This seems rather obivious and an advantage is that you have a relatively large say in your town's governance. We could imagine this happening at the equivalent of a county, state, region, or nation-state level where your vote gets correspondingly diluted, but the consequences of your votes become more impactful. You might have less of a say in how North America deals with shale oil reserves, but there's no way your vote in Wichita, Kansas could ever do anything about shale oil reserves in the first place. We could even extend this to global governance. Imagine if we could vote on climate change issues as a globe and allocate capital via these tokens? Stupidity is high-entropy so noise is relatively easy to distinguish when things are left to an open choice. More specifically in this example, there's a lot of different ways to have cognitive dissonance about climate change. 


## Technical Specficiations
Let's talk a bit about the technicalities of this. Could we even put all births on a blockchain? How can we determine this? How exactly does this relate to DAOs, tokens, and voting?


### Feasibility in a Proof-of-Work
Around [385,000 babies born per day][babies-born-per-day]. If we assume there rate is roughly constant this means approximately 2,674 babies are born every 10 minutes. I'm using 10 minutes because by default that's how often a block is written for bitcoin (cite specific bitcoin code which says this). Each block has a theoretical 4mb limit. Let's imagine the following very basic example mocked up in TypeScript.

{% highlight typescript %}
const block = {
    id: uuid;
    name: string;
    lat: BigInt;
    lon: BigInt;
}
{% endhighlight %}

Typically, a uuid is represented by 32 hexadecimal characters and 4 hyphens. Let's assume 4 bits per character so 144 bits total so far. For the name in the string, let's be liberal and use `UTF-32` which has a fixed length and takes 32 bits per character. We'll arbitrarily limit names to 50 characters so 1600 bits for the name. BigInt can store up to 64 bits so lat and lon together take up 128 bits. All together we have 144 + 1600 + 128 = 1,872 bits. This is the equivalent of 234 bytes. 234 bytes times 2,674 = 625,716 bytes = 0.625 mb. Okay, so we know there is no fundamental limitation to this approach even using something as intensive as proof-of-work as the basis for our blockchain. We could add more required or optional items to each block since we have some headroom. As long as there is no consistent backlog of births, spikes in birth rate are fine (we just postpone writing them to a block for a few block durations). A proof-of-stake model could be even more amenable to this task.


[babies-born-per-day]: https://www.theworldcounts.com/stories/How-Many-Babies-Are-Born-Each-Day