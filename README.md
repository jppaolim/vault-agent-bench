# 204 Runs, 100M Tokens: Which Agent Can Keep a Knowledge Base?

A personal benchmark: 30 models and 13 agent harnesses asked to update an Obsidian vault from one
real, noisy meeting transcript. 204 headless runs, Jul–Oct 2026, each graded on the same
scorecard (quality of the notes + judgment − penalties, out of 100).

- [`index.html`](index.html): the results.
- [`method.html`](method.html) (source: [`METHOD.md`](METHOD.md)): how runs were launched and
  graded, the problems met, and the limits.
- [`runs.csv`](runs.csv): every run, with its score breakdown, time and cost.

The vault and the transcript are private and are not published, so the benchmark cannot be rerun.
The data is here so the conclusions can be checked.

A static site with no build step: serve the folder as is.
