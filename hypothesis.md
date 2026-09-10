UAL Hypothesis

1. Purpose

Universal Application Layer, or UAL, is an infrastructure experiment.

The purpose of this project is to investigate whether application capabilities can become native infrastructure primitives, reducing the need for developers to repeatedly design, implement, secure, authenticate, and maintain the communication layer between an application and the capabilities it uses.

UAL is not being treated as a proven technology.

This document defines the hypothesis that the project must test and the conditions under which the hypothesis should be considered supported, weakened, or rejected.

2. The Problem

Modern applications frequently need capabilities that exist outside the immediate frontend application.

Examples include:

• Artificial intelligence

• Messaging

• Payments

• Storage

• Search

• Notifications

• Data processing

• Authentication

• External business systems

A developer normally creates or configures a communication path between the application and each capability.

A simplified flow is:

Application
    ↓
Application backend
    ↓
API
    ↓
Authentication
    ↓
Provider
    ↓
Capability

The exact architecture varies, but developers repeatedly deal with similar concerns:

• Communication

• Authentication

• Authorization

• Credential management

• Request validation

• Routing

• Rate limiting

• Error handling

• Provider specific integration

• Security boundaries

The hypothesis is that much of this repeated infrastructure responsibility may not need to belong to each individual application.

3. Core Hypothesis

The core hypothesis of UAL is:

«Application capabilities can be exposed as infrastructure primitives through a universal capability layer, allowing applications to request capabilities without developers repeatedly building the traditional communication and integration layer for each capability.»

In this model, the developer thinks primarily in terms of capabilities rather than provider endpoints.

Conceptually:

UAL.invoke("ai.generate", payload)

rather than requiring the application developer to construct and maintain a dedicated backend endpoint for the capability.

The underlying communication mechanism does not have to disappear.

HTTPS, streaming, RPC, queues, or other transport mechanisms may still exist underneath.

The hypothesis concerns where the responsibility for communication infrastructure lives, not whether communication itself ceases to exist.

4. The Proposed Abstraction

UAL proposes an infrastructure boundary between an application and external capabilities.

Conceptually:

Application
    ↓
UAL Client
    ↓
UAL Identity
    ↓
UAL Gateway
    ↓
Capability Registry
    ↓
Capability Handler
    ↓
Capability Provider

The application requests a capability.

UAL determines whether the application is authorized to use that capability, establishes the appropriate communication path, invokes the capability handler, and returns the result.

The provider should remain an implementation detail of the capability.

For example:

Application
    ↓
UAL
    ↓
ai.generate
    ↓
AI provider

The application should not need to understand the provider's internal endpoint, credentials, or communication implementation.

5. What UAL Is Trying to Remove

UAL is specifically investigating whether developers can avoid repeatedly building application specific infrastructure for:

• Provider credential handling

• Dedicated capability endpoints

• Repeated authentication plumbing

• Capability authorization

• Request routing

• Provider integration boundaries

• Repeated transport logic

• Capability specific gateway wiring

The goal is not simply to reduce the number of lines of code.

The goal is to determine whether these responsibilities can become part of a reusable infrastructure primitive.

6. What UAL Is Not

UAL should not become an undefined name for existing technologies.

The project must distinguish itself from:

• A traditional REST API

• A GraphQL API

• An RPC system

• An SDK

• An API gateway

• A serverless proxy

• A backend as a service

• An AI gateway

• A request wrapper

• A frontend library that hides fetch

If the implementation merely changes:

fetch("/api/ai")

into:

ual.invoke("ai.generate")

while the underlying architecture remains an ordinary application API, then the hypothesis has not been proven.

A better developer experience alone is not sufficient evidence of a new infrastructure primitive.

7. The First Capability

The first experimental capability will be AI.

The reason is not that UAL is intended to be an AI platform.

AI is being used as the first capability because it provides a concrete and understandable integration problem through which the architecture can be tested.

The first implementation should therefore expose something conceptually similar to:

ai.generate

The AI capability should be implemented as a handler operating within the UAL capability architecture.

UAL itself should not become AI specific.

8. The Critical Generalization Test

The strongest test of the hypothesis is whether UAL remains structurally valid when a completely unrelated capability is introduced.

For example:

Capability 1
ai.generate

followed later by:

Capability 2
message.send

The second capability must not require a redesign of the fundamental UAL architecture.

Ideally, adding another capability should primarily require:

New capability definition
+
New capability handler
+
Capability configuration

rather than changes to the fundamental identity, authorization, communication, or gateway model.

If every new capability requires custom gateway architecture, then UAL is not sufficiently universal.

9. Success Criteria

