# Tickets: Business units submit a problem and then disappear

## The problem

Lena sends Jonas a shortlist. Two weeks pass. She chases by email, he can't find his login, and the pain point sits in Shortlist. We want to cut the days between "shortlist sent" and "owner answered".

## What we are building

One list for Lena of everything still waiting on someone. A decision request is what fills it. Lena asks one question from the shortlist. Jonas gets an email with one button and answers on one page in about three minutes. If he goes quiet, the system reminds him. If he is the wrong person, he names the right one and Lena sends it on with one click.

## The five tickets

Build them in this order. Each one ships on its own.

| Ticket | What it delivers | Size | Needs | Area |
|---|---|---|---|---|
| DR-1 | Lena sends and cancels a request from the shortlist | M | nothing | Pain points |
| DR-2 | The email, and a link that survives mail scanners | L | DR-1 | Notifications, Shared |
| DR-3 | Jonas answers on one page, Lena sees it on the shortlist | L | DR-1, DR-2 | Pain points, Shared |
| DR-4 | Reminders on days 3 and 7, pause when away, Late on day 8 | L | DR-1, DR-2 | Notifications |
| DR-5 | The Waiting on others list and the four measures | L | DR-3, DR-4 | Pain points, Reports |

Sizes are t-shirt sizes. The whole release is 5 to 7 engineer-weeks.

Test against the sample data at https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/data/sample-data.json. Today in it is 12 Oct 2026.

## Rules for all five tickets

- A request is one question from a manager to one owner about one shortlist. This release has one question: rate this shortlist.
- States: Waiting, Paused, Late, Answered, Cancelled. Waiting, Paused and Late are open. An open request can become Answered or Cancelled. Those two are final.
- One open request per shortlist. Enforce it in the database.
- Days are calendar days from the send date, in Europe/Berlin. Sent 29 Sep 2026: day 3 is 2 Oct, due 6 Oct at 23:59, link expires 13 Oct at 23:59. Age never resets.
- Jobs run at 09:00 on weekdays. Anything due on a weekend runs on Monday.
- Anyone who can edit the pain point can send, cancel, remind, answer on behalf or send on. The event log records who.
- "Tell Lena" means one in-app notification and one email. That is the workspace default in the sample data.
- Nothing closes a request on its own. Day 14 expires the link and nothing else. A Late request stays until Lena acts.
- Finalised means the pain point has left Shortlist. Reminders stop, Late stops, and the request stays open so a late answer is still recorded.
- One event table: request_id, type, at, actor, payload. DR-5 lists the types.
- Sample data: PP-034 is Jonas Keller's. Sent 29 Sep, reminders on 2 and 6 Oct, Late since 7 Oct, day 13 today. PP-040 is Anneke de Vries's, sent 18 Sep, open at day 24.

## Not in this release

Suggested stage change, editing a request after sending, the closing-the-loop email, the daily digest. The brief says why: https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/PRODUCT-BRIEF.md.

---

### DR-1: Send and cancel a request from the shortlist

#### Description

Lena has three startups on the PP-034 shortlist and needs Jonas to say which are worth a call. Today she writes an email and hopes. After this ticket she presses one button, sees that the request is out, and can cancel it.

#### What to build

- Add the button "Ask Jonas Keller to rate this shortlist" to the shortlist header while the pain point is in Shortlist. The name is the pain point's owner.
- The button opens a dialog: the email the owner will get, an optional note to the owner (plain text, 1,000 characters), and "Send request". Until DR-2 exists, show the template text with the names filled in.
- Send creates the request as Waiting, snapshots the shortlist, and logs created. DR-2 sends the email.
- While the request is open, replace the button with its state: "Waiting for Jonas Keller, day 13", "Late: Jonas Keller, day 13" or "Paused: Jonas Keller, day 13". After an answer, DR-3 shows the answers here. After a cancel, bring the button back.
- Add "Cancel request" next to the state. Confirm, take an optional reason, set Cancelled, log cancelled. No email to the owner.
- When a startup is added or removed while a request is open, show "Shortlist changed since the request" with "Cancel and resend". That cancels the request and reopens the dialog with the note filled in.

#### User stories

- As Lena, I want to ask Jonas with one button, so that I stop writing chasing emails.
- As Lena, I want the shortlist to show that a request is out and how old it is, so that I know whether to wait or step in.
- As Tobias, covering for Lena, I want to see that a request is already out, so that I don't send a second one.

#### Acceptance criteria

