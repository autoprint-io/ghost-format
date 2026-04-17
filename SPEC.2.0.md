# Ghost Format Specification

**Version:** 2.0
**Status:** Candidate
**Date:** 2026-04-17
**Extension:** `.ghost`
**Encoding:** UTF-8, no BOM
**Line endings:** `LF` (`CRLF` accepted by consumers)

---

## 1. Abstract

Ghost is a flat, line-oriented, pipe-delimited plain-text format for encoding structured
technical knowledge — documentation, platform metamodels, schema blueprints, API contracts,
code surfaces, runtime catalogs, and workflow definitions — in a form optimized for LLM
context windows.

One entry per physical line. Canonical block order. Dense types and constraints. No nested
blocks. No ad-hoc substructures. Optional machine-first profile governance. Stable graph edges
between first-class subjects. The format is designed to preserve fields, types, constraints,
behaviors, returns, graph relations, notes, and warnings with far fewer tokens than equivalent
Markdown, YAML, or JSON.

Ghost has one design commitment:

> **The LLM is the parser.**

Every rule in this specification follows from that.

This document specifies Ghost v2. Files conformant to Ghost v1.1 remain valid core v2 files.

---

## 2. Design Principles

1. **Token density is the primary goal.** Every byte must justify itself.
2. **No loss of technical meaning** on fields, types, constraints, behaviors, returns, relations, notes, and warnings.
3. **One entry per physical line.** Greppability and diff stability are mandatory.
4. **Canonical ordering everywhere.** Positional regularity improves LLM retrieval.
5. **LLM comprehension over strict parser minimalism.** The format is intentionally regular, not ornamental.
6. **Core grammar is tiny and closed.** Profiles and dialects add vocabulary, never syntax.
7. **Readable quickly, learnable completely.**
8. **No nested named blocks.** Structure is flattened into fields, behaviors, returns, and relations.
9. **Graph edges are first-class.** Relations between subjects are machine-resolvable.
10. **Governance is opt-in.** Core validation is lenient. Machine-first profiles close vocabularies and enforce identity.

---

## 3. File Structure

### 3.1 Header

Every `.ghost` file starts with **exactly 4 lines**:

```text
# <filename>.ghost v<MAJOR>[.<MINOR>] [profile=<name>] [dialect=<name>]
# Purpose: <one-line description>
# Source: <origin or "authored">
# Entries: <integer>
```

Rules:

* Line 1 declares the version and MAY declare a `profile=` and / or a `dialect=`.
* At most one `profile=` and at most one `dialect=` may appear on line 1.
* `profile=` activates machine-first governance (see § 12).
* `dialect=` activates vocabulary extensions (see § 13).
* A file MAY declare both a `profile=` and a `dialect=` when the profile permits the dialect.
* Line 4's entry count MUST equal the number of actual entry lines in the file.
* No 5th header line is allowed.
* Exactly **one blank line** follows the header.

---

### 3.2 Sections

A section is a line matching:

```text
$UPPER_CASE_UNDERSCORED
```

Rules:

* Every entry MUST appear inside a section.
* Exactly **one blank line** precedes every section after the first.
* The first section appears immediately after the single blank line following the header.

Example:

```text
$AUTH_SIGNUP_LOGIN
```

---

### 3.3 Section Comments

A section MAY be followed by zero or more `#`-prefixed section comments.

These comments provide context to the LLM and are not counted as entries.

Example:

```text
$COOKIES
#Visible only when cookie controls are enabled
```

Rules:

* Section comments MAY appear only immediately after a section header.
* No blank line may appear between a section header, its section comments, and its first entry.

---

### 3.4 Entries

An entry is a single physical line beginning with a sigil defined in § 4.

Rules:

* Entries MUST be contiguous within a section.
* No blank lines may appear between entries in the same section.
* One entry equals one physical line. Hard wraps are forbidden.

---

## 4. Entry Sigils

Each entry begins with a one-character sigil.

| Sigil | Kind      | Use for                                                     |
| ----- | --------- | ----------------------------------------------------------- |
| `!`   | Entity    | Types, modules, elements, endpoints, contracts, records     |
| `>`   | Source    | Data sources, providers, origins, inputs                    |
| `^`   | Operation | Callables, pure functions, transformers, expression operators |
| `@`   | Event     | Triggers, hooks, lifecycle callbacks                        |
| `+`   | Action    | Mutations, commands, workflow steps that change state       |
| `?`   | Reserved  | Reserved. MUST NOT be used.                                 |
| `%`   | Reserved  | Reserved. MUST NOT be used.                                 |

Rules:

* `!` is always valid as the default sigil.
* Richer sigils are recommended when they improve readability or retrieval.
* The sigil is followed immediately by the entry name, with no space.
* `?` and `%` are reserved for future specifications. They MUST NOT appear in any conformant
  core v2 file, nor under any profile or dialect defined by this specification.
* An activating specification is required before either sigil may be used.

---

## 5. Entry Model

### 5.1 General Form

```text
<sigil><Entry_Name>|<meta>[|<meta>...][|[Rel]...][|[Fields]...][|[Behavior]...][|[Returns]...][|#note:...][|#warn:...]
```

The first named block (or the first annotation) determines where the meta block ends.

---

### 5.2 Canonical Block Order

The block order is mandatory:

```text
meta → [Rel] → [Fields] → [Behavior] → [Returns] → #note → #warn
```

Rules:

* Any block MAY be omitted.
* Omitted blocks leave no placeholder.
* Only these named blocks are valid:

  * `[Rel]`
  * `[Fields]`
  * `[Behavior]`
  * `[Returns]`

Forbidden examples:

* `[Options]`
* `[Params]`
* `[Config]`
* `[Output]`
* `[Headers]`
* `[Status]`
* `[Errors]`
* `[]`
* `no_fields`
* `none`

---

### 5.3 Meta Block

The meta block is the unmarked prefix between the entry name and the first named block.

Meta fields are separated by `|`.

Every entry MUST begin its meta with:

```text
cat:<category>
```

#### Canonical meta keys

| Key         | Purpose                                                  | Example                            |
| ----------- | -------------------------------------------------------- | ---------------------------------- |
| `cat:`      | Category (required, always first)                        | `cat:callable`, `cat:endpoint`     |
| `id:`       | Stable machine-readable identifier                       | `id:java.com.acme.order.Order`     |
| `kind:`     | Subtype of `cat:` (required under machine-first profiles) | `kind:class`, `kind:http`          |
| `facet:`    | Orthogonal facet of the same logical entry               | `facet:appearance`, `facet:req`    |
| `scope:`    | Runtime or platform scope                                | `scope:native_mobile`              |
| `requires:` | Prerequisite feature, plan, flag, or capability          | `requires:paid_plan`               |
| `context:`  | Where the entry runs or is available                     | `context:workflow_only`            |
| `target:`   | What or whom the entry operates on                       | `target:current_user`              |
| `status:`   | Lifecycle status                                         | `status:beta`, `status:deprecated` |
| `path:`     | Endpoint path (`api_contract` only)                      | `path:"/orders/{order_id}"`        |
| `verb:`     | Endpoint verb (`api_contract` only)                      | `verb:GET`                         |
| `since:`    | Version or date introduced                               | `since:v1.3`                       |
| `sunset:`   | Version or date of planned removal                       | `sunset:v2.0`                      |
| `until:`    | Version or date of effective removal                     | `until:v2.1`                       |

Rules:

* `cat:` is required and MUST be the first meta field.
* `id:` is optional in core and REQUIRED under machine-first profiles.
* `kind:` is optional in core and REQUIRED under machine-first profiles.
* `facet:` is recommended when a logical subject has multiple orthogonal property families.
* `status:` MUST NOT carry HTTP response codes. Status variants are modeled as separate
  response / error surfaces (see § 12.3).
* Meta fields describe **preconditions, scope, identity, or classification**.
* Consequences belong in `[Behavior]`, not in meta.
* Graph edges belong in `[Rel]`, not in meta.

Test:

* **Precondition or classification?** → meta
* **Consequence or effect?** → `[Behavior]`
* **Edge to another subject?** → `[Rel]`

---

### 5.4 `[Rel]`

