# Decision requests: getting business units to answer startup shortlists

Take-home brief for GlassDollar OS, written on 12 Oct 2026 without access to the product, its users or its data. Nordwind Industrial AG and its people are invented so the examples can be concrete. Outside sources are named. Everything else is an assumption.

| Deliverable | Where |
|------|------------|
| Functional prototype | [Live prototype](https://iamdejman.github.io/glassdollar-casestudy-ayodeji/) and [code](https://github.com/IamDejman/glassdollar-casestudy-ayodeji) |
| Tickets, five | Section 6 below, [TICKETS.md](https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/TICKETS.md) and [Linear GLA-5](https://linear.app/glassdollar-case/issue/GLA-5) (invite needed) |
| Brief | This document, sections 1 to 5 |

## 1. Pain point

Number 3: "Business units submit a problem and then disappear."

I chose it because the person who stalls is outside the tool, so the fix has to reach into their inbox, and the result can be measured in days.

## 2. Problem framing

### Who has it

| Person | Role | Context |
|-----|---------|-----------|
| Lena Hartmann | Innovation manager, the primary user (GlassDollar's label; venture client manager elsewhere) | In the tool every day, runs many pain points at once, chases owners herself by email |
| Jonas Keller | Pain point owner, the line manager who raised the problem | Measured on his operation, opens the tool a few times a year, reads email on his phone |
| Dr. Sabine Koch | Head of the unit | Lena's escalation point, not a user of this release |

### When it happens

Lena sends Jonas a shortlist and waits. Jonas means to look at it, but his own work comes first and days go by with no answer. Lena follows up by email, then catches him in the corridor. Sometimes the shortlist has gone to the wrong person: Jonas raised the problem, but someone else on his team is the one who can say whether a startup would work.

### What it costs

- Business unit disengagement is a known top barrier. In Innovation Leader's 2019 survey of 115 large companies, 39% named lack of business unit alignment a top challenge. At a 2019 Mind the Bridge forum, 41% of 36 innovation heads cited scarce business unit involvement. Heads of venture client units say owners are keen but the day job comes first (Haarmann et al. 2024).
- Startups feel it too. Half name slow corporate decisions as a top barrier (Nesta 2016, n=134).
- Each waiting day defers a pilot (about EUR 25,000, Gimmy) and a possible adoption (claimed EUR 2 to 3 million a year, 27pilots, a marketing figure; about a quarter of pilots adopt, Holcim). Value is deferred rather than lost, and the odds at shortlist stage are lower than at pilot stage.
- No source measures response time at the shortlist step itself.

### How we would know it is solved

Four measures, all read from the event log. Nobody measures this step today, so targets are set after one quarter of baseline.

| Measure | Definition |
|------|------------|
| Days to answer (the one that matters) | Median days from sent to answered, with unanswered requests counted as misses, and the share answered by the day 7 due date |
| First touch | Share of owners who rate at least one startup or ask for information within one working day |
| No manual chase | Share of requests answered before Lena presses Remind now. Lena logs her chases for four weeks before launch to give the baseline |
| Late rate | Share of requests open for longer than 1.2 times the unit's median wait |

Three guardrails keep them honest. Need more info counts as a first touch but does not stop the clock. An answer has to carry a reason, or one click would count as an answer. Reports go by request, and owners' names stay out of them until the works council question is answered.

### Assumptions to validate first

| Assumption | Evidence today | How to validate | Time |
|-----------|-------------|-----------|---|
| Owners stall at the shortlist step | The literature measures losses after the pilot. Nobody measures the shortlist step | Sent and answered timestamps for two quarters at two customers | 2 days |
| Lena needs the owner's answer and cannot pick alone | None | Ask Lena which decisions truly need the owner | 1 hour |
| Emailed links survive enterprise mail gateways and SSO | Microsoft Safe Links opens links before delivery. Some tenants enforce SSO on outside links | Send to 10 addresses at one customer and ask IT | 1 week |
| A bounded ask with a date gets an answer where an open-ended one does not | Mixed. Shorter asks get more replies (Edwards et al. 2009). A stated time cost and a deadline showed no effect in the same review | 5 owner interviews, then a prototype test | 2 weeks |

Four more wait for live data: how often the raiser cannot judge the shortlist, whether no login raises completion (forced account creation drives 18% of consumer checkout abandonments, Baymard 2025), whether two reminders and a personal nudge is the right cadence, and whether measuring days per person is allowed at German customers.

## 3. Look around

Patterns borrowed from tools that already get busy outsiders to respond. All from the research notes in the repository.

| Borrowed from | What they do | Used here as |
|-------|------------|----------|
| E-signature (DocuSign) | Sign from one email button with no account. Reminders and expiry are sender settings. Assign to Someone Else covers forwarded mail | One email, one button, no login, 14-day expiry, "Not me" and a handoff |
| Code review (Microsoft's Nudge, Maddila et al.) | An automated nudge after 24 hours of inactivity cut the mean overdue pull request lifetime from 197 hours to 78 | Automatic reminders and a Waiting on others list |
| Scheduling (Doodle, Calendly) | Reply without an account, give a name and email, see a confirmation on screen | One-page form on the phone, then a confirmation that says what happens next |
| Service desks (Jira Service Management, Zendesk) | The clock stops at "Waiting for customer". First reply time excludes automated replies | Need more info counts as a first touch but, on purpose, does not stop the clock. Auto-replies never count as an answer |
| CRMs (Pipedrive, HubSpot) | A deal turns red after a set number of inactive days and sorts first. Email sequences stop on a reply or a bounce, and pause on out-of-office | Late on day 8, Late rows first. Reminders stop on an answer, pause on an auto-reply or a bounce |
| Survey research (Edwards et al. 2009, Van Mol 2014) | Shorter asks get more replies. The first reminder adds the most | Three ratings, "about 3 minutes", two reminders at most |

## 4. Scope

The first release is 5 to 7 engineer-weeks. Before any of it is built, we test the link and SSO at one real customer.

| In the first release | Later or out |
|---|---|
| A Waiting on others list with Late on day 8, and Remind now, Cancel and Answer on behalf on each row | Other request types |
| One request type: rate up to five startups, or "None of these fit" with a reason | Teams or Slack |
| Handoff: the owner names who should answer, Lena sends it on with one click and a restarted clock | Buttons inside the email |
| One email with one button, and a link that lasts 14 days and survives mail scanners | Any owner login or dashboard |
| A one-page form on the phone, then a confirmation | Automatic email to the owner's manager |
| The answer lands on the shortlist, with one notification to Lena | Answering by email reply |
| Reminders on days 3 and 7, paused on an auto-reply | Voting by several stakeholders |
| An event log behind the measures | |

Each item is in because removing it would leave the request stalling, Lena chasing, or the measure missing. Everything else is ordered by value against effort under What comes next.

Email is the channel because every owner already has it and it fits the estimate. What changes is the ask. Today's email says "log in and look at the shortlist". The new one asks for three ratings by a date, and Lena hears on day 8 if nothing comes back. If an owner ignores that too, the problem is incentives, which this release measures and does not fix.

## 5. Prototype

The main flow end to end, from the [clickable prototype](https://iamdejman.github.io/glassdollar-casestudy-ayodeji/). It has a clock to jump to day 3, 7, 8 and 13, and a switch for Jonas being out of office.

**Lena sends one request from the shortlist.** The dialog says the due date, the reminder days and when the request goes late.

<img src="images/send-dialog.png" width="560" alt="The send dialog: Ask Jonas Keller to rate this shortlist, with the due date, the reminder days, a note field and a Send request button">

**Jonas answers on his phone with no login.** One email with one button, then an identity tap so mail scanners cannot trigger anything, then three choices per startup.

<img src="images/jonas-flow.png" width="900" alt="Three phone screens: the email with a Review 3 startups button, the identity page, and the answer page with three choices per startup">

**Lena tracks everything in one list.** On day 8 the request goes Late and she hears about it once. Each row shows owner, age, due date, state and reminders, with Remind now, Cancel and Answer on behalf.

<img src="images/waiting-on-others.png" width="900" alt="The Waiting on others list on day 8: PP-040 and PP-034 both Late, with the notice that Jonas Keller has not answered PP-034">

## 6. Tickets

Five tickets, built in order. Each ships on its own. The full specs, with what to build, user stories, acceptance criteria and edge cases, are in [TICKETS.md](https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/TICKETS.md). DR-1 has no dependencies and is ready to pick up.

| Ticket | Title | What it delivers | Skills | Size | Depends on | Status |
|----|---------|------------------------|-----------|---|------|-------|
| DR-1 | Send and cancel a request from the shortlist | One button opens a dialog with an optional note. Send creates a Waiting request and snapshots the shortlist. The header shows state and age. Cancel takes a reason | Backend, frontend | M | Nothing | Ready |
| DR-2 | The email, and a link that survives mail scanners | One email, one button, a plain-text link. A hashed token that lasts 14 days. Nothing counts until "Yes, I am Jonas Keller" is tapped. Expired links renew up to three times | Backend, email deliverability (SPF, DKIM, DMARC), security | L | DR-1 | Waits on DR-1 |
| DR-3 | Jonas answers on one page, Lena sees it on the shortlist | Phone-first page with three choices per startup, or "None of these fit" with a reason. Answers land on the shortlist with one notification. The owner can name who should answer instead | Frontend (phone-first form, accessibility), backend | L | DR-1, DR-2 | Waits on DR-2 |
| DR-4 | Reminders on days 3 and 7, pause when away, Late on day 8 | Two reminders, skipped if the owner was active in the last 24 hours. Pause on an auto-reply or a bounce. Late on day 8 with one alert to Lena | Backend (scheduled jobs), inbound email parsing | L | DR-1, DR-2 | Waits on DR-2 |
| DR-5 | The Waiting on others list and the four measures | The list with Open, Answered and Cancelled tabs and row actions Remind now, Cancel, Answer on behalf and Send to someone else. The four measures as queries over the event log | Frontend, backend (permissions, reassignment), SQL | L | DR-3, DR-4 | Waits on DR-4 |

All five follow the rules at the top of TICKETS.md: one open request per shortlist, calendar days in Europe/Berlin, jobs at 09:00 on weekdays, nothing closes a request on its own, and one event table. A ticket is done when its acceptance criteria pass against the sample data and the edge cases in the spec are handled.

## What comes next

After launch, in order of value against effort: a daily "what is waiting" email to Lena, sent only on days when something is waiting. A view for the four measures above. A second request type for the decision after a pilot. Two-line plain-language summaries per startup, approved by Lena before they go out.

Three decisions sit with GlassDollar. Which decisions need the owner at all. Whether days 3, 7 and 8 are calendar or business days. Whether the OS should store each owner's manager, which escalation would need.

If the baseline shows most shortlists are already answered within a week, the first request type moves to the decision after a pilot instead.

## AI tools used

| Tool | Used for | Where it helped | Where it got in the way |
|---|---|---|---|
| Claude Code | The research notes, the prototype and the sample data | It built and fixed the prototype over many rounds, and kept the sample data consistent across the files | Two survey figures were paraphrased wrongly and an early pass used the invented company data as evidence. Both were caught when I read the brief |
