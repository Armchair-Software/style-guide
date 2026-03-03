# Armchair Software Style Guide

Status: Draft skeleton (phase 1)

Purpose: capture Armchair Software house style from first-party C++ in:

- `annstorm`
- `polymarket-bot1`
- `webgpu-demo3`
- `bigtort`
- `raindrop`
- `anchorhold`

This document is intentionally staged. We first lock the complete category hierarchy, then fill each section with rules + examples + repo evidence from the full file inventory.

## 1. Scope and Authority

### 1.1 Language scope

Rule:

- House style targets modern C++.
- Baseline language expectation is C++23.
- Adopt C++26 features proactively when toolchain and project constraints permit.

### 1.2 In-scope file types (`.cpp`, `.h`)

Rule:

- This guide applies to first-party C++ source and header files.

### 1.3 Out-of-scope code (third-party, vendored, generated)

Rule:

- Third-party and vendored code (for example `include/`) is not style-authoritative.
- Generated files should migrate toward house style where practical, but generator output may temporarily diverge.

### 1.4 Rule priority when conflicts occur

Rule:

- Explicit guide rules override historical inconsistencies.
- When examples disagree, follow the decision captured in this document.

### 1.5 Migration policy for existing inconsistent code

Rule:

- Style fixes are proactive: when touching files, normalize code toward this guide.
- Historical patterns that conflict with this guide should be treated as debt to remove, not precedent.

## 2. Style Guide Method

### 2.1 Evidence sources and audit manifest
### 2.2 Rule format per section
### 2.3 Handling ambiguity and contradictory examples
### 2.4 Enforcement strategy (manual review / tooling / linting)

## 3. File and Project Structure

### 3.1 Project root layout conventions
### 3.2 Directory naming conventions
### 3.3 Module boundaries and folder ownership
### 3.4 Header/source pairing and location
### 3.5 Public vs private headers
### 3.6 Test/source placement conventions
### 3.7 Asset and non-code file placement

## 4. File Naming Conventions

### 4.1 `.cpp` and `.h` filename casing
### 4.2 Acronyms in names
### 4.3 Prefixes/suffixes (`_manager`, `_renderer`, `_types`, `_forward`)
### 4.4 Special filenames (`main.cpp`, `version.h`, `protocol.h`)

## 5. File Prologue and Includes

### 5.1 Header guard strategy (`#pragma once` vs include guards)

Rule:

- Use `#pragma once` for all first-party headers.
- Convert legacy include guards to `#pragma once` proactively when files are touched.
- Do not use dual guard style (`#pragma once` + `#ifndef`) in new or cleaned-up headers.

Evidence:

- `annstorm` and `polymarket-bot1` are consistently `#pragma once`.
- `webgpu-demo3`, `bigtort`, `raindrop`, and `anchorhold` contain mixed legacy patterns, including files that use both forms.

### 5.2 Include block ordering

Rule:

- In `.cpp` files, include the matching local header first when one exists.
- Then apply this include group order:
  - standard language/library headers
  - operating system headers
  - Boost headers
  - other third-party headers (roughly descending significance/notability)
  - internal shared libraries/components (for example `whateverstorm` and similar)
  - internal project general utilities
  - specific functional/class includes

Evidence:

- Matching-header-first is dominant in implementation files.
- A minority of entrypoint files (`main.cpp` style programs) begin with standard library headers.

### 5.3 Grouping: standard, third-party, project-local

Rule:

- Keep include groups visually separated by one blank line.
- Avoid interleaving groups unless required by platform/compiler constraints.
- Prefer alphabetical sorting within each include group.
- Sorting is primarily tooling-enforced, not manual-review-enforced.

### 5.4 Blank lines between include groups

Draft rule:

- Use one blank line between include groups.
- Avoid extra blank lines inside a single include group.

### 5.5 Forward declaration preferences vs includes

Draft rule (provisional):

- Prefer forward declarations in headers when only pointer/reference members are needed.
- Include full headers in `.cpp` where definitions are used.

### 5.6 Include path style (`"..."` vs `<...>`)

Draft rule:

- Use `"..."` for project-local headers.
- Use `<...>` for standard library and external libraries.

