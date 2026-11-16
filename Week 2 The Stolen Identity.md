# [The Stolen Identity, "Investigating an Identity Attack in Entra ID"]

## Scenario
Looked into a five-stage OAuth consent-phishing attack in an Azure environment. Examined two connected app registrations to understand how the attack worked, identify suspicious activity, and see how an attacker could gain unauthorized access to company resources.

## Environment
One list: platform, services, tools, access level. Honest framing: "live multi-user Azure training tenant, Reader access."

## Investigation
The core. Numbered steps IN YOUR OWN WORDS: what you looked at, what you found, what you concluded at each step. 6 to 12 screenshots of meaningful moments (portal views, query results, before/after).
Step 1- A user was tricked by a phishing attack, completed MFA, and had their session token stolen. The stolen token showed that MFA had already been completed, allowing the attacker to bypass Conditional Access. The user also still had an Owner role on an older application due to outdated permissions. With that information i had a track to follow.

Step 2- Using the Owner permissions, the attacker created a new secret for the older application. This allowed them to access the application directly and use its permissions without needing to sign in as a regular user. The secret was also set to expire almost 100 years later.

## What broke / what surprised me
The most credible section in the document. Dead ends, wrong guesses, the thing that took an hour. Employers know real work is messy. This section separates you from certificate collectors.

## Findings and recommendations
What you determined, plus 2 or 3 recommendations as if you were reporting to the resource owner.

## What I learned
3 to 5 bullets. At least one technical, one "what I'd do differently."
