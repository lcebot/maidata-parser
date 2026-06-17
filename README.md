# maidata-parser

A fast, strict simai chart parser that converts maidata notation to `.ma2` format.

**Download the latest binary from [Releases](https://github.com/lcebot/maidata-parser/releases/latest).**

---

## Overview

maidata-parser takes a `maidata.txt` with metadata scheme, or a plain-text simai chart (the `inote` content inside a `maidata.txt`), and outputs a fully-formed `.ma2` file ready for use, along with a console summary of note counts, timing metadata, and chart properties.

The parser faithfully implements the simai specification including BPM change compensation for holds and slides, pseudo-each (\`) timing offsets, equivalent-BPM slide scaling, preview marker (`###`) extraction, and playability validation.

---

## Download

Pre-built binaries are published on the [Releases](https://github.com/lcebot/maidata-parser/releases/latest) page.

---

## Usage

```
maidata_parser -i <input> [options]
```

The input file can be a full maidata.txt containing `&inote_` and `&answer_` tags, or a raw text file containing only the body of an inote field. If a full maidata.txt is provided, the parser will automatically extract the chart corresponding to the specified difficulty ID.

**Example**

```bash
# Parse and write to output.ma2 (default)
./maidata_parser -i chart.txt

# Specify output path and difficulty ID
./maidata_parser -i chart.txt -o master.ma2 -d 5

# Quiet mode: print stats only, skip ma2 body to stdout
./maidata_parser -i chart.txt -q

# Parse a full maidata.txt directly for Master difficulty default ID 5
./maidata_parser -i maidata.txt

# Parse a full maidata.txt for Re:Master difficulty
./maidata_parser -i maidata.txt -o remaster.ma2 -d 6
```

**Options**

| Flag                   | Default       | Description                                               |
|------------------------|---------------|-----------------------------------------------------------|
| `-i`, `--input`        | *(required)*  | Path to the input maidata.txt or raw simai chart text file |
| `-o`, `--output`       | `output.ma2`  | Path for the output `.ma2` file |
| `-d`, `--difficulty`   | `5`           | Difficulty slot ID (1 Easy · 2 Basic · 3 Adv · 4 Exp · 5 Master · 6 Re:Master) |
| `-a`, `--answer-count` | `4`           | Answer sound count |
| `-q`, `--quiet`        | off           | Suppress `.ma2` body output to stdout; print stats only |

> **Difficulty and touch size** — IDs 2 and 3 (Basic / Advanced) emit large-size touch notes; all other IDs use normal ones.

---

## Console Output

On success, the parser prints a summary:

```
Resolution:     ...
Note count:     ...
Preview window: (..., ...)
is_DX:          ...

Time elapsed:               ... ms
Time consumption per note:  ... μs

Output written to: output.ma2
```

| Field            | Meaning                                                                  |
|------------------|--------------------------------------------------------------------------|
| `Resolution`     | Computed tick resolution (384–1920)                                      |
| `Note count`     | Total notes in the exported chart                                        |
| `Preview window` | `(start_ms, end_ms)` from `###` markers, or auto-derived 20 s window     |
| `is_DX`          | `True` if the chart uses any DX-exclusive note types or connected slides |

---

## Validation

The parser enforces playability constraints and will exit with an error if they are violated:

- **Sensor conflicts** — two notes occupying the same judgment region while one is still active (e.g. a tap landing inside an ongoing hold on the same button)
- **Sensor overcommit** — more than two simultaneous inputs required (touch, touch hold and slide are counted as one hand collectively)
- **Slide geometry** — invalid start/end combinations for straight, curve, V-shape, and opposite-side patterns

Error messages include the measure position and the raw simai fragment that triggered the conflict for easy tracing.

---

## Notes

- This repository currently distributes compiled binaries only.
- The binary is self-contained and has no runtime dependencies.
