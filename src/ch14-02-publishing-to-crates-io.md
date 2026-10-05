## Publishing a Crate to Crates.io

Humne [crates.io](https://crates.io/)<!-- ignore --> se packages ko apne project ki dependencies ke taur par use kiya hai, lekin aap apne packages publish karke apna code doosre logon ke saath share bhi kar sakte hain. [crates.io](https://crates.io/)<!-- ignore --> par crate registry aapke packages ka source code distribute karti hai, is liye yeh primarily us code ko host karti hai jo open source hota hai.

Rust aur Cargo mein aise features hain jo aapke published package ko logon ke liye find aur use karna aasaan banate hain. Ab hum in mein se kuch features ke baare mein baat karenge aur phir explain karenge ke package ko publish kaise karna hai.

### Making Useful Documentation Comments

Apne packages ki accurately documentation karna doosre users ko yeh samajhne mein madad dega ke unhein aapke packages ko kaise aur kab use karna chahiye, is liye documentation likhne mein waqt lagana worth it hai. Chapter 3 mein humne discuss kiya tha ke do slashes, `//`, use karke Rust code ko comment kaise karte hain. Rust mein documentation ke liye ek khaas qisam ka comment bhi hai, jise conveniently *documentation comment* kaha jata hai, aur jo HTML documentation generate karta hai. HTML documentation comments ke contents ko un public API items ke liye display karti hai jo un programmers ke liye intended hain jo yeh jaanna chahte hain ke aapke crate ko *use* kaise karna hai, na ke yeh ke aapka crate *implemented* kaise hai.

Documentation comments do ke bajaye teen slashes, `///`, use karte hain aur text ki formatting ke liye Markdown notation support karte hain. Documentation comments ko us item ke bilkul pehle place karein jise woh document kar rahe hain. Listing 14-1 mein `my_crate` naam ke crate mein ek `add_one` function ke liye documentation comments dikhaye gaye hain.

<Listing number="14-1" file-name="src/lib.rs" caption="A documentation comment for a function">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-01/src/lib.rs}}
```

</Listing>

Yahan, hum `add_one` function kya karta hai uski description dete hain, `Examples` heading ke saath ek section shuru karte hain, aur phir code provide karte hain jo demonstrate karta hai ke `add_one` function ko kaise use karna hai. Hum `cargo doc` run karke is documentation comment se HTML documentation generate kar sakte hain. Yeh command Rust ke saath distributed `rustdoc` tool ko run karti hai aur generated HTML documentation ko *target/doc* directory mein rakhti hai.

Convenience ke liye, `cargo doc --open` run karne se aapke current crate ki documentation ke liye HTML build hoti hai (saath hi aapke crate ki tamam dependencies ki documentation bhi) aur result ko web browser mein open kar deta hai. `add_one` function par navigate karein aur aap dekhenge ke documentation comments mein likha hua text kis tarah render hota hai, jaisa ke Figure 14-1 mein dikhaya gaya hai.

<img alt="Rendered HTML documentation for the `add_one` function of `my_crate`" src="img/trpl14-01.png" class="center" />

<span class="caption">Figure 14-1: The HTML documentation for the `add_one`
function</span>

#### Commonly Used Sections

Humne Listing 14-1 mein `# Examples` Markdown heading ko HTML mein “Examples” title ke saath ek section create karne ke liye use kiya. Yahan kuch doosre sections hain jinhein crate authors apni documentation mein commonly use karte hain:

* **Panics**: Yeh woh scenarios hain jin mein documented function panic kar sakta hai. Function ko call karne wale woh callers jo apne programs ko panic nahi karwana chahte, unhein ensure karna chahiye ke woh in situations mein function ko call na karein.
* **Errors**: Agar function `Result` return karta hai, to un errors ki types describe karna jo occur ho sakti hain aur yeh batana ke kin conditions ki wajah se woh errors return ho sakti hain, callers ke liye helpful ho sakta hai taake woh different kinds of errors ko different ways mein handle karne ke liye code likh sakein.
* **Safety**: Agar function ko call karna `unsafe` ho (hum Chapter 20 mein unsafety discuss karenge), to ek section hona chahiye jo explain kare ke function unsafe kyun hai aur un invariants ko cover kare jinhein function callers se uphold karne ki expectation rakhta hai.

Zyada tar documentation comments ko in tamam sections ki zaroorat nahi hoti, lekin yeh ek achhi checklist hai jo aapko aapke code ke un aspects ki yaad dilati hai jin ke baare mein users jaanne mein interested honge.

#### Documentation Comments as Tests

