---
name: social-engine
description: 'Plans, writes and queues social content across networks in PostSyncer: a full week in one run, or a set of posts for one day. Use for "plan this week''s social", "queue today''s posts", or "social engine".'
---

# Social Engine

The engine takes what a product has actually done or learned and turns it into native posts for each connected network. It then queues those posts in PostSyncer. It never posts anything that isn't grounded in real material, and it never publishes without approval.

## Modes

- `week`: plans and queues seven days of content, starting from the next Monday or from a date you give it.
- `day [n]`: queues `n` posts for today or for a date you give it. The default is 3 posts.
- `review`: reads how recent posts performed and records what worked. A `week` run does this first automatically.

With no argument, the engine runs `week` on Sundays and Mondays and `day` on every other day.

## Voice (`editorial/voice.md`)

The voice lives in `editorial/voice.md` in the site repo. This is the same `editorial/` folder that blog-engine uses. Read the file before writing anything. It is the only source of truth for voice. Rules in the file take precedence over anything stored in PostSyncer.

### When `voice.md` is missing

Create it before planning anything.

1. **Gather evidence**, from strongest to weakest:
   - Writing samples the owner gives in the run.
   - The owner's own recent posts on the connected accounts, read through PostSyncer.
   - Existing blog posts in the repo.
   - `editorial/profile.md`, if it exists.
   - The PostSyncer brand from `get-brand`. Use it only as a starting point.
2. **Draft the file** using the structure below. Quote real lines from the evidence as examples. Mark anything you inferred rather than observed as `(inferred)`.
3. **Show it to the owner.** In an attended run, ask them to confirm or edit it. In an unattended run, commit it as a PR titled `Voice: first draft`, and queue the week's posts as drafts.

### File structure

- **Who is speaking.** Person or brand, point of view (I, we or you), and the relationship with the reader.
- **Traits.** Three to five traits, each with a short "this, not that" pair.
- **Rhythm.** Sentence length, paragraph length, and how a post opens and closes.
- **Words.** Words to use, words never to use, and banned phrases.
- **Formatting.** Emoji, hashtags, lists, line breaks, and calls to action.
- **Per-network notes.** How the voice shifts on each network, such as LinkedIn compared with X.
- **Rules.** What must never be said, and what must always be included.
- **Examples.** Three to six real posts that are on voice, and one or two that are off voice with the reason.
- **Learnings.** What has worked and what has missed. The `review` step updates this section.

### Updating the file

Only the Learnings section changes automatically, and only through `review`. Any other change, including an edit the owner made to a draft that suggests the voice is drifting, is proposed as a PR to `voice.md` and never applied silently.

## Other inputs

- **Raw material.** The engine never writes from nothing. It gathers material from `editorial/` (`profile.md`, and briefs and posts from blog-engine), from `gtm/` if that folder exists (`positioning.md`, `icp.md`), from merged PRs and changelog entries since the last run, and from customer quotes, metrics and notes the owner supplies. Every post traces back to one item.
- **Accounts.** Use `list-workspaces`, `list-accounts` and `get-account` to find the connected networks, their limits and any settings they need, such as Pinterest boards. Use `list-snippets` for saved sign-offs and hashtag groups.

Find the PostSyncer tool names at the start of each run with ToolSearch. If something is missing, say so once and carry on with what is available.

## State (`editorial/social/`)

- `config.md`:
  - the PostSyncer workspace id
  - the networks in use and how many posts each gets per day
  - posting windows per network, in the audience's time zone
  - content pillars and the share each should get
  - the approval mode, which is `draft` (the default) or `schedule`
- `material.jsonl`: raw items, each marked `fresh`, `used` or `retired`, with when it was used and which posts it fed.
- `plans/YYYY-Www.md`: the plan for each week.
- `ledger.jsonl`: one line per queued post, with the PostSyncer post id, network, slot, pillar, angle, material id, and the results gathered later.

## Planning a week

1. **Review first.** Pull last week's posts and their results from PostSyncer. Give each pillar and angle one of three verdicts: double down, keep, or drop. Judge results against each account's own median, not against absolute numbers.
2. **Fill the grid.** The grid is networks against days. Place slots in each network's posting windows and spread the pillars to their target shares. Never use the same pillar on two consecutive days on one network.
3. **Give each slot material.** Each slot gets a material item and an angle: lesson, proof, how-to, opinion, behind the scenes, question, or story. One strong item can feed several slots, but each network gets a different angle, never the same text reworded.
4. **Write natively.** Follow `voice.md` and the network rules below. Write each post from its angle as if that network were the only place it would appear.
5. **Check every post.**
   - The facts match the source item.
   - No figure appears that the owner hasn't given. Where a missing figure would help, use a `{{owner: ...}}` placeholder and keep that post as a draft in every mode.
   - The post follows the rules in `voice.md` and stays within the network's limits.
   - The first line, read on its own, makes a reader stop. If it doesn't, rewrite it.
   - Phrases the voice file bans, filler questions and engagement bait are cut.
   - The post passes the check in "Reading like a person wrote it" below.
