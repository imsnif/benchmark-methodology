# Terminal benchmarks

Output throughput and typing latency of alacritty, ghostty, kitty and zellij,
measured on one laptop on 2026-11-09.

## Results

### Output throughput

Time for the terminal to read and parse `log.txt` (8 MB of colored log output
with CJK text and emoji), printed with `cat`. Mean ± standard deviation over
160 runs per terminal. Lower time is better.

| terminal | time | speed |
|---|---|---|
| alacritty 0.15.1 | 139 ± 8 ms | 60 MB/s |
| ghostty 1.4.0 | 347 ± 33 ms | 24 MB/s |
| kitty 0.50.0 | 132 ± 7 ms | 64 MB/s |
| zellij 0.46.0 | 183 ± 9 ms | 46 MB/s |

### Typing latency

Time from a keypress until the character appears on screen, measured with
Typometer. Mean ± standard deviation over 600 keypresses per terminal. Lower is
better.

| terminal | mean | median | min | max |
|---|---|---|---|---|
| alacritty 0.15.1 | 4.4 ± 0.4 ms | 4.4 ms | 3.0 ms | 5.6 ms |
| ghostty 1.4.0 | 12.2 ± 1.8 ms | 12.0 ms | 9.5 ms | 29.3 ms |
| kitty 0.50.0 | 16.6 ± 7.2 ms | 12.5 ms | 9.6 ms | 30.9 ms |
| zellij 0.46.0 | 4.4 ± 0.4 ms | 4.5 ms | 3.0 ms | 5.4 ms |

ghostty was built from git tag `v1.4.0` following the official
build-from-source instructions (`zig build -p $HOME/.local -Doptimize=ReleaseFast`).

## Files

| file | contents |
|---|---|
| `METHODOLOGY.md` | how both tests were run, step by step, so they can be repeated |
| `log.txt` | the 8 MB file used in the throughput test |
| `data/throughput-results.csv` | every throughput run: terminal, round, run number, time in seconds |
| `data/typing-latency-results.csv` | every Typometer keypress, in ms; one column per terminal and session |
