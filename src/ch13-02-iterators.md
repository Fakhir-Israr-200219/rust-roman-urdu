## Processing a Series of Items with Iterators

Iterator pattern aapko items ki ek sequence par bari bari kuch task perform karne ki ability deta hai. Ek iterator har item par iterate karne ki logic aur yeh determine karne ke liye responsible hota hai ke sequence kab finish ho gayi hai. Jab aap iterators use karte hain, to aapko yeh logic khud se dobara implement nahi karni padti.

Rust mein, iterators *lazy* hote hain, yani jab tak aap aise methods call nahi karte jo iterator ko consume karke use up kar dein, tab tak unka koi effect nahi hota. Misal ke taur par, Listing 13-10 mein code `Vec<T>` par defined `iter` method ko call karke vector `v1` ke items par ek iterator create karta hai. Yeh code apne aap mein koi useful kaam nahi karta.

<Listing number="13-10" file-name="src/main.rs" caption="Creating an iterator">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-10/src/main.rs:here}}
```

</Listing>

Iterator ko `v1_iter` variable mein store kiya jata hai. Ek baar iterator create karne ke baad, hum isay mukhtalif tareeqon se use kar sakte hain. Listing 3-5 mein, humne ek array par `for` loop use karke uske har item par kuch code execute kiya tha. Under the hood, is ne implicitly ek iterator create kiya aur phir usay consume kiya, lekin ab tak humne is baat ko detail mein discuss nahi kiya tha ke yeh exactly kaise kaam karta hai.

Listing 13-11 ke example mein, hum iterator ki creation ko `for` loop mein iterator ke use se separate karte hain. Jab `v1_iter` mein maujood iterator ko use karke `for` loop call kiya jata hai, to iterator ka har element loop ki ek iteration mein use hota hai, jo har value ko print karta hai.

<Listing number="13-11" file-name="src/main.rs" caption="Using an iterator in a `for` loop">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-11/src/main.rs:here}}
```

</Listing>

Un languages mein jin ki standard libraries iterators provide nahi karti, aap shayad isi functionality ko is tarah likhte ke ek variable ko index `0` se start karte, us variable ko vector mein index ke taur par use karke ek value hasil karte, aur phir loop mein variable ki value ko increment karte rehte jab tak woh vector mein items ki total number tak na pohanch jati.

Iterators yeh sari logic aapke liye handle karte hain, jis se repetitive code kam ho jata hai jismein aap se ghalti hone ka imkaan ho sakta hai. Iterators aapko yeh flexibility dete hain ke aap isi logic ko bohat mukhtalif qisam ki sequences ke saath use kar sakein, sirf un data structures ke saath nahi jin mein indexing ki ja sakti ho, jaise vectors. Aaiye examine karte hain ke iterators yeh kaise karte hain.

### The `Iterator` Trait and the `next` Method

Tamam iterators ek `Iterator` naam ke trait ko implement karte hain jo standard library mein defined hai. Is trait ki definition kuch is tarah hai:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
```

Note karein ke yeh definition kuch naya syntax use karti hai: `type Item` aur `Self::Item`, jo is trait ke saath ek associated type define kar rahe hain. Hum Chapter 20 mein associated types ke baare mein detail se baat karenge. Filhaal, aapko sirf itna samajhna hai ke yeh code kehta hai ke `Iterator` trait ko implement karne ke liye aapko ek `Item` type bhi define karna hoga, aur yeh `Item` type `next` method ke return type mein use hota hai. Doosre alfaaz mein, `Item` type woh type hoga jo iterator se return hota hai.

`Iterator` trait implementors se sirf ek method define karne ka taqaza karta hai: `next` method, jo ek waqt mein iterator ka ek item return karta hai, `Some` mein wrapped hota hai, aur jab iteration khatam ho jati hai to `None` return karta hai.

Hum iterators par directly `next` method call kar sakte hain; Listing 13-12 vector se create kiye gaye iterator par `next` ko baar baar call karne se return hone wali values ko demonstrate karti hai.

<Listing number="13-12" file-name="src/lib.rs" caption="Calling the `next` method on an iterator">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-12/src/lib.rs:here}}
```

</Listing>

Note karein ke humein `v1_iter` ko mutable banana pada: Iterator par `next` method call karna us internal state ko change karta hai jise iterator sequence mein apni current position ko track karne ke liye use karta hai. Doosre alfaaz mein, yeh code iterator ko *consume* karta hai, ya use up karta hai. `next` ki har call iterator se ek item ko consume kar leti hai. Jab humne `for` loop use kiya tha to humein `v1_iter` ko mutable banane ki zarurat nahi padi thi, kyun ke loop ne `v1_iter` ki ownership le li aur background mein usay mutable bana diya.

Yeh bhi note karein ke `next` ki calls se jo values humein milti hain woh vector mein maujood values ke immutable references hain. `iter` method immutable references par ek iterator produce karta hai. Agar hum aisa iterator create karna chahte hain jo `v1` ki ownership le aur owned values return kare, to hum `iter` ke bajaye `into_iter` call kar sakte hain. Isi tarah, agar hum mutable references par iterate karna chahte hain, to hum `iter` ke bajaye `iter_mut` call kar sakte hain.

### Methods That Consume the Iterator

`Iterator` trait mein standard library ki taraf se default implementations ke saath bohat se mukhtalif methods provide kiye gaye hain; aap in methods ke baare mein `Iterator` trait ki standard library API documentation mein dekh sakte hain. In mein se kuch methods apni definition mein `next` method ko call karte hain, isi liye jab aap `Iterator` trait ko implement karte hain to aapko `next` method implement karna zaroori hota hai.

