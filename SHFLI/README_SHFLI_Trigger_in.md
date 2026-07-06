# SHFLI Burst DAQ with External Trigger

## Purpose

This notebook performs continuous burst acquisition on a Zurich Instruments SHFLI (tested on DEV12027 / DEV12470) triggered by an external signal on **Trigger In 1** (front-panel BNC). The demodulator runs at 50 MSa/s with the filter bypassed, and each trigger event opens a burst of 250 samples. The DAQ module runs in endless (continuous) mode for a configurable time window (default 60 s), then the collected bursts are displayed as a waterfall and averaged trace. A dedicated analysis section checks for data loss using three independent indicators.

---

## Workflow

1. **Connect** — open a zhinst.toolkit Session and connect the SHFLI
2. **Parameters** — set device channel, trigger source, sample rate, burst length, and acquisition time window
3. **Configure demodulator** — set rate to 50 MSa/s, enable filter bypass, set trigger source to Trigger In 1 (`source = 2`), enable `triggeracq = 1`, set `burstlen`
4. **Configure DAQ module** — set type `burst_trigger`, grid cols = burst length, trigger node = demod TrigIndex, enable endless mode, subscribe to demod R
5. **Run timed acquisition** — execute DAQ, poll every `READ_INTERVAL` seconds for `TIME_WINDOW` seconds, accumulate bursts and header metadata
6. **Waterfall plot** — 2D image of all bursts (burst index vs time within burst) + average trace
7. **Data loss analysis** — three-check summary:
   - Firmware `dataloss` flag (FIFO overflow between bursts)
   - NaN samples in burst data (DAQ could not reconstruct samples)
   - Trigger-number gaps (entire bursts dropped and not returned by `read()`)

---

## Key Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `DEVICE_ID` | `dev12027` | SHFLI serial number |
| `DEMOD_INDEX` | `0` | Demodulator channel (0-based; Demod 1 in UI) |
| `TRIGGER_SOURCE` | `2` | `2` = Trigger In 1 (external); `1024` = software trigger |
| `DEMOD_RATE` | `50e6` Sa/s | Demodulator output rate (filter bypass required) |
| `BURST_LEN` | `250` | Samples per trigger event (= 5 µs at 50 MSa/s) |
| `TIME_WINDOW` | `60.0` s | Total acquisition duration |
| `READ_INTERVAL` | `0.5` s | DAQ poll interval |
| `TRIGGER_DELAY` | `0.0` s | Delay from trigger edge to burst start |

---

## Requirements

### Hardware
- Zurich Instruments SHFLI (tested on DEV12027 / DEV12470)
- External pulse source connected to Trigger In 1 (front-panel BNC)

### Python packages
```
zhinst-toolkit
numpy
matplotlib
```

### Notes
- `demod.bypass(1)` is **required** to achieve 50 MSa/s output rate
- `triggeracq(1)` stamps a `TrigIndex` into the demod sample stream on each trigger edge — the DAQ module uses this as the burst boundary
- Set `TRIGGER_SOURCE = 1024` to use a software trigger for testing without external hardware
- `all_bursts` and `burst_meta` are initialised in the Parameters cell so the analysis cell can be run independently
