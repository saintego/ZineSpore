# ZineSpore: ideas and plan

Notes from the first brainstorm (November 2026). Nothing here is final.

## Vision

An underground zine for the gig scene, running on Nostr through the nostr-post library. It has two audiences:

- **Readers and bands:** a gig calendar, band and album wikis, reviews and reports, all in a zine look.
- **Venues:** a free calendar, poster creator and social layer they can embed on their own site.

It is also a real-world test of nostr-post's theming and plugin system.

## Name

- Working name: **ZineSpore** (earlier ideas: decentraZine, spreadZine, SporeZine).
- Avoid crypto or "decentralized" wording in the public brand. Much of the punk and metal audience distrusts it.
- A spore is a small piece that travels and grows somewhere new, which fits relay replication and venue embeds. Every post or event can be "a spore".
- To do: check domains, social handles and existing zines, bands or festivals with the name.

## Hosting

- Primary: an **nsite** (static site served over Nostr/Blossom).
- Mirror on a normal domain for discovery and for people without nsite gateways.

## Content

| Area | Notes |
|---|---|
| Gig calendar | NIP-52 events (31922/31923) and calendars (31924). Agenda and month views, RSVP, `.ics` import/export. Base: `examples/gigs` in nostr-post. |
| Event discussion | Comments under each event. |
| Concert reports | Reports linked to the event. |
| Reviews | Gig reviews (per band performance) and album reviews. |
| Band wiki | Wiki pages (kind 30818), including a logo field for posters. |
| Album wiki | Wiki pages linked to bands. |
| Media | `plugin-embed`: YouTube, Spotify, SoundCloud, Vimeo, Bandcamp, click to load. |
| Poster creator | New package. Pulls band logos from wiki pages, fills a template with event data, exports PNG/SVG. |
| Languages | Multilanguage support later. |

## Themes

All three run on the same theme tokens (fonts, palette, image filters) exposed as CSS variables.

1. **Zine (default):** typewriter text, black and white, grayscale images.
2. **Dark:** red on black, darkened images.
3. **Hippie:** full of color.

Venues get their own theme on the same token system.

## Venue embed

- nostr-post already uses web components, so the embed is a thin wrapper.
- One script tag plus a custom element, e.g. `<zs-venue-calendar npub="…" theme="…" view="month">`.
- Shadow DOM for isolation. Only theme tokens are exposed, as CSS variables.
- Parameters: `npub`/`naddr`, `view` (agenda, month, poster), `theme`, `lang`, `relays`, `readonly`.
- Alternative: venue settings stored as a NIP-78 manifest, so the embed only needs `<zs-venue npub="…">`.
- Decide early:
  - Default relays and venue overrides.
  - Writes (RSVP, comments) need a signer. Start read-only on venue sites and link out to ZineSpore for login.
  - Serve the script from a stable versioned URL.
- Submission flow: a band or promoter proposes a gig, the venue approves, and it appears in the venue calendar and the main feed.
- Venue profile: address, capacity, map. The nostr-post roadmap already plans moving venue code.

## Seeding with public data

Importing public gig listings makes the site look full from day one.

- Source: Gancio instances (e.g. do.basspistol.org) via their RSS/`.ics` feeds or API.
- Rules:
  - Always link back to the source.
  - Easy opt-out for promoters and venues.
  - Mark imported events clearly so real organizers can claim them.
  - Consider asking instance admins first.

### Existing bridge found

Someone already bridges Basspistol Do to Nostr (pubkey `b47110e769732df94f781a2d9de5f3c20e27ae3f5b51ac7d8d712c837c2e3278`). Example event, kind 31923:

- `d` = Gancio event ID (`18409`), so re-imports overwrite instead of duplicating.
- `a` = `31924:<same pubkey>:basspistol-do`, its own calendar. Trusting that calendar covers the whole feed.
- `D` day tag present.
- Content is only title, address and the Gancio link. Genres arrive as `t` tags.

Problems seen in that event, to avoid in our importer:

- No `start_tzid`, so the app showed Bangkok time for a Berlin gig.
- No machine-readable source tag; the link is only inside the text.
- Duplicate `t` tag (`post-metal` twice).
- Image hotlinked from the Gancio server.
- Only found on one relay (`wss://nos.lol`).
- No bands, bios or ticket link (the feed has less than the event page).

Our importer should: set `start_tzid`, add a source tag, de-duplicate tags, build a fuller description, mirror images, and publish to several relays.

## Filtering scene content from the rest of Nostr

Start from trusted sources and show only those. From strongest to weakest:

1. **Curated calendars:** show events listed in venue or curator calendars (kind 31924).
2. **Publisher allowlist:** npubs of venues, promoters and importer bots, maintained by the scene.
3. **Web of trust:** events from people followed by trusted accounts, a few hops out.
4. **Kind and tag limits:** calendar events and wiki pages with music tags. Cheap but easy to spoof.
5. **Dedicated relay:** only accepts scene content. Fast, but limits reach.
6. **Reports:** respect NIP-56 reports and mute lists.

Start with 1 and 2. Add web of trust once the scene is big enough.

## Fixes for nostr-post `examples/gigs`

- **Timezone:** show times in the venue's timezone from `start_tzid`, with an optional "your time" label. Fall back to the city in `location` when the tag is missing.
- **Raw-data panel:** author npub, kind, `d` tag, `naddr` (copyable), relays found on, all tags, raw JSON, signature. (Prototype already exists.)
- **Source attribution:** show it when a source tag is present.

## Comparison: Gancio (e.g. Basspistol Do)

| | Gancio | ZineSpore |
|---|---|---|
| Network | ActivityPub, one server per instance | Nostr, no server owns the data |
| Identity | Instance accounts | Own keys, usable across apps |
| Bands and albums | Tags and links | Wiki pages with logos |
| Posters | Upload/download flyer | Generated from logos and event data |
| Reviews and reports | None | Gig reviews, album reviews, reports |
| Discussion | None visible | Comments and RSVP |
| Embed | Yes | Web component with venue theme |
| Look | One shared look | Zine themes and per-venue themes |

Gig listing alone won't beat Gancio. The wiki, reviews, posters and the zine look are what make the difference.

## Before building on nostr-post PR #15

- The PR is stacked on #14, not main. Merge the stack first.
- `askClaude` is untested against the real API.
- `build:pages` is not verified end to end.
- No reviews yet.

## Suggested order

1. Merge the nostr-post stack.
2. Fix timezone display and finish the raw-data panel in `examples/gigs`.
3. Build the three themes on the gigs example.
4. Expose it as the venue-calendar web component.
5. Importer for Gancio feeds.
6. Poster package.
7. Reviews, reports, wiki views.
8. Multilanguage.