## 6. Whitespace and Layout

### 6.1 Indentation width

Draft rule:

- Use 2 spaces per indentation level.
- Do not use 4-space block indentation.
- Continuation lines may use either:
  - vertical alignment to a logical anchor, or
  - one extra 2-space step.

Evidence:

- 2-space block indentation is dominant in all six projects.
- Representative examples:
  - `annstorm/annstorm/neat/network.h`
  - `webgpu-demo3/render/webgpu_renderer.cpp`
  - `bigtort/main.cpp`
  - `raindrop/render/tile_renderer.cpp`
  - `anchorhold/main.cpp`

### 6.2 Tabs vs spaces

Rule:

- Use spaces for indentation in first-party C++.
- Tabs are not idiomatic for first-party `.cpp/.h` files.
- Editor configuration should convert tabs to spaces automatically.

Evidence:

- Tabs found only in `version.h` files generated by version tooling (`bigtort/version.h`, `raindrop/version.h`, `anchorhold/version.h`).
- No tab-indented block style observed in hand-edited first-party `.cpp`/`.h`.

### 6.3 Trailing whitespace policy

Rule:

- No trailing whitespace in first-party hand-authored code.
- Generated files may preserve generator output until generation is standardized.
- Editor configuration should strip trailing whitespace and end-of-file whitespace automatically.

Evidence:

- Trailing whitespace appears only in generated `version.h` files.

### 6.4 Consecutive blank line policy

Draft rule:

- Use a single blank line to separate logical blocks.
- Avoid multiple consecutive blank lines inside function bodies except when separating major phases.

Evidence:

- Typical style in all target repos uses single blank-line separators between blocks/declarations.

### 6.5 Horizontal alignment policy

Rule:

- Alignment is allowed for local readability when grouping related declarations.
- Alignment is encouraged when it clearly improves readability.

Observed patterns:

- Aligned assignment blocks (for example UI state toggles) are common:
  - `raindrop/gui/world_gui.cpp`
  - `anchorhold/gui/world_gui.cpp`
- Aligned end-of-line comments are also common:
  - `polymarket-bot1/main_btc_price_orderbook_strategy_bot.cpp`
  - `webgpu-demo3/render/webgpu_renderer.cpp`

### 6.6 Line length policy and exceptions

Rule:

- There is no hard line-length limit.
- Prioritize readability over numeric width targets.
- Avoid aggressively splitting readable long lines just to satisfy an arbitrary length.
- Allow longer lines for:
  - URLs and machine-oriented strings,
  - generated/string-table content,
  - dense expressions or comments where wrapping harms readability.

Evidence:

- Current codebase has many lines >120 columns, including legitimate long-string and table cases.
- Extreme outliers are concentrated in data-heavy declarations (for example spinner tables).

### 6.7 Long statement wrapping and continuation indentation

Rule:

- Prefer operator-leading or operator-trailing wrapped chains with consistent continuation indentation.
- For stream chains and builder-like code, keep one operator per line after wrap.
- Keep wrapped initializer/designated-initializer blocks vertically readable.

Example:

```cpp
logger << "DEBUG: on_close, event_type " << event_type
       << ", websocket " << event->socket
       << " closed" << (event->wasClean ? " cleanly" : ", not cleanly")
       << ", status: " << closeevent_code_to_text(event->code)
       << ", reason: \"" << event->reason << "\"";
```

(from `anchorhold/main.cpp`)

### 6.8 Editor behavior and defaults

Rule:

- Configure editors to:
  - use 2-space tab width
  - insert spaces instead of tab characters
  - strip trailing whitespace and EOF whitespace on save

## 7. Braces and Block Formatting

### 7.1 Function/class/struct brace placement

Draft rule:

- Use attached braces for functions, classes, structs, and namespaces.

Example:

```cpp
class network {
public:
  void run();
};
```

### 7.2 `if/else` brace and `else` placement

Draft rule:

- Opening brace stays on the same line as the condition.
- Place `else` as `} else {`.

Evidence:

- Same-line control braces are dominant across all six projects.
- `else` overwhelmingly appears on the same line as the prior closing brace.

### 7.3 Loop/switch brace style

Draft rule:

- Use attached braces for `for(...)`, `while(...)`, and `switch(...)`.
- Keep `case` labels at the same indentation level as `switch`.
- Indent statements inside each `case` by one level.
- Add braces inside a `case` only when needed to control scope (for example local variable lifetime).

Example:

```cpp
switch(mode) {
case mode_type::a:
  {
    int value{1};
  }
  break;
case mode_type::b:
  execute();
  break;
}
```

### 7.4 Namespace brace style and closing comments

Draft rule (provisional):

- Prefer a closing namespace comment for non-trivial scopes:
  - `} // namespace module::submodule`
- Small/local namespaces may omit this when still clear.

### 7.5 Lambda brace style

Draft rule:

- Use attached braces for lambda bodies.
- Format lambda internals the same way as ordinary function bodies.

Example:

```cpp
auto predicate{[&](int value) {
  if(value < 0) return false;
  return true;
}};
```

### 7.6 Single-line statement policy

Draft rule:

- Single-line guard clauses are acceptable for simple exits:
  - `if(!ready) return;`
- Single-line bodies are also acceptable for simple assignment/function-call bodies when readability stays high.
- Use braces once a branch has multiple statements.

### 7.7 What does not get indented

Rule:

- Do not indent:
  - `public:`, `private:`, `protected:` labels
  - `case` labels under `switch`
  - primary content inside `namespace` scopes

### 7.8 Conditional indentation in special blocks

Rule:

- Indentation may still be used inside preprocessor-controlled blocks and diagnostic blocks when it improves readability:
  - code inside `#ifdef` regions is often indented
  - code inside `#pragma GCC diagnostic push` regions may be indented when helpful

## 8. Spacing Rules

### 8.1 Control statement spacing (`if(` vs `if (` etc.)

Draft rule:

- Use no space before `(` in control statements:
  - `if(...)`, `for(...)`, `while(...)`, `switch(...)`, `catch(...)`

Evidence:

- Strong majority follows this style.
- A small number of `catch (...)` and `if (...)` exceptions exist and should be normalized.

### 8.2 Function declaration/definition spacing

Draft rule:

- No space between function name and `(` in declarations and definitions.
- Trailing return type examples use `auto f()->T` without spaces around `->`.

### 8.3 Operator spacing (arithmetic, assignment, comparison, logical)

Draft rule:

- Use spaces around binary operators.
- Keep unary operators compact (`++i`, `!flag`).

### 8.4 Comma/semicolon/colon spacing

Draft rule:

- One space after commas.
- No space before commas or semicolons.

### 8.5 Pointer/reference spacing

Draft rule:

- Bind pointer/reference symbols to the type side:
  - `type *ptr`
  - `type &ref`
  - `type const &value`

Evidence:

- This style is dominant in all six codebases.

### 8.6 Template angle bracket spacing

Draft rule:

- No extra spaces inside template angle brackets.

### 8.7 Cast spacing

Draft rule:

- Prefer named casts with regular call spacing:
  - `static_cast<T>(value)`

### 8.8 Multi-line comma-separated list formatting

Rule:

- For function parameter lists and similar call-like constructs, continuation alignment under the opening `(` is acceptable and common.
- For most other brace-based lists (especially aggregate/initializer content), prefer:
  - opening brace on the introducing line
  - one element per subsequent line
  - one indentation level for each element

## 9. Naming and Case Conventions

### 9.1 Namespace names

Draft rule:

- Use lowercase snake_case namespace names.
- Prefer nested namespace syntax where suitable (`namespace annstorm::neat {`).

### 9.2 Type names (class/struct/enum/alias)

Draft rule:

- Use lowercase snake_case for project-defined type names.
- `enum class` names follow the same pattern (`exposure_type`, `timescale`).

### 9.3 Function and method names

Draft rule:

- Use lowercase snake_case (`get_open_price`, `apply_level_update`, `read_output`).

### 9.4 Variable names

Draft rule:

- Use lowercase snake_case for local variables and non-member state.
- Allow short loop counters (`i`, `x`, `y`, `z`) in narrow scopes.
- Otherwise, variable names must be meaningful and legible.
- Avoid terse acronyms and short opaque names except where representing true mathematical symbols.

