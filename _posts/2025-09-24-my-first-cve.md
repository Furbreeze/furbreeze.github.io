---
layout: post
title:  "How I found my first CVE (CVE-2025-8447)"
---

I'd always been interested in bug bounty and I REALLY wanted a CVE for nerd cred. However, I always struggled with confidence and sticking with it due to loads of impostor syndrome.

Earlier in the year I found a bug in Windmill that [ended up flying under the radar](https://github.com/windmill-labs/windmill/commit/3c6d1029d19fc822c955681594e500ff0613fd89) and not getting a CVE. The excitement of finding that bug and not getting a CVE drove me to really try and find one. 

So I discussed with some of my colleagues and they pointed out that GitHub was a good program to hack on. Supposedly good response times, good payouts, and a product I like using, so it was a no-brainer for me.

## The Official Description Vulnerability

The bug was assigned [CVE-2025-8447](https://www.cve.org/cverecord?id=CVE-2025-8447). 

>An improper access control vulnerability was identified in GitHub Enterprise Server that allowed users with access to any repository to retrieve limited code content from another repository by creating a diff between the repositories. This vulnerability affected all versions of GitHub Enterprise Server prior to 3.18, and was fixed in versions 3.14.17, 3.15.12, 3.16.8 and 3.17.5.

## The Journey

So after some digging and discussing, it was brought to my attention that GitHub Enterprise Server is *sort of* open source. Apparently if you download the latest version and run a ruby script that you can find out there in the aether, run it against the obfuscated code in the server download, and bob's your uncle! Full source access to GitHub Enterprise Server.

After getting that set up and running on my environment, I started poking at the server while having source code to back up my testing. I spent a few weeks on email verification and found an interesting issue I can't discuss further, but it was ultimately deemed not exploitable. Bummer. Luckily, this time wasn't wasted because I became very familiar with the codebase and the layout of functionality.

So I finally gave up on email verification and decided to just peek at whatever I could. While scrolling through functionality in the repositories, I noticed something odd: A `...` in the request to compare two commits. My brain immediately said "That's not a normal web convention, that's custom. I wonder what they're doing there?".

So after diving in the source and playing around a bit, I saw that I could hit a different code path by using `..` instead of `...`. What's more, I was able to determine the full format of the payload, listed below:

`https://<enterprise_url>/<your_username>/<your_repo>/compare/file-list?range=main..<other_user>:<their_private_repo>:<four_characters_of_a_commit_hash>`

It was interesting to see that I could control the source and destination repository, ie: I could compare a repository I owned to any other repository, even one I didn't own. Surely this couldn't be true, right? Well I fired up burp and it really was that easy :) Just a plain old IDOR sitting there looking right at me.

![GitHub IDOR vulnerability](/images/gh_idor.png)

Well this is exciting, I have a bug! What did I need to exploit it?
1. An account on a GHE server
2. The repository name and owner of another repository
3. A commit hash to diff to and leak from

Since #1 is a given, I'll go through the way to obtain #2 and #3. 

To obtain a repository name and owner, there are few ways to leak this, but you can get orgs, usernames etc. just by browsing the app. In order to get the actual repository name, you can initiate a transfer request between two users. If the user you're transferring to has a repository of the same name, the system will throw an error indicating they already have repository of that name.

Okay, now onto the more challenging part, how do you leak a commit hash? This is a SHA-256 hash, so brute force guessing it is not going to work. Or is it...? 

After poking at the endpoint some more, I noticed that the endpoint only actually requires the first 4 characters of the commit hash. Armed with that knowledge, we've drastically reduced the brute force space to 65536 (16^4) guesses, and since there are no brute force protections in a standard deployment... EZ PZ. It should also be noted that this implies there's only ONE commit on the repo. The search space drastically reduces further the more commits that have been performed in the repo.

Tying all of this together, we finally arrive at full arbitrary read of any repo in the GHE instance, only requiring a valid user account. For the record, I don't know why this only worked on GHE and not on github.com. I have to assume the codebases are a little different and this happened to be a feature where it was.

## Beyond the Official Description
> retrieve limited code content from another repository

This was untrue by following the following algorithm:

1. Perform diff
1. Update attacker repo to contain leaked contents
1. Goto 1

This was communicated to the team, however it did not affect the final severity.

## Impact

The vulnerability could allow unauthorized users to access code they shouldn't have permission to view, potentially exposing sensitive information or intellectual property stored in private repositories.

## Remediation

GitHub fixed this vulnerability in the following versions:
- 3.14.17
- 3.15.12
- 3.16.8
- 3.17.5

All users of GitHub Enterprise Server should update to these versions or later to protect against this vulnerability.

## Timeline

- Issue Reported May 3rd 2025
- Issue Triaged July 22nd 2025
- Issue Awarded $10k bounty August 26th 2025
- Issue Marked as Resolved August 26th 2025
- Issue Disclosed Sept 23rd 2025

## Lessons Learned

I heard a quote from a [bug bounty podcast](https://www.youtube.com/watch?v=yxc2jVKE-jo), "Big Cheese, Big Holes", which really stuck with me. I think I had success here because I was willing to look at a big target like GitHub, something plenty of people hack on, and approach it with an open mind, positivity, and curiosity. In the future, I think keeping that positivity will be just as important as any skilling up I can possibly do. Especially when going after more big targets.

Finally, Bug bounty requires PATIENCE, an attribute I am admittedly lacking in. It was my first real bug submitted and waiting for everything to shake out was BRUTAL.

## Next steps

Good news, there's a sequel :) I will disclose more when I can.
