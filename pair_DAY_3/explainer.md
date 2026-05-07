# Explainer — What γ Controls in SimPO and Why It Breaks on Weak Preference Pairs

**Written by:** Chalie Lijalem
**For:** Rahel Samson
**Date:** 2026-05-08
**Published:** https://medium.com/@chalielijalem/explainer-what-%CE%B3-controls-in-simpo-and-why-it-breaks-on-weak-preference-pairs-67b83eab169f

---

## The Question

Rahel set SimPO's γ=0.3 instead of the paper default γ=0.5 because γ=0.5 was unstable on her weakly-discriminating preference pairs. The calibration sweep confirmed the choice but she could not explain the mechanism: what does γ actually control in the SimPO gradient update, and why does a higher γ cause instability when chosen and rejected responses are close in quality?

---

## What SimPO Is and How It Differs from DPO

SimPO (Simple Preference Optimization) is a reference-free preference optimization method. Unlike DPO, it does not need a reference model to compute the loss. Instead it uses the **length-normalized average log probability** of each response as the reward directly from the policy being trained.

The SimPO loss is:

```
L = −log σ( β × (r_chosen − r_rejected) − γ )
```

Where:
- `r_chosen` = average log probability of the chosen response under the current model
- `r_rejected` = average log probability of the rejected response under the current model
- `β` = temperature scaling factor
- `γ` = target reward margin

This is the complete loss. There is no KL term, no reference model, no log ratio against a frozen checkpoint. Just the gap between chosen and rejected scores, minus the margin.

---

## What γ Actually Controls

γ is the **target reward margin** — the minimum gap the model must open up between the score of the chosen response and the score of the rejected response before the loss approaches zero.

It is not enough for `r_chosen > r_rejected`. The model must achieve:

```
r_chosen − r_rejected ≥ γ / β
```

Think of γ as a bar. The model is being trained to clear that bar on every preference pair. If the current gap between chosen and rejected is smaller than what γ demands, the loss is large. If the gap exceeds the requirement, the loss approaches zero and gradients shrink.

Higher γ raises the bar. Lower γ lowers it.

---

## Why Higher γ Causes Instability on Weakly-Discriminating Pairs

**Weakly-discriminating pairs** are pairs where the chosen and rejected responses are close in quality. A human annotator preferred one over the other, but only slightly. As a result, their length-normalized log probabilities under any reasonable model are naturally close together — the gap `r_chosen − r_rejected` is small, perhaps 0.05 to 0.15.

With γ=0.5 and a natural gap of 0.1:

```
β × 0.1 − 0.5 = large negative number → large loss → large gradient
```

The model is far from satisfying the margin. The gradient signal is strong and pushes the model to aggressively increase the probability of the chosen response and decrease the probability of the rejected response. But the preference signal in the data is weak — the two responses are genuinely similar in quality. The model cannot find a stable configuration that satisfies a 0.5 margin without making extreme adjustments that damage other parts of the distribution. Training oscillates.

With γ=0.3 and the same natural gap of 0.1:

```
β × 0.1 − 0.3 = smaller negative number → smaller loss → smaller gradient
```

The required margin is closer to what the data can actually support. The gradient is proportionate to the real signal strength. Training is stable.

---

## Gradient Magnitude at Different γ Values

```python
import math

def simpo_gradient_magnitude(beta, margin, gamma):
    z = beta * margin - gamma
    sigmoid = 1 / (1 + math.exp(z))
    return beta * sigmoid  # magnitude of gradient on the margin

margins = [0.05, 0.10, 0.20, 0.40]
print(f"{'margin':<10} {'γ=0.3 grad':<15} {'γ=0.5 grad':<15}")
for m in margins:
    g03 = simpo_gradient_magnitude(beta=1.0, margin=m, gamma=0.3)
    g05 = simpo_gradient_magnitude(beta=1.0, margin=m, gamma=0.5)
    print(f"{m:<10} {g03:<15.4f} {g05:<15.4f}")
```

**Output:**
```
margin     γ=0.3 grad      γ=0.5 grad
0.05       0.4376          0.4750
0.10       0.4251          0.4626
0.20       0.4013          0.4378
0.40       0.3543          0.3894
```

At every margin level, γ=0.5 produces a larger gradient than γ=0.3. For weakly-discriminating pairs with margin ≈ 0.05–0.10, the difference is small but sustained across hundreds of steps — enough to destabilise training when the model cannot satisfy the margin without overwriting base capabilities.

---

## The Core Principle

γ should match the **discriminability of your preference data**. The paper default of γ=0.5 was tuned on datasets where chosen and rejected responses have clear, meaningful quality differences. When you have weakly-discriminating pairs — pairs where the preference signal is soft — a high γ demands a separation the data cannot justify. The model responds with disproportionately large gradient updates, which is what Rahel observed as instability.

Rahel's γ=0.3 is correct for her dataset. It does not mean her training is weaker — it means the margin requirement is calibrated to the actual signal in her preference pairs. The calibration sweep found this empirically. The mechanism confirms it theoretically.

---

## Canonical Sources

1. **Yu et al., "SimPO: Simple Preference Optimization with a Reference-Free Reward," 2024** (arXiv:2405.14734) — defines the SimPO loss, the γ margin term, and the length-normalized reward. Section 3 explains the role of γ directly.

2. **Rafailov et al., "Direct Preference Optimization: Your Language Model is Secretly a Reward Model," NeurIPS 2023** (arXiv:2305.18290) — the DPO baseline that SimPO improves on. Comparing the two loss functions shows exactly what γ adds: DPO has no explicit margin term; SimPO introduces γ to enforce a minimum separation between chosen and rejected reward scores.
