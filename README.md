# ScanSafe — On-Device QR Phishing Detection

> **"Look before you tap."**

**Stack:** Android (Kotlin) · OpenCV · Heuristic URL Analysis  
**Type:** Applied Security Research — AIoT Lab, Grambling State University  
**Supervisor:** Dr. Vasanth Iyer  
**Status:** 🔨 In Development (Demo: March 2026)

---

## Overview

**ScanSafe** is an on-device Android security app that uses classical computer vision to detect and risk-score QR codes in real time — with **no cloud dependency and no pretrained machine learning models**.

Point your camera at a QR code. ScanSafe decodes it, analyzes the URL for phishing indicators, and gives you a clear **green / yellow / red** verdict before you ever tap.

---

## Research Question

*Can a fully on-device, classical computer vision pipeline provide meaningful QR phishing protection in low-connectivity and privacy-sensitive environments — and what are the tradeoffs in false positive and false negative rates?*

---

## Why This Matters

QR phishing is a growing attack vector — fake scholarship links, fraudulent financial aid portals, and spoofed event registrations distributed via printed QR codes on bulletin boards. Most QR scanners simply open the URL with no warning.

Existing solutions either:
- Require a cloud lookup (privacy risk, fails offline), or
- Depend on pretrained ML models (opaque, hard to audit)

ScanSafe is built on **explainable, auditable, on-device logic** — every risk flag has a reason you can read.

---

## How It Works

### Vision Pipeline (following Dr. Iyer's OpenCV guide)

```
Camera Frame
    ↓
Grayscale Conversion
    ↓
Gaussian Blur  (noise reduction)
    ↓
Canny Edge Detection  (find boundaries)
    ↓
FindContours  (isolate QR finder pattern squares)
    ↓
QR Decode  (extract URL string)
    ↓
URL Risk Scorer
    ↓
Green / Yellow / Red Verdict
```

### URL Risk Scoring Rules

Each rule adds to a cumulative risk score:

| Rule | Score | Reasoning |
|------|-------|-----------|
| IP address instead of domain | +3 | Legitimate services use domain names |
| Brand name + character substitution (paypa1, amaz0n) | +3 | Classic phishing pattern |
| HTTP instead of HTTPS | +2 | No transport encryption |
| Excessive subdomain depth | +2 | Common in phishing infrastructure |
| Unusually long or random-looking path | +1 | Obfuscation indicator |

**Score mapping:**
- 0–2 → 🟢 Safe
- 3–5 → 🟡 Suspicious — proceed with caution
- 6+ → 🔴 High Risk — do not open

Every verdict includes a **plain-English explanation** of which rules fired.

---

## Evaluation Plan

Testing against 20 QR codes (10 safe, 10 phishing-pattern):

| Metric | Description |
|--------|-------------|
| Detection latency | Time from scan to verdict (ms) |
| True positive rate | Phishing QRs correctly flagged |
| False positive rate | Safe QRs incorrectly flagged |
| Battery impact | Measured via Android battery stats |

---

## Repo Structure

```
scansafe/
├── README.md
├── app/
│   └── src/main/
│       ├── java/com/patselby/scansafe/
│       │   ├── MainActivity.kt
│       │   ├── QRScanner.kt
│       │   └── URLRiskScorer.kt
│       └── res/layout/
│           └── activity_main.xml
├── research/
│   ├── research_question.md
│   ├── pipeline_diagram.png
│   └── evaluation_results.md
└── docs/
    └── demo_notes.md
```

---

## Connection to Lab Work

This project is developed as part of research in the **AIoT Lab at Grambling State University** under Dr. Vasanth Iyer. It applies the classical computer vision pipeline from Dr. Iyer's Android OpenCV guide to a real-world security problem — demonstrating that meaningful threat detection can run entirely at the edge of a constrained mobile device without cloud or ML dependencies.

This aligns with the lab's broader focus on **on-device edge AI** for privacy-sensitive and low-connectivity environments.

---

## Developer

**Patrick Selby**  
B.S. Cybersecurity — Grambling State University  
[GitHub](https://github.com/pat-selby) · [LinkedIn](https://www.linkedin.com/in/patrick-ennin-selby-136253301) · [Portfolio](https://portfolio-v2-gzkg.vercel.app)
