# Tickets: Business units submit a problem and then disappear

## Why this epic exists

Lena sends Jonas a shortlist and then nothing happens for two weeks. She chases him by email, he can't find his login, and the pain point sits in Shortlist. We want to cut the time between "shortlist sent" and "owner answered".

The fix starts with one list for Lena of everything that is still waiting on someone, on whom and for how long. A decision request is what fills it. Lena asks one question from the shortlist. Jonas gets an email with one button, answers on one page, and is done in about three minutes. If he doesn't answer, the system reminds him so Lena doesn't have to. If he isn't the right person, he says who is and Lena sends it on with one click.

Email is the channel because every owner has it. The ask is what changes: three ratings by a date, with Lena told on day 8.

## Order

Build DR-1 to DR-5 in that order. Each ticket stands on its own. The acceptance criteria use the sample data at https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/data/sample-data.json, so you can check your work without asking anyone.

## Terms used in all five tickets

A request is one question from an innovation manager to one owner about one shortlist. This release has one kind of question: rate this shortlist.

Days are counted from the day the request was sent, as calendar days, in the workspace timezone (Europe/Berlin for Nordwind). Day 3 is three calendar days after sending. The jobs that send reminders run at 09:00 on weekdays, so if day 3 falls on a weekend the reminder goes out on Monday.

The owner has until the end of day 7 to answer. The link in the email works until the end of day 14. "Finalised" means the pain point has moved past Shortlist, so an answer no longer changes anything.

A request is always in one of five states:

- Waiting: sent, no answer yet
- Paused: an out-of-office reply or a bounce came back, so reminders are on hold
- Late: day 8 arrived and there is still no answer
- Answered
- Cancelled

Reminders do not change the state. They add one to `reminders_sent` and that is all.

Age is calendar days since sending. Nothing resets it, so the number Lena sees is the real wait.

A notification to Lena is one in-app notification and one email, following the workspace default in the sample data (notification_settings_default: every event emails its recipient at once, no digest). Nothing closes a request on its own. A Late request stays in the list until Lena cancels it, answers on behalf or sends it on. Day 14 only expires the link.

The seed data tells one story. PP-034 is Jonas's pain point about manual load planning for outbound trucks. Lena sent him the request on 29 September 2026. Reminders went out on 2 and 6 October. It went Late on 7 October. On 12 October it is day 13 and still unanswered. Lena's note to Dr. Sabine Koch on 7 October went by email outside the OS, so the system knows nothing about it.

## Left out on purpose

Suggested stage change, editing a request after it is sent, the closing-the-loop email and the daily digest are Should or Later. The brief says why: https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/PRODUCT-BRIEF.md.

---

### DR-1: Create a decision request from a shortlist

Lena has three startups on the PP-034 shortlist and needs Jonas to say which are worth a call. Today she writes an email and hopes. This ticket gives her a button on the shortlist that sends one request, shows her that it is out there, and lets her cancel it.

#### What to build

