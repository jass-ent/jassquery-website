# Legal review checklist — pre-launch

Internal tracking doc. **Not deployed** (lives in `docs/`, outside the `site/` deploy root).

These items were originally embedded as `<!-- LAWYER REVIEW -->` HTML comments inside
`site/privacy-policy.html`. They were moved here on the QueryForge privacy-policy pass so the
reminders survive a lawyer review without shipping internal notes in production page source
(Google's OAuth verification reviewers inspect the live page).

Resolve each before QueryForge go-live. Check the box when a lawyer (or you) has confirmed it.

## Privacy policy (`site/privacy-policy.html`)

- [ ] **§04 — Operational log retention.** Confirm the 90-day log retention stated in the policy
  matches what we actually configure in Azure Application Insights before going live. The policy
  text and the App Insights retention setting must agree.

- [ ] **§05 — Google Limited Use block.** The Limited Use disclosure should match the current
  Google API Services User Data Policy language at launch. Re-check
  https://developers.google.com/terms/api-services-user-data-policy **before submitting the Google
  OAuth app for verification.**

- [ ] **§06 — Microsoft publisher identity.** The policy assumes we complete Microsoft Publisher
  Verification (Microsoft AI Cloud Partner Program) before launch. If verification is still pending
  at launch, remove or soften the "publisher identity" paragraph. Also: if we ever enroll in the
  Microsoft 365 App Compliance Program (Publisher Attestation), the answers in Partner Center must
  match this policy exactly — keep them in lockstep.

- [ ] **§08 — Subprocessor DPAs / SCCs.** Confirm we have (or will have before launch) a signed DPA
  / SCCs in place for each subprocessor listed, especially for EU/UK data subjects.

- [ ] **§09 — EU representative (GDPR Art. 27).** If we expect to market to EU customers, consider
  whether we need an EU representative under GDPR Art. 27, and whether to publish their contact.

- [ ] **§18 — Physical mailing address.** Consider whether a physical mailing address is required
  (CCPA expects at least two methods for submitting requests; a toll-free number or physical address
  is common). If we want to keep this digital-only, confirm the email path satisfies that requirement.

## Terms of service (`site/terms-of-service.html`) — still has inline comments

Not part of the privacy-policy ticket, but flagged for the same treatment later. Four
The inline comments have been removed from the ToS; their substance is tracked here.

- [ ] Liability cap (US$100 / 12-months-of-fees) — confirm appropriateness and enforceability under CO.
- [x] **Venue — RESOLVED 2026-06-08.** Removed the county. §14 now names "the State of Colorado, USA"
  as the venue (no specific county). The LLC is Colorado-registered, so the state is the governing
  venue; binding it to a county was unnecessary and potentially inaccurate.
- [ ] Arbitration — decide whether to use mandatory binding arbitration (AAA Consumer Rules) vs. the
  drafted informal-then-court path; mind class-action-waiver interaction.
- [x] **Physical address — RESOLVED 2026-06-08. No physical address needed.** Researched all three
  drivers: (1) Google OAuth verification does not require a physical address — it requires privacy-policy
  disclosure, domain verification, and consent-screen branding only. (2) Microsoft publisher verification
  verifies the publisher domain, not an address. (3) CCPA has an online-business exception — a business
  operating exclusively online with a direct consumer relationship need only provide an email address for
  requests (no toll-free number, no mailing address). The policy already provides two emails (privacy@,
  help@). ToS §17's "registered business address on file with the Colorado Secretary of State" phrasing
  stays as-is. The ONLY future scenario needing a printed street address is formal DMCA-agent registration
  with the U.S. Copyright Office — optional, not a launch/Google/Microsoft requirement.

## Google OAuth verification submission (ticket 2026-04-19-infra-001, area 5)

Action items in Google Cloud Console — not website edits. Privacy policy is live and ready to reference.

- [ ] Google Cloud Console → APIs & Services → OAuth consent screen → Publish App
- [ ] Follow verification prompts; prepare a demo video showing the OAuth flow and how Drive data is used
- [ ] Confirm privacy policy URL and homepage URL are live and accessible at submission time
