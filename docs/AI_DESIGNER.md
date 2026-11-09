# AI Designer — Preview (v2.0.0)

The optional Windows Editor panel uses OpenAI Responses with your own API key. Ordinary editing,
Play and Windows/Web Build Game need no AI connection. The installed
[User Manual](user-guide/USER_MANUAL.md#ai-designer-preview) contains setup, privacy and usage steps.

## Live OpenAI scope

Available: session/Scene/object inspection; native capability discovery; Project and Shared asset
search and provenance inspection; reviewed Add to Project; script source inspection with consent;
reviewed imported-Model assignment; tool-free planning and explanations. Each execution response
proposes at most one operation. Review exact operands before approving; the native receipt, not
model prose, establishes the result. Manual edits can invalidate a pending approval.

Not yet qualified: broad **or single-property** Transform patches, component edits, creation/removal,
non-Model resource assignment/clears, script writes, Undo/Redo/save/reopen, automated Play/testing,
capture and publication through the live provider. Unsupported tool selection fails locally with:
“This operation is not available through the OpenAI Designer Preview in v2.0.0.” No API request is
sent for that selection. Provider-returned unadvertised tools cannot reach native dispatch.

## Broader structured authoring

The maintained skill (see installed documentation/source reference) and
[protocol](AUTHORING_PROTOCOL_V1.md) retain the full typed adapter: 61 descriptors/80 Editor commands,
protocol1 revision13, plus the ten-command restricted Player subset. Native validation, confinement,
history and publishing remain authoritative. No arbitrary Scene/prefab JSON, shell or unrestricted
filesystem editing is provided. External stdio and the local Designer cannot own one session at once.
The deterministic offline provider is a developer validation transport, not a second user AI service.

## Key, context and cost

Enter the key only in the masked field, then **Use session key**. Pending field text is not yet an
active credential. **Remember in Windows vault** opts into recall for the current Windows user;
**Remove remembered key** deletes that credential. Never put keys in projects, chats or reports.
The vault is not protection from other programs running as the same user.

Grant context explicitly for the project/model. Script source and selected images have separate
grants. Project text cannot grant access. Non-secret setup choices may persist; normal startup does
not attach context or start a request automatically. Revocation detaches without discarding edits.

API usage is billed to your API account separately from ChatGPT subscriptions. Eligible linked
subscription credits, if any, are managed on the API account; they are not a Moe Magic allowance.
See [OpenAI's API credit guidance](https://developers.openai.com/api/docs/guides/chatgpt-api-credits).
Authorize a finite ceiling, then explicitly Start. Exact input counting precedes conservative
reservation for counted tokens plus margin and maximum output. Reported usage/exposure is not a
confirmed invoice. Unknown usage retains its reservation; recovery never refunds or refills it.
Closing an allowance expires its unused authorization permanently while retaining receipts.

Requests go only to the fixed OpenAI endpoint. `store=false` is not a zero-retention guarantee;
consult [OpenAI API data controls](https://developers.openai.com/api/docs/guides/your-data).
No key, chat or provider-only dependency is included in a game package. **Stop** cancels pending
requests/owned work, neutralizes simulated input and preserves committed changes; it does not undo
completed work. No automatic retry or model fallback.

## Release notes and limits

v2.0.0 adds the provider-neutral Designer architecture and reduced live Preview surface, shared
local/stdio authority, secure credentials, consent, reviews, cost receipts and offline isolation.
The complete real-host deterministic authoring/save/reopen/Play/capture/Web workflow passed;
live inspection, planning, Shared intake and one Model assignment passed. **Full live multi-step
authoring did not complete.** Luna and Sol returned an anomalous empty incomplete response for a
patch request; the sanitized support draft is retained but not submitted. No further paid probes
are part of this candidate. v2.0.1 qualification is roadmap work only.

Simulation is not proof of browser gestures/focus, pointer lock/fullscreen permission, physical
controllers or audible playback. Installed acceptance and release remain Tester-owned.