- Lena sends PP-034 on 29 Sep 2026. The request is Waiting, due 6 Oct at 23:59 Europe/Berlin, the link expires 13 Oct at 23:59, reminders_sent is 0, the snapshot holds Pallet.ai, Cargonaut Labs and Parcelgrid with descriptions and comments, and created is logged.
- Tobias opens PP-034 on 12 Oct 2026. He sees "Late: Jonas Keller, day 13" and no send button.
- A shortlist has 2 startups. The button is disabled and reads "Needs 3 to 5 startups".
- Lena adds a fourth startup while the request is open. The banner shows. "Cancel and resend" cancels the old request and opens the dialog with her note. Send creates a new Waiting request with 4 startups.
- Lena cancels PP-034 with the reason "Jonas is on leave". The state is Cancelled, the reason is stored, cancelled is logged, Jonas gets no email, and the button is back.
- Lena and Tobias press Send within a second. One request exists. The other sees "Already sent by Lena Hartmann".
- PP-034 is finalised on 12 Oct while the request is open. It stays Late, the header reads "Finalised, answer still open until 13 Oct 2026", and no reminder goes out.

#### Edge cases

- No owner, owner not a user in the OS, or owner is the sender: disable the button and say which. Example: "Henrik Vos has no email address in the OS" (PP-017).
- Under 3 or over 5 startups: disable the button, "Needs 3 to 5 startups".
- A description or comment edited after sending: no banner. The snapshot stands.

#### Open questions

- Can Lena send to someone other than the owner? Default: no. The wrong owner names the right person (DR-3) and Lena sends it on (DR-5). Ask Lena how often that happens.
- Should finalising cancel the request? Default: no, so a late answer is still recorded. Ask Lena.

---

### DR-2: The email, and a link that survives mail scanners

#### Description

Jonas gets one email with one button. He taps it, confirms it is him, and lands on the answer page. Corporate mail scanners open every link before he does, so nothing counts until a person taps "Yes, I am Jonas Keller".

#### What to build

- Send from "GlassDollar for Nordwind" at a GlassDollar address. Reply-To is Lena, so a human reply reaches her. Return-Path is the OS inbound address, so bounces and auto-replies come back for DR-4.
- Subject: "PP-034: Lena Hartmann needs your view on 3 startups". Body: who asks, what for, "about 3 minutes", Lena's note, the due date. One text button, "Review 3 startups", with the full URL in plain text under it. No startup names.
- The link carries a random token (32 bytes or more, stored hashed) tied to the request and the owner's address, valid until the link expires.
- Opening the link shows "Yes, I am Jonas Keller" and "Not me". The first tap records confirmed_at and sets a session cookie for this request until the link expires. The token stays valid, so the same link works from reminders, other devices and later days. With a session, skip the identity page.
- "Not me": show "Thanks, we've told Lena Hartmann". Change nothing. Tell Lena once per request.
- Expired link: show "This link has expired" with "Request a new link". The new link goes to the same address, lasts 14 days from now, and tells Lena "Jonas Keller asked for a new link". Open requests only, 3 times per request, then "Ask Lena Hartmann for a new link". Answered request: "Your answer was sent on 12 Oct 2026". Cancelled: "Lena no longer needs this".
- SPF, DKIM and DMARC on the sending domain. Add "allowlist the sender" to the customer setup checklist.

#### User stories

- As Jonas, I want to open the request on my phone with no login, so that I can answer between meetings.
- As Jonas, I want the link to work again tomorrow and on my laptop, so that I am not locked out.
- As Lena, I want to know when someone other than Jonas opened the link, so that I can check with him.

#### Acceptance criteria

- Jonas's PP-034 email contains "Review 3 startups", "about 3 minutes" and 6 Oct 2026. It does not contain Pallet.ai, Cargonaut Labs or Parcelgrid.
- A scanner fetches a new link three times. The request is still Waiting, confirmed_at is empty, and no owner activity is logged.
- Jonas taps on his phone on 1 Oct and opens the link on his laptop on 2 Oct. The laptop shows the identity page once, then the answer page. confirmed_at stays 1 Oct.
- Jonas taps on 13 Oct and the page opens. On 14 Oct he sees "This link has expired" with "Request a new link".
- He requests a new link on 14 Oct. It expires 28 Oct at 23:59, renewals is 1, Lena is told once, and the old link stays expired. A fourth request shows "Ask Lena Hartmann for a new link" and sends nothing.
- Anneke de Vries taps "Not me" on a forwarded email. She sees no startup. Lena is told "Someone other than Jonas Keller opened the PP-034 link".
- Lena cancelled PP-034. Jonas opens the link and sees "Lena no longer needs this", with no identity page.
- Images are blocked. The button is readable text and the URL is visible under it.

