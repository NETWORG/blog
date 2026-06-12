# What $200 a month buys on GitHub Copilot vs Claude Code

On June 1, 2026, GitHub Copilot (GHCP) moved from flat pricing to **token based billing** ([GitHub blog](https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/)). For agentic work this is a big change. Some developers posted screenshots of their monthly bill jumping from **tens of dollars to hundreds**. Others said it is not that bad.

So for anyone who does heavy agentic work on their own $200 a month, there is a simple question. How much real work does GHCP give you for that money? And how does it compare to Claude Code?

GHCP is really a router, it can reach multiple models. For this comparison the model is Claude Sonnet, which both GHCP and Claude Code can run. It is the same model with the same context size, so the same task costs the same tokens either way. GHCP charges per token, as credits at Anthropic's API rates. Claude Code does not meter the task at all, it is a flat subscription, gated instead by a rolling time quota.

But Anthropic does not publish how big that quota is, so you cannot look it up. The only way to know is to measure it. So I ran one real coding task, watched how much of the quota it used, and worked back to the full size. Then I priced the same task at GHCP's per token rates, and saw what $200 buys on each side.

In the middle of this, Anthropic shipped a new model, Claude Fable 5. A new model in the rotation can move the limits, so I ran the task a second time after it landed, **still on Sonnet**, to check the quota still holds. Both runs are in the results below.

A quick note before the numbers. Everything here is from 12. of June 2026, and Anthropic changes these limits a lot. So read it as a snapshot of today, not a promise.

## The claims in circulation

You find the same numbers everywhere, copied from post to post, but none of them show a real source or method.

