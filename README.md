#v0.3 Beyond DNS: Unlocking the Internet of AI Agents via the NANDA Quilt of Registries and Verified AgentFacts

**Ramesh Raskar** (raskar@media.mit.edu), **Pradyumna Chari** (pchari@media.mit.edu), **Jared James Grogan** (jared.grogan@post.harvard.edu), **Mahesh Lambe** (mahesh@unifydynamics.com), **Rajesh Ranjan** (Rajeshr2@tepper.cmu.edu), **John Zinky** (jzinky@akamai.com), **Rekha Singhal** (rekha.singhal@tcs.com), **Shailja Gupta** (Shailjag@tepper.cmu.edu), **Robert Lincourt** (robert.lincourt@dell.com), **Raghu Bala** (raghu@synergetics.ai), **Abhishek Singh** (abhi24@media.mit.edu), **Ayush Chopra** (ayushc@media.mit.edu), **Dimitris Stripelis** (dimitris@flower.ai), **Bhuwan B** (bhuwan.b@hcltech.com), **Sumit Kumar** (sumit-k@hcltech.com), **Maria Gorskikh** (Maria.gorskikh1@gmail.com), **Sichao Wang** (wangcan@cisco.com)

**Project NANDA**

***

## Abstract

[cite_start]The Internet is poised to host billions to trillions of autonomous AI agents that negotiate, delegate, and migrate in milliseconds, with workloads that will strain DNS-centered identity and discovery[cite: 1, 2]. [cite_start]This paper describes the **NANDA quilt registry architecture**, envisioned as a means for discoverability, identifiability, and authentication in the internet of AI agents[cite: 2]. [cite_start]We present an architecture where a minimal lean registry resolves to dynamic, cryptographically verifiable **AgentFacts** that support multi-endpoint routing, load balancing, privacy-preserving access, and credentialed capability assertions[cite: 3]. [cite_start]Our architecture design delivers four concrete guarantees: (1) a quilt-like registry proposal supporting both NANDA-native and third-party agents, (2) rapid global resolution for new AI agents, (3) sub-second revocation and key rotation, (4) schema-validated capability assertions, and (5) privacy-preserving discovery across organizational boundaries via verifiable, least-disclosure queries[cite: 4]. [cite_start]We formalize the AgentFacts schema, specify a CRDT-based update protocol, and prototype adaptive routers that cache and invalidate AgentFacts in 50-200 ms at a 10-billion-lookup/s scale[cite: 5]. [cite_start]The result is a lightweight, horizontally scalable foundation for secure, trust-aware collaboration for the next generation of the Internet of AI agents, without abandoning existing web infrastructure[cite: 6].

***

## I. Introduction

[cite_start]Autonomous AI agents are emerging as a critical abstraction layer for executing tasks, communicating, reasoning, and making decisions[cite: 7]. [cite_start]They are expected to scale into trillions of active participants in a globally distributed, interconnected environment[cite: 8]. [cite_start]This evolving reality demands foundational infrastructure that supports privacy, discovery, low-latency resolution, and verifiable trust evaluation of AI agents across a trillion-scale, decentralized, and highly dynamic ecosystem[cite: 9]. [cite_start]While DNS and HTTPS certificates are effective for mapping human-readable domains to static endpoints, their update cycles and trust model are insufficient for agent-centric systems[cite: 10]. [cite_start]We propose an architecture for a lean NANDA registry, paired with an AgentFacts schema, to enable dynamic, privacy-preserving, and low-latency resolution without compromising extensibility, interoperability, and trust[cite: 11].

Our primary contributions are:
1.  [cite_start]A **minimal core registry** with essential static metadata (agent IDs, credential pointers, and dynamic agent facts URLs) of ≤ 120 B per record to reduce update needs[cite: 12]. [cite_start]We also introduce a self-describing and verifiable AgentFacts schema (JSON-LD documents) whose `@context` enables forward-compatible schema evolution, allowing agents to dynamically update capabilities, endpoints, and authentication info without modifying the registry[cite: 13]. [cite_start]TTL-based endpoint resolution supports decoupling, caching, DDOS protection, and flexible deployment[cite: 14].
2.  [cite_start]The registry is designed for **interoperability** with enterprise agent registries, allowing various ways for agents to be visible via the NANDA registry[cite: 15].
3.  [cite_start]A **Dual-Path Privacy Resolution** mechanism enables metadata lookups without revealing the requester’s identity, aligning with zero-trust principles[cite: 16]. [cite_start]AgentFacts also enables cryptographic verification from trusted whitelisted issuers to ensure authenticity and prevent tampering; short-lived verifiable credentials (< 5 min) allow for sub-second revocation[cite: 17].

[cite_start]At a high level, the registry transforms the complexity of a fully connected N×N network into a simpler 2N problem by acting as a shared handshake channel, allowing any two agents to discover and connect through a common interface[cite: 18, 19]. [cite_start]Once the handshake is complete, the registry is no longer required for ongoing communication unless the session expires based on a defined time-to-live (TTL) parameter[cite: 20].

[cite_start]The current Domain Name System (DNS), foundational to the internet since 1983, was designed for static web infrastructure, not the dynamic, context-aware operation of autonomous AI agents[cite: 21]. [cite_start]As we envision a future with trillions of interoperable agents, DNS becomes a bottleneck in scale and architectural assumptions[cite: 22]. [cite_start]We argue that an open, agentic internet cannot be realized within the constraints of DNS[cite: 23]. [cite_start]We propose a new system that decouples identity from metadata, supports multi-hop resolution, and provides a flexible substrate for agent discovery, routing, and capability negotiation[cite: 24]. [cite_start]This infrastructure is a prerequisite for the agentic web itself[cite: 25].

[cite_start]We view the NANDA registry as a **quilt of agents, resources, and tools** across platforms, organizations, and protocols[cite: 27]. [cite_start]This approach allows for global interoperability, discoverability, and flexible governance of agents[cite: 28]. [cite_start]NANDA need not authenticate, authorize, and govern all agents on the internet of AI agents[cite: 28]. [cite_start]Commercial, governmental, and individual entities can determine whether agents are directly certified and visible via the NANDA registry, or if the NANDA registry merely holds a redirect to their platforms, allowing them deeper control over their agents[cite: 29].

***

## II. Design Goals