#### Edge cases

- A sandbox presses the button: confirmed_at is set. Nothing more can happen without choices, and the measures count answers, not taps.
- The request was sent on (DR-5): the old link shows "This request went to Petra Ahlers".
- The owner's address changes after sending: the old link keeps working. Use the new address from the next email.

#### Open questions

- Does Nordwind force SSO on outside links? Ask Nordwind IT.
- One shared GlassDollar sending domain, or one per customer? Ask Nordwind IT.
- German or English? Default: English until the OS has a workspace language. Ask Lena.

---

### DR-3: Jonas answers on one page, Lena sees it on the shortlist

#### Description

After the identity tap, Jonas gets one page with the three startups and three choices each. He picks, adds a note if he wants, and sends. Lena sees the answers on the shortlist and gets one notification. Nobody asks twice.

#### What to build

- One page, phone first, no navigation, behind the DR-2 session. Show the PP-034 title, Lena's note, and one card per startup from the snapshot: name, description, Lena's comment, the choices Interested, Not now and Need more info, and a comment box. Below the cards: a "None of these fit" checkbox with a comment box that replaces the choices, and "Send to Lena".
- Require a comment for Need more info and for None of these fit. Keep Send enabled. On a missing field, say what is missing and move focus there. Send nothing until it is complete.
- After sending, show "Sent to Lena Hartmann." with a summary. A valid link opened again shows the answer read-only.
- On the shortlist, show each choice with who, when and how. Tell Lena once: "Jonas Keller answered PP-034", or "Jonas Keller needs more information on 3 startups" when nothing was rated.
- After an answer, bring the button back as "Ask Jonas Keller again". That is the follow-up to Need more info. The information itself goes by email or a call.
- Under Send: "Someone else should answer this". A name field that suggests OS users as he types. Any name is allowed. "Tell Lena" keeps the request open, tells Lena "Jonas Keller says Petra Ahlers should answer PP-034", and shows "Sent to Lena Hartmann. She'll send PP-034 to Petra Ahlers." If he answers afterwards, the answer wins and the handoff is cleared.

#### User stories

- As Jonas, I want to rate three startups on my phone in a couple of minutes, so that Lena gets my view without a meeting.
- As Jonas, I want to say that someone else should answer, so that requests stop landing on me when they are not mine.
- As Lena, I want the answers on the shortlist with one notification, so that I copy nothing out of an email.

#### Acceptance criteria

- After the tap, Jonas sees Pallet.ai, Cargonaut Labs and Parcelgrid with three choices each, Lena's note, and no navigation.
- Jonas rates 2 of 3, or ticks None of these fit without a comment, or sets Need more info without a comment, and presses Send. Nothing is sent. The page says what is missing and focus moves there.
- Jonas sets Pallet.ai to Interested and the others to Not now and sends on 12 Oct 2026. The request is Answered, via is magic_link, rated is true, the shortlist shows "Jonas Keller, 12 Oct 2026, via link", Lena is told once, and the button reads "Ask Jonas Keller again".
- All three are Need more info, each with a comment. The request is Answered, rated is false, and Lena's notification reads "Jonas Keller needs more information on 3 startups".
- PP-034 is finalised on 12 Oct and Jonas sends on 13 Oct. late is true and the shortlist shows "received after shortlist was finalised".
- Jonas types "Pet" and picks Petra Ahlers: suggested_user_id is U-009. He types "Henrik Vos": the name is stored with no user id.
- Jonas sends a handoff on 1 Oct. The request is still Waiting, Lena is told once, and reopening the link shows "Sent to Lena Hartmann" with Petra's name. He rates all three on 2 Oct: Answered, the handoff is cleared, and the list no longer offers "Send to Petra Ahlers instead".
- Lena answers on behalf at 10:00 and Jonas presses Send at 10:01. He sees "Already answered by Lena Hartmann". Lena's answer stands.

#### Edge cases

- Two tabs submit: the second sees "Already sent".
- Opened after a cancel: "Lena no longer needs this". After a send-on: "This request went to Petra Ahlers". After an answer on behalf: "Already answered by Lena Hartmann".
- Five startups in the snapshot: five cards, same rules.
- Comments: 1,000 characters, with a counter from 100 left.

