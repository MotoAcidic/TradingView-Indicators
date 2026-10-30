# TradingView-Indicators — Technical Notes

## What this repo is

Pine Script (TradingView) indicators only — no Strategy/Indicator build pipeline, no SDK, nothing
to compile locally. Two kinds of file live here (see `README.md` for the catalog):

- **Operator-built, actively maintained** — personal trading tools, iterated on through real
  TradingView Pine Editor compile/runtime errors pasted back verbatim, never guessed blind.
- **Reference / third-party scripts** — credited to their original authors in their own header
  comments, kept here either as source material a `Quantower-storage` strategy was ported FROM, or
  as a reference the operator charts alongside their own tools. Don't edit these as if they were
  the operator's own code; if a change is needed, treat it as a deliberate fork and say so.

Split out of `Quantower-storage/TradingView/` into its own repo on 2026-10-29. If a path inside
`Quantower-storage/CLAUDE.md` still says `TradingView/...`, that now means this repo, not a
subfolder of Quantower-storage — see that file's own note on the move.

## House conventions for this repo

- **No compile-test available here.** Every script gets a first pass, then the operator pastes the
  real error (compile or runtime) back from the TradingView Pine Editor and it gets fixed one error
  at a time. Never claim a script "should work" — say plainly it hasn't been compile-tested yet.
- **Small, independent implementations.** Pine has no practical local-import mechanism for
  unpublished scripts, so swing/structure/zone detection logic is NOT shared between separate
  scripts here — each new indicator reimplements its own version fresh rather than depending on
  another script in this repo.
- **Scope real trading-signal indicators with the operator BEFORE writing code** — given the real
  financial stakes, use clarifying questions for anything that fires buy/sell signals or draws
  stop/target levels, the same way `SmcSignals.pine`/`IctSignals.pine` were scoped.
- **"Close, not a mere touch" discipline for structure breaks.** A level only counts as broken when
  `close` trades through it, not just a wick — a wick-only touch with no close confirmation is a
  bounce/rejection, not a break. This mirrors a hard-won lesson from the Quantower-side
  `srmAutoStrategy` history, where a wick-only touch with no close guard handed back a stop on the
  wrong side of entry.

## Pine v6 gotchas learned the hard way (this repo's own bug history)

- **`type` field defaults must be compile-time constants** (`na`, `false`, literals) — NOT function
  calls like `array.new_line()`. Array-typed UDT fields get no default in the `type` block at all;
  construct them explicitly in a `new...()` factory function instead (`DailyStructureBias.pine`'s
  `newTfState()` is the reference pattern).
- **`bool x = na` is stricter than `float x = na` / `int x = na`.** Pine's type inference treats a
  bool defaulted to `na` as `const bool`, and a later `:=` reassignment errors ("declared with const
  bool type"). Default bools to `false` instead wherever the `na`-ness is never actually relied on.
- **An `if`/`else` block's LAST statement determines an implicit type even when the result is
  discarded.** Mismatched types across branches (one ending in a `bool` assignment, the other in a
  `float` assignment) is a compile error — reorder so both branches end with the same-typed
  statement.
- **`label.new()`/`line.new()`/`box.new()` default to `xloc.bar_index`.** Passing a raw UNIX-ms
  `time` value as the X-coordinate without also setting `xloc = xloc.bar_time` throws a runtime
  error ("cannot be drawn further than 500 bars into the future") — Pine reads the timestamp as an
  absurd bar index. A line/label that starts at a historical TIME-based anchor and a line/label
  that starts at the CURRENT bar can legitimately use different xloc modes from each other; just
  keep x1/x2 (or a single x) internally consistent with whichever mode that object uses.
- **`extend.right` vs. a manually-tracked endpoint.** `extend.right` auto-extends to the current
  right edge every bar with no maintenance — simple, but draws a line all the way across the whole
  visible chart. For a short "stub" style instead (the operator's preferred look, matching
  `smc.pine`'s own PWL/PDL labels and `WicklessCandlesMTF.pine`'s abbreviated level tags), use
  `extend.none` and update `x2` yourself every bar to a near-future point. For a line anchored with
  `xloc.bar_time`, that near-future point has to be a TIMESTAMP, not a bar offset — extrapolate it
  the same way `smc.pine`'s `drawLevels()` does: `time + N * (time - time[1])`, guarding the very
  first bar where `time[1]` is `na`.
- **A `for i = 0 to array.size(arr) - 1` loop still runs once when the array is empty.** When
  `array.size(arr)` is `0`, the bound is `-1`, and Pine's direction auto-detection (active whenever
  no explicit `by` step is given) runs the loop backward from `0` to `-1` instead of skipping it —
  it does NOT simply skip like a normal "from > to" loop would in most languages. An explicit
  `by 1` step did not fix this in practice; guard with `if array.size(arr) > 0` before the loop
  instead.
