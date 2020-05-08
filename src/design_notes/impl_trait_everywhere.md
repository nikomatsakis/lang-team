# impl Trait everywhere



## The master example

This shows every place that `impl Trait` could possibly be used, assigns them names, and describes the state.

```rust
type Named = impl Trait; /* Named */

fn foo<T>(
    arg1: impl Trait, /* Argument position */
    arg2: impl FnOnce(
        impl Trait, /* Paren sugar argument position */
    ) -> impl Trait,
) -> impl Trait /* Return position */
where
    T: PartialEq<impl Trait>, /* Where clause position */
    T: PartialEq<Output = impl Debug>, /* Associated type binding position */
    T: FnOnce(
        impl Trait,           /* Paren sugar argument position */
    ) -> impl Trait,          /* Paren sugar return position */
{
    let x: impl Trait = ...;  /* Let position */
}

impl Trait for Type {
    type Bar = impl Trait; /* Associated type value position */

    fn 
}

trait TraitDefinition {
    type Bar: 
        PartialEq<impl Trait>; /* where clause on associated type definition */

    fn foo(
        arg: impl Trait /* Trait method argument position */
    ) -> impl Trait /* Trait method return position */
    where
        T: PartialEq<impl Trait> /* where clause position, trait method */
    ;
}

const MY_CONSTANT: impl Trait = ...;

static MY_STATIC: impl Trait = ...;

impl Trait for impl Type { /* Impl parameter position */
}

struct Foo<T> {
    x: impl Debug /* Struct field position */
}

let x = |
    arg: impl Debug, /* closure argument position */
| -> impl Debug { /* closure return position */
    ...
};

```

## The master table

| Position | 

## The philosophy

### Input/output positions

We have a pre-existing concept called "input" and "output" positions. Various scoping constructs introduce input positions and an optional paired output positions. These scope constructs are also pointers where type and lifetime parameters can be introduced (sometimes with some limitations).

* Function items have their arguments as inputs and their return value as outputs (this is in all places where a function can appear).
* Trait bounds using `()` notation (e.g., `Fn(A..) -> R`) have their argument types `A..` in input position and their return type `R` in output position.
* `fn(A..) -> R` types similarly introduce an input position and an output position.
* Impls like `impl<..> Trait<P1..Pn> for P0` have the trait input types `P0..Pn` in input position and there are no associated output positions.

Note that each of constructs has an associated binder. In the case of `fn`, the binder is presently limited to lifetimes (i.e., `for<'a> fn(&'a i32)` is legal, but `for<T> fn(&T)` is not) for implementation feasibility reasons (this could plausibly be lifted in the future in some cases).

Some items introduce paired "definition" and "selection" scopes:

* In a struct definition like `struct Foo<'a> { x: &'a u32 }` the `'a` is in a "definition" scope and the field type is a "selection" scope.
* The same is true for type aliases `type Foo<'a>`, traits, and so forth.
* Associated type values in traits and impls can be considered as definition scopes as well.
* Function items are the primary exception.

#### Nested scopes

These scopes are nested. So given an example like:

```rust
impl Foo for &u32 {
    type Item = &Blah; // error, see below for why

    fn method(&self, op: fn(&i32) -> &i32) -> &Type {

    }
}
```

We have the following scopes:

* Impl
    * Input scope:
        * `&u32` appears within
    * Definition (`Item`) scope
        * `&Blah` (an error, see below for why)
    * Function  (`method`) scope
        * Inputs:
            * `&self`
            * `fn` type scope
                * Inputs:
                    * `&i32`
                * Output
                    * `&i32`
        * Output:
            * `&Type`

### Relationship to lifetime elision

In general, the notation `'_` for lifetime elision considers the **innermost containing scope**:

* If that scope is in **input position**, then `'_` expands to a fresh lifetime bound on that definition
* If that scope is in **output position**, then `'_` selects a lifetime from the input scope (which must be unique or given precedence for `self` parameter, etc)
* If that scope is a **definition** scope, then `'_` is an error (currently, anyway)