`[Rel]` is the only named block introduced in v2.

`[Rel]` contains a comma-separated list of typed relations to other first-class subjects:

```text
[Rel]member_of:java.com.acme.order,throws:ext:java.com.acme.order.NotFound
```

Relation item grammar:

```text
role:target[{constraints}]
```

`role` grammar:

```text
lower_case_underscored
```

`target` grammar:

```text
target          = internal_target / external_target
internal_target = id / id "#" facet
external_target = "ext:" ext_ref
ext_ref         = ext_token / quoted_string
ext_token       = 1*(ALPHA / DIGIT / "_" / "." / "-" / "/" / ":" / "(" / ")" / "+")
```

Rules:

* Under machine-first profiles, internal relation targets MUST be `id` or `id#facet`.
* Non-ID targets are permitted only as `ext:` targets.
* A machine-first profile or active dialect MUST explicitly permit the use of `ext:` targets,
  either role-by-role or profile-wide. If no such permission exists, `ext:` targets are
  invalid for that role.
* `ext:` targets identify first-class subjects outside the active validation corpus.
* Core validators MUST NOT attempt to resolve `ext:` targets.
* Bare entry names, prose labels, unqualified path fragments, and quoted strings without
  `ext:` MUST NOT be used as relation targets in machine-first profiles.
* `id#facet` addresses one faceted projection of the logical subject identified by `id`.
* `id#facet` is not a new identity and MUST NOT create a second identity record.
* `id` values themselves MUST NOT contain `#`.
* `facet` values referenced by `id#facet` SHOULD match an existing entry with the same `id:`
  and `facet:` in file-level validation when available, and MUST match one in profile-corpus
  validation.
* Unresolved internal relation targets are a warning in file-only validation (W4) and an
  error in profile-corpus validation (E22).

Boundary rule:

* `[Rel]` encodes graph edges between first-class semantic subjects.
* `[Behavior]` encodes local properties, effects, or guards of the current subject.
* `[Fields]` and `[Returns]` encode structural slots, inputs, and outputs of the current
  subject.
* A registered relation role MUST NOT be encoded in `[Behavior]`.

Repeated roles and cardinality:

* Repeated roles are allowed.
* Exact duplicate `role:target` items within one entry MUST NOT occur.
* Repeated `role` items with different targets are valid if the role semantics permit plurality.
* Profiles MUST define whether a role is cardinality `1`, `0..1`, `0..n`, or `1..n`.
* If a profile does not specify a role cardinality, the default cardinality is `0..n`.

Relation constraints:

* Relation constraints use the same `{}` syntax as field / return constraints.
* Relation constraints qualify the edge, not the target subject.
* Machine-first profiles SHOULD keep relation constraints sparse and profile-registered.

Correct:

```text
[Rel]member_of:java.com.acme.order,throws:ext:java.com.acme.order.NotFound
[Rel]annotated_by:ext:java.jakarta.Inject
[Rel]response:http.orders.get.res_200
[Rel]response:http.orders.get#res_200
```

Incorrect:

```text
[Behavior]extends:java.Base
[Rel]member_of:OrderService
[Rel]response:/orders/{id}
```

---

### 5.5 `[Fields]`

`[Fields]` contains a comma-separated list of typed fields:

```text
[Fields]email:expr<text>,time_hrs:number{0_to_24,default:1}
```

Field item grammar:

```text
field_name:type[{constraints}]
```

Rules:

* Field names use `lower_case_underscored`.
* Types are mandatory.
* Constraints are optional.
* `[Fields]` describes structure, parameters, inputs, configuration shape, or schema members.

---

### 5.6 `[Behavior]`

`[Behavior]` contains a comma-separated list of descriptors.

Two forms are allowed:

```text
single_use
effect:writes_state
```

Behavior item grammar:

```text
descriptor
descriptor:value
descriptor:value{constraints}
```

Rules:

* Use bare atoms for simple facts: `single_use`, `async`, `idempotent`, `cacheable`.
* Use keyed descriptors when the target matters: `effect:writes_state`, `auth:bearer`.
* Behavior SHOULD be specific and non-redundant with the entry name.
* Behavior MAY use quoted values when needed.
* Registered relation roles MUST NOT be encoded in `[Behavior]` (see § 5.4).

Example:

```text
[Behavior]sample:"Price, with VAT"
```

See § 11.3 for the registered universal behavior vocabulary.

---

### 5.7 `[Returns]`

`[Returns]` uses the same grammar as `[Fields]`:

```text
[Returns]temp_password:text,link:url{when_create_link_only,server_side_only}
```

Rules:

* Return names use `lower_case_underscored`.
* Types are mandatory.
* Conditional returns use constraints, not custom blocks.

Correct:

```text
[Returns]link:url{when_create_link_only}
```

Incorrect:

```text
[Returns_when_create_link]link:url
```

---

### 5.8 Facet Decomposition

When one logical subject has multiple wide orthogonal surfaces, split it into multiple entries
using `facet:` instead of inventing new block names.

Example:

```text
!Button|cat:type|id:ui.button|kind:component|facet:appearance|[Fields]label:expr<text>,style:style_ref,opacity:number
!Button|cat:type|id:ui.button|kind:component|facet:layout|[Fields]visible_on_load:bool,width:dim,height:dim,x:number,y:number
!Button|cat:type|id:ui.button|kind:component|facet:states|[Returns]is_hovered:bool,is_pressed:bool,is_visible:bool
!Button|cat:type|id:ui.button|kind:component|facet:conditional|[Behavior]last_match_wins,any_conditionable_property
```

Use `facet:` when:

* the entry would otherwise become too wide
* the subject has clearly distinct domains such as `appearance`, `layout`, `states`,
  `conditional`, `runtime`, `privacy`, `req`, `res_200`, `err_404`

Do not use custom named blocks to achieve the same effect.

`facet:` MUST NOT be used for child containment. Fields, methods, and overloads are child
subjects with their own identities, not facets of the parent subject.

Forbidden facet names (child-as-facet pattern):

* `facet:fields`
* `facet:methods`
* `facet:overloads`

---

### 5.9 Identity and Uniqueness

The rules in this subsection apply only to entries that carry `id:`.

* Core-file conformance: each `(id, facet-or-empty)` tuple MUST be unique within the file.
* Machine-first profile-file conformance: each `(id, facet-or-empty)` tuple MUST be unique
  within the file.
* Machine-first profile-corpus conformance: each `(id, facet-or-empty)` tuple MUST be unique
  across the validation corpus.
* The bare `id` identifies the logical subject across the validation corpus.
* Multiple entries MAY share the same `id` only when each such entry has a distinct non-empty
  `facet:` and all such entries share the same `cat:` and `kind:`.
* An entry with no `id:` has no machine-stable internal address and MUST NOT be the target of
  an internal relation.
* IDs are not globally unique across the internet.
* IDs are not merely file-unique in machine-first use.
* IDs are corpus-logically unique across the validation corpus.

---

## 6. Type System

### 6.1 Core Scalar Types

Core scalar types:

* `text`
* `number`
* `int`
* `bool`
* `date`
* `url`
* `image`
* `any`

`any` means type-known-to-exist but not constrained by the current entry.

---

### 6.2 Generics

| Form        | Meaning                                   |
| ----------- | ----------------------------------------- |
| `expr`      | Dynamic expression of unknown result type |
| `expr<T>`   | Dynamic expression returning `T`          |
| `list<T>`   | Ordered collection of `T`                 |
| `record<T>` | Reference to a record of entity `T`       |
| `map<K,V>`  | Key-value mapping from `K` to `V`         |

Nested generics are allowed:

* `list<list<text>>`
* `expr<list<record<User>>>`
* `map<text,list<number>>`

Ghost forbids nested **named blocks**, not nested **types**.

---

### 6.3 References

Names ending in `_ref` denote references to other first-class objects.

Examples:

* `page_ref`
* `view_ref`
* `element_ref`
* `event_ref`
* `workflow_ref`
* `type_ref`
* `field_ref`
* `state_ref`
* `input_ref`
* `rg_ref`
* `map_ref`
* `table_ref`
* `function_ref`
* `access_ref`
* `style_ref`

Use `_ref` when the target is a first-class platform object and a generic `record<T>` would
be less readable.

Note: `_ref` targets inside `[Fields]` / `[Returns]` describe type shape. Stable graph edges
between first-class subjects belong in `[Rel]`.

---

### 6.4 Compound Types

| Form              | Meaning                 |
| ----------------- | ----------------------- |
| `enum{a/b/c}`     | One-of literal values   |
| `field_changes`   | Field-change tuple set  |
| `key_value_pairs` | Arbitrary KV map        |
| `params`          | Free-form parameter bag |

Use `params` only when the structure is intentionally loose or externally imposed.

---

### 6.5 Unions

Pipes inside a type position denote union:

```text
navigate_to:page_ref|view_ref
```

Rules:

* Use unions only inside type positions.
* If the union visually collides with block boundaries, prefer:

  * `list<page_ref|view_ref>`
  * splitting the field into two fields
  * a dialect-specific named type

---

### 6.6 Dialect Type Extensions

Dialects MAY extend the type system.

Canonical extensions:

| Domain      | Types                                                                 |
| ----------- | --------------------------------------------------------------------- |
| UI          | `color`, `dim`, `border`, `shadow`, `spacing`, `font`                 |
| Rust/SQLite | `struct_ref`, `trait_ref`, `crate_ref`, `sql_type`, `migration_ref`   |
| HTTP/API    | `endpoint_ref`, `header_ref`, `status_code`, `json_path`, `mime_type` |
| CLI         | `flag`, `subcommand_ref`, `path`, `glob`, `env_var`                   |
| Config      | `config_key`, `section_ref`, `env_ref`                                |

A dialect MUST document the types it adds.

---

## 7. Constraints

Constraints attach to a field, return, or relation item using `{}`:

```text
number{0_to_24,default:1,hours}
expr<text>{server_side_only}
response:http.orders.get.res_200{when_if_none_match_absent}
```

Constraints are comma-separated atoms.

---

### 7.1 Canonical Constraint Patterns

| Pattern               | Meaning                           | Example                     |
| --------------------- | --------------------------------- | --------------------------- |
| `when_<X>`            | Conditional visibility or meaning | `{when_change_email}`       |
| `default:<V>`         | Default value                     | `{default:1}`               |
| `requires_<X>`        | Requires another field or state   | `{requires_include_labels}` |
| `deprecated_<reason>` | Deprecated                        | `{deprecated_Jul_2017}`     |
| `max_<N>`             | Maximum value or count            | `{max_50}`                  |
| `min_<N>`             | Minimum value or count            | `{min_1}`                   |
| `<N>_to_<M>`          | Inclusive numeric range           | `{0_to_24}`                 |
| `<unit>`              | Unit annotation                   | `{seconds}`, `{pixels}`     |
| `server_side_only`    | Scope restriction                 | `{server_side_only}`        |
| `single_use`          | Usage restriction                 | `{single_use}`              |

---

### 7.2 Units

Canonical unit atoms:

* `seconds`
* `ms`
* `minutes`
* `hours`
* `days`
* `pixels`
* `em`
* `rem`
* `percent`
* `MB`
* `KB`
* `bytes`
* `degrees`

---

## 8. Escaping and Quoting

Reserved characters inside values MUST be escaped or quoted.

Reserved characters:

```text
| , [ ] { } \ "
```

---

### 8.1 Backslash Escapes

Supported escapes:

```text
\|   \,   \[   \]   \{   \}   \\   \"
```

---

### 8.2 Quoted Strings

Wrap values in double quotes when they contain multiple reserved characters or free text.

Example:

```text
!Quoted_Value_Demo|cat:example|kind:snippet|[Behavior]sample:"Price, with VAT"
```

Inside quotes, only `"` and `\` MUST be escaped.

Quoted and unquoted values are semantically equivalent.

Use whichever is shorter and clearer.

---

## 9. Annotations

### 9.1 `#note:`

Informational:

* tips
* quirks
* cross-references
* non-dangerous edge cases

Example:

```text
|#note:signup_implicitly_calls_Opt_in_to_cookies
```

---

### 9.2 `#warn:`

Risk-bearing:

* destructive operations
* irreversibility
* security implications
* hidden side effects

Example:

```text
|#warn:original_password_irrecoverable
```

---

### 9.3 Ordering

Within an entry:

```text
#note → #warn
```

Rules:

* All `#note:` annotations MUST come before all `#warn:` annotations.
* Multiple notes and warnings are allowed.

---

## 10. Naming

### 10.1 Sections

Sections use:

```text
UPPER_CASE_UNDERSCORED
```

Example:

```text
$AUTH_SIGNUP_LOGIN
```

---

### 10.2 Entry Names

Entry names SHOULD preserve source naming where possible.

Two canonical styles are allowed:

1. **Mixed_Case_Underscored** — for product-visible names
   Example: `!Sign_the_user_up`

2. **lower_case_underscored** — for schema or internal identifiers
   Example: `!dom_print_job`

Rules:

* Preserve source identity when that improves fidelity.
* Use one style consistently within the same semantic family.
* Spaces become underscores.
* `>` MAY appear inside entry names to mirror a source hierarchy.

Example:

```text
!Settings>General>Redirect
```

---

### 10.3 `id:` Values

`id:` values use dot-segmented identifiers:

```text
id:java.com.acme.order.OrderService
id:http.orders.get
id:wf.ingest.validate
```

Rules:

* `id:` values MUST NOT contain `#`.
* `id:` values SHOULD be stable across refactors.
* `id:` values SHOULD reflect corpus-level uniqueness conventions, typically
  `<profile-prefix>.<namespace>.<subject>`.

---

### 10.4 Field Names

Field names use:

```text
lower_case_underscored
```

Example:

```text
email_to_reset
```

---

### 10.5 Meta Keys

Meta keys use:

```text
lower_case:
```

Examples:

* `cat:`
* `id:`
* `kind:`
* `facet:`
* `scope:`

---

### 10.6 Types

Primitive and generic type keywords use lowercase:

* `text`
* `list<T>`
* `expr<T>`

Referenced entity / type names inside generics MAY preserve source casing:

* `record<User>`
* `record<dom_print_job>`

---

### 10.7 Reserved Identifiers

Reserved identifiers include:

* meta keys defined in § 5.3
* named blocks `[Rel]`, `[Fields]`, `[Behavior]`, `[Returns]`
* annotation prefixes `#note:` and `#warn:`

---

## 11. Core Vocabularies

### 11.1 `cat:` values

Normative.

The following are the registered universal `cat:` values:

| `cat:`       | Meaning                                             |
| ------------ | --------------------------------------------------- |
| `module`     | package / module / namespace unit                   |
| `type`       | nominal or structural type-like subject             |
| `member`     | field / property / constant-like child subject      |
| `callable`   | function / method / ctor / hook-like subject        |
| `source`     | provider / input / origin / store subject           |
| `schema`     | database / schema / table / view / index / constraint subject |
| `endpoint`   | externally or logically addressable API surface     |
| `contract`   | request / response / error / payload surface        |
| `event`      | event / hook / stream subject                       |
| `state`      | machine state subject                               |
| `transition` | state transition subject                            |
| `step`       | workflow / protocol step subject                    |
| `example`    | example / golden / sample guidance subject          |
| `test`       | executable or declarative test guidance subject     |
| `metric`     | named metric or SLO subject                         |

Rules:

* This registry is a reserved interoperability vocabulary. It is open for core-file
  conformance and closed only under an active machine-first profile.
* Core-file validators MUST NOT reject an entry solely because its `cat:` value is not in the
  registered universal list.
* Profiles MAY register subsets of the universal list.
* Dialects MAY add additional `cat:` values only if the active profile explicitly permits
  dialect-added categories.
* Profiles and dialects MUST NOT redefine the meaning of a registered universal `cat:` value.
* Machine-first profile validators MUST reject any `cat:` value not registered by the active
  profile, plus any dialect additions explicitly permitted by that profile (E31).

---

### 11.2 Universal Relation Roles

Normative.

The following are the registered universal relation roles:

| Role                | Meaning                                            |
| ------------------- | -------------------------------------------------- |
| `member_of`         | direct addressable child belongs to owner          |
| `part_of`           | structural containment weaker than `member_of`     |
| `owns`              | forward ownership / composition edge               |
| `imports`           | module imports target module / symbol              |
| `exports`           | module exports target symbol                       |
| `depends_on`        | subject requires target for operation or existence |
| `references`        | subject refers to target without ownership         |
| `extends`           | nominal extension / subclassing                    |
| `implements`        | nominal interface implementation                   |
| `conforms_to`       | structural / protocol conformance                  |
| `annotated_by`      | target annotation / decorator / attribute applies  |
| `calls`             | subject invokes target callable / endpoint / step  |
| `reads`             | subject reads from target state / resource         |
| `writes`            | subject writes to target state / resource          |
| `throws`            | subject may raise or emit target error type        |
| `request`           | subject uses target request contract               |
| `response`          | subject uses target response contract              |
| `error`             | subject uses target error contract                 |
| `from`              | transition source state                            |
| `to`                | transition destination state                       |
| `next`              | successful continuation step                       |
| `on_fail`           | failure continuation step                          |
| `emits`             | subject emits target event                         |
| `consumes`          | subject consumes target event                      |
| `for`               | example / test applies to target subject           |
| `compatible_with`   | positive compatibility edge                        |
| `incompatible_with` | negative compatibility edge                        |
| `supersedes`        | newer subject supersedes older one                 |
| `replaced_by`       | deprecated / removed subject replacement           |

Rules:

* This registry is a reserved interoperability vocabulary. It is open for core-file
  conformance and closed only under an active machine-first profile.
* Core-file validators MUST treat any syntactically valid `lower_case_underscored` role as
  syntactically valid.
* Machine-first profile validators MUST reject any relation role not registered by the active
  profile or by an active dialect explicitly permitted by that profile (E23).
* Profiles and dialects MUST NOT redefine the meaning of a registered universal relation role.

---

### 11.3 Universal Behavior Vocabulary

Normative.

Behavior vocabulary is open in core-file conformance and closed at the behavior-key /
bare-atom level in machine-first profiles.

A behavior item is valid in a machine-first profile only if it is one of the following:

1. a registered universal bare atom;
2. a registered universal keyed descriptor;
3. a profile-registered bare atom or keyed descriptor; or
4. a dialect-registered bare atom or keyed descriptor permitted by the active profile.

The value part of a registered keyed descriptor is opaque unless the active profile or
dialect constrains it.

Registered universal bare atoms:

```text
async
awaitable
generator
streaming
concurrent
ordered
initial
terminal
hook
single_use
retriable
handles_pii
audited
destructive
safe
idempotent
cacheable
```

Registered universal keyed descriptors:

```text
vis:public|protected|private|internal
mut:mutable|readonly|immutable
bind:instance|static|class
effect:pure|reads_state|writes_state|transactional|io|network|cpu_bound|io_bound
pre:<atom>
post:<atom>
inv:<atom>
auth:none|basic|bearer|mtls
requires_role:<atom>
complexity:<atom>
latency_p50_ms:<N>
latency_p95_ms:<N>
throughput_rps:<N>
cost:<atom>
```

Rules:

* `pre:`, `post:`, and `inv:` values SHOULD be short, reusable atoms.
* Profiles or dialects MAY register additional behavior atoms or keys.
* Core-file validators without an active machine-first profile MUST NOT reject unknown
  behavior items solely for being unregistered.
* Machine-first profile validators MUST reject any behavior atom or behavior key not
  registered by the active profile, plus any dialect additions explicitly permitted by that
  profile (E32).

---

## 12. Profiles

A profile is a named governance layer declared on header line 1:

```text
# orders-api.ghost v2.0 profile=api_contract
```

Profiles are the enforcement layer of Ghost v2. A core v2 file without `profile=` is valid
but lenient. A file with `profile=` activates machine-first validation and closes the
applicable vocabularies.

### 12.1 What a profile defines

A profile MUST define:

* registered `cat:` values
* allowed `kind:` values per `cat:`
* registered relation roles (and their cardinality, where not the default `0..n`)
* registered ext meta keys
* registered behavior atoms and keyed descriptors (drawn from the universal behavior
  vocabulary unless the profile explicitly extends it)
* profile-file minimum rules
* decomposition rules
* forbidden patterns
* minimum validator checks

A profile MAY permit one or more dialects. A dialect activated without profile permission
MUST NOT add `cat:` values, relation roles, ext meta keys, or behavior vocabulary that the
profile treats as closed.

A profile MUST NOT:

* relax the core grammar
* change canonical block order
* introduce new named blocks
* redefine the meaning of registered universal `cat:` values, relation roles, or behavior
  atoms / keys

### 12.2 Standard profile: `code_surface`

Normative.

**Purpose**
Compact semantic surfaces for source code and SDKs.

**Registered categories**
`module`, `type`, `member`, `callable`
Optional categories: `example`, `test`

**Profile-file minimum**
A conforming file MUST contain at least one entry whose `cat:` is one of the registered or
optional categories. A file MAY omit any registered category that is absent from the
documented surface.

**Allowed / recommended kinds**

* `module`: `package`, `module`, `namespace`
* `type`: `class`, `interface`, `protocol`, `trait`, `struct`, `enum`, `record`, `alias`,
  `annotation`
* `member`: `field`, `property`, `constant`
* `callable`: `function`, `method`, `ctor`, `initializer`, `hook`, `operator`, `dtor`
* `example`: `snippet`, `golden`
* `test`: `positive`, `negative`, `roundtrip`

**Registered ext meta keys**
`since`, `sunset`, `until`

**Registered relation roles**
`member_of`, `imports`, `exports`, `depends_on`, `references`, `extends`, `implements`,
`conforms_to`, `annotated_by`, `calls`, `reads`, `writes`, `throws`, `for`,
`compatible_with`, `incompatible_with`, `supersedes`, `replaced_by`

**Required ID policy**
Every entry MUST have `id:`. The recommended ID style is dot-segmented and prefixed by the
source language or ecosystem (`java.`, `cpp.`, `py.`, `ts.`, etc.). A dialect MAY strengthen
this SHOULD to MUST.

**Decomposition rules**

* Each independently addressable type SHOULD be its own entry.
* Each independently addressable member MUST be its own entry.
* Each independently addressable callable MUST be its own entry.
* A type entry MUST NOT inline its method set as `[Fields]` or `[Behavior]`.
* `facet:` MAY be used for orthogonal projections such as `compat`, `runtime`, `security`,
  `examples`. It MUST NOT be used for `fields`, `methods`, or `overloads`.

**Relation-target rules**

* `member_of` target MUST be a `module` or `type`.
* `extends` / `implements` / `conforms_to` source MUST be `type`.
* `throws` source MUST be `callable`.
* Internal targets MUST be `id` or `id#facet`.
* `ext:` targets MAY be used for any registered role when the target subject lies outside the
  active validation corpus.

**Forbidden patterns**

* `facet:methods`, `facet:fields`, `facet:overloads`
* relation roles encoded in `[Behavior]`
* same `id:` reused for incompatible callable signatures
* bare entry-name targets in `[Rel]`
* mega-type lines enumerating members as prose or descriptors

**Minimum validator checks**

* every entry has `cat/id/kind`
* `member` and `callable` entries MUST have exactly one `member_of`
* same `id` across facets MUST preserve `cat` and `kind`
* only registered kinds are used
* only registered roles are used
* `throws` only on `callable`
* `extends` / `implements` / `conforms_to` only on `type`

### 12.3 Standard profile: `api_contract`

Normative.

**Purpose**
Compact API endpoint and contract surfaces.

**Registered categories**
`endpoint`, `contract`
Optional categories: `example`, `test`

**Profile-file minimum**
A conforming file MUST contain at least one `endpoint` or `contract` entry. A file MAY
contain reusable standalone `contract` entries without an `endpoint`.

**Allowed / recommended kinds**

* `endpoint`: `http`, `rpc`, `webhook`
* `contract`: `request`, `response`, `error`, `headers`
* `example`: `snippet`, `golden`
* `test`: `positive`, `negative`, `roundtrip`

