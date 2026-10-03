## `use` Keyword Ke Saath Paths Ko Scope Mein Lana

Functions ko call karne ke liye paths ko poora likhna inconvenient aur repetitive mehsoos ho sakta hai. Listing 7-7 mein, chahe hum ne `add_to_waitlist` function ke liye absolute path choose kiya ho ya relative path, jab bhi hum `add_to_waitlist` ko call karna chahte the, humein `front_of_house` aur `hosting` ko bhi specify karna padta tha. Khushqismati se, is process ko simplify karne ka ek tareeqa hai: Hum `use` keyword ke saath ek baar kisi path ka shortcut create kar sakte hain aur phir scope mein baqi har jagah shorter name use kar sakte hain.

Listing 7-11 mein, hum `crate::front_of_house::hosting` module ko `eat_at_restaurant` function ke scope mein late hain taa-ke `eat_at_restaurant` mein `add_to_waitlist` function ko call karne ke liye humein sirf `hosting::add_to_waitlist` specify karna pade.

<Listing number="7-11" file-name="src/lib.rs" caption="`use` ke saath module ko scope mein lana">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-11/src/lib.rs}}
```

</Listing>

Kisi scope mein `use` aur path add karna filesystem mein symbolic link create karne jaisa hai. Crate root mein `use crate::front_of_house::hosting` add karne se, `hosting` ab us scope mein ek valid name hai, bilkul aise hi jaise `hosting` module crate root mein define kiya gaya ho. `use` ke saath scope mein laye gaye paths bhi, kisi bhi doosre path ki tarah, privacy ko check karte hain.

Ye note karein ke `use` sirf us particular scope ke liye shortcut create karta hai jahan `use` likha gaya ho. Listing 7-12 mein `eat_at_restaurant` function ko `customer` naam ke ek naye child module mein move kiya gaya hai, jo `use` statement se different scope hai, is liye function body compile nahi hogi.

<Listing number="7-12" file-name="src/lib.rs" caption="`use` statement sirf usi scope mein apply hota hai jahan ye mojood ho.">

```rust,noplayground,test_harness,does_not_compile,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-12/src/lib.rs}}
```

</Listing>

Compiler error dikhata hai ke shortcut ab `customer` module ke andar apply nahi hota:

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-12/output.txt}}
```

Notice karein ke ek warning bhi hai ke `use` ab apne scope mein use nahi ho raha! Is problem ko fix karne ke liye, `use` ko `customer` module ke andar bhi move karein, ya child `customer` module ke andar `super::hosting` ke zariye parent module mein maujood shortcut ko reference karein.


### Idiomatic `use` Paths Banana

Listing 7-11 mein aap ne shayad socha ho ke hum ne `use crate::front_of_house::hosting` specify karke phir `eat_at_restaurant` mein `hosting::add_to_waitlist` call kyun kiya, bajaye is ke ke same result hasil karne ke liye `use` path ko poora `add_to_waitlist` function tak specify karte, jaisa ke Listing 7-13 mein hai.

<Listing number="7-13" file-name="src/lib.rs" caption="`use` ke saath `add_to_waitlist` function ko scope mein lana, jo idiomatic nahi hai">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-13/src/lib.rs}}
```

</Listing>

Agarche Listing 7-11 aur Listing 7-13 dono same task perform karte hain, Listing 7-11 `use` ke saath kisi function ko scope mein lane ka idiomatic tareeqa hai. `use` ke saath function ke parent module ko scope mein lane ka matlab hai ke function ko call karte waqt humein parent module specify karna hota hai. Function ko call karte waqt parent module specify karna ye wazeh karta hai ke function locally defined nahi hai, aur saath hi full path ki repetition ko minimum rakhta hai. Listing 7-13 ka code ye wazeh nahi karta ke `add_to_waitlist` kahan defined hai.

Doosri taraf, jab hum `use` ke saath structs, enums aur doosre items ko scope mein late hain, to full path specify karna idiomatic hai. Listing 7-14 standard library ke `HashMap` struct ko binary crate ke scope mein lane ka idiomatic tareeqa dikhati hai.

<Listing number="7-14" file-name="src/main.rs" caption="Idiomatic tareeqe se `HashMap` ko scope mein lana">

```rust
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-14/src/main.rs}}
```

</Listing>

Is idiom ke peeche koi strong reason nahi hai: Ye bas woh convention hai jo waqt ke saath develop hui hai, aur log is tarah Rust code ko read aur write karne ke aadhi ho gaye hain.

Is idiom ka exception us waqt hota hai jab hum `use` statements ke zariye same name wale do items ko scope mein la rahe hon, kyun ke Rust iski ijazat nahi deta. Listing 7-15 dikhati hai ke same name lekin different parent modules wale do `Result` types ko scope mein kis tarah laya jata hai aur unhein kis tarah refer kiya jata hai.

<Listing number="7-15" file-name="src/lib.rs" caption="Same name wale do types ko ek hi scope mein lane ke liye unke parent modules ko use karna zaroori hai.">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-15/src/lib.rs:here}}
```

