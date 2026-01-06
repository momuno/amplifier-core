# Outline Generation Process - Extracted from Session

**Session ID**: `e056a1e9-a60c-4f74-a33e-941a5c310605`  
**Extracted**: 2026-01-06  
**Purpose**: Document the complete process for generating documentation outlines from existing documents

---

## Process Overview

This process takes an existing document and generates a structured JSON outline that can be used to regenerate that document from source materials. The process includes source discovery, circular reference detection, and validation.

**Input**: Target document path (e.g., `docs/DESIGN_PHILOSOPHY.md`)  
**Output**: 
- Source analysis markdown
- Source tree YAML (dependency DAG)
- Regenerated document in staging
- Complete outline JSON with source mappings

---

## Phase 1: Document Analysis

**Purpose**: Extract the essence and intended scope of the target document

### Steps

1. **Read target document**
   - Input: Document path
   - Tool: `read_file`
   - Output: Full document content

2. **Create document summary**
   - Analyze: What is this document about?
   - Analyze: What is its scope of coverage?
   - Analyze: What is the main point it's communicating?
   - Output: Summary statement capturing essence (2-3 sentences)

### Artifacts Created
- Mental model of document purpose
- Summary statement for validation

### Decision Points
- Is this a technical reference or explanatory synthesis?
- What must be included vs. nice-to-have?

---

## Phase 2: Repository Content Scan

**Purpose**: Build comprehensive understanding of all available source material

### Steps

1. **Discover all repository files**
   - Tool: `bash` with `find . -type f` or similar
   - Filter: Exclude build artifacts, node_modules, .git, etc.
   - Output: Complete file list

2. **Read all files systematically**
   - Prioritize: Documentation, specifications, contracts, core code
   - Tool: `read_file` in batches (parallel when possible)
   - Build: Mental index of what content exists where

### Artifacts Created
- Complete repository content in context
- Understanding of codebase structure

### Key Insight
> Reading ALL files enables evidence-based source identification rather than guessing

---

## Phase 3: Source Discovery & Evidence-Based Analysis

**Purpose**: Identify which files were likely sources for the original document

### Steps

1. **Compare document content to repository files**
   - Look for: Identical phrasing, similar structure, matching concepts
   - Evidence types:
     - Verbatim quotes or near-identical text
     - Same architectural diagrams
     - Matching terminology and definitions
     - Similar section structures

2. **Create initial source file list**
   - Category: Very High Confidence (verbatim matches)
   - Category: High Confidence (strong concept alignment)
   - Category: Medium Confidence (related concepts)
   - Category: Should Have Been Included (based on doc scope)

3. **Document evidence for each source**
   - What content came from this file?
   - What sections does it support?
   - Why is it relevant to the document's purpose?

### Artifacts Created
- Initial source file list with confidence levels
- Evidence mapping (file → document sections)

### Decision Framework
- **Include if**: Clear evidence of content derivation
- **Exclude if**: Only tangentially related
- **Flag if**: Content should exist based on scope but doesn't appear sourced

---

## Phase 4: Cross-Repository Source Discovery

**Purpose**: Identify sources from external repositories (loaded in context)

### Steps

1. **Check loaded context for external sources**
   - Pattern: Look for `@bundle-name:path` references
   - Common locations: `~/.amplifier/cache/{bundle-name}-{hash}/`
   - Examples: `amplifier-foundation`, other bundles

2. **Read external source files**
   - Tool: `read_file` with full cache paths
   - Compare: Does external content appear in target document?
   - Evidence: Same as Phase 3 (verbatim, structural, conceptual)

3. **Get commit hashes for external repos**
   - Navigate to cache directory
   - Tool: `bash` with `git rev-parse HEAD`
   - Record: Repository name and commit hash

### Artifacts Created
- External source file list
- Commit hashes for each external repository
- Evidence mapping for external sources

### Key Pattern
> External sources often provide foundational philosophy or cross-cutting concepts that are synthesized into specific project documents

---

## Phase 5: Circular Reference Detection

**Purpose**: Ensure source tree is a DAG (directed acyclic graph) with no cycles

### Steps

