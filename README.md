# HuntProxy Workflows

A pack of **43 native HuntProxy flows**, ported from the official
[Caido community workflow store](https://github.com/caido-community/workflows).
Import them and they run out of the box — no compatibility layer, no HTTPQL.

Every flow in this pack is a **native HuntProxy flow definition** (edition 1),
produced by a one-time port: TypeScript scripts were compiled and inlined,
match queries were rewritten into HuntProxy's query language, and Caido's
branch-on-match nodes were split into explicit if/else logic. Nothing is
emulated at runtime; HuntProxy has no Caido importer and rejects Caido-format
definitions at the API.

## Importing

Via the UI: **Flows → Import** and pick a file from [`flows/`](./flows).

Via the API:

```sh
curl -X POST http://localhost:8888/api/v1/projects/1/flows/import \
  -H 'content-type: application/json' \
  --data-binary @<(jq -Rs '{definition: .}' flows/evalsso.json)
```

The response is the created flow, ready to enable. Passive flows attach to the
interceptor, active flows run on demand, and convert flows run from the
convert view.

## Workflows

| Workflow | Description | File |
| --- | --- | --- |
| Add Slashes | Add backslashes before special characters in strings | `add-slashes.json` |
| AI Mentor (GroqCloud) | Analyze HTTP requests for potential vulnerabilities with AI | `ai-mentor.json` |
| Base64 Decode | Base64 decode a string | `base64-decode.json` |
| Base64 Encode | Base64 encode a string | `base64-encode.json` |
| Clean HTTP Request | Clean HTTP request by removing unnecessary headers and parameters | `clean-http-request.json` |
| Color Top-Level & IFrame Navigations | Color the top level navigation item based on the current page | `color-top-level-navigation.json` |
| Content-Length Mismatch | Matches requests where the content-length header value does not match the body length | `content-length-mismatch.json` |
| Cookie Reflection | Flags requests that contain text matching a cookie value | `cookie-reflection.json` |
| Copy As Python Requests | Generate python3 requests code from raw http requests | `copy-as-python.json` |
| CORS Checker | Active workflow for testing CORS issues | `cors-checker.json` |
| Detect ASPX Viewstate | Detects if website uses ASPX Viewstate | `detect-viewstate.json` |
| EvalSSO | Create findings based on OAuth/OIDC SSO requests | `evalsso.json` |
| FindSSO | Highlight SSO requests (OAuth, OIDC, SAML) | `findsso.json` |
| Form Data To JSON | Convert form data to JSON | `form-data-to-json.json` |
| GAP "Sus" Params | Flag requests with "sus" parameters (GAP list) | `gap-sus-params.json` |
| Get Epoch Time | Get the current time as epoch timestamp | `get-epoch-time.json` |
| Get Random UUID | Generate a random UUID | `get-random-uuid.json` |
| Hostname in Response | Finds responses which contain the hostname | `hostname-in-response.json` |
| HTML Closure | Detects responses with more than one HTML closing tag | `html-closure.json` |
| HTML Decode Everything | Decode all HTML entities in a string | `html-decode-everything.json` |
| HTML Encode Everything | Encode all HTML entities in a string | `html-encode-everything.json` |
| HTTP Method Checker | Detects HTTP methods exposed by dynamically probing endpoints with OPTIONS requests | `http-method-checker.json` |
| Introspection Query | Ask a GraphQL schema what queries it supports | `introspection-query.json` |
| JSON Escape | Escape all JSON special characters in a string | `json-escape.json` |
| JSON To Form Data | Convert JSON to form data | `json-to-form-data.json` |
| JSON to XML | Convert JSON to XML | `json-to-xml.json` |
| JSON Unescape | Unescape all JSON special characters in a string | `json-unescape.json` |
| JSON Response With Wrong Content-Type | Detects JSON responses with wrong content type | `json-with-wrong-content-type.json` |
| Large Redirect Response | Detects redirect pages with large body | `large-redirect-response.json` |
| Leakz | Find potential leaked secrets, PII, and/or sensitive fields | `leakz.json` |
| Malformed HTTP Header | Detects when HTTP response contains a malformed header | `malformed-http-header.json` |
| Multiple Server Headers | Detects if server returns different server header at different paths | `multiple-server-headers.json` |
| nowafpls - 8KB | Tries to bypass WAF by adding a lot of padding to the request | `nowafpls.json` |
| OWASP Top 25 Vulnerable Parameters | Request contains one of OWASP Top 25 vulnerable parameters | `owasptop25.json` |
| PwnFox Support | Support for PwnFox | `pwnfox-support.json` |
| Redirect To Parameter Value | Detects HTTP responses redirecting to a GET parameter value | `redirect-to-param.json` |
| SameSiteFinder | Extracts issued cookie names and SameSite attributes for CSRF triage | `samesite-finder.json` |
| SecretSniffer | Detect secrets and API key leaks in HTTP responses | `secret-sniffer.json` |
| Unicode Decode | Unicode decode text | `unicode-decode.json` |
| Unicode Encode | Unicode encode text | `unicode-encode.json` |
| URL Decode | Decode a URL | `url-decode.json` |
| URL Encode | Encode a URL | `url-encode.json` |
| Url In Parameter | Detects requests that have URLs in parameter values | `url-in-param.json` |

## Porting notes

- **Queries.** Caido's HTTPQL match queries were rewritten into HuntProxy's
  own query language (the same one the history search and `flow/query-match`
  use). For example `req.raw.cont:"Cookie:"` becomes `request:~"Cookie:"`.
- **Branching.** Caido branches directly off a match node's true/false ports.
  HuntProxy keeps `flow/query-match` as a boolean producer and routes it
  through an explicit `flow/if-else`, so each match node is followed by a
  branch node comparing the match output to `true`.
- **Scripts.** Each TypeScript script was compiled, bundled, and inlined into
  its `flow/js` node's `code`. The scripts run on HuntProxy's QuickJS host,
  which provides a Caido-compatible `sdk` surface. `sdk.findings.create`
  creates real HuntProxy findings attached to the flow's triggering exchange;
  Caido-only fields (`dedupeKey`, `severity`) are accepted and ignored, and
  findings are not deduplicated.
- **Active inputs.** Caido's `request`/`response` references into a script
  collapse to HuntProxy's single shaped `input` (an exchange summary with
  `input.request` / `input.response` handles).
- **Store bugs.** A few store workflows reference nodes that do not exist in
  their published graph (Caido evaluates these as null). The port preserves
  that behaviour with empty values where the definition requires a concrete
  input.

## Adding workflows

These are curated native HuntProxy definitions — there is no importer and no
Caido support at runtime. To add a workflow, write it by hand against the
[flow node catalog](https://github.com/herraristotle/HuntProxy) (Flows →
node reference in the UI, or `GET /api/v1/flow-nodes`), or port one from
another tool offline and drop the resulting edition-1 JSON in `flows/`.

## License

MIT — see [LICENSE](./LICENSE). The original workflows are MIT-licensed
works of the Caido community and their respective authors; this pack is a
derivative port.
