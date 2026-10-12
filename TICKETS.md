# Tickets: Business units submit a problem and then disappear

## The problem

A user sends the pain point owner a shortlist and waits. The owner has no GlassDollar login, the user chases by email, and the pain point sits in Shortlist for weeks. The goal is fewer days between the send and the answer.

## What we are building

Decision requests. The user asks one question from the shortlist. The owner gets an email with one button and answers on one page in about three minutes, with no login. The OS reminds the owner twice and marks the request Late on day 8. The owner can name someone else, and the user resends to that person. One list shows the user every open request.

## The five tickets

Build and deploy them in this order. Nothing is useful before DR-3 is live: DR-1 alone records a request, and DR-2 alone sends a link to an empty page. Two checks come before DR-1: pull past send and answer dates from the OS to see whether owners stall at the shortlist step (two days), and ask the innovation manager at the pilot customer which decisions need the owner (one hour). One more comes before DR-2: send a test link to 10 addresses at that customer to see whether it survives the mail gateway and SSO (one week).

| Ticket | Title | What it delivers | Area | Skills | Size | Depends on | Status |
|---|---|---|---|---|---|---|---|
| DR-1 | Send and cancel requests from the shortlist | One button opens a dialog with the recipient, the owner preselected, and an optional note. Send creates a Waiting request and snapshots the shortlist. The header shows state and age. Cancel takes a reason. Resend to the person the owner named | Pain points | Backend, frontend | M | Nothing | Specified. Build after the stall check and the owner check |
| DR-2 | Email the owner a link that works without a login | One email, one button, a plain-text link. A hashed token that lasts 14 days. Nothing counts until the owner taps "Yes, I am {owner}". An expired link says to ask the user | Notifications, Shared | Backend, email deliverability (SPF, DKIM, DMARC), security | M | DR-1 | Waits on DR-1 and the link test |
| DR-3 | Let the owner answer on one page | A phone-first page with three choices per startup, or None of these fit with a reason. Answers land on the shortlist with one notification. The owner can name who should answer instead | Pain points, Shared | Frontend (phone-first form, accessibility), backend | M | DR-1, DR-2 | Waits on DR-2 |
| DR-4 | Remind the owner and mark requests Late | Two reminders, skipped if the owner was active in the last 24 hours. Late on day 8 with one alert to the user. Pause and resume, and a pending handoff pauses | Notifications | Backend (scheduled jobs) | M | DR-1, DR-2 | Waits on DR-2 |
| DR-5 | List open requests and measure the wait | The list of open requests with Remind now, Cancel, Answer on behalf, Pause and Resume. The two measures as queries over the event log | Pain points, Reports | Frontend, backend (permissions), SQL | M | DR-3, DR-4 | Waits on DR-4 |

Sizes are t-shirt sizes. The whole release is about 3 to 4 engineer-weeks. Each ticket is split into sub-issues, numbered DR-x.y in build order. Each has a story, what to build and acceptance criteria. A sub-issue is done when its acceptance criteria pass against the sample data, and a ticket when all its sub-issues are. Acceptance criteria count days from the send, day 0. A date appears only where the sample data's today matters.

Test against the sample data at https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/data/sample-data.json. In the sample data, today is 12 Oct 2026. It is invented test data, not a baseline. The tickets read it one way: each shortlist is a chain of one request, sent_to_owner_at is created, owner_feedback_at is answered when the status is complete, partial means still open, and reminders_sent counts chases. Read that way it has a median wait of 7 days over 11 answered and 8 of 13 answered by day 7. That meets the brief's switch condition, so a test run exercises it too.

## Rules for all five tickets

**Words used in the tickets**

- User means the person logged into the OS who sends the request, the innovation manager in the brief. Owner means the person the request goes to: the pain point owner unless the user picks someone else. Owners need no login.
- Copy in quotes is a template. {user} is the sender's full name, {owner} the recipient's full name, {named person} the person an owner names instead, {pain point} the pain point id, {due date} the due date, {date} a date, {n} a number.
- A request is one question from a user to one owner about one shortlist. This release has one question: rate this shortlist.
- States: Waiting, Paused, Late, Answered, Cancelled. Waiting, Paused and Late are open. An open request can become Answered or Cancelled. Those two are final.
- A chain is a request and its resends. Resend to a named person and Ask again both create a new request with parent_request_id set to the old one. The list's Age column is per request. The measures count chains from the first request's created, so the clock never stops across a resend.
- Owner activity means identity_confirmed, a view of the answer page with a session, handoff and answered. link_opened is logged for every fetch, scanners included, and is not activity. Activity drives the reminder skip in DR-4.1 and the Last owner action column in DR-5.1.

**Rules**

