## 1. Email requirement

- [x] 1.1 In `EmailProfileCompletionRequirement.evaluate`, require both `isAllowRegistration()` and `isMustVerifyEmailOnRegistration()` before loading the user.
- [x] 1.2 Keep the blank-email and unverified-email checks and the reactive chain. Do not change the filter, security chain, public API, console schema, or stored verification flag.

## 2. Tests

- [x] 2.1 Enable registration in existing `ProfileCompletionFlowTest` cases that expect email enforcement, including the verification-disabled skip so it still proves that flag.
- [x] 2.2 Add the closed-registration regression for a missing email and an unverified email, and assert the user is not loaded.
- [x] 2.3 Cover all four registration and verification combinations: only both true evaluates email state.
- [x] 2.4 With both settings enabled, cover blank or whitespace email, a verified flag without an email, a non-blank unverified email, and a verified non-blank email.
- [x] 2.5 Run the real email requirement ahead of a second requirement and confirm closing registration does not bypass the later step.
- [x] 2.6 Propagate configuration-fetch failures, and user-service failures when enforcement runs. Do not turn those errors into completion.
- [x] 2.7 Enable registration in `OAuth2EmailCompletionFlowIntegrationTest`, which expects enforcement through the real requirement.

## 3. Validation

- [x] 3.1 Run `./gradlew :application:test --tests run.halo.app.security.profile.ProfileCompletionFlowTest --tests run.halo.app.security.ProfileCompletionFilterTest`.
- [x] 3.2 Run `./gradlew :application:test` and `./gradlew spotlessCheck`.

## Verification

- Focused `ProfileCompletionFlowTest` and `ProfileCompletionFilterTest` passed.
- `:application:spotlessJavaCheck` passed after formatting. Root `spotlessCheck` passed for repository sources. A gitignored `.osc-metadata/sync.json` is outside this change and is not formatted by the repository Spotless rules.
- `:application:test` completed 2123 tests. Two failures are existing timezone expectations and pass with `TZ=UTC`: `PostPermalinkPolicyTest.permalinkWithExternalUrl` and `PostFinderImplTest.archives`. They do not use the email requirement.
- No local Halo process was started, so the console toggle, browser session, and live PAT request were not exercised on a development server. The Spring filter integration test still returns `email-not-set` when registration and verification are both enabled, and the flow tests cover closed registration, reopening registration, later requirements, and error propagation.
