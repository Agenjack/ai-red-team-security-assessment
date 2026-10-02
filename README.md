# ai-red-team-security-assessment
# AI-Agent Red Team Security Assessment

## Overview

This repository documents an authorized AI-agent security assessment conducted against a controlled security testing environment.

The assessment focused on identifying vulnerabilities in AI-agent tools, resource handling, input processing, authorization controls, deserialization, prompt handling, and concurrent transaction processing.

The objective was to determine whether an attacker could abuse the exposed functionality to cross security boundaries, access protected resources, execute unintended operations, or manipulate application state.

> **Assessment type:** Authorized security assessment / Red-team exercise  
> **Focus:** AI agents, tool security, application security, and LLM security  
> **Tracks:** Advanced and Expert  
> **Methodology:** Manual testing, controlled exploitation, evidence collection, root-cause analysis, and security mapping

---

# Assessment Areas

The assessment covered six security challenges:

### Advanced Track

- Level 5 — Bad Robot
- Level 7 — Master of Resources
- Level 9 — Blind Network Ninja

### Expert Track

- Level 6 — The Shape Shifter
- Level 7 — The Octopus
- Level 8 — The Heist

---

# Findings Summary

| Level | Challenge | Primary Finding | Result |
|---|---|---|---|
| Advanced 5 | Bad Robot | Unsafe expression evaluation and Python object introspection | Vulnerability demonstrated; final objective not recovered |
| Advanced 7 | Master of Resources | Insufficient authorization around protected MCP resources/tools | Objective completed |
| Advanced 9 | Blind Network Ninja | Prompt injection and system-prompt disclosure | Security behavior demonstrated; final objective not recovered |
| Expert 6 | The Shape Shifter | Unsafe YAML deserialization | Objective completed |
| Expert 7 | The Octopus | Parser differential and authorization-state bypass | Objective completed |
| Expert 8 | The Heist | TOCTOU/concurrency double-spend condition | Objective completed |

---

# 1. Bad Robot

## Result

The assessment identified unsafe expression evaluation behavior.

The calculator functionality accepted expressions that could access Python object internals. Although several direct exploitation attempts were blocked, dynamic construction of restricted identifiers allowed deeper Python object access and ultimately reached process-execution functionality.

The final challenge objective was not recovered.

## Exploit

Testing began with ordinary mathematical expressions to establish expected behavior.

Further testing examined whether the evaluator was processing expressions as Python objects rather than using a restricted mathematical parser.

Python object introspection was then explored through class and subclass relationships.

Direct references to restricted functionality were blocked, so the relevant identifier was constructed dynamically rather than supplied as a literal string.

This allowed access to process-spawning functionality and demonstrated that arbitrary Python capabilities were reachable from the expression evaluator.

## Root Cause

The core issue was the use of a general-purpose Python expression evaluation mechanism for functionality that only required mathematical expressions.

Blacklist-style filtering was insufficient because Python's object model provides alternative paths to restricted functionality.

## Impact & Severity

Potential impact includes:

- Access to Python internals
- Escape from intended calculator functionality
- Process execution
- Potential command execution
- Potential access to application resources depending on runtime permissions

The demonstrated primitive represents a high-risk application security issue.

## Security Mapping

- Unsafe expression evaluation
- Improper input restriction
- CWE-style code execution risk
- Contextual OWASP LLM Excessive Agency considerations where an AI agent exposes the vulnerable capability

---

# 2. Master of Resources

## Result

The MCP service exposed protected resources and tools through insufficiently restricted authorization boundaries.

The assessment successfully accessed the protected flag resource after establishing an MCP session and interacting with the available resource and tool interfaces.

## Exploit

The MCP endpoint was first examined to understand its protocol requirements.

After establishing a valid session, the available resources and tools were enumerated.

The exposed resource and tool functionality revealed privileged operations that were accessible through the MCP interface.

The authorization boundary between ordinary MCP interaction and protected resources was insufficiently enforced.

## Root Cause

Protected MCP resources and privileged tool functionality were exposed without adequate authorization enforcement at the resource/tool boundary.

## Impact & Severity

Potential impact includes:

- Unauthorized access to protected resources
- Unauthorized invocation of privileged functionality
- Exposure of sensitive application data
- Potential privilege escalation depending on the resources exposed

## Security Mapping

- MCP authorization failure
- Broken access control
- Contextual OWASP LLM Excessive Agency considerations
- Application-level authorization weakness

---

# 3. Blind Network Ninja

## Result

The network connectivity service was vulnerable to prompt injection and system-prompt disclosure.

A crafted request caused the agent to disclose internal system instructions describing how the connectivity tool processed user input.

The final challenge objective was not recovered.

## Exploit

Initial testing established the normal behavior of the connectivity tool using benign hostnames and IP addresses.

Further testing examined how the agent interpreted instructions embedded in user input.

A prompt-injection payload successfully caused the agent to disclose its internal system instructions.

The disclosed instructions indicated that user-provided host input was passed directly to the connectivity executor without filtering, sanitization, or modification.

Additional command-injection, shell-metacharacter, internal-network, HTTP, and out-of-band attempts were tested, but no validated flag retrieval was obtained.

## Root Cause

The agent trusted user-supplied instructions and exposed internal system-level instructions through prompt injection.

The underlying architecture also appeared to pass user-controlled input directly toward a network execution mechanism.

## Impact & Severity

Potential impact includes:

- System-prompt disclosure
- Exposure of internal agent architecture
- Disclosure of security controls and implementation details
- Potential assistance to further exploitation
- Potential network interaction depending on executor permissions

## Security Mapping

