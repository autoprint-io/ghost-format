# Ghost Format Specification

**Version:** 1.1
**Status:** Stable
**Date:** 2026-04-16
**Extension:** `.ghost`
**Encoding:** UTF-8, no BOM
**Line endings:** `LF` (`CRLF` accepted by consumers)

---

## 1. Abstract

Ghost is a line-oriented, pipe-delimited plain-text format for encoding structured technical
knowledge — documentation, platform metamodels, schema blueprints, and runtime catalogs — in a
form optimized for LLM context windows.

One entry per physical line. Canonical block order. Dense types and constraints. No nested
blocks. No ad-hoc substructures. The format is designed to preserve fields, types, constraints,
behaviors, returns, notes, and warnings with far fewer tokens than equivalent Markdown, YAML, or
JSON.

Ghost has one design commitment:

> **The LLM is the parser.**

Every rule in this specification follows from that.

---

## 2. Design Principles

1. **Token density is the primary goal.** Every byte must justify itself.
2. **No loss of technical meaning** on fields, types, constraints, behaviors, returns, notes, and warnings.
3. **One entry per physical line.** Greppability and diff stability are mandatory.
4. **Canonical ordering everywhere.** Positional regularity improves LLM retrieval.
5. **LLM comprehension over strict parser minimalism.** The format is intentionally regular, not ornamental.
6. **Extensible by dialect without breaking the core grammar.**
7. **Readable quickly, learnable completely.**
8. **No nested named blocks.** Structure is flattened into fields, behaviors, returns, and meta.

---

## 3. File Structure

### 3.1 Header

Every `.ghost` file starts with **exactly 4 lines**:

```text
# <filename>.ghost v<MAJOR>[.<MINOR>] [dialect=<name>]
# Purpose: <one-line description>
# Source: <origin or "authored">
# Entries: <integer>
```

Rules:

* Line 1 declares the version and may declare a dialect.
* Line 4's entry count must equal the number of actual entry lines in the file.
* No 5th header line is allowed.
* Exactly **one blank line** follows the header.

---

### 3.2 Sections

A section is a line matching:

```text
$UPPER_CASE_UNDERSCORED
```

Rules:

* Every entry must appear inside a section.
* Exactly **one blank line** precedes every section after the first.
* The first section appears immediately after the single blank line following the header.

Example:

```text
$AUTH_SIGNUP_LOGIN
```

---

### 3.3 Section Comments

A section may be followed by zero or more `#`-prefixed section comments.

These comments provide context to the LLM and are not counted as entries.

Example:

```text
$COOKIES
#Visible only when cookie controls are enabled
```

Rules:

* Section comments may appear only immediately after a section header.
* No blank line may appear between a section header, its section comments, and its first entry.

---

### 3.4 Entries

An entry is a single physical line beginning with a sigil defined in § 4.

Rules:

* Entries must be contiguous within a section.
* No blank lines may appear between entries in the same section.
* One entry equals one physical line. Hard wraps are forbidden.

---

## 4. Entry Sigils

Each entry begins with a one-character sigil.

| Sigil | Kind      | Use for                                                     |
| ----- | --------- | ----------------------------------------------------------- |
| `!`   | Entity    | Tables, types, elements, endpoints, records, schema objects |
| `>`   | Source    | Data sources, providers, origins, inputs                    |
| `^`   | Operation | Pure functions, transformers, expression operators          |
| `@`   | Event     | Triggers, hooks, lifecycle callbacks                        |
| `+`   | Action    | Mutations, commands, workflows that change state            |
| `?`   | Query     | Reserved                                                    |
| `%`   | Metric    | Reserved                                                    |

Rules:

* `!` is always valid as the default sigil.
* Richer sigils are recommended when they improve readability or retrieval.
* The sigil is followed immediately by the entry name, with no space.

---

## 5. Entry Model

### 5.1 General Form

```text
<sigil><Entry_Name>|<meta>[|<meta>...][|[Fields]...][|[Behavior]...][|[Returns]...][|#note:...][|#warn:...]
```

The first non-meta block determines where the meta block ends.

---

### 5.2 Canonical Block Order

The block order is mandatory:

```text
meta → [Fields] → [Behavior] → [Returns] → #note → #warn
```

Rules:

* Any block may be omitted.
* Omitted blocks leave no placeholder.
* Only these named blocks are valid:

  * `[Fields]`
  * `[Behavior]`
  * `[Returns]`

Forbidden examples:

* `[Options]`
* `[Params]`
* `[Config]`
* `[Output]`
* `[]`
* `no_fields`
* `none`

---

### 5.3 Meta Block

The meta block is the unmarked prefix between the entry name and the first named block.

Meta fields are separated by `|`.

Every entry must begin its meta with:

```text
cat:<category>
```

#### Canonical meta keys

| Key         | Purpose                                         | Example                            |
| ----------- | ----------------------------------------------- | ---------------------------------- |
| `cat:`      | Category (required, always first)               | `cat:auth`, `cat:element`          |
| `id:`       | Stable machine-readable identifier              | `id:ui.button`                     |
| `facet:`    | Orthogonal facet of the same logical entry      | `facet:appearance`, `facet:layout` |
| `scope:`    | Runtime or platform scope                       | `scope:native_mobile`              |
| `requires:` | Prerequisite feature, plan, flag, or capability | `requires:paid_plan`               |
| `context:`  | Where the entry runs or is available            | `context:workflow_only`            |
| `target:`   | What or whom the entry operates on              | `target:current_user`              |
| `status:`   | Lifecycle status                                | `status:beta`, `status:deprecated` |

Rules:

* `cat:` is required and must be the first meta field.
* `id:` is optional but recommended for stable cross-reference.
* `facet:` is recommended when a logical object has multiple wide property families.
* Meta fields describe **preconditions, scope, identity, or classification**.
* Consequences belong in `[Behavior]`, not in meta.

Test:

* **Precondition or classification?** → meta
* **Consequence or effect?** → `[Behavior]`

---

### 5.4 `[Fields]`

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

### 5.5 `[Behavior]`

`[Behavior]` contains a comma-separated list of descriptors.

Two forms are allowed:

```text
single_use
triggers_event:User_is_logged_in
```

Behavior item grammar:

```text
descriptor
descriptor:value
descriptor:value{constraints}
```

Rules:

* Use bare descriptors for simple facts: `single_use`, `auto_login`, `server_cached`
* Use keyed descriptors when the target matters: `triggers_event:User_is_logged_in`
* Behavior should be specific and non-redundant with the entry name
* Behavior may use quoted values when needed

Example:

```text
[Behavior]sample:"Price, with VAT"
```

---

### 5.6 `[Returns]`

`[Returns]` uses the same grammar as `[Fields]`:

```text
[Returns]temp_password:text,link:url{when_create_link_only,server_side_only}
```

Rules:

* Return names use `lower_case_underscored`
* Types are mandatory
* Conditional returns use constraints, not custom blocks

Correct:

```text
[Returns]link:url{when_create_link_only}
```

Incorrect:

```text
[Returns_when_create_link]link:url
```

---

### 5.7 Facet Decomposition (Recommended)

When one logical object has multiple wide orthogonal surfaces, split it into multiple entries
using `facet:` instead of inventing new block names.

Example:

```text
!Button|cat:ui_element|id:ui.button|facet:appearance|[Fields]label:expr<text>,style:style_ref,opacity:number
!Button|cat:ui_element|id:ui.button|facet:layout|[Fields]visible_on_load:bool,width:dim,height:dim,x:number,y:number
!Button|cat:ui_element|id:ui.button|facet:states|[Returns]is_hovered:bool,is_pressed:bool,is_visible:bool
!Button|cat:ui_element|id:ui.button|facet:conditional|[Behavior]last_match_wins,any_conditionable_property
```

