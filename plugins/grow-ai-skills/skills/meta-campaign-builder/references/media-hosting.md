# Media hosting

Read this when the user's creatives are local files, which is almost always.

Meta's upload tools (`ads_creative_upload_video`, `ads_creative_upload_image`)
take a **public URL only**. There is no local-path parameter. Meta's server
fetches the bytes itself, unauthenticated, so the URL must return raw bytes
with no sign-in and no interstitial page. Google Drive, Dropbox and similar
share links fail for exactly this reason: they return HTML, not media.

Hosting is genuinely temporary — see "Cleanup" for why it can go almost
immediately.

## Step 0: Make sure `gh` is ready

Run both checks before anything else. Handle whichever fails.

```bash
command -v gh >/dev/null && gh auth status 2>&1 | head -5
```

### If `gh` is not installed

Install it with the platform's package manager. This is a routine developer
tool, so install it rather than asking, but say what you are doing.

```bash
# macOS
brew install gh

# Debian / Ubuntu
sudo apt update && sudo apt install -y gh

# Fedora
sudo dnf install -y gh

# Windows
winget install --id GitHub.cli
```

If no package manager is present, or the install needs a password you cannot
supply, hand the user the one command to run and continue when they confirm.

### If `gh` is installed but not authenticated

Authentication is interactive and cannot be driven from here — it opens a
browser and needs a device code. Ask the user to run it themselves:

```
gh auth login
```

Tell them to choose **GitHub.com**, **HTTPS**, and authenticate in the
browser. The `repo` scope is required and is included by default.

Verify afterward with `gh auth status`, then continue.

### Confirm the owner

```bash
gh api user --jq .login
```

**Do not assume the login from `gh auth status`.** That prints a local
account label which is frequently not the API login, and using it 404s on
repo creation. Use whatever `gh api user` returns.

## Step 1: Create and seed the repo

```bash
OWNER=$(gh api user --jq .login)
REPO=ad-media-$(date +%Y%m%d-%H%M%S)

gh repo create "$OWNER/$REPO" --public -d "temporary ad media host"

gh api "repos/$OWNER/$REPO/contents/README.md" -X PUT \
  -f message="init" \
  -f content="$(printf 'Temporary host for ad creatives.\n' | base64)"
```

The seed commit is required: **a release cannot be created on an empty
repo**, and it fails with a 422 that does not explain why.

A timestamped name avoids collisions and makes cleanup unambiguous.

**The repo must be public during upload.** Meta fetches unauthenticated, so a
private repo fails. This is unavoidable, but the exposure window is minutes —
see Cleanup.

## Step 2: Upload videos as release assets

```bash
gh release create v1 --repo "$OWNER/$REPO" -t "creatives" -n "temp"
cd <media-dir> && gh release upload v1 *.mp4 --repo "$OWNER/$REPO"
```

Run in the background when the media is large; several hundred MB is normal
for a batch of video ads.

Video URLs follow a fixed pattern, so build them rather than querying:

```
https://github.com/<owner>/<repo>/releases/download/v1/<filename>
```

## Step 3: Commit images, do not release them

This is the part that is easy to get wrong, and it costs a failed batch.

Release assets serve via a **302 redirect** to a signed, expiring URL.
Meta's *video* fetcher follows that redirect. Meta's *image* fetcher does
not, and reports a misleading error:

```
Image Wasn't Downloaded: ... couldn't be downloaded. Please ensure the image
is accessible via the internet and is not blocked by a robots.txt
```

The image is fine and public. The redirect is the problem. Commit images into
the repo instead and serve from `raw.githubusercontent.com`, which returns
`image/jpeg` directly with no redirect:

```bash
for f in thumbs/*.jpg; do
  gh api "repos/$OWNER/$REPO/contents/thumbs/$(basename "$f")" -X PUT \
    -f message="add $(basename "$f")" \
    -f content="$(base64 -i "$f")"
done
```

Image URLs:

```
https://raw.githubusercontent.com/<owner>/<repo>/main/thumbs/<filename>
```

Do not commit videos this way. GitHub's file API is unsuited to large
binaries, and raw serving has size limits release assets do not.

**Skip this step entirely** if using the `picture` fallback for thumbnails
(see SKILL.md Step 4) — that needs no image hosting at all.

## Step 4: Verify before uploading the batch

One check beats ten failed uploads:

```bash
curl -sIL "<video-url>" | grep -iE "^HTTP|^content-type|^content-length"
curl -sI  "<image-url>" | grep -iE "^HTTP|^content-type|^content-length"
```

Expect `application/octet-stream` and a matching byte count for video after
the redirect, and a direct `200` with `image/jpeg` for images.

## Cleanup

**Meta keeps its own copies of everything, so the repo can be deleted as soon
as the creatives are built.** Verified: after `ads_create_creative`, the
creative's `thumbnail_url` points at Meta's CDN (`fbcdn.net`), with no
reference back to the source URL. Videos are copied into the ad account at
upload time and get their own account-scoped `video_id`.

So the safe deletion point is **immediately after the last creative is
created** — not after ads are built, and not after the user reviews them.
Nothing downstream reads the source URLs again.

```bash
gh repo delete "$OWNER/$REPO" --yes
```

`gh repo delete` needs the `delete_repo` scope, which is not granted by
default. If it fails, either add the scope:

```
gh auth refresh -h github.com -s delete_repo
```

or make the repo private instead, which is equally safe post-upload:

```bash
gh repo edit "$OWNER/$REPO" --visibility private --accept-visibility-change-consequences
```

Do this proactively at the end of the build and tell the user it is done.
Leaving a public repo of a client's unreleased ad creatives sitting around is
a real problem, not a tidiness issue.

## If repo creation is blocked

Permission policy may block `gh repo create`. Do not abandon the approach —
hand the user one runnable command and continue when they confirm:

```
gh repo create <owner>/<repo> --public -d "temporary ad media host" && \
gh release create v1 --repo <owner>/<repo> -t "creatives" -n "temp" && \
cd <media-dir> && gh release upload v1 *.mp4 --repo <owner>/<repo> && echo DONE
```

## Alternatives

Any host serving raw bytes without auth works. Vercel Blob (`vercel blob put`)
is a good second choice, though it needs a store to exist first and is
billable. S3, Cloudflare R2 and plain static hosting are all fine.

**Not usable:** Google Drive, Dropbox, OneDrive, or any link returning an
interstitial or requiring sign-in.
