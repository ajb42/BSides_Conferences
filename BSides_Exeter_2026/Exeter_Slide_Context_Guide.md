# Slide Context Guide
## Cloud and Containers: The Security Puzzle That Locks Tight
### BSides Exeter 2026

**Ashley Barker**

**This page was AI Generated from the slides, talk script and transcript and summary notes for the presenter.**

This document is a companion to the BSides Exeter 2026 talk. It is written for anyone who wants to understand the content and intent of the talk without having seen the presentation. It is not a transcript. It describes what each slide covers, what it argues, and how it fits into the overall structure of the talk.

The deck contains 12 slides. The talk is a blue team talk. The lens throughout is detection, monitoring, and response rather than architecture or platform build. It covers the same six-layer framework as other versions of this talk, but asks a different question at every step: not how you build the controls, but what signals they generate, who should be reading those signals, and what to do when one fires.

---

## The Central Argument

Fragmentation is a vulnerability. Most organisations have the right tools but have not connected them. Every control in a cloud and container environment generates a signal. Most of those signals go unread, unrouted, or unconnected to anything else. This talk walks every layer of the stack and, at each layer, identifies the specific signal it produces, names who owns it, and shows how it feeds into a coherent detection and response loop.

The concept that anchors the whole talk is the **Unified Defence Graph**: the idea that every security control is a node, and every telemetry feed connecting those controls is an edge. The graph only has value when it is operationally integrated, meaning queryable, correlated, and acted on.

---

## Slides

---

### E1 — Title

**Visual Description**
Dark background with a faint circuit-board texture on the left edge. Large bold title centred. Presenter name bottom-left. Disclaimer bottom-right. No data, no diagrams, no callouts.

**Slide Synopsis**
The opening title slide. The talk title, presenter name, and event are the only elements on screen.

---

### E2 — The Scattered Puzzle

**Visual Description**
Six floating puzzle pieces on a dark background, each labelled with one of the talk's six security layers. The pieces are disconnected from one another, with a gap at the centre. A faint Kubernetes cluster diagram in the background places the pieces in context.

**Slide Synopsis**
Introduces the six security layers of the framework as currently fragmented and unconnected, and argues that the absence of integration between them is itself a vulnerability.

**Slide Details**
The six layers shown are: development standards, CI/CD pipeline, platform guardrails, runtime security, visibility and response, and assurance. Each piece carries the layer name and an icon representing the primary tool or concept within it. The central gap in the puzzle is annotated to make the point explicit: the vulnerability is not any single missing control, it is the absence of connection between the controls that exist.

Two specific problems are introduced here that are developed and resolved across the rest of the talk. The first is the correlation gap. Each layer generates security signals, but most organisations are reading those signals in isolation. A Falco runtime alert and an AppSec SBOM finding may relate to the same incident, but if the teams receiving them have no shared visibility, they will be investigated separately and the connection will be missed. The second is the ephemeral container problem. Containers are short-lived by design. When a container involved in an incident is terminated, its local filesystem state is lost with it. This creates a forensic challenge that shapes how logging architecture needs to be designed, and it affects what evidence is available during an investigation.

**Additional Notes**
Both problems introduced here are intentionally brief at this point. The correlation gap is addressed structurally in slide five and operationally in slide ten. The ephemeral container problem is explained mechanically in slide eight and resolved in slide ten. The slide is designed to frame the problem space before any technical content is introduced, so that each subsequent layer is understood not just as a control but as a signal source that either contributes to or undermines the whole.

---

### E3 — Foundations: The Shared Kernel Illusion

**Visual Description**
An exploded isometric diagram of the full container stack, from CPU and kernel at the base up through the operating system, systemd, kubelet, containerd, runc, pods, containers, and applications at the top. Attack arrows annotated with real CVE references point at the runc and kernel layers. A detection panel on the right maps each layer to the audit event it generates and the SIEM destination it routes to. A second panel shows the Four-Word Ban and the mandatory pod security defaults.