#### Open questions

- Should the confirmation promise a reply time, say "within 3 working days"? Ask Lena.
- Should Need more info keep the request open? Default: no. It is Answered with rated false, and Lena sends a new request when she has the information. Ask Lena.

---

### DR-4: Reminders on days 3 and 7, pause when away, Late on day 8

#### Description

Today Lena is the reminder system. This ticket moves it into the OS: two automatic reminders, a pause when the out-of-office replies, and a Late mark on day 8 so Lena knows when to step in.

#### What to build

- On days 3 and 7, send the first email again with "Reminder 1 of 2" or "Reminder 2 of 2" on top and the same link. Subject: "Reminder: PP-034, Lena Hartmann needs your view by 6 Oct 2026". Never more than two.
- Skip a reminder if the owner confirmed identity or opened the page in the last 24 hours, or if Lena pressed Remind now in the last 24 hours.
- On day 8 with no answer, set Late and tell Lena "Jonas Keller has not answered PP-034 (day 8)".
- On an auto-reply, set flag away and state Paused. Stop reminders and the Late transition. Show Lena "Jonas Keller appears to be away" with "Resume reminders". An auto-reply is any of: an Auto-Submitted header other than "no", an X-Auto-Response-Suppress header, Precedence auto_reply, or a subject starting with "Automatic reply", "Automatische Antwort", "Out of office", "Abwesenheit" or "Abwesenheitsnotiz", in any case.
- On a hard bounce, set flag bounced and state Paused. Show Lena "Email not delivered" with "Send again", which is DR-1's cancel and resend to the current address. Let the provider retry soft bounces and don't show them.
- Resume reminders sets Waiting, or Late at once if day 8 has passed, and tells Lena once. Don't send reminders missed while paused. Send the next one due as normal.
- Merge reminders due the same day for one owner into one email, with a section and a button per request.
- Make the jobs idempotent: re-check the state and the event log right before sending. Send only when Waiting, fewer than two sent, and the pain point still in Shortlist.

#### User stories

- As Lena, I want the OS to remind Jonas twice, so that I only chase when that failed.
- As Lena, I want reminders to pause while Jonas is away, so that he does not come back to three nagging emails.
- As Lena, I want to be told on day 8, so that I know when to pick up the phone.

#### Acceptance criteria

- Sent 29 Sep 2026, no answer. Reminders go out on 2 Oct and 6 Oct at 09:00, reminders_sent is 2, and on 7 Oct the state is Late and Lena gets one in-app notification and one email.
- Jonas opened the page on 2 Oct at 08:10. The 09:00 job sends nothing. The day 7 reminder still goes out on 6 Oct.
- "Auto-Submitted: auto-replied" arrives on 3 Oct. No day 7 reminder, the state is Paused, Lena sees "Jonas Keller appears to be away". A subject starting "Automatische Antwort:" on 3 Oct gives the same outcome.
- Lena resumes on 4 Oct: Waiting, and the day 7 reminder goes out on 6 Oct. Lena resumes on 9 Oct: Late at the next run, told once.
- A hard bounce on 29 Sep: Paused, flag bounced, "Email not delivered" with "Send again". Lena corrects the address and presses it: the old request is Cancelled, a new one is Waiting from 30 Sep, and the email goes to the new address.
- Jonas has two requests due the same day. One email, two sections, two buttons.
- PP-034 is finalised on 5 Oct. No reminder on 6 Oct, never Late.
- The sample data on 12 Oct. The jobs send nothing and PP-034 stays Late.

#### Edge cases

- The owner answers between the check and the send: abort.
- An auto-reply after the answer: ignore. On a Late request: store the flag, keep Late, no second notification.
- Resume before day 7: send day 7, skip the missed day 3.
- The job runs twice: one email, because the second run finds the reminder_sent event.
- Day 3 on a Saturday: Monday at 09:00.

#### Open questions

- Which auto-reply headers do Nordwind's mail servers send? Ask Nordwind IT.
- Calendar or working days for 3, 7 and 8? Ask Lena.
- Public holidays: default is to treat them as weekdays. Ask Lena.

---

### DR-5: The Waiting on others list and the four measures

#### Description

Lena needs one place that shows everything still waiting on someone and lets her act from there. The product team needs to know whether any of this worked, so the same ticket adds the four measures as queries over the event log.

#### What to build

