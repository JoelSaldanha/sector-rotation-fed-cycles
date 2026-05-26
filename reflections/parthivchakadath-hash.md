# Individual Reflection

## Evidence of Individual Contribution

My contribution was building NB02 — the cleaning and storage notebook — and polishing it after Joel had built downstream work on top of it.

### Primary contribution

[PR #7 — Draft NB02: data transformation and SQLite storage](https://github.com/lse-ds105/group-project-git-it-done/pull/7), opened from my branch `parthiv/nb02-draft` and merged into `main` (merge commit `cd7b152`).

This PR delivered NB02 and a SQLite database with two normalised tables linked by foreign key on date. The technical decisions that I made in the notebook can be summarised as follows:

- Decisions block at the top of the notebook documents seven design choices upfront. This includes choices on inputs/outputs, schema, the rate cycle rule, date reconciliation, FK integrity, data gaps and vectorisation. In this section, I mimicked Hugo's NB01 structure and style so that the notebooks read cohesively as one team's work rather than three separate approaches to one issue.

- Two normalised tables linked by foreign key rather than a single wide merged frame. Each fact was stored once per the relational principle introduced in W10.

- Rate cycle labelling via `fed_funds.diff().rolling(3).mean()` and `np.select()`. After discussion in the group chat, we agreed that using this method to identify hiking and cutting periods would be the most effective. We chose `np.select()` over `.apply(if/elif)` for vectorisation and decided to use a rolling mean over raw diff since it would create too much noise in the data. We also opted to not use hardcoded FOMC dates because they would introduce a third data dependency that we had not collected.

- Date convention reconciliation between FRED (start of month) and Alpha Vantage (end of month trading day) using monthly-period conversion. At first, the merge silently produced zero matches but I was able to catch it after noting the suspiciously low row count. After that I added row-count prints after every merge in the notebook in order to avoid similar issues from reoccurring.

- Dual-layer FK integrity. I used a pandas filter that drops orphan rows before write, plus `PRAGMA foreign_keys = ON` so the database rejects orphan inserts itself. SQLite ignores FK declarations by default, so declaring them in the schema is documentation rather than enforcement unless the PRAGMA is set.

- Section 11 SQL demonstrations for NB03. I used four queries that were used during the lectures in order to verify whether the database was working adequately as well as give Joel queries that he could lift straight into NB03.

### Follow-up: post-merge polish PR

[PR #16 — Polish NB02 post-merge: FutureWarning fix and Issue #9 cross-reference](https://github.com/lse-ds105/group-project-git-it-done/pull/16) (commit `64ae6ab`), opened, reviewed by Joel, and merged by me.

This pull request worked on two small follow-ups after NB03 and the website had already been built on top of NB02. The changes can be summarised as:

- Wrapped `.isin([...])` in `pd.to_datetime()` to silence a pandas FutureWarning. Joel flagged it in a PR comment and separately opened [PR #8](https://github.com/lse-ds105/group-project-git-it-done/pull/8) with the same fix. I closed his PR as superseded by #16 since #16 had both his fix and my cross-reference.

- Added a cross-reference to [Issue #9](https://github.com/lse-ds105/group-project-git-it-done/issues/9) in Decision 3. Joel had identified during NB03 plotting that the rate-cycle rule produces noise-driven label flips in flat rate periods such as 2010–2015 and 2021–2022. I weighed up the costs and benefits of fixing this issue a day before submission and decided to not make any code changes. If I had made any changes to the code, we would have had to rerun NB03, update the website's finding percentages and reword that entire noise discussion that Joel had noted in both his notebook and the website. The only real benefit of this change would have been a cosmetic improvement to the labels that wouldn't change the true per-cycle averages NB03 actually reports on. I therefore closed Issue #9 as a documented known limitation instead of implementing one of the suggested fixes. The limitation is now visible from the closed Issue, the Decisions block at the top of NB02, NB03's Section 6 and the website's Finding 3.

## Learning Integration

The most influential feedback from MP1 that I applied here was the comments regarding my failure to notice that some of the data collected contained unreasonably low numbers such as -9999. Because I had never checked or dealt with that before aggregation, parts of my averages and plots became distorted, lowering the credibility of my findings. Before receiving this feedback, I'd treated null-checks and missing value detection as something that you would include at the end of a pipeline rather than something that you should do throughout it. In NB02, I have made sure to revise this and include null-checks throughout. I added null counts after every JSON file load in Sections 3 and 5, ran a dedicated validation block in Section 7 before writing anything to the database, and built a separate scratch cell to inspect which specific months had which nulls. Through these checks I was able to catch the October 2025 CPI gap and the April 2026 publication lag issue and flag them. This prevented these gaps from silently distorting the per-cycle averages in NB03. This feedback also influenced Decision 5's dual-layer integrity check, where I checked for orphan rows before the database write rather than relying on the schema to catch them at insert time.

The other piece of feedback that I tried to apply came from MP2's Technical Implementation comments, where the marker noted that there were several issues with my README which incorrectly stated what variables someone would need to update in order to run the code. The true issue was that my documentation described one thing while my code did something else. I attempted to therefore improve my documentation within this project by clearly describing my decisions, how I planned to implement the code and the limitations that exist within my code. For example in NB02, Decision 1 names the exact paths the notebook reads from and writes to. Decision 5 documents the foreign-key integrity guarantees explicitly. Decision 6 specifically names the months with null values rather than giving vague descriptions. The aim was that anyone reading NB02 would know exactly what the code does and where it can fail, without having to read the implementation to find out.
