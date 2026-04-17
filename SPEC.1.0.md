# Ghost Format Specification

**Version:** 1.0
**Status:** Stable
**Date:** 2026-04-16
**Extension:** `.ghost`
**Encoding:** UTF-8, no BOM
**Line endings:** `LF` (parsers accept `CRLF`)

---

## 1. Abstract

Ghost is a line-oriented, pipe-delimited plain-text format for encoding structured technical
knowledge — documentation, platform metamodels, and schema blueprints — in a form optimized for
LLM context windows. One entry per line. Canonical block order. Dense types and constraints.
No nesting. Roughly 70% fewer tokens than equivalent Markdown with zero loss on fields, types,
behaviors, and warnings.

The format's single design commitment: **the LLM is the parser**. Every rule below follows
from that.

---

## 2. Design Principles

1. **Token density is the first goal.** Every byte must justify itself.
2. **Zero information loss** on fields, types, constraints, behaviors, returns, and warnings.
3. **One entry per line.** Greppability and diff stability are non-negotiable.
4. **Canonical ordering everywhere.** Consistent positional cues let an LLM retrieve reliably
   across thousands of entries.
5. **LLM comprehension over strict parseability.** Lenient consumption is the semantics.
6. **Domain-extensible without breaking the core.** New types and sigils never invalidate old
   files.
7. **Progressive disclosure.** Readable in 60 seconds; masterable in 10 minutes.

---

## 3. File Structure

### 3.1 Header (exactly 4 lines)

```
# <filename>.ghost v<MAJOR>[.<MINOR>] [dialect=<name>]
# Purpose: <one-line description>
# Source: <origin or "authored">
# Entries: <integer>
```

Rules:

- Line 1 declares the version and optionally a dialect.
- Line 4's entry count must match the actual number of entry lines in the file.
- No 5th header line is allowed.

### 3.2 Sections

A section is a line matching `^\$[A-Z0-9_]+$`. Sections group semantically related entries.
Exactly one blank line precedes every section except the first (which follows the header).

```
$AUTH_SIGNUP_LOGIN
```

A section may be followed by zero or more `#`-prefixed section comments, which give context
to the LLM:

```
$COOKIES
#Only visible when Settings>General>Do_not_set_cookies_on_new_visitors_by_default is enabled
```

### 3.3 Entries

An entry is a single physical line beginning with a sigil (§ 4). Entries must appear inside a
section. Within a section, entries are contiguous — no blank lines between them.

---

## 4. Entry Sigils

Every entry begins with a single-character sigil that classifies it. Sigils make files
grep-friendly by semantic kind: `rg '^>' file.ghost` returns every source, `rg '^@'` every
event, and so on.

| Sigil | Kind       | Use for                                                       |
|-------|------------|---------------------------------------------------------------|
| `!`   | Entity     | Tables, types, elements, endpoints, commands, records         |
| `>`   | Source     | Data sources, providers, origins, inputs                      |
| `^`   | Operation  | Pure functions, transformers, expression operators            |
| `@`   | Event      | Triggers, hooks, lifecycle callbacks                          |
| `+`   | Action     | Mutations, commands, workflows that change state              |
| `?`   | Query      | *(reserved for future use)*                                   |
| `%`   | Metric     | *(reserved for future use)*                                   |

`!` is the default sigil. Any entry can always be written with `!` regardless of kind. Use the
richer sigils when the extra classification is useful to the reader.

The sigil is followed immediately (no space) by the entry name.

---

## 5. Entry Grammar

```
<sigil><Entry_Name> | <meta> [| [Fields]...] [| [Behavior]...] [| [Returns]...] [| #note:...] [| #warn:...]
```

### 5.1 Canonical block order (mandatory)

```
meta  →  [Fields]  →  [Behavior]  →  [Returns]  →  #note  →  #warn
```

Any block may be omitted. The order of the remaining blocks is preserved. An omitted block
produces no marker — never write `[Fields]` empty, `no_fields`, `none`, or `[]`.

