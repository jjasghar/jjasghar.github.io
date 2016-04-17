---
layout: post
title: "Characteristics of a Successful Chatroom Meeting"
date: 2015-11-18 18:05:58 -0600
comments: true
categories: linux openstack irc
---

Author: [JJ Asghar][jj]

Edited: [Adam Leff][adam]

I've spent some time over the last few years in Chatroom meetings.
Based on my experience, here are some characteristics of a successful Chatroom
meeting for both the organizer and attendees.

## Organizers

- Create an agenda, and allow meeting participants to collaborate on it before the
meeting.
- Be sure to have at least one Chair for the meeting. The Chair opens,
closes, and moderates the meeting.
- Announce the meeting ahead of time using the main way to communicate with your
user group. Use this opportunity to remind them of the meeting and offer them a
link to the agenda for review and collaboration.
- Leverage a tool such as [meetbot][meetbot] as your scribe and timekeeper. If
you can't run something that can do it automatically for you, designate a
note taker and timekeeper. Be sure to announce the note taker and timekeeper
at the beginning of the meeting.
- Before allowing discussion on topics not listed on the agenda, ensure all items on the agenda
have been discussed first.
- As all participants type at different speeds, allow for extra time for comments
from slower typists. State the pause so the participants understand the reason for the silence:

```
Chair: OK, any other comments before we move on to item 2? Waiting 20 seconds for additional comment...
```

- If it appears a topic has come to a conclusion and a decision has been reached,
announce the decision and allow for dissenting opinions. Use this
opportunity to clarify any aspects of the decision that are necessary.
Provide a time-boxed pause to allow users to voice any dissent. For example:

```
Chair: Seems the group has decided that vi is better then emacs. If you disagree,
please '-1'.  Waiting 20 seconds for any feedback...
Attendee1: -1
Chair: Attendee1: can you explain?
Attendee1: Chair: Even though the group has decided this, I think this is still
a matter of personal choice.
Attendee2: Chair: +1
```

- Keep the conversation going by asking relevant questions and seeking out quieter
attendees for their opinions. This helps keep the conversation active while making
all participants feel their voice is being heard.
- Take note of any topics that come up in discussion that were not listed as an agenda
item. Add them to the next agenda to make sure that the issue raised is resolved, or
provide time at the end of the meeting for discussion if time allows.
- Most importantly, be friendly and helpful! The Chair is here to ensure a successful
meeting with a successful outcome, and that can only happen if people feel welcomed,
valued, and comfortable.

## Attendees

- Remember that you are there to share your opinions and gather input for a given
topic or set of topics. Be constructive and helpful, and everyone will benefit.
- The Chair is responsible for keeping to the agenda and moderating the flow of
the meeting. The Chair may have to cut a conversation short in order to ensure
all items on the agenda are discussed. If you wish to continue discussion on a
particular topic over the allotted time, ask for an extension. If an extension is
not possible or feasible, ask for the topic to be added to the next agenda, or
seek out another form of communication, such as the group's mailing list.
- When the meeting starts, announce yourself as an attendee. Announcements such
as `o/`, `hello` or `I'm here` are considered acceptable. In addition to making
it clear you are here to participate, the note taker or meeting bot will have an
accurate record of those in attendance.
- Do your best to stay on topic. This is a forum for synchronous feedback and
it's easy for tangential conversations to muddy the waters. If you feel your
topic needs to be discussed, ask the Chair to add it to the end of the agenda
or propose it for a future meeting.
- When replying, address the user directly. In addition to getting his/her attention,
it helps other attendees follow the conversation flow as if it were happening
face-to-face. For example:

```
Attendee1: I think vi is better then emacs.
Chair: No way emacs is better then vi!
Attendee2: I think sublime text is the clear winner.
Attendee3: Attendee1: What about atom?
Attendee1: Attendee3: I do everything through a terminal, so atom isn't an option.
Attendee2: Chair: Not everyone likes to use parentheses in their configuration files.
```

- An amazing aspect of online meetings is the ability to engage with users from all
around the world on a single topic. These users come from different personal, cultural,
and technical backgrounds. Some may not type as fast or as accurately as you, and some
may be communicating in a language that is not their primary language. Be patient with
your fellow attendees and be extra understanding of written inaccuracies.
- Be friendly and respectful! You are here share your opinion, hear other opinions, and
learn from each other. Everyone's input is valuable. Because tone-of-voice cannot properly
be portrayed in a text-only format, keep jokes and sarcasm to a minimum to ensure no attendee
misinterprets your message. If you feel offended by another attendee's message,
assume no malicious intent and ask him/her to re-explain their message.

[meetbot]: https://wiki.debian.org/MeetBot
[jj]: https://github.com/jjasghar
[adam]: https://github.com/adamleff
