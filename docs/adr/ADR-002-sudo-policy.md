# ADR-002: Sudo Policy for SentinelAI Production Server

## Status
Accepted

## Date
2026-04-14

## Context
* sudo is a fine-grained policy engine, you can grant a user the ability to run exactly one command as root, or you can grant full unrestricted access. 
* A poorly designed sudo policy that does not follow the principle of least privilege can enable an attacker to compromise an account and gain the ability to escalate privilege. 
* Prior to this decision, the server relied on the default Ubuntu sudo group, which granted binary "all-or-nothing" access.The chiheb account had no privileges, and there was no mechanism to allow application operators to manage services without granting them full root access. This lack of granularity created a high risk where a compromised application account could lead to a full system takeover.

## Decision
* The sysadmin group is granted (ALL:ALL) ALL privileges, requiring password authentication for all escalations.
* The appteam group is granted access to exactly five commands (start, stop, restart, status of the application, and journalctl for logs) via a Cmnd_Alias. 
* Inlining five commands works syntactically but it is hard to read and hard to maintain, the professional approach uses a Cmnd_Alias — a named list of commands defined once and reference by name.
* The readonly group is intentionally excluded from the sudoers policy to enforce read-only status.
* All rules are stored in /etc/sudoers.d/ using a numeric prefix convention (10-, 20-) to keep the system /etc/sudoers file clean and update-safe.

## Consequences
* The appteam are able to manage the SentinelAI service, they can not install packages, edit system configuration, create users, or touch the firewall, they can escalate by raising a jira ticket to the system admin team.
* All rules require password authentication. This means automated processes cannot use sudo without a password — any future automation that needs elevated privileges must be designed around this constraint or explicitly request a NOPASSWD exception through a new ADR.
* Adding a new group's sudo policy in the future requires a new numbered file in /etc/sudoers.d/. The naming convention (10-, 20-) means future files should follow the same pattern (30-, 40-).

## Alternatives Considered
* Full sudo privilege to appteam: appteam group gets full-admin grant. Rejected as out of scope privilege to appteam introduces risk of a compromise for appteam account and privilege escalation
* Grant sudo to individual users rather than groups: each sysadmin user gets their own sudo rule. Rejected because it does not scale — adding or removing an engineer requires editing sudoers directly rather than managing group membership. Group-based policy is maintainable; per-user policy creates drift over time.
