# Chapter 4: Screen and Tmux — Terminal Multiplexing

## The Problem

You're SSH'd into a server. You're running a long process. Your WiFi hiccups. Connection lost. Process killed. Dreams shattered.

Or: You need to monitor logs, run a development server, and edit code simultaneously. Do you open three terminal windows like an animal? Three tabs? Three monitors?

There's a better way.

## What Is Terminal Multiplexing?

A terminal multiplexer lets you:
- Run multiple terminal sessions in one window
- Split your terminal into panes
- Detach from sessions and reattach later
- Keep processes running even when you disconnect
- Feel like a hacker in a 90s movie

The two main players are **GNU Screen** (the elder) and **tmux** (the chosen one).

## GNU Screen: The Original Gangster

### History

Screen was created in 1987, which in computer years is approximately the Jurassic period. It's old. It's stable. It's everywhere.

If you SSH into a random Linux server, screen is probably installed. It's the reliable Honda Civic of terminal multiplexers—not fancy, but it gets the job done.

### Basic Screen Usage

Start screen:
```bash
screen
```

That's it. You're now in a screen session. It looks exactly the same, which is both boring and reassuring.

### Essential Screen Commands

Screen uses `Ctrl+A` as its prefix key. You press `Ctrl+A`, then another key to give commands.

