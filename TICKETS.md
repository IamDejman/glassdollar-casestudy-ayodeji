# Tickets: Business units submit a problem and then disappear

## Why this epic exists

Lena sends Jonas a shortlist and then nothing happens for two weeks. She chases him by email, he can't find his login, and the pain point sits in Shortlist. We want to cut the days between "shortlist sent" and "owner answered".

The fix is one list for Lena of everything still waiting on someone. A decision request is what fills it. Lena asks one question from the shortlist. Jonas gets an email with one button and answers on one page in about three minutes. If he doesn't answer, the system reminds him. If he isn't the right person, he says who is and Lena sends it on with one click.

## Order

Build DR-1 to DR-5 in that order. The acceptance criteria use the sample data at https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/data/sample-data.json, so you can check your work without asking anyone.

## Rules that apply to every ticket

- A request is one question from a manager to one owner about one shortlist. This release has one question: rate this shortlist.
- A request is always in one of five states: Waiting, Paused, Late, Answered or Cancelled. Waiting, Paused and Late count as open. An open request can become Answered or Cancelled, and those two are final.
- A shortlist can have only one open request at a time. Enforce that in the database, not only in the interface.
- Days are calendar days counted from the date the request was sent, in the workspace timezone (Europe/Berlin). A request sent on 29 Sep 2026 reaches day 3 on 2 Oct, is due on 6 Oct at 23:59, and its link expires on 13 Oct at 23:59. Age never resets.
- Scheduled jobs run at 09:00 on weekdays. Anything due on a weekend happens on Monday.
- Anyone who can edit the pain point can send, cancel, remind, answer on behalf or send the request on. The event log records who did what.
- When a ticket says Lena is told, that means one in-app notification and one email. That is the workspace default in the sample data.
- Nothing closes a request on its own. Day 14 only expires the link. A Late request stays in the list until Lena acts on it.
- A pain point is finalised when it leaves the Shortlist stage. From then on reminders stop and the request can no longer go Late, but it stays open so a late answer is still recorded.
- Every ticket writes to one event table with the fields request_id, type, at, actor and payload. DR-5 lists all the event types.
- In the sample data, today is 12 Oct 2026. PP-034 is Jonas Keller's pain point. Lena sent the request on 29 Sep, reminders went out on 2 and 6 Oct, it went Late on 7 Oct, and today is day 13. PP-040 belongs to Anneke de Vries, was sent on 18 Sep, and is open at day 24.

## Left out on purpose

Suggested stage change, editing a request after sending, the closing-the-loop email and the daily digest. The brief says why: https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/PRODUCT-BRIEF.md.

---

### DR-1: Create a decision request from a shortlist

#### Description

Lena has three startups on the PP-034 shortlist and needs Jonas to say which are worth a call. Today she writes an email and hopes. This ticket gives her a button on the shortlist that sends one request, shows her that it is out there, and lets her cancel it.

#### What to build

- A button on the shortlist header, "Ask Jonas Keller to rate this shortlist", shown while the pain point is in the Shortlist stage. The name is the pain point's owner.
- The button opens a dialog with three things: the email the owner will get, an optional note to the owner, and a "Send request" button. The email is DR-2's template with this request filled in. Until DR-2 ships, show the template text. The note is plain text, up to 1,000 characters.
- Send creates the request in the Waiting state and logs a created event. Sending the email itself is DR-2's job.
- While the request is open, the header shows its state and age: "Waiting for Jonas Keller, day 13", "Late: Jonas Keller, day 13" or "Paused: Jonas Keller, day 13". Once answered, DR-3 shows the answers in that spot. Once cancelled, the button comes back.
- A "Cancel request" link next to the state. It asks for confirmation and an optional reason, sets the state to Cancelled and logs a cancelled event. The owner gets no email about it.
- If a startup is added to or removed from the shortlist while a request is open, the header shows "Shortlist changed since the request" with a "Cancel and resend" button. That cancels the open request and reopens the send dialog with the note already filled in.
- Store the request with these fields: id, pain_point_id, shortlist_id, owner_id, sent_by, note, state, sent_at, due_at, link_expires_at, startups_snapshot, reminders_sent, flag and cancelled_reason. The snapshot holds each startup's id, name, description and Lena's comment as they were at send time. DR-3 shows the snapshot, not the live shortlist. A new request starts Waiting, with zero reminders and no flag.

