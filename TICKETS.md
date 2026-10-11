# Tickets: Business units submit a problem and then disappear

## Why this epic exists

Lena sends Jonas a shortlist and then nothing happens for two weeks. She chases him by email, he can't find his login, and the pain point sits in Shortlist. We want to cut the time between "shortlist sent" and "owner answered".

The fix starts with one list for Lena of everything that is still waiting on someone, on whom and for how long. A decision request is what fills it. Lena asks one question from the shortlist. Jonas gets an email with one button, answers on one page, and is done in about three minutes. If he doesn't answer, the system reminds him so Lena doesn't have to. If he isn't the right person, he says who is and Lena sends it on with one click.

Email is the channel because every owner has it. The ask is what changes: three ratings by a date, with Lena told on day 8.

## Order

Build DR-1 to DR-5 in that order. Each ticket stands on its own. The acceptance criteria use [data/sample-data.json](data/sample-data.json), so you can check your work without asking anyone.

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

The seed data tells one story. PP-034 is Jonas's pain point about manual load planning for outbound trucks. Lena sent him the request on 29 September 2026. Reminders went out on 2 and 6 October. It went Late on 7 October. On 12 October it is day 13 and still unanswered. Lena's note to Dr. Sabine Koch on 7 October went by email outside the OS, so the system knows nothing about it.

## Left out on purpose