[cite_start]The design of an Internet-scale decentralized AI agent registry infrastructure must account for a wide spectrum of operational and architectural requirements, from performance and scalability to security, privacy, trust, governance, and resilience[cite: 31]. [cite_start]This section outlines the key goals that informed the proposed registry model[cite: 32].

### Problems We Are Solving:
* [cite_start]**Mitigate Registry Writes & Delay**: DNS, BGP, and WHOIS were built for millions of static records, not billions of updates per hour or sub-second worldwide sync[cite: 33].
* **Trust Gap**: A TLS certificate only proves domain ownership, not an agent's code or behavior. [cite_start]We need signed capability proofs and instant revocation[cite: 34].
* **Privacy & Split-Horizon**: Today’s look-ups expose who is asking for what. [cite_start]Enterprises need private, split-horizon resolution and tamper-evident audit logs[cite: 36].
* [cite_start]**Routing Limits**: Fixed A/AAAA records cannot keep up with agents that move every few seconds or require geo-based DDoS shuffling[cite: 37].
* [cite_start]**Governance & Liability**: Without a transparent, append-only log, no one can prove compliance or assign blame when agents misbehave across borders[cite: 38].

Our approach is deliberately reductionist:
* [cite_start]**Isolate the bottleneck**: For each problem, we ask what single property of the current stack makes it untenable at the trillion-agent internet scale[cite: 39].
* [cite_start]**Introduce exactly one constraint-shifting primitive**: e.g., lean-record, privacy path, or VC-signed fact that removes that bottleneck without disturbing layers that already scale[cite: 40].
* [cite_start]**Accept and surface the trade-off**: Every fix carries an equal-and-opposite cost elsewhere; listing pros/cons next to each goal keeps those tensions explicit[cite: 41].

The goals that follow form an orthogonal checklist rather than a monolithic blueprint. [cite_start]An implementer may adopt some goals and defer others based on their priorities[cite: 43]. [cite_start]The following subsections provide the technical detail[cite: 45].

### A. Lightweight Reference Registry
[cite_start]**Goal**: Achieve lean, stable operation with records constrained to ≤120B to ensure scalability and fault tolerance[cite: 46].
[cite_start]**Issues**: Current DNS infrastructure, processing 10B daily lookups for only 300M addresses, demonstrates that direct scaling with dynamic agent state would be unsustainable for billions of unique agents[cite: 47].
[cite_start]**Proposal**: Implement the registry as a static reference layer that maps agent identifiers to endpoints and relevant metadata, reducing write frequency by ~10⁴× relative to DNS[cite: 48].
[cite_start]**Co-dependencies**: This architecture requires a strict separation between static routing information in the registry and dynamic agent state managed at the endpoints[cite: 49].
[cite_start]**Future Evolution**: The minimal core design may need to accommodate additional metadata fields or enhanced routing mechanisms while preserving the static reference mapping principle[cite: 50].

### B. Enabling Diverse Agent Registration Models
[cite_start]**Goal**: A registry architecture that accommodates diverse agent registration models while maintaining privacy, split-horizon governance, and liability management across heterogeneous administrative domains[cite: 51].
[cite_start]**Issues**: Monolithic registry designs cannot address the varied governance, privacy, and liability requirements of agents operating under different frameworks[cite: 52].
[cite_start]**Proposal**: Deploy a quilt-like architecture enabling multiple registration types: agents can be natively registered directly via the NANDA registry or administered by external commercial/governmental entities while maintaining visibility through the NANDA registry interface[cite: 53].
[cite_start]**Co-dependencies**: This model requires interoperability protocols between NANDA and external registries, along with consistent metadata standards across boundaries[cite: 53].
[cite_start]**Future Evolution**: The quilt architecture may expand to support additional registration categories and cross-registry federation mechanisms[cite: 54].

| Entity | Agent Name | Link to Agent and AgentFacts | Comment |
| :--- | :--- | :--- | :--- |
| **Civil Society** | `@agentx` | NANDA native URL | [cite_start]Agents for individuals, or unaffiliated commercial entities [cite: 55] |
| **Government** | `@US:shop` | NANDA native URL | [cite_start]Location-specific domains, similar to .com, .co.uk etc. [cite: 55] |
| **Enterprise (Agent-stores)** | `@company` | Routed via `@company` | [cite_start]Agents within enterprise agent store (Salesforce, Google, Microsoft, Amazon etc.) but only accessible via first going to the agent store [cite: 55] |
| **Enterprise (Agent-stores)** | `@company:shop` | URL administered by ‘company’ | [cite_start]Agents within enterprise agent store (Salesforce, Google etc.) but also directly visible on the registry [cite: 55] |
| **Web3** | `@DID:company` | Routed via company marketplace | [cite_start]Agents within Web3 agent markets, only accessible via first going to the agent store [cite: 55] |
| **Web3** | `@DID:company:agent` | DID administered by company, routed via NANDA | [cite_start]Agents administered by Web3 agent markets, and authenticated via decentralized identifiers [cite: 55] |
[cite_start]***Table 1: The NANDA registry is envisioned as a quilt of directly-registered agents, but also enterprise and governmental agents visible on the registry to varying degrees.*** [cite: 56]

### C. Endpoint Agility and Sub-Second Reachability
[cite_start]**Goal**: Dynamic endpoint management for AI agents in variable deployment environments requiring frequent network changes, while maintaining sub-second reachability[cite: 57].
[cite_start]**Issues**: Static endpoints create scalability bottlenecks and security vulnerabilities, while frequent registry updates would violate the minimal core architecture principles[cite: 58].
[cite_start]**Proposal**: Decouple endpoint resolution from the registry by delegating it to AgentFacts documents with TTL-scoped endpoint lists: static (1-6h), rotating (5-15 min), or adaptive (30-60s)[cite: 59]. [cite_start]The registry stores FactsURL/PrivateFactsURL references and AgentFacts signatures to prevent spoofing[cite: 59].
[cite_start]**Co-dependencies**: This architecture requires TTL parameter support for adaptive caching, cryptographic signature validation for AgentFacts documents, and coordination between registry lookups and facts resolution[cite: 60].
[cite_start]**Future Evolution**: The endpoint agility system may incorporate more granular TTL categories, enhanced load balancing algorithms, and advanced security mechanisms[cite: 61].