- **UDT (`type`) instances and the arrays stored in their fields are reference types.** A function
  that takes a UDT as a parameter and mutates its fields (`s.someField := value`,
  `array.push(s.someArray, x)`) DOES affect the caller's object — this is the pattern
  `DailyStructureBias.pine` uses to give each of its six timeframes independent persistent state
  without Pine's usual per-call-site `var`-inside-function pattern. Because these functions take
  their state as a parameter rather than declaring their own `var`/`varip`, they can safely be
  called conditionally (`if showX ... = someFunc(...)`) instead of needing to run unconditionally
  every bar the way a function with its own internal `var` state would.
- **Self-aggregating a higher timeframe from the chart's own native bars** (the alternative to
  `request.security()`, adopted after `request.security()` proved unreliable at minute-level
  granularity in `WicklessCandlesMTF.pine`'s own bug history) only works when the chart's own
  native timeframe is <= the target period — you cannot synthesize 5-minute bars out of 15-minute
  native ones. Guard with `timeframe.in_seconds() > targetPeriodSeconds` and skip that
  timeframe entirely (not compute it incorrectly) when the chart is too coarse.
- **Two different self-aggregation boundary rules, not one.** A period that evenly divides a day
  (5m/15m/1H/4H) can safely use a flat UTC-ms modulus (`time - time % periodMs`). A DAILY boundary
  cannot — a calendar day does not divide evenly in raw UTC milliseconds once DST transitions are
  involved, and the real session boundary (e.g. CME/Globex's ~6pm ET open) doesn't align to UTC
  midnight anyway. Use real timezone-aware `year()`/`month()`/`dayofmonth()`/`hour()` built-ins
  anchored at the session start hour instead, shifting the raw timestamp forward 24h when past that
  hour (so Pine's own built-ins handle month/year rollover rather than hand-rolled calendar math).
- **`ta.pivothigh()`/`ta.pivotlow()` only count NATIVE bars for their left/right window**, even when
  given an explicit `source` series — they cannot be pointed at a self-aggregated higher-timeframe
  series and expected to count PERIODS on each side. Pivot/fractal detection on self-aggregated bars
  has to be hand-rolled: keep a bounded array of finalized period highs/lows, and check a candidate
  once enough NEWER finalized periods exist on both sides of it (same confirmation-lag rule used
  throughout the Quantower-side C# swing-level engines).

## Cross-references

- `Quantower-storage/CLAUDE.md` — the authoritative doc for Quantower C# strategies/indicators,
  several of which were ported from scripts in this repo (`ocean.pine` → `oceansStackStrategy`,
  `untestedLevels.pine` → the session-levels feature, `Trendlines.pine` +
  `keltnerChannel.pine` + `supportResistanceChannels.pine` → a three-indicator confluence strategy,
  `emaCross.pine` → `emaCrossStrategy`'s "TV Match" exit mode). Check there for the C# side of any
  of these.
