## Improving Our I/O Project

Iterators ke baare mein is naye knowledge ke saath, hum Chapter 12 ke I/O project ko behtar bana sakte hain, iterators ko use karke code ki un jagahon ko zyada clear aur concise bana sakte hain. Aaiye dekhein ke iterators hamari `Config::build` function aur `search` function ki implementation ko kis tarah improve kar sakte hain.

### Removing a `clone` Using an Iterator

Listing 12-6 mein, humne aisa code add kiya tha jo `String` values ki ek slice leta tha aur slice mein indexing karke aur values ko clone karke `Config` struct ka ek instance create karta tha, jis se `Config` struct un values ki ownership le sakta tha. Listing 13-17 mein, humne `Config::build` function ki implementation ko dobara reproduce kiya hai jaisi woh Listing 12-23 mein thi.

<Listing number="13-17" file-name="src/main.rs" caption="Reproduction of the `Config::build` function from Listing 12-23">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-23-reproduced/src/main.rs:ch13}}
```

</Listing>

Us waqt, humne kaha tha ke inefficient `clone` calls ke baare mein fikr na karein kyun ke hum future mein unhein remove kar denge. Ab woh waqt aa gaya hai!

Yahan humein `clone` ki zarurat is liye thi kyun ke parameter `args` mein `String` elements wali ek slice hai, lekin `build` function `args` ki ownership nahi rakhta. `Config` instance ki ownership return karne ke liye, humein `Config` ke `query` aur `file_path` fields se values ko clone karna pada taake `Config` instance apni values ki ownership rakh sake.

Iterators ke baare mein apne naye knowledge ke saath, hum `build` function ko change karke slice ko borrow karne ke bajaye argument ke taur par ek iterator ki ownership lene wala bana sakte hain. Hum slice ki length check karne aur specific locations par indexing karne wale code ke bajaye iterator ki functionality use karenge. Is se yeh zyada clear hoga ke `Config::build` function kya kar raha hai, kyun ke iterator values tak access provide karega.

Jab `Config::build` iterator ki ownership le lega aur borrowing wali indexing operations ko use karna band kar dega, to hum `String` values ko `clone` call karke nayi allocation banane ke bajaye iterator se `Config` mein move kar sakte hain.

#### Using the Returned Iterator Directly

Apne I/O project ki *src/main.rs* file open karein, jo kuch is tarah nazar aani chahiye:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-24-reproduced/src/main.rs:ch13}}
```

Sab se pehle, hum `main` function ke us start ko jo humne Listing 12-24 mein use kiya tha, Listing 13-18 ke code mein change karenge, jo is baar ek iterator use karta hai. Jab tak hum `Config::build` ko bhi update nahi karte, yeh compile nahi hoga.

<Listing number="13-18" file-name="src/main.rs" caption="Passing the return value of `env::args` to `Config::build`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-18/src/main.rs:here}}
```

</Listing>

`env::args` function ek iterator return karta hai! Iterator ki values ko ek vector mein collect karke phir `Config::build` ko slice pass karne ke bajaye, ab hum `env::args` se return hone wale iterator ki ownership directly `Config::build` ko pass kar rahe hain.

Next, humein `Config::build` ki definition ko update karna hoga. Aaiye `Config::build` ke signature ko Listing 13-19 jaisa change karte hain. Yeh abhi bhi compile nahi hoga, kyun ke humein function body ko update karna hoga.

<Listing number="13-19" file-name="src/main.rs" caption="Updating the signature of `Config::build` to expect an iterator">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-19/src/main.rs:here}}
```

</Listing>

`env::args` function ki standard library documentation dikhati hai ke is se return hone wale iterator ka type `std::env::Args` hai, aur yeh type `Iterator` trait ko implement karta hai aur `String` values return karta hai.

Humne `Config::build` function ke signature ko update kiya hai taake parameter `args` ke paas `&[String]` ke bajaye `impl Iterator<Item = String>` trait bounds wala generic type ho. `impl Trait` syntax ka yeh use, jise humne Chapter 10 ke [“Using Traits as Parameters”][impl-trait]<!-- ignore --> section mein discuss kiya tha, iska matlab hai ke `args` koi bhi aisa type ho sakta hai jo `Iterator` trait ko implement karta ho aur `String` items return karta ho.

Kyun ke hum `args` ki ownership le rahe hain aur is par iterate karke `args` ko mutate karenge, is liye hum `args` parameter ki specification mein `mut` keyword add karke isay mutable bana sakte hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-iterator-trait-methods-instead-of-indexing"></a>

#### Using `Iterator` Trait Methods

Ab hum `Config::build` ke body ko fix karenge. Kyun ke `args` `Iterator` trait ko implement karta hai, hum jaante hain ke hum is par `next` method call kar sakte hain! Listing 13-20, Listing 12-23 ke code ko `next` method use karne ke liye update karti hai.