### D. Improving Scalability through Decentralizations
[cite_start]**Goal**: Asynchronous metadata publishing and revision by agents or publishers without registry write-overheads, supporting 10k updates/sec per registry shard[cite: 61].
[cite_start]**Issues**: Centralized metadata updates create bottlenecks and dependencies on registry availability, limiting system scalability and agent autonomy[cite: 62].
[cite_start]**Proposal**: Implement decentralized updates where agents sign AgentFacts JSON-LD documents with W3C Verifiable Credentials, upload to self-selected hosts (IPFS CID, CDN, or enterprise bucket), and publish the resulting URL under existing facts_url pointers in the lean registry[cite: 63].
[cite_start]**Co-dependencies**: Requires Verifiable Credential infrastructure, cryptographic signature validation, and coordination between multiple hosting platforms for facts[cite: 64].
[cite_start]**Future Evolution**: May expand to support additional credential standards and hosting mechanisms while maintaining the core principle of decentralized, authenticated metadata management[cite: 65].

### E. Privacy Preservation
[cite_start]**Goal**: Privacy-preserving agent resolution aligned with data minimization principles, preventing exposure of requester identity except for timing leaks[cite: 66].
[cite_start]**Issues**: Standard resolution exposes access patterns to passive tracking by agents, network observers, and malicious registry operators[cite: 67].
[cite_start]**Proposal**: Deploy dual-path resolution via PrimaryFactsURL (URL1) and PrivateFactsURL (URL2), where URL2 can be hosted by third parties or decentralized storage to shield access patterns[cite: 68].
[cite_start]**Co-dependencies**: Requires third-party hosting infrastructure, protocols for anonymous requests, cryptographic signature validation, and policy frameworks[cite: 69].
[cite_start]**Future Evolution**: May incorporate advanced anti-timing attack mechanisms and enterprise-specific "PrivateFacts-only" modes while balancing privacy protection with latency requirements[cite: 70].

### F. Flexible Routing Choices
[cite_start]**Goal**: Provide every AI agent with latency-, privacy-, and resilience-aware routing choices[cite: 71].
[cite_start]**Issues**: Single-path resolution cannot accommodate diverse operational requirements like geo-distributed load balancing and failover routing[cite: 72].
[cite_start]**Proposal**: Implement three routing mechanisms: static endpoint discovery via PrimaryFactsURL (URL1), privacy-preserving metadata fetch via IPFS/Tor-style relays as PrivateFactsURL (URL2), and a cryptographically signed AdaptiveRouterURL for programmable real-time endpoint selection[cite: 73].
[cite_start]**Co-dependencies**: Requires IPFS/Tor relay infrastructure, real-time routing algorithms, cryptographic signature validation for adaptive routers, and coordination between static and dynamic endpoint selection mechanisms[cite: 74].
[cite_start]**Future Evolution**: May incorporate machine learning-based routing optimization and enhanced shuffle-sharding algorithms[cite: 75].

### G. From Self-Advertising to Audited Metadata
[cite_start]**Goal**: Establish a verifiable claims infrastructure to prevent AI agents from spoofing capabilities, impersonating reputable actors, or conducting malicious activities[cite: 76].
[cite_start]**Issues**: Self-advertising metadata enables malicious agents to make unverifiable claims about capabilities and identity, creating security vulnerabilities and trust deficits[cite: 77].
[cite_start]**Proposal**: Treat AgentFacts as verifiable claims requiring W3C Verifiable Credential v2 cryptographic attestation, with issuers operating within domain-specific trust zones that can cross-sign each other[cite: 78].
[cite_start]**Co-dependencies**: Requires W3C Verifiable Credential infrastructure, ID resolution systems, cross-signing protocols between trust zones, and VC-Status-List revocation mechanisms[cite: 79].
[cite_start]**Future Evolution**: May expand trust zone federation models and enhanced revocation mechanisms while maintaining the core principle of cryptographically attested, auditable agent metadata[cite: 80].

| Goal | Core problem solved | Key pros | Key cons | Status* |
| :--- | :--- | :--- | :--- | :--- |
| **A: Lightweight Reference Registry** | [cite_start]DNS write-overhead & propagation lag [cite: 83] | [cite_start]10⁴ × fewer writes [cite: 83] | [cite_start]Lean schema limits rich search [cite: 83] | ✔ |
| **B: Diverse Agent Registration Models** | [cite_start]Allowing varying degrees of enterprise autonomy over agents [cite: 83] | [cite_start]Quilt of existing agent registries that can be accommodated within NANDA [cite: 83] | [cite_start]Lower quality of information about highly controlled agents, within the NANDA registry/AgentFacts [cite: 83] | ✔ |
| **C: Endpoint Agility and Sub-Second Reachability** | [cite_start]Endpoint churn every few seconds [cite: 83] | [cite_start]< 1 s fail-over / DDoS shuffle [cite: 83] | [cite_start]Resolver cache churn [cite: 83] | △ |
| **D: Improving Scalability through Decentralization** | [cite_start]Registry bottleneck at 10 B updates / h [cite: 83] | [cite_start]Unlimited parallel self-hosted updates [cite: 83] | [cite_start]Replay window risk [cite: 83] | ✔ |
| **E: Privacy Preservation** | [cite_start]Query privacy & split-horizon governance [cite: 83] | [cite_start]Unlinkable look-ups / least-disclosure [cite: 83] | [cite_start]Adds 30-60 ms latency [cite: 83] | △ |
| **F: Flexible Routing Choices** | [cite_start]Geo-LB, fail-over, capability dispatch [cite: 83] | [cite_start]Operator chooses best path [cite: 83] | [cite_start]Client-side complexity [cite: 83] | ✔ |
| **G: From Self-Advertising to Audited Metadata** | [cite_start]Capability spoofing & supply-chain attacks [cite: 83] | [cite_start]Offline VC verification [cite: 83] | [cite_start]VC payload bloat [cite: 83] | ✔ |
[cite_start]***Table 2: Summary of each design goal, the problem it solves, pros, cons, and current maturity*** [cite: 85]
[cite_start]***\* Status legend – ✔ fully specified, △ partial / open items, ✖ design stub.*** [cite: 84]

***

## III. System Architecture Overview

[cite_start]The proposed registry system architecture is organized into a modular, hierarchical stack designed for internet-scale, autonomous AI agent ecosystems[cite: 87]. [cite_start]It separates (1) static identity resolution, (2) verifiable metadata distribution, and (3) dynamic endpoint routing[cite: 88]. [cite_start]This separation enables performance optimization and secure, decentralized operation across heterogeneous networks, cutting registry write load by 10⁴× while maintaining <1 s global resolution[cite: 89]. [cite_start]To meet the requirements of low-latency discovery, cryptographic trust evaluation, privacy, and cross-domain governance, we structure the architecture into three levels: **Lean Registry**, **AgentFacts**, and **Dynamic Resolution**[cite: 90].

