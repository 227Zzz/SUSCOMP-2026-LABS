# Week 5 Lab Notes: Measuring Software Energy Consumption 
## 1. Introduction and method
 
**Goal.** Measure the energy used by running software, and see how much (a) the way it is written
(pure Python loops vs NumPy) and (b) where it runs (CPU vs GPU) changes that energy.
 
**Machine.**
 
| Item | Value |
|------|-------|
| Model | MacBook Air (MacBookAir10,1, Apple M1) |
| CPU / SoC | Apple M1 |
| OS | macOS 15.6.1 |
| Python | Python 3.12.6 |
| Power source during main runs | Battery |
 
**Meters.**
 
| Meter | Scope | Mode | Used in |
|-------|-------|------|---------|
| `macmon` (default) | SoC (CPU + GPU + Neural Engine) | measured (OS power model) | Parts A, B |
| `battery` | whole laptop (battery) | measured (fuel gauge) | Part C |
 
`macmon` reads power from macOS's internal power model, so it is a measurement, but not a reading from a dedicated hardware power meter. It reports CPU, GPU and Neural Engine together as the SoC. RAM power is reported separately in the `extra` column and is not part of the SoC total.
 
**How I kept measurements clean.**
 
- Closed other applications (browser, video calls, cloud sync).
- Kept the same power settings and screen brightness for all runs.
- Measured `idle` as a baseline.
- Each workload runs for about 30 s. The measured time can be longer because it includes start-up.
- One unmeasured warm-up run first (`python workload.py numpy-matmul --seconds 5`).
- 5 repeats per test, alternating between tests instead of running one test 5 times in a row.
- Reported every run, the mean and the standard deviation.
**Setup.**
 
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python workload.py numpy-matmul --seconds 5
```
 
I switched from Intel Homebrew to Apple Silicon Homebrew because macmon requires the arm64 architecture.
 
## 2. Choosing the meter
 
Command:
 
```bash
python measure.py --label idle -- python workload.py idle --seconds 10
```
 
The `scope` column was `SoC (CPU+GPU+ANE)` and the `mode` column was
`measured (OS power model)`. So the meter covers the whole chip, and the number is a measurement
based on the operating system's power model, not an estimate from CPU model and load.
 
This 10 s run was only a practice run; I deleted `runs.csv` afterwards so it is not in the results.
 
## 3. Part A: Does the way software is written matter?
 
**Goal.** Compare the same matrix multiplication written as pure Python loops and with NumPy,
plus an idle baseline.
 
**Command** (5 rounds, alternating):
 
```bash
for i in 1 2 3 4 5; do
  python measure.py --label idle   -- python workload.py idle
  python measure.py --label python -- python workload.py python-matmul
  python measure.py --label numpy  -- python workload.py numpy-matmul