The hypothesis becomes stronger if the following conditions are demonstrated.

9.1 Generic invocation

The client can invoke capabilities through a common mechanism.

Conceptually:

UAL.invoke(capability, payload)

The invocation mechanism should not be fundamentally tied to AI.

9.2 Capability independent gateway

The gateway can process capability requests without containing provider specific business logic.

9.3 Capability based authorization

UAL can determine whether an application is permitted to use a particular capability.

The authorization model should operate at the capability level rather than being hard coded around one provider.

9.4 Credential isolation

Sensitive provider credentials remain within trusted infrastructure.

The application does not receive the provider credential.

9.5 Capability extensibility

A second unrelated capability can be introduced without redesigning the core UAL architecture.

9.6 Reduced application infrastructure

A developer can consume the capability without creating the traditional application specific backend communication layer normally required for the integration.

9.7 Meaningful architectural difference

The resulting system must provide an architectural abstraction that is meaningfully different from simply placing an API gateway or SDK in front of an existing provider.

10. Failure Criteria

The project should be considered unsuccessful or require fundamental redesign if any of the following becomes true.

10.1 UAL becomes an SDK

If the primary value is only hiding HTTP requests or simplifying provider calls, then UAL is functioning as an SDK.

10.2 UAL becomes an API gateway

If every capability is simply another backend endpoint routed through the same gateway, then the project may simply be an API gateway with different terminology.

10.3 UAL remains AI specific

If the architecture works naturally for AI but unrelated capabilities require fundamental changes to the core, the universal abstraction is weak.

10.4 Developers still build the same backend layer

If developers still need to create essentially the same backend endpoints, authentication systems, credential handling, and integration logic, then UAL has not removed the targeted infrastructure responsibility.

10.5 Complexity exceeds the problem

If UAL introduces more infrastructure, configuration, operational complexity, and security burden than the traditional approach it replaces, the abstraction may not be justified.

10.6 No meaningful advantage

If an existing technology already provides essentially the same abstraction, security model, developer experience, and extensibility with comparable complexity, UAL may not represent a sufficiently distinct primitive.

11. The AI Proof of Concept

The first proof of concept should be intentionally small.

The target comparison is:

Traditional approach

Frontend
    ↓
Developer backend
    ↓
AI provider

against:

UAL approach

Frontend
    ↓
UAL
    ↓
AI capability
    ↓
AI provider

The experiment should determine whether the second architecture actually removes meaningful application infrastructure responsibilities.

The goal is not to make AI calls work.

The goal is to determine whether UAL creates a useful capability boundary around that AI capability.

12. Known Limitations

The first proof of concept is not expected to solve every infrastructure problem.

The initial implementation may deliberately omit:

• Full user authentication

• Billing

• Advanced quota management

• Multi provider routing

• Global distributed infrastructure

• Administrative dashboards

• Complex policy engines

• Persistent application databases

• Automatic application discovery

• Automatic understanding of arbitrary source code

These omissions are acceptable if they do not prevent testing the core hypothesis.

The experiment should avoid building infrastructure that does not contribute directly to proving or disproving the hypothesis.

13. The Fundamental Question

The project ultimately asks one question:

«Can the communication and integration responsibilities surrounding application capabilities become a reusable infrastructure primitive rather than something developers repeatedly construct inside individual applications?»

If the answer is no, UAL should not be forced into existence.

If the answer is yes, the proof of concept should reveal the smallest architectural primitive from which a larger system could be developed.

14. Research Position

UAL begins with a hypothesis, not a conclusion.

The project must actively attempt to break its own assumptions.

A successful proof of concept is not one that merely demonstrates that UAL works.

A successful proof of concept is one that demonstrates a meaningful architectural property that existing approaches do not already provide adequately.

The project therefore values falsification over confirmation.

If UAL is simply an API gateway, SDK, proxy, or renamed existing pattern, that result should be recorded honestly.

If the experiment demonstrates a genuinely reusable capability infrastructure model, the next phase should be derived from that evidence rather than from the original idea alone.

15. Current Experimental Scope

The initial scope is intentionally limited to:

Generic UAL capability mechanism
+
Identity boundary
+
Capability authorization
+
Capability registry
+
Gateway
+
One AI capability
+
One example frontend application

Nothing beyond this scope is required to test the first hypothesis.

The immediate objective is therefore:

Build the smallest system capable of proving that an application can consume a capability through UAL while the UAL infrastructure assumes responsibility for the communication and integration concerns that would otherwise be implemented inside the application backend.

The result may validate the hypothesis, weaken it, or disprove it.

All three outcomes are useful.
