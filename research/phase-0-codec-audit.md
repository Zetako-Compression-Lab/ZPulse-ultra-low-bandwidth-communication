# Phase 0 — Original Codec Audit

**Date:** 12 October 2026. **Evidence:** internal Phase 0 audit report. **Status:** characterization, not product release.

## Scope

Two allocation-free C integer implementations were characterized on Darwin ARM64 with Apple Clang 21.0.0. Standard application configuration represents quantized position and preset index in **7 bytes**. The two implementations have incompatible coordinate quantization semantics despite similar APIs.

## Executed checks

| Check | Observed result |
|---|---|
| Original demos | Both compiled and exited successfully |
| Strict C11 builds | `-Wall -Wextra -Wpedantic -Werror`; both demos and probes passed |
| Standard-profile round trips | 10,000 deterministic samples per codec; pinned fixtures passed |
| Width/selected-coordinate census | 123,008 cases per codec |
| Original reference codec drift | 0 / 123,008 |
| Original 32-bit codec re-encode drift | **4,432 / 123,008** |
| Probe contract assertions | 547,445 and 547,444; zero unexpected failures |
| ASan / UBSan | No detected errors; known drift reproduced |
| Original research files | Nine hashes unchanged |

The 4,432 drift cases are **not** a production message failure rate. They refer to selected coordinate/width combinations where encode → decode → re-encode changes the bytes. The standard 21/22 configuration was stable in these tests. Position quantization is controlled-loss; preset indices are exact for configured dictionaries.

## Reproducibility boundaries

Audit was performed on the stated macOS host, not on target MCUs, radios or alternative compilers. Full source, probes and raw fixtures are not included in this public evidence-only repository. Results should not be presented as an embedded portability certification.

## Follow-on

The drift was investigated experimentally in [Phase 0.5](phase-0-5-protocol-research.md).
