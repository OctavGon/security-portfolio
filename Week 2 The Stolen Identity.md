# [The Stolen Identity]

## Scenario
Looked into a five-stage OAuth consent-phishing attack in an Azure environment. Examined two connected app registrations to understand how the attack worked, identify suspicious activity, and see how an attacker could gain unauthorized access to company resources.

## Environment
live multi-user Azure training tenant, Reader access

## Investigation

Step 1- A user was tricked by a phishing attack, completed MFA, and had their session token stolen. The stolen token showed that MFA had already been completed, allowing the attacker to bypass Conditional Access. The user also still had an Owner role on an older application due to outdated permissions. With that information i had a track to follow.

Step 2- Using the Owner permissions, the attacker created a new secret for the older application. This allowed them to access the application directly and use its permissions without needing to sign in as a regular user. The secret was also set to expire almost 100 years later.
<img width="1514" height="971" alt="Screenshot 2026-11-16 092739" src="https://github.com/user-attachments/assets/e2a22850-e26c-4365-b264-aa9650e403a4" />

Step 3- Since the first secret could eventually expire, the attacker created their own application and added it as an Owner of the older app. This gave them a way to create new secrets and keep access to the older application, even if the original secret was discovered and removed. Now i look for the rogue app created by the attacker.
<img width="1552" height="661" alt="Screenshot 2026-11-16 101711" src="https://github.com/user-attachments/assets/5562e403-1d21-4c4d-9f16-737e971a4f93" />

Step 4- As a backup plan, the attacker created a custom permission on the older application. This allowed the application to act as a resource that the attacker's own app could request access to.
<img width="1493" height="763" alt="Screenshot 2026-11-16 103344" src="https://github.com/user-attachments/assets/c6ed0d78-c278-4dad-a3c1-afdd57341af6" />

Step 5-Finally, the attacker set up a redirect link that sent users to a website they controlled. By combining the rogue app's ID, the redirect link, and the custom permission, they created a phishing link. When a signed-in user clicked the link and approved the access request, the authorization code was sent to the attacker's server.

## The method of attack
This is what we call a confused deputy attack. The legacy app was a trusted service that acted on a request is should have not never auithorized.

## What broke / what surprised me
The most credible section in the document. Dead ends, wrong guesses, the thing that took an hour. Employers know real work is messy. This section separates you from certificate collectors.
What i was surprised by was the ability for normal users to make big decisions with standard access like registering apps. It seems like a huge privilege given by accident or was overlooked. 
## Findings and recommendations
What you determined, plus 2 or 3 recommendations as if you were reporting to the resource owner.
What i would do or recommend doing after or future would be revoking the client secret, removing the rogue service principal from owners, 
## What I learned
3 to 5 bullets. At least one technical, one "what I'd do differently."