1.  **Registry Level (Anchor Tier)**:
    * [cite_start]Provides decentralized mapping from agent identifiers (IDs) to metadata URLs (AgentAddr)[cite: 92].
    * [cite_start]Records are cacheable and Ed25519-signed, reducing registry-write overheads and blocking tampering[cite: 93].
    * [cite_start]Supports resolution via short-lived, cacheable records resistant to tampering and unauthorized updates[cite: 95].
2.  **AgentFacts Level (Metadata Distribution Tier)**:
    * [cite_start]Hosts self-describing JSON-LD documents (AgentFacts), signed as W3C VCs, containing endpoint lists, capability descriptors, and more[cite: 96].
    * [cite_start]Enables frequent, independent updates without registry-level intervention[cite: 97].
    * [cite_start]Supports hosting at agent-owned domains or neutral third parties for privacy-preserving resolution[cite: 97].
3.  **Dynamic Resolution Level (Adaptive Routing Tier)**:
    * [cite_start]Dynamically interprets AgentFacts metadata to resolve live endpoints[cite: 98].
    * [cite_start]Includes support for decentralized adaptive resolvers for load-balancing, geolocation, or behavior-routing[cite: 99].
    * [cite_start]TTL-based resolution ensures endpoint freshness while minimizing re-resolution overhead[cite: 100].

[cite_start]This layered model allows the core registry to remain minimal and stable, while pushing richer metadata and operational agility to the agent facts and resolution services[cite: 100].

***

## IV. The Lean Registry

[cite_start]The registry resolver returns a cryptographically signed, cacheable object called **AgentAddr**, a lightweight address record for the agent[cite: 102]. [cite_start]This object encapsulates the agent’s identifier, TTL, and pointers to metadata and optional routing infrastructure[cite: 103].

* [cite_start]`agent_id` (Machine-readable ID): A globally unique agent identifier[cite: 104].
* [cite_start]`agent_name` (URN): A human-readable alias encoded as a URN[cite: 104].
* [cite_start]`facts_url` (FactsURL): A reference to the AgentFacts hosted at the agent's domain[cite: 105].
* [cite_start]`private_facts_url` (PrivateFactsURL): An optional, privacy-enhanced reference to the AgentFacts hosted on a third-party or decentralized service[cite: 106].
* [cite_start]`adaptive_router_url` (optional): An optional endpoint for dynamic routing services[cite: 107].
* [cite_start]`ttl` (Time-To-Live): The maximum cache duration before the client must re-resolve the record[cite: 108].
* [cite_start]`signature`: A cryptographic signature from the registry resolver that binds the contents of the AgentAddr[cite: 109].

[cite_start]The `AgentAddr` object, being signed and cacheable, allows clients to redistribute it and perform verification without repeated lookups[cite: 110]. [cite_start]Its role is similar to a DNS record but extended to include verifiable metadata pointers and flexible routing instructions[cite: 111].

| Field | Value |
| :--- | :--- |
| `agent_id` | [cite_start]`nanda:550e8400-e29b-41d4-a716-4466554400` [cite: 113] |
| `agent_name` | [cite_start]`agent:Company:TranslationAssistant` [cite: 113] |
| `facts_url` | [cite_start]`https://TranslationAssistant.salesforce.com/.agent-facts` [cite: 113] |
| `private_facts_url` | [cite_start]`https://agentfactshost.com/550e8400-e29b-41d4-a716` [cite: 113] |
| `adaptive_router_url` | [cite_start]`https://router.salesforce.com/dispatch/translation` [cite: 113] |
| `ttl` | [cite_start]`3600 (1 hour)` [cite: 113] |
| `signature` | [cite_start]`signature_hash_placeholder` [cite: 114] |
[cite_start]***Table 3: An example entry in the NANDA quilt of registries.*** [cite: 114]

### A. Lean Registry Resolution Paths
[cite_start]The registry supports multiple agent resolution workflows based on context, privacy requirements, and routing logic, all beginning with the resolution of the `AgentName` into an `AgentAddr`[cite: 115, 116].

1.  [cite_start]**Direct Communication (Endpoint Access)**: Used when the client intends to connect to an agent’s endpoint directly, and privacy is not a concern[cite: 117, 118].
    [cite_start]`AgentName → Registry → AgentAddr → Endpoint` [cite: 119]
2.  **Enterprise Registry Resolution**: Used when connecting to an agent within an enterprise registry. [cite_start]The client is routed to the enterprise registry for further resolution[cite: 122].
    [cite_start]`AgentName → Registry → EnterpriseRegistry → FactsURL or PrivateFactsURL or Endpoint` [cite: 123]
3.  **Privacy-Preserving Resolution via PrivateFactsURL - URL2**: When the client wishes to preserve its identity, it uses the `PrivateFactsURL`. [cite_start]This path does not resolve to an endpoint[cite: 124].
    [cite_start]`AgentName → Registry → AgentAddr → PrivateFactsURL → AgentFacts (metadata only)` [cite: 125]
4.  [cite_start]**Adaptive Routing Resolution**: When the agent supports dynamic, context-aware routing, the registry returns the router URL in the `AgentAddr`[cite: 125].
    [cite_start]`AgentName → Registry → AgentAddr → AdaptiveRouter → (Ephemeral) Endpoint` [cite: 128]

[cite_start]These resolution paths are unified by the use of cryptographically signed AgentFacts and credential validation logic that ensures agents cannot falsify capabilities or impersonate others[cite: 128].

### B. Quilt-Like Registry Model
The registry can be deployed as a quilt of registries for scalability. Each entry may:
* [cite_start]Point to an agent registered natively via the NANDA registry[cite: 132].
* [cite_start]Point directly to an enterprise agent registry for further resolution[cite: 132].
* [cite_start]Point to an enterprise agent whose identity is handled by the enterprise registry but is directly registered in the NANDA registry[cite: 132].

[cite_start]Each agent can also be identified by a traditional or decentralized identifier[cite: 133]. [cite_start]This quilt-like approach supports a range of agents, resources, and tools[cite: 135].