1. **Identify documents that reference the target**
   - Search: `grep -r "target-doc-path"` in repository
   - Common locations: README.md, index files, navigation docs
   - Record: Files that link TO the target document

2. **Analyze abstraction hierarchy**
   - Question: Is this referencing document higher or lower level?
   - Higher level: Project overviews, READMEs, index pages
   - Lower level: Detailed specs, implementation guides, API docs
   - Rule: Higher-level docs CANNOT be sources for lower-level docs

3. **Exclude circular references**
   - If document A links to target document B:
     - A is higher in tree
     - A abstracts/summarizes content from B
     - A CANNOT be a source for B (would create cycle)
   - Remove from source list with documented reasoning

4. **Build dependency DAG**
   - Leaf nodes: Source code, specs, foundational docs
   - Mid-level nodes: Explanatory docs, synthesis docs
   - Top-level nodes: READMEs, overviews, navigation

### Artifacts Created
- List of documents that reference the target (excluded as sources)
- Dependency tree visualization
- Rationale for exclusions

### Validation Rules
- **No cycles**: Follow links from target → should never return to target
- **Directionality**: Sources are at same level or lower in abstraction tree
- **Self-reference**: Target document cannot be its own source

### Decision Logic

```yaml
if document_links_to_target(file):
  if is_higher_level(file, target):
    exclude_with_reason: "Higher in abstraction tree - would create cycle"
  else:
    include: "Lower level detail being referenced"
```

---

## Phase 6: Source Tree Documentation

**Purpose**: Create machine-readable record of source dependencies

### Steps

1. **Create source tree YAML file**
   - Location: `.doc-gen/source-trees/{DOC_NAME}.yaml`
   - Structure:
     ```yaml
     document:
       path: docs/target.md
       type: mid-level-synthesis | leaf-node | top-level-overview
       purpose: "Brief description"
       scope: "What it covers"
       referenced_by: [list of higher-level docs]
       references_to: [list of docs it links to]
     
     sources:
       - file: path/to/source.md
         repository: repo-name
         url: https://github.com/org/repo/blob/main/path
         commit: full-commit-hash
         type: leaf | mid-level
         contributes: [list of sections or concepts]
     
     circular_analysis:
       excluded: [list of files excluded with reasons]
       
     dependency_tree: |
       [ASCII visualization]
     
     validation_rules:
       - rule: "Description"
     ```

2. **Document exclusion rationale**
   - For each excluded file: Why it was removed
   - For each included file: Why it was kept
   - Cross-reference decisions

### Artifacts Created
- `.doc-gen/source-trees/{DOC_NAME}.yaml`
- Permanent record for future analysis
- Reference for validating other document dependencies

---

## Phase 7: Document Regeneration (Optional but Recommended)

**Purpose**: Validate sources are sufficient by regenerating the document

### Steps

1. **Generate new document from sources only**
   - Read all approved source files
   - Synthesize content following original structure
   - Constraint: ONLY use material from approved sources
   - Output: `.doc-gen/staging/docs/{DOC_PATH}`

2. **Compare to original**
   - Tool: `diff` or manual comparison
   - Check: What was added that wasn't in original?
   - Check: What was removed from original?
   - Analyze: Do differences make sense?

3. **Iterate if needed**
   - If critical content missing: Add source or note as unique synthesis
   - If inappropriate content added: Remove or find proper source
   - Goal: Alignment with original intent, not verbatim replication

### Artifacts Created
- `.doc-gen/staging/docs/{DOC_PATH}` - Regenerated document
- Comparison analysis (what changed and why)

### Validation Checks
- No invented content (all traceable to sources)
- No missing critical sections
- Proper attribution via sources

---

## Phase 8: Outline Generation

**Purpose**: Create structured JSON outline mapping sections to sources

### Steps

1. **Read outline examples**
   - Files: `.doc-gen/examples/sample-outline.json`, `hooks_outline.json`
   - Understand: Required fields, nesting structure, source format
   - Note: Meta configuration, document structure, section schema

2. **Extract heading structure**
   - Tool: `grep "^#" {doc}` to get all headings
   - Parse: H1, H2, H3, etc. levels
   - Output: Hierarchical section list

