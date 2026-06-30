# SHFLI Microwave Cavity Ring-Down Measurement

## Purpose

This notebook measures the **ring-down time constant (τ) and quality factor (Q)** of a microwave cavity using a Zurich Instruments SHFLI Super-High-Frequency Lock-in Amplifier. A rectangular pulse excites the cavity, and the free-decay envelope is captured at 50 MSa/s with the demodulator filter bypassed. Both the ring-up (cavity charging) and ring-down (cavity decay) transients are fitted with exponential models to extract two independent time constants.

The Q factor is related to τ by: **Q = π × f₀ × τ**

---

## Workflow

1. **Connect** — open a zhinst.toolkit Session and connect the SHFLI (`DEV12470` or `DEV12266`)
2. **Frequency sweep (Part 1)** — run the sweeper module over ±10 MHz around the estimated resonance; fit a Lorentzian to extract f₀, Q, and linewidth
3. **Configure for ring-down (Part 2)** — retune the synthesizer to f₀, set demod rate to 50 MSa/s, enable filter bypass, and set burst-trigger mode
4. **Timeline pulse sequence** — send a rectangular excitation pulse via the Timeline Module; the demodulator captures the full section (ring-up + ring-down) in a single burst
5. **Fit ring-up** — fit `A_ss·(1 − exp(−t/τ_up))` to the rising edge; extract τ_up and Q_up
6. **Fit ring-down** — fit `A₀·exp(−t/τ_down)` to the decaying tail; extract τ_down, Q_down, and linewidth BW
7. **Plot** — linear and log-scale panels showing the envelope, X/Y quadratures, and both fitted curves

---

## Key Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `CENTER_FREQ_GUESS` | 3.75761 GHz | Initial resonance estimate for the sweep |
| `SWEEP_HALF_SPAN` | 10 MHz | Sweep range ±around center |
| `SWEEP_BW` | 1000 Hz | Demodulator bandwidth during CW sweep |
| `PULSE_DURATION` | 1 µs | Rectangular excitation pulse length (Timeline JSON) |
| `RINGDOWN_WINDOW` | 4 µs | Silent capture window after pulse |
| `DEMOD_RATE` | 50 MSa/s | Demodulator output rate (filter bypass ON) |
| `N_AVERAGES` | 1 | Number of ring-down repetitions to average |
| `OP_RANGE` | −20 dBm | Output power range |
| `IP_RANGE` | −30 dBm | Input power range |

---

## Requirements

### Hardware
- Zurich Instruments SHFLI (tested on DEV12470 / DEV12266)
- Microwave cavity connected to SHFLI sigout[0] (drive) and sigin[0] (readout)

### Python packages
```
zhinst-toolkit
numpy
scipy
matplotlib
```

### Notes
- Filter bypass (`demod.bypass = 1`) must be ON for 17 MHz bandwidth at 50 MSa/s
- The Timeline Module JSON controls the actual hardware pulse duration — ensure it matches `PULSE_DURATION` in the Python parameters
- Pulse-off is detected automatically from the steepest downward gradient of the smoothed envelope, so the fit is robust even if the cavity does not fully saturate during the pulse
