---
name: blog-engine
description: 'Runs a product blog built to win AI answers: weekly it builds a brief from AI answer gaps for approval, daily it writes one approved post as a PR and measures earlier posts. Use for "blog engine", "weekly brief", or "write today''s post".'
---

# Blog Engine

Every post is a bet on specific AI prompts. Each one exists to change what ChatGPT, Perplexity, Gemini and Google AI Overviews say when a buyer asks a particular question. Search traffic matters, but it comes second.

The engine runs a three-part loop:

1. **Brief** (weekly): find the prompts where AI answers cite competitors and leave the product out, then turn the best of them into a weekly brief that the owner approves.
2. **Write** (daily): write one post from the approved brief and open a PR.
3. **Measure** (daily, after writing): check earlier posts against the prompts they targeted, and feed the results back into how topics are chosen.

Call the skill with `brief`, `write`, or no argument. With no argument it runs `brief` if this ISO week has no brief yet, and `write` otherwise.

## Where things live

Everything sits in `editorial/` in the site repo. The social-engine skill uses the same folder.

- `profile.md`: what the engine knows about the product. This covers the one-line pitch, the buyer, the buyer's jobs to be done, the competitors, the locale, the blog content path and frontmatter format, the DataForSEO location and language codes, and the ship target (always a PR). Every product fact carries a source: a code path or a live URL.
- `voice.md`: the voice, shared with social-engine. It is the only source of truth for how posts sound. It covers who is speaking, traits, rhythm, words to use and avoid, formatting, notes for each network, rules, examples and learnings. Read it before writing. The engine changes it only through a PR.
- `prompts.yaml`: the prompt set. These are buyer questions grouped into clusters by job to be done. Each prompt has a last-checked date and, for each AI engine, whether the product is mentioned, whether it is cited, and which competitors and URLs are cited.
- `candidates.jsonl`: one line for each possible post. Each line holds the target prompts, the primary search query, the gap score, the volume, the difficulty, the post type (`answer`, `comparison`, `how-to`, `explainer`, `news`) and the state (`candidate`, `briefed`, `approved`, `written`, `rejected`).
- `briefs/YYYY-Www.md`: the weekly brief.
- `ledger.jsonl`: one line per post. Each line holds the slug, URL and PR, the target prompts and search query, the date it went live, and the measurements taken at 7 and 28 days.
- `weights.yaml`: the scoring weights, which the measure step adjusts.

If `editorial/profile.md` or `editorial/voice.md` does not exist, bootstrap first (see the end of this file).

## Tools

At the start of each run, check which of these tools are available. List any that are missing at the top of the run's PR body, and never fill a gap with invented numbers.

- **Sonable MCP** is the main signal. It runs the prompt set across AI engines and returns mentions, citations and whether competitors appear. Look up its tool names when the run starts.
- **DataForSEO MCP** gives search volume, keyword difficulty, live SERPs and People Also Ask. Its AI Optimization endpoints are the fallback when Sonable is unavailable.
  - Confirm every endpoint and payload with `docs_search` before calling `api_request`.
  - Batch requests, and don't query the same keyword again within 30 days. Cache results in `candidates.jsonl`.
- **Google Search Console**, if connected, gives impressions, clicks and position for each target query and page.
- **WebSearch and WebFetch** are for reading the sources AI engines cite, checking facts, and finding news.

## Brief (weekly)

1. **Refresh the prompt set.** Run every prompt in `prompts.yaml` through Sonable and save the results.
   - Add new prompts you come across: follow-up questions the engines suggest, People Also Ask entries for the cluster's queries, and Search Console queries phrased as questions.
   - Keep the set at 40 prompts or fewer. Retire any prompt with no buyer intent.
2. **Find the gaps.** A prompt is a gap when at least one competitor is mentioned or cited and the product is neither. Rank a gap higher when:
   - several engines agree on it;
   - the cited sources are weak or out of date (open them and check);
   - the product can genuinely answer the question.
3. **Group gaps into post candidates.** Put together gaps that one page could win. Give each candidate a primary search query that covers the same need, and add its volume and difficulty from DataForSEO.
4. **Score the candidates.** Score is `gap score × type weight × product fit`, using `weights.yaml`. Volume only breaks ties. Drop any candidate where the product would have to be forced in.
5. **Write the brief.** Pick the top seven candidates. Include a refresh if the measure step flagged one, and at most one news post, and only if it's worth one. For each pick, give:
   - the working title and the target prompts;
   - what the engines say now, and who they cite;
   - the angle that makes this post the better source;
   - the post type and the search query;
   - one line on why it was picked.

   End the brief with anything the owner needs to supply, such as a screenshot, a number or an opinion.
