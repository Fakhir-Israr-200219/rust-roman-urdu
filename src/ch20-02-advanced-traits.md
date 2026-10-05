## Advanced Traits

Humne sab se pehle Chapter 10 ke [“Defining Shared Behavior with
Traits”][traits]<!-- ignore --> section mein traits cover kiye thay, lekin humne
un ki zyada advanced details discuss nahi ki thin. Ab jab aap Rust ke bare mein
zyada jaante hain, hum in ki bareek details mein ja sakte hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="specifying-placeholder-types-in-trait-definitions-with-associated-types"></a> <a id="associated-types"></a>

### Defining Traits with Associated Types

*Associated types* ek type placeholder ko ek trait ke saath connect karte hain,
taa-ke trait method definitions apni signatures mein in placeholder types ko
use kar saken. Trait ka implementor particular implementation ke liye
placeholder type ki jagah use hone wali concrete type specify karega. Is tarah,
hum ek aisa trait define kar sakte hain jo kuch types ko use karta ho, baghair
yeh jaane ke ke woh types exactly kya hain, jab tak trait implement na ho.

Humne is chapter mein zyada tar advanced features ko aise features ke taur par
describe kiya hai jin ki rarely zaroorat hoti hai. Associated types kahin beech
mein hain: Yeh book ke baqi hisson mein explain kiye gaye features ke muqable
mein kam use hote hain, lekin is chapter mein discuss kiye gaye kai doosre
features ke muqable mein zyada commonly use hote hain.

Associated type wale trait ki ek example standard library ka diya hua
`Iterator` trait hai. Associated type ka naam `Item` hai aur yeh un values ki
type ke liye stand in karta hai jin par `Iterator` trait implement karne wali
type iterate kar rahi hoti hai. `Iterator` trait ki definition Listing 20-13 mein
dikhayi gayi hai.

<Listing number="20-13" caption="The definition of the `Iterator` trait that has an associated type `Item`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-13/src/lib.rs}}
```

</Listing>

Type `Item` ek placeholder hai, aur `next` method ki definition dikhati hai ke
yeh `Option<Self::Item>` type ki values return karega. `Iterator` trait ke
implementors `Item` ke liye concrete type specify karenge, aur `next` method
ek `Option` return karega jismein us concrete type ki value hogi.

Associated types ka concept generics ke similar lag sakta hai, kyun ke generics
humein yeh specify kiye baghair function define karne dete hain ke woh kin
types ko handle kar sakta hai. Dono concepts ke darmiyan difference ko examine
karne ke liye, hum `Counter` naam ki type par `Iterator` trait ki ek
implementation dekhenge jo specify karti hai ke `Item` type `u32` hai:

<Listing file-name="src/lib.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-22-iterator-on-counter/src/lib.rs:ch19}}
```

</Listing>

Yeh syntax generics ke syntax ke comparable lagti hai. To phir, hum `Iterator`
trait ko generics ke saath kyun na define karein, jaisa ke Listing 20-14 mein
dikhaya gaya hai?

