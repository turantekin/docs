# Claude review: learning loop plan

13 Oct 2026. A review of "Learning loop plan (13 Oct 2026)" and "Claude full proposal: learning loop (13 Oct 2026)", plus the GitHub repos worth using. This is advice only: nothing in the fleet was changed. Figures marked **(proposed)** are suggestions, not measurements.

## 1. Verdict

The design is good and safe, and it is worth building. But it is built for more data than the fleet produces, and it sits on parts that are broken today. Start smaller, fix the data first, and copy four proven features from open-source projects instead of installing a framework.

## 2. What is strong

- **It starts from evidence.** The audit of where feedback is lost (21 rejections with no reasons, overwritten edits, failures that never become lessons) is the most valuable part.
- **A fixed script decides the risk, not Claude.** Keep this.
- **The safety net is right:** backups, undo, an off switch, protected files, and collecting before changing anything.
- **The main measure is right:** "Uygar rejects the same kind of mistake twice" should fall.

## 3. What to change

1. **Fix the bad data before learning from it.** Today an unticked item is written as rejected, and at least one "rejected" item had already been posted (xavi-20261007-13). Record unticked items as "skipped", and learn only from an explicit no. Do this before anything else, or the bots learn the wrong lessons.
2. **Prioritise Uygar's reasons and in-the-turn capture.** A three-word reason ("no 3: too salesy") is worth more than all the nightly analysis. The bot saving a correction on the spot and replying "Saved as D-0412" is the best feature in the plan. Build it first and make it as easy as possible.
3. **Run the Claude lessons job weekly, not nightly.** There are about 20 rejections a week across many bots, so most nights the job would have little to work on. Keep the free Python collection nightly, and run the Claude step once a week. That gives it more data and costs less.
4. **Replace the 14-day "results got worse" auto-undo.** At this volume it cannot tell a real change from luck. Use the helpful/harmful score in section 6 instead, plus the existing rule: undo straight away when Uygar rejects an item with a reason that contradicts the lesson. Everything else is decided at the Friday review.
5. **Expect the word list to block most social tips.** Words like comment, reply, post and like appear in nearly every social lesson. During the suggestion-only weeks, log what the list would have blocked, then tune it on real results. A likely fix: allow those words in wording-only lessons that contain no number, account or permission word.
6. **Test that bots read their lessons before building on it.** Everything depends on this. Lessons shown at the moment a draft is checked or queued (voice_check, add_item, draft_check, persona_check) are more reliable than a step 0 in the persona. Build those first. Test with one bot and one harmless tip before rolling out to 21.
7. **Fix the foundation (phase 0 is required).** The transcript snapshot has been stale since 8 Oct, Yolk's nightly review has never run, and the memory export is from 4 Oct and missing 10 bots.
8. **Put a spending cap on the Claude job.** claude_job.sh has a timeout but no budget cap. Stop the job if weekly spend passes a limit Uygar chooses, for example $10 a week **(proposed)**. Keep Egg's role minimal, as the plan says, since Egg is about 13 times over its token budget.
9. **Plan for 2 to 3 weeks, not 8 to 9 days.** The estimate is build time only. The routine edits on the server and Uygar's approvals add elapsed time.

## 4. Answers to the three decisions

| Decision | Advice |
| :- | :- |
| 1. Start collecting now (phases 0 to 2)? | **Yes**, and add the weekly lessons summary in suggestion-only mode (phase 3 without auto-apply). Otherwise a week of collecting produces nothing to look at. Fix "unticked = rejected" first. |
| 2. Approve S8 now or after the trial? | **After the trial**, and give it 2 to 3 weeks rather than 1. Approve S8 if most suggestions are ones Uygar would have approved anyway. |
| 3. Fleet-wide wording lessons apply themselves? | **No.** Agree with the proposal: every fleet-wide change gets one look. |

## 5. GitHub repos: which are useful

The bots run on Grok's platform, so no repo can replace how they think or remember. A repo could only plug into one step: turning signals into lessons. That is about 1 of the 8 to 9 build days, and Claude already does it on the membership. Collecting the fleet's own feedback, the safety gate and the loading into bots are specific to this setup, and no repo does them. So: copy features now, and keep one install option for later.

