# ClawPhone | AgentPhone

Scripts, tweaks, and notes for running agent CLIs on Android phones through Termux.

The goal is to keep things as native to Termux as possible, only using `proot` or heavier compatibility layers when Android's userspace makes that unavoidable.

## Antigravity CLI on Termux

The first packaged script is an installer for Google's Antigravity CLI (`agy`) on ARM64 Android/Termux. It installs/checks dependencies, installs the official Linux ARM64 binary, patches the TCMalloc VA assumptions for common Android devices, creates a glibc wrapper, adds shell shortcuts, and supports uninstall.

Shoutout to [Brajesh](https://github.com/Brajesh2022) for figuring out and sharing the working Termux setup this is based on. This repo turns those steps into one script so it is easier for people to install, update, and uninstall.

```bash
curl -fsSL https://raw.githubusercontent.com/marshallrichards/ClawPhone/main/scripts/install-antigravity-termux.sh | bash
source ~/.bashrc
agy --version
```

Details: [`docs/antigravity-termux.md`](docs/antigravity-termux.md)

## OpenClaw Notes

I started running OpenClaw on a cheap Android smartphone as an isolated sandbox for OpenClaw agents with access to phone hardware. It runs in the background inside Termux in a `tmux` session, and I can interact with it over Discord like a normal OpenClaw agent.

You can use a cheap prepaid Android phone or any old Android 8+ phone you have lying around.

Things to note:

1. Install Termux. I recommend `tmux`, a text editor like `nvim`, `nodejs-lts`, and `python`. Also install Termux:API and Termux:GUI if you want hardware and UI access.
2. Install OpenClaw with `npm install -g openclaw@latest`, not the bash installer, since the bash installer can fail on Android.
3. If dependencies fail during install, install the missing packages with `pkg` and rerun the OpenClaw install.
4. `llama.cpp` may compile from source because Termux lacks normal desktop glibc behavior. This can take 15-30 minutes.
5. Errors about missing `systemd` are expected. Run the OpenClaw Gateway in the foreground or inside `tmux`.
6. OpenClaw expects `/tmp/openclaw`, but raw Termux should use `$PREFIX/tmp`. Add this to `~/.bashrc`:

```bash
export TMPDIR="$PREFIX/tmp"
export TMP="$TMPDIR"
export TEMP="$TMPDIR"
```

7. Add Termux-friendly logging to `openclaw.json`:

```json
"logging": {
  "level": "info",
  "file": "/data/data/com.termux/files/usr/tmp/openclaw/openclaw-YYYY-MM-DD.log"
}
```

8. Create the temp directory:

```bash
mkdir -p /data/data/com.termux/files/usr/tmp/openclaw
```

9. Run `source ~/.bashrc`, then `openclaw gateway`.
10. Run `openclaw onboard` and add your model token.
11. Set `gateway.bind` to `lan` so the dashboard listens on `0.0.0.0` and can be reached from the phone's LAN IP.
12. Tell OpenClaw it is running inside Termux on Android, and that Termux:API and Termux:GUI are installed if you want it to use phone hardware.

You can also let OpenClaw write overlays on the screen. Install the Termux:GUI app, run `pkg install termux-gui`, start `overlay_daemon.py` in another `tmux` pane/window, and tell OpenClaw it can use that daemon when it needs to display something to the user.