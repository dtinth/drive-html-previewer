# Drive HTML Previewer

A tiny, **fully client-side** web app that renders HTML files stored on Google
Drive as web pages — safely and privately. There is no backend server: the page
talks directly to Google Drive from your browser, and nothing you open is stored
or sent anywhere else.

👉 **Live app:** https://drive-html-previewer.web.app

## How it works

- Sign in with Google using the narrow **`drive.file`** scope — the app can only
  ever see files you explicitly pick or open via a shared link, never the rest of
  your Drive.
- The selected file is fetched in the browser and rendered inside a sandboxed
  `<iframe>` (scripts allowed, but no access to this page's data, cookies, or
  sign-in token).
- Access tokens live in memory only; nothing is persisted.

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

## Deploy

```sh
npx firebase-tools deploy --only hosting
```

## License

MIT
