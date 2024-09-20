---
title: Sketchware, The History and How I Got Into it
date: 2024-09-20 17:40:12 +0700
categories: Journals
tags: [sketchware, story, journal]
---

Hi all, Iyxan here.

It has been a while since I was ever active on online communities. And starting today, I’m happy to share that I had partially graduated (I’ll explain it later) from my boarding school.

Which means, I won’t be living under a rock anymore, and will interact with you guys and my friends more frequently like I usually do back in the days.

> Some small notes
>
> Psst, in this blog post, I’m trying my absolute best to provide a better reading experience by experimenting with different vocabularies and expressions to make it sound more natural and appealing. It may sound less iyxan-y compared to my previous articles, that’s because it has been LITERALLY 2 YEARS.
>
> I hope this would make my english writing better. Initially, the reason this blogpost was created was really meant for me to get better at writing english so yeah.
>
> Ok, go on.


## Some Life Updates

Okay, let’s get back to the “partial graduation” term that I used, what does that even mean?

The boarding school I attend is probably the most unique private school I’ve ever seen. It’s a school where students are laser-focused on computer subjects, live in a remote (a very green 🌲) place, with the basis of religion and character development.

It categorised itself as a Vocational High School, which—in my country—tends to do something called as “internships”.

Conventional Public Vocational High Schools tend to have a week long, a month long, or at most 3 months long of internships. And they tend to have their teachers look for internships.

At our school, it’s taken to the extreme—as with other things they do. We look for our own internships, and we are given a whole year for us to find one. They even encouraged us to find actual jobs rather than internships whenever possible.

And I’m in that one year period, starting this academic year. After that, I will officially graduate (yay).

Okay, getting back to the article. In this article, I’d like to reflect, and probably explain the details behind the scenes of what I did back in the days, given that it’s been 2 years, and I have matured a lot ever since I attended my boarding school.

I’ll share some lessons-learned moments, and explain pieces of history behind those projects on my perspective. But for now, I’ll be writing some history!

## When it all began: Sketchware

> *Sketchware: Create your own apps!*

For those who haven’t heard of, Sketchware is an android app that enabled people to develop android apps within the confines of their own android phone. Yes, you literally can build an android app within an android phone, isn’t that crazy?

Most people would consider Sketchware as a toy app or some sort, knowing that the app is targeted for kids. But boy, were they wrong.

We never realise that there are literal hundreds of millions (or even billions! perhaps) of people in this world with little to no access to computers. Knowing that the development of an android application requires not just a computer, but a capable computer with minimum requirements, Sketchware is a literal godsend.

According to a research done and published in the website whatsthebigdata.com: https://whatsthebigdata.com/smartphone-stats/, the number of people in the world that has access to a smartphone is approximately a whopping 86% of the population, which is roughly 6.84 billion people.

Now, since I couldn’t find any reliable data sources about the ratio between smartphone owners that has a computer in possession or not. It is arguable that among those smartphone users, there exists a majority of people that does not own a computer or have access to it.

Let’s say, 50% of smartphone users has little to no access to computers. That’s still 3.42 billion people left with no possible ways to develop an android application, because they have little access to computers. Heck, if even some of them do, I’m 100% certain that most of their computers aren’t even capable of running Android Studio for a whole hour without a bluescreen popping up.

Just imagine, the potential of those billions of people in technology. Who in the parallel universe will know that their names may be hung in the same wall as the names of Bill Gates, Linus Torvalds, Steve Jobs, Ken Thompson, Dennis Ritchie, or any other people that has greatly influenced in the field of IT. But they couldn’t, they don’t have a chance, and if they do, it’s too slim for them to slip through.

That’s where Sketchware came in to shine some light upon those people that has an inner drive to study technology. And I was one of those people!

### Iyxan23 joins the story

I had been accustomed to this Sketchware app since 2017. It was fun, yet life-changing at the same time.

On the day I used it for the first time to create an app, I instantly fell in love with it.

The inner curiosity within me screams millions of chemicals of dopamine like a wildfire. The possibilities of android apps you could develop within Sketchware is beyond imaginable (well, for me as a kid back then).

Suddenly, my brain spawned hundreds of app ideas each day. It was up until to the point where I always come up with a new app idea before I could even finish an app. I was too crazy as a kid at the time. At the end of the day, my Sketchware app was filled with a pile of abandoned projects that I probably will never touch anymore in the future.

It’s like what we Gen-Z-s call iPad kids today, I was like that back then, but with Sketchware! Sounds kinda silly doesn’t it?

