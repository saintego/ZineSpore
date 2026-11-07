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
| Reviews | Gig reviews (per band performance) and album reviews. Can also be external review cards (see "Data intake"). |
| Personal lists | Watched bands, liked bands, songs and styles, with suggestions built from them. See "Personal lists and suggestions". |
| Music services | Optional links to MusicBrainz, Bandcamp, Spotify, Last.fm and similar. Fills the site while Nostr content is still thin. See "Music service integrations". |
| Long articles | Long-form posts (NIP-23, kind 30023) through nostr-post: interviews, scene reports, essays, zine issues. |
| Band wiki | Wiki pages (kind 30818), including a logo field for posters. |
| Album wiki | Wiki pages linked to bands. |
| Media | `plugin-embed`: YouTube, Spotify, SoundCloud, Vimeo, Bandcamp, click to load. |
| Poster creator | New package. Pulls band logos from wiki pages, fills a template with event data, exports PNG/SVG. |
| Languages | Translations of UI, articles, reviews and venue texts. See "Multilanguage". |

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

## Data intake

Several ways to feed content in, side by side:

- **`.ics` import:** existing in nostr-post, including Facebook event `.ics` exports.
- **RSS/Atom feeds:** subscribe to feeds from venues, Gancio instances, labels, blogs and review sites.
- **AI sniffing:** paste a URL (or point at a page) and let AI extract structured data: event, band, album or review. The user checks the result before publishing. nostr-post's gigs example already has AI-assisted adding to build on.

### External review cards

For reviews published elsewhere, don't copy the text. Publish a card with:

- Short summary (written fresh, not copied).
- Score, normalized to one scale, with the original scale kept (e.g. 8/10, 4/5).
- Reviewer and publication name.
- Link to the full external review.
- Links to the album or band wiki page.

This fills album pages with reviews while respecting copyright and sending readers to the original.

## Personal lists and suggestions

Each reader keeps their own lists, and the site suggests gigs and music from them.

### Lists

- **Watched bands:** bands you want gig alerts for. Stored as a list of references to band wiki pages (or band npubs).
- **Liked bands:** a separate, lighter list. Can also come from a "like" on a band page.
- **Songs:** saved tracks, referenced from the album wiki page plus a title, or from an external link (Bandcamp, YouTube, etc.).
- **Styles:** genres you follow, such as post-metal or shoegaze. Stored as hashtags.

nostr-post already supports lists (kinds 10003 and 30003). Nostr's list standard (NIP-51) also has an interests list for hashtags, which I believe fits styles, but check the current NIP before relying on it.

### Privacy

- Taste is personal. Lists are **private by default**, encrypted inside the list event (NIP-51 allows private items), with an option to publish a list.
- Public lists make nice profile pages ("my 10 favorite albums") and are what other readers can discover.

### Suggestions

- Computed **on the reader's device** from their lists and public Nostr data. No server, no tracking profile.
- Every suggestion says why: "because you watch X" or "plays with a band you like".
- Output:
  - Upcoming gigs of watched bands, in the reader's city and within a chosen distance.
  - Gigs in your city matching followed styles.
  - Bands related to liked ones.
  - New album pages, reviews and articles about bands on your lists.
- Signals, from simple to harder:
  1. Shared style tags.
  2. Bands that shared a bill (found from calendar events).
  3. Same label, member overlap, or "similar to" links on wiki pages.
  4. Readers with overlapping public lists (opt-in only).
- Alerts without an account: the watched-bands gig list as an `.ics` or RSS feed the reader subscribes to.
- Cold start: pick a few styles and bands when joining, so the first screen isn't empty.

### Data issue to solve first

Genre tags are messy. Imported events carry free-form tags, e.g. `post-metal`, `atmospheric-black-metal`, `sludge`, sometimes duplicated. Matching and suggestions need a **style taxonomy**: canonical style wiki pages with aliases and parent genres (sludge is under doom, for example). Imported tags map to them; unknown tags stay as plain tags.

## Music service integrations

Goal: the site is useful even while Nostr content is empty, and readers can bring their listening taste in from services they already use. Everything is optional. Nothing needs an account.

### Principles

