# Tickets: Business units submit a problem and then disappear

## Why this epic exists

Lena sends Jonas a shortlist and then nothing happens for two weeks. She chases him by email, he can't find his login, and the pain point sits in Shortlist. We want to cut the days between "shortlist sent" and "owner answered".

The fix is one list for Lena of everything still waiting on someone. A decision request is what fills it. Lena asks one question from the shortlist. Jonas gets an email with one button and answers on one page in about three minutes. If he doesn't answer, the system reminds him. If he isn't the right person, he says who is and Lena sends it on with one click.

## Order

Build DR-1 to DR-5 in that order. The acceptance criteria use the sample data at https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/data/sample-data.json, where today is 2026-10-12, so you can check your work without asking anyone.

## Rules that apply to every ticket

- A request is one question from a manager to one owner about one shortlist. This release has one question: rate this shortlist.
- States: Waiting, Paused, Late, Answered, Cancelled. Open means Waiting, Paused or Late. Any open state can go to Answered or Cancelled, which are final.
- One open request per shortlist, enforced in the database.
- Days are calendar days since the date of sending, in the workspace timezone (Europe/Berlin). Sent on 29 September: day 3 is 2 October, due_at is 6 October 23:59, the link expires 13 October 23:59. Age never resets.
- Jobs run at 09:00 on weekdays. Anything due on a weekend happens on Monday.
- Anyone who can edit the pain point can send, cancel, remind, answer on behalf or send on. The event log records who.
- A notification to Lena is one in-app notification and one email, the workspace default in the sample data.
- Nothing closes a request on its own. Day 14 only expires the link. A Late request stays until Lena acts.
- Finalised means the pain point has left Shortlist. Reminders and the Late transition stop. The request stays open so a late answer is still recorded.
- Every ticket writes to one table: event {request_id, type, at, actor, payload}. DR-5 lists the types.
- The seed: PP-034 is Jonas Keller's pain point. Sent 29 September, reminders on 2 and 6 October, Late since 7 October, day 13 on 12 October. PP-040 (Anneke de Vries) was sent 18 September and is open at day 24.

## Left out on purpose

Suggested stage change, editing a request after sending, the closing-the-loop email and the daily digest. The brief says why: https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/PRODUCT-BRIEF.md.

---

### DR-1: Create a decision request from a shortlist

Lena presses one button on the shortlist. The request goes out, she can see it is out there, and she can cancel it.

#### Build

- On the shortlist header of a pain point in the Shortlist stage: the button "Ask Jonas Keller to rate this shortlist". The name is the pain point's owner.
- The button opens a dialog with the email the owner will get (DR-2's template, plain template text until DR-2 ships), an optional plain-text note to the owner of up to 1,000 characters, and "Send request".
- Send creates the request in Waiting and logs created. Sending the email is DR-2.
- While open, the header shows "Waiting for Jonas Keller, day N", "Late: Jonas Keller, day N" or "Paused: Jonas Keller, day N". Once Answered, DR-3 shows the answers there. Once Cancelled, the button comes back.
- Next to the state: "Cancel request". It confirms, takes an optional reason, sets Cancelled and logs cancelled. The owner gets no email.
- A startup added or removed while a request is open shows "Shortlist changed since the request" with "Cancel and resend", which cancels and reopens the dialog with the note prefilled.

Not here: other kinds of question, more than one owner, sending to someone other than the owner (see Open questions), editing after sending.

#### Data

request {id, pain_point_id, shortlist_id, owner_id, sent_by, note, state, sent_at, due_at, link_expires_at, startups_snapshot[], reminders_sent, flag, cancelled_reason}

startups_snapshot holds {startup_id, name, description, manager_comment} as at send time. DR-3 shows the snapshot, not the live shortlist. Defaults: state Waiting, reminders_sent 0, flag none.

#### Edge cases

- Under 3 or over 5 startups: button disabled, "Needs 3 to 5 startups".
- No owner, owner not a user in the OS, or owner is the sender: button disabled, says which.
- Two managers send at once: one request, the other sees "Already sent by Lena Hartmann".
- A description or comment edited after sending: no banner, the snapshot stands.
- Finalised while open: header "Finalised, answer still open until 2026-10-13".

#### Done when

- Lena sends PP-034 on 2026-09-29: Waiting, sent_by Lena Hartmann, due_at 2026-10-06 23:59, link_expires_at 2026-10-13 23:59, snapshot has Pallet.ai, Cargonaut Labs and Parcelgrid with descriptions and Lena's comments, created logged.
- Tobias Brandt opens PP-034 on 2026-10-12: "Late: Jonas Keller, day 13", no send button.
- A shortlist with 2 startups: disabled, "Needs 3 to 5 startups".
- A shortlist whose owner is Henrik Vos, who is not a user (as on PP-017): disabled, "Henrik Vos has no email address in the OS".
- Lena adds a fourth startup while open: banner shows. "Cancel and resend": old request Cancelled, dialog opens with the note, Send creates a new Waiting request with 4 startups.
- Lena cancels PP-034 with the reason "Jonas is on leave": Cancelled, reason stored, cancelled logged, no email to Jonas, button back.
- Lena and Tobias send within a second: one request, "Already sent by Lena Hartmann".
- PP-034 finalised on 2026-10-12 while open: stays Late, header "Finalised, answer still open until 2026-10-13", no reminder.

#### Open questions

- Can Lena send to someone other than the owner? Default no. A wrong owner names the right one (DR-3) and Lena sends it on (DR-5). Ask Lena.
- Should finalising cancel the request? Default no. Ask Lena.

#### Details

- Analytics: decision_request_created {request_id, startups_count, note_length}; decision_request_cancelled {request_id, age_days, has_reason}
- Module: Pain points
- Estimate: M
- Depends on: none

---

### DR-2: Email the owner with a link that survives scanners

One email, one button. Mail scanners open every link first, so nothing counts until a person taps "Yes, I am Jonas Keller".

#### Build

- From "GlassDollar for Nordwind" at a GlassDollar address. Reply-To is Lena, so a human reply reaches her. Return-Path is the OS inbound address, so bounces and auto-replies come back (DR-4).
- Subject "PP-034: Lena Hartmann needs your view on 3 startups". Body: who asks, what for, "about 3 minutes", Lena's note, the due date. One text button "Review 3 startups" with the full URL under it. No startup names.
- The link carries a random token (32 bytes or more, stored hashed) tied to the request and the owner's address, valid until link_expires_at.
- GET shows "Yes, I am Jonas Keller" and "Not me". The tap (POST) sets confirmed_at on first use and starts a session cookie for the request until link_expires_at. The token stays valid, so the same link works from reminders and on other devices. With a live session, GET goes straight to the answer page.
- "Not me" shows "Thanks, we've told Lena Hartmann". Nothing else changes. Lena is told once per request.
- Expired link: "This link has expired" with "Request a new link". Same address, 14 more days, Lena told "Jonas Keller asked for a new link". Open requests only, 3 times at most, then "Ask Lena Hartmann for a new link". Answered: "Your answer was sent on 12 October". Cancelled: "Lena no longer needs this".
- SPF, DKIM and DMARC on the sending domain. Add "allowlist the sender" to the customer setup checklist.

Not here: buttons inside the email, Teams or Slack, a sending domain per customer.

#### Data

link {request_id, token_hash, bound_email, expires_at, confirmed_at, renewals}

An open before any tap is likely_bot when it comes from a known scanner range, or lands within 60 seconds of sending with no tap after it. Events: link_opened {likely_bot}, identity_confirmed, not_me, link_renewed.

#### Edge cases

- A scanner fetches the link: no owner activity recorded.
- A sandbox presses the button: confirmed_at is set. Nothing more can happen without choices, and the measures count answers, not taps.
- Images blocked: the button is text in a table cell, and the plain URL works.
- Forwarded email: "Not me" is for this.
- Sent on to someone else (DR-5): the old link shows "This request went to Petra Ahlers".
- Owner's address changes after sending: the old link keeps working, the new address is used from the next email.

#### Done when

- Jonas's email for PP-034 contains "Review 3 startups", "about 3 minutes" and "2026-10-06", and none of Pallet.ai, Cargonaut Labs or Parcelgrid.
- A scanner fetches a new link three times: still Waiting, confirmed_at empty, no owner activity.
- Jonas taps on his phone on 2026-10-01 and opens the link on his laptop on 2026-10-02: identity page once, then the answer page, confirmed_at still 2026-10-01.
- Jonas taps on 2026-10-13: the page opens. On 2026-10-14: "This link has expired" with "Request a new link".
- He requests a new link on 2026-10-14: new link, expires 2026-10-28 23:59, renewals 1, Lena told once, the old link stays expired.
- A fourth request: "Ask Lena Hartmann for a new link", nothing sent.
- Anneke de Vries taps "Not me" on a forwarded email: no startup shown, Lena told "Someone other than Jonas Keller opened the PP-034 link".
- Request cancelled, Jonas opens the link: "Lena no longer needs this", no identity page.
- Images blocked: button readable, URL visible.

#### Open questions

- Does Nordwind force SSO on outside links? Ask Nordwind IT.
- One shared GlassDollar sending domain, or one per customer? Ask Nordwind IT.
- German or English? Default English until the OS has a workspace language. Ask Lena.

#### Details

- Analytics: decision_link_opened {request_id, likely_bot}; decision_identity_confirmed {request_id, days_since_sent}; decision_link_renewed {request_id, renewals}
- Module: Notifications, Shared
- Estimate: L
- Depends on: DR-1

---

### DR-3: Let the owner answer on one page, and land the answer on the shortlist

Jonas picks a choice per startup and sends. Lena sees it on the shortlist. Nobody asks twice.

#### Build

- One page, phone first, no navigation, behind the DR-2 session. It shows the PP-034 title, Lena's note, and one card per startup from the snapshot: name, description, Lena's comment, the choices Interested, Not now and Need more info, and a comment box. Under the cards: "None of these fit" with a comment box that replaces the choices, and "Send to Lena".
- Need more info and None of these fit require a comment. Send is always enabled. If something is missing, the page says what and moves focus to it.
- After sending: "Sent to Lena Hartmann." with a summary. Reopening a valid link shows the answer read-only.
- On the shortlist, each startup shows the choice, who, when and how. Lena gets one notification: "Jonas Keller answered PP-034", or "Jonas Keller needs more information on 3 startups" when nothing was rated.
- After an answer the button comes back as "Ask Jonas Keller again". That is the follow-up to Need more info. The information itself goes by email or a call.
- Under Send: "Someone else should answer this". A name field that suggests OS users as he types, free text allowed. "Tell Lena" keeps the request open, tells Lena "Jonas Keller says Petra Ahlers should answer PP-034" and shows "Sent to Lena Hartmann. She'll send PP-034 to Petra Ahlers." If he answers anyway, the answer wins and the handoff is cleared.

Not here: editing after sending, suggested stage change, SourceAI profiles, attachments.

#### Data

answer {request_id, items[{startup_id, choice, comment}], none_fit, none_fit_comment, submitted_at, via (magic_link | on_behalf), late, rated}

Sending sets Answered. late is true after finalised. rated is true with at least one Interested or Not now, or none_fit. All Need more info means rated false.

handoff {request_id, suggested_name, suggested_user_id, by, at}. It does not change the state. An answer clears it. Events: answered {via, rated, late}, handoff {matched}.

#### Edge cases

- Two tabs submit: the second sees "Already sent".
- Opened after cancel: "Lena no longer needs this". After send-on: "This request went to Petra Ahlers". After answer on behalf: "Already answered by Lena Hartmann".
- 5 startups in the snapshot: five cards, same rules.
- Comments: 1,000 characters, with a counter from 100 left.

#### Done when

- After the tap: three cards, Lena's note, no navigation.
- 2 of 3 rated, or None of these fit without a comment, or Need more info without a comment: nothing sent, the page says what is missing, focus moves there.
- Pallet.ai Interested, the others Not now, sent 2026-10-12: Answered, via magic_link, rated true, shortlist shows "Jonas Keller, 2026-10-12, via link", one notification, button reads "Ask Jonas Keller again".
- All three Need more info with comments: Answered, rated false, notification "Jonas Keller needs more information on 3 startups".
- Finalised 2026-10-12, sent 2026-10-13: late true, shortlist shows "received after shortlist was finalised".
- He types "Pet" and picks Petra Ahlers: suggested_user_id U-009. He types "Henrik Vos": name stored, no user id.
- Handoff on 2026-10-01: still Waiting, Lena told once, reopening shows "Sent to Lena Hartmann" with her name.
- Handoff on 2026-10-01, then all three rated on 2026-10-02: Answered, handoff cleared, no "Send to Petra Ahlers instead" in the list.
- Lena answers on behalf at 10:00, Jonas sends at 10:01: "Already answered by Lena Hartmann", Lena's answer stands.

#### Open questions

- Should the confirmation promise a reply time, say "within 3 working days"? Ask Lena.
- Should Need more info keep the request open? Default no: Answered with rated false, and Lena sends a new request once she has the information. Ask Lena.

#### Details

- Analytics: decision_answer_submitted {request_id, days_since_sent, via, rated, late, seconds_on_page}; decision_handoff {request_id, matched}
- Module: Pain points, Shared
- Estimate: L
- Depends on: DR-1, DR-2

---

### DR-4: Remind on days 3 and 7, pause when the owner is away, mark Late on day 8

Two automatic reminders, a pause when the out-of-office replies, and a Late mark on day 8 so Lena knows when to step in.

#### Build

- Day 3 and day 7: the first email again with "Reminder 1 of 2" or "Reminder 2 of 2" on top, same link. Subject "Reminder: PP-034, Lena Hartmann needs your view by 2026-10-06". Never more than two.
- Skip a reminder when the owner confirmed identity or opened the page in the last 24 hours, or Lena pressed Remind now in the last 24 hours.
- Day 8 with no answer: Late, and Lena is told "Jonas Keller has not answered PP-034 (day 8)".
- An auto-reply (Auto-Submitted other than "no", X-Auto-Response-Suppress, Precedence auto_reply, or a subject starting "Automatic reply", "Automatische Antwort", "Out of office", "Abwesenheit" or "Abwesenheitsnotiz"): flag away, state Paused, reminders and the Late transition stop. Lena sees "Jonas Keller appears to be away" with "Resume reminders".
- A hard bounce: flag bounced, state Paused, Lena sees "Email not delivered" with "Send again", which is DR-1's cancel and resend to the current address. Soft bounces are retried by the provider and not shown.
- Resume: Waiting, or Late at once if day 8 has passed, and Lena is told once. Reminders missed while paused are not sent.
- Several reminders due the same day for one owner: one email, with a section and a button per request.

Not here: reading return dates out of auto-replies, drafting a note to the owner's manager, customers changing the cadence, public holidays.

#### Data

The jobs are idempotent and re-check the state and the event log right before sending. Fields: reminders_sent, last_owner_activity_at, flag (none | away | bounced), paused_at, resumed_at. A reminder goes out only when Waiting, reminders_sent under 2 and the pain point still in Shortlist. Events: reminder_sent {reminder_no, merged}, reminder_skipped {reason}, paused {flag}, resumed, late.

#### Edge cases

- The owner answers between the check and the send: aborted.
- An auto-reply after the answer: ignored. On a Late request: flag stored, state stays Late, no second notification.
- Resume before day 7: the day 7 reminder still goes out, a missed day 3 does not.
- The job runs twice: one email.
- Day 3 on a Saturday: Monday at 09:00.

#### Done when

- Sent 2026-09-29, no answer: reminders on 2026-10-02 and 2026-10-06 at 09:00, reminders_sent 2, Late on 2026-10-07 with one in-app notification and one email.
- Jonas opened the page on 2026-10-02 at 08:10: nothing at 09:00, day 7 still goes out.
- Auto-Submitted: auto-replied arrives 2026-10-03: no day 7, Paused, "Jonas Keller appears to be away".
- Subject "Automatische Antwort: PP-034: Lena Hartmann needs your view on 3 startups" arrives 2026-10-03: the same.
- Resume on 2026-10-04: Waiting, day 7 goes out on 2026-10-06.
- Resume on 2026-10-09: Late at the next run, Lena told once.
- Hard bounce on 2026-09-29: Paused, bounced, "Email not delivered" with "Send again". Lena fixes the address and presses it: old request Cancelled, new one Waiting from 2026-09-30, email to the new address.
- Two requests due the same day: one email, two sections, two buttons.
- PP-034 finalised on 2026-10-05: no reminder on 2026-10-06, never Late.
- The seed on 2026-10-12: the jobs send nothing, PP-034 stays Late.

#### Open questions

- Which auto-reply headers do Nordwind's servers send? Ask Nordwind IT.
- Calendar or working days for 3, 7 and 8? Ask Lena.
- Public holidays: default treated as weekdays. Ask Lena.

#### Details

- Analytics: decision_reminder_sent {request_id, reminder_no, merged}; decision_reminder_skipped {request_id, reason}; decision_request_paused {request_id, flag}; decision_request_late {request_id}
- Module: Notifications
- Estimate: L
- Depends on: DR-1, DR-2

---

### DR-5: Waiting on others list, and the measures

One place that shows everything waiting on someone, with the actions on each row, plus the four measures as queries.

#### Build

- A page "Waiting on others" under Pain points with the tabs Open (default), Answered and Cancelled. A manager sees their own requests. "All NVCU", default off, adds the unit's.
- Columns: Request, Owner, Age, Due, State, Reminders, Last owner action, Flag. Late first, then oldest, then request id. Empty cells show "-". Empty list: "Nothing is waiting on anyone."
- Actions by state. Waiting and Late: Open shortlist, Remind now, Cancel, Answer on behalf, and "Send to Petra Ahlers instead" when a handoff exists. Paused: the same plus Resume reminders, and Remind now warns "Jonas Keller appears to be away". Answered and Cancelled: Open shortlist.
- Remind now sends the DR-4 reminder at once, adds 1 to reminders_sent, logs remind_now (a manual chase), is not capped, then reads "Reminded today" for 24 hours.
- Cancel is DR-1's.
- Answer on behalf opens the DR-3 form plus a required note, default "recorded by Lena Hartmann from a call". via on_behalf, no email to the owner, and his link shows "Already answered by Lena Hartmann".
- "Send to Petra Ahlers instead": new owner, fresh link, sent_at and due_at restart, reminders_sent 0, the old owner gets nothing, the old link shows "This request went to Petra Ahlers". With no suggested_user_id, Lena is asked for an email address and the OS stores a contact {name, email} with no login. The same button sits on the shortlist header.
- Event types: created, link_opened, identity_confirmed, not_me, link_renewed, reminder_sent, reminder_skipped, remind_now, paused, resumed, late, handoff, reassigned, answered, cancelled.
- Four queries over requests created in a period, no screen yet. Owner wait: the median days from created to answered over answered requests with rated true, and the share of all requests answered by day 7 with unanswered as misses. First touch: the share answered within one working day. No manual chase: the share of answered requests with no remind_now before the answer. Late rate: the share whose wait (answered minus created, or today minus created while open) is more than 1.2 times the unit's median, or 8 days until the unit has 10 answered.

Not here: the digest email, a measures screen, bulk actions, risk flags.

#### Data

Reassign logs reassigned {from, to} and keeps the old owner in the log. contact {id, name, email}. One open request per shortlist, not per owner.

#### Edge cases

- Two managers act at once: the first wins, the second sees "Already answered".
- A late answer: the row shows "received after shortlist was finalised".
- The suggested person is the sender, or has no email and Lena gives none: button disabled, says why.
- Remind now at 08:59 and the job at 09:00: the automatic reminder is skipped.

#### Done when

- The seed on 2026-10-12: PP-034 shows Jonas Keller, age 13, due 2026-10-06, Late, reminders 2, last owner action "-". PP-040 (Anneke de Vries, age 24, Late) sorts above it.
- Tobias Brandt with All NVCU off: his requests only. On: PP-034 and PP-040 too, and his Remind now is logged under his name.
- Lena presses Remind now on PP-034: reminders 3, remind_now logged, Jonas gets the email, "Reminded today" for 24 hours.
- Answer on behalf on PP-034 with the default note: Answered, via on_behalf, note on the shortlist, no email to Jonas, his link "Already answered by Lena Hartmann", counts as chased.
- The seed read as requests (sent_to_owner_at is created, owner_feedback_at is answered when complete, partial is open, reminders_sent are manual chases): median 7 days over 11 answered, 8 of 13 by day 7, 9 of 11 without a chase, late 3 of 13 (SL-004, PP-034, PP-040).
- No open requests: "Nothing is waiting on anyone."
- Handoff to Petra Ahlers, "Send to Petra Ahlers instead" on 2026-10-01: owner Petra Ahlers, sent_at 2026-10-01, due_at 2026-10-08, reminders 0, Waiting, reassigned logged, Petra gets the email, Jonas gets nothing, his link shows "This request went to Petra Ahlers".
- Handoff to "Henrik Vos" with no user id: Lena is asked for an email address, enters one, the request goes to a new contact, he gets the email.

#### Open questions

- "All NVCU" on by default? Ask Lena.
- Read-only access for Dr. Sabine Koch? Default no. Ask Lena.

#### Details

- Analytics: decision_manager_action {request_id, action, age_days}
- Module: Pain points, Reports
- Estimate: L
- Depends on: DR-3, DR-4
