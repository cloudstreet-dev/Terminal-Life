# Chapter 5: Mosh — The Mobile Shell

## The Problem with SSH

Let's set the scene: You're on a train, laptop open, SSH'd into a server. You're in the zone. Code is flowing. Life is good.

Then you enter a tunnel.

Connection lost. Your SSH session hangs. You wait. The loading spinner spins. You wonder if you should `Ctrl+C` or wait longer. You wait. Still nothing. You `Ctrl+C`. Session dead. Your running command? Gone. Your perfectly crafted vim session? Lost. Your will to live? Questionable.

Or maybe you're at a coffee shop. You walk outside for a phone call. Your laptop switches from WiFi to mobile hotspot. SSH: "LOL nope, new connection required."

Or maybe you just close your laptop to move to another room. SSH: "I'm very sensitive about these things."

SSH is amazing, but it has one fatal flaw: it's fragile.

## Enter Mosh

**Mobile Shell (mosh)** is SSH's cooler, more resilient cousin. It's designed for:
- Mobile connections
- WiFi that keeps dropping
- Switching networks
- High-latency connections
- Working from literally anywhere

### What Makes Mosh Different?

**SSH:**
- Connection-based
- Lose connection = lose session
- Lag is painful (every keystroke waits for server response)
- Switching networks kills connection

**Mosh:**
- Uses UDP (not TCP like SSH)
- Survives disconnections
- Resumes when connection returns
- Instant local echo (types appear immediately)
- Survives IP address changes
- Survives laptop sleep

It's like SSH but with the resilience of a Nokia 3310.

## How Mosh Works

Technical bits (brief, I promise):

1. Mosh uses SSH to authenticate and start initial connection
2. Then switches to UDP for communication
3. Uses "State Synchronization Protocol" (SSP) to keep client and server in sync
4. Client predicts what you're typing and shows it immediately
5. Server confirms later (or corrects if wrong)
6. If connection drops, both sides wait patiently for reconnection

Result: Feels instantaneous even over terrible connections.

## Installation

### Client (Your Machine)

```bash
# macOS
brew install mosh

# Ubuntu/Debian
sudo apt install mosh

# Fedora
sudo dnf install mosh

# Arch
sudo pacman -S mosh
```

### Server (Remote Machine)

Same commands on the server. Both sides need mosh installed.

```bash
# On the server
sudo apt install mosh  # or appropriate command for your distro
```

### Firewall Configuration

Mosh uses UDP ports 60000-61000. Your server's firewall needs to allow these:

```bash
# UFW (Ubuntu)
sudo ufw allow 60000:61000/udp

# firewalld (Fedora/CentOS)
sudo firewall-cmd --permanent --add-port=60000-61000/udp
sudo firewall-cmd --reload

# iptables
sudo iptables -A INPUT -p udp --dport 60000:61000 -j ACCEPT
```

Cloud providers (AWS, GCP, Azure): Update security groups to allow UDP 60000-61000.

## Basic Usage

Instead of:
```bash
ssh user@server
```

Use:
```bash
mosh user@server
```

That's it. Seriously.

Everything else is the same. You authenticate with SSH keys (or password). Then mosh takes over.

## Advanced Usage

### Specify SSH Port

```bash
mosh --ssh="ssh -p 2222" user@server
```

### Use Specific UDP Port Range

```bash
mosh -p 60001 user@server
```

### Predict Less Aggressively

Mosh predicts your typing. Usually great. Sometimes annoying (if you have high latency to server).

```bash
mosh --predict=experimental user@server
```

Options:
- `always` — Always predict (default for low latency)
- `never` — Never predict (like SSH)
- `adaptive` — Adapt based on network (default)
- `experimental` — New prediction modes

### With Tmux (The Ultimate Combo)

```bash
mosh user@server -- tmux attach
```

Now you have:
- Mosh's connection resilience
- Tmux's session persistence
- Unstoppable remote work setup

## Real-World Scenarios

### The Commuter

You're on a train, working on a server:

```bash
mosh myserver
tmux attach
# Work work work
# Enter tunnel
# Connection lost... but your session continues
# Exit tunnel
# Connection restored automatically
# No interruption to work
```

Your typing appeared instantly the whole time. When connection returned, mosh synced with server.

### The Coffee Shop Hopper

```bash
mosh myserver
# Working at coffee shop A
# Close laptop
# Walk to coffee shop B
# Open laptop
# Connect to new WiFi
# Mosh reconnects automatically
# Still in same session
```

Your IP address changed. SSH would've died. Mosh doesn't care.

### The International Traveler

You're connecting from Asia to a US server. 300ms latency.

**With SSH:**
- Type 'ls'
- Wait 300ms to see 'l'
- Wait 300ms to see 's'
- Painful

**With Mosh:**
- Type 'ls'
- See 'ls' immediately (predicted)
- Server confirms 300ms later
- Smooth

### The Flaky Internet User

Your connection drops every few minutes:

**With SSH:**
- Reconnect
- Restart tmux
- Find your place
- Resume
- Repeat every 5 minutes
- Lose sanity

**With Mosh:**
- Keep working
- Mosh handles reconnection
- No interruption
- Keep sanity

## Mosh Limitations

Nothing's perfect:

### No Port Forwarding

SSH can forward ports (`ssh -L`). Mosh can't.

Workaround: Use SSH for port forwarding, mosh for interactive sessions.

### No X11 Forwarding

SSH can forward X11 graphics. Mosh can't.