**Slide Synopsis**
Establishes that containers are not virtual machines, that the isolation between containers is a software boundary over a shared kernel, and that breaking through it means reaching the host.

**Slide Details**
Containers share the Linux kernel of the host they run on. The isolation mechanisms, namespaces, cgroups, and Linux capabilities, are software constructs. They are not hardware separation. Two real CVEs are shown: one from 2025 and one from April 2026, both of which exploit weaknesses in runc, the container runtime, to allow a process inside a container to escape to the host kernel. Once on the host kernel, an attacker has access to every other container on the same node and to the node's underlying resources.

The Four-Word Ban lists the four container configurations that must never appear in a production environment: `--privileged`, which grants the container near-root access to the host; `runAsRoot`, which runs the container process as root; `hostPath`, which mounts directories from the host filesystem into the container; and `hostNetwork`, which gives the container access to the host's network stack. These are non-negotiable exclusions rather than configuration preferences. The mandatory pod defaults panel shows the security context settings every pod must carry: `runAsNonRoot: true`, `allowPrivilegeEscalation: false`, and a defined `securityContext`.

The detection strip maps each layer of the stack to the audit event it generates. A privileged container creation produces a K8s audit log event. A namespace manipulation syscall produces a Falco critical alert. A seccomp violation produces a syscall audit event. Each of these routes to the SIEM.

**Additional Notes**
The purpose of the detection strip is to establish early that security controls and detection signals are the same thing viewed from different directions. Every enforcement boundary also generates an event when it is tested. That pattern repeats on every subsequent technical slide. The Four-Word Ban is referenced again in slide seven as the baseline that Kubernetes admission controllers enforce, and it appears in the conclusion as the second of four Monday actions.

---

### E4 — CI/CD: The Pipeline You Should Have

**Visual Description**
A horizontal pipeline diagram showing the stages from code commit through build, scanning, attestation, and deployment. Each stage is represented as a gate. The Rekor transparency log is shown connected to the attestation stage. A VSA gate sits as the final check before deployment. Attack arrows indicate where supply chain compromise can enter the pipeline. A callout references a scanner vulnerability as an example of supply chain risk within the toolchain itself.

**Slide Synopsis**
Reframes the CI/CD pipeline as a security control point where every gate is an auditable event, and introduces SLSA L3+, VSA, and Rekor as the provenance and attestation baseline.

**Slide Details**
The pipeline stages are: SAST (static application security testing) and SCA (software composition analysis) scanning to catch code and dependency vulnerabilities at build time; SBOM (software bill of materials) generation to produce a complete inventory of every component in the image; image signing to cryptographically bind the image to its build identity; SLSA L3+ provenance to certify that the build was hermetic and the output matches the source; VSA (Verification Summary Attestation) issuance by a verifier that is separate from the builder to confirm the image met policy; and deployment through a VSA gate that rejects any image without a valid, externally issued attestation.

SLSA Level 3 requires hermetic builds. A hermetic build cannot be influenced by anything outside the defined build inputs, meaning a compromised build environment cannot silently alter the output. The separation of the builder from the verifier is the critical control: the builder cannot certify its own output. A VSA is a signed statement from an independent verifier that the artefact met the required policy at the time of build.

Rekor is the transparency log that records every build event, including the image digest, the build time, the signing identity, and the VSA status. It creates an append-only, tamper-evident record of every deployment decision. A scanner vulnerability is shown as a callout to make the point that even the tools in the pipeline are supply chain nodes. A compromised scanner can produce clean results for malicious code, which is why provenance and attestation from a separate verifier matters.

**Additional Notes**
Rekor is the thread that connects slides four, five, and ten. It is introduced here as the build-time record. In slide five it is identified as the enrichment source the SOC does not have access to. In slide ten it becomes the first query an L1 analyst runs at triage: querying Rekor by image digest to confirm whether the running container was legitimately built and attested, taking approximately thirty seconds.

---