3. **Map sections to sources**
   - For each section:
     - Identify which source files contain relevant content
     - Determine what each source contributes
     - Write reasoning for source inclusion
   - Use evidence from Phase 3 analysis

4. **Create section prompts**
   - For each section:
     - What should this section accomplish?
     - What specific content from sources should be included?
     - What style/approach (technical, explanatory, etc.)?
   - Keep prompts focused and specific

5. **Get commit hashes**
   - Current repo: `git rev-parse HEAD`
   - External repos: Navigate to cache, get HEAD
   - Record: Full 40-character commit hash for each repo

6. **Build outline JSON**
   - Structure follows examples
   - Meta section: name, model, temperature, max_tokens, document_instruction
   - Document section: title, output path, sections array
   - Each section: heading, level, prompt, sources array, nested sections
   - Each source: file (GitHub URL), reasoning, commit

7. **Validate outline**
   - Check: Valid JSON
   - Check: All required fields present
   - Check: Commit hashes match current state
   - Check: All source files exist at specified commits
   - Check: No circular references in sources

### Artifacts Created
- `.doc-gen/staging/outlines/{doc_name}_outline.json`
- Complete outline ready for doc-gen tool

### Validation Commands

```bash
# Validate JSON structure
python3 -m json.tool outline.json > /dev/null && echo "✓ Valid"

# Count sections
python3 << 'EOF'
import json
with open('outline.json') as f:
    outline = json.load(f)
def count(sections):
    return len(sections) + sum(count(s.get('sections', [])) for s in sections)
total = count(outline['document']['sections'])
print(f"Total sections: {total}")
EOF

# Validate commits
git rev-parse HEAD  # Should match core sources
cd ~/.amplifier/cache/amplifier-foundation-* && git rev-parse HEAD  # Should match foundation sources

# Verify files exist
for file in $(jq -r '.document.sections[].sources[].file' outline.json | grep github.com | sed 's|.*/blob/[^/]*/||'); do
  [ -f "$file" ] && echo "✓ $file" || echo "✗ $file MISSING"
done
```

---

## Key Insights & Patterns

### 1. Evidence-Based Source Identification

Don't guess sources - read everything and find evidence:
- Verbatim text matches
- Structural similarities
- Concept alignment
- Terminology usage

### 2. Cross-Repository Synthesis

Explanatory documents often synthesize from multiple repos:
- Current repository (specific implementation)
- Foundation bundles (philosophy, patterns)
- External dependencies (frameworks, standards)

**Discovery method**: Check loaded context for `@bundle:path` references

### 3. Dependency Tree (DAG) Validation

Documents exist in a hierarchy:
```
Top-level (README.md, index pages)
    ↓ reference/abstract
Mid-level (DESIGN_PHILOSOPHY.md, explanatory docs)
    ↓ reference/derive
Leaf-level (Specs, code, contracts)
```

**Critical rule**: Information flows upward, references flow downward

### 4. Source Tree as Artifact

The source tree YAML serves multiple purposes:
- Documents decisions for future reference
- Enables validation of other documents (check for cycles)
- Provides regeneration roadmap
- Captures institutional knowledge

---

## Inputs Required

### Required
- **Target document path**: Full path to document to generate outline for
- **Working directory**: Repository containing the target document

### Optional
- **Additional repositories**: Paths or bundle names to check for sources
  - Default: Check loaded context for external bundles
  - Example: `amplifier-foundation`, `amplifier-module-*`
- **Scope constraints**: Limit source file types or directories
  - Example: "Only use docs/ and context/, not code files"

---

## Outputs Produced

### 1. Source Analysis (Optional)
- **Path**: `.doc-gen/doc-analysis/{DOC_NAME}_SOURCE_ANALYSIS.md`
- **Content**: Detailed evidence of source file identification
- **Use**: Reference for understanding sourcing decisions

### 2. Source Tree (Required)
- **Path**: `.doc-gen/source-trees/{DOC_NAME}.yaml`
- **Content**: 
  - Document metadata
  - Final approved source list with URLs and commits
  - Circular reference analysis
  - Dependency tree visualization
  - Validation rules
- **Use**: 
  - Prevent circular references when analyzing other docs
  - Regeneration roadmap
  - Decision audit trail

