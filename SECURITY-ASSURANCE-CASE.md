# Strimzi Security Assurance Case

## Table of Contents

* [Metadata](#metadata)
* [Relationship to the Security Self-Assessment](#1-relationship-to-the-security-self-assessment)
* [Scope](#2-scope)
* [Top-Level Claim](#3-top-level-claim)
* [Security Requirements](#4-security-requirements)
* [Assumptions and Shared Responsibilities](#5-assumptions-and-shared-responsibilities)
* [Assets](#6-assets)
* [Trust Boundaries](#7-trust-boundaries)
* [Threat Model](#8-threat-model)
* [Claims, Arguments, and Evidence](#9-claims-arguments-and-evidence)
* [Secure Design Principles](#10-secure-design-principles)
* [Common Implementation Weakness Countermeasures](#11-common-implementation-weakness-countermeasures)
* [Evidence Index](#12-evidence-index)
* [Residual Risks](#13-residual-risks)
* [Maintenance of the Assurance Case](#14-maintenance-of-the-assurance-case)
* [Conclusion](#15-conclusion)

## TODOs

There are a couple of TODOs that should be revisited and resolved before proposing the file into governance repo.
The following list of proposals should be implemented before we will introduce final version of this document.
We should also agree on `Configurable Security for Kafka Connect Internal Communication` thing as it can be considered as a security issue (proposal and implementation for this will be needed).

| SIP                                                                                                              | Description                                                         | Status                     |
|------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------|----------------------------|
| [SIP-100](https://github.com/strimzi/proposals/blob/main/100-external-certificate-manager.md)                    | External Certificate Manager - Cert-Manager integration             | Implementation in-progress |
| [SIP-144](https://github.com/strimzi/proposals/blob/main/144-Strimzi-Gatekeeper-plugin-system.md)                | Gatekeeper Plugin System                                            | Implementation In-progress |
| [SIP-150](https://github.com/strimzi/proposals/blob/main/150-configurable-security-of-internal-communication.md) | Configurable Security of Internal Communication - Istio integration | Implementation In-progress |
| [SIP-152](https://github.com/strimzi/proposals/pull/240)                                                         | Migrate Strimzi images from ubi9-minimal to ubi10-micro             | Proposal approved          |
| KafkaConnect security limitations                                                                                | Configurable Security for Kafka Connect Internal Communication      | TODO                       |
| KafkaExporter maintainability                                                                                    | What we will do with KE, update the doc if we agree on something    | TODO                       |
| Updated all commit hashes and versions before submitting                                                         | -                                                                   | TODO                       |
| Revisit [Evidence still to pin before publication](#evidence-still-to-pin-before-publication)                    | This should be revisited and pinned if agreed                       | TODO                       |

## Metadata

|                       |                                                                                                                                                                                                                                                                                                                         |
|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Assessment Stage      | Draft                                                                                                                                                                                                                                                                                                                   |
| Assessment Date       | 2026-07-26                                                                                                                                                                                                                                                                                                              |
| Software              | [Strimzi](https://github.com/strimzi/strimzi-kafka-operator)                                                                                                                                                                                                                                                            |
| Assessment Basis      | Strimzi `main` at `3c8319e0`                                                                                                                                                                                                                                                                                            |
| Target Release        | Strimzi 1.3.0 or later                                                                                                                                                                                                                                                                                                  |
| Release Status        | Unreleased                                                                                                                                                                                                                                                                                                              |
| Kubernetes Versions   | 1.30+ (as supported by the assessed release)                                                                                                                                                                                                                                                                            |
| Apache Kafka Versions | 4.2.0, 4.2.1, 4.3.0, 4.3.1 (default)                                                                                                                                                                                                                                                                                    |
| Website               | https://strimzi.io                                                                                                                                                                                                                                                                                                      |
| Security Provider     | No — Strimzi manages Apache Kafka on Kubernetes; it is not a security product                                                                                                                                                                                                                                           |
| Languages             | Java 21                                                                                                                                                                                                                                                                                                                 |
| SBOM                  | Published per release via Syft, attached as attestations to container images using Sigstore                                                                                                                                                                                                                             |
| Included Repositories | [strimzi-kafka-operator](https://github.com/strimzi/strimzi-kafka-operator) (1.3.0), [drain-cleaner](https://github.com/strimzi/drain-cleaner) (1.7.0), [strimzi-kafka-bridge](https://github.com/strimzi/strimzi-kafka-bridge) (1.1.0), [strimzi-kafka-oauth](https://github.com/strimzi/strimzi-kafka-oauth) (0.18.0) |
| Purpose               | OpenSSF Best Practices Silver criterion `assurance_case`                                                                                                                                                                                                                                                                |

### Security Links

| Doc                        | URL                                                                                     |
|----------------------------|-----------------------------------------------------------------------------------------|
| Security Policy            | https://github.com/strimzi/.github/blob/main/SECURITY.md                                |
| Security Documentation     | https://strimzi.io/docs/operators/latest/overview#security-overview_str                 |
| Security Self-Assessment   | [SECURITY-SELF-ASSESSMENT.md](SECURITY-SELF-ASSESSMENT.md)                              |
| OpenSSF Best Practices     | https://www.bestpractices.dev/en/projects/13184                                         |
| OpenSSF Scorecard          | https://scorecard.dev/viewer/?uri=github.com/strimzi/strimzi-kafka-operator             |
| GitHub Security Advisories | https://github.com/strimzi/strimzi-kafka-operator/security/advisories                   |
| Vulnerability Disclosure   | [cncf-strimzi-maintainers@lists.cncf.io](mailto:cncf-strimzi-maintainers@lists.cncf.io) |

### Identifier Prefixes

| Prefix | Meaning              |
|--------|----------------------|
| **AC** | Assurance Case claim |
| **SR** | Security Requirement |
| **A**  | Assumption           |
| **TB** | Trust Boundary       |
| **T**  | Threat               |
| **E**  | Evidence             |

> This document presents a concise security assurance case for Strimzi. It is not an independent security audit or a claim that Strimzi is secure in every configuration. It explains why the project considers its stated security requirements to be adequately met when Strimzi is deployed in a supported environment and the assumptions in this document hold.

## 1. Relationship to the Security Self-Assessment

This assurance case is intentionally narrower than the full Strimzi Security Self-Assessment.

The documents have different purposes:

| Document                     | Purpose                                                                                                                                                               |
|------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Security Self-Assessment** | Describes the project, architecture, actors, security features, development practices, vulnerability history, compliance-related activities, and operational context. |
| **Security Assurance Case**  | States security claims and provides arguments, evidence, assumptions, trust boundaries, threat analysis, and residual risks supporting those claims.                  |
| **Internal review**          | Tracks factual corrections, missing evidence, publication blockers, and suggested improvements before either document is published.                                   |

The self-assessment is supporting context for this assurance case, but the assurance case should remain usable and reviewable on its own.

## 2. Scope

### 2.1 In scope

This assurance case covers the Strimzi Cluster Operator and the Strimzi-managed resources and operands represented by the following APIs and components:

- `Kafka` and `KafkaNodePool`;
- `KafkaConnect` and `KafkaConnector`;
- `KafkaMirrorMaker2`;
- `KafkaBridge`;
- `KafkaRebalance` and Cruise Control integration;
- `KafkaTopic` and the Topic Operator;
- `KafkaUser` and the User Operator;
- `StrimziPodSet`;
- Strimzi-generated certificates, Kubernetes RBAC resources, Services, and NetworkPolicy resources;
- Strimzi Drain Cleaner;
- Strimzi OAuth for Apache Kafka;
- Strimzi release container images and their published SBOMs and signatures;
- the development and vulnerability-response practices of the included Strimzi repositories.

Externally maintained components are covered only where explicitly identified by evidence in this document.

### 2.2 Out of scope

The following are not secured or fully controlled by Strimzi and are outside the product assurance claim:

- the Kubernetes API server, etcd, nodes, CNI, CSI, admission controllers, and infrastructure configuration;
- encryption of Kafka data stored on persistent volumes;
- security of customer applications and Kafka clients;
- security of third-party Kafka Connect plugins and systems reached by connectors;
- external identity providers, certificate authorities, proxies, registries, and artifact repositories;
- custom authentication, authorization, and PodSecurityProvider plugins supplied by users;
- user-created listener configurations that disable TLS, authentication, authorization, or network restrictions;
- direct Kafka administration performed outside Strimzi custom resources;
- protection against a Kubernetes cluster administrator or an attacker with equivalent privileges.

## 3. Top-Level Claim

### AC-0 — Strimzi adequately meets its stated security requirements

**Claim:**

> When a patched Strimzi release is deployed on a supported and appropriately secured Kubernetes cluster, and the deployment assumptions in this document are satisfied, Strimzi provides reasonable protection for the confidentiality, integrity, and availability of its operator-managed control plane, credentials, and Kafka communication paths against the threats identified in this assurance case.

**Argument:**

The claim is supported because:

1. security responsibilities and deployment assumptions are explicitly identified;
2. trust boundaries and security-sensitive interfaces are documented;
3. likely threats are analysed using STRIDE;
4. security controls are applied at the Kubernetes, Kafka, container and pod security, and release-supply-chain layers;
5. secure design principles are reflected in the architecture and defaults where practical;
6. relevant common implementation weaknesses are prevented, reduced, detected, or explicitly assigned to the deployment environment;
7. known limitations and residual risks are documented rather than treated as fully mitigated;
8. the project maintains public vulnerability reporting and remediation processes.

AC-0 is supported by claims AC-1 through AC-7 below.

## 4. Security Requirements

| ID       | Security requirement                                                                                                                                                                                     |
|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **SR-1** | Only authorized Kubernetes identities can create or modify Strimzi custom resources and the Kubernetes resources managed from them.                                                                      |
| **SR-2** | Strimzi-managed component identities and credentials are generated, stored, distributed, and rotated without hard-coded shared credentials.                                                              |
| **SR-3** | Security-sensitive Kafka communication paths support authenticated and encrypted communication, with secure internal Kafka communication configured by default unless explicitly changed.                |
| **SR-4** | Strimzi components receive only the Kubernetes privileges required for their documented functions, subject to the limitations of the operator delegation model.                                          |
| **SR-5** | Untrusted or invalid custom-resource input is structurally validated before storage where supported and semantically validated during reconciliation.                                                    |
| **SR-6** | Strimzi releases provide integrity and transparency information that allows users to verify released container images and inspect their software composition.                                            |
| **SR-7** | Security defects can be reported privately, assessed, fixed, and communicated through a documented vulnerability-response process.                                                                       |
| **SR-8** | Reconciliation, rolling updates, and Kubernetes workload management reduce avoidable service disruption, while operational denial-of-service and infrastructure failures remain shared responsibilities. |

These requirements describe what the project intends to provide. They do not imply that every user-created Kafka listener or every Kubernetes deployment is secure without additional configuration.

## 5. Assumptions and Shared Responsibilities

The top-level claim depends on the following assumptions.

| ID       | Assumption or responsibility                                                                                                                                                                                |
|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **A-1**  | The Kubernetes cluster is within its supported version range and is maintained with security updates.                                                                                                       |
| **A-2**  | Kubernetes authentication and RBAC prevent untrusted users from modifying Strimzi custom resources, operator workloads, RBAC bindings, Secrets, admission policies, or release images.                      |
| **A-3**  | Kubernetes Secrets are protected using least-privilege RBAC and, where confidentiality at rest is required, Kubernetes encryption at rest or an equivalent platform control.                                |
| **A-4**  | The installed CNI enforces Kubernetes NetworkPolicy resources. Where source restrictions are required, users configure listener `networkPolicyPeers` or equivalent external controls.                       |
| **A-5**  | Kafka listeners exposed to applications are configured with TLS and appropriate authentication and authorization for the deployment's risk level.                                                           |
| **A-6**  | Component REST APIs, especially the Kafka Connect REST API and Kafka Bridge API, are not exposed to untrusted networks without appropriate network restrictions and an authentication layer where required. |
| **A-7**  | Kafka Connect plugin artifacts, build inputs, and output registries are trusted or independently verified.                                                                                                  |
| **A-8**  | Persistent storage encryption, backup protection, node security, and infrastructure availability are provided by the Kubernetes platform or infrastructure operator.                                        |
| **A-9**  | Users deploy a patched Strimzi release and respond to published Strimzi and Apache Kafka security advisories.                                                                                               |
| **A-10** | Custom plugins and user overrides are reviewed because they can weaken or replace built-in controls.                                                                                                        |

If these assumptions do not hold, compensating controls are required and AC-0 might not be justified for that deployment.

## 6. Assets

The primary assets considered by this assurance case are:

- desired-state configuration in Strimzi custom resources;
- Kafka topic metadata, user configuration, ACLs, and quotas managed through Strimzi;
- Kafka message data and cluster availability;
- Cluster CA and Clients CA private keys;
- broker, operator, and client certificates and passwords;
- Kubernetes ServiceAccount tokens and delegated RBAC authority;
- Kubernetes Secrets and ConfigMaps referenced by Strimzi workloads;
- Kafka Connect connector configuration and plugin code;
- container images, release artifacts, SBOMs, and signing identities;
- reconciliation status, logs, metrics, and audit records;
- the integrity of Strimzi-managed pods, Services, NetworkPolicies, and RBAC resources.

## 7. Trust Boundaries

A trust boundary exists where data, identity, configuration, or execution crosses into a context with a different level of trust.

| ID        | Boundary                                                         | Trust change and risk                                                                      | Primary controls                                                                                                                      | Responsibility                                                      |
|-----------|------------------------------------------------------------------|--------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------|
| **TB-1**  | User or automation → Kubernetes API                              | Untrusted or partially trusted users submit security-relevant desired state.               | Kubernetes authentication and RBAC, CRD schema validation, admission controls, reconciliation-time validation.                        | Kubernetes administrator and Strimzi.                               |
| **TB-2**  | Cluster Operator → Kubernetes API                                | A highly privileged controller creates workloads, Secrets, RBAC, Services, and policies.   | ServiceAccount authentication, scoped Roles and ClusterRoles, namespace watch configuration, audit logging if enabled.                | Strimzi and Kubernetes administrator.                               |
| **TB-3**  | Operand pod → Kubernetes API                                     | Managed workloads may receive API access for rack awareness, custom resources, or Secrets. | Dedicated ServiceAccounts, delegated RBAC, token-mount configuration, namespace scoping.                                              | Strimzi and Kubernetes administrator.                               |
| **TB-4**  | Kafka client → Kafka listener                                    | External identities access Kafka data and administrative operations.                       | Listener TLS, mTLS/SCRAM/OAuth/custom authentication, Kafka authorization, quotas, network restrictions.                              | Strimzi provides configuration; user selects and operates controls. |
| **TB-5**  | Strimzi component → Kafka protocol endpoint                      | Operators and operands perform trusted Kafka operations.                                   | TLS and supported client authentication, generated or referenced credentials, Kafka ACLs where applicable.                            | Strimzi and user configuration.                                     |
| **TB-6**  | User, proxy, or operator → component HTTP/REST API               | REST APIs can expose data or administrative actions outside Kubernetes RBAC.               | Service exposure, NetworkPolicy, TLS where supported, authenticated proxy or service mesh where required.                             | User and platform administrator.                                    |
| **TB-7**  | Kafka Connect build pod → artifact source and container registry | External plugin code enters a privileged build and runtime supply chain.                   | Explicit build configuration, digest/checksum verification where configured, trusted registries, pod security controls, image policy. | User and platform administrator.                                    |
| **TB-8**  | Pod → persistent volume                                          | Kafka data leaves process memory and is stored by external infrastructure.                 | CSI/storage access control, volume encryption, node protection, backups.                                                              | Infrastructure administrator; outside Strimzi.                      |
| **TB-9**  | CI/release workflow → public registry and release hosting        | A compromised release process could publish malicious artifacts.                           | Protected repository workflow, reviewed changes, GitHub OIDC keyless signing, SBOM publication, user verification.                    | Strimzi project and artifact consumer.                              |
| **TB-10** | Namespace-scoped resources → cluster-scoped resources            | Delegated permissions may cross namespace boundaries or access cluster-scoped objects.     | Separate ClusterRoles, restricted RoleBindings/ClusterRoleBindings, feature controls, security regression tests.                      | Strimzi and Kubernetes administrator.                               |

## 8. Threat Model

### 8.1 Method

The project applies STRIDE to the assets and trust boundaries above. Threats are considered from these attacker positions:

- an authenticated Kubernetes user with limited namespace permissions;
- a compromised application or operand pod;
- a network attacker able to reach an exposed listener or REST API;
- a malicious or compromised external plugin or artifact source;
- a compromised CI dependency or release workflow;
- an attacker who has obtained a credential but not cluster-administrator access.

A fully privileged Kubernetes cluster administrator is outside the threat model because that identity can replace Strimzi workloads, Secrets, RBAC, and admission controls.

### 8.2 Threats, controls, and residual risk

| ID       | STRIDE                                          | Threat                                                                                                                               | Boundary/assets                                               | Argument and controls                                                                                                                                                                                                                | Residual risk                                                                                                                                                                                                   |
|----------|-------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **T-01** | Spoofing / Tampering                            | An unauthorized user creates or modifies a Strimzi custom resource to weaken security or execute operator actions.                   | TB-1; custom resources and managed workloads.                 | Kubernetes RBAC controls write access. CRD schemas reject structurally invalid objects. The operator performs semantic checks during reconciliation. Admission policy can impose deployment-specific restrictions.                   | A user authorized to modify a powerful Strimzi resource can intentionally choose insecure listener, image, plugin, template, or namespace settings. Fine-grained admission policy is a platform responsibility. |
| **T-02** | Elevation of privilege                          | The Cluster Operator ServiceAccount or pod is compromised and its reconciliation privileges are abused.                              | TB-2 and TB-10; Kubernetes resources, Secrets, and RBAC.      | The operator uses dedicated ServiceAccounts and separated ClusterRoles/RoleBindings. Namespace watch scope can reduce resource scope. Images are signed and deployments can pin immutable versions.                                  | Operator compromise remains high impact. Some cluster-scoped or delegated permissions are necessary for supported features. Runtime detection and admission policy are external controls.                       |
| **T-03** | Elevation of privilege / Information disclosure | An operand ServiceAccount accesses unrelated custom resources or Secrets.                                                            | TB-3 and TB-10; Secrets and namespace resources.              | Roles are separated by component and recent advisories resulted in narrower privilege behaviour. Users can isolate sensitive deployments in separate namespaces and restrict token mounting.                                         | Namespace-level access can remain broader than a single object. Misconfigured or vulnerable RBAC may expose unrelated resources.                                                                                |
| **T-04** | Spoofing                                        | A rogue broker or component presents an untrusted identity to an internal Kafka endpoint.                                            | TB-5; cluster identity and Kafka traffic.                     | Strimzi-managed internal Kafka identities use certificates issued by the configured Cluster CA, and peers validate trusted CA material. Certificates are renewed and rotated.                                                        | Compromise of a CA key or incorrectly configured custom CA chain can undermine identity. Revocation and client trust-store update remain operational concerns.                                                  |
| **T-05** | Information disclosure / Tampering              | Kafka traffic is intercepted or altered.                                                                                             | TB-4 and TB-5; Kafka credentials and message data.            | Internal Kafka communication uses TLS in the secure default mode. User listeners support TLS and authenticated client mechanisms.                                                                                                    | User listeners can be configured without TLS. TLS does not protect data at rest or a compromised endpoint. Configurable internal security modes must be assessed separately when used.                          |
| **T-06** | Spoofing / Elevation of privilege               | An attacker reaches Kafka Connect REST and creates or changes connectors to access credentials or external data.                     | TB-6; connector configuration, Secrets, and external systems. | Strimzi supports management through `KafkaConnector` resources. Documentation recommends limiting Connect API access using network controls. The API should not be externally exposed without additional authentication.             | Kafka Connect REST security is not automatically equivalent to Kubernetes RBAC. A compromised trusted caller or connector can still exfiltrate data.                                                            |
| **T-07** | Information disclosure                          | TLS private keys, SCRAM passwords, or referenced credentials are read from Kubernetes Secrets.                                       | TB-2 and TB-3; credentials.                                   | Credentials are generated dynamically where applicable and stored in Kubernetes Secrets. Access is controlled using Kubernetes RBAC and namespace boundaries.                                                                        | Kubernetes Secrets require platform encryption-at-rest configuration for storage confidentiality. Any pod or user permitted to read a Secret can disclose it.                                                   |
| **T-08** | Tampering / Elevation of privilege              | A malicious Kafka Connect plugin or build artifact compromises the build pod, Connect process, node, or external systems.            | TB-7; build environment and runtime.                          | Builds are explicit user actions and can be constrained using pod security, image policy, trusted artifact sources, checksums, and isolated build infrastructure. Restricted pod security can prevent incompatible build privileges. | Strimzi does not validate the behaviour of third-party plugins. Plugin code executes with the Kafka Connect process's authority.                                                                                |
| **T-09** | Tampering                                       | A Strimzi container image or release artifact is replaced during publication or distribution.                                        | TB-9; release artifacts.                                      | Release container images are signed using Sigstore-based signing, and SBOMs are published and signed. Users can verify signatures and pin immutable image digests.                                                                   | Signing does not enforce verification at deployment time. Consumers need admission or deployment policy that rejects unverified images.                                                                         |
| **T-10** | Denial of service                               | Excessive custom resources, expensive reconciliation, connector failures, or repeated updates exhaust operator or cluster resources. | TB-1, TB-2, and TB-6; operator and workload availability.     | Kubernetes quotas, admission policy, resource requests/limits, reconciliation locking and retry behaviour, and Kafka quotas reduce some exhaustion scenarios.                                                                        | The operator does not provide a universal tenant-level rate limiter. A user with broad write access can still create disruptive valid configurations.                                                           |
| **T-11** | Information disclosure                          | Kafka data is read from persistent volumes, snapshots, or backups.                                                                   | TB-8; Kafka message data.                                     | Kubernetes storage access controls and infrastructure encryption can protect volumes and backups.                                                                                                                                    | Strimzi does not encrypt Kafka log segments at rest. This threat is outside the Strimzi product boundary and must be addressed by the platform.                                                                 |
| **T-12** | Repudiation                                     | Security-relevant custom-resource or Kafka-side changes cannot be attributed.                                                        | TB-1, TB-2, TB-4; configuration and metadata.                 | Kubernetes audit logs can record API activity when configured. Resource status and operator logs provide operational evidence. Kafka authorization and external audit tooling can add Kafka-side records.                            | Kubernetes auditing is platform-configured and does not capture all direct Kafka Admin API activity. Kubernetes resources do not provide a permanent history of every prior value.                              |
| **T-13** | Information disclosure / Elevation of privilege | An exposed Kafka Bridge API is used by an unauthorized caller.                                                                       | TB-6 and TB-4; Kafka data and client authority.               | The Bridge's Kafka connection supports TLS and authentication. Exposure can be limited using Services, ingress/proxy controls, and NetworkPolicy.                                                                                    | HTTP caller authentication and authorization must be evaluated separately from the Bridge's identity toward Kafka.                                                                                              |
| **T-14** | Tampering                                       | A custom authentication, authorization, PodSecurityProvider, or template override weakens generated controls.                        | TB-1, TB-4, TB-10; generated workloads and access policy.     | Extension points are explicit configuration and are subject to CR validation and Kubernetes admission controls.                                                                                                                      | User-provided code and overrides are trusted inputs. Strimzi cannot guarantee their security.                                                                                                                   |

## 9. Claims, Arguments, and Evidence

### AC-1 — Control-plane changes are mediated by Kubernetes authorization

**Claim:** Unauthorized identities cannot normally change Strimzi desired state or operator-managed resources.

**Argument:**

- Strimzi custom resources are Kubernetes API resources and therefore use Kubernetes authentication and RBAC.
- The Cluster Operator uses its own ServiceAccount and role bindings to reconcile only within its configured authority.
- CRD OpenAPI schemas reject structurally invalid custom resources before storage.
- Semantic errors are detected during reconciliation and reported through resource status and events.
- Deployment-specific rules can be enforced before reconciliation using Kubernetes admission policy.

**Evidence:** E-01, E-02, E-03, E-08.

**Limitations:** Kubernetes administrators and identities granted write access to Strimzi resources are trusted.
Reconciliation-time validation is not equivalent to rejecting the original Kubernetes API request.

### AC-2 — Component and Kafka identities are managed without hard-coded credentials

**Claim:** Strimzi generates or references unique credentials for managed identities and automates certificate lifecycle operations.

**Argument:**

- Strimzi generates Cluster CA and Clients CA material by default, or accepts supported externally managed CA configurations.
- Broker and component certificates are generated for their intended identities.
- `KafkaUser` resources can cause TLS or SCRAM credentials to be generated and stored in Kubernetes Secrets.
- Certificate renewal and CA rotation are reconciled by the operator.

**Evidence:** E-01, E-02, E-09.

**Limitations:** Secret confidentiality depends on Kubernetes RBAC and encryption-at-rest configuration.
Custom CA configuration and trust-store management introduce deployment-specific risk.
Credential revocation and client trust updates may require operational coordination.

### AC-3 — Network communication can be authenticated and encrypted

**Claim:** Strimzi provides secure communication mechanisms for internal Kafka traffic and user-facing Kafka listeners.

**Argument:**

- Secure internal Kafka communication is configured using Strimzi-managed certificates in the default secure mode.
- Kafka listeners support TLS and mTLS, SCRAM-SHA-512, OAuth 2.0, and custom authentication mechanisms.
- Kafka authorization can be enabled at the cluster level through the `Kafka` custom resource.
- Strimzi generates NetworkPolicy resources, and listener peer restrictions can be configured.
- Component REST APIs are treated as separate trust boundaries and require interface-specific exposure controls.

**Evidence:** E-01, E-02, E-04, E-05.

**Limitations:** User-facing listeners can be configured without TLS or authentication.
NetworkPolicy enforcement depends on the CNI.
Generated listener NetworkPolicies do not by themselves guarantee least-access source restrictions.
REST API authentication is not universally provided by Kubernetes RBAC.

### AC-4 — Privileges and workload security are limited where practical

**Claim:** Strimzi applies least privilege and workload hardening controls appropriate to its operator architecture.

**Argument:**

- Components use dedicated or intentionally shared ServiceAccounts based on their functions.
- ClusterRoles are separated by purpose, including operator, watched-resource, broker, entity-operator, client, and leader-election responsibilities.
- Namespace-scoped watch configuration can reduce the set of namespace resources controlled by the operator.
- Strimzi provides Baseline and Restricted pod security providers, with the Restricted provider as the default since Strimzi 1.2.0.
- The Restricted provider applies controls such as non-root execution, disabled privilege escalation, dropped Linux capabilities, and a runtime-default seccomp profile where compatible.

**Evidence:** E-01, E-03, E-10, E-11.

**Limitations:** Some features require cluster-scoped access or additional Linux capabilities.
User templates and custom PodSecurityProviders can override generated settings.
The operator remains a high-value privileged controller.

### AC-5 — Common unsafe input and implementation patterns are reduced

**Claim:** Strimzi reduces common implementation weaknesses through typed APIs, validation, safe libraries, review, automated testing, and analysis.

**Argument:**

- Custom resources use generated Java model classes and Kubernetes CRD schemas.
- The Fabric8 Kubernetes client provides typed serialization for Kubernetes API interaction.
- Semantic validation occurs during reconciliation.
- Pull requests undergo review and automated tests.
- CodeQL, dependency scanning, container scanning, and license scanning are run through project workflows.
- Dependency and base-image vulnerabilities can be remediated through normal releases or the CVE rebuild process.

**Evidence:** E-06, E-07, E-12, E-13.

**Limitations:** Static analysis and testing cannot prove the absence of vulnerabilities.
Java reduces exposure to common memory-corruption errors in project source but does not eliminate vulnerabilities in native libraries, the JVM, Kafka, operating-system packages, or third-party plugins.

### AC-6 — Published artifacts can be independently verified

**Claim:** Consumers can verify the origin of Strimzi container images and inspect published software composition information.

**Argument:**

- Strimzi documents verification of container image signatures.
- Current releases use GitHub OIDC-based keyless signing.
- SBOMs are published in documented formats and are signed.
- Users can pin image digests and enforce verification using deployment or admission policy.

**Evidence:** E-06 and E-14.

**Limitations:** Strimzi publication of signatures does not force a Kubernetes deployment to verify them.
Provenance attestations should be claimed only when an exact published artifact and verification procedure are linked.

### AC-7 — Security defects are handled through a public process

**Claim:** The project provides a mechanism to report, assess, fix, and disclose security vulnerabilities.

**Argument:**

- Security reports are accepted privately through the documented maintainer channel.
- The security policy tells users not to report active vulnerabilities publicly.
- Security advisories identify affected and fixed versions.
- Patch or minor releases are used to deliver fixes based on severity and upstream availability.
- Public advisories provide evidence that identified weaknesses have resulted in corrective changes.

**Evidence:** E-08, E-09, E-15.

**Limitations:** Response times and supported-version commitments should not be inferred beyond the published security policy.
Vulnerabilities in Apache Kafka and external components may depend on upstream fixes.

## 10. Secure Design Principles

The following table provides the assurance argument that commonly accepted secure design principles have been applied.
These are design arguments, not claims that the principles are perfectly achieved in every configuration.

| Principle                       | Application and argument                                                                                                                                                                                                          | Evidence                | Qualification                                                                                                                                                                                          |
|---------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Least Privilege**             | Operator and operand permissions are divided across purpose-specific roles and ServiceAccounts. Watch scope and feature controls can reduce effective authority.                                                                  | E-01, E-03, E-09.       | Some cluster-scoped and delegation permissions are intrinsic to the operator model. Historical advisories show that role design requires continuing review.                                            |
| **Fail-Safe Defaults**          | Internal Kafka identity and encryption are configured securely by default in the standard secure mode. Certificates are generated automatically. NetworkPolicy generation is enabled by default.                                  | E-01, E-02.             | User listeners, REST APIs, pod security provider selection, and listener source restrictions require explicit security decisions. The default is not universally the strongest possible configuration. |
| **Complete Mediation**          | Kubernetes API changes pass through Kubernetes authentication and authorization; Kafka access passes through listener authentication and optional authorization; operator changes are reconciled from declared resources.         | E-01, E-02.             | Direct Kafka administration and component REST APIs are separate paths and are not fully mediated by Kubernetes RBAC.                                                                                  |
| **Separation of Privilege**     | Cluster and Clients CAs separate component identity from client identity. Operator, broker, entity-operator, and client permissions are represented by separate roles. Release publication requires repository and CI identities. | E-03, E-06.             | Some components share pods or ServiceAccounts. Compromise of a CA, operator, or shared identity can have broad impact.                                                                                 |
| **Economy of Mechanism**        | Strimzi reuses Kubernetes RBAC, Secrets, Services, NetworkPolicy, and standard TLS instead of introducing independent security subsystems for each concern.                                                                       | E-01, E-02, E-03.       | The combination of Kubernetes, Kafka, operators, and extensibility remains operationally complex.                                                                                                      |
| **Open Design**                 | Source code, design proposals, documentation, release processes, and vulnerability advisories are publicly available. Security does not rely on secret algorithms or undocumented protocols.                                      | E-06, E-08, E-09, E-16. | Public design does not replace review, secure configuration, or testing.                                                                                                                               |
| **Least Common Mechanism**      | Major controllers and operands are separated into different workloads where practical, with communication through defined APIs and Kafka protocols.                                                                               | E-01, E-03.             | Entity Operator containers and some shared Kubernetes mechanisms create common dependencies and shared failure domains.                                                                                |
| **Psychological Acceptability** | Certificates and many security resources are generated declaratively from the same custom resources used for deployment. Secure client mechanisms are exposed through typed API fields.                                           | E-01, E-02.             | The large configuration surface can still lead to insecure choices. Clear documentation and examples are part of the control.                                                                          |
| **Defense in Depth**            | Identity, TLS, Kafka authorization, Kubernetes RBAC, NetworkPolicy, pod security, signed releases, scanning, and vulnerability response operate at different layers.                                                              | E-01 through E-15.      | No individual layer is sufficient. Several controls depend on user or platform configuration.                                                                                                          |

## 11. Common Implementation Weakness Countermeasures

The project uses relevant entries from the CWE Top 25 and related CWE classes as a checklist. Entries are selected based on Strimzi's Java/Kubernetes/operator architecture.

| CWE         | Weakness                                                 | Countermeasure argument                                                                                                                                                                                          | Evidence and limitations                                                                                                                                                |
|-------------|----------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **CWE-20**  | Improper Input Validation                                | CRD schemas validate structure and generated model types constrain input shape. The operator performs semantic validation during reconciliation.                                                                 | E-01, E-02. Semantic failures may occur after the API server stores the resource. Admission policy is needed for organization-specific constraints.                     |
| **CWE-287** | Improper Authentication                                  | Kubernetes authenticates API callers. Kafka listeners support mTLS, SCRAM-SHA-512, OAuth 2.0, and custom authentication. Internal Kafka identities use managed certificates in secure modes.                     | E-01, E-02. Authentication of component REST API callers must be considered separately.                                                                                 |
| **CWE-306** | Missing Authentication for Critical Function             | Critical Kubernetes mutations require authenticated Kubernetes API access. Kafka authentication can be enabled on listeners. Security guidance identifies REST APIs that must not be exposed without controls.   | E-01, E-04. Not every REST/metrics/health endpoint uses the same authentication model.                                                                                  |
| **CWE-862** | Missing Authorization                                    | Kubernetes RBAC controls custom resources and managed resource access. Kafka authorization can enforce ACLs and other supported broker-authorizer policies.                                                      | E-01, E-02, E-03. Kafka authorization is optional and cluster-scoped, not a per-listener guarantee.                                                                     |
| **CWE-863** | Incorrect Authorization                                  | Roles are separated by function, namespace watch scope is configurable, and authorization defects are tracked through advisories and regression fixes.                                                           | E-03, E-09. The operator delegation model requires careful review and has had historical cross-namespace and excessive-Secret-access defects.                           |
| **CWE-269** | Improper Privilege Management                            | Dedicated ServiceAccounts, separated roles, non-root images/defaults, and the default Restricted pod security provider reduce privilege.                                                                           | E-01, E-03, E-10. Builds and some Kubernetes features require broader capabilities; templates can override generated security contexts.                                 |
| **CWE-200** | Exposure of Sensitive Information                        | Credentials are generated dynamically and stored in Kubernetes Secrets; Kafka connections can use TLS; network exposure can be restricted.                                                                       | E-01, E-02. Secret encryption at rest and protection from authorized namespace readers are Kubernetes responsibilities. Data at rest on Kafka volumes is outside scope. |
| **CWE-798** | Use of Hard-Coded Credentials                            | Cluster, broker, operator, and managed user credentials are generated or supplied through referenced Secrets rather than embedded in source code or default images.                                              | E-01, E-02. User-supplied external systems and plugins may contain their own credentials.                                                                               |
| **CWE-400** | Uncontrolled Resource Consumption                        | Kubernetes resource requests/limits, quotas, Kafka quotas, reconciliation controls, and scoped watch configuration provide layers of resource control.                                                           | E-01, E-02. Strimzi does not implement a universal per-tenant API rate limiter, and valid configurations can still be operationally expensive.                          |
| **CWE-502** | Deserialization of Untrusted Data                        | Kubernetes resources are deserialized through generated models and maintained serialization libraries rather than arbitrary Java object deserialization.                                                         | E-02, E-12. Parser and dependency vulnerabilities remain possible and are addressed through updates and scanning.                                                       |
| **CWE-829** | Inclusion of Functionality from Untrusted Control Sphere | Kafka Connect plugin installation is explicit and can use trusted artifacts, checksums, controlled registries, and admission policy. Third-party plugins are documented as outside Strimzi's security guarantee. | E-01. Plugin behaviour is not sandboxed by Strimzi and remains a significant residual risk.                                                                             |
| **CWE-494** | Download of Code Without Integrity Check                 | Published Strimzi images and SBOMs are signed and can be verified. Connect build artifacts should be pinned or checksum-verified where supported.                                                                | E-06, E-14. Verification must be performed or enforced by the consumer; the existence of a signature alone is not enforcement.                                          |

### Memory-safety weaknesses

Most Strimzi operator source code is Java, which prevents many direct buffer-overflow and use-after-free errors in project code. This does not make memory-safety vulnerabilities irrelevant to the deployed system: the JVM, operating-system packages, native libraries, Apache Kafka dependencies, and third-party components remain part of the runtime supply chain. Dependency and container scanning are therefore used as compensating detection controls rather than treating all memory-safety CWEs as categorically impossible.

## 12. Evidence Index

Evidence should be pinned to the assessed release or commit before publication. Links to `latest` and `main` are convenient working references but can change over time.

| ID       | Evidence                                                                                                                                                         |
|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **E-01** | Strimzi Deploying and Managing documentation: <https://strimzi.io/docs/operators/latest/deploying>                                                               |
| **E-02** | Strimzi Custom Resource API/configuration reference: <https://strimzi.io/docs/operators/latest/configuring.html>                                                 |
| **E-03** | Cluster Operator installation and RBAC manifests: <https://github.com/strimzi/strimzi-kafka-operator/tree/main/install/cluster-operator>                         |
| **E-04** | Guidance for limiting Kafka Connect API access: <https://strimzi.io/docs/operators/latest/deploying#con-securing-kafka-connect-api-str>                          |
| **E-05** | Listener NetworkPolicy guidance: <https://strimzi.io/docs/operators/latest/full/deploying#con-restricting-access-to-listeners-network-policies-str>              |
| **E-06** | Strimzi repository README, image signatures, and SBOM verification: <https://github.com/strimzi/strimzi-kafka-operator/blob/main/README.md#container-signatures> |
| **E-07** | Strimzi GitHub Actions workflows: <https://github.com/strimzi/strimzi-kafka-operator/tree/main/.github/workflows>                                                |
| **E-08** | Strimzi security policy: <https://github.com/strimzi/.github/blob/main/SECURITY.md>                                                                              |
| **E-09** | Strimzi Security Advisories: <https://github.com/strimzi/strimzi-kafka-operator/security/advisories>                                                             |
| **E-10** | Strimzi pod security provider documentation: <https://strimzi.io/docs/operators/latest/deploying#con-pod-security-providers-str>                                 |
| **E-11** | Kubernetes RBAC good practices: <https://kubernetes.io/docs/concepts/security/rbac-good-practices/>                                                              |
| **E-12** | Strimzi source and generated model implementation: <https://github.com/strimzi/strimzi-kafka-operator>                                                           |
| **E-13** | Strimzi contributor and development documentation: <https://github.com/strimzi/strimzi-kafka-operator/blob/main/CONTRIBUTING.md>                                 |
| **E-14** | Strimzi releases: <https://github.com/strimzi/strimzi-kafka-operator/releases>                                                                                   |
| **E-15** | CVE container rebuild workflow: <https://github.com/strimzi/strimzi-kafka-operator/actions/workflows/cve-rebuild.yml>                                            |
| **E-16** | Strimzi proposals repository: <https://github.com/strimzi/proposals>                                                                                             |

### Evidence still to pin before publication

The following evidence should be changed from a repository directory or moving `latest` link to an exact release tag, commit, workflow file, test, or manifest:

- the exact Strimzi 1.3.0 documentation URLs or archived release documentation;
- the exact RBAC YAML files supporting AC-1 and AC-4;
- tests that verify certificate trust, role boundaries, watched-namespace controls, and NetworkPolicy generation;
- exact CodeQL, Snyk, FOSSA, build, release, and CVE-rebuild workflow files;
- the exact governance or branch-protection evidence for review requirements;
- the exact release artifacts that constitute any claimed provenance attestation;
- regression tests associated with the published 2025 and 2026 security advisories.

## 13. Residual Risks

The project accepts or shares responsibility for the following residual risks:

1. **Cluster Operator compromise:** The operator necessarily has powerful reconciliation and delegation authority. Compromise can have broad impact within its configured Kubernetes authority.
2. **Kubernetes platform compromise:** A cluster administrator or compromised control plane can replace Strimzi images, Secrets, RBAC, policies, and custom resources.
3. **Insecure user configuration:** Strimzi intentionally supports plaintext listeners, custom images, custom plugins, template overrides, and other flexible configurations that can weaken security.
4. **Kafka Connect REST and plugins:** Kafka Connect exposes a powerful administrative API and loads third-party code into its runtime. These require strong network isolation and artifact trust.
5. **Credential exposure within Kubernetes:** Kubernetes Secrets can be read by any identity granted access, and storage encryption is not enabled by Strimzi.
6. **Kafka data at rest:** Persistent Kafka data is not encrypted by Strimzi.
7. **Direct Kafka administration:** Changes made directly through Kafka APIs may bypass Kubernetes desired-state and audit mechanisms.
8. **Denial of service:** Broadly authorized users and compromised clients can create high operational load even when requests are syntactically valid.
9. **External dependencies:** Apache Kafka, JVM, base images, Cruise Control, Kafka Exporter, Bridge, plugins, and other dependencies may contain vulnerabilities requiring upstream remediation.
10. **Custom extensions:** Custom security providers and authentication/authorization implementations are trusted code outside the default control set.
11. **Verification enforcement:** Signed images and SBOMs only provide protection when consumers verify or enforce them.

These residual risks do not invalidate the top-level claim when the assumptions and recommended controls are satisfied, but they define the limit of that claim.

## 14. Maintenance of the Assurance Case

This document should be reviewed:

- for every Strimzi minor release;
- whenever a security-relevant proposal is implemented or a default changes;
- after publication of a Strimzi security advisory;
- after material RBAC, certificate-management, listener, build, or release-workflow changes;
- before submitting or renewing the OpenSSF Best Practices Silver badge evidence.

Each review should:

1. update the assessed release and source commit;
2. verify every evidence link;
3. confirm that security requirements still match documented behaviour;
4. add new trust boundaries and threats introduced by new components;
5. map new advisories to failed or improved controls;
6. reassess residual risk and shared-responsibility assumptions.

## 15. Conclusion

Strimzi's security case is based on layered controls rather than a single security mechanism.
Kubernetes RBAC and workload isolation protect the control plane; certificates, TLS, authentication, and Kafka authorization protect communication and identity; validation and reconciliation constrain desired state; signed images, SBOMs, automated analysis, review, and vulnerability response protect the development and release lifecycle.

The assurance is conditional.
Strimzi is a highly configurable Kubernetes operator, and several essential controls are provided or enforced by the Kubernetes platform or selected by the user.
The project therefore considers its security requirements adequately met only for supported, patched deployments that satisfy the assumptions and responsibilities stated in this document.
