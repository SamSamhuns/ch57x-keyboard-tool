# SikaiCase 6-key + knob on macOS

How to program the SikaiCase 6-key + 1-knob macro pad from a Mac with this repo, instead of the vendor's "MINI KEYBOARD" app.
The mapping is saved inside the pad, so nothing has to keep running on the Mac after you upload it.

| Item | Value |
| --- | --- |
| Pad | SikaiCase 6 keys + 1 knob, connected with USB-C |
| USB ID | `1189:8840` (listed as "USB Composite Device") |
| `model` in the config | `ch57x-1` |
| Your mapping | [config/sikaicase-6key.yaml](config/sikaicase-6key.yaml) |
| Factory mapping | [config/sikaicase-6key-factory.yaml](config/sikaicase-6key-factory.yaml) (keys a–f, knob 1/2/3) |

## Current layout

Each key is listed with the letter it typed with the factory mapping, so you can tell which is which.

| Key | Action | Value in the config |
| --- | --- | --- |
| Top left (`a`) | Copy | `cmd-c` |
| Top center (`b`) | Paste | `cmd-v` |
| Top right (`c`) | Undo | `cmd-z` |
| Bottom left (`d`) | Dictation on/off | `dictation` |
| Bottom center (`e`) | Screenshot of a selected area | `cmd-shift-4` |
| Bottom right (`f`) | Lock screen | `ctrl-cmd-q` |
| Knob, turn left | Zoom out | `opt-cmd-minus` |
| Knob, press | Zoom on/off | `opt-cmd-8` |
| Knob, turn right | Zoom in | `opt-cmd-equal` |

### macOS settings this layout needs

- **Zoom (knob):** System Settings → Accessibility → Zoom → turn on **Use keyboard shortcuts to zoom**.
- **Dictation (bottom left):** System Settings → Keyboard → Dictation → turn it on and set **Shortcut** to the microphone key (**Press 🎤**).
  The pad sends the same key code as the 🎤 key on Apple keyboards.

## Setup (once)

Run all commands in this guide from the root of this repo.

```sh
brew install rust
cargo build --release
```

## Change the shortcuts

1. Connect the pad with the USB-C cable. It can't be programmed over wireless.
2. Edit `config/sikaicase-6key.yaml`.
3. Check the file: `./target/release/ch57x-keyboard-tool validate config/sikaicase-6key.yaml`
4. Upload it: `sudo ./target/release/ch57x-keyboard-tool upload config/sikaicase-6key.yaml`
5. Press each key and turn the knob to check.

`sudo` is needed because macOS's own keyboard driver owns the pad, and the USB library this tool uses needs root to take it over while it writes.
Without `sudo` you get `claim interface: Access denied`, and nothing is sent.

To go back to the factory mapping, upload `config/sikaicase-6key-factory.yaml` the same way.

## What you can put on a key or the knob

| Kind | Examples | Notes |
| --- | --- | --- |
| Single key | `a`, `enter`, `escape`, `space`, `tab`, `f13` | All key names: `./target/release/ch57x-keyboard-tool show-keys` |
| Key combo | `cmd-c`, `cmd-shift-4`, `ctrl-opt-cmd-t` | Modifiers: `cmd`, `opt`, `ctrl`, `shift`, plus right-hand `rcmd`, `ropt`, `rctrl`, `rshift` |
| Sequence | `cmd-space,t,e,r,m,enter` | Comma-separated presses, one after another, up to 18 on this pad |
| Sequence with pauses | `{delay(150)}cmd-space,t,e,r,m,enter` | Pause in milliseconds between presses, up to 6000 |
| Media and system | `play`, `next`, `prev`, `mute`, `volumeup`, `volumedown`, `screenbrightnessup`, `screenbrightnessdown`, `dictation` | Can't be combined with modifiers or other keys |
| Mouse | `click`, `rclick`, `mclick`, `wheelup`, `wheeldown`, `wheel(-3)`, `move(20,0)`, `drag(left,0,20)` | Mouse actions only take `ctrl`, `shift` or `alt` as modifiers |
| Raw key code | `<104>` | Any HID keyboard code, in decimal (104 is F13) |

