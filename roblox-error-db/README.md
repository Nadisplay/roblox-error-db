# Roblox ErrorDB

A plug-and-play JSON catalog of **Roblox Studio / Luau** error lines for tooling (for example a `ModuleScript` that explains Output).

## Features

- Curated `messagePattern` substrings aligned with real Studio output
- `priority`, optional `confidenceBoost`, `keywords`, `description`, and `fixes` per entry
- Versioned [`errors.json`](errors.json) and [`version.json`](version.json)

## Case normalization

All `messagePattern` strings are stored in **lowercase**. Before matching, normalize the error line:

`local line = string.lower(rawErrorText)`

Then use plain substring search, for example `string.find(line, pattern, 1, true)`.

`keywords` are also **lowercase** so you can score or filter the same way after lowercasing.

## Matching logic (for implementers)

When more than one entry matches the same line (because one pattern is a substring of another):

1. Prefer the match with **higher `priority`** (larger number = more important / more specific).
2. If priorities tie, prefer the **longer `messagePattern`** (more specific substring).
3. Optionally, iterate entries in a **fixed order** and take the first match only—then list **more specific patterns before broader ones** in your source array.

Example: `attempt to call a nil value` matches both a nil-specific row and the generic `attempt to call a` fallback; the nil-specific entry should win via higher `priority` or longer pattern.

### Optional `confidenceBoost`

Some rows set **`"confidenceBoost": 1.2`** (numeric). After you pick a match (or for ranking), you can multiply score by this value for very stable patterns (e.g. `attempt to index nil`, `is not a valid member`). Omit the field when not needed (treat as `1`).

### Fallback row

Exactly one entry uses **`"fallback": true`** (`id`: **`G002`**): pattern **`attempt to call a`** (no trailing space) with **`priority`: 0**. Use only when no higher-priority row matched, or when your UI needs a generic “non-callable value” bucket.

### Syntax: `expected` vs `unexpected`

The substring **`expected` appears inside `unexpected`**, so a line like `unexpected symbol near` matches both a broad `expected` pattern and `unexpected symbol`. This catalog uses **`unexpected symbol` at priority 4** and **`expected` at priority 2**, so the more specific row wins when both match.

## Setup (consumer)

1. Enable HTTP requests in the game if you load this from GitHub/raw URL.
2. `HttpService:GetAsync` (or bundle the JSON in a `ModuleScript`).
3. Lowercase the error line, then match with `string.find(line, pattern, 1, true)` (plain substring) unless you choose regex.

## Version

See [`version.json`](version.json) (currently aligned with `errors.version` in [`errors.json`](errors.json)).
