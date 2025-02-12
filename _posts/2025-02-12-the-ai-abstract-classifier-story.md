---
layout: post
title: "the ai-abstract-classifier story"
date: 2025-02-12 11:02:32
categories: ai python
---

## Using Granite to help organize Conferences (and more!)

I have helped organize [DevOpsDays Austin][dodatx] for the last 4 to 5 years. If you do not know, [DevOpsDays][dod]
is a volunteer/community-run conference where technology professionals come together regionally to learn
about the ecosystem they work in daily. The best part of DevOpsDays is the "[open spaces][openspaces]," where it is an
unconference style for half the day, and people learn things together. I have learned more about the innovative
technologies we use, and I have learned about a tool that streamlined my workflow. There are talks throughout
the day, and my primary responsibility is the "cfp" (call for paper) lead, where I help recruit and run the
agenda for that part of the conference. We get 100s submissions each year, and it takes significant time to
sift through them. I have even grown a "sixth sense" of reading some of these cfps to find "sales pitches"
or "not in the spirit" of what we are trying to do.

Over the last couple of years, we have noticed an increase in AI written submissions, and it is becoming
a real problem. Asking an LLM "to write me an abstract or call for paper about why the DevOps process is
the best way to run production servers" and copying, pasting, and submitting it has become a real problem.
We will not accept these talks as soon as we realize they are “robotic”; though we must read every one, and
honestly, it is a waste of everyone's time. We want our speakers to write something about themselves and their
experience so you can help grow our community's knowledge. Along with AI-generated abstracts, another thing
we deprioritize is obvious "sales pitches." Our audience does not want to give you our stage for you to
attempt to sell your product. Don’t get me wrong, we have sponsorship time slots where our community
understands it is the correct place for this place interaction, but not our main stage or talk tracks.
We also get a few attempts from marketing teams and developer relations professionals to sneak in these
types of cfps, which are just product pitches.

This brings us to my release of [ai-abstract-classifier][github]. I have created a wrapper application that takes
these abstracts submitted to our conferences and gives a score of 0-100% of being "ai written" or a
possible "sales pitch." This gives us, part of the CFP committee, an ability to de-prioritize talks we
are not looking for. It runs locally, leveraging [Granite][granite] via Ollama, with an interface of AnythingLLM.
It is a straightforward, secure, local, and private way to leverage an open source LLM to save us hours
of reading; ideally, we do not have to do it during the first pass of cfps. If you want give it a shot,
it has support for CSVs and some test abstracts already in place, but the main power all of this is
running through our Granite model, and saving us so so much time.

[dod]: https://devopsdays.org
[openspaces]: https://devopsdays.org/open-space-format/
[github]: https://github.com/jjasghar/ai-abstract-classifier
[granite]: https://ollama.com/library/granite3.1-dense
[dodatx]: https://devopsdays.org/austin
