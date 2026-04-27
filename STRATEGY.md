# Meridian North — Professional LP Strategy Guide

**Target: ~90% win rate | Small consistent profit | Rare losses**

---

## Core Philosophy

> **Win by not losing.**

In DLMM LP, the biggest mistake is optimizing for maximum yield at the expense of capital protection. A 50% loss requires a 100% gain just to break even. This strategy prioritizes:

1. **Fee income as the primary profit driver** — not token price speculation
2. **Tight loss limits** — cut fast, redeploy to better opportunities
3. **Frequent small wins** — compound them relentlessly
4. **Adaptive learning** — every closed position teaches the system

---

## Why This Config Produces ~90% Win Rate

### The Math

In DLMM, you profit when:

```
Fee Income > Impermanent Loss + Gas Costs
```

At `minFeeActiveTvlRatio: 0.08` (8% daily fee/active TVL):
- Pool with $40k active TVL earns $3,200/day in fees
- A 0.5 SOL position (~$75) = ~0.19% of active TVL
- Your fee share = ~$6/day = 8%/day on position size
- Over a 4-hour hold: ~1.3% in fees earned

With `stopLossPct: -10` and `takeProfitPct: 4`:
- Win condition: fees + price ≥ +4%
- Loss condition: fees + IL ≤ -10%
- The asymmetry is strong: you need a large adverse move to lose, but small favorable move to win

### Why -10% Stop Loss is Critical

The default stop loss of -50% is the **single biggest reason bots lose money**. In meme token LP:

- A token can drop 30-80% in minutes on bad news/rug
- Without tight SL, one bad position wipes out 10 consecutive wins
- With -10% SL: worst case = -10% per losing trade
- Win rate needed to be profitable at -10% SL / +4% TP = 72%+
- At 90% win rate with these parameters: extremely profitable

### Why 2 Max Positions

Quality beats quantity. With 3 positions:
- Capital is fragmented across potentially mediocre pools
- You "had to deploy somewhere" pressure → compromised screening
- Management cycles are split attention

With 2 positions:
- Deploy only when you find genuinely excellent pools
- Sometimes sit on cash — that's fine, idle capital beats losing positions
- Bot becomes more selective naturally

---

## Screening Logic Explained

### The 5 Non-Negotiables

| Parameter | Value | Why |
|-----------|-------|-----|
| `minFeeActiveTvlRatio` | 0.08 | Minimum fee income to beat IL |
| `minOrganic` | 72 | Real volume = real fee income |
| `maxBundlePct` | 20 | Low = low coordinated dump risk |
| `stopLossPct` | -10 | Capital preservation above all |
| `blockPvpSymbols` | true | Hard-filter manipulated tokens |

### Pool Selection Sweet Spot

```
TVL: $25,000 – $80,000
 └─ Too small (<$25k): your position dominates, IL dynamics distort
 └─ Too large (>$80k): fees diluted, percentage yield shrinks
 └─ Sweet spot: you earn 3-8% of pool fees per 0.5 SOL deployed

Volume: >$8,000/day
 └─ At 0.8-1.0% fee rate: $8k vol = $64-80 distributed across LPs
 └─ If you're 2-4% of active TVL, you earn $1.3-3.2/day
 └─ Over 4-8 hours: meaningful contribution to 4% take profit

Mcap: $200k – $6M
 └─ Below $200k: too thin, rug risk extreme
 └─ Above $6M: volume-to-mcap ratio drops (mature = stable = less fees)
 └─ $500k-$3M: active growth phase = maximum fee generation
```

### Token Age: The Goldilocks Zone

```
< 3 hours: SKIP — snipers, bundlers still holding, early distribution chaos
3 – 24 hours: BEST — initial distribution done, momentum building, high volume
24 – 72 hours: GOOD — proven volume, established community
72 – 120 hours: OK — still active, declining but still worth it
> 120 hours: SKIP — meme volume has typically collapsed by day 5
```

---

## Position Management Explained

### The Exit Hierarchy (Deterministic Rules — Before LLM)

```
1. STOP LOSS: pnl_pct <= -10%
   → Close immediately, no questions asked
   → Largest single win-rate contributor

2. TAKE PROFIT: pnl_pct >= +4%
   → Close immediately (or trailing TP kicks in at 3%)

3. PUMPED ABOVE RANGE: active_bin > upper_bin + 7
   → Token pumped hard past your range → immediate close
   → You've been holding only SOL (bad side of bid_ask), collect and redeploy

4. OOR TIMEOUT: out of range > 15 minutes
   → Every minute OOR = zero fee income
   → 15 min is long enough to confirm it's not a wick

5. LOW YIELD: fee/TVL24h < 6% AND position age > 30 min
   → Volume collapsed → exit, redeploy to active pool
```

### Trailing Take Profit — Capturing Asymmetric Upside

