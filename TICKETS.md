# Tickets: Business units submit a problem and then disappear

## The problem

A user sends the pain point owner a shortlist and waits. The owner has no login to hand, the user chases by email, and the pain point sits in Shortlist for weeks. The goal is fewer days between the send and the answer.

## What we are building

Decision requests. The user asks one question from the shortlist. The owner gets an email with one button and answers on one page in about three minutes, with no login. The OS reminds the owner twice and marks the request Late on day 8. The owner can name someone else, and the user resends to that person. One list shows the user every open request.

## The five tickets

Build and deploy them in this order. Nothing is useful before DR-3 is live: DR-1 alone records a request, and DR-2 alone sends a link to an empty page. Two checks come before DR-1: ask the innovation manager at the pilot customer which decisions need the owner (one hour), and send a test link to 10 addresses at that customer to see whether mail scanners and SSO let it through (one week).

| Ticket | Title | What it delivers | Area | Skills | Size | Depends on | Status |
|---|---|---|---|---|---|---|---|
| DR-1 | Send and cancel requests from the shortlist | One button opens a dialog with the recipient, the owner preselected, and an optional note. Send creates a Waiting request and snapshots the shortlist. The header shows state and age. Cancel takes a reason. Cancel and resend for a changed shortlist or a named person | Pain points | Backend, frontend | M | Nothing | Ready |
| DR-2 | Email the owner a link that works without a login | One email, one button, a plain-text link. A hashed token that lasts 14 days. Nothing counts until the owner taps "Yes, I am {owner}". An expired link says to ask the user | Notifications, Shared | Backend, email deliverability (SPF, DKIM, DMARC), security | M | DR-1 | Waits on DR-1 |
| DR-3 | Let the owner answer on one page | A phone-first page with three choices per startup, or None of these fit with a reason. Answers land on the shortlist with one notification. The owner can name who should answer instead | Pain points, Shared | Frontend (phone-first form, accessibility), backend | M | DR-1, DR-2 | Waits on DR-2 |
| DR-4 | Remind the owner and mark requests Late | Two reminders, skipped if the owner was active in the last 24 hours. Late on day 8 with one alert to the user. Pause and resume, and a pending handoff pauses | Notifications | Backend (scheduled jobs) | M | DR-1, DR-2 | Waits on DR-2 |
| DR-5 | List open requests and measure the wait | The list of open requests with Remind now, Cancel, Answer on behalf, Pause and Resume. The four measures as queries over the event log | Pain points, Reports | Frontend, backend (permissions), SQL | M | DR-3, DR-4 | Waits on DR-4 |

Sizes are t-shirt sizes. The whole release is about 3 to 4 engineer-weeks. Each ticket is split into sub-issues, numbered DR-x.y in build order, and each sub-issue has its own what to build and acceptance criteria. A sub-issue is done when its acceptance criteria pass against the sample data and its edge cases are handled. The ticket is done when its own acceptance criteria, which cover the whole flow, pass as well.

Test against the sample data at https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/data/sample-data.json. In the sample data, today is 12 Oct 2026. It is test data, not a baseline. Read as requests, it has a median wait of 7 days and 8 of 13 shortlists answered by day 7, which is the brief's switch condition, so a test run exercises that condition too.

## Rules for all five tickets

