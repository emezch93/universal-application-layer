Universal Application Layer Specification

1. Overview

Universal Application Layer, or UAL, is an experimental infrastructure layer for exposing application capabilities through a common capability interface.

UAL separates the application from the implementation and communication details required to access a capability.

The application requests a capability.

UAL is responsible for establishing the infrastructure path required to execute that capability.

The initial implementation is intentionally small and experimental.

2. Core Model

The fundamental UAL model is:

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

Each component has a distinct responsibility.

The architecture must remain generic enough that the first capability does not define the architecture of the entire system.

3. Application

The application is the consumer of UAL capabilities.

The application should not need to know the internal implementation of a capability.

For example, an application should conceptually be able to request:

UAL.invoke("ai.generate", {
    prompt: "Hello"
})

The application should not need to know:

• Which AI provider is being used

• Which provider endpoint is being called

• Where the provider credential is stored

• How the provider request is authenticated

• How the capability is routed internally

The application only needs to know that the capability exists and what input it accepts.

4. UAL Client

The UAL Client is the application facing interface to UAL.

Its primary responsibility is to provide a consistent mechanism for capability invocation.

Conceptually:

UAL.invoke(capability, payload)

The client may also be responsible for:

• Establishing application identity

• Obtaining or maintaining a UAL session identity

• Attaching required identity information to requests

• Sending capability requests

• Receiving responses

• Supporting streaming where required

The client must not contain provider credentials.

The client must not contain provider specific integration logic.

The client should remain capability agnostic.

5. Identity

UAL requires an identity boundary between the application and the infrastructure.

For the initial proof of concept, identity may be deliberately minimal.

The identity mechanism should establish a relationship between:

Application identity
+
Application origin
+
UAL session identity

The initial identity system does not need to become a complete authentication platform.

Its purpose is to establish a trusted identity context that the UAL gateway can verify before processing a capability request.

6. Gateway

The UAL Gateway is the central execution boundary.

Its responsibilities include:

• Receiving capability requests

• Verifying UAL identity

• Validating application authorization

• Applying rate limits

• Resolving the requested capability

• Dispatching the request to the appropriate capability handler

• Returning or streaming the capability result

• Normalizing infrastructure errors

The gateway must remain capability agnostic.

The gateway must not contain AI provider specific logic.

For example, the gateway should not contain logic such as:

if capability == "ai.generate"

followed by provider specific implementation.

Instead, the gateway should resolve the capability through the capability registry.

7. Capability

A capability represents an operation or service that an application can request from UAL.

Examples may eventually include:

ai.generate
message.send
storage.write
search.query
notification.send

These are examples only.

The initial implementation should use a single capability:

ai.generate

The capability name should identify the capability rather than the underlying provider.

For example:

ai.generate

is preferable to exposing:

openai.chat

as the fundamental UAL capability identity.

This allows the implementation behind a capability to change without necessarily changing the application interface.

8. Capability Registry

The capability registry maps capability identifiers to their handlers.

Conceptually:

Capability
    ↓
Handler

Example:

ai.generate
    ↓
AI capability handler

The registry is a critical architectural component.

It must allow new capabilities to be introduced without redesigning the fundamental UAL gateway.

A future capability could therefore be introduced as:

message.send
    ↓
Messaging capability handler

without changing the fundamental request model.

9. Capability Handler

A capability handler contains the implementation of a specific capability.

The handler is responsible for:

• Validating capability specific input

• Executing capability specific logic

• Communicating with the underlying provider when required

• Handling provider specific responses

• Returning a normalized capability result

The handler may access sensitive provider credentials.

Those credentials must remain inside trusted UAL infrastructure.

The application must never receive them.

10. Capability Provider

A provider is the underlying system that implements or supplies a capability.

For the first experiment, the provider will be an AI provider.

The provider is an implementation detail beneath the capability.

The architecture should allow the capability handler to change providers without requiring the application to change its fundamental capability invocation model.

Conceptually:

Application
    ↓
ai.generate
    ↓
UAL AI Handler
    ↓
Provider

The provider should therefore not define the public UAL abstraction.

11. Capability Invocation

A capability request consists conceptually of:

Identity
Capability
Payload

Example:

{
    "capability": "ai.generate",
    "payload": {
        "prompt": "Hello"
    }
}

Identity information may be transported separately or as part of the authenticated request context.

The exact wire format is an implementation concern and may evolve during the experiment.

The important requirement is that the request model remains generic.

It must not be designed specifically around AI.

12. Request Lifecycle

A standard UAL capability request follows this conceptual sequence:

1. Application initializes UAL Client

2. UAL Client establishes or obtains UAL identity

3. Application invokes a capability

4. UAL Client sends the capability request

5. UAL Gateway verifies identity

6. UAL Gateway checks capability authorization

7. UAL Gateway applies applicable limits

8. UAL Gateway resolves the capability

9. Capability Registry returns the appropriate handler

10. Capability Handler executes the capability

11. Handler communicates with the provider when required

12. Handler produces the capability result

13. UAL Gateway returns the result to the application

The application should not be responsible for implementing the provider communication path.

13. Authorization

UAL authorization operates at the capability level.

Conceptually:

Application A
    ↓
Allowed:
ai.generate

An application that is not authorized to use a capability should not be able to invoke it successfully.