Apne documentation comments mein example code blocks add karna aapki library ko use karne ka tareeqa demonstrate karne mein madad kar sakta hai aur iska ek additional bonus bhi hai: `cargo test` run karne se aapki documentation mein diye gaye code examples bhi tests ke taur par run honge! Examples wali documentation se behtar kuch nahi. Lekin un examples se bura bhi kuch nahi jo is liye kaam nahi karte kyun ke documentation likhe jane ke baad code change ho gaya hai. Agar hum Listing 14-1 ke `add_one` function ki documentation ke saath `cargo test` run karein, to humein test results mein ek section nazar aayega jo kuch is tarah hoga:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo test
copy just the doc-tests section below
-->

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

Ab, agar hum function ya example mein is tarah change karein ke example mein `assert_eq!` panic kare, aur phir `cargo test` dobara run karein, to hum dekhenge ke doc tests yeh catch kar lete hain ke example aur code ek doosre ke saath out of sync hain!

<!-- Old headings. Do not remove or links may break. -->

<a id="commenting-contained-items"></a>

#### Contained Item Comments

Doc comment ka style `//!` us item ko documentation provide karta hai jo comments ko *contain* karta hai, bajaye un items ke jo comments ke *baad* aate hain. Hum aam tor par in doc comments ko crate root file (*src/lib.rs* convention ke mutabiq) ke andar ya kisi module ke andar use karte hain taake crate ya module ko as a whole document kiya ja sake.

Misal ke taur par, `add_one` function ko contain karne wale `my_crate` crate ke purpose ko describe karne wali documentation add karne ke liye, hum *src/lib.rs* file ke beginning mein `//!` se shuru hone wale documentation comments add karte hain, jaisa ke Listing 14-2 mein dikhaya gaya hai.

<Listing number="14-2" file-name="src/lib.rs" caption="The documentation for the `my_crate` crate as a whole">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-02/src/lib.rs:here}}
```

</Listing>

Note karein ke `//!` se shuru hone wali aakhri line ke baad koi code nahi hai. Kyun ke humne comments ko `///` ke bajaye `//!` se shuru kiya hai, hum us item ko document kar rahe hain jo is comment ko contain karta hai, na ke us item ko jo is comment ke baad aata hai. Is case mein, woh item *src/lib.rs* file hai, jo crate root hai. Yeh comments poore crate ko describe karte hain.

Jab hum `cargo doc --open` run karte hain, to yeh comments `my_crate` ki documentation ke front page par crate ke public items ki list ke upar display honge, jaisa ke Figure 14-2 mein dikhaya gaya hai.

Items ke andar documentation comments crates aur modules ko describe karne ke liye khaas tor par useful hain. Inhein container ke overall purpose ko explain karne ke liye use karein taake aapke users crate ki organization ko samajh sakein.

<img alt="Rendered HTML documentation with a comment for the crate as a whole" src="img/trpl14-02.png" class="center" />

<span class="caption">Figure 14-2: The rendered documentation for `my_crate`,
including the comment describing the crate as a whole</span>

<!-- Old headings. Do not remove or links may break. -->

<a id="exporting-a-convenient-public-api-with-pub-use"></a>

### Exporting a Convenient Public API

Apne public API ki structure ko samajhna crate publish karte waqt ek major consideration hai. Jo log aapke crate ko use karte hain, woh uski structure se aapse kam familiar hote hain aur agar aapke crate mein large module hierarchy ho to unhein apni zaroorat ke pieces dhoondhne mein difficulty ho sakti hai.

Chapter 7 mein humne cover kiya tha ke `pub` keyword use karke items ko public kaise banaya jata hai aur `use` keyword ke zariye items ko scope mein kaise laya jata hai. Lekin jo structure aapke liye crate develop karte waqt sensible hoti hai, zaroori nahi ke woh aapke users ke liye bhi convenient ho. Aap apne structs ko multiple levels wali hierarchy mein organize karna chah sakte hain, lekin phir jo log hierarchy mein deeply defined kisi type ko use karna chahte hain, unhein yeh pata lagane mein mushkil ho sakti hai ke woh type exist karta hai. Unhein yeh bhi annoying lag sakta hai ke unhein `use
my_crate::some_module::another_module::UsefulType;` enter karna pade, bajaye `use
my_crate::UsefulType;` ke.

Achhi baat yeh hai ke agar structure doosri library se use karne wale logon ke liye *convenient* nahi hai, to aapko apni internal organization ko rearrange karne ki zaroorat nahi. Iske bajaye, aap `pub use` use karke items ko re-export kar sakte hain, taake ek aisi public structure ban sake jo aapki private structure se different ho. *Re-exporting* ek public item ko ek location par leta hai aur use doosri location par public bana deta hai, jaise ke woh doosri location par hi defined ho.