### C. Distributed Registry Model
[cite_start]The registry can be deployed as a globally distributed system for redundancy and load distribution[cite: 136, 137]. Each registry may:
* [cite_start]Be operated by an enterprise, cloud provider, or neutral public body[cite: 138].
* [cite_start]Federate with other registries using verifiable links, credential exchanges, or trust frameworks[cite: 138].

[cite_start]This federated model ensures organizational autonomy, fault tolerance, scalability, and interoperability[cite: 139].

***

## V. AgentFacts Schema and Resolution Mechanism

[cite_start]**AgentFacts** is a structured, cryptographically signed metadata document that encapsulates an AI agent’s dynamic state, capabilities, endpoints, and credentials[cite: 140]. [cite_start]It decouples the lean registry from volatile or sensitive metadata, enabling agile agent discovery and interaction at scale[cite: 141].

### A. Role and Benefits of the AgentFacts
AgentFacts acts as an intermediary data structure between a stable registry record and an agent's operational configuration, offering several advantages:
* [cite_start]**Dynamic Updatability**: AgentFacts can be updated independently of the registry[cite: 144].
* [cite_start]**Credentialed Assertions**: All claims are signed by issuers, ensuring tamper-resistance and reputation integrity[cite: 145].
* [cite_start]**Deployment Flexibility**: Facts can be hosted on the agent’s domain or on neutral infrastructure to meet privacy or reliability needs[cite: 146].
* [cite_start]**Multi-Endpoint Support**: Enables load balancing, routing to specialized instances, and failover mechanisms[cite: 147].

[cite_start]AgentFacts serve as the functional equivalent of a decentralized service manifest, including versioned metadata, capabilities, skills, telemetry endpoints, signed evaluations, and endpoint URIs with resolution TTLs[cite: 148, 149, 150, 151]. [cite_start]All AgentFacts must be signed by an authorized issuer under the NANDA trust framework or a domain-specific federated authority[cite: 151].

### Why another metadata artifact?
[cite_start]While the agentic community has descriptors like Google’s A2A Agent Card, it's important to show what it solves and what it leaves unsolved at a trillion-agent scale[cite: 152, 153].

| Dimension | Google A2A Agent Card | AgentFacts (this paper) |
| :--- | :--- | :--- |
| **Purpose** | [cite_start]Advertise a server-side agent’s endpoint and skills for JSON-RPC interaction [cite: 155] | [cite_start]Convey live endpoints plus cryptographically-signed capabilities, SBOM hash, privacy path, revocation info [cite: 155] |
| **Discovery Path** | [cite_start]One-step fetch (due to absence of registry piece) at `/.well-known/agent.json` [cite: 155] | [cite_start]Two-step: lean registry → FactsURL / PrivateFactsURL [cite: 155] |
| **Endpoint Freshness** | [cite_start]Assumes minutes-to-hours stability (no TTL field) [cite: 155] | [cite_start]TTL-scoped lists: static (1-6 h) or rotating (5-15 m) [cite: 155] |
| **Trust Primitive** | [cite_start]Plain HTTPS and optional token, self-declared attributes [cite: 155] | W3C Verifiable Credential v2 signatures; [cite_start]VC-Status revocation (<1 s), support for third-party audited attributes [cite: 155] |
| **Privacy Option** | [cite_start]None (lookup hits agent host) [cite: 155] | [cite_start]Optional PrivateFactsURL via IPFS/Tor; hides requester [cite: 155] |
| **Schema Weight** | [cite_start]≈ 0.3–1 KB JSON [cite: 155] | [cite_start]1–3 KB JSON-LD + VC [cite: 155] |
| **Best-fit Use Case** | [cite_start]Stable SaaS agents inside a single marketplace [cite: 155] | [cite_start]Highly mobile, privacy-sensitive, or safety-critical agents at trillion scale [cite: 155] |
[cite_start]***Table 4: AgentFacts vs Google A2A Agent Card*** [cite: 156]

[cite_start]**Take-away**: An A2A card is ideal for a single-marketplace, server-hosted agent with infrequent endpoint changes[cite: 156]. [cite_start]AgentFacts adds four capabilities for target workloads: (1) TTL-scoped endpoint lists for minute-level redeploys, (2) verifiable credentials for code-integrity and capability proofs, (3) a privacy path (PrivateFactsURL) for split-horizon governance, and (4) millisecond revocation via VC-Status lists[cite: 157].

### B. Formal Schema Specification
[cite_start]AgentFacts are represented in JSON-LD and conform to a versioned schema[cite: 160].

| | Description |
| :--- | :--- |
| **ID:** | [cite_start]Unique identifier [cite: 162] |
| **AgentName:** | [cite_start]Human readable name [cite: 162] |
| **Endpoints:** | [cite_start]Can be URL or DID, links to various hosted versions of the service, or even a dynamic routing service [cite: 162] |
| **UsageFormat:** | [cite_start]Input output formats, API spec, protocol support, auth [cite: 162] |
| **Certification:** | [cite_start]Certified via a registry or self-certified via DID [cite: 162] |
| **Capabilities:** | [cite_start]Declared capabilities, performance on those capabilities, with the option for third-party auditing and certification [cite: 162] |
| **Discovery:** | [cite_start]Metadata for search, including vector embeddings [cite: 162] |
| **Security:** | [cite_start]Security requirements, including optional third party security auditing [cite: 162] |
[cite_start]***Table 5: A minimal representation of the AgentFacts schema.*** [cite: 163]

[cite_start]This schema includes both static metadata and dynamic fields, all bound by verifiable cryptographic signatures[cite: 163]. [cite_start]AgentFacts can be viewed as a superset of the Agent Card, allowing any conforming A2A server to embed its existing card as a skills extension[cite: 164].

### C. AgentFacts Hosting Patterns
AgentFacts can be resolved through two primary URL schemes:
* [cite_start]**URL1 (PrimaryFactsURL): Primary Facts Hosting**: Hosted under a standardized path on the agent’s domain, recommended for public-facing services or enterprise-controlled deployments[cite: 165].
* [cite_start]**URL2 (PrivateFactsURL): Privacy-Preserving Hosting (Third-Party Hosted Facts)**: Hosted via a neutral or decentralized provider, enabling metadata retrieval without contacting the agent’s infrastructure[cite: 166].

[cite_start]This dual-hosting model allows agent publishers to choose based on trust requirements, latency preferences, and organizational control[cite: 167].