If you need X11, use SSH. (But also, why are you using X11 forwarding?)

### UDP Required

Some restrictive networks block UDP. Some block non-standard ports.

If UDP 60000-61000 is blocked, mosh won't work. Fall back to SSH.

### Scrollback Complications

Mosh doesn't handle scrollback the same as SSH. Use tmux or screen for scrollback. (You should be using tmux anyway.)

### Not a Drop-In Replacement

Mosh is for interactive shell sessions. For:
- SCP/SFTP — Use SSH
- Rsync — Use SSH
- Git over SSH — Use SSH
- Port forwarding — Use SSH
- Automation/scripts — Use SSH

Mosh is for humans typing commands, not for automation.

## The Technical Details (For Nerds)

Mosh's State Synchronization Protocol (SSP):

1. **Datagram-based** — UDP means no connection state
2. **Encrypted** — Uses AES-128-OCB
3. **Authenticated** — Can't be MITM'd
4. **Ordered but not reliable** — Drops old packets, only latest state matters
5. **Differential** — Only sends what changed

The client runs a prediction engine:
- Guesses what server will do with your input
- Shows prediction with underline (you often don't notice)
- Updates when server confirms
- Occasionally wrong (rare), then corrects

## Mosh and SSH Keys

Mosh uses SSH for authentication. Set up SSH keys first:

```bash
# Generate key (if you don't have one)
ssh-keygen -t ed25519

# Copy to server
ssh-copy-id user@server

# Test
ssh user@server
```

Once SSH works with keys, mosh will too.

## Configuration

Mosh has minimal configuration. Most settings are SSH settings.

`~/.ssh/config`:
```
Host myserver
    HostName server.example.com
    User myuser
    Port 22
    IdentityFile ~/.ssh/id_ed25519
```

Then:
```bash
mosh myserver
```

Mosh reads your SSH config.

## Troubleshooting

### "Command not found: mosh"

Install mosh on both client and server.

### "Connection timed out"

Firewall blocking UDP 60000-61000. Fix firewall rules.

### "Nothing received from server on UDP"

Server didn't start mosh. Check:
- Is mosh installed on server?
- Is PATH correct? (`which mosh-server` on server)
- Firewall rules?

### "Mosh didn't work"

Check with verbose output:
```bash
mosh --ssh="ssh -v" user@server
```

### Still doesn't work?

Fall back to SSH. Mosh is great but optional.

## Mosh Best Practices

1. **Always use with tmux** — Mosh + tmux = immortal sessions

2. **Set up SSH keys** — Typing passwords over mosh is annoying

3. **Configure SSH config** — Use `~/.ssh/config` for server shortcuts

4. **Use for interactive work only** — Automation should still use SSH

5. **Test your firewall rules** — Before you rely on mosh

6. **Keep mosh updated** — Both sides should run similar versions

## The Mobile Workflow

My actual workflow:

### Daily Work

```bash
# Morning: At desk
mosh workserver -- tmux attach

# Afternoon: Coffee shop
# Laptop connects to new WiFi
# Mosh automatically reconnects
# Still in same tmux session

# Evening: At home
# Same story
```

One session, all day, multiple locations. No reconnecting. No lost state.

### Travel

```bash
# Before flight: At airport
mosh myserver
tmux attach
# Start long-running job

# During flight: Close laptop
# Mosh disconnects gracefully

# After flight: New country, new network
# Open laptop
# Mosh reconnects
# Long-running job still running
```

### Questionable Internet

Café WiFi cutting out every 2 minutes?

```bash
mosh myserver
tmux attach
# Work normally
# Mosh handles the chaos
```

## Why Mosh Isn't More Popular

Good question! Probably because:

1. **SSH "works"** — Good enough for most
2. **Extra setup** — Firewall rules, installation on both sides
3. **Limitations** — No port forwarding, no file transfer
4. **Niche use case** — Most people work from one location
5. **Hasn't hit critical mass** — Not default anywhere

But for those who need it, it's life-changing.

## The Philosophy of Mosh

Mosh represents a key Terminal Life insight: **Don't let the network control you.**

Network is flaky? Work anyway.
Switching locations? Work anyway.
High latency? Work anyway.

Mosh gives you independence from network quality. And in a world of coffee shop WiFi and mobile hotspots, that's power.

## Mosh and the Future

Some cool things people are doing:

**Eternal Terminal** — Similar to mosh but uses TCP. Different tradeoffs.

**EtherTerm** — SSH/Telnet client with built-in resilience.

**Eternal Terminal** — Yes, I listed this twice. It's that interesting. Look it up.

But mosh remains the standard for "SSH but resilient."

## When Not to Use Mosh

- When you need port forwarding
- When you need file transfer
- When UDP is blocked
- When you're writing automation scripts
- When SSH is working fine and you're not mobile

Don't force mosh into every situation. It's a specialized tool.

## What's Next

You now have:
- A beautiful shell (zsh + Oh My Zsh)
- Terminal persistence (tmux)
- Connection resilience (mosh)

You're basically unstoppable.

Next: Essential CLI tools that make terminal life even better.

---

**Exercise for the Reader:**

1. Install mosh on client and a server you access
2. Configure firewall for UDP 60000-61000
3. Connect with `mosh server`
4. Start tmux session
5. Close laptop or switch networks
6. Resume and marvel at the magic
7. Try typing during a network interruption
8. Watch mosh handle it gracefully
9. Never want to use plain SSH again

Welcome to mobile terminal life.