Use `facet:` when:

* the entry would otherwise become too wide
* the object has clearly distinct domains such as `appearance`, `layout`, `states`, `conditional`, `runtime`, `privacy`

Do not use custom named blocks to achieve the same effect.

---

## 6. Type System

### 6.1 Core Scalar Types

Core scalar types:

* `text`
* `number`
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

Use `_ref` when the target is a first-class platform object and a generic `record<T>` would be less readable.

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

* Use unions only inside type positions
* If the union visually collides with block boundaries, prefer:

  * `list<page_ref|view_ref>`
  * splitting the field into two fields
  * a dialect-specific named type

---

### 6.6 Dialect Extensions

Dialects may extend the type system.

Canonical extensions:

| Domain      | Types                                                                 |
| ----------- | --------------------------------------------------------------------- |
| UI          | `color`, `dim`, `border`, `shadow`, `spacing`, `font`                 |
| Rust/SQLite | `struct_ref`, `trait_ref`, `crate_ref`, `sql_type`, `migration_ref`   |
| HTTP/API    | `endpoint_ref`, `header_ref`, `status_code`, `json_path`, `mime_type` |
| CLI         | `flag`, `subcommand_ref`, `path`, `glob`, `env_var`                   |
| Config      | `config_key`, `section_ref`, `env_ref`                                |

A dialect must document the types it adds.

---

## 7. Constraints

Constraints attach to a field or return type using `{}`:

```text
number{0_to_24,default:1,hours}
expr<text>{server_side_only}
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

Reserved characters inside values must be escaped or quoted.

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
!Quoted_Value_Demo|cat:demo|[Behavior]sample:"Price, with VAT"
```

Inside quotes, only `"` and `\` must be escaped.

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

* All `#note:` annotations must come before all `#warn:` annotations
* Multiple notes and warnings are allowed

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

Entry names should preserve source naming where possible.

Two canonical styles are allowed:

1. **Mixed_Case_Underscored** — for product-visible names
   Example: `!Sign_the_user_up`

2. **lower_case_underscored** — for schema or internal identifiers
   Example: `!dom_print_job`

Rules:

* Preserve source identity when that improves fidelity
* Use one style consistently within the same semantic family
* Spaces become underscores
* `>` may appear inside entry names to mirror a source hierarchy

Example:

```text
!Settings>General>Redirect
```

---

### 10.3 Field Names

Field names use:

```text
lower_case_underscored
```

Example:

```text
email_to_reset
```

---

### 10.4 Meta Keys

Meta keys use:

```text
lower_case:
```

Examples:

* `cat:`
* `id:`
* `facet:`
* `scope:`

---

### 10.5 Types

Primitive and generic type keywords use lowercase:

* `text`
* `list<T>`
* `expr<T>`

Referenced entity/type names inside generics may preserve source casing:

* `record<User>`
* `record<dom_print_job>`

---

### 10.6 Reserved Identifiers

Reserved identifiers include:

* meta keys defined in § 5.3
* named blocks `[Fields]`, `[Behavior]`, `[Returns]`
* annotation prefixes `#note:` and `#warn:`

---

## 11. Validation

A producer should run this checklist before delivering a `.ghost` file.

### 11.1 Required Checks

1. Header line 4 `Entries:` count matches the actual number of entry lines beginning with any defined sigil.
2. No empty block markers.
3. Canonical block order is respected.
4. Only `[Fields]`, `[Behavior]`, `[Returns]` appear as named blocks.
5. Every entry begins its meta with `cat:`.
6. No blank lines between entries within a section.
7. One physical line per entry.
8. Section names are `UPPER_CASE_UNDERSCORED`.
9. Entry names follow one of the allowed naming conventions and are consistent with source naming.
10. No pedagogical prose remnants inside entries.
11. Constraints use `{}`, not `()`.
12. Pipes separate blocks and meta fields; commas separate items within blocks.
13. `#note:` precedes `#warn:` in every entry.

