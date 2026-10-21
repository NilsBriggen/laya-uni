# laya-uni

A small decision model that picks the **university module a downloaded file belongs to**, from the file's
name and the start of its text. It is used by [File Sorter](https://github.com/bobanes1/file-sorter),
which runs it on the student's own computer: nothing is sent anywhere.

laya-uni is a fine-tune of the multilingual checkpoint of
[**Laya** by Convai Innovations](https://huggingface.co/convaiinnovations/laya) (`multilingual/`, an mmBERT-base
encoder with a decision head, 322M parameters). It was trained for degrees of **every faculty**, not just
computer science, in German, English, French and Italian.

## Download

The model is in the [releases](../../releases), one release per model. The newest is **laya-uni-2**:

| file | bytes | SHA-256 |
|---|---|---|
| `model.safetensors` | 643,835,524 | `2b48bd423d1ed84f1745f73f8964588997f771756f1e75481197d46681df9f90` |
| `tokenizer.json` | 34,363,188 | `609d8f4c067cd3950f88594c5a802616cea245823836ef5848ee4fc40aab5b6f` |
| `rl_agent_config.json` | 4,965 | `15c84ac6f5339ddbb6e7cfed4f996e70116bfecbf9e3e4344865eec139c93a66` |
| `encoder-config.json` | 1,939 | `36b7d4898274ed6f06ce9c1f0e34034d8bd39295ab232d7cabc41a3eac821e5f` |

plus `manifest.json`. The layout is flattened. To load it with the `laya` Python package, put the files back as
`model.safetensors`, `rl_agent_config.json`, `encoder/config.json` and `tokenizer/tokenizer.json`.
The same model is on Hugging Face as [NilsBriggen/laya-uni](https://huggingface.co/NilsBriggen/laya-uni), in Laya's own layout.
File Sorter has a native Rust port that gives the same probabilities to four decimals.

## Updates

File Sorter reads `releases/latest/download/manifest.json` from this repository once a day. When the
newest release has a higher `version` than the model it has, it downloads that release's files, checks
their SHA-256, switches over, and then deletes the old model. Each model is therefore its own release,
and **published files are never replaced or deleted**. The manifest also carries the model's confidence
thresholds (`min_probability`, `min_lead`), which are chosen per model.

## How it is asked

There is one `choice` question per file. The instruction is "Which university module does this file belong to?"
The options are the student's modules, written as `Module` or `Module: keyword, keyword`. The state is
`File name: <name>\n\n<text>`, with the first 6000 characters of the extracted text.

## Training (laya-uni-2)

- **Synthetic data:** about 2,550 synthetic first pages (slides, exercise sheets, solutions, script chapters,
  readings, lab protocols, cases, exam prep, and "nothing fits" admin documents) for 54 degree programs of
  every faculty, about 340 of them in another language than their program. These became 5,088 questions.
- **Real course files:** 4,930 openly licensed course PDFs, collected (not generated) from Wikibooks,
  Wikiversité, HAL, Zenodo, MIT OpenCourseWare, German, Swiss and Austrian OER repositories and university
  course pages. They cover 13 faculties: 930 in mathematics, 1,800 in English, 1,700 in German, 800 in French
  and 620 in Italian. Each file became one question, with its program's other modules as options. Repository
  boilerplate (HAL cover pages, wiki headers and footers) was removed, since a student's own files never
  carry it. None of these files come from a course in the real-PDF test set.
- **Questions:** 4–12 options each, worded with and without keywords, codes and program prefixes. About 5 %
  have no right answer and a uniform target.
- **Split:** 8,218 questions for training and 944 for validation. Whole synthetic programs were held out for
  the test, one per faculty group, so the test measures degrees the model never saw. The real files only
  went to training and validation.
- **Run:** `laya.train.finetune` (laya 0.4.12) ran for 3 epochs with an effective batch of 64, shuffled
  option order, 1024 / 256 tokens, and the default RLCD objective, on one RTX 5090. The choice temperature
  was then fitted at 4.29.

The data lists, the scripts and the evaluation are in File Sorter's repository under `training/laya-uni/`.

## Results

**Real course PDFs** (240 openly licensed PDFs from 13 faculties, never trained on):

| | Laya multilingual | laya-uni-1 | laya-uni-2 |
|---|---|---|---|
| right module ranked first | 59.6 % | 78.8 % | **85.0 %** |
| sorted at p ≥ 0.6 and 2× the next | 60.4 % | 62.5 % | **81.7 %** |
| precision of those | 75.9 % | 94.7 % | 93.9 % |
| mathematics (15 files), ranked first | 46.7 % | 46.7 % | **66.7 %** |

**Unseen programs** (855 synthetic files from 9 held-out degrees):

| | Laya multilingual | laya-uni-1 | laya-uni-2 |
|---|---|---|---|
| right module ranked first | 57.5 % | 81.8 % | **84.1 %** |
| sorted at p ≥ 0.6 and 2× the next | 49.4 % | 76.5 % | 78.6 % |
| precision of those | 77.6 % | 93.2 % | 93.0 % |

laya-uni-2's manifest sets p ≥ 0.75 with 2× the next. On validation that sorts 79 % of files at 95 %
precision, and on the real PDFs 76 % at 94.5 %. Files below it go to File Sorter's keyword matching.

## Limits

- **French:** the held-out test has only 16 French files, and they scored 56 %, while French validation files
  score 91 %. Treat French as less tested.
- **Formula-heavy PDFs:** mathematics improved most but is still the weakest faculty on real files.
- **Textbook-heavy real data:** about half of the real training files are Wikibooks or Wikiversité chapters,
  so lecture slides and exams are less represented than in a student's folder.
- Use the confidence threshold and keep a fallback.

## License and credits

Apache-2.0, like the model it is derived from: [convaiinnovations/laya](https://huggingface.co/convaiinnovations/laya)
by Convai Innovations (Apache-2.0), built on [jhu-clsp/mmBERT-base](https://huggingface.co/jhu-clsp/mmBERT-base) (MIT).
laya-uni changes the weights by fine-tuning and the calibration temperature. The tokenizer is unchanged.
Some of the course files it was trained on are licensed CC BY-NC-SA (MIT OpenCourseWare, Exo7 and others);
the model contains no copy of them.