**Registered ext meta keys**
`path`, `verb`, `since`, `sunset`, `until`

**Registered relation roles**
`depends_on`, `request`, `response`, `error`, `emits`, `for`, `compatible_with`,
`incompatible_with`, `supersedes`, `replaced_by`

**Required ID policy**
Every entry MUST have `id:`.

**Decomposition rules**

* An endpoint family MUST choose exactly one of the following identity models:

  1. **linked-contract form**
  2. **endpoint-facet form**

* In linked-contract form, request / response / error surfaces are separate `contract`
  entries with distinct `id:` values.

* In endpoint-facet form, request / response / error surfaces are `endpoint` entries that
  share the base endpoint `id:` and `kind:` and differ only by `facet:`.

* Endpoint facet names MUST use `req`, `headers`, `res_<token>`, or `err_<token>`.

* The two identity models MUST NOT be mixed within one endpoint family.

* Every base endpoint entry MUST declare at least one `[Rel]response:` item.

* Requests MAY be omitted only if there is no request body, parameter, or header surface
  worth representing.

* Headers MUST be represented in `[Fields]`, `[Returns]`, or `contract|kind:headers`. No extra
  blocks are allowed.

* Reused request / response / error surfaces SHOULD use linked-contract form rather than
  duplicated endpoint facets.

**Relation-target rules**

* `request`, `response`, and `error` MUST target either:

  * `contract` entries, when linked-contract form is used; or
  * `id#facet` endpoint projections, when endpoint-facet form is used.

* If endpoint-facet form is used, each facet entry MUST use:

  * `cat:endpoint`
  * the same `id:` as the base endpoint
  * the same `kind:` as the base endpoint
  * a distinct `facet:`

* Base endpoint and endpoint facet entries MUST carry identical `path:` and `verb:` values.

* `path:` and `verb:` are REQUIRED on every `endpoint` entry, including endpoint facet
  entries.

* `status:` MUST NOT carry HTTP response codes.

**Forbidden patterns**

* using `status:404` on an entry
* mixing linked-contract and endpoint-facet form within the same endpoint family
* same `id:` shared between `endpoint` and `contract` entries
* using response codes as behavior descriptors
* mixing multiple status-specific response payloads into one response surface

**Minimum validator checks**

* every `endpoint` has `path:` and `verb:`
* every base endpoint has at least one `response` relation
* `request` / `response` / `error` targets resolve to `contract` entries or endpoint facets
  according to the chosen identity model
* endpoint facets preserve the base endpoint `id:`, `kind:`, `path:`, and `verb:`
* an endpoint family does not mix the two identity models
* `contract` entries use only registered kinds
* internal relation targets are `id` or `id#facet`
* only registered roles are used

### 12.4 API endpoint patterns

Normative.

Pattern:

* one base `endpoint` entry per endpoint identity
* one or more request / response / error surfaces represented using exactly one of:

  * linked-contract form; or
  * endpoint-facet form

* the base endpoint entry uses `[Rel]request`, `[Rel]response`, and `[Rel]error`
* `verb:` and `path:` are endpoint ext meta keys
* security / idempotency / cache semantics go in `[Behavior]`

Do not add `[Headers]`, `[Status]`, or `[Errors]` blocks.

### 12.5 Request / response / error contracts

Normative.

Two valid compact forms exist:

1. **Linked-contract form**
   Separate `contract` entries with distinct IDs. This form is preferred for reuse.

2. **Endpoint-facet form**
   Separate `endpoint` entries sharing the base endpoint `id:` and `kind:`, distinguished
   only by `facet:`. This form is preferred only when the surfaces are tightly local to one
   endpoint family.

Rules:

* A single endpoint family MUST use exactly one of these two forms.
* If multiple status variants exist, they MUST be represented as separate response / error
  surfaces.
* In linked-contract form, distinct status variants are separate `contract` IDs.
* In endpoint-facet form, distinct status variants are separate `res_<token>` /
  `err_<token>` facets.
* HTTP status numbers belong in contract identity or facet naming, not `status:` meta.

### 12.6 Headers and status contracts

Normative.

* Headers belong in `[Fields]` for request surfaces or `[Returns]` for response surfaces.
* Shared header sets MAY use `contract|kind:headers` in linked-contract form or
  `facet:headers` in endpoint-facet form.
* Status variants are modeled as separate response / error surfaces in the chosen identity
  model.
* Status variants MUST NOT be encoded as behavior flags or `status:` meta values.

### 12.7 Profile-level vocabulary scope

The standard profile pack closes `cat:`, `kind:`, relation-role, ext-meta, and
behavior-key / bare-atom vocabularies.

The standard profile pack standardizes:

* allowed `kind:` values per `cat:`
* allowed relation roles per profile
* optional ext meta keys:

  * all standard profiles: `since`, `sunset`, `until`
  * `api_contract`: additionally `path`, `verb`

* behavior items from the universal behavior vocabulary

The standard profile pack does not add profile-specific behavior items beyond the universal
behavior vocabulary. Additional behavior items, if any, MUST come from an active dialect that
the profile permits.

The standard profile pack does not standardize:

* business-domain state names
* algorithm names
* vendor-specific auth schemes beyond the generic `auth:*` key
* long `pre:` / `post:` / `inv:` atom catalogs

Those belong to profile-specific or dialect-specific vocabularies.

---

## 13. Dialects

A dialect is a named vocabulary extension declared on header line 1:

```text
# autoprint-schema.ghost v2.0 dialect=schema_catalog
```

A dialect MAY:

* extend the type system (see § 6.6)
* define additional meta keys
* register additional `cat:` values, relation roles, and behavior vocabulary, subject to
  active profile permission
* recommend standard sections
* recommend `facet:` vocabularies
* add validation rules on top of profile validation

A dialect MUST NOT:

* relax the core grammar
* change canonical block order
* introduce new named blocks
* redefine the meaning of a registered universal `cat:` value, relation role, or behavior
  atom / key

Dialects without an active `profile=` declaration MAY be used in core-file conformance for
informative extension. Under a machine-first profile, dialect vocabulary is valid only if
the active profile explicitly permits that dialect.

### 13.1 Registered Dialects

| Dialect                   | Purpose                                                |
| ------------------------- | ------------------------------------------------------ |
| `documentation_reference` | Vendor, API, CLI, SDK documentation                    |
| `platform_metamodel`      | Platform surface area, reverse-engineered applications |
| `schema_catalog`          | Database schema blueprints and metamodel catalogs      |

### 13.2 Defining a New Dialect

A dialect addendum MUST specify:

* dialect name
* version
* purpose
* added types
* added meta keys
* added `cat:` values (if any)
* added relation roles (if any)
* added behavior vocabulary (if any)
* which profiles, if any, permit it
* recommended sections
* one canonical example

---

## 14. Validation

A producer SHOULD run this checklist before delivering a `.ghost` file.

### 14.1 Required Checks

1. Header line 4 `Entries:` count matches the actual number of entry lines beginning with any
   defined sigil.
2. No empty block markers.
3. Canonical block order is respected.
4. Only `[Rel]`, `[Fields]`, `[Behavior]`, `[Returns]` appear as named blocks.
5. Every entry begins its meta with `cat:`.
6. No blank lines between entries within a section.
7. One physical line per entry.
8. Section names are `UPPER_CASE_UNDERSCORED`.
9. Entry names follow one of the allowed naming conventions and are consistent with source
   naming.
10. No pedagogical prose remnants inside entries.
11. Constraints use `{}`, not `()`.
12. Pipes separate blocks and meta fields; commas separate items within blocks.
13. `#note:` precedes `#warn:` in every entry.
14. Reserved sigils `?` and `%` are not used.

### 14.2 Errors (Non-conformant)

