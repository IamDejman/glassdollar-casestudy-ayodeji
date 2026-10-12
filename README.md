# Decision requests prototype

This is the prototype from my GlassDollar case study. It tackles the third pain point on the list: business units submit a problem and then disappear. The idea is that nothing Lena owns stalls without her knowing, and that the owner can answer in about three minutes.

Try it here: https://iamdejman.github.io/glassdollar-casestudy-ayodeji/

The brief is in PRODUCT-BRIEF.md, and the detail cut from it is in SUPPORTING-NOTES.md. The five engineering tickets are in TICKETS.md, each split into sub-issues in build order with their own acceptance criteria, and mirrored to Linear.

The whole prototype is a single HTML file. You can open index.html from disk or use the link above.

On a desktop you see two screens side by side. On the left is Lena Hartmann's GlassDollar OS, where she manages pain points. Its sidebar, search bar, colours and type follow the app mockups on glassdollar.com/product. The sidebar is decoration; only the shortlist and the Waiting on others list work. On the right is Jonas Keller's phone. The two share one state, so what Lena sends shows up on Jonas's phone, and what Jonas answers shows up on Lena's shortlist. On a narrower screen you see one person at a time and can switch between them.

The green strip at the top guides you. It shows five steps, tells you what to do next and puts a ring around the control to press. The clock only appears when you reach the reminders step. "Start over" clears everything.

Here is the flow in five clicks:

1. As Lena, press "Ask Jonas Keller to rate this shortlist". Jonas is preselected as the person who answers. Press "Send request" and the email lands on his phone.
2. On the phone, open the email, press "Review 3 startups" and confirm it's you.
3. Rate the three startups and press "Send to Lena". The answers show up on her shortlist.
4. Press "Try the reminders" in the guide. A fresh request goes out and the clock appears. Day 3 and day 7 send reminders, and day 8 marks the request Late.
5. Press "Pause reminders" to pause them, then open "Waiting on others" to try Remind now, Cancel and Answer on behalf.

Instead of rating, Jonas can press "Someone else should answer this" and type a name. Lena then sees "Resend to [name]", which opens the send dialog with that person picked. Sending cancels the old request and starts a new clock.

Everything in it is sample data. State lives in memory, so "Start over" or a page reload starts you over.
