# PS5 14.00 / 14.10 Jailbreak Proof of Concept Using Public-Reference (Publapse)

**Experimental PS5 security research and Jailbreak Proof of Concept public-reference collection.**

`publapse` organizes research notes, references, and experimental source material related to PlayStation 5 security and kernel behavior. The repository is intended as a general research reference for the PS5 14.00 / 14.10 jailbreak-research topic, while keeping implementation details and compatibility claims tied to the evidence actually available.

**Project status:** Experimental research. The supplied source includes an AIO lifecycle/concurrency test harness and exploratory memory-reuse instrumentation. The source alone does **not** demonstrate a working jailbreak for firmware 14.00 or 14.10, nor a reliable end-to-end privilege-escalation chain.

**Research disclaimer:** This is not presented as a finished jailbreak or turnkey kernel exploit. Treat compatibility statements, comments, and proposed later-stage techniques as unverified unless they are supported by reproducible evidence for the exact target environment.

---

## Table of Contents

- [Overview](#overview)
- [Background and Related Public Research](#background-and-related-public-research)
- [Research Objectives](#research-objectives)
- [Implementation Overview](#implementation-overview)
- [Source Code Status](#source-code-status)
- [Firmware Compatibility](#firmware-compatibility)
- [Verification and Reproducibility](#verification-and-reproducibility)
- [Known Limitations](#known-limitations)
- [Repository Organization](#repository-organization)
- [Contributing](#contributing)
- [Attribution and Licensing](#attribution-and-licensing)
- [Responsible Research](#responsible-research)

## Overview

The repository's current code artifact is `aio_lapse_exploit.c`, a supplied experimental source file concerned with asynchronous I/O (AIO) request handling, concurrent deletion, object lifetime, and possible memory-reuse behavior in a PlayStation 5 kernel environment.

The AIO subsystem exposes operations for submitting requests, polling their state, cancelling work, and deleting request records. Concurrent operations can make request-state transitions and cleanup behavior worth investigating, but the presence of a test harness does not establish that a race is reachable or exploitable on a particular firmware.

The supplied source includes:

- A low-level system-call wrapper and target-specific syscall definitions.
- Helpers for submitting, polling, cancelling, and deleting AIO requests.
- A repeated AIO workload with concurrent request deletion.
- CPU-affinity helpers and diagnostic reporting.
- Socket-backed activity and IPv6 socket-option allocation experiments.
- Request-state, error, and progress instrumentation.
- Additional exploratory routines related to allocation reuse and double-free detection.

The source includes proposed later-stage research ideas. Those comments should be treated as hypotheses or unfinished work unless separate evidence demonstrates the behavior.

## Background and Related Public Research

This repository sits within a broader public research landscape covering PlayStation kernel internals, FreeBSD-derived behavior, and AIO-related attack surfaces. The following links are useful background references; their inclusion does not prove that every technique in `publapse` derives from them or that their findings apply to firmware 14.00 / 14.10.
- [Public code release](https://gist.github.com/zecoxao/80dff057233c1f203352ff8bf317294d): a publicly shared snapshot of the ongoing Prolapse research effort, brought to public attention by Jose Coixao (@notnotzecoxao).
- [PSFree — `lapse.mjs`](https://github.com/rezaadi0105/PSFree/blob/main/lapse.mjs): public research concerning an AIO-related kernel exploit for earlier PS4/PS5 firmware generations.
- [PS5-14.10-Research — AIO batch investigation](https://github.com/ZigZagBoi1/PS5-14.10-Research/blob/master/docs/AIO_BATCH_14_10.md): notes on AIO syscalls and batch request handling associated with firmware 14.10.
- [PS5-14.10-Research](https://github.com/ZigZagBoi1/PS5-14.10-Research): a broader public research project examining PS5 syscall behavior and kernel attack surfaces.

Previously published vulnerabilities, implementations, and findings remain attributable to their original researchers. Where code or methods are reused, their provenance and license obligations should be documented explicitly.

## Research Objectives

The research questions reflected in the supplied source include:

1. **Request lifecycle consistency:** How do AIO submission, cancellation, polling, and deletion interact under concurrent execution?
2. **Cleanup and object lifetime:** Can concurrent cleanup paths produce inconsistent state or lifetime behavior?
3. **Scheduling and timing:** How do CPU placement, workload timing, and system load affect observations?
4. **Memory reuse:** What evidence, if any, shows that released kernel allocations are reused in ways relevant to the investigated lifecycle?
5. **Evidence quality:** Which conclusions come from source inspection, which have been reproduced at runtime, and which remain hypotheses?

These are research questions, not claims that every condition has been reproduced or that a full exploitation chain has been achieved.

## Implementation Overview

The primary source file is `aio_lapse_exploit.c`. The summary below describes the supplied code at a high level; it is not a claim of successful runtime behavior on any specific console or firmware.

### 1. Low-level system-call interface

The source defines an inline assembly wrapper for issuing system calls and target-specific identifiers for AIO operations and related facilities. These definitions are platform-specific and should not be assumed to apply unchanged to desktop FreeBSD, Linux, or other console firmware versions.

### 2. AIO request management

The source defines request descriptors and helpers for the relevant AIO operations. The main test path creates request batches, observes request state, attempts cancellation, and exercises concurrent deletion. It also reports selected state values and errors for diagnostic purposes.

### 3. Multithreading and scheduling

The harness uses separate execution contexts for the primary test loop, competing deletion activity, and background allocation activity. Affinity helpers query the permitted CPU mask and attempt to select CPUs available to the process. These controls may help characterize test conditions, but they do not make scheduling deterministic or guarantee that a race occurs.

### 4. Socket-backed workloads

Socket pairs and socket options create additional activity around request processing and resource cleanup. IPv6 socket-option experiments appear intended to investigate allocator behavior and possible memory reuse. Whether an option is accepted, which allocator zone it uses, or whether it can reuse a particular object is target-dependent and requires evidence.

### 5. Observation and diagnostic routines

The source includes request-state polling, per-request error collection, affinity reporting, socket-option checks, tagged allocation observations, and progress reporting. Additional allocation-oracle and double-free-detection routines are present, but their presence must not be mistaken for their execution in the default program path.

### 6. Exploratory later-stage research

Comments describe possible follow-up investigations into memory reuse and consequences of inconsistent object lifetimes. They do not establish an arbitrary-free primitive, kernel read/write capability, an ASLR bypass, credential modification, or completed privilege escalation.

## Source Code Status

The source should be evaluated by distinguishing code that is present, code that runs from the default entry point, and behavior that has been experimentally established.

| Component | Status supported by the supplied source |
|---|---|
| Low-level syscall wrapper and AIO helpers | Defined in source |
| Thread creation and affinity helpers | Defined in source |
| Repeated AIO request/deletion workload | Present in the main test path |
| Background socket-option activity | Present in source |
| Request-state and progress reporting | Present in source |
| `reclaim_and_confirm()` | Defined, but not called by `main()` |
| `detect_double_free()` | Defined, but not called by `main()` |
| Completed later-stage exploitation sequence | Not demonstrated by the supplied source |
| Working jailbreak for PS5 firmware 14.00 / 14.10 | Not established by the supplied source |
| Reliable end-to-end privilege escalation | Not established |

This table describes the supplied file only. It does not make claims about separate experiments or results that may exist outside this repository. A defined function is not the same as a successful test, and a comment describing a mechanism is not proof that the mechanism works.

## Firmware Compatibility

The repository title refers to PS5 14.00 / 14.10 jailbreak research as a broad topic label. **The supplied AIO source does not, by itself, establish that the code works on either firmware version.**

The source comments contain claims concerning firmware around the 14.10 generation and a possible change in 14.20. Those are source comments requiring independent evidence, not universal compatibility guarantees for firmware 14.00 or 14.10.

A useful compatibility record should distinguish:

- Firmware versions directly tested on physical hardware.
- Versions examined through static analysis only.
- Versions whose behavior is inferred by comparison.
- Versions not tested.
- Versions for which relevant code or observations are unavailable.

Do not label a version confirmed vulnerable or confirmed fixed solely because it appears in a source comment. Evidence that a syscall exists or behaves normally does not establish that a suspected race is reachable or exploitable.

## Verification and Reproducibility

For each significant finding, record the following information where available:

- **Target:** Console model and exact system-software version.
- **Environment:** Relevant development tools, runtime assumptions, and test conditions.
- **Method:** What the experiment was designed to measure.
- **Expected result:** The behavior being investigated.
- **Observed result:** What actually happened, including negative results.
- **Repeatability:** Attempt count and frequency of the observed outcome.
- **Evidence:** Logs, screenshots, static-analysis notes, or other supporting material.
- **Limitations:** Alternative explanations, missing evidence, and unresolved questions.

Label static-analysis conclusions as static analysis. Runtime claims should include evidence from the environment in which they were observed. A failure to reproduce a race is useful information and should be recorded rather than omitted.

## Known Limitations

### The default path does not call every defined routine

`reclaim_and_confirm()` and `detect_double_free()` are present in the source but are not called by `main()`. Therefore, the default program flow does not automatically perform every verification step described in comments or implied by the presence of those functions.

### Timing-dependent behavior is not guaranteed

Concurrent execution varies with scheduling, system load, resource availability, and other environmental factors. A run without an observed anomaly does not prove that the behavior is impossible.

### Allocator assumptions require evidence

Assumptions about object sizes, allocation reuse, and the relationship between socket-option buffers and kernel-managed allocations require target-specific evidence.

### Firmware claims require independent validation

Firmware changes may affect syscall behavior, validation logic, object layout, and scheduling. Findings should not be generalized to untested versions.

### No completed exploit is established

The supplied source does not establish a reliable chain from the concurrency experiment to arbitrary kernel memory access or privilege escalation, and it does not establish a working jailbreak for PS5 14.00 / 14.10.

## Repository Organization

The current repository package contains:

- `README.md` — Project scope, status, background, and limitations.
- `aio_lapse_exploit.c` — The supplied experimental AIO research source.
- `NOTICE.md` — Attribution and provenance checklist.
- `CONTRIBUTING.md` — Contribution and evidence-reporting guidelines.
- `CHANGELOG.md` — Documentation/package history.
- `LICENSE` — Current licensing-status statement; not a standard open-source license.

Additional directories or test records should be listed here only after they have actually been added.

## Contributing

Contributions that improve accuracy and reproducibility are welcome, including:

- Reviewing source assumptions and platform-specific definitions.
- Documenting reproducible observations and negative results.
- Comparing static-analysis findings with measured behavior.
- Identifying inconsistencies between comments and executed code.
- Improving test reporting and compatibility records.
- Adding attribution for prior work and documenting provenance.

Include the target environment, supporting evidence, and limitations when reporting a finding. Keep demonstrated behavior separate from proposed explanations, and do not describe an untested hypothesis as a confirmed vulnerability or completed exploitation stage. See [`CONTRIBUTING.md`](CONTRIBUTING.md).

## Attribution and Licensing

This repository consolidates research material in an area with substantial prior public work. Identifiable upstream sources, original authors, publications, and reused implementations should be credited in the appropriate locations. Preserve existing copyright notices and license terms.

The current `LICENSE` file does not grant a standard open-source license. Before public redistribution, establish the provenance and licensing status of all included material. Public accessibility alone does not imply permission to copy, modify, or redistribute code. See [`NOTICE.md`](NOTICE.md) for the current provenance checklist.

## Responsible Research

This project is intended for authorized security research, analysis, and education on systems the researcher owns or has permission to test. Kernel-level experiments can destabilize a system or cause data loss. The repository makes no guarantee that the code is safe, portable, reliable, or compatible with a particular firmware version.

## Final Status

`publapse` is an experimental PS5 research repository with a supplied AIO lifecycle/concurrency source file and supporting documentation. Its value should be assessed through source accuracy, attribution, reproducibility, and supporting evidence. Previously published findings should remain credited to their original researchers, and unfinished work should remain labeled as unfinished.

The goal is transparent research—not an unsupported claim of a completed jailbreak or universal firmware exploit.