### 9.5 Member names

Draft rule:

- Use lowercase snake_case without `m_`/`_` prefixes.
- Keep member names identical to conceptual domain terms.

### 9.6 Constants and `constexpr` names

Draft rule:

- Use lowercase snake_case for `constexpr` and `const` object names.
- Macro-style uppercase is reserved for preprocessor constants.

### 9.7 Macro names

Draft rule:

- Use `UPPER_SNAKE_CASE` for preprocessor symbols (`DEBUG_WEBGPU`, `BOOST_NO_EXCEPTIONS`).

### 9.8 Enum value names

Draft rule:

- Use lowercase snake_case enumerators (`none`, `up`, `down`, `diagonal`).

### 9.9 Template parameter names

Draft rule:

- Use short UpperCamel or single-letter template parameter names (`T`, `FromT`, `Tcpp`, `Tc`).

### 9.10 Namespace usage rules

Rule:

- Never use `using namespace ...` for non-literal namespaces.
- `using namespace std::chrono_literals;` and similar literal namespaces are allowed in source files when they improve readability.
- Do not place `using namespace ...literals` in headers, to avoid namespace pollution for includers.
- Namespace aliases are permitted only when they clearly improve legibility and do not introduce ambiguity.

## 10. Declarations and Definitions

### 10.1 `auto` usage policy

Draft rule:

- Use `auto` when type is obvious from initializer or when avoiding repetition improves clarity.
- Avoid `auto` when it obscures important concrete type semantics.

### 10.2 Trailing return type style (`auto f()->T`)

Draft rule:

- Trailing return style is allowed.
- It is encouraged for entry points and callback-heavy signatures.
- Do not use trailing return style when it makes a declaration more verbose than a direct return type.

### 10.3 Reference and const placement style

Draft rule:

- Use east-const with left-bound reference/pointer symbols:
  - `type const &value`
  - `type *ptr`

### 10.4 Declaration ordering inside classes

Rule:

- Keep access sections explicit (`public`, `private`, `protected`).
- Preferred class member order:
  - external references set at initialization (injected dependencies, handles)
  - nested class/struct/enum definitions (except those tightly associated with a specific member)
  - member objects and variables
  - constructors and destructor
  - other member functions
  - friend declarations
- Preserve pragmatism for data layout/packing when ordering member objects.

### 10.5 Initializer list formatting

Draft rule:

- For multi-member constructor initialization, place each initializer on its own line under `:`.

### 10.6 Default member initialization

Draft rule:

- Prefer in-class member initialization with braces for defaults.

### 10.7 `using` aliases and `typedef` policy

Draft rule:

- Prefer `using` over `typedef`.
- Use alias names that preserve existing lowercase snake_case conventions.

### 10.8 `static`/type/`constexpr`/`const` ordering

Rule:

- Prefer declaration ordering:
  - `static` first (if present)
  - type
  - `constexpr` or `const`
  - pointer/reference marker
  - variable name

Example:

```cpp
static std::chrono::seconds constexpr update_interval{5s};
std::string const &name_ref{source.name};
```

## 11. Initialization and Expressions

### 11.1 Uniform initialization (house preference: universal braces)

Rule (confirmed by owner preference):

- Use brace initialization as the default for all declarations.
- This applies to scalars, class types, aggregates, containers, and temporaries where practical.
- Existing declaration-site `=` initialization in legacy files is non-idiomatic and should be migrated when touched.

Example:

```cpp
int retries{3};
std::string name{"anchorhold"};
std::chrono::seconds timeout{5s};
```

### 11.2 Assignment vs initialization distinctions

Rule:

- Use `=` primarily for assignment after declaration.
- At declaration site, prefer braces unless language constraints require another form.

### 11.3 Designated initializers

Rule:

- Use designated initializers wherever possible for aggregate setup.
- Keep one designated field per line for multi-field initialization.

Example:

```cpp
result = item{
  .id{source.id},
  .name{source.name},
};
```

### 11.4 Literal formatting (`1.0f`, digit separators, chrono literals)

Draft rule:

- Use explicit suffixes where type clarity matters (`1.0f`, `5s`).
- Use digit separators for large literals when it improves readability (`100'000'000.0f`).