done
```
 
**Results** 
 
All runs:
### idle

| Run | Time (s) | Energy (J) | Energy (Wh) | Avg power (W) | Work done (reps) | Energy per rep |
|-----|----------|------------|-------------|---------------|------------------|----------------|
| 1 | 30.06 | 5.0 | 0.0014 | 0.17 | 0 | – |
| 2 | 30.07 | 9.6 | 0.0027 | 0.32 | 0 | – |
| 3 | 30.07 | 1.5 | 0.0004 | 0.05 | 0 | – |
| 4 | 30.08 | 2.9 | 0.0008 | 0.09 | 0 | – |
| 5 | 30.07 | 1.6 | 0.0004 | 0.05 | 0 | – |
| **Mean** | **30.07** | **4.1** | **0.0011** | **0.14** | | |
| **SD** | 0.01 | 3.4 | 0.0009 | 0.11 | | |

### python

| Run | Time (s) | Energy (J) | Energy (Wh) | Avg power (W) | Work done (reps) | Energy per rep |
|-----|----------|------------|-------------|---------------|------------------|----------------|
| 1 | 30.10 | 54.8 | 0.0152 | 1.82 | 287 | 191 mJ |
| 2 | 30.14 | 53.3 | 0.0148 | 1.77 | 288 | 185 mJ |
| 3 | 30.14 | 54.6 | 0.0152 | 1.81 | 288 | 190 mJ |
| 4 | 30.12 | 52.8 | 0.0147 | 1.75 | 289 | 183 mJ |
| 5 | 30.14 | 53.6 | 0.0149 | 1.78 | 288 | 186 mJ |
| **Mean** | **30.13** | **53.8** | **0.0149** | **1.79** | | |
| **SD** | 0.02 | 0.9 | 0.0002 | 0.03 | | |

### numpy

| Run | Time (s) | Energy (J) | Energy (Wh) | Avg power (W) | Work done (reps) | Energy per rep |
|-----|----------|------------|-------------|---------------|------------------|----------------|
| 1 | 30.17 | 92.0 | 0.0256 | 3.05 | 1,458,851 | 63.1 µJ |
| 2 | 30.17 | 93.0 | 0.0258 | 3.08 | 1,455,401 | 63.9 µJ |
| 3 | 30.16 | 91.6 | 0.0254 | 3.04 | 1,445,392 | 63.4 µJ |
| 4 | 30.15 | 94.0 | 0.0261 | 3.12 | 1,457,662 | 64.5 µJ |
| 5 | 30.16 | 95.3 | 0.0265 | 3.16 | 1,433,237 | 66.5 µJ |
| **Mean** | **30.16** | **93.2** | **0.0259** | **3.09** | | |
| **SD** | 0.01 | 1.5 | 0.0004 | 0.05 | | |

Energy per GFLOP:
 
### Summary (macmon)

Idle power: 0.14 W.

| Test | Avg power (W) | Speed (GFLOP/s) | Energy per GFLOP | SD | Above idle, per GFLOP | Energy vs. baseline |
|------|---------------|-----------------|------------------|----|-----------------------|---------------------|
| python | 1.79 ± 0.03 | 0.03 | 54.1 J | 995 mJ | 49.9 J | – |
| numpy | 3.09 ± 0.05 | 167.05 | 18.6 mJ | 390 µJ | 17.8 mJ | 2,908.0× less | 

**Notes.** The first loop was interrupted and restarted, so the two incomplete rows
(one idle, one python) were removed from `runs.csv`; the remaining 15 runs alternate
idle/python/numpy five times. Idle power varied a lot between runs (0.05-0.32 W,
SD 3.4 J on a mean of 4.1 J) because the idle draw is so low that 1 s sampling is noisy.
This does not change the python/numpy comparison, since idle energy (about 4 J per 30 s)
is small compared with 54 J and 93 J.
## 4. Part B: Does offloading to the GPU help?

**Goal.** Compare PyTorch matrix multiplication on the CPU and on the Apple GPU (`mps`).
`macmon` covers both CPU and GPU, so one meter is enough.

**Command** (5 rounds, alternating):

```bash
for i in 1 2 3 4 5; do
  .venv/bin/python measure.py --label torch-cpu -- .venv/bin/python workload.py torch-matmul --device cpu
  .venv/bin/python measure.py --label torch-gpu -- .venv/bin/python workload.py torch-matmul --device mps