### 3. Staged Document (Optional)
- **Path**: `.doc-gen/staging/docs/{DOC_PATH}`
- **Content**: Regenerated document from sources only
- **Use**: Validate sources are sufficient

### 4. Outline JSON (Required)
- **Path**: `.doc-gen/staging/outlines/{doc_name}_outline.json`
- **Content**:
  - Meta: model, temperature, document instruction
  - Document: title, output path, hierarchical sections
  - Sections: heading, level, prompt, sources with reasoning
  - Commits: Full hashes for all repositories
- **Use**: Input to doc-gen tool for regeneration

---

## Decision Framework

### When to Include a Source File

**Include if ANY of these are true:**
- ✅ Contains verbatim text that appears in target document
- ✅ Defines concepts/terms used in target document
- ✅ Provides examples or patterns referenced in target
- ✅ Contains philosophical foundation the target builds on
- ✅ Specifies contracts/APIs the target discusses

**Exclude if:**
- ❌ Only tangentially related to topic
- ❌ Referenced BY target (higher in abstraction tree)
- ❌ Is the target document itself
- ❌ Contains content not reflected in target

### Repository Classification

**Current Repository** (always check):
- All files in working directory
- Evidence: Direct content matches

**External Repositories** (check if loaded):
- Bundles in `~/.amplifier/cache/`
- Foundation/framework bundles
- Evidence: Philosophical/pattern matches

**How to find external repos**:
1. Check loaded context at conversation start for `@bundle:path` references
2. Look in `~/.amplifier/cache/` for bundle directories
3. Check `bundle.md` for `includes:` section

### Circular Reference Test

```python
def is_circular_reference(potential_source, target):
    """Check if using potential_source would create a cycle."""
    
    # Does potential_source link TO target?
    if links_to(potential_source, target):
        # Is potential_source higher abstraction level?
        if is_higher_level(potential_source, target):
            return True  # CIRCULAR - don't use
    
    return False  # Safe to use

def is_higher_level(doc_a, doc_b):
    """Determine if doc_a is higher in abstraction hierarchy."""
    
    # README, index, overview docs are typically highest
    if doc_a in ["README.md", "index.md", "overview.md"]:
        return True
    
    # Docs that link to other docs are typically higher
    if links_to(doc_a, doc_b):
        return True
    
    # Path depth can indicate level (fewer dirs = higher)
    return path_depth(doc_a) < path_depth(doc_b)
```

---

## Validation Checks

### Pre-Outline Validation

**Source file existence**:
```bash
# Verify all identified sources exist
for file in $(cat source-list.txt); do
  [ -f "$file" ] && echo "✓ $file" || echo "✗ MISSING: $file"
done
```

**Commit hash accuracy**:
```bash
# Current repo
git rev-parse HEAD

# External repo
cd ~/.amplifier/cache/{bundle}-{hash} && git rev-parse HEAD
```

**Circular reference check**:
```bash
# Find files that reference target
grep -r "target-doc-path" . --include="*.md"

# For each result, check if it's higher level
# Exclude if it is
```

### Post-Outline Validation

**JSON structure**:
```bash
# Validate JSON
python3 -m json.tool outline.json > /dev/null
```

**Required fields**:
```python
# Check all sections have required fields
import json
with open('outline.json') as f:
    outline = json.load(f)

required = ['heading', 'level', 'prompt', 'sources']
for section in walk_sections(outline['document']['sections']):
    for field in required:
        assert field in section, f"Missing {field} in {section['heading']}"
```

**Commit accuracy**:
```bash
# Extract unique commits
jq -r '.. | .commit? | select(.)' outline.json | sort -u

# Verify each commit exists
for commit in $(jq -r '.. | .commit? | select(.)' outline.json | sort -u); do
  git cat-file -e $commit 2>/dev/null && echo "✓ $commit" || echo "✗ $commit"
done
```

---

## File Organization Pattern