---

### 11.2 Errors (Non-conformant)

| Code | Description                                         |
| ---- | --------------------------------------------------- |
| E1   | Header missing or malformed                         |
| E2   | Header count mismatch                               |
| E3   | Entry missing `cat:` meta                           |
| E4   | Non-canonical block order                           |
| E5   | Ad-hoc block name (`[Options]`, `[Params]`, etc.)   |
| E6   | Empty block marker                                  |
| E7   | Entry split across multiple physical lines          |
| E8   | Invalid section name                                |
| E9   | Tab character in file                               |
| E10  | Parentheses used for constraints instead of `{}`    |
| E11  | `#warn:` appears before `#note:` in the same entry  |
| E12  | Entry appears before the first section              |
| E13  | Blank line between entries within a section         |
| E14  | Missing blank line after header or between sections |

---

### 11.3 Warnings (Conformant but Discouraged)

| Code | Description                                           |
| ---- | ----------------------------------------------------- |
| W1   | Pedagogical prose remnants                            |
| W2   | Unresolved cross-reference in `[Behavior]`            |
| W3   | Duplicate entry name within the file                  |
| W4   | Unknown type not in core or declared dialect          |
| W5   | Section contains a single entry                       |
| W6   | Oversized entry likely better split by `facet:`       |
| W7   | Overuse of `params` where stronger typing is possible |

A consumer should flag warnings but continue reading.

---

## 12. Anti-patterns

| Code | Forbidden                                       | Fix                                             |
| ---- | ----------------------------------------------- | ----------------------------------------------- |
| F1   | `[Fields]` empty or `no_fields`                 | Omit the block                                  |
| F2   | Returns before Fields                           | Reorder                                         |
| F3   | YAML/JSON inside a block                        | Flatten into pipe/comma form                    |
| F4   | Entry wrapped across lines                      | Keep on one physical line                       |
| F5   | Prose like “Usually you will want to…”          | Delete                                          |
| F6   | `number(0-24)`                                  | `number{0_to_24}`                               |
| F7   | Lowercase section name                          | Use `UPPER_CASE_UNDERSCORED`                    |
| F8   | Entry with no `cat:`                            | Add `cat:<value>`                               |
| F9   | `[Options]`, `[Params]`, `[Config]`, `[Output]` | Use `[Fields]` or `[Returns]` or split by facet |
| F10  | Blank line between entries                      | Remove                                          |
| F11  | Nested named blocks                             | Flatten                                         |
| F12  | `#warn:` before `#note:`                        | Reorder                                         |
| F13  | Emoji or decorative glyphs inside entries       | Strip                                           |
| F14  | Tabs                                            | Use spaces only                                 |
| F15  | Trailing `.` or `;` on entries                  | Strip                                           |

---

## 13. Dialects

A dialect is a named profile layered on top of the core spec.

A dialect may:

* restrict valid `cat:` values
* extend the type system
* define additional meta keys
* recommend standard sections
* add validation rules
* define recommended `facet:` vocabularies

A dialect must not:

* relax the core grammar
* change canonical block order
* introduce new named blocks

Declare a dialect on header line 1:

```text
# autoprint-schema.ghost v1.1 dialect=schema_catalog
```

---

### 13.1 Registered Dialects

| Dialect                   | Purpose                                                |
| ------------------------- | ------------------------------------------------------ |
| `documentation_reference` | Vendor, API, CLI, SDK documentation                    |
| `platform_metamodel`      | Platform surface area, reverse-engineered applications |
| `schema_catalog`          | Database schema blueprints and metamodel catalogs      |

---

### 13.2 Defining a New Dialect

A dialect addendum must specify:

* dialect name
* version
* purpose
* added types
* added meta keys
* recommended sections
* one canonical example