### D. Resolution Workflow (End-to-End Resolution Workflow with AgentAddr)
The AgentFacts resolution follows a structured, cache-aware workflow:
1.  **Registry Lookup**
    * [cite_start]**Input**: `AgentName` (e.g., `urn:agent:salesforce:starbucks`) [cite: 168]
    * [cite_start]**Output**: A signed `AgentAddr` containing: `{agent_id, facts_url, private_facts_url , adaptive_router_url (optional), ttl, signature}` [cite: 168]
2.  **Metadata Resolution (Optional)**
    * If trust evaluation or capability validation is needed, the client fetches the AgentFacts from either `facts_url` (direct access) or `private_facts_url` (privacy-preserving access). [cite_start]The AgentFacts are verified using digital signatures and credential chains[cite: 169, 170].
3.  **Endpoint Discovery**
    * Clients inspect AgentFacts metadata (if fetched) or AgentAddr (if endpoint is included directly) for:
        * [cite_start]**Static Endpoints**: Stable communication URIs[cite: 171].
        * [cite_start]**Rotating Endpoints**: Short-lived, dynamic URIs[cite: 171].
        * [cite_start]**AdaptiveRouter URL**: A programmable routing microservice[cite: 171].
4.  **Endpoint Resolution & Connection**
    * If `adaptive_router_url` is used, the client sends a request and receives either a redirect to the optimal endpoint or a signed ephemeral token. [cite_start]Connection to the endpoint is authenticated (e.g., via OAuth2 or JWT) and established[cite: 172].

[cite_start]Clients can choose the resolution strategy based on latency, load-balancing needs, privacy preferences, credential strength, and trust policies[cite: 173]. [cite_start]This layered resolution flow separates stable identity resolution from dynamic endpoint management, supporting security, agility, and privacy at a global scale[cite: 174].

### E. TTL and Caching Strategies
[cite_start]The system uses time-to-live (TTL) values to manage caching and resolution frequency across multiple layers: the registry (`AgentAddr`), the agent metadata (`AgentFacts`), and the dynamic routing layer (`AdaptiveRouter`)[cite: 175]. [cite_start]These TTLs are signed, layer-specific, and enforce trust-aware resolution intervals[cite: 176].

1.  [cite_start]**TTL in AgentAddr (Registry Layer)**: The `AgentAddr` object includes a `ttl` field that defines how long the record may be cached by clients, gateways, or edge resolvers before requiring re-validation[cite: 177, 178]. [cite_start]This applies to the entire `AgentAddr` object, including metadata pointers and the optional `adaptive_router_url`[cite: 179].
2.  [cite_start]**TTL in AgentFacts (Metadata Layer)**: Each `AgentFacts` includes its own `ttl` value, governing how long its capabilities, telemetry endpoints, or evaluations may be considered fresh[cite: 182, 183].
3.  [cite_start]**TTL for Routing Metadata (Resolution Layer)**: Endpoint lists and `adaptive_router_url` routing logic may have shorter TTLs than `AgentFacts` metadata[cite: 185]. [cite_start]Common TTLs in this layer could be: Static endpoints: 1-6 hours, Rotating endpoints: 5-15 minutes, and Adaptive routing tokens: 30-60 seconds[cite: 186].

[cite_start]TTLs can be adjusted based on agent criticality, deployment volatility, and trust zone policies[cite: 187].

***

## VI. Adaptive Routing and Multi-Endpoint Management

[cite_start]To meet the demands of globally distributed, privacy-sensitive, and dynamically deployed AI agents, the registry architecture supports adaptive routing via programmable, policy-driven components embedded in the agent resolution process[cite: 188]. [cite_start]These components decouple endpoint selection from static metadata and enable scalable, resilient operation in diverse environments[cite: 189].

### A. Motivation for Adaptive Routing
[cite_start]Traditional agent endpoint resolution, where the agent is tied to a single, static endpoint, is inflexible and fragile[cite: 190]. [cite_start]AI agents today need to serve users from multiple regions with low latency, load balance across backend clusters, and update routing logic in real time without registry writes[cite: 191]. [cite_start]Adaptive routing solves these challenges by introducing runtime resolvers, referred to as **AdaptiveRouters**, that inspect request context and forward traffic to the optimal instance[cite: 192].

### B. Multi-Endpoint Resolution Models
Each `AgentFacts` may contain multiple endpoint references, categorized by their resolution type:
1.  **Static Endpoints**: Explicit URIs that represent stable, always-on service interfaces. [cite_start]Ideal for low-frequency, high-trust agents with low endpoint volatility[cite: 194, 195].
2.  [cite_start]**Rotating Endpoints**: A set of URLs with short TTLs, designed for infrastructure with frequent restarts, redeployments, or geographical rebalancing[cite: 196].
3.  [cite_start]**Adaptive Router URI**: Points to a microservice or gateway that dynamically routes traffic to optimal downstream instances, functioning as a programmable interface for applying routing logic[cite: 197, 198].

### C. AdaptiveRouter Component
[cite_start]To avoid requiring AgentFacts resolution for routing in all cases, the registry can embed the `adaptive_router_url` directly in the `AgentAddr`, enabling immediate routing without further lookups[cite: 199].

This enables the resolution flow: `AgentName → Registry → AgentAddr → AdaptiveRouter → Ephemeral Endpoint`

[cite_start]The **AdaptiveRouter** is an optional but powerful element in the routing stack that accepts requests, inspects context, and forwards the call to the most appropriate agent instance[cite: 201]. Features include:
* [cite_start]**Load Balancing**: Routes traffic to the least-loaded endpoint instance in real time[cite: 202].
* [cite_start]**Geo-Aware Dispatching**: Selects endpoints closest to the request origin[cite: 203].
* [cite_start]**Capability-Specific Routing**: Directs requests to endpoint versions supporting requested skills[cite: 204].
* [cite_start]**Threat Mitigation**: Rotates backend targets or rate-limits sensitive interfaces based on anomaly detection[cite: 204].

[cite_start]The AdaptiveRouter is referenced via the AgentFacts and may issue temporary credentials or endpoint tokens to validate downstream access[cite: 205].

### D. Security and TTL Constraints
[cite_start]All routing logic exposed via the `adaptive_router_url` must declare TTLs, be signed to prevent manipulation, and specify fallback rules[cite: 206].