<Listing number="20-14" caption="A hypothetical definition of the `Iterator` trait using generics">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-14/src/lib.rs}}
```

</Listing>

Difference yeh hai ke jab hum generics use karte hain, jaisa ke Listing 20-14
mein hai, to humein har implementation mein types ko annotate karna padta hai;
kyun ke hum `Iterator<String> for Counter` ya kisi bhi doosri type ko bhi
implement kar sakte hain, is liye hamare paas `Counter` ke liye `Iterator` ki
multiple implementations ho sakti hain. Doosre alfaaz mein, jab kisi trait mein
generic parameter hota hai, to usay ek type ke liye multiple times implement
kiya ja sakta hai, aur har baar generic type parameters ki concrete types ko
change kiya ja sakta hai. Jab hum `Counter` par `next` method use karenge, to
humein type annotations provide karni hongi taa-ke indicate kar saken ke hum
`Iterator` ki kis implementation ko use karna chahte hain.

Associated types ke saath, humein types ko annotate karne ki zaroorat nahi hoti,
kyun ke hum kisi trait ko ek type par multiple times implement nahi kar sakte.
Listing 20-13 mein associated types use karne wali definition ke saath, hum
sirf ek baar choose kar sakte hain ke `Item` ki type kya hogi, kyun ke sirf ek
`impl Iterator for Counter` ho sakta hai. Jab bhi hum `Counter` par `next`
call karte hain, humein har jagah yeh specify karne ki zaroorat nahi hoti ke hum
`u32` values ka iterator chahte hain.

Associated types trait ke contract ka bhi hissa ban jate hain: Trait ke
implementors ko associated type placeholder ki jagah ek type provide karni
hoti hai. Associated types ka naam aksar is baat ko describe karta hai ke type
ko kaise use kiya jayega, aur API documentation mein associated type ko
document karna ek achhi practice hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="default-generic-type-parameters-and-operator-overloading"></a>

### Using Default Generic Parameters and Operator Overloading

Jab hum generic type parameters use karte hain, to hum generic type ke liye ek
default concrete type specify kar sakte hain. Is se trait ke implementors ke
liye concrete type specify karne ki zaroorat khatam ho jati hai agar default
type kaam karti ho. Aap generic type declare karte waqt
`<PlaceholderType=ConcreteType>` syntax ke zariye default type specify karte
hain.

Ek great example jahan yeh technique useful hai *operator overloading* mein,
jismein aap particular situations mein kisi operator (jaise `+`) ke behavior ko
customize karte hain.

Rust aapko apne operators create karne ya arbitrary operators ko overload karne
ki ijazat nahi deta. Lekin aap `std::ops` mein listed operations aur unke
corresponding traits ko operator se associated traits implement karke overload
kar sakte hain. Misal ke taur par, Listing 20-15 mein hum `+` operator ko
overload karte hain taa-ke do `Point` instances ko ek saath add kar saken. Hum
yeh `Point` struct par `Add` trait implement karke karte hain.

<Listing number="20-15" file-name="src/main.rs" caption="Implementing the `Add` trait to overload the `+` operator for `Point` instances">

```rust id="p2s6da"
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-15/src/main.rs}}
```

</Listing>

`add` method do `Point` instances ki `x` values aur do `Point` instances ki
`y` values ko add karke ek naya `Point` create karta hai. `Add` trait mein
`Output` naam ka ek associated type hai jo `add` method se return hone wali type
ko determine karta hai.

Is code mein default generic type `Add` trait ke andar hai. Yeh uski definition
hai:

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

Yeh code generally familiar lagna chahiye: ek method aur ek associated type wala
trait. Naya hissa `Rhs=Self` hai: Is syntax ko *default type parameters* kaha
jata hai. `Rhs` generic type parameter (jo “right-hand side” ka short form hai)
`add` method mein `rhs` parameter ki type define karta hai. Agar hum `Add` trait
implement karte waqt `Rhs` ke liye concrete type specify nahi karte, to `Rhs`
ki type default taur par `Self` ho jayegi, jo woh type hogi jis par hum `Add`
implement kar rahe hain.

Jab humne `Point` ke liye `Add` implement kiya, to humne `Rhs` ka default use
kiya kyun ke hum do `Point` instances ko add karna chahte thay. Ab ek aisi
example dekhte hain jahan hum `Add` trait ko implement karte waqt default use
karne ke bajaye `Rhs` type ko customize karna chahte hain.

Hamare paas do structs, `Millimeters` aur `Meters`, hain jo different units mein
values hold karte hain. Kisi existing type ko doosre struct mein is tarah thin
wrapping karna *newtype pattern* kehlata hai, jise hum [“Implementing
External Traits with the Newtype Pattern”][newtype]<!-- ignore --> section mein
mazeed detail mein describe karte hain. Hum millimeters mein values ko meters
mein values ke saath add karna chahte hain aur chahte hain ke `Add` ki
implementation conversion correctly kare. Hum `Millimeters` ke liye `Add` ko
`Meters` ko `Rhs` ke taur par use karke implement kar sakte hain, jaisa ke
Listing 20-16 mein dikhaya gaya hai.

<Listing number="20-16" file-name="src/lib.rs" caption="Implementing the `Add` trait on `Millimeters` to add `Millimeters` and `Meters`">

```rust,noplayground id="y9k2cn"
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-16/src/lib.rs}}
```

</Listing>

`Millimeters` aur `Meters` ko add karne ke liye, hum `Rhs` type parameter ki
value ko `Self` ke default ke bajaye set karne ke liye `impl Add<Meters>`
specify karte hain.

Aap default type parameters ko do main tareeqon se use karenge:

1. Kisi type ko existing code ko break kiye baghair extend karna
2. Specific cases mein customization allow karna jis ki zyada tar users ko zaroorat nahi hogi

Standard library ka `Add` trait doosre purpose ki ek example hai:
Aam taur par, aap same types ko add karenge, lekin `Add` trait is se aage
customize karne ki ability provide karta hai. `Add` trait ki definition mein
default type parameter use karne ka matlab hai ke zyada tar waqt aapko extra
parameter specify karne ki zaroorat nahi hoti. Doosre alfaaz mein, thori si
implementation boilerplate ki zaroorat nahi hoti, jis se trait ko use karna
aasaan ho jata hai.

Pehla purpose doosre ke similar hai lekin ulta: Agar aap kisi existing trait mein
ek type parameter add karna chahte hain, to aap usay ek default de sakte hain
taa-ke existing implementation code ko break kiye baghair trait ki
functionality ko extend kiya ja sake.

<!-- Old headings. Do not remove or links may break. -->

<a id="fully-qualified-syntax-for-disambiguation-calling-methods-with-the-same-name"></a> <a id="disambiguating-between-methods-with-the-same-name"></a>

### Disambiguating Between Identically Named Methods

Rust mein koi cheez kisi trait ko doosre trait ke method ke same name wala method rakhne se nahi rokti, aur na hi Rust aapko ek hi type par dono traits implement karne se rokta hai. Kisi type par directly bhi aisa method implement karna possible hai jiska name traits ke methods ke same ho.

Jab same name wale methods ko call kiya jaye, to aapko Rust ko batana hoga ke aap in mein se kis method ko use karna chahte hain. Listing 20-17 ke code par ghour karein jahan humne do traits, `Pilot` aur `Wizard`, define kiye hain, jin dono mein `fly` naam ka method hai. Phir hum dono traits ko ek `Human` type par implement karte hain jismein pehle se `fly` naam ka method directly implement kiya gaya hai. Har `fly` method kuch different karta hai.

<Listing number="20-17" file-name="src/main.rs" caption="Two traits are defined to have a `fly` method and are implemented on the `Human` type, and a `fly` method is implemented on `Human` directly.">

```rust id="v8y5k1"
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-17/src/main.rs:here}}
```

</Listing>

Jab hum `Human` ke ek instance par `fly` call karte hain, to compiler default taur par us method ko call karta hai jo directly type par implement kiya gaya hai, jaisa ke Listing 20-18 mein dikhaya gaya hai.

<Listing number="20-18" file-name="src/main.rs" caption="Calling `fly` on an instance of `Human`">

```rust id="j3q7nc"
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-18/src/main.rs:here}}
```

</Listing>

Is code ko run karne se `*waving arms furiously*` print hoga, jo dikhata hai ke Rust ne `Human` par directly implement kiye gaye `fly` method ko call kiya.

`Pilot` trait ya `Wizard` trait ke `fly` methods ko call karne ke liye, humein zyada explicit syntax use karni hogi taa-ke specify kar saken ke hum kis `fly` method ki baat kar rahe hain. Listing 20-19 is syntax ko demonstrate karti hai.

<Listing number="20-19" file-name="src/main.rs" caption="Specifying which trait’s `fly` method we want to call">

```rust id="z6w2mp"
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-19/src/main.rs:here}}
```

</Listing>

Method name se pehle trait name specify karne se Rust ke liye yeh clear ho jata hai ke hum `fly` ki kis implementation ko call karna chahte hain. Hum `Human::fly(&person)` bhi likh sakte hain, jo us `person.fly()` ke equivalent hai jo humne Listing 20-19 mein use kiya tha, lekin agar humein disambiguate karne ki zaroorat na ho to yeh likhne mein thora zyada lamba hai.

Is code ko run karne se yeh output print hoga:

```console id="4f8m1x"
{{#include ../listings/ch20-advanced-features/listing-20-19/output.txt}}
```

Kyun ke `fly` method `self` parameter leta hai, agar hamare paas do *types* hon jo dono ek *trait* ko implement karte hon, to Rust `self` ki type ki bunyaad par yeh pata laga sakta hai ke trait ki kis implementation ko use karna hai.

Lekin associated functions jo methods nahi hain unke paas `self` parameter nahi hota. Jab multiple types ya traits non-method functions ko same function name ke saath define karte hain, to Rust hamesha yeh nahi jaan pata ke aap kis type ki baat kar rahe hain jab tak aap fully qualified syntax use na karein. Misal ke taur par, Listing 20-20 mein hum ek animal shelter ke liye ek trait create karte hain jo tamam baby dogs ka naam Spot rakhna chahta hai. Hum `Animal` trait banate hain jismein `baby_name` naam ka ek associated non-method function hai. `Animal` trait ko `Dog` struct ke liye implement kiya gaya hai, aur hum `Dog` par directly bhi ek associated non-method function `baby_name` provide karte hain.

<Listing number="20-20" file-name="src/main.rs" caption="A trait with an associated function and a type with an associated function of the same name that also implements the trait">

```rust id="r1m6tb"
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-20/src/main.rs}}
```

</Listing>

Hum `Dog` par define kiye gaye `baby_name` associated function mein tamam puppies ka naam Spot rakhne wala code implement karte hain. `Dog` type `Animal` trait ko bhi implement karta hai, jo un characteristics ko describe karta hai jo tamam animals mein hoti hain. Baby dogs ko puppies kaha jata hai, aur yeh baat `Dog` par `Animal` trait ki implementation mein `Animal` trait se associated `baby_name` function ke andar express ki gayi hai.

`main` mein, hum `Dog::baby_name` function call karte hain, jo directly `Dog` par defined associated function ko call karta hai. Yeh code yeh output print karta hai:

```console id="k9x4wd"
{{#include ../listings/ch20-advanced-features/listing-20-20/output.txt}}
```

Yeh output woh nahi hai jo hum chahte thay. Hum `Animal` trait ka woh `baby_name` function call karna chahte hain jo humne `Dog` par implement kiya hai taa-ke code `A baby dog is called a puppy` print kare. Listing 20-19 mein jo trait name specify karne ki technique humne use ki thi, woh yahan madad nahi karti; agar hum `main` ko Listing 20-21 ke code mein change karein, to humein compilation error milega.

<Listing number="20-21" file-name="src/main.rs" caption="Attempting to call the `baby_name` function from the `Animal` trait, but Rust doesn’t know which implementation to use">

```rust,ignore,does_not_compile id="p0c7vn"
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-21/src/main.rs:here}}
```

</Listing>

Kyun ke `Animal::baby_name` mein `self` parameter nahi hai, aur aisi doosri types bhi ho sakti hain jo `Animal` trait ko implement karti hon, Rust yeh figure out nahi kar sakta ke hum `Animal::baby_name` ki kis implementation ko chahte hain. Humein yeh compiler error milega:

```console id="b5n2qh"
{{#include ../listings/ch20-advanced-features/listing-20-21/output.txt}}
```

Disambiguate karne aur Rust ko yeh batane ke liye ke hum `Dog` ke liye `Animal` ki implementation use karna chahte hain, na ke kisi doosri type ke liye `Animal` ki implementation, humein fully qualified syntax use karni hogi. Listing 20-22 demonstrate karti hai ke fully qualified syntax ko kaise use kiya jata hai.

<Listing number="20-22" file-name="src/main.rs" caption="Using fully qualified syntax to specify that we want to call the `baby_name` function from the `Animal` trait as implemented on `Dog`">

```rust id="s7h3qe"
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-22/src/main.rs:here}}
```

</Listing>

Hum angle brackets ke andar Rust ko ek type annotation provide kar rahe hain, jo indicate karti hai ke hum `Dog` par implement kiye gaye `Animal` trait ke `baby_name` method ko call karna chahte hain, yeh keh kar ke hum is function call ke liye `Dog` type ko `Animal` ke taur par treat karna chahte hain. Ab yeh code woh print karega jo hum chahte hain:

```console id="w2d6sf"
{{#include ../listings/ch20-advanced-features/listing-20-22/output.txt}}
```

Generally, fully qualified syntax is tarah define hoti hai:

```rust,ignore id="m4q8jc"
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

Associated functions ke liye jo methods nahi hain, koi `receiver` nahi hoga: Sirf doosre arguments ki list hogi. Aap fully qualified syntax ko har jagah use kar sakte hain jahan aap functions ya methods call karte hain. Lekin aapko is syntax ka koi bhi hissa omit karne ki ijazat hai jise Rust program ki doosri information se khud figure out kar sakta ho. Aapko sirf un cases mein is zyada verbose syntax ko use karne ki zaroorat hoti hai jahan multiple implementations same name use karti hain aur Rust ko yeh identify karne mein madad chahiye hoti hai ke aap kis implementation ko call karna chahte hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-supertraits-to-require-one-traits-functionality-within-another-trait"></a>

### Using Supertraits

Kabhi kabhi aap aisi trait definition likhenge jo kisi doosre trait par depend karti
hai: Kisi type ko pehle trait ko implement karne ke liye, aap chahte hain ke woh
type doosre trait ko bhi implement kare. Aap aisa is liye karenge taa-ke aapki
trait definition doosre trait ke associated items ko use kar sake. Jis trait par
aapki trait definition depend kar rahi hoti hai, usay aapke trait ka
*supertrait* kaha jata hai.

Misal ke taur par, maan lein ke hum `OutlinePrint` trait banana chahte hain
jismein ek `outline_print` method hoga jo di gayi value ko is tarah formatted
print karega ke woh asterisks ke frame mein ho. Yani, agar hamare paas ek
`Point` struct hai jo standard library ke `Display` trait ko implement karta hai
taa-ke result `(x, y)` ki form mein aaye, to jab hum `Point` ke aise instance par
`outline_print` call karein jismein `x` ke liye `1` aur `y` ke liye `3` ho, to
usay yeh print karna chahiye:

```text
**********
*        *
* (1, 3) *
*        *
**********
```

`outline_print` method ki implementation mein hum `Display` trait ki
functionality use karna chahte hain. Is liye, humein specify karna hoga ke
`OutlinePrint` trait sirf un types ke liye kaam karega jo `Display` ko bhi
implement karti hon aur woh functionality provide karti hon jis ki
`OutlinePrint` ko zaroorat hai. Hum trait definition mein `OutlinePrint: Display`
specify karke aisa kar sakte hain. Yeh technique trait mein trait bound add
karne ke similar hai. Listing 20-23 `OutlinePrint` trait ki implementation
dikhati hai.

<Listing number="20-23" file-name="src/main.rs" caption="Implementing the `OutlinePrint` trait that requires the functionality from `Display`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-23/src/main.rs:here}}
```

</Listing>

Kyun ke humne specify kiya hai ke `OutlinePrint` ko `Display` trait ki zaroorat
hai, hum `to_string` function use kar sakte hain jo `Display` implement karne
wali har type ke liye automatically implement hota hai. Agar hum trait name ke
baad colon add karke `Display` trait specify kiye baghair `to_string` use karne
ki koshish karte, to humein ek error milta jo kehta ke current scope mein type
`&Self` ke liye `to_string` naam ka koi method nahi mila.

Ab dekhte hain ke jab hum `OutlinePrint` ko aisi type par implement karne ki
koshish karte hain jo `Display` implement nahi karti, jaise `Point` struct, to
kya hota hai:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/src/main.rs:here}}
```

</Listing>

Humein ek error milta hai jo kehta hai ke `Display` required hai lekin
implement nahi kiya gaya:

```console
{{#include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/output.txt}}
```

Isay fix karne ke liye, hum `Point` par `Display` implement karte hain aur woh
constraint satisfy karte hain jo `OutlinePrint` require karta hai, is tarah:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-03-impl-display-for-point/src/main.rs:here}}
```

</Listing>

Phir, `Point` par `OutlinePrint` trait ko implement karna successfully compile
ho jayega, aur hum `Point` ke instance par `outline_print` call karke usay
asterisks ke outline ke andar display kar sakte hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-newtype-pattern-to-implement-external-traits-on-external-types"></a> <a id="using-the-newtype-pattern-to-implement-external-traits"></a>

### Implementing External Traits with the Newtype Pattern

Chapter 10 ke [“Implementing a Trait on a Type”][implementing-a-trait-on-a-type]<!--
ignore --> section mein humne orphan rule ka zikr kiya tha, jo kehta hai
ke hum kisi type par sirf us waqt trait implement kar sakte hain jab ya to
trait ya type, ya dono, hamare crate ke local hon. Is restriction ko newtype
pattern use karke bypass karna mumkin hai, jismein ek tuple struct mein ek
nayi type create ki jati hai. (Humne Chapter 5 ke [“Creating Different Types with
Tuple Structs”][tuple-structs]<!-- ignore --> section mein tuple structs cover
kiye thay.) Tuple struct mein ek field hogi aur yeh us type ke around ek thin
wrapper hogi jis par hum trait implement karna chahte hain. Phir, wrapper type
hamare crate ke liye local hoti hai, aur hum wrapper par trait implement kar
sakte hain. *Newtype* ek term hai jo Haskell programming language se originate
hui hai. Is pattern ko use karne par runtime performance mein koi penalty nahi
hoti, aur wrapper type compile time par elide kar di jati hai.

Misal ke taur par, maan lein ke hum `Vec<T>` par `Display` implement karna
chahte hain, lekin orphan rule humein seedha aisa karne se rokta hai kyun ke
`Display` trait aur `Vec<T>` type dono hamare crate ke bahar defined hain.
Hum ek `Wrapper` struct bana sakte hain jo `Vec<T>` ka ek instance hold kare;
phir, hum `Wrapper` par `Display` implement kar sakte hain aur `Vec<T>` value
ko use kar sakte hain, jaisa ke Listing 20-24 mein dikhaya gaya hai.

<Listing number="20-24" file-name="src/main.rs" caption="Creating a `Wrapper` type around `Vec<String>` to implement `Display`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-24/src/main.rs}}
```