- User means the person logged into the OS who sends the request, the innovation manager in the brief. Owner means the person the request goes to: the pain point owner unless the user picks someone else. Owners need no login.
- Copy in quotes is a template. {user} is the sender's full name, {owner} the recipient's full name, {named person} the person an owner names instead, {pain point} the pain point id, {due date} the due date, {date} a date, {n} a number.
- A request is one question from a user to one owner about one shortlist. This release has one question: rate this shortlist.
- States: Waiting, Paused, Late, Answered, Cancelled. Waiting, Paused and Late are open. An open request can become Answered or Cancelled. Those two are final.
- One open request per shortlist. Enforce it in the database.
- Days are calendar days from the send date, in Europe/Berlin. Sent 29 Sep 2026: day 3 is 2 Oct, due 6 Oct at 23:59, link expires 13 Oct at 23:59. Age never resets.
- Jobs run at 09:00 on weekdays. Anything due on a weekend runs on Monday.
- Anyone who can edit the pain point can send, cancel, remind, pause, resume or answer on behalf. The event log records who.
- "Tell the user" means one in-app notification and one email. That is the workspace default in the sample data.
- A request closes only when a person answers or cancels it. Day 14 expires the link and nothing else. A Late request stays Late until the user acts.
- Finalising the pain point, which means it leaves Shortlist, cancels any open request with the reason "finalised". The owner gets no email, and the link shows "{user} no longer needs this".
- One event table: request_id, type, at, actor, payload. DR-5.3 lists the types.
- The request record: id, shortlist_id, pain_point_id, sender_id, recipient_id, parent_request_id, state, note, created_at, due_at, expires_at, confirmed_at, answered_at, cancelled_at, cancel_reason, via (magic_link or on_behalf), rated, late, handoff_name, reminders_sent (automatic only), chases (Remind now), token_hash, and the snapshot (startups with description and comment). Each ticket names the fields it writes.
- Event payloads: created {recipient_id, parent_request_id}, link_opened {user_agent}, identity_confirmed {}, not_me {}, reminder_sent {n}, reminder_skipped {reason}, remind_now {}, paused {reason: user or handoff}, resumed {}, late {}, handoff {name}, answered {via, rated, choices}, cancelled {reason}.
- A chain is a request and its resends. Cancel and resend, Resend to a named person and Ask again all create a new request with parent_request_id set to the old one. The list's Age column is per request. The measures count chains from the first request's created, so the clock never stops across a resend.
- Owner activity means identity_confirmed, a view of the answer page with a session, handoff and answered. link_opened is logged for every fetch, scanners included, and is not activity. Activity drives the reminder skip in DR-4.1 and the Last owner action column in DR-5.1.
- Reports go by request and business unit. No query returns an owner's name until the works council question in the brief is answered.
- Sample data: PP-034 was sent on 29 Sep 2026, reminded on 2 and 6 Oct, Late since 7 Oct, day 13 today. PP-040 was sent on 18 Sep, open at day 24. Names come from sample-data.json.

## Open questions

None of these blocks a build. Each has a default the tickets follow until it is answered. "The innovation manager" is the one at the pilot customer.

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
- May reports name owners? Default: no, by request and business unit only, until the works council question in the brief is answered. Ask the innovation manager who at the customer answers works council questions.

## Not in this release

Listed so the cut is explicit. Nothing here is in scope for the first release.

- Auto-reply and bounce detection, with reminders that pause on their own
- Link renewal after day 14
- One-click send-on with a fresh link, and contacts for people outside the OS
- Answered and Cancelled tabs, and a unit-wide view of the list
- Merged reminder emails for one owner
- Suggested stage change
- Editing a request after sending
- The closing-the-loop email
- The daily digest

The brief says why: https://github.com/IamDejman/glassdollar-casestudy-ayodeji/blob/main/PRODUCT-BRIEF.md.

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
| DR-1.4 | Enable the user to cancel and resend when the shortlist or owner changes | S | DR-1.3 |

#### Acceptance criteria for the whole ticket

- The user sends PP-034 on 29 Sep 2026. Every user who can see the pain point sees "Waiting for {owner}, day 0" within the minute, and no email leaves the OS, because DR-2 is not built yet.
- The sample data on 12 Oct 2026. PP-034 shows "Late: {owner}, day 13" and PP-040 shows "Late: {owner}, day 24", each with "Cancel request".
- Send, cancel and resend on one shortlist in a row. The shortlist never has two open requests, each request has its own id, the resend's parent_request_id points at the cancelled one, and the event log reads created, cancelled, created in that order.
- Every created and cancelled event carries the id of the user who acted.

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

