## Appendix A: Keywords

Neeche di gayi lists mein woh keywords shamil hain jo Rust language ke current
ya future use ke liye reserved hain. Isi wajah se inhein identifiers ke taur
par use nahi kiya ja sakta (siwaye raw identifiers ke, jaisa ke hum
[“Raw
Identifiers”][raw-identifiers]<!-- ignore --> section mein discuss karte hain).
*Identifiers* functions, variables, parameters, struct fields, modules, crates,
constants, macros, static values, attributes, types, traits, ya lifetimes ke
names hotay hain.

[raw-identifiers]: #raw-identifiers

### Keywords Currently in Use

Neeche un keywords ki list hai jo filhal use mein hain, aur un ki
functionality describe ki gayi hai.

* **`as`**: Primitive casting perform karta hai, kisi item ko contain karne wale
  specific trait ko disambiguate karta hai, ya `use` statements mein items ke
  names change karta hai.
* **`async`**: Current thread ko block karne ke bajaye ek `Future` return karta hai.
* **`await`**: Execution ko tab tak suspend karta hai jab tak `Future` ka result
  ready na ho jaye.
* **`break`**: Loop se foran bahar nikalta hai.
* **`const`**: Constant items ya constant raw pointers define karta hai.
* **`continue`**: Loop ki next iteration par continue karta hai.
* **`crate`**: Module path mein, crate root ko refer karta hai.
* **`dyn`**: Trait object ke liye dynamic dispatch.
* **`else`**: `if` aur `if let` control flow constructs ke liye fallback.
* **`enum`**: Enumeration define karta hai.
* **`extern`**: Kisi external function ya variable ke saath link karta hai.
* **`false`**: Boolean false literal.
* **`fn`**: Function ya function pointer type define karta hai.
* **`for`**: Iterator se items par loop karta hai, trait implement karta hai, ya
  higher ranked lifetime specify karta hai.
* **`if`**: Conditional expression ke result ki bunyaad par branch karta hai.
* **`impl`**: Inherent ya trait functionality implement karta hai.
* **`in`**: `for` loop syntax ka hissa.
* **`let`**: Variable ko bind karta hai.
* **`loop`**: Baghair kisi condition ke loop karta hai.
* **`match`**: Kisi value ko patterns ke saath match karta hai.
* **`mod`**: Module define karta hai.
* **`move`**: Closure ko uski tamam captures ki ownership lene par majboor karta hai.
* **`mut`**: References, raw pointers, ya pattern bindings mein mutability ko
  denote karta hai.
* **`pub`**: Struct fields, `impl` blocks, ya modules mein public visibility
  ko denote karta hai.
* **`ref`**: Reference ke zariye bind karta hai.
* **`return`**: Function se return karta hai.
* **`Self`**: Us type ka type alias hai jise hum define ya implement kar rahe hain.
* **`self`**: Method subject ya current module.
* **`static`**: Global variable ya aisi lifetime jo poori program execution tak
  rehti hai.
* **`struct`**: Structure define karta hai.
* **`super`**: Current module ka parent module.
* **`trait`**: Trait define karta hai.
* **`true`**: Boolean true literal.
* **`type`**: Type alias ya associated type define karta hai.
* **`union`**: Ek [union][union]<!-- ignore --> define karta hai; yeh sirf tab
  keyword hota hai jab union declaration mein use kiya jaye.
* **`unsafe`**: Unsafe code, functions, traits, ya implementations ko denote
  karta hai.
* **`use`**: Symbols ko scope mein lata hai.
* **`where`**: Aisi clauses ko denote karta hai jo kisi type ko constrain karti hain.
* **`while`**: Kisi expression ke result ki bunyaad par conditionally loop karta hai.

[union]: ../reference/items/unions.html

### Keywords Reserved for Future Use

Neeche diye gaye keywords ki abhi koi functionality nahi hai, lekin Rust ne
inhein possible future use ke liye reserve kiya hua hai:

* `abstract`
* `become`
* `box`
* `do`
* `final`
* `gen`
* `macro`
* `override`
* `priv`
* `try`
* `typeof`
* `unsized`
* `virtual`
* `yield`

### Raw Identifiers

*Raw identifiers* aisi syntax hain jo aapko keywords ko un jagahon par use
karne deti hain jahan aam tor par unhein allow nahi kiya jata. Aap raw
identifier ko kisi keyword ke aage `r#` laga kar use karte hain.

Misal ke taur par, `match` ek keyword hai. Agar aap neeche di gayi function ko
compile karne ki koshish karein jo `match` ko apne name ke taur par use karti hai:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
fn match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}
```

to aapko yeh error milega:

```text
error: expected identifier, found keyword `match`
 --> src/main.rs:4:4
  |
4 | fn match(needle: &str, haystack: &str) -> bool {
  |    ^^^^^ expected identifier, found keyword
```

Error dikhata hai ke aap keyword `match` ko function identifier ke taur par use
nahi kar sakte. `match` ko function name ke taur par use karne ke liye aapko
raw identifier syntax use karni hogi, is tarah:

<span class="filename">Filename: src/main.rs</span>

```rust
fn r#match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}

fn main() {
    assert!(r#match("foo", "foobar"));
}
```

Yeh code baghair kisi error ke compile ho jayega. Function ki definition mein
aur `main` mein function ko call karte waqt function name ke aage `r#` prefix
par ghaur karein.

Raw identifiers aapko kisi bhi word ko identifier ke taur par choose karne dete
hain, chahe woh word reserved keyword hi kyun na ho. Is se humein identifier
names choose karne mein zyada freedom milti hai, aur saath hi humein un
programs ke saath integrate karne ki sahulat milti hai jo kisi aisi language
mein likhe gaye hon jahan yeh words keywords nahi hain. Is ke ilawa, raw
identifiers aapko kisi different Rust edition mein likhi gayi libraries ko
apne crate se different edition ke saath use karne dete hain. Misal ke taur
par, `try` 2015 edition mein keyword nahi hai lekin 2018, 2021, aur 2024
editions mein hai. Agar aap kisi aisi library par depend karte hain jo 2015
edition use karke likhi gayi hai aur us mein `try` function hai, to baad wali
editions mein apne code se us function ko call karne ke liye aapko raw
identifier syntax, is case mein `r#try`, use karni hogi. Editions ke bare mein
mazeed maloomat ke liye [Appendix E][appendix-e]<!-- ignore --> dekhein.

[appendix-e]: appendix-05-editions.html
