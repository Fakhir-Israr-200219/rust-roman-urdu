## `Result` Ke Saath Recoverable Errors

Zyada tar errors itne serious nahi hote ke program ko bilkul stop karna zaroori ho. Kabhi kabhi jab koi function fail hota hai, to uski wajah aisi hoti hai jise aap aasani se samajh sakte hain aur us ke mutabiq response de sakte hain. Misal ke taur par, agar aap kisi file ko open karne ki koshish karein aur operation is liye fail ho jaye ke file exist nahi karti, to shayad aap process ko terminate karne ke bajaye woh file create karna chahein.

Chapter 2 mein [“Handling Potential Failure with `Result`”][handle_failure]<!--
ignore --> se yaad karein ke `Result` enum ko do variants, `Ok` aur `Err`, ke saath define kiya gaya hai, jaisa ke neeche diya gaya hai:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` aur `E` generic type parameters hain: Hum Chapter 10 mein generics ke baare mein mazeed detail mein discuss karenge. Filhal aapko itna jaanne ki zaroorat hai ke `T` us value ki type ko represent karta hai jo success case mein `Ok` variant ke andar return ki jayegi, aur `E` us error ki type ko represent karta hai jo failure case mein `Err` variant ke andar return ki jayegi. Kyun ke `Result` mein ye generic type parameters hain, hum `Result` type aur is par defined functions ko bohat si different situations mein use kar sakte hain, jahan hum jo success value aur error value return karna chahte hain woh different ho sakti hain.

Aaiye ek aisa function call karte hain jo `Result` value return karta hai kyun ke woh fail ho sakta hai. Listing 9-3 mein, hum ek file open karne ki koshish karte hain.

<Listing number="9-3" file-name="src/main.rs" caption="Opening a file">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-03/src/main.rs}}
```

</Listing>

`File::open` ka return type `Result<T, E>` hai. Generic parameter `T` ko `File::open` ki implementation ne success value ki type, `std::fs::File`, ke saath fill kiya hai, jo ek file handle hai. Error value mein use hone wali `E` ki type `std::io::Error` hai. Is return type ka matlab hai ke `File::open` ki call successful ho sakti hai aur ek aisa file handle return kar sakti hai jise hum read ya write kar sakte hain. Function call fail bhi ho sakti hai: Misal ke taur par, file exist nahi karti ho sakti hai, ya shayad humein file access karne ki permission na ho. `File::open` function ke paas humein ye batane ka tareeqa hona chahiye ke woh successful hui ya fail, aur saath hi humein file handle ya error information mein se ek provide karni chahiye. Ye bilkul woh information hai jo `Result` enum convey karti hai.

Jab `File::open` successful hoti hai, to variable `greeting_file_result` mein `Ok` ka ek instance hoga jismein file handle hoga. Jab ye fail hoti hai, to `greeting_file_result` mein `Err` ka ek instance hoga jismein hone wale error ki type ke baare mein mazeed information hogi.

Humein Listing 9-3 ke code mein ye add karna hoga ke `File::open` ki taraf se return hone wali value ke mutabiq different actions liye ja saken. Listing 9-4 ek basic tool, `match` expression, ko use karke `Result` ko handle karne ka ek tareeqa dikhati hai jise hum Chapter 6 mein discuss kar chuke hain.

