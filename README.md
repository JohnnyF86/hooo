# Hooo
Propositional logic with exponentials

To install Hooo, type:

```text
cargo install --example hooo hooo
```

Then, to check a file with Hooo, type:

```text
hooo <file.hooo>
```

### Example: Absurd

```text
fn absurd : false -> a {
    x : false;
    let r = match x : a;
    return r;
}
```

### Working with projects

To check a whole project, type:

```text
hooo <project directory>
```

Hooo will generate a "Hooo.config" file, which you can modify to use other libraries:

```text
dependencies {
    path("../my_library");
}
```

By default, Hooo adds a "std" library which you can use in the following way:

```text
use std::not_double;
```

This will add a term `not_double : a -> !!a`.

Hooo generates a file "hooo-meta_cache.bin" that stores data to make repeated checks faster.
This will usually just take a few MB of storage.
If you want a clean check, then delete this file.

## Introduction to Hooo

Intuitionistic Propositional Logic (IPL) is the most important mathematical language
in the world, because it is the foundation for constructive logic and many type systems.

Usually, IPL is thought of as a "simple language" that is generalized in various ways.
For example, by adding predicates, one gets First Order Logic.

Previously, IPL was thought of as "complete" in the sense that it can derive every
formula that is true about propositions.
To reason about IPL at meta-level, mathematicians relied on some meta-language (e.g. Sequent Calculus)
or some modal logic.

For example, in Provability Logic, `□(a => b)` is introduced by proving `⊢ a => b` in Sequent Calculus.

Recently, I discovered that, while logical implication (`=>`) in IPL corresponds to lambda/closures,
there is no possible way to express the analogue of function pointers (`->`).
People thought previously that, since logical implication is a kind of exponential object,
that IPL covered exponentials in the sense of Category Theory.
However, there can be more than one kind of exponential!
The extension of IPL to include function pointers is called "exponential propositions".

Exponential propositions allows a unification of the meta-language of IPL with its object-language.
This means that IPL in its previous form is "incomplete", in the sense that there are no ways
to express exponential propositions.

Hooo finalizes intuitionistic logic by introducing exponential propositions (HOOO EP).
This solves the previous problems of using a separate meta-language.
Inference rules in Hooo are first-class citizens.

There is no separation between the language and meta-language in Hooo.
All types, including rules, are constructive propositions.

### Relation to Provability Logic

HOOO EP might sound like Provability Logic, but it is not the same thing.
Provability Logic is incompatible with HOOO EP, since Löb's axiom is absurd.
For proof, see `lob_absurd` in "source/std/modal.hooo".

# HOOO EP

"HOOO EP" stands for "Higher Order Operator Overloading Exponential Propositions".

