<!-- Old headings. Do not remove or links may break. -->

<a id="treating-smart-pointers-like-regular-references-with-the-deref-trait"></a> <a id="treating-smart-pointers-like-regular-references-with-deref"></a>

## Smart Pointers ko Regular References ki Tarah Treat Karna

`Deref` trait ko implement karne se aap *dereference operator* `*` ke behavior
ko customize kar sakte hain (ise multiplication ya glob operator ke saath
confuse na karein). `Deref` ko is tarah implement karke ke smart pointer ko
regular reference ki tarah treat kiya ja sake, aap aisa code likh sakte hain jo
references par operate karta ho aur us code ko smart pointers ke saath bhi use
kar sakte hain.

Sab se pehle dekhte hain ke dereference operator regular references ke saath
kaise kaam karta hai. Phir hum ek custom type define karne ki koshish karenge
jo `Box<T>` ki tarah behave kare aur dekhenge ke dereference operator hamare
newly defined type par reference ki tarah kyun kaam nahi karta. Hum explore
karenge ke `Deref` trait ko implement karna kaise smart pointers ke liye
references ki tarah similar tareeqon se kaam karna mumkin banata hai. Phir hum
Rust ke deref coercion feature ko dekhenge aur yeh dekhenge ke yeh humein
references ya smart pointers, dono ke saath kaam karne ki kaise ijazat deta hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="following-the-pointer-to-the-value-with-the-dereference-operator"></a> <a id="following-the-pointer-to-the-value"></a>

### Reference ko Value Tak Follow Karna

Ek regular reference pointer ki ek type hai, aur pointer ko samajhne ka ek tareeqa
yeh hai ke use kisi doosri jagah stored value ki taraf ishara karne wale arrow ke
taur par dekha jaye. Listing 15-6 mein hum ek `i32` value ka reference create
karte hain aur phir reference ko value tak follow karne ke liye dereference
operator use karte hain.

<Listing number="15-6" file-name="src/main.rs" caption="Dereference operator ko use karke ek `i32` value ke reference ko follow karna">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-06/src/main.rs}}
```

</Listing>

Variable `x` ek `i32` value `5` hold karta hai. Hum `y` ko `x` ke reference ke
barabar set karte hain. Hum assert kar sakte hain ke `x` `5` ke barabar hai.
Lekin agar hum `y` mein mojood value ke baare mein assertion karna chahte hain,
to humein reference ko us value tak follow karne ke liye `*y` use karna hoga
jis ki taraf woh point kar raha hai (isi liye ise *dereference* kaha jata hai),
taake compiler actual value ka comparison kar sake. Ek baar `y` ko dereference
karne ke baad, humein us integer value tak access mil jata hai jis ki taraf `y`
point kar raha hai, aur hum uska `5` ke saath comparison kar sakte hain.

Agar hum iske bajaye `assert_eq!(5, y);` likhne ki koshish karein, to humein yeh
compilation error milega:

```console
{{#include ../listings/ch15-smart-pointers/output-only-01-comparing-to-reference/output.txt}}
```

Ek number aur ek number ke reference ka comparison allowed nahi hai kyun ke yeh
different types hain. Humein reference ko us value tak follow karne ke liye
dereference operator use karna hoga jis ki taraf woh point kar raha hai.

### `Box<T>` ko Reference ki Tarah Use Karna

Hum Listing 15-6 ke code ko reference ke bajaye `Box<T>` use karne ke liye
rewrite kar sakte hain; Listing 15-7 mein `Box<T>` par use kiya gaya
dereference operator usi tarah kaam karta hai jaise Listing 15-6 mein reference
par use kiya gaya dereference operator.

<Listing number="15-7" file-name="src/main.rs" caption="`Box<i32>` par dereference operator use karna">

```rust id="w9b3eu"
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-07/src/main.rs}}
```

</Listing>

Listing 15-7 aur Listing 15-6 ke darmiyan main difference yeh hai ke yahan hum
`y` ko ek aise box ka instance banate hain jo `x` ki copied value ki taraf point
karta hai, bajaye iske ke `y` ko `x` ki value ke reference ki taraf point karne
wala banaya jaye. Aakhri assertion mein, hum box ke pointer ko follow karne ke
liye dereference operator ko usi tarah use kar sakte hain jis tarah humne `y`
ke reference hone par kiya tha. Ab hum explore karenge ke `Box<T>` mein aisi
kya khaas baat hai jo humein dereference operator use karne ki ijazat deti hai,
aur iske liye hum apni khud ki box type define karenge.

### Apna Khud ka Smart Pointer Define Karna

Aaiye `Box<T>` type ke jaisa ek wrapper type build karte hain jo standard
library provide karti hai, taake hum experience kar saken ke smart pointer types
default taur par references se kis tarah mukhtalif behave karti hain. Phir hum
dekhenge ke dereference operator ko use karne ki ability kaise add ki jati hai.

> Note: `MyBox<T>` type jo hum abhi build karne wale hain aur asli `Box<T>` ke
> darmiyan ek bara difference hai: Hamara version apna data heap par store nahi
> karega. Hum is example mein `Deref` par focus kar rahe hain, is liye data
> asal mein kahan stored hai, yeh pointer-like behavior ke muqable mein kam
> important hai.

`Box<T>` type aakhir mein ek element wali tuple struct ke taur par define hoti
hai, is liye Listing 15-8 bhi isi tarah ek `MyBox<T>` type define karti hai.
Hum `Box<T>` par defined `new` function ke mutabiq ek `new` function bhi define
karenge.

<Listing number="15-8" file-name="src/main.rs" caption="Ek `MyBox<T>` type define karna">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-08/src/main.rs:here}}
```

