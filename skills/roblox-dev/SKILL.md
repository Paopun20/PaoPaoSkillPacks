---
name: roblox-dev
description: >
  Use this skill when working with Roblox Studio, Luau, Roblox Engine APIs, or Roblox Open Cloud HTTP APIs.
  It should be used for questions about engine classes, properties, methods, events, datatypes, enums, globals,
  libraries, or cloud endpoints such as assets, datastores, developer products, messaging, secrets, and universes.
  Prefer the local reference docs in this skill package over vague memory or generic web advice.
---

# Roblox Development Skill

This skill helps with Roblox-specific development work by grounding answers in the local reference materials bundled with this skill.

## What this skill covers

- Roblox Engine API documentation for classes, properties, methods, events, datatypes, enums, globals, and libraries
- Roblox Open Cloud HTTP API documentation for endpoints and schemas
- Luau-oriented implementation guidance for Studio, services, APIs, and data workflows
- Accurate, source-based answers instead of guessing from memory

## Local reference layout

This skill package includes two reference sets:

- Engine reference: reference/engine/
  - Contains Roblox Engine API docs as YAML files.
  - Use these for class and API lookups such as services, instances, properties, methods, events, enums, datatypes, globals, and libraries.
- Cloud reference: reference/cloud/
  - Contains Roblox Open Cloud HTTP API docs as JSON files.
  - Use these for endpoint-based work such as assets, datastores, developer products, messaging, secrets, universes, and related OpenAPI definitions.

## When to apply this skill

Use this skill when the user asks about:

- Roblox engine classes or APIs
- Luau scripting patterns for Roblox services or instances
- Specific instance members, events, methods, or properties
- Roblox Open Cloud endpoints and request/response shapes
- How to call Cloud APIs from server-side or external code
- Anything that should be answered from the official Roblox reference docs rather than general programming advice

## Working approach

1. Identify the domain first.
   - If the request is about Roblox runtime behavior, instance APIs, services, or Luau semantics, treat it as an engine/API question.
   - If the request is about HTTP endpoints, authentication, request payloads, response bodies, or Open Cloud routes, treat it as a cloud/API question.

2. Use the correct local reference source.
   - For engine topics, search the YAML-backed docs under reference/engine/.
   - For cloud topics, search the JSON/OpenAPI docs under reference/cloud/.

3. Answer from the local docs whenever possible.
   - Prefer documented names, parameter names, and behavior over assumptions.
   - If the docs are incomplete or ambiguous, say so clearly and avoid inventing details.

4. Translate the docs into practical guidance.
   - Give concise explanations, code examples, and implementation notes that fit the user’s request.
   - Keep examples aligned with Roblox conventions and the requested language or environment.

5. Verify the result before finishing.
   - Check that the API name, endpoint path, method, and relevant schema all match the local reference.
   - If the user is asking for implementation, make sure the example is consistent with Roblox patterns and the available docs.

## Guidance for engine questions

When the user asks about Roblox engine APIs:

- Look up the relevant class, service, or datatype in reference/engine/classes/ or the matching index files.
- Prefer documented member names such as properties, methods, events, and callbacks.
- If the request involves a specific instance type, mention the class and describe how it is used in Luau.
- If relevant, include a short example showing how the feature would be used in a Roblox script.

## Guidance for cloud questions

When the user asks about Roblox Open Cloud APIs:

- Find the relevant endpoint in reference/cloud/ and identify the HTTP method, path, and request/response structure.
- Use the OpenAPI document in reference/cloud/openapi.json when the request is broad or cross-endpoint.
- Call out authentication requirements, required headers, and body schema when the docs provide them.
- If the user wants implementation help, provide a practical example using the documented endpoint and expected JSON shape.

## Quality bar

Responses from this skill should be:

- Accurate to the local Roblox reference docs
- Specific about whether the topic is engine-related or cloud-related
- Practical and implementation-oriented when the user wants code or setup help
- Clear about uncertainty instead of inventing unsupported details

## Example prompts

- "Find the Roblox API docs for Players and explain how PlayerAdded works."
- "Show me the Roblox Open Cloud endpoint for creating or updating assets."
- "What does the Roblox datastores API require for a request body?"
- "Help me implement this feature in Luau using the documented Roblox Engine API."

## Related follow-up work

If the user wants to expand this skill further, the next useful additions would be:

- a dedicated prompt for Luau code generation from the engine reference
- a dedicated prompt for Open Cloud HTTP request examples from the JSON reference
- a checklist for validating Roblox API usage before shipping a script or integration
