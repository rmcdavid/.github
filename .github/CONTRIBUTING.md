# Contributing to Omarchy IPTV

Thanks for helping. Bug reports, playlist and guide compatibility reports,
documentation fixes and code are all welcome.

The plugin ships no content. Issues or pull requests that ask for, link to
or add playlists, channels or streams will be closed.

By taking part you agree to follow the
[Code of Conduct](https://github.com/rmcdavid/.github/blob/main/CODE_OF_CONDUCT.md).

## Reporting bugs and asking for features

Use the [issue forms](https://github.com/rmcdavid/omarchy-iptv/issues/new/choose).
Each form asks for what is needed to reproduce the problem.

- Read **Troubleshooting** in the
  [README](https://github.com/rmcdavid/omarchy-iptv#troubleshooting) first.
  Many problems, like a guide that does not show an update, are answered
  there.
- **Never paste a playlist or guide URL, a username or password, an Xtream
  login, or the contents of `state.json`.** Paid provider addresses carry
  your credentials. The plugin only ever shows a host name. If you see a
  full address anywhere, that is a security bug: report it privately as
  described in the
  [security policy](https://github.com/rmcdavid/omarchy-iptv/security/policy),
  not in an issue.
- If you can reproduce a problem with a free public list such as
  `https://iptv-org.github.io/iptv/countries/us.m3u`, say so. That is the
  fastest way to get it fixed.

## Branches: `dev` is the source, `main` is the install

`omarchy plugin add` clones the whole repository, so everything on `main`
ends up in every user's plugin folder. For that reason:

- **`dev`** holds everything: the source, the design and QA documents, the
  tests, the gate and the tooling. **Base your work and your pull request
  on `dev`.**
- **`main`** holds only the files the plugin needs to run. It is generated
  from `dev` by `scripts/release.py build`, using an explicit allowlist.
  Nobody commits to `main` by hand. A pull request against `main` will be
  retargeted or closed.

## Setting up

You need an Omarchy 4.x machine. The gate runs `omarchy plugin validate`
and lints against the shell's files in `/usr/share/omarchy/shell`. You also
need:

- `python3`, `node` and `mpv` (0.41 or newer)
- the Qt 6 tools `qmllint` and `qmltestrunner`. The gate looks in
  `/usr/lib/qt6/bin`. Set `QT_BIN` if yours are somewhere else.

```bash
git clone -b dev https://github.com/rmcdavid/omarchy-iptv.git
```

```bash
cd omarchy-iptv && ./scripts/check.sh
```

Read these before you change anything:

- [`docs/PRODUCT.md`](https://github.com/rmcdavid/omarchy-iptv/blob/dev/docs/PRODUCT.md):
  what the plugin is for and the decisions that are already settled
- [`docs/ARCHITECTURE.md`](https://github.com/rmcdavid/omarchy-iptv/blob/dev/docs/ARCHITECTURE.md):
  data, processes and security
- [`docs/UX.md`](https://github.com/rmcdavid/omarchy-iptv/blob/dev/docs/UX.md):
  how the guide and bar look and behave
- [`CLAUDE.md`](https://github.com/rmcdavid/omarchy-iptv/blob/dev/CLAUDE.md):
  the full engineering rules. It is written for coding agents, and the same
  rules apply to people.

For a larger change, open an issue first so the approach can be agreed
before you write it.

## Trying a change

- **Use the dev harness.** `scripts/dev-harness/run.sh` loads the plugin
  straight from your checkout with a fake shell and scratch directories,
  so you do not need to install it. Its
  [README](https://github.com/rmcdavid/omarchy-iptv/blob/dev/scripts/dev-harness/README.md)
  explains the scenarios.
- **Leave your own install alone.** Do not edit
  `~/.config/omarchy/plugins/io.github.rmcdavid.iptv`. Do not run your
  checkout's helper against the default player socket
  (`$XDG_RUNTIME_DIR/omarchy-iptv/mpv.sock`): a helper from another build
  replaces the player that is running there. The harness uses a scratch
  socket.
- After you change `Service.qml` in a real shell, run
  `omarchy-restart-shell`. Kept-loaded services do not reload on their own.
- Test with a free public list or a synthetic one:
  `scripts/gen-playlist.py` writes M3U and XMLTV files of any size. Write
  them to a scratch directory and never commit them.

## Rules the gate enforces

`./scripts/check.sh` must pass before every commit. It runs `omarchy plugin
validate`, qmllint, the node, Python and QML test suites, and a set of
project guards. Most of the rules below are checked there.

1. **URLs are reduced to scheme and host everywhere they could be seen:**
   notifications, tooltips, guide text, the console, IPC replies, the
   helper's stdout and stderr, and the accessibility bus. Use the model's
   redaction (`redactUrls`). Do not write your own.
2. **Processes are started with an argument list only.** Never use
   `bash -c`, `shell=True` or a command built by string interpolation.
   Playlist data reaches mpv or curl as separate arguments, never as shell
   text.
3. **The helper is Python 3 standard library only.** Do not add a runtime
   dependency.
4. **No `sudo`, and no writes inside the plugin directory at runtime.**
5. **Theme tokens only.** Colors come from `Commons.Color.*` and metrics
   from `Style.*`. A literal value needs a `CONVENTION-EXCEPTION` comment
   explaining why.
6. **Every QML `Text` declares `Text.PlainText`.** The default, AutoText,
   renders markup from a channel name, including images.
7. **Code files are ASCII.** Glyphs are allowed in `.qml` only.
8. **Logic goes in the model, where node can test it.** The model is ES
   modules under `model/`, and `Model.mjs` is generated. Edit the right
   part, then run `node scripts/split-model.mjs --relink`.
9. **The helper is a stub plus parts under `bin/helper/`.** After any edit
   to a part, run `scripts/split-helper.py --seal`.
10. **Shipped files stay under 448 KiB.** A new file the plugin loads at
    runtime must be added to `ALLOWLIST` in `scripts/release.py`. The
    shipped `README.md` must not link to `docs/`, `tests/` or `scripts/` by
    path. Link the `dev` branch by URL instead.

## Tests

- Add a test with every fix and feature. A rule implemented in both
  JavaScript and Python gets one shared JSON fixture that both run.
- `check.sh` holds a minimum count for each suite. When you add tests,
  raise that count in the same commit. Never lower it.
- Test fixtures are synthetic. Never commit a real provider's playlist,
  guide, address or credentials, even with the credentials removed.

## Commits and pull requests

- Keep commits small and logical. Write messages in the present tense and
  explain why, not what the diff already shows.
- Never commit with a failing `check.sh`.
- Defect IDs such as `D-SINK-85` are assigned by the maintainer and tracked
  in `docs/STATUS.md`. The gate rejects an ID cited in a Markdown file that
  has no row there, so do not invent one. Describe the problem in words, or
  link the issue.
- Do not edit `CHANGELOG.md` release headings. If users would notice your
  change, put a draft changelog entry in the pull request description. The
  maintainer writes the release notes when a release is cut.
- Fill in the pull request template, including which Omarchy version you
  tested on.

## License

By contributing, you agree that your contributions are licensed under the
[MIT License](https://github.com/rmcdavid/omarchy-iptv/blob/main/LICENSE).