| Code | Description |
| ---- | ----------- |
| E1   | Header missing or malformed |
| E2   | Header entry count mismatch |
| E3   | Entry missing `cat:` |
| E4   | Non-canonical block order |
| E5   | Ad-hoc block name |
| E6   | Empty block marker |
| E7   | Entry split across multiple physical lines |
| E8   | Invalid section name |
| E9   | Tab character in file |
| E10  | Parentheses used for constraints instead of `{}` |
| E11  | `#warn:` appears before `#note:` |
| E12  | Entry appears before first section |
| E13  | Blank line between entries in a section |
| E14  | Missing blank line after header or between sections |
| E15  | Repeated or malformed `dialect=` / `profile=` declarations on header line 1 |
| E16  | Invalid canonical meta order or relation-item order for a machine-first profile |
| E17  | Missing `id:` or `kind:` in a machine-first profile |
| E18  | Duplicate `(id, facet)` among entries carrying `id:` within the active validation scope |
| E19  | `facet:` used without `id:` in a machine-first profile |
| E20  | Invalid `id:` grammar or forbidden `#` inside `id:` |
| E21  | Invalid `[Rel]` target grammar |
| E22  | Unresolved internal relation target in profile-corpus validation |
| E23  | Unregistered relation role under the active profile / dialect |
| E24  | Role cardinality or required role occurrence violated |
| E25  | Same `id:` reused with different `cat:` or `kind:` across facets |
| E26  | Reserved sigil `?` or `%` used without an activating specification |
| E27  | Relation encoded in `[Behavior]` using a registered relation role |
| E28  | Unregistered `kind:` for the declared `cat:` under the active profile / dialect |
| E29  | Unregistered ext meta key under the active profile / dialect |
| E30  | Profile-specific source / target / meta / facet-consistency rule violated |
| E31  | Unregistered `cat:` under the active profile / dialect |
| E32  | Unregistered behavior atom or behavior key under the active profile / dialect |

Error-scope rules:

* `E16`, `E17`, `E18`, `E19`, `E22`, `E23`, `E24`, `E25`, `E28`, `E29`, `E30`, `E31`, and
  `E32` apply only when an active machine-first profile is being validated.
* `E18` applies only to entries that actually carry `id:`.
* `E26` applies in all conformance layers, core and profile alike.

### 14.3 Warnings (Conformant but Discouraged)

| Code | Description |
| ---- | ----------- |
| W1   | Pedagogical prose remnants |
| W2   | Duplicate entry name without ID collision |
| W3   | Unknown type not in core or declared dialect |
| W4   | Unresolved internal relation target in file-only validation |
| W5   | Oversized entry likely better split by facet or child entry |
| W6   | Overuse of `params` where stronger typing is possible |
| W7   | Section contains a single entry |
| W8   | `facet:` used in a way that looks like child containment (`facet:fields`, `facet:methods`, etc.) |
| W9   | Deprecated or removed entry lacks `replaced_by` when a likely successor exists |
| W10  | Non-canonical relation item ordering in non-machine-first or advisory validation |
| W11  | Non-canonical ext meta key ordering in non-machine-first or advisory validation |
| W12  | Long quoted `pre:` / `post:` / `inv:` value likely drifting into prose |

Warning-scope rules:

* `W4` is file-only / excerpt-only; the corresponding corpus-level failure is `E22`.
* `W10` and `W11` are advisory in non-machine-first or advisory validation; machine-first
  ordering failures are `E16`.

Oversized-entry policy:

* Default warning threshold in machine-first profiles:

  * line length > 320 UTF-8 bytes; or
  * any named block contains > 12 items

* Profiles MAY define a lower threshold. They SHOULD NOT define a higher one without
  benchmarking evidence.

A consumer SHOULD flag warnings but continue reading.

### 14.4 Profile Conformance Checks

A machine-first profile validator MUST check:

* required `profile=` exists
* `cat:` / `id:` / `kind:` are present on every entry
* canonical meta order
* canonical relation-item ordering
* registered `cat:` values only
* registered `kind:` values only
* registered relation roles only
* registered ext meta keys only
* registered behavior atoms / keys only
* relation source / target compatibility
* whether `ext:` targets are permitted for the role under the active profile / dialect
* profile-specific role cardinalities, with unspecified role cardinality defaulting to `0..n`
* uniqueness of `(id, facet)` among entries that carry `id:`
* same `id:` across facets preserves `cat:` and `kind:`
* file or corpus resolution of internal relation targets as applicable
* reserved sigils are not used unless an activating specification exists

---

## 15. Anti-patterns

| Code | Forbidden                                       | Fix                                             |
| ---- | ----------------------------------------------- | ----------------------------------------------- |
| F1   | `[Fields]` empty or `no_fields`                 | Omit the block                                  |
| F2   | Returns before Fields                           | Reorder                                         |
| F3   | YAML / JSON inside a block                      | Flatten into pipe / comma form                  |
| F4   | Entry wrapped across lines                      | Keep on one physical line                       |
| F5   | Prose like “Usually you will want to…”          | Delete                                          |
| F6   | `number(0-24)`                                  | `number{0_to_24}`                               |
| F7   | Lowercase section name                          | Use `UPPER_CASE_UNDERSCORED`                    |
| F8   | Entry with no `cat:`                            | Add `cat:<value>`                               |
| F9   | `[Options]`, `[Params]`, `[Config]`, `[Output]`, `[Headers]`, `[Status]`, `[Errors]` | Use `[Fields]`, `[Returns]`, or split by facet |
| F10  | Blank line between entries                      | Remove                                          |
| F11  | Nested named blocks                             | Flatten                                         |
| F12  | `#warn:` before `#note:`                        | Reorder                                         |
| F13  | Emoji or decorative glyphs inside entries       | Strip                                           |
| F14  | Tabs                                            | Use spaces only                                 |
| F15  | Trailing `.` or `;` on entries                  | Strip                                           |
| F16  | Relation role encoded in `[Behavior]`           | Move to `[Rel]`                                 |
| F17  | Bare entry-name or prose target in `[Rel]`      | Use `id`, `id#facet`, or `ext:<ref>`            |
| F18  | Reused `id:` across incompatible signatures    | Give each a distinct `id:`                      |
| F19  | `facet:fields`, `facet:methods`, `facet:overloads` | Create child entries with their own `id:`     |
| F20  | HTTP status code in `status:` meta              | Use separate response / error surfaces          |
| F21  | Mixing linked-contract and endpoint-facet form in one endpoint family | Choose one form and keep it |

---

## 16. Correct / Incorrect Examples

Normative.

1. **Coarse class vs subtype**

   * Correct: `cat:type|kind:class`
   * Incorrect: `cat:class|kind:type`
   * Why: `cat:` names the coarse semantic class. `kind:` names the subtype of that class.

2. **Linked-contract form vs endpoint-facet form**

   * Correct, endpoint-facet form: `cat:endpoint|id:http.orders.get|kind:http|facet:req`
   * Correct, linked-contract form: `cat:contract|id:http.orders.get.req|kind:request`
   * Incorrect: `cat:contract|id:http.orders.get|kind:request` when the intent is to model a
     facet of the same endpoint subject
   * Why: same-`id` facet form keeps the same `cat:` and `kind:` as the base endpoint.
     Distinct `contract` form requires a distinct `id:`.

3. **Subtype vs projection**

   * Correct: `cat:type|id:ui.button|kind:component|facet:layout`
   * Incorrect: `cat:type|id:ui.button|kind:layout`
   * Why: `layout` is a projection of the same subject, not a subtype of the subject.

4. **Child subjects are not facets**

   * Correct: `cat:type|id:java.User|kind:class` and
     `cat:member|id:java.User.name|kind:field|[Rel]member_of:java.User`
   * Incorrect: `cat:type|id:java.User|kind:class|facet:fields`
   * Why: fields are child subjects with their own identities, not facets of the class
     subject.

5. **Facets share one identity**

   * Correct: same `id:` across `facet:appearance` and `facet:layout`
   * Incorrect: new `id:` for each facet
   * Why: `facet:` addresses projections of one logical subject.

6. **Replacement is new identity**

   * Correct: old entry uses `status:deprecated|[Rel]replaced_by:new.id`
   * Incorrect: same `id:` reused for an incompatible replacement
   * Why: replacement changes subject identity. It is not a rename and not a facet.

7. **Transition is not a state**

   * Correct: `cat:transition|kind:transition|[Rel]from:...,to:...`
   * Incorrect: `cat:state|kind:transition`
   * Why: the coarse semantic class changed.

---

