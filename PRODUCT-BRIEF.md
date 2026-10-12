# Decision requests: getting business units to answer startup shortlists

This brief reads best on a desktop screen. The tables do not fit a phone.

Take-home brief for GlassDollar OS, written on 12 Oct 2026 without access to the product, its users or its data.

| Deliverable | Where |
|------|------------|
| Functional prototype | [Live prototype](https://iamdejman.github.io/glassdollar-casestudy-ayodeji/) and [code](https://github.com/IamDejman/glassdollar-casestudy-ayodeji) |
| Five tickets | [TICKETS.md](https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/TICKETS.md) is the source of truth, mirrored to [Linear GLA-5](https://linear.app/glassdollar-case/issue/GLA-5) (invite needed) |
| Brief | This document, sections 1 to 5 |
| Supporting notes | [SUPPORTING-NOTES.md](https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/SUPPORTING-NOTES.md): what was cut from this brief, with sources |

Context: Nordwind Industrial AG and its people, Lena Hartmann, Jonas Keller and Dr. Sabine Koch, are invented so the examples can be concrete. Where a figure comes from an outside source, the source is named. The rest is assumption.

## 1. Pain point

Number 3: "Business units submit a problem and then disappear."

I chose it because the person who stalls is outside the tool, and the result can be measured in days.

## 2. Problem framing

### Who has it

| Person | Role | Context |
|-----|---------|-----------|
| Lena Hartmann | Innovation manager, the primary user | In the tool every day, runs many pain points at once, chases owners herself by email |
| Jonas Keller | Pain point owner, the line manager who raised the problem | Measured on his operation, opens the tool a few times a year, reads email on his phone |
| Dr. Sabine Koch | Head of the business unit | Lena's escalation point. She sets what Jonas is measured on. Not a user of this release |

### When it happens

Lena sends Jonas a shortlist and waits. Jonas raised the problem because it costs his operation and he wants it gone, but answering costs him time now, and what he gets back is a pilot months away that might not work. His own work comes first and days go by. Lena follows up by email, then catches him in the corridor. Sometimes the shortlist went to the wrong person: Jonas raised the problem, but someone on his team is the one who can judge the startups.

I have not seen the current GlassDollar email. This brief assumes it says "log in and look at the shortlist", with no question, no date and no way to answer without an account. The new one asks for three ratings by a date.

Friction is one of five reasons an owner might go silent, and the only one the OS can reduce. The supporting notes have all five and how the owner's reply points at each.

### What it costs

Nobody measures response time at the shortlist step or what a day of delay costs. Each waiting day delays the evaluation and the pilot after it. The manager's chasing time is unmeasured too, so she logs her chases for four weeks before launch. The formula is in the supporting notes.

### How we would know it is solved

Four numbers from the event log. Days to answer is the main one and the other three explain it. Nobody measures this step today, so the first quarter sets the baseline and targets come after.

| Measure | Definition |
|------|------------|
| Days to answer | Two numbers. The median days from the first send to a rated answer, over rated answers only and counted across resends and Ask again. And the share of requests answered by the day 7 due date, where a request without a rated answer counts as a miss once its due date has passed. A rated answer gives Interested or Not now on at least one startup, or says None of these fit with a reason. All Need more info is not rated |
| First response | Share of requests with a submitted answer within one working day, including an answer that only asks for more information |
| No manual chase | Share of answered requests where nobody pressed Remind now or answered on behalf. It counts follow-ups made in the product. Emails and calls outside it are not recorded |
| Late rate | Share of requests whose wait, counted to the rated answer or to today if there is none, is more than 1.2 times the business unit's median, or more than 8 days until the business unit has 10 rated answers |

Three rules stop the numbers being gamed:

1. Need more info on every startup counts as a first response, not as a rated answer. The request closes as Answered, but the Days to answer clock runs on until the manager's Ask again gets a rated answer. A resend keeps the first send date. If the manager never asks again, the request stays unresolved for the measures: a miss after day 7, counted in Late rate with the clock still running, and never in the median.
2. An answer is a choice for every startup, and Need more info and None of these fit carry a reason, so one click cannot count as an answer.
3. Reports go by request and business unit, and owners' names stay out of them until the works council question in the supporting notes is answered.

### Assumptions to validate first

| Assumption | Evidence today | How to validate | Time |
|-----------|-------------|-----------|---|
| The manager needs the owner's answer and cannot pick alone | None | Ask the manager which decisions truly need the owner | 1 hour |
| Emailed links survive enterprise mail gateways and SSO | Microsoft Safe Links opens links before delivery. Some tenants enforce SSO on outside links | Send to 10 addresses at one customer and ask IT | 1 week |
| A short ask with a date gets an answer where an open-ended one does not | Mixed. Shorter asks get more replies (Edwards et al. 2009). A stated time cost and a deadline showed no effect in the same review | 5 owner interviews, then a prototype test | 2 weeks |

The first has no evidence, and I could not ask the manager for this brief, so that hour comes first. If most shortlists do not need the owner, the first request type becomes the decision after a pilot, where the owner has to answer. The premise that owners stall at this step, and four smaller assumptions, need live data and are in the supporting notes.

## 3. Look around

Three patterns from tools that get busy outsiders to respond. Three more, and the sources for all six, are in the supporting notes.

| Borrowed from | What they do | Used here as |
|-------|------------|----------|
| E-signature ([DocuSign](https://community.docusign.com/signing-7/non-registered-users-3966)) | Sign from one email button with no account. Reminders and expiry are sender settings. Assign to Someone Else covers forwarded mail | One email, one button, no login, 14-day expiry, "Not me" and naming who should answer |
| Code review ([Microsoft's Nudge](https://arxiv.org/abs/2011.12468), Maddila et al.) | An automated nudge after 24 hours of inactivity cut the mean overdue pull request lifetime from 197 hours to 78 | Automatic reminders and a Waiting on others list |
| CRMs ([Pipedrive](https://support.pipedrive.com/en/article/the-rotting-feature), [HubSpot](https://knowledge.hubspot.com/sequences/unenroll-from-sequence)) | A deal turns red after a set number of inactive days and sorts first. Email sequences stop on a reply or a bounce, and pause on out-of-office | The request turns Late on day 8, and Late rows sort first. Reminders stop on an answer. The manager can pause them, and pausing on an auto-reply waits for the next release |

## 4. Scope

The first release is about 3 to 4 engineer-weeks. Before DR-1, we ask the manager which decisions need the owner. Before DR-2, we test the link and SSO at one real customer. The tickets deploy in order, and nothing is useful before DR-3 is live.

| In the first release | Later or out |
|---|---|
| The manager picks who answers, owner preselected, and sends with a note | Other request types |
| One request type: rate up to five startups, or "None of these fit" with a reason | Teams or Slack |
| One email with one button, and a link that works for 14 days without a login | Buttons inside the email |
| A one-page form on the phone, then a confirmation | Any owner login or dashboard |
| The owner can name who should answer, and the manager resends from the same dialog | Automatic email to the owner's manager |
| The answer lands on the shortlist, with one notification to the manager | Answering by email reply |
| Reminders on days 3 and 7, Late on day 8, and Pause when the manager knows the owner is away | Voting by several stakeholders |
| A Waiting on others list of open requests, where each row has Remind now, Cancel and Answer on behalf actions | Auto-reply and bounce detection |
| An event log behind the measures | Link renewal after day 14 |
| | One-click send-on with a fresh link, and contacts outside the OS |
| | Answered and Cancelled tabs, and a unit-wide view |

Each item is in because without it the request still stalls, the manager still chases, or we cannot measure the result. The idea is unproven, so edge-case polish waits for the next release. Two things we accept for now: a wrong email address shows up only as Late on day 8, and an expired link needs the manager to resend.

Email is the channel because every owner already has it and it fits the estimate. The manager hears on day 8 if nothing comes back. If the owner still does not respond, the shorter ask was not enough, and the manager looks into what remains: friction, competing priorities, missing information or authority.

## 5. Prototype

The [clickable prototype](https://iamdejman.github.io/glassdollar-casestudy-ayodeji/) shows the whole flow: the manager sends one request from the shortlist, the owner answers on a phone with no login, and the manager tracks everything in one list. A clock jumps to day 3, 7, 8 and 13. Screenshots are at the end of [TICKETS.md](https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/TICKETS.md#screenshots). The data is invented.

## 6. Tickets

Five tickets, deployed in order, are in [TICKETS.md](https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/TICKETS.md): an overview table, acceptance criteria for the whole flow, and sub-issues in build order with their own criteria. The rules they share, and what done means, are at the top of that file. DR-1 can be built once the one-hour owner check is done, DR-2 waits for the link test as well, and every other open question has a default and blocks nothing.

## What comes next

After launch, in order of value against effort:

1. A daily "what is waiting" email to the manager, only on days when something is waiting.
2. A view for the four measures.
3. Reminders that pause on an auto-reply, and a bounce alert to the manager.
4. A second request type for the decision after a pilot.
5. One-click send-on with a fresh link when the owner names someone else.
6. Two-line plain-language summaries per startup, approved by the manager.
7. Link renewal, the Answered and Cancelled tabs, and a unit-wide view.

Three decisions sit with GlassDollar:

1. Which decisions need the owner at all.
2. Whether days 3, 7 and 8 are calendar or working days.
3. Whether the OS should store each owner's manager, which escalation would need.

If the baseline shows most shortlists are already answered within a week, the first request type moves to the decision after a pilot instead.

## AI tools used

Claude Code wrote the research notes, built the prototype over many rounds and kept the sample data consistent across the files. It paraphrased two survey figures wrongly and once used the invented company data as evidence, both caught when I read the brief. The detail is in the supporting notes.
