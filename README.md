# media-tools

Standalone command-line scripts for audio/video conversion and tracklist cleanup. Everything lives in `bin/`; put that directory on your `PATH` (`mp3320_all` calls `mp3320` by bare name).

```sh
export PATH="$PATH:/path/to/media-tools/bin"
```

**Requirements:** `ffmpeg` built with `libmp3lame` and `libx264`; Python 3 (stdlib only) for `rbtracklist`.

## Audio

### `mp3320 <file>`
Converts one audio file to a 320k MP3 next to the original, keeping metadata (ID3v2.3).

```sh
mp3320 song.wav        # writes song.mp3
```

### `mp3128 <file>`
Same as `mp3320`, but at 128k.

```sh
mp3128 song.wav        # writes song.mp3
```

### `mp3320_all`
Recursively finds `.wav`, `.aif` and `.aiff` files under the current directory and runs `mp3320` on each. Files whose `.mp3` already exists are skipped.

```sh
cd ~/Music/exports && mp3320_all
```

## Video

### `vidmp3 <video>`
Extracts the audio of a video as a 320k MP3 next to the original.

```sh
vidmp3 set.mp4         # writes set.mp3
```

### `mp4shrink <in> [out] [crf=28] [height=720]`
Re-encodes a video as H.264/AAC at a reduced height. The output defaults to `<in>_compressed.mp4`.

```sh
mp4shrink talk.mp4                       # 720p, CRF 28
mp4shrink talk.mp4 small.mp4 24 480      # 480p, CRF 24
```

### `mp4clips [--copy] [--4x5] <video.mp4> <timestamps.txt>`
Cuts clips from a video, one per line of a timestamps file, into `<name>_clips/` next to the input.

Each line is `start end [label...]`. Times use ffmpeg syntax (`SS`, `M:SS`, `H:MM:SS`, optional `.ms`). The label becomes the filename; unlabeled lines are named `<name>_<NN>` by line number. Blank lines and `#` comments are ignored. All lines are validated before any encoding, and clips that already exist are skipped, so a rerun picks up where it stopped.

```text
# stamps.txt
0:01:20 0:02:05 intro
0:10:00 0:10:45
1:05:30 1:06:10 closing remarks
```

```sh
mp4clips talk.mp4 stamps.txt             # frame-accurate re-encode (x264 CRF 18, AAC 192k)
mp4clips --copy talk.mp4 stamps.txt      # fast stream copy; cuts snap to keyframes
mp4clips --4x5 talk.mp4 stamps.txt       # 1080x1350 with a blurred-background fill, for Instagram carousels
```

`--copy` and `--4x5` can't be combined, since resizing needs a re-encode.

## Text

### `rbtracklist <tracklist.txt>`
Reads a UTF-16LE tab-separated tracklist export (columns `#`, Artist, Track Title, Label, as exported by Rekordbox) and writes two UTF-8 files next to it:

- `<name>_label.txt`: `Artist - Title [Label]`
- `<name>_nolabel.txt`: `Artist - Title`

Rows whose first column isn't a number are skipped as headers or blanks.

```sh
rbtracklist mix.txt    # writes mix_label.txt and mix_nolabel.txt
```