- The user sends PP-034 on 29 Sep 2026. The request is Waiting, due 6 Oct at 23:59 Europe/Berlin, the link expires 13 Oct at 23:59, the snapshot holds Pallet.ai, Cargonaut Labs and Parcelgrid with descriptions and comments, and created is logged.
- A shortlist has 2 startups. The button is disabled and reads "Needs 3 to 5 startups".
- The user picks another OS user instead of the owner and sends. The request's recipient is that person.
- The owner of PP-017 is not an OS user. The dialog opens with no recipient and says so.
- Two users press Send on PP-034 within a second. One request exists. The second user sees "Already sent by {user}" with the first sender's name.

**Edge cases**

- A description or comment edited after sending: the snapshot stands. Nothing changes on the request.
- The note is empty: send without it. The email leaves the note out (DR-2.1).

#### DR-1.2 Show the request's state and age on the shortlist

**Story.** As a user, I want the shortlist to show that a request is out and how old it is, so that I know whether to wait or step in. As a second user covering for a colleague, I want to see that a request is already out, so that I do not send another.

**What to build**

- While a request is open, replace the send button with its state and age: "Waiting for {owner}, day {n}", "Late: {owner}, day {n}" or "Paused: {owner}, day {n}". Day 0 is the send date in Europe/Berlin.
- Show the same header to every user who can see the pain point.
- After an answer, DR-3.2 replaces this with the answers. After a cancel, bring the send button back.

**Acceptance criteria**

- A second user opens PP-034 on 12 Oct 2026. The header reads "Late: {owner}, day 13" with the owner's name, and there is no send button.
- The request was sent today. The header reads "Waiting for {owner}, day 0".

**Edge cases**

- The pain point's owner changes while a request is open: the request keeps its recipient. The header adds "Owner is now {named person}", and DR-1.4 adds the resend.

#### DR-1.3 Enable the user to cancel a request

**Story.** As a user, I want to cancel a request I no longer need, so that the owner is not reminded about it.

**What to build**

- Add "Cancel request" next to the state. Ask for confirmation and an optional reason. Set Cancelled and cancelled_at, store the reason, and log cancelled with the reason. Send no email to the owner.
- When the pain point leaves Shortlist, cancel any open request with the reason "finalised". No email. The send button does not come back.
- A cancelled request's link shows "{user} no longer needs this" (DR-2.2).

**Acceptance criteria**

- The user cancels PP-034 with the reason "Owner is on leave". The state is Cancelled, the reason is stored, cancelled is logged, the owner gets no email, and the send button is back.
- PP-034 is finalised on 12 Oct 2026 while the request is open. The request is Cancelled with the reason "finalised", cancelled is logged, the owner gets no email, and the button does not come back.

**Edge cases**

- Cancel on a request that was answered a second earlier: refuse with "Already answered". The answer stands.

#### DR-1.4 Enable the user to cancel and resend when the shortlist or owner changes

**Story.** As a user, I want to resend a changed shortlist, or send it to the person the owner named, without typing the request again.

**What to build**

- When a startup is added or removed while a request is open, show the banner "Shortlist changed since the request" with the button "Cancel and resend".
- Cancel and resend cancels the open request with the reason "resent" and reopens the send dialog with the note filled in. Send creates a new request with parent_request_id set to the cancelled one. The old link then shows "{user} no longer needs this" (DR-2.2).
- When the owner has named someone else (DR-3.3), show "Resend to {named person}" in the header. It is the same cancel and resend with that person preselected.
- When the pain point's owner changes while a request is open, show "Resend to {named person}" the same way, with the new owner preselected.

**Acceptance criteria**

- The user adds a fourth startup while PP-034 is open. The banner shows. Cancel and resend cancels the old request and opens the dialog with the note. Send creates a new Waiting request with 4 startups and parent_request_id set to the old one.
- The owner of PP-034 named someone else on 1 Oct 2026. The user presses "Resend to {named person}" and sends. The old request is Cancelled, the new one is Waiting for the named person from 1 Oct with due_at 8 Oct, reminders_sent 0 and parent_request_id set to the old one.

