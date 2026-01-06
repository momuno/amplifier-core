# DESIGN_PHILOSOPHY.md Source Analysis

**Document Analyzed**: `docs/DESIGN_PHILOSOPHY.md`

**Analysis Date**: 2026-01-05

**Purpose**: Identify source files used to create DESIGN_PHILOSOPHY.md and files that should have been included.

---

## Document Summary

**Scope**: Complete design philosophy for Amplifier kernel - principles, patterns, and decision frameworks guiding all development.

**Main Point**: Keep kernel tiny, stable, boring (~2,600 lines) providing MECHANISMS only. All POLICIES live in replaceable modules at edges. Linux kernel as guiding metaphor.

**Coverage Areas**:
- Core Principles (6 principles)
- Linux Kernel Decision Framework (metaphor mapping, playbook)
- Kernel vs Module Boundaries
- Module Design (Bricks & Studs LEGO model)
- Implementation Patterns (vertical slices, testing, error handling)
- Decision Framework (5 questions)
- Anti-Patterns (kernel, module, design, process)
- Governance & Evolution
- Quick Reference checklists

---

## Likely Source Files (High Confidence)

These files were MOST LIKELY used as sources when writing DESIGN_PHILOSOPHY.md:

### 1. README.md
**Confidence**: VERY HIGH

**Evidence**:
- Architecture diagram appears in both (nearly identical)
- "Mechanisms, Not Policies" section matches DESIGN_PHILOSOPHY Core Principle #1
- Litmus test phrasing is identical: "Could two teams want different behavior?"
- Module type list matches
- Stability guarantees section similar

**Sections sourced**:
- Architecture diagram
- Design Philosophy section
- Core Concepts (module types)

### 2. context/kernel-overview.md
**Confidence**: VERY HIGH

**Evidence**:
- "The center stays still so the edges can move fast" quote appears in both
- Core Tenets section maps directly to Core Principles
- "What kernel provides" / "What kernel does NOT provide" lists match Kernel vs Module Boundaries section
- Litmus test explanation identical
- Module protocols section referenced

**Sections sourced**:
- Core Tenets (becomes Core Principles)
- What kernel provides/doesn't provide
- Litmus test application
- Philosophy grounding

### 3. docs/contracts/README.md
**Confidence**: MODERATE

**Evidence**:
- Module types table structure similar
- Quick start pattern referenced
- Source of Truth section philosophy aligns

**Sections sourced**:
- Module types overview
- Entry point pattern basics

---

## Files That SHOULD Have Been Included

These files contain relevant information that aligns with DESIGN_PHILOSOPHY.md's scope but were NOT used as sources:

### HIGH Priority (Directly Relevant)

#### 1. amplifier_core/coordinator.py
**Why**: Contains actual implementation of:
- Mount points (mentioned in philosophy)
- Capability registry (inversion of control pattern)
- Contribution channels (pull-based aggregation)
- Hook result processing subsystems

**Relevant Content**:
- Lines 39-91: ModuleCoordinator class with infrastructure context
- Lines 146-224: mount/unmount/get methods (module attachment mechanism)
- Lines 256-321: Contribution channels implementation
- Lines 341-500: Hook result processing (inject_context, approval, display)

#### 2. amplifier_core/session.py
**Why**: Contains session lifecycle implementation referenced in philosophy

**Relevant Content**:
- Lines 22-76: AmplifierSession initialization with parent_id for forking
- Lines 95-223: initialize() method (module loading sequence)
- Lines 224-276: execute() method (orchestrator coordination)

#### 3. amplifier_core/loader.py
**Why**: Implements module loading mechanism and validation

**Relevant Content**:
- Lines 29-38: TYPE_TO_MOUNT_POINT mapping (kernel mechanism)
- Lines 170-251: load() method with source resolution
- Lines 253-278: _load_direct() fallback mechanism
- Lines 421-477: _validate_module() before loading

#### 4. amplifier_core/hooks.py
**Why**: Implements event-first observability principle

**Relevant Content**:
- Lines 32-289: HookRegistry class
- Lines 103-189: emit() method with action precedence
- Lines 191-222: _merge_inject_context_results()
- Lines 224-274: emit_and_collect() for aggregation

#### 5. amplifier_core/events.py
**Why**: Defines canonical event taxonomy for observability

**Relevant Content**:
- Lines 1-95: Complete canonical event list
- Event naming convention (namespace:action)

#### 6. docs/specs/MOUNT_PLAN_SPECIFICATION.md
**Why**: Embodies mechanism vs policy in configuration contract

**Relevant Content**:
- Lines 19-35: Mount Plan purpose (app compiles, kernel validates)
- Lines 36-88: Schema structure
- Lines 121-135: agents section semantics (policy data, not modules)
- Lines 376-384: Philosophy section

