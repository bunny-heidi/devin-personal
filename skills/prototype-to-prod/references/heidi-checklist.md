# Heidi feature checklist

Use this in Step 4, after the failure walk. Items marked **Must answer** block the prototype (see the gate in `SKILL.md`). Ask the others when they apply.

Phrase each question for a non-technical reader. The names in `code` help you find the right files, so don't quote them at the requester unless it helps.

## 1. The idea

- **Must answer.** Which user has this problem, and how do they handle it today?
- **Must answer.** How will we know it worked? Name a number that should move.
- **Must answer.** What is out of scope for the first version?

## 2. Who sees it

Heidi has several layers of "who". Ask about each one, and build the audience table in the spec from the answers.

**Account type.** **Must answer.**
- Individual user with no team.
- Team member (`TeamMembershipRole.member`).
- Team owner or admin (`isTeamOwner`, `isTeamAdmin`).
- Organization global admin (`isGlobalAdmin`).
- Managed or enterprise customer (`is_managed`, `isEnterpriseTier`).

**Role.** Teams assign role categories: Admin, Clinician, Management, Support, View-Only. Can a View-Only or Support user see or use this? Does the feature need a permission (`src/helpers/casl/ability.ts`)?

**Plan tier and trial state.** **Must answer.**
- Free, Pro, Pro Plus, Enterprise, the Clinician variants, and Evidence (`SubscriptionTier`).
- Users in a trial, users whose trial just ended, and users with payment trouble (`PAST_DUE`, `CANCELED`).
- Does this feature sell an upgrade? If yes, what does an Enterprise user see instead?

**Entitlements.** Is the feature tied to an entitlement such as Evidence, Coder seat, Translator or Connectors (`ENTITLEMENT`)? Who grants it: the org, the team, or the user?

**Existing users.** What happens to people who are mid-flow, or have old data, on the day this ships?

## 3. Where they are

**Region.** **Must answer.**
- Heidi runs in AU, US, UK, EU, CA and CH, plus enterprise regions (any `-AZ1` region, such as UAE-AZ1 or US-AZ1).
- Does the feature behave differently by region? Does it depend on a vendor, an integration or a law that differs by country?
- Data stays in its region. Does the feature move or show data across regions?

**Language.** Heidi ships in 21 UI languages, including Arabic, which reads right to left. Will the layout survive longer text and right-to-left text? Customer-facing English uses American spelling.

## 4. Medical device and clinical safety

**Must answer.** Does the feature touch anything that generates, edits, shows or exports clinical content? Examples: notes, transcripts, templates, Evidence answers, EHR sync. If yes:

- The area is probably in medical-device scope (`/med-device.json`, owned by the medical device team). Changes there need their review and can affect regulatory documents.
- Could the feature lead a clinician to a wrong or missing note? Describe that failure in the spec.
- Does it touch an EHR integration (Epic, Cerner, Cliniko, Nookal, Gentu, T2Med and others)?

🚩 Confirm with the medical device team which questions they want here and who to name as the reviewer.

**Patient data.** Does the feature show, store, send or log patient or clinician information? Patient data must never reach analytics, logs or a third-party tool.

## 5. Platforms

**Must answer.** Which of these does it run on?

- Web.
- Desktop app on Mac and Windows. Desktop records audio natively, so browser permission prompts and "refresh the page" copy don't apply there.
- Mac App Store build. Apple rules ban Stripe and billing buttons (`isAppleCompliantBuild()`). Any upgrade or payment button needs a plan for this build.
- On-prem: hospitals running Heidi offline on their own servers. No third-party scripts or SaaS tools load there.
- Managed desktop: enterprises that deploy the desktop app through Intune or similar tools on a three-month release cycle. They get the feature months later than everyone else.
- iOS and Android. These apps are a separate codebase on the same backend. If the feature changes what the backend stores, mobile must do the same thing or the backend should own the rule.

## 6. Interruptions

Ask this for any modal, banner, popup, toast or spotlight that appears without a click.

- **Must answer, if it appears on its own.** Which tier is it? Heidi ranks interruptions T0 to T5 (`.ai-context/modal-priority-stack.md`):
  - During a recording, only safety messages may interrupt.
  - Managed and enterprise users never see revenue, onboarding or marketing prompts (T3 to T5).
  - Revenue prompts skip a user's first 3 sessions and show once per login.
  - Marketing prompts skip a user's first 7 days and stop after 2 dismissals.
- What else could show at the same moment, and which one wins?

## 7. Failures

You cover failures step by step in Step 3 with `references/unhappy-paths.md`. When an answer in this checklist creates a new failure (a tier that can't see the button, a platform that lacks the feature), add it to the Failure map before moving on.

## 8. Rollout

- **Must answer.** Who gets it first? Name the flag audience (region, tier, team, percentage).
- What does someone without the flag see?
- If it breaks, what do we switch off, and what happens to data already created?
- Growth features use PostHog flags; most others use Flagsmith. 🚩 Confirm which applies to the requester's team.

## 9. Measurement

- Which events tell you it worked? Describe them in plain words; engineering names them.
- New events go to Snowplow and need a schema. Don't promise Amplitude charts.

## 10. Copy

- Draft every piece of customer-facing text the feature needs, including error messages and empty states.
- Mark the drafts for review with the `microcopy` skill. Don't treat them as final.
