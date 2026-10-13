# Method and problems met

How the 204 runs behind [the results page](index.html) were produced and graded, what went wrong
along the way, and what the numbers can and cannot tell you. Every run is in [runs.csv](runs.csv).

The benchmark itself cannot be rerun by anyone else: the vault and the meeting are private. This
page is here so you can check how the conclusions were reached, not to reproduce them.

## The task

One real 46-minute meeting between two peers, recorded with the other participant's agreement,
transcribed automatically in French. The transcript is noisy: every line appears twice (the
microphone picked up both the voice and the speakers), speaker labels are unreliable, and names and acronyms
are mangled.

The agent receives a one-line prompt and works inside a copy of an Obsidian vault of about 200
files and 15K lines. The vault carries its own instructions (a 145-line guide) and a skill that
describes how to file insights. The agent has to search the vault itself.

A complete answer touches four places, each in the right folder with correct frontmatter, tags
and links:

- a new entry at the top of the Journal, without losing past entries;
- a meeting note;
- one or more insight notes, one idea per note;
- a profile of the other participant.

## Scoring (ScoreV2)

Each run gets a score out of 100: **quality of the notes (50) + judgment (50) − penalties**.
A hand-written reference answer scores 100. A score can go below zero.

| Part | What is checked | Points |
| --- | --- | --- |
| Right files | The four outputs exist, in the right place, with real content | 15 |
| Main content | Each idea of the framework explained during the call, captured accurately | 20 |
| House style | The vault guide's conventions: frontmatter, dated names, links, a takeaway in each insight, no copied text | 10 |
| Extras | Linking anecdotes to existing notes, concrete next steps | 5 |
| Who does what | Three follow-ups agreed in the call, each with the right sender (12 + 5 + 3) | 20 |
| Relationship | The participant described as a peer and connector | 10 |
| Biography | The participant's background, not the vault owner's; no invented roles | 15 |
| Restraint | The vault's reference files about the owner left untouched | 5 |
| Penalty: history lost | Existing content erased, such as past Journal entries | −10 to −40 |
| Penalty: invented biography | A false professional identity given to the participant | −10 to −20 |

The quality half rewards extraction. The judgment half is where models differ: who promised
what to whom, and whose background is whose. Both people share a similar background, which makes
the two biographies easy to blur.

### How the grading was done

- Every run was graded from its preserved output, file by file, against the transcript and the
  vault's previous state.
- A first pass was done by a model that did not know which model produced the run.
- Every judgment item and every penalty was then reviewed a second time against the complete
  output files, for every run with output.
- Each graded item cites the exact lines it relies on, and a script checks that those quotes
  exist in the output.
- Quality points were reviewed less strictly than judgment points. Treat differences of a few
  quality points as noise.

The first scoring system (ScoreV1) mostly rewarded extraction and missed several judgment
errors. It was retired; all 204 runs were regraded with ScoreV2, and only ScoreV2 appears here.

## How a run was launched

- Each run starts from the same clean copy of the vault, on its own Git branch, and is kept
  whatever happens: a failed or empty run is evidence too.
- The agent runs unattended (headless), launched from inside the vault, with the prompt passed
  through standard input and the model named explicitly.
- Before a new harness or provider was used for a numbered run, a separate pilot checked that it
  really found the vault's instructions and skill, and that its read, search, write and edit tools
  worked. Pilots are not numbered runs.
- The full run was timed. For self-hosted models, memory was sampled every second.
- After each run, the complete diff was checked: were the required files written, was the Journal
  preserved, were reference files left alone, did the agent stay inside the vault copy.

### Where the models ran

- **Self-hosted:** on my own machines, through a local inference server.
- **Cloud:** provider APIs, with zero data retention and no training set up.

### Harnesses

The harness is the agent program around the model: its system prompt, tools and limits. Tested:
Pi (with and without a custom system prompt), OpenCode 1 (classic), OpenCode 2, Goose,
little-coder, Junie, Hermes, OpenInterpreter, Kimi Code, Bionic, DSH and Chad. Most runs used
Pi and OpenCode.

## Problems met

Most of the effort went into making runs comparable, not into running them.