Full syntax: [doc/actions.md](doc/actions.md).

### Handy macOS shortcuts

| Action | Value |
| --- | --- |
| Spotlight | `cmd-space` |
| Mission Control / app windows | `ctrl-up` / `ctrl-down` |
| Previous / next desktop | `ctrl-left` / `ctrl-right` |
| Switch app | `cmd-tab` |
| Screenshot: full screen / area / options | `cmd-shift-3` / `cmd-shift-4` / `cmd-shift-5` |
| Area screenshot to clipboard | `ctrl-cmd-shift-4` |
| Emoji picker | `ctrl-cmd-space` |
| Force Quit | `opt-cmd-escape` |
| Hide app / minimize window | `cmd-h` / `cmd-m` |
| Browser: new tab / reopen closed tab | `cmd-t` / `cmd-shift-t` |
| Zoom inside an app: in / out / reset | `cmd-equal` / `cmd-minus` / `cmd-0` |

Knob ideas:

| Use | Turn left (`ccw`) / press / turn right (`cw`) |
| --- | --- |
| Volume | `volumedown` / `mute` / `volumeup` |
| Scroll | `wheelup` / `click` / `wheeldown` |
| Browser tabs | `ctrl-shift-tab` / `cmd-t` / `ctrl-tab` |
| Undo / save / redo | `cmd-z` / `cmd-s` / `cmd-shift-z` |

## Run Shortcuts.app shortcuts (or any app) from a key

The pad can only send keys; it can't start programs by itself. To run a shortcut from Shortcuts.app:

1. Put a combo that nothing else uses on the key, for example `ctrl-opt-shift-cmd-1`, and upload.
2. In Shortcuts.app, open the shortcut, open its details (ⓘ), click **Add Keyboard Shortcut**, and press the pad key.

The same works for any app that lets you record a hotkey.
Avoid `ctrl-opt-shift-cmd-dot`: macOS uses it to start a system diagnostic (sysdiagnose).

## Troubleshooting

- **`claim interface: Access denied`:** run the upload with `sudo`.
- **`CH57x keyboard device not found`:** connect the pad with the USB-C cable. To check that the Mac sees it, run `ioreg -p IOUSB -w0 | grep "USB Composite Device"`.
- **The pad stops typing after an upload:** unplug it and plug it back in.
- **The knob works the wrong way round:** swap the `ccw` and `cw` values.
- **The zoom keys do nothing:** turn on **Use keyboard shortcuts to zoom** (see above).
- **The dictation key does nothing:** check that Dictation is on and its shortcut is **Press 🎤**.
  To use a different trigger instead, choose **Customize…** for the Dictation shortcut, press a combo such as ⌃⌥⇧⌘D on the Mac keyboard,
  and put `ctrl-opt-shift-cmd-d` on the pad key instead of `dictation`.

## Why not the vendor app

Checked on 12 Nov 2026 by downloading the macOS app (`LEN-mac_OS.zip` from sikaicase.com) and inspecting it without running it:

- It has no code signature, so Gatekeeper blocks it, and you would have to override macOS security to open it.
- It is Intel-only (it runs under Rosetta on Apple silicon), built with Qt 5.12.9 from 2020, and still uses the template app ID `com.yourcompany.MINI-KEYBOARD`.
- The main program has no networking or program-launching code; it only sends HID reports to the pad.
  It looks careless rather than malicious, but it is closed source.

This tool is open source, built on your Mac from the code in this folder, and programs every key from one text file.

## Note on the `dictation` key

`dictation` is a local addition to this copy of the tool (`MediaCode` in `src/keyboard/mod.rs`); it is not in the upstream GitHub project.
It sends consumer key code `0xCF`, the code of the 🎤 key on Apple keyboards. Keep the change if you update the repo.