- In the header of the shortlist page of a pain point in the Shortlist stage, a button "Ask Jonas Keller to rate this shortlist". The name is the pain point's owner. Anyone who can edit the pain point can press it.
- The button opens a dialog with the email the owner will get (the DR-2 template with this request's details filled in; until DR-2 ships, the dialog shows the template text), an optional plain-text note to the owner of up to 1,000 characters with line breaks kept, and "Send request".
- Send creates the request in Waiting with sent_at now, due_at the end of day 7 and link_expires_at the end of day 14, both in the workspace timezone, and stores a snapshot of the shortlist. Sending the email is DR-2's job. DR-1 writes the request and the created event.
- One open request per shortlist. Open means Waiting, Late or Paused. Enforce it in the database, not only in the UI.
- While a request is open the header shows its state and age: "Waiting for Jonas Keller, day N", "Late: Jonas Keller, day N" or "Paused: Jonas Keller, day N". N is calendar days since sent_at. Once Answered, DR-3 puts the answers in its place. Once Cancelled, the button comes back.
- Next to the state, "Cancel request". It asks "Cancel the request to Jonas Keller?" with an optional reason, sets the state to Cancelled, writes the cancelled event and brings the button back. The owner gets no email. If he opens the link later, DR-3 tells him Lena no longer needs this. DR-5 puts the same action on the list.
- When a startup is added to or removed from the shortlist while a request is open, the header shows "Shortlist changed since the request" with "Cancel and resend". It cancels the open request and opens the send dialog with the note prefilled.

#### Not in this ticket

Other kinds of question. More than one owner on a request. Sending to someone other than the pain point's owner (see Open questions). Editing a request after it's sent: cancel it and send a new one. The email itself (DR-2).

#### Data

request {id, pain_point_id, shortlist_id, owner_id, sent_by, note, state, sent_at, due_at, link_expires_at, startups_snapshot[], reminders_sent, flag, cancelled_reason}. state starts at Waiting, reminders_sent at 0 and flag at none. startups_snapshot holds {startup_id, name, description, manager_comment} for each startup as it was at send time. DR-3 shows the snapshot, not the live shortlist.

event {request_id, type, at, actor, payload}. DR-1 creates the table and writes created {startups_count, note_length} and cancelled {reason}. Later tickets add their own types and DR-5 lists them all.

State changes: Waiting, Late or Paused can go to Answered or Cancelled. Answered and Cancelled are final. Late and Paused come from DR-4.

Days count from the calendar date of sending. A request sent at 23:30 on 29 September has day 1 on 30 September and due_at on 6 October at 23:59.

#### Edge cases

- Fewer than 3 or more than 5 startups: the button is disabled and says "Needs 3 to 5 startups".
- The pain point has no owner, the owner is not a user in the OS and so has no email address, or the owner is the sender: the button is disabled and says which of the three it is.
- Two managers press Send on the same shortlist at the same time: one request is created and the other sees "Already sent by Lena Hartmann".
- A startup's description or Lena's comment is edited after sending: no banner. The owner sees the snapshot.
- The pain point moves past Shortlist while a request is open: the request stays open until link_expires_at so a late answer is still recorded (DR-3). The header shows "Finalised, answer still open until 2026-10-13" and DR-4 sends no more reminders.
- The request is cancelled while the owner has the answer page open: his Send gets "Lena no longer needs this" (DR-3).

#### Acceptance criteria

- PP-034 has 3 startups and Lena sends on 2026-09-29 at any time of day. The request is Waiting, sent_by is Lena Hartmann, due_at is 2026-10-06 23:59 Europe/Berlin, link_expires_at is 2026-10-13 23:59, reminders_sent is 0, flag is none, startups_snapshot holds Pallet.ai, Cargonaut Labs and Parcelgrid with their descriptions and Lena's comments, and a created event is logged.
- That request is still open and Tobias Brandt opens the shortlist on 2026-10-12. He sees "Late: Jonas Keller, day 13" and no send button.
- A shortlist has 2 startups. The button is disabled and says "Needs 3 to 5 startups".
- A pain point in Shortlist whose owner is Henrik Vos, who is not a user in the OS (as on PP-017). The button is disabled and says "Henrik Vos has no email address in the OS".
- A request is out and Lena adds a fourth startup. The header shows "Shortlist changed since the request" with "Cancel and resend". She presses it: the old request is Cancelled with a cancelled event, the dialog opens with her note prefilled, and Send creates a new Waiting request with 4 startups in the snapshot and reminders_sent 0.
- Lena edits Pallet.ai's description while a request is open. No banner, and the snapshot is unchanged.
- Lena presses "Cancel request" on PP-034 with the reason "Jonas is on leave". The state is Cancelled, the reason is stored, cancelled is logged, Jonas gets no email, and the send button is back.
- Lena and Tobias press Send on the same shortlist within a second. One request exists and the other sees "Already sent by Lena Hartmann".
- Lena moves PP-034 past Shortlist on 2026-10-12 with the request open. The request stays Late, the header shows "Finalised, answer still open until 2026-10-13", and no reminder goes out.

#### Details

- Analytics: decision_request_created {request_id, startups_count, note_length}; decision_request_cancelled {request_id, age_days, has_reason}
- Open questions: should Lena be able to send to someone other than the pain point's owner? The default here is no, and a wrong owner says who is right (DR-3), which Lena acts on (DR-5). Ask Lena how often the owner is not the right judge. Should a request be cancelled automatically when the pain point leaves Shortlist? The default here is no, so a late answer is still recorded. Ask Lena.
- Module: Pain points
- Estimate: M
- Depends on: none

---

### DR-2: Email the owner with a link that survives scanners

Jonas gets one email with one button. He taps it, confirms it's him, and he's on the answer page. Corporate mail scanners open every link before he does, so the link has to survive that: nothing counts until a person taps "Yes, I am Jonas Keller".

#### What to build

- The email comes from "GlassDollar for Nordwind" at a GlassDollar sending address. Reply-To is Lena's address, so a human reply lands in her inbox. Return-Path is the OS's inbound address, so bounces and auto-replies come back to the OS (DR-4). Subject: "PP-034: Lena Hartmann needs your view on 3 startups". The body says who is asking, what for, that it takes about 3 minutes, Lena's note if she left one, and the due date. Then one text button, "Review 3 startups", with the full URL as plain text under it. Startup names never appear in the email.
- The link carries a random token of at least 32 bytes, stored hashed, tied to the request and to the owner's email address, valid until link_expires_at. Opening it (GET) shows a page with "Yes, I am Jonas Keller" and "Not me". The tap (POST) sets confirmed_at the first time and starts a session for that request in a cookie that lasts until link_expires_at. The token stays valid until then, so the same link works from a reminder, from a second device and on a later day. With a live session the link goes straight to the answer page.
- "Not me" shows "Thanks, we've told Lena Hartmann" and nothing else. The request and the token are unchanged. Lena is told once per request.
- On an expired link: "This link has expired" and "Request a new link". The new link goes to the same address, lasts 14 days from now, and tells Lena "Jonas Keller asked for a new link". It works only while the request is open, at most 3 times per request. After that the page says "Ask Lena Hartmann for a new link". On an answered request the expired page says "Your answer was sent on 12 October" and shows nothing else. On a cancelled one, "Lena no longer needs this".
- The sending domain needs SPF, DKIM and DMARC. Add a line to the customer setup checklist asking IT to allowlist the sender.

#### Not in this ticket

Buttons inside the email body. Teams or Slack. A sending domain per customer. Passing the request to someone else is DR-3 (the owner names who) and DR-5 (Lena sends it on).

#### Data

link {request_id, token_hash, bound_email, expires_at, confirmed_at, renewals}. An open before the tap is logged with likely_bot true when it comes from a known scanner range, or lands within 60 seconds of sending with no tap after it. Events: link_opened {likely_bot}, identity_confirmed, not_me, link_renewed {renewals}.

#### Edge cases

- A scanner fetches the link: nothing is recorded as owner activity.
- A sandbox presses the button: confirmed_at is set and identity_confirmed is logged. Nothing more can happen without choices, and the measures count answers, not taps.
- Images are blocked (Outlook desktop): the button is text in a table cell, and the plain URL under it still works.
- The email was forwarded: this is what "Not me" is for. The tap is what stops a colleague answering as Jonas.
- Jonas taps on his phone, then opens the reminder on his laptop: the laptop shows the identity page once, then the answer page.
- The request was sent on to someone else (DR-5): the old link shows "This request went to Petra Ahlers" before any tap.
- The owner's address changes in the OS after sending: the old link keeps working. The new address is used from the next email.

#### Acceptance criteria

- Lena sends PP-034. Jonas's email contains "Review 3 startups", "about 3 minutes" and "2026-10-06", and does not contain "Pallet.ai", "Cargonaut Labs" or "Parcelgrid".
- A scanner fetches a new link three times. The request is still Waiting, confirmed_at is empty and no owner activity is logged.
- Jonas taps "Yes, I am Jonas Keller" on his phone on 2026-10-01 and opens the same link on his laptop on 2026-10-02. The laptop shows the identity page, then the answer page, and confirmed_at stays 2026-10-01.
- Jonas taps "Yes, I am Jonas Keller" on 2026-10-13 and the page opens. On 2026-10-14 he sees "This link has expired" with "Request a new link".
- Jonas presses "Request a new link" on 2026-10-14. A new link arrives at his address with expires_at 2026-10-28 23:59, renewals is 1, Lena is told once, and the old link says "This link has expired".
- A fourth "Request a new link" on the same request shows "Ask Lena Hartmann for a new link" and sends nothing.
- Anneke de Vries gets a forwarded PP-034 email and taps "Not me". She sees no startup, and Lena is told "Someone other than Jonas Keller opened the PP-034 link".
- Lena cancelled PP-034 and Jonas opens the link. He sees "Lena no longer needs this" and no identity page.
- Images are blocked. The button is readable text and the full URL is visible under it.

#### Details

- Analytics: decision_link_opened {request_id, likely_bot}; decision_identity_confirmed {request_id, days_since_sent}; decision_link_renewed {request_id, renewals}
- Open questions: does Nordwind force SSO on links from outside SaaS tools? Ask Nordwind IT. Will Nordwind IT allowlist a shared GlassDollar sending domain, or does it need one per customer? Ask Nordwind IT. German or English? The default here is English until the OS has a workspace language. Ask Lena.
- Module: Notifications, Shared
- Estimate: L
- Depends on: DR-1

---

### DR-3: Let the owner answer on one page, and land the answer on the shortlist

After the identity tap, Jonas lands on one page with the three startups and three choices each. He picks, adds a note if he wants, and sends. Lena sees the answers on the shortlist and gets one notification. Nobody asks twice.

#### What to build

- One page, built for a phone first, with no navigation. It needs the DR-2 session; without one the link shows the identity page first. It shows the PP-034 title, Lena's note, and one card per startup from the snapshot: the name, the description, Lena's comment if she left one, three choices (Interested, Not now, Need more info) and a comment box. Under the cards, a "None of these fit" checkbox with a comment box that replaces the per-startup choices, and a "Send to Lena" button.
- Need more info and "None of these fit" require a comment. Interested and Not now take an optional one. Send is always enabled. If a startup has no choice, or a required comment is empty, the page says what is missing and moves focus to it. Nothing is sent until it's complete.
- After sending: "Sent to Lena Hartmann." with a summary of the choices. Opening the link again while it is valid shows the answer read-only.
- On the shortlist, each startup shows the choice with who answered, when, and how. Lena gets one notification: "Jonas Keller answered PP-034", or "Jonas Keller needs more information on 3 startups" when nothing was rated. After an answer the shortlist button comes back as "Ask Jonas Keller again", so the follow-up to Need more info is a new request with Lena's note. The information itself goes by email or a call. The OS records the ask and the answer.
- Under the Send button, "Someone else should answer this". It asks for a name and suggests users of the OS as he types. He can pick one or type any name. "Tell Lena" sends it. The request stays open, Lena gets one notification, "Jonas Keller says Petra Ahlers should answer PP-034", and the page says "Sent to Lena Hartmann. She'll send PP-034 to Petra Ahlers." If he answers afterwards anyway, the answer wins and the handoff is cleared.

#### Not in this ticket

Editing after sending. Suggested stage change. SourceAI profiles. Attachments. Keeping a Need more info request open (see Open questions).

#### Data

answer {request_id, items[{startup_id, choice, comment}], none_fit, none_fit_comment, submitted_at, via (magic_link | on_behalf), late, rated}. Sending sets the request to Answered. late is true when the answer arrives after the shortlist was finalised. rated is true when at least one startup is Interested or Not now, or none_fit is ticked. All three on Need more info means rated is false. handoff {request_id, suggested_name, suggested_user_id, by, at}. A handoff does not change the state, and an answer clears it. Events: answered {via, rated, late}, handoff {suggested_name, matched}.

#### Edge cases

- Two tabs submit: the second one sees "Already sent".
- The page is opened after the request was cancelled: "Lena no longer needs this".
- The page is opened after Lena sent the request on (DR-5): "This request went to Petra Ahlers".
- Lena answered on behalf (DR-5) while Jonas had the page open: his Send gets "Already answered by Lena Hartmann".
- Jonas sent a handoff, then answers himself: the answer is stored, the handoff is cleared, and Lena's "Send to Petra Ahlers instead" disappears.
- The snapshot has 5 startups: five cards, same rules.
- Comments are capped at 1,000 characters, with a counter from 100 left.

#### Acceptance criteria

- Jonas has confirmed identity. The page shows Pallet.ai, Cargonaut Labs and Parcelgrid with three choices each, Lena's note, and no other navigation.
- Jonas has rated 2 of 3, or ticked "None of these fit" without a comment, or set Cargonaut Labs to Need more info without a comment, and presses "Send to Lena". Nothing is sent, the page says what is missing and focus moves to it.
- Jonas sets Pallet.ai to Interested and the others to Not now, and sends on 2026-10-12. The request is Answered, via is magic_link, rated is true, the shortlist shows the three choices with "Jonas Keller, 2026-10-12, via link", Lena gets one notification, and the shortlist button reads "Ask Jonas Keller again".
- All three are Need more info, each with a comment. The request is Answered, rated is false, and Lena's notification says "Jonas Keller needs more information on 3 startups".
- Lena finalised the shortlist on 2026-10-12 and Jonas sends on 2026-10-13. The answer is stored with late true and the shortlist shows "received after shortlist was finalised".
- Jonas types "Pet" in the handoff box and picks Petra Ahlers. suggested_user_id is U-009. He types "Henrik Vos" instead: suggested_name is stored and suggested_user_id is empty.
- Jonas presses "Someone else should answer this", picks Petra Ahlers and presses "Tell Lena" on 2026-10-01. The request is still Waiting, handoff holds her name, Lena gets one notification, and opening the link again shows "Sent to Lena Hartmann" with her name.
- Jonas sent that handoff on 2026-10-01 and rates all three on 2026-10-02. The request is Answered, the handoff is cleared, and the list shows no "Send to Petra Ahlers instead".
- Lena answers on behalf at 10:00 and Jonas presses Send at 10:01. Jonas sees "Already answered by Lena Hartmann" and the stored answer is Lena's.

#### Details

- Analytics: decision_answer_submitted {request_id, days_since_sent, via, rated, late, seconds_on_page}; decision_handoff {request_id, matched}
- Open questions: should the confirmation promise a reply time, say "within 3 working days"? Ask Lena. Should a Need more info answer keep the request open until Lena replies? The default here is no: it is Answered with rated false, and Lena sends a new request once she has the information. Ask Lena.
- Module: Pain points, Shared
- Estimate: L
- Depends on: DR-1, DR-2

---

### DR-4: Remind on days 3 and 7, pause when the owner is away, mark Late on day 8

Today Lena is the reminder system. This ticket moves that into the OS: two automatic reminders, a pause when Jonas's out-of-office replies, and a clear Late mark on day 8 so Lena knows when to step in herself.

#### What to build

- Reminders on day 3 and day 7, using the same link and the same body as the first email, with "Reminder 1 of 2" or "Reminder 2 of 2" at the top. Subject: "Reminder: PP-034, Lena Hartmann needs your view by 2026-10-06". Never more than two automatic reminders.
- Skip a reminder when the owner confirmed identity or opened the page in the last 24 hours, or when Lena pressed Remind now (DR-5) in the last 24 hours.
- Day 8 with no answer: the state becomes Late and Lena gets one notification, "Jonas Keller has not answered PP-034 (day 8)".
- An auto-reply comes back: an Auto-Submitted header other than "no", an X-Auto-Response-Suppress header, Precedence auto_reply, or a subject starting with "Automatic reply", "Automatische Antwort", "Out of office", "Abwesenheit" or "Abwesenheitsnotiz", case-insensitive. Set flag to away and state to Paused, and stop both reminders and the Late transition. Lena sees "Jonas Keller appears to be away" with a "Resume reminders" button.
- A hard bounce: flag bounced, state Paused, and Lena sees "Email not delivered" with "Send again". Send again is DR-1's cancel and resend, so it goes to the owner's current address with a fresh clock. Soft bounces are retried by the mail provider and not shown.
- Resume reminders: the state goes back to Waiting, or straight to Late if day 8 has passed, and Lena is told once. Reminders missed while paused are not sent. The next one due goes out normally.
- Reminders due the same day for the same owner go out as one email, with a section and a button per request.
- No reminder and no Late transition once the pain point has left Shortlist (DR-1).

#### Not in this ticket

Reading return dates out of auto-replies. Drafting a note to the owner's manager. Letting customers change the cadence. Public holidays.

#### Data

The jobs are idempotent and check the state and the event log again right before sending. Fields: reminders_sent, last_owner_activity_at, flag (none | away | bounced), paused_at, resumed_at. A reminder goes out only when the state is Waiting, reminders_sent is under 2 and the pain point is still in Shortlist. Events: reminder_sent {reminder_no, merged}, reminder_skipped {reason}, paused {flag}, resumed, late.

#### Edge cases

- Lena presses Remind now less than 24 hours before an automatic reminder: the automatic one is skipped.
- The owner answers between the job's check and the send: the send is aborted.
- An auto-reply arrives after the answer: ignored.
- An auto-reply arrives on a Late request: flag away is stored, the state stays Late, and Lena gets no second notification.
- Resume before day 7: a day 3 reminder missed while paused is not sent, and the day 7 reminder goes out on day 7.
- The job runs twice, for example after a retry: one email, because the second run finds the reminder_sent event.
- Day 3 falls on a Saturday: the reminder goes out on Monday at 09:00 (Terms).

#### Acceptance criteria

- A request sent 2026-09-29 with no answer. Reminders go out on 2026-10-02 and 2026-10-06 at 09:00, reminders_sent is 2, age reads 3 and 7 on those days, and on 2026-10-07 the state is Late and Lena gets one in-app notification and one email.
- Jonas opened the page on 2026-10-02 at 08:10. The 09:00 job sends nothing that day, and the day 7 reminder still goes out on 2026-10-06.
- An auto-reply with Auto-Submitted: auto-replied arrives on 2026-10-03. The day 7 job sends nothing, the state is Paused, and Lena sees "Jonas Keller appears to be away".
- An auto-reply with the subject "Automatische Antwort: PP-034: Lena Hartmann needs your view on 3 startups" arrives on 2026-10-03. Same outcome.
- Lena presses Resume reminders on 2026-10-04. The state is Waiting and the day 7 reminder goes out on 2026-10-06.
- Lena presses Resume reminders on 2026-10-09. The next job run sets the state to Late and Lena is notified once, not twice.
- A hard bounce comes back on 2026-09-29. The state is Paused, flag is bounced, and Lena sees "Email not delivered" with "Send again". She corrects the address and presses it: the old request is Cancelled, a new one is Waiting with sent_at 2026-09-30, and the email goes to the new address.
- Jonas has two open requests with reminders due the same day. He gets one email with two sections and two buttons.
- PP-034 moves past Shortlist on 2026-10-05 with the request open. No reminder goes out on 2026-10-06 and the state never becomes Late.
- The seed on 2026-10-12. The jobs send nothing and PP-034 stays Late.

#### Details

- Analytics: decision_reminder_sent {request_id, reminder_no, merged}; decision_reminder_skipped {request_id, reason}; decision_request_paused {request_id, flag}; decision_request_late {request_id}
- Open questions: which auto-reply headers do Nordwind's mail servers send? Ask Nordwind IT. Calendar days or working days for 3, 7 and 8? Ask Lena. Public holidays: the default here is to treat them as weekdays, since the OS has no holiday calendar. Ask Lena.
- Module: Notifications
- Estimate: L
- Depends on: DR-1, DR-2

---

### DR-5: Waiting on others list, and the event log behind the measures

Lena needs one place that shows everything still waiting on someone and lets her act from there. The product team needs to know whether any of this worked, so the same ticket adds the four measures as queries over the event log.

#### What to build

- A page "Waiting on others" under Pain points, with tabs Open (default), Answered and Cancelled. A manager sees their own requests. An "All NVCU" switch, default off, adds every request in the unit. Any manager in the unit can act on any request, and the event log records who did.
- Columns: Request, Owner, Age, Due, State (Waiting, Late, Paused), Reminders, Last owner action, Flag. Late rows first, then oldest sent_at, then request id. Empty cells show "-". When nothing is open: "Nothing is waiting on anyone."
- Actions on each row depend on the state. Waiting and Late: Open shortlist, Remind now, Cancel, Answer on behalf, and "Send to Petra Ahlers instead" when the owner named someone. Paused: the same plus Resume reminders, with Remind now carrying the warning "Jonas Keller appears to be away". Answered and Cancelled: Open shortlist only.
- Remind now sends the DR-4 reminder email at once, adds 1 to reminders_sent, logs remind_now, counts as a manual chase, is not limited by the two automatic reminders, and then reads "Reminded today" for 24 hours.
- Cancel is DR-1's cancel, with the optional reason.
- Answer on behalf opens the DR-3 form with a required note, default "recorded by Lena Hartmann from a call". via is on_behalf, the owner gets no email, and his link then shows "Already answered by Lena Hartmann".
- "Send to Petra Ahlers instead": the request moves to that person with a fresh link, sent_at and due_at restart, reminders_sent goes back to 0, the old owner gets no email, and the old link shows "This request went to Petra Ahlers". When the handoff has no suggested_user_id, Lena is asked for an email address first, and the OS stores a contact {name, email} with no login and uses it as the owner. The same button sits on the shortlist header.
- The event log from DR-1, with the types created, link_opened, identity_confirmed, not_me, link_renewed, reminder_sent, reminder_skipped, remind_now, paused, resumed, late, handoff, reassigned, answered, cancelled.
- The four measures written as queries over requests created in a period, with no screen yet. Owner wait time: the median days from created to answered over answered requests with rated true, and the share of all requests answered by day 7, unanswered counted as misses. First touch within one working day: the share answered within one working day of created. Answered without a manual chase: the share of answered requests with no remind_now before answered. Late rate: the share of requests whose wait, answered minus created or today minus created while open, is more than 1.2 times the unit's median, or more than 8 days until the unit has 10 answered requests.

#### Not in this ticket

The digest email. A measures screen. Bulk actions. Risk flags.

#### Data

Open means Waiting, Late or Paused. Remind now adds 1 to reminders_sent and logs remind_now. Answer on behalf sets via to on_behalf and stores the note. Reassign logs reassigned {from, to} and keeps the old owner in the event log. contact {id, name, email} holds an owner who is not a user. One open request per shortlist, not per owner, so the new owner may already have another request open.

#### Edge cases

- Two managers act at the same time: the first wins, the second sees "Already answered".
- A late answer: the row shows "received after shortlist was finalised".
- Remind now on a Paused request: allowed, with the warning "Jonas Keller appears to be away".
- The suggested person is the sender, or has no email and Lena gives none: the button is disabled and says why.
- Lena presses Remind now at 08:59 and the 09:00 job runs: the automatic reminder is skipped (DR-4).
- Answer on behalf and Cancel are not offered on Answered or Cancelled rows.

#### Acceptance criteria

- The seed on 2026-10-12. Lena opens the list. PP-034 shows Jonas Keller, age 13, due 2026-10-06, Late, reminders 2 and last owner action "-". PP-040 (Anneke de Vries, age 24, Late) sorts above it.
- Tobias Brandt opens the list with "All NVCU" off and sees only his requests. He turns it on, sees PP-034 and PP-040, presses Remind now on PP-034, and the remind_now event names Tobias Brandt.
- Lena presses Remind now on PP-034. Reminders becomes 3, remind_now is logged, Jonas gets the reminder email, and the button reads "Reminded today" for 24 hours.
- Lena opens Answer on behalf for PP-034 and saves with the default note. The request is Answered, via is on_behalf, the shortlist shows the note, Jonas gets no email, his link shows "Already answered by Lena Hartmann", and the request counts as manually chased.
- The sample data, read as requests: sent_to_owner_at is created, owner_feedback_at is answered when the status is complete, partial counts as still open, and reminders_sent are manual chases because before this release every reminder was Lena's. The queries return a median wait of 7 days over 11 answered, 8 of 13 answered by day 7, 9 of 11 answered without a manual chase, and a late rate of 3 of 13 (SL-004 at 9 days, PP-034 open at 13, PP-040 open at 24).
- No open requests. Lena sees "Nothing is waiting on anyone."
- PP-034 has a handoff to Petra Ahlers. Lena presses "Send to Petra Ahlers instead" on 2026-10-01. The owner is Petra Ahlers, sent_at is 2026-10-01, due_at is 2026-10-08, reminders_sent is 0, the state is Waiting, reassigned is logged, Petra gets the DR-2 email, Jonas gets nothing, and his link shows "This request went to Petra Ahlers".
- PP-034 has a handoff to "Henrik Vos" with no suggested_user_id. Lena presses "Send to Henrik Vos instead", is asked for an email address, enters one, and the request goes to a new contact with that name and address. He gets the DR-2 email.

#### Details

- Analytics: decision_manager_action {request_id, action, age_days}
- Open questions: should "All NVCU" be on by default? Ask Lena. Should Dr. Sabine Koch see the list read-only? The default here is no. Ask Lena.
- Module: Pain points, Reports
- Estimate: L
- Depends on: DR-3, DR-4