### E. Implementation Considerations
For real-world deployments, AdaptiveRouter implementations may take the form of:
* [cite_start]**Serverless Functions**: Lightweight routing logic deployed on cloud edge networks[cite: 207].
* [cite_start]**Reverse Proxies (e.g., Envoy, NGINX)**: Using programmable rulesets for routing and policy enforcement[cite: 208].
* [cite_start]**Federated Mesh Gateways**: In multi-organization settings, shared routers may enforce domain-specific policies[cite: 209].

[cite_start]These implementations must support cryptographic verification, auditability, and availability of SLAs to function reliably within the decentralized agent fabric[cite: 210].

***

## VII. Credentialed Trust and Privacy Mechanisms

[cite_start]The architecture emphasizes verifiability, privacy, and modular trust throughout the resolution process[cite: 212]. [cite_start]For agents in sensitive, autonomous, and federated contexts, all metadata must be cryptographically bound and privacy-aware[cite: 213]. The system introduces the separation of:
* [cite_start]**Signed AgentAddr** (returned by the registry)[cite: 214].
* [cite_start]**Credentialed AgentFacts** (resolved optionally for metadata)[cite: 214].
* [cite_start]**Privacy-preserving resolution paths** using third-party infrastructure[cite: 214].

| Artifact | Purpose | Signed By |
| :--- | :--- | :--- |
| **AgentAddr** | [cite_start]Identity, routing pointers, TTL [cite: 217] | [cite_start]Registry resolver [cite: 217] |
| **AgentFacts** | [cite_start]Capabilities, credentials, endpoints [cite: 217] | [cite_start]Credential issuers [cite: 217] |
| **Endpoint Tokens** | [cite_start]Temporary routing/session bindings [cite: 217] | [cite_start]AdaptiveRouter (optional) [cite: 217] |
[cite_start]***Table 6: Artifact within an entry on the NANDA registry architecture, and their purpose*** [cite: 218]

### A. Verifiable Credentials and Signed Metadata
[cite_start]The integrity of agent metadata and its update mechanism depends on W3C Verifiable Credentials (VCs) to assert capabilities, skills, audits, or evaluations and cryptographic signatures to bind these claims to the agent identity[cite: 215]. [cite_start]This ensures agents cannot spoof skills, impersonate trusted services, or hijack reputational pathways[cite: 216].

### B. Credential Governance and Trust Domains
[cite_start]The trustworthiness of an AgentFacts hinges on the validity and recognition of its credential issuers[cite: 218]. [cite_start]The system supports a federated trust governance model where different credential authorities may govern their own "trust zones"[cite: 219]. [cite_start]These zones can define credential schemas, issuance policies, establish revocation mechanisms, and inter-operate using cross-signing, federation agreements, or registry-to-registry bridges[cite: 220]. [cite_start]Clients can configure their own trust preferences, such as accepting only credentials from whitelisted authorities or requiring threshold verification[cite: 221]. [cite_start]This modular design allows the ecosystem to evolve without hardcoded trust anchors[cite: 222].

### C. Privacy-Preserving Resolution via PrivateFactsURL - URL2
[cite_start]To protect accessor privacy, the system supports resolving agent metadata via a third-party or decentralized location specified as `private_facts_url` in the `AgentAddr`[cite: 223]. This model provides:
1.  [cite_start]**Requester anonymity**: The agent domain is never contacted[cite: 224].
2.  [cite_start]**Access decoupling**: Metadata is publicly readable without invoking agent infrastructure[cite: 224].
3.  [cite_start]**Audit independence**: Metadata remains available even if the agent hosting goes offline[cite: 224].

[cite_start]Clients can prefer this route when operating in regulated or high-sensitivity environments[cite: 224].

### D. Revocation and Credential Freshness
[cite_start]Credential freshness is essential for safety in real-time environments[cite: 225]. To support dynamic trust, the system includes:
* [cite_start]**Credential Expiry Fields**: Embedded in all signed objects[cite: 226].
* [cite_start]**Revocation Lists**: Served by credential issuers and queried by clients during interaction[cite: 227].
* [cite_start]**Hash-Linked Chains**: Credential validity can be chained across issuers[cite: 228].

[cite_start]It is advised to recompute TRS on each new session, set short cache expiration policies, and perform multi-source validation for high-risk agents[cite: 229].

### E. Resolution Layer Enforcement
[cite_start]The architecture enforces that `AgentAddr` records are resolved and cached only within their TTL window, and `AgentFacts` metadata is not assumed to be static[cite: 231]. Clients must verify signatures and freshness. [cite_start]Privacy-preserving resolution (via `PrivateFactsURL`) is functionally equivalent to `FactsURL` but adds a layer of access decoupling[cite: 232].

***

## VIII. Deployment Considerations and Use Cases

[cite_start]This section outlines practical implementation strategies and real-world use cases for the proposed registry architecture[cite: 233]. [cite_start]Its lean registry, dynamic resolution, and credentialed metadata design are well-suited for diverse operational environments[cite: 234].

### A. Deployment Models
The registry and AgentFacts system can be deployed under multiple architectural models:
1.  [cite_start]**Enterprise-Controlled Registries**: Enterprises deploy and manage internal registries for their agent namespaces, hosting AgentFacts within their infrastructure[cite: 236, 237].
2.  [cite_start]**Federated Industry Registries**: Vertically integrated industries operate consortium registries, with AgentFacts referencing third-party capabilities from accredited bodies[cite: 238, 239].
3.  [cite_start]**Decentralized Public Registries**: Publicly accessible registries allow open registration, with credential validation and revocation enforced via smart contracts or decentralized protocols[cite: 240, 241]. [cite_start]AgentFacts are hosted on decentralized file systems[cite: 242].
4.  [cite_start]**Hybrid Topologies**: Organizations may register with both private and public registries for broader discoverability[cite: 243].

[cite_start]These models enable the system to adapt to both regulated, closed-loop deployments and open, innovation-driven ecosystems[cite: 245].

### B. Performance and Scaling Considerations
While the registry is lean, the supporting infrastructure must be designed for internet-scale agent traffic. Key strategies include:
* [cite_start]**Edge Caching**: TTL-enforced caching of AgentFacts and routing metadata at CDN edges to reduce resolution latency[cite: 247].
* [cite_start]**Sharded Credential Verification**: Parallel trust scoring engines operating at domain-specific points of presence[cite: 248].
* [cite_start]**Load-Adaptive Routers**: Stateless microservices that scale horizontally and resolve endpoints dynamically[cite: 249].
* [cite_start]**Audit and Monitoring Hooks**: Integration with OpenTelemetry and observability stacks for SLA monitoring and behavior-based trust score computation[cite: 250].