Not in this ticket: other kinds of question, more than one owner on a request, sending to someone other than the owner (see Open questions), and editing a request after it is sent.

#### User stories

- As Lena, I want to ask Jonas for his view with one button, so that I stop writing chasing emails by hand.
- As Lena, I want the shortlist to show that a request is out and how old it is, so that I know whether to wait or step in.
- As Lena, I want to cancel a request, so that a stale question does not reach Jonas after the shortlist changed.
- As Tobias, covering for Lena, I want to see that a request is already out, so that I do not send a second one.

#### Acceptance criteria

- When Lena sends the PP-034 request on 29 Sep 2026, it is Waiting, sent_by is Lena Hartmann, due_at is 6 Oct 2026 at 23:59 Europe/Berlin, link_expires_at is 13 Oct 2026 at 23:59, reminders_sent is 0, and the snapshot holds Pallet.ai, Cargonaut Labs and Parcelgrid with their descriptions and Lena's comments. A created event is logged.
- When Tobias Brandt opens the PP-034 shortlist on 12 Oct 2026, he sees "Late: Jonas Keller, day 13" and no send button.
- When a shortlist has only 2 startups, the button is disabled and reads "Needs 3 to 5 startups".
- When the owner is Henrik Vos, who is not a user in the OS (as on PP-017), the button is disabled and reads "Henrik Vos has no email address in the OS".
- When Lena adds a fourth startup while a request is open, the header shows "Shortlist changed since the request". When she presses "Cancel and resend", the old request is Cancelled, the dialog opens with her note, and Send creates a new Waiting request with four startups in the snapshot.
- When Lena cancels PP-034 with the reason "Jonas is on leave", the state is Cancelled, the reason is stored, a cancelled event is logged, Jonas gets no email, and the send button is back.
- When Lena and Tobias press Send on the same shortlist within a second, one request exists and the other sees "Already sent by Lena Hartmann".
- When PP-034 is finalised on 12 Oct 2026 with the request open, the request stays Late, the header reads "Finalised, answer still open until 13 Oct 2026", and no reminder goes out.

#### Edge cases

- Fewer than 3 or more than 5 startups: the button is disabled and says "Needs 3 to 5 startups".
- The pain point has no owner, the owner is not a user in the OS, or the owner is the person sending: the button is disabled and says which of the three it is.
- Two managers send at the same time: one request is created and the other sees "Already sent by Lena Hartmann".
- A startup's description or Lena's comment is edited after sending: no banner. The owner sees the snapshot.
- The pain point is finalised while the request is open: the header reads "Finalised, answer still open until 13 Oct 2026".

#### Open questions

- Should Lena be able to send to someone other than the pain point's owner? The default here is no: a wrong owner names the right person (DR-3) and Lena sends it on (DR-5). Ask Lena how often the owner is not the right judge.
- Should finalising a pain point cancel its open request? The default here is no, so a late answer is still recorded. Ask Lena.

#### Details

- Analytics: decision_request_created {request_id, startups_count, note_length}; decision_request_cancelled {request_id, age_days, has_reason}
- Module: Pain points
- Estimate: M
- Depends on: none

---

### DR-2: Email the owner with a link that survives scanners

#### Description

Jonas gets one email with one button. He taps it, confirms it is him, and lands on the answer page. Corporate mail scanners open every link before he does, so nothing counts until a person taps "Yes, I am Jonas Keller".

#### What to build

