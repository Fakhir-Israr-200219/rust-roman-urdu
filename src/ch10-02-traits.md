<!-- Old headings. Do not remove or links may break. -->

<a id="traits-defining-shared-behavior"></a>

## Traits Ke Zariye Shared Behavior Define Karna

Ek *trait* woh functionality define karta hai jo kisi particular type ke paas hoti hai aur jo woh doosre types ke saath share kar sakta hai. Hum traits ko use karke shared behavior ko ek abstract tareeqe se define kar sakte hain. Hum *trait bounds* ko use karke specify kar sakte hain ke ek generic type koi bhi aisa type ho sakta hai jis ke paas particular behavior ho.

> Note: Traits un features se milte-julte hain jinhein doosri languages mein aksar *interfaces* kaha jata hai, lekin in dono mein kuch differences hain.

### Trait Define Karna

Kisi type ka behavior un methods par mushtamil hota hai jinhein hum us type par call kar sakte hain. Mukhtalif types ek hi behavior share karte hain agar hum un tamam types par same methods call kar sakte hon. Trait definitions method signatures ko ek jagah group karne ka tareeqa hain taa-ke kisi maqsad ko hasil karne ke liye zaroori behaviors ka ek set define kiya ja sake.

Misal ke taur par, maan lein ke hamare paas multiple structs hain jo mukhtalif qisam aur miktar ka text hold karte hain: ek `NewsArticle` struct jo kisi particular location par file ki gayi news story ko hold karta hai aur ek `SocialPost` jo zyada se zyada 280 characters rakh sakta hai, saath hi metadata bhi hota hai jo batata hai ke woh ek nayi post, repost, ya kisi doosri post ka reply hai.

Hum ek media aggregator library crate banana chahte hain jis ka naam `aggregator` ho, jo aise data ke summaries display kar sake jo `NewsArticle` ya `SocialPost` instance mein store ho sakta hai. Aisa karne ke liye, humein har type se ek summary chahiye hogi, aur hum instance par `summarize` method call karke us summary ka request karenge. Listing 10-12 ek public `Summary` trait ki definition dikhati hai jo is behavior ko express karti hai.

<Listing number="10-12" file-name="src/lib.rs" caption="A `Summary` trait that consists of the behavior provided by a `summarize` method">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-12/src/lib.rs}}
```

</Listing>

Yahan, hum `trait` keyword aur phir trait ka naam use karke ek trait declare karte hain, jo is case mein `Summary` hai. Hum trait ko `pub` bhi declare karte hain taa-ke is crate par depend karne wale crates bhi is trait ko use kar saken, jaisa ke hum kuch examples mein dekhenge. Curly brackets ke andar, hum method signatures declare karte hain jo un types ke behaviors ko describe karti hain jo is trait ko implement karte hain; is case mein `fn summarize(&self) -> String` hai.

Method signature ke baad, curly brackets ke andar implementation provide karne ke bajaye, hum semicolon use karte hain. Is trait ko implement karne wale har type ko method ki body ke liye apna custom behavior provide karna hoga. Compiler enforce karega ke jis bhi type ke paas `Summary` trait hoga, us mein `summarize` method bilkul isi signature ke saath defined ho.

Ek trait ki body mein multiple methods ho sakte hain: Method signatures ko har line par ek ek karke list kiya jata hai, aur har line semicolon par end hoti hai.

### Kisi Type Par Trait Implement Karna

Ab jab hum ne `Summary` trait ke methods ki required signatures define kar li hain, to hum isay apne media aggregator ke types par implement kar sakte hain. Listing 10-13 mein `NewsArticle` struct par `Summary` trait ki ek implementation dikhayi gayi hai jo `summarize` ki return value create karne ke liye headline, author, aur location ko use karti hai. `SocialPost` struct ke liye, hum `summarize` ko username ke baad post ka poora text define karte hain, is assumption ke saath ke post ka content pehle hi 280 characters tak limited hai.

<Listing number="10-13" file-name="src/lib.rs" caption="Implementing the `Summary` trait on the `NewsArticle` and `SocialPost` types">

```rust,noplayground id="q7v2kx"
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-13/src/lib.rs:here}}
```

</Listing>

Kisi type par trait implement karna regular methods implement karne jaisa hi hai. Farq ye hai ke `impl` ke baad hum us trait ka naam likhte hain jise hum implement karna chahte hain, phir `for` keyword use karte hain, aur phir us type ka naam specify karte hain jis par hum trait implement karna chahte hain. `impl` block ke andar, hum woh method signatures rakhte hain jo trait definition mein define ki gayi hain. Har signature ke baad semicolon add karne ke bajaye, hum curly brackets use karte hain aur method body ko us specific behavior se fill karte hain jo hum chahte hain ke particular type ke liye trait ke methods ka ho.

Ab jab library ne `NewsArticle` aur `SocialPost` par `Summary` trait implement kar diya hai, to crate ke users `NewsArticle` aur `SocialPost` ke instances par trait methods ko bilkul usi tarah call kar sakte hain jaise hum regular methods call karte hain. Sirf ek farq ye hai ke user ko types ke saath trait ko bhi scope mein lana hoga. Yahan ek example hai ke ek binary crate hamari `aggregator` library crate ko kaise use kar sakta hai:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-01-calling-trait-method/src/main.rs}}
```

