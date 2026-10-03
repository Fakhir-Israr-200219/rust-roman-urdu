<!-- Old headings. Do not remove or links may break. -->

<a id="defining-modules-to-control-scope-and-privacy"></a>
## Modules Ke Zariye Scope aur Privacy Control Karna

Is section mein hum modules aur module system ke doosre parts ke baare mein baat karenge, yani *paths*, jo aapko items ko name karne dete hain; `use` keyword jo kisi path ko scope mein lata hai; aur `pub` keyword jo items ko public banata hai. Hum `as` keyword, external packages, aur glob operator ke baare mein bhi discuss karenge.

### Modules Cheat Sheet

Modules aur paths ki details mein jane se pehle, yahan hum ek quick reference provide karte hain ke compiler mein modules, paths, `use` keyword, aur `pub` keyword kis tarah kaam karte hain, aur zyada tar developers apne code ko kis tarah organize karte hain. Is chapter mein hum in mein se har rule ki examples dekhenge, lekin modules kis tarah kaam karte hain iski yaad-dihani ke liye ye ek behtareen jagah hai.

* **Crate root se shuru karein**: Jab ek crate compile kiya jata hai, to compiler sab se pehle crate root file (aam tor par library crate ke liye *src/lib.rs* aur binary crate ke liye *src/main.rs*) mein compile kiye jane wale code ko dekhta hai.
* **Modules declare karna**: Crate root file mein aap naye modules declare kar sakte hain; maan lein aap `mod garden;` ke zariye ek “garden” module declare karte hain. Compiler module ka code in jagahon par dekhega:

  * Inline, curly brackets ke andar jo `mod
    garden` ke baad semicolon ki jagah use kiye gaye hain
  * File *src/garden.rs* mein
  * File *src/garden/mod.rs* mein
* **Submodules declare karna**: Crate root ke ilawa kisi bhi file mein aap submodules declare kar sakte hain. Misal ke taur par, aap *src/garden.rs* mein `mod vegetables;` declare kar sakte hain. Compiler parent module ke name wali directory ke andar submodule ka code in jagahon par dekhega:

  * Inline, `mod vegetables` ke bilkul baad, semicolon ki jagah curly brackets ke andar
  * File *src/garden/vegetables.rs* mein
  * File *src/garden/vegetables/mod.rs* mein
* **Modules mein code ke paths**: Jab koi module aapke crate ka hissa ban jata hai, to aap usi crate mein kahin se bhi us module ke code ko refer kar sakte hain, basharte ke privacy rules iski ijazat dein, aur is ke liye code ka path use kiya jata hai. Misal ke taur par, garden vegetables module mein ek `Asparagus` type `crate::garden::vegetables::Asparagus` par milegi.
* **Private vs. public**: Kisi module ke andar ka code by default uske parent modules se private hota hai. Kisi module ko public banane ke liye `mod` ke bajaye `pub mod` se declare karein. Kisi public module ke andar ke items ko bhi public banane ke liye unki declarations se pehle `pub` use karein.
* **`use` keyword**: Kisi scope ke andar, `use` keyword items ke shortcuts create karta hai taa-ke lambe paths ko baar baar likhne ki zaroorat kam ho. Kisi bhi aise scope mein jo `crate::garden::vegetables::Asparagus` ko refer kar sakta ho, aap `use
  crate::garden::vegetables::Asparagus;` ke zariye ek shortcut create kar sakte hain, aur uske baad is scope mein is type ko use karne ke liye aapko sirf `Asparagus` likhna hoga.

Yahan hum `backyard` naam ka ek binary crate create karte hain jo in rules ko illustrate karta hai. Crate ki directory, jiska name bhi *backyard* hai, in files aur directories par mushtamil hai:

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

Is case mein crate root file *src/main.rs* hai, aur is mein ye code hai:

<Listing file-name="src/main.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/main.rs}}
```

</Listing>

`pub mod garden;` line compiler ko batati hai ke *src/garden.rs* mein jo code hai use include kare, jo ye hai:

<Listing file-name="src/garden.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden.rs}}
```

</Listing>

