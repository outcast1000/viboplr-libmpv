# viboplr-libmpv

The libmpv that [Vibo](https://github.com/outcast1000/viboplr) plays through: a trimmed, self-contained build from pinned sources, for macOS arm64 and Windows x64.

Every decoder, demuxer, parser and filter stays. Encoders, muxers, capture devices, Vulkan, Blu-ray/DVD/CD, JavaScript and the external codec libraries go; FFmpeg's own decoders plus dav1d cover playback. Lua stays, because without it mpv has no `ytdl` option and the engine sets that at creation. Everything links statically into one library, and the build fails if that library depends on anything but the OS.

## What is here

| File | What it is |
|---|---|
| `sources.txt` | Every source the build uses: url + sha256. Bump deliberately, then run `./build.sh pin` to refresh the hashes. |
| `build.sh` | The recipe. `./build.sh build` fetches (sha256-checked), builds and packages into `$LIBMPV_WORK/out`; `./build.sh resume` redoes mpv + packaging on top of a finished deps/FFmpeg build. |
| `patches/mpv-*.patch` | Fixes to upstream mpv for this configuration. Each says why it exists and when it can go. |
| `.github/workflows/build-libmpv.yml` | Builds both platforms, runs the app's engine tests against each build, and publishes on request. |

The macOS build runs natively (clang + Xcode or the Command Line Tools). The Windows build runs in MSYS2 UCRT64; the workflow lists the pacman packages it expects.

## Building a new libmpv

```sh
gh workflow run build-libmpv.yml --repo outcast1000/viboplr-libmpv -f publish=true -f mpv_ref=<mpv commit>
gh run list --workflow build-libmpv.yml --repo outcast1000/viboplr-libmpv
```

- `mpv_ref`: build this mpv commit instead of the release in `sources.txt`. The app's `npm run bump` prints the command with mpv master's head filled in when master is ahead of the pin.
- `app_ref` (default `main`): the `outcast1000/viboplr` ref whose engine tests (`cargo test --lib mpv_engine`) must pass against the new build. A skipped suite, or one that loaded any other libmpv, fails the run.
- `publish`: publish the archives as a release named after the build, e.g. `vibo-git-dc4b285c93-84dbcf9b`, tagged on the commit that built it. An asset that already exists is never overwritten, because a released app may already pin those bytes.

The build name is `vibo-<mpv version or git-<sha>>-<recipe>`. `<recipe>` is the first 8 hex digits of the sha256 of `sources.txt`, `build.sh`, the patches and `mpv_ref`, so any change to the recipe, comments included, gives a new name.

## Adopting a build in the app

Nothing here changes the app. In `outcast1000/viboplr`, re-pin each platform in `scripts/libmpv.lock.json` by hand (`url`, `upstream`, `sha256`, `mpvGitHash`, `build`) from the release's assets, then:

```sh
node scripts/fetch-libmpv.mjs && (cd src-tauri && cargo test --lib)
```

Test playback on both platforms before you commit the lock. The next app release ships it.

Builds published before this repo existed stay on the app repo's `vendor-libmpv` release, which released apps pin. Don't move or delete them.

## License

GPL-3.0, like the app. The library this produces is GPL too: it statically links FFmpeg and mpv built with GPL parts.
