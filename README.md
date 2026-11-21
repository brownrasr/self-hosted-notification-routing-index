# Self-Hosted Notification Routing Index

> **Decision:** Choose which layer owns message intake, provider fan-out, mobile delivery, user preferences, deduplication, and escalation instead of stacking overlapping routers.

**For:** Operators and developers routing alerts from scripts and services to phones, browsers, chat, email, and webhooks.  

This is a non-commercial, source-traceable decision guide. It normalizes upstream declarations; it does not claim that repository reachability proves functionality, security, maintenance, or fitness.

**Start here:** [Download the delivery ownership ledger](assets/notification-delivery-map.csv) — twelve projects, located upstream evidence, and explicit unknowns.

## Choose by workload


- Simple HTTP topics to phones and browsers: ntfy; application-token inbox with Android: Gotify.
- Fan out to many third-party services: Apprise API or Shoutrrr.
- Group and deduplicate operational alerts: Alertmanager or Alerta.
- Application notification workflows and user preferences: Novu or Notifo.
- Bridge external events into Matrix: Matrix Hookshot; bridge chat networks: Matterbridge, after a maintenance review.

## Trace one event to its recipient

A router cannot guarantee an external provider. Assign retry, grouping and idempotency to one layer, and account for phone-platform delivery dependencies.

## Event-to-recipient map


- **Direct push servers** — ntfy and Gotify host a receiving inbox and clients; Bark Server focuses an Apple-oriented path.
- **Provider routers** — Apprise API and Shoutrrr translate one call/configuration into third-party destinations; they do not own those providers.
- **Alert lifecycle** — Alertmanager groups, inhibits, and routes Prometheus-style alerts; Alerta aggregates and deduplicates events into an operator console.
- **Product notification platforms** — Novu and Notifo add users, templates, channels, and preferences—more scope than simple infrastructure alerts.
- **Bridges** — Matterbridge relays chat networks; Matrix Hookshot maps external services/webhooks into Matrix rooms.


## Delivery ownership ledger

| Project | Role | Intake and client ownership | Routing and delivery state | Persistence and provider dependency |
|---|---|---|---|---|
| ntfy | direct push server | ntfy (pronounced "notify") is a simple HTTP-based pub-sub | Unknown in the frozen upstream README | Dexie.js (Apache 2.0) is used for web app persistence in IndexedDB |
| Gotify | direct push inbox | send messages via REST-API receive messages via WebSocket manage users, clients and applications Plugins | Unknown in the frozen upstream README | Unknown in the frozen upstream README |
| Apprise API | provider router | Take advantage of Apprise through your network with a user-friendly API. | - Group your services with tags, then notify only the ones that need to know. | - There are no additional dependencies (such as database requirements, etc) should you choose to use the optional persistent store (mounted as /config). |
| Novu | product notification platform | Create a free account and follow the instructions on the dashboard. | Novu provides a unified API to send notifications through multiple channels — Inbox/In-App, Push, Email, SMS, and Chat. Create custom workflows, define per-chan… | Every product and every agent eventually needs to talk to people, across the channels those people already use. Novu is the open-source layer that handles that … |
| Shoutrrr | provider library and CLI | For additional details, visit the full documentation. | You can also use Shoutrrr from a GitHub Actions workflow. | Unknown in the frozen upstream README |
| Bark Server | mobile push server | Bark is an iOS App which allows you to push customed notifications to your iPhone. | Unknown in the frozen upstream README | Just run the server with -dsn=user:pass@tcp(mysqlhost)/bark, it will use MySQL instead of file database Bbolt |
| Notifo | notification platform | Have a look at the presentation for a visual explanation, or read docs/architecture.md for the technical one. | Abstraction over multiple channels and providers. Reliable through retry mechanisms and message queues for all notifications and channels. | a Docker image of Notifo, with one compose file per supported database. See CLAUDE.md for the conventions this repository follows. |
| Matterbridge | chat bridge | - MatterLink (Matterbridge link for Minecraft Forge server chat, archived) - MatterCraft (Matterbridge link for Minecraft Forge server chat) | - MatterLink (Matterbridge link for Minecraft Forge server chat, archived) - MatterCraft (Matterbridge link for Minecraft Forge server chat) | - Configuration - Basic configuration |
| Alertmanager | alert lifecycle router | The Alertmanager handles alerts sent by client applications such as the Prometheus server. It takes care of deduplicating, grouping, and routing them to the cor… | The Alertmanager handles alerts sent by client applications such as the Prometheus server. It takes care of deduplicating, grouping, and routing them to the cor… | Unknown in the frozen upstream README |
| Alerta | alert aggregation console | Unknown in the frozen upstream README | Unknown in the frozen upstream README | The only mandatory dependency is MongoDB or PostgreSQL. Everything else is optional. |
| Matrix Hookshot | Matrix integration bridge | - Webhooks let you connect all kinds of services, with the ability to write rich templates using JavaScript. - No external database is required, instead using M… | Unknown in the frozen upstream README | - No external database is required, instead using Matrix state as a persistent store. |
| Gotify CLI ecosystem | publisher client | watch and push script result changes (gotify watch "curl http://example.com/api / jq '.data'") | Gotify-CLI is a command line client for pushing messages to [gotify/server][gotify/server]. It is not required to push messages. See alternatives. | stores token/url in a config file |

