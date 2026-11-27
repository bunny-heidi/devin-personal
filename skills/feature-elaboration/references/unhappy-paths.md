# Unhappy paths and edge cases

Use this catalog in Step 3. Take one step of the happy path, run it through each category below, then move to the next step. Most steps hit at least four categories.

For every failure you find, record three things:

- **What the user sees.** The exact message, screen or state. "An error" isn't enough.
- **How they recover.** Retry, go back, contact support, or nothing.
- **What happens to their data.** Saved, lost, half-saved, or duplicated.

## 1. It fails

- The request fails. Does the user get a clear message and a retry?
- The request is slow: 5 seconds, then 30 seconds. What shows while they wait?
- Part of it works. Three of five items save, or the save works but the confirmation email doesn't send.
- The user clicks twice, or retries after a timeout that succeeded. Do we create a duplicate?
- A third party is down: an EHR, Stripe, an email provider, an AI model.

## 2. The user leaves partway

- They close the tab, quit the desktop app or refresh. Do they lose their work? Can they pick up where they left off?
- They press the browser back button.
- Their login expires, or Heidi logs them out for inactivity.
- They go offline, on web or desktop, and come back five minutes later.
- They start on one device and continue on another (web and desktop, or two tabs).

## 3. Something changes underneath them

- Their trial ends, or their plan downgrades, during the flow.
- Their payment fails and the account moves to past due.
- An admin removes them from the team or changes their role.
- A teammate edits or deletes the same template, session or setting at the same time.
- The item they're working on gets deleted somewhere else.
- The feature flag turns off mid-session, or they're in the rollout on web but not on desktop.
- Heidi releases a new version while they're mid-flow.

## 4. They can't do it

- Their role doesn't allow it (View-Only, Support, a member instead of an admin).
- Their plan doesn't include it. What do they see instead of the feature, and does it sell an upgrade they can't buy (Mac App Store, Enterprise)?
- Their platform doesn't support it: on-prem, an older managed desktop version, mobile.
- Their region doesn't support it, or the vendor behind it doesn't operate there.
- Their org admin turned it off with an org-level setting.

## 5. Recording and clinical context

- A recording is in progress. Does this interrupt it, slow it, or compete with it for the screen?
- The microphone drops, or the desktop app loses audio, during this flow.
- The note is still generating when this happens.
- The clinician is in front of a patient. Is this safe to show right now?
- The feature changes clinical content. Can it cause a wrong, missing or duplicated note? Can the clinician tell?

## 6. The data is unusual

- No data yet: a brand-new user, an empty team, the first session.
- A lot of data: hundreds of templates, a two-hour recording, a team of 500.
- Old data created before this feature existed, or under an old version of it.
- Long or unusual text: a 60-character name, emoji, Arabic, a pasted Word document.
- Missing fields: no specialty, no team, no patient name.

## 7. It happens again, or not at all

- The user sees it a second time. Should they? After dismissing it? After completing it?
- The user never responds. Does it nag, expire or wait forever?
- Two prompts or banners want the same moment. Which one wins? (See the T0 to T5 rules in the checklist.)
- The user sees it on web, then again on desktop. Does dismissing it on one count for the other?

## 8. Afterwards

- The user wants to undo it. Can they?
- We switch the feature off. What happens to the data and settings people already created?
- The user leaves the team, deletes their account or changes region. What happens to what they made?
- Support gets a ticket about it. Can support see what happened, without seeing patient data?
