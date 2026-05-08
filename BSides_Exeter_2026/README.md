# Cloud & Containers: The Security Puzzle That Locks Tight
**Talk:** Cloud & Containers: The Security Puzzle That Locks Tight  
**Event:** BSides Exeter 2026  
**Presenter:** Ashley Barker  
(LinkedIn: [ashleyjbarker](https://www.linkedin.com/in/ashleyjbarker/) | X: [@ajb_42](https://x.com/ajb_42) | GitHub: [ajb42](https://github.com/ajb42) | Substack: [ajb042](https://substack.com/@ajb042) | infosec.exchange: [ajb42](https://infosec.exchange/@ajb42))
**Recording:** [Youtube](https://www.youtube.com/watch?v=dKxZpcMWCtQ)

---

## Overview

This is the slide deck and supporting materials from my 33–35 minute talk at BSides Exeter 2026.

The framework is my own assembly, built from my thinking and opinions that are widely shared in the community. It builds on Liz Rice's *Container Security* (2nd Ed., November 2025) and draws on Kelsey Hightower's secure-by-default patterns, extending both with a stronger focus on operational detection, blue team ownership, and what good triage and response actually looks like in containerised environments. The approach connects six layers — development standards, CI/CD pipelines, platform guardrails, runtime containment, visibility and response, and assurance and resilience — and asks a different question from the architectural version of this talk: not how you build the puzzle, but how you watch it, and what you do when something fires.

The talk concludes with four actions for Monday, each requiring both technical configuration and a named human owner.

### Content

**Slides:** [PDF slides](https://github.com/ajb42/BSides_Conferences/blob/b31ab00f076e37320489a0e5b91efdb3ba2c4bbf/BSides_Exeter_2026/Container_Def_Sec_v1.2.pdf)

**Talk Guide:** [AI generated slide overview](https://github.com/ajb42/BSides_Conferences/blob/b31ab00f076e37320489a0e5b91efdb3ba2c4bbf/BSides_Exeter_2026/Exeter_Slide_Context_Guide.md)

**Cheat Sheet:** [Exeter Cheat Sheet](https://github.com/ajb42/BSides_Conferences/blob/b31ab00f076e37320489a0e5b91efdb3ba2c4bbf/BSides_Exeter_2026/Cheat_Sheet_Exeter.png)

If you would like any of the images of further information please message me.

---

## Talk Summary

Fragmentation is itself a vulnerability. With 82% of cloud breaches stemming from misconfiguration and human error (Verizon DBIR 2025), and cloud intrusion volume up 75% year-on-year (CrowdStrike 2025), point solutions that don't generate correlated signals leave dangerous operational gaps. This talk takes the same six-layer defence graph as the architectural version, but walks it entirely from the blue team perspective — what each layer generates, who reads it, and what the detection and response loop looks like.

The six layers covered:

- **Foundations** — the shared kernel illusion, CVE exposure, and the four-word ban that forms the non-negotiable baseline
- **CI/CD** — the pipeline as a security control point, Rekor as a queryable provenance store, and SLSA L3+ VSA gates as the first line of supply chain defence
- **Who Sees What** — the structural gap between AppSec and SOC, and why deployment provenance is the missing link at alert triage
- **Cloud guardrails** — cloud-layer controls as detection telemetry, not just policy enforcement
- **Kubernetes** — every admission deny and policy violation as a data point in the SIEM
- **Runtime** — eBPF behavioural detection, impossible-behaviour baselines, and why shift-left alone is blind to runtime threats
- **Visibility** — OpenTelemetry correlation, external log shipping, and the ephemeral container forensics problem
- **SOC triage loop** — L1/L2/L3 runbooks, Rekor at triage, the kill-or-preserve decision, and measured MTTR

It ends with four actions for Monday:
1. Mandate trust (SLSA L3+ + VSA + automated revocation)
2. Enforce containment (ban root + CAP_DROP ALL + default-deny NetworkPolicy)
3. Deploy visibility (eBPF to SIEM + K8s audit + external log shipping)
4. Detect and respond (tuned Falco rules + tested runbooks + measured MTTR + people who own each layer)

## Cheat Sheet

The sheet distils the complete framework:
- 6-layer defence model with signal ownership per layer
- Required baseline controls (non-negotiable bans)
- NIST 800-53 alignment
- Target state metrics (100% attestation coverage, 0% privileged pods, SLSA L3+, measured RTO/RPO/MTTR)
- SOC triage loop with SLA targets

## License

MIT License – feel free to use, modify and share (with attribution). See [LICENSE](LICENSE) for the full text.

## Connect

- LinkedIn: [ashleyjbarker](https://www.linkedin.com/in/ashleyjbarker/)
- X: [@ajb_42](https://x.com/ajb_42)
- GitHub: [ajb42](https://github.com/ajb42)
- Substack: [ajb042](https://substack.com/@ajb042)
- infosec.exchange: [ajb42](https://infosec.exchange/@ajb42)

Feedback, questions or interest in discussing any ideas further — please reach out.

*Views expressed are my own and do not necessarily reflect those of my employer.*
