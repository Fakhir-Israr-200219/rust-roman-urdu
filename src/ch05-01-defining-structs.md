## Defining and Instantiating Structs

Structs tuples ke mushabih hoti hain, jin par [“The Tuple Type”][tuples]<!--
ignore --> section mein baat ki gayi thi, kyun ke dono multiple related values ko hold karti hain. Tuples ki tarah, struct ke pieces different types ke ho sakte hain. Tuples ke baraks, struct mein aap har data piece ko name dete hain taa-ke ye clear ho ke values ka kya matlab hai. Ye names add karne se structs tuples se zyada flexible ho jati hain: Aapko kisi instance ki values ko specify ya access karne ke liye data ke order par depend nahi karna padta.

Struct define karne ke liye, hum `struct` keyword enter karte hain aur poori struct ko name dete hain. Struct ka name un data pieces ki significance ko describe karna chahiye jinhein ek saath group kiya ja raha hai. Phir curly brackets ke andar hum data pieces ke names aur types define karte hain, jinhein hum *fields* kehte hain. Misal ke taur par, Listing 5-1 ek aisi struct dikhati hai jo user account ki information store karti hai.

<Listing number="5-1" file-name="src/main.rs" caption="Ek `User` struct ki definition">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-01/src/main.rs:here}}
```

</Listing>

Struct ko define karne ke baad use karne ke liye, hum us struct ka ek *instance* create karte hain aur har field ke liye concrete values specify karte hain. Hum struct ka name likh kar aur phir curly brackets mein *`key: value`* pairs add karke instance create karte hain, jahan keys fields ke names hoti hain aur values woh data hota hai jo hum un fields mein store karna chahte hain. Humein fields ko usi order mein specify karne ki zaroorat nahi hoti jis order mein hum ne unhein struct mein declare kiya tha. Doosre alfaaz mein, struct definition type ke liye ek general template ki tarah hoti hai, aur instances us template ko particular data se fill karke us type ki values create karte hain. Misal ke taur par, hum ek particular user ko Listing 5-2 mein dikhaye gaye tareeqe se declare kar sakte hain.

<Listing number="5-2" file-name="src/main.rs" caption="`User` struct ka ek instance create karna">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-02/src/main.rs:here}}
```

</Listing>

Struct se koi specific value hasil karne ke liye, hum dot notation use karte hain. Misal ke taur par, is user ka email address access karne ke liye hum `user1.email` use karte hain. Agar instance mutable ho, to hum dot notation use karke aur kisi particular field ko value assign karke us value ko change kar sakte hain. Listing 5-3 dikhati hai ke mutable `User` instance ke `email` field ki value ko kaise change kiya jata hai.

<Listing number="5-3" file-name="src/main.rs" caption="`User` instance ke `email` field ki value change karna">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-03/src/main.rs:here}}
```

</Listing>

Note karein ke poora instance mutable hona zaroori hai; Rust humein sirf kuch specific fields ko mutable mark karne ki ijazat nahi deta. Kisi bhi expression ki tarah, hum function body ke last expression ke taur par struct ka ek naya instance construct kar sakte hain taa-ke woh naya instance implicitly return ho jaye.

Listing 5-4 ek `build_user` function dikhati hai jo diye gaye email aur username ke saath ek `User` instance return karta hai. `active` field ko `true` ki value milti hai, aur `sign_in_count` ko `1` ki value milti hai.

<Listing number="5-4" file-name="src/main.rs" caption="Ek `build_user` function jo email aur username leta hai aur `User` instance return karta hai">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-04/src/main.rs:here}}
```

</Listing>

Struct fields ke same names ke saath function parameters ko name karna sense banata hai, lekin `email` aur `username` field names aur variables ko dobara likhna thora tedious hai. Agar struct mein zyada fields hoti, to har name ko repeat karna aur bhi annoying ho jata. Khush qismati se, ek convenient shorthand mojood hai!

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-field-init-shorthand-when-variables-and-fields-have-the-same-name"></a>


### Using the Field Init Shorthand

Kyun ke Listing 5-4 mein parameter names aur struct field names bilkul same hain, hum *field init shorthand* syntax ko use karke `build_user` ko is tarah rewrite kar sakte hain ke woh bilkul wahi kaam kare, lekin `username` aur `email` ko repeat na karna pade, jaisa ke Listing 5-5 mein dikhaya gaya hai.

