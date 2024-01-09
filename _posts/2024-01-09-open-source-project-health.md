---
layout: post
title: "Open Source Project Health"
date: 2024-01-09 14:43:27
categories: opensource linux ibm
---

> I was writing this up for some internal conversations, but felt like this could be a more generic list for my blog. Hopefully this'll help someone in the future.

## Introduction

These are some criteria for understanding the health of an open-source project. These guidelines should be researched and understood before leveraging anything from upstream. There will be situations where some of these guidelines may clash, and that is expected.  In this situation you should make a judgement call on what the risker option is of the offending guideline, take a moment and balance this. Many of these markers come with experience in the open-source ecosystem, and eventually, you get a “feel” for the health by just looking at the main pages of the open-source project.

**Note**: These guidelines are not in any “order” or "have a value.” You should ask yourself questions before adopting or building upon any open-source project.

## Questions

- How many “watchers” are on the project?

Reason: This lists how many people get emailed if anything happens to the repository. This means that that is the number of people “actively” and opted in to paying attention to it. The higher the number the better the sign of a positive to health of the project.

- When was the last “commit?”

Reason: This shows how often things are fixed, or updates happen on the repository.  This could be as easy as a Documentation PR to a brand new feature, and anything in between. This goes hand in hand with the “watchers.” This also could be a CI platform running daily builds or security checks, this a sign of a positive health for a project; letting the bots keep the health is always a good sign.

- What is the “oldest issue” on the project?

Reason: This shows how long issues stick around on the project, and how quickly they get answered and closed. This is an indicator of the “tending of the garden” of the repository, and how often they look for old questions or issues to resolve. If there is an issue with “old” comments that haven’t continued a conversation, this highlights a possible situation that the project needs to address and this could be the negative health sign of the project.

- Where is the “getting started guide?”

Reason: This shows they can get people to adopt and “run” their project quickly. If they have a bunch of code out there with no clear way to start using it, this highlights a problem with adoption, and is a negative sign for the health of the project.

- Is there a clear path to commitment to the project or ways to help?

Reason: If the documentation doesn’t show a straightforward way to join/engage/help the project, this is a significant red flag for the project. Having an open-source project is about building something together, and if you can’t find a way into the project, this is a negative healthy sign.

- Is there more than “one” contributor?

Reason: If this open-source project only has one contributor, this could be a sign of a “pet project” or something with an attractive contribution policy. More contributors mean there are more voices in the project, which at least implies many eyes on the project, which is a positive sign. The inverses “in general” can be a negative sign for health.

- Is there a security policy or process for reporting security issues?

Reason: With the issues of other open-source projects building upon other bits of open-source, there needs to be a standard process to report security issues when they are discovered. If there isn’t a clear path to report or help update the security issue, this is a negative sign for the project.

- Is there a standard license for the project?

Reason: As open-source developers, we must pay attention to the projects we want to use; if the project uses a “nonstandard” license, this could cause unforseen issues. The standard “acceptable” licenses for generic open source adoption are Apache License 2.0, BSD-3-Clause, and the MIT License. If a project is using something different, and you really need to understand the rules that the upstream project now abides by.

- What is their policy for releasing “numbered” releases?

Reason: If they are going via the SemVer release process, how far are they from 1.0.0? How far are they from a “production” release if they are going through another process? This can show the confidence in the project from the Core maintainers and how “production healthy” they think it is.

## Conclusion

Many of these positives and negatives are situational, and there are many more exceptions than this being a “steadfast” rubric to grade against. Like all open-source and professional developers, you must use your judgment and understand that taking code from upstream has associated risks.

It should be mentioned that even with some of this negotiate traits, if you are thinking about adopting a project you can always help resolve the “marks against it.” Every negative in the open-source space can be turned to an opportunity to help commit and grow the project.

Though more often, the positives outweigh the negatives, which is the foundation upon which open-source is built.
