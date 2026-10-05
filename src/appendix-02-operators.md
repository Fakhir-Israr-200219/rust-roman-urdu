## Appendix B: Operators and Symbols

Is appendix mein Rust ki syntax ka glossary diya gaya hai, jisme operators aur
doosre symbols shamil hain jo apne aap ya paths, generics, trait bounds, macros,
attributes, comments, tuples, aur brackets ke context mein appear hotay hain.

### Operators

Table B-1 mein Rust ke operators, context mein operator kaise appear hota hai
us ki ek example, ek short explanation, aur yeh bataya gaya hai ke kya woh
operator overloadable hai. Agar koi operator overloadable ho, to us operator ko
overload karne ke liye relevant trait bhi list kiya gaya hai.

<span class="caption">Table B-1: Operators</span>

| Operator        | Example                                          | Explanation                                                          | Overloadable?  |
| --------------- | ------------------------------------------------ | -------------------------------------------------------------------- | -------------- |
| `!`             | `ident!(...)`, `ident!{...}`, `ident![...]`      | Macro expansion                                                      |                |
| `!`             | `!expr`                                          | Bitwise ya logical complement                                        | `Not`          |
| `!=`            | `expr != expr`                                   | Nonequality comparison                                               | `PartialEq`    |
| `%`             | `expr % expr`                                    | Arithmetic remainder                                                 | `Rem`          |
| `%=`            | `var %= expr`                                    | Arithmetic remainder aur assignment                                  | `RemAssign`    |
| `&`             | `&expr`, `&mut expr`                             | Borrow                                                               |                |
| `&`             | `&type`, `&mut type`, `&'a type`, `&'a mut type` | Borrowed pointer type                                                |                |
| `&`             | `expr & expr`                                    | Bitwise AND                                                          | `BitAnd`       |
| `&=`            | `var &= expr`                                    | Bitwise AND aur assignment                                           | `BitAndAssign` |
| `&&`            | `expr && expr`                                   | Short-circuiting logical AND                                         |                |
| `*`             | `expr * expr`                                    | Arithmetic multiplication                                            | `Mul`          |
| `*=`            | `var *= expr`                                    | Arithmetic multiplication aur assignment                             | `MulAssign`    |
| `*`             | `*expr`                                          | Dereference                                                          | `Deref`        |
| `*`             | `*const type`, `*mut type`                       | Raw pointer                                                          |                |
| `+`             | `trait + trait`, `'a + trait`                    | Compound type constraint                                             |                |
| `+`             | `expr + expr`                                    | Arithmetic addition                                                  | `Add`          |
| `+=`            | `var += expr`                                    | Arithmetic addition aur assignment                                   | `AddAssign`    |
| `,`             | `expr, expr`                                     | Argument aur element separator                                       |                |
| `-`             | `- expr`                                         | Arithmetic negation                                                  | `Neg`          |
| `-`             | `expr - expr`                                    | Arithmetic subtraction                                               | `Sub`          |
| `-=`            | `var -= expr`                                    | Arithmetic subtraction aur assignment                                | `SubAssign`    |
| `->`            | `fn(...) -> type`, <code>|...| -> type</code>    | Function aur closure return type                                     |                |
| `.`             | `expr.ident`                                     | Field access                                                         |                |
| `.`             | `expr.ident(expr, ...)`                          | Method call                                                          |                |
| `.`             | `expr.0`, `expr.1`, and so on                    | Tuple indexing                                                       |                |
| `..`            | `..`, `expr..`, `..expr`, `expr..expr`           | Right-exclusive range literal                                        | `PartialOrd`   |
| `..=`           | `..=expr`, `expr..=expr`                         | Right-inclusive range literal                                        | `PartialOrd`   |
| `..`            | `..expr`                                         | Struct literal update syntax                                         |                |
| `..`            | `variant(x, ..)`, `struct_type { x, .. }`        | “And the rest” pattern binding                                       |                |
| `...`           | `expr...expr`                                    | (Deprecated, `..=` use karein) In a pattern: inclusive range pattern |                |
| `/`             | `expr / expr`                                    | Arithmetic division                                                  | `Div`          |
| `/=`            | `var /= expr`                                    | Arithmetic division aur assignment                                   | `DivAssign`    |
| `:`             | `pat: type`, `ident: type`                       | Constraints                                                          |                |
| `:`             | `ident: expr`                                    | Struct field initializer                                             |                |
| `:`             | `'a: loop {...}`                                 | Loop label                                                           |                |
| `;`             | `expr;`                                          | Statement aur item terminator                                        |                |
| `;`             | `[...; len]`                                     | Fixed-size array syntax ka hissa                                     |                |
| `<<`            | `expr << expr`                                   | Left-shift                                                           | `Shl`          |
| `<<=`           | `var <<= expr`                                   | Left-shift aur assignment                                            | `ShlAssign`    |
| `<`             | `expr < expr`                                    | Less than comparison                                                 | `PartialOrd`   |
| `<=`            | `expr <= expr`                                   | Less than ya equal to comparison                                     | `PartialOrd`   |
| `=`             | `var = expr`, `ident = type`                     | Assignment/equivalence                                               |                |
| `==`            | `expr == expr`                                   | Equality comparison                                                  | `PartialEq`    |
| `=>`            | `pat => expr`                                    | Match arm syntax ka hissa                                            |                |
| `>`             | `expr > expr`                                    | Greater than comparison                                              | `PartialOrd`   |
| `>=`            | `expr >= expr`                                   | Greater than ya equal to comparison                                  | `PartialOrd`   |
| `>>`            | `expr >> expr`                                   | Right-shift                                                          | `Shr`          |
| `>>=`           | `var >>= expr`                                   | Right-shift aur assignment                                           | `ShrAssign`    |
| `@`             | `ident @ pat`                                    | Pattern binding                                                      |                |
| `^`             | `expr ^ expr`                                    | Bitwise exclusive OR                                                 | `BitXor`       |
| `^=`            | `var ^= expr`                                    | Bitwise exclusive OR aur assignment                                  | `BitXorAssign` |
| <code>|</code>  | <code>pat | pat</code>                           | Pattern alternatives                                                 |                |
| <code>|</code>  | <code>expr | expr</code>                         | Bitwise OR                                                           | `BitOr`        |
| <code>|=</code> | <code>var |= expr</code>                         | Bitwise OR aur assignment                                            | `BitOrAssign`  |
| <code>||</code> | <code>expr || expr</code>                        | Short-circuiting logical OR                                          |                |
| `?`             | `expr?`                                          | Error propagation                                                    |                |

