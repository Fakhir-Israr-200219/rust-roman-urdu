## Concise Control Flow with `if let` and `let...else`

`if let` syntax aapko `if` aur `let` ko combine karne deti hai taa-ke aap un values ko handle karne ka kam verbose tareeqa use kar saken jo ek particular pattern se match karti hain, jabke baqi values ko ignore kar dete hain. Listing 6-6 mein diye gaye program ko dekhein jo `config_max` variable mein mojood `Option<u8>` value par `match` karta hai, lekin sirf us waqt code execute karna chahta hai jab value `Some` variant ho.

<Listing number="6-6" caption="Ek `match` jo sirf us waqt code execute karne ki fikr karti hai jab value `Some` ho">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-06/src/main.rs:here}}
```

</Listing>

Agar value `Some` ho, to hum pattern mein value ko `max` variable ke saath bind karke `Some` variant ke andar mojood value ko print karte hain. Hum `None` value ke saath kuch nahi karna chahte. `match` expression ko satisfy karne ke liye, sirf ek variant ko process karne ke baad humein `_ => ()` add karna padta hai, jo annoying boilerplate code hai.

Is ke bajaye, hum `if let` ko use karke ise chhote tareeqe se likh sakte hain. Following code Listing 6-6 mein di gayi `match` ki tarah hi behave karta hai:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-12-if-let/src/main.rs:here}}
```

`if let` syntax ek pattern aur ek expression leti hai jinhein equal sign separate karta hai. Ye `match` ki tarah hi kaam karti hai, jahan expression ko `match` ko diya jata hai aur pattern uska pehla arm hota hai. Is case mein pattern `Some(max)` hai, aur `max` `Some` ke andar mojood value ke saath bind ho jata hai. Phir hum `if let` block ke body mein `max` ko usi tarah use kar sakte hain jis tarah corresponding `match` arm mein `max` ko use kiya tha. `if let` block ke andar code sirf us waqt run hota hai jab value pattern se match kare.

`if let` use karne ka matlab hai kam typing, kam indentation, aur kam boilerplate code. Lekin is ke badle aap woh exhaustive checking kho dete hain jo `match` enforce karti hai aur jo ye ensure karti hai ke aap kisi case ko handle karna bhool nahi rahe. `match` aur `if let` ke darmiyan choice is baat par depend karti hai ke aapki particular situation mein aap kya kar rahe hain aur kya conciseness hasil karna exhaustive checking khone ka munasib trade-off hai.

Doosre alfaaz mein, aap `if let` ko ek aisi `match` ke liye syntax sugar samajh sakte hain jo value ke ek pattern se match hone par code run karti hai aur phir baqi tamam values ko ignore kar deti hai.

Hum `if let` ke saath `else` bhi include kar sakte hain. `else` ke saath associated code ka block wohi hota hai jo equivalent `if let` aur `else` wali `match` expression mein `_` case ke saath associated hota. Listing 6-4 mein di gayi `Coin` enum ko yaad karein, jahan `Quarter` variant ke andar ek `UsState` value bhi thi. Agar hum quarters ki state announce karte hue un tamam non-quarter coins ko count karna chahte hon jo humein milte hain, to hum ye `match` expression ke saath is tarah kar sakte hain:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-13-count-and-announce-match/src/main.rs:here}}
```

Ya hum `if let` aur `else` expression ko is tarah use kar sakte hain:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-14-count-and-announce-if-let-else/src/main.rs:here}}
```

## `let...else` Ke Saath “Happy Path” Par Rehna

Aam pattern ye hai ke jab koi value present ho to kuch computation perform ki jaye aur warna ek default value return kar di jaye. `UsState` value wale coins ki apni example ko continue karte hue, agar hum quarter par maujood state ki age ke mutabiq kuch funny kehna chahte hon, to hum `UsState` par state ki age check karne ke liye ek method introduce kar sakte hain, jaise ke ye hai:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:state}}
```

Phir hum coin ki type par match karne ke liye `if let` use kar sakte hain, aur condition ki body ke andar ek `state` variable introduce kar sakte hain, jaisa ke Listing 6-7 mein hai.

<Listing number="6-7" caption="Conditionals ko `if let` ke andar nest karke check karna ke kya 1900 mein koi state mojood thi">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:describe}}
```

</Listing>

Ye kaam kar deta hai, lekin is ne sara kaam `if let` statement ki body ke andar push kar diya hai, aur agar kiya jane wala kaam zyada complicated ho, to ye samajhna mushkil ho sakta hai ke top-level branches ek doosre se exactly kis tarah related hain. Hum is fact ka bhi faida utha sakte hain ke expressions ek value produce karti hain, taa-ke ya to `if let` se `state` produce karein ya early return karein, jaisa ke Listing 6-8 mein hai. (Aap `match` ke saath bhi kuch isi tarah kar sakte hain.)

<Listing number="6-8" caption="Value produce karne ya early return karne ke liye `if let` use karna">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-08/src/main.rs:describe}}
```

</Listing>

Lekin apne tareeqe se isay follow karna bhi thora annoying hai! `if let` ki ek branch ek value produce karti hai, jabke doosri poore function se return kar deti hai.

Is common pattern ko express karna zyada behtar banane ke liye, Rust mein `let...else` hai. `let...else` syntax left side par ek pattern aur right side par ek expression leti hai, bilkul `if let` ki tarah, lekin is mein `if` branch nahi hoti, sirf `else` branch hoti hai. Agar pattern match kar jaye, to pattern se value outer scope mein bind ho jati hai. Agar pattern *match nahi* karti, to program `else` arm mein chala jata hai, jahan function se return karna zaroori hota hai.

Listing 6-9 mein aap dekh sakte hain ke `if let` ki jagah `let...else` use karne par Listing 6-8 kaisi nazar aati hai.

<Listing number="6-9" caption="Function ke flow ko clear karne ke liye `let...else` use karna">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-09/src/main.rs:describe}}
```

</Listing>

Notice karein ke is tarah function ki main body mein ye “happy path” par rehti hai, aur `if let` ki tarah dono branches ke liye significantly different control flow nahi hota.

Agar aapki situation mein program ki logic itni verbose ho jati hai ke use `match` ke zariye express karna mushkil ho, to yaad rakhein ke `if let` aur `let...else` bhi aapke Rust toolbox ka hissa hain.

## Summary

Ab hum ne ye cover kar liya hai ke enums ko kaise use karke custom types create kiye jate hain jo enumerated values ke ek set mein se koi ek value ho sakte hain. Hum ne ye bhi dekha ke standard library ka `Option<T>` type errors ko prevent karne ke liye type system ko use karne mein aapki kis tarah madad karta hai. Jab enum values ke andar data ho, to aap `match` ya `if let` ko use karke un values ko extract aur use kar sakte hain, ye is baat par depend karta hai ke aapko kitne cases handle karne hain.

Ab aapke Rust programs apne domain ke concepts ko structs aur enums ke zariye express kar sakte hain. Apni API mein use karne ke liye custom types create karna type safety ensure karta hai: Compiler ye make sure karega ke aapke functions ko sirf usi type ki values milen jis type ki value har function expect karta hai.

Apne users ke liye ek well-organized API provide karne ke liye jo use karne mein straightforward ho aur sirf wahi cheezen expose kare jo aapke users ko zaroorat hongi, ab aaiye Rust ke modules ki taraf chalte hain.

