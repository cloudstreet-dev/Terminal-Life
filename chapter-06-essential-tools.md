# Chapter 6: Essential CLI Tools and Utilities

## The Terminal Toolkit

You've got your shell configured. You've got tmux running. You can survive network outages with mosh. Now it's time to talk about the actual *tools* that make terminal life productive (and fun).

This chapter is organized into categories: the classics everyone should know, the modern replacements that are better, and the specialized tools that solve specific problems beautifully.

## The Classics (You Must Know These)

These are the foundational tools. They've been around for decades. They're on every Unix-like system. They're powerful, cryptic, and essential.

### grep — Finding Needles in Haystacks

**What it does:** Searches text for patterns.

```bash
# Find lines containing "error" in log file
grep "error" application.log

# Case-insensitive search
grep -i "error" application.log

# Show line numbers
grep -n "error" application.log

# Search recursively in directory
grep -r "TODO" src/

# Invert match (lines that DON'T contain pattern)
grep -v "debug" application.log

# Count matches
grep -c "error" application.log

# Context: show 3 lines before and after match
grep -C 3 "error" application.log
```

**Real use:**
```bash
# Find all API endpoints in code
grep -r "app.get\|app.post" src/

# Find config files
grep -r "database.*password" .
```

### sed — Stream Editor

**What it does:** Text transformation. Find and replace, line deletion, insertion.

```bash
# Replace first occurrence per line
sed 's/foo/bar/' file.txt

# Replace all occurrences (global)
sed 's/foo/bar/g' file.txt

# Replace in file (macOS requires empty string after -i)
sed -i '' 's/foo/bar/g' file.txt

# Delete lines matching pattern
sed '/debug/d' file.txt

# Print only lines matching pattern
sed -n '/error/p' file.txt

# Replace only on lines 5-10
sed '5,10s/foo/bar/g' file.txt
```

**Real use:**
```bash
# Change API version in all files
find . -name "*.js" -exec sed -i '' 's/api\/v1/api\/v2/g' {} \;

# Remove all debug statements
sed -i '' '/console.log/d' src/**/*.js
```

### awk — Pattern Scanning and Processing

**What it does:** Text processing powerhouse. Think of it as a tiny programming language for text.

```bash
# Print first column
awk '{print $1}' file.txt

# Print multiple columns
awk '{print $1, $3}' file.txt

# Sum a column
awk '{sum += $1} END {print sum}' numbers.txt

# Conditional processing
awk '$3 > 100 {print $1, $3}' data.txt

# Custom delimiter
awk -F',' '{print $2}' data.csv

# Built-in variables: NR (line number), NF (field count)
awk '{print NR, $0}' file.txt
```

**Real use:**
```bash
# Analyze log files
awk '$9 == 404 {print $7}' access.log | sort | uniq -c

# Process CSV
awk -F',' '$3 > 1000 {print $1}' sales.csv

# Disk usage summary
df -h | awk 'NR>1 {print $5, $6}' | sort -n
```

### find — Finding Files

**What it does:** Searches for files and directories.

```bash
# Find by name
find . -name "*.js"

# Find by type (f=file, d=directory)
find . -type f -name "*.log"

# Find modified in last 7 days
find . -mtime -7

# Find larger than 100MB
find . -size +100M

# Execute command on results
find . -name "*.tmp" -delete

# Complex: Find and execute
find . -name "*.js" -exec grep -l "TODO" {} \;
```

**Real use:**
```bash
# Find large files
find . -type f -size +50M -exec ls -lh {} \;

# Find empty directories
find . -type d -empty

# Find files modified today
find . -type f -mtime 0
```

### Others Worth Knowing

**curl** — Transfer data from/to servers
```bash
curl https://api.example.com/data
curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' https://api.example.com
```

**wget** — Download files
```bash
wget https://example.com/file.zip
wget -r -np https://example.com/files/  # Recursive download
```

**tar** — Archive files
```bash
tar -czf archive.tar.gz directory/  # Compress
tar -xzf archive.tar.gz              # Extract
```

**sort** — Sort lines
```bash
sort file.txt
sort -n numbers.txt  # Numeric sort
sort -r file.txt     # Reverse
```

**uniq** — Remove duplicates (requires sorted input)
```bash
sort file.txt | uniq
sort file.txt | uniq -c  # Count occurrences
```

**cut** — Extract columns
```bash
cut -d',' -f1,3 data.csv  # Fields 1 and 3, comma-delimited
```