<Listing number="9-4" file-name="src/main.rs" caption="Using a `match` expression to handle the `Result` variants that might be returned">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-04/src/main.rs}}
```

</Listing>

Note karein ke `Option` enum ki tarah, `Result` enum aur iske variants bhi prelude ke zariye scope mein laaye gaye hain, is liye `match` arms mein `Ok` aur `Err` variants se pehle humein `Result::` specify karne ki zaroorat nahi hai.

Jab result `Ok` ho, to ye code `Ok` variant ke andar se inner `file` value return karega, aur phir hum us file handle ki value ko variable `greeting_file` mein assign kar dete hain. `match` ke baad, hum file handle ko reading ya writing ke liye use kar sakte hain.

`match` ki doosri arm us case ko handle karti hai jab humein `File::open` se `Err` value milti hai. Is example mein, hum ne `panic!` macro call karne ka intekhab kiya hai. Agar hamari current directory mein *hello.txt* naam ki koi file na ho aur hum ye code run karein, to humein `panic!` macro ki taraf se neeche diya gaya output nazar aayega:

```console
{{#include ../listings/ch09-error-handling/listing-09-04/output.txt}}
```

Hamesha ki tarah, ye output humein bilkul batata hai ke kya ghalat hua hai.

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result

### Different Errors Par Match Karna

Listing 9-4 mein diya gaya code `File::open` ke fail hone ki wajah chahe jo bhi ho, `panic!` karega. Lekin hum different failure reasons ke liye different actions lena chahte hain. Agar `File::open` is liye fail hui ke file exist nahi karti, to hum file create karna aur nayi file ka handle return karna chahte hain. Agar `File::open` kisi aur reason ki wajah se fail hui—misal ke taur par, kyun ke hamare paas file open karne ki permission nahi thi—to hum phir bhi chahte hain ke code bilkul Listing 9-4 ki tarah `panic!` kare. Is ke liye hum ek inner `match` expression add karte hain, jaisa ke Listing 9-5 mein dikhaya gaya hai.

<Listing number="9-5" file-name="src/main.rs" caption="Handling different kinds of errors in different ways">

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-05/src/main.rs}}
```

</Listing>

`Err` variant ke andar `File::open` jo value return karti hai uski type `io::Error` hai, jo standard library ki provide ki hui ek struct hai. Is struct mein `kind` naam ka ek method hai jise hum `io::ErrorKind` value hasil karne ke liye call kar sakte hain. `io::ErrorKind` enum standard library ki taraf se provide ki jati hai aur is mein different qisam ke errors ko represent karne wale variants hote hain jo kisi `io` operation ke natije mein aa sakte hain. Hum jis variant ko use karna chahte hain woh `ErrorKind::NotFound` hai, jo indicate karta hai ke jis file ko hum open karne ki koshish kar rahe hain woh abhi exist nahi karti. Is liye hum `greeting_file_result` par match karte hain, lekin saath hi `error.kind()` par ek inner match bhi karte hain.

Inner match mein hum jis condition ko check karna chahte hain woh ye hai ke `error.kind()` ki taraf se return hone wali value `ErrorKind` enum ka `NotFound` variant hai ya nahi. Agar aisa ho, to hum `File::create` ke zariye file create karne ki koshish karte hain. Lekin kyun ke `File::create` bhi fail ho sakti hai, humein inner `match` expression mein ek second arm ki zaroorat hoti hai. Jab file create nahi ho sakti, to ek different error message print kiya jata hai. Outer `match` ki second arm waisi hi rehti hai, is liye missing file error ke ilawa kisi bhi error par program panic karta hai.

> #### `Result<T, E>` Ke Saath `match` Use Karne Ke Alternatives
>
> Itni saari `match`! `match` expression bohat useful hai, lekin saath hi ye kaafi basic bhi hai. Chapter 13 mein aap closures ke baare mein seekhenge, jo `Result<T, E>` par defined bohat se methods ke saath use hoti hain. Jab aap apne code mein `Result<T, E>` values ko handle kar rahe hon, to ye methods `match` use karne ke muqable mein zyada concise ho sakti hain.
>
> Misal ke taur par, yahan Listing 9-5 mein dikhayi gayi same logic ko likhne ka ek aur tareeqa hai, is baar closures aur `unwrap_or_else` method ko use karte hue:
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore
> use std::fs::File;
> use std::io::ErrorKind;
>
> fn main() {
>     let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create("hello.txt").unwrap_or_else(|error| {
>                 panic!("Problem creating the file: {error:?}");
>             })
>         } else {
>             panic!("Problem opening the file: {error:?}");
>         }
>     });
> }
> ```
>
> Agarche ye code Listing 9-5 jaisa hi behavior rakhta hai, lekin is mein koi `match` expression nahi hai aur ye parhne mein zyada clean hai. Chapter 13 parhne ke baad is example ki taraf dobara aayein aur standard library ki documentation mein `unwrap_or_else` method ko dekhein. Jab aap errors ke saath deal kar rahe hon, to in mein se bohat se aur methods huge, nested `match` expressions ko clean up kar sakte hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="shortcuts-for-panic-on-error-unwrap-and-expect"></a>

#### Error Par Panic Karne Ke Shortcuts

`match` use karna kaafi achha kaam karta hai, lekin ye thora verbose ho sakta hai aur hamesha intent ko achhi tarah communicate nahi karta. `Result<T, E>` type par mukhtalif helper methods defined hain jo different, zyada specific tasks perform karte hain. `unwrap` method ek shortcut method hai jo bilkul usi tarah implement ki gayi hai jaise woh `match` expression jo hum ne Listing 9-4 mein likhi thi. Agar `Result` value `Ok` variant ho, to `unwrap` `Ok` ke andar wali value return karega. Agar `Result` `Err` variant ho, to `unwrap` hamare liye `panic!` macro call karega. Yahan `unwrap` ko action mein dekhte hain:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-04-unwrap/src/main.rs}}
```

