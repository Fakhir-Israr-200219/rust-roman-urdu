## Packages and Crates

Module system ke pehle parts jinhein hum cover karenge woh packages aur crates hain.

Ek *crate* code ki sab se chhoti miktar hai jise Rust compiler ek waqt mein consider karta hai. Agar aap `cargo` ke bajaye `rustc` run karein aur ek single source code file pass karein (jaisa ke hum ne Chapter 1 mein [“Rust Program Basics”][basics]<!-- ignore
--> mein bilkul shuru mein kiya tha), to compiler us file ko ek crate samajhta hai. Crates mein modules ho sakte hain, aur modules doosri files mein define kiye ja sakte hain jo crate ke saath compile hoti hain, jaisa ke hum aanay wale sections mein dekhenge.

Ek crate do forms mein se kisi ek form mein ho sakta hai: binary crate ya library crate. *Binary crates* aise programs hote hain jinhein aap executable mein compile kar sakte hain jise aap run kar sakte hain, jaise command line program ya server. Har binary crate mein `main` naam ka ek function hona zaroori hai jo define karta hai ke executable run hone par kya hota hai. Ab tak hum ne jo tamam crates create kiye hain woh binary crates the.

*Library crates* mein `main` function nahi hota, aur ye executable mein compile nahi hote. Is ke bajaye, ye aisi functionality define karte hain jise multiple projects ke saath share karna hota hai. Misal ke taur par, `rand` crate jise hum ne [Chapter 2][rand]<!-- ignore --> mein use kiya tha, random numbers generate karne wali functionality provide karta hai. Zyada tar waqt jab Rustaceans “crate” kehte hain, to unki murad library crate hoti hai, aur woh “crate” ko general programming concept “library” ke saath interchangeably use karte hain.

*Crate root* ek source file hoti hai jahan se Rust compiler shuru karta hai aur jo aapke crate ka root module banati hai (hum modules ko [“Control Scope and Privacy with Modules”][modules]<!-- ignore --> mein detail mein explain karenge).

Ek *package* ek ya ek se zyada crates ka bundle hota hai jo functionality ka ek set provide karta hai. Package mein ek *Cargo.toml* file hoti hai jo describe karti hai ke un crates ko kis tarah build karna hai. Cargo asal mein ek package hai jisme us command line tool ke liye binary crate shamil hai jise aap apna code build karne ke liye use karte rahe hain. Cargo package mein ek library crate bhi shamil hai jis par binary crate depend karta hai. Doosre projects Cargo library crate par depend kar sakte hain taa-ke woh wahi logic use kar saken jo Cargo command line tool use karta hai.

Ek package mein aap jitne chahein binary crates rakh sakte hain, lekin zyada se zyada sirf ek library crate ho sakta hai. Ek package mein kam az kam ek crate hona zaroori hai, chahe woh library crate ho ya binary crate.

Aaiye dekhte hain ke jab hum ek package create karte hain to kya hota hai. Sab se pehle, hum `cargo new my-project` command enter karte hain:

```console
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

`cargo new my-project` run karne ke baad, hum `ls` use karke dekhte hain ke Cargo kya create karta hai. *my-project* directory mein ek *Cargo.toml* file hoti hai, jo humein ek package provide karti hai. Ek *src* directory bhi hoti hai jisme *main.rs* hoti hai. Apne text editor mein *Cargo.toml* open karein aur note karein ke *src/main.rs* ka koi zikr nahi hai. Cargo ek convention follow karta hai ke *src/main.rs* us binary crate ka crate root hota hai jiska name package ke same hota hai. Isi tarah, Cargo jaanta hai ke agar package directory mein *src/lib.rs* ho, to package mein package ke same name ka ek library crate hota hai, aur *src/lib.rs* uska crate root hota hai. Cargo library ya binary build karne ke liye crate root files ko `rustc` ke paas pass karta hai.

Yahan hamare paas ek aisa package hai jisme sirf *src/main.rs* shamil hai, yani is mein sirf `my-project` naam ka ek binary crate hai. Agar package mein *src/main.rs* aur *src/lib.rs* dono hon, to is mein do crates hote hain: ek binary aur ek library, dono ka name package ke same hota hai. Package mein multiple binary crates bhi ho sakte hain agar files ko *src/bin* directory mein rakha jaye: Har file ek separate binary crate hogi.

[basics]: ch01-02-hello-world.html#rust-program-basics
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
