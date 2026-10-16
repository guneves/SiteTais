# Exam Week Schedule: Stack and GitHub Pages Deployment

This guide explains how to host the exam-week study schedule (17–29 Oct 2026) as a static website on GitHub Pages.

## Recommended stack

Keep it as **plain static HTML**. The page needs no framework and no build step.

| Layer | Choice | Why |
|---|---|---|
| Markup | One `index.html` file | The whole app (schedule data, rendering, checkboxes) already lives in a single file |
| Styling | Inline CSS with CSS custom properties | Light and dark themes via `prefers-color-scheme`; nothing to compile |
| Fonts | Google Fonts (Nunito, Nunito Sans) | Loaded by a `<link>` tag, with system-font fallbacks if offline |
| Logic | Vanilla JavaScript (inline `<script>`) | Renders the day cards from a `DAYS` array |
| Persistence | `localStorage` | Saves which blocks you ticked, per browser and per device |
| Hosting | GitHub Pages, "Deploy from a branch" | Free, HTTPS by default, no CI configuration needed |

What you do **not** need: Node.js, npm, React, Vite, Jekyll themes or GitHub Actions workflows. Adding any of these would only add maintenance.

### Limitation to know about

Progress is stored in `localStorage`, so ticks made on your phone do not appear on your laptop, and clearing browser data erases them. If you later want progress synced across devices, you would need a backend (for example Supabase or Firebase). For a two-week schedule this is usually not worth it.

## Step 1: Make the file standalone

The version published as a Claude artifact is a page fragment, because the artifact host adds the document skeleton automatically. For GitHub Pages, wrap the content yourself:

```html
<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
  <!-- paste here: the <title>, <meta>, <link> and <style> from the artifact -->
</head>
<body>
  <!-- paste here: the <div class="wrap"> … </div> and the <script> … </script> -->
</body>
</html>
```

Then add these two rules at the top of the `<style>` block. The artifact host supplied them before, so you have to add them now:

```css
*,*::before,*::after{box-sizing:border-box}
body{margin:0}
```

Save the result as `index.html` and open it locally in your browser to confirm it looks right before publishing.

## Step 2: Create the repository

1. Sign in at https://github.com and click **New repository**.
2. Name it, for example, `cronograma-provas`.
3. Set it to **Public**. On a free GitHub account, Pages only works with public repositories.
4. Tick **Add a README file** and click **Create repository**.

## Step 3: Upload the file

**Option A, in the browser (easiest)**

1. In the repository, click **Add file → Upload files**.
2. Drag `index.html` in.
3. Click **Commit changes**.

**Option B, with Git on the command line**

```bash
git clone https://github.com/<your-username>/cronograma-provas.git
cd cronograma-provas
cp /path/to/index.html .
git add index.html
git commit -m "Add exam week schedule"
git push origin main
```

## Step 4: Turn on GitHub Pages

1. In the repository, go to **Settings → Pages**.
2. Under **Build and deployment → Source**, choose **Deploy from a branch**.
3. Under **Branch**, choose `main` and the folder `/ (root)`, then click **Save**.
4. Wait one to two minutes. The page appears at:

```
https://<your-username>.github.io/cronograma-provas/
```

The **Actions** tab shows a "pages build and deployment" run; when it turns green, the site is live.

## Step 5: Update the schedule later

Edit the `DAYS` array inside the `<script>` block (times, labels, descriptions), then commit again, either with **Edit (pencil icon) → Commit changes** on GitHub or with `git add`, `git commit` and `git push`. Pages redeploys automatically on every commit to `main`.

Note that ticks are saved by position in each day (`d17-0`, `d17-1` …). If you insert or remove blocks inside a day, ticks for that day can shift to the wrong rows; untick and re-tick as needed.

## Optional extras

- **Add to home screen.** On your phone, open the URL and choose "Add to Home Screen" (iOS Safari) or "Install app / Add to home screen" (Android Chrome) to open it like an app.
- **Custom domain.** In **Settings → Pages → Custom domain** you can point your own domain at the site.
- **Privacy.** Anyone with the URL can see a public Pages site. The schedule has no personal data beyond study topics, but keep that in mind if you add anything private.

## Troubleshooting

| Problem | Likely cause and fix |
|---|---|
| 404 at the Pages URL | Wait a few minutes after enabling; confirm the file is named exactly `index.html` and sits in the repository root |
| Page shows but has no styles | The `<style>` block was left out when wrapping; paste it inside `<head>` |
| Layout is wider than the phone screen | The `viewport` meta tag is missing from `<head>` |
| Fonts look different | Google Fonts is blocked on that network; the system fallback fonts are being used, and the page still works |
| Ticks disappeared | Browser data was cleared, or you are on a different device or browser |

## References

- GitHub Docs. *Quickstart for GitHub Pages.* https://docs.github.com/en/pages/quickstart
- GitHub Docs. *Configuring a publishing source for your GitHub Pages site.* https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site
- MDN Web Docs. *Window: localStorage property.* https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage
