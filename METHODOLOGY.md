# Methodology

Both tests were run on the same laptop:

- CPU: AMD Ryzen 7 7840U with Radeon 780M Graphics
- System: Linux 6.12.12-amd64 x86_64

Terminal versions:

| terminal | version | notes |
|---|---|---|
| alacritty | 0.15.1 | |
| ghostty | 1.4.0 | built from git tag `v1.4.0` with `zig build -p $HOME/.local -Doptimize=ReleaseFast`, following the official build-from-source instructions; it reports itself as `1.4.0-dev+0000000` |
| kitty | 0.50.0 | |
| zellij | 0.46.0 | run as its own window with `zellij terminal` |

## Output throughput

Measures how long a terminal takes to read and parse a large burst of output.

### The file

`log.txt` (8,388,705 bytes, 36,260 lines) imitates the output of a colored
logger. Every line has a dim timestamp, a log level colored by severity, a
`[worker-N]` tag in a 256-color shade, and colored `id=` and `path=` fields.
It also contains:

- tabs between fields, and paths that are clickable links (OSC 8 hyperlinks)
- Chinese, Japanese and Korean text
- emoji, including skin tones, flags and combined emoji such as families
- letters with one to three accent marks stacked on them, and Hebrew and Arabic
  words with vowel marks
- text styled with 16, 256 and 24-bit colors, backgrounds, italic, dim,
  strikethrough, and curly and dotted underlines with their own colors

### How the end of parsing is detected

A terminal can accept bytes faster than it parses them, so the time `cat` takes
to finish is not enough. Right after the file, the terminal is asked for its
cursor position (`ESC [ 6 n`). It can only answer after it has parsed
everything before the question, so the run ends when the answer arrives.

Each timed run executes this inside the terminal:

```bash
exec </dev/tty >/dev/tty
saved="$(stty -g)"
stty -echo -icanon
cat log.txt
printf '\033[6n'
IFS= read -r -s -d R _
stty "$saved"
```

The time therefore also includes starting `bash` and `cat`, which is the same
for every terminal.

### Terminal setup

Every terminal ran with its own configuration files turned off and its default
font and font size. The text area was 48 rows by 191 columns in every terminal.
They were started like this, where `CMD` is the timing command below:

```bash
alacritty --config-file /dev/null -o window.dimensions.columns=191 -o window.dimensions.lines=48 -e CMD
ghostty --config-default-files=false --gtk-single-instance=false --window-width=191 --window-height=48 -e CMD
kitty --config NONE -o initial_window_width=191c -o initial_window_height=48c -o remember_window_size=no CMD
ZELLIJ_CONFIG_FILE=config.kdl zellij terminal --standalone --startup-mode windowed -e CMD
```

zellij used this `config.kdl`. The window is 50 rows tall because zellij's tab
bar and status bar take 2 rows, which leaves a 48-row pane:

```kdl
show_startup_tips false
show_release_notes false
terminal {
    initial_columns 191
    initial_rows 50
}
```

Inside each window, the size was checked with `stty size` before timing; all
windows were 48 x 191.

### Timing

In each window, after waiting 1 second, the screen was reset and cleared
(`printf '\033[m\033[?1l\033[r\033[H\033[2J'`), then the run was timed with
hyperfine 1.19.0:

```bash
hyperfine -N --output=inherit --warmup 2 --runs 10 "bash sync-cat.sh log.txt"
```

`sync-cat.sh` holds the commands shown above. `--output=inherit` sends the
file to the terminal instead of discarding it, and `-N` runs the command
without an extra shell.

### Rounds

The whole test was repeated in 16 rounds. In each round every terminal was
opened in a new window, timed (2 warm-up runs, 10 timed runs) and closed. The
order moved by one place each round (round 1: alacritty, ghostty, kitty, zellij;
round 2: ghostty, kitty, zellij, alacritty; and so on), so each terminal ran in
each position 4 times. This spreads out slowdowns from heat or background
activity.

The results are the mean and standard deviation of all 160 timed runs per
terminal. Speed is the file size divided by the mean time, in MB/s
(1 MB = 1,000,000 bytes).

This test ran under Wayland (KDE Plasma).

## Typing latency

Measures the time from a keypress until the character appears on screen.

### Tool

[Typometer](https://github.com/pavelfatin/typometer) 1.0.1 (the latest
release), with its default settings:

- 200 keypresses per session
- 150 ms between keypresses
- no pauses
- synchronous mode: each keypress waits until the previous character has
  appeared
- native screen access (Xlib), which is faster and more accurate than the Java
  fallback

Typometer sends key events to the focused window and watches a single pixel of
the screen where the character should appear.

### Setup

- X11 with the openbox window manager and no compositor. Typometer recommends
  this, because compositing window managers add delay of their own.
- In every terminal: DejaVu Sans Mono at size 12, ligatures off, black text on
  a white background.

### Runs

Each terminal was measured in 3 separate sessions of 200 keypresses, 600 in
total. The results are the mean, standard deviation, median, minimum and
maximum of all 600 keypresses.

## Data

- `data/throughput-results.csv`: every timed throughput run (terminal, round,
  run number, seconds).
- `data/typing-latency-results.csv`: every Typometer keypress in milliseconds;
  one column per terminal and session, where `kitty2` is kitty's second
  session.
