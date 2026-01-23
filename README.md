# Container Security Cheat Sheet

**Talk:** Cloud & Containers: The Security Puzzle That Locks Tight  
**Event:** BSides London – Rookies Track, December 2025  
**Presenter:** Ashley Barker  
(LinkedIn: [ashleyjbarker](https://www.linkedin.com/in/ashleyjbarker/) | X: [@ajb_42](https://x.com/ajb_42) | GitHub & Stack Exchange: ajb42)

## Overview

This is the one-page cheat sheet from my 15-minute talk at BSides London Rookies Track (13 December 2025).

The framework is my own assembly, built from my thinking and opinions that are widely shared in the community. It was written and submitted before the second edition of Liz Rice's *Container Security* (November 2025) was published. While the core concerns (kernel risks, supply chain integrity, runtime containment) overlap in spirit with her first edition and with many of Kelsey Hightower's secure-by-default patterns, this approach is my own. It extends those ideas with a stronger focus on Zero Trust principles, cloud-native realities, measurable enterprise resilience (RTO, RPO, MTTR), and a unified defence graph that interconnects six layers: development standards, CI/CD pipelines, platform guardrails, runtime containment, visibility and response, and assurance and resilience.

The talk concludes with three immediate actions to move from fragmented tools to provable, interconnected defence.

## Talk Summary

Fragmentation is itself a vulnerability. With 82% of cloud breaches stemming from misconfiguration and human error (Exabeam 2025), point solutions leave dangerous gaps. This framework shifts from scattered tools to an interconnected defence graph, connecting:

- Foundations (shared kernel illusion, non-negotiable bans)  
- CI/CD (signed malware trap, SLSA L3+ plus VSA gates)  
- Guardrails (default-deny networking, mTLS, policy-as-code)  
- Runtime (eBPF behavioural detection, drift auto-kill)  
- Visibility (OpenTelemetry correlation, baselines)  
- Assurance (chaos engineering, RTO/RPO, non-repudiable proof)

It ends with three Monday actions:  
1. Mandate trust (SLSA L3+ plus VSA plus revocation)  
2. Enforce containment (ban root plus CAP_DROP plus default-deny)  
3. Deploy visibility (eBPF baselines plus alerts cluster-wide)

## Cheat Sheet

High-resolution version:  
[Container_Security_Cheat_Sheet.png](Container_Security_Cheat_Sheet.png)

Direct raw link:  
https://raw.githubusercontent.com/ajb42/BSides_London_2025/main/Container_Security_Cheat_Sheet.png

The sheet distils the complete framework:  
- 7-layer model  
- Core puzzle pieces  
- Required baseline controls  
- NIST 800-53 alignment  
- Target state metrics (100% attestation coverage, 0% privileged pods, measured RTO/RPO)

## License

MIT License – feel free to use, modify and share (with attribution). See [LICENSE](LICENSE) for the full text.

## Connect

- LinkedIn: [ashleyjbarker](https://www.linkedin.com/in/ashleyjbarker/)  
- X: [@ajb_42](https://x.com/ajb_42)  
- GitHub & Stack Exchange: ajb42

Feedback, questions or interest in bringing this talk to your team or event? Reach out on LinkedIn or X.

Views expressed are my own and do not necessarily reflect those of my employer.