- The email comes from "GlassDollar for Nordwind" at a GlassDollar sending address. Reply-To is Lena's address, so a human reply lands in her inbox. Return-Path is the OS's inbound address, so bounces and auto-replies come back to the OS for DR-4.
- Subject: "PP-034: Lena Hartmann needs your view on 3 startups". The body says who is asking, what for, that it takes about 3 minutes, Lena's note if she left one, and the due date. Then one text button, "Review 3 startups", with the full URL as plain text under it. Startup names never appear in the email.
- The link carries a random token of at least 32 bytes, stored hashed, tied to the request and to the owner's email address, and valid until the link expires.
- Opening the link shows a page with two buttons, "Yes, I am Jonas Keller" and "Not me". The first tap records confirmed_at and starts a session for that request in a cookie that lasts until the link expires. The token itself stays valid, so the same link works from a reminder email, from a second device and on a later day. With a live session, the link goes straight to the answer page.
- "Not me" shows "Thanks, we've told Lena Hartmann" and nothing else. The request and the token are unchanged. Lena is told once per request.
- An expired link shows "This link has expired" with "Request a new link". The new link goes to the same address, lasts 14 days from that moment, and tells Lena "Jonas Keller asked for a new link". It works only while the request is open and at most three times per request. After that the page says "Ask Lena Hartmann for a new link". On an answered request the expired page says "Your answer was sent on 12 Oct 2026". On a cancelled one, "Lena no longer needs this".
- The sending domain needs SPF, DKIM and DMARC. Add a line to the customer setup checklist asking IT to allowlist the sender.
- Store the link with request_id, token_hash, bound_email, expires_at, confirmed_at and renewals. Log link_opened (with likely_bot), identity_confirmed, not_me and link_renewed. An open before any tap is likely_bot when it comes from a known scanner range, or lands within 60 seconds of sending with no tap after it.

Not in this ticket: buttons inside the email body, Teams or Slack, a sending domain per customer.

#### User stories

- As Jonas, I want to open the request from my phone without a login, so that I can answer in the two minutes I have between meetings.
- As Jonas, I want the link to still work on a later day and on my laptop, so that I am not locked out if I did not finish the first time.
- As Lena, I want to know when someone other than Jonas opened the link, so that I can check with him.
- As Nordwind IT, I want nothing recorded when our mail scanner opens the link, so that scanning cannot fake an answer.

#### Acceptance criteria

- Jonas's email for PP-034 contains "Review 3 startups", "about 3 minutes" and the due date 6 Oct 2026, and does not contain Pallet.ai, Cargonaut Labs or Parcelgrid.
- When a scanner fetches a new link three times, the request is still Waiting, confirmed_at is empty and no owner activity is logged.
- When Jonas taps "Yes, I am Jonas Keller" on his phone on 1 Oct 2026 and opens the same link on his laptop on 2 Oct, the laptop shows the identity page once and then the answer page. confirmed_at stays 1 Oct.
- When Jonas taps the button on 13 Oct 2026, the page opens. On 14 Oct he sees "This link has expired" with "Request a new link".
- When he presses "Request a new link" on 14 Oct, a new link arrives at his address, expires on 28 Oct 2026 at 23:59, renewals is 1, Lena is told once, and the old link stays expired.
- When he asks a fourth time on the same request, the page reads "Ask Lena Hartmann for a new link" and nothing is sent.
- When Anneke de Vries taps "Not me" on a forwarded PP-034 email, she sees no startup and Lena is told "Someone other than Jonas Keller opened the PP-034 link".
- When Lena has cancelled PP-034 and Jonas opens the link, he sees "Lena no longer needs this" and no identity page.
- When images are blocked, the button is readable text and the full URL is visible under it.

#### Edge cases

- A scanner fetches the link: nothing is recorded as owner activity.
- A sandbox presses the button: confirmed_at is set and the event is logged. Nothing more can happen without choices, and the measures count answers, not taps.
- Images are blocked, as in Outlook desktop: the button is text in a table cell and the plain URL under it still works.
- The email was forwarded: this is what "Not me" is for. The tap is what stops a colleague answering as Jonas.
- The request was sent on to someone else (DR-5): the old link shows "This request went to Petra Ahlers" before any tap.
- The owner's address changes in the OS after sending: the old link keeps working and the new address is used from the next email.

#### Open questions

- Does Nordwind force SSO on links from outside SaaS tools? Ask Nordwind IT.
- Will Nordwind IT allowlist one shared GlassDollar sending domain, or does each customer need its own? Ask Nordwind IT.
- German or English? The default here is English until the OS has a workspace language. Ask Lena.