```
Standard: Position closes at exactly +4% every time
Trailing: Position can close at +4%, +7%, +12%, +20%...

How it works:
  Position hits 3% → trailing activates
  Position peaks at 9% → trailing peak = 9%
  Position drops to 7.5% (1.5% drop) → CLOSE at 7.5%

Result: Small wins stay at 4%, lucky big moves captured at 6-20%
        Losing positions still hard-stopped at -10%
```

### Fee Claiming Strategy

Claim at $2 earned (not $5 default):
- More frequent = locked-in profit even if position later fails
- Auto-swap to SOL = ready capital for next deployment
- Example: earn $3 in fees, position then hits SL at -10%
  - Without auto-claim: net loss = -10% + $3 = roughly -7%
  - With auto-claim: $3 secured + position closed at -10%
  - Effective: fees reduce actual loss meaningfully

---

## Darwinian Learning — The Self-Improvement Engine

After every 3 closed positions (not 5 default), the system:

1. **Signal Weighting**: Calculates which of these 10 signals actually predicted wins:
   - organic_score, fee_tvl_ratio, volume, mcap, holder_count
   - smart_wallets_present, narrative_quality, study_win_rate
   - hive_consensus, volatility

2. **Threshold Evolution** (now fully functional after bug fixes):
   - `maxVolatility`: Tightened if losers cluster at high volatility
   - `minFeeActiveTvlRatio`: Raised if winners consistently had higher fee/TVL
   - `minOrganic`: Raised if organic score gap between winners/losers is large

3. **Result**: Bot becomes better at pool selection over time without manual intervention

**Minimum 8 positions** before signal weights carry meaningful statistical weight. Run dry-run for at least 10 simulated cycles before going live.

---

## Recommended Launch Sequence

### Phase 1: Dry Run (Days 1-3)
```
dryRun: true
maxPositions: 2
deployAmountSol: 0.5
```
- Observe which pools the bot selects
- Verify screening produces quality candidates
- Check LLM reasoning in logs
- Target: 10+ simulated positions

### Phase 2: Live with Micro Capital (Days 4-10)
```
dryRun: false
maxPositions: 1
deployAmountSol: 0.1 (just gas + fees)
```
- Verify on-chain execution works
- Confirm Telegram alerts fire correctly
- Check fee claiming works
- Confirm auto-swap after close

### Phase 3: Operational (Day 10+)
```
dryRun: false
maxPositions: 2
deployAmountSol: 0.5
```
- Bot runs autonomously
- Review daily briefing on Telegram
- Let Darwin evolve thresholds
- Run /evolve manually after 5+ closed positions

### Phase 4: Scaling (After 30+ positions, consistent win rate)
```
maxPositions: 2 (keep at 2)
positionSizePct: 0.30 → naturally compounds with wallet growth
```
- Do NOT increase maxPositions — keep quality focus
- Do NOT lower thresholds to "find more pools" — wait for quality
- Let compounding formula scale position size automatically

---

## Red Flags — When to Manually Intervene

| Signal | Action |
|--------|--------|
| Win rate drops below 60% after 15+ positions | Run `/evolve`, check logs, possibly tighten thresholds |
| Bot deploys same pool multiple times | Check `repeatDeployCooldown` is active; run `/thresholds` |
| OOR happening within 5 min of every deploy | Raise `maxVolatility` from 3.5 → 2.5; reduce `maxBinStep` |
| Screening returns 0 candidates consistently | Lower `minFeeActiveTvlRatio` 0.08 → 0.06 (market wide) |
| Large loss despite -10% SL | PnL sanity check; check `pnlSanityMaxDiffPct` in logs |

---

## GMGN Mode (if API key available)

Switch `screeningSource: "gmgn"` in user-config.json and configure gmgn-config.json.

**Additional filters GMGN adds:**
- `maxRatTraderRate: 0.10` — rat traders flip fast; their exit = price drop
- `maxSniperHoldRate: 0.20` — snipers coordinating dump risk
- `requireKol: true, minKolCount: 2` — social proof = real sustained volume
- `indicatorFilter: true` — supertrend bullish confirmation before entry
- `maxRsi: 80` — don't enter tokens already overbought (peak risk)

GMGN mode provides meaningfully better signal quality at the cost of a paid API key. Recommended for serious operation.

---

## Key Insight: Patience is the Strategy

On some screening cycles, the bot will find **zero candidates**. This is correct behavior.

**Do not lower thresholds to force a deploy.** An idle 0.5 SOL earning 0% is better than a 0.5 SOL in a bad pool losing 10%.

The strategy wins through:
1. Strict entry criteria (90% of the work)
2. Fast exit on anything that goes wrong
3. Consistent compounding of 4% wins
4. Darwin learning improving entries over time

> 90 wins of +4% each = +260% with compounding
> 10 losses of -10% each = -65% cumulative damage
> Net at 90% win rate over 100 trades = massive profit