Yahan, `pub mod vegetables;` ka matlab hai ke *src/garden/vegetables.rs* mein mojood code bhi include kiya jaye. Woh code ye hai:

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden/vegetables.rs}}
```

Ab aaiye in rules ki details mein jate hain aur inhein action mein demonstrate karte hain!

### Modules Mein Related Code Ko Group Karna

*Modules* humein ek crate ke andar code ko readability aur easy reuse ke liye organize karne dete hain. Modules humein items ki *privacy* control karne ki bhi ijazat dete hain, kyun ke module ke andar ka code by default private hota hai. Private items internal implementation details hote hain jo bahar se use ke liye available nahi hote. Hum modules aur unke andar mojood items ko public banane ka intekhab kar sakte hain, jo unhein expose karta hai taa-ke external code unhein use aur un par depend kar sake.

Misal ke taur par, aaiye ek library crate likhte hain jo ek restaurant ki functionality provide karti hai. Hum functions ke signatures define karenge lekin unki bodies khaali chhor denge taa-ke restaurant ki implementation ke bajaye code ki organization par focus kiya ja sake.

Restaurant industry mein, restaurant ke kuch parts ko front of house aur doosre parts ko back of house kaha jata hai. *Front of house* woh jagah hai jahan customers hote hain; is mein woh jagah shamil hai jahan hosts customers ko seat karte hain, servers orders aur payment lete hain, aur bartenders drinks banate hain. *Back of house* woh jagah hai jahan chefs aur cooks kitchen mein kaam karte hain, dishwashers safai karte hain, aur managers administrative kaam karte hain.

Apne crate ko is tarah structure karne ke liye, hum iske functions ko nested modules mein organize kar sakte hain. `cargo new
restaurant --lib` run karke `restaurant` naam ki ek nayi library create karein. Phir Listing 7-1 ka code *src/lib.rs* mein enter karein taa-ke kuch modules aur function signatures define kiye ja saken; ye front of house section hai.

<Listing number="7-1" file-name="src/lib.rs" caption="Ek `front_of_house` module jisme mazeed modules hain aur un modules ke andar functions hain">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-01/src/lib.rs}}
```

</Listing>

Hum `mod` keyword ke baad module ka name likh kar module define karte hain (is case mein `front_of_house`). Phir module ki body curly brackets ke andar hoti hai. Modules ke andar hum doosre modules bhi rakh sakte hain, jaisa ke is case mein `hosting` aur `serving` modules ke saath hai. Modules mein doosre items ki definitions bhi ho sakti hain, jaise structs, enums, constants, traits, aur, jaisa ke Listing 7-1 mein hai, functions.

Modules ko use karke hum related definitions ko ek saath group kar sakte hain aur ye name kar sakte hain ke woh kis wajah se related hain. Is code ko use karne wale programmers tamam definitions ko parhne ke bajaye groups ki bunyaad par code ko navigate kar sakte hain, jis se unke liye relevant definitions dhoondhna aasaan ho jata hai. Is code mein new functionality add karne wale programmers ko bhi pata hoga ke program ko organized rakhne ke liye code ko kahan place karna hai.

Pehle hum ne mention kiya tha ke *src/main.rs* aur *src/lib.rs* ko *crate
roots* kaha jata hai. Unka ye name hone ki wajah ye hai ke in dono mein se kisi bhi file ka content crate ki module structure ke root par `crate` naam ka ek module banata hai, jise *module tree* kaha jata hai.

Listing 7-2 Listing 7-1 ki structure ke liye module tree dikhati hai.

<Listing number="7-2" caption="Listing 7-1 ke code ka module tree">

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

</Listing>

Ye tree dikhata hai ke kuch modules doosre modules ke andar nested hain; misal ke taur par, `hosting`, `front_of_house` ke andar nested hai. Tree ye bhi dikhata hai ke kuch modules *siblings* hain, yani woh ek hi module mein define kiye gaye hain; `hosting` aur `serving` `front_of_house` ke andar define kiye gaye siblings hain. Agar module A, module B ke andar contained ho, to hum kehte hain ke module A, module B ka *child* hai aur module B, module A ka *parent* hai. Notice karein ke poora module tree implicit module `crate` ke neeche rooted hai.

Module tree shayad aapko apne computer ke filesystem ke directory tree ki yaad dilaye; ye comparison bilkul munasib hai! Bilkul filesystem ki directories ki tarah, aap modules ko apne code ko organize karne ke liye use karte hain. Aur bilkul directory mein files ki tarah, humein apne modules ko dhoondhne ke liye ek tareeqe ki zaroorat hoti hai.