**Edge cases**

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

#### Acceptance criteria for the whole ticket

- The user sends PP-034. The owner's email arrives with one button, the first tap shows the identity page, "Yes, I am {owner}" opens the answer page (a placeholder until DR-3), and confirmed_at is set once.
- Ten fetches of the link by three different user agents with no tap leave the request Waiting with confirmed_at empty.
- The same link works on day 1, on day 7 from a reminder, and on day 13. On day 14 it shows the expired page.
- A test send to a mailbox outside GlassDollar passes SPF, DKIM and DMARC alignment.
- Nothing in the email, the token or any page reveals a startup name before the identity tap.

#### DR-2.1 Send the request email

**Story.** As an owner, I want one short email that says who is asking, what for and by when, so that I can decide at a glance whether to open it.

**What to build**

- Send the email when DR-1.1 creates the request. From "GlassDollar for {customer}" at a GlassDollar address. Reply-To is the user's address, so a human reply, an auto-reply or a bounce reaches the user and not the OS.
- Subject: "{pain point}: {user} needs your view on {n} startups". Body: who asks, what for, "about 3 minutes", the user's note if there is one, and the due date. One text button, "Review {n} startups", with the full URL in plain text under it. No startup names in the email.
- Set up SPF, DKIM and DMARC on the sending domain. Add "allowlist the sender" to the customer setup checklist.

**Acceptance criteria**

- The PP-034 email contains "Review 3 startups", "about 3 minutes" and 6 Oct 2026. It does not contain Pallet.ai, Cargonaut Labs or Parcelgrid.
- The user's note "Call me if anything is unclear." appears in the body. With no note, the body has no empty note block.
- Images are blocked. The button is readable text and the URL is visible under it.
- The owner replies to the email. The reply lands in the user's inbox.

**Edge cases**

- The owner's email address changes after sending: the sent email stays sent. The next email, a reminder, uses the new address.

#### DR-2.2 Issue a link that lasts 14 days without a login

**Story.** As an owner, I want the link to work again tomorrow and on my laptop, so that I am not locked out.

**What to build**

- The link carries a random token of 32 bytes or more. Store only its hash in token_hash, tied to the request and the recipient's address. Log link_opened with the user agent on every fetch.
- The token is valid until expires_at. The same link works from reminders, other devices and later days.
- Expired link: "This link has expired. Ask {user} for a new one." The user's cancel and resend (DR-1.4) issues the new one. Answered request: "Your answer was sent on {date}". Cancelled request: "{user} no longer needs this". None of these pages shows a startup.

**Acceptance criteria**

- The owner opens the link on 13 Oct 2026 and the page opens. On 14 Oct the page reads "This link has expired. Ask {user} for a new one." and nothing is sent.
- The user cancelled PP-034. The owner opens the link and sees "{user} no longer needs this", with no identity page.
- The request was cancelled and resent to a named person (DR-1.4). The old link shows "{user} no longer needs this" and the new link works.
- The database holds no plain token.

**Edge cases**

- A link opened with a wrong or altered token: a plain "This link is not valid" page, with no hint about the request.

#### DR-2.3 Ask the owner to confirm identity before anything counts

**Story.** As an owner, I want one tap to say it is me, not a login. As a user, I want to know when someone other than the owner opened the link, so that I can check with the owner.

**What to build**

- Opening a valid link shows "Yes, I am {owner}" and "Not me". Nothing before the first tap counts as owner activity.
- The first tap records confirmed_at, logs identity_confirmed and sets a session cookie for this request until the link expires. With a session, skip this page.
- "Not me": show "Thanks, we've told {user}", change nothing on the request, log not_me, and tell the user once per request: "Someone other than {owner} opened the {pain point} link".

**Acceptance criteria**

