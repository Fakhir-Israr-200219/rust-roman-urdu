## Structs Ko Use Karne Wala Ek Example Program

Ye samajhne ke liye ke hum structs ko kab use karna chahenge, aaiye ek aisa program likhte hain jo ek rectangle ka area calculate karta hai. Hum pehle individual variables ko use karne se shuru karenge aur phir program ko refactor karte jayenge, yahan tak ke hum structs use kar rahe hon.

Aaiye Cargo ke saath *rectangles* naam ka ek naya binary project banate hain jo pixels mein specify ki gayi rectangle ki width aur height lega aur rectangle ka area calculate karega. Listing 5-8 hamare project ki *src/main.rs* mein exactly ye kaam karne ke ek tareeqe ke saath ek chhota program dikhati hai.

<Listing number="5-8" file-name="src/main.rs" caption="Separate width aur height variables ke zariye specify kiye gaye rectangle ka area calculate karna">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```

</Listing>

Ab is program ko `cargo run` ke zariye run karein:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

Ye code har dimension ke saath `area` function ko call karke rectangle ka area successfully calculate kar leta hai, lekin hum is code ko aur clear aur readable banane ke liye mazeed behtar kar sakte hain.

Is code ka issue `area` ki signature mein wazeh hai:

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

`area` function ka kaam ek rectangle ka area calculate karna hai, lekin jo function hum ne likha hai us mein do parameters hain, aur hamare program mein kahin bhi ye clear nahi hai ke ye parameters aapas mein related hain. Width aur height ko ek saath group karna zyada readable aur manageable hoga. Hum ne Chapter 3 ke [“The Tuple Type”][the-tuple-type]<!-- ignore --> section mein pehle hi ek tareeqa discuss kiya tha jis se hum ye kar sakte hain: tuples ko use karke.

### Refactoring with Tuples

Listing 5-9 hamare program ka ek aur version dikhati hai jo tuples ko use karta hai.

<Listing number="5-9" file-name="src/main.rs" caption="Tuple ke zariye rectangle ki width aur height specify karna">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```

</Listing>

Ek tareeqe se ye program behtar hai. Tuples humein thori si structure add karne deti hain, aur ab hum sirf ek argument pass kar rahe hain. Lekin doosre tareeqe se ye version kam clear hai: Tuples apne elements ko name nahi karti, is liye humein tuple ke parts mein index ke zariye access karna padta hai, jis ki wajah se hamara calculation kam obvious ho jata hai.

Width aur height ko aapas mein mix kar dene se area calculation par koi farq nahi padega, lekin agar hum rectangle ko screen par draw karna chahein, to farq padega! Humein yaad rakhna padega ke `width` tuple ka index `0` hai aur `height` tuple ka index `1` hai. Agar koi doosra shakhs hamara code use kare, to uske liye ye samajhna aur yaad rakhna aur bhi mushkil hoga. Kyun ke hum ne apne code mein apne data ka meaning convey nahi kiya, is liye ab errors introduce karna aasaan ho gaya hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="refactoring-with-structs-adding-more-meaning"></a>

### Refactoring with Structs

Hum data ko label karke us mein meaning add karne ke liye structs use karte hain. Hum jis tuple ko use kar rahe hain use ek struct mein transform kar sakte hain, jiska poore group ke liye bhi ek name ho aur uske parts ke liye bhi names hon, jaisa ke Listing 5-10 mein dikhaya gaya hai.

<Listing number="5-10" file-name="src/main.rs" caption="Ek `Rectangle` struct define karna">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

</Listing>

Yahan hum ne ek struct define ki aur uska name `Rectangle` rakha. Curly brackets ke andar hum ne fields ko `width` aur `height` ke taur par define kiya, aur dono ki type `u32` hai. Phir, `main` mein hum ne `Rectangle` ka ek particular instance create kiya jis ki width `30` aur height `50` hai.

