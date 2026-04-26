# Compound Interest Calculator — Android App

A clean, modern Android app built with **Jetpack Compose + Material3** that calculates compound interest and visualises growth over time.

## Features

| Feature | Description |
|---|---|
| **Formula Precision** | A = P(1 + r/n)ⁿᵗ with support for Daily / Weekly / Monthly / Quarterly / Semi-Annually / Annually compounding |
| **Monthly Contributions** | Optional regular contributions using the future-value annuity formula |
| **Growth Chart** | Interactive line chart (MPAndroidChart) showing *Total Value* vs *Amount Invested* year-by-year |
| **What If?** | Slider to explore "what if I started X years earlier?" with instant comparison |
| **Frequency Impact** | Side-by-side table showing how each compounding frequency affects the final result |
| **Input Validation** | Numeric keyboards, single-decimal enforcement, and graceful zero-rate handling |
| **Real-time Updates** | Results and chart refresh instantly as any input changes |
| **Dynamic Color** | Material You dynamic theming on Android 12+; static purple/teal palette on older devices |

## Tech Stack

- Kotlin
- Jetpack Compose + Material3
- MPAndroidChart v3.1.0 (via JitPack)
- Gradle 8.6 / AGP 8.2.2 / Kotlin 1.9.22

## Building

```bash
./gradlew assembleDebug
```

Install on device / emulator (API 24+):

```bash
./gradlew installDebug
```
