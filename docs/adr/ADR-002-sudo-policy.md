# ADR-002: Sudo Policy for SentinelAI Production Server

## Status
Accepted

## Date
2026-04-14

## Context
* sudo is a fine-grained policy engine, you can grant a user the ability to run exactly one command as root, or you can grant full unrestricted access. 
* A poorly designed sudo policy that does not follow the principle of least privilege can enable an attacker to compromises an account and gains the ability to escalate privilege. 

## Decision
* Attribute a full admin access, password required, on all hosts, this is the standard full-admin grant, simple, clear, auditable.
* Attribute a command-scoped for appteam, to be able to manage the SentinelAI service — start, stop, restart and read application logs, we grant exactly what they need.
* Attribute nothing for readonly, the readonly group has read access to files via group permissions
* Putting five commands on one line works syntactically but is hard to read and hard to maintain, the professional approach uses a Cmnd_Alias — a named list of commands defined once and reference by name.

## Consequences
* The appteam are able to manage the SentinelAI service, they can not install packages, edit system configuration, create users, or touch the firewall, they can escalate by raising a jira ticket to the system admin team.

## Alternatives Considered
* Full sudo privilege to appteam: appteam group gets full-admin grant. Rejected as out of scope privilege to appteam introduces risk of a compromise for appteam account and privilege escalation

