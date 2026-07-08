# Drive HTML Previewer

A tiny, **fully client-side** web app that renders HTML files stored on Google
Drive as web pages — safely and privately. There is no backend server: the page
talks directly to Google Drive from your browser, and nothing you open is stored
or sent anywhere else.

👉 **Live app:** https://drive-html-previewer.web.app

## Why?

AI tools like Claude increasingly produce **self-contained HTML artifacts** — a
single `.html` file that's a complete little web page. For example:

- a [research explainer](https://thariqs.github.io/html-effectiveness/15-research-concept-explainer.html)
- an [implementation proposal](https://thariqs.github.io/html-effectiveness/16-implementation-plan.html)
- a [design system](https://thariqs.github.io/html-effectiveness/05-design-system.html)
- an [interactive prototype](https://thariqs.github.io/html-effectiveness/08-prototype-interaction.html)
- a [slide deck](https://thariqs.github.io/html-effectiveness/09-slide-deck.html)

_See: [_The unreasonable effectiveness of HTML_](https://claude.com/blog/using-claude-code-the-unreasonable-effectiveness-of-html)
and [the `/teach` skill](https://x.com/mattpocockuk/status/2064023481398824993)_

The catch: when the content is confidential, you don't want to publish it on a
public site. But if you share the raw `.html` file over Slack, Discord, or Google
Drive, the recipient just sees the source code — they have to download it and open
it in a browser themselves, which is especially clumsy on mobile.

This app fixes that. Upload the HTML file to Google Drive, share it with your
recipient (so they have Drive access), then paste the file's link here to turn it
into a clean, viewable URL. Send them that link and they see the rendered page —
privately, with access still governed by Drive's own permissions.

## How it works

- Sign in with Google using the narrow **`drive.file`** scope — the app can only
  ever see files you explicitly pick or open via a shared link, never the rest of
  your Drive.
- **Public files** (shared as _"anyone with the link"_) render straight away with
  just the API key — no sign-in and no picker. Private files fall back to sign-in
  plus a one-click picker confirmation for that single file.
- The selected file is fetched in the browser and rendered inside a sandboxed
  `<iframe>` (scripts and form submission allowed, but — with no
  `allow-same-origin` — the frame runs at a null origin and can't touch this
  page's data, cookies, or sign-in token).
- The app never stores any file content. To avoid re-authing on every visit it
  keeps two small things, and nothing more:
  - the short-lived access token in **`sessionStorage`** (this browser tab only,
    cleared when the tab closes), reused on reload until it nears expiry; and
  - a boolean flag in **`localStorage`** remembering you've signed in before, so
    a return visit can **restore your session silently** (no popup).

  Signing out clears both, and an expired/rejected token is dropped automatically.

### Sharing a file

Open a file, click **Copy share link**, and you get:

```
https://drive-html-previewer.web.app/?id=FILE_ID
```

The recipient opens the link and sees a picker pre-pointed at that file. One
click grants `drive.file` access to just that one file, and it renders. They must
already have Drive permission to the file — the link alone grants nothing.

## Project layout

```
public/index.html   The entire app — markup, styles, and logic in one file.
firebase.json       Firebase Hosting config.
.firebaserc         Firebase project alias.
```

## Setup & configuration

The `window.APP_CONFIG` block at the top of [`public/index.html`](public/index.html)
holds the Google Cloud credentials (`CLIENT_ID`, `API_KEY`, `PROJECT_NUMBER`).
Full step-by-step setup instructions live in the HTML comment at the bottom of
that file.

The committed keys are client-side credentials restricted by HTTP referrer /
authorized JavaScript origin, so they're safe to publish.

## Quotas & rate limits

Google Drive API usage is metered against the **Cloud project**, not the API key
— the key just identifies which project a request bills to. All traffic
(everyone's, across both the signed-in and public paths) draws from one shared
pool. Default limits:

| Scope                          | Limit                     |
| ------------------------------ | ------------------------- |
| Per project, per minute        | 1,000,000 quota units     |
| Per user, per project, per min | 325,000 quota units       |
| Per project, per day           | 400,000,000 quota units   |

"Quota units" are weighted per operation, not per request: a metadata read
(`files.get`) costs ~5 units and a content download (`?alt=media`) ~200, so one
rendered file is ~**205 units** — roughly **4,800 renders/min** project-wide and
**~1,500/min per user** before throttling. For the per-user bucket, a signed-in
user is keyed by their Google account; on the keyless **public** path it falls
back to the caller's IP address.

Exceeding a limit returns **`403 userRateLimitExceeded`** (or a `429` from
backend throttling); retry with exponential backoff. The public path is the
main exposure, since anonymous shared-link traffic is unauthenticated and
project-scoped. Live usage and quota-increase requests live in
**Google Cloud Console → APIs & Services → Google Drive API → Quotas**.

See [Google Drive API — Usage limits](https://developers.google.com/workspace/drive/api/guides/limits).

## Deploy

```sh
npx firebase-tools deploy --only hosting
```

## License

MIT