### 5.2 Meta block

The meta block is unmarked: it is whatever sits between the entry name and the first `[...]`
block. Meta fields are separated by `|`.

**Required:** every entry must start its meta with `cat:<category>`.

**Canonical meta keys:**

| Key         | Purpose                               | Example                             |
|-------------|---------------------------------------|-------------------------------------|
| `cat:`      | Category (required)                   | `cat:auth`, `cat:element`           |
| `scope:`    | Platform or runtime constraint        | `scope:native_mobile`               |
| `requires:` | Prerequisite (plan, feature, flag)    | `requires:paid_plan`                |
| `context:`  | Where the entry runs or applies       | `context:reset_pw_page`             |
| `target:`   | Who the entry operates on             | `target:current_logged_in_user`     |
| `status:`   | Lifecycle status                      | `status:beta`, `status:deprecated`  |

Side effects, event triggers, and equivalences are **not** meta — they belong in `[Behavior]`.
Test: *"Is this a precondition, or a consequence?"* Preconditions are meta. Consequences are
behavior.

### 5.3 `[Fields]`

Comma-separated list of `field_name:type[{constraints}]`:

```
[Fields]email:expr<text>,time_hrs:number{0_to_24,default:1}
```

### 5.4 `[Behavior]`

Comma-separated list of descriptors — either bare (`single_use`) or keyed
(`triggers_event:User_is_logged_in`):

```
[Behavior]creates_user,auto_login,triggers_event:User_is_signed_up
```

Behaviors should be specific and non-redundant with the entry name: an entry named
`!Delete_thing` does not need `deletes_thing` in behavior unless there is a nuance.

### 5.5 `[Returns]`

Same grammar as `[Fields]`. Conditional returns use a `{when_X}` constraint — never a
separate `[Returns_when_X]` block.

```
[Returns]temp_password:text,link:text{when_just_create_link,server_side_only}
```

---

## 6. Type System

### 6.1 Primitives

`text`, `number`, `bool`, `date`, `url`, `image`

### 6.2 Generics

| Form           | Meaning                                           |
|----------------|---------------------------------------------------|
| `expr`         | Dynamic expression of indeterminate type          |
| `expr<T>`      | Dynamic expression of type `T`                    |
| `list<T>`      | Ordered collection of `T`                         |
| `record<T>`    | Reference to a record of table `T`                |

Nested generics are allowed: `list<list<text>>`, `expr<list<record<dom_user>>>`.

### 6.3 References

Names ending in `_ref` denote references to other first-class objects:

```
page_ref, view_ref, element_ref, event_ref, workflow_ref, type_ref, field_ref,
state_ref, input_ref, rg_ref, map_ref, table_ref, index_ref, function_ref, access_ref
```

### 6.4 Compound types

| Form                | Meaning                              |
|---------------------|--------------------------------------|
| `enum{a/b/c}`       | One-of enumerated literal values     |
| `field_changes`     | Field-change tuple                   |
| `key_value_pairs`   | Arbitrary KV map                     |
| `params`            | Free-form parameter bag              |

### 6.5 Unions

Pipes inside a type position denote union:

```
navigate_to:page_ref|view_ref
```

When a union would be adjacent to a block boundary, wrap it inside generics or split it into
multiple fields to avoid ambiguity.

### 6.6 Dialect type extensions

Dialects may extend the type system. Canonical extensions:

| Domain      | Types                                                                        |
|-------------|------------------------------------------------------------------------------|
| UI          | `color`, `dim`, `border`, `shadow`, `spacing`, `font`                        |
| Rust/SQLite | `struct_ref`, `trait_ref`, `crate_ref`, `sql_type`, `migration_ref`          |
| HTTP/API    | `endpoint_ref`, `header_ref`, `status_code`, `json_path`, `mime_type`        |
| CLI         | `flag`, `subcommand_ref`, `path`, `glob`, `env_var`                          |
| Config      | `config_key`, `section_ref`, `env_ref`                                       |

