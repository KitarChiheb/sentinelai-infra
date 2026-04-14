# ADR-001: User and Group Architecture for SentinelAI Production Server

## Status
Accepted

## Date
2026-04-14

## Context
* Everything we do on this server — firewall rules, sudo policies, service isolation, file permissions,audit logging — depends on having a correct user and group model.
* If the identity layer is wrong,every security control built on top of it will cause problems in a production server.
* The previous state of the server has one human user: chihe, with a messy group membership inherited from a desktop Ubuntu install.
* The server was not a production identity model and does not fllow the priciples ofnleast privilege.

## Decision
* Upgrade the user account architecture and the group design to have a consistent production identity model by creating new groups and users
* sysadmin group (Members of this group are infrastructure engineers), they get full sudo access (configured in INFRA-22), any human who needs to administer this server must be in this group.
* appteam group (Members of this group are application operators), they can restart the SentinelAI application service and read application logs. They cannot touch system configuration.
* readonly group Members of this group can read logs and configuration files for monitoring and debugging purposes. They cannot change anything.
* chiheb — This is your correct production username. Not chihe — that was a typo at machine creation.This is the human sysadmin account. It belongs to the sysadmin group as its primary group.
* sentinelai-app — This is a service account. It is the user that the SentinelAI FastAPI application runs as.It belongs to the appteam group. It has exactly the permissions the application needs and nothing else. This is the principle of least privilege applied to service isolation.
* chihe account retained temporarily pending full transition to chiheb i 

## Consequences
* Solidify and upgrade the overall user and group architecture for the SentinelAI production server. 
* creating a sentinelai-app service account withcnologin means we can never interactively debug the application as that user — we will need to use sudo -u sentinelai-app for any direct testing. 

## Alternatives Considered
* leaving only one sudo user will increase simplicity but also will increase security risks 
* Attribute and manage the roles for groups and users will be done in INFRA22