- One open request per shortlist. Enforce it in the database.
- Days are calendar days from the send date, in Europe/Berlin. Sent 29 Sep 2026: day 3 is 2 Oct, due 6 Oct at 23:59, link expires 13 Oct at 23:59. Age never resets.
- Jobs run at 09:00 on weekdays. Anything due on a weekend runs on Monday.
- Anyone who can edit the pain point can send, cancel, remind, pause, resume or answer on behalf. The event log records who.
- "Tell the user" means one in-app notification and one email. That is the workspace default in the sample data.
- A request closes only when a person answers or cancels it. Day 14 expires the link and nothing else. A Late request stays Late until the user acts.
- Finalising the pain point, which means it leaves Shortlist, cancels any open request with the reason "finalised". The owner gets no email, and the link shows "{user} no longer needs this".
- Reports go by request and business unit. No query returns an owner's name until the works council question in the supporting notes is answered.

**Data**

- One event table: request_id, type, at, actor, payload. DR-5.3 lists the types.
- The request record: id, shortlist_id, pain_point_id, sender_id, recipient_id, parent_request_id, state, note, created_at, due_at, expires_at, confirmed_at, answered_at, cancelled_at, cancel_reason, via (magic_link or on_behalf), rated, handoff_name, reminders_sent (automatic only), chases (Remind now), token_hash, and the snapshot (startups with description and comment). Each ticket names the fields it writes.
- Event payloads: created {recipient_id, parent_request_id}, link_opened {user_agent}, identity_confirmed {}, not_me {}, reminder_sent {n}, reminder_skipped {reason}, remind_now {}, paused {reason: user or handoff}, resumed {}, late {}, handoff {name}, answered {via, rated, choices}, cancelled {reason}.
- Sample data: the examples use two pain points from sample-data.json, load planning and supplier onboarding. The load planning request was sent on 29 Sep 2026, reminded on 2 and 6 Oct, Late since 7 Oct, day 13 today. The supplier onboarding request was sent on 18 Sep, open at day 24. Names come from the same file.

## Open questions

None of these blocks a build. The three checks that do are above, under The five tickets. Each question here has a default the tickets follow until it is answered. "The innovation manager" is the one at the pilot customer.

- Who can the user send to? Default: any OS user with an email address, the owner preselected. Ask the innovation manager how often the right person is outside the OS.
- Should finalising cancel the request? Default: yes, so nothing stays open on a closed pain point. Ask the innovation manager whether late answers are worth keeping.
- Does the pilot customer force SSO on outside links? Default: assume not, and the one-week link test settles it. Ask the customer's IT.
- One shared GlassDollar sending domain, or one per customer? Default: one shared domain, set up once. Ask the customer's IT.
- German or English? Default: English until the OS has a workspace language. Ask the innovation manager.
- Should the confirmation promise a reply time, say "within 3 working days"? Default: no promise. Ask the innovation manager.
- Should Need more info keep the request open? Default: no. It is Answered with rated false, and the user's Ask again starts a new request in the same chain, so the clock does not stop. Ask the innovation manager.
- Calendar or working days for 3, 7 and 8? Ask the innovation manager.
- Public holidays: default is to treat them as weekdays. Ask the innovation manager.
- Read-only access for the head of the business unit? Default: no. Ask the innovation manager.
- May reports name owners? Default: no, by request and business unit only, until the works council question in the supporting notes is answered. Ask the innovation manager who at the customer answers works council questions.

## Not in this release

Listed so the cut is explicit. Nothing here is in scope for the first release.

- Auto-reply and bounce detection, with reminders that pause on their own
- Link renewal after day 14
- Forwarding to a colleague with a fresh link in one step, and contacts for people outside the OS
- Answered and Cancelled tabs, and a unit-wide view of the list
- Merged reminder emails for one owner
- Suggested stage change
- Editing a request after sending
- Cancel and resend in one step when the shortlist or the pain point's owner changes. The user cancels and sends again
- First response and Late rate as measures. They wait until a quarter of data is in
- The closing-the-loop email
- The daily digest

---

### DR-1: Send and cancel requests from the shortlist

#### Description

The user has three to five startups on a shortlist and needs the owner's view on each. Today that is an email and a wait. After this ticket the user sends the request with one button, sees on the shortlist that it is out and how old it is, and can cancel or resend it. The owner gets no email until DR-2.

Writes: id, shortlist_id, pain_point_id, sender_id, recipient_id, parent_request_id, state, note, created_at, due_at, expires_at, cancelled_at, cancel_reason and the snapshot. Events: created, cancelled.

