# Furuta Pendulum — Hybrid Swing-Up + LQR Control (Hardware-in-the-Loop)

A custom-built **rotary inverted (Furuta) pendulum** that swings itself up from rest and balances upright using a **hybrid controller**: an energy-based **swing-up** mode that hands off to a state-feedback **LQR** stabilizer. Control runs **hardware-in-the-loop** — a Simulink model drives the real rig through an Arduino Mega.

> Course project, *Hybrid Control Systems* (MCT411), Ain Shams University. See *Credits*.

## Highlights
- **Hybrid control:** energy-based **swing-up** → automatic switch to **LQR** balance near the upright (the "hybrid" in hybrid control).
- **LQR over 4 states:** arm angle & rate, pendulum angle & rate — full state feedback for upright stabilization.
- **Hardware-in-the-Loop (HIL):** Simulink controller ↔ **Arduino Mega** (reads the pendulum/motor encoders, drives the DC motor via an H-bridge in real time).
- **System identification:** estimated the DC-motor model parameters from collected step/response data before designing the controller.
- **Auto-tuned weights:** searched the LQR **Q/R** matrices with **Bayesian optimization** against an **ITAE** cost to trade off settling time vs. effort.

## Hardware
Arduino Mega · DC motor with encoder (arm drive) · pendulum encoder + coupler · H-bridge motor driver · bench power supply · custom mechanical frame (CAD-designed).

## Tech stack
MATLAB · Simulink (HIL) · Arduino (C/C++) · LQR / optimal control · system identification · Bayesian optimization

## Repo structure
```
matlab/
├── Sys_model_and_Lqr.mlx   # DC-motor/gearbox params → state-space model → LQR gain
├── Swing_Up.mlx            # energy-based swing-up logic
└── Tune.mlx                # LQR Q/R weight tuning
simulink/
├── Furuta_3.slx            # Simscape Multibody plant (imported from our CAD) + Furuta_3_DataFile.m
├── Param_estimation.slx    # motor parameter estimation
├── Data_collecting.slx     # data-acquisition model for system ID
└── HIL_2.slx               # hardware-in-the-loop model (Arduino Mega)
```

## Dependencies
MATLAB/Simulink · Simscape Multibody (for the imported plant) · Control System Toolbox (`lqr`). The **HIL model uses giampy1969's "Simulink Device Drivers" package** (third-party, MATLAB File Exchange) for the Arduino encoder/IO blocks — install it separately; it is **not** included in this repo.

## Implementation
The full **modeling, control, and tuning** stack (all the MATLAB/Simulink work):
- **System identification** — collected motor data and estimated the DC-motor / gearbox parameters (`Data_collecting.slx`, `Param_estimation.slx`).
- **Plant model** — imported the Furuta mechanism from CAD into a **Simscape Multibody** model (`Furuta_3.slx`).
- **State-space + LQR** — derived the linearized state-space model and designed the LQR feedback (`Sys_model_and_Lqr.mlx`).
- **Swing-up** — energy-based swing-up that hands off to the LQR near upright (`Swing_Up.mlx`).
- **Live Bayesian auto-tuning** — a **`bayesopt`** loop that tunes the LQR **Q/R** weights against an **ITAE** cost **while the pendulum runs on the Arduino HIL rig** (`Tune.mlx`), replacing manual trial-and-error.
- **Hardware-in-the-loop** — the Simulink ↔ **Arduino Mega** model (`HIL_2.slx`).

## Results
- Repeatable swing-up and upright balance under the LQR controller.
- LQR weights tuned via Bayesian optimization (ITAE) rather than manual trial-and-error.

## Credits
Team project at Ain Shams University (Mechatronics, Hybrid Control Systems / MCT411): Mohamed Refaie, Haneen Amr Ahmed, and Noran Tarek Hassan.