**Create new window:**
```
Ctrl+A, C
```
(That's Ctrl+A, then release, then press C)

**Switch between windows:**
```
Ctrl+A, N  # Next window
Ctrl+A, P  # Previous window
Ctrl+A, 0-9  # Jump to window number
```

**List windows:**
```
Ctrl+A, "
```
You'll get a menu. Use arrows to select.

**Detach from session:**
```
Ctrl+A, D
```

This is the magic. Your session keeps running. Close your terminal, reboot your laptop, it doesn't matter. The session lives on the server.

**Reattach to session:**
```bash
screen -r
```

If you have multiple sessions:
```bash
screen -ls  # List sessions
screen -r [session-id]  # Attach to specific session
```

**Split horizontally:**
```
Ctrl+A, S
```

**Split vertically:**
```
Ctrl+A, |
```

**Switch between regions:**
```
Ctrl+A, Tab
```

**Kill current window:**
```
Ctrl+A, K
```
It'll ask for confirmation. Because screen respects you.

**Exit screen entirely:**
Just `exit` from all windows, or:
```
Ctrl+A, \
```

### Real-World Screen Example

You're deploying an app:

```bash
# Start screen
screen

# Window 0: Watch logs
tail -f /var/log/application.log

# Create new window (Ctrl+A, C)
# Window 1: Run the app
npm start

# Create new window (Ctrl+A, C)
# Window 2: Monitor resources
htop

# Detach (Ctrl+A, D)
```

Now close your laptop, go get coffee, come back, SSH in:

```bash
screen -r
```

Everything's still running. Your logs are still scrolling. The app didn't restart. You're a wizard.

### Screen Configuration

Create `~/.screenrc`:

```bash
# Turn off startup message
startup_message off

# Set scrollback buffer
defscrollback 10000

# Status line at bottom
hardstatus alwayslastline
hardstatus string '%{= kG}[ %{G}%H %{g}][%= %{= kw}%?%-Lw%?%{r}(%{W}%n*%f%t%?(%u)%?%{r})%{w}%?%+Lw%?%?%= %{g}][%{B} %m-%d %{W}%c %{g}]'

# Enable 256 colors
term screen-256color

# Use UTF-8
defutf8 on
```

Is that status line readable? No. Does it work? Yes. This is screen.

### Why You'd Use Screen Today

1. **It's already installed** — On almost every server
2. **It's simple** — Limited features = less to learn
3. **It's stable** — Decades of bug fixes
4. **It works over flaky connections** — Perfect for SSH

### Why You Wouldn't Use Screen Today

1. **Tmux exists** — Better in almost every way
2. **Configuration is painful** — That status line above? That's the "easy" version
3. **Fewer features** — No mouse support, limited customization
4. **The year is 2025** — We have better options now

## Tmux: The New King

### What Is Tmux?

**Terminal Multiplexer**

Born in 2007, tmux is screen's spiritual successor. It does everything screen does, but better. And it looks prettier while doing it.

### Installation

```bash
# macOS
brew install tmux

# Ubuntu/Debian
sudo apt install tmux

# Fedora
sudo dnf install tmux
```

### Basic Tmux Usage

Start tmux:
```bash
tmux
```

You'll see a green status bar at the bottom. Already better than screen.

### Essential Tmux Commands

Tmux uses `Ctrl+B` as its prefix key. (But everyone rebinds it to `Ctrl+A` because muscle memory.)

**Create new window:**
```
Ctrl+B, C
```

**Switch between windows:**
```
Ctrl+B, N  # Next
Ctrl+B, P  # Previous
Ctrl+B, 0-9  # Jump to number
Ctrl+B, W  # Visual menu
```

**Split panes horizontally:**
```
Ctrl+B, "
```
(The whole screen splits horizontally)

**Split panes vertically:**
```
Ctrl+B, %
```

**Navigate between panes:**
```
Ctrl+B, Arrow Keys
```
Or:
```
Ctrl+B, O  # Cycle through panes
```

**Resize panes:**
```
Ctrl+B, Ctrl+Arrow Keys
```
Hold Ctrl and press arrows. Feels great.

**Close pane:**
```
Ctrl+B, X
```
Or just `exit`.

**Detach from session:**
```
Ctrl+B, D
```

**Reattach to session:**
```bash
tmux attach
# Or shorthand:
tmux a
```

**List sessions:**
```bash
tmux ls
```

**Create named session:**
```bash
tmux new -s mysession
```

**Attach to named session:**
```bash
tmux a -t mysession
```

**Kill session:**
```bash
tmux kill-session -t mysession
```

### Tmux Windows vs Panes

**Windows** are like tabs. Full-screen views you switch between.

**Panes** are splits within a window. Multiple panes visible at once.

Typical workflow:
- Window 1: Code editor (single pane)
- Window 2: Dev server + logs (two panes, split)
- Window 3: Database client (single pane)
- Window 4: htop (single pane)

Switch between windows, split windows into panes as needed.

### Copy Mode

Want to scroll back? Enter copy mode:
```
Ctrl+B, [
```

Now use:
- Arrow keys to move
- PgUp/PgDn to scroll
- `/` to search
- `q` to quit

Copy text (Vi mode):
- `Space` to start selection
- `Enter` to copy
- `Ctrl+B, ]` to paste

### The .tmux.conf File

Tmux configuration lives in `~/.tmux.conf`. Here's a good starter config:

```bash
# Rebind prefix to Ctrl+A (because screen muscle memory)
unbind C-b
set-option -g prefix C-a
bind-key C-a send-prefix

# Split panes using | and -
bind | split-window -h
bind - split-window -v
unbind '"'
unbind %

# Switch panes using Alt+Arrow without prefix
bind -n M-Left select-pane -L
bind -n M-Right select-pane -R
bind -n M-Up select-pane -U
bind -n M-Down select-pane -D

# Enable mouse mode
set -g mouse on

# Don't rename windows automatically
set-option -g allow-rename off

# Start window numbering at 1
set -g base-index 1

# Faster command sequences
set -s escape-time 0

# Increase scrollback buffer
set -g history-limit 10000

# Enable 256 colors
set -g default-terminal "screen-256color"

# Status bar styling
set -g status-bg black
set -g status-fg white
set -g status-left '#[fg=green]#S '
set -g status-right '#[fg=yellow]#H #[fg=cyan]%Y-%m-%d %H:%M'

# Reload config
bind r source-file ~/.tmux.conf \; display "Config reloaded!"
```

After creating this, reload:
```bash
tmux source ~/.tmux.conf
```

Or from inside tmux:
```
Ctrl+B, :
source ~/.tmux.conf
```

### Tmux Plugins

Yes, tmux has plugins. Because of course it does.

**TPM (Tmux Plugin Manager)**

Install:
```bash
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

Add to `.tmux.conf`:
```bash
# List of plugins
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-continuum'

# Initialize TPM (keep this line at the very bottom)
run '~/.tmux/plugins/tpm/tpm'
```

Install plugins: `Ctrl+B, I`

**Useful Plugins:**

**tmux-sensible** — Sensible default settings

**tmux-resurrect** — Save and restore tmux sessions
- Save: `Ctrl+B, Ctrl+S`
- Restore: `Ctrl+B, Ctrl+R`

**tmux-continuum** — Auto-save sessions every 15 minutes

**tmux-yank** — Better copy/paste integration with system clipboard

### Real-World Tmux Example

Development workflow:

```bash
# Create named session
tmux new -s myproject

# Window 0: Editor
nvim

# New window (Ctrl+B, C)
# Window 1: Split for server and logs
# Split vertically (Ctrl+B, %)
npm run dev
# Switch pane (Ctrl+B, O)
tail -f logs/app.log

# New window (Ctrl+B, C)
# Window 2: Git
git status

# Detach (Ctrl+B, D)
```

Come back later:
```bash
tmux a -t myproject
```

Everything exactly as you left it.

### Tmux and SSH

The killer combo: SSH + tmux on server.

```bash
# SSH to server
ssh myserver

# Start or attach to tmux session
tmux a || tmux

# Do work in tmux
```

Benefits:
- Connection drops? Session persists.
- Need to close laptop? Detach and go.
- Multiple terminals? One SSH connection, multiple tmux windows.
- Collaborating? Both attach to same session. Pair programming via terminal.

### Tmux Power User Tips

**Zoom a pane:**
```
Ctrl+B, Z
```
Makes one pane full-screen. Press again to unzoom.

**Synchronize panes:**
```
Ctrl+B, :
setw synchronize-panes on
```
Now typing in one pane types in all. Great for running commands on multiple servers. Turn off with `off`.

**Swap windows:**
```
Ctrl+B, :
swap-window -s 2 -t 1
```

**Clock:**
```
Ctrl+B, T
```
Displays a clock. Why? Because tmux.

**Command prompt:**
```
Ctrl+B, :
```
Runs tmux commands directly. Power user territory.

### Tmux Scripts

Automate your layouts with scripts:

```bash
#!/bin/bash
# start-dev.sh

SESSION="myproject"

# Create session
tmux new-session -d -s $SESSION

# Window 0: Editor
tmux rename-window -t $SESSION:0 'editor'
tmux send-keys -t $SESSION:0 'nvim' C-m

# Window 1: Server
tmux new-window -t $SESSION:1 -n 'server'
tmux send-keys -t $SESSION:1 'npm run dev' C-m

# Window 2: Logs and git
tmux new-window -t $SESSION:2 -n 'logs'
tmux split-window -h -t $SESSION:2
tmux send-keys -t $SESSION:2.0 'tail -f logs/app.log' C-m
tmux send-keys -t $SESSION:2.1 'git status' C-m

# Attach to session
tmux attach-t $SESSION
```

Run it:
```bash
chmod +x start-dev.sh
./start-dev.sh
```

Instant perfect workspace.

## Screen vs Tmux: The Verdict

| Feature | Screen | Tmux |
|---------|--------|------|
| Age | Ancient | Modern |
| Availability | Everywhere | Almost everywhere |
| Configuration | Painful | Reasonable |
| Defaults | Bad | Good |
| Mouse support | No | Yes |
| Pane management | Basic | Excellent |
| Plugins | No | Yes |
| Status bar | Ugly | Pretty |
| Community | Stable | Active |

**Use Screen if:**
- It's already there and you need something quick
- You're on a minimal system
- You learned it first and don't want to change

**Use Tmux if:**
- You have a choice
- You want modern features
- You like pretty things
- You value your sanity

## Common Mistakes

**Mistake 1: Not using tmux/screen for remote work**
If you SSH without a multiplexer, you're living dangerously.

**Mistake 2: Running tmux inside tmux**
Don't nest multiplexers. Prefix keys get confusing. Use one session with multiple windows.

**Mistake 3: Not naming sessions**
`tmux new -s descriptive-name` is better than `tmux` and then forgetting what's in each session.

**Mistake 4: Not learning detach/attach**
The whole point is persistence. Use it.

**Mistake 5: Ignoring the config file**
Default bindings are awkward. Customize them.

## The Philosophy of Multiplexing

Terminal multiplexing embodies key Terminal Life principles:

1. **Persistence** — Your work survives disconnections
2. **Organization** — Multiple tasks, one window
3. **Efficiency** — Keyboard-driven pane management
4. **Resilience** — Flaky connections can't stop you
5. **Power** — Complex workflows, simply managed

## What's Next

Now you can run multiple terminals, split panes, and survive disconnections. You're becoming powerful.

Next: We'll talk about mosh, which makes SSH even better for mobile connections.

---

**Exercise for the Reader:**

1. Install tmux (if not already installed)
2. Create a `.tmux.conf` with at least 5 customizations
3. Start a tmux session with:
   - 3 windows
   - At least one window with split panes
4. Detach and reattach
5. Kill the session
6. Feel the power
7. Never go back to plain SSH again

Welcome to terminal persistence.
