# gtm-skills

Go-to-market skills for AI coding agents. Each skill runs one GTM process end to end, keeps its state in the product's own repo, and ships its work as pull requests or drafts for the owner to approve.

| Skill | What it does | Cadence |
| --- | --- | --- |
| [`blog-engine`](skills/blog-engine/SKILL.md) | Finds the buyer prompts where AI answers cite competitors and leave your product out, turns them into a weekly brief for approval, writes one approved post a day as a PR, and measures each post against the prompts it targeted. | Weekly brief, daily write |
| [`social-engine`](skills/social-engine/SKILL.md) | Turns real product material (posts, releases, quotes) into native posts for every connected network and queues them in PostSyncer, as a full week or as a set of posts for one day. | Weekly plan, optional daily top-ups |

Both skills share one voice file, `editorial/voice.md`, so the blog and social posts sound the same.

## Requirements

- An agent that supports Agent Skills (Claude Code, Codex, Cursor, OpenCode and others).
- Node.js 18 or newer, for the `skills` CLI.
- The skills run inside the repo of the product's website, with permission to create branches and open PRs.

Data sources and tools. Each skill checks what is connected at the start of a run, reports anything missing, and carries on without inventing numbers.

| Tool | Used by | For |
| --- | --- | --- |
| Sonable MCP | blog-engine | Running buyer prompts across AI engines: mentions, citations, competitor presence |
| DataForSEO MCP | blog-engine | Search volume, keyword difficulty, SERPs, People Also Ask, AI answer fallback |
| Google Search Console MCP | blog-engine | Impressions, clicks and position per query and page |
| PostSyncer MCP | social-engine | Connected accounts, creating draft or scheduled posts, post results |
| Web search and fetch | both | Reading cited sources, checking facts, news |

## Installation

```bash
# See which skills the repo contains
npx skills add oitozero/gtm-skills --list

# Install every skill into the current project
npx skills add oitozero/gtm-skills

# Install one skill
npx skills add oitozero/gtm-skills --skill blog-engine

# Install globally for Claude Code only, without prompts
npx skills add oitozero/gtm-skills --skill social-engine -g -a claude-code -y
```

Project installs (the default) live under the agent's skills folder in the repo, so they can be committed. Global installs (`-g`) are available in every project.

To update or check what is installed:

```bash
npx skills list
npx skills add oitozero/gtm-skills   # re-run to pull the latest version
```

Manual install also works: copy `skills/<name>/` into `.claude/skills/` in the project or `~/.claude/skills/` for your user.

## Setup

1. Install the skills in the website's repo.
2. Connect the MCP servers you have (Sonable, DataForSEO, Search Console, PostSyncer).
3. Run each skill once with someone present. The first run bootstraps `editorial/`:
   - `profile.md` with product facts, each with a source, and `TODO(owner)` where it could not find one.
   - `voice.md`, drafted from your own posts and existing blog posts, for you to confirm.
   - `prompts.yaml` and `weights.yaml` for blog-engine.
   - `social/config.md` for social-engine, where you set the PostSyncer workspace, networks, posting windows, pillars and approval mode.
4. Review and merge the bootstrap PR, filling the `TODO(owner)` items.
5. Schedule the runs (see below).

Resulting layout in the product's repo:

```
editorial/
  profile.md          product facts with sources
  voice.md            shared voice for blog and social
  prompts.yaml        buyer prompts and AI answer results
  candidates.jsonl    possible posts and their state
  briefs/2026-W48.md  weekly briefs
  ledger.jsonl        published posts and their measurements
  weights.yaml        scoring weights, tuned by results
  social/
    config.md
    material.jsonl
    plans/2026-W48.md
    ledger.jsonl
```

## Examples

### blog-engine

```text
/blog-engine brief
```
Refreshes the prompt set through Sonable, finds prompts where competitors are cited and you are not, scores them, and opens a PR with `editorial/briefs/2026-W48.md` holding seven picks. Merge it to approve; delete any pick you do not want.

```text
/blog-engine write
```
Takes the next approved pick, studies the pages AI engines cite for it, writes the post into the blog folder, runs the build, and opens a PR on `post/<slug>`. Then measures any posts that reached 7 or 28 days.

```text
/blog-engine
```
Runs `brief` if this week has no brief yet, otherwise `write`.

### social-engine

```text
/social-engine week
```
Reviews last week's results, plans a grid of posts across the connected networks for next Monday to Sunday, and creates them in PostSyncer as drafts. Shows a calendar with one line per post and any `{{owner: ...}}` placeholders to fill.

```text
/social-engine day 5
```
Queues five posts for today across the networks, inside their posting windows, using the freshest material.

With the approval mode set to `schedule` in `editorial/social/config.md`, `week` and `day` schedule the posts once you approve the plan in the conversation, instead of saving drafts.

```text
/social-engine review
```
Ranks recent posts against each account's median, writes the learnings into `editorial/voice.md` through a PR, and lists posts with unanswered questions.

## Scheduling

Any scheduler that can start your agent in the repo works. Suggested cadence:

| Run | When |
| --- | --- |
| `blog-engine brief` | Sunday morning |
| `blog-engine write` | Every weekday morning |
| `social-engine week` | Sunday afternoon, after the brief |
| `social-engine day 3` | Optional, weekday mornings |

Example with Claude Code headless mode and cron:

```bash
# m h dom mon dow
0 7 * * 0   cd ~/code/site && claude -p "/blog-engine brief"
0 7 * * 1-5 cd ~/code/site && claude -p "/blog-engine write"
0 15 * * 0  cd ~/code/site && claude -p "/social-engine week"
```

Unattended runs never publish. The blog opens PRs, and social posts are saved as PostSyncer drafts until you approve them.

## Guardrails

- Work is shipped as PRs or drafts. Nothing is pushed to the default branch or published without approval.
- No invented customers, quotes, statistics or results. Missing figures become placeholders for the owner.
- Product facts come from the code and live site, with sources, and are never changed by a post.
- A step that fails twice stops the run and reports what failed.

## Adding a skill

Create `skills/<name>/SKILL.md` with `name` and `description` frontmatter. Keep state in the product's repo, under `editorial/` for content skills or `gtm/` for strategy skills, and follow the same guardrails.