**wc** — Word/line/character count
```bash
wc -l file.txt   # Line count
wc -w file.txt   # Word count
```

## The Modern Replacements (Better Versions)

The classics are great, but modern tools improve on them with better defaults, syntax highlighting, and speed.

### ripgrep (rg) — Better grep

**What it does:** Searches text, but faster and with better defaults.

Install:
```bash
brew install ripgrep
sudo apt install ripgrep
```

Usage:
```bash
# Basic search (automatically ignores .git, node_modules, etc.)
rg "error"

# Search specific file types
rg -t js "function"

# Case insensitive
rg -i "error"

# Show context
rg -C 3 "error"

# Search hidden files
rg --hidden "config"

# Ignore specific directories
rg "error" --glob '!test'
```

**Why it's better:**
- Faster than grep (sometimes 10x+)
- Respects .gitignore by default
- Syntax highlighting
- Better defaults (recursive, line numbers)
- Simpler syntax

### fd — Better find

**What it does:** Finds files, but more intuitive and faster.

Install:
```bash
brew install fd
sudo apt install fd-find  # Command is 'fdfind' on Ubuntu
```

Usage:
```bash
# Find files (much simpler than find)
fd pattern

# Find by extension
fd -e js

# Search hidden files
fd -H pattern

# Execute commands
fd -e txt -x cat

# Exclude patterns
fd pattern -E node_modules
```

**Why it's better:**
- Faster than find
- Simpler syntax
- Colored output
- Respects .gitignore
- Regex by default

### bat — Better cat

**What it does:** Shows file contents with syntax highlighting and line numbers.

Install:
```bash
brew install bat
sudo apt install bat  # Command is 'batcat' on Ubuntu
```

Usage:
```bash
# View file with syntax highlighting
bat file.js

# Show line numbers
bat -n file.js

# Show changes from git
bat --diff file.js

# Multiple files
bat file1.js file2.js
```

**Why it's better:**
- Syntax highlighting for 100+ languages
- Git integration
- Line numbers
- Paging built-in
- Theme support

Alias it to replace cat:
```bash
alias cat='bat'
```

### exa — Better ls

**What it does:** Lists files with colors and git status.

Install:
```bash
brew install exa
sudo apt install exa
```

Usage:
```bash
# Basic list
exa

# Long format with git status
exa -l --git

# Tree view
exa --tree

# Icons (requires Nerd Font)
exa --icons

# Group directories first
exa --group-directories-first
```

**Why it's better:**
- Colors by default
- Git status integration
- Tree view built-in
- Better formatting
- Shows file icons

Aliases:
```bash
alias ls='exa'
alias ll='exa -l --git'
alias la='exa -la --git'
alias lt='exa --tree'
```

### fzf — Fuzzy Finder

**What it does:** Interactive fuzzy finder for anything.

Install:
```bash
brew install fzf
# Run install script for shell integration
$(brew --prefix)/opt/fzf/install
```

Usage:
```bash
# Find files
fzf

# Find and open in editor
vim $(fzf)

# Search command history (Ctrl+R)
# (Automatically works after install)

# Find and cd into directory
cd $(fd -t d | fzf)

# Kill process by name
kill $(ps aux | fzf | awk '{print $2}')

# Git branch checkout
git checkout $(git branch | fzf)
```

**Why it's essential:**
- Interactive selection
- Fuzzy matching
- Preview windows
- Can pipe anything into it
- Replaces Ctrl+R for history search

Add to .zshrc for more power:
```bash
# Use fd instead of find
export FZF_DEFAULT_COMMAND='fd --type f --hidden --follow --exclude .git'

# Preview files with bat
export FZF_CTRL_T_OPTS="--preview 'bat --color=always {}'"
```

### htop — Better top

**What it does:** Process monitor with better UI.

Install:
```bash
brew install htop
sudo apt install htop
```

Usage:
```bash
htop
```

Then:
- Arrow keys to navigate
- F9 to kill process
- F6 to sort by column
- F5 for tree view
- / to search

**Why it's better:**
- Colored output
- Mouse support
- Tree view
- Easier to read
- Better sorting

### tldr — Better man pages

**What it does:** Simplified, practical man pages.

Install:
```bash
brew install tldr
npm install -g tldr
```

Usage:
```bash
tldr tar
tldr git
tldr ssh
```

**Why it's better:**
- Practical examples first
- No overwhelming documentation
- Community-maintained
- Gets you started fast