---

## 14. Examples

### 14.1 Minimal Valid File

```text
# minimal.ghost v1.1
# Purpose: Smallest conformant Ghost file.
# Source: authored
# Entries: 1

$ROOT
!Ping|cat:probe|[Behavior]returns_pong
```

---

### 14.2 Documentation Reference Dialect

```text
# bubble-auth.ghost v1.1 dialect=documentation_reference
# Purpose: Bubble.io auth actions.
# Source: https://manual.bubble.io
# Entries: 3

$AUTH_SIGNUP_LOGIN
+Sign_the_user_up|cat:auth|[Fields]email:expr<text>,password:expr<text>,type_of_user:type_ref{default:User},other_fields:field_changes|[Behavior]creates_user,auto_login,triggers_event:User_is_signed_up|#note:equivalent_to_Create_a_new_thing_plus_Log_the_user_in
+Log_the_user_in|cat:auth|[Fields]email:expr<text>,password:expr<text>,stay_logged_in:bool{default:true}|[Behavior]creates_session,triggers_event:User_is_logged_in|#note:respects_Settings>General>Cookie_duration
+Log_the_user_out|cat:auth|[Behavior]triggers_event:Current_user_is_logged_out,clears_session
```

---

### 14.3 Platform Metamodel Dialect

```text
# bubble-metamodel.ghost v1.1 dialect=platform_metamodel
# Purpose: Bubble application metamodel.
# Source: architecture_map extraction
# Entries: 5

$ELEMENT_TYPES
!Container|cat:element|[Fields]layout:enum{row/col/fixed},responsive:bool|[Behavior]can_contain_children,participates_in_stacking_context
!Repeating_Group|cat:element|[Fields]data_source:expr<list<record<User>>>,layout:enum{vertical/horizontal/ext_vertical/ext_horizontal},rows:number,cols:number|[Behavior]renders_template_per_item|#note:pagination_via_workflow_not_builtin

$EXPRESSION_OPS
^get_field|cat:op|[Fields]target:expr<record<User>>,field:field_ref|[Returns]value:any|#note:respects_privacy_rules
>Current_User|cat:source|[Returns]user:record<User>|#note:null_when_not_logged_in
@Page_is_loaded|cat:event|scope:frontend|[Behavior]fires_once_per_navigation
```

---

### 14.4 Schema Catalog Dialect

```text
# autoprint-schema.ghost v1.1 dialect=schema_catalog
# Purpose: autoprint.io schema blueprint.
# Source: authored 2026-04-16
# Entries: 2

$DOMAIN
!dom_print_job|cat:domain|[Fields]id:record<dom_print_job>,org:record<dom_org>,status:record<sys_job_status>|[Behavior]multi_tenant_by_org,audited
!dom_order_line|cat:domain|[Fields]sku:text,description:text,qty:number{min_1}|[Behavior]sample:"Order line, inc. tax"|#note:description_example_contains_escaped_comma
```

---

### 14.5 Facet Decomposition Example

```text
# ui-surface.ghost v1.1 dialect=platform_metamodel
# Purpose: UI element surface split by facet.
# Source: authored
# Entries: 4

$UI_BUTTON
!Button|cat:ui_element|id:ui.button|facet:appearance|[Fields]label:expr<text>,style:style_ref,opacity:number,font_size:dim
!Button|cat:ui_element|id:ui.button|facet:layout|[Fields]visible_on_load:bool,width:dim,height:dim,x:number,y:number,margins:spacing,padding:spacing
!Button|cat:ui_element|id:ui.button|facet:states|[Returns]is_hovered:bool,is_pressed:bool,is_visible:bool
!Button|cat:ui_element|id:ui.button|facet:conditional|[Behavior]last_match_wins,condition_can_override_any_conditionable_property
```

---

## Appendix A — Informative ABNF