### 11.5 Cast policy (`static_cast`, C-style cast restrictions)

Rule:

- Prefer named casts (`static_cast`, `reinterpret_cast`, etc.).
- C-style casts must be proactively removed from first-party C++.
- When encountered, convert to the appropriate named cast.

Current migration targets identified during scan:

- `vectorstorm/matrix/matrix3.h` and `vectorstorm/matrix/matrix4.h` (`(T)` casts in assignment operators, mirrored across projects)
- `memorystorm/memorystorm.cpp` (`(UINT_PTR)` casts, mirrored across projects)

### 11.6 Ternary operator formatting

Draft rule:

- Keep short ternaries inline.
- For long ternaries, wrap cleanly and keep branches simple enough to scan.

### 11.7 Boolean expression clarity

Rule:

- Prefer explicit boolean conditions and guard clauses.
- Avoid deeply negated or dense chained expressions without intermediate naming.

Example (prefer):

```cpp
bool const timed_out{now >= deadline};
bool const has_capacity{queue_size < max_queue_size};
if(timed_out || !has_capacity) return false;
```

Example (avoid):

```cpp
if(!(now < deadline && (queue_size < max_queue_size || allow_overflow))) {
  return false;
}
```

## 12. Control Flow Style

### 12.1 Early exit vs deep nesting

Draft rule:

- Prefer early exits (`return`, `continue`) to reduce nested indentation.
- Use guard clauses for invalid/empty/error states.
- Keep scopes as small as practical; create inner scopes in long functions to limit object lifetimes.

### 12.2 Condition complexity thresholds

Draft rule:

- Keep complex conditions readable; extract named booleans when expressions become dense.
- Prefer `if` init-statements when a temporary is only needed for the condition/body.

### 12.3 `switch` style and exhaustiveness handling

Draft rule:

- Prefer `switch` for closed enum dispatch.
- Avoid silent fallback behavior; enforce exhaustiveness where practical.
- When intentionally exhaustive, omit `default` and document intent.

### 12.4 Loop form preferences (`for`, range-for, `while`)

Draft rule:

- Prefer range-for when iterating container elements directly.
- Use indexed `for` where indices are semantically meaningful.
- Use `while` for open-ended loops tied to runtime state flags.
- Prefer modern ranges/views/zip-style iteration helpers over manual iterator increment code when viable.

### 12.5 `continue`/`break` usage

Rule:

- For unsigned index iteration, prefer `!= end` termination when iterating over known bounds.
- Use `continue` for explicit fast-path skips when it simplifies loop structure.
- Use `break` when explicit early termination improves clarity over condition mutation.

## 13. Functions and APIs

### 13.1 Function size and responsibility
### 13.2 Parameter passing conventions
### 13.3 Output parameters and return values
### 13.4 Overload design rules
### 13.5 `[[nodiscard]]` usage
### 13.6 `noexcept` guidance

## 14. Classes, Structs, and OOP

### 14.1 `struct` vs `class` usage
### 14.2 Access section ordering
### 14.3 Constructor and destructor conventions
### 14.4 Rule of 0/3/5 defaults
### 14.5 Inheritance formatting
### 14.6 Virtual and override conventions

## 15. Templates and Generic Code

### 15.1 Template declaration formatting

Rule:

- Use compact template declaration formatting:
  - `template<typename T>`
- Keep template parameter lists readable; wrap only when needed.

### 15.2 Constraints/concepts style

Rule:

- Prefer modern constraints/concepts when they improve correctness and readability.
- Avoid verbose SFINAE patterns when a modern alternative is available.

### 15.3 Specialization formatting

Rule:

- Keep specializations visually close to primary templates where practical.
- Preserve the same naming and formatting style as primary template definitions.

### 15.4 Generic naming conventions

Rule:

- Use conventional short template parameter naming (`T`, `FromT`, `Tcpp`, etc.) unless a semantic name improves clarity.

### 15.5 Modern C++ feature preference

Rule:

- Prefer the most modern C++ feature set available to the target toolchain (C++23 baseline, C++26 adoption when viable).
- Prefer modern constructs over legacy equivalents when readability and maintainability improve.