</Listing>

Agar hum ye code *hello.txt* file ke baghair run karein, to humein `unwrap` method ki taraf se ki gayi `panic!` call ka error message nazar aayega:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-04-unwrap
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:4:49:
called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

Isi tarah, `expect` method humein `panic!` ka error message khud choose karne ki bhi ijazat deti hai. `unwrap` ke bajaye `expect` use karna aur achhe error messages provide karna aapke intent ko clear kar sakta hai aur panic ke source ko track down karna aasaan bana sakta hai. `expect` ki syntax kuch is tarah hoti hai:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-05-expect/src/main.rs}}
```

</Listing>

Hum `expect` ko `unwrap` ki tarah hi use karte hain: file handle return karne ke liye ya `panic!` macro call karne ke liye. `expect` ki taraf se `panic!` ko di jane wali error message woh parameter hogi jo hum `expect` ko pass karte hain, na ke woh default `panic!` message jo `unwrap` use karta hai. Ye is tarah nazar aati hai:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-05-expect
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:5:10:
hello.txt should be included in this project: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

Production-quality code mein, zyada tar Rustaceans `unwrap` ke bajaye `expect` ko choose karte hain aur is baat ke baare mein zyada context provide karte hain ke operation ke hamesha successful hone ki umeed kyun hai. Is tarah, agar kabhi aapki assumptions ghalat sabit hon, to debugging mein use karne ke liye aapke paas zyada information hoti hai.

### Errors Propagate Karna

Jab kisi function ki implementation kisi aisi cheez ko call karti hai jo fail ho sakti hai, to error ko function ke andar hi handle karne ke bajaye, aap error ko calling code ko return kar sakte hain taa-ke woh decide kar sake ke kya karna hai. Isay *error propagate karna* kaha jata hai aur is se calling code ko zyada control milta hai, kyun ke wahan aapke code ke context mein available information ya logic se zyada maloomat ya logic ho sakti hai jo ye decide karti hai ke error ko kis tarah handle karna chahiye.

Misal ke taur par, Listing 9-6 ek aisa function dikhati hai jo ek file se username read karta hai. Agar file exist nahi karti ya read nahi ki ja sakti, to ye function un errors ko us code ko return kar dega jis ne function ko call kiya tha.

<Listing number="9-6" file-name="src/main.rs" caption="A function that returns errors to the calling code using `match`">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-06/src/main.rs:here}}
```

</Listing>