### Relationship to `impl Trait`

The notation `impl Trait` considers the **innermost containing scope** as well:

* If that scope is in **input position**, then `impl Trait` expands to a new existential bound in the scope containing that input position.
* If that scope is in **output position**, then `impl Trait` expands to an existential XXX the binder part is hard to *quite* make it work =)

Expectation:

```rust
fn foo(x: impl FnOnce(&impl Debug) -> impl Debug)
//becomes
fn foo<R: Debug>(x: impl for<A: Debug> FnOnce(&A) -> R)
```

Wait, does that make...any sense...? The argument position does, but the return position does...not. After all, to be consistent, it should be able to "capture" the input types. This should really desugar to something that we can't quite express with our current types. `impl forall<A: Debug> { exists<R: Debug> { FnOnce(A) -> R } }`. Plausibly chalk could accommodate this, but it would require extending with this notion of impl trait, or else having something with HKT like:

```rust
fn foo<R<A>: Debug>(x: impl for<A: Debug> FnOnce(&A) -> R<A>)
```

I think this convinces me that we want to rule this out.

XXX Actually the HKT interpretation -- or maybe "one like it" -- is correct, as is the above "skipping" semantics, if you think *very carefully* about what's going on. Really there are two scopes:

* Item scope (binds existentials)
    * Input scope (binds universals)

and the `-> impl Trait` is binding into that outer scope. This is why we wind up creating `type Foo<'a, T>` etc for each captured parameter.

In other words, these rules I came up with *are* right but...

impl Trait behavior should roughly match the behavior of `'_`:

* In cases where `'_` matches against an existing lifetime (outputs), this means `impl Trait` is "existential" in nature. The "scope" of this existential should be the enclosing scope where the `impl Trait` appears.
* In cases where `'_` introduces a fresh lifetime variable (inputs), this means `impl Trait` is "universal" in nature, and binds to the same place that the lifetime would.
* In cases where `'_` is not allowed, that means:
    * It may be that there is a natural interpretation, but it's kind of confusing for reasons that are unrelated to `impl Trait`. Use the natural interpretation to guide the choice.
    * It may be that the interpretation is not clear for readers, in which case `impl Trait` should also be disallowed.

## Determining the value of an existential type

[RFC 2071] provided the following constraints on how the concrete type for an impl Trait is determined:

* Existential types are similar to normal type aliases, except that their concrete type is determined from the scope in which they are defined (usually a module or a trait impl)
* Each function that references an existential type must **independently constrain it to be the same type**
* Each existential type declaration must be constrained by at least one function body or const/static initializer. A body or initializer must either fully constrain or place no constraints upon a given existential type.
* Existential type aliases cannot be used in impl blocks

### Limiting to higher-ranked pattern matching and defining uses

The implementation added some additional constraints which are required for type inference to be well-defined:

* Any use of the existential type (within its defining scope) must only name generic type parameters that were introduced "in between" the impl trait.

For example:

```rust
type Foo<T> = impl PartialOrd<T>;

// Legal: `U` is a generic type introduced within the scope of `Foo`
fn bar<U>() -> Foo<U> { .. }

// Illegal: `u32` is not a generic type introduced within the scope of the `impl` trait
fn bar() -> Foo<u32> { }
```