</Listing>

Hum `MyBox` naam ka ek struct define karte hain aur ek generic parameter `T`
declare karte hain kyun ke hum chahte hain ke hamari type kisi bhi type ki
values hold kar sake. `MyBox` type ek tuple struct hai jisme `T` type ka ek
element hai. `MyBox::new` function `T` type ka ek parameter leti hai aur ek
`MyBox` instance return karti hai jo pass ki gayi value ko hold karta hai.

Aaiye Listing 15-7 ke `main` function ko Listing 15-8 mein add karte hain aur
use change karke `Box<T>` ke bajaye woh `MyBox<T>` type use karte hain jo humne
define ki hai. Listing 15-9 ka code compile nahi hoga, kyun ke Rust ko nahi
pata ke `MyBox` ko dereference kaise karna hai.

<Listing number="15-9" file-name="src/main.rs" caption="`MyBox<T>` ko usi tarah use karne ki koshish karna jis tarah humne references aur `Box<T>` ko use kiya">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-09/src/main.rs:here}}
```

</Listing>

Yahan resultant compilation error hai:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-09/output.txt}}
```

Hamari `MyBox<T>` type ko dereference nahi kiya ja sakta kyun ke humne apni
type par yeh ability implement nahi ki. `*` operator ke saath dereferencing
enable karne ke liye hum `Deref` trait implement karte hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="treating-a-type-like-a-reference-by-implementing-the-deref-trait"></a>

### `Deref` Trait ko Implement Karna

Jaisa ke Chapter 10 mein [“Implementing a Trait on a Type”][impl-trait]<!-- ignore --> mein
discuss kiya gaya hai, kisi trait ko implement karne ke liye humein us trait ke
required methods ki implementations provide karni hoti hain. Standard library
ki taraf se provide kiya gaya `Deref` trait humse `deref` naam ka ek method
implement karne ka taqaza karta hai jo `self` ko borrow karta hai aur inner data
ka ek reference return karta hai. Listing 15-10 mein `MyBox<T>` ki definition
mein add karne ke liye `Deref` ki implementation di gayi hai.

