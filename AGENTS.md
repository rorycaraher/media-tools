# AGENTS.md

This file provides guidance to AI Agents when working with code in this repository.

## Overview

A flat collection of standalone command-line scripts in `bin/` for audio/video conversion and tracklist cleanup. There is no build system, package manager, test suite, or linter. Each script is executable and self-contained; the directory is meant to be put on `PATH`, which `mp3320_all` relies on (it calls `mp3320` by bare name).

Dependencies: `ffmpeg` (with `libmp3lame`, `libx264`) for the shell scripts; Python 3 stdlib only for `rbtracklist`.

## Scripts

- `mp3320 <file>` / `mp3128 <file>` — convert one audio file to MP3 (320k / 128k) next to the original, preserving metadata (ID3v2.3). `mp3128` is a copy of `mp3320` with a different bitrate; keep the two in sync when changing shared behaviour (note `mp3128`'s header comment and usage string still say 320k/`mp3320`).
- `mp3320_all` — recursively finds `.wav/.aif/.aiff` under the cwd and runs `mp3320` on each, skipping files whose `.mp3` already exists. The `</dev/null` on the `mp3320` call is deliberate: ffmpeg would otherwise consume the `find` file list from stdin.
- `vidmp3 <video>` — extract audio from a video to a 320k MP3 (`-vn`).
- `mp4shrink <in> [out] [crf=28] [height=720]` — H.264/AAC re-encode at reduced height (`scale=-2:H`).
- `mp4clips [--copy] [--4x5] <video.mp4> <timestamps.txt>` — cuts clips into `<name>_clips/` next to the input. Each timestamps line is `start end [label...]` (ffmpeg time syntax; blank and `#` lines skipped; label becomes the filename, else `<name>_<NN>` by line number). All lines are validated (format, start < end, duplicate names) before any encoding; existing clips are skipped so reruns resume. Default is a frame-accurate libx264 CRF 18 / AAC 192k re-encode; `--copy` stream-copies (keyframe-snapped); `--4x5` exports 1080x1350 with the video fitted over a blurred, cropped copy of itself (incompatible with `--copy`).
- `rbtracklist <tracklist.txt>` — reads a UTF-16LE tab-separated export (columns: `#`, Artist, Track Title, Label — Rekordbox-style), writes `<name>_label.txt` (`Artist - Title [Label]`) and `<name>_nolabel.txt` (`Artist - Title`) alongside the input. Rows whose first column isn't numeric are skipped as headers/blank.

## Conventions

- Shell scripts: `#!/usr/bin/env bash`, `set -euo pipefail`, a one-line purpose comment, arg-count check with a `Usage:` message to stderr, and `exec ffmpeg ...` as the final command. Output path is always `"${input%.*}.mp3"` (same dir, same basename).
- Scripts are invoked directly (`bin/mp3320 song.wav`); there is no way to run a "single test" — verify by running the script on a sample file.