- A scanner fetches a new link three times. The request is still Waiting, confirmed_at is empty, link_opened is logged three times, and no owner activity is logged.
- The owner taps on a phone on 1 Oct 2026 and opens the link on a laptop on 2 Oct. The laptop shows the identity page once, then the answer page. confirmed_at stays 1 Oct.
- Someone else taps "Not me" on a forwarded email. That person sees no startup. The user is told "Someone other than {owner} opened the PP-034 link". A second "Not me" tells the user nothing more.

**Edge cases**

- A sandbox presses "Yes, I am {owner}": confirmed_at is set. Nothing more can happen without choices, and the measures count answers, not taps.
- Accepted risk, to be signed off by the innovation manager before launch: anyone holding the email can tap "Yes, I am {owner}" and see the startup names, descriptions and the user's comments. That is the trade for no login. "Not me" tells the user, and the link expires after 14 days.

---

### DR-3: Let the owner answer on one page

#### Description

After the identity tap the owner gets one page with the shortlisted startups and three choices each. The owner picks, adds a note where one is needed, and sends. The user sees the answers on the shortlist and gets one notification. The owner can also say that someone else should answer.

Writes: answered_at, via, rated, late, handoff_name. Events: answered, handoff, paused with the reason handoff.

| Sub-issue | Title | Size | Depends on |
|---|---|---|---|
| DR-3.1 | Enable the owner to rate the startups on one page | S | DR-2.3 |
| DR-3.2 | Show the answer on the shortlist and notify the user | XS | DR-3.1 |
| DR-3.3 | Enable the owner to hand the request to someone else | S | DR-3.2 |

#### Acceptance criteria for the whole ticket

- The owner opens the PP-034 link, confirms identity, rates three startups and sends. Within the minute the shortlist shows the three choices with "{owner}, {date}, via link", the user has one notification and one email, and the request is Answered with rated true.
- No partial answer is ever stored. Every path to Answered has a choice for every startup or None of these fit, and a comment wherever one is required.
- A handoff never closes the request and an answer always does, in whichever order they happen.
- Every text on the owner's pages takes the user's and owner's names from the request. No name is hard-coded.

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
- The owner sets Pallet.ai to Interested and the others to Not now and sends on 12 Oct 2026. The request is Answered, via is magic_link, rated is true, and answered is logged.
- All three are Need more info, each with a comment. The request is Answered and rated is false.
- The owner ticks None of these fit with the comment "All three are consumer tools" and sends. The request is Answered and rated is true.
- Five startups in the snapshot: five cards, same rules.

**Edge cases**

- Two tabs submit: the second sees "Already sent".
- Opened after a cancel, including a cancel and resend: "{user} no longer needs this". After an answer on behalf (DR-5.2): "Already answered by {user}".
- The user answers on behalf at 10:00 and the owner presses Send at 10:01: the owner sees "Already answered by {user}". The user's answer stands.

#### DR-3.2 Show the answer on the shortlist and notify the user

**Story.** As a user, I want the answers on the shortlist with one notification, so that I copy nothing out of an email.

**What to build**

- On the shortlist, show each startup's choice and comment, with who answered, when and how: "{owner}, {date}, via link". When rated is false, add "Not rated: needs more information".
- Tell the user once: "{owner} answered {pain point}", or "{owner} needs more information on {n} startups" when rated is false.
- Replace the state header with the button "Ask {owner} again". It opens the DR-1.1 dialog, and the new request gets parent_request_id set to the answered one. This is the follow-up to Need more info. The information itself goes by email or a call.
- An answer that lands after the pain point left Shortlist, which happens only when Send and the finalise cross: store it, set late true, and show "received after shortlist was finalised".

**Acceptance criteria**

- The owner answers PP-034 on 12 Oct 2026 with one Interested and two Not now. The shortlist shows "{owner}, 12 Oct 2026, via link" with the three choices, the user is told once, and the button reads "Ask {owner} again".
- All three are Need more info. The notification reads "{owner} needs more information on 3 startups" and the shortlist shows "Not rated: needs more information".
- The user presses "Ask {owner} again" and sends. The new request is Waiting with parent_request_id set to the answered one, and the earlier answer stays visible under it.
- PP-034 is finalised at 10:00:00 and the owner's Send, started before, lands at 10:00:02. The answer is stored, late is true, and the shortlist shows "received after shortlist was finalised".

