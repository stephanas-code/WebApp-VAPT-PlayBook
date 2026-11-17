# Insecure Deserialization - Remediation and Secure Development

## Summary
Insecure deserialization can lead to RCE, privilege escalation, authentication bypass, data tampering, and DoS. Prevent it by eliminating unsafe deserialization of untrusted data and applying strict validation, integrity controls, and safe formats.

## Design Principles
- Do not deserialize untrusted data. Prefer safe data formats (JSON) with strict schemas.
- Treat all client-controlled data as untrusted, including cookies, headers, hidden fields, and serialized state.
- Enforce allowlists for accepted types/classes. Block unknown or dangerous types by default.
- Implement integrity and authenticity protection for any serialized state.
- Fail closed: if validation fails, reject the data and log securely.

## Language-/Framework-Specific Guidance

### Java
- Avoid native Java serialization (ObjectInputStream) and XMLDecoder.
- If unavoidable, use ObjectInputFilter (JEP 290) to allowlist classes.
- Prefer JSON with strict schema validation (Jackson with afterburner disabled for polymorphic types; avoid enableDefaultTyping).
- Keep libraries up to date to remove known gadget chains (e.g., Commons Collections).
- Isolate deserialization in a low-privilege sandbox (SecurityManager is deprecated; use containerization/AppArmor/SELinux).

### .NET
- Avoid BinaryFormatter, NetDataContractSerializer, LosFormatter. These are insecure by design.
- Use safe serializers: System.Text.Json, DataContractJsonSerializer with known types, XmlSerializer with known types.
- For legacy ViewState, enable MAC (ViewStateUserKey), set ViewStateEncryptionMode=Always, and disable ViewState if not needed.
- Apply type name handling restrictions in Json.NET (TypeNameHandling=None) and use SerializationBinder to restrict types.

### PHP
- Avoid unserialize() on untrusted input. If legacy, pass allowed_classes=false or an allowlist array.
- Disable PHAR stream wrappers for user-controlled paths (phar.readonly=On in php.ini) and validate uploads strictly.
- Avoid magic methods side-effects (__wakeup, __destruct) or ensure they are safe and idempotent.
- Store server-side state; do not trust client-side serialized objects.

### Python
- Never use pickle/yaml.load on untrusted data. Use pickle only for trusted, internal data.
- For PyYAML, use safe_load. For JSON, validate against a schema (jsonschema).
- Avoid eval/ast.literal_eval on untrusted input; use explicit parsing.

### Node.js
- Avoid node-serialize and unsafe revivers. Use JSON.parse with strict validation and ajv schemas.
- Prevent prototype pollution by freezing prototypes and validating input; use Object.create(null) for maps.

## Controls and Hardening
- Integrity protection: sign and/or encrypt serialized state (HMAC with rotating keys). Store secrets securely.
- Transport security: enforce HTTPS to protect tokens and serialized state in transit.
- Least privilege: run app with minimal OS and DB permissions; isolate deserialization code paths.
- Monitoring: detect Java magic bytes (0xACED), PHAR headers, Pickle opcodes, suspicious $type fields.
- Rate limiting and circuit breakers to mitigate DoS from crafted payloads.

## Detection and Testing
- Static: search for dangerous APIs (ObjectInputStream, BinaryFormatter, unserialize, pickle.loads, yaml.load).
- Dynamic: fuzz endpoints carrying serialized data (cookies, headers, hidden fields). Attempt known payloads with instrumentation.
- Use SCA/Dependency checks to flag vulnerable libraries/gadget chains.
- Employ RASP/WAF rules for serialization signatures and block/alert.

## Example Fixes

### Java (ObjectInputFilter)
```java
ObjectInputFilter filter = ObjectInputFilter.Config.createFilter("maxdepth=20;package.com.myapp.*;!java.util.concurrent.*,!*com.sun.*;!*org.apache.commons.collections.*");
ois.setObjectInputFilter(filter);
```

### .NET (Json.NET with Binder)
```csharp
var settings = new JsonSerializerSettings {
  TypeNameHandling = TypeNameHandling.None,
  SerializationBinder = new KnownTypesBinder { KnownTypes = new[] { typeof(MySafeType) } }
};
```

### PHP (unserialize allowlist)
```php
$data = unserialize($input, ['allowed_classes' => ['SafeDTO']]);
```

### Python (PyYAML safe)
```python
import yaml
data = yaml.safe_load(user_input)
```

## Incident Response
- Immediately rotate keys if tokens/state integrity is suspected compromised.
- Invalidate sessions and server-side cache tied to client state.
- Patch and redeploy libraries closing gadget chains.
- Review logs for indicators of exploitation and scope of impact.

## Business Impact and Risk Treatment
- Potential for full compromise (RCE) warrants Critical severity; prioritize patching and architectural changes.
- Track as a top risk in threat models. Add unit/integration tests to prevent regressions.

## 2026 Modern & AI-Assisted Defensive Updates
- **Schema-First Data Formats & Runtime Type Validators**: Replace native object serialization entirely with strictly typed, schema-validated formats (Protocol Buffers, FlatBuffers, or JSON validated at the boundary via Zod, Valibot, Pydantic v2, or JSON Schema Draft 2020-12).
- **Platform Deprecation & Removal of Legacy Deserializers**:
  - **.NET 9/10+**: `BinaryFormatter` runtime implementations are removed/throw by default; migrate legacy state to `System.Text.Json` source generators with explicit `JsonSerializable` contexts.
  - **Java 21+**: Enforce JVM-wide `jdk.serialFilter` and `jdk.serialFilterFactory` policies at startup to deny all classes except explicitly allowlisted DTOs, and prefer Java `record` types for immutable data transfer.
  - **Python / ML Model Security**: Ban `pickle` and `torch.load(..., weights_only=False)` for untrusted ML artifacts; require `safetensors` or `weights_only=True` with cryptographic provenance verification (Sigstore/SLSA).
- **AI-Assisted Reachability Analysis & Automated Refactoring**:
  - Combine Software Composition Analysis (SCA) with AI-assisted call-graph reachability in CI to identify whether transitive gadget chains are reachable from untrusted deserialization sinks.
  - Use AI-assisted code migrations to automatically refactor legacy `pickle`/`unserialize`/`ObjectInputStream`/`yaml.load` calls into schema-validated JSON/Protobuf DTOs with accompanying regression tests.

---
*Part of the Web Application VAPT Guide - Insecure Deserialization Module*