### Non-operator Symbols

Neeche di gayi tables mein woh tamam symbols shamil hain jo operators ke taur
par function nahi karte; yani, woh function ya method call ki tarah behave nahi
karte.

Table B-2 un symbols ko dikhati hai jo apne aap appear hotay hain aur mukhtalif
jagahon par valid hain.

<span class="caption">Table B-2: Stand-alone Syntax</span>

| Symbol                                                                | Explanation                                                                             |
| --------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| `'ident`                                                              | Named lifetime ya loop label                                                            |
| Digits immediately followed by `u8`, `i32`, `f64`, `usize`, and so on | Specific type ka numeric literal                                                        |
| `"..."`                                                               | String literal                                                                          |
| `r"..."`, `r#"..."#`, `r##"..."##`, and so on                         | Raw string literal; escape characters process nahi kiye jatay                           |
| `b"..."`                                                              | Byte string literal; string ke bajaye bytes ka array construct karta hai                |
| `br"..."`, `br#"..."#`, `br##"..."##`, and so on                      | Raw byte string literal; raw aur byte string literal ka combination                     |
| `'...'`                                                               | Character literal                                                                       |
| `b'...'`                                                              | ASCII byte literal                                                                      |
| <code>|...| expr</code>                                               | Closure                                                                                 |
| `!`                                                                   | Diverging functions ke liye hamesha-empty bottom type                                   |
| `_`                                                                   | “Ignored” pattern binding; integer literals ko readable banane ke liye bhi use hota hai |

Table B-3 un symbols ko dikhati hai jo module hierarchy ke through kisi item
tak path ke context mein appear hotay hain.

<span class="caption">Table B-3: Path-Related Syntax</span>

| Symbol                                  | Explanation                                                                                                                    |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `ident::ident`                          | Namespace path                                                                                                                 |
| `::path`                                | Crate root ke relative path (yani, explicitly absolute path)                                                                   |
| `self::path`                            | Current module ke relative path (yani, explicitly relative path)                                                               |
| `super::path`                           | Current module ke parent ke relative path                                                                                      |
| `type::ident`, `<type as trait>::ident` | Associated constants, functions, aur types                                                                                     |
| `<type>::...`                           | Aisi type ka associated item jise directly name nahi kiya ja sakta (misal ke taur par, `<&T>::...`, `<[T]>::...`, aur waghera) |
| `trait::method(...)`                    | Us trait ka name de kar method call ko disambiguate karna jo usay define karta hai                                             |
| `type::method(...)`                     | Us type ka name de kar method call ko disambiguate karna jis ke liye woh define hai                                            |
| `<type as trait>::method(...)`          | Trait aur type dono ka name de kar method call ko disambiguate karna                                                           |

Table B-4 un symbols ko dikhati hai jo generic type parameters ko use karne ke
context mein appear hotay hain.

<span class="caption">Table B-4: Generics</span>