Misal ke taur par, maan lein ke humne artistic concepts ko model karne ke liye `art` naam ki ek library banayi. Is library ke andar do modules hain: ek `kinds` module jismein `PrimaryColor` aur `SecondaryColor` naam ke do enums hain aur ek `utils` module jismein `mix` naam ka function hai, jaisa ke Listing 14-3 mein dikhaya gaya hai.

<Listing number="14-3" file-name="src/lib.rs" caption="An `art` library with items organized into `kinds` and `utils` modules">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-03/src/lib.rs:here}}
```

</Listing>

Figure 14-3 dikhata hai ke `cargo doc` ke zariye generate ki gayi is crate ki documentation ka front page kaisa nazar aayega.

<img alt="Rendered documentation for the `art` crate that lists the `kinds` and `utils` modules" src="img/trpl14-03.png" class="center" />

<span class="caption">Figure 14-3: The front page of the documentation for `art`
that lists the `kinds` and `utils` modules</span>

Note karein ke `PrimaryColor` aur `SecondaryColor` types front page par listed nahi hain, aur na hi `mix` function hai. Unhein dekhne ke liye humein `kinds` aur `utils` par click karna hoga.

Ek doosra crate jo is library par depend karta hai, usay items ko `art` se scope mein lane ke liye `use` statements ki zaroorat hogi, jismein woh module structure specify karni hogi jo filhaal defined hai. Listing 14-4 ek aise crate ki example dikhati hai jo `art` crate se `PrimaryColor` aur `mix` items ko use karta hai.

<Listing number="14-4" file-name="src/main.rs" caption="A crate using the `art` crate’s items with its internal structure exported">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-04/src/main.rs}}
```

</Listing>

Listing 14-4 mein code ke author ko, jo `art` crate use kar raha hai, yeh figure out karna pada ke `PrimaryColor` `kinds` module mein hai aur `mix` `utils` module mein hai. `art` crate ki module structure un developers ke liye zyada relevant hai jo `art` crate par kaam kar rahe hain, bajaye un logon ke jo ise use kar rahe hain. Internal structure kisi aise shakhs ke liye koi useful information provide nahi karti jo yeh samajhne ki koshish kar raha ho ke `art` crate ko kaise use karna hai; balki yeh confusion create karti hai kyun ke jo developers ise use karte hain unhein figure out karna padta hai ke kahan dekhna hai, aur `use` statements mein module names specify karne padte hain.

Internal organization ko public API se remove karne ke liye, hum Listing 14-3 mein `art` crate ke code ko modify karke top level par items ko re-export karne ke liye `pub use` statements add kar sakte hain, jaisa ke Listing 14-5 mein dikhaya gaya hai.

<Listing number="14-5" file-name="src/lib.rs" caption="Adding `pub use` statements to re-export items">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-05/src/lib.rs:here}}
```

</Listing>

Is crate ke liye `cargo doc` jo API documentation generate karega, ab front page par re-exports ko list aur link karega, jaisa ke Figure 14-4 mein dikhaya gaya hai, jis se `PrimaryColor` aur `SecondaryColor` types aur `mix` function ko dhoondhna aasaan ho jayega.

<img alt="Rendered documentation for the `art` crate with the re-exports on the front page" src="img/trpl14-04.png" class="center" />

<span class="caption">Figure 14-4: The front page of the documentation for `art`
that lists the re-exports</span>

`art` crate ke users ab bhi Listing 14-3 ki internal structure ko dekh aur use kar sakte hain, jaisa ke Listing 14-4 mein demonstrate kiya gaya hai, ya woh Listing 14-5 ki zyada convenient structure use kar sakte hain, jaisa ke Listing 14-6 mein dikhaya gaya hai.

<Listing number="14-6" file-name="src/main.rs" caption="A program using the re-exported items from the `art` crate">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-06/src/main.rs:here}}
```

</Listing>

Jin cases mein bohat se nested modules hon, wahan `pub use` ke zariye types ko top level par re-export karna crate use karne wale logon ke experience mein significant difference la sakta hai. `pub use` ka ek aur common use yeh hai ke current crate mein kisi dependency ki definitions ko re-export kiya jaye, taake us crate ki definitions aapke crate ke public API ka hissa ban jayein.

Ek useful public API structure create karna science se zyada ek art hai, aur aap iterate karke woh API dhoondh sakte hain jo aapke users ke liye best kaam karti ho. `pub use` choose karne se aapko apne crate ko internally structure karne mein flexibility milti hai aur aapki internal structure ko us cheez se decouple kar deta hai jo aap apne users ko present karte hain. Kuch un crates ka code dekhein jo aapne install kiye hain aur check karein ke kya unki internal structure unke public API se different hai.


