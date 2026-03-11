# ScanSafe — AI Coding Directives

## Layer 1: Project Identity
ScanSafe is an on-device Android security app that detects and risk-scores QR code URLs in real time using classical computer vision. It runs entirely on-device — no cloud calls, no pretrained ML security models, no internet required for threat detection.

**Tagline:** "Look before you tap."
**Research Question:** Can a fully on-device classical CV pipeline provide meaningful QR phishing protection in low-connectivity and privacy-sensitive environments — and what are the tradeoffs in false positive and false negative rates?

---

## Layer 2: Folder Structure

```
scansafe/
├── app/
│   └── src/main/
│       ├── java/com/patselby/scansafe/
│       │   ├── MainActivity.kt          # Entry point, camera permission handling
│       │   ├── CameraFragment.kt        # Live camera feed using CameraX
│       │   ├── QRScanner.kt             # OpenCV pipeline (blur → Canny → contours)
│       │   ├── URLRiskScorer.kt         # Heuristic URL risk scoring logic
│       │   └── ResultFragment.kt        # Green/yellow/red verdict UI
│       └── res/
│           ├── layout/
│           │   ├── activity_main.xml
│           │   ├── fragment_camera.xml
│           │   └── fragment_result.xml
│           └── values/
│               ├── colors.xml           # green=#2ECC71, yellow=#F39C12, red=#E74C3C
│               └── strings.xml
├── claude.md                            # This file
├── README.md
└── research/
    └── research_question.md
```

---

## Layer 3: Tech Stack & Constraints

### Languages & Tools
- **Language:** Kotlin
- **Min SDK:** API 24 (Android 7.0)
- **Camera:** CameraX (androidx.camera)
- **Computer Vision:** OpenCV for Android (classical pipeline only)
- **QR Decode:** ML Kit Barcode Scanning (decode only — NOT a security model)
- **Build:** Gradle with Kotlin DSL

### OpenCV Pipeline (strictly classical — no ML)
The vision pipeline must follow this exact sequence on every camera frame:
1. Convert frame to grayscale
2. Apply Gaussian blur (kernel size 5x5) — noise reduction
3. Apply Canny edge detection — find boundaries
4. Run FindContours — isolate shapes
5. Identify QR finder pattern (three square markers)
6. Pass detected QR to ML Kit for URL string extraction only
7. Send URL string to URLRiskScorer

### URL Risk Scoring Rules
URLRiskScorer must evaluate these rules in order and accumulate a score:

| Rule | Points | Reason |
|------|--------|--------|
| URL is an IP address (not domain) | +3 | Legitimate services use domain names |
| Brand name + character substitution (paypa1, amaz0n, g00gle) | +3 | Classic phishing pattern |
| HTTP instead of HTTPS | +2 | No transport encryption |
| Subdomain count > 3 | +2 | Common in phishing infrastructure |
| Path length > 50 characters | +1 | Obfuscation indicator |
| Random-looking domain (high consonant ratio) | +1 | Generated domain pattern |

**Score mapping:**
- 0–2 → GREEN — Safe
- 3–5 → YELLOW — Suspicious, proceed with caution
- 6+ → RED — High Risk, do not open

Every verdict must show a plain-English reason (e.g. "This URL uses an IP address instead of a domain name — a common phishing indicator.")

---

## Layer 4: Orchestration Rules (for AI agent)

- **Never** use cloud APIs for security decisions — all scoring must be local
- **Never** add pretrained ML models for URL classification — rules only
- **Never** store or log any scanned URLs — privacy by design
- **Always** ask before adding a new dependency to build.gradle
- **Always** keep MainActivity lean — delegate to fragments
- **Always** add a comment above each OpenCV step explaining what it does and why
- **Always** write URLRiskScorer as a pure function (input: String, output: RiskResult) so it can be unit tested independently
- When in doubt about UI, keep it minimal — one camera screen, one result screen

---

## Layer 5: Research Constraints

This app is being developed as part of applied security research in the AIoT Lab at Grambling State University under Dr. Vasanth Iyer. The classical CV constraint is intentional and must be preserved — it is the research contribution. Do not suggest replacing the OpenCV pipeline with a pretrained QR detection model even if it would be simpler.

The evaluation metrics are:
- Detection latency (ms from scan to verdict)
- True positive rate (phishing QRs correctly flagged)
- False positive rate (safe QRs incorrectly flagged)
- Battery impact

Code should be written with these metrics in mind — prefer efficient operations over elegant ones where there is a tradeoff.