<Listing number="5-5" file-name="src/main.rs" caption="Ek `build_user` function jo field init shorthand use karta hai kyun ke `username` aur `email` parameters ke names struct fields ke names ke same hain">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```

</Listing>

Yahan hum `User` struct ka ek naya instance create kar rahe hain, jisme `email` naam ka ek field hai. Hum `email` field ki value ko `build_user` function ke `email` parameter mein mojood value par set karna chahte hain. Kyun ke `email` field aur `email` parameter ka name same hai, humein `email: email` ke bajaye sirf `email` likhne ki zaroorat hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-instances-from-other-instances-with-struct-update-syntax"></a>


### Creating Instances with Struct Update Syntax

Aksar ye useful hota hai ke ek struct ka naya instance create kiya jaye jisme kisi doosre same type ke instance ki zyada tar values shamil hon, lekin kuch values different hon. Aap ye struct update syntax ko use karke kar sakte hain.

Sab se pehle, Listing 5-6 mein hum dikhate hain ke `user2` mein ek naya `User` instance regular tareeqe se kaise create kiya jata hai, yani update syntax ke baghair. Hum `email` ke liye ek nayi value set karte hain, lekin baqi tamam values Listing 5-2 mein create kiye gaye `user1` se use karte hain.

<Listing number="5-6" file-name="src/main.rs" caption="`user1` ki ek value ke ilawa tamam values ko use karke ek naya `User` instance create karna">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-06/src/main.rs:here}}
```

</Listing>

Struct update syntax ko use karke, hum kam code ke saath bilkul wahi result hasil kar sakte hain, jaisa ke Listing 5-7 mein dikhaya gaya hai. Syntax `..` specify karti hai ke jo remaining fields explicitly set nahi kiye gaye, unki value diye gaye instance ke corresponding fields ke same honi chahiye.

<Listing number="5-7" file-name="src/main.rs" caption="Ek `User` instance ke liye nayi `email` value set karna aur baqi values `user1` se use karne ke liye struct update syntax ka istemal">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-07/src/main.rs:here}}
```

</Listing>

Listing 5-7 ka code `user2` mein ek aisa instance bhi create karta hai jis mein `email` ki value different hai, lekin `username`, `active`, aur `sign_in_count` fields ki values `user1` ke same hain. `..user1` ka aakhir mein hona zaroori hai taa-ke specify kiya ja sake ke jo bhi remaining fields hain, unki values `user1` ke corresponding fields se li jani chahiye, lekin hum jitni fields chahein unke liye values kisi bhi order mein specify kar sakte hain, chahe woh order struct ki definition mein fields ke order se different hi kyun na ho.

Note karein ke struct update syntax `=` ko assignment ki tarah use karti hai; iski wajah ye hai ke ye data ko move karti hai, bilkul usi tarah jaise hum ne [“Variables and Data Interacting with Move”][move]<!-- ignore --> section mein dekha tha. Is example mein, `user2` create karne ke baad hum `user1` ko ab use nahi kar sakte, kyun ke `user1` ke `username` field mein mojood `String` ko `user2` mein move kar diya gaya hai. Agar hum `user2` ko `email` aur `username` dono ke liye nayi `String` values dete, aur is tarah `user1` se sirf `active` aur `sign_in_count` ki values use karte, to `user2` create karne ke baad bhi `user1` valid rehta. `active` aur `sign_in_count` dono aisi types hain jo `Copy` trait implement karti hain, is liye [“Stack-Only Data: Copy”][copy]<!-- ignore --> section mein discuss kiya gaya behavior yahan apply hota. Hum is example mein `user1.email` ko bhi use kar sakte hain, kyun ke iski value `user1` se move nahi hui.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-tuple-structs-without-named-fields-to-create-different-types"></a>

### Creating Different Types with Tuple Structs

Rust aisi structs ko bhi support karta hai jo tuples jaisi nazar aati hain, jinhein *tuple structs* kaha jata hai. Tuple structs mein struct name ki wajah se additional meaning hota hai, lekin inke fields ke saath names associated nahi hote; is ke bajaye, sirf fields ki types hoti hain. Tuple structs us waqt useful hoti hain jab aap poore tuple ko ek name dena chahte hon aur tuple ko doosre tuples se different type banana chahte hon, aur jab regular struct ki tarah har field ko name dena verbose ya redundant ho.

Tuple struct define karne ke liye, `struct` keyword aur struct name se start karein, aur uske baad tuple mein types likhein. Misal ke taur par, yahan hum `Color` aur `Point` naam ki do tuple structs define aur use kar rahe hain:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-01-tuple-structs/src/main.rs}}
```

</Listing>