</Listing>

Jaisa ke aap dekh sakte hain, parent modules ko use karna dono `Result` types ko distinguish karta hai. Agar is ke bajaye hum `use std::fmt::Result` aur `use std::io::Result` specify karte, to hamare paas same scope mein do `Result` types hote, aur jab hum `Result` use karte to Rust ko pata na hota ke hamari murad kis wale se hai.

### `as` Keyword Ke Saath Naye Names Dena

`use` ke saath same name wale do types ko ek hi scope mein lane ki problem ka ek aur solution hai: Path ke baad hum `as` aur type ke liye ek naya local name, ya *alias*, specify kar sakte hain. Listing 7-16 dikhati hai ke `as` use karke dono `Result` types mein se ek ka name change karne ke zariye Listing 7-15 ke code ko ek aur tareeqe se kaise likha ja sakta hai.

<Listing number="7-16" file-name="src/lib.rs" caption="`as` keyword ke saath kisi type ko scope mein late waqt uska name change karna">

```rust,noplayground id="w9u2ne"
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-16/src/lib.rs:here}}
```

</Listing>

Doosre `use` statement mein, hum ne `std::io::Result` type ke liye naya name `IoResult` choose kiya, jo `std::fmt` ke `Result` ke saath conflict nahi karega, kyun ke hum ne usay bhi scope mein laya hai. Listing 7-15 aur Listing 7-16 dono ko idiomatic mana jata hai, is liye choice aapki hai!

### `pub use` Ke Saath Names Ko Re-export Karna

Jab hum `use` keyword ke saath kisi name ko scope mein late hain, to woh name us scope ke liye private hota hai jahan hum ne usay import kiya hai. Kisi doosre scope ke code ko is qabil banane ke liye ke woh us name ko aise refer kar sake jaise woh usi scope mein define kiya gaya ho, hum `pub` aur `use` ko combine kar sakte hain. Is technique ko *re-exporting* kaha jata hai, kyun ke hum kisi item ko scope mein late hain aur saath hi us item ko doosron ke liye bhi available bana dete hain taa-ke woh usay apne scope mein la saken.

Listing 7-17 mein Listing 7-11 ka code dikhaya gaya hai, jisme root module ke `use` ko `pub use` mein change kiya gaya hai.

<Listing number="7-17" file-name="src/lib.rs" caption="`pub use` ke saath kisi naye scope se kisi bhi code ke use karne ke liye name ko available banana">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-17/src/lib.rs}}
```

</Listing>

Is change se pehle, external code ko `add_to_waitlist` function ko call karne ke liye path `restaurant::front_of_house::hosting::add_to_waitlist()` use karna padta, aur is ke liye `front_of_house` module ko bhi `pub` se mark karna zaroori hota. Ab jab `pub use` ne `hosting` module ko root module se re-export kar diya hai, external code is ke bajaye path `restaurant::hosting::add_to_waitlist()` use kar sakta hai.

Re-exporting us waqt useful hoti hai jab aapke code ka internal structure us tareeqe se different ho jis tarah aapke code ko call karne wale programmers domain ke baare mein sochte hain. Misal ke taur par, is restaurant metaphor mein restaurant chalane wale log “front of house” aur “back of house” ke baare mein sochte hain. Lekin restaurant mein aane wale customers shayad restaurant ke parts ke baare mein in terms mein na sochen. `pub use` ke saath hum apne code ko ek structure ke saath likh sakte hain lekin ek different structure expose kar sakte hain. Aisa karne se hamari library un programmers ke liye achhi tarah organized rehti hai jo library par kaam kar rahe hain aur un programmers ke liye bhi jo library ko call kar rahe hain. Chapter 14 mein [“Exporting a Convenient Public API”][ch14-pub-use]<!-- ignore --> mein hum `pub use` ki ek aur example aur ye dekhenge ke ye aapke crate ki documentation ko kis tarah affect karta hai.

### External Packages Ko Use Karna

Chapter 2 mein hum ne ek guessing game project program kiya tha jo random numbers hasil karne ke liye `rand` naam ke ek external package ko use karta tha. Apne project mein `rand` ko use karne ke liye hum ne *Cargo.toml* mein ye line add ki thi:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:

* ch01-01-installation.md
* ch02-00-guessing-game-tutorial.md
* ch14-03-cargo-workspaces.md
-->

<Listing file-name="Cargo.toml">

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:9:}}
```

</Listing>