| Sub-issue | Title | Size | Depends on |
|---|---|---|---|
| DR-1.1 | Enable the user to send a request from the shortlist | S | - |
| DR-1.2 | Show the request's state and age on the shortlist | XS | DR-1.1 |
| DR-1.3 | Enable the user to cancel a request | XS | DR-1.2 |
| DR-1.4 | Enable the user to resend to the person the owner named | XS | DR-1.3 |

#### DR-1.1 Enable the user to send a request from the shortlist

**Story.** As a user, I want to ask the owner for a rating with one button, so that I stop writing chasing emails.

**What to build**

- Add the button "Ask {owner} to rate this shortlist" to the shortlist header while the pain point is in Shortlist. {owner} is the pain point owner.
- Disable the button under 3 or over 5 startups, with the text "Needs 3 to 5 startups".
- The button opens a dialog with: the recipient, preselected to the owner and changeable to any OS user with an email address; the email the recipient will get, shown as the template text with names filled in until DR-2 exists; an optional note to the recipient, plain text, 1,000 characters; and the button "Send request".
- If the owner is missing, is not an OS user or has no email address in the OS, open the dialog with no recipient and say why: "{owner} is not an OS user". The user picks someone or stops. The sender cannot be the recipient.
- Send creates the request as Waiting with created_at now, due_at on day 7 at 23:59 and expires_at on day 14 at 23:59, copies the shortlist into the snapshot (each startup's name, description and the user's comment), and logs created with recipient_id.
- Enforce one open request per shortlist in the database. A second Send on the same shortlist fails and shows "Already sent by {user}".

**Acceptance criteria**

- The user sends the load planning request. The request is Waiting, due on day 7 at 23:59 Europe/Berlin, the link expires on day 14 at 23:59, the snapshot holds Pallet.ai, Cargonaut Labs and Parcelgrid with descriptions and comments, and created is logged.
- A shortlist has 2 startups. The button is disabled and reads "Needs 3 to 5 startups".
- The user picks another OS user instead of the owner and sends. The request's recipient is that person.
- The pain point's owner is not an OS user. The dialog opens with no recipient and says so.
- Two users press Send on the same pain point within a second. One request exists. The second user sees "Already sent by {user}" with the first sender's name.
- A description or comment edited after sending: the snapshot stands. Nothing changes on the request.
- The note is empty: send without it. The email leaves the note out (DR-2.1).

#### DR-1.2 Show the request's state and age on the shortlist

**Story.** As a user, I want the shortlist to show that a request is out and how old it is, so that I know whether to wait or step in. As a second user covering for a colleague, I want to see that a request is already out, so that I do not send another.

**What to build**

- While a request is open, show its state and age under the title: a Waiting, Late or Paused label, then "{owner}, day {n}". Day 0 is the send date in Europe/Berlin.
- Show the same header to every user who can see the pain point.
- After an answer, DR-3.2 replaces this with the answers. After a cancel, bring the send button back.

**Acceptance criteria**

- The sample data on 12 Oct 2026. A second user opens the load planning shortlist. The header shows the Late label and "{owner}, day 13" with "Cancel request", and there is no send button. The supplier onboarding shortlist shows day 24 the same way.
- The request was sent today. Every user who can see the pain point sees the Waiting label and "{owner}, day 0" within the minute, and no email leaves the OS until DR-2 is built.
- The pain point's owner changes while a request is open: the request keeps its recipient and the header does not change. The user can cancel and send to the new owner.

#### DR-1.3 Enable the user to cancel a request

**Story.** As a user, I want to cancel a request I no longer need, so that the owner is not reminded about it.

**What to build**

- Add "Cancel request" next to the state. Ask for confirmation and an optional reason. Set Cancelled and cancelled_at, store the reason, and log cancelled with the reason. Send no email to the owner.
- When the pain point leaves Shortlist, cancel any open request with the reason "finalised". No email. The send button does not come back.
- A cancelled request's link shows "{user} no longer needs this" (DR-2.2).

**Acceptance criteria**

- The user cancels the load planning request with the reason "Owner is on leave". The state is Cancelled, the reason is stored, cancelled is logged, the owner gets no email, and the send button is back.
- The shortlist is finalised while the request is open. The request is Cancelled with the reason "finalised", cancelled is logged, the owner gets no email, and the button does not come back.
- Cancel on a request that was answered a second earlier: refuse with "Already answered". The answer stands.

#### DR-1.4 Enable the user to resend to the person the owner named

**Story.** As a user, I want to send the request to the person the owner named without typing it again.

**What to build**

- When the owner has named someone else (DR-3.3), show "Resend to {named person}" in the header.
- Resend cancels the open request with the reason "resent" and reopens the send dialog with that person preselected and the note filled in. Send creates a new request with parent_request_id set to the cancelled one. The old link then shows "{user} no longer needs this" (DR-2.2).

**Acceptance criteria**

