## Context

`EmailProfileCompletionRequirement` loads the user setting group and, when `mustVerifyEmailOnRegistration` is true, loads the user. A blank or unverified email becomes the `/complete-profile` step with problem type `email-not-set`. `ProfileCompletionFilter` applies that step to authenticated users who are not super-role, including PAT calls to protected APIs, before ordinary authorization.

The user-setting form shows `mustVerifyEmailOnRegistration` only while `allowRegistration` is true. Closing registration hides the checkbox and leaves the saved boolean unchanged. Sites that enabled verification, then closed registration, keep blocking every existing user who has no verified email, with no visible way to turn the check off except temporarily reopening registration.

## Goals / Non-Goals

**Goals:**

- Adopt the reporter's first remedy: require both `isAllowRegistration()` and `isMustVerifyEmailOnRegistration()` before loading the user.
- Keep blank, whitespace, and unverified email checks when both settings are enabled.
- Leave the stored verification flag untouched so opening registration again restores enforcement.
- Keep later profile-completion requirements in effect when the email requirement does not apply.
- Propagate configuration-fetch and user-load errors instead of treating them as permission to continue.

**Non-Goals:**

- Do not add a public setting that independently blocks unverified accounts.
- Do not change `ProfileCompletionFilter`, the security filter chain, roles, permissions, or the console form.
- Do not limit enforcement to newly registered users only.
- Do not clear or migrate `mustVerifyEmailOnRegistration`.

## Decisions

### 1. Gate enforcement on both existing booleans

- **Choice**: In `evaluate`, filter the fetched `SystemSetting.User` with `isAllowRegistration() && isMustVerifyEmailOnRegistration()` before `userService.getUser`. Keep the current email predicate and `map` to the existing step.
- **Rationale**: This is the reporter's first acceptable remedy. Both flags already exist. The console hides verification precisely when registration is closed, so tying enforcement to that visibility matches the setting administrators can see. Skipping the user load avoids work and avoids treating a user-service failure as relevant when the policy is off.
- **Alternative considered**: Add a new "block unverified accounts" option, as suggested in the issue discussion. Rejected for this fix. It would be a new public setting and a security-configuration change, and it needs explicit maintainer approval rather than being folded into the bugfix.
- **Alternative considered**: Apply the check only to users created through signup or OAuth2 registration. Rejected because it needs new user-origin state and would leave the hidden setting able to block some accounts without a visible control.

### 2. Do not write the verification flag

- **Choice**: Read both booleans and do not update system configuration when registration is closed.
- **Rationale**: Reopening registration must restore enforcement without an administrator setting the hidden checkbox again. The policy applies to existing users and PATs as well as new registrations: closing registration removes this email gate for current non-super-admin sessions, and opening it again with the flag still true blocks unverified users again.

### 3. Leave the filter and security chain alone

- **Choice**: Do not change matchers, super-role exemption, problem details, or redirects.
- **Rationale**: `ProfileCompletionFlow.findNext` already orders this requirement ahead of any later requirements. When the email requirement completes empty, the flow still returns the next real step. When every requirement is empty, the filter continues the chain and ordinary authorization decides access. Removing the email step is not an authorization grant.

## Risks / Trade-offs

- **[Risk] Sites that depended on the hidden flag to block existing unverified users will stop blocking them while registration is closed.**
  - **Mitigation**: That is the reported bug. The stored flag remains, so enabling registration again turns the same block back on. Document the effect in the user-facing release note.
- **[Risk] Maintainers may prefer an independent block that stays on after registration is closed.**
  - **Mitigation**: Record that option here and do not implement it in this change. Any new setting or security-configuration change waits for repository-required approval.
- **[Risk] Callers could mistake a completed email requirement for access to the requested endpoint.**
  - **Mitigation**: The filter only stops inserting the completion response. Role and permission checks that already follow it are unchanged.

## Migration Plan

No data migration. Existing `allowRegistration` and `mustVerifyEmailOnRegistration` values stay in the user setting group. Rollback is reverting the predicate in `EmailProfileCompletionRequirement`.

User-facing release note for the pull request template:

```release-note
Closing registration no longer blocks existing users or personal access tokens that lack a verified email when email verification on registration was left enabled. Opening registration again with that setting still enabled blocks those users again. Endpoint permissions are unchanged.
```

AI assistance materially contributed to this change and should be disclosed in the pull request description. The pull request targets `main` and uses the upstream template: type Bug fix, `Fixes #10356`.

## Open Questions

Whether maintainers want a separate control for blocking unverified accounts, independent of registration, remains open. This change does not add that control. Do not broaden the fix into a new public setting or security-configuration change without the repository-required approval.
