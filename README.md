# laya-uni

A small decision model that picks the **university module a downloaded file belongs to**, from the file's
name and the start of its text. It is used by [File Sorter](https://github.com/bobanes1/file-sorter),
which runs it on the student's own computer: nothing is sent anywhere.

laya-uni is a fine-tune of the multilingual checkpoint of
[**Laya** by Convai Innovations](https://huggingface.co/convaiinnovations/laya) (`multilingual/`, an mmBERT-base
encoder with a decision head, 322M parameters). It was trained for degrees of **every faculty**, not just
computer science, in German, English, French and Italian.

## Download

The model is in the [releases](../../releases). Release `laya-uni-1` has these files:

| file | bytes | SHA-256 |
|---|---|---|
| `model.safetensors` | 643,835,524 | `2a73a52757ff1557d59abcdb43db469327df077146ce68140292a943572925bb` |
| `tokenizer.json` | 34,363,188 | `609d8f4c067cd3950f88594c5a802616cea245823836ef5848ee4fc40aab5b6f` |
| `rl_agent_config.json` | 4,987 | `c06f9dff24ed9f1132e5e44020381d380d3b5ea30e650f90a82ff6378215c6ff` |
| `encoder-config.json` | 1,939 | `36b7d4898274ed6f06ce9c1f0e34034d8bd39295ab232d7cabc41a3eac821e5f` |

The layout is Laya's own. To load it with the `laya` Python package, put the files back as
`model.safetensors`, `rl_agent_config.json`, `encoder/config.json` and `tokenizer/tokenizer.json`.
File Sorter has a native Rust port that gives the same probabilities to four decimals.

## How it is asked

There is one `choice` question per file. The instruction is "Which university module does this file belong to?"
The options are the student's modules, written as `Module` or `Module: keyword, keyword`. The state is
`File name: <name>\n\n<text>`, with the first 6000 characters of the extracted text.

## Training

- **Data:** about 2,550 synthetic first pages (slides, exercise sheets, solutions, script chapters,
  readings, lab protocols, cases, exam prep, and "nothing fits" admin documents) for 54 degree programs.
  - **Faculties:** medicine and health, law and theology, humanities, social sciences and education,
    economics and business, natural sciences, mathematics, engineering and architecture, computer science,
    arts and agriculture, plus French- and Italian-language programs.
  - **Mixed language:** about 340 of them are in another language than their program, such as English papers
    in German degrees.
  - **Questions:** these became 5,087 questions with 4–15 options each. The options are the program's
    other modules, sometimes with a few from other programs, worded with and without keywords, codes and
    program prefixes. About 9 % have no right answer and a uniform target.
- **Split:** whole programs were held out for the test, one per faculty group, so the test measures degrees
  the model never saw.
- **Run:** `laya.train.finetune` (laya 0.4.12) ran for 3 epochs with an effective batch of 64, shuffled
  option order, 1024 / 256 tokens, and the default RLCD objective. The choice temperature was then fitted
  at 2.95. It took about 20 minutes on one RTX 4090.

The data, the scripts and the evaluation are in File Sorter's repository under `training/laya-uni/`.

## Results

**Unseen programs** (855 synthetic files from 9 held-out degrees):

| | Laya multilingual | laya-uni |
|---|---|---|
| right module ranked first | 57.5 % | **81.8 %** |
| sorted at p ≥ 0.6 and 2× the next | 49.4 % | 76.5 % |
| precision of those | 77.6 % | **93.2 %** |

**Real course PDFs** (240 openly licensed PDFs from 13 faculties, never trained on):

| | Laya multilingual | laya-uni |
|---|---|---|
| right module ranked first | 59.6 % | **78.8 %** |
| sorted at p ≥ 0.6 and 2× the next | 60.4 % | 62.5 % |
| precision of those | 75.9 % | **94.7 %** |

On validation, the threshold pair p ≥ 0.7 with 2× the next sorts 76 % of files at 96.5 % precision.
File Sorter uses that pair. Files below it go to File Sorter's keyword matching.

## Limits

- **Formula-heavy PDFs:** mathematics PDFs whose text comes out as formulas score no better than the base
  model (47 % on 15 real files).
- **French:** the held-out test had only 16 French files. They scored low, while French validation files
  scored 88 %. Treat French as less tested.
- **Synthetic training data:** real course material can look different. Use a confidence threshold and keep
  a fallback.

## License and credits

Apache-2.0, like the model it is derived from: [convaiinnovations/laya](https://huggingface.co/convaiinnovations/laya)
by Convai Innovations (Apache-2.0), built on [jhu-clsp/mmBERT-base](https://huggingface.co/jhu-clsp/mmBERT-base) (MIT).
laya-uni changes the weights by fine-tuning and the calibration temperature. The tokenizer is unchanged.
