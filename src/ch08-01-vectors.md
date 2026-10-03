## Vectors Ke Zariye Values Ki Lists Store Karna

Pehli collection type jise hum dekhenge woh `Vec<T>` hai, jise vector bhi kaha jata hai.
Vectors aapko ek hi data structure mein ek se zyada values store karne dete hain, jo
memory mein tamam values ko ek doosre ke paas rakhta hai. Vectors sirf ek hi type ki
values store kar sakte hain. Ye us waqt useful hote hain jab aapke paas items ki koi
list ho, jaise kisi file mein text ki lines ya shopping cart mein items ki prices.


### Naya Vector Create Karna

Ek naya, empty vector create karne ke liye hum `Vec::new` function call karte hain, jaisa ke
Listing 8-1 mein dikhaya gaya hai.

<Listing number="8-1" caption="`i32` type ki values rakhne ke liye ek naya, empty vector create karna">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-01/src/main.rs:here}}
```

</Listing>

Note karein ke hum ne yahan type annotation add ki hai. Kyun ke hum is vector mein koi
values insert nahi kar rahe, Rust ko nahi pata ke hum kis type ke elements store karna
chahte hain. Ye ek important point hai. Vectors generics ko use karke implement kiye jate
hain; hum Chapter 10 mein dekhenge ke apne types ke saath generics ko kaise use kiya jata
hai. Filhal, itna jaan lein ke standard library ki provide ki hui `Vec<T>` type kisi bhi
type ko hold kar sakti hai. Jab hum kisi specific type ko hold karne ke liye vector create
karte hain, to hum angle brackets ke andar type specify kar sakte hain. Listing 8-1 mein
hum ne Rust ko bataya hai ke `v` mein mojood `Vec<T>` `i32` type ke elements hold karega.

Zyada tar waqt, aap initial values ke saath `Vec<T>` create karenge, aur Rust us value ka
type infer kar lega jise aap store karna chahte hain, is liye aapko ye type annotation
bohat kam hi karni padegi. Rust sahulat ke liye `vec!` macro provide karta hai, jo aapki
di hui values ko hold karne wala ek naya vector create karega. Listing 8-2 mein ek naya
`Vec<i32>` create kiya gaya hai jo `1`, `2`, aur `3` values ko hold karta hai. Integer type
`i32` hai kyun ke ye default integer type hai, jaisa ke hum ne Chapter 3 ke [“Data
Types”][data-types]<!-- ignore --> section mein discuss kiya tha.

<Listing number="8-2" caption="Values par mushtamil ek naya vector create karna">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-02/src/main.rs:here}}
```

</Listing>

Kyun ke hum ne initial `i32` values di hain, Rust infer kar sakta hai ke `v` ki type
`Vec<i32>` hai, aur type annotation ki zaroorat nahi hai. Agley section mein, hum dekhenge
ke vector ko kaise modify kiya jata hai.


### Vector Ko Update Karna

Ek vector create karne ke baad us mein elements add karne ke liye hum `push` method
use kar sakte hain, jaisa ke Listing 8-3 mein dikhaya gaya hai.

<Listing number="8-3" caption="Vector mein values add karne ke liye `push` method use karna">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-03/src/main.rs:here}}
```

</Listing>

Jaisa ke kisi bhi variable ke saath hota hai, agar hum uski value ko change karna
chahte hain, to humein `mut` keyword use karke usay mutable banana hota hai, jaisa
ke Chapter 3 mein discuss kiya gaya hai. Andar rakhe gaye numbers sab `i32` type ke
hain, aur Rust data se is type ko infer kar leta hai, is liye humein `Vec<i32>`
annotation ki zaroorat nahi hai.

### Vectors Ke Elements Read Karna

Vector mein store ki gayi value ko reference karne ke do tareeqe hain: indexing ke zariye ya `get` method ko use karke. Neeche diye gaye examples mein, zyada clarity ke liye hum ne in functions se return hone wali values ki types bhi annotate ki hain.

Listing 8-4 vector mein kisi value ko access karne ke dono methods dikhati hai, indexing syntax aur `get` method ke saath.

<Listing number="8-4" caption="Vector mein kisi item ko access karne ke liye indexing syntax aur `get` method use karna">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-04/src/main.rs:here}}
```

</Listing>

Yahan kuch details note karein. Hum third element hasil karne ke liye index value `2` use karte hain kyun ke vectors ki indexing zero se shuru hoti hai. `&` aur `[]` use karne se humein diye gaye index value par element ka reference milta hai. Jab hum `get` method ko argument ke taur par index pass karke use karte hain, to humein ek `Option<&T>` milta hai jise hum `match` ke saath use kar sakte hain.

