# Contributing to PS5 14.00 / 14.10 Jailbreak Research

Thanks for helping improve the accuracy, attribution, and reproducibility of this research repository.

## Useful contributions

- Correcting documentation that does not match the supplied source.
- Recording the provenance of reused material.
- Clarifying which routines are executed by the default program path.
- Adding carefully documented test observations and negative results.
- Clarifying platform assumptions and firmware-specific limitations.
- Comparing static-analysis conclusions with measured behavior.
- Improving attribution for prior public research.

## Evidence standards

When reporting a finding, distinguish clearly between:

1. **Static analysis** — inferred from source, disassembly, or other offline inspection.
2. **Runtime observation** — observed during a test, with the target environment recorded.
3. **Hypothesis** — a proposed explanation that has not yet been demonstrated.

Where possible, include the exact console model, firmware version, environment details, attempt count, expected behavior, observed behavior, and supporting logs. Do not describe a hypothesis as confirmed solely because a comment or function name says so.

## Source accuracy and firmware scope

In the supplied source, `reclaim_and_confirm()` and `detect_double_free()` are defined but are not called by `main()`. Documentation should preserve this distinction.

The repository title refers to PS5 14.00 / 14.10 jailbreak research, but the supplied source alone does not establish compatibility or a working jailbreak on either version. Do not generalize findings from other firmware versions without evidence.

## Attribution and licensing

Before submitting copied or adapted material, identify its origin and preserve applicable notices and license terms. Do not remove third-party attribution or assume that publicly available code has no reuse restrictions. The current `LICENSE` file is a licensing-status statement, not a standard open-source license.

## Pull requests

A useful pull request should explain:

- What changed and why.
- Which files and claims are affected.
- What evidence supports the change.
- What remains unverified.

Keep claims specific, factual, and appropriately qualified.