<Listing number="15-10" file-name="src/main.rs" caption="`MyBox<T>` par `Deref` ko implement karna">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-10/src/main.rs:here}}
```

</Listing>

`type Target = T;` syntax `Deref` trait ke liye use hone wali ek associated type
define karti hai. Associated types generic parameter declare karne ka thora
different tareeqa hain, lekin filhaal aapko unke baare mein fikr karne ki
zaroorat nahi; hum Chapter 20 mein unhein zyada detail mein cover karenge.

Hum `deref` method ke body ko `&self.0` se fill karte hain taake `deref` us value
ka reference return kare jise hum `*` operator ke saath access karna chahte hain;
Chapter 5 mein [“Creating Different Types with Tuple Structs”][tuple-structs]<!--
ignore --> se yaad karein ke `.0` tuple struct mein pehli value ko access karta
hai. Listing 15-9 ka `main` function jo `MyBox<T>` value par `*` call karta hai
ab compile ho jata hai, aur assertions pass ho jati hain!

`Deref` trait ke baghair, compiler sirf `&` references ko dereference kar sakta
hai. `deref` method compiler ko yeh ability deta hai ke woh kisi bhi aisi type
ki value le sake jo `Deref` implement karti ho aur reference hasil karne ke liye
`deref` method ko call kar sake, jise woh dereference karna jaanta hai.

Jab humne Listing 15-9 mein `*y` likha, to background mein Rust ne asal mein
yeh code run kiya:

```rust,ignore
*(y.deref())
```

Rust `*` operator ko `deref` method ki call se replace karta hai aur phir ek
plain dereference karta hai, taake humein yeh sochna na pade ke humein `deref`
method ko call karne ki zaroorat hai ya nahi. Rust ka yeh feature humein aisa
code likhne deta hai jo bilkul ek jaisa function karta hai, chahe hamare paas
ek regular reference ho ya koi aisi type jo `Deref` implement karti ho.

`deref` method kisi value ka reference return kyun karta hai, aur
`*(y.deref())` mein parentheses ke bahar wala plain dereference ab bhi kyun
zaroori hai, iska taalluq ownership system se hai. Agar `deref` method value ka
reference return karne ke bajaye directly value return karta, to value `self`
se move ho jati. Is case mein, ya zyada tar un cases mein jahan hum
dereference operator use karte hain, hum `MyBox<T>` ke andar mojood value ki
ownership nahi lena chahte.

Note karein ke `*` operator ko `deref` method ki call aur phir `*` operator ki
call se sirf ek baar replace kiya jata hai, har baar jab hum apne code mein `*`
use karte hain. Kyun ke `*` operator ki substitution infinitely recurse nahi
karti, is liye aakhir mein humein `i32` type ka data milta hai, jo Listing 15-9
mein `assert_eq!` ke andar `5` se match karta hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="implicit-deref-coercions-with-functions-and-methods"></a> <a id="using-deref-coercions-in-functions-and-methods"></a>

### Functions aur Methods mein Deref Coercion Use Karna

*Deref coercion* ek aise type ke reference ko, jo `Deref` trait implement karta
hai, kisi doosre type ke reference mein convert karta hai. Misal ke taur par,
deref coercion `&String` ko `&str` mein convert kar sakta hai kyun ke `String`
`Deref` trait ko is tarah implement karta hai ke woh `&str` return karta hai.
Deref coercion ek convenience hai jo Rust functions aur methods ke arguments par
perform karta hai, aur yeh sirf un types par kaam karta hai jo `Deref` trait
implement karti hain. Yeh automatically us waqt hota hai jab hum kisi
particular type ki value ka reference kisi aise function ya method ko argument
ke taur par pass karte hain jo function ya method definition mein diye gaye
parameter type se match nahi karta. `deref` method ki calls ki ek sequence
hamare provide kiye gaye type ko us type mein convert karti hai jiski parameter
ko zaroorat hoti hai.

Deref coercion ko Rust mein is liye add kiya gaya tha taake functions aur
methods likhne wale programmers ko `&` aur `*` ke saath itne explicit references
aur dereferences add na karne padein. Deref coercion feature humein aisa zyada
code likhne ki bhi ijazat deta hai jo references ya smart pointers, dono ke liye
kaam kar sakta hai.

Deref coercion ko action mein dekhne ke liye, aaiye Listing 15-8 mein define
kiye gaye `MyBox<T>` type ke saath-saath Listing 15-10 mein add ki gayi `Deref`
ki implementation ko use karte hain. Listing 15-11 ek aise function ki
definition dikhati hai jiska parameter string slice hai.

<Listing number="15-11" file-name="src/main.rs" caption="Ek `hello` function jiska `name` parameter `&str` type ka hai">

```rust id="k4z7xq"
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-11/src/main.rs:here}}
```

</Listing>

Hum `hello` function ko argument ke taur par string slice ke saath call kar
sakte hain, jaise misal ke taur par `hello("Rust");`. Deref coercion ki wajah
se `hello` ko `MyBox<String>` type ki value ke reference ke saath call karna
bhi mumkin hai, jaisa ke Listing 15-12 mein dikhaya gaya hai.

<Listing number="15-12" file-name="src/main.rs" caption="`MyBox<String>` value ke reference ke saath `hello` ko call karna, jo deref coercion ki wajah se kaam karta hai">

```rust id="m9j5cv"
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-12/src/main.rs:here}}
```

</Listing>

Yahan hum `hello` function ko `&m` argument ke saath call kar rahe hain, jo
`MyBox<String>` value ka reference hai. Kyun ke humne Listing 15-10 mein
`MyBox<T>` par `Deref` trait implement kiya hai, Rust `deref` ko call karke
`&MyBox<String>` ko `&String` mein convert kar sakta hai. Standard library
`String` par `Deref` ki implementation provide karti hai jo string slice return
karti hai, aur yeh `Deref` ki API documentation mein diya gaya hai. Rust
`&String` ko `&str` mein convert karne ke liye dobara `deref` call karta hai,
jo `hello` function ki definition se match karta hai.

Agar Rust deref coercion implement na karta, to `&MyBox<String>` type ki value
ke saath `hello` ko call karne ke liye humein Listing 15-12 ke code ke bajaye
Listing 15-13 mein diya gaya code likhna padta.

<Listing number="15-13" file-name="src/main.rs" caption="Agar Rust mein deref coercion na hota to humein yeh code likhna padta">

```rust id="9x3fhm"
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-13/src/main.rs:here}}
```

</Listing>

`(*m)` `MyBox<String>` ko dereference karke `String` mein convert karta hai.
Phir `&` aur `[..]` `String ka ek string slice lete hain jo poori string ke
barabar hota hai, taake `hello` ke signature se match kiya ja sake. Deref
coercions ke baghair yeh code in tamam symbols ki wajah se parhna, likhna aur
samajhna zyada mushkil hai. Deref coercion Rust ko yeh conversions hamare liye
automatically handle karne deta hai.

