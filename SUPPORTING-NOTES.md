# Supporting notes

Details behind [PRODUCT-BRIEF.md](https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/PRODUCT-BRIEF.md), written on 12 Oct 2026. These notes hold what was cut from the brief and allow me to talk more in depth about the brief and how it was built. Roles are used throughout:

- the manager is the innovation manager
- the owner is the pain point owner
- the head of unit is the owner's boss

## Why owners go silent

Five reasons an owner might not answer:

1. Friction and competing work: the decision ask is open-ended, needs a login, and the owner's own work comes first, which is what heads of venture client units report ([Haarmann et al. 2024](https://www.designsociety.org/publication/47638/The+Venture+Client+Model+and+Its+Challenges+for+Established+Companies), built from 7 interviews with venture client unit heads, 5 workshops and a 35-person world cafe: business units are keen, but day-to-day business often takes precedence).
2. The shortlist does not look credible, or the owner needs more to judge it.
3. The owner has no budget or authority to commit to a pilot.
4. The problem was solved another way, or went away.
5. Backing a startup is a political risk in the owner's unit.

The OS can only reduce the first. Once the ask is short, the owner's reply shows which one: a request for more information points at 2, a rejection with a reason at 2 or 4. If the owner still does not respond, the shorter ask was not enough. The manager then looks into remaining friction, competing priorities, missing information or authority, and takes incentives to the head of unit, who sets what the owner is measured on.

