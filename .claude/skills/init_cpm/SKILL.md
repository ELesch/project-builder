---
name: init_cpm
description: Initialize Claude Project Manager - verify and install dependencies
disable-model-invocation: true
allowed-tools: Bash, Read
---

# Initialize Claude Project Manager

Verify that all required dependencies are installed and offer to install any missing ones.

## Process

### Step 1: Check Required Dependencies

Run these checks and report status:

```bash
# Node.js (required)
node --version

# Git (required)
git --version

# jq (optional but recommended)
jq --version

# GitHub CLI (optional)
gh --version
```

### Step 2: Report Status

Create a status table:

| Dependency | Required | Status | Version |
|------------|----------|--------|---------|
| Node.js | Yes | ✓ / ✗ | x.x.x |
| Git | Yes | ✓ / ✗ | x.x.x |
| jq | No | ✓ / ✗ | x.x.x |
| GitHub CLI | No | ✓ / ✗ | x.x.x |

### Step 3: Offer to Install Missing Dependencies

For each missing **required** dependency, offer to install:

**Windows (winget):**
```bash
winget install OpenJS.NodeJS.LTS  # Node.js
winget install Git.Git            # Git
winget install jqlang.jq          # jq
winget install GitHub.cli         # GitHub CLI
```

**macOS (brew):**
```bash
brew install node    # Node.js
brew install git     # Git
brew install jq      # jq
brew install gh      # GitHub CLI
```

**Cross-platform (npm):**
```bash
npm install -g vercel  # Vercel CLI (optional)
```

### Step 4: Verify Hooks Work

After dependencies are confirmed, test that the audit hook can parse JSON:

```bash
echo '{"hook_event_name":"test","session_id":"123"}' | node -e "
  let d='';
  process.stdin.on('data',c=>d+=c);
  process.stdin.on('end',()=>{
    try{console.log(JSON.parse(d).hook_event_name)}
    catch(e){console.log('PARSE_ERROR')}
  })
"
```

Expected output: `test`

### Step 5: Report Summary

```
## CPM Initialization Complete

✓ All required dependencies installed
✓ Hook JSON parsing verified
✓ Ready to create projects

Run `/help` to see available commands.
```

## If Dependencies Cannot Be Installed

If the user cannot install dependencies (permissions, corporate policy, etc.):

1. Document which features will be limited
2. Hooks will fall back to basic grep parsing (less reliable)
3. Some GitHub features may not work without `gh`

## Reference

See `DEPENDENCIES.md` for full dependency documentation.