- OWASP LLM01 — Prompt Injection
- OWASP LLM07 — System Prompt Leakage
- Contextual LLM Excessive Agency considerations
- Prompt injection / system-prompt extraction concepts in MITRE ATLAS

---

# 4. The Shape Shifter

## Result

The configuration-processing functionality was vulnerable to unsafe YAML deserialization.

Controlled YAML payloads demonstrated that Python object construction could invoke subprocess functionality.

The vulnerability was subsequently used to read the protected challenge resource.

## Exploit

The configuration processor was tested with YAML object-construction syntax.

A controlled payload invoking a subprocess function demonstrated command execution.

A second controlled payload accessed the protected challenge file.

This demonstrated that attacker-controlled YAML could result in arbitrary Python-level operations.

## Root Cause

The application performed unsafe YAML deserialization that allowed attacker-controlled data to instantiate Python objects and invoke executable functionality.

## Impact & Severity

Potential impact includes:

- Arbitrary command execution
- Unauthorized file access
- Access to application secrets
- Full compromise of the application process depending on runtime privileges

The demonstrated capability represents a critical security issue.

## Security Mapping

- CWE-502 — Deserialization of Untrusted Data
- Unsafe YAML deserialization
- Arbitrary command execution
- Contextual OWASP LLM Excessive Agency considerations

---

# 5. The Octopus

## Result

The assessment identified a parser differential and authorization-state bypass.

A specially constructed input caused the security validation layer and application parser to interpret the same data differently.

This enabled a privileged operation that was initially denied.

## Exploit

The available tools were enumerated and their expected parameters were tested.

A controlled file was uploaded to establish the expected cache-update workflow.

The initial attempt to access the protected root resource was denied.

A specially crafted session key was then placed into an application-controlled cache workflow.

The resulting parser differential allowed the cache state to satisfy the authorization condition.

A chained operation subsequently accessed the protected root resource.

## Root Cause

The security validation layer and application parser did not interpret structured input consistently.

The discrepancy allowed security-relevant data to pass validation while being interpreted differently by the application.

## Impact & Severity

Potential impact includes:

- Authorization bypass
- Privilege escalation
- Access to protected resources
- Security-control bypass
- Potential compromise of other privileged agent operations

## Security Mapping

- Parser differential
- Inconsistent input validation
- Authorization bypass
- Contextual OWASP LLM Excessive Agency considerations

---

# 6. The Heist

## Result

The trading workflow was vulnerable to a concurrency/TOCTOU condition.

Two pending transactions were created against a limited balance and then executed concurrently.

Both transactions succeeded even though the available balance should not have supported both operations.

## Exploit

The initial account balance was established.

Two separate purchase orders were created against the available balance.

Instead of executing the transactions sequentially, both pending orders were submitted for execution concurrently.

Both executions succeeded.

This demonstrated that the balance validation and transaction deduction were not performed as one atomic operation.

## Root Cause

The application contained a time-of-check/time-of-use race condition.

Balance verification and balance deduction were separated in a way that allowed concurrent requests to observe the same available balance before either transaction had fully committed its deduction.

## Impact & Severity

Potential impact includes:

- Double spending
- Unauthorized value transfer
- Duplicate purchases
- Inventory or entitlement duplication
- Financial inconsistency
- Potential loss of funds depending on the real-world application

## Security Mapping

- CWE-367 — Time-of-Check Time-of-Use Race Condition
- Concurrency/race-condition vulnerability
- Contextual OWASP LLM Excessive Agency considerations

---

# Methodology

The assessment followed a controlled red-team workflow:

1. Establish normal application behavior.
2. Enumerate available functionality.
3. Identify security boundaries.
4. Test input validation and authorization.
5. Construct controlled exploitation attempts.
6. Validate observed behavior.
7. Determine root cause.
8. Assess potential impact.
9. Map findings to relevant security frameworks.
10. Preserve evidence for reproducibility.

The assessment emphasized evidence-based conclusions. Where a final challenge objective was not recovered, the result was documented as incomplete rather than claiming an unverified success.

---

# Key Security Lessons

The assessment demonstrated several recurring security principles:

### 1. Never rely on blacklist filtering for code execution controls

Restricted keywords can often be bypassed through alternative language features or dynamic construction.

### 2. AI-agent tools require strong authorization boundaries

Exposing a tool or resource through an agent does not remove the need for conventional access control.

### 3. System prompts should not be treated as security boundaries

Prompt instructions can be exposed through prompt injection and should not contain secrets or serve as the sole enforcement mechanism.

### 4. Deserialize untrusted data safely

Unsafe object construction can transform apparently harmless configuration data into arbitrary code execution.

### 5. Security validation and application parsing must agree

Parser differentials can create unexpected authorization bypasses.

### 6. Financial operations must be atomic

Balance checks and deductions should be performed transactionally so concurrent requests cannot reuse the same balance.

---

# Disclaimer

This repository is a sanitized portfolio representation of an authorized security assessment.

Sensitive platform information, credentials, proprietary assessment material, internal infrastructure details, and challenge-specific secrets are intentionally omitted.

The purpose of this repository is to demonstrate security assessment methodology, vulnerability analysis, exploitation reasoning, and technical reporting.

---

# Skills Demonstrated

- AI-agent security testing
- Red-team methodology
- Prompt injection testing
- System-prompt leakage analysis
- Application security
- MCP security
- Authorization testing
- Input-validation analysis
- Unsafe deserialization analysis
- Python security
- YAML security
- Race-condition analysis
- TOCTOU vulnerability analysis
- Vulnerability impact assessment
- Security documentation
- OWASP security mapping
- CWE mapping