- The load planning owner named someone else on day 2. The user presses "Resend to {named person}" and sends. The old request is Cancelled, the new one is Waiting for the named person from that day with its own day 7 due date, reminders_sent 0 and parent_request_id set to the old one.
- Send, resend and cancel on one shortlist in a row. The shortlist never has two open requests, each request has its own id, the resend's parent_request_id points at the cancelled one, the event log reads created, cancelled, created in that order, and every event carries the id of the user who acted.
- The named person is the sender, or is not an OS user: open the dialog with no recipient and say why, so the user picks someone.
- The new recipient already has a request open on another shortlist: allowed. The limit is one open request per shortlist, not per person.

---

### DR-2: Email the owner a link that works without a login

#### Description

The owner gets one email with one button. Tapping it opens a page that asks the owner to confirm who they are, then the answer page from DR-3. Corporate mail scanners open every link before a person does, so nothing counts until a person taps "Yes, I am {owner}". The link lasts 14 days and needs no account.

Writes: token_hash, confirmed_at. Events: link_opened, identity_confirmed, not_me.

| Sub-issue | Title | Size | Depends on |
|---|---|---|---|
| DR-2.1 | Send the request email | S | DR-1.1 |
| DR-2.2 | Issue a link that lasts 14 days without a login | S | DR-2.1 |
| DR-2.3 | Ask the owner to confirm identity before anything counts | S | DR-2.2 |

#### DR-2.1 Send the request email

**Story.** As an owner, I want one short email that says who is asking, what for and by when, so that I can decide at a glance whether to open it.

**What to build**

- Send the email when DR-1.1 creates the request. From "GlassDollar for {customer}" at a GlassDollar address. Reply-To is the user's address, so a human reply or an out-of-office reply reaches the user and not the OS. Bounces go to the sending address, which nobody reads in this release, so a wrong address shows up only as Late on day 8.
- Subject: "{pain point}: {user} needs your view on {n} startups". Body: who asks, what for, "about 3 minutes", the user's note if there is one, and the due date. One text button, "Review {n} startups", with the full URL in plain text under it. No startup names in the email.
- Set up SPF, DKIM and DMARC on the sending domain. Add "allowlist the sender" to the customer setup checklist.

**Acceptance criteria**

- The load planning email contains "Review 3 startups", "about 3 minutes" and the due date written out. It does not contain Pallet.ai, Cargonaut Labs or Parcelgrid.
- The user's note "Call me if anything is unclear." appears in the body. With no note, the body has no empty note block.
- Images are blocked. The button is readable text and the URL is visible under it.
- The owner replies to the email. The reply lands in the user's inbox.
- A test send to a mailbox outside GlassDollar passes SPF, DKIM and DMARC alignment.
- The owner's email address changes after sending: the sent email stays sent. The next email, a reminder, uses the new address.

#### DR-2.2 Issue a link that lasts 14 days without a login

**Story.** As an owner, I want the link to work again tomorrow and on my laptop, so that I am not locked out.

**What to build**

- The link carries a random token of 32 bytes or more. Store only its hash in token_hash, tied to the request and the recipient's address. Log link_opened with the user agent on every fetch.
- The token is valid until expires_at. The same link works from reminders, other devices and later days.
- Expired link: "This link has expired. Ask {user} for a new one." The user cancels and sends again (DR-1.3, DR-1.1) to issue a new one. Answered request: "Your answer was sent on {date}". Cancelled request: "{user} no longer needs this". None of these pages shows a startup.

**Acceptance criteria**

- The same link works on day 1, on day 7 from a reminder, and on day 14. On day 15 the page reads "This link has expired. Ask {user} for a new one." and nothing is sent.
- The user cancelled the load planning request. The owner opens the link and sees "{user} no longer needs this", with no identity page.
- The request was cancelled and resent to a named person (DR-1.4). The old link shows "{user} no longer needs this" and the new link works.
- The database holds no plain token.
- A link opened with a wrong or altered token: a plain "This link is not valid" page, with no hint about the request.

#### DR-2.3 Ask the owner to confirm identity before anything counts

**Story.** As an owner, I want one tap to say it is me, not a login. As a user, I want to know when someone other than the owner opened the link, so that I can check with the owner.

**What to build**

- Opening a valid link shows "Yes, I am {owner}" and "Not me". Nothing before the first tap counts as owner activity.
- The first tap records confirmed_at, logs identity_confirmed and sets a session cookie for this request until the link expires. With a session, skip this page.
- "Not me": show "Thanks, we've told {user}", change nothing on the request, log not_me, and tell the user once per request: "Someone other than {owner} opened the {pain point} link".

**Acceptance criteria**