Is function ko bohat chhote tareeqe se likha ja sakta hai, lekin error handling ko explore karne ke liye hum shuru mein iska kaafi hissa manually likhenge; aakhir mein hum iska shorter tareeqa dikhayenge. Sab se pehle function ke return type ko dekhein: `Result<String, io::Error>`. Is ka matlab hai ke function `Result<T, E>` type ki ek value return kar raha hai, jahan generic parameter `T` ko concrete type `String` ke saath aur generic type `E` ko concrete type `io::Error` ke saath fill kiya gaya hai.

Agar ye function baghair kisi problem ke successful hota hai, to jo code is function ko call karta hai usay `Ok` value milegi jismein ek `String` hogi—yani woh `username` jo is function ne file se read kiya. Agar is function ko koi problem encounter hoti hai, to calling code ko ek `Err` value milegi jismein `io::Error` ka ek instance hoga jo problems ke baare mein mazeed information contain karta hai. Hum ne is function ke return type ke taur par `io::Error` is liye choose kiya kyun ke ye ittefaq se un dono operations se return hone wali error value ki type hai jinhein hum is function ke body mein call kar rahe hain aur jo fail ho sakte hain: `File::open` function aur `read_to_string` method.

Function ki body `File::open` function ko call karne se shuru hoti hai. Phir hum `Result` value ko Listing 9-4 ke `match` jaisi `match` ke zariye handle karte hain. Agar `File::open` successful hoti hai, to pattern variable `file` mein file handle ki value mutable variable `username_file` ki value ban jati hai aur function continue karta hai. `Err` case mein, `panic!` call karne ke bajaye hum `return` keyword use karke function se foran aur poori tarah bahar return karte hain aur `File::open` ki error value, jo ab pattern variable `e` mein hai, calling code ko is function ki error value ke taur par pass kar dete hain.

Is liye, agar `username_file` mein file handle mojood ho, to function variable `username` mein ek naya `String` create karta hai aur `username_file` mein mojood file handle par `read_to_string` method call karke file ke contents ko `username` mein read karta hai. `read_to_string` method bhi ek `Result` return karti hai kyun ke ye fail ho sakti hai, agarche `File::open` successful hui ho. Is liye humein us `Result` ko handle karne ke liye ek aur `match` ki zaroorat hai: Agar `read_to_string` successful hoti hai, to hamara function successful ho gaya hai, aur hum file se read kiya gaya username, jo ab `username` mein hai, `Ok` mein wrap karke return karte hain. Agar `read_to_string` fail hoti hai, to hum error value ko bilkul usi tarah return karte hain jis tarah hum ne `File::open` ki return value ko handle karne wali `match` mein error value return ki thi. Lekin humein explicitly `return` kehne ki zaroorat nahi hai, kyun ke ye function ka last expression hai.

Jo code is code ko call karta hai, woh phir ya to username contain karne wali `Ok` value hasil karega ya `io::Error` contain karne wali `Err` value. In values ke saath kya karna hai, ye calling code par depend karta hai. Agar calling code ko `Err` value milti hai, to woh, misal ke taur par, `panic!` call karke program ko crash kar sakta hai, default username use kar sakta hai, ya file ke ilawa kisi aur jagah se username lookup kar sakta hai. Humein itni information nahi hai ke calling code asal mein kya karne ki koshish kar raha hai, is liye hum tamam success ya error information ko upar propagate kar dete hain taa-ke woh usay munasib tareeqe se handle kar sake.

Errors propagate karne ka ye pattern Rust mein itna common hai ke Rust isay aasaan banane ke liye question mark operator `?` provide karta hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-shortcut-for-propagating-errors-the--operator"></a>

#### `?` Operator Ka Shortcut

Listing 9-7 mein `read_username_from_file` ki ek aisi implementation dikhayi gayi hai jo Listing 9-6 jaisi hi functionality rakhti hai, lekin is implementation mein `?` operator use kiya gaya hai.

