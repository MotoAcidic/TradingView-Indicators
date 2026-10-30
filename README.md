# TradingView Indicators

Pine Script (TradingView) indicators — personal trading tools plus reference scripts that some of
the operator's [Quantower-storage](https://github.com/MotoAcidic/Quantower-storage) strategies were
ported FROM. See `CLAUDE.md` for repo conventions and Pine-specific lessons learned.

This repo was split out of `Quantower-storage/TradingView/` on 2026-10-29.

## Operator-built, actively maintained

| Script | What it does |
| --- | --- |
| `DailyStructureBias.pine` | Marks confirmed swing highs ("Bear High") and swing lows ("Bull Low") on Daily/4H/1H/15m/5m/1m, each self-aggregated from the chart's own native bars; a level is solid while active, dashed once a close breaks it, with a top-right box showing the day's overall bullish/bearish bias. |
| `SmcSignals.pine` | Buy/sell signal generator for Smart Money Concepts trading — trend via break-of-structure, entries on retrace into an order block/FVG, with stop/target levels drawn on chart. |
| `IctSignals.pine` | Buy/sell signal generator for ICT trading — liquidity sweep + change-of-character + FVG retrace, restricted to the London/NY killzones, with stop/target levels drawn on chart. |
| `WicklessCandlesMTF.pine` | Flags wickless (one-sided) candles across 1m/5m/15m/1H/4H and draws a line + abbreviated label ("5m WLS H") out to the level, clearing once a candle closes fully through it. |

## Reference / third-party scripts

Scripts credited to their original authors in their own header comments; kept here either as
source material a Quantower strategy was ported from, or as a reference the operator charts
alongside their own tools.

| Script | What it does |
| --- | --- |
| `smc.pine` | "Smart Money Concepts [LuxAlgo]" — the reference SMC structure/order-block/FVG indicator the operator's own `SmcSignals.pine` and several Quantower ports draw on. |
| `Trendlines.pine` | "Trendlines with Breaks [LuxAlgo]" — automatic trendline detection and break signals; one of three indicators combined into a confluence-gated Quantower strategy. |
| `keltnerChannel.pine` | "Keltner Channel signals" — Keltner Channel mean-reversion signal script; the other of the three combined into that same confluence strategy. |
| `supportResistanceChannels.pine` | "Support Resistance Channels [SRchannel]" (LonesomeTheBlue) — S/R channel detection and breaks; the third of the three combined into that confluence strategy. |
| `VolumizedOrderBlocks.pine` | "Volumized Order Blocks \| Flux Charts" — order block detection weighted by volume. |
| `untestedLevels.pine` | "Sessions Suite [claysul]" — session high/low/PWL/PDL level marking; source material for the Quantower session-levels port. |
| `ScalpingSwingTradingTool.pine` | "Scalping Swing Trading Tool R1-6" (JustUncleL, Pine v3) — a two-pane 15m/1m scalping/swing toolkit. |

## Other scripts (not yet reviewed in depth this pass)

Titles taken from each script's own `indicator()`/`study()` declaration — these haven't had a full
read-through documented in `CLAUDE.md` yet, so treat the description below as a label, not a
verified summary.

| Script | Title |
| --- | --- |
| `ocean.pine` | Ocean's Stack v2 |
| `oceanVwap.pine` | Ocean σ-VWAP \| Session Bands |
| `priceLevels.pine` | QQQ Purple Price Levels Extended |
| `goldOrb.pine` | Gold ORB Strategy - Live Coach |
| `esOrb.pine` | ES Futures ORB Strategy |
| `absorption.pine` | Absorption Detector |
| `emaCross.pine` | 3 Fib EMAs – XO edition (fees) |
| `srm.pine` | Session Range Map — 1m Fixed |
| `vwap.pine` | VWAP + EMA9 With Signals |

## License

See `LICENSE`.