A dialect must document the types it adds.

---

## 7. Constraints

Constraints attach to a type via `{}` and contain one or more comma-separated atoms:

```
number{0_to_24,default:1,hours}
expr<text>{no_email_sent_if_user_not_found}
```

### 7.1 Canonical patterns

| Pattern                 | Meaning                           | Example                    |
|-------------------------|-----------------------------------|----------------------------|
| `when_<X>`              | Conditional visibility            | `{when_change_email}`      |
| `default:<V>`           | Default value                     | `{default:1}`              |
| `requires_<X>`          | Requires another field or state   | `{requires_include_labels}`|
| `deprecated_<reason>`   | Deprecated                        | `{deprecated_Jul_2017}`    |
| `max_<N>`               | Maximum value or count            | `{max_50}`                 |
| `min_<N>`               | Minimum value or count            | `{min_1}`                  |
| `<N>_to_<M>`            | Inclusive numeric range           | `{0_to_24}`                |
| `<unit>`                | Unit annotation                   | `{seconds}`, `{pixels}`    |
| `server_side_only`      | Scope restriction                 | `{server_side_only}`       |
| `single_use`            | Behavior restriction              | `{single_use}`             |

### 7.2 Units

Canonical units: `seconds`, `ms`, `minutes`, `hours`, `days`, `pixels`, `em`, `rem`, `percent`,
`MB`, `KB`, `bytes`, `degrees`.

---

## 8. Escaping

Reserved characters inside values must be escaped or quoted.

### 8.1 Backslash escapes

```
\|   \,   \[   \]   \{   \}   \\   \"
```

### 8.2 Quoted strings

For values containing multiple reserved characters or free text, wrap in double quotes. Inside
quotes, only `"` and `\` need escaping:

```
!Text|cat:visual|[Content]label:"Price, with VAT"
```

Quoted and unquoted values are semantically equivalent — the quotes are transparent to the
consumer. Use whichever is shorter.

---

## 9. Annotations

### 9.1 `#note:`

Informational: tips, cross-references, edge cases, platform quirks.

```
|#note:signup_implicitly_calls_Opt_in_to_cookies
```

### 9.2 `#warn:`

Risk: destructive operations, irreversibility, security implications.

```
|#warn:original_password_irrecoverable
```

### 9.3 Ordering

Within an entry, all `#note:` annotations must precede all `#warn:` annotations. Multiple
annotations of either kind are allowed.

---

## 10. Naming

| Element        | Convention                  | Example                 |
|----------------|-----------------------------|-------------------------|
| Entry names    | `Mixed_Case_Underscored`    | `!Sign_the_user_up`     |
| Section names  | `UPPER_CASE_UNDERSCORED`    | `$AUTH_SIGNUP_LOGIN`    |
| Field names    | `lower_case_underscored`    | `email_to_reset`        |
| Meta keys      | `lower_case` + `:`          | `cat:`, `scope:`        |
| Types          | `lower_case`                | `expr<list<text>>`      |
| Constraint atoms| `lower_case_underscored`   | `{when_is_admin}`       |
| Category values| `lower_case_underscored`    | `cat:workflow_control`  |

Entry names preserve source casing where possible, with spaces replaced by underscores. `>`
may appear inside entry names to mirror a source hierarchy (`Settings>General>Redirect`).

Reserved identifiers: the meta keys in § 5.2, the block names `[Fields]` `[Behavior]`
`[Returns]`, and the annotation prefixes `#note:` `#warn:`.

---

## 11. Validation

A producer runs this 13-item checklist before delivering any `.ghost` file:

1. Header line 4 `Entries:` count matches the actual `!`/`>`/`^`/`@`/`+` entry count.
2. No empty block markers (`no_fields`, `[]`, `none`).
3. Canonical block order is respected in every entry.
4. Only `[Fields]`, `[Behavior]`, `[Returns]` appear as named blocks.
5. Every entry begins its meta with `cat:`.
6. No blank lines between entries within a section.
7. One entry per line.
8. Section names are `UPPER_CASE_UNDERSCORED`.
9. Entry names are `Mixed_Case_Underscored`, matching source naming.
10. No pedagogical prose remnants (*"usually"*, *"for example"*, *"learn more"*).
11. Constraints use `{}`, never `()`.
12. Pipes separate blocks; commas separate items within a block.
13. `#note:` precedes `#warn:` in every entry.

### 11.1 Errors (file is non-conformant)

| Code | Description                                                          |
|------|----------------------------------------------------------------------|
| E1   | Header missing or malformed                                          |
| E2   | Header count mismatch                                                |
| E3   | Entry missing `cat:` meta                                            |
| E4   | Non-canonical block order                                            |
| E5   | Ad-hoc block name (`[Options]`, `[Params]`, etc.)                    |
| E6   | Empty block marker                                                   |
| E7   | Entry split across multiple lines                                    |
| E8   | Invalid section name                                                 |
| E9   | Tab character in file                                                |
| E10  | Parentheses used for constraints instead of `{}`                     |
| E11  | `#warn:` appears before `#note:` in the same entry                   |
| E12  | Entry appears before the first `$SECTION`                            |
| E13  | Blank lines between entries within a section                         |

### 11.2 Warnings (ill-advised but conformant)

| Code | Description                                                          |
|------|----------------------------------------------------------------------|
| W1   | Pedagogical prose remnants                                           |
| W2   | Unresolved cross-reference in `[Behavior]`                           |
| W3   | Duplicate entry name within the file                                 |
| W4   | Unknown type (not in § 6 or declared dialect)                        |
| W5   | Section contains a single entry                                      |

A consumer should flag warnings but continue reading. The format is optimized for graceful
LLM interpretation, not strict parser failure.

---

## 12. Anti-patterns

| Code | Forbidden                                         | Fix                                          |
|------|---------------------------------------------------|----------------------------------------------|
| F1   | `[Fields]` empty or `no_fields`                   | Omit the block                               |
| F2   | Returns before Fields                             | Reorder                                      |
| F3   | YAML/JSON inside a block                          | Flatten to pipe/comma                        |
| F4   | Entry wrapped across lines                        | Keep on one line                             |
| F5   | Prose like "Usually you will want to…"            | Delete                                       |
| F6   | `number(0-24)`                                    | `number{0_to_24}`                            |
| F7   | Lowercase section name                            | UPPER_CASE                                   |
| F8   | Entry with no `cat:`                              | Add `cat:<value>`                            |
| F9   | `[Options]`, `[Params]`, `[Config]`, `[Output]`   | Use `[Fields]` or `[Returns]`                |
| F10  | Blank line between entries                        | Remove                                       |
| F11  | Nested blocks                                     | Flatten                                      |
| F12  | `#warn:` before `#note:`                          | Reorder                                      |
| F13  | Emoji or decorative glyphs inside entries         | Strip                                        |
| F14  | Tabs                                              | Spaces only, inside values                   |
| F15  | Trailing `.` or `;` on entries                    | Strip                                        |

---

## 13. Dialects

A dialect is a named profile layered on top of the core spec. A dialect may:

- Restrict which `cat:` values are valid.
- Extend the type system.
- Define additional meta keys.
- Recommend standard sections.
- Prescribe additional validation rules.

A dialect must not relax the core grammar or change canonical block order.

Dialects are declared optionally in header line 1:

```
# autoprint-schema.ghost v1 dialect=schema_catalog
```

### 13.1 Registered dialects

| Dialect                   | Purpose                                                 |
|---------------------------|---------------------------------------------------------|
| `documentation_reference` | Vendor, API, CLI, SDK documentation                     |
| `platform_metamodel`      | Platform surface area, reverse-engineered applications  |
| `schema_catalog`          | Database schema blueprints (hybrid metamodel + domain)  |