- **No keys of ours.** Anything that needs an API key uses the **reader's own key**. The key is stored locally and synced between that reader's devices through **evolustr** (local-first, encrypted sync). It is never published to Nostr and never sent to any server of ours.
- **Order of preference:** sources with no key first, then user-key sources, then file imports.
- **Only IDs and links go to Nostr,** not copied third-party data. Provider terms limit storing and redistributing their data, so cache it locally and publish only our own content (wiki text, reviews, links).
- **Listening history stays on the device.** It feeds the local suggestions (see "Personal lists and suggestions") and nothing else.

### Per service (checked November 2026, re-verify before building)

| Service | What it gives | Access | Notes |
|---|---|---|---|
| **MusicBrainz** | Canonical band, album and track IDs (MBIDs), relations, and places/events | No key (I believe; needs a proper User-Agent and low request rate) | Use as the ID backbone for band and album wiki pages, and to seed stub pages. Check its data licence before bulk use. |
| **ListenBrainz** | Open listening history and recommendations | Reader's token | Open alternative to Last.fm. I believe it also has a Last.fm-compatible API, so one code path may cover both. Verify. |
| **Last.fm** | Similar artists, tags, a reader's top artists and recent plays | Free key for non-commercial use | Terms: credit and link back, cap on stored data (100 MB), commercial use needs an agreement. Reader's own key keeps us out of shared-key limits. Accepts MBIDs. |
| **Spotify** | Reader's followed artists and saved tracks, embeds | Reader's own developer app | Heavily restricted for new apps: development mode allows few users, Premium required, search results capped, and related artists/recommendations/audio features closed to new apps. Extended access needs a large business. So only for power users. Embeds need no API. |
| **Spotify data export** | Full streaming history | Reader requests their data export and loads the file locally | No API at all. Likely the best Spotify route for normal readers. |
| **Bandcamp** | Listen and buy | Embed players and links only | No public fan API. Developer APIs are for labels and merch partners. The undocumented fan endpoints are fragile, so don't build on them. Existing `plugin-embed` covers players. |
| **Discogs** (optional) | Releases, labels, credits | Reader's token | Useful for label and member relations. |

### What the integrations feed

- **Stub pages:** a band or album with no Nostr wiki page yet shows a read-only page built from MusicBrainz (name, albums, links). The first person to edit it publishes a real wiki page, which then takes over.
- **External IDs on wiki pages:** MBID, Bandcamp URL, Spotify artist, Last.fm URL. These also let imported gigs match to bands.
- **List import:** Last.fm or ListenBrainz top artists, a Spotify export or followed artists become watched/liked bands, matched through MBIDs. Reader reviews and approves before saving.
- **Suggestions:** Last.fm "similar artists" and tag data as another signal next to shared styles and shared bills.
- **Style taxonomy:** MusicBrainz and Last.fm tags help map messy genre tags to canonical styles.

### Open questions

- **Browser access (CORS):** some of these APIs may not allow calls from a web page. Bandcamp's doesn't, and I haven't checked the others. If a service blocks it, options are a small proxy the reader runs, or leaving that service out. That would conflict with "no servers", so test it first.
- Whether to support Spotify's login flow at all, given how limited new apps are.
- Matching quality: band names are ambiguous, so always match by MBID where possible and let the reader fix wrong matches.
- Check each service's current terms before shipping, since Spotify changed its rules twice in two years.

## Multilanguage

Goal: translations for articles, reviews and venue descriptions, without paying for a translation API. Should live in nostr-post so every app on it gets it.

### Principles

- The **original** is the source of truth and keeps its language tag.
- A **translation is its own event**, linked to the original. It can come from the author, a community translator, or a person who corrected a machine draft.
- Reader sees their language if a translation exists, otherwise the original. Always show a label (original, by author, by translator, machine) and a "show original" toggle.
- Not translated: band names, album and song titles, venue names, genre tags.

### Data model (proposal)

I don't know of a standard Nostr convention for translations, so this is a proposal to check against current NIPs before building:

- Long article translation: a separate kind 30023 event, with `["l", "<lang>", "ISO-639-1"]` for its language and a reference tag pointing to the original, e.g. `["a", "30023:<pubkey>:<d>", "", "translation-of"]`.
- Same pattern for reviews and wiki pages (kind 30818).
- Wiki pages: one page per language, linked together.
- **Venue descriptions** and other manifest texts (NIP-78): localized fields in the manifest, e.g. `description: { en: "…", cs: "…" }`, with a fallback order. This fits nostr-post's field-level manifest design.

### Picking which translation to show