<Listing number="9-7" file-name="src/main.rs" caption="A function that returns errors to the calling code using the `?` operator">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-07/src/main.rs:here}}
```

</Listing>

Kisi `Result` value ke baad lagaya gaya `?` lagbhag usi tarah kaam karta hai jis tarah Listing 9-6 mein `Result` values ko handle karne ke liye hum ne `match` expressions define ki thi. Agar `Result` ki value `Ok` ho, to `Ok` ke andar wali value is expression se return ho jayegi aur program continue karega. Agar value `Err` ho, to `Err` poore function se aise return ho jayega jaise hum ne `return` keyword use kiya ho, taa-ke error value calling code ko propagate ho jaye.

Listing 9-6 ki `match` expression aur `?` operator ke behavior mein ek farq hai: Jin error values par `?` operator call kiya jata hai, woh standard library ke `From` trait mein defined `from` function se guzarti hain, jo values ko ek type se doosri type mein convert karne ke liye use hota hai. Jab `?` operator `from` function ko call karta hai, to receive hone wali error type ko current function ke return type mein defined error type mein convert kar diya jata hai. Ye us waqt useful hota hai jab koi function tamam possible failure ways ko represent karne ke liye ek hi error type return karta hai, chahe function ke different parts kai different reasons ki wajah se fail ho sakte hon.

Misal ke taur par, hum Listing 9-7 mein `read_username_from_file` function ko change karke `OurError` naam ka ek custom error type return karwa sakte hain jo hum khud define karein. Agar hum `impl From<io::Error> for OurError` bhi define karein taa-ke `io::Error` se `OurError` ka instance construct kiya ja sake, to `read_username_from_file` ki body mein `?` operator ki calls `from` ko call karengi aur error types ko convert kar dengi, bina function mein koi aur code add kiye.

Listing 9-7 ke context mein, `File::open` call ke end par maujood `?` `Ok` ke andar wali value ko variable `username_file` ko return karega. Agar koi error hota hai, to `?` operator poore function se foran return karega aur koi bhi `Err` value calling code ko de dega. Yahi baat `read_to_string` call ke end par maujood `?` par bhi apply hoti hai.

`?` operator bohat saara boilerplate khatam kar deta hai aur is function ki implementation ko simpler bana deta hai. Hum `?` ke foran baad method calls ko chain karke is code ko aur bhi short kar sakte hain, jaisa ke Listing 9-8 mein dikhaya gaya hai.

<Listing number="9-8" file-name="src/main.rs" caption="Chaining method calls after the `?` operator">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-08/src/main.rs:here}}
```

</Listing>

Hum ne `username` mein naya `String` create karne ko function ke shuru mein move kar diya hai; ye hissa change nahi hua. `username_file` naam ka variable create karne ke bajaye, hum ne `read_to_string` ki call ko directly `File::open("hello.txt")?` ke result ke saath chain kar diya hai. `read_to_string` call ke end par ab bhi `?` maujood hai, aur jab `File::open` aur `read_to_string` dono successful hote hain, to hum ab bhi `username` ko contain karne wali `Ok` value return karte hain, errors return nahi karte. Functionality phir se Listing 9-6 aur Listing 9-7 jaisi hi hai; bas ise likhne ka ye ek different, zyada ergonomic tareeqa hai.

Listing 9-9 mein `fs::read_to_string` ko use karke is code ko aur bhi short karne ka tareeqa dikhaya gaya hai.

