---
title: "Krümels Bug Bounty Adventures - The Beginning"
date: 2025-03-23
description: "Krümels Bug Bounty Adventures - The Beginning"
tags: ["bug-bounty-adventures"]
---

## Introduction and goal
Hi, I'm Krümel,  
and this blog post series: **Krümels Bug Bounty Adventures** will be about my adventures in the bug bounty space.

The goal of this journey is simple: **Get 100k in rewards**  
... to join the 100k club known from the likes of NahamSec (https://x.com/NahamSec) and Rhynorater (https://x.com/Rhynorater).

This goal is pretty far fetched (for now) and as I don't know what is going to await me, I'll not define it in more detail. I'll set myself quarterly and weekly SMART goals to get there step by step. Here and on X (https://x.com/kruemelxyz) you can follow my progress.

I'm not starting out completely clueless, I have professional experience in web development and also in the webappsec industry. Besides that I'm a long time lurker in the infosec and bug bounty realm on X, Reddit and various Discord servers. Still, black box web application penetration testing is something I did not spend a huge amount of time doing, therefor, especially at the beginning, I'll spend some time studying various bug classes, techniques, concepts and tooling.

With that said, good luck to us!

## Progress - Week 1 (CW 12)
Following the principle of "hack to learn and not learn to hack" I wanted to get into active hunting as soon as possible. 
My approach is to learn with the challenges I encounter starting right away testing on a (hardened production) target. In my experience the theory is often quiet easy, but practically testing on a target brings often many small, sometimes big, challenges to overcome with it.

For the beginning I chose two public targets, one with a very broad scope and one with a single main application. For now I will stick to the target with the single main application, as it feels much less intimidating because of the narrow scope. I chose this target because I did not experience problems creating multiple accounts to test for authorization issues. 

For the beginning I will test for two bug classes only, "Reflected and stored XSS" and "Authorization issues". I know about other bug classes too, but I want to go first really deep into only a few topics, create my methodology for it and then move on to the next one. As you will read soon, that's not that easy when there is so much to learn about.

### Breakdown working hours
Overall: 36h 45min.  

Setup - Hunting: 13h  
Learning - XSS: 4h  
Hunting - AuthZ: 4h  
Blog - Writing: 3h 30min.  
Learning - AuthZ: 2h 30min.  
Learning - Recon: 2h  
Setup - Administration: 2h  
Hunting - XSS: 1h 30min.

### XSS
To warm up I skimmed through Zseanos bug bounty guide and did most of the XSS Portswigger Academy Labs. Then I started to analyze the input/output sanitization/encoding of the target. The target is mostly based on API calls returning JSON and a React front end, which handles the output encoding. I did not find any issues for now, but tested only a very small subset of the parameters to understand the application behaviour. I alse recognized I will have to take some time to learn much more about payload obfuscation, this will help not only for finding XSS issues but will be helpful testing for many other bugclasses.

#### XSS ressources
- Portswigger Academy Labs: https://portswigger.net/web-security/cross-site-scripting
- Zseanos bug bounty guide: https://www.bugbountyhunter.com/zseano/

### Authorization
I listened to the "BBRE" podcast with Douglas Day and couldn't agree more with most of his arguments. It is very hard to write tests for all kind of authorization scenarios for an application based on deep role concepts. Additionally is the behaviour of an application context based, which makes it hard for automated testing. This sounds pretty juice to me, when it comes to hunting for this bug class. 

On our single main app target it is possible to assign four different roles out of the box (no further customizing) to the user. I created one user for each role and already had a first small challenge to overcome, to conveniently login and gather cookies for all users without a big hassle. As the application created already some cookies for the unauthenticated session based on who knows what to send a legitimate login request, writing a simple login script was not possible. Then I tried some Firefox plugins to manage cookies, I finally decided for "Cookie Editor" as this was one of the only plugins which could also extract "http-only" flagged cookies (this cookies can't be read by JavaScript) and additionally offered a very convenient "copy cookie header as string" functionality. The Firefox "Multi-Account Containers" plugin completed the setup to easy manage multiple logins and cookie access in a single browser windows.

I set up and tried two Burp plugins for testing authorization scenarios: "AuthMatrix" and "AuthAnalyzer". In comparsion "AuthAnalyzer" is faster (subjective), and comes with some very handy functionalities like value replacement (e.g.: CSRF token) and semi automatic session handling which I missed using "AuthMatrix".

As for the testing itself, I gathered the cookies for all users and created sessions in AuthAnalyzer for them and also an unauthenticated session. Then logged in as an administrator I went through a few features of the admin center (more to come next week) and reviewed EVERY request. Only after a few hours of testing I had two, what I thought, legit findings.

#### Finding 1
One endpoint responded with detailed information about the instance and was accessible unauthenticated (I found over 1200 instances online where this endpoint was accessible). The response revealed information about the features bought by the instance owner, the subscription plan and also their connected X and Facebook profiles. I could imagine that this information is maybe needed for some third party integration and the issue issua is a false positive, but as I was not sure, I reported it. The report got closed as "Informative".

#### Finding 2
The second finding was a endpoint which responded with detailed information about all instance administrators (Name, Phone number, E-Mail, Last logon, ...) and was only accessible for the administrator and the second lowest privileged user. IMHO this was already a sign that it's an actual issue, as the user which had less privileges then the administrator, but more privileges then the user who could access this endpoint, was not authorized. Also I could imagine that this endpoint was hard to write automatic tests for, as it was hidden in multi-step workflow and had preconditions to get there. This finding would have been fixed pretty fast in past penetration tests I executed professionally. Sadly also this report got closed as "Informative".

#### Authorization ressources
- Portswigger Academy Labs: https://portswigger.net/web-security/access-control
- AuthMatrix Burp Plugin: https://github.com/SecurityInnovation/AuthMatrix
- Auth-Analyzer Burp Plugin: https://github.com/PortSwigger/auth-analyzer
- AuthMatrix Video: https://www.youtube.com/watch?v=pMXTmXUsEL8
- BBRE podcast: https://www.youtube.com/watch?v=-YzAwKRMXK0
- D Day on X: https://x.com/ArchAngelDDay
- Cookie Editor Firefox Plugin: https://cookie-editor.com/
- Firefox Container Plugin: https://support.mozilla.org/en-US/kb/containers

### Other topics
As already mentioned, I had a hard time sticking to my two chosen bug classes and hunt for them. On the way I read something here and there, remembered bookmarked tweets or articles and did set up and tested multiple other tools and techniques. Obviously that is the beauty in bug bounty, you can learn and do what interests you, but it's also a challenge for beginners as there is such a vast amount of things you can do an learn which leads to potential success. In the future when I actually utilize this tools and newly gathered knowledge I will write in more detail about it. But here is a bullet-point list on what I spent also time (actually most of it) this week. All tools referenced are the ones I liked and will add to my arsenal.

#### 4xx bypasses
- https://github.com/lobuhi/byp4xx

#### Paramter discovery
- https://github.com/s0md3v/Arjun
- https://github.com/PortSwigger/param-miner

#### Note taking and time tracking
- https://obsidian.md/
- https://github.com/projecthamster/hamster

#### Others
- https://github.com/xnl-h4ck3r/waymore
- https://github.com/lc/gau
- https://github.com/xnl-h4ck3r/GAP-Burp-Extension
- https://github.com/GerbenJavado/LinkFinder
- https://github.com/xnl-h4ck3r/xnLinkFinder
- https://github.com/trufflesecurity/trufflehog

## Generic learnings
- Dont get distracted so easily and stick to what I'm working on
- If I see something potential useful, bookmark it and analyze it later on (I got lost 2 days setting up and trying out tools I somewhen will need in the future, but not now)
- Only impactful findings are rewarded, sometimes not even those (subjective opinion and a little bit of frustration ^^)
- Writing a blog takes time 