- The owner taps the email button. The first tap shows the identity page, "Yes, I am {owner}" opens the answer page (a placeholder until DR-3), and confirmed_at is set once.
- Scanners fetch a new link ten times from three user agents with no tap. The request is still Waiting, confirmed_at is empty, link_opened is logged ten times, and no owner activity is logged.
- Nothing in the email, the token or any page reveals a startup name before the identity tap.
- The owner taps on a phone on day 2 and opens the link on a laptop on day 3. The laptop shows the identity page once, then the answer page. confirmed_at stays day 2.
- Someone else taps "Not me" on a forwarded email. That person sees no startup. The user is told "Someone other than {owner} opened the {pain point} link". A second "Not me" tells the user nothing more.
- A sandbox presses "Yes, I am {owner}": confirmed_at is set. Nothing more can happen without choices, and the measures count answers, not taps.
- Accepted risk, to be signed off by the innovation manager before launch: anyone holding the email can tap "Yes, I am {owner}" and see the startup names, descriptions and the user's comments. That is the trade for no login. "Not me" tells the user, and the link expires after 14 days.

---

### DR-3: Let the owner answer on one page

#### Description

After the identity tap the owner gets one page with the shortlisted startups and three choices each. The owner picks, adds a note where one is needed, and sends. The user sees the answers on the shortlist and gets one notification. The owner can also say that someone else should answer.

Writes: answered_at, via, rated, handoff_name. Events: answered, handoff, paused with the reason handoff.

| Sub-issue | Title | Size | Depends on |
|---|---|---|---|
| DR-3.1 | Enable the owner to rate the startups on one page | S | DR-2.3 |
| DR-3.2 | Show the answer on the shortlist and notify the user | XS | DR-3.1 |
| DR-3.3 | Enable the owner to hand the request to someone else | S | DR-3.2 |

#### DR-3.1 Enable the owner to rate the startups on one page

**Story.** As an owner, I want to rate three startups on my phone in a couple of minutes, so that the user gets my view without a meeting.

**What to build**

- One page, phone first, no navigation, behind the DR-2.3 session. Show the pain point title, the user's note, and one card per startup from the snapshot: name, description, the user's comment, the choices Interested, Not now and Need more info, and a comment box.
- Below the cards: a "None of these fit" checkbox with a comment box. Ticking it replaces the choices. Then the button "Send to {user}".
- Require a choice for every startup, a comment for every Need more info, and a comment for None of these fit. Keep Send enabled. On a missing field, say what is missing and move focus there. Send nothing until the page is complete.
- Send sets Answered, answered_at and via magic_link, and sets rated: true for None of these fit and for any Interested or Not now, false when every startup is Need more info. Log answered with via, rated and the choices.
- After sending, show "Sent to {user}." with a summary. The link opened again shows the answer read-only.
- Comments: 1,000 characters, with a counter from 100 left.

**Acceptance criteria**

- After the tap, the owner sees Pallet.ai, Cargonaut Labs and Parcelgrid with three choices each, the user's note, and no navigation.
- The owner rates 2 of 3, or ticks None of these fit without a comment, or sets Need more info without a comment, and presses Send. Nothing is sent. The page says what is missing and focus moves there.
- The owner sets Pallet.ai to Interested and the others to Not now and sends. The request is Answered, via is magic_link, rated is true, and answered is logged.
- All three are Need more info, each with a comment. The request is Answered and rated is false.
- The owner ticks None of these fit with the comment "All three are consumer tools" and sends. The request is Answered and rated is true.
- Five startups in the snapshot: five cards, same rules.
- No path to Answered stores a partial answer, including answer on behalf (DR-5.2): every startup has a choice or None of these fit is ticked, and every required comment is there.
- Every text on the owner's pages takes the user's and owner's names from the request. No name is hard-coded.
- Two tabs submit: the second sees "Already sent".
- Opened after a cancel, including a resend to a named person: "{user} no longer needs this". After an answer on behalf (DR-5.2): "Already answered by {user}".
- The user answers on behalf at 10:00 and the owner presses Send at 10:01: the owner sees "Already answered by {user}". The user's answer stands.

#### DR-3.2 Show the answer on the shortlist and notify the user

**Story.** As a user, I want the answers on the shortlist with one notification, so that I copy nothing out of an email.

**What to build**

- On the shortlist, show each startup's choice and comment, with who answered, when and how: "{owner}, {date}, via link". When rated is false, add "Not rated: needs more information".
- Tell the user once: "{owner} answered {pain point}", or "{owner} needs more information on {n} startups" when rated is false.
- Replace the state header with the button "Ask {owner} again". It opens the DR-1.1 dialog, and the new request gets parent_request_id set to the answered one. This is the follow-up to Need more info. The information itself goes by email or a call.
- An answer that lands after the pain point left Shortlist, which happens only when Send and the finalise cross, follows the cancellation rule: nothing is stored, and the owner sees "{user} no longer needs this".