### Setting Up a Crates.io Account

Is se pehle ke aap koi crates publish kar sakein, aapko [crates.io](https://crates.io/)<!-- ignore --> par ek account create karna hoga aur ek API token lena hoga. Iske liye [crates.io](https://crates.io/)<!-- ignore --> ke home page par jaakar GitHub account ke zariye log in karein. (Filhaal GitHub account ek requirement hai, lekin future mein site account create karne ke doosre tareeqe bhi support kar sakti hai.) Jab aap log in ho jayein, to apni account settings par https://crates.io/me/<!-- ignore --> jaakar apni API key retrieve karein. Phir `cargo login` command run karein aur jab prompt aaye to apni API key paste karein, jaisa ke yahan dikhaya gaya hai:

```console id="e4a8d2"
$ cargo login
abcdefghijklmnopqrstuvwxyz012345
```

Yeh command Cargo ko aapke API token ke baare mein inform karegi aur use locally *~/.cargo/credentials.toml* mein store karegi. Note karein ke yeh token ek secret hai: Ise kisi aur ke saath share na karein. Agar aap kisi bhi wajah se ise kisi ke saath share kar dete hain, to aapko ise revoke karna chahiye aur [crates.io](https://crates.io/)<!-- ignore
--> par ek naya token generate karna chahiye.

### Adding Metadata to a New Crate

Maan lein ke aapke paas ek crate hai jise aap publish karna chahte hain. Publish karne se pehle, aapko crate ki *Cargo.toml* file ke `[package]` section mein kuch metadata add karna hoga.

Aapke crate ka ek unique name hona zaroori hai. Jab aap locally kisi crate par kaam kar rahe hote hain, to aap crate ka naam apni marzi se rakh sakte hain. Lekin [crates.io](https://crates.io/)<!-- ignore --> par crate names first-come, first-served basis par allocate hote hain. Ek baar crate ka naam le liya jaye, to koi aur us naam se crate publish nahi kar sakta. Crate publish karne ki koshish karne se pehle, jis name ko aap use karna chahte hain usay search karein. Agar woh name pehle se use ho chuka hai, to aapko doosra name dhoondhna hoga aur *Cargo.toml* file mein `[package]` section ke andar `name` field ko naye name ke saath edit karna hoga, jaisa ke yahan hai:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
```

Chahe aapne unique name choose kiya ho, jab aap is point par crate publish karne ke liye `cargo publish` run karenge, to aapko pehle ek warning aur phir ek error milega:

<!-- manual-regeneration
Create a new package with an unregistered name, making no further modifications
  to the generated package, so it is missing the description and license fields.
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
See https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata for more info.
--snip--
error: failed to publish to registry at https://crates.io

Caused by:
  the remote server responded with an error (status 400 Bad Request): missing or empty metadata fields: description, license. Please see https://doc.rust-lang.org/cargo/reference/manifest.html for more information on configuring these fields
```

Yeh error is liye aata hai kyun ke aap kuch crucial information miss kar rahe hain: Ek description aur license required hain taake log jaan sakein ke aapka crate kya karta hai aur woh ise kin terms ke tehat use kar sakte hain. *Cargo.toml* mein ek description add karein jo sirf ek ya do sentences ki ho, kyun ke yeh search results mein aapke crate ke saath appear hogi. `license` field ke liye, aapko ek *license identifier value* deni hogi. [Linux Foundation’s Software Package Data Exchange (SPDX)][spdx] un identifiers ki list deta hai jinhein aap is value ke liye use kar sakte hain. Misal ke taur par, yeh specify karne ke liye ke aapne apne crate ko MIT License ke tehat license kiya hai, `MIT` identifier add karein:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
license = "MIT"
```

Agar aap aisi license use karna chahte hain jo SPDX mein appear nahi hoti, to aapko us license ka text ek file mein rakhna hoga, file ko apne project mein include karna hoga, aur phir `license` key use karne ke bajaye `license-file` se us file ka name specify karna hoga.

Aapke project ke liye kaunsi license appropriate hai, iski guidance is book ke scope se bahar hai. Rust community mein bohat se log Rust ki tarah apne projects ko `MIT OR Apache-2.0` ki dual license ke saath license karte hain. Yeh practice demonstrate karti hai ke aap multiple license identifiers ko `OR` se separate karke apne project ke liye multiple licenses bhi specify kar sakte hain.

Ek unique name, version, description, aur license add karne ke baad, publish ke liye ready project ki *Cargo.toml* file kuch is tarah nazar aa sakti hai:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2024"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

[Cargo’s documentation](https://doc.rust-lang.org/cargo/) mein doosre metadata ko describe kiya gaya hai jo aap specify kar sakte hain taake doosre log aapke crate ko zyada aasani se discover aur use kar sakein.

### Publishing to Crates.io

Ab jab aap ne account bana liya hai, apna API token save kar liya hai, apne
crate ke liye ek name choose kar liya hai, aur required metadata specify kar
diya hai, to aap publish karne ke liye ready hain! Crate ko publish karne se
[crates.io](https://crates.io/)<!-- ignore --> par ek specific version upload
ho jata hai taa-ke doosre log usay use kar saken.

Ehtiyat karein, kyun ke publish karna *permanent* hota hai. Version ko kabhi
overwrite nahi kiya ja sakta, aur code ko kuch khaas circumstances ke ilawa
delete nahi kiya ja sakta. Crates.io ka ek major goal code ke ek permanent
archive ke taur par kaam karna hai taa-ke un tamam projects ki builds jo
[crates.io](https://crates.io/)<!-- ignore --> ke crates par depend karti hain
continues kaam karti rahen. Version deletions ki ijazat dena is goal ko poora
karna namumkin bana dega. Haan, aap jitne chahein utne crate versions publish
kar sakte hain; is ki koi limit nahi hai.

`cargo publish` command dobara run karein. Ab yeh successfully complete honi
chahiye:

<!-- manual-regeneration
go to some valid crate, publish a new version
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
    Packaged 6 files, 1.2KiB (895.0B compressed)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
    Uploaded guessing_game v0.1.0 to registry `crates-io`
note: waiting for `guessing_game v0.1.0` to be available at registry
`crates-io`.
You may press ctrl-c to skip waiting; the crate should be available shortly.
   Published guessing_game v0.1.0 at registry `crates-io`
```

Mubarak ho! Ab aap ne apna code Rust community ke saath share kar diya hai, aur
ab koi bhi aasani se aapke crate ko apne project ki dependency ke taur par add
kar sakta hai.

### Publishing a New Version of an Existing Crate

Jab aap apne crate mein changes kar chuke hon aur new version release karne ke
liye ready hon, to apni *Cargo.toml* file mein specify ki gayi `version` value
change karein aur crate ko dobara publish karein. Aap ne jo changes kiye hain,
un ki type ki bunyaad par appropriate next version number decide karne ke liye
[Semantic Versioning rules][semver] use karein. Phir new version ko upload karne
ke liye `cargo publish` run karein.

<!-- Old headings. Do not remove or links may break. -->

<a id="removing-versions-from-cratesio-with-cargo-yank"></a> <a id="deprecating-versions-from-cratesio-with-cargo-yank"></a>

### Deprecating Versions from Crates.io

Halanke aap crate ke previous versions ko remove nahi kar sakte, lekin aap
future projects ko unhein new dependency ke taur par add karne se rok sakte
hain. Yeh us waqt useful hota hai jab kisi wajah se crate ka koi version
broken ho. Aisi situations mein Cargo crate version ko yank karne ki support
deta hai.

*Yanking* kisi version ko naye projects ke liye us version par depend karne se
rok deta hai, jabke tamam existing projects jo us par depend karte hain unhein
continue karne deta hai. Asal mein, yank ka matlab yeh hai ke jin tamam
projects ke paas *Cargo.lock* hai woh break nahi honge, aur future mein
generate hone wali koi bhi *Cargo.lock* file yanked version ko use nahi karegi.

Crate ke kisi version ko yank karne ke liye, us crate ki directory mein jise aap
pehle publish kar chuke hain, `cargo yank` run karein aur specify karein ke
aap kis version ko yank karna chahte hain. Misal ke taur par, agar hum ne
`guessing_game` naam ka crate version `1.0.1` publish kiya hai aur hum use
yank karna chahte hain, to hum `guessing_game` ki project directory mein
following command run karenge:

<!-- manual-regeneration:
cargo yank carol-test --version 2.1.0
cargo yank carol-test --version 2.1.0 --undo
-->

```console
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
```

Command mein `--undo` add karke aap yank ko undo bhi kar sakte hain aur
projects ko dobara us version par depend karne ki ijazat de sakte hain:

```console
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
```

Yank *kisi bhi code ko* delete **nahi** karta. Misal ke taur par, yeh galti se
upload kiye gaye secrets ko delete nahi kar sakta. Agar aisa ho jaye, to aapko
un secrets ko foran reset karna hoga.

[spdx]: https://spdx.org/licenses/
[semver]: https://semver.org/