**Edge cases**

- Two users act on the same request at once: the first wins, the second sees "Already answered".

#### DR-3.3 Enable the owner to hand the request to someone else

**Story.** As an owner, I want to say that someone else should answer, so that requests stop landing on me when they are not mine.

**What to build**

- Under Send, add "Someone else should answer this" with a plain name field and the button "Tell {user}".
- Tell {user} stores handoff_name, logs handoff with the name, sets the request to Paused with the reason handoff (the DR-4.3 pause: no reminders and no Late while the user decides), and tells the user once: "{owner} says {named person} should answer {pain point}". The owner sees "Sent to {user}. They will send {pain point} to {named person}."
- On the shortlist header this shows as "Resend to {named person}" (DR-1.4). Resend or Resume ends the pause.
- If the owner answers after a handoff, the answer wins and handoff_name is cleared.

**Acceptance criteria**

- The owner sends a handoff on 1 Oct 2026. The request is Paused with the reason handoff, handoff is logged, the user is told once, no reminder goes out on 2 Oct, and reopening the link shows "Sent to {user}" with the named person.
- The owner then rates all three on 2 Oct. The request is Answered, handoff_name is cleared, and the header no longer offers "Resend to {named person}".
- The name field is empty and the owner presses Tell {user}. Nothing is sent and the field is marked.

**Edge cases**

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

#### Acceptance criteria for the whole ticket

- Sent 29 Sep 2026 with no owner activity and no user action. The owner gets two emails after the first, on 2 Oct and 6 Oct at 09:00, the user is told once on 7 Oct, and the event log reads created, reminder_sent 1, reminder_sent 2, late.
- A pause at any point in that timeline stops both reminders and the Late transition. Resume continues from the current day without catching up.
- Running the job again on the same day changes nothing after the first run.
- No reminder goes out for a request that is Paused, Late, Answered or Cancelled, or whose pain point has left Shortlist.

#### DR-4.1 Send reminders on days 3 and 7

**Story.** As a user, I want the OS to remind the owner twice, so that I only chase when that has failed.

**What to build**

- A job at 09:00 on weekdays, Europe/Berlin. On day 3 and day 7 of a Waiting request, send the first email again with "Reminder 1 of 2" or "Reminder 2 of 2" on top and the same link. Subject: "Reminder: {pain point}, {user} needs your view by {due date}". Never more than two automatic reminders per request.
- Skip the reminder, and log reminder_skipped with the reason, when the owner confirmed identity or opened the answer page in the last 24 hours, or when the user pressed Remind now (DR-5.2) in the last 24 hours.
- Make the job idempotent: right before sending, re-check that the request is Waiting, has fewer than two reminder_sent events, and the pain point is still in Shortlist. Log reminder_sent with n. Remind now never uses up an automatic reminder.
- A day that falls on a weekend runs on Monday.

**Acceptance criteria**

- Sent 29 Sep 2026, no answer. Reminders go out on 2 Oct and 6 Oct at 09:00, and reminders_sent is 2.
- The owner opened the page on 2 Oct at 08:10. The 09:00 job sends nothing and logs reminder_skipped. The day 7 reminder still goes out on 6 Oct.
- The user pressed Remind now on 30 Sep. reminders_sent stays 0. The reminders still go out on 2 Oct and 6 Oct.
- PP-034 is finalised on 5 Oct. The request is Cancelled and no reminder goes out on 6 Oct.
- The sample data on 12 Oct 2026. The job sends nothing.

**Edge cases**

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

- Sent 29 Sep 2026, no answer. On 7 Oct at 09:00 the state is Late, late is logged, and the user gets one in-app notification and one email.
- The sample data on 12 Oct 2026. PP-034 stays Late and the user is not told again.

