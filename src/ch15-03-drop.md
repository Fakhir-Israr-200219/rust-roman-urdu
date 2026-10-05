## Cleanup ke Waqt Code Run Karna with the `Drop` Trait

Smart pointer pattern ke liye doosra important trait `Drop` hai, jo aapko yeh
customize karne deta hai ke jab koi value scope se bahar jane wali ho to kya
hota hai. Aap kisi bhi type par `Drop` trait ki implementation provide kar sakte
hain, aur us code ko files ya network connections jaise resources release
karne ke liye use kiya ja sakta hai.

Hum `Drop` ko smart pointers ke context mein introduce kar rahe hain kyun ke
`Drop` trait ki functionality almost hamesha smart pointer implement karte waqt
use hoti hai. Misal ke taur par, jab `Box<T>` drop hota hai, to woh heap par
us space ko deallocate kar deta hai jis ki taraf box point karta hai.

Kuch languages mein, kuch types ke liye programmer ko har baar un types ke
instance ka use khatam karne par memory ya resources free karne ke liye code
call karna padta hai. Examples mein file handles, sockets aur locks shamil hain.
Agar programmer bhool jaye, to system overloaded ho sakta hai aur crash kar
sakta hai. Rust mein aap specify kar sakte hain ke jab bhi koi value scope se
bahar jaye to particular code ka ek hissa run ho, aur compiler is code ko
automatically insert kar dega. Iske result mein, aapko program mein har jagah
cleanup code place karne ke baare mein careful hone ki zaroorat nahi hoti jahan
kisi particular type ke instance ka kaam khatam hota hai—phir bhi resources
leak nahi honge!

Jab koi value scope se bahar jaye to run hone wala code aap `Drop` trait
implement karke specify karte hain. `Drop` trait ke liye aapko `drop` naam ka
ek method implement karna hota hai jo `self` ka mutable reference leta hai.
Yeh dekhne ke liye ke Rust `drop` ko kab call karta hai, filhaal `println!`
statements ke saath `drop` implement karte hain.

Listing 15-14 mein ek `CustomSmartPointer` struct hai jiski sirf custom
functionality yeh hai ke jab iska instance scope se bahar jayega to yeh
`Dropping CustomSmartPointer!` print karega, taake yeh dikhaya ja sake ke Rust
`drop` method ko kab run karta hai.

<Listing number="15-14" file-name="src/main.rs" caption="Ek `CustomSmartPointer` struct jo `Drop` trait implement karta hai, jahan hum apna cleanup code rakhenge">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-14/src/main.rs}}
```

</Listing>

`Drop` trait prelude mein included hai, is liye humein ise scope mein lane ki
zaroorat nahi hai. Hum `CustomSmartPointer` par `Drop` trait implement karte
hain aur `drop` method ke liye ek implementation provide karte hain jo
`println!` call karti hai. `drop` method ki body mein aap woh logic rakh sakte
hain jo aap chahte hain ke aapke type ka instance scope se bahar jane par run
ho. Yahan hum Rust ke `drop` call karne ka process visually demonstrate karne
ke liye kuch text print kar rahe hain.

`main` mein hum `CustomSmartPointer` ke do instances create karte hain aur phir
`CustomSmartPointers created` print karte hain. `main` ke end par hamare
`CustomSmartPointer` instances scope se bahar chale jayenge, aur Rust `drop`
method mein rakhe gaye code ko call karega, jo hamara final message print karega.
Note karein ke humein `drop` method ko explicitly call karne ki zaroorat nahi
padi.

Jab hum is program ko run karenge, to humein following output nazar aayega:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-14/output.txt}}
```

Rust ne automatically hamare liye `drop` call kiya jab hamare instances scope
se bahar chale gaye, aur woh code call kiya jo humne specify kiya tha. Variables
unki creation ke reverse order mein drop hote hain, is liye `d`, `c` se pehle
drop hua. Is example ka maqsad aapko yeh visually samjhana hai ke `drop` method
kaise kaam karta hai; aam taur par aap print message ke bajaye woh cleanup code
specify karenge jo aapke type ko run karna hota hai.

<!-- Old headings. Do not remove or links may break. -->
<a id="dropping-a-value-early-with-std-mem-drop"></a>

