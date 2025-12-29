<div align="center">

# etsi-watchdog
### Real-Time Data Drift Detection. Extensible. Production-Ready.

[![License](https://img.shields.io/badge/License-BSD_2--Clause-orange.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.8%2B-blue)](https://www.python.org/)
[![PyPI](https://img.shields.io/pypi/v/etsi-watchdog.svg)](https://pypi.org/project/etsi-watchdog/)
[![Status](https://img.shields.io/badge/Status-Production-green)]()

> **Don't let data drift silently kill your model's performance.**

`etsi-watchdog` is a modular, production-grade library designed to detect data drift in machine learning pipelines. Whether you need static report generation, real-time rolling window monitoring, or custom drift algorithms, Watchdog fits seamlessly into your stack.

[Features](#key-features) • [Installation](#installation) • [Architecture](#plug-in-architecture-new-in-v3) • [Quickstart](#quickstart) • [Integrations](#integrations)

</div>

---

## Why Watchdog?

Most drift detection tools are either too heavy or too rigid. Watchdog v3.0.0 changes that.

* **Plug-in Architecture**: Don't like our defaults? Register your own drift algorithms dynamically at runtime.
* **Production First**: Built-in **Slack** integration for real-time alerting when data deviates.
* **Versatile Monitoring**: Supports both **Static** (train vs. inference) and **Rolling** (time-series) monitoring.
* **Lightweight Core**: Built on top of the PyData stack (Pandas, NumPy, Scikit-learn) with minimal overhead.

---

## Key Features

* **Multi-Method Detection**: Out-of-the-box support for PSI (Population Stability Index), KS-Test, and Wasserstein Distance.
* **Rolling Windows**: Monitor data streams over time with configurable time-frequency windows (e.g., "check every 24 hours").
* **Model-Based Drift**: Integrated **Isolation Forest** support for detecting anomalies in high-dimensional data.
* **Alerting System**: Send drift notifications directly to your Slack workspace with severity levels.
* **Report Generation**: Export drift results to JSON or generate visual reports for stakeholders.

---

## Installation

### Prerequisites
* Python (3.8 or later)

### From PyPI (Stable)
```bash
pip install etsi-watchdog
```

### From Source (Development)
```bash
git clone https://github.com/etsi-ai/etsi-watchdog.git
cd etsi-watchdog
pip install -e .
```

---

## Plug-in Architecture (New in v3)
Watchdog v3 introduces a dynamic registry, allowing you to bring your own drift logic.

```python
from etsi.watchdog import register_drift_algorithm, DriftResult

# 1. Define your custom drift function
def my_custom_drift(ref_df, curr_df, feature, **kwargs):
    # Your custom logic here...
    score = 0.5 
    return DriftResult(method="custom", score=score, threshold=0.2)

# 2. Register it
register_drift_algorithm("my_algo", my_custom_drift)

# 3. Use it immediately
from etsi.watchdog import DriftCheck
check = DriftCheck(reference_data, algorithm="my_algo")
```

---

## Quickstart

### 1. Static Drift Check
Compare your training data (reference) against new production data (current).

```python
import pandas as pd
from etsi.watchdog import DriftCheck

# Load data
ref = pd.read_csv("training_data.csv")
live = pd.read_csv("production_data.csv")

# Initialize check (uses PSI by default)
check = DriftCheck(reference_df=ref, algorithm="psi")

# Run check on specific features
results = check.run(current_df=live, features=["age", "salary"])

for feature, result in results.items():
    print(result.summary())
    # Output: Drift Detected: True | Score: 0.45 | Threshold: 0.2
```

### 2. Rolling Monitoring
Monitor a stream of data over time, perfect for daily batch jobs.

```python
from etsi.watchdog import Monitor

# Initialize monitor with reference data
monitor = Monitor(reference_df=ref)

# Watch rolling data (e.g., daily batches)
results = monitor.watch_rolling(
    df=time_indexed_live_df,
    window=30,      # Look back 30 samples
    freq="D",       # Check daily
    features=["age"]
)
```

### 3. Isolation Forest (Anomaly Detection)
Use ML-based approaches to find outliers in high-dimensional space.

```python
from etsi.watchdog.models import IsolationForestModel

model = IsolationForestModel(contamination=0.1)

# Fit on healthy data
model.fit(ref[["age", "salary"]])

# Predict anomalies (-1 = anomaly, 1 = normal)
preds = model.predict(live[["age", "salary"]])
```

---

## Integrations

### Slack Alerting
Receive notifications instantly when drift is detected.

```python
from etsi.watchdog import Monitor

monitor = Monitor(ref)

# Enable Slack Integration
monitor.enable_slack_alerts(
    token="xoxb-your-slack-token",
    channel="#data-alerts"
)

# Run monitoring (Alerts sent automatically if drift found)
monitor.watch(live, features=["age"])
```

---

## Contributing

Pull requests are welcome!

Please refer to [CONTRIBUTING.md](https://github.com/etsi-ai/etsi-watchdog/blob/main/CONTRIBUTING.md) and [CODE_OF_CONDUCT.md](https://github.com/etsi-ai/etsi-watchdog/blob/main/CODE_OF_CONDUCT.md) before submitting a Pull Request.

---

## Join the Community

Connect with the **etsi.ai** team and other contributors on our Discord.

[![Discord](https://img.shields.io/badge/Discord-Join%20the%20Server-7289DA?style=for-the-badge&logo=discord&logoColor=white)](https://discord.com/invite/VCeY6H72rq)

---

## License

This project is distributed under the **BSD-2-Clause License**. See the [LICENSE](https://github.com/etsi-ai/etsi-watchdog/blob/main/LICENSE) for details.

---

Built with ❤️ by etsi.ai "Making machine learning simple, again."