#### 7. docs/MODULE_SOURCE_PROTOCOL.md
**Why**: Demonstrates kernel extension points and mechanism-only design

**Relevant Content**:
- Lines 9-13: "How modules are discovered is app-layer policy"
- Lines 153-170: Kernel responsibilities (what it does/doesn't do)
- Lines 247-256: Kernel invariants

#### 8. docs/SESSION_FORK_SPECIFICATION.md
**Why**: Shows parent_id mechanism and policy separation

**Relevant Content**:
- Lines 9-11: Kernel provides mechanism, app provides policy
- Lines 58-68: Kernel responsibilities for forking
- Lines 213-242: What kernel does NOT provide

#### 9. docs/CAPABILITY_REGISTRY.md
**Why**: Demonstrates inversion of control pattern

**Relevant Content**:
- Lines 1-3: Purpose (module ↔ app communication without dependencies)
- Lines 7-24: API pattern
- Lines 33-59: Architecture diagram showing separation

#### 10. docs/specs/CONTRIBUTION_CHANNELS.md
**Why**: Demonstrates pull-based aggregation mechanism

**Relevant Content**:
- Lines 17-30: Purpose and key properties
- Lines 36-65: Coordinator API
- Lines 117-132: Failure handling (non-interference)

### MEDIUM Priority (Supportive)

#### 11. amplifier_core/interfaces.py
**Why**: Protocol definitions referenced throughout philosophy

**Relevant Content**:
- Lines 33-58: Orchestrator protocol
- Lines 61-125: Provider protocol  
- Lines 128-152: Tool protocol
- Lines 155-202: ContextManager protocol
- Lines 205-220: HookHandler protocol

#### 12. amplifier_core/models.py
**Why**: Core data models including HookResult

**Relevant Content**:
- Lines 82-268: HookResult with all capabilities
- Lines 270-288: ModelInfo
- Lines 290-342: ConfigField and ProviderInfo
- Lines 345-386: ModuleInfo and SessionStatus

#### 13. docs/HOOKS_API.md
**Why**: Complete hook system reference for observability

**Relevant Content**:
- Lines 19-30: Hook capabilities overview
- Lines 70-99: Action precedence hierarchy
- Lines 367-397: Best practices

#### 14. docs/contracts/PROVIDER_CONTRACT.md
**Why**: Example of mechanism/policy separation in practice

**Relevant Content**:
- Lines 27-29: Purpose statement
- Lines 47-66: Protocol definition
- Lines 125-138: Observability via contribution channels

### LOW Priority (Contextual)

#### 15. amplifier_core/validation/ directory
**Why**: Shows validation as a mechanism

**Relevant Content**: Validation framework implementation

#### 16. amplifier_core/testing.py
**Why**: Testing utilities mentioned in testing strategy

**Relevant Content**: Mock implementations for testing

---

## Analysis Summary

### Files DEFINITELY Used as Sources (3)
1. README.md - Architecture, core principles, litmus test
2. context/kernel-overview.md - Tenets, what kernel does/doesn't do  
3. docs/contracts/README.md - Module types, patterns

### Files SHOULD Have Been Included (10 HIGH priority)
1. amplifier_core/coordinator.py - Mount points, capabilities, channels
2. amplifier_core/session.py - Session lifecycle implementation
3. amplifier_core/loader.py - Module loading mechanism
4. amplifier_core/hooks.py - Event system implementation
5. amplifier_core/events.py - Canonical event taxonomy
6. docs/specs/MOUNT_PLAN_SPECIFICATION.md - Configuration contract
7. docs/MODULE_SOURCE_PROTOCOL.md - Extension points
8. docs/SESSION_FORK_SPECIFICATION.md - Forking mechanism
9. docs/CAPABILITY_REGISTRY.md - Inversion of control
10. docs/specs/CONTRIBUTION_CHANNELS.md - Aggregation mechanism

### Why These Matter
The document discusses principles and patterns that are IMPLEMENTED in these source files. Including them would:
- Provide concrete examples of the principles in practice
- Show how mechanism/policy separation works in real code
- Demonstrate the "tiny, stable, boring" claim with actual code
- Ground abstract principles in working implementations

---

## Recommendations for Regeneration

When regenerating DESIGN_PHILOSOPHY.md with doc-gen:

1. **Include the 3 confirmed sources** for continuity
2. **Add the 10 HIGH priority source files** to ground principles in implementation
3. **Reference specific line ranges** for key implementations (mount points, event emission, etc.)
4. **Use reasoning field** to explain why each source is relevant

This ensures the philosophy document is grounded in actual code, not just abstract principles.
