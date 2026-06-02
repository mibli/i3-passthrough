# i3-passthrough

**Currently just a parked idea for useful utility, will get to it when I have time and a lot of coffee**

A ultra-lightweight, single-shot Unix utility that dynamically switches your `i3wm` mode to suspend global desktop keybindings when focusing developer-centric windows (like `zellij`, `wezterm`, or `xterm`).

## The Problem
Tiling window managers (like `i3`) grab key combinations globally. If you use `$mod+h` to move your i3 focus left, you cannot easily use `$mod+h` inside terminal workspaces or multiplexers like Zellij. 

Traditional workarounds force you to manually toggle an i3 "passthrough mode" using a shortcut, or mess around with heavy config mapping files.

## The Solution
`i3-passthrough` follows the Unix philosophy: do one thing and do it well. 

It is designed as a **single-shot executable** written in Rust. When triggered, it scans the focused X11 window properties (`WM_NAME` and `WM_CLASS`). If the application matches your user-defined target rules, it instantly switches i3 into an isolated layout mode, cleanly releasing all keyboard hooks to your terminal app. The moment focus shifts away, your global hotkeys snap right back.

## Features
- **Zero Background Latency:** Runs instantly on-demand and exits; no persistent daemon eating up memory.
- **X11 Class Aware:** Works perfectly with legacy terminal emulators like `xterm` and modern tools like WezTerm or Zellij.
- **Simple Configuration:** Control targets with a streamlined TOML list.

## Installation

```bash
# Clone the repository
git clone https://github.com
cd i3-passthrough

# Build the release binary
cargo build --release

# Move to your path
sudo cp target/release/i3-passthrough /usr/local/bin/
```

## Configuration

### 1. The i3 Window Manager Setup
Add an empty mode to your `~/.config/i3/config`. This acts as the container where i3 gracefully yields control of the keyboard:

```i3config
# ~/.config/i3/config

# Define the isolated input ring
mode "passthrough-active" {
    # Leave empty so all inputs pass straight to the focused window.
    # (Optional) You can add an emergency fallback exit key:
    # bindsym Mod4+Escape mode "default"
}

# Execute i3-passthrough automatically on every window focus change
# Alternatively, trigger this inside your custom window wrappers like i3-switch
bind_sym --whole-window focus exec_always --no-startup-id i3-passthrough
```

### 2. The Application Rules Setup
Create a simple configuration file at `~/.config/i3-passthrough/config.toml` listing the application terms or X11 classes you want to intercept:

```toml
targets = [
    "zellij",
    "wezterm",
    "xterm",
    "tmux-remote"
]
```

## How It Works Under the Hood

1. **IPC Query:** Connects cleanly to the synchronous i3 IPC socket.
2. **Node Traversal:** Iterates over the running layout tree to extract the active focused window wrapper.
3. **Evaluation:** Evaluates both window title text and underlying instance properties against your targets.
4. **State Transition:** Executes `mode "passthrough-active"` if matched, otherwise resets back to `mode "default"`.

## License
MIT
