# 🛡️ Custom C2 Framework — Detection Research for Blue Teams

> **Detection engineering research for blue teams:** Detection engineering research for blue teams: lab-generated C2 telemetry, IOCs, MITRE ATT&CK mapping, and Sigma rules. Documentation only; no source code or binaries..

## ⚠️ Disclaimer

This repository contains **defensive security research only**. No source code or binaries are provided — only architecture documentation, behavioral analysis, IOCs, and detection content. All testing was conducted in a fully isolated VMware lab with no external network connectivity. The purpose of this work is to improve SOC and blue team detection capabilities against custom (non-commodity) C2 tooling that evades signature-based controls.

## 🎯 Research Question

**How do custom-built C2 frameworks — the kind that don't match any known signature — operate in practice, and what behavioral, network, and registry indicators allow SOC analysts to detect them reliably?**

To answer this, lab-generated C2-like telemetry was used as the object of study (no Metasploit, no Cobalt Strike, no Sliver) in an isolated lab to generate authentic telemetry. The framework was then analyzed from a defender's perspective across every phase: initial callback, command execution, feature modules (keylogging, screen capture, audio/video), persistence, and session recovery.

## 🛡️ Detection Content (Jump To)

- [Attack → Detection Mapping Table](#-attack--detection-mapping)
- [Recommended Defensive Controls](#-recommended-defensive-controls)
- [Sigma Rules for C2 Detection](#-sigma-rules-for-c2-detection)
- [MITRE ATT&CK Coverage](#-mitre-attck-coverage)
- [Lessons for Defenders](#-lessons-for-defenders)

## 🏗️ Framework Architecture (Object of Study)