#### Details

- Analytics: decision_link_opened {request_id, likely_bot}; decision_identity_confirmed {request_id, days_since_sent}; decision_link_renewed {request_id, renewals}
- Module: Notifications, Shared
- Estimate: L
- Depends on: DR-1

---

### DR-3: Let the owner answer on one page, and land the answer on the shortlist

#### Description

After the identity tap, Jonas lands on one page with the three startups and three choices each. He picks, adds a note if he wants, and sends. Lena sees the answers on the shortlist and gets one notification. Nobody asks twice.

#### What to build

- One page, built for a phone first, with no navigation, behind the DR-2 session. It shows the PP-034 title, Lena's note, and one card per startup from the snapshot: the name, the description, Lena's comment if she left one, the three choices Interested, Not now and Need more info, and a comment box. Under the cards sit a "None of these fit" checkbox with its own comment box, which replaces the per-startup choices, and a "Send to Lena" button.
- Need more info and None of these fit require a comment. Interested and Not now take an optional one. Send is always enabled. If something is missing, the page says what and moves focus to it. Nothing is sent until it is complete.
- After sending, the page reads "Sent to Lena Hartmann." with a summary of the choices. Opening a still-valid link again shows the answer read-only.
- On the shortlist, each startup shows the choice with who answered, when and how. Lena is told once: "Jonas Keller answered PP-034", or "Jonas Keller needs more information on 3 startups" when nothing was rated.
- After an answer, the shortlist button comes back as "Ask Jonas Keller again". That is how Lena follows up on Need more info: she sends a new request with a note once she has the information. The information itself goes by email or a call.
- Under the Send button, "Someone else should answer this". It asks for a name and suggests users of the OS as he types, but any name is allowed. "Tell Lena" keeps the request open, tells Lena "Jonas Keller says Petra Ahlers should answer PP-034", and shows "Sent to Lena Hartmann. She'll send PP-034 to Petra Ahlers." If Jonas answers afterwards anyway, the answer wins and the handoff is cleared.
- Store the answer with request_id, one item per startup (startup_id, choice, comment), none_fit, none_fit_comment, submitted_at, via (magic_link or on_behalf), late and rated. Sending sets the request to Answered. late is true when the answer arrives after the pain point was finalised. rated is true when at least one startup is Interested or Not now, or None of these fit is ticked. Three times Need more info means rated is false. Store the handoff with request_id, suggested_name, suggested_user_id, by and at. Log answered (with via, rated and late) and handoff (with whether the name matched a user).

Not in this ticket: editing after sending, a suggested stage change, SourceAI profiles, attachments.

#### User stories

- As Jonas, I want to rate three startups on my phone in a couple of minutes, so that Lena gets my view without a meeting.
- As Jonas, I want to say that someone else should answer, so that the request stops landing on me when it is not mine.
- As Lena, I want the answers to appear on the shortlist with one notification, so that I do not copy anything out of an email.
- As Lena, I want a Need more info answer to tell me what is missing, so that I can send the information and ask again.

#### Acceptance criteria

- After the identity tap, Jonas sees Pallet.ai, Cargonaut Labs and Parcelgrid with three choices each, Lena's note, and no other navigation.
- When Jonas has rated 2 of 3, or ticked "None of these fit" without a comment, or set Cargonaut Labs to Need more info without a comment, and presses Send, nothing is sent. The page says what is missing and focus moves to it.
- When Jonas sets Pallet.ai to Interested and the others to Not now and sends on 12 Oct 2026, the request is Answered, via is magic_link, rated is true, the shortlist shows the three choices with "Jonas Keller, 12 Oct 2026, via link", Lena is told once, and the button reads "Ask Jonas Keller again".
- When all three are Need more info, each with a comment, the request is Answered, rated is false, and Lena's notification reads "Jonas Keller needs more information on 3 startups".
- When Lena finalised PP-034 on 12 Oct 2026 and Jonas sends on 13 Oct, the answer is stored with late true and the shortlist shows "received after shortlist was finalised".
- When Jonas types "Pet" in the handoff field and picks Petra Ahlers, suggested_user_id is U-009. When he types "Henrik Vos" instead, the name is stored with no user id.
- When Jonas sends a handoff on 1 Oct 2026, the request is still Waiting, Lena is told once, and reopening the link shows "Sent to Lena Hartmann" with Petra's name.
- When Jonas sent that handoff on 1 Oct and rates all three on 2 Oct, the request is Answered, the handoff is cleared, and the list no longer offers "Send to Petra Ahlers instead".
- When Lena answers on behalf at 10:00 and Jonas presses Send at 10:01, Jonas sees "Already answered by Lena Hartmann" and Lena's answer stands.

