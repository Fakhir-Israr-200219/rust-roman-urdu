## Tests Ki Organization

Jaisa ke chapter ke shuru mein mention kiya gaya tha, testing ek complex discipline hai, aur different log different terminology aur organization use karte hain. Rust community tests ko do main categories ke taur par dekhti hai: unit tests aur integration tests. *Unit tests* chhote aur zyada focused hote hain, ek waqt mein ek module ko isolation mein test karte hain, aur private interfaces ko test kar sakte hain. *Integration tests* aapki library se completely external hote hain aur aapke code ko usi tarah use karte hain jis tarah koi doosra external code karega, sirf public interface ko use karte hue aur potentially har test mein multiple modules ko exercise karte hue.

Dono qisam ke tests likhna important hai taake yeh ensure kiya ja sake ke aapki library ke pieces separately aur together, dono surat mein, wohi kaam kar rahe hain jo aap un se expect karte hain.

### Unit Tests

Unit tests ka maqsad yeh hota hai ke code ke har unit ko baqi code se isolation mein test kiya jaye, taake jaldi se pinpoint kiya ja sake ke code ka kaunsa hissa expected tareeqe se kaam kar raha hai aur kaunsa nahi. Aap unit tests ko *src* directory mein, har us file ke andar rakhenge jis mein woh code maujood hai jise tests kar rahe hain. Convention yeh hai ke har file mein test functions ko contain karne ke liye `tests` naam ka ek module create kiya jaye aur us module ko `cfg(test)` se annotate kiya jaye.


#### `tests` Module aur `#[cfg(test)]`

`tests` module par `#[cfg(test)]` annotation Rust ko batata hai ke test code ko sirf us waqt compile aur run kare jab aap `cargo test` run karein, `cargo build` run karte waqt nahi. Is se jab aap sirf library build karna chahte hain to compile time bachta hai aur resulting compiled artifact mein space bhi bachta hai, kyun ke tests us mein include nahi hote. Aap dekhenge ke integration tests ek different directory mein hote hain, is liye unhein `#[cfg(test)]` annotation ki zaroorat nahi hoti. Lekin kyun ke unit tests usi files mein hote hain jin mein code hota hai, aap `#[cfg(test)]` use karenge taake specify kiya ja sake ke unhein compiled result mein include nahi kiya jana chahiye.

Yaad karein ke jab humne is chapter ke pehle section mein naya `adder` project generate kiya tha, to Cargo ne hamare liye yeh code generate kiya tha:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground id="m8q3vt"
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

Automatically generated `tests` module par `cfg` attribute *configuration* ke liye hai aur Rust ko batata hai ke following item ko sirf tab include kiya jaye jab koi certain configuration option di gayi ho. Is case mein configuration option `test` hai, jo Rust tests ko compile aur run karne ke liye provide karta hai. `cfg` attribute ko use karke Cargo hamare test code ko sirf us waqt compile karta hai jab hum actively `cargo test` ke sath tests run karte hain. Is mein is module ke andar maujood koi bhi helper functions bhi shamil hain, un functions ke ilawa jo `#[test]` se annotated hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="testing-private-functions"></a>

#### Private Function Tests

Testing community mein is baat par debate hai ke private functions ko directly test karna chahiye ya nahi, aur doosri languages private functions ko test karna mushkil ya impossible bana deti hain. Aap testing ki jis ideology ko bhi follow karein, Rust ke privacy rules aapko private functions ko test karne ki ijazat dete hain. Listing 11-12 mein diye gaye code par ghour karein, jismein private function `internal_adder` hai.