### 13.2 Defining a new dialect

A new dialect publishes a short addendum with: name, version, purpose, added sections, added
meta keys, added types, one canonical example.

---

## 14. Examples

### 14.1 Minimal valid file

```
# minimal.ghost v1
# Purpose: Smallest conformant Ghost file.
# Source: authored
# Entries: 1

$ROOT
!Ping|cat:probe|[Behavior]returns_pong
```

### 14.2 Documentation reference dialect

```
# bubble-auth.ghost v1 dialect=documentation_reference
# Purpose: Bubble.io auth actions.
# Source: https://manual.bubble.io
# Entries: 3

$AUTH_SIGNUP_LOGIN

+Sign_the_user_up|cat:auth|[Fields]email:expr<text>,password:expr<text>,type_of_user:type_ref{default:User},other_fields:field_changes|[Behavior]creates_user,auto_login,triggers_event:User_is_signed_up|#note:equivalent_to_Create_a_new_thing_plus_Log_the_user_in
+Log_the_user_in|cat:auth|[Fields]email:expr<text>,password:expr<text>,stay_logged_in:bool{default:true}|[Behavior]creates_session,triggers_event:User_is_logged_in|#note:respects_Settings>General>Cookie_duration
+Log_the_user_out|cat:auth|[Behavior]triggers_event:Current_user_is_logged_out,clears_session
```

### 14.3 Platform metamodel dialect

```
# bubble-metamodel.ghost v1 dialect=platform_metamodel
# Purpose: Bubble application metamodel.
# Source: architecture_map extraction
# Entries: 4

$ELEMENT_TYPES

!Container|cat:element|[Fields]layout:enum{row/col/fixed},responsive:bool|[Behavior]can_contain_children,participates_in_stacking_context
!Repeating_Group|cat:element|[Fields]data_source:expr<list>,layout:enum{vertical/horizontal/ext_vertical/ext_horizontal},rows:number,cols:number|[Behavior]renders_template_per_item|#note:pagination_via_workflow_not_builtin

$EXPRESSION_OPS

^get_field|cat:op|[Fields]target:expr<thing>,field:field_ref|[Returns]value:expr|#note:respects_privacy_rules
>Current_User|cat:source|[Returns]user:record<User>|#note:null_when_not_logged_in
```

### 14.4 Schema catalog dialect (escaping demo)

```
# autoprint-schema.ghost v1 dialect=schema_catalog
# Purpose: autoprint.io schema blueprint.
# Source: authored 2026-04-16
# Entries: 2

$DOMAIN

!dom_print_job|cat:domain|[Fields]id:record<dom_print_job>,org:record<dom_org>,status:record<sys_job_status>|[Behavior]multi_tenant_by_org,audited
!dom_order_line|cat:domain|[Fields]sku:text,description:"Order line, inc. tax",qty:number{min_1}|#note:description_contains_escaped_comma
```

---

## Appendix A — ABNF Grammar

Informative. A ghost file approximates this grammar; a conformant consumer interprets it
leniently (§ 2.5).