#### Edge cases

- Two tabs submit: the second one sees "Already sent".
- The page is opened after Lena cancelled: "Lena no longer needs this". After she sent it on: "This request went to Petra Ahlers". After she answered on behalf: "Already answered by Lena Hartmann".
- The snapshot has five startups: five cards, same rules.
- Comments are capped at 1,000 characters, with a counter from 100 left.

#### Open questions

- Should the confirmation promise a reply time, say "within 3 working days"? Ask Lena.
- Should a Need more info answer keep the request open until Lena replies? The default here is no: it is Answered with rated false, and Lena sends a new request once she has the information. Ask Lena.

#### Details

- Analytics: decision_answer_submitted {request_id, days_since_sent, via, rated, late, seconds_on_page}; decision_handoff {request_id, matched}
- Module: Pain points, Shared
- Estimate: L
- Depends on: DR-1, DR-2

---

### DR-4: Remind on days 3 and 7, pause when the owner is away, mark Late on day 8

#### Description

Today Lena is the reminder system. This ticket moves that into the OS: two automatic reminders, a pause when Jonas's out-of-office replies, and a clear Late mark on day 8 so Lena knows when to step in herself.

#### What to build

- On day 3 and day 7, send the first email again with "Reminder 1 of 2" or "Reminder 2 of 2" at the top and the same link. Subject: "Reminder: PP-034, Lena Hartmann needs your view by 6 Oct 2026". Never more than two automatic reminders.
- Skip a reminder when the owner confirmed identity or opened the page in the last 24 hours, or when Lena pressed Remind now (DR-5) in the last 24 hours.
- On day 8 with no answer, set the state to Late and tell Lena "Jonas Keller has not answered PP-034 (day 8)".
- When an auto-reply comes back, set flag to away and state to Paused, and stop both the reminders and the Late transition. Lena sees "Jonas Keller appears to be away" with a "Resume reminders" button. An auto-reply is any of these: an Auto-Submitted header other than "no", an X-Auto-Response-Suppress header, Precedence auto_reply, or a subject starting with "Automatic reply", "Automatische Antwort", "Out of office", "Abwesenheit" or "Abwesenheitsnotiz", in any letter case.
- When a hard bounce comes back, set flag to bounced and state to Paused. Lena sees "Email not delivered" with "Send again", which is DR-1's cancel and resend to the owner's current address. Soft bounces are retried by the mail provider and not shown.
- Resume reminders puts the state back to Waiting, or straight to Late if day 8 has passed, and tells Lena once. Reminders missed while paused are not sent. The next one due goes out normally.
- When one owner has several reminders due on the same day, send one email with a section and a button per request.
- The jobs are idempotent: they check the state and the event log again right before sending. A reminder goes out only when the state is Waiting, fewer than two have been sent, and the pain point is still in Shortlist. Fields: reminders_sent, last_owner_activity_at, flag (none, away or bounced), paused_at and resumed_at. Log reminder_sent (with the number and whether it was merged), reminder_skipped (with the reason), paused (with the flag), resumed and late.

Not in this ticket: reading return dates out of auto-replies, drafting a note to the owner's manager, letting customers change the cadence, public holidays.

#### User stories

- As Lena, I want the OS to remind Jonas twice, so that I only chase when the automatic reminders failed.
- As Lena, I want reminders to pause when Jonas is away, so that he does not come back to three nagging emails.
- As Lena, I want to be told on day 8, so that I know when to pick up the phone.
- As Jonas, I want one email even when two requests are due, so that I am not spammed.