done
```

**Results:**

torch-cpu

| Run | Time (s) | Energy (J) | Energy (Wh) | Avg power (W) | Work done (reps) | Energy per rep |
|-----|----------|------------|-------------|---------------|------------------|----------------|
| 1 | 31.92 | 72.8 | 0.0202 | 2.28 | 1,038 | 70.1 mJ |
| 2 | 31.91 | 75.4 | 0.0209 | 2.36 | 1,040 | 72.5 mJ |
| 3 | 31.86 | 74.7 | 0.0207 | 2.34 | 1,033 | 72.3 mJ |
| 4 | 31.53 | 70.8 | 0.0197 | 2.24 | 1,036 | 68.3 mJ |
| 5 | 31.60 | 70.9 | 0.0197 | 2.24 | 1,041 | 68.1 mJ |
| **Mean** | **31.76** | **72.9** | **0.0203** | **2.29** | | |
| **SD** | 0.18 | 2.1 | 0.0006 | 0.06 | | |

torch-gpu

| Run | Time (s) | Energy (J) | Energy (Wh) | Avg power (W) | Work done (reps) | Energy per rep |
|-----|----------|------------|-------------|---------------|------------------|----------------|
| 1 | 32.93 | 76.3 | 0.0212 | 2.32 | 1,398 | 54.6 mJ |
| 2 | 33.23 | 78.6 | 0.0218 | 2.36 | 1,205 | 65.2 mJ |
| 3 | 33.42 | 76.6 | 0.0213 | 2.29 | 1,466 | 52.2 mJ |
| 4 | 32.70 | 75.2 | 0.0209 | 2.30 | 1,448 | 52 mJ |
| 5 | 32.25 | 74.3 | 0.0206 | 2.30 | 1,473 | 50.4 mJ |
| **Mean** | **32.91** | **76.2** | **0.0212** | **2.31** | | |
| **SD** | 0.46 | 1.6 | 0.0004 | 0.03 | | |

Energy per GFLOP (idle power 0.14 W):

| Test | Avg power (W) | Speed (GFLOP/s) | Energy per GFLOP | SD | Above idle, per GFLOP |
|------|---------------|-----------------|------------------|----|-----------------------|
| torch-cpu | 2.29 ± 0.06 | 593.95 | 4.09 mJ | 122 µJ | 3.85 mJ |
| torch-gpu | 2.31 ± 0.03 | 800.28 | 3.19 mJ | 347 µJ | 3.01 mJ |

**Notes.**

- Matrix size n = 2048 (17.18 GFLOP per rep). Idle power from Part A (0.14 W) was reused
  as the baseline.
- Run 2 of torch-gpu was slower than the others (1,205 reps vs about 1,450), so its energy
  per rep is higher (65.2 mJ vs about 52 mJ). I kept it in the results.
- The measured time (31.5-33.4 s) is longer than the 30 s workload because loading PyTorch
  (and initialising the GPU) happens inside the measured window, so some start-up energy is
  included in the torch results.
## 5. Part C: Do two meters agree?

**Goal.** Repeat Part A with a second meter (`battery`, charger unplugged) and explain how
and why it differs from `macmon`.

**Commands** (5 rounds, alternating):

```bash
python measure.py --label idle   --meter battery -- python workload.py idle
python measure.py --label python --meter battery -- python workload.py python-matmul
python measure.py --label numpy  --meter battery -- python workload.py numpy-matmul
```

**Results:**
## Meter: battery

Scope: whole laptop (battery). Mode: measured (fuel gauge). Machine: Apple M1 (Darwin).

### idle

| Run | Time (s) | Energy (J) | Energy (Wh) | Avg power (W) | Work done (reps) | Energy per rep |
|-----|----------|------------|-------------|---------------|------------------|----------------|
| 1 | 30.10 | 117.6 | 0.0327 | 3.91 | 0 | – |
| 2 | 30.05 | 215.9 | 0.0600 | 7.18 | 0 | – |
| 3 | 30.06 | 107.5 | 0.0299 | 3.58 | 0 | – |
| 4 | 30.05 | 186.5 | 0.0518 | 6.21 | 0 | – |
| 5 | 30.06 | 105.9 | 0.0294 | 3.52 | 0 | – |
| **Mean** | **30.06** | **146.7** | **0.0407** | **4.88** | | |
| **SD** | 0.02 | 51.0 | 0.0142 | 1.70 | | |

### python

| Run | Time (s) | Energy (J) | Energy (Wh) | Avg power (W) | Work done (reps) | Energy per rep |
|-----|----------|------------|-------------|---------------|------------------|----------------|
| 1 | 30.17 | 77.9 | 0.0216 | 2.58 | 285 | 273 mJ |
| 2 | 30.14 | 180.1 | 0.0500 | 5.98 | 286 | 630 mJ |
| 3 | 30.13 | 69.2 | 0.0192 | 2.30 | 290 | 239 mJ |
| 4 | 30.06 | 164.2 | 0.0456 | 5.46 | 289 | 568 mJ |
| 5 | 30.14 | 69.9 | 0.0194 | 2.32 | 290 | 241 mJ |
| **Mean** | **30.13** | **112.3** | **0.0312** | **3.73** | | |
| **SD** | 0.04 | 55.1 | 0.0153 | 1.83 | | |

### numpy

| Run | Time (s) | Energy (J) | Energy (Wh) | Avg power (W) | Work done (reps) | Energy per rep |
|-----|----------|------------|-------------|---------------|------------------|----------------|
| 1 | 30.23 | 142.9 | 0.0397 | 4.73 | 1,457,484 | 98.1 µJ |
| 2 | 30.12 | 143.7 | 0.0399 | 4.77 | 1,463,015 | 98.2 µJ |
| 3 | 30.11 | 128.0 | 0.0356 | 4.25 | 1,461,178 | 87.6 µJ |
| 4 | 30.13 | 145.0 | 0.0403 | 4.81 | 1,462,899 | 99.1 µJ |
| 5 | 30.11 | 134.8 | 0.0374 | 4.48 | 1,462,054 | 92.2 µJ |
| **Mean** | **30.14** | **138.9** | **0.0386** | **4.61** | | |
| **SD** | 0.05 | 7.3 | 0.0020 | 0.24 | | |

### Summary (battery)

Idle power: 4.88 W.

| Test | Avg power (W) | Speed (GFLOP/s) | Energy per GFLOP | SD | Above idle, per GFLOP | Energy vs. baseline |
|------|---------------|-----------------|------------------|----|-----------------------|---------------------|
| python | 3.73 ± 1.83 | 0.03 | 113 J | 55.6 J | -34.8 J | – |
| numpy | 4.61 ± 0.24 | 168.34 | 27.5 mJ | 1.44 mJ | -1.62 mJ | 4,106.0× less |
### Comparison with macmon

All values below use all five runs. Differences are calculated from the displayed mean powers.

| Test | Battery power (W, mean ± SD) | Macmon power (W, mean ± SD) | Battery − macmon (W) |
|------|-----------------------------|---------------------------|---------------------|
| idle | 4.88 ± 1.70 | 0.14 ± 0.11 | 4.74 |
| python | 3.73 ± 1.83 | 1.79 ± 0.03 | 1.94 |
| numpy | 4.61 ± 0.24 | 3.09 ± 0.05 | 1.52 |

### Notes

- The charger was unplugged. The battery meter covers the whole laptop, while macmon covers the SoC, so higher battery readings are expected.
- Battery idle and Python readings were unusually high in rounds 2 and 4. I kept all five runs. The cause of this variation is unclear.
- Both Python and NumPy have negative above-idle energy in the battery summary. These values are retained as calculated, but the variable idle baseline makes them unsuitable for estimating additional workload energy.
- NumPy used 27.5 mJ/GFLOP with battery and 18.6 mJ/GFLOP with macmon, about 1.5 times as much energy per GFLOP. Both meters show much lower energy per GFLOP for NumPy than Python: about 4,106 times lower with battery and 2,908 times lower with macmon.
- The power gaps differ between tests. Because the meters were used in separate runs and the battery readings varied, these gaps cannot precisely identify the power used by components outside the SoC.
 
## 6. Worked example (one row by hand)
 
Python run 1:
- E = 54.8 J, t = 30.10 s
- P = E / t = 54.8 / 30.10 = **1.82 W**
- E in Wh = 54.8 / 3600 = **0.0152 Wh**
- P(idle) = 0.14 W
- Energy above idle = 54.8 - (0.14 x 30.10) = 54.8 - 4.2 = **50.6 J**
- Work done = 287 reps x 0.003456 GFLOP (2 x 120^3 FLOP) = 0.992 GFLOP
- Energy per GFLOP = 54.8 / 0.992 = **about 55 J/GFLOP**

## 7. Comparison
 
| Version | Mean power (W) | Mean energy per GFLOP (J/GFLOP) | Estimated GFLOP in 30 s |
|---------|----------------|----------------------------------|--------------------|
| Python loops | 1.79 | 54.1 J | about 1 |
| NumPy | 3.09 | 0.0186 J | about 5,000 |
| Torch CPU | 2.29 | 0.00409 J | about 17,819 |
| Torch GPU (mps) | 2.31 | 0.00319 J | about 24,008 |
 
- NumPy used about 2,900x less energy per GFLOP than pure Python (18.6 mJ vs 54.1 J).
  - This is not because power went down: NumPy's average power was higher (3.09 W vs 1.79 W, about 1.7x).
  - It is because the work finished much faster: 167 GFLOP/s vs 0.033 GFLOP/s, about 5,000x.
  - Since energy = power x time, 1.7x more power for 1/5,000 of the time gives roughly 1/2,900 of the energy.

- GPU used about 22.0% less energy per GFLOP than torch CPU (3.19 mJ vs 4.09 mJ).
  - This is not because power went down: GPU power was slightly higher (2.31 W vs 2.29 W, about 0.9%).
  - It is because the GPU processed work faster: 800.28 GFLOP/s vs 593.95 GFLOP/s, about 34.7% faster.
  - Since energy = power x time, the GPU used less energy per unit of work despite its slightly higher power consumption.
## 8. Scaling up: 100 TFLOP per year
 
## 8. Scaling up: 100 TFLOP per year

100 TFLOP = 100,000 GFLOP.

- Energy (kWh) = energy per GFLOP (J/GFLOP) × 100,000 / 3,600,000
- Run time (hours) = 100,000 / throughput (GFLOP/s) / 3,600

| Version | Energy per GFLOP (J/GFLOP) | Energy (kWh/year) | Run time (hours/year) |
|---------|--------------------------:|-----------------:|---------------------:|
| Python loops | 54.1 | 1.503 | about 842 |
| NumPy | 0.0186 | 0.0005167 | 0.1663 |
| Torch CPU | 0.00409 | 0.0001136 | 0.04677 |
| Torch GPU (mps) | 0.00319 | 0.00008861 | 0.03471 |

These are approximate projections from the measured energy per GFLOP and reported throughput, assuming similar performance.
## 9. Reflections

- **What surprised me:** NumPy used about 2,900x less energy per GFLOP than Python despite drawing more power. The GPU also used about 22% less energy per GFLOP than the CPU.

- **How the two meters compared:** `battery` reported higher power because it measures the whole laptop, but its results were much less consistent than `macmon`.

- **Which meter I trust:** I trust `macmon` more because its results were more stable across repeated runs, although it only measures the SoC.

- **What the meters cannot see:** `macmon` excludes the screen, RAM and other components. `battery` includes them but cannot measure them separately. Neither can isolate program energy perfectly or account for external network energy and manufacturing emissions.