```abnf
ghost-file      = header 1*section
header          = hdr-name hdr-purpose hdr-source hdr-entries
hdr-name        = "#" SP 1*VCHAR ".ghost" SP "v" 1*DIGIT ["." 1*DIGIT] [SP "dialect=" 1*LOWER-US] LF
hdr-purpose     = "#" SP "Purpose:" SP 1*VCHAR LF
hdr-source      = "#" SP "Source:" SP 1*VCHAR LF
hdr-entries     = "#" SP "Entries:" SP 1*DIGIT LF

section         = LF section-header *section-comment 1*entry
section-header  = "$" 1*(UPPER / DIGIT / "_") LF
section-comment = "#" *VCHAR LF

entry           = sigil entry-name "|" meta-block *("|" named-block) *annotation LF
sigil           = "!" / ">" / "^" / "@" / "+"
entry-name      = 1*(ALPHA / DIGIT / "_" / ">")

meta-block      = meta-field *("|" meta-field)
meta-field      = meta-key ":" meta-value
meta-key        = "cat" / "scope" / "requires" / "context" / "target" / "status" / ext-key
ext-key         = 1*LOWER-US
meta-value      = 1*VCHAR-UNRESERVED

named-block     = fields-block / behavior-block / returns-block
fields-block    = "[Fields]" item-list
behavior-block  = "[Behavior]" item-list
returns-block   = "[Returns]" item-list
item-list       = item *("," item)

item            = identifier [":" type-or-value] [constraint-set]
identifier      = 1*LOWER-US
type-or-value   = type / quoted-string / literal-value
type            = primitive / generic / reference / compound / union
primitive       = "text" / "number" / "bool" / "date" / "url" / "image"
generic         = identifier "<" type ">"
reference       = identifier "_ref"
compound        = "enum" "{" enum-values "}" / "field_changes" / "key_value_pairs" / "params"
enum-values     = identifier *("/" identifier)
union           = type "|" type *("|" type)

constraint-set  = "{" constraint *("," constraint) "}"
constraint      = 1*VCHAR-CONSTRAINT

quoted-string   = DQUOTE *qchar DQUOTE
qchar           = VCHAR-NOTQUOTE / "\" ("\"" / "\\")
literal-value   = 1*VCHAR-UNRESERVED

annotation      = "|" ("#note:" / "#warn:") 1*VCHAR

LF              = %x0A
SP              = %x20
DQUOTE          = %x22
UPPER           = %x41-5A
LOWER           = %x61-7A
DIGIT           = %x30-39
ALPHA           = UPPER / LOWER
LOWER-US        = LOWER / DIGIT / "_"
VCHAR           = %x20-7E
VCHAR-NOTQUOTE  = VCHAR - DQUOTE
VCHAR-UNRESERVED= VCHAR - ("|" / "," / "[" / "]" / "{" / "}" / DQUOTE)
VCHAR-CONSTRAINT= VCHAR - ("{" / "}" / ",")
```

---

## Appendix B — Quick Reference

```
HEADER (4 lines, exact)
  # <name>.ghost v1 [dialect=<d>]
  # Purpose: <one line>
  # Source: <origin>
  # Entries: <N>

SECTION
  $UPPER_CASE_UNDERSCORED
  #optional comment

ENTRY (one line)
  <sigil><Mixed_Case>|cat:x[|meta:v][|[Fields]a:T{c},b:T][|[Behavior]k,k:v][|[Returns]r:T][|#note:...][|#warn:...]

SIGILS
  !  entity       >  source        ^  operation
  @  event        +  action        ?  query (reserved)   %  metric (reserved)

BLOCK ORDER (mandatory)
  meta → [Fields] → [Behavior] → [Returns] → #note → #warn

TYPES
  text number bool date url image
  expr  expr<T>  list<T>  record<T>  <name>_ref
  enum{a/b/c}  field_changes  key_value_pairs  params
  union: A|B  (inside type positions only)

CONSTRAINTS  {}
  when_X   default:V   requires_X   deprecated_reason
  max_N    min_N       N_to_M
  units: seconds ms minutes hours days pixels em rem percent MB KB bytes degrees

ESCAPING
  \|  \,  \[  \]  \{  \}  \\  \"
  "quoted string with, reserved chars"

ANNOTATIONS
  #note:  informational
  #warn:  risk / destructive
  Notes precede warnings.

NAMING
  Entries   Mixed_Case_Underscored
  Sections  UPPER_CASE_UNDERSCORED
  Fields    lower_case_underscored
  Types     lower_case

VALIDATION (13 checks before delivery — see § 11)
```

---

*End of specification.*