<Listing number="11-12" file-name="src/lib.rs" caption="Testing a private function">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-12/src/lib.rs}}
```

</Listing>

Note karein ke `internal_adder` function ko `pub` mark nahi kiya gaya. Tests sirf Rust code hain, aur `tests` module bhi sirf ek aur module hai. Jaisa ke humne [“Paths for Referring to an Item in the Module Tree”][paths]<!-- ignore --> mein discuss kiya tha, child modules ke items apne ancestor modules ke items ko use kar sakte hain. Is test mein hum `tests` module ke parent se belong karne wale tamam items ko `use super::*` ke zariye scope mein laate hain, aur phir test `internal_adder` ko call kar sakta hai. Agar aap nahi samajhte ke private functions ko test karna chahiye, to Rust mein aisi koi cheez nahi jo aapko aisa karne par majboor kare.

### Integration Tests

Rust mein integration tests aapki library se entirely external hote hain. Yeh aapki library ko bilkul usi tarah use karte hain jaise koi aur code karega, jis ka matlab hai ke yeh sirf un functions ko call kar sakte hain jo aapki library ke public API ka hissa hain. In ka maqsad yeh test karna hai ke aapki library ke bohat se parts mil kar sahi tareeqe se kaam karte hain ya nahi. Code ke woh units jo apne taur par sahi tareeqe se kaam karte hain, unhein integrate karne par problems ho sakti hain, is liye integrated code ki test coverage bhi important hai. Integration tests create karne ke liye, aapko sab se pehle ek *tests* directory ki zaroorat hoti hai.

#### The *tests* Directory

Hum apne project directory ke top level par, *src* ke paas, ek *tests* directory create karte hain. Cargo jaanta hai ke integration test files ko is directory mein dhoondhna hai. Phir hum jitni test files chahein bana sakte hain, aur Cargo in mein se har file ko ek individual crate ke taur par compile karega.

Aaiye ek integration test create karte hain. Listing 11-12 ka code abhi bhi *src/lib.rs* file mein maujood ho, to ek *tests* directory banayein, aur *tests/integration_test.rs* naam ki ek nayi file create karein. Aapki directory structure kuch is tarah honi chahiye:

```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

Listing 11-13 ka code *tests/integration_test.rs* file mein enter karein.

<Listing number="11-13" file-name="tests/integration_test.rs" caption="An integration test of a function in the `adder` crate">

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-13/tests/integration_test.rs}}
```

</Listing>

*tests* directory mein har file ek separate crate hoti hai, is liye humein apni library ko har test crate ke scope mein lana hota hai. Isi wajah se hum code ke top par `use adder::add_two;` add karte hain, jo humein unit tests mein karne ki zaroorat nahi thi.

Humein *tests/integration_test.rs* mein kisi bhi code ko `#[cfg(test)]` se annotate karne ki zaroorat nahi hai. Cargo *tests* directory ko specially treat karta hai aur is directory mein maujood files ko sirf tab compile karta hai jab hum `cargo test` run karte hain. Ab `cargo test` run karein:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-13/output.txt}}
```

Output ke teen sections mein unit tests, integration test, aur doc tests shamil hain. Note karein ke agar kisi section mein koi test fail ho jaye, to uske baad wale sections run nahi kiye jayenge. Misal ke taur par, agar koi unit test fail ho jaye, to integration aur doc tests ke liye koi output nahi hoga, kyun ke woh tests tabhi run kiye jayenge jab tamam unit tests pass ho rahe hon.

Unit tests ka pehla section wahi hai jo hum dekhte aa rahe hain: har unit test ke liye ek line (Listing 11-12 mein humne jo `internal` naam ka test add kiya tha, woh bhi) aur phir unit tests ke liye ek summary line.

Integration tests section `Running tests/integration_test.rs` line se shuru hota hai. Iske baad us integration test mein har test function ke liye ek line hoti hai aur integration test ke results ke liye ek summary line hoti hai, us se bilkul pehle jab `Doc-tests adder` section shuru hota hai.

Har integration test file ka apna section hota hai, is liye agar hum *tests* directory mein aur files add karein, to integration test ke aur sections honge.

Hum ab bhi kisi particular integration test function ko `cargo test` ke argument ke taur par test function ka naam specify karke run kar sakte hain. Kisi particular integration test file mein tamam tests run karne ke liye, `cargo test` ka `--test` argument use karein aur uske baad file ka naam dein:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-05-single-integration/output.txt}}
```

Yeh command sirf *tests/integration_test.rs* file mein maujood tests ko run karti hai.

#### Submodules in Integration Tests

Jab aap zyada integration tests add karte hain, to aap unhein organize karne mein madad ke liye *tests* directory mein aur files banana chah sakte hain; misal ke taur par, aap test functions ko us functionality ke mutabiq group kar sakte hain jise woh test kar rahe hain. Jaisa ke pehle bataya gaya hai, *tests* directory mein har file apne alag separate crate ke taur par compile hoti hai, jo separate scopes create karne ke liye useful hai taake us tareeqe ko zyada closely imitate kiya ja sake jis tarah end users aapki crate ko use karenge. Lekin iska matlab yeh hai ke *tests* directory ki files ka behavior *src* ki files jaisa nahi hota, jaisa ke aapne Chapter 7 mein code ko modules aur files mein separate karne ke hawale se seekha tha.