### E5 — Who Sees What and the Missing Venn

**Visual Description**
Three columns representing Security Engineering, AppSec, and the SOC. Each column lists the data sources and tools that team is connected to. Security Engineering and AppSec show a green "Connected" indicator at the base. The SOC column shows a red "Not built" indicator. A fourth column lists the five questions an L1 analyst needs to answer within 15 minutes of an alert firing, alongside the data source required for each and an approximate query time. A caption below the columns reads: "AppSec sees the SBOM finding. The SOC sees the Falco alert. Neither knows the other fired."

**Slide Synopsis**
Identifies the structural gap between the teams that hold supply chain and deployment provenance data and the SOC that needs it at triage, and makes the case that the SOC is currently operating without the context required to answer basic questions about a running container.

**Slide Details**
Security Engineering has policy-as-code enforcement via Kyverno or OPA, VSA gate controls, and admission controller logs showing every admission decision made by the cluster. AppSec has SBOMs, shift-left CVE triage, licence review, and dependency tracking findings from tools such as Dependency-Track. Both teams are connected to the data their work produces. The SOC is not.

When a Falco alert fires against a running container, the L1 analyst needs to answer five questions within their 15-minute SLA. Was this a scheduled deployment, answerable by looking up the ITSM change ticket? Is this the approved image version, answerable by querying Rekor for a valid VSA? Has this image digest been seen before in legitimate deployments, answerable from Rekor provenance history? What is the business impact of the affected service, answerable from the CMDB or a BIA classification? Was this a new push or a scheduled release, answerable from the change management window? Each of these queries takes approximately 30 seconds if the connections have been built. Most SOC teams have not built them.

The structural result is that AppSec sees a SBOM finding and the SOC sees a Falco alert, and neither team knows the other fired. They may be looking at the same incident from different directions and investigating independently.

**Additional Notes**
The gap named here is deployment provenance: at the moment an alert fires, the SOC cannot quickly establish whether the running container was legitimately built, attested, and deployed through an approved process. The slide argues that this is not a tooling gap but a connection gap. The data exists in Rekor, in the ITSM system, and in the CMDB. The missing piece is the workflow that surfaces it to the analyst at the moment of triage. This is resolved in the SOC triage loop shown in slide ten.

---

### E6 — Cloud Guardrails as Sensors

**Visual Description**
An isometric layered disc representing the cloud platform. The disc shows five elements: IAM and org policy gate, SCPs and organisational guardrails, KMS encryption, CSPM and posture monitoring, and a WAF. Each element carries an annotation showing the event it generates and where that event routes. A highlighted pillar on the right edge of the disc represents the cloud audit log pipeline routing through Security Hub or Defender to the SIEM. A small annotation strip names the three ownership roles: cloud ops configures, security engineering tunes, SOC reads and acts.

**Slide Synopsis**
Reframes cloud controls as detection surfaces rather than compliance tools, showing that IAM, SCP, KMS, CSPM, and WAF events all produce actionable signals in near-real time if the routing to the SIEM has been built.

**Slide Details**
IAM generates AssumeRole anomalies and credential creation events to CloudTrail. An AssumeRole call from an unexpected geography or at an unusual time is a detectable indicator of credential compromise or lateral movement. SCPs generate policy violation findings to Security Hub when an action is attempted that the organisational policy prohibits. KMS generates unusual decrypt events to CloudTrail when keys are accessed in patterns that deviate from normal usage. CSPM generates drift events via EventBridge when a resource configuration deviates from the defined policy baseline, for example a security group rule being opened to 0.0.0.0/0. The WAF generates rule trigger events to the SIEM when inbound traffic matches an attack signature.

Security Hub and Defender for Containers act as the normalisation layer for the cloud layer, taking findings from GuardDuty, Inspector, Macie, and third-party tools and routing them to the SIEM in a standardised format (ASFF for AWS). Without that routing, the findings exist in their source systems but are not visible to the SOC in a unified alert queue.

