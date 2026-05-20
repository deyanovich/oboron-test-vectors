# oboron-test-vectors

Cross-language test vectors for the
[oboron](https://oboron.org/) protocol — a *string-in,
string-out* symmetric encryption protocol that combines
encryption and encoding into one operation.

These vectors are intended to be consumed directly by any
language implementation. They're distributed as plain JSONL
files with stable schemas. No tooling required to read
them.

## Files

- `test-vectors.jsonl` — secure-scheme vectors (a-tier +
  u-tier: `aags`, `aasv`, `apgs`, `apsv`, `upbc`).
- `ztier-test-vectors.jsonl` — z-tier vectors (`zrbcx`).
- `legacy-test-vectors.jsonl` — legacy scheme vectors.

## Hardcoded test key

All vectors in `test-vectors.jsonl` and
`ztier-test-vectors.jsonl` use the same hardcoded test key,
which the oboron CLIs activate via the `-K` / `--keyless`
flag. The legacy vectors use a separate secret carried in
their meta line — see [`legacy-test-vectors.jsonl`](#legacy-test-vectorsjsonl)
below.

The key is **insecure by design** (it's published here) and
exists solely for cross-implementation conformance testing.

The canonical definition lives in the
[oboron CLI spec, section 8](https://oboron.org/cli-spec-v1-rev1#s8).
Implementations MUST use these exact values to produce or
consume the obtext strings in this repo's vectors.

### Master key (a-tier + u-tier)

Hex (128 chars, 64 bytes) — canonical form:

```
381284633d02ea5f35df8596b5cc4218310060468e8b465455a415174ea6e966a9f48eec4ba446ddfc8b78587895356f45a75a1ab7419454dd9f7aa8a95dbdd5
```

Base64url, no padding (86 chars) — historical form,
supported by pre-0.8 oboron implementations:

```
OBKEYz0C6l8134WWtcxCGDEAYEaOi0ZUVaQVF06m6Wap9I7sS6RG3fyLeFh4lTVvRadaGrdBlFTdn3qoqV291Q
```

### Z-tier secret

The `zrbcx` scheme operates on the first 32 bytes of the
master key. As hex (64 chars):

```
381284633d02ea5f35df8596b5cc4218310060468e8b465455a415174ea6e966
```

Base64url, no padding (43 chars):

```
OBKEYz0C6l8134WWtcxCGDEAYEaOi0ZUVaQVF06m6WY
```

## Vector schema

Each non-meta line in every file is a self-contained JSON
object:

```json
{"format": "aags.c32", "plaintext": "hello", "obtext": "..."}
```

- `format` — `<scheme>.<encoding>` (e.g. `aags.c32`,
  `aasv.b64`, `upbc.hex`). See the
  [oboron spec](https://oboron.org/) for the full format
  grammar.
- `plaintext` — input string for `enc`, expected output
  string for `dec`.
- `obtext` — output string for `enc` (deterministic
  schemes), or input string for `dec`.
- `description` (optional) — human-readable context.

### `test-vectors.jsonl`

Secure-scheme vectors. No meta line. The hardcoded master
key (above) applies.

### `ztier-test-vectors.jsonl`

Z-tier vectors for `zrbcx`. Same schema; no meta line. The
hardcoded z-tier secret (above; the first 32 bytes of the
master key) applies.

### `legacy-test-vectors.jsonl`

Legacy-scheme vectors. The **first line** is a meta object
carrying the secret used to derive the key for all
subsequent vectors:

```json
{"type": "meta", "secret": "<43-char base64url secret>"}
```

Subsequent lines follow the standard vector schema.
Implementations should read the meta line, extract
`secret`, and pass it through their CLI's `-s <secret>`
flag (or equivalent library call) when running the rest of
the vectors.

## Roundtrip vs. exact-match testing

- **Deterministic schemes** (`aags`, `aasv`, `zrbcx`,
  `legacy`): obtext is fully determined by plaintext + key.
  `enc(plaintext)` must produce exactly the vector's
  `obtext`; `dec(obtext)` must produce exactly the
  vector's `plaintext`.
- **Probabilistic schemes** (`apgs`, `apsv`, `upbc`):
  obtext varies per call. Skip the `enc` exact-match
  check; `dec(obtext)` must still produce exactly the
  vector's `plaintext`. To exercise the encrypt path,
  re-encrypt the plaintext and decrypt the fresh output
  back to plaintext.

## License

MIT — see [LICENSE](LICENSE).
