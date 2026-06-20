# FFUF Web Fuzzing

Expert guidance for [ffuf](https://github.com/ffuf/ffuf) (Fuzz Faster U Fool) web fuzzing during authorized penetration testing — authenticated fuzzing with raw requests, auto-calibration, and result analysis.

> Vendored from [jthack/ffuf_claude_skill](https://github.com/jthack/ffuf_claude_skill) (MIT). Credit to [@jthack](https://github.com/jthack). Bundled here so the whole team gets it via the same install flow as the other skills.

## Usage

```
/ffuf-web-fuzzing
```

Or just describe the goal — e.g. "fuzz the /api endpoint on example.com for hidden paths", "enumerate subdomains for target.com".

## Prerequisites

`ffuf` must be installed:

```bash
# macOS
brew install ffuf

# Linux (Go)
go install github.com/ffuf/ffuf/v2@latest
```

## How It Works

- Configures ffuf appropriately for the stated testing goal
- Supports authenticated fuzzing via raw request templates (`resources/REQUEST_TEMPLATES.md`)
- Auto-calibration + result filtering/analysis
- Helps select wordlists (`resources/WORDLISTS.md`)
- Ships a `ffuf_helper.py` for output handling

## Safety & Ethics

For **authorized** security testing only — test only systems you own or have explicit permission to test, respect rate limits, and follow responsible disclosure. Unauthorized testing is illegal.

## Install

```bash
cp -R ffuf-skill ~/.claude/skills/
```