| Symbol                         | Explanation                                                                                                                                                       |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `path<...>`                    | Type mein generic type ke parameters specify karta hai (misal ke taur par, `Vec<u8>`)                                                                             |
| `path::<...>`, `method::<...>` | Expression mein generic type, function, ya method ke parameters specify karta hai; ise aksar *turbofish* kaha jata hai (misal ke taur par, `"42".parse::<i32>()`) |
| `fn ident<...> ...`            | Generic function define karta hai                                                                                                                                 |
| `struct ident<...> ...`        | Generic structure define karta hai                                                                                                                                |
| `enum ident<...> ...`          | Generic enumeration define karta hai                                                                                                                              |
| `impl<...> ...`                | Generic implementation define karta hai                                                                                                                           |
| `for<...> type`                | Higher ranked lifetime bounds                                                                                                                                     |
| `type<ident=type>`             | Aisi generic type jismein ek ya zyada associated types ki specific assignments hoti hain (misal ke taur par, `Iterator<Item=T>`)                                  |

Table B-5 un symbols ko dikhati hai jo generic type parameters ko trait bounds
ke saath constrain karne ke context mein appear hotay hain.

<span class="caption">Table B-5: Trait Bound Constraints</span>

| Symbol                        | Explanation                                                                                                                                                      |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `T: U`                        | Generic parameter `T` un types tak constrained hai jo `U` implement karti hain                                                                                   |
| `T: 'a`                       | Generic type `T` ko lifetime `'a` se outlive karna zaroori hai (yani, type mein transitively aise references nahi ho saktay jin ki lifetimes `'a` se chhoti hon) |
| `T: 'static`                  | Generic type `T` mein `'static` references ke ilawa koi borrowed references nahi hotay                                                                           |
| `'b: 'a`                      | Generic lifetime `'b` ko lifetime `'a` se outlive karna zaroori hai                                                                                              |
| `T: ?Sized`                   | Generic type parameter ko dynamically sized type hone ki ijazat deta hai                                                                                         |
| `'a + trait`, `trait + trait` | Compound type constraint                                                                                                                                         |

Table B-6 un symbols ko dikhati hai jo macros ko call ya define karne aur kisi
item par attributes specify karne ke context mein appear hotay hain.

<span class="caption">Table B-6: Macros and Attributes</span>

| Symbol                                      | Explanation        |
| ------------------------------------------- | ------------------ |
| `#[meta]`                                   | Outer attribute    |
| `#![meta]`                                  | Inner attribute    |
| `$ident`                                    | Macro substitution |
| `$ident:kind`                               | Macro metavariable |
| `$(...)...`                                 | Macro repetition   |
| `ident!(...)`, `ident!{...}`, `ident![...]` | Macro invocation   |

Table B-7 un symbols ko dikhati hai jo comments create karte hain.

<span class="caption">Table B-7: Comments</span>

| Symbol     | Explanation             |
| ---------- | ----------------------- |
| `//`       | Line comment            |
| `//!`      | Inner line doc comment  |
| `///`      | Outer line doc comment  |
| `/*...*/`  | Block comment           |
| `/*!...*/` | Inner block doc comment |
| `/**...*/` | Outer block doc comment |

Table B-8 un contexts ko dikhati hai jin mein parentheses use kiye jatay hain.

<span class="caption">Table B-8: Parentheses</span>

| Symbol            | Explanation                                                                                                      |
| ----------------- | ---------------------------------------------------------------------------------------------------------------- |
| `()`              | Empty tuple (aka unit), literal aur type dono                                                                    |
| `(expr)`          | Parenthesized expression                                                                                         |
| `(expr,)`         | Single-element tuple expression                                                                                  |
| `(type,)`         | Single-element tuple type                                                                                        |
| `(expr, ...)`     | Tuple expression                                                                                                 |
| `(type, ...)`     | Tuple type                                                                                                       |
| `expr(expr, ...)` | Function call expression; tuple `struct`s aur tuple `enum` variants ko initialize karne ke liye bhi use hota hai |

Table B-9 un contexts ko dikhati hai jin mein curly brackets use kiye jatay hain.

<span class="caption">Table B-9: Curly Brackets</span>

| Context      | Explanation      |
| ------------ | ---------------- |
| `{...}`      | Block expression |
| `Type {...}` | Struct literal   |

Table B-10 un contexts ko dikhati hai jin mein square brackets use kiye jatay hain.

<span class="caption">Table B-10: Square Brackets</span>

| Context                                            | Explanation                                                                                                                                                        |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `[...]`                                            | Array literal                                                                                                                                                      |
| `[expr; len]`                                      | Array literal jismein `expr` ki `len` copies hoti hain                                                                                                             |
| `[type; len]`                                      | Array type jismein `type` ke `len` instances hotay hain                                                                                                            |
| `expr[expr]`                                       | Collection indexing; overloadable (`Index`, `IndexMut`)                                                                                                            |
| `expr[..]`, `expr[a..]`, `expr[..b]`, `expr[a..b]` | Collection indexing jo collection slicing ka impression deta hai, jismein `Range`, `RangeFrom`, `RangeTo`, ya `RangeFull` ko “index” ke taur par use kiya jata hai |
