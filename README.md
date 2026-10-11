# Decision requests prototype

This is the prototype from my GlassDollar case study. It tackles the third pain point on the list: business units submit a problem and then disappear. The idea is to get an answer from the business unit owner.

Try it here: https://iamdejman.github.io/glassdollar-casestudy-ayodeji/

The whole prototype is a single HTML file. You can open index.html from disk or use the link above.

On a desktop you see two screens side by side. On the left is Lena Hartmann's GlassDollar OS, where she manages pain points. Its sidebar, search bar, colours and type follow the app mockups on glassdollar.com/product. The sidebar is decoration; only the shortlist and the Waiting on others list work. On the right is Jonas Keller's phone. The two share one state, so what Lena sends shows up on Jonas's phone, and what Jonas answers shows up on Lena's shortlist. On a narrower screen you see one person at a time and can switch between them.

The green strip at the top guides you. It shows five steps, tells you what to do next and puts a ring around the control to press. The clock and the out-of-office switch only appear when you reach the reminders step. "Start over" clears everything.

Here is the flow in five clicks:

1. As Lena, press "Ask Jonas Keller to rate this shortlist", then "Send request". The email lands on Jonas's phone.
2. On the phone, open the email, press "Review 3 startups" and confirm it's you.
3. Rate the three startups and press "Send to Lena". The answers show up on her shortlist.
4. Press "Try the reminders" in the guide. A fresh request goes out and the clock appears. Day 3 and day 7 send reminders, and day 8 marks the request Late.
5. Turn on "Jonas is out of office" to pause reminders, then open "Waiting on others" to try Remind now, Cancel and Answer on behalf.

Everything in it is sample data. State lives in memory, so "Start over" or a page reload starts you over.