#### Acceptance criteria

- When a request sent on 29 Sep 2026 gets no answer, reminders go out on 2 Oct and 6 Oct at 09:00, reminders_sent is 2, and on 7 Oct the state is Late and Lena gets one in-app notification and one email.
- When Jonas opened the page on 2 Oct at 08:10, the 09:00 job sends nothing that day and the day 7 reminder still goes out on 6 Oct.
- When an auto-reply with "Auto-Submitted: auto-replied" arrives on 3 Oct, the day 7 job sends nothing, the state is Paused, and Lena sees "Jonas Keller appears to be away".
- When an auto-reply with the subject "Automatische Antwort: PP-034: Lena Hartmann needs your view on 3 startups" arrives on 3 Oct, the outcome is the same.
- When Lena presses Resume reminders on 4 Oct, the state is Waiting and the day 7 reminder goes out on 6 Oct.
- When Lena presses Resume reminders on 9 Oct, the next job run sets the state to Late and Lena is told once, not twice.
- When a hard bounce comes back on 29 Sep, the state is Paused with flag bounced and Lena sees "Email not delivered" with "Send again". When she corrects the address and presses it, the old request is Cancelled, a new one is Waiting from 30 Sep, and the email goes to the new address.
- When Jonas has two open requests with reminders due the same day, he gets one email with two sections and two buttons.
- When PP-034 is finalised on 5 Oct with the request open, no reminder goes out on 6 Oct and the request never becomes Late.
- On the sample data on 12 Oct 2026, the jobs send nothing and PP-034 stays Late.

#### Edge cases

- The owner answers between the job's check and the send: the send is aborted.
- An auto-reply arrives after the answer: ignored. On a Late request: the flag is stored, the state stays Late, and Lena gets no second notification.
- Resume before day 7: the day 7 reminder still goes out. A day 3 reminder missed while paused does not.
- The job runs twice, for example after a retry: one email, because the second run finds the reminder_sent event.
- Day 3 falls on a Saturday: the reminder goes out on Monday at 09:00.

#### Open questions

- Which auto-reply headers do Nordwind's mail servers send? Ask Nordwind IT.
- Calendar days or working days for 3, 7 and 8? Ask Lena.
- Public holidays: the default here is to treat them as weekdays, since the OS has no holiday calendar. Ask Lena.

#### Details

- Analytics: decision_reminder_sent {request_id, reminder_no, merged}; decision_reminder_skipped {request_id, reason}; decision_request_paused {request_id, flag}; decision_request_late {request_id}
- Module: Notifications
- Estimate: L
- Depends on: DR-1, DR-2

---

### DR-5: Waiting on others list, and the measures

#### Description

Lena needs one place that shows everything still waiting on someone and lets her act from there. The product team needs to know whether any of this worked, so the same ticket adds the four measures as queries over the event log.

#### What to build

- A page "Waiting on others" under Pain points, with the tabs Open (default), Answered and Cancelled. A manager sees their own requests. An "All NVCU" switch, off by default, adds every request in the unit.
- Columns: Request, Owner, Age, Due, State, Reminders, Last owner action and Flag. Late rows come first, then the oldest, then by request id. Empty cells show "-". When nothing is open: "Nothing is waiting on anyone."
- The actions on a row depend on its state. Waiting and Late rows offer Open shortlist, Remind now, Cancel, Answer on behalf, and "Send to Petra Ahlers instead" when the owner named someone. Paused rows offer the same plus Resume reminders, and their Remind now carries the warning "Jonas Keller appears to be away". Answered and Cancelled rows offer Open shortlist only.
- Remind now sends the DR-4 reminder at once, adds one to reminders_sent, and logs a remind_now event, which counts as a manual chase. It is not limited by the two automatic reminders. Afterwards it reads "Reminded today" for 24 hours.
- Cancel is DR-1's cancel, with the optional reason.
- Answer on behalf opens the DR-3 form with a required note, defaulting to "recorded by Lena Hartmann from a call". The answer is stored with via on_behalf, the owner gets no email, and his link then shows "Already answered by Lena Hartmann".
- "Send to Petra Ahlers instead" moves the request to that person with a fresh link. sent_at and due_at restart, reminders_sent goes back to zero, the old owner gets no email, and the old link shows "This request went to Petra Ahlers". If the handoff did not match a user, Lena is asked for an email address and the OS stores a contact with that name and address, with no login. The same button sits on the shortlist header.
- The event log holds these types: created, link_opened, identity_confirmed, not_me, link_renewed, reminder_sent, reminder_skipped, remind_now, paused, resumed, late, handoff, reassigned, answered and cancelled. A reassigned event records the old and the new owner.
- Four queries over the requests created in a period, with no screen yet. Owner wait time is the median days from created to answered over answered requests with rated true, plus the share of all requests answered by day 7, with unanswered counted as misses. First touch is the share answered within one working day. No manual chase is the share of answered requests with no remind_now before the answer. Late rate is the share whose wait (answered minus created, or today minus created while still open) is more than 1.2 times the unit's median, or more than 8 days until the unit has ten answered requests.