**The answer key leaked.** For a while, the notes describing the expected answer sat in an
instruction file that agents could see from the vault. 25 runs saw it. They are kept and tagged,
but excluded from the rankings. From then on, the benchmark's administration notes were hidden
during every launch and restored afterwards.

**Each harness discovers instructions differently.** One harness reads the vault guide, another
ignores it; a skill present on disk may never be loaded. Links that the vault relies on
disappeared in fresh copies because version control ignores them. The only reliable check was a
pilot run showing the instructions and skill actually being read.

**Tool schemas that lie.** Some harnesses advertised one tool format to the model and rejected
it when called. The model then spends the run fighting its own tools. This counts as a harness
failure, not a model failure.

**Context limits that disagree.** The inference server can enforce a smaller context than the
harness believes it has, so the harness never compacts its history and the run overflows. The
output allowance also eats into the context. Server and harness limits had to be aligned before
each self-hosted run.

**Effort settings are often only hints.** Asking for "high" reasoning does not mean the server
applies a real thinking budget. For self-hosted models, the setting that actually reached the
server mattered more than anything else: the same model scored about 65 at medium effort and
86–97 at extra-high. Extra-high can also run out of output room: one run stopped before writing
anything.

**Finishing is not the same as succeeding.** Harnesses exited cleanly after writing only some of
the files, or none. Some runs stalled, some looped. Every outcome was classified from the diff,
never from the exit status. Fourteen runs produced no output at all and score 0, marked
incomplete.

**One run proves little.** The same setup scored 93, 93, 57, 40 and 92 on five runs. Rankings
use the median run per model, and single-run results are flagged as such.

**Measuring memory on Apple silicon.** The usual process memory figure misses most of the GPU
allocation, so it badly understates usage. The server's own memory gauge was used instead, and
the sampler had to find the server again after every restart.

**Sandboxes that look like outages.** One coding agent's network sandbox silently blocked the
local inference server, which then looked as if it were down. The same request outside the
sandbox worked.

**Costs are not reported the same way.** Some providers report the cost per request, others only
token usage, which the harness converts with its own price list. Costs in the CSV are what was
reported or estimated for that run; missing costs are left blank, not guessed.

## Evidence tags

Every run carries one tag, also in the CSV:

| Tag | Meaning | Used in rankings |
| --- | --- | --- |
| `verified-clean` | Verified that the agent could not see the expected answer | Yes |
| `pre-answer-key` | Run before the answer notes existed | Yes |
| `answer-key-assisted` | The agent could see the expected answer | No |
| `instruction-validity-unknown` | Harness instruction loading was not checked | No |

Rankings also exclude incomplete runs.

## Limits

- One task, one meeting, in French. Results say how models handle this kind of work, not which
  model is best in general.
- Few runs per setup: many cloud configurations were run once or twice.
- Settings differ across runs (effort, output limits, sampling, harness versions). Two cells are
  compared only when those differences are stated.
- Cloud sampling settings and memory are not visible, and are recorded as unknown.

## The data: runs.csv

One row per run, 204 rows.

| Column | Meaning |
| --- | --- |
| `run` | Run number, in chronological order |
| `model` | Model, with quantization or packaging variant after "·" |
| `family` | Model family used for the rankings |
| `where` | `self-hosted` or `cloud` |
| `harness` | Agent program; OpenCode before run 92 is OpenCode 1, from run 92 OpenCode 2 unless named |
| `evidence` | Evidence tag, see above |
| `completed` | `no` if the run stalled, was aborted or produced no usable output |
| `ranked` | `yes` if counted in the rankings (completed and clean or pre-answer-key) |
| `local_pick` | `yes` for the runs of the recommended local setup (plain Qwen3.8 27B, 4-bit, extra-high, Pi) |
| `score` | ScoreV2 total |
| `quality`, `judgment` | The two halves, out of 50 each |
| `who_does_what`, `relationship`, `biography`, `restraint` | Judgment items, out of 20, 10, 15 and 5 |
| `penalty`, `penalty_detail` | Points subtracted and why |
| `seconds` | Wall time of the run, blank when not recorded |
| `cost_usd` | Reported or estimated cost of the run, blank when unknown or self-hosted |
