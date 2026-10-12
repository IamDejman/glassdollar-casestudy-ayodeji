# Decision requests: getting business units to answer startup shortlists

Take-home brief for GlassDollar OS, written on 12 Oct 2026 without access to the product, its users or its data. Nordwind Industrial AG and its people are invented so the examples can be concrete. Where a figure comes from an outside source, the source is named. The rest is assumption.

| Deliverable | Where |
|------|------------|
| Functional prototype | [Live prototype](https://iamdejman.github.io/glassdollar-casestudy-ayodeji/) and [code](https://github.com/IamDejman/glassdollar-casestudy-ayodeji) |
| Five tickets | [TICKETS.md](https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/TICKETS.md) and [Linear GLA-5](https://linear.app/glassdollar-case/issue/GLA-5) (invite needed) |
| Brief | This document, sections 1 to 5 |

## 1. Pain point

Number 3: "Business units submit a problem and then disappear."

I chose it because the person who stalls is outside the tool, and the result can be measured in days.

## 2. Problem framing

### Who has it

| Person | Role | Context |
|-----|---------|-----------|
| Lena Hartmann | Innovation manager, the primary user (GlassDollar's label; venture client manager elsewhere) | In the tool every day, runs many pain points at once, chases owners herself by email |
| Jonas Keller | Pain point owner, the line manager who raised the problem | Measured on his operation, opens the tool a few times a year, reads email on his phone |
| Dr. Sabine Koch | Head of the unit | Lena's escalation point when an owner stays silent. If the cause turns out to be incentives rather than friction, she is the lever, because she sets what Jonas is measured on. Not a user of this release |

Jonas raised the problem because it costs his operation, and he wants it gone. What answering gets him is a pilot that might fix it, months away, paid for with time he has to find today. That is the trade he makes every time the shortlist email arrives.

### When it happens

Lena sends Jonas a shortlist and waits. Jonas means to look at it, but his own work comes first and days go by with no answer. Lena follows up by email, then catches him in the corridor. Sometimes the shortlist has gone to the wrong person: Jonas raised the problem, but someone else on his team is the one who can say whether a startup would work.

Today's email asks for the wrong thing. It says "log in and look at the shortlist", with no question, no date and no way to answer without an account. The new one asks for three ratings by a date.

### Why owners go silent

Five reasons Jonas might not answer. This release works on the first one only.

1. Friction and competing work: the ask is open-ended, needs a login, and his own work comes first, which is what heads of venture client units report (Haarmann et al. 2024).
2. The shortlist does not look credible, or he needs more to judge it.
3. He has no budget or authority to commit to a pilot.
4. The problem was solved another way, or went away.
5. Backing a startup is a political risk in his unit.

The first is the only one the OS can remove on its own, and removing it exposes the others: a request for more information points at 2, a rejection with a reason at 2 or 4, and silence after a bounded ask with a date at 3 or 5, which is Sabine's conversation, not the tool's.

### What it costs

- Lena's own cost is the one nobody measures: shortlists per quarter, times days waiting, times chases per waiting day, times minutes per chase. The four-week chase log before launch fills this in, and it becomes the baseline for No manual chase.
- Startups feel it too. Half name slow corporate decisions as a top barrier (Nesta 2016, n=134).
- Each waiting day defers a pilot (about EUR 25,000, Gimmy) and a possible adoption (claimed EUR 2 to 3 million a year, 27pilots, a marketing figure; about a quarter of pilots adopt, Holcim). Value is deferred rather than lost, and the odds at shortlist stage are lower than at pilot stage.
- No source measures response time at the shortlist step itself.

### How we would know it is solved

We would watch four numbers, all pulled from the event log. Days to answer is the headline and the other three explain it. Nobody measures this step today, so the first quarter is for finding the baseline. Targets come after that.

| Measure | Definition |
|------|------------|
| Days to answer | Median days from the first send to answered, counted across resends, with unanswered requests counted as misses, and the share answered by the day 7 due date |
| First touch | Share of owners who rate at least one startup or ask for information within one working day |
| No manual chase | Share of requests answered before Lena chases by hand. Lena logs her chases for four weeks before launch to give the baseline |
| Late rate | Share of requests, open or answered, whose wait is more than 1.2 times the unit's median, or more than 8 days until the unit has 10 answered requests |

Three rules stop the numbers being gamed:

1. A request for more information counts as a first touch but does not stop the clock. A resend keeps the first send date.
2. An answer is a choice for every startup, and Need more info and None of these fit carry a reason, so one click cannot count as an answer.
3. Reports go by request, and owners' names stay out of them until the works council question is answered.

### Assumptions to validate first

| Assumption | Evidence today | How to validate | Time |
|-----------|-------------|-----------|---|
| Owners stall at the shortlist step | The literature measures losses after the pilot. Nobody measures the shortlist step | Sent and answered timestamps for two quarters at two customers | 2 days |
| Lena needs the owner's answer and cannot pick alone | None | Ask Lena which decisions truly need the owner | 1 hour |
| Emailed links survive enterprise mail gateways and SSO | Microsoft Safe Links opens links before delivery. Some tenants enforce SSO on outside links | Send to 10 addresses at one customer and ask IT | 1 week |
| A bounded ask with a date gets an answer where an open-ended one does not | Mixed. Shorter asks get more replies (Edwards et al. 2009). A stated time cost and a deadline showed no effect in the same review | 5 owner interviews, then a prototype test | 2 weeks |

The second assumption has no evidence yet. I could not ask Lena for this brief, so that hour is the first hour of the project, before the link test. If most shortlists turn out not to need the owner, the first request type moves to the decision after a pilot, where the owner's answer is unavoidable.

Four more wait for live data:

1. How often the raiser cannot judge the shortlist.
2. Whether no login raises completion. Forced account creation drives 18% of consumer checkout abandonments (Baymard 2025).
3. Whether two reminders and a personal nudge is the right cadence.
4. Whether measuring days per person is allowed at German customers.

## 3. Look around

These patterns come from tools that already get busy outsiders to respond. The detail is in the research notes in the repository.

| Borrowed from | What they do | Used here as |
|-------|------------|----------|
| E-signature (DocuSign) | Sign from one email button with no account. Reminders and expiry are sender settings. Assign to Someone Else covers forwarded mail | One email, one button, no login, 14-day expiry, "Not me" and naming who should answer |
| Code review (Microsoft's Nudge, Maddila et al.) | An automated nudge after 24 hours of inactivity cut the mean overdue pull request lifetime from 197 hours to 78 | Automatic reminders and a Waiting on others list |
| Scheduling (Doodle, Calendly) | Reply without an account, give a name and email, see a confirmation on screen | One-page form on the phone, then a confirmation that says what happens next |
| Service desks (Jira Service Management, Zendesk) | The clock stops at "Waiting for customer". First reply time excludes automated replies | Need more info counts as a first touch but, on purpose, does not stop the clock. Auto-replies never count as an answer |
| CRMs (Pipedrive, HubSpot) | A deal turns red after a set number of inactive days and sorts first. Email sequences stop on a reply or a bounce, and pause on out-of-office | Late on day 8, Late rows first. Reminders stop on an answer. Lena can pause them, and pausing on an auto-reply waits for the next release |
| Survey research (Edwards et al. 2009, Van Mol 2014) | Shorter asks get more replies. The first reminder adds the most | Three ratings, "about 3 minutes", two reminders at most |

## 4. Scope

The first release is about 3 to 4 engineer-weeks. Before any of it is built, we ask Lena which decisions need the owner and test the link and SSO at one real customer. The tickets deploy in order, and DR-1 to DR-3 together are the first slice a user gets value from.

| In the first release | Later or out |
|---|---|
| Lena picks who answers, owner preselected, and sends with a note | Other request types |
| One request type: rate up to five startups, or "None of these fit" with a reason | Teams or Slack |
| One email with one button, and a link that lasts 14 days and survives mail scanners | Buttons inside the email |
| A one-page form on the phone, then a confirmation | Any owner login or dashboard |
| The owner can name who should answer, and Lena resends from the same dialog | Automatic email to the owner's manager |
| The answer lands on the shortlist, with one notification to Lena | Answering by email reply |
| Reminders on days 3 and 7, Late on day 8, and Pause when Lena knows the owner is away | Voting by several stakeholders |
| A Waiting on others list of open requests, with Remind now, Cancel and Answer on behalf on each row | Auto-reply and bounce detection |
| An event log behind the measures | Link renewal after day 14 |
| | One-click send-on with a fresh link, and contacts outside the OS |
| | Answered and Cancelled tabs, and a unit-wide view |

Each item is in because without it the request still stalls, Lena still chases, or we cannot measure the result. Everything else is ranked by value against effort under What comes next. The release tests an unproven idea, so anything that only polishes an edge case waits for the next one. The price is that a wrong email address shows up only as Late on day 8, and an expired link needs Lena to resend.

Email is the channel because every owner already has it and it fits the estimate. Lena hears on day 8 if nothing comes back. If an owner ignores a bounded ask with a date too, the problem is incentives. This release will show that, but it cannot fix it.

## 5. Prototype

The [clickable prototype](https://iamdejman.github.io/glassdollar-casestudy-ayodeji/) covers the main flow end to end: Lena sends one request from the shortlist, Jonas answers on his phone with no login, and Lena tracks everything in one list. It has a clock to jump to day 3, 7, 8 and 13, and a Pause reminders button for when Jonas is away. Screenshots are at the end of [TICKETS.md](https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/TICKETS.md).

## 6. Tickets

There are five tickets, deployed in order. DR-1 to DR-3 together are the first slice a user gets value from. The overview table and the full specs, with what to build, user stories, acceptance criteria and edge cases, are in [TICKETS.md](https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/TICKETS.md). DR-1 has no dependencies and is ready to pick up.

All five follow the rules at the top of TICKETS.md:

1. One open request per shortlist.
2. Calendar days in Europe/Berlin.
3. Jobs at 09:00 on weekdays.
4. Nothing closes a request on its own.
5. One event table.

A ticket is done when its acceptance criteria pass against the sample data and the edge cases in the spec are handled.

## What comes next

After launch, in order of value against effort:

1. A daily "what is waiting" email to Lena, sent only on days when something is waiting.
2. A view for the four measures above.
3. Reminders that pause on their own on an auto-reply, and a bounce alert to Lena.
4. A second request type for the decision after a pilot.
5. One-click send-on with a fresh link when the owner names someone else.
6. Two-line plain-language summaries per startup, approved by Lena before they go out.
7. Link renewal, the Answered and Cancelled tabs, and a unit-wide view of the list.

Three decisions sit with GlassDollar:

1. Which decisions need the owner at all.
2. Whether days 3, 7 and 8 are calendar or business days.
3. Whether the OS should store each owner's manager, which escalation would need.

If the baseline shows most shortlists are already answered within a week, the first request type moves to the decision after a pilot instead.

## AI tools used

| Tool | Used for | Where it helped | Where it got in the way |
|---|---|---|---|
| Claude Code | The research notes, the prototype and the sample data | It built and fixed the prototype over many rounds, and kept the sample data consistent across the files | Two survey figures were paraphrased wrongly and an early pass used the invented company data as evidence. Both were caught when I read the brief |