Ye code `1 new post: horse_ebooks: of course, as you probably already know, people` print karta hai.

Doosre crates jo `aggregator` crate par depend karte hain, woh bhi `Summary` trait ko scope mein la sakte hain taa-ke apne types par `Summary` implement kar saken. Yahan ek restriction note karna zaroori hai: Hum kisi type par trait sirf us waqt implement kar sakte hain jab trait ya type, ya dono, hamare crate ke local hon. Misal ke taur par, hum standard library ke traits jaise `Display` ko apne custom type `SocialPost` par apne `aggregator` crate ki functionality ke taur par implement kar sakte hain, kyun ke type `SocialPost` hamare `aggregator` crate ke liye local hai. Hum apne `aggregator` crate mein `Vec<T>` par `Summary` bhi implement kar sakte hain, kyun ke trait `Summary` hamare `aggregator` crate ke liye local hai.

Lekin hum external traits ko external types par implement nahi kar sakte. Misal ke taur par, hum apne `aggregator` crate ke andar `Vec<T>` par `Display` trait implement nahi kar sakte, kyun ke `Display` aur `Vec<T>` dono standard library mein defined hain aur hamare `aggregator` crate ke liye local nahi hain. Ye restriction ek property ka hissa hai jise *coherence*, aur zyada specifically *orphan rule*, kaha jata hai; iska naam is liye rakha gaya hai kyun ke parent type maujood nahi hota. Ye rule ensure karta hai ke doosre logon ka code aapke code ko break na kar sake aur aapka code bhi unke code ko break na kare. Is rule ke baghair, do crates ek hi type ke liye same trait implement kar sakte the, aur Rust ko pata na hota ke kaunsi implementation use karni hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="default-implementations"></a>

### Default Implementations Use Karna

Kabhi kabhi trait ke kuch ya tamam methods ke liye default behavior rakhna useful hota hai, bajaye is ke ke har type par tamam methods ki implementations dena zaroori ho. Phir, jab hum kisi particular type par trait implement karte hain, to hum har method ke default behavior ko rakh sakte hain ya usay override kar sakte hain.

Listing 10-14 mein, hum `Summary` trait ke `summarize` method ke liye sirf method signature define karne ke bajaye ek default string specify karte hain, jaisa ke hum ne Listing 10-12 mein kiya tha.

<Listing number="10-14" file-name="src/lib.rs" caption="Defining a `Summary` trait with a default implementation of the `summarize` method">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-14/src/lib.rs:here}}
```

</Listing>

`NewsArticle` ke instances ko summarize karne ke liye default implementation use karne ke liye, hum `impl Summary for NewsArticle {}` ke saath ek empty `impl` block specify karte hain.

Agarche ab hum `NewsArticle` par directly `summarize` method define nahi kar rahe, lekin hum ne ek default implementation provide ki hai aur specify kiya hai ke `NewsArticle` `Summary` trait implement karta hai. Is ke result mein, hum ab bhi `NewsArticle` ke instance par `summarize` method call kar sakte hain, jaise:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-02-calling-default-impl/src/main.rs:here}}
```