## 17. Examples

### 17.1 Minimal Valid Core File

```text
# minimal.ghost v2.0
# Purpose: Smallest conformant Ghost file.
# Source: authored
# Entries: 1

$ROOT
!Ping|cat:callable|[Behavior]safe|[Returns]pong:bool
```

### 17.2 Core v1.1 Compatibility File

```text
# bubble-auth.ghost v2.0 dialect=documentation_reference
# Purpose: Bubble.io auth actions.
# Source: https://manual.bubble.io
# Entries: 3

$AUTH_SIGNUP_LOGIN
+Sign_the_user_up|cat:auth|[Fields]email:expr<text>,password:expr<text>,type_of_user:type_ref{default:User},other_fields:field_changes|[Behavior]creates_user,auto_login,triggers_event:User_is_signed_up|#note:equivalent_to_Create_a_new_thing_plus_Log_the_user_in
+Log_the_user_in|cat:auth|[Fields]email:expr<text>,password:expr<text>,stay_logged_in:bool{default:true}|[Behavior]creates_session,triggers_event:User_is_logged_in|#note:respects_Settings>General>Cookie_duration
+Log_the_user_out|cat:auth|[Behavior]triggers_event:Current_user_is_logged_out,clears_session
```

### 17.3 `code_surface` — Java

```text
# java-order.ghost v2.0 profile=code_surface
# Purpose: Java order service surface.
# Source: authored
# Entries: 5

$JAVA__COM_ACME_ORDER
!com_acme_order|cat:module|id:java.com.acme.order|kind:package
!Order|cat:type|id:java.com.acme.order.Order|kind:class|[Rel]member_of:java.com.acme.order|[Behavior]vis:public
!OrderLookup|cat:type|id:java.com.acme.order.OrderLookup|kind:interface|[Rel]member_of:java.com.acme.order|[Behavior]vis:public
!OrderService|cat:type|id:java.com.acme.order.OrderService|kind:class|[Rel]member_of:java.com.acme.order,implements:java.com.acme.order.OrderLookup,annotated_by:ext:java.jakarta.Inject|[Behavior]vis:public
^find|cat:callable|id:java.com.acme.order.OrderService.find(text)|kind:method|[Rel]member_of:java.com.acme.order.OrderService,throws:ext:java.com.acme.order.NotFound|[Fields]order_id:text|[Behavior]effect:reads_state,vis:public|[Returns]order:record<Order>
```

### 17.4 `code_surface` — C++

```text
# cpp-raster.ghost v2.0 profile=code_surface
# Purpose: C++ raster job surface.
# Source: authored
# Entries: 4

$CPP__ACME_PRINT
!acme_print|cat:module|id:cpp.acme.print|kind:namespace
!RasterJob|cat:type|id:cpp.acme.print.RasterJob|kind:struct|[Rel]member_of:cpp.acme.print|[Behavior]mut:mutable,vis:public
!Options|cat:type|id:cpp.acme.print.Options|kind:struct|[Rel]member_of:cpp.acme.print|[Behavior]vis:public
^submit|cat:callable|id:cpp.acme.print.RasterJob.submit(text+Options)|kind:method|[Rel]member_of:cpp.acme.print.RasterJob,depends_on:ext:cpp.acme.io.Spooler|[Fields]file:text,opts:record<Options>|[Behavior]effect:io,vis:public|[Returns]ok:bool
```

### 17.5 `code_surface` — Python

```text
# py-jobs.ghost v2.0 profile=code_surface
# Purpose: Python print job surface.
# Source: authored
# Entries: 3

$PY__AUTOPRINT_JOBS
!autoprint_jobs|cat:module|id:py.autoprint.jobs|kind:module
!PrintJob|cat:type|id:py.autoprint.jobs.PrintJob|kind:class|[Rel]member_of:py.autoprint.jobs,conforms_to:ext:py.autoprint.protocols.Runnable,annotated_by:ext:py.dataclasses.dataclass|[Behavior]vis:public
^run|cat:callable|id:py.autoprint.jobs.PrintJob.run()|kind:method|[Rel]member_of:py.autoprint.jobs.PrintJob,throws:ext:py.autoprint.errors.JobError|[Behavior]async,effect:writes_state|[Returns]ok:bool
```

### 17.6 `code_surface` — TypeScript

```text
# ts-pricing.ghost v2.0 profile=code_surface
# Purpose: TypeScript pricing surface.
# Source: authored
# Entries: 4

$TS__AUTOPRINT_PRICING
!autoprint_pricing|cat:module|id:ts.autoprint.pricing|kind:module|[Rel]exports:ts.autoprint.pricing.JobSpec,exports:ts.autoprint.pricing.PriceQuote,exports:ts.autoprint.pricing.quote(JobSpec)
!JobSpec|cat:type|id:ts.autoprint.pricing.JobSpec|kind:interface|[Rel]member_of:ts.autoprint.pricing
!PriceQuote|cat:type|id:ts.autoprint.pricing.PriceQuote|kind:interface|[Rel]member_of:ts.autoprint.pricing
^quote|cat:callable|id:ts.autoprint.pricing.quote(JobSpec)|kind:function|[Rel]member_of:ts.autoprint.pricing|[Fields]spec:record<JobSpec>|[Behavior]async|[Returns]quote:record<PriceQuote>
```

### 17.7 `api_contract` — Linked-contract form

```text
# orders-api.ghost v2.0 profile=api_contract
# Purpose: Orders API contract surface.
# Source: authored
# Entries: 4

$HTTP__ORDERS
!Get_Order|cat:endpoint|id:http.orders.get|kind:http|path:"/orders/{order_id}"|verb:GET|[Rel]request:http.orders.get.req,response:http.orders.get.res_200,error:http.orders.get.err_404|[Behavior]auth:bearer,cacheable,idempotent,safe
!Get_Order_Request|cat:contract|id:http.orders.get.req|kind:request|[Fields]order_id:text,if_none_match:text
!Get_Order_Response_200|cat:contract|id:http.orders.get.res_200|kind:response|[Returns]order_id:text,customer_id:text,total_cents:int,etag:text
!Get_Order_Error_404|cat:contract|id:http.orders.get.err_404|kind:error|[Returns]code:text,message:text
```

### 17.8 Core workflow / protocol (no profile)

```text
# ingest-flow.ghost v2.0
# Purpose: Core-only ingest workflow sketch.
# Source: authored
# Entries: 6

$INGEST_FLOW
@received|cat:event|id:ev.ingest.received|kind:event|[Returns]job_id:text
+receive|cat:step|id:wf.ingest.receive|kind:step|[Rel]next:wf.ingest.validate,emits:ev.ingest.received|[Fields]job_id:text|[Behavior]effect:writes_state
+validate|cat:step|id:wf.ingest.validate|kind:step|[Rel]next:wf.ingest.nest,on_fail:wf.ingest.reject|[Fields]job_id:text|[Behavior]pre:file_present,post:art_normalized
+nest|cat:step|id:wf.ingest.nest|kind:step|[Rel]next:wf.ingest.schedule|[Fields]job_id:text|[Behavior]effect:cpu_bound
+schedule|cat:step|id:wf.ingest.schedule|kind:step|[Fields]job_id:text|[Behavior]effect:writes_state,terminal
+reject|cat:step|id:wf.ingest.reject|kind:step|[Behavior]terminal
```

### 17.9 Facet decomposition — UI surface

```text
# ui-surface.ghost v2.0 dialect=platform_metamodel
# Purpose: UI element surface split by facet.
# Source: authored
# Entries: 4

$UI_BUTTON
!Button|cat:type|id:ui.button|kind:component|facet:appearance|[Fields]label:expr<text>,style:style_ref,opacity:number,font_size:dim
!Button|cat:type|id:ui.button|kind:component|facet:layout|[Fields]visible_on_load:bool,width:dim,height:dim,x:number,y:number,margins:spacing,padding:spacing
!Button|cat:type|id:ui.button|kind:component|facet:states|[Returns]is_hovered:bool,is_pressed:bool,is_visible:bool
!Button|cat:type|id:ui.button|kind:component|facet:conditional|[Behavior]last_match_wins,condition_can_override_any_conditionable_property
```

