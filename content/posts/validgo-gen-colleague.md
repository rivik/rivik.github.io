---
title: "validgo-gen: OpenAPI → Go Validation Done Right"
date: 2026-06-06
draft: false
categories: ["Engineering"]
summary: "My colleague Yury built validgo-gen — an OpenAPI 3.0 → Go generator that finally distinguishes missing fields from explicit nulls from zero values. Two-layer validation, chi integration, idiomatic output."
---

A short one — this time about a colleague's work worth sharing.

[Yury Usishchev](https://www.linkedin.com/posts/usishchev_github-sintoniastrategyvalidgo-gen-activity-7442165068840067072-B6Bi) built [validgo-gen](https://github.com/sintoniastrategy/validgo-gen) — an OpenAPI 3.0 → Go code generator focused on the one thing Go's JSON handling is notoriously bad at: **input validation**.

The core problem: after `json.Unmarshal`, a missing field, an explicit `null`, and an empty `""` all collapse into the same zero value. Your handler can't tell them apart — and for a strict API contract, those are three very different cases.

validgo-gen solves it with two-layer validation:

- **Pre-deserialization** — inspects raw JSON before unmarshaling: missing required fields and invalid nulls get caught while they're still distinguishable.
- **Post-deserialization** — go-playground/validator struct tags (`min`, `max`, `oneof`, `email`, `unique`) on the resulting structs.

What I like about the generated code:

- **chi integration** — plugs into existing middleware, no custom runtime lock-in.
- **Per-operation interfaces** — clean dependency injection instead of one giant server interface.
- **Idiomatic types** — `*string` for optionals, `decimal.Decimal`, `time.Time`. No Java-flavored Go.
- **Built via `go/ast`** — output is valid, gofmt-compliant Go by construction, not by template luck.

If you've been burned by oapi-codegen (no validation), ogen (non-standard runtime coupling), or openapi-generator (Java-style everything) — give it a look.
