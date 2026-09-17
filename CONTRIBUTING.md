# Contributing to ScopeMap

ScopeMap's source stays in this private repository — it is not open source,
and there is no public source repo to send pull requests against. This guide
is for Jamf engineers contributing internally.

## Before you start

ScopeMap is **read-only by default**: outside of one clearly bounded feature it
never creates, modifies, or deletes anything on a Jamf server. The single
exception is the optional **Destructive Mode**, which is off by default, hidden
behind an explicit opt-in in Preferences → Advanced, and gated by a
confirmation dialog before any delete. Contributions that add new write or
delete behavior are out of scope unless they extend that existing Destructive
Mode and preserve its safeguards — off by default, hidden until enabled, and
never destructive without an explicit user confirmation. New write paths that
run in the app's default read-only state will not be merged.

## Reporting bugs and requesting features

Open an issue against this repository. For bugs, please include:

- What you expected to happen and what actually happened.
- Steps to reproduce.
- Your ScopeMap version, macOS version (and Apple silicon vs. Intel), and the
  relevant Jamf Pro version if it matters.
- Screenshots for anything visual (the graph, Compare, Cleanup, etc.).

Please **do not** paste real server URLs, credentials, device names, usernames,
or other identifying data from your environment into issues. Redact first.

For security issues, follow [SECURITY.md](SECURITY.md) instead of opening a
public issue.

## Development setup

1. Clone the repository and open `ScopeMap.xcodeproj` in Xcode 16 or later.
2. In **Signing & Capabilities** for the `ScopeMap` target, set the Team to your
   own Apple Developer team.
3. Build and run with `⌘R`. There are no external dependencies — the app is pure
   Swift/SwiftUI.

Requires macOS 14.0 or later.

## Making changes

- **Branch** off `main` for your work.
- **Match the existing style.** Keep the project warning-clean and follow the
  conventions already in the surrounding code.
- **Keep concurrency correct.** The app does meaningful work off the main actor
  (layout, pagination, parsing); preserve actor isolation and avoid introducing
  main-thread stalls.
- **Add or update tests** in `ScopeMapTests` for logic changes, especially scope
  resolution and cleanup rules. Run the full test suite before submitting
  (`⌘U`).
- **Update the README** if you change behavior, requirements, or supported
  object types.

## Submitting a pull request

1. Make sure the project builds and all tests pass.
2. Open a PR against `main` with a clear description of what changed and why.
   Link any related issue.
3. Include before/after screenshots for UI changes.
4. Keep PRs focused — one logical change per PR is easier to review than a large
   mixed one.

ScopeMap is distributed under the [Jamf Concepts Use Agreement](LICENSE).