To sum it all up, Sketchware made me the person I am today. And not just me, but also the millions of kids from different backgrounds and countries around the world!

It shaped my future, and I’m sure it shaped others too.

I can’t stress more to give out real and gigantic kudos to the original Sketchware developers. They should be aware that because of their creation, the lives of millions of kids that were otherwise limited and uncertain, had been enabled to discover their passion in programming, undoubtedly reshaping not just their future, but also the future of technology itself!

The more people getting aboard the programming bandwagon, the more opportunities and innovations that could flourish in the technology space!

Genuinely, from the bottom of my heart, thank you so much!

### The Fall of Sketchware

Sketchware was rising in popularity, the devs were happy, the community is thriving, and its local project store is constantly flooded with more projects from the community. Sketchware has reached its peak.

But as we all know, good things must always come to an end. And for Sketchware, the end is nearer than what we originally thought.

The downfall of Sketchware started with a sprinkle of controversy, that eventually became larger and larger over time. As more people and more people get involved with the case, the actual story got a bit shrouded, as multiple perspectives mix together to form a jumble of mess.

> I actually tried writing multiple perspective of this controversy, with hopes of you, the readers, to be able to make your own judgement about what actually happened, considering how complicated the situation is now.
But apparently after contacting the true Sketchware OG AdityaKapal362 and asking specifically about these topics, I finally understood the real reason behind all of this mess.
>

Contrary to popular beliefs, the TRUE reason of the downfall of Sketchware is caused by **ASD blocks.**

“Add source directly”, or the commonly-known nickname “ASD Block” (aptly named from the block’s text) is a block that allows users to insert java code directly into a Sketchware project.

Every android apps are essentially just JVM bytecode (that further gets converted into dalvik) that you get from compiling a java program (or any other JVM-compilable languages such as Kotlin or Scala). And the process is no different on Sketchware, they utilize an entire compiler toolchain on their app, isn’t that crazy?

Sketchware builds upon this architecture to develop a block-based code abstraction that essentially transpiles into java code. And as such with any abstraction, we lose detail from the lower level abstractions, being traded-off in order to make programming easier. The ASD Block was then developed to address this big hole, allowing anybody to write any java code they wanted, effectively bypassing restrictions imposed by the block-based code model.

But ASD Blocks were accessible only to select few in the community. It was revolutionary, but they need to keep this “cure” from anybody to ensure a level playing field. But why a “level-playing field”, you might ask? Because let’s see what happened at the time when ASDs were first publicized by Sketchware.

People were amazed by the endless possibilities of ASD Blocks, and those that are able to use it began to use it a lot. But the other blocks-only side does not, they avoided using ASD Blocks as it was adding too much complexity to their projects, and that they probably have no idea how to program in Java.

It was all going awesome for the community, until a decision by the Sketchware team made the blocks-only community upset.

The Sketchware team awarded an ASD-filled project as “editor-choice” in their project store.

And apparently some people got mad at that, since they believe projects that are awarded as “editor-choice” in the Sketchware project store should not use ASD, and only be a pure blocks-only project.

Lengthy clashes happen at the Sketchware Slack community. The Sketchware community was divided into two: Pro-ASD and Anti-ASD. Each throwing their arguments at each other, “You just envy people that can use ASD blocks”, “Sketchware should be kept simple, and never turn to be complicated”.

Until the Sketchware devs stepped in to end it all with a decision: they removed ASD blocks. They were doing that under the impression that they wanted to keep Sketchware “back to its original track”, which is to **code only with blocks.**

Can you guess what happened next? A lot more people got mad because of it.

Not only did they trigger the Pro-ASD people, but the casual users that had been trying out this new ASD block all along, until they realize the team removed it in the next update. It was a bit ridiculous.

This was around the 2017/2018 era of Sketchware, and I was one of those casual people trying out ASD blocks. At the time, I felt somewhat surprised to the fact that the team added and removed ASD blocks in a short time span. I have used it quite extensively across my project codebase, but after the update, I couldn’t use it anymore. It was quite a bummer.

Considering how many people started to adopt ASD Blocks, and to not be able to use it in the next update made a lot of people frustrated.

### The Rise of Sketchware Mods

From excitement, turns into frustration. From frustration, turns into despair. From despair, turns into ideas!

A group of geniuses in the community found a breach in how Sketchware processes these blocks. They were the first few people to be able to decrypt, parse, and understand the inner workings of Sketchware save files. They figured out a way to “smuggle” in custom Java code by editing the save files manually, which received a ton of attention from the community!