The distinction between event-driven alerting and periodic posture checking is important. CSPM via EventBridge fires within seconds of a specific configuration change. Periodic posture checks provide broader coverage but run on a cycle of 12 to 24 hours. Both are necessary and serve different purposes.

**Additional Notes**
The ownership model established here, cloud ops configures, security engineering tunes, SOC reads and acts, repeats in the Kubernetes layer on slide seven. It is a pattern the talk returns to at each layer to make clear that detection is not a single team's responsibility. Each layer has a different owner for configuration and a different owner for response, and the SOC's role is specifically to consume and act on the signals those layers produce.

---

### E7 — K8s Guardrails as Sensors

**Visual Description**
The same layered disc as slide six, now with the Kubernetes cluster fully lit and the cloud layer dimmed at the edges. Five controls are shown around the cluster disc: Kyverno and OPA admission policies, NetworkPolicy, RBAC, Pod Security Standards, and Vault for secrets management. An annotation strip maps each control to the event it generates, the destination of that event, and the team that owns the response action.

**Slide Synopsis**
Applies the same detection framing as slide six to the Kubernetes layer, showing that admission denies, network blocks, RBAC violations, and secret access events all produce auditable signals, and that those signals are only useful if they are shipped to the SIEM.

**Slide Details**
Kyverno and OPA admission policy denies produce K8s API audit events. Every time a pod deployment is blocked because it requests `--privileged` or mounts a `hostPath`, that block is written to the K8s API audit log. NetworkPolicy blocks generate flow logs that show attempted connections between workloads that were denied by policy. RBAC violations produce K8s audit events when a service account or user attempts an action their role does not permit. Pod Security Standard violations produce admission denies in the API audit trail. Vault secret reads produce Vault audit entries for every secret access, including the identity, the path, and the time.

All of these signal paths converge on the K8s API audit log as the central connective layer before routing to the SIEM. The critical dependency is that K8s API audit logs must be shipped externally. In most clusters, audit logging is enabled but configured to write to local disk on the control plane node. The logs exist but are inaccessible during an incident investigation unless they have been forwarded to an external store. Without external shipping, every signal from this layer is invisible to the SOC at the moment it is needed most.

**Additional Notes**
The Four-Word Ban from slide three is closed here: Kyverno and OPA are the enforcement mechanisms, and every enforcement action they take is a data point in the audit log. The connection between policy and detection is direct. A cluster that enforces the Four-Word Ban and ships its admission logs externally is both more secure and more observable than one that does neither.

---

### E8 — Runtime: eBPF Signal Generation

**Visual Description**
A split-panel layout. The left panel shows a CI/CD chain leading to a container, annotated to show the categories of threat that shift-left scanning cannot detect. The right panel shows the eBPF monitoring stack, a process lineage chain illustrating an impossible behaviour example, and a list of specific runtime events that eBPF detects. A callout contrasts container-level logs, which are lost on termination, with eBPF logs, which are written at the kernel level and survive it.

**Slide Synopsis**
Introduces eBPF as the kernel-level detection layer that provides visibility into container behaviour that build-time scanning cannot reach, and explains the mechanism by which eBPF logs survive container termination when container-local logs do not.

**Slide Details**
Shift-left scanning, meaning SAST, SCA, and image scanning at build time, cannot detect zero-day exploits that were not present in the CVE database at scan time, living-off-the-land techniques that use legitimate system binaries to carry out malicious actions, runtime drift where a container changes behaviour after deployment, or the forensic challenges created by ephemeral containers.

eBPF (extended Berkeley Packet Filter) is a Linux kernel technology that allows instrumentation programs to run safely inside the kernel without modifying kernel source code or loading kernel modules. Security tools such as Falco, Tetragon, and Cilium use eBPF to observe every syscall, every process creation, every network connection, and every file access made by every process on the host, including those running inside containers. This visibility is kernel-level and cannot be evaded by anything running in userspace inside a container.

