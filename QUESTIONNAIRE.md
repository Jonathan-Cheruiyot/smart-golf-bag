# Post-Product Questionnaire: Smart Golf Bag

User evaluation protocol. Run with 3 people outside the class.
Budget 15 to 20 minutes per person.

---

## How to run it

**Recruit:** 3 people who have played golf, even casually. Family, friends,
teammates. A mix of experience levels is better than three identical golfers,
because a beginner and a regular will notice different things.

**Setup:** have the prototype already running on localhost before they sit
down. Do not make them watch it load.

**Ground rule for you:** Parts A and B happen **before you explain anything**.
The moment you narrate what the interface does, you lose the ability to learn
whether it explains itself. Sit on your hands.

**Recording:** write answers down as they speak, or record audio with
permission. Do not rely on memory. The write-up needs specifics, and
paraphrase from memory turns into mush.

### The honesty problem

People are reluctant to criticize something you made, to your face. Your
results will be inflated unless you actively work against it. Three things
that help:

- Lead with tasks, not opinions. What someone *does* is more honest than what
  they say.
- Ask for the worst thing, not the best thing. "What would annoy you about
  this?" gets better data than "what do you think?"
- Say out loud at the start: "This is a class project, the grade is for what
  I learn, not for whether you like it. Being harsh genuinely helps me."

---

## Part A: Background and needs

Ask these **before showing anything**. These recover the user-needs data the
project asks for, and they stand on their own as pre-design research if you
run them in a separate earlier session.

1. How often do you play, and do you carry, push, or ride?
2. Walk me through what you do with your bag during a round. Start at the
   first tee.
3. Have you ever left a club behind? What happened, how did you find out, and
   how long did it take?
4. Have you ever left the whole bag somewhere, or worried you might?
5. What do you check on your phone during a round?
6. Do you ever change which clubs are in your bag depending on the course or
   the weather? How do you decide?
7. If your bag could tell you one thing, what would it be?

> Question 7 is the most valuable one in this section. Record the answer
> verbatim. If all three people name something your interface does not do,
> that belongs in the Future Work section of your write-up.

---

## Part B: First impressions, no explanation

Show them the prototype. Say only: "This is a golf bag with a screen on it,
and the phone it pairs with." Nothing else.

8. Look at this for a moment and tell me what you think it is telling you.
9. What is the biggest thing on the bag screen, and what do you think it
   means?
10. What do you think the phone is for, as opposed to the bag?
11. Is there anything here you do not understand, or that looks like it does
    something but you are not sure what?

> Question 9 tests whether the club count reads as a club count. If someone
> guesses "holes" or "score," that is a real finding and it goes in the
> write-up whether or not you fix it.

---

## Part C: Tasks

Now give them the machine. Ask them to think out loud. **Do not help unless
they are completely stuck**, and if you do help, write down that you had to.

| # | Task | What to watch for |
|---|---|---|
| T1 | Start a round | Do they find the control, or look at the phone first? |
| T2 | Pretend you just pulled your 7 iron to hit a shot, then walk to the next hole without putting it back | Do they notice the alert on their own, or only when prompted? |
| T3 | Tell me what the bag is warning you about and what you would do next | Does "left behind on hole 3" tell them to walk back? |
| T4 | Swap one club out of your bag for a different one | Do they go to the phone, or hunt on the bag screen? |
| T5 | Turn off the alerts | Do they find it on either device? |
| T6 | Tell me how many shots you have taken | Do they know where to look? |

For each task record: completed unaided, completed with a hint, or failed.
Also record the time if it is noticeably long, and write down the first place
they looked, which is often more informative than whether they succeeded.

---

## Part D: Probing the design decisions

These test the specific choices you will have to defend in your presentation.

12. Some information is on the bag and some is only on the phone. Does that
    split make sense to you? Would you move anything?
13. The bag warns you about a missing club. The phone warns you when you have
    walked away from the whole bag. Do both of those feel useful, or is one
    of them unnecessary?
14. The bag screen is plain, high contrast, and does not look like a phone
    app. Does that look right for something strapped to a golf bag, or does
    it look unfinished?
15. When you pull a club and still have it in your hand, the bag stays calm
    and only warns you after you change holes. Is that the right moment to
    warn you? Should it be earlier or later?
16. Would you want this on your own bag? What would stop you?

> Question 15 probes the single most important piece of logic in the project.
> If users want the alert earlier, that is a finding worth reporting even if
> you disagree with it, and explaining why you disagree is good write-up
> material.
>
> Question 14 is the design-language question. If all three say
> "unfinished," you have a real problem. If they say "looks like a tool,"
> your e-ink direction is validated.

---

## Part E: Closing

17. What is the single most annoying thing about this?
18. What is missing?
19. If you could delete one thing from either screen, what would it be?

> Question 19 is the best question on this sheet. People will add features
> all day but rarely volunteer what to remove, and forcing a deletion surfaces
> what they found useless.

---

## Recording sheet

Copy this once per participant.

```
Participant: P1 / P2 / P3
Golf experience:
Carries / pushes / rides:
Date:

PART A, background and needs
1.
2.
3.
4.
5.
6.
7.  (verbatim)

PART B, first impressions
8.
9.   Guessed the big number means:
10.
11.  Confused by:

PART C, tasks          unaided / hinted / failed     first place they looked
T1  start a round
T2  leave a club
T3  explain the alert
T4  swap a club
T5  mute alerts
T6  find shot count

PART D, design probes
12.  bag vs phone split
13.  two alert types
14.  visual style
15.  alert timing
16.  would you use it

PART E, closing
17.  most annoying
18.  missing
19.  would delete

OBSERVED, not asked:
- moments of hesitation
- anything they tried that did not work
- anything they said unprompted
```

---

## Turning this into write-up content

The documentation asks you to report what you learned. Structure it like
this rather than as a transcript dump:

**What I expected.** The assumptions you built from, stated plainly. Club
loss is the main pain point, the count should be the dominant element, and
configuration belongs on the phone.

**What three users told me.** Findings grouped by theme, not by person. Use
one short verbatim quote per theme, and attribute by participant number
rather than name.

**Where I was wrong.** The most valuable section, and the one graders reward.
Anything a user did not understand, could not find, or disagreed with.

**What I changed, and what I did not.** If you keep a design despite feedback
against it, say so and give the reason. Defending a decision with evidence is
stronger than silently complying with every comment.

**Revised user needs and design requirements.** The table from
`REQUIREMENTS.md`, updated with anything the interviews surfaced. Mark which
rows came from users and which were your own assumptions, so the reader can
tell them apart.

> Note on method, for your write-up: if you ran this after building rather
> than before, say so in one sentence. "I designed from my own assumptions as
> a golfer, then validated them with three users and revised accordingly" is
> a real methodology, honestly described. An invented timeline is the thing
> that actually costs you credibility.
