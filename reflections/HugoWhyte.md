
# Individual Reflection: Hugo Whyte

## Evidence of Contribution

**Formative pitch**

After Joel proposed the research question in our first team meeting, I turned it into a PowerPoint deck for the EC1B1 Macroeconomics formative class pitch. The slides structured the project end-to-end before any code was written: the research question, the motivation for using Fed rate regimes, the ten SPDR ETFs as instruments, the three-notebook pipeline, the SQLite schema, the three planned visualisations, and the milestone timeline through to the 26 May deadline. The open question on the hiking-cycle definition was surfaced in those slides and later resolved in NB02 via the rolling 3-month mean which we debated on on WhatsApp. Presenting it to the class forced the team to commit to a scope before the temptation to expand it arose.

**NB01 - Data Collection ([Issue #6](https://github.com/lse-ds105/group-project-git-it-done/issues/6))**

My primary technical contribution was the data collection pipeline. I wrote `requests.get()` calls to FRED for three series (FEDFUNDS, CPIAUCSL, T10Y2Y) and to Alpha Vantage for all ten SPDR ETFs using the `TIME_SERIES_MONTHLY_ADJUSTED` endpoint. Rate limiting was handled with `time.sleep(15)` between ETF calls to stay within the free tier's 5 calls/minute ceiling. All raw responses were saved as JSON to `data/raw/` immediately on receipt, before any transformation. API keys were loaded exclusively via `load_dotenv()` and `os.getenv()`, never printed, never hardcoded.

The first version of NB01 was pushed directly to `main` without a branch, which was a workflow mistake. Joel left a review note on that commit flagging that the rate-limit `break` lost state about which tickers had been fetched versus which were pending. The raw JSON files had already saved correctly so NB02 was unaffected, but it was a fair catch. Every contribution after that, PR #17 and all review comments, went through branches and pull requests.

**README refresh - [PR #17](https://github.com/lse-ds105/group-project-git-it-done/pull/17)**

With submission imminent I updated the README to reflect what shipped rather than the pitch plan. Changes: pipeline and visualisations sections moved to past tense, a headline finding added near the top, a Limitations section written from scratch covering all four caveats (crisis-driven cutting cycles, XLRE's 124-observation history, neutral sample bias, rate-cycle labelling noise with a link to [Issue #9](https://github.com/lse-ds105/group-project-git-it-done/issues/9)), and a Reproduction section so a stranger could clone and run without hitting environment-specific paths. Research question, team roles, data sources, and schema were left untouched. The reproduction section stemmed from previous feedback in my mini projects where it wasn't exactly fully clear how to get the code to run, so I thought it would be a good idea to do so and it also helped to make sure it would run smoothly as I went through how someone would get it working for them (examiner). Merged 26 May 2026.

**Review comments across the pipeline**

Left inline comments on Parthiv's [PR #7](https://github.com/lse-ds105/group-project-git-it-done/pull/7): on the rolling 3-month mean approach to rate-cycle labelling (confirming it over a raw month-to-month diff) and on the Section 7 validation checks before the database write. Left comments on Joel's [PR #13](https://github.com/lse-ds105/group-project-git-it-done/pull/13) on the `INNER JOIN` logic, the annualisation formula, and the neutral exclusion from the bar chart. On [PR #19](https://github.com/lse-ds105/group-project-git-it-done/pull/19) I commented on three separate decisions: the heatmap `zmax` cap at 15% (without it the neutral column dominates the colour gradient and the hiking/cutting contrast disappears), the PNG fallback inside `<noscript>` (makes the figures folder do double duty as both notebook output and accessibility backup), and the crisis annotation labels on the Fed Funds timeline (what makes the cutting-cycle caveat readable without a separate footnote).

---

## Learning Integration

**From MP1: paths and AI constraints.**

The MP1 marker flagged that both notebooks used absolute Nuvolos paths, copy-pasted from the file browser without thinking about portability. Every file operation in this project's NB01 uses a relative path (`data/raw/fedfunds.json`, `data/raw/av_XLK.json`), and the README Reproduction section exists specifically so the paths work for anyone who clones the repo, not just us in Nuvolos. The second MP1 note was on AI usage: *"What the log doesn't show is you telling Claude what constraints matter for this course."* For this project I stated the constraints upfront before asking for any code help, `requests` only, no wrappers, `time.sleep()` for rate limiting, `load_dotenv()` for keys, relative paths throughout. The direct-push to `main` on the first NB01 commit is honest evidence the branch-and-PR habit wasn't fully formed at the start of the project; the PR workflow on everything afterwards is evidence it was by the end.

**From MP2: parameters as variables, and not overclaiming.**

MP2 noted that the collection date and key parameters were buried inside URLs rather than declared at the top of the notebook where they're visible and changeable without hunting through code. In NB01, all parameters, start date, end date, FRED series IDs, the ETF ticker list, are declared as named variables in an early cell before any request is made. The same instinct shaped the README: the Limitations section doesn't say "defensives outperform when the Fed cuts." It says the finding holds *conditional on the fact that every cutting cycle in this sample coincides with a recession*, and it links the Fed Funds timeline as the visual that makes that conditioning legible. MP2's overclaiming note came from an insight title that went further than the chart supported. Writing the README Limitations section, and in particular the "What this analysis does not claim" callout, was a direct application of that feedback.