## 16. Lambdas and Callables

### 16.1 Capture style

Rule:

- Capture only what is needed.
- Prefer explicit captures over broad `[=]`/`[&]` in complex contexts.

### 16.2 Parameter and return formatting

Rule:

- Follow normal function formatting for lambda parameters and return type annotations.
- Omit empty `()` for parameterless lambdas (modern C++ style).

Example:

```cpp
auto tick{[] {
  return true;
}};
```

### 16.3 Multi-line lambda formatting

Rule:

- Use attached braces and normal block indentation.
- For lambdas passed inline, wrap arguments cleanly and keep the lambda body readable.

### 16.4 Callback conventions

Rule:

- Use trailing return types on callbacks when clarity improves (especially in complex signatures).
- Keep callback side effects explicit and small where possible.

## 17. Attributes, Macros, and Preprocessor

### 17.1 Standard attributes (`[[nodiscard]]`, `[[maybe_unused]]`, `[[likely]]`)

Draft rule:

- Use standard attributes where they communicate API intent or branch likelihood.
- Place attributes immediately before the declaration they modify.

### 17.2 Compiler attributes (`[[gnu::pure]]`, etc.)

Draft rule:

- Compiler attributes are encouraged when likely to be useful.
- Prefer GNU-compatible attributes because target toolchains are GCC and Clang.
- Use non-GNU attribute forms only when strictly necessary for the task.

### 17.3 `#define` naming and scope

Draft rule:

- Use `UPPER_SNAKE_CASE` macro names.
- Keep debug feature toggles local to file scope when possible.
- Avoid object-like macros for values that can be `constexpr`.

### 17.4 Conditional compilation (`#if`, `#ifdef`, `#ifndef`) style

Draft rule:

- Use `#ifdef`/`#ifndef` for feature flags and debug toggles.
- Keep branch bodies indented to surrounding block style.

### 17.5 Indentation and comments for preprocessor blocks

Draft rule:

- `#if`/`#ifdef` directives may be indented to match surrounding code.
- Close conditionals with explicit comments:
  - `#endif // DEBUG_FEATURE`

Evidence:

- End-of-block comments on `#endif` are the dominant style in all six projects.

Example:

```cpp
#ifdef DEBUG_WEBGPU
  logger << "DEBUG: Adapter info: " << adapter_info.description;
#endif // DEBUG_WEBGPU
```

### 17.6 Debug flag conventions

Draft rule:

- Use `DEBUG_*` naming for file/module debug toggles.
- Keep toggles near file top for discoverability.

## 18. Comments and Documentation

### 18.1 Line comments vs block comments

Rule:

- Prefer `//` comments over block comments for most code documentation.
- Use block comments sparingly (large doc blocks or temporarily disabled regions during active refactor).

### 18.2 API doc comments (`///`)

Rule:

- Use `///` for concise API intent and behavioral notes in headers and key implementation points.

### 18.3 Inline rationale comments

Rule:

- Prefer same-line comments where appropriate.
- Align inline comments to start at column 80 when practical.
- If code already extends beyond column 80, place comment after one space rather than forcing a wrap.
- Use shared tooling for alignment when possible:
  - `/home/slowriot/code/scripts/comment_aligner.sh`
  - `/home/slowriot/code/scripts/comment_aligner_project.sh`
  - `/home/slowriot/code/scripts/comment_aligner_cmake.sh`

### 18.4 TODO/FIXME/HACK conventions

Rule:

- Keep TODO-style comments specific and actionable.
- Prefer brief reason + intent over vague placeholders.

### 18.5 Commented-out code policy

Rule:

- Avoid retaining commented-out code long-term.
- If temporarily kept during active change, annotate intent and remove promptly.

## 19. Error Handling and Diagnostics

### 19.1 Exception use and catch formatting
### 19.2 Error logging style
### 19.3 Assertions and defensive checks
### 19.4 Failure message quality and consistency

## 20. Concurrency and Threading Style

### 20.1 Thread lifecycle conventions
### 20.2 Synchronization primitive usage
### 20.3 Atomic usage and memory-order style
### 20.4 Queue/event loop patterns

