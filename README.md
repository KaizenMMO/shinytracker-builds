# shinytracker-builds

Download host for Shiny Tracker Overlay releases, and the place Linux and macOS
preview packages are built.

## Downloads

Grab the newest package from [Releases](https://github.com/KaizenMMO/shinytracker-builds/releases/latest).
The in-app updater reads `manifest.json` from that same release, so an installed
copy will offer the update by itself.

## Building the Unix preview packages

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
`SHINYTRACKER_SOURCE_TOKEN` secret, so the source itself stays private. Build
logs in this repository are public: keep secrets out of build output.

The Windows package is not built here. It is built on the developer's machine
and uploaded to the release by hand.