***

## IX. Conclusion

[cite_start]The rise of autonomous AI agents places unprecedented demands on digital infrastructure[cite: 251]. [cite_start]Traditional internet protocols are insufficient for the fluid, high-volume interactions required by agent-scale autonomy[cite: 252]. [cite_start]This paper introduces a lean, modular registry architecture as a foundational layer for the decentralized Internet of AI Agents[cite: 253]. Grounded in the NANDA Protocol, the design advances the state of the art across multiple dimensions:
* [cite_start]**Minimal Core Registry**: Decouples static identifiers from dynamic metadata, reducing update frequency, exposure, and attack surfaces[cite: 254].
* [cite_start]**Credentialed AgentFacts**: Enables rapid, verifiable updates to agent capabilities, routing logic, and telemetry[cite: 255].
* [cite_start]**Timed Resolution & Adaptive Routing**: Supports latency optimization, DDoS mitigation, and geo-aware deployments[cite: 256].
* [cite_start]**Privacy-Preserving Discovery**: Protects accessor identities and aligns with zero-trust security models[cite: 257].
* [cite_start]**Federated Trust & Governance**: Enables scalable oversight across jurisdictions, organizations, and policy domains[cite: 258].

[cite_start]Together, these components establish the groundwork for an open, secure, and extensible agent ecosystem capable of supporting trillions of agents operating cooperatively across digital and physical environments[cite: 259].

***

## X. Future Work

Several avenues for future research and development build upon this foundation:
1.  [cite_start]**Trust Layer Extensions**: Future work will explore decentralized trust score computation, integration with behavioral monitoring systems, and real-time reputation feeds[cite: 260].
2.  [cite_start]**Governance Frameworks and Revocation Models**: We will formalize revocation protocols, dispute resolution mechanisms, and multi-jurisdictional governance workflows to enforce trust at scale[cite: 261].
3.  [cite_start]**Open Standards and Interoperability**: Standardizing the AgentFacts schema with IETF/W3C bodies and enabling interoperability with existing frameworks will ensure ecosystem adoption[cite: 263].
4.  [cite_start]**Privacy Enhancements with Zero-Knowledge Proofs (ZKPs)**: To further privacy guarantees, ZKP-based credential assertions can be introduced, allowing agents to prove capabilities or compliance without revealing full certificate chains[cite: 264].

***

## XI. Open Questions and Discussion

[cite_start]While this paper proposes a comprehensive registry and metadata architecture for AI agents, several key areas remain open for further exploration, community alignment, and empirical validation[cite: 265]. [cite_start]These questions span technical design, trust models, privacy trade-offs, economic incentives, and future governance structures[cite: 266]. [cite_start]Their resolution will shape the long-term viability and adaptability of decentralized agent infrastructure at a global scale[cite: 267].

### A. Hosting and Accessibility of Agent Metadata
A central question is the hosting model for AgentFacts. [cite_start]The current design permits multiple options: (i) self-hosting by the agent, (ii) hosting through secure third-party services, or (iii) provisioning by a neutral infrastructure provider like NANDA[cite: 268, 269]. [cite_start]Each approach offers distinct trade-offs in privacy, trust, and update latency[cite: 270]. [cite_start]A flexible model is desirable, but the trade-offs between decentralization, performance, and control remain unresolved[cite: 273].

### B. Registry Visibility and Discoverability
[cite_start]Unlike DNS, the agent registry may be intentionally opaque to prevent abuse or sensitive agent enumeration[cite: 274]. [cite_start]At the same time, open discovery is crucial for enabling federation, crawling, and reputation-based search[cite: 275].
[cite_start]**Open issue**: Should the registry support machine-readable scraping or indexing, and if so, under what access controls or throttling policies? [cite: 276]

### C. Terminology Alignment with Prior Art
[cite_start]The term "AgentFacts" may overlap or cause confusion with prior frameworks like A2A[cite: 278]. [cite_start]While our schema significantly extends prior work, we may consider renaming it and clearly describing its relationship to A2A[cite: 279]. [cite_start]Additionally, terms like "PrimaryFactsURL" and "PrivateFactsURL" may be refined to more functional descriptors[cite: 280].
[cite_start]**Open issue**: What terminology best communicates extensibility while acknowledging lineage and compatibility with existing systems? [cite: 281]

### D. Resolution Flow and Return Semantics
[cite_start]Current resolution logic implies that accessing an endpoint requires first resolving and validating the AgentFacts[cite: 282]. [cite_start]However, it is unclear if the client must always parse the facts or if a signed `AgentAddr` or `AgentLoc` object would suffice[cite: 283].
[cite_start]**Open issue**: Should the resolver return a lightweight signed object, distinct from the full facts, optimized for runtime communication? [cite: 284]

### E. TTL Strategy and Update Constraints
[cite_start]While the paper introduces TTL-based caching, important design questions remain[cite: 286].
[cite_start]**Open issue**: How do we strike a balance between agility and stability in TTL assignments, particularly for high-frequency or regulated agents? [cite: 289]

### F. Credential Governance and Delegation
[cite_start]Another unresolved dimension is the delegation model for updates and credential issuance[cite: 290].
[cite_start]**Open issue**: What are the minimal viable trust constraints to allow automated updates while preventing spoofing, abuse, or trust inflation? [cite: 293]

### G. Adaptive Routing Reliability
[cite_start]The adaptive routing layer introduces flexibility and performance but also poses new failure modes[cite: 294].
[cite_start]**Open issue**: What guarantees must the resolution and routing layer provide to support mission-critical or regulated agent deployments? [cite: 297]

### H. Economic Incentives and Monetization Models
[cite_start]The sustainability of agent registries and metadata services may depend on viable economic incentives, yet the architecture must avoid rent-seeking behavior or exclusionary gatekeeping[cite: 298, 299].
**Open issue**: Should AgentFacts hosting or TTL adjustments be linked to micropayments, staking mechanisms, or usage tiers? [cite_start]How can such models preserve openness and composability? [cite: 300, 301]

***

## References
A full list of references can be found in the original document.

***

## Appendix: Full AgentFacts Schema
[cite_start]The full AgentFacts schema, including fields new to AgentFacts and those present in Google AgentCard, is available in the original document's appendix[cite: 320, 321]. It provides a detailed look at the structure and fields of the schema.