### 17.10 Schema catalog dialect

```text
# autoprint-schema.ghost v2.0 dialect=schema_catalog
# Purpose: autoprint.io schema blueprint.
# Source: authored 2026-04-17
# Entries: 2

$DOMAIN
!dom_print_job|cat:schema|kind:table|[Fields]id:record<dom_print_job>,org:record<dom_org>,status:record<sys_job_status>|[Behavior]multi_tenant_by_org,audited
!dom_order_line|cat:schema|kind:table|[Fields]sku:text,description:text,qty:number{min_1}|[Behavior]sample:"Order line, inc. tax"|#note:description_example_contains_escaped_comma
```

---

## Appendix A — Informative ABNF

This grammar is informative, not normative. Consumers interpret Ghost leniently.

```abnf
ghost-file      = header 1*section

header          = hdr-name hdr-purpose hdr-source hdr-entries blankline
hdr-name        = "#" SP 1*(VCHAR) ".ghost" SP "v" 1*DIGIT ["." 1*DIGIT]
                  [SP "profile=" 1*(LOWER / DIGIT / "_")]
                  [SP "dialect=" 1*(LOWER / DIGIT / "_")] LF
hdr-purpose     = "#" SP "Purpose:" SP 1*VCHAR LF
hdr-source      = "#" SP "Source:" SP 1*VCHAR LF
hdr-entries     = "#" SP "Entries:" SP 1*DIGIT LF
blankline       = LF

section         = section-header *section-comment 1*entry
section-header  = "$" 1*(UPPER / DIGIT / "_") LF
section-comment = "#" *VCHAR LF

entry           = sigil entry-name "|" meta-field *("|" meta-field)
                  ["|" rel-block]
                  ["|" fields-block]
                  ["|" behavior-block]
                  ["|" returns-block]
                  *annotation LF
sigil           = "!" / ">" / "^" / "@" / "+"
entry-name      = 1*(ALPHA / DIGIT / "_" / ">")

meta-field      = meta-key ":" meta-value
meta-key        = "cat" / "id" / "kind" / "facet" / "scope" / "requires"
                / "context" / "target" / "status" / "path" / "verb"
                / "since" / "sunset" / "until" / ext-key
ext-key         = 1*(LOWER / DIGIT / "_")
meta-value      = 1*(meta-char)

rel-block       = "[Rel]" rel-item *("," rel-item)
rel-item        = role ":" rel-target [constraint-set]
role            = 1*(LOWER / DIGIT / "_")
rel-target      = internal-target / external-target
internal-target = id [ "#" facet ]
external-target = "ext:" ext-ref
ext-ref         = ext-token / quoted-string
ext-token       = 1*(ALPHA / DIGIT / "_" / "." / "-" / "/" / ":" / "(" / ")" / "+")
id              = 1*(ALPHA / DIGIT / "_" / "." / "-" / "/" / "(" / ")" / "+")
facet           = 1*(LOWER / DIGIT / "_")

fields-block    = "[Fields]" field-item *("," field-item)
returns-block   = "[Returns]" field-item *("," field-item)
field-item      = identifier ":" type [constraint-set]

behavior-block  = "[Behavior]" behavior-item *("," behavior-item)
behavior-item   = identifier [":" behavior-value] [constraint-set]
behavior-value  = quoted-string / literal

type            = scalar / generic / reference / compound / union
scalar          = "text" / "number" / "int" / "bool" / "date" / "url" / "image" / "any"
generic         = identifier "<" type *("," type) ">"
reference       = identifier "_ref"
compound        = "enum" "{" enum-values "}" / "field_changes"
                / "key_value_pairs" / "params"
union           = type "|" type *("|" type)
enum-values     = identifier *("/" identifier)

constraint-set  = "{" constraint *("," constraint) "}"
constraint      = 1*(constraint-char)

annotation      = "|" ("#note:" / "#warn:") 1*VCHAR

identifier      = 1*(ALPHA / DIGIT / "_")
literal         = 1*(literal-char)
quoted-string   = DQUOTE *(qchar) DQUOTE
qchar           = %x20-21 / %x23-5B / %x5D-7E / "\" DQUOTE / "\" "\"

meta-char       = %x20-7E
literal-char    = %x20-7E
constraint-char = %x20-7E

SP              = %x20
LF              = %x0A
DQUOTE          = %x22
UPPER           = %x41-5A
LOWER           = %x61-7A
ALPHA           = UPPER / LOWER
DIGIT           = %x30-39
VCHAR           = %x20-7E
```

Note: sigils `?` and `%` are reserved by the specification and are intentionally absent from
the productive `sigil` rule above.

---

## Appendix B — Quick Reference

```text
HEADER (exactly 4 lines)
  # <name>.ghost v2.0 [profile=<p>] [dialect=<d>]
  # Purpose: <one line>
  # Source: <origin>
  # Entries: <N>

BLANK LINES
  1 blank line after header
  1 blank line before each section after the first
  0 blank lines between entries in a section

SECTION
  $UPPER_CASE_UNDERSCORED
  #optional section comment

ENTRY (one physical line)
  <sigil><Entry_Name>|cat:x[|id:y][|kind:k][|facet:z][|meta:v]
                     [|[Rel]role:target,role:target]
                     [|[Fields]a:T{c},b:T]
                     [|[Behavior]k,k:v]
                     [|[Returns]r:T]
                     [|#note:...][|#warn:...]

SIGILS
  ! entity   > source   ^ operation   @ event   + action
  ? % reserved — MUST NOT be used

BLOCK ORDER
  meta → [Rel] → [Fields] → [Behavior] → [Returns] → #note → #warn

RELATIONS
  role:id
  role:id#facet
  role:ext:<external_ref>

FIELDS / RETURNS
  name:type{constraints}

BEHAVIOR
  flag
  key:value
  key:value{constraints}

TYPES
  text number int bool date url image any
  expr  expr<T>  list<T>  record<T>  map<K,V>
  <name>_ref
  enum{a/b/c}  field_changes  key_value_pairs  params
  union: A|B

META KEYS
  cat:  id:  kind:  facet:  scope:  requires:  context:
  target:  status:  path:  verb:  since:  sunset:  until:

CONSTRAINTS {}
  when_X  default:V  requires_X  deprecated_reason
  max_N   min_N      N_to_M
  units: seconds ms minutes hours days pixels em rem percent MB KB bytes degrees

UNIVERSAL cat:
  module type member callable source schema endpoint contract
  event state transition step example test metric

UNIVERSAL RELATION ROLES
  member_of part_of owns imports exports depends_on references
  extends implements conforms_to annotated_by
  calls reads writes throws
  request response error
  from to next on_fail emits consumes
  for compatible_with incompatible_with supersedes replaced_by

UNIVERSAL BEHAVIOR (bare atoms)
  async awaitable generator streaming concurrent ordered
  initial terminal hook single_use retriable
  handles_pii audited destructive safe idempotent cacheable

UNIVERSAL BEHAVIOR (keyed)
  vis: mut: bind: effect:
  pre: post: inv:
  auth: requires_role: complexity:
  latency_p50_ms: latency_p95_ms: throughput_rps: cost:

ESCAPING
  \|  \,  \[  \]  \{  \}  \\  \"
  "quoted string with, reserved chars"

ANNOTATIONS
  #note: informational
  #warn: risk / destructive
  notes precede warnings

NAMING
  Sections: UPPER_CASE_UNDERSCORED
  Entries:  Mixed_Case_Underscored or lower_case_underscored
  Fields:   lower_case_underscored
  IDs:      dot-segmented, no #
  Types:    lowercase keywords; source-preserved names inside generics/refs

FACETS
  facet:appearance / facet:layout / facet:states / facet:conditional
  facet:req / facet:res_<token> / facet:err_<token> / facet:headers
  Do NOT use facet:fields / facet:methods / facet:overloads

PROFILES (governance)
  profile=code_surface    → module / type / member / callable
  profile=api_contract    → endpoint / contract (+ linked / facet forms)

DIALECTS (extension)
  dialect=documentation_reference
  dialect=platform_metamodel
  dialect=schema_catalog
```

---

*End of specification.*
