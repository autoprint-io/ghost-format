# JOOT Format Specification

**Version:** 0.1 (Draft)
**Status:** Working Draft
**Date:** 2026-04-17
**Extension:** `.joot`
**Media Type:** `application/joot` (provisional, unregistered)
**Encoding:** UTF-8, no BOM
**Line Endings:** `LF` (`CRLF` accepted by consumers, normalized on read)

---

## Status of This Specification

This document is a Working Draft of the JOOT Format Specification, version 0.1.

It is a work in progress and MAY change substantially before reaching Candidate or Stable
status. Implementers SHOULD NOT treat this document as a stable standard.

A future revision will be promoted to Candidate status once:

- the normative grammar in Appendix A is fully aligned with the main text;
- at least two independent conforming implementations exist;
- a conformance test suite (§25) has been published and is passing against those
  implementations;
- all profiles designated Standard by this revision have complete normative addenda.

This document is maintained by the JOOT editors. Feedback and proposed changes are welcome.

---

## Normative Language and Terminology

### Requirement Levels

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**,
**SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be
interpreted as described in RFC 2119 and RFC 8174 when, and only when, they appear in all
capitals.

When used in lowercase, these words carry no special meaning and are used only as ordinary
English.

### Definitions

The following terms have precise meanings throughout this specification.

**Block.** A syntactic region of an entry delimited by a named marker (`[Rel]`, `[Fields]`,
`[Behavior]`, `[Returns]`) or by position (the meta block and annotations).

**Consumer.** A program that reads and interprets a JOOT document.

**Corpus.** A set of JOOT documents treated as a single validation scope. Identity uniqueness
and target resolution can be defined at the corpus level. See §12.5.

**Dialect.** A named vocabulary extension that adds types, meta keys, categories, relation
roles, or behavior atoms to JOOT. A dialect does not relax the core grammar. See §16.

**Document.** A single `.joot` file, or an equivalent in-memory representation.

**Entry.** A single logical unit of JOOT content, written as one physical line, representing
one addressable subject or one facet of a subject. See §6.

**Facet.** An orthogonal projection of the same logical subject, addressed by a `facet:` meta
key. Facets share one identity. See §12.3.

**Identity.** The logical address of a subject, denoted by the `id:` meta key, unique within
a corpus under profile conformance. See §12.

**Meta Block.** The unmarked region of an entry between the entry name and the first named
block, containing key-value classification fields such as `cat:`, `id:`, `kind:`. See §6.3.

**Processor.** A program that performs any operation on JOOT content: parsing, validating,
querying, transforming, emitting.

**Producer.** A program that generates JOOT content.

**Profile.** A named governance layer declared in the header, which closes vocabularies and
enforces strict validation rules. See §15.

**Relation.** An edge from the current subject to another first-class subject, declared in
the `[Rel]` block. See §6.4.

**Sigil.** The single-character prefix of an entry name that classifies the kind of subject.
See §5.

**Subject.** A first-class addressable entity represented by one or more JOOT entries sharing
the same `id:`.

**Validator.** A processor that verifies conformance of a document or corpus and reports
errors and warnings.

### Reserved Terms

The following terms are reserved by this specification and MUST NOT be redefined by profiles
or dialects:

- the meta keys listed in §6.3.4
- the named block markers `[Rel]`, `[Fields]`, `[Behavior]`, `[Returns]`
- the annotation prefixes `#note:` and `#warn:`
- the sigil characters listed in §5.1
- the terms "profile", "dialect", "corpus", "facet", "identity", and "subject" as defined
  above

---

## Abstract

JOOT is a line-oriented, pipe-delimited plain-text format for encoding structured technical
knowledge — API contracts, code surfaces, database schemas, workflow definitions, platform
metamodels, and documentation references — in a form optimized for consumption by Large
Language Models and autonomous agents.

JOOT represents one addressable subject per physical line. Each line contains classification
metadata, graph relations to other subjects, structural fields, behavioral properties, and
declared return or output shapes, separated by pipe characters and grouped into canonically
ordered blocks. The format prohibits nesting of named blocks, hard-wrapped lines, and
ad-hoc block inventions. Every element of syntax is designed to maximize token density and
positional regularity while preserving semantic fidelity.

JOOT defines three layers:

- a minimal closed syntax (Layer 0);
- an explicit semantic model comprising identity, graph, and core vocabularies (Layer 1);
- an optional governance layer of profiles and dialects that close vocabularies and enforce
  validation (Layer 2).

A document conforming only to Layer 0 is valid and readable. A document declaring a profile
activates strict validation suitable for machine-first interchange, code generation, and
agent orchestration.

JOOT is designed as an interchange format. It is intended to be produced from and emitted to
heterogeneous technical representations — OpenAPI, JSON Schema, SQL DDL, programming language
source code, Markdown documentation — acting as a canonical intermediate representation for
semantic technical surfaces across ecosystems.

---

## Scope and Non-Goals

### In Scope

JOOT is designed to represent:

1. **API surfaces.** HTTP endpoints, RPC methods, webhook contracts, request and response
   shapes, error surfaces, headers, authentication schemes, and rate limits.

2. **Code surfaces.** Modules, namespaces, packages, types (classes, interfaces, structs,
   enums, traits, protocols), members (fields, properties, constants), and callables
   (functions, methods, constructors, hooks, operators) from any statically or dynamically
   typed language.

3. **Data schemas.** Database tables, views, columns, indexes, constraints, foreign keys,
   and type definitions for relational, document, and graph stores.

4. **Workflow and protocol surfaces.** States, transitions, events, steps, guards, and edges
   in state machines, workflows, and protocol specifications.

5. **Platform metamodels.** Reverse-engineered surfaces of platforms, frameworks, and
   applications, including component libraries, plugin interfaces, and extension points.

6. **Documentation references.** Vendor documentation, SDK references, CLI surfaces, and
   configuration specifications represented as structured addressable subjects.

### Explicitly Out of Scope

JOOT is not designed to represent, replace, or compete with:

1. **Executable code.** JOOT describes surfaces, not implementations. It captures contracts,
   signatures, and structure, not algorithmic behavior beyond declared effects.

2. **Binary or non-textual data.** JOOT is strictly UTF-8 text. Binary payloads, images,
   audio, and non-textual artifacts MUST be represented by reference (URL, hash, or external
   identifier), not inlined.

3. **Free-form prose documentation.** JOOT is not a prose format. Explanatory notes use
   `#note:` and `#warn:` annotations, which are constrained single-line atoms. Long-form
   documentation MUST live outside JOOT.

4. **Configuration files for runtime systems.** JOOT describes configuration shapes, not
   runtime configuration values. A JOOT schema may describe what a valid `.env` or
   `config.yaml` looks like, but JOOT itself is not that file.

5. **Agent runtime state.** Agent runtime state, conversation histories, and tool invocation
   logs are out of scope.

6. **Exchange of arbitrary structured data.** JOOT is not a general-purpose alternative to
   JSON, YAML, or CBOR. It is constrained to the domain of technical surface representation.

### Relationship to LLM Context Windows

JOOT is optimized for, but not exclusive to, LLM and agent consumption. A design principle
is that large corpora are substantially more context-efficient than equivalent JSON, YAML,
or Markdown representations. This is a design pressure that shapes every syntactic decision,
not a guaranteed compression ratio.

Human readability is a secondary but maintained concern. A JOOT file is legible without
tooling, grep-friendly, and diff-stable.

---

## Design Principles

JOOT is shaped by ten principles. Every rule in this specification derives from one or more
of them. When a design question arises that the specification does not answer, implementers
and contributors should resolve it by appeal to these principles, in their declared order.

### P1 — Token density is a first-order goal.

Every byte in a JOOT file must justify itself semantically. Decorative punctuation, redundant
keys, nested syntax, and repetition of structural information are rejected. When two forms
express the same meaning, the shorter form is canonical.

### P2 — No loss of technical meaning.

Density does not come at the cost of precision. Fields, types, constraints, visibility,
effects, cardinality, preconditions, postconditions, graph relations, deprecation, and
provenance must all be representable without ambiguity. The format trades human verbosity for
machine clarity, never semantic fidelity.

### P3 — One entry per physical line.

A single addressable subject (or one facet of a subject) occupies exactly one physical line.
Hard wraps are forbidden. This guarantees greppability, diff stability, streaming
processability, and positional retrieval by LLMs.

### P4 — Canonical ordering everywhere.

Positional regularity improves machine retrieval. Blocks, meta keys, relation items, fields,
and annotations all have canonical orderings defined in the specification. Producers SHOULD
emit canonical order; machine-first validators MUST enforce it.

### P5 — LLMs and agents are first-class consumers.

The format is designed with LLMs and autonomous agents as primary consumers, and human
readers as maintained secondary consumers. Where a choice exists between a shorter syntax
that is hard to parse and a slightly longer syntax that is immediately clear to a reader or
model, the latter wins. Token density never trumps comprehensibility.

### P6 — Core grammar is tiny and closed. Extensibility lives in profiles and dialects.

The syntactic core of JOOT (Layer 0) does not grow. New vocabulary, new categories, new
relation roles, and new constraint atoms are introduced through profiles and dialects, never
through new syntax.

### P7 — Graph edges are first-class citizens.

Relations between subjects are not hidden in prose, behavior descriptors, or metadata
comments. They live in a dedicated `[Rel]` block with typed roles, resolvable targets, and
optional cardinality. This allows machine-readable graph construction from any JOOT corpus.

### P8 — Identity is explicit, stable, and corpus-unique.

Every addressable subject under a profile has a canonical identifier. Two entries describing
the same subject share that identifier. Facets of the same subject share the same identifier
and differ only by facet name. Identity enables cross-reference, deduplication, and
cross-corpus interoperability.

### P9 — Governance is opt-in.

A document without a declared profile is validated leniently under core conformance rules.
A document that declares a profile activates strict machine-first validation, vocabulary
closure, and identity enforcement. This allows casual and exploratory use of JOOT alongside
production-grade standardized corpora.

### P10 — Ambiguities must be resolved by the specification.

Where the specification leaves a choice, two independent implementations will diverge.
Divergence kills interoperability. Every design decision that affects conformance —
ordering, resolution, normalization, duplicate handling, conflict resolution — is to be
defined normatively in this document. When v0.1 leaves an ambiguity open, it is flagged
explicitly; no ambiguity reaches Stable status unresolved.

---

## Relationship to Other Formats

JOOT occupies a specific layer in the ecosystem of technical data formats.

### JOOT is NOT

- **A replacement for JSON or YAML.** JSON and YAML are general-purpose data serialization
  formats. JOOT is a specialized format for semantic technical surfaces.

- **A replacement for OpenAPI or JSON Schema.** OpenAPI and JSON Schema are dominant formats
  for API and data contract specification in their respective ecosystems. JOOT is designed
  to interoperate with them, not to displace them.

- **A replacement for Markdown documentation.** Markdown remains the correct format for
  long-form prose, tutorials, and narrative explanations. JOOT captures the structured,
  addressable, machine-queryable surfaces that often lie *behind* Markdown documentation.

- **A programming language or DSL.** JOOT does not express control flow, computation, or
  runtime semantics. It is a static declarative format.

### JOOT IS

- **A canonical intermediate representation for technical surfaces.** JOOT is designed to be
  a common target for ingestion from heterogeneous sources and a common source for emission
  to generators.

- **A context-efficient format for LLMs and agents.** JOOT's density, positional regularity,
  and graph-first structure are optimized for in-context reasoning about technical surfaces.

- **A graph-oriented semantic model.** Every JOOT corpus is implicitly a directed graph of
  subjects linked by typed relations.

- **A governance-capable standard.** Through profiles, JOOT supports strict machine-first
  validation, closed vocabularies, identity enforcement, and conformance testing.

### Interoperability with OpenAPI and JSON Schema

JOOT is intended to represent a substantial overlapping subset of the content expressible in
OpenAPI and JSON Schema. Companion mapping documents MAY define bidirectional mappings for
those subsets. Full roundtrip fidelity is not guaranteed by this specification and depends
on the JOOT profile and the features used in the source document. Detailed mapping guidance
is out of scope for this core specification.

A comparative analysis with adjacent formats is provided informatively in Appendix D.

---

# Part I — Processing Model

---

## 1. Processing Model

This section defines what a JOOT processor is, what operations it performs, and what
conformance levels apply to its outputs. It does not prescribe implementation details; it
establishes the conceptual contract that any processor MUST satisfy.

### 1.1 What a JOOT Processor Does

A JOOT processor is any program that performs one or more of the following operations on
JOOT content:

1. **Parse.** Read a UTF-8 byte sequence and produce an in-memory representation of the
   document's structure (header, sections, entries, blocks, fields, annotations).
2. **Validate.** Check a parsed document against the rules of this specification, optionally
   against a declared profile, and produce a set of errors and warnings.
3. **Query.** Retrieve subjects, relations, fields, or facets from a parsed document or
   corpus.
4. **Transform.** Produce a new JOOT document or corpus derived from an existing one
   (normalization, facet splitting, profile conversion).
5. **Emit.** Serialize an in-memory representation back to a valid `.joot` byte sequence.
6. **Ingest from non-JOOT sources.** Convert external formats (OpenAPI, JSON Schema, source
   code, SQL DDL) into JOOT documents.
7. **Export to non-JOOT formats.** Generate external artifacts (type bindings, documentation,
   schema files) from JOOT content.

A processor is not required to implement all operations. A minimally useful processor
implements parse and validate.

### 1.2 Conformance Layers

JOOT defines three conformance layers. Each layer builds on the previous one. A document or
processor declares which layer applies and is judged accordingly.

#### 1.2.1 Core Conformance

**Core conformance** is the minimal level. A document is core-conformant if it satisfies:

- the file structure rules in §4 (header, sections, blank lines, one entry per line);
- the entry grammar in §6 (sigils, canonical block order, meta block present with `cat:`
  first);
- the escaping and quoting rules in §9;
- the annotation ordering rule in §10.3;
- the reserved sigil prohibition in §5.3;
- the naming conventions in §11.

Core conformance does NOT require:

- the presence of `id:` or `kind:` on entries;
- closed vocabularies for `cat:`, relation roles, or behavior atoms;
- resolution of relation targets;
- corpus-level identity uniqueness.

A core-conformant document is valid JOOT but uses open vocabularies. It is suitable for
exploratory authoring, human-facing references, and cases where strict interoperability is
not required.

#### 1.2.2 Profile Conformance

**Profile conformance** applies when a document declares `profile=<name>` in its header. A
profile-conformant document MUST satisfy core conformance PLUS all additional requirements
defined by the declared profile, including:

- presence of `id:` and `kind:` on every entry;
- canonical ordering of meta keys and relation items;
- closed vocabularies (only profile-registered `cat:`, `kind:`, relation roles, behavior
  atoms, ext meta keys are permitted);
- profile-specific decomposition rules;
- profile-specific relation source/target type compatibility;
- identity uniqueness of `(id, facet)` tuples within the document.

A profile MUST define its complete set of additional requirements in its normative addendum
(§15). A validator MUST NOT apply rules that are not in the declared profile's addendum.

#### 1.2.3 Corpus Conformance

**Corpus conformance** applies when a set of JOOT documents is validated as a single scope.
A corpus-conformant document set MUST satisfy profile conformance for each document PLUS:

- identity uniqueness of `(id, facet)` tuples across the entire corpus;
- resolution of all internal relation targets to existing subjects in the corpus;
- preservation of `cat:` and `kind:` across all facets sharing the same `id:`.

Corpus conformance is typically validated by a corpus-aware validator with access to all
documents in the set. A single document validated in isolation cannot satisfy corpus
conformance.

### 1.3 Validation Modes

A validator operates in one of four modes. The mode determines which checks apply and how
unresolved references are reported.

1. **Core mode.** The document is validated against core conformance rules only. No profile
   rules are applied, even if `profile=` is declared.
2. **Profile mode.** The document is validated against the declared profile's rules in
   addition to core rules. Internal relation targets that cannot be resolved within the
   document produce warnings, not errors.
3. **Corpus mode.** Multiple documents are validated together against the declared profile.
   Internal relation targets must resolve within the corpus; unresolved targets produce
   errors.
4. **Structural mode.** The validator applies only physical-file and syntactic checks (file
   layout, blank lines, header, entry line structure, pipe and comma separators, escaping).
   No semantic checks, no vocabulary checks, no identity checks, no resolution. Useful for
   partial or work-in-progress documents and for tooling that only needs to confirm a file
   is syntactically well-formed JOOT.

A validator MUST document which mode it is operating in. A validator SHOULD default to
profile mode when a profile is declared, and to core mode otherwise.

### 1.4 Producer and Consumer Roles

The specification distinguishes producers and consumers because their obligations differ.

**Producer obligations:**

- emit canonical ordering wherever defined by the specification;
- include `id:` and `kind:` when emitting profile-conformant content;
- ensure header line 4 entry count matches actual entry lines;
- escape or quote reserved characters per §9;
- avoid ad-hoc blocks, reserved sigils, and forbidden facet patterns.

**Consumer obligations:**

- accept both `LF` and `CRLF` line endings, normalizing on read;
- accept both quoted and unquoted equivalent values;
- tolerate non-canonical ordering in core mode (producing warnings, not errors);
- tolerate unknown types in core mode (producing warnings, not errors);
- follow the error recovery rules in §19.5.

The asymmetry is intentional: producers are strict to guarantee interoperability; consumers
are lenient to tolerate drafts, partials, and third-party content.

### 1.5 Lenient vs Strict Consumption

JOOT's design privileges graceful consumption. A consumer encountering a syntactically
well-formed document that violates semantic or vocabulary rules SHOULD continue reading and
report the violations, rather than abort. This supports:

- incremental authoring, where a document may be temporarily incomplete;
- exploratory corpora, where vocabularies are not fully registered;
- compatibility with drafts and work-in-progress content.

A consumer MAY abort on syntactic errors (malformed header, entry split across lines, tab
characters, unclosed quoted strings). A consumer SHOULD NOT abort on semantic errors
(unresolved targets, unknown vocabularies) unless explicitly configured to do so.

**Important principle.** Lenient consumption is a property of the consumer, not of the
language. JOOT syntax is strict: case is significant, canonical ordering is required where
specified, reserved characters must be escaped or quoted. A lenient consumer that recovers
from a minor error in a non-conformant document does NOT thereby make the document
conformant, and does NOT extend the language definition. Recovery behavior is opaque to
other consumers. Relying on a specific consumer's recovery behavior for interoperability is
a conformance violation.

### 1.6 Open Questions for Future Revisions

The following aspects of the processing model are deliberately left incomplete in v0.1 and
will be resolved in future revisions:

- **Streaming parsers.** Whether and how a parser can emit parsed entries incrementally
  without loading the full document is not specified. v0.2 may define streaming semantics.
- **Canonical serialization for diff and hashing.** A single normalized output form for any
  semantically equivalent input is not defined in v0.1. §19.7 sketches the requirement.
- **Cross-version resolution.** Resolving a relation target against documents declaring
  different JOOT versions is not specified.

---

## 2. Media Type and File Identification

This section defines how JOOT content is identified on disk, over a network, and in
protocol metadata.

### 2.1 Media Type

The provisional media type for JOOT content is:

```
application/joot
```

This media type is currently unregistered. Registration with IANA is planned for a future
revision. A future revision will include the formal IANA registration template.

No alternative media type is defined by this draft. Implementations MAY use private
agreements or local configuration during the pre-registration period, but such identifiers
are out of scope for conformance.

### 2.2 File Extension

The file extension for JOOT documents is:

```
.joot
```

No alternative extensions are defined. Files with other extensions MAY contain JOOT content
but do not claim conformance by extension alone.

### 2.3 Header-Based Detection (Informative)

The first header line MAY be used as an informative heuristic by editors, indexers, and
language-detection tooling. The literal substring `.joot v` near the start of a file is a
reasonable signal that the file is JOOT content. This is not a normative identification
mechanism; JOOT does not define a magic number or binary signature.

### 2.4 Encoding

A JOOT document MUST be encoded in UTF-8.

A JOOT document MUST NOT begin with a UTF-8 byte-order mark (BOM, bytes `EF BB BF`). A
consumer encountering a BOM MUST either reject the document as malformed or strip the BOM
and report a warning.

A JOOT document MUST NOT contain bytes that are not valid UTF-8. A parser encountering
invalid UTF-8 sequences MUST report a syntactic error and SHOULD abort parsing.

### 2.5 Line Endings

A JOOT document SHOULD use `LF` (`U+000A`) as its line terminator.

A consumer MUST accept `CRLF` (`U+000D U+000A`) as an equivalent line terminator and
normalize it to `LF` on read. A consumer MUST NOT accept bare `CR` (`U+000D`) as a line
terminator; a bare `CR` in the middle of a line is a syntactic error.

A producer SHOULD emit `LF` exclusively. A producer MAY emit `CRLF` when targeting a
platform that requires it, but this is discouraged for interchange.

### 2.6 Whitespace

The following whitespace rules apply **outside quoted strings and free-text content**
(§9.4, §3.3):

- Horizontal tab characters (`U+0009`) MUST NOT appear anywhere in a JOOT document. Tabs
  are a syntactic error.
- Space characters (`U+0020`) are permitted in limited positions defined by the grammar
  (specifically: after `#` in header lines and between header tokens). They are not
  syntactically meaningful between block boundaries.
- Line feed (`U+000A`) and carriage return (`U+000D`, only as part of CRLF per §2.5) are
  permitted as line terminators only.
- No other whitespace characters (vertical tab `U+000B`, form feed `U+000C`, etc.) are
  permitted.
- Leading or trailing spaces on entry lines are prohibited.

**Inside quoted strings and free-text content**, any valid UTF-8 code point is permitted
subject to the rules in §3.3 (which forbids control characters in `U+0000`–`U+001F` except
LF/CR, forbids surrogate code points, and forbids stray BOMs). Horizontal tab remains
forbidden everywhere, including inside quoted strings, because it is a hard syntactic error
per this section.

Space characters inside quoted strings are preserved literally.

### 2.7 File Size

This specification imposes no maximum file size for a JOOT document. Implementations MAY
document their own limits. A producer SHOULD split a logical corpus across multiple files
when a single file exceeds practical editor, parser, or LLM context thresholds.

---

## 3. Internationalization and Unicode

JOOT is a UTF-8 text format. This section specifies how JOOT treats Unicode content:
which code points are permitted where, how identifiers are compared, and how identity
collisions between visually similar characters are handled.

### 3.1 UTF-8 Requirement

All JOOT content MUST be valid UTF-8 (§2.4). This is the only encoding recognized by the
specification.

### 3.2 Unicode Normalization

All JOOT content SHOULD be in Unicode Normalization Form C (NFC). A producer SHOULD emit NFC.
A consumer SHOULD accept non-NFC input but MAY normalize to NFC internally for comparison
purposes.

A validator MUST NOT treat non-NFC input as a syntactic error in core mode. A machine-first
profile MAY require NFC and treat non-NFC input as a warning.

In JOOT v0.1, identifier positions are ASCII-only (see §3.3). Unicode normalization
therefore has no effect on identifier comparison in this revision. Normalization matters
only for quoted strings and free-text content.

A future revision MAY extend identifier positions to permit non-ASCII Unicode (§3.7), at
which point normalization-based identity comparison rules will be defined.

### 3.3 Permitted Code Points by Position

Different positions in a JOOT document permit different Unicode ranges. The following rules
are normative.

**In structural syntax** (sigils, block markers, pipes, commas, brackets, braces, colons,
meta keys, section names, identifiers, reserved keywords, type names):

Only ASCII printable characters (`U+0020` to `U+007E`) are permitted. Non-ASCII characters
in these positions are a syntactic error.

**In entry names** (§11.2):

ASCII letters, ASCII digits, underscore (`_`), and the hierarchy separator (`>`) are
permitted. Other characters are prohibited. This restriction ensures entry names are
universally addressable, diff-stable, and free of confusable characters in contexts where
they function as identifiers.

**In field names, meta key names, relation roles, behavior atoms, and constraint atoms**:

ASCII lowercase letters, ASCII digits, and underscore (`_`) only. This is a strict subset of
the identifier grammar (§11).

**In `id:` values** (§11.3):

ASCII letters, ASCII digits, underscore (`_`), dot (`.`), hyphen (`-`), slash (`/`),
parentheses (`(`, `)`), and plus (`+`). The hash character (`#`) is explicitly forbidden
inside `id:` values; `#` is reserved as the facet separator in relation targets (§6.4.4).

**In quoted string values and free-text content** (annotation content, behavior values
when quoted, constraint values when explicitly declared as free-text):

Any valid UTF-8 code point is permitted, with the exceptions below in "Forbidden code points
everywhere". This is where internationalization matters: a human-facing label, a
documentation snippet, or an example value may contain any script (Latin, Cyrillic, CJK,
Arabic, Devanagari, etc.), any punctuation, and any Unicode symbol.

**Forbidden code points everywhere:**

- control characters in the range `U+0000` to `U+001F` except `U+000A` (LF) and `U+000D`
  (CR, only as part of a CRLF line ending);
- the tab character `U+0009` (explicit re-statement of §2.6);
- code points in the surrogate range `U+D800` to `U+DFFF`;
- the byte-order mark `U+FEFF` when it appears anywhere other than a stripped leading BOM.

### 3.4 Case Sensitivity

JOOT syntax is case-sensitive. This applies uniformly to:

- entry names (a subject named `OrderService` is distinct from `orderService`);
- `id:` values;
- nominal type names inside generics (§7.2.1);
- section names;
- meta key names (`cat:`, `id:`, `kind:` etc. MUST be lowercase as defined in §6.3.4);
- named block markers (`[Rel]`, `[Fields]`, `[Behavior]`, `[Returns]` MUST be written in
  exactly the casing shown in this specification);
- relation roles, behavior atoms, constraint atoms, and field names (all lowercase, as
  required by §11).

A meta key written as `Cat:` or `CAT:`, or a block marker written as `[REL]` or `[rel]`,
is non-conformant. It is a syntactic error reported as `SYN-010` (§19.3).

A consumer operating in structural or core mode (§1.3) MAY attempt recovery of
case-incorrect input as a best-effort behavior, reporting the recovery as a warning. Such
recovery is not equivalence: the source document remains non-conformant. Recovery is a
property of the consumer, not of the language.

### 3.5 Confusable Character Handling

The Unicode standard defines families of characters that are visually similar but
distinct (for example, Latin `a` U+0061 vs Cyrillic `а` U+0430). In identifier positions,
such confusables can create security and correctness issues.

Because JOOT restricts all identifier positions (entry names, IDs, field names, meta keys,
relation roles, etc.) to ASCII, confusable characters cannot appear in identifiers. This
eliminates the confusable-identifier attack surface entirely for JOOT v0.1.

In quoted strings and free-text content, any Unicode character is permitted, and
confusables are allowed. Consumers processing such content for display SHOULD apply the
mitigations recommended in UTS #39 (Unicode Security Mechanisms) as appropriate to their
use case. This specification does not mandate such mitigations.

### 3.6 Bidirectional Text

Bidirectional (bidi) text in quoted strings and free-text content is permitted. JOOT does
not use bidi control characters (`U+202A` through `U+202E`, `U+2066` through `U+2069`) for
syntactic purposes and does not prohibit them in free-text.

However, bidi control characters can alter the visual rendering of source code in a way that
does not match its logical byte order ("Trojan Source" attack). A producer SHOULD NOT emit
bidi control characters in JOOT content unless they are semantically required for
representing real bidirectional content. A validator MAY warn on the presence of bidi
control characters outside quoted strings.

### 3.7 Open Questions for Future Revisions

The following aspects of internationalization are left incomplete in v0.1:

- **Locale-aware collation.** Sorting JOOT entries or relation items by identifier may
  require locale rules in specific profiles. v0.1 specifies byte-wise comparison only;
  profiles MAY override.
- **Non-ASCII identifier support.** Extending identifier positions to permit a controlled
  subset of non-ASCII Unicode (for example, to support non-English source code) is deferred.
  A future revision may define an opt-in Unicode identifier mode with UTS #39 restriction
  profiles applied.

---

# Part II — Syntax (Layer 0)

---

## 4. File Structure

This section defines the physical layout of a JOOT document: the header, sections, entries,
blank-line rules, and normalization requirements. It governs what a file looks like as a
byte sequence, independent of semantic meaning.

### 4.1 Physical File Layout

A JOOT document consists of exactly three structural parts, appearing in this order:

1. a **header** of exactly four lines (§4.2);
2. a single blank line;
3. one or more **sections** (§4.3), each containing at least one **entry** (§4.4).

No other structural elements are permitted at the file level. A document MUST NOT begin with
blank lines, comments before the header, or content other than the header.

A document MUST end with either exactly one final `LF` after the last entry line, or no
trailing byte after the last entry line. See §4.5 for the full producer/consumer contract
on end-of-file handling.

### 4.2 Header

Every JOOT document MUST begin with exactly four header lines, each starting with `#`.

```
# <filename>.joot v<MAJOR>[.<MINOR>] [profile=<name>] [dialect=<name>]
# Purpose: <one-line description>
# Source: <origin or "authored">
# Entries: <integer>
```

#### 4.2.1 Header Lines

- **Line 1** — identification line. Declares the filename, version, and optionally a profile
  and/or a dialect. See §4.2.2, §4.2.3, §4.2.4.
- **Line 2** — purpose. A one-line human-readable description of the document's content.
  Free text. MUST NOT contain line breaks.
- **Line 3** — source. The origin of the content. Either a URL, a path, a citation, or the
  literal string `authored` for hand-written documents. Free text. MUST NOT contain line
  breaks.
- **Line 4** — entry count. The literal string `# Entries: ` followed by a non-negative
  integer equal to the actual count of entry lines in the document. If the counts disagree,
  the file is non-conformant.

No fifth header line is permitted. The line immediately following line 4 MUST be a blank
line (§4.5).

#### 4.2.2 Version Declaration

The version token follows the literal substring `.joot v` on line 1. Its form is:

```
v<MAJOR>[.<MINOR>]
```

Where `<MAJOR>` and `<MINOR>` are non-negative decimal integers. The minor version MAY be
omitted, in which case it is treated as 0. Examples:

- `v0.1` — major 0, minor 1
- `v1` — major 1, minor 0
- `v2.3` — major 2, minor 3

A consumer MUST reject a document whose declared major version it does not support.

#### 4.2.3 Profile Declaration

An optional `profile=<name>` token MAY appear on line 1, after the version. Its presence
activates profile conformance (§1.2.2) against the named profile. At most one `profile=`
token MAY appear on line 1. Profile names follow the grammar defined in §11.10.

A document MAY declare a `profile=<name>` where `<name>` is any syntactically valid profile
name. Whether the name is recognized determines validator behavior, not syntactic validity:

- In **structural mode** (§1.3), the validator MUST NOT reject a document for having an
  unrecognized profile name. The validator MAY report a warning.
- In **core mode**, the validator MUST NOT reject a document for having an unrecognized
  profile name. The validator MAY report a warning.
- In **profile mode** or **corpus mode**, the validator cannot establish profile
  conformance against an unrecognized profile. It MUST report an error and MUST NOT
  declare the document profile-conformant.

A syntactically malformed `profile=` token (missing value, repeated token, invalid
characters in the name) is always a syntactic error, regardless of mode.

#### 4.2.4 Dialect Declaration

An optional `dialect=<name>` token MAY appear on line 1. Dialect declarations activate
vocabulary extensions (§16). At most one `dialect=` token MAY appear on line 1. Dialect
names follow the same grammar as profile names (§11.10).

A document MAY declare both a `profile=` and a `dialect=`. When both are present, the
dialect's additions are only valid if the active profile explicitly permits that dialect
(§16.2).

The order of `profile=` and `dialect=` tokens on line 1 is not significant, but the
canonical order SHOULD be `profile=` before `dialect=`.

Unknown dialect names follow the same mode-dependent policy as unknown profile names
(§4.2.3): warning in structural and core mode, error in profile and corpus mode.

#### 4.2.5 Repeated or Malformed Header Tokens

Line 1 MUST NOT contain more than one `profile=` token or more than one `dialect=` token.
It MUST NOT contain tokens other than the filename, version, and the optional profile and
dialect declarations. Violations are syntactic errors reported against the header.

### 4.3 Sections

A section groups semantically related entries. Every entry MUST appear inside a section.

A section header is a single line matching:

```
$<NAME>
```

Where `<NAME>` uses uppercase ASCII letters, ASCII digits, and underscore. The `$` character
MUST be the first character of the line. Section names are case-sensitive and MUST be
uppercase per §11.1.

Exactly one blank line MUST precede every section except the first. The first section
appears immediately after the single blank line that follows the header.

#### 4.3.1 Section Comments

A section header MAY be followed by zero or more section comment lines, each starting with
`#`. Section comments provide context to readers and consumers. They are not entries and
are not counted in the header's `Entries:` total.

```
$AUTH_SIGNUP_LOGIN
# Covers signup, login, and session lifecycle actions.
# Does not cover password reset, which is in $AUTH_RECOVERY.
```

No blank line MAY appear between a section header and its section comments, or between
section comments and the first entry of the section.

### 4.4 Entries

An entry is a single physical line beginning with a sigil defined in §5. Entries MUST be
contiguous within a section (no blank lines between entries of the same section). One entry
equals one physical line. Hard wraps are forbidden.

Entry grammar and semantics are defined in §6.

### 4.5 Blank Line Rules

Blank lines in a JOOT document serve as structural separators. Their placement is normative.

- Exactly **one** blank line MUST appear between the header (line 4) and the first section.
- Exactly **one** blank line MUST appear before every section header except the first.
- **Zero** blank lines MAY appear between a section header and its section comments, or
  between section comments and the first entry.
- **Zero** blank lines MAY appear between consecutive entries within a section.

A blank line is a line whose content is empty (the line consists of a single `LF`, or a
single `CRLF` that is normalized to `LF` on read). A line consisting only of spaces is NOT
a valid blank line; spaces-only lines are a syntactic error.

**End of file.**

A producer SHOULD emit exactly one final `LF` after the last entry line and no trailing
blank lines.

A consumer MUST accept either:

- a document ending with a single final `LF`, or
- a document ending with no trailing byte after the last entry line.

A consumer MAY accept one trailing blank line at end of file, reporting a warning. More
than one trailing blank line is a syntactic error.

### 4.6 Physical File Normalization

A producer SHOULD emit a physically normalized file. Normalization means:

- single `LF` line endings;
- no BOM;
- no tab characters;
- no trailing spaces on any line;
- at most one trailing `LF` at end of file;
- exactly the blank lines required by §4.5.

A consumer SHOULD accept and normalize equivalent non-normalized input:

- `CRLF` normalized to `LF`;
- a leading BOM stripped and reported as a warning;
- trailing spaces stripped and reported as a warning.

Tabs (§2.6) remain a hard syntactic error and MUST NOT be silently stripped.

### 4.7 Line Length Guidance

This specification does not impose a hard maximum line length in v0.1.

A producer SHOULD keep lines within a practical length appropriate to the document's
consumers. Very long entry lines degrade readability, diff quality, and LLM retrieval, and
often indicate that the entry should be decomposed into facets or child subjects. Oversized
entries are tracked as a warning (see §19.3 and the facet-decomposition guidance in §12.3).

A future revision MAY introduce a soft threshold or a per-profile recommendation. v0.1
leaves the choice to producers and profiles.

### 4.8 Open Questions for Future Revisions

- Whether to introduce a normative line-length threshold in a future revision.
- Whether section headers MAY nest via a dotted naming convention (`$AUTH.SIGNUP` vs
  `$AUTH_SIGNUP`). v0.1 treats dots as invalid in section names.
- Whether multi-document files (multiple JOOT documents concatenated with a separator) are
  in scope. v0.1 treats one file as one document.

---

## 5. Entry Sigils

An entry sigil is the single character at the start of an entry name. It classifies the kind
of subject the entry represents, provides visual and structural grouping, and enables trivial
grep-based retrieval by subject kind.

### 5.1 Sigil Table

This specification defines the following sigils. The table is normative.

| Sigil | Name      | Subject kind                                                   |
|-------|-----------|----------------------------------------------------------------|
| `!`   | Entity    | Types, modules, schemas, endpoints, contracts, general subjects |
| `>`   | Source    | Data sources, providers, inputs, origins                       |
| `^`   | Operation | Callables, pure functions, transformers, expression operators  |
| `@`   | Event     | Events, triggers, hooks, lifecycle callbacks                   |
| `+`   | Action    | Mutations, commands, workflow steps that change state          |

### 5.2 Sigil Semantics

The sigil is advisory classification. It assists retrieval and human reading but does not
alter the structural parsing of the entry. An entry is parsed identically regardless of
which sigil it uses, subject to the reserved-sigil prohibition in §5.3.

The `!` sigil is the default. Any entry MAY use `!` regardless of its subject kind. The
richer sigils (`>`, `^`, `@`, `+`) are RECOMMENDED when they improve readability or
retrieval for the expected consumers.

A validator MUST NOT reject an entry solely because its sigil does not match its `cat:`
value. Sigil-to-category correlation is advisory, not enforced.

### 5.3 Reserved Sigils

The following characters are reserved as sigils and MUST NOT appear at the start of any
entry name in a JOOT v0.1 document:

- `?`
- `%`

Their meaning is reserved for future revisions of this specification. A document using a
reserved sigil is non-conformant at every conformance layer (core, profile, corpus). This
is a hard error and MUST NOT be recovered by lenient consumption.

No other characters are reserved as sigils in v0.1.

### 5.4 Default Sigil Rule

When a producer is unsure which sigil best classifies a subject, or when the subject does
not cleanly fit the richer categories, the producer MUST default to `!`. A document that
uses only `!` for all entries is valid.

The default rule exists to ensure that producers never need to invent new sigils or contort
a subject into a poorly-fitting category. When in doubt, `!`.

### 5.5 Sigil Position

The sigil is the first character of the entry line. It is followed immediately — with no
space — by the entry name. The entry name is followed immediately by a pipe character `|`
introducing the meta block (§6.3).

```
!OrderService|cat:type|...
>Current_User|cat:source|...
^find|cat:callable|...
@User_Signed_Up|cat:event|...
+Log_the_user_in|cat:callable|...
```

A space between the sigil and the entry name is a syntactic error.

---

## 6. Entry Model

This section defines the structure of an entry: the canonical block order, the meta block
and its canonical meta key order, the `[Rel]` block with its canonical item order, and the
`[Fields]`, `[Behavior]`, and `[Returns]` blocks.

The entry model is the core of the JOOT format. Its rules are strict. Normative closures
that this section establishes for the first time in the JOOT lineage include the complete
meta key ordering and the complete relation item ordering.

### 6.1 General Form

An entry is a single physical line with the following general structure:

```
<sigil><entry_name>|<meta_block>[|<named_block>...][|<annotation>...]
```

Where:

- `<sigil>` is one of `!`, `>`, `^`, `@`, `+` (§5.1);
- `<entry_name>` follows §11.2;
- `<meta_block>` is a sequence of meta fields separated by `|`, beginning with `cat:...`
  (§6.3);
- `<named_block>` is one of `[Rel]...`, `[Fields]...`, `[Behavior]...`, `[Returns]...` in
  canonical order (§6.2);
- `<annotation>` is `#note:...` or `#warn:...` in canonical order (§10).

The meta block has no named marker. It begins immediately after the first `|` following the
entry name and continues until either the first named block marker (`[Rel]`, `[Fields]`,
`[Behavior]`, `[Returns]`) or the first annotation prefix (`#note:`, `#warn:`), whichever
comes first.

### 6.2 Canonical Block Order

The block order within an entry is normative. When a block is present, it MUST appear in
this exact position relative to the other blocks:

```
meta → [Rel] → [Fields] → [Behavior] → [Returns] → #note → #warn
```

Any block MAY be omitted. An omitted block leaves no placeholder in the entry. The order of
the present blocks MUST be preserved.

Only the four named blocks listed above are valid. A document containing any other named
block (e.g., `[Options]`, `[Params]`, `[Config]`, `[Output]`, `[Headers]`, `[Status]`,
`[Errors]`) is non-conformant.

Empty block markers are forbidden. A block is either present with at least one item, or
absent entirely. Writing `[Fields]` with no items, or the literals `no_fields`, `none`, or
`[]`, is a syntactic error.

#### 6.2.1 Repeated Block Occurrence

Each named block MAY appear at most once per entry. Multiple `[Rel]` blocks, multiple
`[Fields]` blocks, etc. on the same entry are a syntactic error. All items of a given block
MUST be consolidated into a single occurrence of that block.

Annotations (`#note:` and `#warn:`) MAY repeat: an entry MAY have multiple `#note:`
annotations followed by multiple `#warn:` annotations (§10.3).

### 6.3 Meta Block

The meta block describes classification, identity, and preconditions of the subject. It does
not describe consequences (which belong in `[Behavior]`) or graph edges (which belong in
`[Rel]`).

The test for whether a piece of information belongs in meta:

- **Precondition, classification, or identity?** → meta
- **Consequence or effect?** → `[Behavior]`
- **Edge to another subject?** → `[Rel]`

#### 6.3.1 Required Meta Fields

Every entry MUST begin its meta block with:

```
cat:<category>
```

This is the only meta field that is required in every entry at every conformance level. All
other meta fields are optional at core conformance; specific profiles MAY require additional
meta fields (most commonly `id:` and `kind:`).

#### 6.3.2 Canonical Meta Key Order

**This is a normative closure that JOOT establishes for the first time in its lineage.**

When meta fields are present, they MUST appear in the following canonical order:

1. `cat:` (required, always first)
2. `id:`
3. `kind:`
4. `facet:`
5. `scope:`
6. `context:`
7. `target:`
8. `requires:`
9. `path:`
10. `verb:`
11. `status:`
12. `since:`
13. `sunset:`
14. `until:`
15. profile-registered or dialect-registered additional meta keys, in the order defined
    by the active profile or dialect

The rationale of this ordering is: identity and classification first (1–4), scope and
preconditions next (5–8), subject addressing (9–10, primarily relevant to `api_contract`),
then lifecycle metadata (11–14). Profile- and dialect-registered extensions follow.

Keys in positions 2–14 are optional. Any subset may be omitted, but the relative order of
those that are present MUST match the order above.

Under core conformance, a non-canonical meta key order produces a warning, not an error.
Under profile conformance, it is an error (`CON-006`, §19.5).

A producer MUST emit canonical order. A consumer in structural or core mode MAY accept
non-canonical order with a warning. A consumer in profile or corpus mode MUST reject
non-canonical order.

#### 6.3.3 Duplicate Meta Keys

A meta key MUST NOT appear more than once on the same entry, with one exception: profile-
or dialect-registered meta keys MAY be declared as repeatable by the registering profile or
dialect. None of the canonical keys in positions 1–14 is repeatable; each MAY appear at
most once.

A duplicate non-repeatable meta key is a syntactic error.

#### 6.3.4 Canonical Meta Keys

The normative list of meta keys registered by this specification.

| Key         | Purpose                                                    | Defined in |
|-------------|------------------------------------------------------------|------------|
| `cat:`      | Category (required, always first)                          | §15.1      |
| `id:`       | Stable machine-readable identifier                         | §11.3, §12 |
| `kind:`     | Subtype of `cat:` (required under profile conformance)     | §15        |
| `facet:`    | Orthogonal facet projection of the same logical subject    | §12.3      |
| `scope:`    | Runtime or platform scope                                  | §6.3.4.1   |
| `context:`  | Where the entry runs or applies                            | §6.3.4.1   |
| `target:`   | What or whom the entry operates on                         | §6.3.4.1   |
| `requires:` | Prerequisite feature, plan, flag, or capability            | §6.3.4.1   |
| `path:`     | Endpoint path (used by `api_contract` profile)             | §15.3      |
| `verb:`     | Endpoint verb (used by `api_contract` profile)             | §15.3      |
| `status:`   | Lifecycle status (e.g., `beta`, `deprecated`)              | §6.3.4.1   |
| `since:`    | Version or date introduced                                 | §6.3.4.1   |
| `sunset:`   | Version or date of planned removal                         | §6.3.4.1   |
| `until:`    | Version or date of effective removal                       | §6.3.4.1   |

Additional rules:

- `status:` MUST NOT carry HTTP response codes or other status-variant identifiers. Status
  variants of API responses are modeled as separate response or error surfaces (§15.3).
- The value of `facet:` MUST NOT be one of the forbidden child-as-facet patterns listed in
  §12.3.3.
- Profiles and dialects MAY register additional meta keys; those additions are appended
  after position 14 in the canonical order, in the order declared by the profile or
  dialect.

#### 6.3.4.1 Semantic Guidance for Canonical Meta Keys

The following brief notes establish minimal operational semantics for meta keys whose
purpose has not been fully specified in a dedicated section.

- **`scope:`** — restricts applicability to a platform, runtime, or deployment target. The
  value is an opaque atom. Common values include `native_mobile`, `web`, `server`,
  `frontend`, `backend`. Consumers use `scope:` to filter subjects by applicability.

- **`context:`** — names the location or surface where the subject runs or is available.
  Distinct from `scope:`: `scope:` is platform/runtime, `context:` is functional location
  (e.g., `workflow_only`, `admin_panel`, `login_page`). Value is an opaque atom.

- **`target:`** — names the entity that the subject operates on. Value is an opaque atom
  (e.g., `current_user`, `active_session`, `all_records`).

- **`requires:`** — declares a prerequisite the subject depends on to operate or exist.
  Value is an opaque atom (e.g., `paid_plan`, `feature_flag_x`, `admin_role`).

- **`status:`** — declares lifecycle status of the subject. Value MUST be one of:
  `stable` (default, SHOULD be omitted when stable), `experimental`, `beta`,
  `deprecated`, `removed`. `status:` MUST NOT carry HTTP response codes or other variant
  identifiers.

- **`since:`**, **`sunset:`**, **`until:`** — lifecycle dates or versions. Value is either
  a semver string (`v1.3`, `2.0.1`) or an ISO 8601 date (`2026-04-17`). `since:` names
  when the subject was introduced; `sunset:` names the planned removal version or date;
  `until:` names the effective final version or date of availability.

- **`path:`** — endpoint path. Specific to `api_contract` profile. See §15.3.

- **`verb:`** — endpoint verb. Specific to `api_contract` profile. See §15.3.

Profiles and dialects MAY add further constraints or narrower value sets for any of these
keys. A profile MUST NOT redefine the meaning of a canonical meta key.

#### 6.3.5 Meta Key Precedence and Conflict Resolution

When a meta key is declared both by this specification and by an active profile or dialect,
the canonical specification definition takes precedence. A profile or dialect MUST NOT
redefine the meaning of a canonical meta key (§15.1, §16.1).

When a meta key is registered by both an active profile and an active dialect, and the
profile's rules and the dialect's rules conflict, the profile's rules take precedence.

### 6.4 `[Rel]` Block

The `[Rel]` block declares graph edges from the current subject to other first-class
subjects. It is the only mechanism in JOOT for expressing inter-subject relationships.
Relationships MUST NOT be encoded in `[Behavior]`, in annotations, or in meta.

#### 6.4.1 Relation Item Grammar

A relation item has the form:

```
<role>:<target>[<constraint_set>]
```

Where:

- `<role>` is a lowercase underscored identifier naming the relation role (§15.2);
- `<target>` is either an internal target (`<id>` or `<id>#<facet>`) or an external target
  (`ext:<ext_ref>`) (§6.4.3);
- `<constraint_set>` is an optional `{...}` constraint block (§8) qualifying the edge.

Multiple relation items in the same `[Rel]` block are separated by commas.

```
[Rel]member_of:java.com.acme.order,implements:java.com.acme.order.OrderLookup,throws:ext:java.com.acme.order.NotFound
```

#### 6.4.2 Canonical Relation Item Order

**This is a normative closure that JOOT establishes for the first time in its lineage.**

Relation items within a `[Rel]` block MUST be sorted by the following canonical keys, in
order:

1. **Primary key: role name, byte-wise lexicographic (alphabetical) order.** Roles are
   sorted by their string name. Universal roles (§15.2) and profile- or dialect-registered
   roles are sorted together, not grouped separately.

2. **Secondary key: target type.** Within a single role, internal targets appear before
   external targets.

3. **Tertiary key: target string, byte-wise lexicographic order.** Within the same role
   and target type, targets are sorted by the full target string (the full `<id>` or
   `<id>#<facet>` for internal, the full `ext:<ext_ref>` for external).

This ordering is fully deterministic: given any set of relation items, exactly one canonical
arrangement exists, computable without consulting any registry.

Under core conformance, non-canonical relation item order produces a warning. Under profile
conformance, it is an error (`CON-007`, §19.5).

A producer MUST emit canonical order. A consumer in structural or core mode MAY accept
non-canonical order with a warning. A consumer in profile or corpus mode MUST reject
non-canonical order.

Example:

```
[Rel]annotated_by:ext:java.jakarta.Inject,extends:ext:java.lang.Object,implements:java.com.acme.order.OrderLookup,implements:java.com.acme.order.Searchable,member_of:java.com.acme.order
```

Alphabetical by role: `annotated_by` → `extends` → `implements` → `member_of`. Within
`implements`, two internal targets sorted lexicographically.

#### 6.4.3 Internal vs External Targets

A relation target is either **internal** or **external**.

**Internal target grammar:**

```
<id>
<id>#<facet>
```

An internal target addresses a subject whose entry exists — or is expected to exist — within
the active validation corpus. An internal target without a facet component addresses the
logical subject as a whole. An internal target with `#<facet>` addresses a specific facet
projection of that subject.

`<id>` values MUST NOT contain `#`. The `#` character is reserved as the facet separator in
relation targets.

**External target grammar:**

```
ext:<ext_ref>
```

An external target addresses a first-class subject outside the active validation corpus
(a library, an external API, a vendor type, a framework class). The `<ext_ref>` is an
opaque string that identifies the external subject. Core validators MUST NOT attempt to
resolve external targets.

External target syntax permits a broader character set than internal IDs:

```
<ext_ref>   = <ext_token> | <quoted_string>
<ext_token> = 1*(ALPHA | DIGIT | "_" | "." | "-" | "/" | ":" | "(" | ")" | "+")
```

A quoted string form is provided for external references that contain reserved characters.
Inside the quoted form, the standard quote escaping rules of §9.3 apply.

**Policy on external targets in profiles.**

A profile MUST declare, either role-by-role or profile-wide, whether external targets are
permitted for each registered role. If a profile does not explicitly permit external targets
for a role, external targets are invalid for that role in that profile.

v0.1 sets the following default policies for the two closed profiles:

- `code_surface` — external targets MAY be used for any registered role when the target
  subject lies outside the active validation corpus. This is the default policy stated
  globally for the profile, and does not require role-by-role enablement.
- `api_contract` — external targets are permitted for the relation roles `depends_on`,
  `compatible_with`, `incompatible_with`, `supersedes`, and `replaced_by`. External targets
  are NOT permitted for `request`, `response`, or `error`, which MUST target entries in the
  active validation corpus.

#### 6.4.4 Relation Target Resolution

Target resolution is the process by which a validator matches an internal target to an
actual entry (or set of entries) in the document or corpus.

**Semantics of internal targets.**

An internal target of the form `<id>` addresses the **logical subject** identified by
`<id>`, as a whole. It does not address any specific facet.

An internal target of the form `<id>#<facet>` addresses a specific **facet projection** of
the subject. This is a finer-grained reference than the bare form.

A role MAY require that its target be addressed at the facet level. When a role is defined
to require facet-level addressing, using a bare `<id>` target for that role is a profile
conformance error. Universal relation roles (§15.2) do not require facet-level addressing
by default; specific profiles MAY impose it.

**Resolution algorithm.**

Given an internal target and a validation scope (document or corpus):

1. For a target of the form `<id>`:

   - Let `E` be the set of entries in the scope whose `id:` meta value equals `<id>`.
   - If `E` is empty → resolution fails.
   - If `E` contains exactly one entry and that entry has no `facet:` → resolution
     succeeds; the target binds to that single entry.
   - If `E` contains one or more entries, all of which have a `facet:` meta value (i.e.,
     the subject is fully decomposed into facets, with no unfaceted entry) → resolution
     succeeds; the target binds to the logical subject as a whole, which is the union of
     those facet entries. Consumers that need to consult a specific facet MUST resolve
     `<id>#<facet>` separately.
   - If `E` contains one unfaceted entry plus one or more faceted entries → this is an
     identity modeling conflict (see §12.3). The unfaceted entry and faceted entries
     describing the same logical subject are mutually exclusive ways of decomposing
     identity and MUST NOT coexist. This is a profile conformance error.

2. For a target of the form `<id>#<facet>`:

   - Let `e` be the entry in the scope whose `id:` equals `<id>` AND whose `facet:` equals
     `<facet>`.
   - If `e` exists uniquely → resolution succeeds.
   - If no such `e` exists → resolution fails.
   - If more than one such `e` exists → identity uniqueness violation (§12.2).

3. Comparison is byte-wise exact. Case-sensitive. No normalization beyond what §3.2 and
   §11.3.2 specify.

4. In document-scope validation (structural mode, core mode, profile mode without corpus):
   resolution is attempted against the single document only.

5. In corpus-scope validation (corpus mode): resolution is attempted against the full
   corpus.

**Resolution outcomes.**

- Target resolves successfully → no diagnostic.
- Target fails to resolve in document-scope validation under core or profile mode →
  warning.
- Target fails to resolve in corpus-scope validation → error.
- Identity modeling conflict (unfaceted + faceted for the same `id:`) → error.
- Identity uniqueness violation (two entries with identical `(id, facet)`) → error.

#### 6.4.5 Role Cardinality Declaration Model

A profile MUST declare the cardinality of each relation role it registers. Cardinality
values are:

- `1` — exactly one occurrence required;
- `0..1` — at most one occurrence;
- `1..n` — at least one occurrence, no upper bound;
- `0..n` — any number of occurrences, default when unspecified.

If a profile does not specify a cardinality for a registered role, the default is `0..n`.

A profile violation of cardinality (e.g., zero occurrences of a `1..n` role, or two
occurrences of a `0..1` role) is an error under profile conformance.

Exact duplicate relation items (same role, same target, same constraints) within a single
`[Rel]` block are forbidden at every conformance level. Repeating the same role with
different targets is permitted when the role's cardinality allows plurality (`1..n` or
`0..n`).

### 6.5 `[Fields]` Block

The `[Fields]` block describes the structural shape of a subject: its parameters, its input
slots, its configuration surface, or its schema members. It describes what the subject
contains or accepts, not what it does or produces.

#### 6.5.1 Field Item Grammar

A field item has the form:

```
<field_name>:<type>[<constraint_set>]
```

Where:

- `<field_name>` is a lowercase underscored identifier (§11.4);
- `<type>` is a type expression as defined in §7;
- `<constraint_set>` is an optional `{...}` constraint block (§8).

Multiple field items in the same `[Fields]` block are separated by commas.

```
[Fields]order_id:text,customer_id:text,items:list<record<OrderItem>>,total_cents:int{min_0}
```

#### 6.5.2 Field Item Ordering

Field items within `[Fields]` SHOULD appear in the order most natural to the source
artifact (source code declaration order, API definition order, schema column order).

v0.1 does not impose a canonical field item ordering. Profiles MAY define one. A future
revision MAY introduce a universal canonical ordering for specific profiles where
deterministic output matters for canonical serialization.

#### 6.5.3 Field Names

Field names are case-sensitive and MUST use lowercase underscored identifiers per §11.4.
Duplicate field names within a single `[Fields]` block are a syntactic error.

### 6.6 `[Behavior]` Block

The `[Behavior]` block describes local properties, effects, preconditions, postconditions,
visibility, and guards of the current subject. It does not describe structural shape
(which belongs in `[Fields]`) or edges to other subjects (which belong in `[Rel]`).

#### 6.6.1 Behavior Item Grammar

A behavior item has one of three forms:

```
<atom>
<key>:<value>
<key>:<value><constraint_set>
```

Where:

- `<atom>` is a lowercase underscored identifier denoting a bare behavioral property
  (e.g., `async`, `idempotent`, `safe`);
- `<key>` is a lowercase underscored identifier;
- `<value>` is a literal value or quoted string;
- `<constraint_set>` is an optional `{...}` constraint block.

Multiple behavior items in the same `[Behavior]` block are separated by commas.

```
[Behavior]async,effect:writes_state,auth:bearer,vis:public,latency_p95_ms:150
```

#### 6.6.2 Behavior Item Ordering and Namespacing

Behavior items within `[Behavior]` SHOULD be grouped as follows:

1. bare atoms, sorted alphabetically;
2. keyed descriptors, sorted alphabetically by key;
3. profile- or dialect-registered items, following the same alphabetical ordering.

v0.1 recommends this grouping but does not strictly enforce it. A future revision MAY
promote behavior item ordering to a normative canonical order.

#### 6.6.3 Prohibition on Encoding Relations in Behavior

A registered relation role (§15.2) MUST NOT be used as a behavior key. Encoding a graph edge
in `[Behavior]` (e.g., `[Behavior]member_of:java.com.acme.order`) is a conformance error at
every layer. The check is simple: if the behavior key is in the universal relation role
registry or in the active profile's registered roles, it is forbidden here and must be moved
to `[Rel]`.

### 6.7 `[Returns]` Block

The `[Returns]` block describes the output shape of a subject: return values, response
bodies, emission payloads. It uses the same grammar as `[Fields]`.

#### 6.7.1 Return Item Grammar

A return item has the form:

```
<return_name>:<type>[<constraint_set>]
```

Identical in structure to field item grammar (§6.5.1). The only difference is semantic:
`[Fields]` describes input/structure; `[Returns]` describes output/result.

```
[Returns]order_id:text,total_cents:int,etag:text{when_if_none_match_absent}
```

#### 6.7.2 Return Item Ordering

Return items follow the same ordering policy as field items (§6.5.2). Source-natural order
is SHOULD; a canonical ordering is deferred to a future revision.

#### 6.7.3 Conditional Returns

Conditional outputs MUST be expressed via constraints (`{when_<condition>}`), not by
inventing conditional block markers. Writing `[Returns_when_create_link]...` is forbidden
(it would be an ad-hoc block name, §6.2).

Correct:

```
[Returns]link:url{when_create_link_only,server_side_only}
```

Incorrect:

```
[Returns_when_create_link]link:url
```

### 6.8 Open Questions for Future Revisions

- Whether field and return item ordering should be promoted to a normative canonical order
  in specific profiles (notably where canonical serialization requires deterministic output).
- Whether `[Behavior]` item ordering should be normative rather than recommended.
- Whether the `[Rel]` role grouping order should permit a profile-declared override (e.g.,
  a profile declares a specific role as "first among equals" for domain reasons).
- Whether repeatable profile-registered meta keys need a canonical intra-key ordering rule.

---

## 7. Type System

The type system defines the shape of values that appear in `[Fields]`, `[Returns]`, and
(where applicable) in constraint and behavior values. A type is a syntactic expression that
a consumer can interpret semantically.

This section is the **legible source of truth** for the type system. Appendix A contains the
formal ABNF grammar. The two sources MUST describe the same language. If a discrepancy is
discovered, it is a specification defect and MUST be resolved before the next revision.
Neither source is authoritative over the other; they are aligned by construction.

### 7.1 Core Scalar Types

JOOT defines the following core scalar types. The table is normative.

| Type     | Meaning                                                               |
|----------|-----------------------------------------------------------------------|
| `text`   | A sequence of Unicode characters. No length bound unless constrained. |
| `number` | A numeric value, integer or real. No range unless constrained.        |
| `int`    | A signed integer value. No range unless constrained.                  |
| `bool`   | A boolean value (`true` or `false`).                                  |
| `date`   | A calendar date or timestamp. Format interpretation is profile-specific. |
| `url`    | A URL string. Syntactic validity is the consumer's responsibility.    |
| `image`  | A reference to an image resource (typically a URL or opaque handle).  |
| `any`    | A value of type known-to-exist but not further constrained here.      |

Rules:

- `int` is a specialization of `number` restricted to integers. A profile MAY treat `int`
  and `number` as interchangeable for typing purposes, but the token `int` communicates
  intent and MUST be preserved on emission.
- `any` means "this slot holds a value whose type exists but is not characterized by this
  entry". It is distinct from an absent type declaration (which is a syntactic error — every
  field, return item, and generic argument MUST have a type).
- `date` does not prescribe a format in core. Profiles MAY require ISO 8601.

### 7.2 Generics

Generic types express parameterized shapes. JOOT defines four generic constructors, one of
which (`expr`) has both nullary and unary forms.

| Form           | Meaning                                                          |
|----------------|------------------------------------------------------------------|
| `expr`         | A dynamic expression of unknown result type.                     |
| `expr<T>`      | A dynamic expression whose result type is `T`.                   |
| `list<T>`      | An ordered collection of values of type `T`.                     |
| `record<T>`    | A reference to or instance of a record of type `T`.              |
| `map<K,V>`     | A key-value mapping from keys of type `K` to values of type `V`. |

Nested generics are permitted at any depth. Examples:

```
list<list<text>>
expr<list<record<User>>>
map<text,list<number>>
map<text,record<Order>>
```

JOOT forbids nesting of **named blocks**, not of types.

**Note on `record<T>`.** The `record<T>` constructor is intended for nominal or schema-like
entity types — that is, `T` is typically a nominal type (`record<User>`, `record<Order>`)
or a nested generic over nominal types (`list<record<OrderItem>>`). Core grammar does not
syntactically forbid scalar or primitive arguments (e.g., `record<text>`, `record<number>`),
but such usage is semantically unusual and MAY be restricted by profiles.

#### 7.2.1 Nominal Types

**This subsection establishes a closure that JOOT inherits as a fix from earlier lineage
attempts.**

Inside a generic constructor, a type argument MAY be a **nominal type** — a named entity or
schema type referenced by name. Nominal types are necessary to express `record<User>`,
`record<Order>`, `list<OrderItem>`, and similar constructions.

A nominal type is grammatically an identifier. Its syntactic form is:

```
nominal_type = identifier *("." identifier)
```

Where `identifier` starts with an alphabetic character and continues with alphanumerics or
underscore (§7.7). The dotted form permits qualified nominal types (e.g.,
`com.acme.order.Order`).

Nominal types MAY preserve source casing (e.g., `User`, `OrderItem`, `dom_print_job`).
Primitive type keywords (`text`, `number`, `int`, `bool`, `date`, `url`, `image`, `any`)
MUST be lowercase (§11.6); nominal type names MAY use any valid identifier casing.

Nominal types are resolved by consumers according to profile and dialect rules. Core
conformance does not require that a nominal type resolve to any specific entry in the
document or corpus. Profile conformance MAY require resolution; see §15 and §16.

**Consequence.** The earlier ambiguity in the JOOT lineage, where the formal grammar did
not admit a production for bare nominal type arguments inside generics, is resolved. The
normative grammar in Appendix A MUST contain a `nominal_type` production reachable from
the `type` production through `non_union_type`.

#### 7.2.2 Generic Arity and Well-Formedness

Each generic constructor has a fixed arity:

| Constructor | Arity | Arguments          |
|-------------|-------|--------------------|
| `expr`      | 0 or 1 | Optional type      |
| `list`      | 1     | Element type       |
| `record`    | 1     | Entity type        |
| `map`       | 2     | Key type, value type |

Incorrect arity is a syntactic error:

- `list<>` → error (missing argument)
- `list<text,number>` → error (wrong arity, `list` takes one)
- `map<text>` → error (wrong arity, `map` takes two)
- `map<text,number,bool>` → error (wrong arity)
- `expr<text,number>` → error (`expr` takes zero or one)

Whitespace inside generic argument lists is not permitted in core v0.1. `list<text>` is
valid; `list< text >` is not. This applies uniformly: no whitespace inside `<...>`, no
whitespace inside `{...}`, no whitespace around the `,` separator.

Dialects MAY introduce additional generic constructors. Dialect-introduced constructors
MUST declare their arity explicitly in the dialect addendum (§16.4).

### 7.3 References

Names ending in `_ref` denote references to first-class objects. They are a syntactic
convention for expressing "a pointer or handle to another subject" in a way that is more
readable than `record<Subject>` for platform-specific constructs.

Canonical reference type examples:

```
page_ref, view_ref, element_ref, event_ref, workflow_ref, type_ref, field_ref,
state_ref, input_ref, map_ref, table_ref, function_ref, access_ref, style_ref
```

**Reference types describe shape, not edges.** A `_ref` appearing in `[Fields]` or
`[Returns]` is a structural slot that holds a reference. It is NOT an entry in the graph of
subjects and does NOT create a `[Rel]` edge. When a first-class relationship between
subjects exists, it MUST be expressed in `[Rel]` (§6.4), regardless of whether a related
field also uses a `_ref` type.

Profile and dialect registries MAY expand the list of canonical reference types. Unknown
reference types are permitted at core conformance (any identifier ending in `_ref` is
syntactically valid); profile conformance MAY restrict them.

### 7.4 Compound Types

JOOT defines four compound types — syntactic shapes that are not primitive scalars but that
the core grammar treats as first-class.

| Form              | Meaning                                            |
|-------------------|----------------------------------------------------|
| `enum{a/b/c}`     | A one-of enumeration of literal values.            |
| `field_changes`   | A set of field-change tuples (opaque in core).     |
| `key_value_pairs` | An arbitrary key-value mapping (opaque in core).   |
| `params`          | A free-form parameter bag (opaque in core).        |

Rules:

- `enum{...}` values are separated by `/` (forward slash). Values are identifiers (lowercase
  underscored by convention). The enum body MUST contain at least one value. Empty
  enumerations (`enum{}`) are a syntactic error.
- `field_changes`, `key_value_pairs`, and `params` are structurally opaque in the core
  grammar. Profiles MAY define finer-grained interpretation.
- `params` is a semantic safety valve. Its overuse is a conformance concern (warning
  W6 in §19.3). Prefer stronger typing when possible.

### 7.5 Unions

A union is a type that accepts one of several alternative types. Syntax:

```
<type>|<type>[|<type>...]
```

Examples:

```
navigate_to:page_ref|view_ref
payload:record<JobSpec>|record<JobDraft>
```

#### 7.5.1 Union Parsing Precedence

The pipe character `|` is used both as a block/meta separator at the entry level and as a
union separator at the type level. Parsing precedence is defined by position:

- A `|` appearing inside a `<...>` generic argument or inside a constraint block `{...}` is
  a union separator.
- A `|` appearing outside any `<...>` or `{...}` in an entry is a block/meta separator.

This means a top-level union in a field type position is permitted:

```
[Fields]navigate_to:page_ref|view_ref
```

To avoid parser ambiguity at boundary positions, producers SHOULD prefer:

- wrapping unions in a generic when a boundary is near: `list<page_ref|view_ref>`;
- splitting the field into two separate fields when the union has distinct names available;
- using a dialect-specific named type that subsumes the union.

A strictly linear parser MUST handle top-level unions correctly. Appendix A defines the
precedence formally.

#### 7.5.2 Union Disambiguation

Unions are structurally unordered: `A|B` and `B|A` denote the same type. Producers SHOULD
emit union alternatives in byte-wise lexicographic order for canonical form.

Duplicate alternatives in a union (`text|text`) are a syntactic error.

### 7.6 Dialect Type Extensions

Dialects MAY extend the type system by declaring additional types in their addendum.

| Domain         | Typical types                                                             |
|----------------|---------------------------------------------------------------------------|
| UI             | `color`, `dim`, `border`, `shadow`, `spacing`, `font`                     |
| Rust / SQLite  | `struct_ref`, `trait_ref`, `crate_ref`, `sql_type`, `migration_ref`       |
| HTTP / API     | `endpoint_ref`, `header_ref`, `status_code`, `json_path`, `mime_type`     |
| CLI            | `flag`, `subcommand_ref`, `path`, `glob`, `env_var`                       |
| Config         | `config_key`, `section_ref`, `env_ref`                                    |

Dialect type names MUST follow core identifier rules. A dialect MUST NOT redefine the
meaning of a core scalar, generic, or compound type.

### 7.7 Normative Type Grammar

This subsection states the type grammar in compact form. It is normative and MUST match the
ABNF in Appendix A.

```
type            = union_type / non_union_type

non_union_type  = scalar_type
                / generic_type
                / reference_type
                / compound_type
                / nominal_type

scalar_type     = "text" / "number" / "int" / "bool" / "date" / "url" / "image" / "any"

generic_type    = "expr"   [ "<" type ">" ]
                / "list"   "<" type ">"
                / "record" "<" type ">"
                / "map"    "<" type "," type ">"

reference_type  = identifier "_ref"

compound_type   = "enum" "{" enum_value *("/" enum_value) "}"
                / "field_changes"
                / "key_value_pairs"
                / "params"

union_type      = non_union_type "|" non_union_type *("|" non_union_type)

nominal_type    = identifier *("." identifier)

identifier      = ALPHA *(ALPHA / DIGIT / "_")

enum_value      = identifier
```

Key grammar decisions summarized:

- `type` is fundamentally either a `union_type` or a `non_union_type`. `nominal_type` is a
  first-class member of `non_union_type`, reachable from every position where `type` is
  expected.
- Unions never nest directly. `A|B|C` is a single `union_type`, not nested unions.
- Compound type `enum` uses `/` (not `,`) to separate its values.
- Whitespace is not permitted inside type expressions in v0.1.
- Nominal type names and reference type identifiers start with a letter. The leading
  underscore form (`_foo`) is not permitted as a type identifier in v0.1.

### 7.8 Type Equivalence

Two types are equivalent under these rules:

1. **Scalar equivalence.** Two scalar types are equivalent iff they are the same token.
   `int` is not equivalent to `number` in the type system, even though `int` is
   semantically a subset of `number`.

2. **Generic equivalence.** Two generic types are equivalent iff they have the same
   constructor and equivalent arguments in the same positions.

3. **Reference equivalence.** Two reference types are equivalent iff their identifiers are
   byte-wise identical. `page_ref` ≢ `Page_ref` (case matters).

4. **Compound equivalence.**
   - Two `enum{...}` types are equivalent iff their value sets are equal as sets.
   - `field_changes`, `key_value_pairs`, `params` are equivalent to themselves only.

5. **Union equivalence.** Two unions are equivalent iff their alternative sets are equal as
   sets. `A|B` ≡ `B|A`.

6. **Nominal equivalence.** Two nominal types are equivalent iff their fully-qualified
   names are byte-wise identical under the comparison rules in §11.3.2. Case-sensitive.

Type compatibility (whether a value of type `A` may flow into a slot of type `B`) is not
defined in v0.1 and is out of scope. Profiles MAY define compatibility.

### 7.9 Open Questions for Future Revisions

- Whether to introduce explicit subtype relationships (e.g., `int` <: `number`) in v0.2.
- Whether to permit whitespace inside generic argument lists with producer normalization.
- Whether to introduce an `optional<T>` or `nullable<T>` generic constructor in v0.2.
- Whether enum values may be quoted strings as well as identifiers.
- Whether union alternative order has any semantic weight.
- Whether `date` should be split into finer-grained types in a future revision (e.g.,
  `date` for calendar dates, `datetime` for timestamps, `time` for time-of-day, `duration`
  for intervals).

---

## 8. Constraints

Constraints attach refinements or annotations to a field, return item, relation item, or
behavior descriptor. Their syntax is a comma-separated list of atoms inside curly braces,
immediately following the item they qualify.

```
number{0_to_24,default:1,hours}
expr<text>{server_side_only}
response:http.orders.get.res_200{when_if_none_match_absent}
```

### 8.1 Constraint Syntax

A constraint block has the form:

```
{<atom>[,<atom>...]}
```

Where each atom is one of three syntactic classes:

```
atom           = bare_atom / keyed_atom / range_atom

bare_atom      = LOWER *(LOWER / DIGIT / "_")
keyed_atom     = bare_atom ":" value
range_atom     = 1*DIGIT "_to_" 1*DIGIT

value          = identifier / number_literal / quoted_string

number_literal = 1*DIGIT [ "." 1*DIGIT ]
```

Rules:

- A constraint block opens with `{` and closes with `}`. Mismatched braces are syntactic
  errors.
- Atoms within a constraint block are separated by `,` (no whitespace).
- A `bare_atom` starts with a lowercase ASCII letter and continues with lowercase letters,
  digits, and underscores. This is the canonical form for most constraint atoms.
- A `keyed_atom` has a bare-atom-form key, a colon, and a value. The value MAY be an
  identifier, a `number_literal` (integer or decimal, non-negative in v0.1), or a quoted
  string (§9.4).
- A `range_atom` expresses an inclusive numeric range, with the form `N_to_M` where `N` and
  `M` are non-negative decimal integers. This is the only atom form that begins with a
  digit.
- Whitespace inside `{...}` is forbidden in v0.1.

**Note on `number_literal`.** v0.1 permits only non-negative numeric literals (integer or
decimal) as constraint values. Signed literals and scientific notation are not supported; a
future revision MAY extend the grammar.

Empty constraint blocks (`{}`) are a syntactic error. If no constraints apply, the block is
omitted.

Duplicate constraint atoms on the same item are a syntactic error. The same atom (whether
bare, keyed, or range) MUST NOT appear more than once within a single constraint block. A
future revision MAY introduce list-valued repeatable constraints declared by profiles; v0.1
does not permit repetition.

### 8.2 Canonical Constraint Patterns

JOOT v0.1 registers the following constraint atom patterns. This is an **initial whitelist**,
not a closed vocabulary (see §8.7).

| Pattern                 | Meaning                                    | Example                     |
|-------------------------|--------------------------------------------|-----------------------------|
| `when_<X>`              | Conditional visibility or meaning          | `{when_change_email}`       |
| `default:<V>`           | Default value                              | `{default:1}`               |
| `requires_<X>`          | Requires another field or state            | `{requires_include_labels}` |
| `deprecated_<reason>`   | Deprecated                                 | `{deprecated_Jul_2017}`     |
| `max_<N>`               | Maximum value or count                     | `{max_50}`                  |
| `min_<N>`               | Minimum value or count                     | `{min_1}`                   |
| `<N>_to_<M>`            | Inclusive numeric range                    | `{0_to_24}`                 |
| `<unit>`                | Unit annotation                            | `{seconds}`, `{pixels}`     |
| `server_side_only`      | Scope restriction                          | `{server_side_only}`        |
| `client_side_only`      | Scope restriction                          | `{client_side_only}`        |
| `single_use`            | Usage restriction                          | `{single_use}`              |
| `read_only`             | Mutability                                 | `{read_only}`               |
| `write_only`            | Mutability                                 | `{write_only}`              |
| `nullable`              | Accepts null                               | `{nullable}`                |
| `required`              | Required (explicit presence annotation)    | `{required}`                |
| `optional`              | Optional (explicit presence annotation)    | `{optional}`                |

**Note on presence annotations.** The atoms `required`, `optional`, and `nullable` are
**explicit presence annotations** whose operational semantics are **profile-defined**.
Their universal registration establishes that these atoms are recognized and reserved, not
that they mean exactly the same thing across domains.

Specifically:

- In API contract contexts (`api_contract` profile), `required` on a request field typically
  means "the client MUST send this field". `optional` means "the client MAY omit this
  field". `nullable` means "the value, if present, MAY be null".
- In schema contexts (future `schema_model` profile), `required` typically maps to
  `NOT NULL` semantics; `nullable` to `NULL` permitted.
- In code surface contexts (`code_surface` profile), these atoms annotate language-specific
  nullability and optionality in the way the target language expresses them.

Profiles MAY further constrain the semantics of these atoms in their addenda. A document
using these atoms under core conformance without a profile activated has only the
interpretation that the annotation is declared; the operational meaning is informal.

**Note on `N_to_M` ranges.** The `N_to_M` pattern is the only constraint atom form that
begins with a digit. Its grammar is defined in §8.1 as `range_atom`. Producers and
consumers MUST treat it as a first-class atom class, not as a `bare_atom` violation.

### 8.3 Units

The following unit atoms are normatively registered. All unit atoms are `bare_atom`-form
identifiers (lowercase, starting with a letter).

| Unit       | Meaning                |
|------------|------------------------|
| `seconds`  | Time, seconds          |
| `ms`       | Time, milliseconds     |
| `minutes`  | Time, minutes          |
| `hours`    | Time, hours            |
| `days`     | Time, days             |
| `pixels`   | Length, pixels         |
| `em`       | Length, em units       |
| `rem`      | Length, root em units  |
| `percent`  | Ratio, percent         |
| `mb`       | Size, megabytes        |
| `kb`       | Size, kilobytes        |
| `bytes`    | Size, bytes            |
| `degrees`  | Angle, degrees         |

Additional units MAY be registered by profiles or dialects. A unit appears as a `bare_atom`
within a constraint block:

```
number{max_30,seconds}
number{0_to_360,degrees}
number{max_10,mb}
```

**Note on casing.** v0.1 requires `mb` and `kb` (lowercase) for uniformity with the overall
`bare_atom` grammar. This departs from common scientific-unit usage (where MB, KB are
uppercase). A producer that extracts size values from systems using uppercase
abbreviations MUST normalize to lowercase on emission. This decision favors grammatical
uniformity over visual-convention compatibility.

### 8.4 Constraint Registry and Namespacing

v0.1 establishes constraints as a **registered but extensible vocabulary**. Specifically:

1. The patterns in §8.2 and §8.3 form the **universal constraint vocabulary**. These atoms
   are recognized at every conformance level.

2. A profile MAY register additional constraint atoms in its addendum (§15). Profile-
   registered atoms are valid when the profile is active.

3. A dialect MAY register additional constraint atoms in its addendum (§16). Dialect-
   registered atoms are valid when the dialect is active and permitted by the active
   profile.

4. A document MAY use constraint atoms that are not registered at any level. This is
   permitted at core conformance (with warning in §19.3), but is an error under profile
   conformance when the profile declares its constraint vocabulary closed (§8.7).

The universal constraint vocabulary is **append-only**: future revisions of this
specification MAY register additional universal atoms, but MUST NOT remove or redefine
existing ones.

### 8.5 Constraint Composition and Conflict Resolution

Multiple constraint atoms on a single item compose additively. Their combined meaning is
the intersection of individual meanings.

```
number{min_0,max_100,default:50,percent}
```

is interpreted as: a number, between 0 and 100 inclusive, with default value 50, expressed
as a percentage.

When two constraint atoms **conflict** (declare mutually incompatible restrictions), the
conflict is a semantic error. Examples of conflicts:

- `{min_10,max_5}` — contradictory range.
- `{required,optional}` — contradictory presence declarations.
- `{read_only,write_only}` — contradictory mutability.
- `{nullable,required}` under a strict interpretation — a profile MAY define this as a
  conflict.

Conflict detection is a conformance check. v0.1 does not exhaustively enumerate conflicts;
it establishes the principle that conflicting constraints are semantic errors and tasks
validators with catching the obvious contradictions.

### 8.6 Canonical Constraint Ordering

Constraint atoms within a single constraint block MUST be sorted in the following canonical
order:

1. **Non-keyed atoms first.** All `bare_atom` and `range_atom` forms appear before any
   `keyed_atom` forms. Within this group, atoms are sorted by byte-wise lexicographic order
   of the full atom text (for `bare_atom`, the identifier; for `range_atom`, the full
   `N_to_M` string).

2. **Keyed atoms after.** All `keyed_atom` forms appear after the non-keyed group. Within
   this group, atoms are sorted by byte-wise lexicographic order of the key (the portion
   before `:`).

This two-group ordering treats `bare_atom` and `range_atom` as a single non-keyed class for
sorting purposes, avoiding an unnecessary third ordering tier while accommodating the
digit-initial `range_atom` form cleanly.

Example:

```
{0_to_24,client_side_only,deprecated_Jul_2017,hours,max_24,min_0,default:1}
```

Non-keyed group sorted alphabetically: `0_to_24` (digit sorts before lowercase letters in
ASCII), `client_side_only`, `deprecated_Jul_2017`, `hours`, `max_24`, `min_0`. Keyed group
after: `default:1`.

Under core conformance, non-canonical constraint order produces a warning. Under profile
conformance, it is an error (`CON-008`, §19.5).

### 8.7 Constraint Closure Under Profiles

A profile MAY declare its constraint vocabulary **closed** or **open**.

- **Closed vocabulary (recommended for machine-first profiles).** Only atoms registered by
  the universal vocabulary, the profile itself, or dialects explicitly permitted by the
  profile are valid. Unknown atoms are errors under profile conformance.

- **Open vocabulary.** Atoms not registered at any level are accepted with a warning.

The standard profiles defined in this specification (`code_surface`, `api_contract`) declare
**closed** constraint vocabularies. This is normative.

Profile closure applies to constraint **atoms** (the `<bare_atom>` form and the `<key>` of
keyed atoms). Profile closure does NOT apply to constraint **values** (the `<value>` in
keyed atoms), which are free-form opaque strings unless a specific key's value format is
registered.

### 8.8 Constraint Placement

Constraints attach to four kinds of items:

1. **Field items** (in `[Fields]`): `number{max_24}`
2. **Return items** (in `[Returns]`): `url{server_side_only}`
3. **Relation items** (in `[Rel]`): `response:http.orders.get.res_200{when_if_none_match_absent}`
4. **Behavior items with keyed values** (in `[Behavior]`): `auth:bearer{optional}`

Constraints are placed immediately after the value they qualify, with no whitespace between
the value and the opening `{`.

Constraints on bare behavior atoms are not permitted in v0.1. If additional qualification
of a behavior atom is needed, the atom SHOULD be promoted to a keyed form.

### 8.9 Open Questions for Future Revisions

- Whether the universal constraint vocabulary should be **fully closed** (not append-only,
  but fixed) at some future revision.
- Whether constraint values (the right-hand side of keyed atoms) should have their own
  type-like grammar, or remain opaque strings.
- Whether a mechanism for list-valued repeatable constraints is needed.
- Whether constraint composition needs a formal intersection semantics defined in the
  specification, or remains informally "additive".
- Whether `number_literal` should be extended to support signed values or scientific
  notation in a future revision.

---

## 9. Escaping and Quoting

This section defines how values containing reserved characters are represented. It is
deliberately strict: the reserved character set is closed, the escape alphabet is closed,
and the semantic equivalence between quoted and unquoted forms is defined unambiguously.

### 9.1 Reserved Characters

The following ASCII characters are **reserved** in JOOT. They have syntactic meaning in at
least one position of the format and MUST be escaped or quoted when they appear inside a
value.

```
|   ,   [   ]   {   }   \   "
```

The role of each reserved character:

| Character | Role                                                                      |
|-----------|---------------------------------------------------------------------------|
| `|`       | Separator between blocks and between meta fields; union separator in types. |
| `,`       | Separator between items within a block; separator between generic arguments. |
| `[`, `]`  | Delimit named block markers (`[Rel]`, `[Fields]`, `[Behavior]`, `[Returns]`). |
| `{`, `}`  | Delimit constraint blocks and `enum{...}` compound type values.            |
| `\`       | Introduces escape sequences.                                               |
| `"`       | Delimits quoted strings.                                                   |

This list is closed. No other ASCII characters are reserved at the core grammar level.

Three characters have constrained roles in specific positions but are NOT reserved in the
above sense:

- `#` — starts header lines, section comments, and annotations. It is forbidden inside `id:`
  values (§11.3) and has no special meaning inside quoted strings or value positions. It
  does not require escaping.
- `:` — separates meta keys from values, field names from types, roles from targets. It
  does not require escaping.
- `<`, `>` — delimit generic type arguments. In entry names, `>` is additionally permitted
  as a hierarchy separator (§11.2). They do not require escaping.

### 9.2 Where Escaping Applies

Escaping rules depend on position:

**In unquoted values** (meta values, field type arguments that are literals, behavior values
that are literals, annotation content before `:`, `#note:`/`#warn:` payloads):

Reserved characters MUST be either backslash-escaped (§9.3) or the entire value MUST be
wrapped in a quoted string (§9.4).

**In quoted strings:**

Only `"` (U+0022) and `\` (U+005C) MUST be escaped. All other reserved characters MAY
appear literally inside a quoted string without escaping.

**In identifier positions** (meta keys, block markers, entry names, `id:` values, field
names, relation roles, type names, sigil characters):

Reserved characters MUST NOT appear at all. Escaping does not extend identifier grammars.
An `id:` value cannot contain `|` even escaped; instead the grammar of `id:` restricts its
character set (§11.3.1). `id:` values MUST be emitted and consumed in unquoted form only
(§11.3.4).

### 9.3 Backslash Escapes

The following backslash escape sequences are defined. This list is closed.

| Escape | Represents |
|--------|------------|
| `\|`   | `U+007C` pipe         |
| `\,`   | `U+002C` comma        |
| `\[`   | `U+005B` left bracket |
| `\]`   | `U+005D` right bracket |
| `\{`   | `U+007B` left brace   |
| `\}`   | `U+007D` right brace  |
| `\\`   | `U+005C` backslash    |
| `\"`   | `U+0022` double quote |

Rules:

- A backslash followed by any character not in the list above is a syntactic error. There is
  no `\n`, `\t`, `\u....`, or any other escape form in v0.1.
- Escapes are evaluated left to right. `\\|` means "a literal backslash followed by the
  block separator pipe", which is valid only if the context allows an unescaped pipe; if
  not, this is a syntactic error.
- A literal newline (`LF`) cannot appear inside any value, escaped or quoted. Entries are
  strictly single-line (§4.4). A value that logically contains a line break MUST express
  it by other means (e.g., via a string representation agreed by consumer convention, or
  by splitting into multiple entries).

### 9.4 Quoted Strings

A quoted string is delimited by double quotes `"..."`.

Inside a quoted string:

- `"` MUST be escaped as `\"`.
- `\` MUST be escaped as `\\`.
- All other characters (including reserved characters `|`, `,`, `[`, `]`, `{`, `}`) appear
  literally.
- Unicode content follows the rules in §3.3 for quoted and free-text content. Any valid
  UTF-8 code point is permitted, subject to the forbidden code points listed in §3.3.
- The `LF` character MUST NOT appear inside a quoted string. Quoted strings, like all
  values, are single-line.

Example:

```
[Behavior]sample:"Price, with VAT"
```

The quoted form is useful when a value contains multiple reserved characters, free-text
content intended for human reading, or characters outside the ASCII printable range.

### 9.5 Quoted vs Unquoted Equivalence

Quoted and unquoted forms are **semantically equivalent** when they represent the same
byte sequence after unescaping. That is:

- `"hello"` and `hello` represent the same value.
- `"a,b"` and `a\,b` represent the same value.
- `"a\"b"` and `a\"b` (where the unquoted form escapes the quote) represent the same value.

A producer SHOULD choose the shorter form. When both are equal in length, the unquoted form
is canonical. When a value contains two or more reserved characters, the quoted form is
typically shorter and canonical.

Canonical form rules for producers:

1. If the value contains no reserved characters and no leading/trailing whitespace, emit
   unquoted.
2. If the value contains exactly one reserved character, the producer MAY emit either form;
   the shorter byte sequence is canonical.
3. If the value contains two or more reserved characters, or any character that cannot
   legally appear unquoted in the current position (e.g., a space at the start, a non-ASCII
   character in a position where the grammar permits it only inside quoted strings), emit
   quoted.
4. If the value is empty, it MUST be emitted as `""` (the empty quoted string). An unquoted
   empty value is a syntactic error.

Consumers MUST treat quoted and unquoted forms as equivalent for identity and comparison
purposes. Two entries whose values differ only in quoting style are semantically identical.

**Exception for `id:` values.** `id:` values are exempt from the quoted/unquoted
equivalence rule: they are restricted to unquoted form only (§11.3.4). This exception
exists because the `id:` character set excludes all reserved characters, making quoting
unnecessary.

### 9.6 Characters That Can Only Appear Quoted

The following characters, if needed inside a value, MUST appear inside a quoted string.
They cannot be represented in unquoted form even with escaping:

- The leading space. A value beginning with `U+0020` cannot be unquoted (the space would be
  interpreted as whitespace boundary). Wrap in quotes.
- The trailing space. Same reason.
- Any Unicode character outside the ASCII range (U+0080 and above) when the position's
  grammar does not permit non-ASCII identifiers or literals. Non-ASCII content is permitted
  in quoted strings and free-text (§3.3), and in those positions it MAY appear without
  quoting only if the position's grammar admits it.
- The empty value, as noted above.

### 9.7 Escaping Is Orthogonal to Encoding

Escaping in JOOT is a syntactic mechanism, not an encoding mechanism. Escaping does not
introduce additional character representations (no `\x41` for `A`, no `\u00e9` for `é`).
Unicode content is represented directly in UTF-8.

### 9.8 Open Questions for Future Revisions

- Whether to introduce limited escape forms for control characters that are currently
  forbidden (e.g., a `\n` escape to permit a logical newline inside a quoted string,
  processed to `LF` by consumers). v0.1 forbids this to preserve the strict single-line
  property of entries.
- Whether to permit triple-quoted multi-line strings in a hypothetical future multi-line
  entry form. v0.1 entries are strictly single-line, so this is not applicable.

---

## 10. Annotations

Annotations attach human-readable notes and warnings to an entry. They are the only
mechanism for inline commentary inside an entry. Annotations are advisory; they do not alter
the structural or semantic meaning of the entry.

### 10.1 `#note:`

The `#note:` annotation carries informational content: tips, quirks, cross-references,
non-dangerous edge cases, clarifications, and provenance.

Syntax:

```
|#note:<content>
```

Rules:

- `#note:` is preceded by a `|` separator from the previous block.
- `<content>` is free text subject to the quoting and escaping rules of §9.
- `<content>` MUST NOT contain line breaks. Annotations are single-line.
- Multiple `#note:` annotations on the same entry are permitted.
- An empty `#note:` annotation (no content after the colon) is a syntactic error.

Examples:

```
|#note:equivalent_to_Create_a_new_thing_plus_Log_the_user_in
|#note:"Respects Settings > General > Cookie duration"
```

### 10.2 `#warn:`

The `#warn:` annotation carries risk-bearing content: destructive operations,
irreversibility, security implications, hidden side effects, and caveats a consumer needs
to be aware of.

Syntax:

```
|#warn:<content>
```

Rules identical to `#note:` (§10.1).

Examples:

```
|#warn:original_password_irrecoverable
|#warn:"Deleting this entry cascades to all dependent subjects"
```

### 10.3 Ordering

Within a single entry, annotations MUST appear in this order:

1. All `#note:` annotations first, in the order the producer chooses (no canonical order
   among notes is defined).
2. All `#warn:` annotations after, in the order the producer chooses (no canonical order
   among warnings is defined).

Mixing notes and warnings (a note after a warning) is a syntactic error. The order of
notes among themselves, and of warnings among themselves, is not normatively canonicalized
in v0.1.

### 10.4 Annotations Carry No Structural Semantics

Annotations are opaque text for structural parsing purposes. A validator MAY inspect
annotation content for advisory checks (e.g., flagging a `#warn:` on a subject marked
`status:stable` as a contradiction), but such checks are optional and do not affect
conformance.

A consumer MUST NOT derive structural or identity information from annotations. A
`#note:replaced_by_new.id` is not a relation; it carries no edge. When a consumer needs to
express a graph edge, it MUST use `[Rel]` (§6.4).

### 10.5 Open Questions for Future Revisions

- Whether a canonical ordering among `#note:` annotations and among `#warn:` annotations
  should be defined in a future revision (for deterministic serialization).
- Whether additional annotation prefixes (e.g., `#todo:`, `#deprecated:`) should be
  registered as universal. v0.1 limits the set to `#note:` and `#warn:`.

---

## 11. Naming Conventions

This section defines the naming rules for every kind of identifier in a JOOT document. The
rules are strict, case-sensitive, and ASCII-only in v0.1.

### 11.1 Sections

Section names follow:

```
section_name = "$" (UPPER / DIGIT) *(UPPER / DIGIT / "_")
```

Rules:

- Section names MUST use uppercase ASCII letters, digits, and underscore.
- Section names MUST begin with an uppercase ASCII letter or a digit, immediately after the
  `$` prefix. Leading underscore is not permitted, and a section header of `$` alone is a
  syntactic error.
- Section names MUST contain at least one character after the `$`.
- Section names are case-sensitive: `$USERS` and `$users` are distinct, but `$users` is
  non-conformant because it violates the uppercase rule.
- Dots, hyphens, and other characters are not permitted.

### 11.2 Entry Names

Entry names preserve source identity where possible. Two casing styles are permitted and a
producer SHOULD choose one consistently within a semantic family.

**Mixed_Case_Underscored** — for product-visible or human-facing names:

```
!Sign_the_user_up
!Order_Service
```

**lower_case_underscored** — for schema or internal identifiers:

```
!dom_print_job
!order_item
```

Entry name grammar:

```
entry_name      = entry_segment *(">" entry_segment)
entry_segment   = entry_name_char *entry_name_char
entry_name_char = ALPHA / DIGIT / "_"
```

Rules:

- Permitted characters: ASCII letters, digits, underscore, and the hierarchy separator `>`.
- An entry name MUST begin with a letter or digit. It MUST NOT begin with `_` or `>`.
- The `>` character MAY appear inside an entry name to mirror a source hierarchy
  (e.g., `Settings>General>Redirect`). It MUST NOT begin or end an entry name, and
  consecutive separators (`A>>B`) are forbidden: each `>` MUST be surrounded by
  non-empty segments.
- An entry name MUST NOT begin with `_`. The grammar admits `_` only inside a segment
  after the first character has been produced by the prose rule above; validators MAY
  enforce this as a syntactic error.
- Case-sensitive.
- No whitespace. Spaces in a source name become underscores.
- No other punctuation.

### 11.3 `id:` Values

The `id:` meta field carries the stable, machine-readable identifier of a subject. It is
the logical address through which relations resolve, across facets and across files in a
corpus. Its rules are the strictest in the naming system because identity depends on them.

#### 11.3.1 ID Grammar

An `id:` value follows this grammar:

```
id_value    = id_segment *("." id_segment)
id_segment  = id_char_1 *id_char_n

id_char_1   = ALPHA / DIGIT / "_"
id_char_n   = ALPHA / DIGIT / "_" / "-" / "/" / "(" / ")" / "+"
```

Rules:

- An `id:` value is one or more segments separated by dots (`.`). Dots are the only
  permitted segment separator.
- Each segment MUST be non-empty. The value MUST NOT contain empty segments (e.g.,
  `foo..bar` is invalid, as is `.foo`, as is `foo.`).
- Each segment's first character MUST be an ASCII letter, digit, or underscore. Subsequent
  characters of a segment MAY additionally include hyphen (`-`), slash (`/`), parenthesis
  (`(`, `)`), and plus (`+`).
- The full `id:` value MUST match the combined grammar above across all its segments.

Permitted examples:

```
id:ui.button
id:java.com.acme.order.OrderService
id:http.orders.get
id:wf.ingest.validate
id:java.com.acme.order.OrderService.find(text)
id:cpp.acme.print.RasterJob.submit(text+Options)
id:ts.autoprint.pricing.quote(JobSpec)
```

Non-permitted examples:

```
id:foo..bar          (empty segment)
id:.foo              (leading empty segment)
id:foo#bar           (# is explicitly forbidden; see §11.3.4)
id:foo bar           (space is forbidden)
id:foo@bar           (@ is not in the permitted character set)
id:foo:bar           (: is not in the permitted character set)
```

#### 11.3.2 ID Normalization and Comparison

IDs are compared **byte-wise, exact, case-sensitive**.

Specifically:

1. Two `id:` values are equal iff their UTF-8 byte sequences are identical.
2. Comparison is case-sensitive. `OrderService` and `orderservice` are distinct IDs.
3. **No Unicode normalization is applied in v0.1.** Because `id:` values are ASCII-only
   (the permitted character set in §11.3.1 is a subset of ASCII), Unicode normalization
   has no effect: NFC, NFD, NFKC, and NFKD all produce identical byte sequences for ASCII
   content. A future revision MAY extend `id:` grammar to non-ASCII characters, at which
   point normalization rules will be defined.
4. No trimming or case-folding is applied. Leading or trailing whitespace cannot appear in
   an `id:` value (the grammar forbids it).
5. `id:` values are restricted to unquoted form (§11.3.4); the quoted/unquoted equivalence
   rule of §9.5 does NOT apply to `id:`.

**Identity uniqueness** is defined over `(id, facet)` tuples (§12.2). Two entries with
byte-identical `(id, facet)` are duplicates; two entries that differ in `id:` by even a
single byte are distinct subjects.

#### 11.3.3 ID Stability Rules

`id:` values are the stable address of a subject across time, across documents, and across
tooling. Stability is a producer responsibility, not a grammar rule, but this specification
gives normative guidance.

**RECOMMENDED stability practices:**

- An `id:` SHOULD remain stable across refactors, renames, and reorganizations of the
  source artifact. A subject that is conceptually the same entity SHOULD retain the same
  `id:` even if its source name changes.
- An `id:` SHOULD NOT be reused to refer to a different subject. Once an `id:` has been
  retired from use (the subject is removed, split, or renamed), the `id:` SHOULD NOT be
  assigned to a new subject. Reuse of retired IDs breaks cross-corpus references and
  archived documents.
- An `id:` SHOULD reflect the logical namespace of the source ecosystem:
  `<ecosystem>.<package>.<subject>`. Examples: `java.com.acme.order.Order`,
  `py.autoprint.jobs.PrintJob`, `http.orders.get`, `sql.public.orders`.

**Identity-preserving operations** — the following changes SHOULD NOT require changing the
`id:`:

- renaming the surface identifier (e.g., Java class renamed) when the logical role is
  preserved;
- moving the subject between files within the same corpus, provided the logical namespace
  does not change;
- adding or removing facets of the subject (facet additions/removals do not change the
  logical subject's `id:`).

**Identity-changing operations** — the following SHOULD trigger a new `id:`:

- splitting a subject into two distinct subjects;
- merging two subjects into one (the result has its own `id:`; the originals are retired);
- promoting a subject from internal to externally visible, or vice versa, when the change
  implies a distinct logical role;
- changing `cat:` or `kind:` to a semantically incompatible value. (See §12.3 for the
  related rule that all facets of the same `id:` MUST share the same `cat:` and `kind:`.)

Deprecation is expressed with `status:deprecated` and, where applicable, a
`[Rel]replaced_by:<new_id>` edge. The old entry continues to exist under its original `id:`
and SHOULD NOT be deleted immediately; this preserves backward resolution for existing
consumers.

#### 11.3.4 Forbidden ID Patterns

The following patterns are explicitly forbidden in `id:` values:

1. **The `#` character is forbidden.** `#` is reserved as the facet separator in relation
   targets (§6.4.3). An `id:` containing `#` is a syntactic error.

2. **Empty segments are forbidden.** `foo..bar`, `.foo`, and `foo.` are all syntactic
   errors. Every segment between dots MUST contain at least one character.

3. **Characters outside the permitted set are forbidden.** Whitespace, punctuation not
   listed in §11.3.1, and any non-ASCII character are syntactic errors.

4. **Quoted-form identifiers are forbidden.** An `id:` value MUST be emitted and consumed
   in unquoted form only. Because the permitted `id:` character set excludes all reserved
   characters that would require quoting, quoted-form IDs are unnecessary and non-conformant
   in v0.1. A consumer encountering an `id:"..."` form MUST report a syntactic error.

5. **Leading or trailing dot is forbidden.** `.foo` and `foo.` are invalid (rule #2 case,
   restated for clarity).

6. **Uppercase-only policy is not enforced.** `id:` values MAY contain uppercase letters
   (see the Java example: `id:java.com.acme.order.OrderService`). This is not a forbidden
   pattern; it is an explicit permission to preserve source casing.

### 11.4 Field Names

Field names (in `[Fields]` and `[Returns]`) follow:

```
field_name = field_char_1 *field_char_n
field_char_1 = LOWER / "_"
field_char_n = LOWER / DIGIT / "_"
```

Rules:

- Lowercase ASCII letters, digits, and underscore only.
- MUST begin with a lowercase letter or underscore.
- Case-sensitive (though all lowercase by rule).
- No dots, hyphens, or other punctuation.
- No whitespace.

### 11.5 Meta Keys

Meta keys use the same grammar as field names (lowercase underscored). The canonical meta
keys are listed in §6.3.4. Profiles and dialects MAY register additional meta keys following
the same grammar.

A meta key is always followed immediately by a colon `:` with no whitespace. `cat:auth` is
valid; `cat :auth` and `cat: auth` are not.

### 11.6 Type Names

Type names fall into three classes, each with its own rules.

**Primitive type keywords** (`text`, `number`, `int`, `bool`, `date`, `url`, `image`,
`any`):

- Lowercase ASCII only.
- MUST match exactly one of the registered primitive keywords (§7.1).
- Unknown lowercase identifiers in a scalar type position are a syntactic error at profile
  conformance, warning at core conformance.

**Generic type constructor keywords** (`expr`, `list`, `record`, `map`):

- Lowercase ASCII only.
- MUST match exactly one of the registered generic constructors (§7.2).

**Reference type identifiers** (identifier ending in `_ref`):

- Lowercase ASCII only by convention, though the grammar permits source casing inside the
  identifier prefix.
- MUST begin with a letter (per §7.7).
- The `_ref` suffix is normative.

**Nominal type names** (used as arguments to generic constructors):

- Alphabetic first character (per §7.7).
- Alphanumeric and underscore thereafter; may include dots as segment separators.
- MAY preserve source casing: `User`, `OrderItem`, `dom_print_job`, `com.acme.Order`.
- Case-sensitive.

### 11.7 Reserved Identifiers

The following identifiers are reserved and MUST NOT be redefined or used in positions other
than their reserved role:

**Canonical meta keys** (§6.3.4):

```
cat, id, kind, facet, scope, context, target, requires, path, verb, status, since, sunset, until
```

**Named block markers** (§6.2):

```
[Rel], [Fields], [Behavior], [Returns]
```

**Annotation prefixes** (§10):

```
#note:, #warn:
```

**Sigil characters** (§5.1):

```
!, >, ^, @, +
```

**Reserved sigils** (§5.3):

```
?, %
```

**Primitive type keywords** (§7.1):

```
text, number, int, bool, date, url, image, any
```

**Generic type constructors** (§7.2):

```
expr, list, record, map
```

**Compound type keywords** (§7.4):

```
enum, field_changes, key_value_pairs, params
```

**The identifier suffix `_ref`** is reserved for reference types (§7.3). A custom type name
ending in `_ref` is interpreted as a reference type.

A profile or dialect MAY register additional reserved identifiers in its addendum. Profile-
or dialect-reserved identifiers MUST NOT collide with the universal reserved identifiers
listed above.

### 11.8 Summary Table

| Position             | Character set | Case rule | Starts with      | Section |
|----------------------|---------------|-----------|------------------|---------|
| Section name         | UPPER, DIGIT, `_` | Uppercase | UPPER or DIGIT   | §11.1   |
| Entry name           | ALPHA, DIGIT, `_`, `>` | Mixed or lower | Letter or digit | §11.2 |
| `id:` value          | ALPHA, DIGIT, `_`, `-`, `/`, `(`, `)`, `+`, `.` as separator | Mixed allowed | Letter, digit, or `_` | §11.3 |
| Field name           | lower, DIGIT, `_` | Lowercase | lower or `_`     | §11.4   |
| Meta key             | lower, DIGIT, `_` | Lowercase | lower or `_`     | §11.5   |
| Type keyword         | lower | Lowercase | Fixed keywords   | §11.6   |
| Nominal type         | ALPHA, DIGIT, `_`, `.` as separator | Mixed allowed | Letter | §11.6 |
| Reference type       | ALPHA, DIGIT, `_` ending in `_ref` | Mixed allowed | Letter | §11.6 |
| Profile/dialect name | lower, DIGIT, `_` | Lowercase | lower | §11.10 |

### 11.9 Open Questions for Future Revisions

- Whether entry names should be normatively restricted to a single casing style (Mixed vs
  lower) per profile, to prevent cross-family inconsistency.
- Whether `id:` grammar should be extended to permit non-ASCII Unicode identifiers, with a
  corresponding normalization rule (NFC-based comparison). v0.1 restricts to ASCII.
- Whether a more expressive ID syntax (e.g., URI-style schemes, DOI-like hierarchies) should
  be offered as an alternative form in specific profiles.

### 11.10 Profile and Dialect Names

Profile and dialect names follow the same grammar:

```
name = LOWER *(LOWER / DIGIT / "_")
```

Rules:

- Lowercase ASCII letters, digits, and underscore only.
- MUST begin with a lowercase letter.
- Case-sensitive (lowercase by rule).
- No dots, hyphens, or other punctuation.
- No whitespace.

Examples of valid profile and dialect names:

```
code_surface
api_contract
documentation_reference
platform_metamodel
schema_catalog
```

Examples of invalid profile and dialect names:

```
code-surface       (hyphen not permitted)
Code_Surface       (uppercase not permitted)
1surface           (must begin with a letter)
code.surface       (dot not permitted)
```

A syntactically invalid profile or dialect name on the header (§4.2.3, §4.2.4) is always a
syntactic error, regardless of validation mode.

---

# Part III — Semantics (Layer 1)

Part II defined what a JOOT document *looks like* as a byte sequence. Part III defines what
a JOOT document *means* once parsed. It specifies the identity model that makes subjects
addressable, the graph model that connects them, and (in §14) the core vocabularies that
populate categories, relation roles, and behavior atoms.

Layer 1 is the foundation that any useful consumer needs beyond mere parsing: without an
identity model, relation targets cannot resolve; without a graph model, the corpus is just
a list of lines; without core vocabularies, `cat:` and `[Rel]` roles have no interpretation.

This part is normative. Profiles (§15) and dialects (§16) extend Layer 1 by registering
additional vocabularies; they do not redefine the underlying model.

---

## 12. Identity Model

This section formalizes what makes a JOOT subject a first-class addressable entity. It
consolidates and completes the identity mechanics that §6.3, §6.4.4, and §11.3 anticipate,
and it closes the decision procedure between facet decomposition and child-subject
decomposition.

### 12.1 Subjects

A **subject** is a first-class addressable entity represented by one or more JOOT entries
sharing the same `id:`.

- A subject represented by **exactly one** entry is an **unfaceted subject**. Its single
  entry has no `facet:` meta key, and all the subject's meta, relations, fields, behavior,
  and returns are expressed on that one line.
- A subject represented by **two or more** entries is a **faceted subject**. Each entry has
  a distinct `facet:` meta value, and the subject's full specification is the logical union
  of all its facet entries (§12.3).

Unfaceted and faceted representations are mutually exclusive for any given `id:` (§12.3.2).
A single logical subject is either faceted or it is not; the two modes cannot coexist.

The subject is the unit of identity, cross-reference, deduplication, and corpus-level
uniqueness. Entries are the syntactic vehicle; subjects are the semantic atom.

### 12.2 Identity Uniqueness

Identity uniqueness applies at the `(id, facet)` tuple level. The rules are:

1. **Within a document.** Two entries with the same `(id:, facet:)` tuple are a syntactic
   duplicate and an identity uniqueness violation. For an unfaceted subject, the tuple is
   `(id, ∅)` where `∅` denotes absence of `facet:`. Two unfaceted entries with the same
   `id:` are duplicates. Two faceted entries with the same `id:` but different `facet:`
   are distinct facets and are permitted.

2. **Within a corpus.** Under corpus conformance (§1.2.3), `(id, facet)` uniqueness applies
   across the entire corpus. The same subject MUST NOT be re-declared in two different
   documents with conflicting content. A corpus validator MUST detect and report
   duplicates across documents.

3. **Comparison rules.** `id:` values are compared byte-wise exact, case-sensitive, per
   §11.3.2. `facet:` values are compared the same way.

4. **Absent `id:`.** An entry without an `id:` meta key is anonymous. Anonymous entries do
   not participate in identity uniqueness and cannot be addressed by relation targets.
   Under profile conformance, most profiles require `id:` on every entry (§15); core
   conformance does not.

**Outcomes of violation.**

| Situation                                                    | Outcome                         |
|--------------------------------------------------------------|---------------------------------|
| Two unfaceted entries with same `id:`                        | Identity uniqueness violation — error under profile/corpus |
| Two faceted entries with same `(id, facet)`                  | Identity uniqueness violation — error under profile/corpus |
| One unfaceted entry + one or more faceted entries, same `id:` | Identity modeling conflict — error under profile/corpus (§12.3.2) |
| Two faceted entries, same `id:`, distinct `facet:`           | Valid — distinct facets of the same subject |

### 12.3 Facets

A **facet** is an orthogonal projection of the same logical subject. Facets exist to
decompose a subject that has multiple independent dimensions (e.g., an API endpoint has a
request side and a response side; a UI widget has a structural side and a styling side).
Facets let each dimension be expressed on its own line without inventing separate subjects.

The rules governing facets are the most delicate in the identity model, because facets
deliberately relax the "one subject = one entry" assumption. Getting them wrong leads to
corpora that parse but don't make sense.

#### 12.3.1 Facet Mechanics

A faceted subject is declared by emitting two or more entries with:

- the same `id:` meta value, and
- distinct `facet:` meta values.

Each facet entry carries its own `cat:` and `kind:` (subject to §12.3.4), its own `[Rel]`,
`[Fields]`, `[Behavior]`, and `[Returns]` blocks, and its own annotations. The facets are
parsed independently and their content is logically unioned to form the complete picture
of the subject.

Example (illustrative):

```
!http.orders.get|cat:endpoint|id:http.orders.get|facet:request|[Fields]...
!http.orders.get|cat:endpoint|id:http.orders.get|facet:response|[Returns]...
!http.orders.get|cat:endpoint|id:http.orders.get|facet:error|[Returns]...
```

These three entries together describe the single logical endpoint `http.orders.get`. Each
facet carries what belongs to it: the request shape on the `request` facet, the response
shape on the `response` facet, and the error surface on the `error` facet.

Resolution of internal relation targets against this subject follows the rules of §6.4.4:
a bare `<id>` target binds to the logical union; a `<id>#<facet>` target binds to a
specific facet projection.

#### 12.3.2 Unfaceted + Faceted Coexistence Is Prohibited

A subject MUST be either fully unfaceted (one entry with no `facet:`) or fully faceted
(two or more entries, each with a distinct `facet:`). Mixing the two is a semantic error:

- An unfaceted entry claims to describe the subject completely in one line.
- A faceted entry claims to describe one projection of the subject.

These claims are mutually incompatible. A validator MUST reject documents where a single
`id:` appears on both an unfaceted entry and one or more faceted entries (§6.4.4).

If a producer needs to migrate a subject from unfaceted to faceted (because new
decomposition dimensions emerge), the unfaceted entry MUST be replaced in a single atomic
change. The old entry is removed; the new facet entries are added. No interim state with
both is permitted in a conformant document.

#### 12.3.3 Forbidden Facet Patterns

The `facet:` meta key exists to decompose a subject along **orthogonal dimensions**, not to
name a **child subject**. The following patterns are forbidden and MUST NOT appear as
`facet:` values:

1. **Child-as-facet.** A `facet:` value naming a distinct sub-entity that should itself be
   a first-class subject. Examples of what NOT to do:
   - `facet:order_items` on an `Order` subject (order items are their own subjects).
   - `facet:admin_user` on a `User` subject (admin user is a kind of user, not a facet).
   - `facet:usa_customer` on a `Customer` subject (regional customer is not an orthogonal
     projection).

2. **Instance-as-facet.** A `facet:` value naming a specific instance or example of the
   subject. Instances are data, not structural projections. A `Product` subject does not
   have a `facet:iphone_15` facet.

3. **Version-as-facet.** A `facet:` value naming a version or revision of the subject.
   Versions are expressed via `since:`, `sunset:`, and `until:` meta keys, or via distinct
   `id:` values for breaking changes. `facet:v2` is forbidden.

4. **Status-as-facet.** A `facet:` value naming a lifecycle status. Status is expressed in
   `status:` meta. `facet:deprecated` is forbidden.

5. **Environment-as-facet.** A `facet:` value naming a deployment environment (production,
   staging, test). Environments are `scope:` or `context:` meta values, not facets.

**The test for a valid facet value:** if the alternative decomposition is "this is a
different subject on its own", then it is a child subject, not a facet. Emit it as a
separate entry with its own `id:` and link it via `[Rel]` (§13).

**The test for an invalid facet value:** if the `facet:` value answers "which variant?"
rather than "which projection of the same thing?", it is forbidden.

#### 12.3.4 Facet Consistency Constraints

All facets of the same subject MUST satisfy these consistency rules:

1. **Shared `cat:`.** All facets of the same `id:` MUST declare the same `cat:` value. A
   subject is of one category; its facets project different dimensions of that one thing.

2. **Shared `kind:`.** All facets of the same `id:` MUST declare the same `kind:` value.
   Facets project a subject; they do not specialize it.

3. **Distinct `facet:`.** No two facets of the same `id:` MAY share a `facet:` value (this
   is the `(id, facet)` uniqueness rule of §12.2, restated).

4. **Independent meta (profile-restrictable).** Facets MAY carry independent `scope:`,
   `context:`, `target:`, `requires:`, `path:`, `verb:`, `status:`, `since:`, `sunset:`,
   `until:` values unless restricted by an active profile. The core rule is that these
   keys are not required to match across facets, because they may genuinely differ per
   projection. Profiles MAY require equality across facets for specific keys where
   projection semantics demand it (for example, `api_contract` requires all facets of
   the same endpoint to share identical `path:` and `verb:` values, since an endpoint's
   request, response, and error facets address the same HTTP surface).

5. **Independent blocks.** Each facet's `[Rel]`, `[Fields]`, `[Behavior]`, and `[Returns]`
   blocks are independent. A relation declared on one facet is an edge from that facet;
   a field declared on one facet belongs to that facet's structural projection.

Violation of rules 1–3 is an error under profile and corpus conformance.

#### 12.3.5 Decision Procedure: Facet or Child Subject?

Producers frequently face the decision: "Should this sub-structure be a facet of the
parent, or a separate subject linked by `[Rel]`?" The decision is not arbitrary. The
following ordered procedure MUST be used to decide:

**Step 1: Does it have its own identity?**

If the sub-structure has a name by which it is referenced from elsewhere (other subjects
link to it, tooling queries it individually, humans refer to it by name), it has its own
identity. It MUST be a separate subject. Link with `[Rel]`.

Examples:
- `OrderItem` is referenced by `Order`, by `Invoice`, by `ShipmentLine`. Independent
  identity. Separate subject.
- An API endpoint's `headers` collection is not referenced from elsewhere; it's a property
  of the endpoint. No independent identity. Candidate for facet (or field, Step 4).

**Step 2: Is it an orthogonal projection, or a specialization?**

If the sub-structure is a **projection of the same thing from a different angle** (request
view, response view, error view; structural view, behavioral view; input view, output
view), it is a facet. Emit as a facet of the parent.

If the sub-structure is a **kind of the thing**, or a **variant**, or a **version**, or a
**role**, it is a specialization. It MUST be a separate subject. Link with `[Rel]` using
an appropriate role (typically `refines`, `specializes`, or `variant_of`).

Examples:
- `request` and `response` are orthogonal projections of an endpoint. Facets.
- `AdminUser` is a kind of `User`. Separate subject.
- `v2` is a version of the subject. Separate subject; link with `supersedes`.

**Step 3: Does it share `cat:` and `kind:` with the parent?**

If yes, it can be a facet. If no (different category, different kind), it cannot be a
facet — facets share `cat:` and `kind:` with their subject (§12.3.4). It MUST be a separate
subject.

**Step 4: Can it be expressed as a field or a constraint on the parent?**

If the sub-structure is a simple property (a scalar value, a short list, a bounded
enumeration), it belongs in `[Fields]` or `[Behavior]` of the parent. Not a facet, not a
separate subject.

Example:
- An endpoint's `auth:bearer` is a property. Belongs in `[Behavior]`, not a facet.
- An endpoint's `request` shape (with multiple fields, constraints, conditional members) is
  structural and deserves its own facet.

**Summary table:**

| Situation                                                              | Decision                |
|------------------------------------------------------------------------|-------------------------|
| Has own identity (referenced by name from elsewhere)                   | Separate subject + `[Rel]` |
| Kind of / variant of / version of the parent                           | Separate subject + `[Rel]` |
| Different `cat:` or `kind:` from the parent                            | Separate subject + `[Rel]` |
| Orthogonal projection of the same subject, same `cat:`/`kind:`         | Facet                   |
| Simple property (scalar, short list, enumeration)                      | Field or behavior on parent |

A profile MAY refine this decision procedure for its domain. A profile MUST NOT override
the core rule that facets share `cat:` and `kind:` with their subject.

#### 12.3.6 Facet Naming

A `facet:` value is a lowercase underscored identifier:

```
facet_value = LOWER *(LOWER / DIGIT / "_")
```

Rules:

- Lowercase ASCII letters, digits, and underscore only.
- MUST begin with a lowercase letter.
- Case-sensitive.
- No dots, no hyphens, no whitespace.

Facet values SHOULD be concise and descriptive of the projection they represent. Common
facet value examples:

```
request, response, error, headers, params, body,
structural, behavioral, styling, visual,
input, output, state, schema,
create, read, update, delete
```

A profile MAY register a canonical set of facet values for specific `cat:` or `kind:`
combinations. For example, the `api_contract` profile registers `request`, `response`, and
`error` as the canonical facets of an endpoint subject.

### 12.4 Identity Continuity Across Revisions

Identity is a contract with consumers and with future versions of the corpus. While §11.3.3
specifies the **grammatical** stability of the `id:` string, this section specifies the
**semantic** stability of the subject the `id:` refers to.

- An `id:` identifies the same logical subject across time. A consumer that resolved
  `com.acme.order.Order` yesterday MUST be able to resolve it today to the same conceptual
  entity, even if the entry has been revised.
- **Identity-preserving revisions** do not require changing the `id:` (see §11.3.3 for the
  enumerated list).
- **Identity-changing revisions** SHOULD trigger a new `id:`: splitting, merging, or
  fundamentally changing the semantic role of the subject (see §11.3.3).
- **Deprecation** is expressed with `status:deprecated` and, when a replacement exists,
  with a `[Rel]replaced_by:<new_id>` edge. The deprecated entry SHOULD remain in the
  corpus for at least one revision cycle after the new entry is published, to preserve
  backward resolution.
- **Removal** is expressed with `status:removed` on the last revision containing the entry,
  followed by complete deletion in the subsequent revision. A corpus MAY retain a
  tombstone entry (a minimal entry with `status:removed` and a `[Rel]replaced_by:` edge if
  applicable) to aid archival resolution.

### 12.5 Corpus Boundaries

A **corpus** is a set of JOOT documents treated as a single validation scope. Corpus
boundaries are declared out-of-band by the consumer or validator: a build configuration, a
CLI invocation, a repository layout. This specification does not prescribe how corpus
membership is determined.

Corpus boundaries matter because:

- `(id, facet)` uniqueness is validated across the corpus (§12.2).
- Internal relation targets resolve across the corpus (§6.4.4 under corpus mode).
- Profile conformance is declared per-document; corpus conformance requires profile
  conformance of each member plus cross-document consistency.

A producer SHOULD choose corpus boundaries that are stable, semantically coherent, and
useful to consumers (e.g., "all JOOT documents describing the Acme Order Service",
"all JOOT documents under this Git repository's /joot directory").

A subject's `id:` SHOULD be unique within its corpus. Two different corpora MAY use the
same `id:` for genuinely different subjects (e.g., two organizations both using
`com.acme.Order` for distinct `Order` concepts), but cross-corpus interoperability then
requires explicit namespace mapping by the consumer.

### 12.6 Anonymous Entries

An entry without an `id:` meta key is **anonymous**.

- Anonymous entries are permitted at core conformance.
- Anonymous entries are forbidden at profile conformance under all standard profiles.
- Anonymous entries cannot be targeted by relations; their lack of identity precludes
  resolution.
- Anonymous entries do not participate in `(id, facet)` uniqueness.
- An entry without `id:` MUST NOT declare `facet:`. A `facet:` meta key has no semantic
  value in the absence of `id:`, because facets are projections of a logical subject and
  no logical subject exists without an identifier. Combining `facet:` with an absent
  `id:` is a syntactic error at every conformance level.

Anonymous entries exist to support exploratory authoring and casual documentation. Any
production-grade JOOT corpus is expected to activate a profile, which eliminates anonymous
entries.

### 12.7 Open Questions for Future Revisions

- Whether a formal **facet vocabulary registry** (like `cat:` and relation roles) should be
  introduced in v0.2, so that facet values are closed under profile conformance.
- Whether **cross-corpus identity** (a mechanism for authoritative `id:` ownership across
  independent corpora) should be addressed in a future revision — analogous to how package
  names in programming languages encode organizational namespaces.
- Whether **subject aliasing** (two `id:` values that are declared equivalent) should be
  supported in v0.2 for migration scenarios.
- Whether an explicit **parent relation** (the subject that this subject's identity nests
  under, e.g., "this method belongs to that class") should be elevated from ordinary `[Rel]`
  to a dedicated meta key. v0.1 keeps it in `[Rel]member_of:` universally.

---

## 13. Graph Model

This section defines the graph structure implicit in every JOOT corpus: what constitutes an
edge, how edges are typed and directed, how paths are traversed, and how cycles,
unresolved targets, and edge cardinality are handled.

Part II (§6.4) defined the **syntax** of `[Rel]`. §13 defines its **semantics**.

### 13.1 The Corpus as a Directed Graph

Every JOOT corpus is implicitly a **directed, typed, multi-edge graph**:

- **Nodes** are subjects (§12.1). A node may have one or more entries behind it (one entry
  for unfaceted subjects, multiple for faceted).
- **Edges** are relation items. Each edge is directed, typed by a relation role, and
  annotated with optional constraints.
- The graph is a **multigraph**: multiple distinct edges MAY exist between the same pair
  of nodes, provided they have different roles, different targets-with-facets, or
  different constraint annotations (see §13.4).

The model is comparable to a directed labeled graph and can be projected into triple-like
representations by consumers that require them. JOOT consumers building a knowledge graph
from a corpus MAY construct it directly from `[Rel]` blocks without additional inference.

### 13.2 Edge Semantics

Every edge has four components:

1. **Source node.** The subject that owns the `[Rel]` block in which the edge is declared.
   For a faceted subject, the source is the specific facet that declares the edge.

2. **Role.** The typed label of the edge, drawn from the universal role registry (§14.2)
   or from a profile/dialect addendum.

3. **Target.** Either an internal target (another subject or facet in the scope) or an
   external target (`ext:<ext_ref>`).

4. **Constraints.** An optional `{...}` block qualifying the edge (§8). Common constraint
   uses on edges: conditional applicability (`{when_feature_flag_x}`), scope restriction
   (`{server_side_only}`), or role-specific qualifications registered by profiles.

Edges are **one-directional**. A declaration of `member_of:java.com.acme.order` on
`!OrderService` states that `OrderService` is a member of `java.com.acme.order`; it does
not automatically create a reverse edge declaring that `java.com.acme.order` contains
`OrderService`. If both directions are needed, both edges MUST be declared.

This asymmetry is intentional: it makes the graph precisely what the source document
claims, without inference. Profile-specific inverse-edge derivation is possible but is a
validator service, not a JOOT language feature.

### 13.3 Source: Subject-Level vs Facet-Level

When the source subject is faceted, edges are declared on individual facets, not on the
subject as a whole. The source of an edge is always the specific entry (and thus facet)
whose `[Rel]` block contains it.

Two facets of the same subject MAY declare different edges. An endpoint's `request` facet
might declare `request:OrderRequest{body}`, while its `response` facet declares
`response:OrderResponse{body}`. Each edge's source is the specific facet.

When consumers need "all edges from this subject", they take the union of `[Rel]` blocks
across all facets. This is a consumer-side aggregation, not a JOOT-defined operation, but
validators and graph builders commonly perform it.

### 13.4 Edge Identity and Duplication

Two edges are **identical** if and only if all four components match exactly:

- same source node (same facet if source is faceted),
- same role,
- same target (including facet component and internal-vs-external distinction),
- same constraint set (atom-for-atom equal after canonical ordering).

Identical edges within the same `[Rel]` block are prohibited (§6.4.5). Identical edges
across different facets of the same source subject are permitted but redundant; producers
SHOULD avoid them.

Two edges with the same source, role, and target but **different constraints** are
distinct edges. For example, two `depends_on:OrderService` edges with different `when_*`
constraints declare two conditional dependencies, not a duplicate.

### 13.5 Target Resolution

Target resolution is specified in full in §6.4.4. The semantic interpretation is:

- A successfully resolved internal target binds the edge to a concrete node in the corpus
  graph. The edge is now a realized connection.
- An unresolved internal target in core or profile mode (document scope) leaves the edge
  **dangling**. The edge's role and target string are retained; the edge exists in the
  graph but points to an unknown node. Consumers MAY treat dangling edges as placeholders
  for expected-but-absent subjects.
- An unresolved internal target in corpus mode is an error. The graph is malformed.
- An external target (`ext:<ext_ref>`) is always "unresolved" from the perspective of the
  active corpus, but it is not dangling: it is an intentional reference to an entity
  outside the validation scope. External-target edges are part of the graph and carry
  semantic weight (typically "depends on this external thing"), but they do not resolve to
  an in-corpus node.

### 13.6 Cycles

JOOT graphs MAY contain cycles. The format does not forbid them.

Cycles arise naturally in:

- mutual recursion between types (`A` has field `b:record<B>`; `B` has field `a:record<A>`);
- bidirectional relations (`A` contains `B`; `B` references back to `A`);
- dependency loops in workflow or state-machine subjects.

Consumers that traverse the graph (for serialization, analysis, or code generation) MUST be
cycle-safe. JOOT does not impose a cycle-detection obligation on producers, except where a
specific profile declares a role or relation pattern as necessarily acyclic (see §13.8).

Under corpus conformance, the validator is not required to detect cycles. A future revision
MAY introduce a dedicated cycle-safety conformance check for specific role classes.

### 13.7 Path Traversal

A **path** is a sequence of edges `e_1, e_2, ..., e_n` where the target of `e_i` is the
source of `e_{i+1}`. Path traversal is a consumer operation, not a JOOT-defined primitive.
However, the specification makes the following guarantees to consumers:

1. **Determinism.** Given the same corpus and the same starting subject, path traversal
   produces a deterministic ordering when edges are emitted in canonical order (§6.4.2).
   A canonical corpus yields canonical traversal.

2. **Termination.** Path traversal terminates for a bounded-depth query, regardless of
   cycles, provided the consumer implements standard cycle-safety (e.g., visited-set
   tracking).

3. **Completeness.** All edges declared in the corpus are visible to the traversal. JOOT
   defines no hidden, inferred, or implicit edges; the graph is exactly the union of
   `[Rel]` declarations.

Profile-specific traversal semantics (e.g., "resolve `extends` chains to the root class")
belong to the profile or to the consumer, not to the core specification.

### 13.8 Edge Cardinality

Edge cardinality is declared at the role level, not at the edge level. §6.4.5 specified
the cardinality declaration model. The semantic consequences are:

- A role with cardinality `1` MUST have exactly one edge per source node (per facet if
  source is faceted). Zero edges or two or more edges are errors under profile conformance.
- A role with cardinality `0..1` MAY have zero or one edge.
- A role with cardinality `1..n` MUST have at least one edge.
- A role with cardinality `0..n` (the default) has no restriction on count.

Cardinality applies **per source node**, not globally. A role with cardinality `1..n` does
not require that the corpus as a whole contain at least one edge of that role; it requires
that each source node that declares the role has at least one edge if the role is
applicable to that node.

Profiles MAY declare cardinality conditionally (e.g., "`response` has cardinality `1..n`
for endpoints with `verb:GET` and `0..n` for `verb:HEAD`"). Such conditional cardinality
rules are profile-specific and MUST be documented in the profile's addendum.

### 13.9 Role Compatibility

Not every role is applicable to every source-target subject pair. A profile declares, for
each role it registers:

1. **Source type constraints.** Which `cat:` / `kind:` values the source subject must
   have. For example, `member_of` is applicable to types and callables, not to events.

2. **Target type constraints.** Which `cat:` / `kind:` values the target subject must
   have. For example, in `api_contract`, the `request`, `response`, and `error` roles
   target either `contract` subjects (linked-contract form) or endpoint facets of the
   same logical endpoint (endpoint-facet form), as defined by the profile (§15.3).

3. **Scope constraints.** Whether internal-only, external-only, or either target type is
   permitted (§6.4.3).

A profile MUST document these compatibility rules for each role it registers. A validator
under profile conformance MUST enforce them.

Under core conformance, role compatibility is not enforced; any role MAY connect any two
subjects, with only the universal syntactic rules applying.

### 13.10 Reverse Edges and Inverse Relations

JOOT does not define **automatic inverse edges**. A declaration of `extends:Base` on
`!Derived` does not create an implicit `extended_by:Derived` edge on `!Base`.

Some profiles or consumers benefit from inverse views (e.g., "show me all subjects that
extend Base"). Profiles MAY define a derived-edge computation as part of their consumer
semantics, but the derived edges are not part of the document's declared graph.

A producer that wants the inverse edge to be part of the corpus MUST declare it explicitly.
This is the same principle as §13.2: the graph is exactly what is declared, nothing more.

### 13.11 Edge Weight and Ranking

JOOT edges are **unweighted** in the core model. There is no numerical weight, strength,
or priority attached to an edge by the core grammar.

When a domain needs edge weighting, it is expressed via edge constraints:

```
[Rel]uses:LibraryA{weight:0.8,since:v2.1}
```

The `weight:0.8` here is a constraint on the edge, interpreted by the consumer. It is not a
universal primitive.

Profiles MAY register `weight:`, `priority:`, `rank:`, or similar numeric constraint keys
for their roles. These are profile-specific extensions, not core features.

### 13.12 Graph Operations Defined Elsewhere

The following graph operations are mentioned throughout the specification and are grouped
here for reference. Each is defined in its dedicated section:

| Operation              | Defined in |
|------------------------|------------|
| Target resolution      | §6.4.4     |
| Cardinality enforcement | §6.4.5, §13.8 |
| Role registration      | §14.2, §15.2 |
| Role compatibility     | §13.9, §15 |
| Canonical edge ordering | §6.4.2     |
| Identity uniqueness    | §12.2      |
| Facet-level source     | §13.3      |

### 13.13 Open Questions for Future Revisions

- Whether automatic **inverse-edge derivation** should be standardized for specific roles
  (e.g., `extends` / `extended_by`, `member_of` / `has_member`) as a consumer service,
  with a registered inverse-role mapping table.
- Whether **edge identity** (a syntactic way to reference a specific edge, for
  annotations or for external cross-reference) should be introduced in a future revision.
  v0.1 edges are anonymous.
- Whether **graph metrics** (depth limits, fan-out limits, size thresholds) should be
  introduced as conformance checks in specific profiles.
- Whether **transitive role classes** (roles where the transitive closure is
  well-defined, e.g., `extends`) should be formally registered as such in the role
  registry.

---

## 14. Core Vocabularies

This section defines the universal vocabularies that populate `cat:`, relation roles, and
the behavior block in JOOT. The vocabularies are deliberately small and transversal. Each
entry must be meaningful across domains without requiring a profile; domain-specific
content belongs to §15 (profiles) and §16 (dialects).

§14 is the reference registry for Layer 1. Profiles close these vocabularies; dialects
extend them. Core conformance treats them as open (additions permitted with warning);
profile conformance treats them as closed unless the profile declares otherwise.

### 14.1 Universal Category Registry (`cat:`)

The `cat:` meta field classifies the kind of subject. The following categories are
normatively registered as universal.

| `cat:` value | Meaning                                                          |
|--------------|------------------------------------------------------------------|
| `module`     | A namespace, package, module, or logical container of other subjects. |
| `type`       | A nominal type: class, interface, struct, trait, protocol, enum type, record type. |
| `member`     | A field, property, constant, or slot declared inside a type or module. |
| `callable`   | A function, method, constructor, operator, or any invocable subject. |
| `source`     | A data source, provider, input, or origin of values (user-driven or system-driven). |
| `schema`     | A data schema: table, collection, document shape, database definition. |
| `endpoint`   | A network-addressable surface: HTTP endpoint, RPC method, webhook, socket. |
| `contract`   | A reusable interchange contract: request/response shape, message body, payload definition. |
| `event`      | An event, trigger, hook, lifecycle callback, or notification. |
| `state`      | A state in a state machine, workflow, or protocol. |
| `transition` | A transition between states, guarded or unguarded. |
| `step`       | A step in a workflow, pipeline, or multi-stage process. |
| `example`    | A concrete example, sample, or canonical instance of another subject. |
| `test`       | A test case, assertion, or conformance scenario. |
| `metric`     | A measurement, KPI, or observability signal. |

**Rules.**

1. The `cat:` value categorizes the logical kind of subject. It is the broadest
   classification; finer distinctions belong to `kind:` (§6.3.4) and to profile-registered
   subcategories (§15).
2. A profile MAY register additional `cat:` values in its addendum. Profile-registered
   categories apply only under that profile; they are not universal.
3. A profile MAY declare its `cat:` vocabulary closed (only universal + profile-registered
   values accepted) or open (other values accepted with warning).
4. Under core conformance, unknown `cat:` values are accepted with a warning.
5. Each `cat:` value MUST map to one logical kind. A subject is of exactly one `cat:`; it
   cannot simultaneously be `type` and `callable` (see §14.1.1 for guidance on ambiguous
   cases).

#### 14.1.1 Category Selection Guidance

When a subject plausibly fits two categories, choose the category that best describes its
**primary role** in the corpus graph. A callable that also defines a type is a `callable`;
the type it defines is a separate subject with `cat:type`. A state that also acts as an
event trigger is a `state`; the event is separate.

Values explicitly **not** universal and left to profiles or `kind:`:

- `class`, `interface`, `struct`, `trait`, `protocol` → use `cat:type` with `kind:class`,
  `kind:interface`, etc.
- `table`, `view`, `column`, `index` → use `cat:schema` with `kind:table`, etc.
- `ui_element`, `component`, `widget` → profile-specific (e.g., UI dialect).
- `request`, `response`, `error` → NOT categories; these are **facets** of an endpoint
  (§12.3) or **kinds** of a contract.

### 14.2 Universal Relation Role Registry

The following relation roles are normatively registered as universal. Each role has:
**Meaning**, **Source** (permitted `cat:` of the source subject), **Target** (permitted
`cat:` of the target subject), **Default cardinality**, and **`ext:` default** (whether
external targets are permitted by default without profile opt-in).

| Role               | Meaning                                                       | Source               | Target                           | Cardinality | `ext:` default |
|--------------------|---------------------------------------------------------------|----------------------|----------------------------------|-------------|----------------|
| `member_of`        | Source is a member of target (type-of-member relationship).   | `type`, `member`, `callable` | `type`, `module`       | `0..1`      | yes            |
| `part_of`          | Source is structurally part of target (composition).          | any                  | any                              | `0..n`      | yes            |
| `owns`             | Source owns target (lifetime/ownership).                      | any                  | any                              | `0..n`      | no             |
| `imports`          | Source imports target (module-level dependency).              | `module`, `type`     | `module`, `type`                 | `0..n`      | yes            |
| `exports`          | Source exports target (makes target visible).                 | `module`             | any                              | `0..n`      | no             |
| `depends_on`       | Source depends on target to function.                         | any                  | any                              | `0..n`      | yes            |
| `references`       | Source references target without strong dependency.           | any                  | any                              | `0..n`      | yes            |
| `extends`          | Source extends target (inheritance).                          | `type`               | `type`                           | `0..n`      | yes            |
| `implements`       | Source implements target (interface conformance).             | `type`               | `type`                           | `0..n`      | yes            |
| `conforms_to`      | Source conforms to target contract or specification.          | any                  | `contract`, `schema`, `type`     | `0..n`      | yes            |
| `annotated_by`     | Source is annotated/decorated by target.                      | any                  | `type`, `callable`               | `0..n`      | yes            |
| `calls`            | Source (callable) calls target (callable).                    | `callable`           | `callable`                       | `0..n`      | yes            |
| `reads`            | Source reads from target (data access, non-mutating).         | `callable`, `step`   | `schema`, `source`, `member`     | `0..n`      | yes            |
| `writes`           | Source writes to target (data mutation).                      | `callable`, `step`   | `schema`, `source`, `member`     | `0..n`      | yes            |
| `throws`           | Source throws/raises target as an error.                      | `callable`, `step`   | `type`, `contract`               | `0..n`      | yes            |
| `request`          | Source declares target as its request-side shape.             | `endpoint`           | `contract`, or endpoint `#request` facet | `0..1` | no  |
| `response`         | Source declares target as its response-side shape.            | `endpoint`           | `contract`, or endpoint `#response` facet | `0..n` | no |
| `error`            | Source declares target as a possible error response.          | `endpoint`           | `contract`, or endpoint `#error` facet | `0..n` | no   |
| `from`             | Transition origin: source transitions from target state.      | `transition`         | `state`                          | `1`         | no             |
| `to`               | Transition destination: source transitions to target state.   | `transition`         | `state`                          | `1`         | no             |
| `next`             | Sequential successor: source step precedes target step.       | `step`, `state`      | `step`, `state`                  | `0..n`      | no             |
| `on_fail`          | Fallback path: source transitions to target on failure.       | `step`, `state`, `transition` | `step`, `state`         | `0..1`      | no             |
| `emits`            | Source emits target event.                                    | any                  | `event`                          | `0..n`      | yes            |
| `consumes`         | Source consumes (listens to) target event.                    | `callable`, `step`   | `event`                          | `0..n`      | yes            |
| `for`              | Source is defined for target subject (example-for, test-for). | `example`, `test`, `metric` | any                        | `0..n`      | yes            |
| `compatible_with`  | Source is compatible with target (version/contract).          | any                  | any                              | `0..n`      | yes            |
| `incompatible_with`| Source is incompatible with target.                           | any                  | any                              | `0..n`      | yes            |
| `supersedes`       | Source supersedes target (successor relationship).            | any                  | any (same `cat:` typically)      | `0..n`      | yes            |
| `replaced_by`      | Source is replaced by target (predecessor relationship).      | any                  | any (same `cat:` typically)      | `0..1`      | yes            |
| `refines`          | Source refines target (more specific version of).             | any                  | any (same `cat:` typically)      | `0..1`      | yes            |
| `specializes`      | Source specializes target (kind-of relationship).             | any                  | any (same `cat:` typically)      | `0..1`      | yes            |

**Rules.**

1. "Source: any" means the role accepts any `cat:` value for the source subject. "Target:
   any" means the role accepts any `cat:` value for the target subject. Specific `cat:`
   lists are constraints.
2. **Default cardinality** is the role's baseline cardinality when profiles do not override
   it. A profile MAY **narrow** a default cardinality (e.g., from `0..n` to `0..1`, or
   from `0..1` to `1`). A profile MAY **widen** a default cardinality (e.g., from `0..1`
   to `0..n`) only when the profile addendum explicitly documents the reason and the
   resulting interoperability impact.
3. **`ext:` default = yes** means external targets (`ext:<ref>`) are permitted without a
   profile opt-in. **`ext:` default = no** means external targets require explicit profile
   permission (§6.4.3).
4. Roles marked "Source: any" or "Target: any" are maximally flexible; profiles SHOULD
   narrow these for clarity in their domain.
5. Profiles MAY register additional relation roles in their addendum. Registered
   profile-specific roles sort alphabetically alongside universal roles for canonical
   ordering (§6.4.2).
6. A profile MUST NOT redefine the meaning of a universal role. It MAY narrow the role's
   source/target constraints, tighten its cardinality, or restrict `ext:` permission.

#### 14.2.1 Roles Deliberately Not Universal

The following roles were considered and **excluded** from the universal registry, because
their meaning varies too much across domains or overlaps with narrower roles:

- `uses` — too vague; use `calls`, `reads`, `depends_on`, or `references` instead.
- `handles` — ambiguous between "processes" (callable) and "catches" (error); use
  `consumes` (for events) or `throws` (for errors).
- `serves` — too narrow to HTTP; profiles MAY register it.
- `wraps` — usually covered by `implements` or `refines`.
- `contains` — usually covered by `part_of` (reverse) or `owns`.
- `links_to` — too generic; use `references`.

Profiles MAY register any of these names if the domain warrants a specific meaning.

### 14.3 Universal Behavior Vocabulary

The `[Behavior]` block supports bare atoms (single-token properties) and keyed descriptors
(`key:value` pairs). The following are normatively registered as universal.

#### 14.3.1 Universal Bare Atoms

| Atom          | Meaning                                                          |
|---------------|------------------------------------------------------------------|
| `async`       | Subject executes asynchronously or produces a future/promise.    |
| `awaitable`   | Subject is awaitable in the host language's concurrency model.   |
| `ordered`     | Subject relies on or produces an ordering that consumers must respect. |
| `initial`     | Subject is an initial state, step, or entry point.              |
| `terminal`    | Subject is a terminal state, step, or exit point.               |
| `single_use`  | Subject may be invoked or entered at most once per lifecycle.    |
| `retriable`   | Subject may be safely retried on failure.                        |
| `destructive` | Subject irreversibly deletes, overwrites, or invalidates data.   |
| `safe`        | Subject is read-only and has no side effects (stronger than `idempotent`). |
| `idempotent`  | Subject may be re-invoked with the same arguments without additional effect. |
| `cacheable`   | Subject's output may be cached by consumers.                     |

#### 14.3.2 Universal Keyed Descriptors

| Key              | Allowed values                                                 | Meaning                                    |
|------------------|----------------------------------------------------------------|--------------------------------------------|
| `vis:`           | `public` / `protected` / `private` / `internal`                | Visibility.                                |
| `mut:`           | `mutable` / `readonly` / `immutable`                           | Mutability stance.                         |
| `bind:`          | `instance` / `static` / `class`                                | Binding or dispatch style.                 |
| `effect:`        | `pure` / `reads_state` / `writes_state` / `transactional` / `io` / `network` / `cpu_bound` / `io_bound` | Effect classification. |
| `pre:`           | `<atom>`                                                       | Precondition atom (profile- or dialect-registered). |
| `post:`          | `<atom>`                                                       | Postcondition atom.                        |
| `inv:`           | `<atom>`                                                       | Invariant atom.                            |
| `auth:`          | `none` / `basic` / `bearer` / `mtls`                           | Authentication requirement.                |
| `requires_role:` | `<atom>`                                                       | Required authorization role.               |
| `complexity:`    | `<atom>`                                                       | Informal complexity class (e.g., `o_1`, `o_n`, `o_log_n`). |
| `latency_p50_ms:`| `<number_literal>`                                             | Median latency in milliseconds.            |
| `latency_p95_ms:`| `<number_literal>`                                             | 95th percentile latency in milliseconds.   |
| `throughput_rps:`| `<number_literal>`                                             | Throughput in requests per second.         |
| `cost:`          | `<atom>`                                                       | Informal cost tier (e.g., `low`, `high`).  |

**Rules.**

1. Bare atoms and keyed descriptors MUST follow the grammar of §6.6.1 and the vocabulary
   discipline of §6.6.3 (registered relation roles MUST NOT appear as behavior keys).
2. `effect:` values compose additively when a profile permits comma-separated lists; v0.1
   core permits exactly one `effect:` value per behavior item. Profiles MAY extend.
3. `pre:`, `post:`, `inv:` carry opaque atoms whose semantics are profile-registered or
   application-defined. Core does not interpret them.
4. Numeric descriptors (`latency_*`, `throughput_rps`) carry `number_literal` values per
   §8.1; units are encoded in the key name (no separate unit atom).
5. A profile MAY narrow the allowed values of a keyed descriptor (e.g., restrict `auth:`
   to `bearer` and `mtls` in an API contract profile) but MUST NOT redefine the key's
   meaning.
6. A profile MAY register additional bare atoms and keyed descriptors. These extensions
   apply only under the profile.

#### 14.3.3 Explicitly Out of Scope for Universal

The following categories of behavior descriptors are **not** registered universally and
belong in profile or dialect addenda:

- **Business semantics** (e.g., `billing_tier:`, `customer_type:`): domain-specific.
- **Vendor authentication schemes** (e.g., `auth:firebase`, `auth:aws_sig_v4`): dialect.
- **Framework behaviors** (e.g., `lifecycle:react_mount`, `handler:express`): dialect.
- **Interpretive or editorial qualifiers** (e.g., `quality:high`, `risk:critical`):
  subjective, profile-specific if needed.

### 14.4 Rules for Profile and Dialect Additions

Profiles (§15) and dialects (§16) extend the vocabularies above. The following rules
govern all such extensions.

#### 14.4.1 Core Registry Open, Profile Registry Closed

1. Under **core conformance**, the universal registries of §14.1, §14.2, and §14.3 are
   **open**: values not in the universal registry are accepted with a warning.
2. Under **profile conformance**, the active profile's effective registry (universal +
   profile-registered) is **closed by default**: unregistered values are errors. A profile
   MAY declare itself open for one or more of its registries (cat, roles, behavior) if its
   domain warrants leniency, but this SHOULD be the exception.
3. Under **corpus conformance**, closure rules are inherited from the active profile.

#### 14.4.2 Append-Only

The universal registries in §14.1, §14.2, and §14.3 are **append-only across revisions**:

1. A future revision of this specification MAY add new universal entries.
2. A future revision MUST NOT remove a universal entry.
3. A future revision MUST NOT redefine the meaning of an existing universal entry.

The same append-only rule applies to profile registries once a profile reaches Stable
status: a Stable profile MAY add entries in subsequent revisions but MUST NOT remove or
redefine them.

#### 14.4.3 No Semantic Redefinition

A profile or dialect MAY **narrow** the meaning or applicability of a universal entry
(restrict cardinality, restrict source/target `cat:`, restrict allowed values of a keyed
descriptor). A profile or dialect MUST NOT **change** or **invert** the meaning.

Examples of permitted narrowing:

- `api_contract` narrows `response` cardinality from default `0..n` to `1..n` for
  endpoints with `verb:GET`.
- A profile MAY forbid a universal keyed descriptor where it is not semantically relevant.
  For example, `code_surface` MAY disallow `auth:` on ordinary code entities unless the
  profile explicitly models runtime authorization concerns.

Examples of forbidden redefinition:

- A profile redefines `writes` to mean "no-op" under its domain (forbidden; changes
  meaning).
- A profile inverts `supersedes` so that source is the predecessor (forbidden; inverts
  direction).

#### 14.4.4 Additions Must Declare Full Metadata

Every profile- or dialect-registered addition MUST declare, in the profile or dialect
addendum, the complete metadata for its kind:

**For a new `cat:` value:**
- name
- meaning (one sentence)
- whether it admits facets (and if so, the canonical facet names)
- relationship to universal `cat:` values, if any

**For a new relation role:**
- name
- meaning (one sentence)
- permitted source `cat:` values
- permitted target `cat:` values
- default cardinality
- whether `ext:` targets are permitted
- whether it is transitive (for future transitive closure features)

**For a new bare behavior atom:**
- name
- meaning (one sentence)
- which `cat:` values it applies to (if constrained)
- whether it composes with existing atoms or conflicts

**For a new keyed behavior descriptor:**
- key name
- permitted values (closed list or free-form)
- meaning of each value if closed
- composability with existing descriptors

Additions that omit any of this metadata are ill-formed and MUST NOT be registered.

### 14.5 Open Questions for Future Revisions

- Whether to introduce a **facet vocabulary registry** (universal facet names that apply
  across `cat:` values).
- Whether specific universal roles should be declared **transitive** or **symmetric** for
  consumer-side closure computation.
- Whether a **severity classification** for behavior atoms (informational, advisory,
  normative) should be introduced.
- Whether **numeric units** in keyed descriptors (like `ms`, `rps`) should be promoted
  from name-encoding to a separate structured form (e.g., `latency_p50:150{ms}`).
- Whether `pre:`, `post:`, `inv:` should support more structured value grammar (not just
  opaque atoms) in v0.2.

---

# Part IV — Governance (Layer 2)

Part III established the universal vocabularies that every JOOT document can rely on. Part
IV defines the mechanisms by which those vocabularies are **closed, extended, and
registered**: profiles (§15), dialects (§16), and the registry system (§17).

Governance is opt-in. A document that declares no profile and no dialect is validated under
core conformance against the universal model defined in Parts I–III. A document that
declares a profile activates strict machine-first validation, vocabulary closure, and
identity enforcement, per the mechanics defined in this part.

This part is normative. Profiles close what core leaves open; dialects extend what core and
profiles leave incomplete. Neither may contradict the core model.

---

## 15. Profiles

A **profile** is a named governance layer that, when activated on a document, closes the
universal vocabularies of §14, registers profile-specific vocabulary, and enforces strict
validation rules. Profiles are the primary mechanism by which JOOT is specialized for a
concrete domain (code surfaces, API contracts, documentation references, schema catalogs).

This section defines profile mechanics (§15.1–§15.2), specifies two normatively registered
profiles in full — `code_surface` (§15.3) and `api_contract` (§15.4) — and closes cross-
cutting concerns in §15.5.

### 15.1 Profile Mechanics

#### 15.1.1 What a Profile Defines

A profile MUST define, as part of its normative addendum:

1. **Name.** A lowercase underscored identifier conforming to §11.10.
2. **Version.** A major.minor version number. Versions are append-only once Stable (§15.1.7).
3. **Status.** One of `Draft`, `Candidate`, or `Stable`. See §15.1.7.
4. **Scope statement.** A one-paragraph description of the domain the profile addresses.
5. **`cat:` registry.** The set of `cat:` values accepted under the profile, drawn from
   §14.1 and extended by profile-registered additions. The profile MUST declare whether
   its `cat:` vocabulary is **closed** (only universal + profile-registered values) or
   **open** (other values accepted with warning). Machine-first profiles SHOULD declare
   closed.
6. **`kind:` registry per `cat:`.** For each accepted `cat:`, the set of permitted
   `kind:` values. This is the profile's finest-grained subject classification.
7. **Relation role registry.** Which universal roles from §14.2 are permitted, with any
   narrowed source/target constraints, cardinality overrides, and `ext:` policy.
   Additionally, any profile-registered roles with full metadata per §14.4.4.
8. **Behavior vocabulary.** Which universal bare atoms and keyed descriptors from §14.3
   are permitted, with any narrowing of allowed values. Additionally, any profile-
   registered atoms or descriptors.
9. **Meta key registry.** Which canonical meta keys (§6.3.4) are REQUIRED beyond the
   universal requirement of `cat:`. Additionally, any profile-registered meta keys.
10. **Facet vocabulary.** For each `cat:` (or `(cat:, kind:)` combination) that admits
    facets, the canonical facet names the profile accepts.
11. **Decomposition rules.** Rules that distinguish facet decomposition from child-subject
    decomposition within the profile's domain (§12.3.5 applied to the profile).
12. **Conformance checks.** A minimal enumeration of checks a validator MUST perform
    beyond core conformance.
13. **Cross-profile compatibility statement.** Whether and how documents declaring this
    profile interoperate with documents declaring other profiles in a corpus (§15.5.4).

A profile addendum that omits any of the above MUST NOT reach `Stable` status.

#### 15.1.2 What a Profile May Close

A profile MAY close:

- `cat:` vocabulary;
- relation role registry;
- behavior vocabulary (both bare atoms and keyed descriptors);
- constraint atom vocabulary (§8.4);
- facet vocabulary per `cat:`/`kind:`;
- `kind:` vocabulary per `cat:`.

Closure means: values not registered by the universal layer or by the profile produce
**errors** under profile conformance, not warnings. This is the primary difference
between core and profile conformance.

A profile MAY close some vocabularies and leave others open. For example, a profile might
close `cat:` and `kind:` but leave constraint atoms open. The addendum MUST state closure
explicitly per vocabulary.

#### 15.1.3 What a Profile Must Not Redefine

A profile MUST NOT:

1. Alter core syntax (§§4–11). Profile-specific content lives in semantics, not syntax.
2. Redefine the meaning of a universal `cat:`, relation role, behavior atom, or meta key
   (§14.4.3). Narrowing is permitted; redefinition is not.
3. Invert a relation role's direction.
4. Remove universal entries from the combined registry. A profile may **forbid** a
   universal entry in its domain (e.g., forbid `auth:` on code entities per §14.4.3), but
   this is a domain-level restriction, not a deregistration.
5. Grant itself authority to override core conformance rules (§1.2.1). Profile conformance
   **adds** to core conformance; it never subtracts.

#### 15.1.4 Profile Activation

A profile is activated by declaring `profile=<n>` on header line 1 (§4.2.3). When a
profile is activated:

- The document MUST satisfy core conformance (§1.2.1) plus the profile's addendum rules.
- The validator enters profile mode (§1.3) by default.
- Identity uniqueness is enforced at `(id, facet)` granularity (§12.2).
- Canonical ordering rules are enforced as errors, not warnings (§6.3.2, §6.4.2, §8.6).

A document without `profile=` is validated under core conformance only. A document with
`profile=<unknown_name>` follows the mode-dependent policy of §4.2.3.

#### 15.1.5 Profile Precedence over Dialects

When both a profile and a dialect are active, profile rules take precedence over dialect
rules in cases of conflict (§6.3.5). Specifically:

1. If both register the same vocabulary entry (same `cat:`, same role, same atom), the
   profile's declaration wins.
2. If the profile forbids a universal entry and the dialect permits it, the profile wins.
3. If the profile declares a vocabulary **closed** and the dialect attempts to add entries,
   the addition is rejected unless the profile explicitly permits that dialect (§16.2).

A dialect is active only when the profile explicitly permits it. A profile MUST declare,
in its addendum, the list of dialects it permits (possibly empty). See §16.2 for the
full interaction model.

#### 15.1.6 Multiple Profiles

A document MAY declare at most one `profile=` token (§4.2.5). Multiple simultaneous
profiles are not supported in v0.1. A future revision MAY introduce profile composition
(an explicit rule for combining profiles), but v0.1 treats the profile as singular.

When a corpus contains documents declaring different profiles, each document is validated
against its declared profile. Cross-profile interoperability is governed by §15.5.4.

#### 15.1.7 Profile Lifecycle

A profile progresses through three status levels:

- **Draft.** The profile's addendum is incomplete or under active revision. Profile-
  registered vocabulary MAY change. Consumers SHOULD treat Draft profiles as experimental.
- **Candidate.** The profile's addendum is complete per §15.1.1 and has at least one
  independent implementation. Profile-registered vocabulary is append-only from this point.
- **Stable.** The profile has been implemented by at least two independent consumers and
  has a published conformance test suite. Stable profiles MUST NOT remove or redefine
  registered vocabulary.

This specification designates two profiles as Candidate in v0.1: `code_surface` and
`api_contract`. Both reach Stable status in a future revision, after the conformance test
suite (§25) is published.

### 15.2 Profile Addendum Structure

A profile addendum is the normative text that specifies the profile. Its structure is:

1. **Header** (name, version, status, scope statement).
2. **`cat:` and `kind:` registry** (with tables).
3. **Relation role registry** (with narrowed constraints and cardinality overrides).
4. **Behavior vocabulary** (permitted universals with narrowing; profile-registered
   additions).
5. **Meta key registry** (required meta, profile-registered meta).
6. **Facet vocabulary** (canonical facets per `(cat:, kind:)`).
7. **Decomposition rules** (facet vs child subject within the profile's domain).
8. **Conformance checks** (minimal enumeration of checks the validator MUST perform).
9. **Cross-profile compatibility statement**.

This structure is applied to `code_surface` in §15.3 and to `api_contract` in §15.4.

### 15.3 Profile: `code_surface`

#### 15.3.1 Header

- **Name:** `code_surface`
- **Version:** 0.1
- **Status:** Candidate
- **Scope:** Structural representation of code surfaces: modules, namespaces, packages,
  nominal types (classes, interfaces, structs, enums, traits, protocols), and their
  members (fields, properties, constants, methods, functions, constructors). Intended for
  use as a canonical intermediate representation of source code APIs across programming
  languages.

#### 15.3.2 `cat:` Registry

**Vocabulary closure: closed.** Only the `cat:` values listed below are valid under
`code_surface`.

| `cat:`       | Meaning in `code_surface`                                       |
|--------------|-----------------------------------------------------------------|
| `module`     | A namespace, package, or module in the source language.         |
| `type`       | A nominal type (class, interface, struct, enum, trait, protocol). |
| `member`     | A field, property, constant, or slot declared inside a type or module. |
| `callable`   | A method, function, constructor, operator overload, or procedure. |
| `example`    | A usage example of another subject.                             |
| `test`       | A test case targeting another subject.                          |

`cat:` values not listed above are **not permitted** under `code_surface`. In particular,
`endpoint`, `contract`, `event`, `state`, `transition`, `step`, `source`, `schema`, and
`metric` belong to other profiles.

#### 15.3.3 `kind:` Registry per `cat:`

**`cat:module`** — `kind:` values:

```
package, namespace, module, crate
```

**`cat:type`** — `kind:` values:

```
class, interface, struct, enum, trait, protocol, record_type, union_type, alias_type
```

**`cat:member`** — `kind:` values:

```
field, property, constant, associated_type
```

**`cat:callable`** — `kind:` values:

```
method, function, constructor, destructor, operator, accessor, mutator
```

**`cat:example`** — `kind:` values:

```
usage, snippet, tutorial
```

**`cat:test`** — `kind:` values:

```
unit, integration, property, benchmark
```

Under `code_surface`, the `kind:` vocabulary is **closed**. Other values are rejected.

#### 15.3.4 Relation Role Registry

**Permitted universal roles** (from §14.2), with any narrowing:

| Role            | Narrowing under `code_surface`                                                 |
|-----------------|---------------------------------------------------------------------------------|
| `member_of`     | Source: `member`, `callable`, `type`. Target: `type`, `module`. Cardinality narrowed to `1` when source is `member` or `callable` (every member belongs to exactly one parent). `ext:` permitted. |
| `part_of`       | Source: any `code_surface` `cat:`. Target: `module`. Cardinality `0..1`. `ext:` permitted. |
| `owns`          | Not used in `code_surface` v0.1 (reserved for future RAII/ownership modeling). |
| `imports`       | Source: `module`. Target: `module`. `ext:` permitted (external modules/libraries). |
| `exports`       | Source: `module`. Target: `type`, `callable`, `member`, `module`. `ext:` forbidden. |
| `depends_on`    | Source: any. Target: any. `ext:` permitted.                                     |
| `references`    | Source: any. Target: any. `ext:` permitted.                                     |
| `extends`       | Source: `type`. Target: `type`. `ext:` permitted. Cardinality `0..1` for languages with single inheritance (`kind:class`); `0..n` for `kind:interface` and `kind:trait`. |
| `implements`    | Source: `type`. Target: `type` with `kind:interface` or `kind:protocol` or `kind:trait`. `ext:` permitted. |
| `conforms_to`   | Source: `type`, `callable`. Target: `type`, `contract`. `ext:` permitted.       |
| `annotated_by`  | Source: any. Target: `type` (typically `kind:annotation` or `kind:decorator`). `ext:` permitted. |
| `calls`         | Source: `callable`. Target: `callable`. `ext:` permitted.                       |
| `reads`         | Source: `callable`. Target: `member`. `ext:` permitted.                         |
| `writes`        | Source: `callable`. Target: `member`. `ext:` permitted.                         |
| `throws`        | Source: `callable`. Target: `type` (typically exception types). `ext:` permitted. |
| `for`           | Source: `example`, `test`. Target: any `code_surface` subject. `ext:` forbidden. |
| `supersedes`    | Source: any. Target: any (same `cat:`). `ext:` forbidden.                       |
| `replaced_by`   | Source: any. Target: any (same `cat:`). `ext:` forbidden.                       |
| `refines`       | Source: any. Target: any (same `cat:`). `ext:` forbidden.                       |
| `specializes`   | Source: `type`. Target: `type`. `ext:` permitted.                               |
| `compatible_with` | Source: any. Target: any. `ext:` permitted.                                   |
| `incompatible_with` | Source: any. Target: any. `ext:` permitted.                                 |

**Forbidden universal roles** (not applicable to `code_surface`):

- `request`, `response`, `error` — applicable only to endpoints; forbidden here.
- `from`, `to`, `next`, `on_fail` — state/transition roles; forbidden here.
- `emits`, `consumes` — event roles; forbidden here.

**Profile-registered roles:** none in v0.1.

#### 15.3.5 Behavior Vocabulary

**Permitted universal bare atoms** (§14.3.1):

```
async, awaitable, ordered, single_use, retriable, destructive, safe,
idempotent, cacheable
```

Excluded from `code_surface`: `initial`, `terminal` (state-only semantics).

**Permitted universal keyed descriptors** (§14.3.2), with narrowing:

| Key              | Narrowing under `code_surface`                                         |
|------------------|-------------------------------------------------------------------------|
| `vis:`           | Full set: `public` / `protected` / `private` / `internal`.              |
| `mut:`           | Full set: `mutable` / `readonly` / `immutable`.                         |
| `bind:`          | Full set: `instance` / `static` / `class`.                              |
| `effect:`        | Full set permitted.                                                     |
| `pre:`, `post:`, `inv:` | Opaque atoms; permitted.                                       |
| `auth:`          | **Forbidden** on ordinary code entities. The profile does not model runtime authorization. A future `code_security` dialect may reintroduce it. |
| `requires_role:` | **Forbidden**, same rationale.                                          |
| `complexity:`    | Permitted.                                                              |
| `latency_*`, `throughput_rps:`, `cost:` | Permitted but rarely applicable to static code surfaces; typically appear on contract-bearing subjects. |

**Profile-registered bare atoms:** none in v0.1.

**Profile-registered keyed descriptors:** none in v0.1.

#### 15.3.6 Meta Key Registry

**Required meta keys beyond `cat:`:**

- `id:` — REQUIRED on every entry.
- `kind:` — REQUIRED on every entry.

**Permitted canonical meta keys** (from §6.3.4): `facet:`, `scope:`, `context:`, `target:`,
`requires:`, `status:`, `since:`, `sunset:`, `until:`. The `path:` and `verb:` keys are
NOT applicable and MUST NOT appear.

**Profile-registered meta keys:** none in v0.1.

#### 15.3.7 Facet Vocabulary

`code_surface` makes sparing use of facets. Most code subjects are unfaceted. The profile
registers the following canonical facets:

**`cat:type`, any `kind:`** — canonical facets (used rarely, only when the type has
substantial independent projections):

```
structural, behavioral
```

**`cat:callable`** — canonical facets (used when method contracts and implementations are
authored as separate projections):

```
contract, implementation
```

Other `cat:` values in `code_surface` are unfaceted by default. A profile extension MAY
register additional facets per `(cat:, kind:)`.

#### 15.3.8 Decomposition Rules: Facet vs Child Subject

Specific to `code_surface`, the decision procedure of §12.3.5 is refined:

1. **Members of a type are always child subjects, never facets.** A class's fields and
   methods are distinct subjects with their own `id:`, linked to the parent type via
   `member_of`. They are not `facet:field` of the class.

2. **Inheritance is always a relation, never a facet.** A derived class is a separate
   subject linked to its base via `extends`. It is not `facet:derived` of the base.

3. **Overloads are distinct subjects with distinct IDs.** See §15.3.9 for the overload
   identity rule.

4. **Language-specific variants are separate subjects.** A method and its async wrapper,
   a sync and its async variant, or a Java method and its Kotlin extension are distinct
   subjects. Link with `refines`, `specializes`, or `calls` as appropriate.

5. **Facets are reserved for cases where a single logical subject genuinely has two
   orthogonal projections** (typically: a contract view vs an implementation view of a
   callable; a structural view vs a behavioral view of a type).

#### 15.3.9 Overload Identity and Callable Signatures

In languages that support overloading (Java, C++, C#, Swift, etc.), multiple callables
MAY share the same surface name but differ by signature. `code_surface` resolves this as
follows:

**Overloads are distinct subjects with distinct `id:` values.** Each overload carries its
parameter-type signature in its `id:` to ensure uniqueness. The canonical signature
encoding is:

```
id_value = <base_id>(<type_1>[,<type_2>...])
```

Where `<base_id>` is the normal dotted identifier and the parenthesized parameter list
encodes the ordered types. Parameter type encoding uses nominal type names as they appear
in the type's own `id:` or in its nominal form (§7.2.1); separators are commas with no
whitespace.

Examples:

```
id:java.com.acme.order.OrderService.find(text)
id:java.com.acme.order.OrderService.find(text,int)
id:cpp.acme.print.RasterJob.submit(text+Options)
```

Callables without parameters encode an empty parameter list: `id:java.com.acme.Order.toString()`.

Callables in languages without overloading MAY omit the parameter list when the base `id:`
is already unique. Profiles targeting specific language corpora MAY make the parameter
list mandatory.

**Return type is NOT part of callable identity.** Languages where overload resolution
considers return type are rare and are handled by profile extensions when needed.

#### 15.3.10 Internal vs External Targets

`code_surface` permits external targets (`ext:<ref>`) on any role where the universal
registry or the narrowing in §15.3.4 permits them. External targets are used to reference
subjects outside the active corpus: standard library types, third-party libraries,
platform APIs.

An `ext:` reference in `code_surface` typically takes one of these forms:

- `ext:java.lang.Object` — a fully qualified external type.
- `ext:std::vector` — a C++ standard library type (quoted if needed per §6.4.3).
- `ext:npm:react/Component` — a package-qualified reference.
- `ext:"java.util.Map<java.lang.String,java.lang.Object>"` — quoted when the reference
  contains characters outside the `ext_token` grammar.

External targets are not resolved by validators; their syntactic validity is checked, but
their existence is assumed.

#### 15.3.11 Minimal Conformance Checks

A `code_surface` validator MUST perform, at minimum, the following checks beyond core
conformance:

1. Every entry has `cat:`, `id:`, and `kind:` present.
2. `cat:` value is in §15.3.2.
3. `kind:` value is in §15.3.3 for the entry's `cat:`.
4. `(id, facet)` uniqueness is enforced at document and corpus scope.
5. Canonical meta key order (§6.3.2) is respected — errors, not warnings.
6. Canonical `[Rel]` item order (§6.4.2) is respected — errors.
7. Canonical constraint atom order (§8.6) is respected — errors.
8. Relation roles used on an entry are permitted by §15.3.4; source/target `cat:`
   constraints match; cardinality is respected.
9. Behavior atoms and keyed descriptors used are in §15.3.5.
10. Constraint atoms used are in the universal registry (§8.2, §8.3).
11. Callable overloads have distinct `id:` values per §15.3.9.
12. `member_of` on a member or callable has cardinality `1` (every member belongs to a
    parent).
13. No `path:`, `verb:`, or endpoint-specific content appears.

#### 15.3.12 Cross-Profile Compatibility

A `code_surface` document in a corpus with `api_contract` documents interoperates as
follows:

- An `api_contract` endpoint MAY target a `code_surface` `callable` via `conforms_to` or
  `refines`, indicating that a specific handler or implementation realizes the endpoint.
- A `code_surface` `type` MAY be targeted as a `request`/`response`/`error` body by an
  `api_contract` endpoint, when the type is the language-level representation of the
  contract.
- Bare `<id>` targets resolve across profile boundaries as long as the `id:` values are
  corpus-unique.

### 15.4 Profile: `api_contract`

#### 15.4.1 Header

- **Name:** `api_contract`
- **Version:** 0.1
- **Status:** Candidate
- **Scope:** HTTP API surfaces: endpoints, request/response/error payloads, headers,
  authentication requirements, and contract definitions. Intended as a canonical
  intermediate representation of API specifications across OpenAPI, gRPC, and similar
  ecosystems, at the level of structural and behavioral contracts.

#### 15.4.2 `cat:` Registry

**Vocabulary closure: closed.** Only the `cat:` values listed below are valid under
`api_contract`.

| `cat:`       | Meaning in `api_contract`                                         |
|--------------|--------------------------------------------------------------------|
| `endpoint`   | An HTTP-addressable operation (path + verb).                       |
| `contract`   | A reusable payload shape: request body, response body, or error body. |
| `type`       | A nominal type referenced by contracts (typically a data schema).  |
| `example`    | A usage example of an endpoint or contract.                        |
| `test`       | A conformance test against an endpoint.                            |
| `metric`     | An observability metric associated with an endpoint.               |

Not permitted: `module`, `member`, `callable`, `source`, `schema`, `event`, `state`,
`transition`, `step`.

#### 15.4.3 `kind:` Registry per `cat:`

**`cat:endpoint`** — `kind:` values:

```
rest, rpc, graphql, webhook, grpc_unary, grpc_streaming
```

**`cat:contract`** — `kind:` values:

```
request_body, response_body, error_body, request_header, response_header, path_parameter, query_parameter
```

**`cat:type`** — `kind:` values:

```
record_type, enum_type, union_type, alias_type
```

**`cat:example`** — `kind:` values:

```
request_example, response_example, curl_invocation
```

**`cat:test`** — `kind:` values:

```
contract_conformance, integration, load
```

**`cat:metric`** — `kind:` values:

```
latency, throughput, error_rate, availability
```

Vocabulary is **closed**.

#### 15.4.4 Relation Role Registry

**Permitted universal roles** with `api_contract`-specific narrowing:

| Role              | Narrowing under `api_contract`                                                  |
|-------------------|----------------------------------------------------------------------------------|
| `request`         | Source: `endpoint`. Target: `contract` (linked-contract form) OR `endpoint#request` facet (endpoint-facet form). `ext:` forbidden. Cardinality `0..1` per endpoint (or per request facet if faceted). |
| `response`        | Source: `endpoint`. Target: `contract` OR `endpoint#response` facet. `ext:` forbidden. Cardinality `1..n` for `verb:GET`, `verb:POST`, `verb:PUT`, `verb:PATCH`; `0..n` for `verb:HEAD`, `verb:OPTIONS`. |
| `error`           | Source: `endpoint`. Target: `contract` OR `endpoint#error` facet. `ext:` forbidden. Cardinality `0..n`. |
| `conforms_to`     | Source: `contract`, `endpoint`. Target: `contract`, `type`. `ext:` permitted.    |
| `references`      | Source: any. Target: `contract`, `type`. `ext:` permitted.                       |
| `depends_on`      | Source: `endpoint`. Target: `endpoint`, `contract`. `ext:` permitted (external APIs). |
| `annotated_by`    | Source: `endpoint`, `contract`. Target: `type`. `ext:` permitted.                |
| `emits`           | Source: `endpoint`. Target: `event` is NOT in `api_contract` `cat:`; so `emits` targets are limited to external refs. `ext:` permitted. |
| `for`             | Source: `example`, `test`, `metric`. Target: `endpoint`, `contract`. `ext:` forbidden. |
| `compatible_with` | Source: `endpoint`. Target: `endpoint`. `ext:` permitted (API versioning).       |
| `incompatible_with` | Source: `endpoint`. Target: `endpoint`. `ext:` permitted.                      |
| `supersedes`      | Source: `endpoint`, `contract`. Target: `endpoint`, `contract`. `ext:` permitted (cross-corpus version lineage). |
| `replaced_by`     | Source: `endpoint`, `contract`. Target: `endpoint`, `contract`. `ext:` permitted (cross-corpus version lineage). |
| `refines`         | Source: `contract`. Target: `contract`. `ext:` forbidden.                        |
| `specializes`     | Source: `contract`. Target: `contract`. `ext:` forbidden.                        |

**Forbidden universal roles** (not applicable):

- `member_of`, `part_of`, `owns`, `imports`, `exports`, `extends`, `implements`,
  `calls`, `reads`, `writes`, `throws` — code-surface roles; forbidden here.
- `from`, `to`, `next`, `on_fail` — state/transition roles; forbidden.
- `consumes` — event role; forbidden.

**Profile-registered roles:** none in v0.1.

#### 15.4.5 Linked-Contract vs Endpoint-Facet Model

`api_contract` supports **two models** for associating an endpoint with its request,
response, and error shapes. A single endpoint MUST use exactly one of them. Mixing them
on the same logical endpoint is a profile conformance error.

**Model A — linked-contract form.**

The endpoint is an **unfaceted** subject. Its request, response, and error shapes are
separate `cat:contract` subjects, each with its own `id:`, referenced by the endpoint
via `request:`, `response:`, `error:` relation roles.

Example (illustrative entries):

```
!http.orders.get|cat:endpoint|id:http.orders.get|kind:rest|path:/orders/{id}|verb:GET|[Rel]request:http.orders.get.request,response:http.orders.get.response_200,response:http.orders.get.response_304,error:http.orders.get.error_404,error:http.orders.get.error_500

!http.orders.get.request|cat:contract|id:http.orders.get.request|kind:request_body|...
!http.orders.get.response_200|cat:contract|id:http.orders.get.response_200|kind:response_body|...
!http.orders.get.response_304|cat:contract|id:http.orders.get.response_304|kind:response_body|...
!http.orders.get.error_404|cat:contract|id:http.orders.get.error_404|kind:error_body|...
```

Linked-contract form is **preferred** when:

- contracts are shared across multiple endpoints;
- contracts have their own lifecycle and versioning;
- the corpus benefits from clear contract-level subjects for indexing and discovery.

**Model B — endpoint-facet form.**

The endpoint is a **faceted** subject. Its request, response, and error shapes are
facets of the same endpoint subject, each a separate entry with the same `id:` and
distinct `facet:` values.

Example (illustrative entries):

```
!http.orders.get|cat:endpoint|id:http.orders.get|kind:rest|facet:main|path:/orders/{id}|verb:GET|...
!http.orders.get|cat:endpoint|id:http.orders.get|kind:rest|facet:request|path:/orders/{id}|verb:GET|[Fields]...
!http.orders.get|cat:endpoint|id:http.orders.get|kind:rest|facet:response|path:/orders/{id}|verb:GET|[Returns]...
!http.orders.get|cat:endpoint|id:http.orders.get|kind:rest|facet:error|path:/orders/{id}|verb:GET|[Returns]...
```

The canonical facet names for `cat:endpoint` under `api_contract` are:

```
main, request, response, error
```

**Semantics of the canonical facets** (endpoint-facet form only):

- `request`, `response`, and `error` carry the respective contract shapes and apply
  **only** when using endpoint-facet form (Model B). They never appear in linked-contract
  form (Model A).
- `main` is the **optional top-level projection** of the endpoint in endpoint-facet
  form. When present, it carries the endpoint's top-level descriptors (auth, behavior,
  endpoint-wide annotations). `main` is the correct facet name to use when a producer
  needs a non-request/response/error projection of the endpoint.
- If `main` is omitted, the endpoint's top-level descriptors MAY be distributed across
  the `request`, `response`, and `error` facets. In that case, `path:` and `verb:` MUST
  still appear on every facet per §15.4.6, and top-level behavior descriptors (`auth:`,
  `requires_role:`, etc.) MUST be repeated consistently across facets or omitted from
  all of them — partial declaration on only some facets is a profile conformance error.

**Relation to the two models:**

- **Model A (linked-contract form).** The endpoint is **unfaceted**. The entry carries
  no `facet:` meta key. All endpoint top-level descriptors live on that single entry.
  The canonical facet names `main`, `request`, `response`, `error` do NOT appear.
- **Model B (endpoint-facet form).** The endpoint is **faceted**. Every entry for that
  endpoint `id:` carries a `facet:` meta key drawn from `{main, request, response, error}`.
  A producer MAY use `main` alone plus any combination of `request`/`response`/`error`,
  or omit `main` entirely.

Per §12.3.2, a given endpoint `id:` MUST be either fully unfaceted (Model A) or fully
faceted (Model B). Coexistence of an unfaceted entry and faceted entries with the same
`id:` is a profile conformance error.

Endpoint-facet form is **preferred** when:

- contracts are tightly coupled to the endpoint and not shared;
- the corpus prioritizes minimal entry counts per endpoint;
- facet-level editing (updating only the response shape without touching others) is a
  primary authoring workflow.

**Choosing between models.** A producer MUST choose one model per endpoint and declare all
relevant projections in that model. A validator MUST reject documents where a single
`id:` appears in both forms (e.g., both an unfaceted `!http.orders.get` entry and faceted
`facet:request`/`facet:response` entries with the same `id:`). This is an application of
§12.3.2 (unfaceted + faceted coexistence is prohibited).

Within a corpus, different endpoints MAY use different models. Consistency within a family
of endpoints (e.g., all endpoints under `/orders/*`) is RECOMMENDED but not required.

#### 15.4.6 `path:` and `verb:` Requirements

For `cat:endpoint` subjects in `api_contract`:

- **`path:`** is REQUIRED on every endpoint entry.
- **`verb:`** is REQUIRED on every endpoint entry.
- **`verb:` value** MUST be one of: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`,
  `OPTIONS`, `TRACE`, `CONNECT`. Verbs are written in uppercase per HTTP convention.
- **`path:` value** is a literal path template (e.g., `/orders/{id}`, `/users/{user_id}/items`).
  Syntactic validation of path templates is the consumer's responsibility; the profile
  does not impose a grammar beyond "non-empty string, single line".

**Equality across facets:** In endpoint-facet form (§15.4.5 Model B), all facets of the
same endpoint `id:` MUST declare identical `path:` and identical `verb:` values. This
enforces §12.3.4 rule 4 (profile-restrictable independent meta): `api_contract` requires
equality of these keys across facets because all facets address the same HTTP surface.

Violation: two facets of the same `id:` declaring different `path:` or `verb:` values is
a profile conformance error.

In linked-contract form (Model A), `path:` and `verb:` appear only on the single
endpoint entry; they do not appear on contract entries.

**Contracts do not carry `path:` or `verb:`.** `cat:contract` subjects MUST NOT declare
`path:` or `verb:` meta keys — contracts are payload shapes, not endpoints. Presence is
an error.

#### 15.4.7 Response and Error Multiplicity

`api_contract` endpoints commonly return multiple possible response shapes (e.g., `200`,
`304`, `404`). The profile models this via multiplicity on the `response:` and `error:`
relation roles:

- **`response:` cardinality** is `1..n` for endpoints with `verb:GET`, `verb:POST`,
  `verb:PUT`, `verb:PATCH`. Every such endpoint MUST have at least one response.
- **`response:` cardinality** is `0..n` for endpoints with `verb:HEAD`, `verb:OPTIONS`,
  `verb:DELETE`, `verb:TRACE`, `verb:CONNECT`. Responses are permitted but not required.
- **`error:` cardinality** is `0..n` for all verbs. Error responses are optional in the
  contract.

**Disambiguating responses.** When an endpoint has multiple responses, consumers need a
way to tell them apart. `api_contract` uses **status-code-bearing contract IDs** as the
canonical disambiguator:

- In linked-contract form: the contract's `id:` includes the status code
  (`http.orders.get.response_200`, `http.orders.get.response_304`).
- In endpoint-facet form: each response facet declares a `status_code:` meta key
  registered by the profile (see §15.4.9).

**Status codes are NOT carried in `status:`.** The core specification (§6.3.4) states that
`status:` is the lifecycle status, not the HTTP status code. `api_contract` does not
override this.

#### 15.4.8 `ext:` Policy

The `ext:` target form is permitted in `api_contract` on the following roles:

- `conforms_to`, `references`, `depends_on`, `annotated_by`, `emits`, `compatible_with`,
  `incompatible_with`, `supersedes`, `replaced_by`.

The `ext:` form is **forbidden** on:

- `request`, `response`, `error` — these MUST target in-corpus contracts or endpoint
  facets. An external request/response shape is modeled by declaring the external shape
  as a `cat:contract` entry in the corpus with its own `id:` and referencing from the
  endpoint; or by using `conforms_to:ext:<external_contract>` to cite an external
  specification.
- `for`, `refines`, `specializes` — in-corpus only, because these roles express
  refinement relationships among subjects within the same specification.

This is an application of §6.4.3 (profile declares role-level `ext:` policy) and is
consistent with the default policy set there. Allowing `ext:` on `supersedes` and
`replaced_by` lets API producers declare cross-corpus version lineage (e.g., citing a
predecessor API hosted in another specification).

#### 15.4.9 Meta Key Registry

**Required meta keys beyond `cat:`:**

- `id:` — REQUIRED on every entry.
- `kind:` — REQUIRED on every entry.
- `path:` — REQUIRED on every `cat:endpoint` entry.
- `verb:` — REQUIRED on every `cat:endpoint` entry.

**Profile-registered meta keys:**

- `status_code:` — declares the HTTP status code associated with a response or error
  payload. Valid on:
  - `cat:contract` entries with `kind:response_body` or `kind:error_body` (linked-
    contract form, §15.4.5 Model A), and
  - `cat:endpoint` entries with `facet:response` or `facet:error` (endpoint-facet
    form, §15.4.5 Model B).
  Value is a three-digit integer. Position in canonical meta order: after `verb:` and
  before `status:`. Example: `status_code:200`.

#### 15.4.10 Behavior Vocabulary

**Permitted universal bare atoms:**

```
async, idempotent, safe, cacheable, retriable, destructive, single_use
```

**Permitted universal keyed descriptors**, with narrowing:

| Key              | Narrowing under `api_contract`                                          |
|------------------|--------------------------------------------------------------------------|
| `auth:`          | Permitted values: `none` / `basic` / `bearer` / `mtls`. Default if absent: `none`. |
| `requires_role:` | Permitted. Opaque atom.                                                  |
| `effect:`        | Permitted. Restricted to `pure`, `reads_state`, `writes_state`, `transactional`. |
| `latency_p50_ms:`, `latency_p95_ms:`, `throughput_rps:`, `cost:` | Permitted. |
| `vis:`, `mut:`, `bind:`, `complexity:` | Forbidden — not applicable to API contracts. |
| `pre:`, `post:`, `inv:` | Permitted as opaque atoms.                                        |

**Profile-registered bare atoms:** none in v0.1.

**Profile-registered keyed descriptors:** none in v0.1.

#### 15.4.11 Facet Vocabulary

Canonical facets per `(cat:, kind:)`:

**`cat:endpoint`** — all `kind:` values admit facets:

```
main, request, response, error
```

**`cat:contract`** — unfaceted by default. Contracts are typically single-projection
subjects; splitting a contract into facets is unusual and profile extensions may
register additional facet names if a use case emerges.

**`cat:type`, `cat:example`, `cat:test`, `cat:metric`** — unfaceted.

#### 15.4.12 Decomposition Rules: Facet vs Child Subject

Specific to `api_contract`:

1. **Request, response, and error shapes are facets** (Model B) **or child contracts**
   (Model A), never intermixed on the same endpoint.

2. **Headers, path parameters, and query parameters are NOT facets.** They belong in
   `[Fields]` of the appropriate contract facet or linked contract. A future profile
   extension MAY promote them to separate contract subjects with `kind:request_header`,
   etc., but v0.1 treats them as fields.

3. **Versioned endpoints (`v1`, `v2`) are distinct endpoint subjects** with different
   `id:` values, linked via `supersedes` / `replaced_by`. Not facets.

4. **Environment-specific endpoints** (staging, production) are expressed via `scope:` or
   `context:`, not via separate subjects or facets.

5. **Deprecated endpoints** retain their `id:` and carry `status:deprecated` plus a
   `replaced_by:` relation when applicable.

#### 15.4.13 Minimal Conformance Checks

An `api_contract` validator MUST perform, at minimum, the following checks beyond core
conformance:

1. Every entry has `cat:`, `id:`, `kind:` present.
2. Every `cat:endpoint` entry has `path:` and `verb:` present.
3. `cat:` value is in §15.4.2.
4. `kind:` value is in §15.4.3 for the entry's `cat:`.
5. `verb:` value is in the registered HTTP verbs set (§15.4.6).
6. `(id, facet)` uniqueness is enforced at document and corpus scope.
7. Canonical meta key order (§6.3.2), relation item order (§6.4.2), and constraint
   order (§8.6) are respected — errors.
8. Relation roles used are permitted by §15.4.4 with correct source/target `cat:` and
   cardinality.
9. Endpoint-facet form: all facets of the same endpoint `id:` declare identical `path:`
   and identical `verb:`.
10. An endpoint `id:` does NOT appear in both unfaceted and faceted form (§15.4.5).
11. `status_code:` on a contract entry is a three-digit integer.
12. `status:` on any entry does NOT carry an HTTP status code (e.g., `status:200` is
    forbidden; use `status:deprecated` / `status:stable` only).
13. `cat:contract` entries do NOT carry `path:` or `verb:`.
14. `ext:` targets on `request`, `response`, `error`, `for`, `refines`, `specializes`
    are rejected. `ext:` targets on `depends_on`, `compatible_with`, `incompatible_with`,
    `supersedes`, `replaced_by` are permitted, consistent with §6.4.3.
15. Response cardinality matches the verb's requirement (§15.4.7).
16. Behavior atoms and keyed descriptors used are in §15.4.10 with narrowed values.

#### 15.4.14 Cross-Profile Compatibility

An `api_contract` document in a corpus with `code_surface` documents interoperates per
§15.3.12. Specifically:

- `api_contract` endpoints MAY reference `code_surface` types via `conforms_to` or
  `references` when the type represents the contract's data shape in a source language.
- `api_contract` contracts MAY be realized by `code_surface` callables via `conforms_to`
  (from the callable's side) — indicating that the callable implements the endpoint.
- `api_contract` MUST NOT introduce code-surface roles (`member_of`, `extends`, etc.)
  into its own entries, even when referencing code-surface subjects. The relation role
  used on the `api_contract` side is drawn from §15.4.4.

### 15.5 Cross-Cutting Concerns

This section closes four concerns that apply to all profiles, not just `code_surface` and
`api_contract`.

#### 15.5.1 Vocabulary Scope

Profile vocabulary is scoped to the document declaring the profile. A `code_surface`-
registered relation role (if any existed in v0.1) would be valid only on entries in
documents declaring `profile=code_surface`. The same role name in an `api_contract`
document is either a universal role or an `api_contract`-registered role; the
`code_surface` definition does not apply.

This scoping rule has two consequences:

1. **Corpus-wide consistency is not automatic.** If two documents in a corpus declare
   different profiles, they have different effective vocabularies. Targets resolve
   across document boundaries (§6.4.4), but vocabulary does not.

2. **Profile role name collision policy.** A profile-registered role name MUST NOT
   redefine the meaning of a universal role name (restated from §14.4.3 for emphasis).
   Beyond that, stable profiles MUST NOT reuse an existing stable profile's role name
   with a different meaning: a role name registered by a `Stable` profile is reserved
   across the stable profile ecosystem for its declared semantics. `Draft` and
   `Candidate` profiles SHOULD NOT reuse role names from other profiles with different
   meanings, but this is not enforced until the profile reaches `Stable` status.

   The rationale is that consumers reasoning across a multi-profile corpus must be able
   to assume that a given role name carries a single, stable meaning. Silent semantic
   divergence across profiles would break cross-profile interoperability and cannot be
   detected without an external alignment registry.

A future revision MAY introduce a cross-profile vocabulary alignment registry (§17) to
formalize this discipline.

#### 15.5.2 Profile–Dialect Precedence (Restated)

The profile–dialect precedence rule (§15.1.5) is restated here for emphasis:

1. When a document declares both `profile=` and `dialect=`, the dialect is active only if
   the profile explicitly permits it in its addendum's dialect whitelist.
2. In conflicts, profile rules win over dialect rules.
3. A dialect cannot open a vocabulary the profile has closed.
4. A dialect cannot override a profile-registered entry.
5. A dialect MAY add entries in vocabularies the profile has left open, subject to the
   profile's approval.

The two profiles defined in this specification (`code_surface`, `api_contract`) have
**empty dialect whitelists** in v0.1. No dialects are permitted to extend them. Future
revisions MAY open specific dialect extension points (e.g., `code_surface` permitting a
`rust_extensions` dialect, `api_contract` permitting an `openapi_compatibility` dialect).

#### 15.5.3 Cross-Profile Interoperability

Within a corpus, documents MAY declare different profiles. Interoperability is governed
by these rules:

1. **Identity is profile-neutral.** An `id:` resolves the same way regardless of profile.
   A `code_surface` entry and an `api_contract` entry MAY reference each other via `id:`.

2. **Relation roles on an edge are drawn from the source document's profile.** An edge
   declared in an `api_contract` document uses `api_contract`-permitted roles, even if
   the target is in a `code_surface` document.

3. **Type compatibility across profiles is the consumer's responsibility.** If an
   `api_contract` `contract` references a `code_surface` `type`, whether the type's
   structure matches the contract's expected shape is not checked by either profile's
   validator. Cross-profile consistency checks are deferred to corpus-level tooling.

4. **Vocabulary translation across profiles is not automatic.** If a code-surface subject
   needs to be "viewed" as an api-contract subject (or vice versa), a consumer MAY
   produce a translated representation, but the original documents are not altered.

5. **Conformance is per-document.** Corpus conformance is the logical conjunction of each
   document's profile conformance plus corpus-level identity uniqueness. A corpus does
   not have a single profile; it has profiled documents.

#### 15.5.4 Forward Compatibility

**Profile versioning in v0.1 is deliberately minimal.** The header syntax
`profile=<n>` carries the profile name without a version qualifier. In v0.1:

1. Profile names appearing in the header are **unversioned**. A declaration
   `profile=code_surface` names the profile but does not identify a specific version
   of it.
2. **Version negotiation is out of scope for v0.1.** A consumer validates a document
   against the profile definition shipped with the specification revision the consumer
   implements. Two consumers implementing different revisions may reach different
   conformance verdicts on the same document; this is an expected consequence of
   unversioned declarations.
3. **Append-only evolution of profile vocabularies** (§14.4.2, §15.1.7) ensures that a
   document conforming to an older version of a profile will continue to parse and
   validate syntactically under a newer version; semantic additions do not invalidate
   older documents.
4. **Unknown profile names** fall back to core conformance per §1.3 and §4.2.3. A
   consumer that does not recognize the declared profile validates the document against
   core rules only.

A future revision MAY introduce a `profile=<n>@<version>` syntax to enable version
negotiation. Until then, profile versioning is resolved out-of-band (by the consumer's
implementation choice), not by the document.

### 15.6 Open Questions for Future Revisions

- Whether **multiple simultaneous profiles** on a single document should be supported
  (profile composition).
- Whether a **profile version qualifier** should be introduced in the header syntax
  (`profile=code_surface@0.3`).
- Whether **cross-profile vocabulary alignment** (a registry mapping identical concepts
  across profiles) should be formalized in §17.
- Whether additional profiles — `documentation_reference`, `platform_metamodel`,
  `schema_catalog`, `workflow_definition` — should be promoted to normatively registered
  Standard profiles in a future revision.
- Whether a **profile conformance test suite** format (machine-readable spec test cases)
  should be defined normatively in v0.2.

---

## 16. Dialects

A **dialect** is a named, opt-in vocabulary extension layer that adds types, meta keys,
categories, relation roles, behavior atoms, behavior keys, and constraint atoms to a JOOT
document. Dialects exist so that domain-specific or language-specific vocabulary does not
pollute the universal registries of §14 and so that profiles can remain tightly scoped
while still admitting targeted extensions when a domain needs them.

This section defines the dialect framework: what a dialect can extend (§16.1), what a
dialect must not touch (§16.2), activation and interaction with profiles (§16.3–§16.4),
addendum structure (§16.5), naming and namespacing rules (§16.6), conflict resolution
(§16.7), and handling of unknown dialects (§16.8).

**v0.1 registers no concrete dialects.** This section is a framework specification. Future
revisions or external parties MAY register dialects; the rules here govern how such
registrations are declared and how consumers treat them.

### 16.1 Rule of Dialects

A dialect extends vocabulary; it does not change grammar or core semantics.

This is the single most important rule in §16. Every other rule in this section is a
consequence of it. Specifically:

1. A dialect MUST NOT alter the syntax of JOOT (§§4–11).
2. A dialect MUST NOT introduce new named blocks, new sigils, new meta separators, new
   quoting rules, or any other syntactic mechanism.
3. A dialect MUST NOT redefine the meaning of a universal entry (§14.4.3).
4. A dialect MUST NOT change the identity model (§12), the graph model (§13), or any of
   the structural rules of Parts I–III.
5. A dialect MUST NOT override or relax profile rules when a profile is active (§15.1.5).

What a dialect can do is **strictly additive**: it declares new vocabulary entries that
become recognized by a validator when the dialect is active.

### 16.2 What a Dialect Can Extend

A dialect MAY register additions in the following vocabularies:

| Vocabulary                  | Extension form                                              | Defined in |
|-----------------------------|-------------------------------------------------------------|------------|
| `cat:` registry             | Additional category values                                  | §14.1      |
| `kind:` per `cat:`          | Additional `kind:` values for existing or new categories    | §6.3.4     |
| Relation role registry      | Additional relation roles                                   | §14.2      |
| Behavior bare atoms         | Additional single-token behavior properties                 | §14.3.1    |
| Behavior keyed descriptors  | Additional `key:value` behavior descriptors                 | §14.3.2    |
| Constraint atoms            | Additional `bare_atom` or `keyed_atom` constraints          | §8.2       |
| Meta keys                   | Additional canonical meta keys                              | §6.3.4     |
| Types                       | Additional named types registered for `[Fields]`/`[Returns]` | §7.6      |
| Facet vocabulary            | Additional canonical facet names per `(cat:, kind:)`        | §12.3.6    |
| Unit atoms                  | Additional measurement unit atoms                           | §8.3       |

Each addition MUST declare the full metadata required by §14.4.4 (name, meaning,
constraints, cardinality, closure behavior where applicable). A dialect addendum that
registers additions without complete metadata is ill-formed and MUST NOT be activated.

### 16.3 What a Dialect Cannot Do

A dialect MUST NOT:

1. **Alter core grammar or syntactic rules.** This includes sigils, block markers, pipe
   separators, comma separators, quoting rules, whitespace rules, and escape sequences.
2. **Remove or deprecate universal entries.** The universal registries are append-only
   across revisions (§14.4.2); a dialect cannot subtract from them.
3. **Redefine universal entries.** A dialect MAY narrow applicability within its own
   activation scope (e.g., "when this dialect is active, `auth:firebase` is permitted as
   a value of `auth:`"), but it MUST NOT change the meaning of a universal key or atom.
4. **Reopen vocabulary that the active profile has closed.** If a profile declares its
   `cat:` registry closed, a dialect cannot add `cat:` values in that document even if
   the dialect is on the profile's whitelist — unless the profile explicitly opens its
   `cat:` registry to that specific dialect.
5. **Override profile-registered entries.** When a profile registers a role, atom, or
   descriptor, the dialect cannot redefine it.
6. **Activate without profile permission.** See §16.4 for the activation model.
7. **Alter the identity, graph, or conformance models of Parts I–III.**
8. **Introduce runtime execution semantics.** Dialects describe static vocabulary, not
   behavior at execution time.

### 16.4 Activation

A dialect is activated by declaring `dialect=<n>` on header line 1 (§4.2.4). At most one
`dialect=` token MAY appear per header in v0.1.

**Activation rule.** When no profile is active, dialect activation is evaluated under
core conformance. When a profile is active, dialect activation additionally requires
explicit profile permission.

The specific cases:

1. A document declaring `dialect=<n>` without a `profile=<m>` declaration activates the
   dialect under core conformance. Core conformance permits dialect additions with the
   leniency of core mode (unknown entries produce warnings, not errors). No profile
   whitelist applies, because no profile is active.
2. A document declaring both `profile=<m>` and `dialect=<n>` activates the dialect only
   if the profile's addendum explicitly lists `<n>` in its dialect whitelist
   (§15.1.5).
3. If a profile is active and its whitelist is empty or does not include `<n>`, the
   dialect declaration is a profile conformance error. The validator MUST reject the
   document under profile or corpus mode.
4. In structural or core mode, a non-whitelisted dialect produces a warning, not an
   error, even when a profile is declared.

The two profiles registered in this specification (`code_surface`, `api_contract`) have
**empty dialect whitelists in v0.1**. No dialect can extend them in v0.1. Future
revisions MAY open specific dialect extension points for each profile.

### 16.5 Dialect Addendum Structure

A dialect is specified by a normative addendum. Every dialect addendum MUST declare:

1. **Header.**
   - Name (lowercase underscored identifier per §11.10)
   - Version (major.minor)
   - Status (`Draft`, `Candidate`, or `Stable`, per the lifecycle of §15.1.7 applied to
     dialects)
   - Scope statement (one paragraph)
   - Compatibility statement (which profiles the dialect is designed to extend)

2. **Registered `cat:` additions**, if any, with full metadata per §14.4.4.

3. **Registered `kind:` additions per `cat:`**, if any.

4. **Registered relation role additions**, if any, each with:
   - name
   - meaning
   - permitted source `cat:` values
   - permitted target `cat:` values
   - default cardinality
   - `ext:` policy

5. **Registered behavior additions**, if any:
   - bare atoms with meaning and applicability
   - keyed descriptors with allowed values

6. **Registered constraint atom additions**, if any, with meaning and applicability.

7. **Registered meta key additions**, if any, with canonical position in the meta
   ordering (§6.3.2) and whether they are repeatable.

8. **Registered type additions**, if any (§7.6), with arity where applicable.

9. **Registered facet vocabulary additions**, if any, per `(cat:, kind:)`.

10. **Registered unit atom additions**, if any.

11. **Known conflicts with other dialects**, declared explicitly when the dialect author
    is aware of collisions with other registered dialects. See §16.7.

A dialect addendum that omits required metadata on any registered addition is ill-formed
and MUST NOT be activated.

### 16.6 Naming and Namespacing

Dialect-registered additions are subject to strict naming discipline to prevent
collisions with universal entries and with other dialects.

#### 16.6.1 Naming Rules

1. Every dialect-registered addition MUST have a name that conforms to the naming grammar
   of the vocabulary it extends (§11 for identifiers; §11.3 for IDs; etc.).
2. A dialect-registered name MUST NOT collide with any entry already registered in the
   universal vocabularies of §14. Colliding with a universal name is an ill-formed
   addendum.
3. A dialect-registered name MUST NOT collide with any entry registered by another
   `Stable` dialect that may be simultaneously active under the same profile, unless the
   addendum explicitly documents the conflict and the resolution (§16.7).

#### 16.6.2 Prefix Convention (Recommended, Not Required)

To reduce the risk of dialect-vs-dialect collisions, dialect authors SHOULD prefix
dialect-specific vocabulary entries with a dialect-derived token. For example, a dialect
named `rust_extensions` might register types as `rust_struct_ref`, `rust_trait_ref`, and
so on.

This is a recommendation, not a requirement. A dialect addendum that registers unprefixed
names is conformant, provided the names do not collide with universal or other Stable
dialect entries. The registry-obligatory discipline of §16.5 is the primary collision
defense; prefixes are an additional readability and debuggability measure.

#### 16.6.3 Registry Obligation

Every dialect-registered addition MUST appear in the dialect's addendum with complete
metadata. Additions that appear in documents without corresponding addendum
registration are not dialect extensions; they are unknown vocabulary and are treated per
§16.8.

There is no mechanism in v0.1 for declaring vocabulary additions outside the addendum
system. Ad-hoc extensions are not a supported extension mechanism.

### 16.7 Conflict Resolution

Conflicts can arise between a dialect and other layers of the JOOT vocabulary stack. The
resolution rules are:

#### 16.7.1 Dialect vs Universal

A dialect-registered name that collides with a universal entry (§14) is an **ill-formed
addendum**. Such a dialect MUST NOT be activated; the collision is a specification error
of the dialect itself, not a runtime validation error.

If a dialect addendum has passed review and is registered, no collision with universal
entries exists by construction.

#### 16.7.2 Dialect vs Profile

A dialect-registered name that collides with a profile-registered entry is resolved in
favor of the profile (§15.1.5). The profile's declaration wins; the dialect's declaration
is suppressed for that name in documents declaring that profile.

The validator SHOULD emit a warning when it detects a dialect–profile collision, even if
the resolution is well-defined, because collisions often indicate design mismatches that
the dialect author may want to resolve through renaming.

**Editorial obligation on the profile side.** A profile addendum that whitelists a
dialect (§15.1.5) and, by collision, intentionally suppresses one or more dialect-
registered entries SHOULD document those suppressions explicitly in its addendum. The
rationale is transparency: a dialect may be "permitted" by a profile but partially
amputated by collisions with profile-registered entries, and consumers of the profile
need to know which dialect entries are effectively unavailable. Without explicit
documentation, the amputation is silent and complicates cross-profile reasoning about
dialect behavior.

This obligation is SHOULD in v0.1; a future revision MAY elevate it to MUST once the
profile/dialect registry mechanism in §17 matures.

#### 16.7.3 Dialect vs Dialect

Because v0.1 permits at most one `dialect=` token per document, two dialects cannot be
simultaneously active in the same document. Dialect–dialect collisions therefore do not
arise at validation time in v0.1.

A future revision that introduces multi-dialect activation MUST define a precedence rule
for dialect–dialect collisions (e.g., order of declaration, explicit priority, or
corpus-wide resolution).

In the current revision, the only dialect–dialect concern is **conceptual**: two
dialects registering the same name with different meanings creates fragility if one
future profile permits both. Dialect authors SHOULD avoid colliding names across
registered dialects that target the same domain. See §16.6.2 for the prefix
recommendation that mitigates this risk.

#### 16.7.4 Dialect vs Document

A document MAY contain vocabulary entries that the active dialect has not registered.
These are handled under §16.8 (unknown vocabulary), not as dialect-level conflicts.

### 16.8 Unknown Dialects and Unknown Additions

Two related cases of "unknown" content arise with dialects:

**Unknown dialect name.** The document declares `dialect=<n>` where `<n>` is not
recognized by the consumer.

- In **structural mode** or **core mode**: the unknown dialect name produces a warning.
  The validator falls back to core conformance for vocabulary checks. This is consistent
  with the treatment of unknown profile names (§4.2.3).
- In **profile mode** or **corpus mode**: the unknown dialect name is an error if the
  document declares a profile, because the profile's whitelist cannot be checked without
  recognizing the dialect. The validator MUST report the error and MUST NOT declare the
  document profile-conformant.
- A document that declares only `dialect=<n>` without `profile=` and the dialect is
  unknown falls back to core mode validation.

**Unknown addition under a known dialect.** The document uses a vocabulary entry (a
`cat:` value, role name, atom, etc.) that the named dialect has not registered.

- This is handled as unknown vocabulary, per the effective profile/dialect closure rules
  (§14.4.1). In core mode, unknown entries produce warnings; in profile mode with a
  closed vocabulary, they are errors.
- A dialect does not implicitly open vocabularies. If the profile declares its `cat:`
  registry closed, an unregistered `cat:` value is an error even when the active dialect
  is known, unless the dialect itself registers that value.

### 16.9 Dialect Lifecycle

Dialects progress through the same lifecycle as profiles (§15.1.7):

- **Draft.** The addendum is incomplete or under active revision. Dialect-registered
  entries MAY change. Consumers SHOULD treat Draft dialects as experimental.
- **Candidate.** The addendum is complete per §16.5 and has at least one independent
  implementation. Registered entries are append-only from this point.
- **Stable.** The dialect has been implemented by at least two independent consumers.
  Stable dialects MUST NOT remove or redefine registered entries.

The append-only evolution rule of §14.4.2 applies to dialects: a Candidate or Stable
dialect MAY add new registered entries in subsequent revisions but MUST NOT remove or
redefine existing ones.

### 16.10 Registration

Dialect registration — the process by which a dialect addendum is published, indexed,
and made discoverable to consumers — is the subject of §17 (Registries). Every Stable
dialect MUST be discoverable through the registry mechanism defined there.

In v0.1, no dialects are registered. The registry machinery of §17 is prepared for
future additions.

### 16.11 Open Questions for Future Revisions

- Whether **multi-dialect activation** (two or more `dialect=` tokens per document)
  should be supported, with a precedence rule for dialect–dialect conflicts.
- Whether a **dialect version qualifier** (`dialect=<n>@<version>`) should be
  introduced in parallel with the analogous profile mechanism (§15.5.4).
- Whether **dialect inheritance** (a dialect extending another dialect) should be
  supported as a first-class mechanism.
- Whether **profile-scoped dialect permissions** should be expressible with fine
  granularity, e.g., "profile permits dialect X but only its `cat:` additions, not its
  relation role additions".
- Whether a **dialect conformance test suite** format should be defined in v0.2,
  analogous to the profile conformance test suite (§15.6).

---

## 17. Registries

A **registry** in JOOT v0.1 is a declarative catalog of named entries — categories,
roles, atoms, types, profiles, dialects — together with the metadata required to use
them and the rules under which they are promoted, retired, and discovered. Registries
are how JOOT captures the authoritative definition of vocabulary and governance entities
without prescribing a specific tooling implementation.

This section is **declarative, not operational**. It specifies what registries contain,
how entries are named and promoted, and how consumers find authoritative definitions. It
does NOT prescribe a central server, a DNS hierarchy, a signing infrastructure, or any
network-level retrieval protocol. Operational aspects are deferred to future revisions.

### 17.1 What a Registry Is

A registry is:

1. A **set of named entries**, each with complete metadata (§17.3).
2. A **lifecycle** through which each entry passes (§17.5).
3. A set of **rules** for naming, collision, reservation, and supersession (§17.4).
4. A **source of truth** — explicit about who holds authoritative definitions (§17.7).

A registry is NOT, in v0.1:

- A specific file format.
- A specific server, URL, or API.
- A cryptographically signed artifact.
- A federated, replicated, or distributed store.
- A runtime-accessible service.

These concerns are real but are out of scope for v0.1. Consumers obtain registry entries
through the mechanisms of §17.6, not through a runtime protocol.

### 17.2 Types of Registries

JOOT v0.1 recognizes three categories of registry: universal registries, the profile
registry, and the dialect registry.

#### 17.2.1 Universal Registries

The universal registries are the normative vocabulary catalogs established by this
specification itself. They are authoritative and their sole source of truth is this
document.

| Registry                       | Entries                                          | Defined in |
|--------------------------------|--------------------------------------------------|------------|
| Universal category registry    | `cat:` values                                     | §14.1      |
| Universal relation role registry | Relation role names                             | §14.2      |
| Universal behavior vocabulary  | Bare atoms and keyed descriptors                  | §14.3      |
| Universal constraint vocabulary | Constraint atoms and unit atoms                  | §8.2, §8.3 |
| Universal type vocabulary      | Scalar types, generic constructors, compound types | §7.1–§7.4 |
| Universal sigil set            | Entry sigils                                      | §5.1, §5.3 |

Universal registries are **append-only** across revisions (§14.4.2). A future revision
of this specification MAY add entries; it MUST NOT remove or redefine them.

#### 17.2.2 Profile Registry

The **profile registry** is the catalog of named profiles (§15). In v0.1, this
specification normatively registers two profiles:

| Profile name    | Status    | Defined in |
|-----------------|-----------|------------|
| `code_surface`  | Candidate | §15.3      |
| `api_contract`  | Candidate | §15.4      |

No other profiles are registered in v0.1. Future revisions of this specification or
external parties MAY register additional profiles per the mechanism of §17.

#### 17.2.3 Dialect Registry

The **dialect registry** is the catalog of named dialects (§16). In v0.1, this
specification registers no dialects. The registry machinery is prepared for future
additions.

### 17.3 Registry Entry Requirements

Every registry entry MUST declare its **identity** and its **content** as two distinct
concerns. The separation is deliberate: identity is how an entry is named and referred
to; content is what the entry defines. Conflating them leads to fragile references.

#### 17.3.1 Registry Identity

Every entry in any registry MUST have:

1. **Canonical name.** A lowercase underscored identifier conforming to the grammar of
   its vocabulary. For profiles and dialects, §11.10. For vocabulary entries, the
   grammar of the vocabulary's position (§11).
2. **Registry type.** Which registry the entry belongs to (one of §17.2.1, §17.2.2,
   §17.2.3, or a sub-registry of the universal set).
3. **Status.** One of `Draft`, `Candidate`, `Stable`, `Deprecated`, `Retired`, `Reserved`
   (§17.5).
4. **Version.** A major.minor version applicable to profiles and dialects. Vocabulary
   entries (roles, atoms, types) do not carry their own versions; they inherit the
   version of the specification revision or profile/dialect that registered them.
5. **Authority / editor.** The entity responsible for the entry's definition. For
   entries in universal registries, the authority is the JOOT editors. For profiles and
   dialects, the authority is the profile/dialect author or consortium.

#### 17.3.2 Registry Content

Every entry MUST declare the full metadata required by its kind:

- **For a universal vocabulary entry:** the metadata of §14.4.4 applied to its
  vocabulary class (cat, role, atom, type, etc.).
- **For a profile entry:** the addendum of §15.2 in full.
- **For a dialect entry:** the addendum of §16.5 in full.

An entry whose content fails to meet these requirements MUST NOT reach `Stable` status
(§17.5.4). An entry at `Draft` status MAY have incomplete content, flagged as such.

#### 17.3.3 Separation of Identity and Content

A consumer that needs to *reference* an entry uses its identity (name + registry type +
version + authority). A consumer that needs to *interpret* an entry uses its content
(the full addendum or vocabulary declaration).

Identity is small, stable, and MAY be transmitted in JOOT documents (e.g., `profile=` in
the header). Content is large, revisable, and lives in the specification revision or in
external addenda (§17.6).

### 17.4 Name Collision, Reservation, and Supersession

This section is the rules layer of the registry: it governs how names interact across
registries and across time.

#### 17.4.1 No Reclamation of Universal Names

A name registered in any of the universal registries (§17.2.1) MUST NOT be reclaimed by
a profile, dialect, or external party for a different meaning. Universal names are
reserved for the semantics assigned by this specification in perpetuity.

A profile or dialect MAY reference a universal name (use it, narrow its applicability,
forbid its use in the profile's domain per §14.4.3), but it MUST NOT redefine the name
to mean something else.

#### 17.4.2 Dialect–Dialect Name Collisions

Two dialects MUST NOT register the same name with different meanings if both dialects
have reached `Stable` status.

Specifically:

1. A `Stable` dialect's registered names are reserved across the Stable dialect
   ecosystem for their declared semantics.
2. A `Draft` or `Candidate` dialect SHOULD NOT reuse a name already registered by a
   `Stable` dialect with a different meaning. If the Draft/Candidate dialect proceeds
   toward Stable status, the collision MUST be resolved (typically by renaming) before
   promotion.
3. Two `Draft` dialects MAY temporarily share a name; the first to reach `Stable`
   acquires priority. This is documented so that dialect authors know early-stage
   collisions are expected and resolvable.

The prefix recommendation in §16.6.2 is the primary mitigation: dialect authors who
prefix their entries with a dialect-derived token substantially reduce the risk of
collision from the outset.

#### 17.4.3 Profile–Dialect Collisions

Resolution follows §16.7.2: the profile's declaration wins. The dialect entry is
suppressed in documents declaring that profile. The profile addendum SHOULD document
the suppression per §16.7.2's editorial obligation.

#### 17.4.4 Reservation Before Stabilization

A profile or dialect entering `Candidate` status MAY reserve names for future use even
before the full content is finalized. A reserved name:

- Occupies a slot in the registry under the claiming profile/dialect.
- Cannot be claimed by another profile or dialect for a different meaning.
- Carries status `Reserved` until the content is filled in (at which point it transitions
  to `Candidate`).
- Expires after a reasonable window (to be defined by registry governance, §17.8) if
  content is not provided, at which point the name is released.

Reservation is a soft claim. It prevents accidental collision during stabilization
without committing to a final definition.

#### 17.4.5 Retirement and Tombstoning

When a registry entry is retired (removed from active use by its authority), it does NOT
become immediately available for reuse. The name remains **tombstoned** for a deprecation
window:

- A tombstoned entry retains its identity but has `status: Retired`.
- Its content MAY remain in the registry as historical reference, or MAY be replaced by
  a tombstone marker pointing to a successor entry via `replaced_by:`.
- The name MUST NOT be reclaimed by a different party for a different meaning during
  the deprecation window.
- After the deprecation window closes, the name enters `Reserved` status under registry
  governance and MAY be released for reuse only with explicit editorial approval.

Deprecation windows are profile/dialect-specific and are documented in the registry
entry's metadata. v0.1 does not prescribe a minimum window duration.

### 17.5 Lifecycle

Registry entries progress through the following lifecycle states:

| State        | Meaning                                                                    |
|--------------|----------------------------------------------------------------------------|
| `Reserved`   | Name is claimed but content is not yet provided (§17.4.4).                 |
| `Draft`      | Content is incomplete or under active revision. Addition MAY change.       |
| `Candidate`  | Content is complete; at least one independent implementation exists.       |
| `Stable`     | Content is complete; at least two independent implementations exist; append-only from this point. |
| `Deprecated` | Content is still valid but the entry is discouraged; a successor typically exists. |
| `Retired`    | Entry is withdrawn; tombstoned for a deprecation window (§17.4.5).         |

#### 17.5.1 Promotion

Promotion from one state to the next requires meeting the state's entry criteria:

- **`Reserved` → `Draft`:** content is filled in (at least partially).
- **`Draft` → `Candidate`:** content meets §17.3.2 requirements; at least one independent
  implementation exists.
- **`Candidate` → `Stable`:** at least two independent implementations exist; conformance
  test suite is published; no known unresolved defects.

Promotion is the responsibility of the entry's authority (§17.3.1 point 5).

#### 17.5.2 Demotion

Demotion (moving an entry backward through the lifecycle) is exceptional and SHOULD be
avoided. The common case where demotion applies is when a `Stable` entry is found to
have a critical flaw; in that case, the entry is marked `Deprecated`, not demoted to
`Candidate`. True demotion (`Stable` → `Candidate`) is only permitted when the
specification itself revises the entry's rules and the authority explicitly requests
demotion to signal that consumers SHOULD re-verify their implementations.

#### 17.5.3 Deprecation

An entry is deprecated by its authority when:

- A successor entry exists that supersedes it.
- The entry is found to be harmful or to conflict with core principles.
- The entry's domain is no longer maintained.

Deprecation does not invalidate existing documents that use the entry. It signals to
producers that they SHOULD migrate to a successor.

#### 17.5.4 Append-Only Guarantee

Once an entry reaches `Candidate` status, its registered additions are append-only in
the same sense as the universal registries (§14.4.2):

- A `Candidate` or `Stable` entry MAY have further sub-entries (roles, atoms, etc.)
  added in subsequent revisions.
- A `Candidate` or `Stable` entry MUST NOT have sub-entries removed or redefined.
- Structural changes to the entry itself (name, identity, authority) are not permitted
  after `Candidate`.

### 17.6 Discovery

Discovery is how a consumer obtains the authoritative definition of a registry entry.
Identity (§17.3.1) tells the consumer *what* to look for; discovery tells the consumer
*where* to look. v0.1 recognizes four discovery mechanisms.

#### 17.6.1 In-Spec Reference

The primary discovery mechanism in v0.1 is **in-specification reference**: the entry is
defined by text in this document. Consumers read the specification and interpret the
entry accordingly.

All universal registry entries (§17.2.1) are discovered in-spec. The two profiles
registered in v0.1 (`code_surface`, `api_contract`) are also discovered in-spec, at
§15.3 and §15.4 respectively.

#### 17.6.2 Out-of-Band Addendum

An entry MAY be defined by an addendum document outside this specification. The addendum
is authored by the registry entry's authority (§17.3.1 point 5) and published through a
channel the authority designates (e.g., a specification repository, a published document,
a standards-body submission).

Consumers discover out-of-band addenda through authority-published channels. v0.1 does
not prescribe how these channels operate. A profile addendum author communicates its
location through documentation, not through runtime protocol.

#### 17.6.3 Local Configuration

A consumer MAY be configured with local knowledge of registry entries. This is the
common case for validators shipped with specific profile or dialect support built in:
the consumer's documentation lists which entries it recognizes, and the consumer loads
their definitions from its own codebase or configuration files.

Local configuration is **authoritative for the consumer** but is not authoritative
across consumers. Two consumers with different local configurations may reach different
verdicts on the same document (§15.5.4 forward compatibility).

#### 17.6.4 Implementation-Bundled Registries

Validators, parsers, and other JOOT tooling MAY ship with bundled registry catalogs
(collections of profile addenda, dialect addenda, and vocabulary definitions). Bundled
registries are a convenience layered over local configuration: the tool's authors have
curated a set of entries that the tool recognizes out of the box.

Bundled registries are NOT authoritative by virtue of being bundled. Their authority
derives from the entries' registered authorities (§17.3.1 point 5). A bundled registry
that disagrees with an entry's authoritative source is in error.

### 17.7 Canonical Sources of Truth

The canonical source of truth for each registry entry in v0.1 is:

| Registry                         | Canonical source                                              |
|----------------------------------|---------------------------------------------------------------|
| Universal vocabulary registries  | This specification.                                           |
| Profile `code_surface`           | This specification, §15.3.                                    |
| Profile `api_contract`           | This specification, §15.4.                                    |
| Additional profiles              | The profile's authority-published addendum (out-of-band).     |
| Dialects                         | The dialect's authority-published addendum (out-of-band).     |

When two sources disagree, the canonical source wins. A consumer MUST prefer the
canonical source over bundled or locally configured copies whenever they conflict.

### 17.8 Registry Governance

Registry governance in v0.1 is minimal and **out-of-band**. The specification establishes
the rules (§17.1–§17.7); it does not establish an operational body, a submission
process, a review committee, or a timeline.

The responsibilities of registry governance are:

1. **Editorial stewardship** of the universal registries (amending this specification).
2. **Registration** of profiles and dialects that meet promotion criteria.
3. **Collision arbitration** when §17.4 rules are ambiguous or contested.
4. **Reservation management** (§17.4.4) and tombstone oversight (§17.4.5).
5. **Deprecation and retirement announcements**.

In v0.1, these responsibilities are held by the JOOT editors. A future revision MAY
formalize a broader governance model (consortium, standards body, federated authorities).

**What v0.1 does NOT specify:**

- A central registry server.
- Mandatory URLs for registry entries.
- A cryptographic signature scheme for authoritative definitions.
- A DNS or other network-based name resolution.
- An automatic registry retrieval protocol.

Any consumer or tooling author MAY implement any of these layers on top of the
declarative framework defined here; none of them is required for conformance.

### 17.9 Open Questions for Future Revisions

- Whether a **centralized JOOT registry** (a canonical online catalog of profiles and
  dialects) should be established and, if so, under what governance model.
- Whether **cryptographic signatures** on authoritative addenda should be required or
  optional.
- Whether a **version negotiation protocol** between documents and consumers should be
  formalized (linked to §15.5.4 and §16.11 open questions on version qualifiers).
- Whether **federated registries** (multiple authoritative catalogs coordinated by a
  common protocol) should be supported.
- Whether **automated conformance verification** (a validator that fetches and verifies
  registry entries at runtime) should be specified as an optional consumer feature.
- Whether **registry entry signing and attestation** by the registered authority should
  be introduced as a trust mechanism.
- Whether a **cross-profile vocabulary alignment registry** (a map between identical
  concepts registered under different profiles) should be introduced, tying back to
  §15.6 open questions.

---

# Part V — Conformance

Parts I–IV defined the JOOT format (syntax, semantics, governance) and the mechanisms by
which vocabularies are registered and extended. Part V defines the **conformance system**
that governs how documents, producers, consumers, validators, profiles, and corpora
satisfy the specification. It consolidates the obligations scattered across Parts I–IV
into a unified normative framework and introduces the error catalog (§19) and validator
requirements (§20) that tooling implementers rely on.

Part V does not redefine concepts introduced elsewhere. Where it touches topics already
covered (validation modes in §1.3, core/profile/corpus conformance in §1.2, target
resolution in §6.4.4, etc.), it cross-references those sections rather than duplicating
them. Part V's contribution is the **normative enumeration of obligations** per role and
the **error catalog** that validators use to report violations uniformly.

---

## 18. Conformance

This section defines six distinct **conformance angles**, each covering a different role
or artifact in the JOOT ecosystem. An implementation or document MAY be conformant along
some angles and not others; the angles are independent in principle but related in
practice (§18.8).

### 18.1 Overview

| Angle                    | Applies to                                     | What it asserts                                                          | Defined in |
|--------------------------|------------------------------------------------|--------------------------------------------------------------------------|------------|
| Document conformance     | A `.joot` file (or in-memory equivalent)       | The document satisfies syntactic and semantic rules for its declared layer. | §18.2      |
| Producer conformance     | A tool or process that emits JOOT              | Emitted output is always document-conformant and canonical.               | §18.3      |
| Consumer conformance     | A tool or process that reads JOOT              | Reader correctly parses conformant documents and handles non-conformant input per specification. | §18.4      |
| Validator conformance    | A tool that checks JOOT documents or corpora   | Validator implements the specification's validation rules correctly for a declared mode. | §18.5      |
| Profile conformance      | A document with a `profile=` declaration       | The document satisfies the declared profile's rules in addition to core. | §18.6      |
| Corpus conformance       | A set of JOOT documents treated as one scope   | All documents satisfy profile conformance and cross-document invariants hold. | §18.7      |

Each angle is normatively specified in its dedicated subsection. A tool or document
seeking to claim conformance MUST specify **which angle and at which layer** (core,
profile, or corpus) the claim applies. Generic claims of "JOOT conformance" without
qualification are insufficient.

### 18.2 Document Conformance

A document is **conformant** when it satisfies the requirements of the conformance layer
applicable to it. The layers are defined in §1.2.

- **Core-conformant document.** Satisfies the syntactic and structural rules of §§4–11,
  the identity model of §12 (where `id:` values are present), the graph model of §13
  (where relations are present), and the universal vocabulary rules of §14 under core
  closure (unknown entries produce warnings, not errors).
- **Profile-conformant document.** Core-conformant plus all additional requirements of
  the declared profile per §15 and the profile's addendum.
- **Corpus-conformant document.** Profile-conformant plus all cross-document invariants
  of §1.2.3 (`(id, facet)` uniqueness across the corpus, internal target resolution
  across the corpus, `cat:`/`kind:` preservation across facets).

A document declares certain conformance-relevant information through its header (§4.2),
most notably its JOOT version and any active profile or dialect. Core conformance is the
default when no profile is declared. Corpus conformance is evaluated out-of-band by the
validator over a set of documents (§12.5). A validator evaluating conformance MUST
evaluate against the declared layer, not a weaker or stronger one, unless explicitly
configured otherwise.

**Conformance is binary per layer.** A document either is or is not conformant at a
given layer. Partial conformance ("mostly conformant with three warnings") is not a
conformance status; such a document is non-conformant at the layer where violations
exist, even if the violations are minor. Consumers MAY still process such documents
under the lenient-consumption rules of §1.5, but the document's non-conformant status is
unchanged by consumer leniency.

### 18.3 Producer Conformance

A **producer** is a tool, process, or party that emits JOOT content. Producer conformance
is the stricter end of the producer–consumer asymmetry of §1.4: producers are held to
emitting canonical, conformant output; consumers are permitted leniency on input.

A conformant producer MUST:

1. Emit documents that are conformant at the claimed document layer (core or profile).
   A producer that emits or assembles an entire corpus MAY additionally claim corpus-
   conformant output only when the corpus as a whole satisfies §18.7.
2. Emit **canonical ordering** wherever defined by the specification: canonical meta key
   order (§6.3.2), canonical `[Rel]` item order (§6.4.2), canonical constraint atom
   order (§8.6), canonical block order within entries (§6.2).
3. Emit `id:` and `kind:` on every entry when producing profile-conformant content
   under the standard profiles (§15.3, §15.4).
4. Ensure the header's `Entries:` count (§4.2.1 line 4) matches the actual entry count
   in the document.
5. Escape or quote reserved characters per §9, choosing the canonical form per §9.5.
6. Avoid ad-hoc blocks (§6.2), reserved sigils (§5.3), and forbidden facet patterns
   (§12.3.3).
7. Emit only characters permitted by §3.3 in each position.
8. Produce physically normalized files per §4.6 (single `LF`, no BOM, no tabs, no
   trailing spaces, correct blank-line layout).

A conformant producer SHOULD:

- Emit `LF` line endings exclusively, even on platforms that default to `CRLF`.
- Keep entry lines within practical length thresholds (§4.7) and decompose oversized
  entries into facets or child subjects.
- Emit canonical quoted vs unquoted forms per §9.5 when both are valid.
- Prefer unfaceted subjects when facet decomposition is not justified by §12.3.5.

A conformant producer MUST NOT:

- Emit non-canonical ordering when targeting profile or corpus conformance (canonical
  ordering is an error at those layers, not a warning).
- Emit anonymous entries (without `id:`) when targeting profile conformance under any
  standard profile (§15).
- Rely on consumer recovery behavior for interoperability (§1.5).

**Partial producer conformance.** A producer MAY be conformant only at a subset of
layers (e.g., a producer that emits only core-conformant output without profile support).
Such a producer MUST clearly declare which layers it supports and MUST NOT claim
higher-layer conformance.

### 18.4 Consumer Conformance

A **consumer** is a tool, process, or party that reads JOOT content. Consumer conformance
is the lenient end of the asymmetry: consumers are obligated to handle conformant input
correctly and to handle non-conformant input gracefully per §1.5.

A conformant consumer MUST:

1. Correctly parse every core-conformant document per the rules of Parts I–II.
2. Accept both `LF` and `CRLF` line endings, normalizing to `LF` on read (§2.5).
3. Accept both quoted and unquoted equivalent values as semantically identical (§9.5),
   except where the specification forbids a specific form (e.g., `id:` values MUST be
   unquoted, §11.3.4).
4. Apply the graceful-consumption behavior of §1.5: continue reading past semantic
   violations and report them, rather than aborting.
5. Abort only on hard syntactic errors that prevent further parsing: malformed header
   (§4.2), entry split across lines (§4.4), tab characters (§2.6), unclosed quoted
   strings (§9.4), invalid UTF-8 (§2.4), BOM in unexpected position (§2.4).
6. Treat non-canonical ordering as a **warning** at core conformance and as an **error**
   only when evaluating profile or corpus conformance (§6.3.2, §6.4.2, §8.6).
7. Treat unknown `cat:`, `kind:`, role, or atom as a **warning** at core conformance and
   as an **error** only when the active profile's declared closure rules apply (§14.4.1).
8. Resolve relation targets per §6.4.4, applying the scope appropriate to the validation
   mode (§1.3).

A conformant consumer SHOULD:

- Normalize BOM and trailing spaces on read and report them as warnings (§4.6).
- Report all violations detected, not just the first one, to the extent practical.
- Preserve source document structure when round-tripping (parse-and-emit) so that
  parse-emit-parse reaches a fixed point for canonical input.

A conformant consumer MUST NOT:

- Reject a core-conformant document on grounds not specified in §1.5 or this section.
- Silently alter the document's semantic content during parse or emit.
- Derive structural or identity information from annotations (§10.4).
- Treat recovery behavior as part of the language (§1.5).

### 18.5 Validator Conformance

A **validator** is a consumer specialized for verifying conformance and reporting
violations. Validator conformance is the strictest consumer-role conformance; it adds
obligations beyond §18.4.

A conformant validator MUST:

1. Declare the validation mode it operates in (§1.3): structural, core, profile, or
   corpus. A validator that supports multiple modes MUST document its default and the
   mechanism for selecting among them.
2. Apply only the rules appropriate to the declared mode. A validator in core mode MUST
   NOT apply profile rules even when a profile is declared in the document header; a
   validator in profile mode MUST apply both core and profile rules.
3. Report every detected violation using the error catalog of §19. Each reported
   violation MUST carry:
   - An error or warning code (§19).
   - A source location (line number, and column number where practical).
   - A short description.
   - Optionally, a reference to the specification section that the violation breaches.
4. Distinguish errors from warnings per the severity rules of §19.
5. For profile and corpus mode, correctly implement the profile's conformance checks as
   enumerated in the profile's addendum (§15.3.11 for `code_surface`, §15.4.13 for
   `api_contract`).
6. For corpus mode, correctly resolve relation targets across the corpus and enforce
   cross-document identity uniqueness per §12.2.

A conformant validator SHOULD:

- Provide actionable error messages that name the exact problem and suggest a fix when
  one is apparent.
- Report violations in document order (source-position order) when practical.
- Support partial-document validation (validating a range or subset of a document) for
  tooling integration.

A conformant validator MUST NOT:

- Silently modify the document during validation.
- Apply recovery rules that are not specified in §1.5.
- Invent violations not specified in this document or in a registered profile or
  dialect addendum.
- Declare a document conformant if it has **any** error at the target conformance layer,
  even if all errors are individually minor.

Detailed validator requirements, including the shape of diagnostic output and minimum
recovery behavior, are specified in §20.

### 18.6 Profile Conformance

**Profile conformance** is a document-level angle that builds on core conformance by
additionally applying the rules of a declared profile. It is defined at §1.2.2 and
§15.1.4; this subsection consolidates the obligations without redefining them.

A document is profile-conformant when:

1. Its header declares `profile=<n>` for a registered profile (§17).
2. It is core-conformant.
3. It satisfies every rule in the declared profile's addendum:
   - `cat:` and `kind:` vocabulary closure (§15.1.2).
   - Required meta keys (typically `id:`, `kind:`; profile-specific additions).
   - Canonical ordering enforced as errors, not warnings (§15.1.4).
   - Relation role registry and its narrowed source/target constraints (§15.2).
   - Behavior vocabulary narrowing (§15.3.5, §15.4.10).
   - Facet vocabulary and decomposition rules (§15.3.8, §15.4.12).
   - Minimum conformance checks per the profile's addendum (§15.3.11, §15.4.13).
4. `(id, facet)` identity uniqueness holds at document scope (§12.2).

A document that declares a profile but fails to satisfy the profile's rules is
**non-conformant** at the profile layer, even if it remains core-conformant. A validator
evaluating in profile mode MUST report the violations and MUST NOT declare the document
profile-conformant.

A document that declares an unknown profile follows the mode-dependent behavior of
§4.2.3: warning in structural and core mode, error in profile and corpus mode.

### 18.7 Corpus Conformance

**Corpus conformance** is a set-level angle that applies when multiple JOOT documents
are validated together as a single scope. It is defined at §1.2.3 and refined here.

A corpus is conformant when:

1. Every document in the corpus is profile-conformant (§18.6). A corpus where even one
   member document fails profile conformance is itself non-conformant.
2. `(id, facet)` identity uniqueness holds **across** the corpus, not only within
   individual documents (§12.2).
3. Every internal relation target (§6.4.3, §6.4.4) resolves to an entry within the
   corpus. Unresolved internal targets that would be warnings in document scope become
   errors at corpus scope.
4. `cat:` and `kind:` values are preserved across all facets of the same `id:`, even
   when facets reside in different documents (§12.3.4 rules 1 and 2).
5. No two documents declaring the same profile disagree on profile-internal conventions
   (e.g., two `api_contract` documents modeling the same endpoint under different
   linked-contract/endpoint-facet forms is a corpus conformance error per §15.4.5).

Corpus boundaries are declared out-of-band (§12.5). The specification does not prescribe
how a corpus is assembled; it prescribes only what conformance requires of the assembled
set.

A corpus validator — a validator operating in corpus mode (§1.3) — MUST have access to
all documents in the corpus and MUST apply the cross-document checks above. A validator
that evaluates documents individually cannot establish corpus conformance and MUST NOT
claim to validate corpus conformance.

### 18.8 Relationships Between Conformance Angles

The six conformance angles are logically independent but operationally related. This
subsection makes the relationships explicit to prevent confusion.

**Angles that depend on others:**

- **Profile conformance implies core conformance.** A document cannot be profile-
  conformant without also being core-conformant (§18.6 point 2).
- **Corpus conformance implies profile conformance** for each member document (§18.7
  point 1). A corpus of core-conformant-only documents does not reach corpus conformance
  under any standard profile.
- **Validator conformance implies consumer conformance.** A validator MUST satisfy all
  consumer obligations (§18.4) in addition to its validator-specific obligations
  (§18.5).

**Angles that are independent:**

- **A document can be conformant even if no conformant validator exists.** Documents are
  judged against the specification text, not against tool implementations. A
  hypothetical document satisfying all rules is conformant whether or not a tool can
  verify it.
- **A consumer can be conformant without being a producer.** Read-only tools (linters,
  viewers, analyzers) satisfy consumer conformance without emitting JOOT.
- **A producer can be conformant without being a consumer.** Code generators that emit
  JOOT from an external source (e.g., converting OpenAPI to JOOT) need not parse JOOT
  input.
- **Profile conformance of a document is independent of corpus conformance.** A
  single-document profile-conformant file is profile-conformant whether or not it is
  ever placed in a corpus. Corpus conformance adds checks; it does not remove them.
- **Two documents can each be profile-conformant yet together fail corpus
  conformance.** The cross-document invariants of §18.7 may be violated by a combination
  of individually-conformant documents (e.g., two documents both defining the same
  `(id, facet)`).

**Angles with asymmetric strictness:**

- **Producer conformance is strictly stronger than consumer conformance.** Producers
  must emit canonical, strictly-conformant output; consumers must tolerate a wider input
  envelope per §1.5.
- **Validator conformance is strictly stronger than consumer conformance.** Validators
  must additionally report violations uniformly per §19 and implement mode-specific
  rules.

### 18.9 Open Questions for Future Revisions

- Whether **partial conformance levels** (e.g., "profile-conformant with at most N
  warnings") should be formalized as conformance sub-grades, or remain absent. v0.1
  treats conformance as binary per layer.
- Whether a **certified conformance** badge or attestation mechanism for producers,
  consumers, and validators should be introduced, and under what governance (§17.8).
- Whether **corpus conformance** should support progressive validation (validating
  documents as they are added to the corpus, rather than validating the corpus as a
  whole), and how that interacts with cross-document invariants.
- Whether **round-trip fidelity** (a conformance angle asserting that parse-emit preserves
  all semantic content exactly) should be promoted from a SHOULD in §18.4 to a normative
  angle of its own.
- Whether **multi-profile corpora** (corpora containing documents declaring different
  profiles) need a dedicated cross-profile conformance angle.

---

## 19. Error Catalog

This section is the normative catalog of error and warning codes that validators use to
report violations uniformly. Every forward-reference to §19 elsewhere in this
specification is fulfilled here: each violation mentioned in Parts I–IV receives a
catalog entry, and §19.3 through §19.6 enumerate every universal code.

The catalog has four goals:

1. **Uniformity.** Every conformant validator (§18.5) reports the same violation with
   the same code, so that diagnostic output is portable across tools.
2. **Discoverability.** A consumer encountering a code can look it up here and find the
   exact specification section that defines the violation.
3. **Stability.** Codes are append-only and never reused (§19.1.3), so downstream
   tooling can rely on them across revisions.
4. **Extensibility.** Profiles and dialects can register their own codes without
   colliding with the universal catalog (§19.9).

### 19.1 Error Code Scheme

#### 19.1.1 Format

Every code has the form:

```
<FAMILY>-<NNN>
```

Where:

- `<FAMILY>` is a three-letter uppercase ASCII prefix identifying the error family.
- `<NNN>` is a three-digit zero-padded decimal number in the range `001` to `999`.
- The separator is a single hyphen-minus (`U+002D`).

Examples: `SYN-001`, `SEM-042`, `CON-120`, `WRN-007`.

#### 19.1.2 Universal Families

The four universal error families are:

| Family   | Scope                                                               | Range              |
|----------|---------------------------------------------------------------------|--------------------|
| `SYN`    | **Syntactic errors.** Violations of the grammar and structural rules of Parts I–II (file layout, header, entries, blocks, quoting, escaping, identifiers). | `SYN-001`–`SYN-999` |
| `SEM`    | **Semantic errors.** Violations of the identity, graph, and vocabulary rules of Part III (subject identity, facet rules, relation semantics, type equivalence). | `SEM-001`–`SEM-999` |
| `CON`    | **Conformance errors.** Violations of profile, corpus, or validator-specific rules of Parts IV–V (profile closure, corpus cross-document invariants, validator mode rules). | `CON-001`–`CON-999` |
| `WRN`    | **Warnings.** Non-critical violations that do not invalidate conformance at the applicable layer but are worth reporting. | `WRN-001`–`WRN-999` |

#### 19.1.3 Stability Rules

1. **`000` is reserved** in every family. It MUST NOT be assigned to any violation. It
   is reserved for future use (e.g., as a "family catchall" or "unknown code within
   family" indicator).
2. **Codes are append-only across revisions.** A future revision of this specification
   MAY add new codes; it MUST NOT remove or redefine existing codes.
3. **Codes are never reused.** If a code is deprecated or retired (§19.1.4), its number
   remains retired; no subsequent violation receives that number.
4. **Family prefixes MUST NOT be reused** across different families. The four universal
   prefixes `SYN`, `SEM`, `CON`, `WRN` are reserved in perpetuity.

#### 19.1.4 Deprecation and Retirement

A code MAY be deprecated when the violation it reports is no longer detectable (e.g.,
because the specification rule that generated it has been removed or superseded). A
deprecated code:

- Retains its number and position in the catalog.
- Is marked `Deprecated` in the catalog.
- MUST NOT be emitted by validators conforming to revisions that deprecate it, but
  consumers of older diagnostic output MUST still recognize the code.
- MAY transition to `Retired` in a subsequent revision, at which point the code becomes
  historical and is no longer part of the active catalog but remains reserved.

No universal codes are deprecated or retired in v0.1.

### 19.2 Severity Model

Each code carries a **severity** that determines whether it invalidates conformance and
under which modes.

#### 19.2.1 Severity Values

| Severity | Meaning                                                                          |
|----------|----------------------------------------------------------------------------------|
| `error`  | Invalidates conformance at the applicable layer. A document with any `error`-severity violation is non-conformant at that layer. |
| `warning`| Does not invalidate conformance. Reported for information and quality purposes. |

#### 19.2.2 Severity Varies by Mode

Many codes have **mode-dependent severity**: the same violation is a warning in core
mode and an error in profile or corpus mode. For example, non-canonical meta key order
(§6.3.2) is:

- `warning` in structural or core mode,
- `error` in profile or corpus mode.

Every catalog entry in §19.3–§19.6 specifies its severity per mode. Where a single
severity applies across all modes, the catalog entry states that directly. Where the
severity varies, the catalog entry provides a per-mode table.

#### 19.2.3 Hard Syntactic Errors

A small set of violations are **hard syntactic errors** that a consumer MAY abort on
(§1.5, §18.4 point 5). These are always `error` severity regardless of mode and cannot
be demoted to warnings. They are marked `hard` in the catalog.

Hard errors, by code:

| Code      | Violation                                     |
|-----------|-----------------------------------------------|
| `SYN-001` | Invalid UTF-8 sequence                        |
| `SYN-002` | BOM in unexpected position                    |
| `SYN-003` | Bare `CR` not part of `CRLF`                  |
| `SYN-004` | Tab character present                         |
| `SYN-010` | Incorrect case in reserved identifier         |
| `SYN-011` | Malformed header — missing line               |
| `SYN-012` | Malformed header — extra line                 |
| `SYN-013` | Invalid version token on header line 1        |
| `SYN-030` | Entry line ends inside a block                |
| `SYN-056` | Unclosed quoted string                        |
| `SYN-057` | Newline inside quoted string                  |

These are the only codes marked `error (hard)` in §19.3. A consumer MAY abort parsing
on any of them; all other `error`-severity codes do not justify aborting under §1.5.

#### 19.2.4 Assigning Severity to Detected Violations

When a validator detects a violation, it MUST:

1. Identify the code from this catalog.
2. Determine the validation mode (§1.3).
3. Apply the code's mode-dependent severity to produce the effective severity for this
   report.
4. Emit the diagnostic per §19.7.

A validator MUST NOT report a code at a severity different from what the catalog
specifies. Severity is not a validator-configurable behavior.

### 19.3 SYN Catalog — Syntactic Errors

This family covers grammatical and structural violations detected during parsing. Most
SYN codes are mode-independent (they are errors in every mode) because they prevent
meaningful interpretation of the document.

| Code      | Title                                           | Section    | Severity (all modes) |
|-----------|-------------------------------------------------|------------|----------------------|
| `SYN-001` | Invalid UTF-8 sequence                          | §2.4       | `error (hard)`       |
| `SYN-002` | BOM in unexpected position                      | §2.4       | `error (hard)`       |
| `SYN-003` | Bare `CR` not part of `CRLF`                    | §2.5       | `error (hard)`       |
| `SYN-004` | Tab character present                           | §2.6       | `error (hard)`       |
| `SYN-005` | Disallowed whitespace character                 | §2.6       | `error`              |
| `SYN-006` | Leading or trailing space on entry line         | §2.6       | `error`              |
| `SYN-007` | Control character in forbidden range            | §3.3       | `error`              |
| `SYN-008` | Surrogate code point                            | §3.3       | `error`              |
| `SYN-009` | Non-ASCII character in structural syntax        | §3.3       | `error`              |
| `SYN-010` | Incorrect case in reserved identifier           | §3.4       | `error (hard)`       |
| `SYN-011` | Malformed header — missing line                 | §4.2       | `error (hard)`       |
| `SYN-012` | Malformed header — extra line                   | §4.2       | `error (hard)`       |
| `SYN-013` | Invalid version token on header line 1          | §4.2.2     | `error (hard)`       |
| `SYN-014` | Unsupported major version                       | §4.2.2     | `error`              |
| `SYN-015` | Malformed `profile=` token                      | §4.2.3     | `error`              |
| `SYN-016` | Repeated `profile=` token                       | §4.2.3     | `error`              |
| `SYN-017` | Malformed `dialect=` token                      | §4.2.4     | `error`              |
| `SYN-018` | Repeated `dialect=` token                       | §4.2.4     | `error`              |
| `SYN-019` | Unknown token on header line 1                  | §4.2.5     | `error`              |
| `SYN-020` | Header `Entries:` count mismatch                | §4.2.1     | `error`              |
| `SYN-021` | Missing blank line between header and sections  | §4.5       | `error`              |
| `SYN-022` | Missing blank line before section               | §4.5       | `error`              |
| `SYN-023` | Blank line in forbidden position                | §4.5       | `error`              |
| `SYN-024` | Spaces-only line                                | §4.5       | `error`              |
| `SYN-025` | Multiple trailing blank lines                   | §4.5       | `error`              |
| `SYN-026` | Invalid section header format                   | §4.3, §11.1 | `error`             |
| `SYN-027` | Empty section name                              | §11.1      | `error`              |
| `SYN-028` | Invalid section name character                  | §11.1      | `error`              |
| `SYN-029` | Entry outside any section                       | §4.4       | `error`              |
| `SYN-030` | Entry line ends inside a block                  | §4.4       | `error (hard)`       |
| `SYN-031` | Reserved sigil used (`?`, `%`)                  | §5.3       | `error`              |
| `SYN-032` | Invalid sigil character                         | §5.1       | `error`              |
| `SYN-033` | Space between sigil and entry name              | §5.5       | `error`              |
| `SYN-034` | Invalid character in entry name                 | §11.2      | `error`              |
| `SYN-035` | Entry name begins with `_` or `>`               | §11.2      | `error`              |
| `SYN-036` | Entry name ends with `>`                        | §11.2      | `error`              |
| `SYN-037` | Consecutive `>` in entry name                   | §11.2      | `error`              |
| `SYN-038` | Meta block missing or `cat:` not first          | §6.3.1     | `error`              |
| `SYN-039` | Unknown block marker                            | §6.2       | `error`              |
| `SYN-040` | Repeated block on same entry                    | §6.2.1     | `error`              |
| `SYN-041` | Empty named block                               | §6.2       | `error`              |
| `SYN-042` | Duplicate non-repeatable meta key               | §6.3.3     | `error`              |
| `SYN-043` | Meta key without colon separator                | §6.3, §11.5 | `error`             |
| `SYN-044` | Space around meta key colon                     | §11.5      | `error`              |
| `SYN-045` | Duplicate field name in `[Fields]`              | §6.5.3     | `error`              |
| `SYN-046` | Field or return item missing type               | §7.1       | `error`              |
| `SYN-047` | Invalid generic constructor arity               | §7.2.2     | `error`              |
| `SYN-048` | Whitespace inside generic arguments             | §7.2.2     | `error`              |
| `SYN-049` | Empty enum                                      | §7.4       | `error`              |
| `SYN-050` | Duplicate union alternative                     | §7.5.2     | `error`              |
| `SYN-051` | Empty constraint block                          | §8.1       | `error`              |
| `SYN-052` | Duplicate constraint atom                       | §8.1       | `error`              |
| `SYN-053` | Invalid constraint atom grammar                 | §8.1       | `error`              |
| `SYN-054` | Invalid number literal                          | §8.1       | `error`              |
| `SYN-055` | Invalid escape sequence                         | §9.3       | `error`              |
| `SYN-056` | Unclosed quoted string                          | §9.4       | `error (hard)`       |
| `SYN-057` | Newline inside quoted string                    | §9.4       | `error (hard)`       |
| `SYN-058` | Empty unquoted value                            | §9.5       | `error`              |
| `SYN-059` | Reserved character unescaped in unquoted value  | §9.2       | `error`              |
| `SYN-060` | Empty annotation content                        | §10.1, §10.2 | `error`            |
| `SYN-061` | Annotation ordering violation (note after warn) | §10.3      | `error`              |
| `SYN-062` | Invalid `id:` character                         | §11.3.1    | `error`              |
| `SYN-063` | `#` character in `id:` value                    | §11.3.4    | `error`              |
| `SYN-064` | Empty `id:` segment                             | §11.3.1    | `error`              |
| `SYN-065` | Quoted `id:` value                              | §11.3.4    | `error`              |
| `SYN-066` | Invalid field name character                    | §11.4      | `error`              |
| `SYN-067` | Invalid profile or dialect name                  | §11.10     | `error`              |

Codes `SYN-068` through `SYN-999` are unassigned and available for future additions.

### 19.4 SEM Catalog — Semantic Errors

This family covers violations of the semantic models of Part III: identity, graph, and
vocabulary. Most SEM codes have mode-dependent severity because semantic violations are
often permitted at core conformance but prohibited under profile or corpus conformance.

| Code      | Title                                                | Section    | Core     | Profile | Corpus  |
|-----------|------------------------------------------------------|------------|----------|---------|---------|
| `SEM-001` | Anonymous entry (no `id:`)                           | §12.6      | allowed  | `error` | `error` |
| `SEM-002` | `facet:` declared without `id:`                      | §12.6      | `error`  | `error` | `error` |
| `SEM-003` | Duplicate `(id, facet)` tuple                         | §12.2      | `error`  | `error` | `error` |
| `SEM-004` | Unfaceted + faceted coexistence for same `id:`       | §12.3.2    | `error`  | `error` | `error` |
| `SEM-005` | Forbidden facet pattern (child-as-facet)             | §12.3.3    | `error`  | `error` | `error` |
| `SEM-006` | Forbidden facet pattern (instance-as-facet)          | §12.3.3    | `error`  | `error` | `error` |
| `SEM-007` | Forbidden facet pattern (version-as-facet)           | §12.3.3    | `error`  | `error` | `error` |
| `SEM-008` | Forbidden facet pattern (status-as-facet)            | §12.3.3    | `error`  | `error` | `error` |
| `SEM-009` | Forbidden facet pattern (environment-as-facet)       | §12.3.3    | `error`  | `error` | `error` |
| `SEM-010` | Facets disagree on `cat:`                            | §12.3.4    | `warning`| `error` | `error` |
| `SEM-011` | Facets disagree on `kind:`                           | §12.3.4    | `warning`| `error` | `error` |
| `SEM-012` | Invalid facet value grammar                          | §12.3.6    | `error`  | `error` | `error` |
| `SEM-013` | Unresolved internal relation target (document scope) | §6.4.4     | `warning`| `warning`| `error`|
| `SEM-014` | Unresolved internal relation target (corpus scope)   | §6.4.4     | n/a      | n/a     | `error` |
| `SEM-015` | Identity modeling conflict on resolution             | §6.4.4     | `error`  | `error` | `error` |
| `SEM-016` | Exact duplicate relation edge                        | §6.4.5, §13.4 | `error` | `error`| `error` |
| `SEM-017` | Registered role used as behavior key                 | §6.6.3     | `error`  | `error` | `error` |
| `SEM-018` | Conditional output block name invented                | §6.7.3    | `error`  | `error` | `error` |
| `SEM-019` | Unknown nominal type in generic argument             | §7.2.1     | `warning`| `error` | `error` |
| `SEM-020` | Conflicting constraint atoms                         | §8.5       | `error`  | `error` | `error` |
| `SEM-021` | Unknown universal `cat:` value                       | §14.1      | `warning`| `error` | `error` |
| `SEM-022` | Unknown universal relation role                      | §14.2      | `warning`| `error` | `error` |
| `SEM-023` | Unknown universal behavior atom                      | §14.3.1    | `warning`| `error` | `error` |
| `SEM-024` | Unknown universal keyed descriptor                   | §14.3.2    | `warning`| `error` | `error` |
| `SEM-025` | Unknown universal constraint atom                    | §8.2, §8.3 | `warning`| `error` | `error` |
| `SEM-026` | Non-NFC Unicode content                              | §3.2       | `warning`| `warning`| `warning` |
| `SEM-027` | Annotation carries structural-looking content         | §10.4     | `warning`| `warning`| `warning` |

Codes `SEM-028` through `SEM-999` are unassigned and available for future additions.

### 19.5 CON Catalog — Conformance Errors

This family covers profile, corpus, and validator-specific violations. Most CON codes
apply only under profile or corpus mode; at core conformance, the underlying rules do
not apply.

| Code      | Title                                                         | Section      | Core     | Profile | Corpus  |
|-----------|---------------------------------------------------------------|--------------|----------|---------|---------|
| `CON-001` | Unknown profile name                                          | §4.2.3       | `warning`| `error` | `error` |
| `CON-002` | Unknown dialect name                                          | §4.2.4       | `warning`| `error` | `error` |
| `CON-003` | Dialect not in profile whitelist                              | §15.1.5, §16.4 | n/a    | `error` | `error` |
| `CON-004` | Missing required meta key (`id:`)                             | §15          | allowed  | `error` | `error` |
| `CON-005` | Missing required meta key (`kind:`)                           | §15          | allowed  | `error` | `error` |
| `CON-006` | Non-canonical meta key order                                  | §6.3.2       | `warning`| `error` | `error` |
| `CON-007` | Non-canonical `[Rel]` item order                              | §6.4.2       | `warning`| `error` | `error` |
| `CON-008` | Non-canonical constraint atom order                           | §8.6         | `warning`| `error` | `error` |
| `CON-009` | Profile-closed vocabulary: unregistered `cat:`                | §14.4.1      | n/a      | `error` | `error` |
| `CON-010` | Profile-closed vocabulary: unregistered `kind:`               | §14.4.1      | n/a      | `error` | `error` |
| `CON-011` | Profile-closed vocabulary: unregistered role                  | §14.4.1      | n/a      | `error` | `error` |
| `CON-012` | Profile-closed vocabulary: unregistered behavior atom         | §14.4.1      | n/a      | `error` | `error` |
| `CON-013` | Profile-closed vocabulary: unregistered keyed descriptor      | §14.4.1      | n/a      | `error` | `error` |
| `CON-014` | Profile-closed vocabulary: unregistered constraint atom       | §8.7         | n/a      | `error` | `error` |
| `CON-015` | Role source `cat:` mismatch under profile                     | §13.9, §15   | n/a      | `error` | `error` |
| `CON-016` | Role target `cat:` mismatch under profile                     | §13.9, §15   | n/a      | `error` | `error` |
| `CON-017` | Role cardinality violated under profile                       | §6.4.5, §13.8, §15 | n/a | `error` | `error` |
| `CON-018` | `ext:` target on role where profile forbids it                | §6.4.3, §15  | n/a      | `error` | `error` |
| `CON-019` | Forbidden behavior descriptor under profile                   | §15          | n/a      | `error` | `error` |
| `CON-020` | Behavior descriptor value outside profile-narrowed set         | §15         | n/a      | `error` | `error` |
| `CON-021` | `api_contract`: endpoint missing `path:`                      | §15.4.6      | n/a      | `error` | `error` |
| `CON-022` | `api_contract`: endpoint missing `verb:`                      | §15.4.6      | n/a      | `error` | `error` |
| `CON-023` | `api_contract`: invalid `verb:` value                         | §15.4.6      | n/a      | `error` | `error` |
| `CON-024` | `api_contract`: facets disagree on `path:`                    | §15.4.6      | n/a      | `error` | `error` |
| `CON-025` | `api_contract`: facets disagree on `verb:`                    | §15.4.6      | n/a      | `error` | `error` |
| `CON-026` | `api_contract`: contract carries `path:` or `verb:`           | §15.4.6      | n/a      | `error` | `error` |
| `CON-027` | `api_contract`: `status:` carries HTTP status code            | §15.4.7      | n/a      | `error` | `error` |
| `CON-028` | `api_contract`: endpoint mixes linked-contract and endpoint-facet forms | §15.4.5 | n/a | `error` | `error`|
| `CON-029` | `api_contract`: response cardinality mismatch for verb        | §15.4.7      | n/a      | `error` | `error` |
| `CON-030` | `api_contract`: `status_code:` not a three-digit integer      | §15.4.9      | n/a      | `error` | `error` |
| `CON-031` | `code_surface`: member `member_of` cardinality not `1`        | §15.3.11     | n/a      | `error` | `error` |
| `CON-032` | `code_surface`: callable overloads share `id:`                | §15.3.9      | n/a      | `error` | `error` |
| `CON-033` | `code_surface`: `path:` or `verb:` on code surface entry      | §15.3.6      | n/a      | `error` | `error` |
| `CON-034` | Corpus: cross-document `(id, facet)` duplicate                | §12.2, §18.7 | n/a      | n/a     | `error` |
| `CON-035` | Corpus: cat/kind preservation violated across facets          | §12.3.4, §18.7 | n/a   | n/a     | `error` |
| `CON-036` | Corpus: profile-internal convention disagreement              | §18.7        | n/a      | n/a     | `error` |

Codes `CON-037` through `CON-999` are unassigned and available for future additions.

### 19.6 WRN Catalog — Warnings

This family covers non-critical violations that do not invalidate conformance but merit
reporting.

| Code      | Title                                                      | Section    | Severity        |
|-----------|------------------------------------------------------------|------------|-----------------|
| `WRN-001` | BOM stripped on read                                       | §2.4, §4.6 | `warning`       |
| `WRN-002` | `CRLF` normalized to `LF` on read                          | §2.5, §4.6 | `warning`       |
| `WRN-003` | Trailing spaces stripped on read                           | §4.6       | `warning`       |
| `WRN-004` | One trailing blank line at EOF                             | §4.5       | `warning`       |
| `WRN-005` | Oversized entry line                                       | §4.7       | `warning`       |
| `WRN-006` | Overuse of `params` type                                   | §7.4       | `warning`       |
| `WRN-007` | Non-canonical quoted vs unquoted form                      | §9.5       | `warning`       |
| `WRN-008` | Non-canonical union alternative order                      | §7.5.2     | `warning`       |
| `WRN-009` | Non-canonical behavior item grouping                       | §6.6.2     | `warning`       |
| `WRN-010` | Non-canonical field item order (profile-deferred)          | §6.5.2     | `warning`       |
| `WRN-011` | Sigil does not match `cat:` (advisory mismatch)            | §5.2       | `warning`       |
| `WRN-012` | Dialect–profile name collision detected                    | §16.7.2    | `warning`       |
| `WRN-013` | Bidi control character outside quoted string               | §3.6       | `warning`       |
| `WRN-014` | `#warn:` annotation on `status:stable` entry (contradiction) | §10.4    | `warning`       |
| `WRN-015` | Recovery applied for case-incorrect reserved identifier     | §3.4      | `warning`       |

Codes `WRN-016` through `WRN-999` are unassigned and available for future additions.

### 19.7 Diagnostic Shape

Every reported violation MUST be emitted by a conformant validator as a diagnostic with
the following minimum structure:

1. **Code.** The full error or warning code (e.g., `SYN-042`, `CON-021`).
2. **Severity.** The effective severity for the current mode (`error` or `warning`).
3. **Title.** The short title as listed in §19.3–§19.6.
4. **Source location.** At minimum, the line number (1-indexed). Column number SHOULD be
   provided when practical. For corpus-scope violations, the document identifier
   (filename or URI) MUST be provided in addition to the line.
5. **Message.** A human-readable description of the specific violation, ideally citing
   the offending token or value.
6. **Specification reference.** The section number of the rule that was violated (as
   listed in the catalog entry's "Section" column).

A validator MAY emit additional fields (hints, fix suggestions, related locations, code
snippets), but the six above are the mandatory core.

Output encoding is not prescribed by this specification. A validator MAY emit diagnostics
as plain text, JSON, SARIF, compiler-style messages (`file:line:col: severity: code:
message`), or any other format. Interoperability between tools MAY require a common
format, but the choice of format is a tooling concern, not a spec concern.

### 19.8 Error Recovery

Error recovery is governed by §1.5 (lenient consumption). This section restates the key
principles in the context of the error catalog:

1. **Hard syntactic errors** (those marked `error (hard)` in §19.3) MAY cause a consumer
   to abort parsing. A consumer is not obligated to continue past a hard error.
2. **Non-hard errors and warnings** SHOULD NOT cause a consumer to abort. The consumer
   reports the violation and continues. This is the graceful-consumption rule of §1.5.
3. **Recovery behavior is not part of the language.** A consumer that recovers from a
   violation in a specific way does not thereby legitimize the violation, and another
   consumer MAY handle the same input differently. Producers MUST NOT rely on specific
   consumer recovery behavior for interoperability (§18.3, §18.4).
4. **Recovery actions themselves MAY be reported as warnings** (e.g., `WRN-001` reports
   that a BOM was stripped; `WRN-015` reports that a case-incorrect reserved identifier
   was recovered). These warnings signal to downstream consumers that the document was
   not strictly conformant even if the validator chose to proceed.

5. **Recovery warnings do not replace the underlying error.** When a consumer recovers
   from a violation in structural or core mode, it MUST report the underlying error
   (e.g., `SYN-010` for a case-incorrect reserved identifier) and MAY additionally emit
   the corresponding recovery warning (`WRN-015`) to document the recovery action. The
   warning is an annotation on the error, not a substitute for it. Emitting `WRN-015`
   alone without `SYN-010` would contradict the principle of §1.5 that recovery is a
   property of the consumer, not a property of the language.

### 19.9 Extensibility

Profiles and dialects MAY register their own error and warning codes. Such extensions
MUST use distinct family prefixes to avoid colliding with the universal catalog.

#### 19.9.1 Profile Extension Prefixes

A profile MAY introduce error codes under a three-letter prefix that begins with `P`
followed by two letters derived from the profile name. Recommended prefixes for the two
profiles registered in v0.1:

| Profile         | Recommended prefix | Example codes              |
|-----------------|-------------------|-----------------------------|
| `code_surface`  | `PCS`              | `PCS-001`, `PCS-002`, ...   |
| `api_contract`  | `PAC`              | `PAC-001`, `PAC-002`, ...   |

**Note.** In v0.1, all profile-specific violations fit into the universal `CON` family
(e.g., `CON-021` through `CON-033` are `api_contract` and `code_surface` specific).
Dedicated profile prefixes (`PCS`, `PAC`) are reserved for future use: when a profile's
rule set grows beyond what the shared `CON` catalog can accommodate cleanly, the profile
MAY migrate its codes to its own prefix. The recommended prefixes are registered here
to prevent collision with future extensions.

#### 19.9.2 Dialect Extension Prefixes

A dialect MAY introduce error codes under a three-letter prefix that begins with `D`
followed by two letters derived from the dialect name. Examples of recommended
prefixes for hypothetical future dialects:

| Dialect (hypothetical)     | Recommended prefix | Example codes         |
|----------------------------|-------------------|------------------------|
| `platform_metamodel`       | `DPM`              | `DPM-001`, `DPM-002`  |
| `rust_extensions`          | `DRE`              | `DRE-001`, `DRE-002`  |

No dialects are registered in v0.1; dialect prefixes are reserved for future use.

#### 19.9.3 Prefix Collision Rules

1. Profile and dialect prefixes MUST NOT collide with universal prefixes (`SYN`, `SEM`,
   `CON`, `WRN`) or with each other.
2. A profile or dialect registering a prefix MUST document the prefix in its addendum.
3. Two profiles or dialects MUST NOT register the same prefix. The profile/dialect
   registry (§17) is responsible for coordinating prefix assignments.
4. Profile- or dialect-registered codes follow the same stability and append-only rules
   as universal codes (§19.1.3).

### 19.10 Open Questions for Future Revisions

- Whether a **standardized diagnostic output format** (e.g., SARIF profile, structured
  JSON schema) should be normatively specified for tool interoperability.
- Whether **fix hints** and **auto-fix actions** should become part of the diagnostic
  shape, with a normative hint grammar.
- Whether the **recovery actions** that currently produce warnings (WRN-001, WRN-002,
  WRN-003, WRN-015) should be configurable by consumer (suppressible) or always
  reported.
- Whether a **severity override mechanism** for specific codes (e.g., downgrading a
  `warning` to `info` or upgrading a specific `warning` to `error`) should be
  standardized or left to tooling.
- Whether the distinction between `CON` (universal conformance) and profile-specific
  prefixes (`PCS`, `PAC`) should be revisited once profile rule sets mature.

---

## 20. Validator Requirements

This section specifies the normative obligations of conformant validators (§18.5): what
they MUST report, what they MUST reject, what they SHOULD tolerate, the shape of their
diagnostic output, minimum recovery behavior, corpus-specific requirements, and the
minimum self-declaration a validator must provide to its users.

§20 is the operational complement to §18 and §19. Where §18 defines **what conformance
means** and §19 defines **which codes report violations**, §20 defines **how a validator
behaves** when it encounters those violations.

This section is deliberately conservative. It does not prescribe an API, a CLI surface,
an output format, a performance envelope, or a tooling protocol. Those concerns are
enumerated in §20.9 as out of scope for v0.1.

### 20.1 Validator Roles and Modes

A validator is a consumer (§18.4) that additionally implements the validator obligations
of §18.5 and this section. Every conformant validator operates in exactly one of the
four validation modes defined in §1.3.

| Mode         | Scope                                                  | Profile rules | Corpus checks | Ref.    |
|--------------|--------------------------------------------------------|---------------|---------------|---------|
| `structural` | Physical file and grammar checks only                  | No            | No            | §1.3    |
| `core`       | Core conformance (§1.2.1)                              | No            | No            | §1.3    |
| `profile`    | Core + declared profile rules                          | Yes           | No            | §1.3    |
| `corpus`     | Profile rules + cross-document invariants              | Yes           | Yes           | §1.3    |

A validator MAY support multiple modes and select among them per invocation. A validator
MUST declare which modes it supports (§20.8) and MUST document how a specific mode is
selected (command-line flag, configuration, or default).

A validator operating in a specific mode MUST apply the rules of that mode and MUST NOT
apply rules of other modes. A `core` mode validator encountering a `profile=` header
does not apply profile rules; it notes the declaration (and MAY emit `CON-001` if the
profile name is unknown) but validates against core conformance only.

### 20.2 Minimum Reporting Requirements

A validator MUST report violations using codes from §19. This section defines which
code families apply per mode and which codes are mandatory to report.

**General rule.** A validator MUST report every violation it detects that is in scope
for its current mode. "In scope" is defined per mode below. A validator MUST NOT silently
suppress in-scope violations; suppression is a conformance breach of §18.5.

#### 20.2.1 Structural Mode

Applicable code families: **`SYN` (syntactic)** only.

| Obligation        | Scope                                                         |
|-------------------|---------------------------------------------------------------|
| MUST report       | All `SYN` codes the validator detects.                        |
| MAY report        | `WRN` codes related to physical-file normalization (§4.6): `WRN-001` (BOM stripped), `WRN-002` (`CRLF` normalized), `WRN-003` (trailing spaces stripped), `WRN-004` (one trailing blank line). |
| MUST NOT report   | `SEM`, `CON` codes. Semantic and conformance rules are out of scope for structural mode. |
| Out of scope      | Identity, graph, vocabulary, profile, corpus checks.          |

A structural-mode validator is useful for tooling that needs to confirm a file is
parsable JOOT without investing in semantic verification (editors, linters, pre-commit
hooks).

#### 20.2.2 Core Mode

Applicable code families: **`SYN`, `SEM`, `WRN`**. `CON` codes remain in the `CON`
family but apply only in the limited subset that affects core conformance: in core mode,
unknown profile or dialect declarations do not prevent core validation; if reported,
they are reported as warnings rather than conformance errors. The codes themselves
(`CON-001`, `CON-002`) retain their `CON` family identity; only their effective severity
in core mode is reduced to `warning` per §19.5.

| Obligation        | Scope                                                         |
|-------------------|---------------------------------------------------------------|
| MUST report       | All `SYN` codes; all `SEM` codes with severity `error` in core mode (from §19.4); `CON-001` and `CON-002` at warning severity if the document declares an unknown profile/dialect. |
| MUST report       | `WRN` codes for recovery actions applied (§19.8) and for quality warnings (`WRN-005`, `WRN-006`, `WRN-007`, etc.) when detected. |
| MAY report        | `SEM` codes with `warning` severity in core mode (unresolved targets, non-NFC content, unknown universals). |
| MUST NOT report as errors | `CON` codes that apply only to profile or corpus conformance (`CON-003` through `CON-036` except `CON-001`/`CON-002`). |
| Out of scope      | Profile closure enforcement, corpus-level checks, profile-specific conformance checks (§15.3.11, §15.4.13). |

A core-mode validator verifies that a document is syntactically well-formed and
semantically coherent under core rules, without applying profile-specific discipline.

#### 20.2.3 Profile Mode

Applicable code families: **`SYN`, `SEM`, `CON`, `WRN`**, plus any profile-registered
extension prefixes (§19.9.1) for the declared profile.

| Obligation        | Scope                                                         |
|-------------------|---------------------------------------------------------------|
| MUST report       | All `SYN` codes; all `SEM` codes with severity `error` in profile mode; all `CON` codes applicable to profile conformance with severity `error` in profile mode. |
| MUST report       | The minimum conformance checks of the declared profile (§15.3.11 for `code_surface`, §15.4.13 for `api_contract`), each mapped to the corresponding `CON` code. |
| MUST report       | Non-canonical ordering as errors (`CON-006`, `CON-007`, `CON-008`), not warnings, per §18.6. |
| MUST report       | All relevant `WRN` codes for recovery actions and quality concerns. |
| MAY report        | `SEM` and `CON` codes with `warning` severity in profile mode. |
| Out of scope      | Cross-document invariants (those are corpus mode, §20.2.4).   |

A profile-mode validator enforces the declared profile's complete rule set on a single
document. This is the mode most production tooling operates in when working with
profile-conformant corpora one document at a time.

#### 20.2.4 Corpus Mode

Applicable code families: all (`SYN`, `SEM`, `CON`, `WRN`, plus profile/dialect
extensions).

| Obligation        | Scope                                                         |
|-------------------|---------------------------------------------------------------|
| MUST report       | Everything a profile-mode validator reports, for every document in the corpus. |
| MUST report       | Corpus-scope `CON` codes (`CON-034`, `CON-035`, `CON-036`) when detected across documents. |
| MUST report       | `SEM-013` and `SEM-014` at `error` severity for unresolved internal targets, applying the corpus as the resolution scope. |
| MUST report       | Cross-document `(id, facet)` uniqueness violations (`CON-034`). |
| MUST report       | `cat:`/`kind:` preservation violations across facets in different documents (`CON-035`). |
| MUST report       | Profile-internal convention disagreements across documents of the same profile (`CON-036`). |
| Out of scope      | Nothing from §19 is out of scope in corpus mode. Corpus mode is the strictest.  |

A corpus-mode validator requires access to the complete document set (§18.7). A
validator that cannot load the full corpus MUST NOT claim to validate corpus conformance.

### 20.3 Rejection Requirements

A validator **rejects** a document (or corpus) when it declares the document non-
conformant at the target layer. Rejection is a normative outcome of validation; it is
not the same as "aborting" (which is a consumer behavior described in §20.6).

A validator MUST reject the document when:

1. **Any `error`-severity violation applicable to the current mode is detected.** A single
   in-scope error is sufficient for rejection. Conformance is binary per layer (§18.2).
2. **A profile is declared but the profile name is unknown**, when operating in profile or
   corpus mode (`CON-001`, §19.5). Unknown profile in structural or core mode is a
   warning, not a rejection.
3. **A dialect is declared but is not whitelisted by the active profile**, when operating
   in profile or corpus mode (`CON-003`). Unknown dialect in structural or core mode is
   a warning.
4. **Corpus-specific invariants are violated**, when operating in corpus mode (`CON-034`
   through `CON-036`).

A validator MUST NOT reject the document for:

1. **Warnings alone.** A document with only `warning`-severity violations is conformant
   at the target layer. The validator reports the warnings and returns success.
2. **Violations outside the current mode's scope.** A structural-mode validator MUST NOT
   reject a document for `SEM` or `CON` violations; those are out of scope for structural
   mode.
3. **Mode mismatches.** A core-mode validator encountering a document with `profile=`
   declared does not reject the document; it validates against core rules and optionally
   notes the unrecognized profile declaration as `CON-001` (warning).

When a document is rejected, the validator MUST:

- Return a failure status (the mechanism is validator-specific, out of scope for v0.1).
- Report all in-scope violations, not only the first one detected (to the extent
  practical; see §20.4 on practical limits).
- Indicate which violations caused the rejection (i.e., which reports were `error`
  severity).

### 20.4 Tolerance Requirements

A validator MUST tolerate input that is conformant, non-canonical but recoverable, and
out-of-scope violations not applicable to its mode. "Tolerate" here means: continue
reading, apply the appropriate severity, and report without aborting.

#### 20.4.1 What a Validator SHOULD Tolerate

A validator SHOULD continue past and report (not abort on):

1. **All non-hard errors.** Hard errors (§19.2.3) MAY abort parsing; all other errors
   SHOULD NOT.
2. **Recoverable physical-file issues.** BOM, CRLF line endings, trailing spaces — these
   are normalized per §4.6 and reported via `WRN-001` through `WRN-003`.
3. **Non-canonical ordering.** Canonical ordering violations at core conformance are
   warnings and SHOULD NOT cause abort. At profile conformance they are errors but still
   SHOULD NOT cause abort unless the validator explicitly documents otherwise.
4. **Unknown vocabulary at core conformance.** Unknown `cat:`, `kind:`, role, atom, or
   descriptor values are warnings in core mode.
5. **Unresolved internal relation targets at document scope.** These are warnings in
   core and profile mode (errors only in corpus mode).

#### 20.4.2 What a Validator SHOULD NOT Tolerate

A validator SHOULD NOT continue past:

1. **Hard syntactic errors.** Invalid UTF-8, unclosed quoted strings, and similar errors
   prevent meaningful further parsing (§19.2.3). A validator MAY abort; continuing is
   permitted but not required.
2. **Truncated input.** A document that ends mid-entry or mid-header is malformed; the
   validator reports the truncation and MAY abort.

#### 20.4.3 Report Limits

A validator MAY impose a limit on the number of violations reported per document to
avoid pathological output on heavily malformed input. If a limit is imposed, the validator
MUST:

- Document the limit and its rationale in its self-declaration (§20.8).
- When the limit is reached, emit a final diagnostic indicating that further violations
  exist and were not reported.
- Make the limit configurable by the user when practical.

Default recommendation: no hard limit at v0.1. Implementations typically report all
violations unless output size becomes operationally problematic.

#### 20.4.4 Recovery Is Not Tolerance

Tolerance means "continue reading and report"; **recovery** means "attempt to interpret
malformed input as if it were correctly formed". A validator MAY recover (and MUST
report the recovery per §19.8 and §20.6.5), but tolerance does not imply recovery. A
validator MAY tolerate a violation by reporting it without attempting recovery — the
violation is still reported, but the downstream interpretation of the malformed content
may be degraded or skipped. See §20.6 for recovery obligations.

### 20.5 Diagnostic Output Shape

Every violation a validator reports MUST be emitted as a diagnostic with the minimum
fields of §19.7, repeated here for normative emphasis:

1. **Code** — the full error or warning code (e.g., `SYN-042`, `CON-021`).
2. **Severity** — the effective severity for the current mode (`error` or `warning`).
3. **Title** — the short title as listed in §19.3–§19.6.
4. **Source location** — line number (1-indexed) at minimum; column number SHOULD be
   provided; document identifier (filename or URI) MUST be provided in corpus mode.
5. **Message** — a human-readable description citing the offending token or value.
6. **Specification reference** — the section number of the rule violated.

A validator MAY emit additional fields (hints, fix suggestions, related locations, code
snippets). The six above are mandatory.

#### 20.5.1 Output Format Is Not Prescribed

The specification does not prescribe an output format. A validator MAY emit diagnostics
as:

- plain text in any layout,
- compiler-style `file:line:col: severity: code: message`,
- JSON (with any schema),
- SARIF,
- any structured or unstructured format the validator's audience requires.

Interoperability between validators is a tooling concern. The specification's obligation
ends with the six required fields.

#### 20.5.2 Source Location in Corpus Mode

In corpus mode, every diagnostic MUST include a document identifier (filename, URL, or
opaque corpus-scoped handle) in addition to the line number. Without a document
identifier, a corpus-mode diagnostic is ambiguous about which document it references.

The specific format of the document identifier is out of scope. A validator SHOULD use
filenames or URIs when the corpus was loaded from a filesystem or URL-addressable
source; for in-memory or synthesized corpora, an opaque stable identifier is sufficient.

#### 20.5.3 Ordering of Diagnostics

Validators SHOULD emit diagnostics in source-order, and when multiple diagnostics attach
to the same source location, in code-order (alphanumeric by the full code string, e.g.,
`SYN-010` before `SYN-042` before `CON-006`). Source-order means earlier lines first,
then earlier columns.

This two-level ordering is a readability and diff-stability concern, not a conformance
requirement: a validator that emits diagnostics in a different order (e.g., grouped by
severity, grouped by code family, grouped by document) remains conformant. The
recommendation exists because source-then-code order produces stable, diffable output
that is substantially easier to consume in CI logs, editor panels, and code review
contexts.

When multiple diagnostics target the same source location, the relative order among
them SHOULD be stable across runs on identical input.

### 20.6 Recovery Behavior Requirements

Recovery is the validator's attempt to interpret malformed input as if it were
conformant. Recovery is permitted but not required (§1.5); this section defines the
minimum obligations when a validator chooses to recover.

#### 20.6.1 Recovery Is Optional

A validator MAY implement recovery for specific violations (typically those enumerated
as recoverable in §4.6 and §3.4). A validator that implements no recovery and simply
reports all violations is conformant.

#### 20.6.2 Recovery Actions Listed

The following recovery actions are permitted by this specification:

| Recovery action                                  | Triggering code | Recovery warning | Spec ref |
|--------------------------------------------------|-----------------|------------------|----------|
| Strip BOM on read                                | `SYN-002`       | `WRN-001`        | §2.4, §4.6 |
| Normalize `CRLF` to `LF`                         | n/a (accepted)  | `WRN-002`        | §2.5, §4.6 |
| Strip trailing spaces on read                    | n/a (accepted)  | `WRN-003`        | §4.6     |
| Accept case-incorrect reserved identifier        | `SYN-010`       | `WRN-015`        | §3.4     |

No other recovery actions are permitted in v0.1. A validator MUST NOT silently recover
from violations not listed above.

#### 20.6.3 Recovery Does Not Replace the Underlying Report

When a validator recovers from a violation, it MUST:

1. Report the underlying error code (e.g., `SYN-010` for a case-incorrect identifier) at
   the severity the catalog specifies for the current mode.
2. MAY additionally emit the corresponding recovery warning (e.g., `WRN-015`) to
   document the recovery action.

Emitting only the warning without the error is a conformance breach. The warning is an
annotation on the error; it does not replace it. This rule is stated normatively in
§19.8 point 5 and restated here because it is central to validator behavior.

#### 20.6.4 Recovery Actions Are Scoped

A validator MAY apply recovery for some actions and not others. A validator's self-
declaration (§20.8) MUST list which recovery actions it implements, so consumers of its
output can interpret the absence of a recovery warning correctly.

#### 20.6.5 Recovery Is a Validator Property, Not a Language Property

The specification's normative content does not depend on recovery. A producer MUST NOT
emit documents that rely on validator recovery for correct interpretation (§18.3). A
consumer MUST NOT assume that another consumer will apply the same recovery actions
(§1.5). Recovery is local to a specific validator and is not portable.

### 20.7 Multi-Document and Corpus Validators

A validator operating in corpus mode faces requirements that do not apply in the other
modes. This section consolidates them.

#### 20.7.1 Corpus Access

A corpus-mode validator MUST have **simultaneous access** to all documents in the
corpus. A validator that processes documents one-at-a-time without cross-document
awareness cannot check corpus-scope invariants and MUST NOT claim corpus mode.

The mechanism by which the validator accesses the corpus (filesystem scan,
configuration-specified list, on-demand loading) is out of scope.

#### 20.7.2 Corpus Boundaries

Corpus boundaries are declared out-of-band (§12.5). The validator MUST be told which
documents constitute the corpus before validation begins. A validator MUST NOT
heuristically guess corpus boundaries.

#### 20.7.3 Cross-Document Identity

The validator MUST enforce `(id, facet)` uniqueness across the entire corpus, not only
within individual documents. Two documents declaring the same `(id, facet)` tuple is a
`CON-034` error regardless of which documents are considered.

#### 20.7.4 Cross-Document Target Resolution

The validator MUST attempt to resolve every internal relation target against the complete
corpus before declaring the target unresolved. An internal target that resolves in a
different document than the one declaring the edge is a successful resolution; only
targets that cannot be matched anywhere in the corpus are `SEM-014` errors.

#### 20.7.5 Cross-Document Consistency

The validator MUST check that all facets of the same `id:` declare identical `cat:` and
`kind:`, even when facets reside in different documents (§12.3.4, `CON-035`). The
validator MUST also check that documents declaring the same profile agree on profile-
internal conventions (`CON-036`, see §15.4.5 for the example of linked-contract vs
endpoint-facet consistency).

#### 20.7.6 Incremental Validation

Incremental corpus validation — validating the corpus after adding, removing, or
modifying a single document without re-validating the whole set — is **out of scope for
v0.1**. A corpus validator MAY support incremental operation, but its correctness is
tooling-specific and not specified here.

### 20.8 Conformance Self-Declaration

A validator claiming conformance MUST provide a self-declaration stating the exact scope
of its implementation. Self-declaration is how consumers of the validator's output know
what to expect.

#### 20.8.1 Required Fields

A conformant validator's self-declaration MUST specify, at minimum:

1. **Supported validation modes.** Which of `structural`, `core`, `profile`, `corpus`
   the validator implements. At least one mode MUST be supported.
2. **JOOT revision implemented.** The specific revision of this specification the
   validator targets (e.g., "JOOT v0.1").
3. **Supported profiles.** For profile-mode and corpus-mode support, the list of
   registered profiles whose addenda the validator implements. A validator supporting
   profile mode but implementing no specific profile can only validate documents
   declaring no profile or declaring an unrecognized profile.
4. **Corpus-mode support.** Explicit statement of whether the validator supports corpus
   mode. If yes, whether cross-document target resolution and cross-document
   `(id, facet)` uniqueness are implemented.
5. **Recovery actions implemented.** Which recovery actions from §20.6.2 the validator
   applies. Validators that apply no recovery state so explicitly.
6. **Diagnostic output.** Whether all six required fields of §19.7 / §20.5 are emitted
   for every diagnostic, and the output format(s) supported (plain text, JSON, SARIF,
   etc.).

#### 20.8.2 Optional Fields

A self-declaration MAY additionally specify:

- Supported dialects.
- Report limits (§20.4.3).
- Incremental corpus-mode support (§20.7.6).
- Performance characteristics.
- Integration points (CI/CD, editor, language server protocol).
- Extension error codes implemented (§19.9).

#### 20.8.3 Vague Claims Are Not Conformant Self-Declarations

A validator that claims "JOOT conformance" without specifying the fields of §20.8.1 is
not providing a conformant self-declaration. Consumers of such a tool cannot rely on
what it verifies. A tool author MUST provide the minimum fields; a self-declaration
without them is insufficient regardless of how strong the tool's implementation actually
is.

#### 20.8.4 Self-Declaration Location

The location of a validator's self-declaration is not prescribed. Common locations
include: tool documentation, a `--version` or `--conformance` command-line output, a
README, a project website. The specification does not prescribe; it requires only that
the declaration exist and be discoverable by users.

### 20.9 Out of Scope in v0.1

The following concerns are deliberately **out of scope** for this specification in v0.1.
A validator MAY address any of them in its own documentation; none is a conformance
requirement.

- **API shape.** Function signatures, return types, data structures of a validator's
  programmatic interface.
- **Command-line interface.** Standard flags, arguments, option names.
- **Runtime protocol.** LSP integration, streaming validation protocols, incremental
  update negotiation.
- **Performance requirements.** Throughput, latency, memory bounds.
- **Concurrency guarantees.** Thread safety, parallelism behavior.
- **Output format standardization.** While §20.5 lists formats validators MAY emit,
  none is standardized as the normative format.
- **Fix-application semantics.** A validator that emits fix hints is not obligated to
  apply them; auto-fix tooling is out of scope.
- **Error catalog internationalization.** Diagnostic `title` and `message` localization.
- **Registry retrieval protocols.** §17.6 mechanisms are declarative; operational
  retrieval is unspecified.
- **Signing and attestation** of validator self-declarations or tool binaries.

These concerns may be addressed in future revisions (§20.10) or by informative companion
documents.

### 20.10 Open Questions for Future Revisions

- Whether a **standardized diagnostic output format** (likely SARIF or a JOOT-specific
  JSON schema) should be normatively specified to enable tool interoperability.
- Whether a **standard CLI surface** for validators should be specified (e.g., exit
  codes, `--mode`, `--profile` flags) for cross-tool consistency.
- Whether **incremental corpus validation** (§20.7.6) should be formalized as a
  conformance angle with its own requirements.
- Whether **recovery action configuration** (enabling or disabling specific recoveries)
  should be a mandated validator feature rather than an implementation choice.
- Whether a **validator test suite** — a set of JOOT documents with expected diagnostic
  output — should be published alongside the specification to enable conformance
  verification of validators themselves.
- Whether a **severity override mechanism** should be standardized (user downgrading
  specific warnings or upgrading specific warnings to errors).
- Whether **fix-hint grammar** should be formalized, enabling interoperable auto-fix
  tooling across validators.

---

# Part VI — Operational

Parts I–V defined JOOT as a format, a semantic model, a governance framework, and a
conformance system. Part VI addresses the operational concerns that surround those
layers: how versions evolve (§21), security (§22), privacy (§23), change control
(§24), and the conformance test suite (§25).

Part VI is normative but deliberately less prescriptive than Parts I–V. Operational
concerns vary more across deployments than the format itself, so the specification
establishes principles, obligations, and open questions rather than fixed algorithms.

---

## 21. Versioning and Compatibility Policy

This section defines the versioning scheme for JOOT, the compatibility guarantees the
specification provides across revisions, and the obligations of producers and consumers
with respect to version changes. It is the foundation for §22–§25: security, privacy,
change control, and test-suite evolution all depend on a stable versioning framework.

### 21.1 Versioning Scheme

JOOT follows a **major.minor** versioning scheme. Every revision of this specification
carries a version identifier of the form:

```
v<MAJOR>.<MINOR>
```

Where `<MAJOR>` and `<MINOR>` are non-negative decimal integers. The minor component
MAY be omitted in casual reference (`v1` ≡ `v1.0`); in the document header (§4.2.2),
the minor MAY be omitted and is treated as 0.

#### 21.1.1 Major Version

A **major version change** (increment of `<MAJOR>`) indicates a revision that breaks
backward compatibility. A document valid under version `v1.x` is NOT guaranteed to be
valid under version `v2.0`.

Major changes are rare and deliberate. They are triggered only by structural evolutions
that cannot be absorbed through appending (§21.8).

#### 21.1.2 Minor Version

A **minor version change** (increment of `<MINOR>` with `<MAJOR>` unchanged) indicates a
revision that is backward-compatible within its major family. A document valid under
`v1.x` is valid under any `v1.y` where `y >= x`.

Minor revisions MAY:

- Add universal registry entries (§14.4.2).
- Add profiles and dialects (§17).
- Add error codes (§19.1.3).
- Clarify ambiguities in prose.
- Add open-ended features that do not alter existing behavior.

Minor revisions MUST NOT:

- Remove or redefine existing registry entries.
- Remove or redefine error codes.
- Alter the grammar of existing constructs.
- Invalidate documents that conformed under an earlier minor version of the same major.

#### 21.1.3 This Specification

This specification is **JOOT v0.1**. Version 0.x is the Draft phase of the format; no
backward-compatibility guarantees apply **between** 0.x revisions until v1.0. From v1.0
onward, §21.1.1–§21.1.2 rules apply strictly.

### 21.2 Compatibility Guarantees

Compatibility guarantees are organized along three axes: syntax, semantics, and
registry. Each axis has its own stability contract.

#### 21.2.1 Syntax Compatibility

A minor revision MUST NOT alter the grammar of any existing construct. Specifically:

1. Sigils (§5.1) are append-only.
2. Named block markers (`[Rel]`, `[Fields]`, `[Behavior]`, `[Returns]`) are fixed for a
   major version.
3. Canonical meta key order positions 1–14 (§6.3.2) MUST NOT be reordered or redefined.
4. Reserved characters (§9.1) and escape alphabet (§9.3) are closed.
5. Identifier grammars (§11) are closed for a major version.
6. Header layout (§4.2) is closed for a major version.

A major revision MAY alter any of the above. A producer emitting for a specific major
version targets that major's grammar; syntax that is new in `v2.0` is not available in
`v1.x`.

#### 21.2.2 Semantic Compatibility

A minor revision MUST preserve the meaning of every construct defined by an earlier
minor in the same major. Specifically:

1. Identity model (§12) MUST NOT change meaning.
2. Graph model (§13) MUST NOT change meaning.
3. Facet decomposition rules (§12.3) MUST NOT change meaning.
4. Relation role semantics (§14.2) MUST NOT change.
5. Behavior atom semantics (§14.3) MUST NOT change.
6. Type equivalence rules (§7.8) MUST NOT change.

A minor revision MAY:

- Add new relation roles, atoms, types with append-only semantics.
- Clarify ambiguous cases through prose without changing rule meaning.
- Resolve open questions in ways that do not affect previously conformant documents.

A major revision MAY alter any semantic rule, subject to the breaking-changes policy of
§21.8.

#### 21.2.3 Registry Compatibility

Registry compatibility is governed by the append-only rules of §14.4.2 and the lifecycle
rules of §17.5:

1. Universal registry entries are append-only across minor revisions.
2. Profile-registered entries of `Candidate` or `Stable` profiles are append-only.
3. Dialect-registered entries of `Candidate` or `Stable` dialects are append-only.
4. Error codes (§19) are append-only.
5. Reserved code numbers (§17.4.5) remain reserved across minor revisions.
6. Tombstoned codes and deprecated entries (§19.1.4) retain their identities across
   minor revisions.

A major revision MAY retire or redefine registry entries, subject to the breaking-
changes policy of §21.8.

### 21.3 Document Version Declaration

Every JOOT document declares its version in header line 1 per §4.2.2. The version
token is of the form `v<MAJOR>[.<MINOR>]`.

Rules:

1. Every document MUST declare a version.
2. The declared version identifies the major and minor of the specification the document
   was authored against.
3. A consumer MUST reject a document whose declared major version it does not support
   (§4.2.2).
4. A consumer MUST accept a document whose declared minor version is lower than the
   consumer's implemented minor, within the same major.
5. A consumer MAY accept a document whose declared minor version is higher than the
   consumer's implemented minor; behavior is specified in §21.4.3.

### 21.4 Consumer Compatibility Obligations

A conformant consumer (§18.4) MUST handle version variations per the following rules.

#### 21.4.1 Unsupported Major Version

When a consumer encounters a declared major version it does not support:

1. The consumer MUST reject the document.
2. The consumer SHOULD emit a diagnostic indicating the declared major and the
   majors the consumer supports.
3. The consumer MUST NOT attempt to parse the document under an incompatible major.

#### 21.4.2 Supported Major, Lower or Equal Minor

When a consumer encounters a declared major it supports and a minor `m_declared <=
m_implemented`:

1. The consumer MUST parse and validate the document as if it were authored against
   `m_implemented`.
2. The consumer MUST NOT report diagnostics that are only applicable to minor versions
   higher than `m_declared`.
3. Append-only additions in minor versions between `m_declared` and `m_implemented` do
   not apply retroactively; a document declaring `v1.2` is not held to rules introduced
   in `v1.3`.

#### 21.4.3 Supported Major, Higher Minor

When a consumer encounters a declared major it supports but a minor `m_declared >
m_implemented`:

1. The consumer MAY parse the document on a best-effort basis.
2. The consumer MUST emit a diagnostic warning that the declared minor exceeds the
   implemented minor.
3. Constructs introduced in minor versions between `m_implemented` and `m_declared` MAY
   produce unknown-vocabulary or unknown-construct diagnostics; these are not
   conformance errors of the document (the document is authored against a later
   specification), but they do indicate the consumer's limitations.
4. The consumer MUST NOT declare the document conformant without qualification; any
   conformance claim MUST state the minor version against which validation occurred.

#### 21.4.4 Version Negotiation

JOOT v0.1 does not specify a version-negotiation protocol between producers and
consumers. Producers declare a version; consumers accept, reject, or process on best
effort per §21.4.1–§21.4.3. Interactive negotiation is out of scope (§21.9).

### 21.5 Producer Compatibility Obligations

A conformant producer (§18.3) MUST declare the version its output targets and MUST
produce output valid under that version.

#### 21.5.1 Version Declaration

1. Every emitted document MUST carry a version declaration in header line 1 per §4.2.2.
2. The declared version MUST match the specification revision against which the producer
   was implemented, or a lower revision the producer emits strictly to (e.g., a producer
   implemented against `v1.3` MAY emit `v1.0`-targeted documents if it confines itself
   to `v1.0` features).

#### 21.5.2 No Use of Newer Features in Older-Declared Documents

A producer emitting a document declaring version `v1.x` MUST NOT include constructs that
were introduced in a later minor of the same major. Specifically, a producer emitting
`v1.0` MUST restrict itself to constructs defined in `v1.0`, even if the producer is
implemented against `v1.3`.

Violation of this rule produces documents that fail §21.4.2 on consumers implementing
the declared version.

#### 21.5.3 Version Downgrade Is Not Automatic

A producer MUST NOT "downgrade" a document declaring a higher minor version by
rewriting it to a lower declared minor unless the producer guarantees that the
downgraded document is constructively valid at the lower minor. Mechanical downgrade
without structural re-validation is forbidden.

### 21.6 Profile and Dialect Versioning

Profiles and dialects each have their own version (§15.1.1, §16.5.1). These are
distinct from the document's JOOT version.

#### 21.6.1 In v0.1, Profile/Dialect Versions Are Not Carried in the Document

The document header (§4.2.1) declares the JOOT version and MAY declare a profile and/or
a dialect **by name only**. The profile's or dialect's version is **not** part of the
document's header syntax in v0.1.

Consumers and validators apply profile/dialect rules according to their own implemented
versions (§15.5.4, §16.4). Two consumers implementing different versions of the same
profile MAY reach different conformance verdicts on the same document; this is a known
property of v0.1 and is documented in §15.5.4 and §16.8.

#### 21.6.2 Profile/Dialect Version Negotiation Is Out of Scope for v0.1

The syntax `profile=<n>@<version>` is reserved for a future revision (§15.5.4, §16.11).
v0.1 does not permit, and does not validate, version qualifiers on profile or dialect
declarations. A `@<version>` appearing in a `profile=` or `dialect=` token is a syntax
error under `SYN-015` or `SYN-017` respectively.

#### 21.6.3 Append-Only Applies to Profile and Dialect Versions

Profile and dialect version evolution follows the append-only discipline of §17.5.4:

1. `Candidate` and `Stable` profiles MAY add entries in subsequent versions but MUST
   NOT remove or redefine them.
2. The same rule applies to `Candidate` and `Stable` dialects.
3. The document's JOOT version is unrelated to the profile's or dialect's version;
   JOOT-version append-only (§21.2) and profile/dialect-version append-only (§17.5.4)
   are independent stability contracts.

### 21.7 Deprecation and Retirement Timeline

Deprecation and retirement are first-class states in the registry lifecycle (§17.5,
§19.1.4). This subsection specifies the timing rules.

#### 21.7.1 Minimum Deprecation Window

When an entry transitions from `Stable` to `Deprecated`:

1. The entry MUST remain in the registry with status `Deprecated` for at least one
   minor revision cycle of the specification before transitioning to `Retired`.
2. During the deprecation window, the entry's semantics MUST NOT change.
3. Producers SHOULD migrate to the replacement entry (when one exists) before the
   deprecation window ends.
4. Consumers MUST continue to recognize the entry for the duration of the deprecation
   window.

#### 21.7.2 Retirement

When an entry transitions from `Deprecated` to `Retired`:

1. The entry is removed from the active catalog of the current specification revision.
2. Its name (for registry entries) or its code number (for error codes) MUST NOT be
   reused in the same major version.
3. Historical revisions of the specification retain the entry's definition for
   archival purposes.
4. Documents authored against earlier revisions that reference retired entries are not
   thereby invalidated; they remain conformant at their authored revision.

#### 21.7.3 Major Version Boundaries and Retirement

Retired entries MAY be reused in a new major version with different semantics, provided
the major version transition itself is a breaking change (§21.8).

### 21.8 Breaking Changes Policy

A **breaking change** is a modification that invalidates documents previously conformant
at the earlier version. This section specifies when breaking changes are permitted.

#### 21.8.1 Breaking Changes Require a Major Version Increment

A breaking change MUST NOT occur within a minor version. Any breaking change
MUST be introduced in a new major version. Violation of this rule — a minor revision
that silently breaks earlier documents — is a specification defect.

#### 21.8.2 What Constitutes a Breaking Change

The following are breaking changes:

1. Removing a universal registry entry (§14.4.2).
2. Redefining the meaning of a universal registry entry.
3. Reordering or renaming canonical meta keys in positions 1–14 (§6.3.2).
4. Altering the grammar of any core construct (sigils, blocks, meta, quoting, escaping,
   identifiers).
5. Retiring an error code and reusing its number in the same major.
6. Changing the semantics of identity, graph, or facet decomposition.
7. Changing the media type or file extension.
8. Removing support for a declared conformance mode.

The following are NOT breaking changes:

1. Adding a universal registry entry (append-only).
2. Adding an error code.
3. Adding a profile or dialect.
4. Clarifying prose that does not alter rule meaning.
5. Resolving an open question in a way that is consistent with existing conforming
   documents.
6. Deprecating an entry (the entry remains recognized).

#### 21.8.3 Justification for Major Version Increments

A major version increment MUST be justified by a structural evolution that cannot be
accommodated through appending. The specification editors MUST document the
justification in the major revision's release notes.

Frivolous major version increments — those motivated by aesthetic cleanup or
accumulated minor irritations rather than substantive design evolution — SHOULD NOT be
issued. The ecosystem cost of major increments is substantial (consumers must re-
implement, corpora must potentially migrate), and minor revisions SHOULD accommodate
evolution whenever feasible.

#### 21.8.4 Transition Support

When a major version is issued:

1. The previous major version's specification MUST remain published and
   accessible.
2. A migration guide SHOULD accompany the major revision, identifying the breaking
   changes and providing migration paths.
3. Dual-support in tooling (accepting both the old and new major) is
   encouraged but not mandated.

### 21.9 Out of Scope in v0.1

The following versioning-related concerns are deliberately out of scope in v0.1:

- **Profile/dialect version qualifiers** in the document header (reserved for future
  revisions per §15.5.4, §16.11).
- **Interactive version negotiation** between producers and consumers (§21.4.4).
- **Automatic migration tooling** between major versions.
- **Cross-major corpus interoperability** (a corpus mixing documents of different
  majors).
- **Backport policy** for breaking changes discovered after a major release.
- **Signed or attested version claims** in document headers.
- **A canonical change history format** for tracking specification evolution across
  revisions.

These concerns MAY be addressed in future revisions. v0.1 establishes the versioning
framework; operational sophistication is layered on top when the ecosystem requires it.

### 21.10 Open Questions for Future Revisions

- Whether a **profile/dialect version qualifier** in the header (`profile=<n>@<version>`,
  `dialect=<n>@<version>`) should be introduced and, if so, with what grammar and
  precedence rules relative to the JOOT version.
- Whether an **interactive negotiation protocol** (HTTP content negotiation-like)
  should be specified for producer/consumer communication.
- Whether a **migration guide template** should be normatively specified for major
  revision release notes.
- Whether a **canonical change log format** (machine-readable diff between spec
  revisions) should be produced alongside each revision.
- Whether **long-term-support (LTS)** designations for specific minor or major versions
  should be formalized.
- Whether **forward-compatibility probes** (a way for a consumer to query the set of
  minor-version-introduced features without parsing a document) should be specified.

---

## 22. Security Considerations

This section identifies security-relevant aspects of producing, consuming, and
validating JOOT content. JOOT is a static declarative format with no executable
semantics; the security surface is therefore narrower than for programming languages
or runtime protocols. Nonetheless, parsing untrusted input, applying vocabulary
closures, and operating validators in shared environments raise concrete concerns.

### 22.1 Scope

JOOT documents are text-only (§2.4). They do not carry executable code, network
references that a consumer is obligated to dereference, or active content of any kind.
Security concerns therefore focus on:

1. Parser and validator robustness when processing adversarial input.
2. Information leakage through document content, annotations, or diagnostic output.
3. Integrity of registry entries (profile and dialect addenda).
4. Supply-chain concerns in tooling bundled with JOOT support.

### 22.2 Parser and Validator Robustness

A conformant parser or validator MUST:

1. **Bound resource consumption.** A conformant parser or validator MUST process any
   single document without unbounded recursion or exponential expansion induced by
   JOOT syntax. Resource consumption MUST be bounded as a function of input size and
   implementation-defined limits.
2. **Reject malformed input.** Invalid UTF-8, unclosed quoted strings, malformed
   headers, and similar hard errors (§19.2.3) MUST be reported and parsing MAY abort.
3. **Resist pathological input.** Adversarial documents (extremely long lines,
   deeply nested types, large entry counts, maximum-length identifiers) MUST be
   processed without excessive memory or time consumption relative to input size.

**Known attack surfaces that v0.1 mitigates by design:**

- **No executable content.** JOOT has no eval, no include, no macro, no directive that
  changes parser behavior at runtime.
- **No URL dereference.** Relation targets (`ext:<ref>`) are opaque strings; parsers
  and validators MUST NOT fetch them automatically (§22.4).
- **Bounded grammar.** Every construct has a defined shape with a statically bounded
  arity (§7.2.2).
- **ASCII-only identifiers.** Identifier positions exclude Unicode confusables by
  construction (§3.5).

### 22.3 Confusable Characters

Quoted strings and free-text content accept arbitrary UTF-8 (§3.3). Confusable
characters (visually similar Unicode characters from different scripts) MAY appear in
that content. Consumers processing such content for display SHOULD apply UTS #39
recommendations as appropriate. v0.1 does not mandate confusable-detection, but
security-sensitive consumers SHOULD apply it.

Identifier positions are ASCII-only, so confusable-identifier attacks are impossible
by construction in v0.1. A future revision extending identifiers to non-ASCII MUST
introduce explicit confusable mitigation.

### 22.4 Relation Target Resolution

Internal relation targets (§6.4.3) resolve against the active corpus; external
targets (`ext:<ref>`) are opaque. A validator MUST NOT automatically dereference
external targets over the network or by filesystem access based solely on their
textual form. Consumers that choose to resolve `ext:` targets MUST do so with explicit
configuration and MUST apply their own safety checks (URL allow-lists, authentication,
timeouts).

### 22.5 Diagnostic Output as Leakage Vector

A validator's diagnostic output (§19.7, §20.5) includes message text that cites
offending tokens or values from the input document. Consumers of this output (CI
systems, log aggregators, editor panels) MAY expose it to audiences broader than the
original document's intended readership.

Producers SHOULD be aware that:

1. Annotation content (`#note:`, `#warn:`) is included in many error messages.
2. Quoted string values MAY be cited verbatim in diagnostics.
3. `id:` values and field names appear in most diagnostic messages.

Producers handling sensitive data in JOOT documents (e.g., internal schema, proprietary
vocabulary) SHOULD assume that validator output may be captured and stored.

### 22.6 Registry Entry Integrity

Profile and dialect addenda (§15, §16) are consulted by validators to enforce
conformance rules. If an attacker can alter a profile addendum that a validator
consults, they can cause valid documents to appear non-conformant or cause non-
conformant documents to appear valid.

v0.1 does not prescribe a signature or attestation mechanism for addenda (§20.9). In
the absence of such mechanisms, consumers SHOULD:

1. Source addenda from trusted channels (authoritative publications per §17.7).
2. Cache addenda locally after review, rather than re-fetching on each validation.
3. Treat bundled-with-tool addenda as equivalent in trust to the tool itself.

Signature and attestation are identified as open questions in §22.9.

### 22.7 Supply Chain of Tooling

Validators, parsers, and emitters that claim JOOT conformance are software artifacts
subject to the same supply-chain concerns as any other dependency. Consumers of JOOT
tooling SHOULD apply their standard supply-chain hygiene: verified provenance, pinned
versions, reproducible builds, vulnerability disclosure processes. These concerns are
not JOOT-specific and are not further addressed by this specification.

### 22.8 Out of Scope

The following are out of scope for JOOT v0.1 security considerations:

- Access control on JOOT documents (filesystem permissions, HTTP authentication).
- Encryption at rest or in transit.
- Audit logging of JOOT document modifications.
- Signing of JOOT documents or their addenda.
- Runtime enforcement of declared contracts (JOOT is a description format, not a
  runtime system).

### 22.9 Open Questions for Future Revisions

- Whether a **signing mechanism** for JOOT documents and/or registry addenda should be
  specified.
- Whether **consumer profiles** restricting specific features for hardened
  environments (e.g., "JOOT-restricted" with no `ext:` targets) should be formalized.
- Whether a **Unicode identifier extension** (§3.7) should be introduced and, if so,
  with what confusable-detection mandate.

---

## 23. Privacy Considerations

JOOT is a description format for technical surfaces. It is not primarily a format for
personal data. Nonetheless, some privacy considerations arise from its use.

### 23.1 Scope

JOOT documents typically describe:

- Code surfaces (modules, types, members, callables).
- API contracts (endpoints, request/response shapes, error bodies).
- Data schemas (tables, columns, constraints).
- Platform metamodels, workflows, events.

Under normal use, JOOT documents do NOT contain personal data of end users. They
describe the structure around which personal data flows, not the data itself.

However, JOOT documents MAY incidentally contain:

1. Author names, email addresses, or handles in `Source:` header fields (§4.2.1) when
   the source is "authored by <person>".
2. Organization identifiers (e.g., company names, internal namespace prefixes) in `id:`
   values.
3. Natural-language text in annotations (`#note:`, `#warn:`) or quoted strings.

### 23.2 Handling of Incidental Personal Data

Producers and consumers SHOULD treat JOOT documents that contain incidental personal
data according to their applicable privacy regulations (GDPR, CCPA, similar). The
specification does not prescribe handling; it acknowledges that incidental content
exists and that responsibility for its handling lies with the parties controlling the
document.

### 23.3 Diagnostic Output

As noted in §22.5, validator diagnostic output MAY contain verbatim excerpts from
input documents. When documents contain personal data, diagnostic output inherits that
sensitivity. Consumers of diagnostic output SHOULD apply the same privacy disciplines
to logs, CI output, and editor panels as to the source documents.

### 23.4 Registry and Addendum Privacy

Profile and dialect addenda are published artifacts (§17.6). Authors SHOULD NOT
include personal data in addenda; addenda are intended to be broadly discoverable and
are inappropriate for sensitive content.

### 23.5 Corpus-Wide Patterns

A corpus of JOOT documents, taken together, MAY reveal organizational patterns,
vendor relationships, internal naming conventions, and other information that is not
explicitly personal but may be sensitive from a business or operational standpoint.
Organizations publishing JOOT corpora publicly SHOULD apply the same operational-
sensitivity review as they would to any technical documentation release.

### 23.6 Out of Scope

- Pseudonymization or anonymization techniques for JOOT content.
- Retention policies for JOOT documents.
- Data-subject access, erasure, or portability rights as applied to JOOT content.
- Cross-border transfer restrictions for JOOT documents.

These are properly handled by each organization's privacy framework, not by this
specification.

### 23.7 Open Questions for Future Revisions

- Whether JOOT should define a **privacy-sensitive marker** (a meta key or profile
  convention) indicating that a document contains content requiring special handling.
- Whether **annotation redaction** should be specified as a consumer operation for
  contexts where annotations may leak information.

---

## 24. Change Control and Standard Evolution

This section specifies how JOOT as a standard evolves: how proposals are considered,
how revisions are published, who holds editorial authority, and how the community
participates.

### 24.1 Editorial Authority

Editorial authority for this specification is held by the **JOOT editors** in v0.1.
The editors are responsible for:

1. Evaluating proposed changes against the principles of §§P1–P10 and the
   append-only discipline of §14.4.2, §17.5.4, and §21.2.
2. Maintaining the universal registries (§14, §17.2.1).
3. Coordinating profile and dialect registrations (§17).
4. Publishing revisions on a deliberate cadence.
5. Arbitrating disputes about interpretation.

Editorial authority is currently centralized. A future revision MAY formalize a
broader governance model (consortium, standards body, federated authorities — §17.8).

### 24.2 Proposal Process

Proposed changes to JOOT MAY originate from any source: editors, implementers,
profile authors, dialect authors, or the general community. The process for
incorporating a proposal into a revision consists of the following phases:

1. **Proposal.** The change is described in writing: problem statement, proposed
   solution, rationale, examples, and impact analysis. Proposals MAY be submitted to
   the editors through any channel they designate.
2. **Review.** The editors evaluate the proposal against the specification's
   principles and against compatibility rules (§21). Review MAY be public or private
   at the editors' discretion.
3. **Draft.** Accepted proposals are incorporated into a draft revision. The draft is
   published for community review.
4. **Finalization.** After community review, the draft is either accepted as a new
   revision, returned for further iteration, or rejected.
5. **Publication.** The new revision is published with a unique version identifier
   (§21.1) and accompanying release notes.

The duration of each phase is not prescribed. Minor revisions typically complete in
weeks to months; major revisions typically require longer.

### 24.3 Revision Publication

Every published revision MUST include:

1. The full specification text, versioned per §21.1.
2. A release notes document identifying what changed relative to the prior revision.
3. For minor revisions: the list of new universal registry entries, new error codes,
   and new profiles/dialects.
4. For major revisions: a migration guide (§21.8.4) identifying breaking changes and
   migration paths.
5. Updated conformance test cases (§25) when rule changes affect validation.

Release notes MUST identify additions, deprecations, retirements, and supersessions
affecting universal registries, profiles, dialects, and error codes. This obligation
complements §24.6 (deprecation announcements) by requiring that the full lifecycle
history of each revision — what was added, what was deprecated, what was retired, and
what superseded what — be visible in a single authoritative place per revision.

Revisions MUST be published through a stable channel that preserves historical
revisions indefinitely. Deletion of an old revision's publication breaks the
append-only promise of §21.

### 24.4 Community Participation

The specification welcomes community contribution through:

1. **Proposals** for new features, clarifications, or corrections.
2. **Bug reports** identifying inconsistencies, ambiguities, or defects.
3. **Reference implementations** that demonstrate feasibility of proposed features or
   establish conformance test cases.
4. **Profile and dialect authorship** per §15 and §16.

The mechanism for community participation is not prescribed. The editors SHOULD
provide clearly documented contribution channels; the specification itself does not
prescribe them.

### 24.5 Conflict Resolution

When two parties disagree about the interpretation of a specification rule, the
following resolution procedure applies:

1. If the rule's text is unambiguous, the rule's text controls. Parties disagreeing
   with the text MAY submit a proposal to revise it.
2. If the rule's text is ambiguous, the editors issue an interpretation. The
   interpretation is incorporated into the next minor revision as a clarification.
3. If the ambiguity affects active implementations, the editors SHOULD issue the
   interpretation promptly and publish it as an advisory outside the regular revision
   cycle.
4. Interpretations MUST NOT constitute breaking changes. If the intended interpretation
   would break conformant documents, the change is a major revision (§21.8).

### 24.6 Deprecation Announcements

When the editors deprecate a universal registry entry, a profile, a dialect, or an
error code, they MUST:

1. Publish the deprecation announcement at least one minor revision cycle before the
   deprecation takes effect (§21.7.1).
2. Provide a migration path when a replacement exists.
3. Maintain the deprecated entry's functionality throughout the deprecation window.

### 24.7 Out of Scope

- The specific tools, venues, and formats for proposals, bug reports, and community
  discussion.
- Governance models beyond the v0.1 centralized-editors model.
- Legal and licensing aspects of the specification itself.
- Copyright and attribution norms for specification contributors.

### 24.8 Open Questions for Future Revisions

- Whether a **formal governance model** (charter, advisory board, working groups)
  should be established.
- Whether **reference implementation requirements** should be promoted from
  Candidate-promotion criteria (§15.1.7, §17.5.1) to normative obligations for the
  spec editors.
- Whether **interpretation advisories** (§24.5) should be published as a separate
  document class with their own lifecycle and identifier.

---

## 25. Conformance Test Suite

This section specifies the purpose, structure, and obligations of the JOOT conformance
test suite. The test suite is the primary instrument by which validator implementations
demonstrate conformance to §18.5, §19, and §20, and by which the specification editors
demonstrate that the specification itself is implementable and unambiguous.

### 25.1 Purpose

The conformance test suite exists to:

1. Verify that validator implementations correctly report violations with the codes
   and severities specified in §19.
2. Verify that consumer implementations correctly tolerate and recover from malformed
   input per §1.5 and §20.6.
3. Provide a shared reference against which interoperability between implementations
   is measured (§15.1.7 and §17.5.1 cite "at least two independent implementations"
   for `Stable` promotion).
4. Expose specification ambiguities and defects through concrete cases that
   implementations disagree on.

### 25.2 Publication Status

The JOOT conformance test suite **is not published with this v0.1 specification**. Its
publication is a prerequisite for profiles and dialects to reach `Stable` status
(§15.1.7 point Stable). The editors (§24.1) are responsible for its publication on a
schedule coordinated with profile and dialect stabilization.

v0.1 establishes the framework (this section). The suite itself is a forthcoming
artifact.

### 25.3 Test Case Structure

Each test case in the suite MUST consist of:

1. **Test identifier.** A stable unique identifier for the case (e.g.,
   `syn-010-case-01`). The identifier MUST NOT be reused across cases.
2. **Input document(s).** One or more `.joot` source files.
3. **Applicable validation mode.** One or more of `structural`, `core`, `profile`,
   `corpus`.
4. **Expected diagnostics.** The list of diagnostic codes (§19) a conformant
   validator MUST emit when processing the input in the specified mode.
5. **Expected severity per mode.** For mode-dependent codes, the severity applicable
   to the case.
6. **Expected conformance verdict.** Whether the document (or corpus) is conformant,
   non-conformant, or structurally invalid in the specified mode.
7. **Specification references.** The sections that the test exercises.
8. **Rationale.** A brief description of what the test case demonstrates.

Optional fields:

- Expected source-location annotations (which line and column each diagnostic
  references).
- Expected recovery actions applied and their recovery warnings.
- Prose notes on interpretation edge cases.

### 25.4 Test Case Categories

The suite MUST cover, at minimum, the following categories:

1. **Grammar tests.** Exercise every relevant production of the normative syntax
   defined in Part II and Appendix A. Include positive cases (valid input) and
   negative cases (invalid input with expected `SYN` codes).
2. **Identity tests.** Exercise identity uniqueness, facet decomposition, forbidden
   facet patterns, anonymous entry rules (§12). Include positive and negative cases.
3. **Graph tests.** Exercise target resolution, dangling edges, external targets,
   cardinality constraints, role compatibility (§13).
4. **Vocabulary closure tests.** Exercise open-vocabulary behavior in core mode and
   closed-vocabulary behavior in profile mode (§14.4.1).
5. **Profile tests.** Exercise each registered profile's addendum (§15.3, §15.4),
   including every `CON` code the profile triggers.
6. **Dialect tests.** Exercise dialect activation, profile whitelist interaction, and
   dialect-specific vocabulary (§16).
7. **Error catalog tests.** Exercise at least one case per code in §19.3–§19.6.
8. **Validator behavior tests.** Exercise each of the minimum reporting, rejection,
   tolerance, and recovery requirements of §20.
9. **Version compatibility tests.** Exercise consumer behavior on documents with
   declared versions higher, lower, and equal to the consumer's implemented version
   (§21.4).
10. **Corpus tests.** Exercise cross-document identity uniqueness, cross-document
    target resolution, and cross-document consistency (§18.7, §20.7).

### 25.5 Test Suite Organization

The suite SHOULD be organized in a directory hierarchy that mirrors the specification's
part/section structure. Each test case SHOULD be a self-contained artifact with its
inputs and expected outputs in the same directory.

The suite MAY be delivered as:

- A versioned repository of test case files, one per case.
- A single machine-readable manifest listing all cases and their expectations.
- Both simultaneously.

The specific format is not prescribed; the editors choose and document it when the
suite is published.

### 25.6 Validator Conformance via the Suite

A validator claims conformance to a specific JOOT revision by:

1. Executing the published test suite for that revision against the validator.
2. Producing a conformance report identifying which test cases pass, which fail, and
   which the validator cannot execute (e.g., because they test a mode or profile the
   validator does not support).
3. Publishing the conformance report as part of the validator's self-declaration
   (§20.8).

A validator that fails any test case in a mode it claims to support is non-conformant
in that mode for that revision. A validator MAY claim partial conformance (covering
some modes but not others); partial conformance MUST be explicitly stated in the
self-declaration.

### 25.7 Test Suite Evolution

The test suite evolves alongside the specification. Each specification revision MAY:

1. Add new test cases exercising new features, new error codes, or new profiles.
2. Correct existing test cases whose expected outputs were incorrect.
3. Retire test cases whose underlying rules are retired (consistent with §17.4.5).

The test suite MUST NOT silently alter the expected output of an existing test case
whose underlying rules are unchanged. Such alterations are specification defects.

### 25.8 Reference Implementations

Reference implementations — at least one for `Candidate` status, at least two for
`Stable` (§15.1.7, §17.5.1) — are distinct from the test suite. A reference
implementation is a validator (or broader tool) that passes the test suite; the test
suite is the criterion, the implementations are the demonstrations.

The editors SHOULD identify reference implementations in the revision's release notes
(§24.3) when profiles or dialects reach `Candidate` or `Stable` status.

### 25.9 Out of Scope

- The specific file format of the test suite (directory structure, manifest schema).
- The specific test harness (executor) for running the suite against a validator.
- Performance or timing expectations for validator execution on the suite.
- Test case minimization or coverage metrics.

These concerns are addressed by the editors when the suite is published.

### 25.10 Open Questions for Future Revisions

- Whether a **machine-readable manifest format** for the test suite should be
  normatively specified.
- Whether **fuzz-testing corpora** (automatically generated adversarial input) should
  be published alongside the curated test suite.
- Whether **coverage reporting** (which specification rules are exercised by which
  tests) should be part of the suite's publication.
- Whether the editors should maintain a **public validator registry** of conformance
  reports from registered implementations.

---

*End of §25 and Part VI. Part VII (Examples) continues with §26.*

---

# Part VII — Examples

Part VII provides worked examples that illustrate the specification through concrete
JOOT documents. The examples are informative; they do not introduce new normative
rules. Their purpose is to make the specification's intent clear through demonstration,
serving authors, implementers, and reviewers.

Each example is self-contained and annotated with a brief rationale. Readers MAY treat
the examples as templates for their own documents, subject to the normative rules of
Parts I–VI.

---

## 26. Canonical Examples

This section contains ten canonical examples spanning the range of JOOT's intended
use cases. Examples are numbered §26.1 through §26.10 and each targets a distinct
domain or feature set.

### 26.1 Minimal Core-Conformant Document

A minimal document demonstrating core conformance: header, a single section, a single
entry.

```
# example-01.joot v0.1
# Purpose: Minimal core-conformant document demonstrating the smallest valid file.
# Source: authored
# Entries: 1

$EXAMPLES
!hello_world|cat:example
```

**Rationale.** Shows the minimum structure: four header lines, one blank line, one
section, one entry with only the required `cat:` meta key. No profile is declared, so
core conformance applies. Entry count (1) matches header line 4.

### 26.2 Simple API Contract — Linked-Contract Form

An HTTP GET endpoint with its request, two response variants, and an error contract,
using the linked-contract form (Model A of §15.4.5).

```
# example-02.joot v0.1 profile=api_contract
# Purpose: Simple GET endpoint using linked-contract form.
# Source: authored
# Entries: 5

$ORDERS
!http.orders.get|cat:endpoint|id:http.orders.get|kind:rest|path:/orders/{id}|verb:GET|[Rel]error:http.orders.get.error_404,request:http.orders.get.request,response:http.orders.get.response_200,response:http.orders.get.response_304|[Behavior]auth:bearer,cacheable,idempotent,safe
!http.orders.get.request|cat:contract|id:http.orders.get.request|kind:request_body|[Fields]order_id:text,if_none_match:text{optional}
!http.orders.get.response_200|cat:contract|id:http.orders.get.response_200|kind:response_body|status_code:200|[Fields]order_id:text,customer_id:text,total_cents:int,etag:text
!http.orders.get.response_304|cat:contract|id:http.orders.get.response_304|kind:response_body|status_code:304
!http.orders.get.error_404|cat:contract|id:http.orders.get.error_404|kind:error_body|status_code:404|[Fields]error_code:text,message:text
```

**Rationale.** The endpoint is unfaceted. Its four related contracts (one request, two
responses, one error) are separate subjects linked via `[Rel]`. `status_code:` meta on
each response/error disambiguates. `[Rel]` items are canonically ordered alphabetically
by role. Canonical meta key order (`cat:`, `id:`, `kind:`, `path:`, `verb:`) observed.

### 26.3 Simple API Contract — Endpoint-Facet Form

The same endpoint as §26.2 represented in the endpoint-facet form (Model B of §15.4.5).

```
# example-03.joot v0.1 profile=api_contract
# Purpose: Simple GET endpoint using endpoint-facet form.
# Source: authored
# Entries: 4

$ORDERS
!http.orders.get|cat:endpoint|id:http.orders.get|kind:rest|facet:main|path:/orders/{id}|verb:GET|[Behavior]auth:bearer,cacheable,idempotent,safe
!http.orders.get|cat:endpoint|id:http.orders.get|kind:rest|facet:request|path:/orders/{id}|verb:GET|[Fields]order_id:text,if_none_match:text{optional}
!http.orders.get|cat:endpoint|id:http.orders.get|kind:rest|facet:response|path:/orders/{id}|verb:GET|status_code:200|[Returns]order_id:text,customer_id:text,total_cents:int,etag:text
!http.orders.get|cat:endpoint|id:http.orders.get|kind:rest|facet:error|path:/orders/{id}|verb:GET|status_code:404|[Returns]error_code:text,message:text
```

**Rationale.** Four facets (`main`, `request`, `response`, `error`) of the same
endpoint `id:`. All facets declare identical `path:` and `verb:` per §15.4.6. The
`main` facet carries top-level behavior; the others carry their specific shapes.
Compare entry counts with §26.2: 4 vs 5.

### 26.4 Code Surface — Java Class with Overloaded Methods

A Java class with two overloaded methods, demonstrating overload identity (§15.3.9).

```
# example-04.joot v0.1 profile=code_surface
# Purpose: Java class with overloaded methods demonstrating signature-based identity.
# Source: authored
# Entries: 5

$ORDER_SERVICE
!OrderService|cat:type|id:java.com.acme.order.OrderService|kind:class|[Rel]implements:java.com.acme.order.OrderLookup,member_of:java.com.acme.order|[Behavior]vis:public
!find_single|cat:callable|id:java.com.acme.order.OrderService.find(text)|kind:method|[Rel]member_of:java.com.acme.order.OrderService,throws:java.com.acme.order.NotFound|[Fields]order_id:text|[Returns]order:record<Order>|[Behavior]vis:public,bind:instance
!find_batch|cat:callable|id:java.com.acme.order.OrderService.find(text,int)|kind:method|[Rel]member_of:java.com.acme.order.OrderService|[Fields]order_id:text,batch_size:int{min_1,max_100}|[Returns]orders:list<record<Order>>|[Behavior]vis:public,bind:instance
!OrderLookup|cat:type|id:java.com.acme.order.OrderLookup|kind:interface|[Rel]member_of:java.com.acme.order|[Behavior]vis:public
!NotFound|cat:type|id:java.com.acme.order.NotFound|kind:class|[Rel]extends:ext:java.lang.RuntimeException,member_of:java.com.acme.order|[Behavior]vis:public
```

**Rationale.** Two overloads of `find` have distinct `id:` values including parameter
type signatures: `find(text)` and `find(text,int)`. Both are child subjects of
`OrderService` via `member_of` with cardinality `1` (§15.3.11 check 12). The class
extends an external type (`ext:java.lang.RuntimeException`). `[Rel]` items are
canonically ordered.

### 26.5 Database Schema — Table with Foreign Keys

A relational schema fragment: two tables with a foreign key relationship. Uses
`cat:schema` with `code_surface`-adjacent conventions; this example is illustrative of
how a schema dialect (not defined in v0.1) might work.

```
# example-05.joot v0.1
# Purpose: Relational schema fragment with foreign key relationship.
# Source: authored
# Entries: 2

$ORDERS_SCHEMA
!orders|cat:schema|id:sql.public.orders|kind:table|[Rel]references:sql.public.customers|[Fields]id:int{required},customer_id:int{required},total_cents:int{required,min_0},created_at:date{required},status:enum{draft/submitted/approved/shipped/cancelled}{required}
!customers|cat:schema|id:sql.public.customers|kind:table|[Fields]id:int{required},email:text{required},name:text{required},created_at:date{required}
```

**Rationale.** Core conformance (no profile). Two schema subjects with an `references`
edge from `orders` to `customers`. Constraints on fields (`min_0`, `required`). Enum
type in `status`. This illustrates how schema content can be expressed in core JOOT
before a dedicated schema profile exists.

### 26.6 Workflow — State Machine with Transitions

A simple state machine: draft → submitted → approved, with an on_fail edge.

```
# example-06.joot v0.1
# Purpose: State machine with ordered transitions and failure paths.
# Source: authored
# Entries: 6

$ORDER_WORKFLOW
!draft|cat:state|id:wf.order.draft|kind:state|[Behavior]initial
!submitted|cat:state|id:wf.order.submitted|kind:state
!approved|cat:state|id:wf.order.approved|kind:state|[Behavior]terminal
!rejected|cat:state|id:wf.order.rejected|kind:state|[Behavior]terminal
!submit|cat:transition|id:wf.order.submit|kind:transition|[Rel]from:wf.order.draft,on_fail:wf.order.rejected,to:wf.order.submitted
!approve|cat:transition|id:wf.order.approve|kind:transition|[Rel]from:wf.order.submitted,on_fail:wf.order.rejected,to:wf.order.approved
```

**Rationale.** States and transitions as separate subjects. Transitions carry their
own `[Rel]` block with `from:` / `to:` / `on_fail:`. Initial and terminal behavior
atoms mark the start and end states. Canonical ordering in `[Rel]` observed.

### 26.7 Faceted Type — Structural and Behavioral Projections

A type split into two facets: structural view (fields) and behavioral view (invariants).

```
# example-07.joot v0.1 profile=code_surface
# Purpose: Type faceted into structural and behavioral projections.
# Source: authored
# Entries: 2

$MONEY_TYPE
!Money|cat:type|id:java.com.acme.util.Money|kind:class|facet:structural|[Fields]amount_cents:int,currency:text{length_3}|[Behavior]vis:public,mut:immutable
!Money|cat:type|id:java.com.acme.util.Money|kind:class|facet:behavioral|[Behavior]inv:amount_cents_nonnegative,inv:currency_iso_4217,post:arithmetic_preserves_currency
```

**Rationale.** One logical subject (`Money`), two facets. Shared `id:`, `cat:`,
`kind:`. The structural facet carries fields and basic behavior; the behavioral facet
carries domain invariants. This is the valid use of facet decomposition for a type
with independently-maintained projections.

### 26.8 Deprecation and Supersession

A deprecated endpoint with a pointer to its replacement.

```
# example-08.joot v0.1 profile=api_contract
# Purpose: Deprecation of an old endpoint and pointer to its successor.
# Source: authored
# Entries: 2

$LEGACY_ORDERS
!http.orders.legacy_get|cat:endpoint|id:http.orders.legacy_get|kind:rest|path:/v1/orders/{id}|verb:GET|status:deprecated|sunset:v2.0|[Rel]replaced_by:http.orders.get,request:http.orders.legacy_get.request,response:http.orders.legacy_get.response_200|[Behavior]auth:bearer|#note:Superseded by /v2/orders/{id}; retained for client compatibility until v2.0.|#warn:Do not use for new integrations.
!http.orders.get|cat:endpoint|id:http.orders.get|kind:rest|path:/v2/orders/{id}|verb:GET|since:v2.0|[Rel]supersedes:http.orders.legacy_get|[Behavior]auth:bearer
```

**Rationale.** `status:deprecated`, `sunset:v2.0`, `replaced_by:` relation on the old
entry. `since:v2.0`, `supersedes:` relation on the new entry. Annotations
(`#note:`, `#warn:`) carry human-readable context. Note the canonical meta key order
is preserved (`status:` before `sunset:`, `since:` before `supersedes:` in its row).

### 26.9 Metric and Test for an Endpoint

An endpoint with associated observability metric and conformance test.

```
# example-09.joot v0.1 profile=api_contract
# Purpose: Endpoint with associated observability metric and conformance test.
# Source: authored
# Entries: 3

$ORDERS_OBSERVABILITY
!http.orders.list|cat:endpoint|id:http.orders.list|kind:rest|path:/orders|verb:GET|[Behavior]auth:bearer,cacheable,idempotent,latency_p95_ms:200,safe,throughput_rps:500
!latency_p95|cat:metric|id:metrics.http.orders.list.latency_p95|kind:latency|[Rel]for:http.orders.list|[Fields]threshold_ms:int,window_seconds:int
!conformance_basic|cat:test|id:tests.http.orders.list.basic|kind:contract_conformance|[Rel]for:http.orders.list|[Fields]expected_status:int,expected_fields:list<text>
```

**Rationale.** Three subjects linked by `for:` relation. The endpoint declares latency
and throughput expectations in its behavior block; the metric subject describes the
measurement that verifies them; the test subject describes the conformance scenario.

### 26.10 Corpus Spanning Profiles

A two-document corpus: one `code_surface` document and one `api_contract` document,
with cross-profile references.

**Document A — code-surface.joot:**

```
# code-surface.joot v0.1 profile=code_surface
# Purpose: Code surface describing the handler for an API endpoint.
# Source: authored
# Entries: 2

$ORDER_HANDLERS
!OrderHandler|cat:type|id:java.com.acme.http.OrderHandler|kind:class|[Rel]member_of:java.com.acme.http|[Behavior]vis:public
!get|cat:callable|id:java.com.acme.http.OrderHandler.get(text)|kind:method|[Rel]conforms_to:http.orders.get,member_of:java.com.acme.http.OrderHandler|[Fields]order_id:text|[Returns]response:record<OrderResponse>|[Behavior]vis:public,bind:instance,effect:reads_state
```

**Document B — api-contract.joot:**

```
# api-contract.joot v0.1 profile=api_contract
# Purpose: API contract for the orders endpoint, targeted by OrderHandler.
# Source: authored
# Entries: 1

$ORDERS
!http.orders.get|cat:endpoint|id:http.orders.get|kind:rest|path:/orders/{id}|verb:GET|[Behavior]auth:bearer,cacheable,idempotent,safe
```

**Rationale.** Two documents declare different profiles. The `code_surface`
document's callable declares `conforms_to:http.orders.get`, referencing a subject in
the `api_contract` document. Under corpus conformance, the cross-document reference
resolves (§15.5.3, §20.7.4). Each document remains profile-conformant in its own
scope; the corpus is conformant when both documents are validated together.

---

# Appendix A — Normative ABNF Grammar

This appendix is **normative**. It is the formal grammar of JOOT v0.1 in Augmented
Backus-Naur Form (ABNF) per RFC 5234 and RFC 7405. This appendix MUST be consistent
with the prose specification of Parts I–II and §7 (Type System). In the event of a
discrepancy, the resolution is a specification defect (§7 rule, restated here).

## A.1 Core Rules (from RFC 5234)

```abnf
ALPHA          = %x41-5A / %x61-7A   ; A-Z / a-z
DIGIT          = %x30-39              ; 0-9
UPPER          = %x41-5A              ; A-Z
LOWER          = %x61-7A              ; a-z
LF             = %x0A
CR             = %x0D
CRLF           = CR LF
SP             = %x20
HT             = %x09                 ; FORBIDDEN in JOOT
```

## A.2 Document Structure

```abnf
document       = header blank-line sections [final-lf]

header         = header-line-1 LF header-line-2 LF header-line-3 LF header-line-4 LF

header-line-1  = "#" SP filename ".joot" SP version-token
                 [SP profile-token] [SP dialect-token]
header-line-2  = "#" SP "Purpose:" SP purpose-text
header-line-3  = "#" SP "Source:" SP source-text
header-line-4  = "#" SP "Entries:" SP 1*DIGIT

version-token  = "v" 1*DIGIT ["." 1*DIGIT]
profile-token  = "profile=" profile-name
dialect-token  = "dialect=" dialect-name
profile-name   = LOWER *(LOWER / DIGIT / "_")
dialect-name   = LOWER *(LOWER / DIGIT / "_")

blank-line     = LF
final-lf       = LF

sections       = section *(blank-line section)
section        = section-header *(section-comment) 1*entry-line
section-header = "$" section-name LF
section-name   = (UPPER / DIGIT) *(UPPER / DIGIT / "_")
section-comment = "#" section-comment-text LF
```

## A.3 Entry Grammar

```abnf
entry-line     = sigil entry-name "|" meta-block
                 ["|" rel-block]
                 ["|" fields-block]
                 ["|" behavior-block]
                 ["|" returns-block]
                 *("|" note-annotation)
                 *("|" warn-annotation)
                 LF

sigil          = "!" / ">" / "^" / "@" / "+"

entry-name     = entry-segment *(">" entry-segment)
entry-segment  = entry-name-char *entry-name-char
entry-name-char = ALPHA / DIGIT / "_"
```

## A.4 Meta Block

```abnf
meta-block     = cat-field *("|" meta-field)
cat-field      = "cat:" meta-value
meta-field     = meta-key ":" meta-value
meta-key       = LOWER *(LOWER / DIGIT / "_")
meta-value     = unquoted-value / quoted-string
```

## A.5 Named Blocks

```abnf
rel-block      = "[Rel]" rel-item *("," rel-item)
rel-item       = role ":" target [constraint-block]
role           = LOWER *(LOWER / DIGIT / "_")
target         = internal-target / external-target
internal-target = id-value ["#" facet-value]
external-target = "ext:" (ext-token / quoted-string)
ext-token      = 1*(ALPHA / DIGIT / "_" / "." / "-" / "/" / ":" / "(" / ")" / "+")

fields-block   = "[Fields]" field-item *("," field-item)
field-item     = field-name ":" type [constraint-block]
field-name     = LOWER *(LOWER / DIGIT / "_")

behavior-block = "[Behavior]" behavior-item *("," behavior-item)
behavior-item  = bare-atom / (key ":" value [constraint-block])

returns-block  = "[Returns]" return-item *("," return-item)
return-item    = field-item

note-annotation = "#note:" annotation-content
warn-annotation = "#warn:" annotation-content
```

## A.6 Types

```abnf
type           = union-type / non-union-type

non-union-type = scalar-type
               / generic-type
               / reference-type
               / compound-type
               / nominal-type

scalar-type    = "text" / "number" / "int" / "bool" / "date" / "url" / "image" / "any"

generic-type   = "expr"   [ "<" type ">" ]
               / "list"   "<" type ">"
               / "record" "<" type ">"
               / "map"    "<" type "," type ">"

reference-type = identifier "_ref"

compound-type  = "enum" "{" enum-value *("/" enum-value) "}"
               / "field_changes"
               / "key_value_pairs"
               / "params"

union-type     = non-union-type "|" non-union-type *("|" non-union-type)

nominal-type   = identifier *("." identifier)
identifier     = ALPHA *(ALPHA / DIGIT / "_")
enum-value     = identifier
```

## A.7 Constraints

```abnf
constraint-block = "{" atom *("," atom) "}"
atom             = bare-atom / keyed-atom / range-atom
bare-atom        = LOWER *(LOWER / DIGIT / "_")
keyed-atom       = bare-atom ":" value
range-atom       = 1*DIGIT "_to_" 1*DIGIT
value            = identifier / number-literal / quoted-string
number-literal   = 1*DIGIT [ "." 1*DIGIT ]
```

## A.8 Identifiers

```abnf
id-value       = id-segment *("." id-segment)
id-segment     = id-char-1 *id-char-n
id-char-1      = ALPHA / DIGIT / "_"
id-char-n      = ALPHA / DIGIT / "_" / "-" / "/" / "(" / ")" / "+"
facet-value    = LOWER *(LOWER / DIGIT / "_")
```

## A.9 Quoting and Escaping

```abnf
unquoted-value = 1*(unreserved-char / escape-sequence)
unreserved-char = %x21-7E ; printable ASCII
                 except %x7C "|" / %x2C "," / %x5B "[" / %x5D "]"
                 / %x7B "{" / %x7D "}" / %x5C "\" / %x22 "\""

escape-sequence = "\|" / "\," / "\[" / "\]" / "\{" / "\}" / "\\" / "\""

quoted-string  = DQUOTE *(quoted-char / quoted-escape) DQUOTE
DQUOTE         = %x22
quoted-char    = %x20-21 / %x23-5B / %x5D-7E / non-ascii-utf8
quoted-escape  = "\\" / "\""

non-ascii-utf8 = <any valid UTF-8 code point per §3.3 forbidden-set>

annotation-content = 1*(printable-char-except-lf-pipe / escape-sequence) / quoted-string
```

## A.10 Alignment With Prose

This grammar is intended to match the prose specification of Parts I–II and §7
exactly. If an implementer finds a production missing or inconsistent relative to the
prose, the discrepancy is a specification defect and MUST be reported to the editors.
The prose and the ABNF are co-authoritative: neither overrides the other.

---

# Appendix B — Quick Reference

This appendix is **informative**. It summarizes the most frequently consulted aspects
of the specification for readers who already understand the framework. It is not a
substitute for the normative text.

## B.1 Entry Anatomy

```
<sigil><entry_name>|cat:<X>|id:<Y>|kind:<Z>|<meta...>|[Rel]<rels>|[Fields]<fields>|[Behavior]<behavior>|[Returns]<returns>|#note:...|#warn:...
```

Block order is fixed (§6.2). Annotations last. Notes before warnings (§10.3).

## B.2 Sigils

| `!` Entity | `>` Source | `^` Operation | `@` Event | `+` Action |

## B.3 Canonical Meta Key Order

```
cat: → id: → kind: → facet: → scope: → context: → target: → requires:
    → path: → verb: → status: → since: → sunset: → until: → <profile/dialect>
```

## B.4 Canonical Block Order

```
meta → [Rel] → [Fields] → [Behavior] → [Returns] → #note → #warn
```

## B.5 Universal Types

- Scalars: `text`, `number`, `int`, `bool`, `date`, `url`, `image`, `any`
- Generics: `expr[<T>]`, `list<T>`, `record<T>`, `map<K,V>`
- Compound: `enum{a/b/c}`, `field_changes`, `key_value_pairs`, `params`
- References: `<name>_ref`

## B.6 Reserved Characters

```
|   ,   [   ]   {   }   \   "
```

Escape with `\` or wrap in `"..."`.

## B.7 Error Code Families

- `SYN-xxx` syntactic
- `SEM-xxx` semantic
- `CON-xxx` conformance
- `WRN-xxx` warning
- `000` reserved per family. Append-only. No reuse.

## B.8 Conformance Layers

- **Core:** §§4–14, open vocabularies
- **Profile:** core + profile addendum, closed vocabularies, canonical ordering enforced
- **Corpus:** profile + cross-document invariants

## B.9 Validation Modes

`structural` → `core` → `profile` → `corpus` (strictness ascending, §1.3)

## B.10 Profiles in v0.1

- `code_surface` (Candidate, §15.3)
- `api_contract` (Candidate, §15.4)

---

# Appendix C — Design Rationale

This appendix is **informative**. It records the reasoning behind non-obvious design
decisions in JOOT v0.1.

## C.1 Why Line-Oriented?

One entry per line (§P3) was chosen to guarantee greppability, diff stability, and
positional retrieval by LLMs. Multi-line entries would have required quoting or
heredoc-style continuation, both of which add parser complexity and degrade the
core use case: fast machine retrieval.

## C.2 Why Pipe Separators?

Pipe (`|`) is rarely used in technical identifiers, rarely appears in prose, and is
visually distinct from comma (`,`), colon (`:`), and slash (`/`) which serve other
roles in JOOT. Its visual distinctiveness reduces ambiguity when reading a dense line.

## C.3 Why ASCII-Only Identifiers in v0.1?

ASCII identifiers eliminate the confusable-character attack surface (§3.5), simplify
normalization, and match the conventions of most programming languages and data
formats JOOT is intended to interoperate with. A future revision may open identifiers
to Unicode with explicit UTS #39 restrictions (§3.7).

## C.4 Why Facets Rather Than Nested Blocks?

Faceted subjects (§12.3) use multiple entries sharing an `id:` instead of a single
entry with nested block hierarchies. This preserves the one-entry-per-line invariant
while permitting orthogonal projection decomposition. Nested blocks would have
broken §P3 and introduced parser complexity.

## C.5 Why Append-Only Registries?

Append-only evolution (§14.4.2, §17.5.4) preserves backward compatibility within a
major version and prevents silent breakage of existing corpora. The cost — registries
grow over time — is acceptable because registries are catalogs, not queries; growth
does not hurt lookup.

## C.6 Why Two Separate Profile/Document Version Declarations?

The JOOT version declares the format revision (§4.2.2, §21.1). Profile names declare
vocabulary closure but **not** a profile version in v0.1 (§21.6.1). This allows JOOT
and profiles to evolve on independent cadences, at the cost of some ambiguity about
which profile version applies (documented in §15.5.4).

## C.7 Why No PostgreSQL/SQL Keywords as Reserved?

JOOT avoids mirroring SQL, JSON, YAML, or language-specific reserved words because it
is intended as a vendor-neutral IR. Profiles and dialects introduce domain vocabulary;
the core stays lean.

## C.8 Why `request`/`response`/`error` as Universal Roles?

API contracts are pervasive in technical surface description. Making these roles
universal (§14.2) rather than `api_contract`-specific enables cross-profile reference
(a `code_surface` callable can `conforms_to:` an `api_contract` endpoint) without
needing dialect activation.

## C.9 Why Canonical Orderings Are Errors Under Profile Conformance?

Under core conformance, non-canonical ordering is a warning to support exploratory
authoring and human tooling. Under profile conformance, it is an error because
machine-first consumption relies on canonical ordering for deterministic retrieval,
streaming processing, and diff stability (§P4).

## C.10 Why So Many Forward-References in the Draft?

v0.1 is written to be read forward but needs to establish foundational concepts
(identity, graph, conformance) before laying out their details. Forward-references
are explicit (every one of them) and were resolved before v0.1 publication. Future
revisions SHOULD aim to minimize forward-references through editorial restructuring.

---

# Appendix D — Format Comparison

This appendix is **informative**. It situates JOOT relative to adjacent technical
data formats, identifying overlap and distinct focus areas.

## D.1 JOOT vs JSON

- **JSON** is a general-purpose data serialization format. It represents arbitrary
  trees of strings, numbers, booleans, arrays, and objects.
- **JOOT** is a specialized format for semantic technical surfaces. It represents
  addressable subjects with typed relations, canonical vocabularies, and graph
  structure.
- Overlap: both carry structured data.
- Distinct: JSON has no identity model, no graph model, no vocabulary discipline, and
  no line-oriented structure.

## D.2 JOOT vs YAML

- **YAML** is human-friendly data serialization. It supports nested hierarchies,
  anchors, references, and typed scalars.
- **JOOT** rejects nested structural hierarchies in favor of flat line-oriented
  entries with graph relations.
- Overlap: both are plain-text, human-readable.
- Distinct: YAML's flexibility is JOOT's anti-pattern; JOOT's rigidity enables
  machine-first retrieval.

## D.3 JOOT vs OpenAPI

- **OpenAPI** is a format for HTTP API specification. It has extensive vocabulary for
  endpoints, schemas, security, servers, and examples.
- **JOOT** with `api_contract` profile covers a substantial overlap with OpenAPI:
  endpoints, request/response shapes, errors, authentication.
- Overlap: API contract representation.
- Distinct: OpenAPI is JSON/YAML-based and HTTP-specific; JOOT is line-oriented and
  multi-domain. JOOT does not replace OpenAPI; it offers a machine-first IR that MAY
  be derived from OpenAPI via a mapping document (out of scope for v0.1).

## D.4 JOOT vs JSON Schema

- **JSON Schema** is a format for describing the shape of JSON documents. It has rich
  validation vocabulary (types, patterns, ranges, enums, composition keywords).
- **JOOT** with a schema-oriented profile (not yet normatively defined in v0.1) can
  express similar shape information, but its primary orientation is toward semantic
  subjects and relations, not runtime value validation.
- Overlap: describing data shapes.
- Distinct: JSON Schema's runtime validation role vs JOOT's semantic catalog role.

## D.5 JOOT vs Markdown + Structured Annotations

- **Markdown with structured annotations** (e.g., JSDoc, Doxygen, Docstrings) is the
  common way to embed machine-readable metadata in human-readable documentation.
- **JOOT** separates the machine-readable catalog from the human-readable prose,
  operating alongside Markdown rather than within it.
- Overlap: both capture semi-structured technical metadata.
- Distinct: Markdown is prose-first with machine affordances; JOOT is machine-first
  with human legibility as a secondary property.

## D.6 JOOT vs RDF / Turtle

- **RDF** (represented as Turtle, JSON-LD, N-Triples, etc.) is a graph data format
  with open semantics and URI-based identity.
- **JOOT** is a graph format with closed core grammar and opaque-string identity.
- Overlap: graph representation of relations between named entities.
- Distinct: RDF's flexibility and URI system vs JOOT's rigid grammar and compact
  identifiers. JOOT does not aim to be RDF; it borrows the triple-like model
  (§13.1) but operates at a different layer.

## D.7 JOOT vs Language-Specific Reflection Output

- **Reflection output** (e.g., `javap` for Java, TypeScript type declarations,
  Python `inspect` dumps) exposes the surface of a codebase to tooling.
- **JOOT** with `code_surface` profile provides a language-neutral canonical
  representation of the same information.
- Overlap: code surface description.
- Distinct: reflection output is language-specific and tool-specific; JOOT
  normalizes across languages.

## D.8 Positioning Summary

JOOT is best understood as a **canonical intermediate representation** for technical
surfaces, designed for LLM and agent consumption, with selective overlap with each
adjacent format but distinct primary orientation. It does not seek to replace any of
them; it complements them by offering a shared target for ingestion from and emission
to diverse source formats.

---

*End of appendices. End of JOOT Format Specification v0.1.*
