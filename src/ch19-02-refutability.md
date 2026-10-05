## Refutability: Whether a Pattern Might Fail to Match

Patterns do forms mein hoti hain: refutable aur irrefutable. Woh patterns jo pass ki jane wali kisi bhi possible value ke saath match karengi *irrefutable* hoti hain. Is ki ek misaal `let x = 5;` statement mein `x` hai, kyun ke `x` kisi bhi cheez se match karta hai aur is liye match karne mein fail nahi ho sakta. Woh patterns jo kisi possible value ke liye match karne mein fail ho sakti hain *refutable* hoti hain. Is ki ek misaal `if let Some(x) = a_value` expression mein `Some(x)` hai, kyun ke agar `a_value` variable mein value `Some` ke bajaye `None` ho, to `Some(x)` pattern match nahi karegi.

Function parameters, `let` statements, aur `for` loops sirf irrefutable patterns accept kar sakte hain kyun ke jab values match na karein to program kuch meaningful nahi kar sakta. `if let` aur `while let` expressions aur `let...else` statement refutable aur irrefutable dono patterns accept karte hain, lekin compiler irrefutable patterns ke against warning deta hai kyun ke definition ke mutabiq, in constructs ka maqsad possible failure ko handle karna hota hai: conditional ki functionality is baat mein hai ke woh success ya failure ke mutabiq different tareeqe se perform kar sakti hai.

Aam tor par, aapko refutable aur irrefutable patterns ke darmiyan distinction ki zyada fikr nahi honi chahiye; lekin aapko refutability ke concept se familiar hona zaroori hai taake jab aap ise error message mein dekhein to us ka response de sakein. Un cases mein, aapko ya to pattern change karna hoga ya us construct ko jiske saath aap pattern use kar rahe hain, yeh code ke intended behavior par depend karta hai.

Aaiye ek example dekhte hain ke jab hum refutable pattern ko wahan use karne ki koshish karte hain jahan Rust ko irrefutable pattern chahiye, aur vice versa, to kya hota hai. Listing 19-8 mein ek `let` statement hai, lekin pattern ke liye humne `Some(x)` specify kiya hai, jo ek refutable pattern hai. Jaisa ke aap expect kar sakte hain, yeh code compile nahi hoga.

<Listing number="19-8" caption="Attempting to use a refutable pattern with `let`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-08/src/main.rs:here}}
```

</Listing>

Agar `some_option_value` ki value `None` hoti, to yeh `Some(x)` pattern ke saath match karne mein fail hoti, yani yeh pattern refutable hai. Lekin `let` statement sirf ek irrefutable pattern accept kar sakta hai kyun ke `None` value ke saath code kuch valid nahi kar sakta. Compile time par, Rust shikayat karega ke humne wahan refutable pattern use karne ki koshish ki hai jahan irrefutable pattern required hai:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-08/output.txt}}
```

Kyun ke humne `Some(x)` pattern ke saath har valid value ko cover nahi kiya (aur cover kar bhi nahi sakte the!), Rust bilkul durust tor par compiler error produce karta hai.

Agar hamare paas wahan refutable pattern ho jahan irrefutable pattern ki zaroorat hai, to hum pattern ko use karne wale code ko change karke ise fix kar sakte hain: `let` use karne ke bajaye, hum `let...else` use kar sakte hain. Phir, agar pattern match nahi karta, to curly brackets mein maujood code us value ko handle karega. Listing 19-9 dikhati hai ke Listing 19-8 ke code ko kaise fix kiya ja sakta hai.

<Listing number="19-9" caption="Using `let...else` and a block with refutable patterns instead of `let`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-09/src/main.rs:here}}
```

</Listing>

Humne code ko ek exit de diya hai! Yeh code bilkul valid hai, halaanke iska matlab hai ke hum irrefutable pattern ko warning receive kiye baghair use nahi kar sakte. Agar hum `let...else` ko aisa pattern dein jo hamesha match kare, jaise `x`, jaisa ke Listing 19-10 mein dikhaya gaya hai, to compiler warning dega.

<Listing number="19-10" caption="Attempting to use an irrefutable pattern with `let...else`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-10/src/main.rs:here}}
```

</Listing>

Rust shikayat karta hai ke `let...else` ko irrefutable pattern ke saath use karna sense nahi banata kyun ke `else` kabhi reach nahi hoga:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-10/output.txt}}
```

Isi wajah se, `match` arms ko refutable patterns use karni chahiye, siwaye last arm ke, jo irrefutable pattern ke saath baqi reh jane wali tamam values se match karna chahiye. Rust humein sirf ek arm wali `match` mein irrefutable pattern use karne ki permission deta hai, lekin yeh syntax khaas useful nahi hai aur isay ek simpler `let` statement se replace kiya ja sakta hai.

Ab jab aap jaante hain ke patterns ko kahan use karna hai aur refutable aur irrefutable patterns mein kya difference hai, to aaiye un tamam syntax ko cover karte hain jinhein hum patterns create karne ke liye use kar sakte hain.
