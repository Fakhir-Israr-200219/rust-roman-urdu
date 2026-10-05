## Cargo Workspaces

Chapter 12 mein hum ne ek aisa package banaya tha jisme ek binary crate aur ek
library crate shamil tha. Jaise jaise aapka project develop hota hai, aap dekh
sakte hain ke library crate lagataar bara hota ja raha hai aur aap apne package
ko mazeed multiple library crates mein split karna chahte hain. Cargo ek
feature provide karta hai jise *workspaces* kehte hain, jo multiple related
packages ko manage karne mein madad kar sakta hai jo ek saath develop kiye ja
rahe hon.

### Creating a Workspace

Ek *workspace* packages ka ek set hota hai jo ek hi *Cargo.lock* aur output
directory share karta hai. Aaiye workspace ko use karte hue ek project banate
hain—hum trivial code use karenge taa-ke hum workspace ki structure par focus
kar saken. Workspace ko structure karne ke multiple tareeqe hain, is liye hum
sirf ek common tareeqa dikhayenge. Hamare paas ek workspace hoga jisme ek
binary aur do libraries hongi. Binary, jo main functionality provide karegi,
un dono libraries par depend karegi. Ek library `add_one` function provide
karegi aur doosri library `add_two` function provide karegi. Yeh teenon crates
ek hi workspace ka hissa hongi. Hum workspace ke liye ek nayi directory bana
kar shuru karenge:

```console
$ mkdir add
$ cd add
```

Ab *add* directory mein hum *Cargo.toml* file create karte hain jo poore
workspace ko configure karegi. Is file mein `[package]` section nahi hoga.
Is ke bajaye, yeh `[workspace]` section se start hogi jo humein workspace mein
members add karne degi. Hum apne workspace mein Cargo ke resolver algorithm
ka latest aur greatest version use karne ke liye `resolver` value ko `"3"` par
set karte hain:

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-01-workspace/add/Cargo.toml}}
```

Ab hum *add* directory ke andar `cargo new` run karke `adder` binary crate
create karenge:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-01-adder-crate/add
remove `members = ["adder"]` from Cargo.toml
rm -rf adder
cargo new adder
copy output below
-->

```console
$ cargo new adder
     Created binary (application) `adder` package
      Adding `adder` as member of workspace at `file:///projects/add`
```

Workspace ke andar `cargo new` run karne se newly created package
automatically workspace ke *Cargo.toml* mein `[workspace]` definition ke
`members` key mein bhi add ho jata hai, bilkul is tarah:

```toml
{{#include ../listings/ch14-more-about-cargo/output-only-01-adder-crate/add/Cargo.toml}}
```

Is point par, hum `cargo build` run karke workspace ko build kar sakte hain.
Aapki *add* directory ki files is tarah nazar aani chahiye:

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

Workspace mein top level par ek *target* directory hoti hai jisme compiled
artifacts rakhe jayenge; `adder` package ki apni *target* directory nahi hoti.
Agar hum *adder* directory ke andar se `cargo build` run karein, tab bhi
compiled artifacts *add/target* mein hi jayenge, *add/adder/target* mein nahi.
Cargo workspace mein *target* directory ko is tarah structure karta hai kyun ke
workspace ke crates ka maqsad ek doosre par depend karna hota hai. Agar har
crate ki apni *target* directory hoti, to har crate ko workspace ke doosre
har crate ko dobara compile karna parta taa-ke artifacts uski apni *target*
directory mein rakhe ja saken. Ek hi *target* directory share karne se crates
ghair-zaroori rebuilding se bach sakte hain.

### Creating the Second Package in the Workspace

Ab, aaiye workspace mein ek aur member package create karte hain aur iska naam
`add_one` rakhte hain. `add_one` naam ka ek naya library crate generate karein:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-02-add-one/add
remove `"add_one"` from `members` list in Cargo.toml
rm -rf add_one
cargo new add_one --lib
copy output below
-->

```console
$ cargo new add_one --lib
     Created library `add_one` package
      Adding `add_one` as member of workspace at `file:///projects/add`
```

Ab top-level *Cargo.toml* mein `members` list ke andar *add_one* ka path bhi
include hoga:

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/Cargo.toml}}
```

Aapki *add* directory mein ab yeh directories aur files honi chahiye:

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

*add_one/src/lib.rs* file mein, aaiye ek `add_one` function add karte hain:

<span class="filename">Filename: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/add_one/src/lib.rs}}
```

Ab hum `adder` package mein apne binary ko `add_one` package par depend kara
sakte hain, jisme hamari library hai. Sab se pehle, humein *adder/Cargo.toml*
mein `add_one` par ek path dependency add karni hogi.

<span class="filename">Filename: adder/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/adder/Cargo.toml:6:7}}
```

Cargo yeh assume nahi karta ke workspace mein maujood crates ek doosre par
depend karenge, is liye humein dependency relationships ko explicitly specify
karna zaroori hai.

Ab, aaiye `adder` crate mein `add_one` crate se `add_one` function use karte
hain. *adder/src/main.rs* file open karein aur `main` function ko change karein
taa-ke woh `add_one` function ko call kare, jaisa ke Listing 14-7 mein hai.

<Listing number="14-7" file-name="adder/src/main.rs" caption="Using the `add_one` library crate from the `adder` crate">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-07/add/adder/src/main.rs}}
```

</Listing>

Ab top-level *add* directory mein `cargo build` run karke workspace ko build
karte hain!

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.22s
```