Not in this ticket: the digest email, a measures screen, bulk actions, risk flags.

#### User stories

- As Lena, I want one list of everything waiting on someone, so that nothing I own stalls without me knowing.
- As Lena, I want to remind, cancel or answer on behalf from the list, so that I do not open each pain point to act.
- As Lena, I want to send a request on to the person Jonas named with one click, so that the right person gets it without a new email from me.
- As the product team, we want the four measures as queries, so that after a quarter we can tell whether the wait got shorter.

#### Acceptance criteria

- On the sample data on 12 Oct 2026, Lena opens the list and PP-034 shows Jonas Keller, age 13, due 6 Oct 2026, Late, 2 reminders and last owner action "-". PP-040 (Anneke de Vries, age 24, Late) sorts above it.
- When Tobias Brandt opens the list with "All NVCU" off, he sees only his requests. When he turns it on, he sees PP-034 and PP-040 too, and his Remind now is logged under his name.
- When Lena presses Remind now on PP-034, reminders become 3, a remind_now event is logged, Jonas gets the reminder email, and the button reads "Reminded today" for 24 hours.
- When Lena answers on behalf for PP-034 with the default note, the request is Answered with via on_behalf, the shortlist shows the note, Jonas gets no email, his link shows "Already answered by Lena Hartmann", and the request counts as manually chased.
- When the sample data is read as requests (sent_to_owner_at is created, owner_feedback_at is answered when the status is complete, partial counts as open, and reminders_sent are manual chases because every reminder before this release was Lena's), the queries return a median wait of 7 days over 11 answered requests, 8 of 13 answered by day 7, 9 of 11 answered without a chase, and a late rate of 3 of 13 (SL-004, PP-034 and PP-040).
- When no requests are open, Lena sees "Nothing is waiting on anyone."
- When PP-034 has a handoff to Petra Ahlers and Lena presses "Send to Petra Ahlers instead" on 1 Oct 2026, the owner is Petra Ahlers, sent_at is 1 Oct, due_at is 8 Oct, reminders_sent is 0, the state is Waiting, a reassigned event is logged, Petra gets the DR-2 email, Jonas gets nothing, and his link shows "This request went to Petra Ahlers".
- When the handoff names "Henrik Vos" with no user id and Lena presses "Send to Henrik Vos instead", she is asked for an email address. When she enters one, the request goes to a new contact with that name and address, and he gets the email.

#### Edge cases

- Two managers act on the same request at the same time: the first wins and the second sees "Already answered".
- A late answer: the row shows "received after shortlist was finalised".
- The suggested person is the sender, or has no email and Lena gives none: the button is disabled and says why.
- Lena presses Remind now at 08:59 and the job runs at 09:00: the automatic reminder is skipped.
- The new owner may already have another request open. The limit is one open request per shortlist, not per owner.

#### Open questions

- Should "All NVCU" be on by default? Ask Lena.
- Should Dr. Sabine Koch see the list read-only? The default here is no. Ask Lena.

#### Details

- Analytics: decision_manager_action {request_id, action, age_days}
- Module: Pain points, Reports
- Estimate: L
- Depends on: DR-3, DR-4
