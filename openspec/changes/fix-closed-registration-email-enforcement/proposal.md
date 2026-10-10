## Why

Halo 2.26 applies `mustVerifyEmailOnRegistration` to existing non-super-admin users, including personal access token (PAT) requests. The console hides that checkbox whenever registration is closed (`if: "$get(allowRegistration).value === true"`) but keeps the saved value. Users without a verified email then receive HTTP 403 with type `email-not-set`, and the control that would turn the check off is no longer visible. Issue #10356.

The reporter's first acceptable remedy is to make `EmailProfileCompletionRequirement` also honor `allowRegistration`, so the hidden verification setting does not apply while registration is closed. The other remedies they listed (limit the check to newly registered users, or stop hiding the checkbox) are not this change. A maintainer suggested a separate option for whether unverified accounts are blocked. That suggestion is recorded as a tradeoff, not as work in this change.

## What Changes

- Evaluate the email profile-completion requirement only when both `allowRegistration` and `mustVerifyEmailOnRegistration` are true, and only then load the user.
- Keep the existing blank-email and unverified-email checks, the reactive chain, and the stored verification flag.
- Do not add a setting, change the profile-completion filter, security chain, public API, console schema, or persisted configuration.

## Capabilities

### New Capabilities

- `registration-email-enforcement`: Email profile completion follows the visible registration policy. Closing registration stops the hidden verification setting from blocking existing users, and opening registration again restores that block without rewriting the flag.

### Modified Capabilities

- None.

## Impact

- `EmailProfileCompletionRequirement` is the only production change. `ProfileCompletionFlow` and `ProfileCompletionFilter` already call it, so callers, filter matchers, and security configuration stay as they are.
- Authorization: the filter still runs for authenticated non-super-admin sessions, including PATs, and still returns the `email-not-set` problem or a completion redirect when a requirement produces a step. This change only stops the email requirement from producing that step unless registration is open and verification is required. The request then continues to ordinary authorization. Absence of the email step does not grant endpoint permission. Super-role exemption, anonymous handling, and problem responses stay in the filter.
- Existing users and PATs are affected the same way as new registrations: closing registration lifts this email gate for every non-super-admin user; reopening registration with the stored verification flag still true puts the gate back.
- No plugin or theme API change. `ProfileCompletionRequirement` is unchanged, and later requirements still run.
- No database migration, new dependency, or i18n change. The console already hides the verification checkbox when registration is closed, and this change does not alter that form.
- A separate "block unverified accounts" setting would be a new public control and a security-configuration change. It needs maintainer approval before implementation and is not part of this fix.