The behavioural detection capability shown includes: a web server process such as Nginx spawning a shell, which is impossible behaviour for a legitimate web server and indicates command execution by an attacker; filesystem access patterns in `/proc` consistent with container namespace escape attempts; reflective DLL loading and LD_PRELOAD injection used to execute malicious code in the memory space of a legitimate process; sleep loops and CronJob abuse used to maintain persistence; and eBPF rootkit cloaking attempts where malware tries to hide its own kernel-level activity.

The ephemeral log contrast is the mechanistic explanation of the problem seeded in slide two. When a container is terminated, its overlay filesystem is destroyed. Any logs or forensic artefacts written inside the container are lost. eBPF operates at the kernel level, outside the container's filesystem boundary. If eBPF events are shipped to an external store as they are generated, the process lineage, syscall history, and network activity of every container process remain available for investigation even after the container has been terminated.

**Additional Notes**
Tetragon's TracingPolicy kill action is noted as an active response capability. When a process matches a defined kill condition, for example spawning a shell from a web server process, Tetragon can terminate the container automatically via policy rather than waiting for a human to act. This moves the response from detection-then-manual-action to automated containment at the kernel level.

---

### E9 — The Unified Defence Graph

**Visual Description**
A layered architecture diagram with signal sources on the left feeding into a central collection and correlation stack, which routes to a SIEM and incident management system on the right. Signal sources include eBPF kernel events, K8s audit logs, application traces, service mesh telemetry, cloud audit logs, and Rekor shown with a dashed border as an on-demand enrichment source. The central stack shows a kernel and runtime layer at the base, an OTel collector above it, and a correlation engine at the top. Alert severity tiers on the right show named owners for each level.

**Slide Synopsis**
Names and explains the Unified Defence Graph, distinguishes it from a SIEM, and shows the routing, correlation, and ownership model that makes it operational.

**Slide Details**
The Unified Defence Graph is the state in which every security control is a node and every telemetry feed connecting those controls is an edge. The distinction from a SIEM is important. A SIEM that collects logs is a store. The Unified Defence Graph is operational integration: an alert from a Falco rule can be enriched in the same triage workflow with Rekor provenance data, correlated with a K8s admission audit event, and cross-referenced with a cloud IAM change, producing a chain of evidence rather than a list of disconnected alerts.

OpenTelemetry (OTel) is the collection standard that normalises signals from different sources into a consistent format before routing. Its security value is in the contextual chain it preserves: user identity, pod identity, and kernel execution form a connected sequence that reveals attacker movement across layers. Without that chain, the same sequence of events appears as three separate alerts with no visible relationship.

Rekor sits outside the continuous ingestion pipeline and is queried on demand during triage. This is by design: Rekor is not a high-volume log source. It is a lookup against a specific image digest to answer a specific question about provenance and attestation status.

The alert routing and ownership model assigns high-severity events such as container escapes to SOC L2/L3 for immediate investigation, medium-severity events such as runtime drift to platform team and security engineering for auto-remediation with a ticket, and low-severity events such as anomalous syscalls to the SOC investigation queue. The four roles involved in operating the graph are: SOC L1 triages, SOC L2 and L3 investigate, security engineering configures detection logic, platform maintains infrastructure.

**Additional Notes**
The anti-pattern called out explicitly on this slide is fragmented tooling. Most organisations have Falco, CloudTrail, and K8s audit logs. Fewer have built the routing that connects them. Without routing, correlation is manual and slow. The talk argues that the Unified Defence Graph is not a product to be purchased but a state to be built, and that the routing decisions are the work.

---

### E10 — SOC Triage, Investigation and Response Loop

**Visual Description**
A three-section layout. The left section shows data ingestion paths with each signal source alongside its route to the SIEM. The centre section shows a tiered triage loop: L1 at the top with a 15-minute SLA, L2 below with a 30-minute SLA, and L3 with security engineering at the base. Within the L2 tier, a fork shows the kill-or-preserve decision as two distinct paths with their trade-offs annotated. The right section shows response primitives organised by layer.