**Acceptance criteria**

- The owner answers the load planning request with one Interested and two Not now. Within the minute the shortlist shows "{owner}, {date}, via link" with the day's date and the three choices, the user has one notification and one email, the request is Answered with rated true, and the button reads "Ask {owner} again".
- All three are Need more info. The notification reads "{owner} needs more information on 3 startups" and the shortlist shows "Not rated: needs more information".
- The user presses "Ask {owner} again" and sends. The new request is Waiting with parent_request_id set to the answered one, and the earlier answer stays visible under it.
- The shortlist is finalised at 10:00:00 and the owner's Send, started before, lands at 10:00:02. Nothing is stored, the request stays Cancelled with the reason "finalised", and the owner sees "{user} no longer needs this".
- Two users act on the same request at once: the first wins, the second sees "Already answered".

#### DR-3.3 Enable the owner to hand the request to someone else

**Story.** As an owner, I want to say that someone else should answer, so that requests stop landing on me when they are not mine.

**What to build**

- Under Send, add "Someone else should answer this" with a plain name field and the button "Tell {user}".
- Tell {user} stores handoff_name, logs handoff with the name, sets the request to Paused with the reason handoff (the DR-4.3 pause: no reminders and no Late while the user decides), and tells the user once: "{owner} says {named person} should answer {pain point}". The owner sees "Sent to {user}. They will send {pain point} to {named person}."
- On the shortlist header this shows as "Resend to {named person}" (DR-1.4). Resend or Resume ends the pause.
- If the owner answers after a handoff, the answer wins and handoff_name is cleared.
- A handoff on a Late request stays Late: handoff is logged, the header shows "Resend to {named person}", and nothing else changes.

**Acceptance criteria**

- The owner sends a handoff on day 2. The request is Paused with the reason handoff, handoff is logged, the user is told once, no reminder goes out on day 3, and reopening the link shows "Sent to {user}" with the named person.
- The owner sends a handoff on day 9, when the request is Late. It stays Late, handoff is logged, the user is told once, and the header shows "Resend to {named person}".
- The owner then rates all three on day 3. The request is Answered, handoff_name is cleared, and the header no longer offers "Resend to {named person}".
- The name field is empty and the owner presses Tell {user}. Nothing is sent and the field is marked.
- A handoff never closes the request and an answer always does, in whichever order they happen.
- The owner names the user: store it and tell the user anyway. The user decides.
- The owner hands off twice: the second name replaces the first, and both are logged.

---

### DR-4: Remind the owner and mark requests Late

#### Description

Today the user is the reminder system. This ticket moves that into the OS: two automatic reminders, a Late state on day 8 so the user knows when to step in, and a pause for when the owner is away.

Writes: reminders_sent, the states Late and Paused. Events: reminder_sent, reminder_skipped, late, paused, resumed.

| Sub-issue | Title | Size | Depends on |
|---|---|---|---|
| DR-4.1 | Send reminders on days 3 and 7 | S | DR-2.3 |
| DR-4.2 | Mark the request Late on day 8 and tell the user | XS | DR-4.1 |
| DR-4.3 | Enable the user to pause and resume reminders | S | DR-4.2 |

#### DR-4.1 Send reminders on days 3 and 7

**Story.** As a user, I want the OS to remind the owner twice, so that I only chase when that has failed.

**What to build**

- A job at 09:00 on weekdays, Europe/Berlin. On day 3 and day 7 of a Waiting request, send the first email again with "Reminder 1 of 2" or "Reminder 2 of 2" on top and the same link. Subject: "Reminder: {pain point}, {user} needs your view by {due date}". Never more than two automatic reminders per request.
- Skip the reminder, and log reminder_skipped with the reason, when the owner confirmed identity or opened the answer page in the last 24 hours, or when the user pressed Remind now (DR-5.2) in the last 24 hours.
- Make the job idempotent: right before sending, re-check that the request is Waiting, has fewer than two reminder_sent events, and the pain point is still in Shortlist. Log reminder_sent with n. Remind now never uses up an automatic reminder.
- A day that falls on a weekend runs on Monday.

**Acceptance criteria**

- No answer. Reminders go out on day 3 and day 7 at 09:00, and reminders_sent is 2.
- The owner opened the page on day 3 at 08:10. The 09:00 job sends nothing and logs reminder_skipped. The day 7 reminder still goes out.
- The user pressed Remind now on day 1. reminders_sent stays 0. The reminders still go out on day 3 and day 7.
- The shortlist is finalised on day 6. The request is Cancelled and no reminder goes out on day 7.
- The sample data on 12 Oct 2026. The job sends nothing.
- No reminder goes out for a request that is Paused, Late, Answered or Cancelled, or whose pain point has left Shortlist.
- The owner answers between the check and the send: abort.
- The job runs twice: one email, because the second run finds the reminder_sent event.
- Day 3 falls on a Saturday: Monday at 09:00.
- Remind now at 08:59 and the job at 09:00: skip the automatic reminder.

