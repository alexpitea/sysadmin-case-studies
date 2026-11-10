# Halving the CPU use of a Python service on a small ARM board

## Context

A building-automation controller runs on a small ARM board. One Python process does most of the work: it polls devices on a serial bus, runs the control logic and serves an HTTP API that the configuration app and the phone app call.

That process was using between 52% and 65% of one core while nothing in particular was happening, with spikes close to 100%. The same process has to stay responsive to the bus.

## Measuring before guessing

It would have been easy to assume that the bus polling was the expensive part. Instead of acting on an assumption I attached a sampling profiler (`py-spy`, built for ARM and run from a temporary folder, so nothing was installed on the board) to the running process and let it collect for a few minutes of normal operation.

The profile said something else:

| Where the time went | Share |
|---|---|
| Building and serialising the HTTP status response | about 41% |
| Control logic | about 30% |
| Bus polling | about 10% |

The status endpoint was the problem. Each request rebuilt the whole state of the building from scratch and serialised it to JSON in pure Python, which took around 1.5 seconds of processor time. And the configuration app asked for it every few seconds, whether anyone was looking or not.

## What changed

1. **Made the status cheaper to produce.** Parts of the response that had not changed were no longer rebuilt, and a legacy copy of the state that was computed on every request, only to be thrown away, was removed. A request went from about 1.5 s to about 0.9 s of processor time.
2. **Asked less often.** The app's polling interval was raised to 15 seconds. Nothing on that screen needs fresher data than that.
3. **Removed work from hot loops.** A checksum that was computed bit by bit in Python was replaced with a table lookup, and a group of outputs that had been written one by one was written as a block.

## Result

The process went from 52-65% of a core to about 30%, with the peaks down from 98% to 76%.

One number in the app still read "100%" afterwards. That turned out to be an estimate drawn by the app itself, not a measurement from the board, and was fixed separately. It was a useful reminder to check where a number comes from before chasing it.

## What I took from it

- Profile the running system, not the code as you imagine it. The obvious suspect used 10% of the time.
- The cheapest request is the one that is not made. Changing a polling interval is a small change with a large effect.
- A sampling profiler that attaches from outside is ideal for a production device: no code change, no restart, and it can be removed without a trace.
- Measure again after each change, on the device, under normal load.