6. **Ship the brief for approval.** Open the brief as a PR together with the updated state files.
   - Merging the PR approves the brief. The owner can edit or delete topics in the PR before merging.
   - On merge, the remaining picks change from `briefed` to `approved`.
   - Picks the owner deleted become `rejected`, with the reason if the owner left one in a comment.

## Write (daily)

1. **Choose a candidate.** Work from the default branch only. Take the next `approved` candidate from the latest merged brief. If there isn't one, write nothing: say so in the run output and stop. Never write anything that hasn't been approved.
2. **Study what the engines cite.** Read the pages the engines currently cite for the target prompts, and the top organic results for the search query. Note:
   - what those pages claim;
   - what they get wrong or leave out;
   - what an engine would need in order to cite this post instead.
3. **Write the post.** Follow the principles below, the voice in `voice.md` and the locale in `profile.md`. Where the two conflict, `voice.md` wins on tone and `profile.md` wins on facts. Then run the check in "Reading like a person wrote it" over the draft.
4. **Wire it into the site.**
   - Use the repo's frontmatter and add `target_prompts` and `target_query` fields.
   - Set the title and meta description, add structured data that matches the post type, and add alt text.
   - Link out to related posts and product pages, and add links to the new post from older posts.
5. **Check the build.** Run lint, type checks and the build. Render the page if you can, and confirm that links resolve and the post appears in the sitemap or feed.
6. **Open a PR** on the branch `post/<slug>`. The PR body gives the target prompts with their current state, the angle, the sources used, and any placeholders the owner needs to fill.
7. **Update the state.** Set the candidate to `written`, and add the post to `ledger.jsonl` with the go-live date left as null. A later run fills in the date once the PR has been merged.

## Measure (daily, after Write)

1. Find each post in the ledger that went live 7 or 28 days ago (allowing 2 days either side) and has no measurement for that checkpoint yet.
2. Run its target prompts through Sonable again, and pull Search Console data for its query and page.
3. Record the outcome:
   - **Win:** the product is cited or mentioned for most of the target prompts.
   - **Partial:** it is cited or mentioned for some of them.
   - **Miss:** it is cited or mentioned for none of them.
4. At 28 days, a miss becomes a refresh candidate, with a note on what the cited sources have that the post lacks. A partial with strong Search Console impressions also becomes a refresh candidate.
5. Update `weights.yaml`. Move each post type's weight slightly towards that type's win rate over the last eight weeks, by no more than 10% per week. Report the changes in one line in the next brief.

## Writing principles

- **Be the source an engine would quote.** Each section opens with a claim that stands on its own and is specific enough to lift whole: who it applies to, a number, a condition. Explanation comes after the claim.
- **Be explicit about names.** Name the product, the category and the competitors in full. Never leave the reader to infer which tool is meant.
- **Make it checkable.** Every external fact links to its primary source. Every product fact matches `profile.md`. If a fact can't be checked, leave it out.
- **Know something the cited pages don't.** That can be how the product actually behaves, a worked example, a decision table, or the owner's view. If you can't find something like that, reject the candidate with the reason rather than rewriting what competitors already say.
- **Compare honestly.** A comparison says where each competitor is the better choice, and gives the date its facts were checked.
- **Fit the length to the job.** Stop once the target prompts are answered.

## Reading like a person wrote it

Readers and editors stop trusting text that reads as machine-written, and so do the people who decide what engines cite. These checks come from Wikipedia's field guide, [Signs of AI writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing). Each pattern is a symptom: a sentence that sounds important but says less than it seems to. Models drift toward the generic and replace an unusual, specific fact with a broad, positive claim. Fix a flagged sentence by putting the specific fact back, or delete it. Never just swap in a synonym. That hides the pattern and leaves the empty sentence.

**Content**