Compare:
```bash
man tar    # 10 pages of flags
tldr tar   # 5 practical examples
```

### jq — JSON Processor

**What it does:** Parse and manipulate JSON from command line.

Install:
```bash
brew install jq
sudo apt install jq
```

Usage:
```bash
# Pretty-print JSON
curl https://api.example.com/data | jq

# Extract field
jq '.name' data.json

# Filter array
jq '.[] | select(.age > 30)' users.json

# Map and transform
jq '[.[] | {name: .name, email: .email}]' users.json

# Count items
jq '. | length' data.json
```

**Real use:**
```bash
# Get all repo names from GitHub API
curl https://api.github.com/users/octocat/repos | jq '.[].name'

# Extract specific fields from AWS CLI output
aws ec2 describe-instances | jq '.Reservations[].Instances[] | {id: .InstanceId, state: .State.Name}'
```

### z (or zoxide) — Smart cd

**What it does:** Jump to frecent directories (frequent + recent).

Install:
```bash
brew install zoxide
# Add to .zshrc:
eval "$(zoxide init zsh)"
```

Usage:
```bash
# After visiting /Users/you/projects/my-app a few times:
z my-app
# Jumps there from anywhere

# Partial matches work
z proj
# Goes to most frecent match

# Interactive selection
zi proj
```

**Why it's essential:**
- No more `cd ../../../../../../projects/thing`
- Learns your habits
- Saves massive amounts of typing

## Specialized Tools

### tree — Directory Visualization

```bash
brew install tree

# Show directory structure
tree

# Limit depth
tree -L 2

# Show only directories
tree -d

# Show file sizes
tree -h
```

### watch — Repeat Commands

```bash
# Run command every 2 seconds
watch -n 2 'ls -l'

# Monitor disk usage
watch -n 5 'df -h'

# Highlight differences
watch -d 'netstat -an | grep ESTABLISHED | wc -l'
```

### ncdu — Disk Usage Analyzer

```bash
brew install ncdu
sudo apt install ncdu

# Analyze current directory
ncdu

# Analyze specific directory
ncdu /var/log
```

Navigate with arrows, `d` to delete, `q` to quit.

### httpie — Better curl for APIs

```bash
brew install httpie

# GET request (clean output)
http https://api.example.com/data

# POST with JSON
http POST https://api.example.com/users name=john email=john@example.com

# Custom headers
http GET https://api.example.com/data Authorization:"Bearer token"
```

Syntax is cleaner than curl, output is colored and formatted.

### entr — Run Commands on File Change

```bash
brew install entr

# Run tests when files change
ls src/**/*.js | entr npm test

# Restart server on change
ls *.go | entr -r go run main.go
```

Great for development workflows.

### tmuxinator — Tmux Session Manager

```bash
gem install tmuxinator

# Create project template
tmuxinator new myproject
```

Edit the YAML config to define windows, panes, and commands. Then:

```bash
tmuxinator start myproject
```

Instant workspace.

### ranger — Terminal File Manager

```bash
brew install ranger

ranger
```

Vim-like keybindings, preview pane, image preview support.

### ag (The Silver Searcher) — Another grep Alternative

```bash
brew install the_silver_searcher

ag "pattern"
```

Faster than grep, slower than ripgrep, but has been around longer.

## Pipes and Composition

The real power is chaining tools:

```bash
# Find large files and sort by size
find . -type f -size +10M -exec ls -lh {} \; | awk '{print $5, $9}' | sort -h

# Most common words in file
cat file.txt | tr ' ' '\n' | sort | uniq -c | sort -rn | head -10

# Monitor failed requests in real-time
tail -f access.log | grep '404' | awk '{print $7}' | sort | uniq -c

# Git commit stats
git log --pretty=format:'%an' | sort | uniq -c | sort -rn

# Kill all processes matching pattern
ps aux | grep 'node' | grep -v grep | awk '{print $2}' | xargs kill -9
```

This is the Unix philosophy in action. Simple tools, powerful combinations.

## Aliases and Functions (Build Your Own Tools)

Add these to `.zshrc`:

```bash
# Quick server
alias serve='python3 -m http.server'

# Weather
alias weather='curl wttr.in'

# External IP
alias myip='curl ifconfig.me'

# Git shortcuts
alias gs='git status'
alias gp='git pull'
alias gc='git commit -m'

# System monitoring
alias ports='netstat -tulanp'
alias mem='free -h'

# Directory navigation
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'

# Functions
# Create directory and cd into it
mkcd() {
    mkdir -p "$1" && cd "$1"
}

# Find process by name
psgrep() {
    ps aux | grep -v grep | grep -i -e VSZ -e "$1"
}

# Extract any archive
extract() {
    if [ -f "$1" ]; then
        case "$1" in
            *.tar.bz2)  tar xjf "$1"     ;;
            *.tar.gz)   tar xzf "$1"     ;;
            *.bz2)      bunzip2 "$1"     ;;
            *.rar)      unrar x "$1"     ;;
            *.gz)       gunzip "$1"      ;;
            *.tar)      tar xf "$1"      ;;
            *.tbz2)     tar xjf "$1"     ;;
            *.tgz)      tar xzf "$1"     ;;
            *.zip)      unzip "$1"       ;;
            *.Z)        uncompress "$1"  ;;
            *.7z)       7z x "$1"        ;;
            *)          echo "'$1' cannot be extracted via extract()" ;;
        esac
    else
        echo "'$1' is not a valid file"
    fi
}
```

## Dotfile Management

Keep your configs in version control:

```bash
# Create dotfiles repo
mkdir ~/dotfiles
cd ~/dotfiles

# Move configs
mv ~/.zshrc ~/dotfiles/zshrc
mv ~/.tmux.conf ~/dotfiles/tmux.conf

# Create symlinks
ln -s ~/dotfiles/zshrc ~/.zshrc
ln -s ~/dotfiles/tmux.conf ~/.tmux.conf

# Git it
git init
git add .
git commit -m "Initial dotfiles"
git remote add origin <your-repo>
git push
```

Now your configs are backed up and portable.

## Building Your Toolbox

Don't install everything at once. Start with:

1. **Week 1:** ripgrep, fd, bat, fzf
2. **Week 2:** exa, htop, tldr
3. **Week 3:** jq, zoxide, httpie
4. **Week 4:** Experiment with others

Learn each tool well. Add aliases. Make them yours.

## The Terminal Mindset

With these tools, the terminal becomes your IDE:

- `vim` + `fzf` + `ripgrep` = Code editor
- `tmux` + `htop` + `tail` = System monitor
- `curl` + `jq` + `httpie` = API explorer
- `git` + `diff` + `bat` = Version control UI

You're not limited by what some GUI decides to show you. You compose tools to solve problems.

## What We've Covered

You now know:
- The classic Unix tools everyone should know
- Modern replacements that improve the experience
- Specialized tools for specific tasks
- How to compose tools with pipes
- How to customize with aliases and functions

## The Journey Continues

This book covered the essentials:
- Shells and terminal emulators
- Oh My Zsh for beautiful, productive prompts
- Terminal multiplexing for power and persistence
- Mosh for resilient connections
- Essential tools for getting work done

But Terminal Life is a continuous journey. There are always new tools, new tricks, new workflows to discover.

The terminal is a craft. You'll spend years mastering it. You'll discover new tools monthly. You'll refine your dotfiles endlessly. You'll help newcomers and learn from veterans.

And you'll never go back to living without it.

## Final Thoughts

The terminal is powerful because it's:
- **Composable** — Chain simple tools into complex workflows
- **Efficient** — Keyboard-driven, no mouse required
- **Remote-friendly** — Works over SSH from anywhere
- **Scriptable** — Automate everything
- **Timeless** — Learn it once, use it for decades

But more than that, it's *fun*. There's joy in a perfectly crafted one-liner. There's satisfaction in a well-tuned `.zshrc`. There's pride in a smooth terminal workflow.

Welcome to Terminal Life.

You're home.

---

**Final Exercise:**

1. Install at least 5 modern tools from this chapter
2. Create 10 useful aliases
3. Write 3 custom functions
4. Set up dotfiles in version control
5. Solve a real problem with a pipe chain
6. Show someone your setup
7. Never look back

**Additional Resources:**

- [explainshell.com](https://explainshell.com) — Understand complex commands
- [tldr.sh](https://tldr.sh) — Quick command examples
- [Awesome Shell](https://github.com/alebcay/awesome-shell) — Curated list of tools
- Your `man` pages — They're not as scary as they look

**Remember:**

```bash
alias rtfm='man'
alias please='sudo'
alias oops='git reset --hard HEAD'
```

Now go forth and live Terminal Life.

```
$ _
```