<Listing number="13-20" file-name="src/main.rs" caption="Changing the body of `Config::build` to use iterator methods">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-20/src/main.rs:here}}
```

</Listing>

Yaad rakhein ke `env::args` ke return value mein pehli value program ka naam hoti hai. Hum ise ignore karna chahte hain aur next value tak pohanchna chahte hain, is liye sab se pehle hum `next` call karte hain aur uske return value ke saath kuch nahi karte. Phir, hum `next` call karke woh value lete hain jo hum `Config` ke `query` field mein rakhna chahte hain. Agar `next` `Some` return karta hai, to hum value ko extract karne ke liye `match` use karte hain. Agar yeh `None` return karta hai, to iska matlab hai ke kafi arguments provide nahi kiye gaye, aur hum `Err` value ke saath jaldi return kar dete hain. Hum `file_path` value ke liye bhi yahi kaam karte hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="making-code-clearer-with-iterator-adapters"></a>

### Clarifying Code with Iterator Adapters

Hum apne I/O project ke `search` function mein bhi iterators ka faida utha sakte hain, jo yahan Listing 13-21 mein usi tarah reproduce kiya gaya hai jaise yeh Listing 12-19 mein tha.

<Listing number="13-21" file-name="src/lib.rs" caption="The implementation of the `search` function from Listing 12-19">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:ch13}}
```

</Listing>

Hum iterator adapter methods ko use karke is code ko zyada concise tareeqe se likh sakte hain. Aisa karne se hum ek mutable intermediate `results` vector ko bhi avoid kar sakte hain. Functional programming style code ko zyada clear banane ke liye mutable state ki miktar ko kam rakhna prefer karta hai. Mutable state ko remove karne se future mein searching ko parallel mein karne ki enhancement mumkin ho sakti hai, kyun ke humein `results` vector tak concurrent access ko manage nahi karna padega. Listing 13-22 is change ko dikhati hai.

<Listing number="13-22" file-name="src/lib.rs" caption="Using iterator adapter methods in the implementation of the `search` function">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-22/src/lib.rs:here}}
```

</Listing>

Yaad rakhein ke `search` function ka maqsad `contents` ki un tamam lines ko return karna hai jin mein `query` maujood ho. Listing 13-16 ke `filter` example ki tarah, yeh code sirf un lines ko rakhne ke liye `filter` adapter use karta hai jin ke liye `line.contains(query)` `true` return karta hai. Phir hum matching lines ko `collect` ke zariye ek doosre vector mein collect kar lete hain. Kaafi simple! Aap `search_case_insensitive` function mein bhi iterator methods use karne ke liye yahi change kar sakte hain.

Mazeed improvement ke liye, `collect` ki call ko remove karke aur return type ko `impl Iterator<Item = &'a str>` mein change karke `search` function se ek iterator return karein, taake function khud ek iterator adapter ban jaye. Note karein ke aapko tests bhi update karne honge! Is change se pehle aur baad mein apne `minigrep` tool ko use karke ek bari file mein search karein aur behavior mein difference observe karein. Is change se pehle, program tab tak koi result print nahi karega jab tak woh tamam results ko collect na kar le, lekin change ke baad, har matching line milte hi results print ho jayenge kyun ke `run` function mein `for` loop iterator ki laziness ka faida utha sakta hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="choosing-between-loops-or-iterators"></a>

### Choosing Between Loops and Iterators

Agla logical sawal yeh hai ke aapko apne code mein kaunsa style choose karna chahiye aur kyun: Listing 13-21 mein original implementation ya Listing 13-22 mein iterators use karne wala version (yeh maan kar ke hum results return karne se pehle tamam results ko collect kar rahe hain, na ke iterator return kar rahe hain). Zyada tar Rust programmers iterator style ko prefer karte hain. Shuru mein iski aadat dalna thora mushkil hota hai, lekin jab aapko mukhtalif iterator adapters aur unke kaam karne ka tareeqa samajh aa jata hai, to iterators ko samajhna zyada aasaan ho sakta hai. Loop ke mukhtalif parts ke saath fiddle karne aur naye vectors banane ke bajaye, code loop ke high-level objective par focus karta hai. Yeh kuch commonplace code ko abstract away kar deta hai, jis se un concepts ko dekhna aasaan ho jata hai jo is code ke liye unique hain, jaise woh filtering condition jise iterator ke har element ko pass karna zaroori hai.

Lekin kya dono implementations waqai equivalent hain? Intuitive assumption yeh ho sakti hai ke lower-level loop zyada fast hoga. Aaiye performance ke baare mein baat karte hain.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