Jab involved types ke liye `Deref` trait defined ho, Rust types ka analysis
karega aur `Deref::deref` ko jitni baar zaroori ho utni baar use karega taake
parameter ke type se match karta hua reference hasil ho sake.
`Deref::deref` ko kitni baar insert karna hai, yeh compile time par resolve ho
jata hai, is liye deref coercion ka faida uthane ki koi runtime penalty nahi hoti!

<!-- Old headings. Do not remove or links may break. -->

<a id="how-deref-coercion-interacts-with-mutability"></a>

### Mutable References ke Saath Deref Coercion Handle Karna

Jis tarah aap immutable references par `*` operator ko override karne ke liye
`Deref` trait use karte hain, usi tarah mutable references par `*` operator ko
override karne ke liye `DerefMut` trait use kar sakte hain.

Rust deref coercion tab karta hai jab use teen cases mein types aur trait
implementations milti hain:

1. `&T` se `&U` jab `T: Deref<Target=U>`
2. `&mut T` se `&mut U` jab `T: DerefMut<Target=U>`
3. `&mut T` se `&U` jab `T: Deref<Target=U>`

Pehle do cases ek jaise hain, siwaye iske ke doosre case mein mutability
implement hoti hai. Pehla case kehta hai ke agar aapke paas `&T` hai, aur `T`
kisi type `U` ke liye `Deref` implement karta hai, to aap transparently `&U`
hasil kar sakte hain. Doosra case kehta hai ke mutable references ke liye bhi
yahi deref coercion hoti hai.

Teesra case thora zyada tricky hai: Rust mutable reference ko immutable
reference mein bhi coerce karega. Lekin iska reverse *possible* nahi hai:
Immutable references kabhi mutable references mein coerce nahi hongi. Borrowing
rules ki wajah se, agar aapke paas mutable reference hai, to woh mutable
reference us data ka ek hi reference hona chahiye (warna program compile nahi
hoga). Ek mutable reference ko ek immutable reference mein convert karna
borrowing rules ko kabhi break nahi karega. Lekin ek immutable reference ko
mutable reference mein convert karne ke liye zaroori hoga ke initial immutable
reference hi us data ka eklauta immutable reference ho, lekin borrowing rules
is baat ki guarantee nahi dete. Is liye Rust yeh assumption nahi kar sakta ke
immutable reference ko mutable reference mein convert karna possible hai.

[impl-trait]: ch10-02-traits.html#implementing-a-trait-on-a-type
[tuple-structs]: ch05-01-defining-structs.html#creating-different-types-with-tuple-structs
