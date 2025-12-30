# Claude Code Review Action

A multi-agent Claude Code review system that posts inline comments on pull requests.

## Features

- **Multi-agent review pipeline**: 4 parallel agents (2 Sonnet for CLAUDE.md compliance, 2 Opus for bug detection)
- **Inline comments**: Posts comments directly on specific PR lines with suggested fixes
- **High-signal only**: Validation agents confirm issues before posting to reduce false positives
- **CLAUDE.md compliance**: Checks code against your project's coding standards
- **Duplicate prevention**: Won't re-review PRs that have already been reviewed

## How it works

1. **Pre-check** (Haiku): Verifies PR needs review (not closed/draft/already reviewed)
2. **CLAUDE.md discovery** (Haiku): Finds relevant coding standards files
3. **PR summary** (Sonnet): Summarizes the changes
4. **Parallel review** (4 agents):
   - 2 Sonnet agents check CLAUDE.md compliance
   - 2 Opus agents scan for bugs and security issues
5. **Validation**: Secondary agents confirm high-signal issues
6. **Post comments**: Inline comments for issues, or summary if none found

## Setup

### 1. Create a GitHub App

1. Go to **Settings > Developer settings > GitHub Apps > New GitHub App**
2. Configure:
   - **Name**: `your-org-claude-code-review` (must be unique)
   - **Homepage URL**: Your repo URL
   - **Webhook**: Uncheck "Active" (not needed)
   - **Permissions**:
     - Contents: Read & write
     - Issues: Read & write
     - Pull requests: Read & write
   - **Where can this app be installed?**: Only on this account
3. Create the app
4. Note the **App ID**
5. Generate a **Private Key** and download it

### 2. Install the GitHub App

1. Go to your app's settings
2. Click **Install App**
3. Select your repository

### 3. Add secrets to your repository

Go to **Settings > Secrets and variables > Actions** and add:

| Secret | Value |
|--------|-------|
| `ANTHROPIC_API_KEY` | Your Anthropic API key |
| `CLAUDE_APP_ID` | The App ID from step 1 |
| `CLAUDE_APP_PRIVATE_KEY` | The private key file contents |

### 4. Add the workflow

Copy `.github/workflows/claude-code-review.yml` to your repository.

### 5. (Optional) Add CLAUDE.md

Create a `CLAUDE.md` file in your repo root with your coding standards:

```markdown
# CLAUDE.md

## Code Style

- Use TypeScript strict mode
- Prefer `const` over `let`
- Use async/await over callbacks

## Architecture

- All API endpoints go in `src/api/`
- Business logic goes in `src/services/`
```

## Customization

### Change the bot name check

Edit the workflow to check for your specific bot name:

```yaml
- Claude has already commented on this PR (check for comments from 'your-bot-name[bot]')
```

### Filter by file paths

Uncomment and modify the `paths` section:

```yaml
on:
  pull_request:
    paths:
      - "src/**/*.ts"
      - "src/**/*.tsx"
```

### Use different models

Modify the agent prompts to use different models:

```
model: haiku   # Fast, cheap - good for simple checks
model: sonnet  # Balanced - good for compliance checks
model: opus    # Most capable - good for bug detection
```

## Cost

Approximate cost per review (varies by PR size):
- Small PR (~100 lines): ~$0.05-0.10
- Medium PR (~500 lines): ~$0.10-0.20
- Large PR (~1000+ lines): ~$0.20-0.50

## Troubleshooting

### "MCP server failed to connect"

Ensure the workflow has proper permissions and the GitHub App is installed correctly.

### "Unknown skill: code-review"

This workflow doesn't use the skill system - it embeds the prompt directly. Ignore this error.

### No comments posted

Check the workflow logs for the Claude output. Common issues:
- PR was already reviewed (check for existing bot comments)
- PR is a draft or closed
- No issues were found (check for summary comment)

## Credits

Based on Anthropic's [code-review plugin](https://github.com/anthropics/claude-code/tree/main/plugins/code-review).

## License

MIT