#### DR-4.2 Mark the request Late on day 8 and tell the user

**Story.** As a user, I want to be told on day 8, so that I know when to pick up the phone.

**What to build**

- In the same job, on day 8 or later with the request still Waiting, set Late, log late, and tell the user once: "{owner} has not answered {pain point} (day {n})".
- A Late request stays Late until the user acts. Nothing closes it.

**Acceptance criteria**

- No answer. On day 8 at 09:00 the state is Late, late is logged, the user gets one in-app notification and one email, and the event log reads created, reminder_sent 1, reminder_sent 2, late.
- The sample data on 12 Oct 2026. The load planning request stays Late and the user is not told again.
- Day 8 falls on a Sunday: Late on Monday.
- The owner answers on day 9: Answered. The Late state stays in the event log.

#### DR-4.3 Enable the user to pause and resume reminders

**Story.** As a user, I want to pause reminders when I know the owner is away, so that the owner does not come back to three nagging emails.

**What to build**

- Add "Pause reminders" on the shortlist header and in the DR-5.1 list for Waiting requests. It sets Paused and logs paused with the reason user.
- Paused stops reminders and the Late transition. A handoff (DR-3.3) pauses the same way with the reason handoff.
- "Resume reminders" sets Waiting, or Late at once if day 8 has passed, logs resumed, and tells the user once if it went Late. Do not send reminders missed while paused. Send the next one due as normal.
- Pause on a Late request: it stays Late and only the pause is logged. Resume changes nothing.

**Acceptance criteria**

- The user pauses on day 4. The state is Paused, paused is logged with the reason user, and no day 7 reminder goes out.
- The user resumes on day 5: Waiting, and the day 7 reminder goes out.
- The user resumes on day 10: Late at once, told once.
- The owner sends a handoff on day 2 and the user does nothing. No reminder on day 3 or day 7, never Late. The user presses Resume on day 10: Late at once, told once.
- Resume before day 7: send day 7, skip the missed day 3.
- Pause and resume on the same day: nothing is sent or skipped.

---

### DR-5: List open requests and measure the wait

#### Description

The user needs one place that shows every request still waiting on someone and lets them act from there. The product team needs to know whether any of this worked, so the same ticket adds the two measures as queries over the event log.

Writes: chases, via on_behalf. Events: remind_now, answered with via on_behalf.

| Sub-issue | Title | Size | Depends on |
|---|---|---|---|
| DR-5.1 | Show the user a list of open requests | S | DR-4.3 |
| DR-5.2 | Enable the user to act on a request from the list | S | DR-5.1 |
| DR-5.3 | Compute the two measures from the event log | S | DR-5.2 |

#### DR-5.1 Show the user a list of open requests

**Story.** As a user, I want one list of everything waiting on someone, so that nothing I own stalls without me knowing.

**What to build**

- Add the page "Waiting on others" under Pain points. It lists the open requests sent by the logged-in user. Answered and cancelled requests show on their shortlists, not here.
- Columns: Request, Owner, Age, Due, State, Reminders, Last owner action. Reminders shows reminders_sent, with "+1 manual" when chases is 1 or more. Last owner action is the latest owner activity from the rules. Show "-" in empty cells.
- Sort Late first, then oldest, then request id. Empty list: "Nothing is waiting on anyone."
- Each row links to its pain point.

**Acceptance criteria**

- The sample data on 12 Oct 2026. The list shows the two open requests only, every column filled or "-". The load planning request shows the owner's name, age 13, due 6 Oct, Late, 2 reminders, last owner action "-". The supplier onboarding request (age 24, Late) sorts above it.
- A second user sees only the requests they sent.
- No open requests: "Nothing is waiting on anyone."
- A request with reminders_sent 2 and chases 1 shows "2 +1 manual".
- The pain point's owner changed after the send: the Owner column shows the request's recipient, not the new owner.

#### DR-5.2 Enable the user to act on a request from the list

**Story.** As a user, I want to remind, cancel, pause or answer on behalf from the list, so that I do not open each pain point to act.

**What to build**