| Repo | Verdict | Why |
| :- | :- | :- |
| [kayba-ai/agentic-context-engine](https://github.com/kayba-ai/agentic-context-engine) (ACE, Apache-2.0, about 2.6k stars, active) | **Copy its features now. Install later only if needed.** | Best match. It keeps a "Skillbook" of lessons, learns from corrections (learn_from_feedback) and from past logs (learn_from_traces), and removes duplicates. Against installing now: it bills per use through an API key rather than the membership, Grok is not listed as a supported model, and it duplicates the step Claude already does. |
| [ace-agent/ace](https://github.com/ace-agent/ace) (the original ACE research code, Apache-2.0) | Reference for the lesson format | Research code, not for production. Its playbook format (lesson id, helpful and harmful counters, sections) is the one to copy. |
| [NousResearch/hermes-agent-self-evolution](https://github.com/NousResearch/hermes-agent-self-evolution) (MIT) | **Maybe later, for phase 6** | It improves a SKILL.md by testing new versions against past real tasks, then hands over the change for review instead of applying it. The fleet's skills use the same SKILL.md format. It needs adapting and costs about $2 to $10 per run, so try it only after a few weeks of approvals and rejections exist. |
| NousResearch/hermes-agent | Copy one rule | Its curator only edits skills it wrote itself, and never deletes. |
| letta-ai/letta | Copy one idea | Version history for memory, kept like code (see feature 4). |
| LangMem, ExpeL, Mem0 | Skip | LangMem needs the LangChain framework. ExpeL is research code for test tasks (its upvote and downvote idea is covered by feature 1). Mem0 stores memory, and the fleet's problem is capturing feedback, not storing it. |

## 6. Four features to add to the plan

### Feature 1: A helpful and harmful score on every lesson (from ACE and ExpeL)

- Add two columns to lessons.csv: helpful and harmful.
- In the weekly job, Claude tags which lessons each decided item touched. Approved unedited adds 1 to helpful. Rejected or edited against the lesson adds 1 to harmful.
- Flag a lesson for the Friday review when harmful is greater than helpful. Retire it when harmful is at least helpful plus 2 **(proposed)**.
- This replaces the 14-day statistical trial in plan section 4.5.

### Feature 2: Small edits only, never rewrites (from ACE)

- Claude may only propose single-line operations: add, update one line, retire, or score.
- learn.py rejects any proposal that rewrites a whole skill, lesson list or brief. The ACE research found that full rewrites slowly lose detail.
- Keep the plan's rule of merging duplicates into the existing row's evidence.

### Feature 3: Only edit what it wrote, and never delete (from Hermes Agent)

- The automatic part may only touch the learned-tips blocks and lesson rows it created.
- Retiring marks a row retired and never deletes it.
- For the first month, removing anything from Grok memory needs Uygar's OK, including duplicates. This is stricter than plan section 5.3.

### Feature 4: Use git for version history (from Letta's versioned memory)

- Put shared/fleet/lessons.csv and the workflows/ skills folder in git. Every change becomes a saved version whose note names the change id and lesson id. Undo is one revert.
- This replaces three custom pieces: the .bak copies, the per-skill HISTORY.md files and most of lesson-changes.csv. It probably saves about a day of build.
- Test first that the Grok skill loader ignores a .git folder inside workflows/.

Small extra from ACE: split each bot's brief into two headings, "What works" and "Mistakes to avoid". It is easier for the bot to follow.

## 7. Updated build order

| Step | What | Changes anything? |
| :- | :- | :- |
| Week 1: fix the foundation | Spending cap on the Claude job. Unticked recorded as skipped, not rejected. Fix the stale transcript snapshot. Start Yolk's nightly. Re-export memory for all bots. Test lesson reading with one bot and one harmless tip. | Script fixes only |
| Week 1 to 2: capture | lessons.csv with helpful and harmful columns, under git. Nightly Python collection. The "why" field on ticks. Bots save Uygar's corrections on the spot (learn.py add). Keep first drafts and final sent text. | No, it only collects |
| Weeks 2 to 4: suggestions | Weekly Claude lessons job using the ACE format and single-line operations. Friday one-page summary. Everything needs Uygar's tick. Log what the word list would block. | Suggestions only |
| Week 4: decide | Review the suggestions. Tune the word list. Decide on S8. | Uygar's decision |
| After S8 | Low-risk lessons apply themselves. Lessons shown at the point of use first, then the persona step 0. | Yes |
| Later (optional) | Try Hermes self-evolution, or the plan's replay check, to test skill changes against past items. Install Kayba ACE only if Claude's weekly lessons prove poor; the swap is small because the format already matches. | Only after Uygar's OK |

## Sources

- [kayba-ai/agentic-context-engine](https://github.com/kayba-ai/agentic-context-engine)
- [ace-agent/ace](https://github.com/ace-agent/ace) and the paper [Agentic Context Engineering (arXiv 2510.04618)](https://huggingface.co/papers/2510.04618)
- [NousResearch/hermes-agent-self-evolution](https://github.com/NousResearch/hermes-agent-self-evolution) and [how it works](https://www.besthub.dev/articles/how-hermes-agent-s-self-evolution-framework-works-dspy-diagnoses-gepa-evaluates-f779dd0a77c0)
- [Hermes Agent docs](https://hermes-agent.nousresearch.com/docs) and [The two loops of Hermes Agent](https://levelup.gitconnected.com/the-two-loops-of-hermes-agent-33922ba8d154)
- [Letta overview](https://www.everydev.ai/tools/letta) and [Letta blog](https://www.letta.com/blog)
- [ExpeL paper](https://arxiv.org/html/2308.10144v3)
- [LangMem launch](https://www.langchain.com/blog/langmem-sdk-launch)
- [OpenAI cookbook: Self-Evolving Agents](https://developers.openai.com/cookbook/examples/partners/self_evolving_agents/autonomous_agent_retraining)