<Listing number="9-9" file-name="src/main.rs" caption="Using `fs::read_to_string` instead of opening and then reading the file">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-09/src/main.rs:here}}
```

</Listing>

Kisi file ko string mein read karna ek kaafi common operation hai, is liye standard library convenient `fs::read_to_string` function provide karti hai jo file ko open karti hai, ek naya `String` create karti hai, file ke contents ko read karti hai, un contents ko us `String` mein daalti hai, aur usay return karti hai. Beshak, `fs::read_to_string` use karne se humein tamam error handling ko explain karne ka mauqa nahi milta, is liye hum ne pehle isay longer way mein kiya.

<!-- Old headings. Do not remove or links may break. -->

<a id="where-the--operator-can-be-used"></a>

#### `?` Operator Kahan Use Karna Hai

`?` operator sirf un functions mein use kiya ja sakta hai jin ka return type us value ke saath compatible ho jis par `?` use kiya ja raha hai. Is ki wajah ye hai ke `?` operator function se value ko foran return karne ke liye define kiya gaya hai, bilkul usi tarah jaise woh `match` expression jo hum ne Listing 9-6 mein define ki thi. Listing 9-6 mein, `match` ek `Result` value use kar raha tha, aur early return wali arm ne `Err(e)` value return ki thi. Function ka return type `Result` hona zaroori hai taa-ke woh is `return` ke saath compatible ho.

Listing 9-10 mein dekhte hain ke agar hum `?` operator ko aise `main` function mein use karein jis ka return type us value ki type ke saath compatible nahi hai jis par hum `?` use kar rahe hain, to humein kya error milega.

<Listing number="9-10" file-name="src/main.rs" caption="Attempting to use the `?` in the `main` function that returns `()` won’t compile.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-10/src/main.rs}}
```

</Listing>

Ye code ek file open karta hai, jo fail ho sakti hai. `File::open` ki taraf se return hone wali `Result` value ke baad `?` operator hai, lekin is `main` function ka return type `()` hai, `Result` nahi. Jab hum is code ko compile karte hain, to humein neeche diya gaya error message milta hai:

```console
{{#include ../listings/ch09-error-handling/listing-09-10/output.txt}}
```

Ye error batata hai ke humein `?` operator sirf aise function mein use karne ki ijazat hai jo `Result`, `Option`, ya kisi doosri aisi type return karta ho jo `FromResidual` implement karti ho.

Is error ko fix karne ke liye aapke paas do choices hain. Ek choice ye hai ke apne function ka return type us value ke saath compatible kar dein jis par aap `?` operator use kar rahe hain, jab tak aapko rokne wali koi restriction na ho. Doosri choice ye hai ke `Result<T, E>` ko us tareeqe se handle karne ke liye `match` ya `Result<T, E>` ke methods mein se kisi ek ko use karein jo appropriate ho.

Error message ne ye bhi mention kiya tha ke `?` ko `Option<T>` values ke saath bhi use kiya ja sakta hai. `Result` par `?` use karne ki tarah, aap `Option` par `?` sirf aise function mein use kar sakte hain jo `Option` return karta ho. `Option<T>` par `?` operator call hone par iska behavior `Result<T, E>` par call hone ke behavior jaisa hai: Agar value `None` ho, to us point par function se `None` foran return ho jayega. Agar value `Some` ho, to `Some` ke andar wali value expression ka resultant value hogi aur function continue karega. Listing 9-11 mein aise function ki example hai jo diye gaye text ki first line ka last character find karta hai.

