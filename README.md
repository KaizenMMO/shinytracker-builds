# shinytracker-builds

Download host for Shiny Tracker Overlay releases, and the place Linux and macOS
preview packages are built.

## Downloads

Grab the newest package from [Releases](https://github.com/KaizenMMO/shinytracker-builds/releases/latest).
The in-app updater reads `manifest.json` from that same release, so an installed
copy will offer the update by itself.

Linux: [ShinyTrackerOverlay-Linux-x86_64.tar.gz](https://github.com/KaizenMMO/shinytracker-builds/releases/latest/download/ShinyTrackerOverlay-Linux-x86_64.tar.gz)
is a permanent link to the newest Linux build. Extract it and run
`ShinyTrackerOverlay/ShinyTrackerOverlay` from an X11 session, with
`tesseract-ocr` and `tesseract-ocr-eng` installed.

## Linux packages for every release

`linux-release.yml` builds Linux for a **published** release and attaches it to
that same release. The source repo's `packaging/publish_release.py` starts it
after every Windows publish, so nothing needs doing by hand. To run it yourself:

```sh
gh workflow run linux-release.yml --repo KaizenMMO/shinytracker-builds \
  -f source_sha=<pushed commit in the source repo> -f release_tag=v3.8
```

It checks the source's version matches the tag, runs the full suite, builds on
ubuntu-22.04 (old glibc, so the build runs on every current distribution),
smoke-tests the frozen app, uploads `ShinyTrackerOverlay-Linux-x86_64.tar.gz`
and its `.sha256` under that fixed name, then adds a `linux` entry to the
release's `manifest.json`. Linux copies of the app read only that entry and
offer the download; Windows clients read the top level, which is untouched.

The source repo is private. The workflow reads it with a read-only deploy key
stored here as `SHINYTRACKER_SOURCE_SSH_KEY`. To set it up or replace it:

```sh
ssh-keygen -t ed25519 -N "" -C "shinytracker-builds CI" -f st_deploy
gh repo deploy-key add st_deploy.pub --repo KaizenMMO/ShinyTrackerOverlay --title "shinytracker-builds CI (read-only)"
gh secret set SHINYTRACKER_SOURCE_SSH_KEY --repo KaizenMMO/shinytracker-builds < st_deploy
rm st_deploy st_deploy.pub
```

## Building the Unix preview packages (macOS)

This repository is public, so its Actions runs are free. The same build used to
run in the private source repository, where macOS runners are billed at ten
times their actual wall-clock time — ten short Mac builds there cost roughly 890
of the account's 2,000 monthly minutes in September 2026. Run it here instead.

1. Create (or reuse) a **draft release** in this repository with the tag you
   want the packages attached to. The workflow uploads into an existing tag; it
   does not create one. A public repo also cannot write artifacts back into a
   private one, which is why the draft release is the hand-off point.
2. Start the build against an exact source commit:

   ```sh
   gh workflow run native-preview.yml \
     --repo KaizenMMO/shinytracker-builds \
     -f source_sha=<commit sha in the source repo> \
     -f release_tag=<the draft release tag> \
     -f run_tests=true
   ```

   Pass `-f run_tests=false` only when that exact commit has already passed the
   suite on this same matrix.

3. The matrix produces three packages — `Linux-x86_64`, `macOS-arm64` and
   `macOS-x86_64` — and uploads them to the draft release. Publish the release
   when they have been checked.

The workflow checks the source out at the requested commit using the
`SHINYTRACKER_SOURCE_SSH_KEY` deploy key, so the source itself stays private. Build
logs in this repository are public: keep secrets out of build output.

The Windows package is not built here. It is built on the developer's machine
and uploaded to the release by hand.