**Slide Synopsis**
Shows the full operational loop from signal ingestion through triage, investigation, and response, including the specific actions at each tier, the Rekor enrichment step, and the kill-or-preserve decision unique to container incident response.

**Slide Details**
Signal ingestion paths are: cloud audit logs via Security Hub or Defender to the SIEM; K8s API audit logs via a Fluent Bit DaemonSet to the SIEM; Falco and Tetragon alerts via Falco sidekick to the SIEM; service mesh telemetry via the OTel collector to the SIEM; and application logs via the OTel collector to the SIEM. Rekor is queried on demand during L1 triage rather than continuously ingested.

L1 triage actions within a 15-minute SLA are: receive and acknowledge the alert; perform an ITSM lookup to check whether the affected service has an open change ticket covering this deployment window; query Rekor by image digest to confirm whether the running image has a valid VSA and was built through the expected pipeline; check the behavioural baseline to determine whether the alert represents a deviation from normal for this workload; and enrich with CMDB data to identify the business impact classification of the affected service. The triage produces one of two outcomes: close with a tuning note if the alert is a false positive, or escalate to L2 with full enrichment context.

L2 investigation actions within a 30-minute SLA are: trace the process lineage chain using eBPF event data to reconstruct what the container process did at the kernel level; correlate K8s audit log events to determine what API calls were made around the time of the alert; correlate cloud audit log events to identify any IAM, network, or storage activity associated with the incident; and correlate OTel traces to map application-level behaviour. L2 then makes the kill-or-preserve decision. Killing the container terminates the attack and removes the attack surface, but destroys the container's overlay filesystem and any forensic artefacts it contains. Preserving the container retains the evidence but allows the attack to continue. The eBPF logs shipped externally before termination remain available regardless of which path is chosen.

L3 and security engineering response actions are: revoke affected IAM credentials and rotate secrets; identify the pipeline run that produced the image and trace whether other deployments were affected; identify the detection rule that should have fired earlier but did not, and close the gap; and update runbooks with findings from the investigation.

**Additional Notes**
This slide resolves both threads seeded in slide two. The ephemeral container problem is answered by the external eBPF log store: the container is gone, but the kernel-level record of everything it did is still available. The correlation gap is answered by the triage loop itself: L1 has Rekor, ITSM, and CMDB lookups as standard steps, meaning the SOC is connected to the provenance and change data that slide five identified as missing. The kill-or-preserve decision is unique to containerised environments. In a traditional host investigation, preserving the system is the default. In a containerised environment, the container is transient by design and the evidence that matters most, the eBPF process lineage, is already external. The choice depends on the incident severity, the availability of external log coverage, and whether the blast radius is bounded.

---

### E11 — Assurance

**Visual Description**
Two cards side by side on a dark background. One card covers MTTR as the primary assurance metric. The other covers chaos engineering as the mechanism for validating that the detection and response loop performs as designed.

**Slide Synopsis**
Argues that deploying controls is not the same as assurance, and presents MTTR and scheduled chaos engineering as the two mechanisms that provide evidence the detection and response loop actually works.

**Slide Details**
MTTR (Mean Time to Respond) is presented as the metric that reflects operational readiness for a containerised environment. The target presented is under ten minutes for critical container security incidents, covering the time from detection to containment. Achieving that target requires that Rekor enrichment queries are pre-built and available to L1 analysts, that runbooks are written, tested, and accessible, that escalation paths are clear and practised, and that alert routing is tuned to reduce noise so that genuine signals are not buried.

Chaos engineering as runbook rehearsal means deliberately injecting a simulated container security incident into a non-production environment and running the full triage and response loop against it. The exercise answers four questions: does the alert fire correctly; does the L1 analyst follow the runbook and reach the correct decision within the SLA; does the Rekor enrichment return the expected result for a known-good and a known-bad image; and is the measured end-to-end response time within the target MTTR.