- Claude Max 20x is "worth about $5,000 of API compute, around 25 times the price." This traces to one developer's estimate, passed around pricing blogs with no measurement shown ([findskill](https://findskill.ai/blog/claude-code-subscription-pricing-guide/)).
- Max 20x gets "roughly 220,000 tokens per 5 hour window." This is stated as a flat figure with no method behind it ([faros](https://www.faros.ai/blog/claude-code-token-limits)).

The gap is there because **Anthropic does not publish a token number for the quota** ([usage limits](https://support.claude.com/en/articles/11647753-how-do-usage-and-length-limits-work)). They describe the limit in relative terms, not in fixed tokens. But the number is not unknowable. Tools like ccusage read Claude Code's own local logs and report the tokens and the API cost of each run. So that is what I use here.

## How the two meters work

I need one common unit, tokens. A token is a small piece of text. Every request mixes a few kinds, and the comparison depends on which one each side counts.

- **input**, what you send to the model
- **output**, what the model sends back
- **cache read**, context reused from earlier and billed much cheaper than input
- **cache write**, context stored so it can be reused, billed a little more than input

GHCP meters every token at the model's own rate, the same price Anthropic charges on its API. Then it turns the total into AI credits, **$0.01 each**. GitHub publishes the numbers it uses ([GitHub docs](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing)):

| Claude Sonnet 4.6 (per 1M tokens) | Rate |
|---|---|
| Input | $3.00 |
| Cache read | $0.30 |
| Cache write | $3.75 |
| Output | $15.00 |

Claude works different. The $200 Max 20x plan is **flat**. Your usage is gated by a rolling five hour quota ([higher limits](https://www.anthropic.com/news/higher-limits-spacex)). Anthropic gives no token number for it, only that **Max 20x is twenty times the Pro plan** ([Max plan](https://support.claude.com/en/articles/11049741-what-is-the-max-plan)). Go past the quota and you can keep going on usage credits, billed at the same standard API rates ([usage credits](https://support.claude.com/en/articles/12429409-manage-usage-credits-for-paid-claude-plans)). So once the included usage runs out, the per token price is the same on both sides. The only question is how much each $200 includes before you get there.

The five hour quota is not the only ceiling. Anthropic also caps usage over **a rolling week**, again with no token number. I measured both. The weekly cap sits well above a normal working day, so the five hour quota is the one that decides the numbers below.

These limits used to bite harder. Through 2025 Claude capped usage hard and throttled in peak hours. Then in May 2026, after a compute deal with SpaceX, Anthropic **doubled the five hour limits and dropped the peak hour throttling** for Pro and Max ([higher limits](https://www.anthropic.com/news/higher-limits-spacex)). So the quota a 9 to 5 dev hits today is bigger than the one those older blog numbers were written against.

## Method

The method is simple. I run one fixed task, I measure each run the same way, and I run it more than once, so a single reading cannot mislead.

The task: read three small JavaScript files, find the bugs, then write the fixed files, a test suite, docs, a short review and a migration guide. One run is one heavy task, a real agentic job.

I captured each run two ways:

- [ccusage](https://github.com/ryoppippi/ccusage), an open source CLI that reads Claude Code's local logs and reports token counts and the API equivalent cost
- the Claude Code /usage screen, which shows the percent of the 5 hour quota used

I read both before and after a run. The difference is the cost of that one task.

I used a **Claude Pro account ($20), not Max 20x ($200)**. Anthropic defines Max 20x as 20 times the Pro usage, and the quota scales the same way ([Max plan](https://support.claude.com/en/articles/11049741-what-is-the-max-plan)). So for the Max 20x figures I just **multiply the Pro numbers by 20**.

## Results

Both runs were on **Sonnet 4.6**, same task, same measurement. First on June 5, second on June 10.

| Run | Quota used | Input | Output | Cache write | Cache read | Total tokens | API cost |
|---|---|---|---|---|---|---|---|
| First run, June 5 | 11% | 7 | 50,638 | 38,178 | 74,870 | 163,693 | $0.92 |
| Second run, June 10 | 21% | 44 | 100,588 | 274,392 | 1,989,520 | 2,364,544 | $3.76 |

The table has all four token types from the meter section. But only **output** drives the 5 hour quota. Input, cache write and cache read can swing a lot and the quota does not move, so from here I read the quota in output. The other columns are for context and cost. The second run's cache writes were billed at Anthropic's 1 hour cache rate, $6 per million instead of $3.75 ([prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)), so its $3.76 only adds up with that higher rate.

The first run made 50,638 output tokens for 11 percent of the 5 hour quota. That puts a full quota near **460,000 output tokens** (50,638 / 0.11). So at 11 percent per task, one Pro quota holds about **9 heavy tasks**. Times 20, **Max 20x holds about 182 tasks per quota**.

I did the second run five days later, for two reasons. One was just to be sure, since a single measurement is easy to doubt. The other was timing. On June 9 Anthropic released **Claude Fable 5** and gave it to Pro and Max for free, for a two week launch window ([TechCrunch](https://techcrunch.com/2026/06/09/anthropics-claude-fable-5-is-a-version-of-mythos-the-public-can-access-today/)). A new free model in the rotation can move the limits, so the quota was worth a re-check before I publish.

It held. And it shows why output is the right unit. The same prompt does not give the same output twice. The agent takes more or fewer turns, and this run made about twice as much as the first. Output went **50,638 to 100,588**, and the quota moved with it, **11 to 21 percent**. The other tokens did not follow. Cache read alone went from 74,870 to almost 2 million, more than twenty five times more, and the quota only doubled. Total tokens, same story, fourteen times more for twice the quota. The full quota this implies, near 479,000, sits within about 4 percent of the first. So the number is stable, before the new model and after.

A third run checked the other limit. The /usage screen shows a weekly percent next to the five hour one, so I read the same task against both, before and after. At 111,097 output tokens, close to the second run, it moved the five hour quota **21 percent, but the weekly quota only 2 percent**. So in work terms the weekly quota is about ten times the five hour one. To use it up, a dev would have to run something like ten full five hour quotas back to back, inside one week.

## The $200 comparison

Same work, same tokens either way. Only the meter differs. The number to follow is **output tokens**, because that is what the Claude quota meters.

To turn tokens into money I anchor on one measured run. The first run was 50,638 output tokens, in a mix of 38,178 cache write, 74,870 cache read and 7 input. At Anthropic's published Sonnet rates (output 50,638 x $15, cache write 38,178 x $3.75, cache read 74,870 x $0.30, input 7 x $3, all per million) that mix costs **$0.92, so 92 credits** on GHCP at $0.01 each. The runs varied, the later ones were near 100,000 output, so 50,638 is just one normal task, not a fixed size. It is the lower, cleaner run, so I take it as the careful anchor.

It is careful for a reason. GHCP bills every token, cache included. The Claude quota does not count cache at all. So the more an agent reuses context, the more GHCP pays, while the Claude quota does not move. This first run is light on cache, so it gives GHCP its best case. The gap below is a floor, heavier work only widens it.

Now I follow the output tokens on each side.

**Claude Max 20x, for $200:**

1. The plan is flat, gated by the output metered 5 hour quota, no credits.
2. The first run used 11 percent of the Pro quota for 50,638 output tokens, so a full quota is about **460,000 output tokens** (50,638 / 0.11). Times 20 for Max 20x is about **9.2 million output per quota**.
3. A working day is about 1.6 quotas, a month about 22 days, so about 35 quotas. That is about **324 million output tokens a month**.
4. At the reference task's 50,638 output, that is about **6,400 heavy tasks**.

**GHCP, for $200:**

1. The GHCP Max plan is $100 a month, and its base and flex allotments come to $200 of AI credits, which is 20,000 at $0.01 each ([GitHub blog](https://github.blog/news-insights/company-news/github-copilot-individual-plans-introducing-flex-allotments-in-pro-and-pro-and-a-new-max-plan/)). The other $100 buys 10,000 more as overage, so $200 gives **30,000 credits**.
2. The reference task is 92 credits for 50,638 output, about 550 output tokens per credit at that mix.
3. 30,000 credits is about **16.5 million output tokens**, then the credits are gone.
4. That is about **326 heavy tasks**.

| For $200 a month (Sonnet) | GHCP | Claude Max 20x |
|---|---|---|
| What $200 covers | Max $100 (20,000 cr) + $100 overage | flat plan |
| Output tokens per month | ~16.5M | ~324M |
| Heavy tasks per month | ~326 | ~6,400 |

So that is about **20 times more** usable Claude work for the same money. 324 million output tokens against 16.5 million. Or about 6,400 heavy tasks against 326. In money, those 6,400 tasks are about **$5,900 of API value a month**, at $0.92 each.

## Limits of this measurement

- The Max 20x numbers are Pro measurements times 20. The multiplier is Anthropic's own plan definition, but still, it is not a direct Max 20x measurement.
- The monthly Claude figure assumes about 1.6 five hour quotas in a working day. A lighter day gives a lower number.
- The weekly cap does not bind at a 9 to 5 pace, see the third run above. But that weekly reading is rough. It moved only two percentage points, and /usage rounds to whole numbers, so the real cost per task is somewhere between about 1.5 and 2.5 percent of the week. At the high end the weekly ceiling and the working day pace are about even. Either way the weekly limit does not pull the monthly figure below 6,400. The one account that did hit the weekly cap every week ([issue #61426](https://github.com/anthropics/claude-code/issues/61426)) was an all day power user on the pricier Opus model, far past a 9 to 5 pace.
- GHCP's credit and flex structure is new since June 1 and can change over time, which would move the credit math either way ([GitHub docs](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-individuals)).

## Conclusion and arguments other than price

So the regular 9 to 5 dev should go for Claude today, it gives him way, way more usage, 20x more to be exact. A result like that raises a question - how is it even possible? My best guess is that Anthropic heavily subsidizes their subscription pricing to stay competitive. Other sources seem to confirm this. Get developers in, get them used to the workflow, worry about the money later. But as we saw with GHCP, this may not last forever.

### Other things worth considering

Claude clearly wins on usage. But a few other things are worth considering:

- GHCP's real strength is that it is **a router, not a model**. One subscription gives you Claude, GPT 5.5 and Gemini, and you pick per task. For a big company that can beat the per token math.
- Microsoft is an established vendor, so the tool is often already cleared by procurement, which is half the work in a big org.
- At scale the billing can favor the buyer. Claude's enterprise plan is a per seat fee plus standard API rates, with **no included usage and no caps** ([enterprise plan](https://support.claude.com/en/articles/9797531-what-is-the-enterprise-plan)). The subsidized individual and team plans are **not sold past about 150 members**, so a big org pays API rates either way ([pricing](https://claude.com/pricing)).
- For autocomplete and short chats on GHCP, little changed, because **completions stay free** ([GitHub docs](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing)). The gap opens once the work becomes agentic.
- Anthropic's murky usage caps are one of its biggest weaknesses. The flat plan is a strong deal today, but Anthropic does not publish the token size of the 5 hour quota, and they have historically changed and throttled it a lot, so that deal is not promised either. Limits and pricing move with little notice, sometimes up, as the May increase showed ([higher limits](https://www.anthropic.com/news/higher-limits-spacex)), and a generous flat plan can be re-cut the very same way.

But none of that changes the answer for the person this post is about. For a 9 to 5 developer doing agentic work on their own $200, Claude Code today is the better deal, by a wide margin.

## Sources

- GitHub Copilot moving to usage based billing: https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/
- GitHub Copilot models and pricing, per token rates: https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing
- Anthropic, prompt caching pricing (Sonnet cache write rates, $3.75 five minute and $6.00 one hour): https://platform.claude.com/docs/en/build-with-claude/prompt-caching
- GitHub Copilot flex allotments and the Max plan: https://github.blog/news-insights/company-news/github-copilot-individual-plans-introducing-flex-allotments-in-pro-and-pro-and-a-new-max-plan/
- GitHub Copilot usage based billing for individuals: https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-individuals
- Anthropic, what is the Max plan: https://support.claude.com/en/articles/11049741-what-is-the-max-plan
- Anthropic, how usage and length limits work: https://support.claude.com/en/articles/11647753-how-do-usage-and-length-limits-work
- Anthropic, manage usage credits for paid plans (overage billed at standard API rates): https://support.claude.com/en/articles/12429409-manage-usage-credits-for-paid-claude-plans
- Anthropic, higher usage limits and the SpaceX compute deal: https://www.anthropic.com/news/higher-limits-spacex
- Anthropic, what is the Enterprise plan: https://support.claude.com/en/articles/9797531-what-is-the-enterprise-plan
- Claude plans and pricing: https://claude.com/pricing
- ccusage: https://github.com/ryoppippi/ccusage
- Claude Code issue #61426, a Max 20x user's 30 day usage report: https://github.com/anthropics/claude-code/issues/61426
- TechCrunch, Anthropic releases Claude Fable 5 to the public: https://techcrunch.com/2026/06/09/anthropics-claude-fable-5-is-a-version-of-mythos-the-public-can-access-today/
