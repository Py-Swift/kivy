# Phase 0 — Clock

**File:** `kivy/kivy/clock.py` only. No changes to `_clock.pyx`.

---

## Problem

`ClockBaseBehavior.idle()` sleeps to enforce `maxfps` then measures
wall time to compute `_dt` / `_last_tick`. When an external caller
already owns the cadence, that sleep is purely additive latency.

## What changes

**`idle(self, dt=None)`** — add the `dt` parameter:
- If `dt` is given: skip the sleep entirely. Set `_dt = dt`, advance
  `_last_tick += dt`, return `_last_tick`. No wall-clock measurement.
- If `dt` is None: existing behaviour unchanged (sleep + measure).

**`tick(self, dt=None)`** — add the `dt` parameter and pass it to `idle()`.

That is all. `_process_events()` in `_clock.pyx` only reads `self._last_tick`,
which `idle()` already owns — nothing else needs to change.

## Call path

```
external caller: Clock.tick(dt)
    → pre_idle()               # release weak refs
    → idle(dt)                 # _last_tick += dt, no sleep, returns _last_tick
    → post_idle(ts, current)   # _process_events() → fires due schedule_interval callbacks
```

## Verification (standalone, no Swift)

```python
from kivy.clock import Clock
import time

Clock.schedule_interval(lambda dt: print(f"dt={dt:.4f}"), 1.0)

# simulate an external driver at 60 fps
for _ in range(300):
    Clock.tick(1/60)
    time.sleep(1/60)
```

Expected: callback fires every ~60 ticks (1 second), `dt ≈ 1.0` each time.