Rust kisi element ko reference karne ke ye dono tareeqe provide karta hai taa-ke aap ye choose kar saken ke jab aap existing elements ki range se bahar ki index value use karne ki koshish karein to program kis tarah behave kare. Misal ke taur par, aaiye dekhein ke kya hota hai jab hamare paas paanch elements wala vector ho aur phir hum har technique ke zariye index `100` par mojood element ko access karne ki koshish karein, jaisa ke Listing 8-5 mein dikhaya gaya hai.

<Listing number="8-5" caption="Paanch elements wale vector mein index 100 par element ko access karne ki koshish karna">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-05/src/main.rs:here}}
```

</Listing>

Jab hum is code ko run karte hain, to pehla `[]` method program ko panic karwa dega kyun ke ye ek aise element ko reference karta hai jo mojood nahi hai. Ye method us waqt behtar use hota hai jab aap chahte hain ke vector ke end se aage kisi element ko access karne ki koshish hone par aapka program crash ho jaye.

Jab `get` method ko aisi index pass ki jati hai jo vector ke bahar ho, to ye panic kiye baghair `None` return karta hai. Aap is method ko us waqt use karenge jab normal circumstances mein kabhi kabhar vector ki range se bahar kisi element ko access karna mumkin ho. Phir aapke code mein `Some(&element)` ya `None` mein se kisi ek ko handle karne ke liye logic hoga, jaisa ke Chapter 6 mein discuss kiya gaya hai. Misal ke taur par, index kisi person ke number enter karne se aa sakti hai. Agar woh ghalti se aisa number enter kare jo bohat bara ho aur program ko `None` value mile, to aap user ko bata sakte hain ke current vector mein kitne items hain aur unhein valid value enter karne ka ek aur mauqa de sakte hain. Ye typo ki wajah se program crash karne ke muqable mein user ke liye zyada friendly hoga!

Jab program ke paas ek valid reference hota hai, to borrow checker ownership aur borrowing rules (jinhein Chapter 4 mein cover kiya gaya hai) enforce karta hai taa-ke ye ensure kiya ja sake ke ye reference aur vector ke contents ke doosre tamam references valid rahen. Woh rule yaad karein jo kehta hai ke aap ek hi scope mein mutable aur immutable references nahi rakh sakte. Ye rule Listing 8-6 par apply hota hai, jahan hum vector ke first element ka immutable reference hold karte hain aur end mein ek element add karne ki koshish karte hain. Agar hum function mein baad mein us element ko dobara reference karne ki koshish karein, to ye program kaam nahi karega.

<Listing number="8-6" caption="Kisi item ka reference hold karte hue vector mein ek element add karne ki koshish karna">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-06/src/main.rs:here}}
```

</Listing>

Is code ko compile karne par ye error milega:

```console
{{#include ../listings/ch08-common-collections/listing-08-06/output.txt}}
```

Listing 8-6 ka code dekh kar lag sakta hai ke isay kaam karna chahiye: Aakhir first element ke reference ko vector ke end mein hone wali changes se kya lena dena hai? Ye error vectors ke kaam karne ke tareeqe ki wajah se hota hai: Kyun ke vectors values ko memory mein ek doosre ke paas rakhte hain, vector ke end par ek naya element add karne ke liye nayi memory allocate karna aur purane elements ko nayi jagah par copy karna zaroori ho sakta hai, agar vector ki current stored location par tamam elements ko ek doosre ke paas rakhne ke liye kafi space na ho. Aisi surat mein, first element ka reference deallocated memory ko point kar raha hota. Borrowing rules programs ko is situation mein pahunchne se rokte hain.

> Note: `Vec<T>` type ki implementation details ke baare mein mazeed jaanne ke liye [“The
> Rustonomicon”][nomicon] dekhein.

### Vector Mein Mojood Values Par Iterate Karna

Vector ke har element ko bari bari access karne ke liye, hum indices ko ek ek karke access karne ke bajaye tamam elements par iterate karenge. Listing 8-7 dikhati hai ke `i32` values wale vector ke har element ka immutable reference hasil karne aur unhein print karne ke liye `for` loop ko kaise use kiya jata hai.

<Listing number="8-7" caption="`for` loop ke zariye elements par iterate karke vector ke har element ko print karna">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-07/src/main.rs:here}}
```

</Listing>

Hum mutable vector ke har element ke mutable references par bhi iterate kar sakte hain taa-ke tamam elements mein changes kiye ja saken. Listing 8-8 mein `for` loop har element mein `50` add karega.

<Listing number="8-8" caption="Vector ke elements ke mutable references par iterate karna">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-08/src/main.rs:here}}
```