- Add the page "Waiting on others" under Pain points with the tabs Open (default), Answered and Cancelled. A manager sees their own requests. An "All NVCU" switch, off by default, adds the unit's.
- Columns: Request, Owner, Age, Due, State, Reminders, Last owner action, Flag. Sort Late first, then oldest, then request id. Show "-" in empty cells. Empty list: "Nothing is waiting on anyone."
- Row actions by state. Waiting and Late: Open shortlist, Remind now, Cancel, Answer on behalf, and "Send to Petra Ahlers instead" when a handoff exists. Paused: the same plus Resume reminders, and Remind now warns "Jonas Keller appears to be away". Answered and Cancelled: Open shortlist only.
- Remind now sends the DR-4 reminder now, adds 1 to reminders_sent, and logs remind_now, which counts as a manual chase. It is not limited by the two automatic reminders. Then show "Reminded today" for 24 hours.
- Cancel is DR-1's cancel.
- Answer on behalf opens the DR-3 form with a required note, default "recorded by Lena Hartmann from a call". via is on_behalf. No email to the owner. His link then shows "Already answered by Lena Hartmann".
- "Send to Petra Ahlers instead" moves the request to that person with a fresh link. sent_at and due_at restart, reminders_sent goes to 0, the old owner gets nothing, and the old link shows "This request went to Petra Ahlers". If the handoff matched no user, ask Lena for an email address and store a contact (name, email, no login). Put the same button on the shortlist header.
- Event types: created, link_opened, identity_confirmed, not_me, link_renewed, reminder_sent, reminder_skipped, remind_now, paused, resumed, late, handoff, reassigned (old owner, new owner), answered, cancelled.
- Four queries over the requests created in a period. No screen yet.
  - Owner wait time: the median days from created to answered over answered requests with rated true, and the share of all requests answered by day 7, with unanswered counted as misses.
  - First touch: the share answered within one working day.
  - No manual chase: the share of answered requests with no remind_now before the answer.
  - Late rate: the share whose wait (answered minus created, or today minus created while open) is more than 1.2 times the unit's median, or more than 8 days until the unit has 10 answered requests.

#### User stories

- As Lena, I want one list of everything waiting on someone, so that nothing I own stalls without me knowing.
- As Lena, I want to remind, cancel or answer on behalf from the list, so that I don't open each pain point to act.
- As the product team, we want the four measures as queries, so that after a quarter we know whether the wait got shorter.

#### Acceptance criteria

- The sample data on 12 Oct 2026. PP-034 shows Jonas Keller, age 13, due 6 Oct, Late, 2 reminders, last owner action "-". PP-040 (Anneke de Vries, age 24, Late) sorts above it.
- Tobias with All NVCU off sees only his requests. With it on, he sees PP-034 and PP-040 too, and his Remind now is logged under his name.
- Lena presses Remind now on PP-034. Reminders is 3, remind_now is logged, Jonas gets the email, and the button reads "Reminded today" for 24 hours.
- Lena answers on behalf for PP-034 with the default note. Answered, via on_behalf, the note is on the shortlist, Jonas gets no email, his link shows "Already answered by Lena Hartmann", and the request counts as manually chased.
- The sample data read as requests (sent_to_owner_at is created, owner_feedback_at is answered when complete, partial is open, reminders_sent are manual chases): a median of 7 days over 11 answered, 8 of 13 by day 7, 9 of 11 without a chase, and a late rate of 3 of 13 (SL-004, PP-034, PP-040).
- No open requests. "Nothing is waiting on anyone."
- PP-034 has a handoff to Petra Ahlers. Lena presses "Send to Petra Ahlers instead" on 1 Oct. The owner is Petra Ahlers, sent_at is 1 Oct, due_at is 8 Oct, reminders_sent is 0, the state is Waiting, reassigned is logged, Petra gets the DR-2 email, Jonas gets nothing, and his link shows "This request went to Petra Ahlers".
- The handoff names "Henrik Vos" with no user id. Lena is asked for an email address, enters one, the request goes to a new contact, and he gets the email.

#### Edge cases

- Two managers act at once: the first wins, the second sees "Already answered".
- A late answer: the row shows "received after shortlist was finalised".
- The suggested person is the sender, or has no email and Lena gives none: disable the button and say why.
- Remind now at 08:59 and the job at 09:00: skip the automatic reminder.
- The new owner may already have another request open. The limit is per shortlist, not per owner.

#### Open questions

- "All NVCU" on by default? Ask Lena.
- Read-only access for Dr. Sabine Koch? Default: no. Ask Lena.