HOOO EP was first developed in the [Prop](https://github.com/advancedresearch/prop) library,
which exploited function pointers in Rust's type system.

There are 3 axioms in HOOO EP:

```text
pow_lift : a^b -> (a^b)^c
tauto_hooo_imply : (a => b)^c -> (a^c => b^c)^true
tauto_hooo_or : (a | b)^c -> (a^c | b^c)^true
```

The philosophy of HOOO EP is that the axioms are intuitive.
This is how people can know that the axioms can be trusted.

From these axioms, there are infinitely complex logical consequences.
It is important to keep the axioms few and simple to not cause trouble later on.

However, some statements many people find "obviously true" can be difficult to prove.
This is why a library is needed to show how to prove such statements.

For example, in Hooo, one can prove that function composition is possible:

```text
fn pow_transitivity : b^a & c^b -> c^a {
    use hooo_imply;
    use pow_lift;
    use refl;
    use hooo_rev_and;

    x : b^a;
    y : c^b;

    fn f : a -> ((b^a & c^b) => c) {
        x : a;

        lam g : (b^a & c^b) => c {
            y : b^a;
            z : c^b;

            let x2 = y(x) : b;
            let x3 = z(x2) : c;
            return x3;
        }
        return g;
    }

    let x2 = hooo_imply(f) : (b^a & c^b)^a => c^a;
    let x3 = pow_lift(x) : (b^a)^a;
    let y3 = pow_lift(y) : (c^b)^a;
    let x5 = hooo_rev_and(x3, y3) : (b^a & c^b)^a;
    let x6 = x2(x5) : c^a;
    return x6;
}
```

Notice how complex this proof is compared to proving lambda/closure composition:

```text
fn imply_transitivity : (a => b) & (b => c) -> (a => c) {
    x : a => b;
    y : b => c;
    lam f : (a => c) {
        arg : a;
        let x2 = x(arg) : b;
        let x3 = y(x2) : c;
        return x3;
    }
    return f;
}
```

Intuitively, function composition is possible, so most people just take it for granted.
Previously, mathematicians needed a meta-language to prove such properties.
Now, people can prove these properties directly using Hooo-like languages.

Hooo is designed for meta-theorem proving in constructive/intuitionistic logic.
This means that Hooo can reason about its own rules.
From existing rules, you can generate new rules, by leveraging the full
power of meta-theorem proving in constructive logic.

An exponential proposition is a function pointer, or an inference rule.
You can write the type of function pointers as `a -> b` or `b^a`.

## Syntax

```text
a -> b     Exponential/function pointer/inference rule
b^a        Exponential/function pointer/inference rule
a => b     Imply (lambda/closure)
a & b      And (struct type)
a | b      Or (enum type)
!a         Not (sugar for `a => false`)
a == b     Equal (sugar for `(a => b) & (b => a)`)
a =^= b    Pow equal (sugar for `b^a & a^b`)
excm(a)    Excluded middle (sugar for `a | !a`)
sd(a, b)   Symbolic distinction (see section [Symbolic distinction])
~a         Path semantical qubit (see section [Path Semantics])
a ~~ b     Path semantical quality (see section [Path Semantics])
(a, b)     Ordered pair (see section [Avatar Logic])
true       True (unit type)
false      False (empty type)
all(a)     Lifts `a` to matching all types
□a         Necessary `a` (modal logic)
◇a         Possibly `a` (modal logic)
foo'       Symbol `foo`
foo'(a)    Apply symbol `foo` to `a`

x : a      Premise/argument
let y      Theorem/variable

return x   Helps the solver make a conclusion

sym foo;                    Declare a symbol `foo'`.
axiom foo : a               Introduce axiom `foo` of type `a`
() : a                      Prove `a`, e.g. `() : true`
f(x)                        Apply one argument `x` to `f`
f(x, y, ...)                Apply multiple arguments
match x : a                 If `x : false`
match x (l, r) : c          If `x : (a | b)`, `l : a => c` and `r : b => c`
use <tactic>                Imports tactic
fn <name> : a -> b { ... }  Function
lam <name> : a => b { ... } Lambda/closure
```

The `^` operator has high precedence, while `->` has low precedence.

E.g. `b^a => b` is parsed as `(b^a) => b`.
`b -> a => b` is parsed as `b -> (a => b)`.

### Symbols

The current version of Hooo uses simple symbols.
This means that instead of declaring data structures,
one can just use e.g. `foo'(a, b)`.
A symbol must be declared before use:

```text
sym foo;
```

Symbols are global, so `foo'` is `foo'` everywhere.

### Congruence

An operator `f` is normal congruent when:

```text
a == b  ->  f(a) == f(b)
```

An operator `f` is tautological congruent when:

```text
(a == b)^true -> f(a) == f(b)
```

Most operators are normal congruent.
Some theories treat all operators within the theory
as normal congruent, e.g. predicates in First Order Logic.

However, in [Path Semantics](https://github.com/advancedresearch/path_semantics/tree/master) there
are some operators which are tautological congruent.

For example, path semantical quality and symbolic distinction are tautological congruent.

### Symbolic distinction

Paper: [Symbolic distinction](https://github.com/advancedresearch/path_semantics/blob/master/papers-wip2/symbolic-distinction.pdf).

In normal logic, there is no way to distingish propositions from each other
without adding explicit assumptions.
With other words, logic is not allowed to know internally
the accurate representation of data.
This is because logic is used to reason hypothetically.

For example, you know that the symbol `x'` is identical to `x'`.
Yet, logic can not know this internally, because it would lead to unsoundness.
If one has two indistinct symbols which are non-equal, then this would be unsound in logic.

You must always be allowed to treat propositions as equal,
although they are strictly symbolic distinct:

- `a == b` (normal equality)
- `(a == b)^true` (tautological equality)

There should be no stronger notion of equality in logic than tautological equality.
Most operators are congruent by normal equality,
but a few operators are only congruent by tautological equality.

However, there are no problems in logic by using symbolic distinction.

- `sd(a, b)` (`a` and `b` are symbolic distinct)

For example, in Hooo, one can distinguish two symbols `foo'` and `bar'`:

`let x = () : sd(foo', bar');`

This is useful when you are working with theories that need
some form of uniqueness.

For example, in Type Theory, a member of a type is
only allowed to have a unique type.
When we work with Type Theory in logic,
we are allowed to assume that two types are equal,
but the theory decides whether this is permitted.
The theory can not prevent us from making assumptions,
but it can control the consequences.
Without symbolic distinction, there is no way to tell
in logic that two types are not permitted to be equal.

Symbolic distinction allows you to add axioms for such cases
and still be able to reason hypothetically.

### Path Semantics

[Path Semantics](https://github.com/advancedresearch/path_semantics)
is an expressive language for mathematical programming.

Mathematical programming usually deals with higher dimensions compared to normal programming.
In normal programming, you have just one dimension of evaluation.
In higher dimensional programming, you can have multiple dimensions of evaluation.
It is not as simple as parallelism, because you can evaluate functions as boundaries of a "function surface".
Such surfaces, called "normal paths", must be treated as mathematical objects on their own,
which requires a foundation of higher dimensional programming.

For example:

```text
len(concat(a, b)) <=> add(len(a), len(b))
```

To write this as a normal path:

```text
concat[len] <=> add
```

In Category Theory, `concat[len]` is an "open box" which is closed by `add`.

Now, since `add[even] <=> eqb`, one can prove:

```text
concat[len][even] <=> add[even]
concat[len][even] <=> eqb
concat[even . len] <=> eqb
```

The idea is that normal paths compose in an "orthogonal dimension" to normal computation.
One uses this notation because it does not require variables,
hence being a style of "point-free" theorem proving.

The foundation of higher dimensional programming is notoriously hard,
so you should not feel bad if you do not understand the entire theory.
There is a lot to take in, so take your time.

Path Semantics is just one approach to higher dimensional programming.
Another approach is Cubical Type Theory.

#### Changing perspective of type hierarchies

Since higher dimensions often explode in combinatorial complexity,
there is no way we can explore the entire space of possibilities.
Therefore, we have to be satisfied with proving some properties
across multiple dimensions.

Naturally, we are used to think of types as a way of organizing data. However, if you have a powerful logic
such as HOOO EP, then it also makes sense to reason
about type hierarchies as "moments in time".
Each moment in time is a local space for reasoning in IPL.

Path Semantics at its fundamental level is the mechanism that allows propositions to transition
from one moment in time to another moment.

This is expressed in the core axiom of Path Semantics:

```text
ps_core : (a ~~ b) & (a : c) & (b : d) -> (c ~~ d)
```

The operator `~~` is path semantical quality,
which is a partial equivalence that lifts bi-conditions using symbolic distinction:

```text
q_lift : sd(a, b) & (a == b) -> a ~~ b
```

This means, that when one assumes equality in some
moment of time between two propositions that can be
proved to be symbolic distinct, one is allowed to
introduce quality and hence get a way to propagate
new quality propositions in future moments.

The reason for this mechanism is that in any moment of time,
the information is relative to its internal state.
So, there must be at least two separate physical states
to store information from the past.
Path semantical quality is a logical model of this phenomena.

If you find this hard to think about, then you can just
use the thumb rule "if two symbols are qual `a ~~ b`, then their meanings `(a : c) & (b : d)` are qual `c ~~ d`". This is a proper way of handling semantics of symbols in logic. Notice that you should not use "equal" because reflexivity and logical implication makes this unsound. The `~~` operator is sometimes thought of as a path, so the meaning of symbols using paths was why this field became "Path Semantics".

In philosophy, path semantical quality is closely
related to Hegel's philosophy of Being.
Hegel's philosophy was rejected by Russel,
who founded analytic philosophy.
This turned out to be a mistake, likely because
Russel was influenced by the language bias of First Order Logic, where all predicates are normal congruent.
By using tautological congruence, one can reason about
Hegel's philosophy just fine.
However, this requires HOOO EP.

Self quality `a ~~ a` is equivalent to `~a`,
which is called a "qubit".
In classical logic, one generates a random truth table of `~a` using `a` as the seed to the random generator.
This makes `~a` behave as it is in super-position of all propositions,
hence the name "qubit".
One can also think about it as introducing a new proposition.

Path semantical quality and qubit are tautological congruent.

### Uniqueness, solutions and the imaginary

Path semantical quality `~~`, or qubit `~`, are often used as
"flags" for other theories to express uniqueness or solutions.

- `!~a` an expression that `a` is unique
- `~a` an expression that `a` has some solution

By default, you are not able to prove `~a` or `!~a`,
which means that you have a choice.
You can design your theory leaning toward `~a`,
or you can go in the opposite direction and lean toward `!~a`.
The direction a theory is designed is called "language bias".
Notice that they both are expressions of the underlying idea "there exists a solution".
The way they differ is in plurarity,
where `!~a` means "one" and `~a` means "many".

Theories that lean toward `!~a` are called "Seshatic"
from [Seshatism](https://github.com/advancedresearch/path_semantics/blob/master/papers-wip2/seshatism.pdf).
On the other hand, theories that lean toward `~a` are called "Platonic" from Platonism.
Seshatism is the dual of Platonism.
They differ from each other in the way existence is
biased by language, either toward "one-ness" (concrete)
or "many-ness" (abstract).

Intuitively, an abstract object can be copied,
so it has a kind of "many-nesss" built into it.
A concrete object can not be copied,
because this would destroy its uniqueness.

When mathematics works in absence of solutions
we call it "imaginary" that has some kind of "zero-ness" to it,
e.g. the imaginary unit `i` in complex numbers.
In logic, this is simply theorems that do not need
`!~a` or `~a`.

### Avatar Logic

Avatar Logic is an alternative to First Order Logic
which is more suitable for higher dimensional mathematics.
The reason is that it gets rid of predicates and replaces
them with binary relations, avatars and roles.
This design generalizes better over
multiple dimensions of evaluation.

You can experiment with Avatar Logic using [Avalog](https://github.com/advancedresearch/avalog), which is a Prolog-like language.

There are two basic axioms in Avatar Logic:

```text
ava_univ : !~b & (a, b) & (b : c)  ->  c(a) == b
ava_ava : (a, b(c)) & (b(c) : d)  ->  d(a) => b(c)
```

The first axiom `ava_univ` says that if `b` is unique, (expressed as `!~b`) and `b` has the role `c` (expressed as `b : c`) then it is sufficient to say `(a, b)` (an ordered pair) to determine `c(a) == b`.

In some sense, `ava_univ` says what it means to "compute" `b`. Another way to think of it is as a property `c` of `a`. When `b` is unique,
it is forced to behave that way for all other objects.
Every other object needs to use `b` the same way.
This is why `b` is thought of as a unique universal binary relation.

However, unique universal binary relations are too restrictive in many cases. This is why we need an "avatar". An avatar is a symbol which wraps another expression, e.g. `ava'(a)` where `ava'` is the avatar and `a` is the expression wrapped by `ava'`.

The second axiom `ava_ava` tells that when `b(c) : d`
it is sufficient to say `(a, b(c))` to determine
`d(a) => b(c)`. Notice that this is a weaker conclusion.

For example, if `(carl', parent'(alice'))` and
`parent'(alice') : mom'` then `mom'(carl') => parent'(alice')`. In this case, Carl is allowed to
have more than one mom.

If `!~parent'(alice')`, then Carl has only one mom Alice, because `mom'(carl') == parent'(alice')`.

Avatar Logic is well suited to handle complex relations
between objects that are often found in the natural world, such as family relations. In some sense,
Avatar Logic handles "exceptions to the rule" very well.

There is one more axiom that handles collision between avatars:

```text
ava_collide :
  (b, q1(a1)) & (b, q2(a2)) &
  (q1(a1) : p) & (q2(a2) : p) -> !sd(q1, q2)
```

For example, if you try to use `foo'` and `bar'` as avatars in the same place,
one can prove that they will collide:

```text
sym foo;
sym bar;
sym p;

fn test :
    (b, foo'(a1)) & (b, bar'(a2)) &
    (foo'(a1) : p') & (bar'(a2) : p')
-> false {
    arg : (b, foo'(a1)) & (b, bar'(a2)) &
          (foo'(a1) : p') & (bar'(a2) : p');
    use std::ava_collide;
    let x = ava_collide(arg) : !sd(foo', bar');
    let y = () : sd(foo', bar');
    let r = x(y) : false;
    return r;
}
```

If you want to treat a role `p` as unique for all its members,
then you can use the following theorem:

```text
ava_lower_univ : !~p & (b, a) & (a : p) -> p(b) == a
```
