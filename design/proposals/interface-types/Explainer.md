# Interface Types Explainer

This explainer introduces the Interface Types proposal, as the second proposal
of the [Component Model] specification, building on the [Module Linking]
proposal. The Interface Types proposal extends the set of types available to
adapter modules to include a new collection of high-level value types
(including strings, lists, records and variants).

This proposal also formally defines the concept of "component" to be an adapter
module that exclusively uses Interface Types in its public interface (imports
and exports).

1. [Problem](#problem)
2. [Use Cases](#use-cases)
3. [Additional Requirements](#additional-requirements)
4. [Walkthrough](#walkthrough)
   1. [Interface Types](#interface-types)
   2. [Interface Types in Imports and Exports](#interface-types-in-imports-and-exports)
   2. [Interface Subtyping](#interface-subtyping)
   3. [Canonical Adapter Functions](#canonical-adapter-functions)
   4. [Components](#components)
5. [Web Platform Integration](#web-platform-integration)  


## Problem

As a compiler target, Core WebAssembly provides only low-level types that aim
to be as close to the underlying machine as possible, allowing each source
language to implement its own unique set of source-language types efficiently.
However, when WebAssembly modules compiled from different languages are linked
together (e.g., using [Module Linking]) and need to communicate through
import/export calls, there remains the question of how to bridge these distinct
representations. The same question arises when WebAssembly modules call
host-defined imports or have their exports called by the host.

Additionally, the only current practical way for WebAssembly modules to pass
compound data types (e.g., strings or lists) across module boundaries is for
both modules to share a single linear memory so that an `i32` pointer can be
passed to the call. While appropriate for language-specific ABI conventions,
this "shared-everything" style of linking conflicts with the Component Model's
high-level [shared-nothing] goals.


## Use Cases

The [Component Model use cases] doc lists a number of relevant use cases for
components that relate to Interface Types. Copying the most directly-relevant
bullets here:

* A JS developer imports a component (via [ESM-integration]) and calls the
  component's exports as JS functions, passing high-level JS values like
  strings, objects and arrays which are automatically coerced according to the
  high-level, typed interface of the invoked component.
* A host defines imports in terms of explicit high-level value types (e.g.,
  numbers, strings, lists, records and variants) that can be automatically
  bound to the calling component's source-language values.
* Developers importing or exporting functions use high-level value types in
  their function signatures that include strings, lists, records, variants and
  arbitrarily-nested combinations of these. Both developers (the caller and
  callee) get to use the idiomatic values of their respective languages. Values
  are passed by copy so that there is no shared mutation, ownership or
  management of these values before or after the call that either developer
  needs to worry about.
* A component runtime implements value passing between component instances
  without ever creating an intermediate O(n) copy of aggregate data types,
  outside of either component instance's explicitly-allocated linear memory.


## Additional Requirements

* plan for multiple representations
  * linear memory vs. GC memory
  * extensibility over time
  * thus need explicit "adapter" that can vary over time
* HOWEVER, don't need to support generic adaptation in the MVP, can start with single "canonical"
  ABI with fixed parameters
  * link to GeneralizedAdapterFunctions.md


## Walkthrough

### Interface Types
 
```
intertype      ::= float32 | float64
                 | s8 | u8 | s16 | u16 | s32 | u32 | s64 | u64
                 | char
                 | (list <intertype>)
                 | (record (field <name> <id>? <intertype>)*)
                 | (variant (case <name> <id>? <intertype>?)*)
                 | <specialization>
specialization ::= string
                 | (tuple <intertype>*)
                 | (flags <name>*)
                 | bool
                 | (enum <name>*)
                 | (option <intertype>)
                 | (union <intertype>*)
                 | (expected <intertype>? (error <intertype>)?)
```

* table of meanings of the non-specializations

```
                                      string ≡ (list char)
                        (tuple <intertype>*) ≡ (record ("𝒊" <intertype>)*) for 𝒊=0,1,...
                             (flags <name>*) ≡ (record (field <name> bool)*)
                                        bool ≡ (variant (case "false") (case "true"))
                              (enum <name>*) ≡ (variant (case <name>)*)
                        (option <intertype>) ≡ (variant (case "none") (case "some" <intertype>))
                        (union <intertype>*) ≡ (variant (case "𝒊" <intertype>)*) for 𝒊=0,1,...
(expected <intertype>? (error <intertype>)?) ≡ (variant (case "ok" <intertype>?) (case "error" <intertype>?))
```

### Interface Types in Imports and Exports

* Reminder from Module Linking:
```
import       ::= (import <name> <def-type>)
type         ::= (type $id <def-type>)
def-type     ::= <instancetype>
               | <moduletype>
               | <core:functype>
               | <core:tabletype>
               | <core:memtype>
               | <core:globaltype>
instancetype ::= (instance <id>? (export <name> <def-type>)*)
moduletype   ::= (module <id>? (import <name> <def-type>)* (export <name> <def-type>)*)
```

* Addition:
```
def-type          ::= ...
                    | <adapter-func-type>
adapter-func-type ::= (func (param <intertype>)* (result <intertype>)*)
```

* Note: will add direct `<intertype>` imports shortly

### Canonical Adapter Functions

## Web Platform Integration



[Component Model]: ../../high-level
[Component Model use cases]: ../../high-level/UseCases.md
[Module Linking]: ../module-linking
[Shared-Nothing]: ../../high-level/Choices.md
[Design Choices]: ../../high-level/Choices.md

[ESM-integration]: https://github.com/WebAssembly/esm-integration
