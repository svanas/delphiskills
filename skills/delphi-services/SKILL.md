---
name: delphi-services
description: Implement or review Delphi HTTP and REST clients, WebBroker handlers, DataSnap or RAD Server integrations, and service boundaries.
---

# HTTP clients and Delphi services

For Delphi 11 Alexandria and newer. If not already loaded, read [the shared workflow](../delphi/SKILL.md) once; do not recursively reload it. Apply only the parts relevant to the requested change.

## Choose the existing abstraction

Identify whether the code uses `System.Net.HttpClient`, `TNetHTTPClient`, REST components, WebBroker, DataSnap, RAD Server, or another framework. Preserve the application's established abstraction unless the requested change calls for migration. The books demonstrate multiple alternatives; they are not instructions to combine them all. [E ch. 11–13]

Separate serialization and business operations from transport handlers. A form event or web action should translate input, call the operation, and translate its result. Do not make every unit depend on a UI form or an HTTP request object. Inspect the server's concurrency and module-caching model before keeping mutable per-request state in a web module field.

## HTTP clients

Specify timeouts and expected status codes. A transport success is not necessarily an application success; a successful HTTP response may have an empty body or a different content type. Check before parsing. Preserve response headers needed for pagination, rate limits, versioning, or caching.

Keep response streams, request bodies, client objects, and callback targets alive for the complete operation. Rewind streams where the next consumer expects position zero. Decode text with the protocol's encoding contract. Do not load an arbitrary binary response into a memo or string because a demonstration downloads a text file. [E pp. 301–303]

Long network operations must not block ordinary UI interaction. Marshal completion safely and handle cancellation/close races. Validate TLS normally; do not install an accept-all certificate callback to make a request succeed. Encode path segments and query values correctly rather than concatenating unchecked values. Keep credentials/tokens out of URLs and logs when the protocol offers a proper header mechanism.

Retry only failures for which replay is safe. A timeout after a POST does not prove the server made no change. Use service-supported idempotency or reconcile state before retrying non-idempotent operations. Bound retries and surface meaningful failure. These are editorial production requirements beyond the introductory client demos.

## Server handlers

Match HTTP methods, routes, status codes, and body schemas to the service contract. Do not adopt tutorial-style mutating GET endpoints as the default design. Validate body size, content type, required fields, and domain constraints before mutation. Authentication establishes identity; authorization must still check access to the requested object or operation.

Finish the response once, set the handled state as required by the hosting framework, and translate expected errors consistently. Avoid returning raw exception messages, connection strings, or stack traces to clients. Retain useful diagnostics in the application's logging system.

Use request-scoped or otherwise safely managed database resources, and keep transaction ownership explicit. A shared singleton connection in a cached web module is not automatically safe for concurrent requests. Test simultaneous requests, not just a single browser request. [E ch. 12–13; data skill]

## Legacy and deployment boundaries

DataSnap and RAD Server have distinct deployment and licensing requirements; verify them for the selected edition rather than repeating old MIDAS pricing. WebBroker is an HTTP application framework, not a complete authentication or persistence policy by itself. Keep an existing SOAP/WSDL integration when that is the external contract.

The annotated Delphi 5 book explicitly marks ActiveForms/ActiveX browser embedding as obsolete and notes the loss of Delphi CORBA support. Do not use those as new web architecture recommendations. COM itself remains relevant on Windows and belongs in the interop skill. [M PDF 943, 1001, 1007]

## Verification

Test non-2xx responses, malformed/empty bodies, unexpected media types, timeout after mutation, cancellation, authorization failure, duplicate requests, and server concurrency. Use a controlled local stub for deterministic client tests plus integration tests against the intended service. Verify deployment configuration separately from handler unit tests.

## Sources and interpretation

E: *Expert Delphi*, ch. 11 pp. 299–322, ch. 12 pp. 323–347, ch. 13 pp. 349–366. Concrete PDF anchors: 324–325 (HTTP client), 350 and 356–357 (WebBroker demo). M: *Mastering Delphi 5, 2025 Annotated Edition*, PDF 943 footnote 408 and PDF 1001/1007 footnotes 431, 443–444. Retry, authorization, TLS, and concurrency safeguards are editorial extensions, not copied book recipes.