<Listing number="9-11" caption="Using the `?` operator on an `Option<T>` value">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-11/src/main.rs:here}}
```

</Listing>

Ye function `Option<char>` return karta hai kyun ke mumkin hai ke wahan koi character ho, lekin ye bhi mumkin hai ke wahan koi character na ho. Ye code `text` string slice argument leta hai aur is par `lines` method call karta hai, jo string ki lines par ek iterator return karti hai. Kyun ke ye function first line ko examine karna chahta hai, ye iterator par `next` call karta hai taa-ke iterator se first value hasil ki ja sake. Agar `text` empty string ho, to `next` ki ye call `None` return karegi, aur is surat mein hum `?` use karke `last_char_of_first_line` se `None` ko foran stop karke return karte hain. Agar `text` empty string nahi hai, to `next` ek `Some` value return karega jismein `text` ki first line ka string slice hoga.

`?` string slice ko extract karta hai, aur hum us string slice par `chars` call karke uske characters ka iterator hasil kar sakte hain. Hum is first line ke last character mein interested hain, is liye hum iterator mein se last item return karne ke liye `last` call karte hain. Ye ek `Option` hai kyun ke mumkin hai ke first line empty string ho; misal ke taur par, agar `text` ek blank line se start ho lekin doosri lines mein characters hon, jaisa ke `"\nhi"` mein hai. Lekin agar first line mein koi last character hai, to woh `Some` variant mein return hoga. Darmiyan mein `?` operator humein is logic ko concise tareeqe se express karne deta hai, jis ki wajah se hum function ko ek hi line mein implement kar sakte hain. Agar hum `Option` par `?` operator use nahi kar sakte, to humein is logic ko mazeed method calls ya `match` expression ke zariye implement karna padta.

Note karein ke aap `Result` return karne wale function mein `Result` par `?` operator use kar sakte hain, aur `Option` return karne wale function mein `Option` par `?` operator use kar sakte hain, lekin aap dono ko mix and match nahi kar sakte. `?` operator automatically `Result` ko `Option` ya `Option` ko `Result` mein convert nahi karta; un cases mein, aap conversion ko explicitly karne ke liye `Result` par `ok` method ya `Option` par `ok_or` method jaise methods use kar sakte hain.

Ab tak hum ne jitne bhi `main` functions use kiye hain woh `()` return karte hain. `main` function special hai kyun ke ye executable program ka entry point aur exit point hota hai, aur is baat par restrictions hoti hain ke program ke expected tareeqe se behave karne ke liye iska return type kya ho sakta hai.

Khush qismati se, `main` `Result<(), E>` bhi return kar sakta hai. Listing 9-12 mein Listing 9-10 ka code hai, lekin hum ne `main` ka return type change karke `Result<(), Box<dyn Error>>` kar diya hai aur end par return value `Ok(())` add kar di hai. Ab ye code compile hoga.

<Listing number="9-12" file-name="src/main.rs" caption="Changing `main` to return `Result<(), E>` allows the use of the `?` operator on `Result` values.">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-12/src/main.rs}}
```

</Listing>

`Box<dyn Error>` type ek trait object hai, jiske baare mein hum Chapter 18 mein [“Using Trait Objects to Abstract over Shared Behavior”][trait-objects]<!-- ignore --> mein baat karenge. Filhal, aap `Box<dyn Error>` ko “kisi bhi qisam ka error” samajh sakte hain. `Box<dyn Error>` error type wale `main` function mein `Result` value par `?` use karne ki ijazat hai kyun ke ye kisi bhi `Err` value ko foran return karne ki ijazat deta hai. Agarche is `main` function ki body sirf `std::io::Error` type ke errors return karegi, `Box<dyn Error>` specify karne se ye signature tab bhi correct rahegi agar `main` ki body mein aisa mazeed code add kiya jaye jo doosre errors return karta ho.

Jab `main` function `Result<(), E>` return karta hai, to executable `0` ki value ke saath exit karega agar `main` `Ok(())` return kare, aur agar `main` `Err` value return kare to executable nonzero value ke saath exit karega. C mein likhe gaye executables exit hone par integers return karte hain: Jo programs successfully exit hote hain woh integer `0` return karte hain, aur jo programs error ke saath exit hote hain woh `0` ke ilawa koi integer return karte hain. Rust bhi is convention ke saath compatible rehne ke liye executables se integers return karta hai.

`main` function un tamam types ko return kar sakta hai jo [the
`std::process::Termination` trait][termination]<!-- ignore --> implement karti hain, jismein ek `report` function hota hai jo `ExitCode` return karta hai. Apni types ke liye `Termination` trait implement karne ke baare mein mazeed maloomat ke liye standard library ki documentation dekhein.

Ab jab ke hum ne `panic!` call karne ya `Result` return karne ki details discuss kar li hain, to aaiye dobara is topic ki taraf chalte hain ke different cases mein in mein se kisay use karna munasib hai.

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result
[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[termination]: ../std/process/trait.Termination.html