This breakthrough led to the creation of Sketchware Mods, custom versions of the app that not only reintroduced ASD Blocks, but also other advanced features that weren’t possible for the original devs to make. These mods spread like wildfire, and many users migrated to these versions to continue their projects with community-led features, not under the dictation of a single team.

This ultimately leads to the split of Sketchware users: Those who use mods and are continuously modding it, and those who still believe the stance of the original team.

If you’ve been following this whole shenanigan unfold upon your eyes, you probably have heard of names such as Agus JCoderz, Aldi Sayuti, IndoSW, and Mike Anderson. Some of these people are the pinnacle of the resistance against the original Sketchware team, developing mods by modifying the Sketchware App in a dalvik bytecode-level.

There has been a lot of Sketchware Mods made by a lot of people in the community. Familiar names such as Sketchware Studio, Sketchware Pro, or Sketchware Gold. One mod that stood out from the crowd until today is Sketchware Pro. It has transformed to be an open-source github repo where anyone can contribute and help the development of the mod itself. Here is their github: [https://github.com/Sketchware-Pro/Sketchware-Pro](https://github.com/Sketchware-Pro/Sketchware-Pro), started as a fork of Agus JCoderz’s mod, then led by Mike Anderson, now maintained by people like Hasrat-ali and Jbk0 until this very day.

And hey I actually contributed something to them as well! The development of [zipalign-java](https://github.com/iyxan23/zipalign-java) was primarily intended to fix Sketchware Pro’s problem of not being able to use the `zipalign` dynamically-linked binary on some devices. And I achieved that :)

Coming back to the main topic, the rise of Sketchware mods really did mark a significant turning point in the community. It shows that the power of user-driven innovation and the desire for more advanced features, despite the original philosophy of Sketchware saying otherwise. Not only does this split bring good to the community, but it was also in some way, harming the original developers of Sketchware.

Sketchware’s primary source of money is only by displaying ads. And it’s surprisingly easy to shave off admob identification in an application to make it ad-free! That’s what they did in modded versions. They remove the ads, and offer advanced features.

This is like adding insult to injury, being the fact that the Sketchware team suffered a lot of blows from previous controversies, now Sketchware is being replaced by mods. The team was unable to keep up with the rapid development of advanced features of Sketchware mods.

They have always been spreading the word “using any sketchware mod is illegal” on their Slack channels to their remaining people that still believe in the original team of developing Sketchware.

Their ad revenue was going down, and patreons started to diminish. An app with over 5+ million downloads on play store was crumbling, brought to it knees by the community, started with just some minor controversies, up until literally being replaced.

As we see today, original Sketchware is no more. They have ceased its development years prior, then removed the listing of Sketchware from play store, whether it was done by the team or removed by google itself. You aren’t even able to open the original Sketchware without turning off your internet, because the it wasn’t able to connect to its servers, which had been shut down years ago.

It’s a really sad sight to see such impactful app meet its demise.

### Are Sketchware Mods the culprit in the death of Sketchware?

I wouldn’t say they are the main culprit, there were a lot of factors that came in before Sketchware Mods were developed in the first place. After all, it was like the community is shouting for the original Sketchware developers that they wanted advanced features, but their concerns were mostly dismissed, or weren’t able to be developed quick enough, as they were racing against the rapid development of Sketchware mods.

In my own personal opinion, from what I could understand over this whole massive drama, the reason of Sketchware’s death stems from the original team’s inadequate response to the concerns of the community. Taking an incorrect course of action, and angering a side of the user base. Which led a community effort to take the matters into the community’s hands, building a modified version that is superior to the original.

And that was the final blow of Sketchware, the modded version flourished with contributions from people all over the community, with ads removed, making it the default choice for users to use. Simply because it is just better and improved.

## End

I guess that is all I could relay to you guys about the history of Sketchware, the app that acted as a gateway for me to this world of programming. It’s really sad seeing the state of Sketchware today; unable to be used, and the slowly dying community. Reminiscing the old days, being able to know and learn from a lot of people from all over the world, being a part of the community and helping each other. Those memories will always resonate in my heart until the end of time.

But at least we have Sketchware Pro, still actively maintained by wonderful people that believed Sketchware should be able to be used for everyone, even after its demise. Before you see them as terrible, I have to tell you all that I do not hold a grudge against these people. They are literally the replacement of the original Sketchware team that took the burden of maintaining a complex, one-of-a-kind application still alive for years (and more years to come).

They do not ask for money or any incentives, they just wanted Sketchware to still be alive, be able to inspire young coders, and help a new generation of coders to grow in the confines of their phone. Despite its pivotal role in the death of Sketchware, I am still grateful that it exists.

iyxan23, signing off.