Note karein ke `black` aur `origin` values different types hain kyun ke ye different tuple structs ke instances hain. Aap jo bhi struct define karte hain, woh apni ek separate type hoti hai, chahe struct ke andar fields ki types same hi kyun na hon. Misal ke taur par, ek aisa function jo `Color` type ka parameter leta hai, `Point` ko argument ke taur par nahi le sakta, halaanke dono types teen `i32` values par mushtamil hain. Is ke ilawa, tuple struct instances tuples ke mushabih hoti hain kyun ke aap unhein unke individual pieces mein destructure kar sakte hain, aur kisi individual value ko access karne ke liye `.` ke baad index use kar sakte hain. Tuples ke baraks, tuple structs ko destructure karte waqt aapko struct ki type ka name dena zaroori hota hai. Misal ke taur par, `origin` point ki values ko `x`, `y`, aur `z` naam ke variables mein destructure karne ke liye hum `let Point(x, y, z) = origin;` likhenge.

<!-- Old headings. Do not remove or links may break. -->

<a id="unit-like-structs-without-any-fields"></a>

### Defining Unit-Like Structs

Aap aisi structs bhi define kar sakte hain jin mein koi fields nahi hoti! Inhein *unit-like structs* kaha jata hai kyun ke ye `()`, yani unit type, ki tarah behave karti hain, jis ka hum ne [“The Tuple Type”][tuples]<!-- ignore --> section mein zikr kiya tha. Unit-like structs us waqt useful ho sakti hain jab aapko kisi type par trait implement karna ho lekin aapke paas koi aisa data na ho jo aap type ke andar store karna chahte hon. Hum Chapter 10 mein traits par baat karenge. Yahan `AlwaysEqual` naam ki ek unit struct ko declare aur instantiate karne ki example hai:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-04-unit-like-structs/src/main.rs}}
```

</Listing>

`AlwaysEqual` ko define karne ke liye, hum `struct` keyword, apna desired name, aur phir ek semicolon use karte hain. Curly brackets ya parentheses ki koi zaroorat nahi! Phir, hum `subject` variable mein `AlwaysEqual` ka ek instance bhi isi tarah hasil kar sakte hain: Jo name hum ne define kiya hai use bina kisi curly brackets ya parentheses ke use karke. Tasawwur karein ke baad mein hum is type ke liye aisa behavior implement karenge ke `AlwaysEqual` ka har instance hamesha kisi bhi doosri type ke har instance ke barabar ho, shayad testing purposes ke liye ek known result hasil karne ke liye. Is behavior ko implement karne ke liye humein kisi data ki zaroorat nahi hogi! Chapter 10 mein aap dekhenge ke traits ko kaise define kiya jata hai aur unhein kisi bhi type par, including unit-like structs, kaise implement kiya jata hai.


> ### Struct Data ki Ownership
>
> Listing 5-1 mein `User` struct ki definition mein hum ne `&str` string slice type ke bajaye owned `String` type use ki thi. Ye ek deliberate choice hai kyun ke hum chahte hain ke is struct ka har instance apne tamam data ka owner ho aur woh data utni der tak valid rahe jitni der tak poora struct valid hai.
>
> Structs ke liye ye bhi mumkin hai ke woh kisi aur ki owned data ke references store karein, lekin aisa karne ke liye *lifetimes* ka use zaroori hota hai, jo Rust ka ek feature hai aur jis par hum Chapter 10 mein baat karenge. Lifetimes ensure karti hain ke struct jis data ko reference karta hai woh utni der tak valid rahe jitni der tak struct valid hai. Maan lein ke aap lifetimes specify kiye baghair struct mein ek reference store karne ki koshish karte hain, jaisa ke *src/main.rs* mein following example mein hai; ye kaam nahi karega:
>
> <Listing file-name="src/main.rs">
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore,does_not_compile
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
>
> fn main() {
>     let user1 = User {
>         active: true,
>         username: "someusername123",
>         email: "someone@example.com",
>         sign_in_count: 1,
>     };
> }
> ```
>
> </Listing>
>
> Compiler complain karega ke use lifetime specifiers ki zaroorat hai:
>
> ```console
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
>
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
>
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` (bin "structs") due to 2 previous errors
> ```
>
> Chapter 10 mein hum discuss karenge ke in errors ko kaise fix kiya jaye taa-ke aap structs mein references store kar saken, lekin filhaal hum is tarah ke errors ko references jaise `&str` ke bajaye `String` jaise owned types use karke fix karenge.

<!-- manual-regeneration
for the error above
after running update-rustc.sh:
pbcopy < listings/ch05-using-structs-to-structure-related-data/no-listing-02-reference-in-struct/output.txt
paste above
add `> ` before every line -->

[tuples]: ch03-02-data-types.html#the-tuple-type
[move]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move
[copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