**Edge cases**

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

- The user pauses on 3 Oct 2026. The state is Paused, paused is logged with the reason user, and no day 7 reminder goes out.
- The user resumes on 4 Oct: Waiting, and the day 7 reminder goes out on 6 Oct.
- The user resumes on 9 Oct: Late at the next run, told once.
- The owner sends a handoff on 1 Oct and the user does nothing. No reminder on 2 Oct or 6 Oct, never Late. The user presses Resume on 9 Oct: Late at the next run, told once.

**Edge cases**

- Resume before day 7: send day 7, skip the missed day 3.
- Pause and resume on the same day: nothing is sent or skipped.

---

### DR-5: List open requests and measure the wait

#### Description

The user needs one place that shows every request still waiting on someone and lets them act from there. The product team needs to know whether any of this worked, so the same ticket adds the four measures as queries over the event log.

Writes: chases, via on_behalf. Events: remind_now, answered with via on_behalf.

| Sub-issue | Title | Size | Depends on |
|---|---|---|---|
| DR-5.1 | Show the user a list of open requests | S | DR-4.3 |
| DR-5.2 | Enable the user to act on a request from the list | S | DR-5.1 |
| DR-5.3 | Compute the four measures from the event log | S | DR-5.2 |

#### Acceptance criteria for the whole ticket

- The sample data on 12 Oct 2026. The list shows the open requests only, PP-040 then PP-034, with every column filled or "-", and every row action does what its sub-issue says without leaving the list.
- Every action from the list writes the same record and event as the same action from the shortlist header.
- The four queries run on the sample data return the figures in DR-5.3, with no owner name or user id in any column.
- A user without edit rights on a pain point sees none of its row actions.

#### DR-5.1 Show the user a list of open requests

**Story.** As a user, I want one list of everything waiting on someone, so that nothing I own stalls without me knowing.

**What to build**

- Add the page "Waiting on others" under Pain points. It lists the open requests sent by the logged-in user. Answered and cancelled requests show on their shortlists, not here.
- Columns: Request, Owner, Age, Due, State, Reminders, Last owner action. Reminders shows reminders_sent, with "+1 manual" when chases is 1 or more. Last owner action is the latest owner activity from the rules. Show "-" in empty cells.
- Sort Late first, then oldest, then request id. Empty list: "Nothing is waiting on anyone."
- Each row links to its pain point.

**Acceptance criteria**

- The sample data on 12 Oct 2026. PP-034 shows the owner's name, age 13, due 6 Oct, Late, 2 reminders, last owner action "-". PP-040 (age 24, Late) sorts above it.
- A second user sees only the requests they sent.
- No open requests: "Nothing is waiting on anyone."
- A request with reminders_sent 2 and chases 1 shows "2 +1 manual".

**Edge cases**

- The pain point's owner changed after the send: the Owner column shows the request's recipient, not the new owner.

#### DR-5.2 Enable the user to act on a request from the list

**Story.** As a user, I want to remind, cancel, pause or answer on behalf from the list, so that I do not open each pain point to act.

**What to build**

- Row actions for Waiting and Late: Open shortlist, Remind now, Cancel, Answer on behalf, Pause reminders, and "Resend to {named person}" when the owner has named someone. Paused: the same with Resume reminders instead of Pause, and Remind now warns "Reminders are paused".
- Remind now sends the DR-4.1 reminder email now, adds 1 to chases, and logs remind_now. It never uses up an automatic reminder. Then show "Reminded today" on the row for 24 hours.
- Cancel is DR-1.3's cancel. Pause and Resume are DR-4.3's. "Resend to {named person}" is DR-1.4's cancel and resend.
- Answer on behalf opens the DR-3.1 form for the user with a required note, default "Recorded by {user} from a call on {date}". Set via on_behalf. Send no email to the owner. The owner's link then shows "Already answered by {user}".
- Any user who can edit the pain point can take these actions. The event log records who.