- Row actions for Waiting: Open shortlist, Remind now, Cancel, Answer on behalf, Pause reminders, and "Resend to {named person}" when the owner has named someone. Late: the same without Pause reminders. Paused: the Waiting set with Resume reminders instead of Pause, and Remind now warns "Reminders are paused".
- Remind now sends the DR-4.1 reminder email now, adds 1 to chases, and logs remind_now. It never uses up an automatic reminder. Then show "Reminded today" on the row for 24 hours.
- Cancel is DR-1.3's cancel. Pause and Resume are DR-4.3's. "Resend to {named person}" is DR-1.4's resend.
- Answer on behalf opens the DR-3.1 form for the user with a required note, default "Recorded by {user} from a call on {date}". Set via on_behalf. Send no email to the owner. The owner's link then shows "Already answered by {user}".
- Any user who can edit the pain point can take these actions. The event log records who.

**Acceptance criteria**

- The sample data on 12 Oct 2026. The user presses Remind now on the load planning request. reminders_sent stays 2, chases is 1, the Reminders cell reads "2 +1 manual", remind_now is logged, the owner gets the email, and the button reads "Reminded today" for 24 hours.
- The user answers on behalf for the load planning request with the default note. Answered, via on_behalf, the note is on the shortlist, the owner gets no email, and the owner's link shows "Already answered by {user}".
- The load planning request has a handoff. The user presses "Resend to {named person}" on day 2 and sends. The old request is Cancelled, the new one is Waiting for the named person from that day, the named person gets the DR-2.1 email, and the old owner gets nothing.
- The user pauses the load planning request from the list on day 4 and resumes on day 10: Paused, then Late at once, told once.
- A Late row offers no Pause reminders. A Waiting row does.
- A second user opens the load planning shortlist and presses Remind now there: remind_now is logged under that user's name.
- Every row action completes without leaving the list and writes the same record and event as the same action from the shortlist header.
- A user without edit rights on a pain point sees none of its row actions.
- Two users act on the same request at once: the first wins, the second sees "Already answered".
- Remind now on a Paused request: the warning shows and nothing is sent until the user confirms.

#### DR-5.3 Compute the two measures from the event log

**Story.** As the product team, we want the two measures as queries, so that after a quarter we know whether the wait got shorter.

**What to build**

- Event types in the table: created, link_opened, identity_confirmed, not_me, reminder_sent, reminder_skipped, remind_now, paused, resumed, late, handoff, answered, cancelled. Every sub-issue above writes its own.
- Two queries over the chains whose first request was created in a period. No screen yet. A chain cancelled before its due date is left out. A chain cancelled after day 7 with no rated answer counts as a miss for the day 7 share and is not in the median, so cancelling a late request does not hide it. No query returns an owner's name or user id; results go by request id and business unit.
  - Days to answer: the median days from the chain's first created to answered, over chains answered with rated true only, and the share of chains with a rated answer by day 7, where a chain without one counts as a miss once its due_at has passed.
  - No manual chase: the share of answered chains with no remind_now and no on_behalf answer. Follow-ups outside the product are not recorded and do not count.
- A chain is resolved by a rated answer or a cancel. A chain whose last request is Answered with rated false stays unresolved, and its clock runs until a rated answer.

**Acceptance criteria**

- The sample data, read as described under The five tickets: a median of 7 days over 11 answered, 8 of 13 by day 7, and 9 of 11 without a chase.
- A chain of two requests, the first sent on day 0 and resent on day 2, answered on day 4: Days to answer counts 4 days, not 2.
- The owner answers all three with Need more info on day 3 and the user never presses Ask again. On day 13 the chain is a miss for the day 7 share and it is not in the median.
- A query result lists chains by request id and business unit. No column holds an owner's name or user id.
- A chain cancelled on day 4 with no answer: left out of both measures. A chain cancelled on day 9 with no rated answer: a miss for the day 7 share, not in the median, and not in No manual chase.

---

## Prototype screenshots

From the [live prototype](https://iamdejman.github.io/glassdollar-casestudy-ayodeji/).

**The user sends one request from the shortlist.** The dialog shows who will get it, the due date, the reminder days and when the request goes Late.

<img src="images/send-dialog.png" width="560" alt="The send dialog: Ask someone to rate this shortlist, with the owner picked as the person who answers, the due date, the reminder days, a note field and a Send request button">

**The owner answers on a phone with no login.** One email with one button. A tap to confirm it is the owner stops mail scanners from answering for them. Then each startup gets one of three choices.

<img src="images/jonas-flow.png" width="900" alt="Three phone screens: the email with a Review 3 startups button, the identity page, and the answer page with three choices per startup">

**The user tracks everything in one list.** On day 8 the request goes Late and the user hears about it once. Each row shows owner, age, due date, state and reminders, with Remind now, Cancel and Answer on behalf.

<img src="images/waiting-on-others.png" width="900" alt="The Waiting on others list on day 8: both requests Late, with the notice that the owner has not answered">