**Additional Notes**
The argument is that tool coverage and rule count are outputs of configuration work, not evidence that the configuration works. An organisation can have Falco deployed, Rekor integrated, and runbooks written, and still have an MTTR of three hours if the runbooks have never been tested under realistic conditions. Chaos engineering converts the detection and response loop from an untested configuration into a measured capability with a tracked performance baseline.

---

### E12 — Four Actions for Monday

**Visual Description**
An assembled puzzle at the top of the slide, with all six pieces from slide two now locked together and connected by green junction lines. Below it, four equal-width cards in a horizontal row. The first three cards carry steel-blue borders. The fourth carries a muted violet border to distinguish it from the others.

**Slide Synopsis**
Closes the talk with four concrete, sequenced actions that compress the full argument into immediate starting points.

**Slide Details**
The assembled puzzle at the top is the visual resolution of slide two: the six fragmented layers are now connected. The four cards are sequenced so that each one enables the next.

Card one, Mandate Trust: enforce SLSA L3+ provenance with VSA gate controls on every production deployment, and implement automated revocation so that a compromised signing identity can be invalidated without manual intervention. The governing principle is that no attested image means no deployment. The statistic shown is a 431% year-on-year increase in software supply chain attacks, from the CNCF 2025 report.

Card two, Enforce Containment: enforce the Four-Word Ban via admission controllers, drop all Linux capabilities using `CAP_DROP ALL` as the default, and apply default-deny NetworkPolicy so that workloads cannot communicate unless a policy explicitly permits it. The governing principle is that the secure configuration should be the only available path. The statistic shown is that 82% of cloud breaches originate from misconfiguration, from Verizon DBIR 2025.

Card three, Deploy Visibility: ship eBPF kernel events, K8s API audit logs, and cloud audit logs to the SIEM with the routing and enrichment pipelines described in slides nine and ten. The governing principle is that the signals already exist; the work is building the routing that makes them available to the SOC. The statistic shown is that 61% of runtime exploits are detectable at the kernel level.

Card four, Detect and Respond: tune Falco rules so that alerts reflect real threat behaviour for the specific workloads running in the environment; test the runbooks described in slide ten so that the kill-or-preserve decision and the L1 triage steps are practised before an incident; measure MTTR against the target set in slide eleven; and ensure that named individuals own each layer of the Unified Defence Graph. The governing principle is that the graph does not operate itself.

**Additional Notes**
Card four is visually distinguished from the first three because it is the addition that makes this version of the talk different from all previous versions. Cards one through three describe configuration work that security and platform engineering teams can complete. Card four describes operational ownership: someone tunes the rules, someone tests the runbooks, someone measures the MTTR, and someone is accountable for each layer. The talk argues that without card four, the first three cards produce a well-configured environment that nobody is actively operating.

---

## Threads That Run Through the Talk

Three ideas are seeded early and paid off later. They are worth noting when reading the slides in sequence.

**Ephemeral logs** are introduced in slide two as a forensic problem created by the short-lived nature of containers, explained mechanically in slide eight as the reason eBPF logs must be written externally at the kernel level, and resolved operationally in slide ten where the external log store provides the process lineage needed for the L2 investigation even after the container has been terminated.

**Rekor** is introduced in slide four as the transparency log that records every build and attestation event in the CI/CD pipeline, identified in slide five as the enrichment source the SOC does not currently have access to, and used in slide ten as the first query an L1 analyst runs at triage to determine whether a running container was legitimately built and deployed.

**The Four-Word Ban** is shown in slide three as the four container configurations that must never appear in production, enforced in slide seven by Kyverno and OPA admission controllers, and carried into the conclusion as card two. The same four configurations, `--privileged`, `runAsRoot`, `hostPath`, and `hostNetwork`, appear at the foundation, the enforcement layer, and the closing actions.

---

*Views expressed are my own and do not necessarily reflect those of my employer.*
*© Ashley Barker 2026*