</Listing>

`Display` ki implementation inner `Vec<T>` ko access karne ke liye `self.0`
use karti hai kyun ke `Wrapper` ek tuple struct hai aur `Vec<T>` tuple mein
index 0 par item hai. Phir, hum `Wrapper` par `Display` trait ki functionality
use kar sakte hain.

Is technique ka downside yeh hai ke `Wrapper` ek new type hai, is liye is ke
paas us value ke methods nahi hote jise yeh hold kar rahi hai. Humein
`Vec<T>` ke tamam methods directly `Wrapper` par implement karne padenge taa-ke
woh methods `self.0` ko delegate karein, jo humein `Wrapper` ko bilkul
`Vec<T>` ki tarah treat karne ki ijazat dega. Agar hum chahte ke new type ke
paas inner type ka har method ho, to `Wrapper` par `Deref` trait implement karna
ek solution hota jo inner type return kare (humne Chapter 15 ke [“Treating
Smart Pointers Like Regular References”][smart-pointer-deref]<!-- ignore -->
section mein `Deref` trait ko implement karne par discussion ki thi). Agar hum
nahi chahte ke `Wrapper` type ke paas inner type ke tamam methods hon—misal ke
taur par, `Wrapper` type ke behavior ko restrict karna ho—to humein sirf woh
methods manually implement karne padenge jo hum chahte hain.

Yeh newtype pattern tab bhi useful hai jab traits involved na hon. Ab apna
focus badalte hain aur Rust ke type system ke saath interact karne ke kuch
advanced tareeqon ko dekhte hain.

[newtype]: ch20-02-advanced-traits.html#implementing-external-traits-with-the-newtype-pattern
[implementing-a-trait-on-a-type]: ch10-02-traits.html#implementing-a-trait-on-a-type
[traits]: ch10-02-traits.html
[smart-pointer-deref]: ch15-02-deref.html#treating-smart-pointers-like-regular-references
[tuple-structs]: ch05-01-defining-structs.html#creating-different-types-with-tuple-structs