**Acceptance criteria**

- The user presses Remind now on PP-034 on 12 Oct 2026. reminders_sent stays 2, chases is 1, the Reminders cell reads "2 +1 manual", remind_now is logged, the owner gets the email, and the button reads "Reminded today" for 24 hours.
- The user answers on behalf for PP-034 with the default note. Answered, via on_behalf, the note is on the shortlist, the owner gets no email, and the owner's link shows "Already answered by {user}".
- PP-034 has a handoff. The user presses "Resend to {named person}" on 1 Oct and sends. The old request is Cancelled, the new one is Waiting for the named person from 1 Oct, the named person gets the DR-2.1 email, and the old owner gets nothing.
- The user pauses PP-034 from the list on 3 Oct and resumes on 9 Oct: Paused, then Late at the next run, told once.
- A second user opens PP-034 from the pain point and presses Remind now there: remind_now is logged under that user's name.

**Edge cases**

- Two users act on the same request at once: the first wins, the second sees "Already answered".
- Remind now on a Paused request: the warning shows and nothing is sent until the user confirms.

#### DR-5.3 Compute the four measures from the event log

**Story.** As the product team, we want the four measures as queries, so that after a quarter we know whether the wait got shorter.

**What to build**

- Event types in the table: created, link_opened, identity_confirmed, not_me, reminder_sent, reminder_skipped, remind_now, paused, resumed, late, handoff, answered, cancelled. Every sub-issue above writes its own.
- Four queries over the chains whose first request was created in a period. No screen yet. Cancelled chains are left out. No query returns an owner's name or user id; results go by request id and business unit.
  - Days to answer: the median days from the chain's first created to answered over chains answered with rated true, and the share of chains answered by day 7, with open chains counted as misses.
  - First touch: the share of chains answered within one working day of the first created. This is the one measure in working days, as the brief defines it.
  - No manual chase: the share of answered chains with no remind_now and no on_behalf answer.
  - Late rate: the share of chains whose wait (answered minus first created, or today minus first created while open) is more than 1.2 times the business unit's median, or more than 8 days until the business unit has 10 answered chains.

**Acceptance criteria**

- The sample data read as chains of one request each (sent_to_owner_at is created, owner_feedback_at is answered when complete, partial is open, the sample data's reminders_sent are chases): a median of 7 days over 11 answered, 8 of 13 by day 7, 9 of 11 without a chase, and a late rate of 3 of 13 (PP-019, PP-034, PP-040).
- A chain of two requests, the first sent 29 Sep 2026 and resent 1 Oct, answered 3 Oct: Days to answer counts 4 days, not 2.
- A query result lists chains by request id and business unit. No column holds an owner's name or user id.

**Edge cases**

- A chain whose last request is cancelled with no answer: left out of all four measures.
- A business unit with fewer than 10 answered chains: Late rate uses the 8-day cutoff.

---

## Prototype screenshots

From the [live prototype](https://iamdejman.github.io/glassdollar-casestudy-ayodeji/).

**The user sends one request from the shortlist.** The dialog shows who will get it, the due date, the reminder days and when the request goes Late.

<img src="images/send-dialog.png" width="560" alt="The send dialog: Ask someone to rate this shortlist, with the owner picked as the person who answers, the due date, the reminder days, a note field and a Send request button">

**The owner answers on a phone with no login.** One email with one button. A tap to confirm it is the owner stops mail scanners from answering for them. Then each startup gets one of three choices.

<img src="images/jonas-flow.png" width="900" alt="Three phone screens: the email with a Review 3 startups button, the identity page, and the answer page with three choices per startup">

**The user tracks everything in one list.** On day 8 the request goes Late and the user hears about it once. Each row shows owner, age, due date, state and reminders, with Remind now, Cancel and Answer on behalf.

<img src="images/waiting-on-others.png" width="900" alt="The Waiting on others list on day 8: PP-040 and PP-034 both Late, with the notice that the owner has not answered PP-034">