Ye code `New article available! (Read more...)` print karta hai.

Default implementation create karne ke liye humein Listing 10-13 mein `SocialPost` par `Summary` ki implementation mein koi change karne ki zaroorat nahi hai. Is ki wajah ye hai ke default implementation ko override karne ki syntax bilkul wohi hai jo aise trait method ko implement karne ki syntax hai jis ki koi default implementation nahi hoti.

Default implementations usi trait ke doosre methods ko call kar sakti hain, chahe un doosre methods ki koi default implementation na ho. Is tarah, ek trait bohat si useful functionality provide kar sakta hai aur implementors se sirf us ka ek chhota hissa specify karne ka taqaza kar sakta hai. Misal ke taur par, hum `Summary` trait ko is tarah define kar sakte hain ke is mein `summarize_author` method ho jis ki implementation required ho, aur phir `summarize` method define kar sakte hain jis ki default implementation `summarize_author` method ko call karti ho:

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:here}}
```

`Summary` ke is version ko use karne ke liye, humein sirf `summarize_author` define karna hoga jab hum kisi type par trait implement karein:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:impl}}
```

`summary_author` define karne ke baad, hum `SocialPost` struct ke instances par `summarize` call kar sakte hain, aur `summarize` ki default implementation hamari provide ki hui `summarize_author` definition ko call karegi. Kyun ke hum ne `summarize_author` implement kiya hai, `Summary` trait ne humein `summarize` method ka behavior provide kar diya hai, aur humein koi aur code likhne ki zaroorat nahi padi. Ye is tarah nazar aata hai:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/main.rs:here}}
```

Ye code `1 new post: (Read more from @horse_ebooks...)` print karta hai.

Note karein ke kisi method ki overriding implementation ke andar usi method ki default implementation ko call karna possible nahi hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="traits-as-parameters"></a>

### Traits Ko Parameters Ke Taur Par Use Karna

Ab jab aap jaante hain ke traits ko kaise define aur implement kiya jata hai, to hum explore kar sakte hain ke traits ko use karke aise functions kaise define kiye jate hain jo bohat se different types ko accept kar saken. Hum Listing 10-13 mein `NewsArticle` aur `SocialPost` types par implement kiye gaye `Summary` trait ko use karke ek `notify` function define karenge jo apne `item` parameter par `summarize` method call karta hai. Ye parameter kisi aise type ka hai jo `Summary` trait implement karta hai. Aisa karne ke liye, hum `impl Trait` syntax use karte hain, jaise:

```rust,ignore id="r8m2kx"
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-04-traits-as-parameters/src/lib.rs:here}}
```

`item` parameter ke liye kisi concrete type ko specify karne ke bajaye, hum `impl` keyword aur trait ka naam specify karte hain. Ye parameter kisi bhi aise type ko accept karta hai jo specified trait implement karta ho. `notify` ki body mein, hum `item` par woh tamam methods call kar sakte hain jo `Summary` trait se aate hain, jaise `summarize`. Hum `notify` ko call karke us mein `NewsArticle` ya `SocialPost` ka koi bhi instance pass kar sakte hain. Aisa code jo function ko kisi doosre type, jaise `String` ya `i32`, ke saath call karega, compile nahi hoga, kyun ke ye types `Summary` implement nahi karte.

<!-- Old headings. Do not remove or links may break. -->

<a id="fixing-the-largest-function-with-trait-bounds"></a>

#### Trait Bound Syntax

`impl Trait` syntax seedhe cases ke liye kaam karti hai, lekin asal mein ye ek zyada lambi form ke liye syntax sugar hai jise *trait bound* kaha jata hai; ye is tarah nazar aati hai:

```rust,ignore
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

Ye lambi form pichle section ke example ke equivalent hai, lekin zyada verbose hai. Hum generic type parameter ki declaration ke saath, colon ke baad aur angle brackets ke andar trait bounds place karte hain.