```
.doc-gen/
├── config.yaml                          # doc-gen tool config
├── cache/                               # Outline storage (managed by tool)
├── examples/                            # Example outlines for reference
│   ├── sample-outline.json
│   └── hooks_outline.json
├── doc-analysis/                        # Source discovery artifacts
│   └── {DOC_NAME}_SOURCE_ANALYSIS.md
├── source-trees/                        # Dependency DAGs
│   └── {DOC_NAME}.yaml
└── staging/
    ├── docs/                            # Regenerated documents
    │   └── {DOC_PATH}
    └── outlines/                        # Generated outlines
        └── {doc_name}_outline.json
```

---

## Process Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│ PHASE 1: Document Analysis                                  │
├─────────────────────────────────────────────────────────────┤
│ Input: Target document path                                 │
│ 1. Read target document                                     │
│ 2. Extract essence, purpose, scope                          │
│ Output: Document summary                                    │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────────┐
│ PHASE 2: Repository Content Scan                            │
├─────────────────────────────────────────────────────────────┤
│ 1. Find all files in repository                             │
│ 2. Read all files systematically                            │
│ Output: Complete repo content in context                    │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────────┐
│ PHASE 3: Source Discovery & Evidence Analysis               │
├─────────────────────────────────────────────────────────────┤
│ 1. Compare doc content to all repo files                    │
│ 2. Identify evidence-based sources                          │
│ 3. Create initial source list with confidence levels        │
│ Output: Source file list with evidence                      │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────────┐
│ PHASE 4: Cross-Repository Discovery                         │
├─────────────────────────────────────────────────────────────┤
│ 1. Check loaded context for external bundles                │
│ 2. Read external source files from cache                    │
│ 3. Get commit hashes for each repository                    │
│ Output: External source files with commits                  │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────────┐
│ PHASE 5: Circular Reference Detection                       │
├─────────────────────────────────────────────────────────────┤
│ 1. Find documents that reference target                     │
│ 2. Analyze abstraction hierarchy                            │
│ 3. Exclude higher-level docs (prevent cycles)               │
│ 4. Build dependency DAG                                     │
│ Output: Validated source list (acyclic)                     │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────────┐
│ PHASE 6: Source Tree Documentation                          │
├─────────────────────────────────────────────────────────────┤
│ 1. Create source tree YAML                                  │
│ 2. Document exclusions and inclusions                       │
│ 3. Record dependency structure                              │
│ Output: .doc-gen/source-trees/{DOC_NAME}.yaml               │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────────┐
│ PHASE 7: Document Regeneration (Optional)                   │
├─────────────────────────────────────────────────────────────┤
│ 1. Generate document from sources only                      │
│ 2. Compare to original                                      │
│ 3. Iterate if needed                                        │
│ Output: .doc-gen/staging/docs/{DOC_PATH}                    │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────────┐
│ PHASE 8: Outline Generation                                 │
├─────────────────────────────────────────────────────────────┤
│ 1. Read outline examples                                    │
│ 2. Extract heading structure                                │
│ 3. Map sections to sources                                  │
│ 4. Create section prompts                                   │
│ 5. Get commit hashes                                        │
│ 6. Build outline JSON                                       │
│ 7. Validate outline                                         │
│ Output: .doc-gen/staging/outlines/{doc}_outline.json        │
└─────────────────────────────────────────────────────────────┘
```

---

## Recipe Input/Output Contract

### Recipe Inputs

```yaml
context:
  target_document: ""           # Required: e.g., "docs/DESIGN_PHILOSOPHY.md"
  additional_repos: []          # Optional: ["amplifier-foundation", "other-bundle"]
  include_regeneration: true    # Optional: Whether to regenerate doc for validation
  outline_only: false           # Optional: Skip source tree creation