No source measures this directly. The nearest evidence is that follow-up contact raises survey response (odds 1.35, [Edwards et al. 2009](https://researchonline.lshtm.ac.uk/id/eprint/5119)) and that one named recipient gets more replies than a group (38% alone against 14% in groups of 15 or 50, [Blair et al. 2005](https://psychology.ecu.edu/wp-content/pv-uploads/sites/216/2019/03/Blair-Thompson-Wuensch-2005.pdf), students and a small favour). None of these studies tests a manager rewarded for other work.

## What it costs

- Nobody measures response time at the shortlist step, and nobody has measured what a day of delay costs. Both costs below are unmeasured.
- Each waiting day delays the evaluation and whatever pilot follows it. What a day of delay costs is unknown. The wait hurts startups too: about half of 134 startups named slow corporate decisions as a top barrier ([Nesta, Scaling Together, 2016](https://www.scaleupinstitute.org.uk/wp-content/uploads/2019/11/scaling_together.pdf)).
- Nobody counts the manager's own time. The formula is shortlists per quarter, times days waiting, times chases per waiting day, times minutes per chase. The manager logs her chases for four weeks before launch to fill it in. That log is the baseline for her time, not for the No manual chase measure, which only sees what happens in the product.

## Assumptions that need live data

The four assumptions in the brief can be checked before the build. These four need the product running.

1. How often the raiser cannot judge the shortlist.
2. Whether no login raises completion. There is no evidence for B2B approvals, so we test it, then the prototype test with and without a login.
3. Whether two reminders and a personal nudge is the right amount of chasing. In one survey the first reminder added 10.3 points on a 6.2% invite response, the second 8.6 and a late third 6.1 ([Van Mol 2014](https://www.nidi.nl/shared/content/output/papers/nidi-wp-2014-15.pdf), 15,651 students).
4. Whether a German works council allows measuring wait time per named owner. This is the works council question the brief and the tickets refer to. Until it is answered, reports go by request and business unit.

## Questions behind the checks

The three checks before the build and the owner interviews each have three questions to answer.

**The two-day pull of past dates.** How many days pass between a shortlist going to the owner and the first answer, as a median and a spread? How many shortlists have no answer after 14 days? Does the wait differ by business unit or by who raised the pain point?

**The hour with the manager.** Of the last ten shortlists, which ones needed the owner's view and which could she have decided alone? When the owner did answer, what did the answer change? Who else in the owner's unit could judge the startups?

**The week-long link test.** Does the link arrive, and does the mail client rewrite it? Does a scanner open it before the owner does, and how many times? Does the tenant force a login on outside links?

**The five owner interviews.** When a shortlist arrived, what did you do first, and what stopped you answering the same day? What would you need to see to pick one startup over another? Who in your unit would you hand this to, and when?

## Look around, all six patterns

| Borrowed from | What they do | Used here as | Source |
|-------|------------|----------|------|
| E-signature (DocuSign) | Sign from one email button with no account. Reminders and expiry are sender settings. Assign to Someone Else covers forwarded mail. Decline to Sign takes a reason | One email, one button, no login, 14-day expiry, "Not me" and naming who should answer. None of these fit with a required reason | https://community.docusign.com/signing-7/non-registered-users-3966 and https://community.docusign.com/esignature-111/can-you-forward-email-to-other-people-to-sign-97 |
| Code review (Microsoft's Nudge, Maddila et al.) | A batch job flags pull requests past their predicted lifetime, skips any active in the last 24 hours, names the blocker and nudges once. In the trial the mean overdue pull request lifetime fell from 197 hours to 78 over 12,356 pull requests in 147 repositories | Automatic reminders and a Waiting on others list. A reminder is not owner activity | https://arxiv.org/abs/2011.12468 |
| Scheduling (Doodle, Calendly) | Reply without an account, give a name and email, see a confirmation on screen. Calendly's optional email code is a gate: an unverified booking is never made | One-page form on the phone, then a confirmation that says what happens next. No code to type | https://help.doodle.com/en/articles/9457279-how-do-i-participate-in-a-group-poll and https://calendly.com/help/how-to-require-email-verification-for-event-types |
| Service desks (Jira Service Management, Zendesk) | The clock stops at "Waiting for customer". First reply time excludes automated replies. Zendesk splits requester wait from agent wait | Need more info closes the request as Answered but, on purpose, does not stop the clock. Auto-replies never count as an answer | https://confluence.atlassian.com/servicemanagementserver105/example-sla-configurations-1540732977.html and https://support.zendesk.com/hc/en-us/articles/4408834848154 |
| CRMs (Pipedrive, HubSpot) | A deal turns red after a set number of inactive days and sorts first. Email sequences stop on a reply or a bounce, and pause on out-of-office | The request turns Late on day 8, and Late rows sort first. Reminders stop on an answer. The manager can pause them, and pausing on an auto-reply waits for the next release | https://support.pipedrive.com/en/article/the-rotting-feature and https://knowledge.hubspot.com/sequences/unenroll-from-sequence |
| Survey research (Edwards et al. 2009, Van Mol 2014) | Shorter asks get more replies (odds 1.64). Personalised asks a little more (1.14). The first reminder adds the most. A stated time cost and a deadline showed no effect | Three ratings, "about 3 minutes" as a label only, two reminders at most, the owner addressed by name | https://researchonline.lshtm.ac.uk/id/eprint/5119 and https://www.nidi.nl/shared/content/output/papers/nidi-wp-2014-15.pdf |

Enterprise email, the third assumption in the brief: Microsoft Defender's Safe Links scans a URL before delivery, whether or not it rewrites the link, so at least one automated request hits the link before the owner does. A link that is spent on first open would be spent by the scanner. The answer page therefore does nothing until the owner taps a button on it. Source: https://learn.microsoft.com/en-us/defender-office-365/safe-links-about.

## Two metrics kept for later

The first release computes Days to answer and No manual chase. They are out because each needs rules a first quarter does not: one counts in working days while everything else counts in calendar days, and the other needs a running clock for unanswered requests and a per-unit median.

| Measure | Definition |
|------|------------|
| First response | Share of requests with a submitted answer within one working day, including an answer that only asks for more information |
| Late rate | Share of requests whose wait, counted to the rated answer or to today if there is none, is more than 1.2 times the business unit's median, or more than 8 days until the business unit has 10 rated answers |

## Ticket rules

All five tickets follow the rules at the top of [TICKETS.md](https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/TICKETS.md):

1. One open request per shortlist.
2. Calendar days in Europe/Berlin.
3. Jobs at 09:00 on weekdays.
4. A request closes only when a person answers or cancels it.
5. One event table.

A sub-issue is done when its acceptance criteria pass against the sample data, and a ticket when all its sub-issues are. Three checks come before the build: pull past Send and Answer dates from the OS to see whether owners stall at the shortlist step (two days, before DR-1), ask the manager at the pilot customer which decisions need the owner (one hour, before DR-1), and send a test link to 10 addresses at that customer to see whether it survives the mail gateway and SSO (one week, before DR-2).

## AI tools used

| Tool | Used for | Where it helped | Where it got in the way |
|---|---|---|---|
| Claude Code | Drafts of the brief, the tickets and the research notes, the prototype and the sample data. | It built and fixed the prototype over many rounds, kept the sample data consistent, and the reviews caught gaps between the brief, the tickets and the prototype | It paraphrased two survey figures wrongly and once used the invented company data as evidence. I caught both when reading. |