</Listing>

Mutable reference jis value ko refer karta hai usay change karne ke liye, humein `+=` operator use karne se pehle `i` mein mojood value tak pohanchne ke liye `*` dereference operator use karna hota hai. Hum Chapter 15 ke [“Following the
Reference to the Value”][deref]<!-- ignore --> section mein dereference operator ke baare mein mazeed baat karenge.

Vector par iterate karna, chahe immutably ho ya mutably, borrow checker ke rules ki wajah se safe hai. Agar hum Listing 8-7 aur Listing 8-8 ke `for` loop bodies mein items insert ya remove karne ki koshish karein, to humein us compiler error jaisa error milega jo humein Listing 8-6 ke code ke saath mila tha. `for` loop ke paas vector ka jo reference hota hai, woh poore vector mein ek hi waqt mein modification hone se rokta hai.

### Multiple Types Store Karne Ke Liye Enum Use Karna

Vectors sirf un values ko store kar sakte hain jo same type ki hon. Ye inconvenient ho sakta hai; aise use cases bilkul maujood hain jahan different types ke items ki list store karna zaroori hota hai. Khush qismati se, enum ke variants ek hi enum type ke under define kiye jate hain, is liye jab humein different types ke elements ko represent karne ke liye ek type ki zaroorat ho, to hum ek enum define aur use kar sakte hain!

Misal ke taur par, maan lein ke hum spreadsheet ki kisi row se values hasil karna chahte hain jismein row ke kuch columns mein integers, kuch mein floating-point numbers, aur kuch mein strings hon. Hum ek enum define kar sakte hain jiske variants different value types ko hold karenge, aur tamam enum variants ko same type samjha jayega: yani enum ka type. Phir hum ek vector create kar sakte hain jo us enum ko hold kare aur is tarah, aakhirkar, different types ko hold kar sake. Hum ne Listing 8-9 mein iski example di hai.

<Listing number="8-9" caption="Ek vector mein different types ki values store karne ke liye enum define karna">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-09/src/main.rs:here}}
```

</Listing>

Rust ko compile time par ye pata hona zaroori hai ke vector mein kaun se types honge taa-ke woh exactly ye jaan sake ke har element ko store karne ke liye heap par kitni memory required hogi. Humein ye bhi explicitly batana hota hai ke is vector mein kaun se types allowed hain. Agar Rust kisi vector ko kisi bhi type ko hold karne ki ijazat deta, to mumkin tha ke ek ya zyada types vector ke elements par perform ki jane wali operations ke saath errors cause kar dete. Enum ke saath `match` expression use karne ka matlab hai ke Rust compile time par ensure karega ke har possible case handle kiya gaya hai, jaisa ke Chapter 6 mein discuss kiya gaya hai.

Agar aapko un types ka exhaustive set maloom nahi hai jo program runtime par vector mein store karne ke liye receive karega, to enum wali technique kaam nahi karegi. Is ke bajaye, aap trait object use kar sakte hain, jise hum Chapter 18 mein cover karenge.

Ab jab ke hum ne vectors ko use karne ke kuch sab se common tareeqon par baat kar li hai, to `Vec<T>` par standard library ki taraf se define kiye gaye tamam useful methods ke liye [API documentation][vec-api]<!-- ignore --> zaroor review karein. Misal ke taur par, `push` ke ilawa, ek `pop` method last element ko remove karke return karta hai.

### Vector Drop Hone Par Uske Elements Bhi Drop Ho Jate Hain

Kisi bhi doosre `struct` ki tarah, vector bhi us waqt free ho jata hai jab woh scope se bahar nikalta hai, jaisa ke Listing 8-10 mein annotate kiya gaya hai.

<Listing number="8-10" caption="Ye dikhana ke vector aur uske elements kahan drop hote hain">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-10/src/main.rs:here}}
```

</Listing>

Jab vector drop hota hai, to uske tamam contents bhi drop ho jate hain, yani us mein store kiye gaye integers bhi clean up ho jate hain. Borrow checker ensure karta hai ke vector ke contents ke kisi bhi reference ko sirf us waqt use kiya jaye jab vector khud valid ho.

Aaiye ab next collection type ki taraf chalte hain: `String`!

[data-types]: ch03-02-data-types.html#data-types
[nomicon]: ../nomicon/vec/vec.html
[vec-api]: ../std/vec/struct.Vec.html
[deref]: ch15-02-deref.html#following-the-pointer-to-the-value-with-the-dereference-operator