- **State the fact, not its significance.** Cut "plays a pivotal role", "is a testament to", "marks a shift", "underscores the importance of", "the evolving landscape", "sets the stage for". If something matters, the fact should show why.
- **No analysis tacked on with "-ing".** Cut tails like ", highlighting its commitment to…", ", ensuring…", ", reflecting…", ", contributing to…", ", fostering…". Either it is a claim with a source, written as its own sentence, or it goes.
- **Name the relationship.** Don't write "associated with", "in connection with" or "linked to". Say who built, owns, uses, replaced or integrates with what.
- **Name who said it.** Don't write "experts argue", "industry reports suggest", "observers note", "critics say", and don't say "several sources" when there is one. Name the person or publication and link to it, or cut the claim.
- **Coverage is not a fact about the product.** Don't list outlets that "featured" or "profiled" something, or say that a directory "lists" it. Say what the product does.
- **No formula endings.** Skip "Despite its strengths, X faces challenges…", "Future outlook" sections, and closers that start "In summary", "In conclusion" or "Overall". Put a real limitation, stated concretely, where it applies.

**Language**

- **Use plain verbs.** Write "is", "are" and "has". Avoid "serves as", "stands as", "functions as", "boasts", "features", "offers" (when it means "has") and "represents".
- **No negative parallelisms.** Avoid "It's not X, it's Y", "not just X but Y", "not only… but also" and "Y rather than X" used for effect. Say Y.
- **No reflexive threes.** List as many items as actually exist. Two is fine.
- **Watch for AI vocabulary in clusters.** Words like "additionally" (to open a sentence), "align with", "bolster", "crucial", "delve", "enhance", "foster", "garner", "highlight", "interplay", "intricate", "landscape", "leverage", "meticulous", "multifaceted", "pivotal", "robust", "showcase", "tapestry", "testament", "underscore", "valuable" and "vibrant". One of these words is usually harmless. Several in one paragraph means rewrite the paragraph. The list shifts as models change, so read for the cluster, not for single words.
- **No brochure adjectives.** Avoid "groundbreaking", "renowned", "seamless", "nestled", "in the heart of", "rich", "profound", "a diverse array of" and "commitment to".
- **Repeat the term.** Call the product, the feature or the competitor by the same name each time. Rotating synonyms to avoid repetition makes the reader guess whether it's the same thing.
- **No gap-filling hedges.** Avoid "It's important to note", "based on available information", "not widely documented", and "should not be confused with" when nobody is confused. If information is missing, leave the point out or add a `TODO(owner)` placeholder.

**Format**

- Headings are in sentence case, without emoji.
- Use bold at most for a key term, not in every paragraph. Don't break prose into lists of bold label plus colon; write a paragraph or use a table.
- Use em dashes sparingly. Commas, colons and periods usually work better. Match the punctuation, quotes and Markdown of the existing posts.

**Leftovers that give it away outright**

- Chat residue: "I hope this helps", "Certainly!", "Let me know if…", "Here is a…", anything about a knowledge cutoff.
- Placeholder text: `example.com` links, `2025-XX-XX` dates, `INSERT_…` tokens. The only placeholders allowed are `TODO(owner)` items that are listed in the PR body.
- Links with tracking added by AI tools (`utm_source=chatgpt.com`, `utm_source=openai`, `utm_source=copilot.com`, `referrer=grok.com`). Strip the parameters. Open every cited link: it must load and say what the post says it says. A dead link, a DOI or ISBN that doesn't resolve, or a book cited without a page number points to a made-up source.

**The check.** Before opening the PR, read the draft once against this section and rewrite every hit. Don't use AI detectors to judge the result. They are wrong often in both directions. The test is whether every sentence makes a specific claim a reader could check.

## Guardrails

- Write only to the repo, and only through PRs. Never push to the default branch, publish anywhere else, or contact anyone.
- Write at most one new post or refresh per day, and only from an approved brief.
- Never change product facts, prices or claims. If the code disagrees with `profile.md`, flag it in the PR.
- Never remove or redirect an existing post. If one should go, say so in the brief.
- If a step fails twice, stop, open nothing, and report what failed.

## Bootstrap (no `editorial/` folder yet)

1. Read the repo to learn how the blog works.
2. Write `profile.md`. Mark anything you can't establish as `TODO(owner)`.
3. If `voice.md` is missing, draft it in the same structure social-engine uses. Base it on the owner's own writing samples, the existing blog posts and the owner's recent social posts. Quote real lines as examples, and mark anything inferred as `(inferred)`. If `voice.md` already exists, leave it as it is.
4. Write a first version of `prompts.yaml`: three to six clusters, each with five to eight buyer prompts.
5. Write `weights.yaml` with every type weight set to 1.0.
6. Run Brief, and include the bootstrap files in the same PR.
