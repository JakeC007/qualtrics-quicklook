# Qualtrics QuickLook

A set of three Jupyter notebooks that take a raw Qualtrics export, flag the
low-quality responses, and show you what people answered. No coding required.

## What this is for

Qualtrics gives you a CSV that is hard to use directly. It has three stacked
header rows, one question is split across many columns, and the columns are
named `Q2`, `Q83`, `Q3_1` rather than anything readable. Before you can look at
your results you also have to throw out the junk: people who clicked through in
40 seconds, bots, and respondents who picked the same answer down every grid.

QuickLook does both. You run three notebooks in order and you get back a
readable codebook, a quality report telling you which responses to drop and
why, and plain frequency tables for the questions you care about.

## Who it's for

People analyzing their own Qualtrics survey who have little or no Python.
If you can install Python, open a Jupyter notebook, and press Shift+Enter, you
can use this. You never edit code. The one file you touch is a small settings
file (`config.json`), and you edit it by reading question text from a codebook
the tool generates for you, not by memorizing question numbers.

It runs on any operating system. Everything uses relative paths; there are no
Windows, Mac, or Linux specifics anywhere.

## How to use it

1. **Install.** Get Python 3, then from this folder run:
   `pip install -r requirements.txt`
2. **Add your data.** Export your survey from Qualtrics as a CSV and drop it
   into `sample_data/`. (A synthetic example is already there so you can try
   the tool first.)
3. **Start Jupyter.** Run `jupyter notebook` (or `jupyter lab`) and open the
   `notebooks/` folder.
4. **Run `01_ingest.ipynb`.** Click into the first cell and press Shift+Enter
   through to the end. It creates three files in `output/`:
   - `codebook.csv` - every question number next to its full text
   - `config.json` - your settings, with friendly names suggested for you
   - `responses_clean.csv` - the data the next notebooks read
5. **Edit `output/config.json` once.** Open `codebook.csv` alongside it. Under
   `"questions"` you will see entries like `"age": "Q2"`. Rename the left side
   to whatever you want to call each question, and keep on the right side the
   question number it maps to. Delete the questions you don't care about. If
   your survey has an attention check, add it under `"attention_checks"` as
   `{"Q6": "Strongly agree"}` using the answer that proves the person read it.
6. **Run `02_quality_check.ipynb`.** This writes `output/quality_report.csv`,
   one row per respondent, a column for each warning sign, and an
   `exclude_recommended` column. It deletes nothing; you stay in control.
7. **Run `03_explore.ipynb`.** This prints a table for each question you named.
   By default it first drops the responses the quality check recommended
   excluding.

That is the whole workflow. Edit `config.json`, rerun the notebooks any time.

### Using your own survey instead of the sample

The repo ships with a `config.json` set up for the example data. When you
switch to your own export, **delete `output/config.json` and rerun
`01_ingest.ipynb`** so it regenerates a config that matches your questions.
Notebook 1 never overwrites an existing `config.json`, so your edits are safe
across reruns.

## What the quality check looks for

All cutoffs live in `config.json` under `"quality"`, so you can tune them
without touching code. The four independent signals are:

- **Speeding** - the whole survey finished faster than a floor you set, or too
  many pages were submitted faster than a person could read them.
- **Low reCAPTCHA** - Qualtrics' own bot score is below your cutoff.
- **Attention** - the respondent failed an instructed-response check, counted
  only for the people who were actually shown one.
- **Straightlining** - the same answer given down a whole matrix grid. Short
  grids are ignored, because a uniform answer across four items is plausible.

A response is recommended for exclusion when at least two independent signals
trip (the count is configurable). One lone signal is treated as noise rather
than grounds for removal, so you keep borderline-but-real respondents.

## Why mapping by question number

Typing a full question label into a settings file is error-prone and breaks the
moment Qualtrics rewords a question. Numbers are stable. The catch is that a
bare `Q83` means nothing on its own, so notebook 1 writes a codebook that puts
the question text right next to every number. You read the codebook, you write
the numbers. That keeps the config short and stable without asking you to
remember what any code means.

## What's not done yet

**Visualization.** Notebook 3 produces tables only, no charts. That was a
deliberate stopping point, not an oversight. The natural next step is a
`04_visualize.ipynb` that turns the same per-question summaries into bar charts
for categorical and matrix questions and histograms for numeric ones. The data
it would need is already computed in notebook 3; what's missing is a plotting
layer (for example matplotlib) and a small amount of config for choosing which
questions to chart. If you pick this up, build it on top of
`responses_clean.csv` and `config.json` so it stays consistent with the rest.

Other things intentionally left out: weighting, significance testing, and
open-text coding. This tool is for a first clean look at a survey, not full
analysis.

## What's in the repo

```
qualtrics-quicklook/
  notebooks/
    01_ingest.ipynb          load export, build codebook + config + clean data
    02_quality_check.ipynb   flag low-quality responses
    03_explore.ipynb         frequency tables per question
  sample_data/
    sample_qualtrics_export.csv   synthetic data so the tool runs immediately
  output/                    everything the notebooks generate (config.json ships here)
  requirements.txt
  README.md
  LICENSE
```

Each notebook is self-contained: it carries its own setup and reads from disk,
so you can open any one of them on its own without hidden state.
