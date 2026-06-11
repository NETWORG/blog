# GitHub Copilot vs Claude Code: what $200 buys, measured

On June 1, 2026, GitHub Copilot switched from flat pricing to **token based billing** ([GitHub blog](https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/)). For agentic work the change was large. Some developers posted screenshots of monthly bills going from **tens of dollars to hundreds**. Others said it was not as bad as it looked.

That leaves a concrete question for anyone doing heavy agentic work on their own $200 a month. How much real work does GitHub Copilot deliver for that money, and how does it compare to Claude Code? The two tools do not meter the same way. Copilot counts credits spent, Claude Code counts a rolling time window. Anthropic does not publish how big that window is, so there is nothing to look up, and that is why this comparison exists. The honest way to do it is to measure how many tokens the window actually holds, price the same work at the per token rates Copilot bills, and see what $200 buys on each side. Most public answers are estimates with no method behind them. This post measures it on a real account and shows every step.

Anthropic also shipped a new model partway through this work, Claude Fable 5. A new model in the rotation can move usage limits, so the task was measured a second time after it landed, **still on Sonnet**, to confirm the window held. Both runs are in the results.

## The claims in circulation

The same numbers come up again and again, but the sources behind them are either missing or have no clear methodology.

- Claude Max 20x is "worth about $5,000 of API compute, around 25 times the price." This traces to one developer's estimate, passed around pricing blogs with no measurement shown ([findskill](https://findskill.ai/blog/claude-code-subscription-pricing-guide/)).
- Max 20x gets "roughly 220,000 tokens per 5 hour window." This is stated as a flat figure with no method behind it ([faros](https://www.faros.ai/blog/claude-code-token-limits)).

The gap exists because **Anthropic does not publish a token number for the window** ([usage limits](https://support.claude.com/en/articles/11647753-how-do-usage-and-length-limits-work)), and it describes limits in relative terms rather than fixed token counts. The number is not unknowable though. Community tools like ccusage read Claude Code's own local logs and report the tokens and the API equivalent cost of each run. That is the method this post uses.

## How the two meters work

Both tools charge by tokens. A token is a small chunk of text. Every request mixes a few kinds, and the comparison depends on which one each side counts:

- **input**, what you send to the model
- **output**, what the model writes back
- **cache read**, context reused from earlier and billed much cheaper than input
- **cache write**, context stored so it can be reused, billed a little more than input

Copilot meters every token at the model's own rate, the same price Anthropic charges on its API, and converts the total to AI credits at **$0.01 each**. GitHub publishes the figures it uses ([GitHub docs](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing)):

| Claude Sonnet 4.6 (per 1M tokens) | Rate |
|---|---|
| Input | $3.00 |
| Cached input | $0.30 |
| Cache write | $3.75 |
| Output | $15.00 |

Claude is different. The $200 Max 20x plan is **flat**, and usage is gated by a rolling five hour window ([higher limits](https://www.anthropic.com/news/higher-limits-spacex)). Anthropic gives no token number for that window, only that **Max 20x is twenty times the Pro plan** ([Max plan](https://support.claude.com/en/articles/11049741-what-is-the-max-plan)).

The five hour window is not the only ceiling. Anthropic also caps usage over **a rolling week**, again with no published token number. This post measures both. The results show the weekly cap sitting well above what a normal working day reaches, so the five hour window is the one that decides the figures below.

These limits used to bite harder. Through 2025 Claude capped usage aggressively and throttled during peak hours. In May 2026, after a compute deal with SpaceX, Anthropic **doubled the five hour limits and removed the peak hour throttling** for Pro and Max ([higher limits](https://www.anthropic.com/news/higher-limits-spacex)). So the window a 9 to 5 developer runs into today is more generous than the one the older blog numbers were written against.

## Method

The method is simple. Run one fixed task, measure each run the same way, and run it more than once so a single reading cannot mislead.

The task: read three small JavaScript files, find the bugs, then write the corrected files, a test suite, docs, a short review, and a migration guide. One run is one heavy task, a realistic agentic job rather than a single question.

Each run was captured two ways:

- [ccusage](https://github.com/ryoppippi/ccusage), an open source CLI that reads Claude Code's local logs and reports token counts and the API equivalent cost
- the Claude Code /usage screen, which shows the percent of the 5 hour window used

Reading both before and after a run gives the cost of that single task.

The account used is **Claude Pro ($20), not Max 20x ($200)**. Since Anthropic defines Max 20x as 20 times Pro usage, with the window scaling the same way ([Max plan](https://support.claude.com/en/articles/11049741-what-is-the-max-plan)), the Pro measurements are **multiplied by 20** for the Max 20x figures.

## Results

Both runs were on **Sonnet 4.6**, the same task measured the same way. The first was on June 5, the second on June 10.

| Run | Window used | Input | Output | Cache write | Cache read | Total tokens | API cost |
|---|---|---|---|---|---|---|---|
| First run, June 5 | 11% | 7 | 50,638 | 38,178 | 74,870 | 163,693 | $0.92 |
| Second run, June 10 | 21% | 44 | 100,588 | 274,392 | 1,989,520 | 2,364,544 | $3.76 |

The table carries all four token types from the meter section, but only one drives the 5 hour window, and that is **output**. Input, cache write, and cache read can swing by a lot without moving the window, so from here the window is read in output terms. The rest of the columns are kept for context and for the cost, which is built from all of them. The second run's cache writes were billed at Anthropic's 1 hour cache rate, $6 per million rather than $3.75 ([prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)), so its $3.76 only rebuilds with that higher rate.

The first run produced 50,638 output tokens for 11 percent of the 5 hour window, which puts a full window near **460,000 output tokens** (50,638 / 0.11). At 11 percent per task, one Pro window holds about **9 heavy tasks**. Multiplied by 20, **Max 20x holds about 182 tasks per window**.

The second run was done five days later, for two reasons. One was plain certainty, since a single measurement is easy to doubt. The other was timing. On June 9 Anthropic released **Claude Fable 5** and gave it to Pro and Max subscribers at no extra cost for a two week launch window ([TechCrunch](https://techcrunch.com/2026/06/09/anthropics-claude-fable-5-is-a-version-of-mythos-the-public-can-access-today/)). A new and free model in the rotation can move usage limits, so the window was worth re-checking before publishing.

It held, and it showed why output is the right unit. The same prompt does not produce the same output twice. An agent takes more or fewer turns etc. and this run generated about twice as much as the first. Output went **50,638 to 100,588**, and the window moved with it, **11 to 21 percent**. The other tokens did not track. Cache read alone went from 74,870 to almost 2 million, more than twenty five times as many, while the window only doubled. Total tokens told the same story, fourteen times as many for twice the window. The implied full window, near 479,000, lands within about 4 percent of the first. The number is stable, before the new model and after it.

A third run checked the other limit. The /usage screen reports a weekly percentage next to the five hour one, so the same task was read against both, before and after. At 111,097 output tokens, close to the second run, it moved the five hour window **21 percent but the weekly window only 2 percent**. In work terms the weekly window is about ten times the five hour one. To exhaust it a developer would have to run roughly ten back to back saturated five hour windows inside a single week.

## The $200 comparison

Same model on both sides, so the same work costs the same tokens either way. Only the meter differs. The quantity to follow is **output tokens**, what the Claude window meters and the bulk of the cost on both sides.

Turning tokens into money needs a reference, so anchor on one measured run. The first run was 50,638 output tokens, in a mix of 38,178 cache write, 74,870 cache read and 7 input. At Anthropic's published Sonnet rates (output 50,638 x $15, cache write 38,178 x $3.75, cache read 74,870 x $0.30, input 7 x $3, all per million) that mix costs **$0.92, which is 92 credits** on Copilot at $0.01 each. The runs varied, the later ones landed near 100,000 output, so 50,638 is one representative task, not a fixed size. It is the lower, cleaner run, so it makes the conservative anchor.

Now follow the output tokens on each side.

**Claude Max 20x, for $200:**

1. The plan is flat, gated by the output metered 5 hour window, no credits.
2. The first run used 11 percent of the Pro window for 50,638 output tokens, so a full window is about **460,000 output tokens** (50,638 / 0.11). Times 20 for Max 20x is about **9.2 million output per window**.
3. A working day is about 1.6 windows, a month about 22 days, so about 35 windows. That is about **324 million output tokens a month**.
4. At the reference task's 50,638 output, that is about **6,400 heavy tasks**.

**GitHub Copilot, for $200:**

1. The Max plan is $100 a month, and its base and flex allotments come to $200 of credits, which is 20,000 credits at $0.01 each ([GitHub blog](https://github.blog/news-insights/company-news/github-copilot-individual-plans-introducing-flex-allotments-in-pro-and-pro-and-a-new-max-plan/)). The other $100 buys 10,000 more as overage, so $200 gives **30,000 credits**.
2. The reference task is 92 credits for 50,638 output, about 550 output tokens per credit at that mix.
3. 30,000 credits is about **16.5 million output tokens**, then the credits are gone.
4. That is about **326 heavy tasks**.

| For $200 a month (Sonnet) | GitHub Copilot | Claude Max 20x |
|---|---|---|
| What $200 covers | Max $100 (20,000 cr) + $100 overage | flat plan |
| Output tokens per month | ~16.5M | ~324M |
| Heavy tasks per month | ~326 | ~6,400 |

That is about **20 times more** usable Claude work for the same money, 324 million output tokens against 16.5 million, the same gap as about 6,400 heavy tasks against 326. In dollar terms, those 6,400 tasks are about **$5,900 of API equivalent value a month**, at $0.92 each.

## Limits of this measurement

- The Max 20x numbers are Pro measurements multiplied by 20. The multiplier is Anthropic's own plan definition, but it is not a direct Max 20x measurement.
- The monthly Claude figure assumes about 1.6 five hour windows in a working day. A lighter day gives a lower number.
- The weekly cap does not bind at a 9 to 5 pace, see the third run above, but that weekly reading is coarse. It moved only two percentage points and /usage rounds to whole numbers, so the true cost per task is somewhere between about 1.5 and 2.5 percent of the week. At the high end the weekly ceiling and the working day pace are about even. Either way the weekly limit does not pull the monthly figure below the 6,400. The one account that did hit the weekly cap every week ([issue #61426](https://github.com/anthropics/claude-code/issues/61426)) was an all day power user on the pricier Opus model, far past a 9 to 5 pace.
- Copilot's credit and flex structure is new as of June 1 and can change over time, which would move the credit math in either direction ([GitHub docs](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-individuals)).

## Conclusion and arguments other than price

So, we learned the regular 9 to 5 dev should currently go for Claude, as it gives him way, way more usage, 20x more to be exact. Of course, with a result like that, a question pops up. **How is it possible?** My best guess is that Anthropic heavily subsidizes their subscription pricing to stay competitive. Get developers in, get them used to the workflow, worry about the money later. But, as we saw with Copilot, this may not last forever.

### Other things worth considering

So, Claude clearly wins on usage. A few arguments come to mind that are still good to consider:

- Copilot's real strength is that it is **a router, not a model**. One subscription reaches Claude, GPT 5.5, and Gemini, and you pick per task. For a large company that can outweigh the per token math.
- Microsoft is an established vendor, so the tool is often already cleared by procurement, which is half the work in a big org.
- The billing can favor the buyer at scale. Claude's enterprise plan is a per seat fee plus standard API rates, with **no included usage and no caps** ([enterprise plan](https://support.claude.com/en/articles/9797531-what-is-the-enterprise-plan)). The subsidized individual and team plans are **not offered past about 150 members**, so a large org pays API rates either way ([pricing](https://claude.com/pricing)).
- For autocomplete and short chats on GitHub Copilot, little changed, because **completions stay free** ([GitHub docs](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing)). The gap opens once the work becomes agentic.
- Anthropic's murky usage caps are one of its biggest weaknesses. The flat plan is a strong deal today, but Anthropic does not publish the token size of the 5 hour window, and they have historically changed and throttled it a lot, so that deal is not promised either. Limits and pricing move with little notice, sometimes up, as the May increase showed ([higher limits](https://www.anthropic.com/news/higher-limits-spacex)), and a generous flat plan can be re-cut the very same way.

None of that changes the answer for the person this post is about. For a 9 to 5 developer doing agentic development on their own $200, Claude Code today is the better deal by a wide margin.

## Sources

- GitHub Copilot moving to usage based billing: https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/
- GitHub Copilot models and pricing, per token rates: https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing
- Anthropic, prompt caching pricing (Sonnet cache write rates, $3.75 five minute and $6.00 one hour): https://platform.claude.com/docs/en/build-with-claude/prompt-caching
- GitHub Copilot flex allotments and the Max plan: https://github.blog/news-insights/company-news/github-copilot-individual-plans-introducing-flex-allotments-in-pro-and-pro-and-a-new-max-plan/
- GitHub Copilot usage based billing for individuals: https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-individuals
- Anthropic, what is the Max plan: https://support.claude.com/en/articles/11049741-what-is-the-max-plan
- Anthropic, how usage and length limits work: https://support.claude.com/en/articles/11647753-how-do-usage-and-length-limits-work
- Anthropic, higher usage limits and the SpaceX compute deal: https://www.anthropic.com/news/higher-limits-spacex
- Anthropic, what is the Enterprise plan: https://support.claude.com/en/articles/9797531-what-is-the-enterprise-plan
- Claude plans and pricing: https://claude.com/pricing
- ccusage: https://github.com/ryoppippi/ccusage
- Claude Code issue #61426, a Max 20x user's 30 day usage report: https://github.com/anthropics/claude-code/issues/61426
- TechCrunch, Anthropic releases Claude Fable 5 to the public: https://techcrunch.com/2026/06/09/anthropics-claude-fable-5-is-a-version-of-mythos-the-public-can-access-today/
