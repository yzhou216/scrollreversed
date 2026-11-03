<!--
SPDX-FileCopyrightText: 2026 Yiyu Zhou <yiyu@yiyuzhou.io>

SPDX-License-Identifier: GPL-3.0-or-later
-->

# scrollreversed

Darwin daemon reversing mouse or trackpad scrolling

System Settings applies natural scrolling to the mouse and the trackpad alike.
`scrollreversed` reverses one of the two, so they scroll in opposite
directions.

It runs as a launchd agent with a Quartz event tap.  The tap looks up the
device behind each scroll event in the I/O Registry and classifies it as
System Settings does, so a Magic Mouse counts as a mouse.  Events from unknown
senders are classified by gesture phase.  For the selected device, every delta
in the event is negated, including the one in the embedded HID event that
WebKit reads.

## Building

`nix build` builds the binary and manual pages, cross-compiling for Apple
silicon on GNU/Linux.  CI uploads both builds as artifacts, which unpack to the
same layout as `result`.

In `nix develop`, `cargo build --target aarch64-apple-darwin` also works on
GNU/Linux, and `cargo xtask man` renders the manual pages into `target/man`.

## Installing

```sh
sudo mkdir -p /usr/local/bin
sudo install result/bin/scrollreversed /usr/local/bin/
```

To install the manual pages as well, use GNU Stow instead:

```sh
sudo mkdir -p /usr/local/stow/scrollreversed
sudo cp -R result/bin result/share /usr/local/stow/scrollreversed/
sudo stow -d /usr/local/stow scrollreversed
```

Then load the launchd agent:

```sh
cp launchd/io.yiyuzhou.scrollreversed.plist ~/Library/LaunchAgents/
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/io.yiyuzhou.scrollreversed.plist
```

On first start, allow `scrollreversed` under System Settings > Privacy &
Security > Accessibility.  The agent restarts once allowed and logs to
`/tmp/scrollreversed.log`.

The permission belongs to the binary's signature, which changes with the
binary.  After an upgrade, remove `scrollreversed` from the Accessibility list,
run `launchctl kickstart -k gui/$(id -u)/io.yiyuzhou.scrollreversed`, and allow
it again.

## Uninstalling

```sh
launchctl bootout gui/$(id -u)/io.yiyuzhou.scrollreversed
rm ~/Library/LaunchAgents/io.yiyuzhou.scrollreversed.plist
sudo rm /usr/local/bin/scrollreversed
```

With Stow, run `sudo stow -D -d /usr/local/stow scrollreversed` in place of the
last command.

## Usage

See scrollreversed(1) and scrollreversed(5).

## License

scrollreversed is free software: you can redistribute it and/or modify it
under the terms of the GNU General Public License as published by the Free
Software Foundation, either version 3 of the License, or any later version.

This program is distributed in the hope that it will be useful, but WITHOUT
ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with
this program.  If not, see <https://www.gnu.org/licenses/>.
