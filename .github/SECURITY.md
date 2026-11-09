# Security policy

Omarchy IPTV handles provider credentials. Most paid playlist and guide
addresses carry a username and password inside the URL, and the plugin's
job is to use them without showing them anywhere. A bug that exposes them,
or one that lets a playlist or guide file make the plugin do something it
should not, is a security bug, and it should be reported privately.

## Supported versions

Only the latest release gets fixes. Fixes ship as a new release on `main`.

| Version | Supported |
|---|---|
| Latest release (see [`CHANGELOG.md`](https://github.com/rmcdavid/omarchy-iptv/blob/main/CHANGELOG.md)) | Yes |
| Anything older | No: update first |

To update, run `omarchy plugin update io.github.rmcdavid.iptv`, then
`omarchy-restart-shell`. The running shell keeps the old version loaded
until it restarts.

## How to report a vulnerability

**Do not open a public issue, pull request or discussion for a security
problem.**

Report it privately through GitHub:
**[Report a vulnerability](https://github.com/rmcdavid/omarchy-iptv/security/advisories/new)**
(the repository's **Security** tab, then **Report a vulnerability**). Only
the maintainer can see the report.

Please include:

- The plugin version (`grep '"version"' ~/.config/omarchy/plugins/io.github.rmcdavid.iptv/manifest.json`),
  your Omarchy version and your `mpv --version` first line.
- What an attacker controls (a playlist, a guide file, a stream, a logo, a
  local program) and what they gain.
- Steps to reproduce, with the smallest playlist or guide file that shows it.
- Where the secret or the effect shows up: a notification, the guide, the
  bar tooltip, the shell journal, a process command line, the accessibility
  bus, a file, a network request.

**Never send your real provider address, username or password, even in a
private report.** Use a free public list, or a made-up address with the
same shape, such as `http://user:p%40ss@example.invalid:8080/get.php?type=m3u_plus`.
If the bug depends on a particular character in a username or password (a
quote, a comma, an `@`, an unusual space), replace the letters and digits
but keep that character where it was.

If you think your real credentials have already been exposed, change them
with your provider. Fixing the plugin does not undo a leak.

## What happens next

This is a one-maintainer project, so these times are goals, not guarantees:

- You should get an acknowledgement within 7 days.
- The maintainer will confirm whether it reproduces, agree a fix with you
  and say when a release is planned.
- The fix ships as a new release. After that, a GitHub security advisory is
  published and the release notes describe the fix. You are credited by
  name or handle unless you ask not to be.

Please give the fix time to ship before you disclose publicly.

## Scope

**In scope.** Anything in this repository that breaks one of the plugin's
own promises:

- **Credentials stay private.** A playlist or guide URL, or a username or
  password from one, appears somewhere other than the plugin's private
  state file (`~/.local/state/omarchy-iptv/state.json`, mode 0600).
  Examples: a notification, guide or bar text, the shell journal or
  console, the helper's output, a process command line, the desktop
  accessibility bus, your shell history, or `~/.config/omarchy/shell.json`.
  The same applies when one guide's credentials are sent to another server.
- **Provider data is untrusted.** Content from a playlist, guide file,
  stream or logo that:
  - adds markup, links or images to a notification or guide text
  - makes the plugin connect to this computer or your local network
  - reaches a shell, or reaches mpv as an option rather than as a stream
    address
  - writes outside the plugin's own cache, state and runtime directories,
    or anything inside the plugin's install directory
  - uses unbounded memory or time while being downloaded or parsed
- **The player's safety settings hold.** A way for playlist data or your
  `mpvArgs` setting to turn off the options the plugin sets on the player
  after your own.
- **File permissions.** A plugin file that is meant to be private (mode
  0600, or 0700 for a directory) but is readable by other accounts.

**Out of scope:**

- The streams, playlists and guides themselves, and the security of any
  IPTV provider. The plugin ships no content.
- Bugs in mpv, Quickshell, Hyprland or Omarchy that the plugin does not
  cause or make worse. Report those upstream. If the plugin makes one
  reachable, that part is in scope.
- Attacks that need code already running as your user, unless the plugin
  documents a defense against exactly that. For example, the helper makes
  itself non-dumpable so other programs you run cannot read its
  environment.
- Copies you make yourself, such as a backup of `state.json` or a URL you
  type into a terminal. The README explains why each of these carries your
  credentials.
- Channel logos being fetched from the servers a playlist names, once you
  turn on `showLogos`. That is documented and off by default. A logo fetch
  that reaches your local network is in scope.