*add* directory se binary crate ko run karne ke liye, hum `-p` argument aur
package name ko `cargo run` ke saath use karke specify kar sakte hain ke
workspace mein hum kis package ko run karna chahte hain:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo run -p adder
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo run -p adder
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

Yeh *adder/src/main.rs* mein maujood code ko run karta hai, jo `add_one` crate
par depend karta hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="depending-on-an-external-package-in-a-workspace"></a>

### Depending on an External Package

Notice karein ke workspace mein top level par sirf ek *Cargo.lock* file hai,
bajaye is ke ke har crate ki directory mein ek *Cargo.lock* ho. Yeh ensure karta
hai ke tamam crates apni tamam dependencies ka same version use kar rahe hain.
Agar hum *adder/Cargo.toml* aur *add_one/Cargo.toml* files mein `rand` package
add karein, to Cargo dono ko `rand` ke ek hi version par resolve karega aur us
information ko us ek *Cargo.lock* mein record karega. Workspace ke tamam crates
ko same dependencies use karwana yeh ensure karta hai ke crates hamesha ek
doosre ke saath compatible rahen. Aaiye *add_one/Cargo.toml* file ke
`[dependencies]` section mein `rand` crate add karte hain taa-ke hum
`add_one` crate mein `rand` crate ko use kar saken:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:

* ch01-01-installation.md
* ch02-00-guessing-game-tutorial.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
-->

<span class="filename">Filename: add_one/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add/add_one/Cargo.toml:6:7}}
```

Ab hum *add_one/src/lib.rs* file mein `use rand;` add kar sakte hain, aur
*add* directory mein `cargo build` run karke poore workspace ko build karne se
`rand` crate aa jayega aur compile ho jayega. Humein ek warning milegi kyun ke
hum us `rand` ko refer nahi kar rahe jo humne scope mein laya hai:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.10.1
   --snip--
   Compiling rand v0.10.1
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` (part of `#[warn(unused)]`) on by default

warning: `add_one` (lib) generated 1 warning (run `cargo fix --lib -p add_one` to apply 1 suggestion)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.95s
```

Ab top-level *Cargo.lock* mein `add_one` ki `rand` par dependency ke baare mein
information maujood hai. Lekin, halanke workspace mein kahin `rand` use ho raha
hai, hum workspace ke doosre crates mein ise use nahi kar sakte jab tak hum
unke *Cargo.toml* files mein bhi `rand` add na karein. Misal ke taur par, agar
hum `adder` package ke *adder/src/main.rs* file mein `use rand;` add karein, to
humein ek error milega:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-03-use-rand/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

Isay fix karne ke liye, `adder` package ki *Cargo.toml* file edit karein aur
indicate karein ke `rand` us ke liye bhi ek dependency hai. `adder` package ko
build karne se *Cargo.lock* mein `adder` ki dependencies ki list mein `rand`
add ho jayega, lekin `rand` ki additional copies download nahi hongi. Cargo
ensure karega ke workspace ke har package mein har woh crate jo `rand` package
use kar raha hai, `rand` ka same version use kare, jab tak woh `rand` ke
compatible versions specify karte hain. Is se space bachegi aur yeh ensure
hoga ke workspace ke crates ek doosre ke saath compatible rahen.

Agar workspace ke crates ek hi dependency ke incompatible versions specify
karte hain, to Cargo un mein se har version ko resolve karega, lekin phir bhi
jitne kam possible versions hon, unhein resolve karne ki koshish karega.

### Adding a Test to a Workspace

Ek aur enhancement ke liye, aaiye `add_one` crate ke andar
`add_one::add_one` function ka ek test add karte hain:

<span class="filename">Filename: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add/add_one/src/lib.rs}}
```

Ab top-level *add* directory mein `cargo test` run karein. Is tarah structured
workspace mein `cargo test` run karne se workspace ke tamam crates ke tests
run honge:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test
copy output below; the output updating script doesn't handle subdirectories in
paths properly
-->

```console
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.20s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-3a47283c568d2b6a)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Output ka pehla section dikhata hai ke `add_one` crate mein `it_works` test
pass ho gaya. Agla section dikhata hai ke `adder` crate mein zero tests mile,
aur phir aakhri section dikhata hai ke `add_one` crate mein zero documentation
tests mile.

Hum top-level directory se workspace ke kisi ek particular crate ke tests bhi
run kar sakte hain. Is ke liye `-p` flag use karein aur us crate ka naam
specify karein jise hum test karna chahte hain:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test -p add_one
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo test -p add_one
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Yeh output dikhata hai ke `cargo test` ne sirf `add_one` crate ke tests run kiye
aur `adder` crate ke tests run nahi kiye.

Agar aap workspace ke crates ko [crates.io](https://crates.io/)<!-- ignore -->
par publish karte hain, to workspace ke har crate ko separately publish karna
hoga. `cargo test` ki tarah, hum apne workspace ke kisi particular crate ko
`-p` flag use karke aur us crate ka naam specify karke publish kar sakte hain
jise hum publish karna chahte hain.

Mazeed practice ke liye, `add_one` crate ki tarah is workspace mein ek
`add_two` crate add karein!

Jaise jaise aapka project bara hota jaye, workspace use karne par consider
karein: Is se aap ek bari code ki blob ke bajaye chhote aur aasani se samajhne
wale components ke saath kaam kar sakte hain. Is ke ilawa, workspace mein
crates ko rakhna crates ke darmiyan coordination ko aasaan bana sakta hai agar
unhein aksar ek hi waqt mein change kiya jata ho.