*tests* directory ki files ka different behavior sab se zyada us waqt noticeable hota hai jab aapke paas helper functions ka ek set ho jise multiple integration test files mein use karna ho, aur aap unhein ek common module mein extract karne ke liye Chapter 7 ke [“Separating Modules into Different Files”][separating-modules-into-files]<!-- ignore --> section ke steps follow karne ki koshish karein. Misal ke taur par, agar hum *tests/common.rs* create karein aur us mein `setup` naam ka ek function rakhein, to hum `setup` mein kuch aisa code add kar sakte hain jise hum multiple test files mein maujood multiple test functions se call karna chahte hain:

<span class="filename">Filename: tests/common.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/tests/common.rs}}
```

Jab hum tests ko dobara run karenge, to test output mein *common.rs* file ke liye ek naya section nazar aayega, halaan ke is file mein koi test functions maujood nahi hain aur na hi humne kahin se `setup` function ko call kiya hai:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/output.txt}}
```

Test results mein `common` ka `running 0 tests` ke sath appear hona woh nahi tha jo hum chahte the. Hum sirf doosri integration test files ke sath kuch code share karna chahte the. `common` ko test output mein appear hone se rokne ke liye, *tests/common.rs* create karne ke bajaye, hum *tests/common/mod.rs* create karenge. Ab project directory is tarah nazar aati hai:

```text
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

Yeh woh older naming convention hai jise Rust bhi samajhta hai, jiska humne Chapter 7 mein [“Alternate File Paths”][alt-paths]<!-- ignore --> mein zikr kiya tha. File ko is tarah naam dene se Rust ko pata chalta hai ke `common` module ko integration test file ke taur par treat nahi karna hai. Jab hum `setup` function ke code ko *tests/common/mod.rs* mein move kar dete hain aur *tests/common.rs* file ko delete kar dete hain, to test output mein section dobara appear nahi hoga. *tests* directory ki subdirectories mein maujood files separate crates ke taur par compile nahi hoti aur na hi test output mein unke sections hote hain.

Jab hum *tests/common/mod.rs* create kar lete hain, to hum ise kisi bhi integration test file se ek module ke taur par use kar sakte hain. Yahan *tests/integration_test.rs* mein `it_adds_two` test se `setup` function ko call karne ki ek example hai:

<span class="filename">Filename: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-13-fix-shared-test-code-problem/tests/integration_test.rs}}
```

Note karein ke `mod common;` declaration bilkul usi module declaration jaisi hai jo humne Listing 7-21 mein demonstrate ki thi. Phir, test function mein hum `common::setup()` function ko call kar sakte hain.

#### Integration Tests for Binary Crates

Agar hamara project ek binary crate hai jo sirf ek *src/main.rs* file par mushtamil hai aur is mein *src/lib.rs* file nahi hai, to hum *tests* directory mein integration tests create nahi kar sakte aur na hi *src/main.rs* file mein defined functions ko `use` statement ke zariye scope mein la sakte hain. Sirf library crates hi aise functions expose karti hain jinhein doosri crates use kar sakti hain; binary crates ka maqsad apne aap run hona hota hai.

Yeh un reasons mein se ek hai jis ki wajah se woh Rust projects jo ek binary provide karte hain, un mein ek straightforward *src/main.rs* file hoti hai jo us logic ko call karti hai jo *src/lib.rs* file mein maujood hota hai. Is structure ko use karte hue, integration tests library crate ko `use` ke zariye test kar sakte hain taake important functionality available ho. Agar important functionality kaam karti hai, to *src/main.rs* file mein maujood chhoti si code ki miktar bhi kaam karegi, aur code ki us chhoti si miktar ko test karne ki zaroorat nahi hoti.

## Summary

Rust ki testing features code ke expected behavior ko specify karne ka ek tareeqa provide karti hain, taake yeh ensure kiya ja sake ke changes karne ke bawajood code aapki expectation ke mutabiq kaam karta rahe. Unit tests library ke different parts ko separately exercise karte hain aur private implementation details ko bhi test kar sakte hain. Integration tests check karte hain ke library ke bohat se parts mil kar sahi tareeqe se kaam karte hain, aur code ko test karne ke liye library ke public API ko use karte hain, bilkul usi tarah jaise external code library ko use karega. Agarche Rust ka type system aur ownership rules kuch qisam ke bugs ko prevent karne mein madad karte hain, phir bhi tests important hain taake un logic bugs ko kam kiya ja sake jo aapke code ke expected behavior se related hote hain.

Aaiye is chapter aur previous chapters mein seekhi hui knowledge ko combine karke ek project par kaam karte hain!

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[separating-modules-into-files]: ch07-05-separating-modules-into-different-files.html
[alt-paths]: ch07-05-separating-modules-into-different-files.html#alternate-file-paths