*Cargo.toml* mein `rand` ko dependency ke taur par add karne se Cargo ko pata chalta hai ke [crates.io](https://crates.io/) se `rand` package aur uski tamam dependencies download karni hain aur `rand` ko hamare project ke liye available banana hai.

Phir, `rand` ki definitions ko apne package ke scope mein lane ke liye, hum ne `use` ki ek line add ki jo crate ke name, `rand`, se shuru hoti thi aur un items ko list karti thi jinhein hum scope mein lana chahte the. Yaad karein ke Chapter 2 mein [“Generating a Random Number”][rand]<!-- ignore --> mein hum ne `rand::prelude` module ke items ko scope mein laya tha aur `rand::rng` function ko call kiya tha:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:ch07-04}}
```

Rust community ke members ne [crates.io](https://crates.io/) par bohat se packages available kiye hain, aur in mein se kisi bhi package ko apne package mein lane ke liye yahi steps involve hote hain: Unhein apne package ki *Cargo.toml* file mein list karna aur unke crates se items ko scope mein lane ke liye `use` ka istemal karna.

Ye note karein ke standard `std` library bhi ek aisa crate hai jo hamare package ke liye external hai. Kyun ke standard library Rust language ke saath ship hoti hai, is liye `std` ko include karne ke liye humein *Cargo.toml* mein koi change karne ki zaroorat nahi hoti. Lekin wahan se items ko apne package ke scope mein lane ke liye humein `use` ke zariye usay refer karna padta hai. Misal ke taur par, `HashMap` ke saath hum ye line use karenge:

```rust
use std::collections::HashMap;
```

Ye ek absolute path hai jo `std` se shuru hota hai, jo standard library crate ka name hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-nested-paths-to-clean-up-large-use-lists"></a>

### `use` Lists Ko Clean Up Karne Ke Liye Nested Paths Use Karna

Agar hum ek hi crate ya same module mein defined multiple items ko use kar rahe hon, to har item ko apni alag line par list karna hamari files mein bohat zyada vertical space le sakta hai. Misal ke taur par, Listing 2-4 mein guessing game mein hamare paas ye do `use` statements the jo `std` se items ko scope mein late hain:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-01-use-std-unnested/src/main.rs:here}}
```

</Listing>

Is ke bajaye, hum nested paths use karke same items ko ek hi line mein scope mein la sakte hain. Is ke liye hum path ka common part specify karte hain, uske baad do colons, aur phir paths ke different parts ki list ko curly brackets ke andar likhte hain, jaisa ke Listing 7-18 mein dikhaya gaya hai.

<Listing number="7-18" file-name="src/main.rs" caption="Same prefix wale multiple items ko scope mein lane ke liye nested path specify karna">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-18/src/main.rs:here}}
```

</Listing>

Bade programs mein, same crate ya module se bohat se items ko nested paths ke zariye scope mein lane se separate `use` statements ki zaroorat kaafi kam ho sakti hai!

Hum path ke kisi bhi level par nested path use kar sakte hain, jo us waqt useful hota hai jab hum do aise `use` statements ko combine karna chahte hon jo ek subpath share karte hain. Misal ke taur par, Listing 7-19 mein do `use` statements dikhaye gaye hain: ek jo `std::io` ko scope mein lata hai aur doosra jo `std::io::Write` ko scope mein lata hai.

<Listing number="7-19" file-name="src/lib.rs" caption="Do `use` statements jahan ek doosre ka subpath hai">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-19/src/lib.rs}}
```

</Listing>

In dono paths ka common part `std::io` hai, aur ye pehle path ka poora hissa hai. In dono paths ko ek `use` statement mein merge karne ke liye, hum nested path mein `self` use kar sakte hain, jaisa ke Listing 7-20 mein dikhaya gaya hai.

<Listing number="7-20" file-name="src/lib.rs" caption="Listing 7-19 ke paths ko ek `use` statement mein combine karna">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-20/src/lib.rs}}
```

</Listing>

Ye line `std::io` aur `std::io::Write` ko scope mein lati hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="the-glob-operator"></a>

### Glob Operator Ke Saath Items Import Karna

Agar hum kisi path mein define kiye gaye *tamam* public items ko scope mein lana chahte hain, to hum us path ke baad `*` glob operator specify kar sakte hain:

```rust
use std::collections::*;
```

Ye `use` statement `std::collections` mein define kiye gaye tamam public items ko current scope mein le aata hai. Glob operator use karte waqt ehtiyat karein! Glob ki wajah se ye batana mushkil ho sakta hai ke kaun se names scope mein hain aur aapke program mein use kiya gaya koi name kahan define hua tha. Is ke ilawa, agar dependency apni definitions change karti hai, to aapke imported items bhi change ho jate hain. Misal ke taur par, agar dependency upgrade karne par woh aisi definition add kar de jiska name same scope mein aapki kisi definition ke name ke saath same ho, to compiler errors aa sakte hain.

Glob operator ko aksar testing ke waqt `tests` module ke andar test ki ja rahi tamam cheezen lane ke liye use kiya jata hai; hum Chapter 11 mein [“How to Write Tests”][writing-tests]<!-- ignore --> mein is ke baare mein baat karenge. Glob operator kabhi kabhi prelude pattern ke hissa ke taur par bhi use hota hai: Is pattern ke baare mein mazeed maloomat ke liye [standard library documentation](../std/prelude/index.html#other-preludes)<!-- ignore --> dekhein.

[ch14-pub-use]: ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
[writing-tests]: ch11-01-writing-tests.html#how-to-write-tests
