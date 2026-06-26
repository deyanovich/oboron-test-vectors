# oboron-test-vectors

Cross-language test vectors for the [oboron](https://oboron.org/)
protocol — a *string-in, string-out* authenticated symmetric
encryption protocol that combines encryption and encoding into one
operation.

These vectors are consumed directly by any language implementation.
They are plain JSONL files with a stable schema; no tooling is
required to read them.

## Files

- `test-vectors.jsonl` — core schemes (`dsiv`, `dgcmsiv`, `psiv`,
  `pgcmsiv`).
- `obu-test-vectors.jsonl` — obu schemes (`upcbc`, `zdcbc`).
- `negative-test-vectors.jsonl` — core-scheme inputs that MUST be
  rejected (see Negative vectors below).
- `obu-negative-test-vectors.jsonl` — obu-scheme inputs that MUST be
  rejected (run via the `obu` binary).
- `legacy-test-vectors.jsonl` — legacy-scheme vectors (separate
  secret; see below).

## Keys

All vectors in `test-vectors.jsonl` and `obu-test-vectors.jsonl`
use one hardcoded test key, **insecure by design** (it is published
here) and intended solely for cross-implementation conformance
testing. The CLIs activate it via the `-K` / `--keyless` flag. The
canonical definition lives in the Oboron CLI Specification (Fixed
Public Test Key).

The core schemes and the obu schemes use different key material, and
are activated through different binaries:

### Master key — core schemes (via `ob -K`)

`dsiv`, `dgcmsiv`, `psiv`, and `pgcmsiv` use a single 512-bit master
key. Hex (128 chars, 64 bytes):

```
381284633d02ea5f35df8596b5cc4218310060468e8b465455a415174ea6e966a9f48eec4ba446ddfc8b78587895356f45a75a1ab7419454dd9f7aa8a95dbdd5
```

`dsiv`/`psiv` use the full 64-byte key as the AES-SIV key.
`dgcmsiv`/`pgcmsiv` derive a 32-byte AES-GCM-SIV key from it with
`HKDF-Expand` (info `gcmsiv`, shared by both; see the protocol spec).

### obu secret — `upcbc` and `zdcbc` (via `obu -K`)

The obu schemes use a 256-bit secret, which for these vectors is the
first 32 bytes of the master key. Hex (64 chars):

```
381284633d02ea5f35df8596b5cc4218310060468e8b465455a415174ea6e966
```

`upcbc` uses the full 32-byte secret as the AES-256-CBC key. `zdcbc`
uses bytes 0–15 as the AES-128-CBC key and bytes 16–31 as the
constant IV.

## Vector schema

Each non-meta line is a self-contained JSON object:

```json
{"format": "dsiv.c32", "plaintext": "hello", "obtext": "..."}
```

- `format` — `<scheme>.<encoding>` (e.g. `dgcmsiv.c32`, `dsiv.b64`,
  `upcbc.hex`). See the protocol spec for the format grammar.
- `plaintext` — input string for `enc`; expected output for `dec`.
- `obtext` — output string for `enc` (deterministic schemes), or
  input string for `dec`.
- `description` (optional) — human-readable context.

The obtext is the encoding of the scheme output directly — there is
no scheme marker.

### `legacy-test-vectors.jsonl`

Legacy-scheme vectors. The **first line** is a meta object carrying
the secret used for all subsequent vectors:

```json
{"type": "meta", "secret": "<base64url secret>"}
```

Implementations read the meta line, extract `secret`, and pass it
through their CLI's secret flag for the rest of the vectors. These
vectors are retained for legacy compatibility and are outside the
1.0 scheme set.

## Negative vectors

`negative-test-vectors.jsonl` (core schemes, via `ob`) and
`obu-negative-test-vectors.jsonl` (obu schemes, via `obu`) hold inputs
a conforming implementation MUST reject. They mirror the positive split
so each binary is fed only the schemes it owns. Each line:

```json
{"op": "dec", "format": "dsiv.c32", "input": "...", "reason": "..."}
```

- `op` — `enc` or `dec`: the operation to run.
- `format` — the format string.
- `input` — the input fed to `op` (a plaintext for `enc`, an obtext
  for `dec`), under the hardcoded test key.
- `reason` — informative; the rule being exercised.

Running `op` on `input` MUST fail; through the CLI this is a non-zero
exit (exit 1, an operation failure). Categories: non-canonical
encodings (wrong case, padding, out-of-alphabet character, impossible
length, non-zero trailing bits), scheme outputs shorter than the
layout minimum, authentication failures (a tampered obtext, for the
authenticated schemes), and the empty plaintext.

## Roundtrip vs. exact-match testing

- **Deterministic schemes** (`dsiv`, `dgcmsiv`, `zdcbc`): the obtext
  is fully determined by plaintext + key. `enc(plaintext)` must
  produce exactly the vector's `obtext`, and `dec(obtext)` must
  produce exactly the `plaintext`.
- **Probabilistic schemes** (`psiv`, `pgcmsiv`, `upcbc`): the obtext
  varies per call. Skip the `enc` exact-match check; `dec(obtext)`
  must still produce exactly the `plaintext`. To exercise the
  encrypt path, re-encrypt the plaintext and decrypt the fresh
  output back to the plaintext.

## License

MIT — see [LICENSE](LICENSE).
