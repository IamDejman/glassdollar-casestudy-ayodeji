# Decision requests: an answer from the business unit

Brief for GlassDollar OS, pain point 3: "Business units submit a problem and then disappear". Written on 12 October 2026 without access to the product, its users or its data. I chose it because the person who stalls is outside the tool, so the fix has to reach into their inbox and the result can be measured in days. Nordwind Industrial AG, its people and its numbers are invented so the examples can be concrete. Outside sources are named. Everything else is an assumption.

- Prototype: https://iamdejman.github.io/glassdollar-casestudy-ayodeji/
- Code and this brief: https://github.com/IamDejman/glassdollar-casestudy-ayodeji
- Tickets: https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/TICKETS.md, and in Linear at https://linear.app/glassdollar-case/issue/GLA-5 (invite needed)

## Problem

A business unit head submits a problem. The innovation manager scouts startups and sends a shortlist. Then nothing comes back. The manager chases by email and in corridors.

### How real it is

In Innovation Leader's 2019 survey of 115 large companies, 39% named lack of business unit alignment a top challenge, behind time and resources, lengthy contracts and clashing internal processes. At a 2019 Mind the Bridge forum, 41% of 36 innovation heads cited scarce business unit involvement as a barrier. Heads of venture client units say owners are keen but day-to-day business comes first (Haarmann et al. 2024). GlassDollar's own guide says a business unit "loses interest halfway through". None of these sources measures response time at the shortlist step.

### What it costs

27pilots claims an adopted solution is worth EUR 2 to 3 million a year (a marketing figure). About a quarter of started pilots reach adoption (Holcim, self-reported). Gimmy quotes about EUR 25,000 for a software pilot. Every day a shortlist waits defers that expected value. It is deferred rather than lost, and the odds at shortlist stage are lower than at pilot stage. For startups, half name slow corporate decisions as a top barrier (Nesta 2016, n=134).

## Users

Lena Hartmann is the innovation manager (GlassDollar's label; venture client manager elsewhere). She is in the tool every day, runs many pain points at once, and today chases owners herself by email.

Jonas Keller is the pain point owner, the line manager who raised the problem. He is measured on his operation and opens the tool a few times a year. He reads email on his phone. Often the person who raised the problem is not the one who can judge the shortlist.

Dr. Sabine Koch, head of the unit, is who Lena escalates to. She is not a user of this release.

## Assumptions and what we would validate

| Assumption | Evidence today | How to validate | Time |
|---|---|---|---|
| Owners go quiet because the ask is open-ended. A bounded ask with a date and a consequence gets an answer | Mixed. Shorter asks get more replies (Edwards et al. 2009). A stated time cost and a deadline showed no effect in the same review | 5 owner interviews, then a prototype test | 2 weeks |
| The owner who raised the problem is often not the one who can judge the shortlist | None | Ask Lena who answered her last ten shortlists, then count handoffs in the first quarter | 1 quarter |
| Owners stall at the shortlist step | The literature measures losses after the pilot. Nobody measures the shortlist step | Sent and answered timestamps for two quarters at two customers | 2 days |
| Lena needs the owner's answer and cannot pick alone | None | Ask Lena which decisions truly need the owner | 1 hour |
| Removing the login raises completion | No test with this audience. In consumer checkouts, forced account creation drives 18% of abandonments (Baymard 2025) | A/B at one customer, link versus login | 1 quarter |
| Emailed links survive enterprise gateways | Microsoft Safe Links opens links before delivery. Some tenants enforce SSO on outside links | Send to 10 addresses at one customer and ask IT | 1 week |
| Two automatic reminders and then a personal nudge is the right cadence | The first reminder adds the most (Van Mol 2014). Nudged pull requests close in 78 hours against 197 without a nudge (Maddila et al.) | Share answered by day 3 and by day 7 | 1 quarter |
| Measuring days per person is allowed at German customers | Works council co-determination may apply. This is not a legal reading | Ask one customer's HR and works council | 2 weeks |

## Scope

### What the first release includes

Estimated at 5 to 7 engineer-weeks.

- A Waiting on others list: every open request, on whom and for how long, with a Late state on day 8, and Remind now, Cancel and Answer on behalf on each row.
- One request type. The owner rates up to five startups as Interested, Not now or Need more info, or ticks "None of these fit" and says why.
- Handoff. The owner names who should answer instead, and Lena sends it on with one click and a restarted clock.
- One email with one button. The link lasts 14 days, survives mail scanners, and shows nothing until the "Yes, I am Jonas Keller" tap.
- A one-page form on the phone, then a confirmation that says what happens next.
- The answer lands on the shortlist, with one notification to Lena.
- Automatic reminders on days 3 and 7, paused when an auto-reply comes back.
- An event log behind the measures.

Before any of it is built, we test the link and SSO at one real customer.

Email is the channel because every owner already has it and it fits the estimate. What changes is the ask. Today's email says "log in and look at the shortlist". The new one asks for three ratings by a date, and Lena hears on day 8 if nothing comes back. If an owner ignores that too, the problem is incentives, which this release measures and does not fix.

### What it leaves out

- Other request types
- Teams or Slack
- Buttons inside the email
- Any owner login or dashboard
- Automatic email to the owner's manager
- Answering by email reply
- Voting by several stakeholders

Each item is in because removing it would leave the request stalling, Lena chasing, or the measure missing. Everything else is ordered by value against effort under What comes next.

## How we would measure success

Four measures, all read from the event log. Nobody measures this step today, so targets are set after one quarter of baseline.

- Days from sent to answered. The median, with unanswered requests counted as misses, and the share answered by the day 7 due date. This is the measure that matters.
- First touch within one working day. The share of owners who rate at least one startup or ask for information on the first day. An early sign that the ask lands.
- Answered without a manual chase. The share of requests answered before Lena pressed Remind now. Lena logs her chases for four weeks before launch to give the baseline.
- Requests gone late. The share open for longer than 1.2 times the unit's median wait.

Three things would distort them. Need more info counts as a first touch but cannot stop the clock. An answer has to carry a reason, or one click counts as an answer. Reports go by request, and owners' names stay out of them until the works council question is answered.

## What comes next

After launch, in order of value against effort: a daily "what is waiting" email to Lena, sent only on days when something is waiting. A view for the four measures above. A second request type for the decision after a pilot. Two-line plain-language summaries per startup, approved by Lena before they go out.

Three decisions sit with GlassDollar. Which decisions need the owner at all. Whether days 3, 7 and 8 are calendar or business days. Whether the OS should store each owner's manager, which escalation would need.

If the baseline shows most shortlists are already answered within a week, the first request type moves to the decision after a pilot instead.

## AI tools used

I used Claude Code for the research notes and the prototype.

It helped most on the prototype, which it built and fixed over many rounds, and on keeping the sample data consistent across the files.

Two survey figures were paraphrased wrongly and an early pass used the invented company data as evidence. Both were caught when I read the brief.