```

### Recipe Outputs

**Always created**:
1. `.doc-gen/source-trees/{DOC_NAME}.yaml` - Source dependency tree
2. `.doc-gen/staging/outlines/{doc_name}_outline.json` - Complete outline

**Conditionally created**:
3. `.doc-gen/doc-analysis/{DOC_NAME}_SOURCE_ANALYSIS.md` - If detailed analysis requested
4. `.doc-gen/staging/docs/{DOC_PATH}` - If `include_regeneration: true`

---

## Success Criteria

The process is complete when:

✅ **Source tree YAML exists** with all required sections  
✅ **No circular references** in source list  
✅ **All sources have commit hashes** (validated to exist)  
✅ **Outline JSON is valid** and has all required fields  
✅ **All source files exist** at specified commits  
✅ **Section count matches** document heading count  
✅ **Each section has sources** with reasoning  
✅ **Commits are current** (match HEAD of each repo)  

---

## Error Handling

### Missing Source Files
- **Symptom**: Source file doesn't exist at commit
- **Cause**: File was moved, renamed, or deleted
- **Fix**: Find current location or remove from sources

### Circular References Detected
- **Symptom**: Document A sources B, B sources A
- **Cause**: Incorrect abstraction hierarchy analysis
- **Fix**: Determine which is higher level, break cycle

### External Repo Not Found
- **Symptom**: Can't find bundle in cache
- **Cause**: Bundle not loaded, different bundle name
- **Fix**: Check bundle.md includes, verify cache directory

### Content Not Traceable to Sources
- **Symptom**: Document has content not in any source
- **Cause**: Unique synthesis or missing source
- **Fix**: Note as "original synthesis" or find missing source

---

## Notes for Recipe Author

### Agent Selection

**Recommended agent**: `foundation:explorer` or similar file-scanning agent
- Needs ability to read many files efficiently
- Should support parallel file reading
- Must be able to analyze text for similarities

### Batch Operations

**Optimize for**:
- Parallel file reads (Phase 2, 4)
- Single-pass analysis where possible
- Caching read content for reuse

### Conditional Logic

**Key decision points**:
- Should we scan external repos? (check loaded context)
- Is regeneration needed? (validation vs. speed trade-off)
- Create detailed analysis? (debug artifact vs. minimal)

### State Management

**Track between phases**:
- Source file list (builds incrementally)
- Evidence mappings (section → sources)
- Exclusion rationale (why files were removed)
- Commit hashes (per repository)

### User Interaction Points

**Potential confirmations**:
- Approve final source list before outline generation
- Review circular reference exclusions
- Validate regenerated document before using for outline

---

## Example Recipe Context

```yaml
context:
  # Required
  target_document: "docs/DESIGN_PHILOSOPHY.md"
  
  # Optional - auto-detect from loaded context if not specified
  additional_repos:
    - name: "amplifier-foundation"
      path: "~/.amplifier/cache/amplifier-foundation-*"
  
  # Optional - defaults
  include_regeneration: true
  create_analysis_doc: false
  outline_name: "design-philosophy-outline"
```

---

## Process Variations

### Minimal (Fast)
1. Document Analysis
2. Repository Scan
3. Source Discovery
4. Outline Generation (skip tree, skip regeneration)

**Use when**: Quick outline for simple documents

### Standard (Recommended)
1. Document Analysis
2. Repository Scan  
3. Source Discovery
4. Cross-Repo Discovery
5. Circular Reference Detection
6. Source Tree Documentation
7. Outline Generation

**Use when**: Most documents, ensures quality

### Comprehensive (Thorough)
1. All Standard phases
2. Plus: Document Regeneration
3. Plus: Detailed Source Analysis artifact

**Use when**: Complex documents, multiple repos, need validation

---

## Lessons Learned from Session

### 1. Read Everything First
- Don't assume sources - discover them through evidence
- Reading all repo files enables accurate source identification
- Parallel reads make this feasible

### 2. External Sources Are Common
- Explanatory docs often synthesize across repos
- Foundation bundles provide philosophy/patterns
- Check loaded context for `@bundle:` references

### 3. Circular References Are Real
- README.md often abstracts from detailed docs
- Test: Does file A link to target? → A is higher → Can't be source
- Document the DAG to prevent issues

### 4. Source Tree Is Critical Artifact
- Prevents circular references in future analysis
- Documents "why" for decisions
- Enables regeneration and validation

### 5. Commit Hashes Must Be Accurate
- Get from current HEAD, not guessed
- Validate all files exist at specified commits
- Handle multi-repo commits separately

---

## Next Steps for Recipe Creation

1. **Extract the exact process** from this document into recipe stages
2. **Map each phase** to appropriate agents and tools
3. **Add validation checks** between stages
4. **Handle conditional logic** (external repos, regeneration)
5. **Create approval gates** where user input is valuable
6. **Test with different document types** (API docs, specs, explanatory)