Ab hamara `area` function ek parameter ke saath define hai, jiska hum ne name `rectangle` rakha hai, aur jis ki type ek `Rectangle` struct instance ka immutable borrow hai. Jaisa ke Chapter 4 mein mention kiya gaya tha, hum struct ki ownership lene ke bajaye use borrow karna chahte hain. Is tarah `main` uski ownership retain karta hai aur `rect1` ko use karna jaari rakh sakta hai. Isi wajah se hum function signature mein aur function ko call karte waqt `&` use karte hain.

`area` function `Rectangle` instance ke `width` aur `height` fields ko access karta hai (note karein ke borrowed struct instance ke fields ko access karne se field values move nahi hoti, isi liye aap aksar structs ke borrows dekhte hain). Ab hamara `area` function signature bilkul wahi baat kehta hai jo hum mean karte hain: `Rectangle` ka area calculate karein, uske `width` aur `height` fields ko use karte hue. Ye convey karta hai ke width aur height ek doosre se related hain, aur values ko tuple ke index values `0` aur `1` ke bajaye descriptive names deta hai. Clarity ke liye ye ek behtari hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-useful-functionality-with-derived-traits"></a>

### Adding Functionality with Derived Traits

Debugging ke dauran `Rectangle` ke ek instance ko print karna aur uske tamam fields ki values dekhna useful hoga. Listing 5-11 mein hum [`println!` macro][println]<!-- ignore --> ko use karne ki koshish karte hain, jaisa ke hum ne pichlay chapters mein kiya hai. Lekin ye kaam nahi karega.

<Listing number="5-11" file-name="src/main.rs" caption="Ek `Rectangle` instance ko print karne ki koshish">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```

</Listing>

Jab hum is code ko compile karte hain, to humein is core message ke saath ek error milta hai:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

`println!` macro bohat qisam ki formatting kar sakta hai, aur by default, curly brackets `println!` ko `Display` ke naam se known formatting use karne ke liye kehte hain: yani aisi output jo directly end user ke use ke liye intended ho. Ab tak jin primitive types ko hum ne dekha hai, woh by default `Display` implement karti hain kyun ke kisi user ko `1` ya kisi doosri primitive type ko dikhane ka sirf ek hi tareeqa hota hai. Lekin structs ke saath ye kam clear hai ke `println!` ko output ko kis tarah format karna chahiye kyun ke display ki zyada possibilities hoti hain: Kya aap commas chahte hain ya nahi? Kya aap curly brackets print karna chahte hain? Kya tamam fields show ki jani chahiye? Is ambiguity ki wajah se, Rust ye guess karne ki koshish nahi karta ke hum kya chahte hain, aur structs ke paas `Display` ki koi provided implementation nahi hoti jise `println!` aur `{}` placeholder ke saath use kiya ja sake.

Agar hum errors ko parhna jaari rakhein, to humein ye helpful note milega:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

Aaiye ise try karte hain! `println!` macro call ab `println!("rect1 is {rect1:?}");` ki tarah nazar aayegi. Curly brackets ke andar `:?` specifier rakhne se `println!` ko bataya jata hai ke hum `Debug` naam ka output format use karna chahte hain. `Debug` trait humein apni struct ko aise tareeqe se print karne deta hai jo developers ke liye useful ho, taa-ke hum apne code ko debug karte waqt uski value dekh saken.

Is change ke saath code ko compile karein. Uff! Humein ab bhi ek error milta hai:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

Lekin ek baar phir, compiler humein ek helpful note deta hai:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

Rust mein debugging information print karne ki functionality *mojood* hai, lekin apni struct ke liye is functionality ko available karne ke liye humein explicitly opt in karna padta hai. Is ke liye, hum struct definition se bilkul pehle outer attribute `#[derive(Debug)]` add karte hain, jaisa ke Listing 5-12 mein dikhaya gaya hai.

