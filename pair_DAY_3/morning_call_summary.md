# Morning Call Summary — Day 3

**Date:** 2026-05-08
**Participants:** Chalie Lijalem & Rahel Samson

---

Rahel's original question described a symptom — γ=0.5 was unstable — but not the mechanism. Interrogation by Chalie ("what is γ actually doing to each gradient update?") moved it to the specific observable: instability on weakly-discriminating pairs where chosen and rejected responses are close in quality, and the consequence of being unable to tune γ for a new domain. Chalie's original question was two parts; Rahel interrogated which part would change a concrete artifact, and the ORPO vs DPO comparison was set aside as taxonomy with no artifact consequence. The LoRA part was sharpened: Chalie named `training/train_orpo.py` (r=32, lora_alpha=32) as the specific artifact and the stake — knowing whether to tune rank up or down if the adapter fails to generalise. Both questions were confirmed unambiguous and resolvable in one explainer.
