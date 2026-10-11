# Decision requests prototype

This is the prototype from my GlassDollar case study. It tackles the third pain point on the list: business units submit a problem and then disappear. The idea is to get an answer from the business unit owner without asking them to log in to anything.

Try it here: https://iamdejman.github.io/glassdollar-casestudy-ayodeji/

The whole prototype is a single HTML file. There is nothing to build and nothing to sign in to. You can open index.html from disk or use the link above.

On a desktop you see two screens side by side. On the left is Lena Hartmann's GlassDollar OS, where she manages pain points. On the right is Jonas Keller's phone. The two share one state, so what Lena sends shows up on Jonas's phone, and what Jonas answers shows up on Lena's shortlist. On a narrower screen you see one person at a time and can switch between them.

Here is the flow in five clicks:

1. As Lena, press "Ask Jonas Keller to rate this shortlist", then "Send request". The email lands on Jonas's phone.
2. On the phone, open the email, press "Review 3 startups" and confirm it's you.
3. Rate the three startups and press "Send to Lena". The answers show up on her shortlist.
4. Reset, send again, then use the clock. Day 3 and day 7 send reminders, and day 8 marks the request Late.
5. Turn on "Jonas is out of office" to pause reminders, then open "Waiting on others" to try Remind now, Cancel and Answer on behalf.

Everything in it is sample data. State lives in memory, so Reset or a page reload starts you over.