<Listing number="5-12" file-name="src/main.rs" caption="`Debug` trait derive karne ke liye attribute add karna aur debug formatting use karke `Rectangle` instance ko print karna">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/src/main.rs}}
```

</Listing>

Ab jab hum program run karenge, to humein koi errors nahi milenge, aur humein following output nazar aayegi:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

Nice! Ye sab se khoobsurat output nahi hai, lekin ye is instance ke tamam fields ki values show karti hai, jo debugging ke dauran definitely helpful hogi. Jab hamare paas larger structs hon, to aisi output rakhna useful hota hai jo parhne mein thori aasaan ho; un cases mein hum `println!` string mein `{:?}` ke bajaye `{:#?}` use kar sakte hain. Is example mein, `{:#?}` style use karne se following output milegi:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

`Debug` format use karke value print karne ka ek aur tareeqa [`dbg!`
macro][dbg]<!-- ignore --> use karna hai, jo ek expression ki ownership leta hai (`println!` ke baraks, jo ek reference leta hai), apne code mein us `dbg!` macro call ke hone wali file aur line number ko expression ki resultant value ke saath print karta hai, aur phir value ki ownership return kar deta hai.

> Note: `dbg!` macro ko call karne se standard error console stream
> (`stderr`) par output print hoti hai, jabke `println!` standard output
> console stream (`stdout`) par print karta hai. Hum [“Redirecting Errors to
> Standard Error” section in Chapter 12][err]<!-- ignore --> mein `stderr` aur
> `stdout` ke baare mein mazeed baat karenge.

Yahan ek example hai jahan humein `width` field ko assign hone wali value ke saath saath `rect1` mein poori struct ki value mein bhi interest hai:

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/src/main.rs}}
```

Hum expression `30 * scale` ke around `dbg!` rakh sakte hain aur, kyun ke `dbg!` expression ki value ki ownership return karta hai, `width` field ko wahi value milegi jo `dbg!` call na hone ki surat mein milti. Hum nahi chahte ke `dbg!` `rect1` ki ownership le, is liye agli call mein hum `rect1` ka reference use karte hain. Yahan dekhein ke is example ki output kaisi nazar aati hai:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/output.txt}}
```

Hum dekh sakte hain ke output ka pehla hissa *src/main.rs* ki line 10 se aaya, jahan hum `30 * scale` expression ko debug kar rahe hain, aur iski resultant value `60` hai (`Debug` formatting jo integers ke liye implement ki gayi hai, sirf unki value print karti hai). *src/main.rs* ki line 14 par `dbg!` call `&rect1` ki value output karti hai, jo `Rectangle` struct hai. Ye output `Rectangle` type ki pretty `Debug` formatting use karti hai. `dbg!` macro us waqt waqai helpful ho sakta hai jab aap ye figure out karne ki koshish kar rahe hon ke aapka code kya kar raha hai!

`Debug` trait ke ilawa, Rust ne `derive` attribute ke saath use karne ke liye kai traits provide kiye hain jo hamari custom types mein useful behavior add kar sakte hain. Un traits aur unke behaviors ki list [Appendix C][app-c]<!--
ignore --> mein mojood hai. Hum Chapter 10 mein in traits ko custom behavior ke saath implement karne ke tareeqe ke saath saath apne khud ke traits create karne ka tareeqa bhi cover karenge. `derive` ke ilawa bhi bohat se attributes hain; mazeed maloomat ke liye [the “Attributes” section of the Rust Reference][attributes] dekhein.

Hamara `area` function bohat specific hai: Ye sirf rectangles ka area calculate karta hai. Is behavior ko hamari `Rectangle` struct ke saath zyada closely tie karna useful hoga kyun ke ye kisi doosri type ke saath kaam nahi karega. Aaiye dekhein ke `area` function ko hamari `Rectangle` type par define kiye gaye `area` method mein convert karke hum is code ko mazeed kaise refactor kar sakte hain.

[the-tuple-type]: ch03-02-data-types.html#the-tuple-type
[app-c]: appendix-03-derivable-traits.md
[println]: ../std/macro.println.html
[dbg]: ../std/macro.dbg.html
[err]: ch12-06-writing-to-stderr-instead-of-stdout.html
[attributes]: ../reference/attributes.html