Jo methods `next` ko call karte hain unhein *consuming adapters* kaha jata hai, kyun ke unhein call karna iterator ko use up kar deta hai. Ek example `sum` method hai, jo iterator ki ownership leta hai aur baar baar `next` call karke items par iterate karta hai, aur is tarah iterator ko consume karta hai. Iterate karte hue, yeh har item ko ek running total mein add karta hai aur jab iteration complete ho jati hai to total return karta hai. Listing 13-13 mein ek test hai jo `sum` method ke ek use ko illustrate karta hai.

<Listing number="13-13" file-name="src/lib.rs" caption="Calling the `sum` method to get the total of all items in the iterator">

```rust,noplayground id="f7g9w2"
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-13/src/lib.rs:here}}
```

</Listing>

Humein `sum` call karne ke baad `v1_iter` ko use karne ki ijazat nahi hai, kyun ke `sum` us iterator ki ownership le leta hai jis par hum isay call karte hain.

### Methods That Produce Other Iterators

*Iterator adapters* `Iterator` trait par defined woh methods hain jo iterator ko consume nahi karte. Is ke bajaye, woh original iterator ke kisi aspect ko change karke different iterators produce karte hain.

Listing 13-14 mein `map` iterator adapter method ko call karne ki ek example dikhayi gayi hai, jo ek closure leta hai aur items par iterate karte waqt har item par us closure ko call karta hai. `map` method ek naya iterator return karta hai jo modified items produce karta hai. Yahan closure ek naya iterator create karta hai jismein vector ka har item 1 se increment kiya jayega.

<Listing number="13-14" file-name="src/main.rs" caption="Calling the iterator adapter `map` to create a new iterator">

```rust,not_desired_behavior id="g6q2pm"
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-14/src/main.rs:here}}
```

</Listing>

Lekin, yeh code ek warning produce karta hai:

```console id="w8d3ka"
{{#include ../listings/ch13-functional-features/listing-13-14/output.txt}}
```

Listing 13-14 mein code kuch nahi karta; humne jo closure specify kiya hai woh kabhi call hi nahi hota. Warning humein yaad dilati hai ke kyun: Iterator adapters lazy hote hain, aur yahan humein iterator ko consume karna zaroori hai.

Is warning ko fix karne aur iterator ko consume karne ke liye, hum `collect` method use karenge, jo humne Listing 12-1 mein `env::args` ke saath use kiya tha. Yeh method iterator ko consume karta hai aur resultant values ko ek collection data type mein collect karta hai.

Listing 13-15 mein, hum `map` ki call se return hone wale iterator par iteration ke results ko ek vector mein collect karte hain. Yeh vector aakhir mein original vector ke har item ko 1 se increment ki hui value contain karega.

<Listing number="13-15" file-name="src/main.rs" caption="Calling the `map` method to create a new iterator, and then calling the `collect` method to consume the new iterator and create a vector">

```rust id="r3n5vy"
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-15/src/main.rs:here}}
```

</Listing>

Kyun ke `map` ek closure leta hai, hum har item par perform kiye jane wale kisi bhi operation ko specify kar sakte hain. Yeh is baat ki ek achhi example hai ke closures kis tarah aapko kuch behavior ko customize karne ki ability dete hain, jab ke `Iterator` trait jo iteration behavior provide karta hai usay dobara use kiya ja sakta hai.

Aap complex actions ko readable tareeqe se perform karne ke liye iterator adapters ki multiple calls ko chain kar sakte hain. Lekin kyun ke tamam iterators lazy hote hain, is liye iterator adapters ki calls se results hasil karne ke liye aapko consuming adapter methods mein se kisi ek ko call karna hota hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-closures-that-capture-their-environment"></a>

### Closures That Capture Their Environment

Bohat se iterator adapters closures ko arguments ke taur par lete hain, aur aam tor par jo closures hum iterator adapters ko arguments ke taur par specify karenge, woh aise closures honge jo apne environment ko capture karte hain.

Is example ke liye, hum `filter` method use karenge jo ek closure leta hai. Closure iterator se ek item leta hai aur ek `bool` return karta hai. Agar closure `true` return karta hai, to value `filter` se produce hone wale iterator mein include ho jayegi. Agar closure `false` return karta hai, to value include nahi hogi.

Listing 13-16 mein, hum `filter` ko ek aise closure ke saath use karte hain jo apne environment se `shoe_size` variable ko capture karta hai, taake `Shoe` struct instances ki ek collection par iterate kiya ja sake. Yeh sirf un shoes ko return karega jo specified size ke hain.

<Listing number="13-16" file-name="src/lib.rs" caption="Using the `filter` method with a closure that captures `shoe_size`">

```rust,noplayground id="m4c7zs"
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-16/src/lib.rs}}
```

</Listing>

`shoes_in_size` function shoes ke ek vector ki ownership aur shoe size ko parameters ke taur par leta hai. Yeh ek aisa vector return karta hai jismein sirf specified size ke shoes hote hain.

`shoes_in_size` ki body mein, hum `into_iter` call karke ek aisa iterator create karte hain jo vector ki ownership leta hai. Phir, hum `filter` call karke us iterator ko ek naye iterator mein adapt karte hain jismein sirf woh elements hote hain jin ke liye closure `true` return karta hai.

Closure environment se `shoe_size` parameter ko capture karta hai aur us value ka har shoe ke size ke saath comparison karta hai, aur sirf specified size ke shoes ko rakhta hai. Aakhir mein, `collect` ko call karna adapted iterator se return hone wali values ko ek vector mein collect karta hai, jo function ke zariye return kiya jata hai.

Test dikhata hai ke jab hum `shoes_in_size` ko call karte hain, to humein sirf woh shoes wapas milte hain jin ka size us value ke same hota hai jo humne specify ki thi.
