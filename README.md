# ScopeMap

ScopeMap is a native macOS app that answers the question Jamf Pro's web interface can't: what actually reaches a device, and why. It scans a Jamf Pro environment, models every object that participates in scoping — computers and mobile devices, groups, policies, profiles, apps, packages, scripts, users, and more — and evaluates effective scope across all of them so you can see blast radius, trace a single device's delivery path, and find the cruft that's accumulated.

![The Map tab — an interactive graph of a Jamf Pro environment, showing smart groups, policies, and configuration profiles connected to the devices they target](docs/images/01-map.png)

- [Features](#features)
- [Installing](#installing)
- [Requirements](#requirements)
- [Connecting to a server](#connecting-to-a-server)
- [Jamf Pro API privileges](#jamf-pro-api-privileges)
- [What ScopeMap can and can't evaluate](#what-scopemap-can-and-cant-evaluate)
- [Deleting objects (Destructive Mode)](#deleting-objects-destructive-mode)
- [Privacy and security](#privacy-and-security)
- [Logs](#logs)
- [Troubleshooting](#troubleshooting)
- [Reporting a bug](#reporting-a-bug)
- [Getting help](#getting-help)
- [License](#license)

## Features

Seven tabs, in the order they usually get used. Tab order is customizable in Preferences.

### Visualize

- **Map** — an interactive graph with separate Computers and Devices views, covering smart and static groups, policies, configuration profiles, Mac and mobile apps, App Catalog deployments, Restricted Software, Patch Titles, packages, scripts, printers, PreStage enrollments, Enrollment Customizations, extension attributes, Blueprints *(beta)*, webhooks, Jamf users and user groups, and your buildings, departments, and categories.
- **Journey** — the full path a device takes from enrollment (PreStage) through group membership to every policy, profile, app, package, and script it would receive — including cascades, where installed software triggers smart-group membership that delivers more software.
- **Compare** — opens in its own window for side-by-side comparison of two computers, mobile devices, policies, macOS or mobile configuration profiles, or Blueprints, including effective-scope evaluation ("applies / does not apply / unknown").
- **Inspector** — select any object to see its scope, its relationships in both directions, and a direct link to open it in Jamf Pro. Blueprints additionally report per-device deployment status: Jamf's own succeeded / failed / pending counts, each expandable to the named devices behind it.

![The Journey tab — the path a device takes from PreStage enrollment through group membership to everything it receives](docs/images/02-journey.png)

### Investigate

- **Search** — full-text search across configuration profile payload settings, so you can find which profile actually sets a given key. Requires a deep scan (see [Scans and snapshots](#scans-and-snapshots)) for complete results, since payload contents are only fetched then.
- **Summary** — import a Jamf Pro Summary file (text or JSON) to review licensing, database health, server configuration, and account hygiene.
- **Compliance** — Jamf Compliance Benchmark results pulled from the Jamf Platform API, with per-benchmark pass rates and drill-down to individual rules. Read-only.
- **Filters** — narrow the map by name, object type, site, staleness, scope status, managed state, group emptiness, policy enabled state, Blueprint deployment state, Lost Mode, device compliance, user assignment, Inventory Preload, and saved advanced searches. Benchmark-generated objects and legacy Jamf Remote policies can be hidden outright.
- **Flags and notes** — flag any object and attach a note as you work. A toolbar badge counts flagged items, a dedicated view lists them, and the flagged set can be included in an export.

### Catch what's easy to miss

ScopeMap marks objects on the canvas and in the Inspector when something looks wrong:

- **Policies that can never fire** — a login or startup trigger whose check-in configuration means the event never reaches the policy.
- **Scope you can't confirm** — objects whose targeting depends on limitations ScopeMap can't evaluate, marked rather than guessed at.
- **Unscoped objects** — enabled and deployable, but targeting nothing.
- **Complex groups** — smart groups carrying ten or more criteria, or five or more "Member of" criteria that point at other smart groups.
- **App license pressure** — VPP apps low on or out of available licenses.
- **FileVault** — devices whose escrowed recovery key is invalid or in an unknown state.
- **Profiles a PreStage won't keep** — a PreStage installs a configuration profile during enrollment regardless of that profile's own scope, but Jamf only *keeps* it on a device while the scope covers that device. If it doesn't, the profile silently falls off days later with nothing in Jamf Pro to say so. The Inspector names the computers affected so you can trace one in Journey, and states how many of the PreStage's enrolled computers the finding was measured against — devices too newly enrolled to have been evaluated by a smart group are deliberately excluded rather than counted as misses.
- **PreStages with packages but no distribution point** — the packages will silently not deploy.
- **Patch Titles** — configurations whose extension attributes haven't been accepted.
- **Inventory Preload** — devices whose building, department, or user assignment came from a preload record rather than from the device itself, which changes what their scope actually means.
- **Device compliance** — compliant / non-compliant dots, derived from the two smart groups you identify as your Device Compliance groups (see [Device Compliance](#device-compliance)).
- **Lost Mode** — mobile devices currently in Lost Mode.
- **Webhooks created by Jamf Routines** — marked so they aren't mistaken for hand-made ones. Jamf Routines creates a webhook in Jamf Pro for each event-driven routine, in the same list as your own, with nothing in Jamf Pro to distinguish them.

### Clean up

- **Cleanup** — surfaces orphaned packages and scripts, unscoped enabled policies, profiles, apps, App Catalog deployments and Restricted Software, empty static groups, disabled policies and disabled webhooks, unused buildings, departments, printers, and enrollment customizations, blueprints scoped to nothing, webhooks whose target smart group no longer exists, legacy Jamf Remote policies, unmanaged devices, and other environment cruft, with exportable findings. An optional, off-by-default **Destructive Mode** can delete surfaced objects directly (see [Deleting objects](#deleting-objects-destructive-mode)).

![The Cleanup tab — orphaned packages, unscoped policies, and empty static groups surfaced for review](docs/images/03-cleanup.png)

### Work with the results

- **Export** — CSV, JSON, Markdown, HTML, and PNG, depending on the report. Node inventories and relationship maps export as CSV/JSON/Markdown, canvas snapshots as PNG. Cleanup, Compare, Compliance, Journey, and Summary each have their own export sheet, with Compliance defaulting to a printable HTML report.
- **Activity log** — every API call, timing, and error in one window, with a toolbar badge when something fails during a scan.

### Scans and snapshots

A standard scan fetches the objects and relationships needed to draw the map. A **deep scan** goes further and pulls the detail records behind them — macOS and mobile configuration profile payload contents, script and package details, Mac app details, smart-group criteria, extension attribute definitions, Blueprint details, and each computer's installed-package receipts — which is what Search needs and what makes some risk warnings and Journey's cascade detection possible.

Every scan is cached per server, so reopening a server loads the last snapshot instantly, with no API traffic, and works offline. The snapshot records when its data was actually fetched, not when it was last written, so a stale cache says so.

### Device Compliance

Jamf exposes no API for which smart groups are configured under Settings → Global → Device Compliance, so ScopeMap asks you to identify them: an "applicable" group and a "compliant" group, per platform, per server. Everything derived from them — the node dots, the compliance filter — reflects *your* assertion about that configuration, not something Jamf reported. This is separate from the Compliance tab, which reads benchmark results from the Platform API.

## Installing

Download the latest notarized installer package from the [Releases](../../releases) page and run it. The app is signed and notarized by Apple, so it opens normally — no Gatekeeper warnings and no right-click workaround needed.

To build from source instead, clone the repository and open `ScopeMap.xcodeproj`. There is one scheme and one set of Debug/Release configurations.

A short guided tour runs the first time you launch the app, and Help → ScopeMap Help (⌘?) opens an in-app reference covering how scanning works, credentials, and privacy.

## Requirements

- **macOS 14.0 or later** (Apple silicon or Intel)
- **Jamf Pro** with API access (Classic + Pro API)
- **Jamf Platform API credentials** — only for Blueprints and the Compliance tab. Everything else works without them.
- **Xcode 26 or later** — only to build from source

## Connecting to a server

ScopeMap manages a list of saved Jamf Pro servers, so you can keep sandbox, staging, and production side by side and switch between them without re-entering anything. Each server keeps its own cached snapshot, its own Device Compliance group selections, and its own credentials in the Keychain.

Both auth modes are supported: a Jamf Pro user account (username/password) or an API client (client ID/secret).

## Jamf Pro API privileges

By default ScopeMap is a **read-only reporting tool** — it never creates, modifies, or deletes anything on your server. Every request it makes is a `GET`.

Privilege names differ depending on how you authenticate, so both lists are below. If a privilege is missing, ScopeMap doesn't fail — that object type is simply absent from the map — so you can trim either list for a narrower use case.

You won't have to guess which one is missing. After a scan, ScopeMap reports every object type it was refused: a banner above the map names the count, and its details sheet lists each object type, what the gap costs you on the map, and the exact privilege to grant, named the way *your* auth mode names it. The same verdict is written to the activity log. It's advisory only — a narrow role producing a narrow map is a supported way to use ScopeMap, and nothing is blocked.

### If you use an API client (client ID/secret)

Create a dedicated API role with these **Read** privileges:

<details>
<summary>39 read privileges</summary>

**Devices** — Read Computers · Read Mobile Devices

**Groups** — Read Smart Computer Groups · Read Static Computer Groups · Read Smart Mobile Device Groups · Read Static Mobile Device Groups · Read Smart User Groups · Read Static User Groups

**Deliverables** — Read Policies · Read macOS Configuration Profiles · Read iOS Configuration Profiles · Read Mac Applications · Read Mobile Device Applications · Read Restricted Software · Read Packages · Read Scripts · Read Printers

**Patch** — Read Patch Management Software Titles · Read Patch Policies

**Enrollment** — Read Computer PreStage Enrollments · Read Mobile Device PreStage Enrollments · Read Enrollment Customizations · Read Device Enrollment Program Instances

**Extension attributes** — Read Computer Extension Attributes · Read Mobile Device Extension Attributes

**Advanced searches** — Read Advanced Computer Searches · Read Advanced Mobile Device Searches

**Organization** — Read Buildings · Read Departments · Read Categories · Read Self Service · Read Sites · Read Distribution Points

**Accounts** — Read Accounts · Read Account Groups · Read User

**Settings** — Read Computer Check-In · Read Inventory Preload Records · Read Webhooks

</details>

Two of those are easy to miss. **Read Self Service** is required alongside Read Categories, because Jamf gates the categories endpoint on both — ScopeMap doesn't read Self Service itself. And the patch endpoints need **both** Read Patch Management Software Titles and Read Patch Policies; either alone fails.

App Catalog deployments are the one gap. The App Installers endpoint isn't covered by either of Jamf's published privilege tables and has no matching API role privilege, so if App Catalog deployments don't appear on your map, that endpoint is the thing to check.

Blueprints and Compliance are Jamf Platform features and use separate Platform scopes rather than Jamf Pro privileges: `blueprints read` and `compliance-benchmarks read`.

### If you use a Jamf Pro user account (username/password)

The **Auditor** role covers the object privileges. Confirm these four settings-level privileges on your server, since they sit outside what Auditor is built around: Read Computer Check-In, Read Inventory Preload Records, Read Device Enrollment Program Instances, and Read Sites.

Eight privileges are named differently on a user account than in an API role, which is worth knowing if you're translating between the two:

| Object | User account | API role |
| --- | --- | --- |
| Jamf Pro accounts and groups | Read - User accounts and groups | Read Accounts **and** Read Account Groups |
| Computer extension attributes | Read - Extension Attributes | Read Computer Extension Attributes |
| Mac apps | Read - Mac App Store Apps | Read Mac Applications |
| Mobile device apps | Read - Mobile Device Apps | Read Mobile Device Applications |
| Distribution points | Read - File Share Distribution Points | Read Distribution Points |
| Restricted software | Read - Restricted Software Records | Read Restricted Software |
| Jamf Pro users | Read - Users | Read User |
| Mobile device profiles | Read - Mobile Device Configuration Profiles | Read iOS Configuration Profiles |

### Privileges ScopeMap never needs

No **Create**, **Update**, or **Send** privileges, ever. ScopeMap issues no MDM commands, no pushes, and no log flushes.

No key-viewing privileges either. FileVault escrow status comes from the computer inventory record, so **View Disk Encryption Recovery Key** and **View Local Admin Password** are not required.

The one exception to read-only is the optional Destructive Mode described below. It is disabled by default and its delete controls stay hidden unless you turn it on. If you don't enable it, no delete privileges are needed and ScopeMap only ever issues read requests.

## What ScopeMap can and can't evaluate

Scope evaluation is the core of the app, so it's worth being precise about where it stops.

**Evaluated** — scopes built from All Computers/Devices, individual devices, smart and static groups, buildings, departments, and individual Jamf users, as both targets and exclusions. Jamf-user matching uses each device's assigned username and is therefore approximate when assignments are stale; the UI labels these matches accordingly.

**Detected but not evaluated** — limitations (LDAP users and groups, network segments, iBeacons) and Jamf user-group targets. These depend on network, directory, or sign-in state at check-in time, which no amount of inventory data can reconstruct. Objects using them are never reported as unscoped, and Compare reports them as "unknown" rather than guessing.

**Inferred, not confirmed** — cascades in Journey, where a package installed by one policy makes a device eligible for a smart group that delivers more software. ScopeMap can show that a cascade is possible from criteria and package names; it can't always confirm one occurred, and labels its confidence accordingly.

**Not reported by Jamf at all** — webhook delivery history. No Jamf API exposes whether a webhook has ever fired, is currently failing, or what it last returned, so ScopeMap can only report a webhook's configuration. "Disabled" and "target group no longer exists" are the two problems detectable without delivery data, and both are surfaced in Cleanup.

**Not yet covered as object types** — eBooks and Classes.

## Deleting objects (Destructive Mode)

Cleanup surfaces environment cruft — orphaned packages and scripts, empty static groups, unscoped or disabled objects. To act on those findings from within ScopeMap, you can enable **Destructive Mode**, which deletes surfaced objects through the Jamf Pro API.

Destructive Mode is built to be hard to trigger by accident:

- It is **off by default**. All delete controls are hidden and the app behaves as a pure read-only reporting tool until you turn it on.
- It lives in **Preferences → Advanced**, deliberately separated from everyday controls, and turning it on requires an explicit confirmation.
- Deleting is a two-step action: select items in Cleanup, then confirm in a dialog that lists exactly what will be removed.
- Every deletion is **permanent, cannot be undone, and is recorded in the activity log**.

Deletion is supported for scripts, packages, policies, macOS and mobile configuration profiles, restricted software, static computer and mobile groups, buildings, departments, printers, Mac and mobile apps, webhooks, enrollment customizations, and blueprints. **Computers and mobile devices are never deletable** — removing their records carries MDM consequences — and App Catalog deployments must be removed in Jamf Pro directly.

Blueprints are the one type deleted through the Jamf Platform API rather than the Jamf Pro API, so blueprint deletes require Platform API credentials to be configured for the server; without them the delete is blocked with a message in the activity log.

**Webhooks created by Jamf Routines are protected from deletion**, even with Destructive Mode on. Deleting one would silently break the routine that owns it, with nothing in Jamf Pro to explain why — so those rows are report-only and say so. Webhooks you created yourself delete normally.

Destructive Mode needs **Delete** privileges on the object types you intend to remove, in addition to the Read privileges above. The Auditor role includes none of them. If your account has only Read access, deletes fail with a permissions error and nothing is changed.

<details>
<summary>15 delete privileges</summary>

| Object | API role | User account |
| --- | --- | --- |
| Scripts | Delete Scripts | Delete - Scripts |
| Packages | Delete Packages | Delete - Packages |
| Policies | Delete Policies | Delete - Policies |
| macOS configuration profiles | Delete macOS Configuration Profiles | Delete - macOS Configuration Profiles |
| Mobile device configuration profiles | Delete iOS Configuration Profiles | Delete - Mobile Device Configuration Profiles |
| Restricted software | Delete Restricted Software | Delete - Restricted Software Records |
| Static computer groups | Delete Static Computer Groups | Delete - Static Computer Groups |
| Static mobile device groups | Delete Static Mobile Device Groups | Delete - Static Mobile Device Groups |
| Buildings | Delete Buildings | Delete - Buildings |
| Departments | Delete Departments | Delete - Departments |
| Printers | Delete Printers | Delete - Printers |
| Mac apps | Delete Mac Applications | Delete - Mac App Store Apps |
| Mobile device apps | Delete Mobile Device Applications | Delete - Mobile Device Apps |
| Webhooks | Delete Webhooks | Delete - Webhooks |
| Enrollment customizations | Delete Enrollment Customizations | Delete - Enrollment Customizations |

</details>

Grant only the object types you actually intend to clean up. Note that the **smart** group delete privileges are never needed — only empty static groups are deletable — and neither are Delete Computers, Delete Mobile Devices, or Delete User.

## Privacy and security

- Server credentials are stored only in the **local macOS Keychain**, never on disk in plain text.
- Cached server snapshots are stored locally in the app's sandboxed container, as **unencrypted JSON** — one file per server, holding the Jamf object data from that server's last scan. Treat them as you would any other local copy of your Jamf inventory.
- **Removing a saved server removes everything belonging to it**: its Keychain credentials, its cached snapshot, its imported Summary, its Device Compliance selections, and any flags and notes you saved for it. Nothing is removed from Jamf Pro itself.
- The app is sandboxed with outgoing-network and user-selected-file entitlements only.
- **Minimal, anonymous usage analytics.** ScopeMap sends a single anonymous "launched" signal to [TelemetryDeck](https://telemetrydeck.com) at startup, so the Concepts team can see adoption. No device, server, credentials, or Jamf object data is ever included. Opt out anytime in Preferences → General → Privacy (takes effect on next launch).
- **Version check.** At launch ScopeMap fetches a small JSON file from a public GitHub repository listing versions that shouldn't be run — how we stop a build with a serious defect from staying in circulation. The request is an anonymous `GET`; it sends no identifying information and no data about you, your Mac, or your server. If the request fails, the app starts normally.

Besides those two startup requests, ScopeMap talks only to the Jamf Pro server you point it at — and to us, when *you* file a bug report. Nothing else is transmitted automatically, and nothing leaves without you reading it first.

See [SECURITY.md](SECURITY.md) to report a vulnerability. Do not use a public issue for security reports.

For Jamf's corporate privacy practices, see the [Jamf Privacy Policy](https://www.jamf.com/trust-center/privacy/privacy-policy/).

## Logs

ScopeMap doesn't write a log file to disk on its own. Every API call, its timing, and any error is recorded in the in-app **Activity log** (toolbar icon, badged when something fails during a scan) for the current session — it isn't persisted between launches. There is no separate debug/verbose mode to enable; the Activity log already shows full request timing and error detail.

The log window's **Export** menu saves or copies the whole session log, which is more than the bug reporter sends — that one carries only the last 300 entries. Two variants: **Full**, which includes your Jamf Pro URL and account name and is meant for your own diagnosis, and **Redacted**, which applies the same substitutions a bug report does — server URL, hostname, server name and API account — for pasting into a ticket.

If ScopeMap quits unexpectedly, the next launch says so and offers to open a bug report pre-filled with the version, timing and last major operation of the session that died. Nothing is sent automatically — as with any bug report, you read it and press Send. If it was a genuine crash, macOS also wrote a symbolicated report to `~/Library/Logs/DiagnosticReports`; attaching that makes it far more diagnosable. A Force Quit or a Mac restart looks identical from inside the app, so the prompt is safe to ignore in those cases.

## Troubleshooting

- **"Certificate not trusted" or TLS errors connecting to a Jamf Pro server** — this usually means the server uses a self-signed or internally-issued certificate that isn't in your Mac's trust store. Add the certificate to your login or System keychain and mark it trusted, then reconnect.
- **An object type is missing from the map** — the account is almost certainly missing its Read privilege. Look for the banner above the map after a scan and open its details sheet, which names the object type and the privilege to grant. Note that a Jamf Pro user account is often refused with a **401** where an API role is refused with a **403** — both mean the same thing here.
- **Search finds nothing in profile payloads** — payload contents are only fetched during a deep scan. Run one, then search again.
- **The Compliance tab or Blueprints are empty** — both require Jamf Platform API credentials for the connected server. Everything else works without them.
- **Destructive Mode delete fails with a permissions error** — your account has Read access but not the Delete privilege for that object type. See [Deleting objects](#deleting-objects-destructive-mode).
- **A scan hangs or never completes** — open the Activity log (toolbar) to see which API call is in progress and whether it's retrying. Very large environments can take longer on the first scan; subsequent scans use the cached snapshot.
- **The data looks out of date** — check the snapshot timestamp. Cached snapshots are shown as-is until you rescan.
- **Still stuck** — use Help → Report a Bug so the report includes your app/macOS version, object counts, and the tail of the activity log. If the problem happened early in a long scan, export the full log from the log window's Export menu and attach that too — the bug report only carries the last 300 entries.

## Reporting a bug

Use **Help → Report a Bug** in the app, or the bug icon in the toolbar. It gathers the context that makes a report actionable — app and macOS version, object counts, active filters, and the tail of the activity log — and lets you attach screenshots.

Your **server URL, hostname, server name, and API account are redacted** from the report before it's sent. Jamf object names (policies, groups, packages) are kept, because they're usually what the bug is about. The full text is shown to you for review before anything is sent, and there's a Copy Report button if you'd rather send it yourself. Attached screenshots can't be redacted — check them for your server URL before adding one.

## Getting help

- **In the app** — Help → ScopeMap Help (⌘?) covers how scanning works, what credentials are needed, and what the app does and doesn't send anywhere.
- **Something wrong?** — [Report a bug](#reporting-a-bug).
- **A security issue?** — [SECURITY.md](SECURITY.md), not a public issue.
- **Want to contribute?** — [CONTRIBUTING.md](CONTRIBUTING.md).

## License

ScopeMap is provided under the [Jamf Concepts Use Agreement](LICENSE). This software is closed-source; no source code is published or distributed publicly.
