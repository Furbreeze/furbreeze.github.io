---
layout: post
title:  "Timescape Version 0.1 alpha"
---

Time management has always been a struggle for me. I have a million hobbies and never enough time to complete any of them. But as a person who strives for continuous improvement, I decided to take matters into my own hands.

## Inspirations

### Inspiration 1, A New Years Resolution

![Resolution](/images/resolution.gif){: .center}

I play a lot of video games, probably to an unhealthy amount. On a whim, as a resolution for 2025 New Year's, I challenged myself to take the year off of video games. This is something I hadn't done since I was 3 years old. Rightfully so, many of those close to me said there was "no way" I'd finish it.

So far, I've managed to keep to my resolution, and this made for an amazing year for *productivity*. I hit my first bug bounties, earned a CVE, took on and finished a hardware project, built a cyber shed to do my work out of, built an outdoor smart birdcage to house my cockatiels year round, most of which you can read about here on this blog, maybe all at some point. It turns out not spending so much time on video games gives you time to do other things in life. Other things with lasting fulfillment rather than temporary satisfaction. While great sounding on paper, something was still missing. I still yearned to play video games, specifically ONE video game. 

### Inspiration 2, Enter Old School Runescape (OSRS)

![OSRS](/images/osrs_dance.gif){: .center}

For those of you who haven't heard of it, [OSRS](https://oldschool.runescape.com/) is a medieval fantasy game centered around fighting monsters, getting loot, and leveling up. This game holds a special place in my heart since the day it was introduced to me back in grade school. More than even the nostalgia, I still have fun playing the game. 

One of the finer points of OSRS is "the grind". The game is all about gaining experience, to do the thing, to gain experience, to do thing, etc. The gameplay loop for many is extremely addictive, because in my opinion, it short-circuits your reward loops. It does this in the form of:

1. Constant real time experience tracking
2. Loot drops with known rarities
3. Challenging in game bosses and activities
4. Quests and Experience Requirements

The game has knowingly (or unknowingly) made a perfect system for setting and achieving goals, complete with metrics for tracking progress. Many would say this is the real "magic" and point of OSRS and why the game can be played in so many ways. I found this loop incredibly addictive and keeping me up many nights just trying to do "that one more thing". Something about seeing number go up makes big things seem achievable in the moment.

![Number](/images/number.gif){: .center}

Despite all that I've accomplished this year, it still just hasn't been as fulfilling as sitting and playing OSRS. I really wanted a way to make things that are challenging and exhausting, as addictive and fun as playing OSRS.

### Inspiration 3, The Effective Engineer

![Eng](/images/eff_eng.png){: .center}

I started reading the [Effective Engineer](https://www.amazon.com/Effective-Engineer-Engineering-Disproportionate-Meaningful/dp/0996128107/) this year in order to focus on some self improvement in my day job. I would HIGHLY recommend it to any engineer. Immediately after cracking the book and reading the first chapter, I had learned a useful nugget of information (paraphrased): Spending time learning and improving processes is a multiplicative increase to productivity and effectiveness. This is something that really stuck out to me and was something I firmly believe in, but often find it hard to prioritize.

This struggle to prioritize stems from a lack of feedback when learning. How do I know I'm learning something? How can I measure if this is more useful than just doing things? Am I wasting my time? This torrent of questions that berates me every time I try to carve out time specifically for learning led to an idea.

## The Idea

Wouldn't it be nice if the goal setting achievement based loop of OSRS could be applied to life? I already know how addictive this is to me, so if I could harness it, it should mean I'll enjoy *anything*, right?

![Timescape](/images/timescape.gif){: .center}

Above is the version 0.1 of what I'm calling "Timescape". It's a device for tracking time while doing "skills" and offers real time experience and leveling to accompany that. Skills are user defined and are stored locally on the device, with the ability to sync remotely to a git repository, in my case, this blog. See [TimeScape](/timescape.html) to see my most up-to-date experience levels.

### The Implementation

The experience curve for any skill follows the [OSRS XP Curve](https://oldschool.tools/experience-table). I then mapped level 99 experience to 10,000 hours of time in a skill, which roughly maps to the following formula. 

```python
def xp_to_seconds(xp):
    # Calculate days: (total_xp * 0.000767) / 24
    days = (xp * 0.000767) / 24
    
    # Convert to seconds
    seconds = days * 24 * 60 * 60
    
    return round(seconds)
```

Each skill starts at level 3, the same as they do in OSRS. I think this will be exciting for me since the early levels will be easy to get and thus make it more motivating to start and keep a habit of training a skill. The nice thing about the XP curve is that if you've managed to go from one level to the next, you've roughly been through 90% of what it takes to get to the next level. So each level, you just have to go a little further than you did last time.

Finally, experience is stored locally on the device, with the ability to sync remotely. I'm currently syncing the data to this blog, allowing me to easily host and share my data as well as giving me a platform to build out metrics and better displays.

### The Device

Frankly, this is all still a work in progress and covering all of this could be it's own separate post. To summarize I used the following:

1. Raspberry Pi Pico 2W
1. SSD1306 for the screen
1. Momentary switches for buttons
1. Fusion 360 for modeling
1. Micropython for programming lang
1. Wires and prayers to glue it all together
1. This blog for storing and showing the data

As an aside, this was my first time using Fusion 360, or really any 3d modeling software. A quick moment of silence for the fallen brethren that led to the current print.

![Fallen](/images/fallen.jpg){: .center}

All of this details for this can be found in my [Github Repo](https://github.com/Furbreeze/timescape). I will update the repo as well as this blog as any major updates are made.

## The Result

I already love this thing. Immediately after I added real time xp tracking to the main screen I found myself excited to watch the number go up. Learning a skill is type 2 fun, meaning the end-goal is what's fun, not necessarily the "doing" to get there. This device gives me a way to turn type 2 fun into type 1 fun. While I may not specifically enjoy strength training, I'm able to enjoy watching the number go up. It allows me to have measurable and enjoyable checkpoints along the way to the ultimate goal at the end.

## The Updates

I'm definitely not done iterating on this device, but after dragging my feet on a few things, I thought it was best to just start using it. This way I can prioritize updates I actually need:
1. More detailed web frontend for displaying graphs + historical data
1. Battery powered and portable
1. Bigger screen
1. Smaller form factor with custom PCB
1. Rounded corners so it's not a weapon
1. Increased dopamine through buzzing, sounds, animations 
1. Increased dopamine through loot, drops, tiers, quests
1. Whatever else comes up through use