Suggested stage change, editing a request after it is sent, the closing-the-loop email and the daily digest are Should or Later. [README.md](README.md#scope) says why.

---

### DR-1: Create a decision request from a shortlist

Lena has three startups on the PP-034 shortlist and needs Jonas to say which are worth a call. Today she writes an email and hopes. This ticket gives her a button on the shortlist that sends one request, and shows her that it's out there.

#### What to build

- On a shortlist with 3 to 5 startups, a button "Ask Jonas Keller to rate this shortlist". It opens a preview of the email (DR-2), an optional note to the owner of up to 1,000 characters, and Send.
- One open request per shortlist. The request keeps a snapshot of the startups as they were at send time.
- While a request is open, the shortlist header says "Waiting for Jonas Keller, day N".

#### Not in this ticket

Other kinds of question. More than one owner on a request. Editing a request after it's sent: cancel it and send a new one.

#### Data

request {id, pain_point_id, shortlist_id, owner_id, sent_by, note, state, sent_at, due_at, link_expires_at, startups_snapshot[], reminders_sent, flag}. Any open state can go to Answered or Cancelled.

#### Edge cases

- Fewer than 3 or more than 5 startups: the button is disabled and says "Needs 3 to 5 startups".
- The shortlist changes after sending: show "Shortlist changed since the request" with Cancel and resend.
- The owner has no email address, or the owner is the sender: the button is disabled and says why.

#### Acceptance criteria

- PP-034 has 3 startups and Lena sends on 2026-09-29. The request is Waiting, due_at is 2026-10-06 23:59 Europe/Berlin, link_expires_at is 2026-10-13 23:59 and reminders_sent is 0.
- That request is still open and Tobias Brandt opens the shortlist on 2026-10-12. He sees "Waiting for Jonas Keller, day 13" and can't create another.
- A shortlist has 2 startups. The button is disabled and says "Needs 3 to 5 startups".
- A request is out and Lena adds a fourth startup. The shortlist shows "Shortlist changed since the request" with Cancel and resend.

#### Details

- Analytics: decision_request_created {request_id, startups_count, note_length}
- Open questions: none
- Module: Pain points
- Estimate: M
- Depends on: none

---

### DR-2: Email the owner with a link that survives scanners

Jonas gets one email with one button. He taps it, confirms it's him, and he's on the answer page. Corporate mail scanners open every link before he does, so the link has to survive that: nothing counts until a person taps "Yes, I am Jonas Keller".

#### What to build

- The email comes from "GlassDollar for Nordwind" with reply-to set to Lena. Subject: "PP-034: Lena Hartmann needs your view on 3 startups". The body says who is asking, what for, that it takes about 3 minutes, and the due date. Then one text button, "Review 3 startups", with the full URL as plain text under it. Startup names never appear in the email.
- The link carries a token tied to the request and to the owner's email address, valid for 14 days. Opening it (GET) shows a page with "Yes, I am Jonas Keller" and "Not me". Only the tap (POST) uses up the token. A GET never does.
- The sending domain needs SPF, DKIM and DMARC. Add a line to the customer setup checklist asking IT to allowlist the sender.

#### Not in this ticket

Buttons inside the email body. Teams or Slack. Passing the request to someone else is DR-3 (the owner names who) and DR-5 (Lena sends it on). "Not me" on the identity page only tells Lena.

#### Data

link {request_id, token_hash, bound_email, expires_at, confirmed_at}. An open before the tap is logged with likely_bot true when it comes from a known scanner range, or lands within 60 seconds of sending with no tap after it.

#### Edge cases

- A scanner fetches the link: nothing is recorded as owner activity.
- The link has expired: show "This link has expired" and "Request a new link". A new link resets the expiry and tells Lena.
- Images are blocked (Outlook desktop): the button is text in a table cell, and the plain URL under it still works.
- The email was forwarded: this is what "Not me" is for. The tap is what stops a colleague answering as Jonas.

#### Acceptance criteria

- Lena sends PP-034. Jonas's email contains "Review 3 startups", "about 3 minutes" and "2026-10-06", and does not contain "Pallet.ai", "Cargonaut Labs" or "Parcelgrid".
- A scanner fetches a new link three times. The request is still Waiting, confirmed_at is empty and no owner activity is logged.
- Jonas taps "Yes, I am Jonas Keller" on 2026-10-13 and the page opens. On 2026-10-14 he sees "This link has expired" with "Request a new link".
- Anneke de Vries gets a forwarded PP-034 email and taps "Not me". She sees no startup, and Lena is told "Someone other than Jonas Keller opened the PP-034 link".
- Images are blocked. The button is readable text and the full URL is visible under it.

#### Details

- Analytics: decision_link_opened {request_id, likely_bot}; decision_identity_confirmed {request_id, days_since_sent}
- Open questions: does Nordwind force SSO on links from outside SaaS tools? Ask Nordwind IT.
- Module: Notifications, Shared
- Estimate: L
- Depends on: DR-1

---

### DR-3: Let the owner answer on one page, and land the answer on the shortlist

After the identity tap, Jonas lands on one page with the three startups and three choices each. He picks, adds a note if he wants, and sends. Lena sees the answers on the shortlist and gets one notification. Nobody asks twice.

#### What to build

- One page, built for a phone first, with no navigation. It shows the PP-034 title, Lena's note, and one card per startup: the name, the one-line description from the snapshot, Lena's comment if she left one, three choices (Interested, Not now, Need more info) and an optional comment. Under the cards, a "None of these fit" checkbox with a required comment that replaces the per-startup choices, and a "Send to Lena" button.
- Send is always enabled. If a startup has no choice, or "None of these fit" is ticked without a comment, the page says what is missing and moves focus to it. Nothing is sent until it's complete.
- After sending: "Sent to Lena Hartmann." with a summary of the choices. Opening the link again shows the answer read-only.
- On the shortlist, each startup shows the choice with who answered, when, and how. Lena gets one notification in the OS and one email.
- Under the Send button, "Someone else should answer this". It asks for a name, and "Tell Lena" sends it. The request stays open, Lena gets one notification, "Jonas Keller says Petra Ahlers should answer PP-034", and the page says "Sent to Lena Hartmann. She'll send PP-034 to Petra Ahlers."

#### Not in this ticket

Editing after sending. Suggested stage change. SourceAI profiles. Attachments.

#### Data

answer {request_id, items[{startup_id, choice, comment}], none_fit, none_fit_comment, submitted_at, via (magic_link | on_behalf), late, rated}. Sending sets the request to Answered. late is true when the answer arrives after the shortlist was finalised. rated is true when at least one startup is Interested or Not now, or none_fit is ticked. All three on Need more info means rated is false. handoff {request_id, suggested_name, by, at}. A handoff does not change the state.

#### Edge cases

- Two tabs submit: the second one sees "Already sent".
- The page is opened after the request was cancelled: "Lena no longer needs this".
- Comments are capped at 1,000 characters.

#### Acceptance criteria

- Jonas has confirmed identity. The page shows Pallet.ai, Cargonaut Labs and Parcelgrid with three choices each, Lena's note, and no other navigation.
- Jonas has rated 2 of 3, or ticked "None of these fit" without a comment, and presses "Send to Lena". Nothing is sent, the page says what is missing and focus moves to it.
- Jonas sets Pallet.ai to Interested and the others to Not now, and sends on 2026-10-12. The request is Answered, via is magic_link, rated is true, the shortlist shows the three choices with "Jonas Keller, 2026-10-12, via link", and Lena gets one notification.
- All three are Need more info. The request is Answered, rated is false, and Lena's notification says "Jonas Keller needs more information on 3 startups".
- Lena finalised the shortlist on 2026-10-12 and Jonas sends on 2026-10-13. The answer is stored with late true and the shortlist shows "received after shortlist was finalised".
- Jonas presses "Someone else should answer this", types "Petra Ahlers" and presses "Tell Lena" on 2026-10-01. The request is still Waiting, handoff holds her name, Lena gets one notification, and opening the link again shows "Sent to Lena Hartmann" with her name.

#### Details

- Analytics: decision_answer_submitted {request_id, days_since_sent, via, rated, late, seconds_on_page}
- Open questions: should the confirmation promise a reply time, say "within 3 working days"? Ask Lena.
- Module: Pain points, Shared
- Estimate: L
- Depends on: DR-1, DR-2

---

### DR-4: Remind on days 3 and 7, pause when the owner is away, mark Late on day 8

Today Lena is the reminder system. This ticket moves that into the OS: two automatic reminders, a pause when Jonas's out-of-office replies, and a clear Late mark on day 8 so Lena knows when to step in herself.

#### What to build

- Reminders on day 3 and day 7, using the same link. Subject: "Reminder: PP-034, Lena Hartmann needs your view by 2026-10-06". Never more than two automatic reminders.
- Skip a reminder when the owner confirmed identity or opened the page in the last 24 hours.
- Day 8 with no answer: the state becomes Late and Lena gets one notification, "Jonas Keller has not answered PP-034 (day 8)".
- An auto-reply comes back (an Auto-Submitted header, or an out-of-office subject): set flag to away and state to Paused, and stop both reminders and the Late transition. Lena sees "Jonas Keller appears to be away" with a "Resume reminders" button.
- A hard bounce: flag bounced, state Paused, and Lena sees "Email not delivered".
- Reminders due the same day for the same owner go out as one email, with a section and a button per request.

#### Not in this ticket

Reading return dates out of auto-replies. Drafting a note to the owner's manager. Letting customers change the cadence.

#### Data

The jobs are idempotent and check the state again right before sending. Fields: reminders_sent, last_owner_activity_at, flag (none | away | bounced), paused_at. A reminder goes out only when the state is Waiting and reminders_sent is under 2. On Resume, if day 8 has already passed, the state becomes Late straight away and Lena is told once.

#### Edge cases

- Lena presses Remind now less than 24 hours before an automatic reminder: the automatic one is skipped.
- The owner answers between the job's check and the send: the send is aborted.
- An auto-reply arrives after the answer: ignored.

#### Acceptance criteria

- A request sent 2026-09-29 with no answer. Reminders go out on 2026-10-02 and 2026-10-06 at 09:00, reminders_sent is 2, age reads 3 and 7 on those days, and on 2026-10-07 the state is Late and Lena is notified once.
- Jonas opened the page on 2026-10-02 at 08:10. The 09:00 job sends nothing that day, and the day 7 reminder still goes out on 2026-10-06.
- An auto-reply with Auto-Submitted: auto-replied arrives on 2026-10-03. The day 7 job sends nothing, the state is Paused, and Lena sees "Jonas Keller appears to be away".
- Lena presses Resume reminders on 2026-10-09. The next job run sets the state to Late and Lena is notified once, not twice.
- Jonas has two open requests with reminders due the same day. He gets one email with two sections and two buttons.
- The seed on 2026-10-12. The jobs send nothing and PP-034 stays Late.

#### Details

- Analytics: decision_reminder_sent {request_id, reminder_no, merged}; decision_reminder_skipped {request_id, reason}; decision_request_paused {request_id, flag}; decision_request_late {request_id}
- Open questions: which auto-reply headers do Nordwind's mail servers send? Ask Nordwind IT. Calendar days or working days for 3, 7 and 8? Ask Lena.
- Module: Notifications
- Estimate: L
- Depends on: DR-1, DR-2

---

### DR-5: Waiting on others list, and the event log behind the measures

Lena needs one place that shows everything still waiting on someone and lets her act from there. The product team needs to know whether any of this worked, so the same ticket adds an event log and the four measures as queries.

#### What to build

- A page "Waiting on others" under Pain points, with tabs Open (default), Answered and Cancelled. An "All NVCU" switch adds Tobias Brandt's requests.
- Columns: Request, Owner, Age, Due, State (Waiting, Late, Paused), Reminders, Last owner action, Flag. Late rows first, then oldest. Empty cells show "-". When nothing is open: "Nothing is waiting on anyone."
- Actions on each row: Remind now (once per 24 hours, counts as a manual chase), Cancel (optional reason), Answer on behalf (the DR-3 form plus a required note, default "recorded by Lena Hartmann from a call"; the owner gets no email), and, when the owner named someone else, "Send to Petra Ahlers instead".
- "Send to [name] instead": the request moves to that person with a fresh link, sent_at and due_at restart, reminders_sent goes back to 0, and the old link shows "This request went to Petra Ahlers". If the name isn't in the OS, Lena is asked for an email address first. The same button sits on the shortlist header.
- An event log: {request_id, type, at, actor, payload} with the types created, link_opened, identity_confirmed, reminder_sent, reminder_skipped, remind_now, paused, resumed, late, handoff, reassigned, answered, cancelled.
- The four measures written as queries, with no screen yet: owner wait time (median days from created to answered among rated requests, plus the share answered by day 7 with unanswered counted as misses); first touch within one working day; answered without a manual chase (no remind_now before answered); late rate (open requests older than 1.2 times the unit's median, or day 8 until there is a median).

#### Not in this ticket

The digest email. A measures screen. Bulk actions. Risk flags.

#### Data

Open means Waiting, Late or Paused. Remind now adds 1 to reminders_sent and logs remind_now. Answer on behalf sets via to on_behalf and stores the note. Reassign logs reassigned and keeps the old owner in the event log.

#### Edge cases

- Two managers act at the same time: the first wins, the second sees "Already answered".
- A late answer: the row shows "received after shortlist was finalised".
- Remind now on a Paused request: allowed, with the warning "Jonas Keller appears to be away".
- The suggested person is the sender, or has no email: the button is disabled and says why.

#### Acceptance criteria

- The seed on 2026-10-12. Lena opens the list. PP-034 shows Jonas Keller, age 13, due 2026-10-06, Late, reminders 2 and last owner action "-". PP-040 (Anneke de Vries, age 18, Late) sorts above it.
- Lena presses Remind now on PP-034. Reminders becomes 3, remind_now is logged, and the button is disabled for 24 hours.
- Lena opens Answer on behalf for PP-034 and saves with the default note. The request is Answered, via is on_behalf, the shortlist shows the note, Jonas gets no email, and the request counts as manually chased.
- The two sample quarters. The measure queries return the median wait, the share by day 7, the share without a manual chase and the late rate, with unanswered requests counted as misses.
- No open requests. Lena sees "Nothing is waiting on anyone."
- PP-034 has a handoff to Petra Ahlers. Lena presses "Send to Petra Ahlers instead" on 2026-10-01. The owner is Petra Ahlers, sent_at is 2026-10-01, due_at is 2026-10-08, reminders_sent is 0, the state is Waiting, Petra gets the DR-2 email, and Jonas's link shows "This request went to Petra Ahlers".

#### Details

- Analytics: decision_manager_action {request_id, action, age_days}
- Open questions: should Tobias Brandt's requests show by default? Ask Lena.
- Module: Pain points, Reports
- Estimate: L
- Depends on: DR-3, DR-4
