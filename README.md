Universal Application Layer

Universal Application Layer, or UAL, is an experimental infrastructure project investigating whether application capabilities can become native infrastructure primitives.

The project explores a simple question:

«What if applications could request capabilities directly, while the infrastructure responsible for identity, authorization, communication, routing, and provider integration existed outside the application itself?»

UAL is currently an architectural experiment, not a finished platform.

The Problem

Modern applications frequently need capabilities provided by external systems.

Examples include:

• Artificial intelligence

• Messaging

• Payments

• Storage

• Search

• Notifications

• Data processing

To use these capabilities, developers commonly create application specific backend logic that connects the application to the external provider.

A simplified architecture looks like:

Frontend Application
        ↓
Application Backend
        ↓
API
        ↓
External Provider
        ↓
Capability

This approach works, but many applications repeatedly implement similar infrastructure concerns:

• Authentication

• Authorization

• Credential management

• Request validation

• Routing

• Rate limiting

• Provider integration

• Error handling

• Communication

UAL investigates whether these responsibilities can become reusable infrastructure instead of being repeatedly constructed inside individual applications.

The UAL Idea

UAL introduces a capability layer between the application and the systems that provide those capabilities.

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

The application requests a capability rather than directly managing the provider communication path.

Conceptually:

UAL.invoke("ai.generate", {
    prompt: "Hello"
})

The application does not need to know which provider implements the capability, where the provider endpoint exists, or where the provider credential is stored.

UAL handles those infrastructure concerns within the trusted infrastructure boundary.

What UAL Is Not

UAL is not intended to simply become:

• An SDK

• A REST API

• A GraphQL API

• An RPC framework

• An API gateway

• A serverless proxy

• An AI gateway

• A request wrapper

If UAL simply hides an ordinary API request behind a different interface, the experiment has not succeeded.

The project must demonstrate a meaningful capability abstraction rather than a renamed version of an existing integration pattern.

The Core Principle

The central idea is:

«Applications request capabilities. UAL manages the infrastructure required to safely and consistently reach those capabilities.»

This creates a separation between:

Application
        ↓
Capability
        ↓
Implementation
        ↓
Provider

The application depends on the capability rather than directly depending on the provider implementation.

First Experiment

The first capability being tested is artificial intelligence.

This does not mean UAL is an AI platform.

AI is simply the first concrete capability through which the architecture will be tested.

The initial target is:

Frontend Application
        ↓
UAL Client
        ↓
UAL Identity
        ↓
UAL Gateway
        ↓
Capability Registry
        ↓
AI Capability Handler
        ↓
AI Provider

The first implementation may also use a simple test capability such as:

test.echo

before introducing AI.

This allows the generic capability mechanism to be tested independently of provider specific complexity.

The Critical Test

The most important test is not whether UAL can make an AI request.

Existing technologies can already do that.

The important test is whether the underlying UAL architecture remains generic when additional capabilities are introduced.

For example:

ai.generate

should be able to exist alongside a future capability such as:

message.send

without requiring a redesign of the fundamental UAL architecture.

Ideally, adding another capability should require a new capability handler and configuration rather than a new gateway architecture.

If this cannot be achieved, the universal capability abstraction becomes questionable.

Architecture

The current conceptual architecture consists of five major infrastructure components.

UAL Client

The application facing interface.

Responsible for capability invocation, identity interaction, request transmission, and response handling.

It must not contain provider credentials.

UAL Identity

Establishes the identity context required for UAL requests.

The initial implementation will use a deliberately minimal identity mechanism.

UAL Gateway

The central infrastructure boundary.

Responsible for identity verification, authorization, rate limiting, capability resolution, dispatch, and response handling.

The gateway must remain capability agnostic.

Capability Registry

Maps capability identifiers to capability handlers.

Conceptually:

ai.generate → AI Handler

The registry exists to keep the gateway independent from individual capability implementations.

Capability Handler

Implements a specific capability.

For AI, the handler contains the provider specific integration and provider credential access.

The handler operates inside trusted UAL infrastructure.

Security Boundary

The frontend is treated as an untrusted environment.

Provider credentials must remain inside trusted UAL infrastructure.

The intended boundary is:

UNTRUSTED

Frontend
UAL Client

----------------------------

TRUSTED

UAL Identity
UAL Gateway
Capability Handlers
Provider Credentials

UAL is responsible for establishing the security boundary between the application and the underlying capability provider.

Current Scope

The first proof of concept is intentionally small.

It focuses on:

• Generic capability invocation

• Identity

• Capability authorization

• Capability registry

• Gateway

• One test capability

• One AI capability

• One example frontend application

The project does not currently attempt to build:

• Payments

• Billing

• Administrative dashboards

• A capability marketplace

• Complex databases

• Multi provider orchestration

• Automatic source code understanding

• Automatic application discovery

• A new programming language

• Global distributed infrastructure

Those concerns can only be considered after the core architectural hypothesis has been tested.

Project Status

Status: Experimental

UAL has not yet been proven to represent a new infrastructure primitive.

The current phase is implementation and experimentation.

The project will deliberately compare UAL against conventional approaches including:

Frontend
    ↓
Backend API
    ↓
Provider

and determine whether UAL provides a meaningful architectural advantage.

Possible outcomes include:

1. UAL demonstrates a useful new capability abstraction.

2. UAL provides a useful architectural improvement but is not a fundamentally new primitive.

3. UAL is essentially an existing pattern such as an API gateway, SDK, or proxy and should be abandoned or redesigned.

All three outcomes are considered valid results.

Design Philosophy

UAL is being developed from first principles.

The project should not assume that the original idea is correct.

Every major architectural decision should be evaluated against the core hypothesis.

The implementation should remain as small as possible while still providing enough evidence to make a meaningful judgment.

Complexity should be introduced only when experimentation demonstrates that it is necessary.

Repository Documents

The repository currently contains three foundational documents:

"hypothesis.md"

Defines the problem, hypothesis, success criteria, failure criteria, and experimental scope.

"ual_spec.md"

Defines the proposed UAL architecture, components, responsibilities, security boundaries, and capability model.

"README.md"

Provides the public overview of the project.

Current Objective

The immediate objective is to build the smallest working UAL system capable of testing the core hypothesis.

The target flow is:

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
Provider

The first capability will be AI.

The goal is not merely to make an AI request work.

The goal is to determine whether UAL can establish a reusable infrastructure boundary around application capabilities while removing meaningful communication and integration responsibilities from individual applications.

Long Term Possibility

If the core primitive survives experimentation, UAL could eventually evolve into broader infrastructure for application capabilities.

Potential future areas may include:

• Multiple capability types

• Multiple providers

• More sophisticated identity

• Fine grained authorization

• Usage policies

• Capability discovery

• Developer tooling

• Distributed execution

• New application programming models

These are future possibilities, not current commitments.

The current project exists to answer a much smaller question first:

Can the capability abstraction itself work?