This grammar is informative, not normative. Consumers interpret Ghost leniently.

```abnf
ghost-file      = header 1*section

header          = hdr-name hdr-purpose hdr-source hdr-entries blankline
hdr-name        = "#" SP 1*(VCHAR) ".ghost" SP "v" 1*DIGIT ["." 1*DIGIT] [SP "dialect=" 1*(LOWER / DIGIT / "_")] LF
hdr-purpose     = "#" SP "Purpose:" SP 1*VCHAR LF
hdr-source      = "#" SP "Source:" SP 1*VCHAR LF
hdr-entries     = "#" SP "Entries:" SP 1*DIGIT LF
blankline       = LF

section         = section-header *section-comment 1*entry
section-header  = "$" 1*(UPPER / DIGIT / "_") LF
section-comment = "#" *VCHAR LF

entry           = sigil entry-name "|" meta-field *("|" meta-field) *("|" named-block) *annotation LF
sigil           = "!" / ">" / "^" / "@" / "+" / "?" / "%"
entry-name      = 1*(ALPHA / DIGIT / "_" / ">")

meta-field      = meta-key ":" meta-value
meta-key        = "cat" / "id" / "facet" / "scope" / "requires" / "context" / "target" / "status" / ext-key
ext-key         = 1*(LOWER / DIGIT / "_")
meta-value      = 1*(meta-char)

named-block     = fields-block / behavior-block / returns-block

fields-block    = "[Fields]" field-item *("," field-item)
returns-block   = "[Returns]" field-item *("," field-item)
field-item      = identifier ":" type [constraint-set]

behavior-block  = "[Behavior]" behavior-item *("," behavior-item)
behavior-item   = identifier [":" behavior-value] [constraint-set]
behavior-value  = quoted-string / literal

type            = scalar / generic / reference / compound / union
scalar          = "text" / "number" / "bool" / "date" / "url" / "image" / "any"
generic         = identifier "<" type *("," type) ">"
reference       = identifier "_ref"
compound        = "enum" "{" enum-values "}" / "field_changes" / "key_value_pairs" / "params"
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

---

## Appendix B — Quick Reference

```text
HEADER (exactly 4 lines)
  # <name>.ghost v1.1 [dialect=<d>]
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
  <sigil><Entry_Name>|cat:x[|id:y][|facet:z][|meta:v][|[Fields]a:T{c},b:T][|[Behavior]k,k:v][|[Returns]r:T][|#note:...][|#warn:...]

SIGILS
  ! entity   > source   ^ operation   @ event   + action   ? reserved   % reserved

BLOCK ORDER
  meta → [Fields] → [Behavior] → [Returns] → #note → #warn

FIELDS / RETURNS
  name:type{constraints}

BEHAVIOR
  flag
  key:value
  key:value{constraints}

TYPES
  text number bool date url image any
  expr  expr<T>  list<T>  record<T>  map<K,V>
  <name>_ref
  enum{a/b/c}  field_changes  key_value_pairs  params
  union: A|B

META KEYS
  cat:  id:  facet:  scope:  requires:  context:  target:  status:

CONSTRAINTS {}
  when_X  default:V  requires_X  deprecated_reason
  max_N   min_N      N_to_M
  units: seconds ms minutes hours days pixels em rem percent MB KB bytes degrees

ESCAPING
  \|  \,  \[  \]  \{  \}  \\  \"
  "quoted string with, reserved chars"

ANNOTATIONS
  #note: informational
  #warn: risk / destructive
  notes precede warnings

NAMING
  Sections: UPPER_CASE_UNDERSCORED
  Entries: Mixed_Case_Underscored or lower_case_underscored
  Fields: lower_case_underscored
  Types: lowercase keywords; source-preserved names allowed inside generics/refs

FACETS
  Use facet:appearance / facet:layout / facet:states / facet:conditional / ...
  Do not invent new named blocks
```

---

*End of specification.*