## 21. CMakeLists.txt Style (Brief)

### 21.1 Indentation and list formatting

Draft rule:

- Use 2-space indentation inside command argument blocks.
- Keep long lists (`set(...)`, `add_executable(...)`, warning flags) one item per line.
- Use uppercase command names consistently as already established in these repos.

### 21.2 Condition and build-type block style

Draft rule:

- Normalize build type with `string(TOLOWER ...)` and branch on explicit values.
- Keep one responsibility per conditional block (build mode, exception mode, feature toggles).
- Emit clear `message(STATUS ...)` lines for selected mode.

### 21.3 Target declarations and grouping

Draft rule:

- Group sources by role using comments:
  - project-specific
  - shared/internal libs
  - third-party
- Keep `target_link_libraries`, `target_compile_options`, and `target_link_options` in separate clear blocks.

### 21.4 Compile/link options layout

Draft rule:

- Keep compile/link options in named variables when reused.
- Keep warnings and optimization flags grouped with comment headers for readability.

## 22. Bash Script Style (Brief)

### 22.1 Shebang and safety defaults

Draft rule:

- Use `#!/bin/bash`.
- Prefer explicit failure checks (`|| exit 1`) for critical commands.
- For user-facing build scripts, preserve existing explicit flow over implicit shell options.

### 22.2 Variable naming and quoting

Draft rule:

- Use lowercase snake_case for script variables (`build_dir`, `compiled_resources_total`).
- Quote variable expansions unless intentional word-splitting is required.

### 22.3 Conditionals, loops, and spacing

Draft rule:

- Use `[ ... ]` condition style consistently with spaces around brackets.
- Use straightforward `for` loops over arrays for file batches.
- Keep condition bodies compact and readable.

### 22.4 Command failure handling

Draft rule:

- Use `|| exit 1` on build-critical commands (`cmake`, `make`, resource compilation).
- Check presence of optional tools before use (`which ccache`, `which naga`).

### 22.5 Logging output conventions

Draft rule:

- Use short, explicit `echo` status messages.
- Preserve the current convention of redirecting stdout to stderr for script status logs where used (`exec 1>&2`).

## 23. Tooling and Enforcement

### 23.1 `clang-format` policy

Rule:

- Maintain a project `clang-format` configuration aligned with this guide.
- Use tooling for mechanical consistency (especially include sorting and routine wrapping), while preserving readability-first exceptions.

### 23.2 `clang-tidy` policy

Rule:

- Use `clang-tidy` checks to enforce modernization and safety where practical.
- Treat C-style cast detection and modernization checks as proactive cleanup targets.

### 23.3 CI validation hooks

Rule:

- Prefer automated checks for style-invariant rules (include order, whitespace, forbidden constructs).

### 23.4 Review checklist

Rule:

- Human review should focus on readability and intent, not mechanical formatting already handled by tooling.

### 23.5 Editor configuration baseline

Rule:

- Editors should be configured to:
  - use 2-space indentation
  - convert tabs to spaces
  - trim trailing and EOF whitespace on save

## 24. Provisional Baseline (from current audit; pending confirmation)

```cpp
namespace example {

struct run_state {
  bool keep_running{true};
  std::chrono::seconds timeout{5s};
};

auto process_item(item const &source, item &target)->bool {
  if(!source.valid()) return false;

  target = item{
    .id{source.id},
    .name{source.name},
  };

  return true;
}

} // namespace example
```

Notes:

- Uses 2-space indentation.
- Uses attached braces.
- Uses no space before control parentheses.
- Uses reference style `type const &name`.
- Uses brace initialization and designated initializers.

## 25. Resolved Decisions (2026-03-03 Review)

1. Control keywords use attached parentheses: `if(`, `for(`, `catch(`.
2. Headers use `#pragma once`; legacy include guards should be proactively migrated.
3. Include order is standardized with explicit group tiers from local header to specific project includes.
4. No hard line-length limit; readability first, and avoid forced wrapping.
5. Vertical alignment is encouraged when it improves clarity.
6. `switch`/`case` formatting uses case labels aligned with `switch`, with case body statements indented.
7. Inline comments are preferred where appropriate and aligned to column 80 when practical.