The authorization mechanism must remain separate from capability implementation.

The AI handler should not determine whether an application is authorized to use AI.

The UAL infrastructure should determine that before dispatching the request.

14. Rate Limiting

UAL may enforce rate limits at the infrastructure boundary.

The initial implementation should support a simple application level rate limit.

For example:

Application
    ↓
Capability request
    ↓
UAL rate limit
    ↓
Capability handler

Rate limiting is an infrastructure responsibility because it applies independently of the internal implementation of a capability.

Per user quotas are outside the initial proof of concept unless required by the experiment.

15. Security Boundary

The browser or frontend application is an untrusted environment.

Provider credentials must therefore never be placed in the frontend.

The intended boundary is:

Untrusted

Frontend
UAL Client

----------------------------

Trusted

UAL Identity
UAL Gateway
Capability Handlers
Provider Credentials

The UAL gateway must verify identity and authorization before invoking a protected capability.

Capability handlers must treat all application input as untrusted input.

16. Origin Binding

The initial identity mechanism should associate the application identity with an expected origin.

Conceptually:

Application identity
        +
Origin
        ↓
UAL session

This is intended to reduce the usefulness of a copied public application identifier when used from an unauthorized origin.

Origin binding is not assumed to solve every browser security problem.

It is an initial security boundary for the proof of concept.

17. Credential Management

Provider credentials belong exclusively to trusted UAL infrastructure.

The expected flow is:

Frontend
    ↓
UAL
    ↓
Capability Handler
    ↓
Provider credential
    ↓
Provider

The provider credential must not travel to the frontend.

The application should not need to store provider credentials.

The developer should not need to expose provider credentials through client side code.

18. Transport

UAL does not initially attempt to invent a new network transport protocol.

The underlying transport may use existing mechanisms such as HTTPS.

The experimental question is not:

«Can UAL replace the internet transport layer?»

The experimental question is:

«Can UAL provide a reusable capability infrastructure abstraction above the underlying communication mechanisms?»

Transport implementation details must therefore remain separate from the capability abstraction.

19. Error Model

UAL should provide a normalized infrastructure error model where practical.

Errors should distinguish between categories such as:

Identity failure
Authorization failure
Rate limit failure
Invalid capability
Invalid payload
Capability execution failure
Provider failure
Infrastructure failure

Provider specific error details should not unnecessarily become part of the application's dependency on a provider.

20. Streaming

Some capabilities may require streaming responses.

The UAL architecture should therefore avoid assuming that every capability produces one immediate response.

The transport layer may support:

Request
    ↓
Streaming capability result

The first proof of concept may implement streaming only if required by the selected AI provider.

Streaming must remain a transport and capability execution concern rather than changing the fundamental capability model.

21. Genericity Requirement

UAL must remain generic at its core.

The following should not be fundamentally hard coded into the UAL gateway:

AI provider names
AI models
AI prompts
AI specific request fields
AI specific response fields

These belong inside the AI capability implementation.

The core should understand:

Identity
Capability
Authorization
Payload
Dispatch
Result

This distinction is essential to the experiment.

22. Extensibility Test

The architecture must eventually be tested by introducing a capability unrelated to AI.

For example:

ai.generate

followed by:

message.send

The second capability should be implementable without changing the fundamental architecture of:

UAL Client
UAL Identity
UAL Gateway
Capability Registry

If adding the second capability requires significant redesign of these components, the generic abstraction should be reconsidered.

23. Initial Implementation

The first implementation should contain only the components required to test the hypothesis.

Target structure:

UAL Client
UAL Identity
UAL Gateway
Capability Registry
Test Capability
AI Capability
Example Application

The first development stage may initially use:

test.echo

as a trivial capability to verify the generic invocation and dispatch mechanism before introducing AI.

The test capability should not become part of the final conceptual product.

It is an experimental instrument.

24. Implementation Constraints

The first proof of concept should not include:

• Payments

• Billing

• Administrative dashboards

• Complex databases

• Multi provider orchestration

• Automatic application analysis

• Automatic source code understanding

• A new programming language

• Global infrastructure deployment

• Enterprise account management

• A large capability marketplace

These may become relevant in later phases only if the core hypothesis survives experimentation.

25. Architectural Principle

The central architectural principle is:

«The application requests capabilities. UAL manages the infrastructure required to safely and consistently reach those capabilities.»

The capability implementation should remain replaceable.

The provider should remain replaceable.

The transport should remain replaceable.

The UAL capability model should remain stable.

26. Experimental Status

This specification describes the architecture being tested.

It does not establish that UAL is superior to existing approaches.

The implementation must be compared against conventional application architectures.

The comparison should consider:

• Developer complexity

• Amount of application specific backend code

• Security responsibilities

• Credential management

• Capability authorization

• Extensibility

• Operational complexity

• Performance

• Failure handling

• Conceptual clarity

• Whether the abstraction is genuinely different from existing API and gateway patterns

The architecture should be revised if experimentation demonstrates that its assumptions are incorrect.

27. Current Target

The immediate target is a working proof of concept in which:

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

works without the example application implementing its own dedicated backend endpoint for the AI capability.

The implementation should first prove the generic capability mechanism and then prove that AI can operate as one capability within that mechanism.

The project should not proceed to larger infrastructure features until this architecture has been tested and evaluated.
