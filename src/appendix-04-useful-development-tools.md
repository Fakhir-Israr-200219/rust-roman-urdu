## Appendix D: Useful Development Tools

Is appendix mein hum kuch useful development tools ke bare mein baat karte
hain jo Rust project provide karta hai. Hum automatic formatting, warnings ko
quickly fix karne ke tareeqon, ek linter, aur IDEs ke saath integration dekhenge.

### Automatic Formatting with `rustfmt`

`rustfmt` tool aapke code ko community code style ke mutabiq reformat karta hai.
Bohat se collaborative projects `rustfmt` use karte hain taake Rust likhte waqt
kis style ko use karna chahiye is par arguments se bacha ja sake: Har shakhs
tool ka use karke apne code ko format karta hai.

Rust installations mein `rustfmt` by default included hota hai, is liye aapke
system par pehle se `rustfmt` aur `cargo-fmt` programs hone chahiye. Yeh dono
commands `rustc` aur `cargo` ke analogous hain, is maayne mein ke `rustfmt`
zyada fine-grained control deta hai aur `cargo-fmt` us project ki conventions ko
samajhta hai jo Cargo use karta hai. Kisi bhi Cargo project ko format karne ke
liye yeh enter karein:

```console
$ cargo fmt
```

Is command ko run karne se current crate ka tamam Rust code reformat ho jata
hai. Is se sirf code style change honi chahiye, code ki semantics nahi. `rustfmt`
ke bare mein mazeed information ke liye [its documentation][rustfmt] dekhein.

### Fix Your Code with `rustfix`

`rustfix` tool Rust installations mein included hota hai aur un compiler
warnings ko automatically fix kar sakta hai jinhein correct karne ka clear
tareeqa mojood ho aur jo likely wohi ho jo aap chahte hain. Aapne shayad pehle
compiler warnings dekhi hongi. Misal ke taur par, is code ko dekhein:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let mut x = 42;
    println!("{x}");
}
```

Yahan hum variable `x` ko mutable define kar rahe hain, lekin hum asal mein
kabhi isay mutate nahi karte. Rust humein is ke bare mein warning deta hai:

```console
$ cargo build
   Compiling myprogram v0.1.0 (file:///projects/myprogram)
warning: variable does not need to be mutable
 --> src/main.rs:2:9
  |
2 |     let mut x = 0;
  |         ----^
  |         |
  |         help: remove this `mut`
  |
  = note: `#[warn(unused_mut)]` on by default
```

Warning suggest karti hai ke hum `mut` keyword remove kar dein. Hum `rustfix`
tool ke zariye `cargo fix` command run karke is suggestion ko automatically
apply kar sakte hain:

```console
$ cargo fix
    Checking myprogram v0.1.0 (file:///projects/myprogram)
      Fixing src/main.rs (1 fix)
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

Jab hum dobara *src/main.rs* dekhte hain, to humein nazar aayega ke `cargo fix`
ne code change kar diya hai:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let x = 42;
    println!("{x}");
}
```

Ab variable `x` immutable hai, aur warning dobara appear nahi hoti.

Aap `cargo fix` command ko apne code ko different Rust editions ke darmiyan
transition karne ke liye bhi use kar sakte hain. Editions ko [Appendix
E][editions]<!--
ignore --> mein cover kiya gaya hai.

### More Lints with Clippy

Clippy lints ka ek collection hai jo aapke code ko analyze karta hai taake aap
common mistakes ko catch kar saken aur apne Rust code ko improve kar saken.
Clippy standard Rust installations mein included hota hai.

Kisi bhi Cargo project par Clippy ke lints run karne ke liye yeh enter karein:

```console
$ cargo clippy
```

Misal ke taur par, maan lein ke aap ek aisa program likhte hain jo kisi
mathematical constant, jaise pi, ki approximation use karta hai, jaisa ke yeh
program karta hai:

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = 3.1415;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

Is project par `cargo clippy` run karne se yeh error result hota hai:

```text
error: approximate value of `f{32, 64}::consts::PI` found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: `#[deny(clippy::approx_constant)]` on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

Yeh error aapko batata hai ke Rust mein pehle se zyada precise `PI` constant
defined hai, aur agar aap is constant ko use karein to aapka program zyada
correct hoga. Phir aap apne code ko `PI` constant use karne ke liye change
karenge.

Neeche diya gaya code Clippy ki taraf se koi errors ya warnings result nahi
karta:

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = std::f64::consts::PI;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

Clippy ke bare mein mazeed information ke liye [its documentation][clippy]
dekhein.

### IDE Integration Using `rust-analyzer`

IDE integration mein madad ke liye Rust community [`rust-analyzer`][rust-analyzer]<!-- ignore --> use karne ki recommendation karti hai. Yeh tool
compiler-centric utilities ka ek set hai jo [Language Server Protocol][lsp]<!--
ignore --> mein communicate karta hai, jo IDEs aur programming languages ke
ek doosre ke saath communicate karne ki specification hai. Mukhtalif clients
`rust-analyzer` ko use kar sakte hain, jaise [the Rust analyzer plug-in for
Visual Studio Code][vscode].

Installation instructions ke liye `rust-analyzer` project ke [home page][rust-analyzer]<!-- ignore -->
par jayein, phir apne particular IDE mein language server support install
karein. Aapke IDE mein autocompletion, definition par jump karna, aur inline
errors jaisi capabilities aa jayengi.

[rustfmt]: https://github.com/rust-lang/rustfmt
[editions]: appendix-05-editions.md
[clippy]: https://github.com/rust-lang/rust-clippy
[rust-analyzer]: https://rust-analyzer.github.io
[lsp]: http://langserver.org/
[vscode]: https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer
