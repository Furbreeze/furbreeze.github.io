---
layout: post
title:  "Vibecoding my way to a crit on Github"
---

All good stories have a sequel. Most of the time, that's not a good thing but in this case I'm happy with it. So, after a few months of waiting on the payout from the [last bug]({% post_url 2025-09-24-my-first-cve %}), I decided to start looking at Github again.

## The Bug

Dependency confusion on github.com, leading to remote code execution across multiple services.

## The Journey

I mentioned in my last blog, that I'm new to bug bounty. So I still haven't hammered out what my workflow looks like, or really how to manage my time and stay motivated and constantly hacking. In between this bug and the previous, I spent some time doing penetration testing consulting, working on a few personal projects, and ultimately just fretting while I waited in the dark for my previous bug to shake out.

Well it finally did shake out, which meant it was time to move on. And I had some time between consulting gigs, so I decided it was time to go ahead and get to hacking. Now that I'd decided to start, I had to pick a goal, which I settled on "Let's check for dependency confusion on gitlab, since they're open source."

So I grabbed [Gitlab source](https://gitlab.com/rluna-gitlab/gitlab-ce) and checked out their `package.json` files. From there it was as easy as extracting them with a little jq magic and throwing them in Burp intruder to check for 404s. As you'd expect, this had been picked clean. Darn :(

> dependency confusion on gitlab, since they're open source.

Wait a minute... Technically Github Enterprise is "open source". And it probably hasn't been picked clean because that's a lesser known fact... Let's do it! So I followed the same methodology, scraped out the packages and sent them off to intruder to find...

![Nothing](/images/nothing.gif){: .center}

**Nothing**. Just more 404s. As I was already about to give up, I had a thought: "Is dependency confusion a thing in Ruby?". After a bit of googling, I found an [article](https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610) telling me that it was! This felt super promising since it's a lesser known language + a lesser known code base. The more I can find myself in these lesser known places, the more successful I'll be. Let's do this for a third time!

![Again](/images/again.gif){: .center}

For a third time, following the same methodology, I extracted a list of packages from the `Gemfile` and `Gemfile.lock` files that I could find. And after using Burp Intruder to send them all, I found 126 hits for 404s! Now my blood is pumping and I need to operationalize, how can I get this done? I was roughly going to:

1. Develop a ruby gem payload that executes code and uses DNS exfiltration (an idea I stole from [the previous article](https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610)) to get my data out.
1. Configure my [interactsh](https://github.com/projectdiscovery/interactsh) listener to receive wildcard DNS requests.
1. Write a script to decode the incoming payloads in order to extract the information from the domain name.
1. Write a script to take a list of ruby gems, generate a template based on the payload
1. Write a script to build all of the gems
1. Write a script to push all of the gems to rubygems.org
1. Sit back and wait for profit.


**WOW**. That was a pain to type, let alone actually create. I want my results *fast* so I can quickly iterate on the idea. If only there were a way to do this...

![VibeCoding](/images/ai.png){: .center}

Enter our newest friend in 2025, in this case, ✨ [Claude.ai](https://claude.ai/) ✨. Since I already knew what I wanted to do, and I wanted to do it quickly, this was the way to get it done. I was able to accomplish the following tasks with ease:

1. ✅
1. 🛑
1. ✅
1. ✅
1. ✅
1. ✅
1. 🛑

Now it wasn't all AI sunshine and AI rainbows and there were a few hiccups I encountered along the way, but 5/7 ain't bad. Briefly describing, some of the hiccups I had:

1. Domain name [length restrictions](https://superuser.com/questions/1843857/what-is-the-maximum-length-of-a-domain-name) in the DNS exfiltration script
1. Getting the payload to execute on "require" in ruby
1. Finding the stupid flag for getting interactsh to accept wildcard domains (`n=0` which is **not** listed in their docs from what I can tell)

So I finally had written everything up, tested the payloads locally and successfully decoded the incoming information, there was only one thing left to do. Press the money printer button and pray. I did this at around 6 AM on Monday (labor day) and finally fell asleep until 2PM. 

![Christmas](/images/christmas.gif){: .center}

When I woke up, I immediately ran out to my office like it was Christmas morning, hoping I had some callbacks. And sure enough I did :) At that point, nearly 1200 of them. With some pretty interesting and suspicious names mind you. So after a long night and not enough sleep to follow, I rushed to get my bug submitted. Mission accomplished, I'd achieved a critical on github.

Since it was labor day, the issue was not handled until the following day (Tuesday). The team asked me to neuter the malicious packages I'd registered, as well as to not pivot any further into their network. My callbacks finally stopped after ~2000 callbacks and within 24 hours of submitting the bug. JACKPOT baby. They're taking this seriously.

## Impact

I had code execution across the following services:

```json
[
    {
        "username": "github", 
        "hostname": "buildkitsandbox", 
        "ip": "<redacted>", 
        "filepath": "<redacted>"
    },
    {
        "username": "central-cf6fcb55f-j87b4", 
        "hostname": "<redacted>", 
        "ip": "<redacted>", 
        "filepath": "<redacted>"
    },
    {
        "username": "codespaces-52e12d", 
        "hostname": "vscode", 
        "ip": "<redacted>", 
        "filepath": "<redacted>"
    },
    {
        "username": "github", 
        "hostname": "central-cf6fcb55f-j87b4", 
        "ip": "<redacted>", 
        "filepath": "<redacted>"
    }
]
```

The execution was coming from one specific package, but I had 125 more I was asked to remove before I had a chance to get callbacks from them.

## Timeline

1. Sept 1st 1AM: Had idea to start hunting.
1. Sept 1st 2AM: Found 404s on package names in Github Enterprise.
1. Sept 1st 6AM: Executed payload and began waiting.
1. Sept 1st 2PM: Got callbacks, submitted bug.
1. Sept 2nd 2PM: Asked to take down malicious gems.
1. Sept 3rd 11AM: Callbacks stopped.
1. Oct 28th 7PM: Issue closed, bounty of $20k rewarded.

## My feelings about the bounty

I want to remain professional about how this went and only state the facts to start. Later I will express my feelings but try to remain uninflammatory.

### The facts

1. I had code execution on many services that looked like the above. Those included a build process, dev code workspaces, and something with the host `central-cf6fcb55f-j87b4`.
1. There were many more services than those listed, but those encompassed the "four types" of services.
1. The bounty they paid me was for a critical, but the absolute minimum range.
1. I stopped executing my payloads and neutered when they requested.
1. I did not pivot further per their request.
1. It took them 58 days to resolve this.
1. They parked many more packages on rubygems than I had found.
1. They would not disclose this issue on Hackerone, but I'm allowed to blog about it.
1. When asked to explain the bounty and impact, I did not receive a response.
1. Their bounty page states they pay [$30k for criticals](https://bounty.github.com/#:~:text=Critical%20severity%20issues%20present%20a%20direct), despite what is says on hackerone.
1. They awarded me 200 swag points.

### My opinions
1. I am disappointed in the payout.
1. Given the level of impact, number of services, I believe this was more than the minimum severity critical.
1. Given that they fixed it so quickly, they thought it was impactful.
1. Given the number of total packages parked, this was widespread impact and something they were previously unprotected from.
1. I could have pivoted further and increased impact if I was not on a bug bounty program, being as delicate as possible.
1. The inability to disclose the issue, leads me to believe they don't want it to be known, and thus it's impactful.
1. I would have preferred either public disclosure or a bounty more reflective of the impact over swag points.
1. The communication on this ticket was canned and unhelpful, on the few instances I received any.

## Lessons Learned

I learned a lot from this one. Specifically around package managers, ruby gems, and a deeper understanding of how to use `interactsh`. Additionally, about how useful AI can be for rapid prototyping. It was awesome to take an idea to an operationalized and automated exploit in 5 hours.

Finally, I learned it's time to move on. I've spent some time on this program and found some good bugs, but ultimately I feel underappreciated. It's not so much the payouts as the lack of communication. Maybe I'll come back in the future, but for now, it's just not for me.

## Next steps

There's a potential sequel to this as well :) If it ever shakes out I will be sure to publish something about it. Also, if enough people reach out, I'll share "my" scripts for autopwning the ruby gems.