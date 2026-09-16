# Security Policy

ScopeMap is a native macOS app that connects to Jamf Pro and Jamf Platform environments using credentials you supply, mapping effective scope across every managed object — computers, groups, policies, profiles, apps, packages, scripts, and more. It is read-only by default, with an optional Destructive Mode that can delete objects surfaced in the Cleanup tab; any vulnerability affecting credential storage, network communication, or the authorization gates around delete operations could have fleet-wide impact.

For Jamf Concepts' broader security posture, see **[concepts.jamf.com/en/security](https://concepts.jamf.com/en/security/)**.

## Supported versions

Security fixes are applied to the latest released version. Please make sure you can reproduce an issue on the most recent release before reporting it.

## Reporting a vulnerability

**Please do not open a public GitHub issue for security vulnerabilities.**

Instead, report privately using one of:

- Jamf's [Vulnerability Disclosure Program](https://www.jamf.com/trust-center/vulnerability-disclosure/), or
- The **Report a vulnerability** button under this repository's **Security** tab.

Please include:

- A description of the issue and its potential impact.
- Steps to reproduce, or a proof of concept.
- The ScopeMap version and macOS version you observed it on.

You can expect an initial acknowledgement within a few business days. Once a fix is available, the report will be disclosed publicly with credit to the reporter, unless you ask to remain anonymous.

## Scope

In-scope examples: mishandling or leakage of stored credentials or tokens (Keychain storage, logging, or disk writes), insecure network handling (TLS certificate validation bypasses), or any path that lets ScopeMap modify or delete objects on a Jamf server without Destructive Mode being explicitly enabled — for example a delete request issued while the app is in its default read-only state, or a way to bypass the Destructive Mode confirmation gates.

Out of scope: vulnerabilities in Jamf Pro, the Jamf Platform API, macOS, or other third-party services. Please report those to the relevant vendor.
