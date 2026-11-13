# Agent Instructions

Guidance for AI agents and humans working in this repo. Claude Code reads it through CLAUDE.md.

## Where things go

- **Issues are the plan.** Every feature, fix, open decision and piece of work is a GitHub issue. A new idea becomes an issue first, not a new section in IDEAS.md.
- **IDEAS.md is the concept.** It holds the vision, principles, name and research notes (what we learned about Gancio, Spotify, Last.fm and so on). It is not a task list or a build order. Every section links to its issues.
- **Issue #37 is the build order.** Add new issues to its checklist; don't keep an order anywhere else.
- Add to IDEAS.md only when there is new research or a new principle, and link the issues that come from it.
- When an issue is added, closed or renamed, update the links in IDEAS.md and the checklist in #37.

## Issue conventions

- One issue per deliverable. Short title. Body: what and why, then these lines where they apply:
  - `Depends on nostr-post: saintego/nostr-post#N`
  - `Related: #N`
  - `See IDEAS.md "<section>"`
- Labels: `enhancement`, `documentation`, and the area (`theme`, `venues`, `content`, `trust`, `import`, `integrations`, `i18n`, `deploy`, `lists`, `chat`, `ops`, `community`, `brand`).

## nostr-post and ZineSpore

- **nostr-post** gets what is generic and reusable: list features, mute handling, article manifests, time zone handling, translation data model.
- **ZineSpore** gets what is scene-specific: gig, album and review manifests, posters, style taxonomy, music service integrations, themes, and configuration.
- **Importer / cron** is a separate repo (nostr-post #122), not part of the app.
- Before opening a nostr-post issue, search its open and closed issues for a duplicate, and show the draft to the user for approval first.
- Link both ways: the ZineSpore issue names `Depends on nostr-post: …`, and the nostr-post issue ends with `Needed by saintego/zinespore#N`.

## Principles

- Local scenes, global information: a spore decides what it shows, not what exists.
- No crypto or "decentralized" wording in public text; explain in zine language.
- Users' API keys and listening history stay on their device (synced through evolustr), never published to Nostr.
- Third-party data is cached locally; only our own content and IDs or links go to Nostr.
- Imported content always links to its source and can be opted out of and claimed.
- Private and opt-in by default: lists and chat.

## Working in the repo

- Check facts before writing them down (NIPs, API terms, browser support). Mark anything unchecked as such.
- Keep commit messages short and in the imperative.