1. Author's own translation.
2. Translator the author or venue has endorsed (allowlist, same idea as the publisher allowlist).
3. Other community translations.
4. On-device machine translation, not published.

### Free machine translation

Run it in the reader's browser, so there is no server and no per-request cost.

- **Chrome/Edge Translator API:** on-device, free, works offline once the language pack is downloaded. Chrome 138+ and Edge 148+, desktop only. Not on mobile, Firefox or Safari. Supports Czech, German, English, Spanish, French and many more. The first use downloads a model and needs a user click.
- **Bundled fallback:** an opt-in in-browser model (e.g. Bergamot/Firefox-style or Opus-MT through Transformers.js). Works on more browsers, but the user downloads tens of MB per language pair and phones will be slow.
- **Last resort:** a link to the browser's own page translation.
- **Self-hosted LibreTranslate** is free software but needs a server, so only if we run one later.
- Machine translations are cached locally and **not published** by default, to avoid flooding relays with low-quality copies.

Limits: slang, scene jargon and band names will translate badly. Treat machine output as a draft.

### Correction flow

1. Reader clicks "translate", gets a machine draft on their device.
2. "Improve and publish" opens it in an editor.
3. They sign and publish it as a translation event, credited to them.

This turns machine drafts into good manual translations over time, for free.

### UI strings

Separate from content. Key/value JSON catalogs in the repo, one file per language, contributed by pull request. Language picked from browser settings, with a manual switch.

### Open questions

- How to detect the original language when the author didn't tag it (the Language Detector API is built into Chrome, otherwise a small bundled detector).
- Which languages first. Suggest English plus the scene's main local ones (Czech and German as a start).
- Whether the importer should translate imported events. Probably not: keep the original text and link to the source.

## Deploy your own (nsite)

Anyone can run their own ZineSpore: a venue, a collective, a city scene. Each one is a separate nsite on the same data.

- **Template repo / fork:** one config file holds the instance settings: name, theme, default language, relays, calendar and publisher allowlist, which venues to feature.
- **Publish in one step:** a build script, plus a GitHub Action that publishes to nsite on push. nostr-post already has an nsite preview build in CI on PR #15, so this can build on it. I haven't checked the current nsite tooling, so confirm that first.
- **Instances share data, not servers:** events, wikis and reviews live on Nostr, so every instance shows the same underlying content through its own filter and look. A new scene starts with content, not an empty site.
- **Own domain optional:** an nsite can also be mirrored to a normal domain.
- **Doubles as the venue story:** a venue can choose the embed (see Venue embed) or a full site of its own.

## About / manifesto page

A page that explains why it is built this way, in zine language and not tech language. It also helps sell it to bands and venues who don't care about protocols.

Ideas for the content:

- **The scene's memory shouldn't belong to one company.** Gig pages, flyers, reviews and band histories disappear when a platform changes its rules, buys a competitor, or deletes an account.
- **Flyers on walls, not feeds.** No algorithm decides which gig you see. No ads.
- **Photocopy logic.** Zines spread because anyone can copy and pass them on. Here, every copy of a post sits on someone else's relay and can't be pulled back by one switch.
- **Your key is your name.** Bands and venues own their identity and can take it anywhere.
- **Censorship.** One company or government can't remove the page for everyone. Be honest about it: individual relays can still refuse content, and a lost key can't be reset. Say it plainly. Don't claim it can't be censored.
- **No crypto talk.** Nostr is not a blockchain and there is no coin. Say that on the page, since much of the audience distrusts crypto.

Look and format:

- Made as a zine page: photocopied collage, typewriter text, cut-out headlines, hand-drawn arrows, a stamped "spore" logo.
- Short, loud sections instead of long text. One A4-style spread per idea.
- Printable as a real PDF zine that venues can hand out or put up.
- Translated like everything else (see Multilanguage).
- Also a "How to join" page: get a key, back it up, what happens if you lose it, how a band claims its page.

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
7. Reviews, reports, wiki views, long articles.
8. RSS intake, AI sniffing and external review cards.
9. Multilanguage: UI string catalogs first, then translation events and the on-device translate button, then localized venue fields.
10. Style taxonomy, personal lists and on-device suggestions (can start earlier, after the wiki views).
11. Music service integrations: MusicBrainz stub pages first, then Last.fm/ListenBrainz and file imports, with keys synced through evolustr.
12. nsite deploy template and the instance config file.
13. About / manifesto page and "How to join", in all themes.