Note that no limitation applies to 'non-defining uses' ([playground](https://play.rust-lang.org/?version=nightly&mode=debug&edition=2018&gist=f1eba16f552f27645351489361723684)), but those uses also must treat the type opaquely ([playground](https://play.rust-lang.org/?version=nightly&mode=debug&edition=2018&gist=eaeb58dd92440faa67ae3c98bf152461)).

This is in contrast to what [RFC 2071] specifies:

```rust
fn add_to_foo_2(x: Foo) {
    let x: i32 = x;
    x + 1
}
```

## Implementation limitations

The actual implementation is somewhat different in that it identifies

* Defining use sites

A *defining use site* is currently limited to function return position?

In addition, we encountered some challenges around region inference, giving rise to the addition of member constraints in region inference (tracking issue [#61997]). This has some known complications and limitations around impl Trait for local variable types ([#61773]).

## Unresolved design questions

### Elided lifetimes in impl Trait ([#49287])

What should `fn foo(x: impl Trait<'_>)` mean ([#49287])? Does it mean

* `fn foo<'a>(x: impl Trait<'a>)` or
* `fn foo(x: for<'a> impl Trait<'a>)`

Note that this *is* settled for `impl FnOnce(&u32)`.

Adopting the principles proposed above suggests it means the former, `fn foo<'a>(x: impl Trait<'a>)`, because that is the innermost binding scope for `<>`.

I think the same applies to where clauses like `where T: PartialEq<impl Trait<'_>>` and so forth.

### Interaction of `impl Trait` within `Fn` sugar ([#45994])

Do we permit `fn(impl Trait)`, `dyn Fn(impl Trait)`, or `dyn Fn() -> impl Trait`? ([#45994]) If so, what does it mean?

The principles proposed above actually give us an answer for this:

* `FnOnce(impl Trait)` is desugared to `for<A> FnOnce(A)` (presently disallowed, but eventually permissible).
* `FnOnce() -> impl Trait` is desugared to `FnOnce<..., Output = impl Trait>`, which leads to the `impl Trait` becoming either existential or universal depending on its context.
* `fn(impl Trait)` is an error, because `for<T> fn(T)` is currently not a legal type and we have no plans to make it legal.
* `fn() -> impl Trait`, however, could work.

## Design limitations

### Encapsulating lifetimes that are not named

The current rules prohibit lifetimes from that are not explicitly named, but we sometimes wish to encapsulate lifetimes ([#60670]). For example:

```rust
impl Trait<'b> for Cell<&'a u32> { }
fn foo(x: Cell<&'x u32>) -> impl Trait<'y> where 'x: 'y { x }
```

You can write code like this using `Box<dyn Trait<'y> + 'y>`, but not with `impl Trait`. (The cell is needed here because otherwise lifetime subtyping kicks in and solves it for us.) 

## RFC Record and summary

* [RFC 1522] -- Conservative impl trait
    * Permit `impl Trait` in function argument and return position (excluding trait methods).
    * Estabished auto trait leakage and the basic principle of how the 'existential' type works.
* [RFC 1951] -- Expand impl trait
    * Permit `impl Trait` in function argument position and function return position (excluding trait methods).
    * Designated that all type parameters are in scope for `impl Trait` plus any lifetimes that are explicitly named.
    * Did not propose any means of explicitly specifying the values for `impl Trait` in argument position (turbofish).
* [RFC 2071] -- Impl trait existential types
    * Add the ability to create named existential types and support impl Trait in let, const, and static declarations.
    * Specified details of how existential type inference works
    *  
* [RFC 2515] -- Type alias impl trait

## Implementation summary

[RFC 1522]: https://github.com/rust-lang/rfcs/blob/master/text/1522-conservative-impl-trait.md
[RFC 1951]: https://github.com/rust-lang/rfcs/blob/master/text/1951-expand-impl-trait.md
[RFC 2071]: https://github.com/rust-lang/rfcs/blob/master/text/2071-impl-trait-existential-types.md
[RFC 2515]: https://github.com/rust-lang/rfcs/blob/master/text/2515-type_alias_impl_trait.md
[#61997]: https://github.com/rust-lang/rust/issues/61997
[#61773]: https://github.com/rust-lang/rust/issues/61773
[#60670]: https://github.com/rust-lang/rust/issues/60670
[#49287]: https://github.com/rust-lang/rust/issues/49287
[#45994]: https://github.com/rust-lang/rust/issues/45994

## Testing notes