Unfortunately, automatic `drop` functionality ko disable karna
straightforward nahi hai. `drop` ko disable karna aam tor par zaroori nahi
hota; `Drop` trait ka poora point hi yeh hai ke yeh automatically handle hota
hai. Lekin kabhi kabhi aap kisi value ko jaldi clean up karna chah sakte hain.
Ek example un smart pointers ka hai jo locks ko manage karte hain: Aap
`drop` method ko force karna chah sakte hain jo lock ko release karta hai, taake
usi scope mein doosra code lock acquire kar sake. Rust aapko `Drop` trait ke
`drop` method ko manually call karne ki ijazat nahi deta; iske bajaye, agar aap
kisi value ko uske scope ke end se pehle drop karwana chahte hain, to aapko
standard library ka provided `std::mem::drop` function call karna hota hai.

Listing 15-14 ke `main` function ko modify karke `Drop` trait ke `drop` method
ko manually call karne ki koshish kaam nahi karegi, jaisa ke Listing 15-15 mein
dikhaya gaya hai.

<Listing number="15-15" file-name="src/main.rs" caption="Jaldi cleanup karne ke liye `Drop` trait ke `drop` method ko manually call karne ki koshish">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-15/src/main.rs:here}}
```

</Listing>

Jab hum is code ko compile karne ki koshish karenge, to humein yeh error
milega:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-15/output.txt}}
```

Yeh error message batata hai ke humein `drop` ko explicitly call karne ki
ijazat nahi hai. Error message *destructor* ki term use karta hai, jo us
function ke liye general programming term hai jo kisi instance ko clean up
karta hai. Ek *destructor*, ek *constructor* ke analogous hota hai, jo ek
instance create karta hai. Rust mein `drop` function ek particular destructor
hai.

Rust humein `drop` ko explicitly call karne nahi deta, kyun ke Rust phir bhi
`main` ke end par value par automatically `drop` call karega. Is se double
free error hoga kyun ke Rust ek hi value ko do baar clean up karne ki koshish
karega.

Hum `drop` ki automatic insertion ko disable nahi kar sakte jab koi value scope
se bahar jati hai, aur hum `drop` method ko explicitly call bhi nahi kar sakte.
Is liye, agar humein kisi value ko jaldi clean up karne ke liye force karna ho,
to hum `std::mem::drop` function use karte hain.

`std::mem::drop` function, `Drop` trait ke `drop` method se different hai. Hum
ise us value ko argument ke taur par pass karke call karte hain jise hum
force-drop karna chahte hain. Yeh function prelude mein hai, is liye hum
Listing 15-15 mein `main` ko modify karke `drop` function call kar sakte hain,
jaisa ke Listing 15-16 mein dikhaya gaya hai.

<Listing number="15-16" file-name="src/main.rs" caption="Kisi value ke scope se bahar jane se pehle use explicitly drop karne ke liye `std::mem::drop` call karna">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-16/src/main.rs:here}}
```

</Listing>

Is code ko run karne par following print hoga:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-16/output.txt}}
```

Text ``Dropping CustomSmartPointer with data `some data`!`` ko
`CustomSmartPointer created` aur `CustomSmartPointer dropped before the end
of main` text ke darmiyan print kiya jata hai, jo dikhata hai ke `drop` method
ka code us point par `c` ko drop karne ke liye call hota hai.

Aap `Drop` trait implementation mein specified code ko cleanup ko convenient
aur safe banane ke liye kai tarah se use kar sakte hain: Misal ke taur par, aap
ise apna memory allocator create karne ke liye use kar sakte hain! `Drop` trait
aur Rust ke ownership system ke saath, aapko cleanup yaad rakhne ki zaroorat
nahi hoti, kyun ke Rust ise automatically karta hai.

Aapko un problems ke baare mein bhi fikr karne ki zaroorat nahi hoti jo
accidentally un values ko clean up karne se result ho sakti hain jo abhi use
mein hain: Ownership system jo ensure karta hai ke references hamesha valid
hon, woh yeh bhi ensure karta hai ke `drop` sirf ek baar call ho jab value
use nahi ki ja rahi hoti.

Ab jab hum `Box<T>` aur smart pointers ki kuch characteristics examine kar
chuke hain, to aaiye standard library mein defined kuch aur smart pointers ko
dekhte hain.