6. **Queue.** Create each post with `create-post` on the right account, at its slot time, with the settings that network needs.
   - In `draft` mode, posts are saved as drafts.
   - In `schedule` mode, posts are scheduled, but only after the owner approves the plan in the conversation.
   - A scheduled run with nobody there to approve always creates drafts.
7. **Write the plan file.** Include each post's id or link. Give the owner a calendar view with one line per post, plus a list of placeholders they need to fill.

## Posts for one day

1. Fill `n` slots across networks within today's posting windows, taking the freshest material first.
2. Skip any slot that already has a post in PostSyncer.
3. Check and queue posts with the same rules as a week run.
4. If there isn't enough fresh material for `n` good posts, queue fewer and say why.

## Network rules

Confirm current limits with `get-account` or the platform's own documentation. These are starting points. Where `voice.md` has a note for a network, the note takes precedence.

- **LinkedIn.** Write in the first person about what happened and what changed. The first two lines have to carry the post. Keep paragraphs short. Put links in the body only if the voice file allows it, otherwise in the first comment. Use at most three hashtags. Tag people only with the tags `find-linkedin-mentions` returns.
- **X.** One sharp idea per post. Use a thread only when the material has three or more real steps. Keep within the account's character limit.
- **Threads and Bluesky.** Conversational and shorter than LinkedIn. Respect each network's limit.
- **Mastodon.** Plain, informative, and free of hype. Add a content warning where the instance's norms expect one.
- **Instagram, TikTok, YouTube and Pinterest.** These need images or video.
  - Use media from the PostSyncer library or files the owner supplies. Use `analyze-video` to understand a video before writing about it.
  - With no media, leave the network out of that slot and say so.
  - Open the caption with the hook.
- **Facebook and Google Business Profile.** Practical updates and offers with one clear call to action. A Google Business Profile post is about the business, never opinion.

## Reading like a person wrote it

People scroll past posts that read as machine-written. These checks come from Wikipedia's [Signs of AI writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing). Each pattern is a sign that a line says less than it seems to. Fix it by putting the specific detail back (the number, the name, what actually happened) or by cutting the line. Never just swap in a synonym.

- **Say what happened, not why it's significant.** Cut "a pivotal moment", "a testament to", "marks a shift", "in today's evolving landscape", and tails like ", highlighting…" or ", ensuring…".
- **No "It's not X, it's Y".** Also skip "not just X, but Y" and "Y rather than X" used for effect. Say Y.
- **No reflexive threes.** Give as many items as there really are.
- **Plain verbs.** Write "is" and "has", not "serves as", "stands as" or "boasts".
- **Watch for AI vocabulary in clusters.** Look for "delve", "crucial", "pivotal", "robust", "seamless", "leverage", "foster", "enhance", "showcase", "underscore", "game-changer" and "unlock". One is usually fine. Several in one post means rewrite it.
- **No vague attribution.** Don't write "experts say" or "studies show" unless you name and link them.
- **Format like a person.** No emoji used as bullets or headers, no bold label plus colon lists, few em dashes. Follow `voice.md` where it says otherwise.
- **No leftovers.** No chat residue ("Here's a post…", "I hope this helps"), no placeholder text other than `{{owner: ...}}`, and no tracking parameters added by AI tools on links (`utm_source=chatgpt.com`, `utm_source=openai`).

Don't use AI detectors to judge a post. They are often wrong in both directions. The test is whether every line makes a specific point that only this product or this owner could make.

## Review

1. For each post in the ledger that has been live for at least 72 hours, record its reach and engagement from PostSyncer, plus a reply count from `get-comment-summary`.
2. Rank the posts by their multiple of the account's median.
3. Turn the top three and bottom three into one-line learnings in the Learnings section of `voice.md`. Commit the change in a PR.
4. List posts that have unanswered questions for the owner. The engine doesn't reply to them.

## Guardrails

- Nothing is published or scheduled without approval in the conversation. Drafts are the default.
- The engine never replies, comments, sends DMs, follows anyone or tags anyone unless the owner asks.
- It never invents customers, quotes, numbers or results.
- It never deletes or edits a published post.
- It never adds or removes connected accounts, and never changes the PostSyncer brand.
- If a PostSyncer call fails twice, it stops and reports which posts were queued and which weren't.
