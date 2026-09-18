# hulab-apps-assets

Public sample files for the CADA demo in
[hulab-apps](https://github.com/hulab-emory/hulab-apps-vite-migration) — the
unauthenticated walkthrough at `/cada/demo`, where each project template opens
its real annotator UI against a fixture instead of live data.

**Everything here is public.** It is served without a session, so nothing
patient-identifiable, internal or licence-restricted belongs in this repo.

## Layout

```
afib/            test1–test6.json    pre-decoded waveforms
notes/           0–2.json            clinical notes + one LLM answer each
pdf/             sample.pdf, 0.json  a document and its extraction sidecar
pub-llm-eval/    three papers        PDFs for the publication-review template
crc_eval/  pdf/  two source PDFs     coordinator-training review
           result/  one JSON each    questions + per-model answers
```

## Formats

These are consumed directly by the browser, so the shapes are fixed by the
client rather than by any service.

**`afib/*.json` — waveforms.** `{ StartTime, OffsetInSec, TicksPerSec,
SamplesPerChannel, WaveformData }`. **Pre-decoded on purpose:** the real
endpoint (`/api/cada/file/waveform`) shells out to Python to read `.adibin`, and
a static file mount cannot. Adding a waveform means decoding it first — dropping
a raw `.adibin` here will not work.

**`notes/*.json` — notes.** `{ index, text | note, trigger_word, concept,
gpt_answer }`. The body key is **`text` on some files and `note` on others**;
both are read, so don't normalise one into the other expecting a fix. A
`trigger_word` that doesn't appear in its text is a real case (`0.json`) — the
prediction is still listed for review, just not highlighted.

**`pdf/0.json` — extraction sidecar.** `{ filePath, info: [...] }`, found by
swapping the PDF's extension for `.json`. `info` is model output whose keys vary
per pipeline; the client walks it generically rather than assuming a vocabulary.

**`crc_eval/` — coordinator-training review.** Split across two folders, matching
the bucket layout the app already expects: `result/<name>.json` holds the
questions, and its `file_name` names a PDF in the **sibling `pdf/` folder**, not
its own. Each question carries `LLMs: { "<model>": { response, FKG } }`.

**The last two questions are load-bearing.** The reviewer UI renders them with a
different form, side by side against `info[1]` — they are the translations of
that answer. Both files here run: summarize → summarize as bullets → translate
to Chinese → translate to Spanish, and the translations carry `FKG: null`
because the readability score is meaningless for them. A file with fewer than
three questions breaks that layout.

The source PDFs are NCI PDQ patient-information summaries (acupuncture; anxiety
and distress in cancer) saved from cancer.gov. PDQ summaries are US Government
works and NCI states their text may be used freely.

**`pub-llm-eval/*.pdf`.** The filenames contain literal `%2F` (an encoded `/`
in the DOI) and that is **part of the filename, not encoding**. Anything
building a URL must encode per path segment, or the server decodes `%2F` back
to a slash and the file 404s. Don't "clean up" these names.

## Adding a file

1. Confirm it is safe to publish — this repo is world-readable.
2. Put it in the folder for its template, pre-decoded if it's a waveform.
3. Commit here; consumers pick it up however they reference this repo.