Values are short excerpts from the frozen upstream README, not feature-test scores. The CSV retains the located evidence for each cell. “Unknown” means the retrieved document did not establish that axis; it is not a claim that the feature is absent.

## Assemble the minimal stack


Write one sample event and trace its identifier through retries, grouping, rate limits, destination failure, and replay. Put secrets in the narrowest layer. Avoid chaining two routers that both retry or deduplicate without stable idempotency keys; a temporary provider outage must not become a message storm.


## Adoption notes by project

### ntfy

- **Choose it for:** Simple self-hosted push topics.
- **License at the frozen source:** [Apache-2.0 or GPL-2.0 depending on component; verify repository files](https://raw.githubusercontent.com/binwiederhier/ntfy/18c15d706595587c45bd1e773e3f828f2ff6cf8d/LICENSE).
- **Adoption test:** Test authentication, topic guessing, attachment retention, upstream mobile delivery, rate limits, and database backup.
- **Source:** [binwiederhier/ntfy](https://github.com/binwiederhier/ntfy).

### Gotify

- **Choose it for:** Personal or team notification inbox.
- **License at the frozen source:** [MIT](https://raw.githubusercontent.com/gotify/server/912af9f37414ab08fae0cfcc3941c4fd95c183aa/LICENSE).
- **Adoption test:** Official client coverage is Android-focused; test browser/background delivery, retention, auth, and restore.
- **Source:** [gotify/server](https://github.com/gotify/server).

### Apprise API

- **Choose it for:** Centralizing many provider adapters behind one API.
- **License at the frozen source:** [MIT](https://raw.githubusercontent.com/caronc/apprise-api/c83b54bf39171cfb0c4ee3339cfa6b34b7b3e82d/LICENSE).
- **Adoption test:** It relies on external destinations; secure stored URLs, choose persistence mode, disable unused providers, and test retries.
- **Source:** [caronc/apprise-api](https://github.com/caronc/apprise-api).

### Novu

- **Choose it for:** Applications that need user-facing notification orchestration.
- **License at the frozen source:** [MIT core; /enterprise and subdirectories use a commercial license](https://raw.githubusercontent.com/novuhq/novu/ced3a4569dde75c77d238f0b736ecdb83156cbef/README.md).
- **Adoption test:** The /enterprise tree and its children use commercial terms, not MIT. Check self-hosted edition availability separately from managed-service features.
- **Source:** [novuhq/novu](https://github.com/novuhq/novu).

### Shoutrrr

- **Choose it for:** Adding provider fan-out without a separate platform.
- **License at the frozen source:** [MIT](https://raw.githubusercontent.com/containrrr/shoutrrr/ccf81390b700948f85ce1c79f6de9ee75c68d9e5/LICENSE).
- **Adoption test:** It does not provide durable queueing or a user inbox by itself; encode secrets safely and test provider-specific limits.
- **Source:** [containrrr/shoutrrr](https://github.com/containrrr/shoutrrr).

### Bark Server

- **Choose it for:** Small Apple-centric personal notifications.
- **License at the frozen source:** [MIT](https://raw.githubusercontent.com/Finb/bark-server/a365aa9ddc9362495efffb3db4358b7c12beb963/LICENSE).
- **Adoption test:** APNs credentials, client ownership, privacy, rate behavior, and non-iOS requirements need separate planning.
- **Source:** [Finb/bark-server](https://github.com/Finb/bark-server).

### Notifo

- **Choose it for:** Products needing a self-hosted notification service.
- **License at the frozen source:** [MIT](https://raw.githubusercontent.com/notifo-io/notifo/54da324ff6f4c51ca959751bdc6572e20c10a6e4/LICENSE).
- **Adoption test:** Confirm channel providers, retry semantics, user preference/export, tenant boundary, and current release support.
- **Source:** [notifo-io/notifo](https://github.com/notifo-io/notifo).

### Matterbridge

- **Choose it for:** Cross-chat routing rather than mobile push.
- **License at the frozen source:** [Apache-2.0](https://raw.githubusercontent.com/42wim/matterbridge/c4157a4d5b49fce79c80a30730dc7c404bacd663/LICENSE).
- **Adoption test:** Re-check maintained releases and adapter behavior before deployment; no dated inactivity conclusion is certified here.
- **Source:** [42wim/matterbridge](https://github.com/42wim/matterbridge).

### Alertmanager

- **Choose it for:** Prometheus-style operational alert routing.
- **License at the frozen source:** [Apache-2.0](https://raw.githubusercontent.com/prometheus/alertmanager/5dea4dedd70f7e770c34d33b96449ee27e5c1aae/LICENSE).
- **Adoption test:** It expects upstream alerts and external receivers; test HA convergence, grouping keys, silences, inhibition, and receiver failures.
- **Source:** [prometheus/alertmanager](https://github.com/prometheus/alertmanager).

### Alerta

- **Choose it for:** Operations teams needing a cross-source alert console.
- **License at the frozen source:** [Apache-2.0](https://raw.githubusercontent.com/alerta/alerta/4ac0106300cd428b26ad4b373cf4a78f55f6b542/LICENSE).
- **Adoption test:** Validate source adapters, duplicate keys, retention, auth, plugins, and maintenance cadence; it is not a mobile provider by itself.
- **Source:** [alerta/alerta](https://github.com/alerta/alerta).

### Matrix Hookshot

- **Choose it for:** Teams standardizing external events in Matrix.
- **License at the frozen source:** [Apache-2.0](https://raw.githubusercontent.com/matrix-org/matrix-hookshot/18312dc6dda798ac158832e5d6379707364ea437/LICENSE).
- **Adoption test:** It is Matrix-specific and not a general phone push server; test bot permissions, encryption, webhook code, and room-state recovery.
- **Source:** [matrix-org/matrix-hookshot](https://github.com/matrix-org/matrix-hookshot).

### Gotify CLI ecosystem

- **Choose it for:** Operators integrating shell jobs with an existing Gotify deployment.
- **License at the frozen source:** [MIT](https://raw.githubusercontent.com/gotify/cli/1a3712544653db1ca4d12bb52779be80ea152c79/LICENSE).
- **Adoption test:** It is a client, not another server; pin it to the server API and secure application tokens.
- **Source:** [gotify/cli](https://github.com/gotify/cli).

## Replay a destination outage

1. Publish one stable event identifier and trace its recipient.
2. Fail a destination, then restore it and inspect retries and duplicates.
3. Separate queue state, user preferences, provider credentials and silence state.
4. Rebuild the chosen layer and replay without a message storm.

## What this guide covers

Event delivery, operator alerts and application notifications, not telemetry collection or campaign marketing.

## Located sources and update policy

[`sources.json`](sources.json) binds each cited declaration to a commit, document, exact line range and local snapshot hash. [`assets/projects.json`](assets/projects.json) holds the normalized roles and decision axes. Current HEAD reachability is recorded separately; it is not a maintenance or security assessment. Licensing is version- and directory-specific. Re-read the selected release before adoption.

Run the local evidence and package checks, then opt into public-source reachability:

```bash
python3 scripts/verify_sources.py
python3 scripts/verify_sources.py --online
```

These checks validate source binding, excerpts, hashes, language, the decision asset and nonempty public content. They do not certify feature behavior, migration success or restore success. No popularity score, paid placement or affiliate ranking is used. Restricted-license and historical examples are explicit contrasts, not silently recommended open-source defaults.