`impl Trait` syntax convenient hai aur simple cases mein zyada concise code provide karti hai, jabke full trait bound syntax doosre cases mein zyada complexity express kar sakti hai. Misal ke taur par, hamare paas do parameters ho sakte hain jo `Summary` implement karte hon. `impl Trait` syntax ke saath aisa karna is tarah nazar aata hai:

```rust,ignore
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

`impl Trait` use karna us waqt munasib hai jab hum chahte hon ke ye function `item1` aur `item2` ko different types rakhne ki ijazat de, jab tak dono types `Summary` implement karte hon. Lekin agar hum dono parameters ko same type ka rakhna force karna chahte hain, to humein trait bound use karna hoga, jaise:

```rust,ignore
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

`item1` aur `item2` parameters ke type ke taur par specify kiya gaya generic type `T` function ko is tarah constrain karta hai ke `item1` aur `item2` ke arguments ke taur par pass ki gayi values ka concrete type same hona zaroori hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="specifying-multiple-trait-bounds-with-the--syntax"></a>


#### `+` Syntax Ke Saath Multiple Trait Bounds

Hum ek se zyada trait bounds bhi specify kar sakte hain. Maan lein ke hum chahte hain ke `notify`, `item` par `summarize` ke saath display formatting bhi use kare: Hum `notify` ki definition mein specify karte hain ke `item` ko `Display` aur `Summary` dono implement karna hoga. Hum `+` syntax ko use karke aisa kar sakte hain:

```rust,ignore
pub fn notify(item: &(impl Summary + Display)) {
```

`+` syntax generic types par trait bounds ke saath bhi valid hai:

```rust,ignore
pub fn notify<T: Summary + Display>(item: &T) {
```

In dono trait bounds ko specify karne ke baad, `notify` ki body `summarize` ko call kar sakti hai aur `item` ko format karne ke liye `{}` use kar sakti hai.

#### `where` Clauses Ke Saath Zyada Clear Trait Bounds

Bohat zyada trait bounds use karne ke apne nuqsanat hain. Har generic ka apna trait bounds hota hai, is liye multiple generic type parameters wale functions mein function ke naam aur parameter list ke darmiyan bohat saari trait bound information aa sakti hai, jis se function signature ko parhna mushkil ho jata hai. Isi wajah se, Rust mein function signature ke baad `where` clause ke andar trait bounds specify karne ke liye alternate syntax maujood hai. Is liye, is tarah likhne ke bajaye:

```rust,ignore id="q1m7sa"
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

hum `where` clause use kar sakte hain, jaise:

```rust,ignore id="j8c4vx"
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-07-where-clause/src/lib.rs:here}}
```

Is function ka signature kam cluttered hai: Function name, parameter list, aur return type ek doosre ke qareeb hain, bilkul us function ki tarah jis mein bohat zyada trait bounds na hon.

### Aise Types Return Karna Jo Traits Implement Karte Hon

Hum return position mein bhi `impl Trait` syntax ko use karke kisi aise type ki value return kar sakte hain jo kisi trait ko implement karta ho, jaisa ke yahan dikhaya gaya hai:

```rust,ignore id="m3k7qp"
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-05-returning-impl-trait/src/lib.rs:here}}
```

Return type ke liye `impl Summary` use karke, hum specify karte hain ke `returns_summarizable` function kisi aise type ko return karta hai jo `Summary` trait implement karta ho, baghair concrete type ka naam bataye. Is case mein, `returns_summarizable` ek `SocialPost` return karta hai, lekin is function ko call karne wale code ko ye jaanne ki zaroorat nahi hoti.

Return type ko sirf us trait ke zariye specify karne ki ability jise woh implement karta hai, khaas taur par closures aur iterators ke context mein useful hai, jinhein hum Chapter 13 mein cover karenge. Closures aur iterators aise types create karte hain jinhein sirf compiler jaanta hai, ya aise types jinhein specify karna bohat lamba hota hai. `impl Trait` syntax aapko ye concise tareeqe se specify karne deti hai ke ek function kisi aise type ko return karta hai jo `Iterator` trait implement karta ho, baghair bohat lambe type ko poora likhne ki zaroorat ke.

Lekin aap `impl Trait` sirf us waqt use kar sakte hain jab aap ek hi type return kar rahe hon. Misal ke taur par, ye code jo `impl Summary` ko return type ke taur par specify karte hue ya to `NewsArticle` ya `SocialPost` return karta hai, kaam nahi karega:

```rust,ignore,does_not_compile id="t9x2nc"
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-06-impl-trait-returns-one-type/src/lib.rs:here}}
```

`NewsArticle` ya `SocialPost` mein se kisi ek ko return karna allowed nahi hai, kyun ke `impl Trait` syntax ko compiler mein implement karne ke tareeqe se related restrictions hain. Hum Chapter 18 ke [“Using Trait Objects to Abstract over Shared Behavior”][trait-objects]<!-- ignore --> section mein cover karenge ke is behavior wala function kaise likha jata hai.

### Trait Bounds Use Karke Methods Ko Conditionally Implement Karna

Generic type parameters use karne wale `impl` block ke saath trait bound use karke, hum un types ke liye conditionally methods implement kar sakte hain jo specified traits implement karte hon. Misal ke taur par, Listing 10-15 mein `Pair<T>` type hamesha `new` function implement karta hai jo `Pair<T>` ka ek naya instance return karta hai (Chapter 5 ke [“Method Syntax”][methods]<!-- ignore --> section se yaad karein ke `Self`, `impl` block ke type ke liye ek type alias hai, jo is case mein `Pair<T>` hai). Lekin agley `impl` block mein, `Pair<T>` sirf us waqt `cmp_display` method implement karta hai jab is ka inner type `T`, `PartialOrd` trait implement karta ho jo *comparison* ko enable karta hai, aur `Display` trait implement karta ho jo printing ko enable karta hai.

<Listing number="10-15" file-name="src/lib.rs" caption="Conditionally implementing methods on a generic type depending on trait bounds">

```rust,noplayground id="4qk9tp"
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-15/src/lib.rs}}
```

</Listing>

Hum kisi aise type ke liye bhi conditionally ek trait implement kar sakte hain jo koi doosra trait implement karta ho. Aisi trait implementations jo har us type par hoti hain jo trait bounds ko satisfy karta ho, *blanket implementations* kehlati hain aur Rust standard library mein extensively use hoti hain. Misal ke taur par, standard library `ToString` trait ko har us type par implement karti hai jo `Display` trait implement karta hai. Standard library ka `impl` block is code jaisa nazar aata hai:

```rust,ignore id="p3v7yx"
impl<T: Display> ToString for T {
    // --snip--
}
```

Standard library ki is blanket implementation ki wajah se, hum kisi bhi aise type par `ToString` trait ka defined `to_string` method call kar sakte hain jo `Display` trait implement karta ho. Misal ke taur par, hum integers ko unki corresponding `String` values mein is tarah convert kar sakte hain kyun ke integers `Display` implement karte hain:

```rust id="w6nq5e"
let s = 3.to_string();
```

Blanket implementations trait ki documentation mein “Implementors” section ke andar nazar aati hain.

Traits aur trait bounds humein aisa code likhne dete hain jo duplication ko reduce karne ke liye generic type parameters use karta hai, aur saath hi compiler ko ye bhi specify karta hai ke hum chahte hain generic type ka koi particular behavior ho. Phir compiler trait bound ki information ko use karke check kar sakta hai ke hamare code ke saath use kiye gaye tamam concrete types correct behavior provide karte hain. Dynamically typed languages mein, agar hum kisi aise type par method call karein jo us method ko define nahi karta, to humein runtime par error milega. Lekin Rust in errors ko compile time par le aata hai, taa-ke humein problems ko us waqt fix karna pade jab hamara code run hone ke qabil bhi nahi hua hota. Is ke ilawa, humein runtime par behavior check karne wala code likhne ki zaroorat nahi hoti, kyun ke hum pehle hi compile time par check kar chuke hote hain. Is se performance improve hoti hai aur saath hi generics ki flexibility bhi compromise nahi hoti.

[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[methods]: ch05-03-method-syntax.html#method-syntax
