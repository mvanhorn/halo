## Purpose

Define when a missing or unverified email forces profile completion, and how that rule follows the registration settings administrators can see.

## ADDED Requirements

### Requirement: Email completion follows both registration settings

The system SHALL produce the email profile-completion step only when both `allowRegistration` and `mustVerifyEmailOnRegistration` are true. When either setting is false, the system SHALL produce no email step and SHALL NOT load the user for this requirement. Closing registration SHALL NOT change the stored `mustVerifyEmailOnRegistration` value. Opening registration again while that stored value remains true SHALL enforce the email step again.

This rule applies to existing non-super-admin users and personal access tokens, not only to accounts created while registration is open. Super-role exemption stays in the profile-completion filter.

#### Scenario: Closed registration skips a missing or unverified email

- **WHEN** registration is disabled and email verification on registration is enabled
- **AND** the user has no email or has an email that is not verified
- **THEN** profile completion produces no email step
- **AND** the user record is not loaded for this requirement
