## Advanced Functions and Closures

Yeh section functions aur closures se related kuch advanced features explore
karta hai, jin mein function pointers aur closures return karna shamil hai.

### Function Pointers

Humne baat ki hai ke functions ko closures kaise pass kiye jate hain; aap
regular functions ko bhi functions mein pass kar sakte hain! Yeh technique us
waqt useful hoti hai jab aap koi naya closure define karne ke bajaye pehle se
defined function pass karna chahte hain. Functions `fn` type (lowercase *f*) mein
coerce hoti hain, isay `Fn` closure trait ke saath confuse nahi karna chahiye.
`fn` type ko *function pointer* kaha jata hai. Functions ko function pointers
ke saath pass karne se aap functions ko doosre functions ke arguments ke taur
par use kar sakte hain.

Yeh specify karne ka syntax ke koi parameter function pointer hai, closures ke
syntax ke similar hai, jaisa ke Listing 20-28 mein dikhaya gaya hai, jahan humne
ek `add_one` function define kiya hai jo apne parameter mein 1 add karta hai.
`do_twice` function do parameters leta hai: kisi bhi aise function ka function
pointer jo `i32` parameter leta ho aur `i32` return karta ho, aur ek `i32` value.
`do_twice` function `f` function ko do baar call karta hai, usay `arg` value
pass karta hai, phir dono function calls ke results ko aapas mein add karta hai.
`main` function `do_twice` ko `add_one` aur `5` arguments ke saath call karta hai.

<Listing number="20-28" file-name="src/main.rs" caption="Using the `fn` type to accept a function pointer as an argument">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-28/src/main.rs}}
```

</Listing>

Yeh code `The answer is: 12` print karta hai. Hum specify karte hain ke
`do_twice` mein parameter `f` ek `fn` hai jo `i32` type ka ek parameter leta
hai aur `i32` return karta hai. Phir hum `do_twice` ke body mein `f` ko call
kar sakte hain. `main` mein hum function name `add_one` ko `do_twice` ke pehle
argument ke taur par pass kar sakte hain.

Closures ke unlike, `fn` ek trait ke bajaye ek type hai, is liye hum `fn` ko
directly parameter type ke taur par specify karte hain, bajaye is ke ke `Fn`
traits mein se kisi ek ko trait bound ke taur par use karke generic type
parameter declare karein.

Function pointers closure ke teeno traits (`Fn`, `FnMut`, aur `FnOnce`) ko
implement karte hain, jis ka matlab hai ke aap hamesha function pointer ko aise
function ke argument ke taur par pass kar sakte hain jo closure expect karta hai.
Functions ko generic type aur closure traits mein se kisi ek ke saath likhna
behtar hota hai taa-ke aapke functions functions ya closures dono ko accept kar
sakein.

Is ke bawajood, ek example jahan aap sirf `fn` accept karna chahenge aur
closures nahi, woh external code ke saath interface karte waqt hai jahan
closures available nahi hoti: C functions functions ko arguments ke taur par
accept kar sakte hain, lekin C mein closures nahi hoti.

Ek example jahan aap inline defined closure ya named function mein se kisi ko
bhi use kar sakte hain, us ke liye standard library mein `Iterator` trait ke
provided `map` method ke ek use ko dekhte hain. Numbers ki ek vector ko strings
ki vector mein convert karne ke liye `map` method use karte hue, hum ek closure
use kar sakte hain, jaisa ke Listing 20-29 mein hai.

<Listing number="20-29" caption="Using a closure with the `map` method to convert numbers to strings">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-29/src/main.rs:here}}
```

</Listing>

Ya hum closure ke bajaye `map` ko argument ke taur par ek function ka naam de
sakte hain. Listing 20-30 dikhati hai ke yeh kaisa nazar aayega.

<Listing number="20-30" caption="Using the `String::to_string` function with the `map` method to convert numbers to strings">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-30/src/main.rs:here}}
```

</Listing>

Note karein ke humein woh fully qualified syntax use karni hogi jis par humne
[“Advanced Traits”][advanced-traits]<!-- ignore --> section mein baat ki thi,
kyun ke `to_string` naam ke multiple functions available hain.

Yahan, hum `ToString` trait mein defined `to_string` function use kar rahe hain,
jise standard library ne har us type ke liye implement kiya hai jo `Display` ko
implement karti hai.

Chapter 6 ke [“Enum Values”][enum-values]<!-- ignore --> section se yaad karein
ke har enum variant ka naam jo hum define karte hain, ek initializer function
bhi ban jata hai. Hum in initializer functions ko function pointers ke taur par
use kar sakte hain jo closure traits ko implement karte hain, jis ka matlab hai
ke hum initializer functions ko un methods ke arguments ke taur par specify kar
sakte hain jo closures lete hain, jaisa ke Listing 20-31 mein dekha gaya hai.

<Listing number="20-31" caption="Using an enum initializer with the `map` method to create a `Status` instance from numbers">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-31/src/main.rs:here}}
```

</Listing>

Yahan, hum `Status::Value` ke initializer function ko use karke `map` par call
ki gayi range mein har `u32` value se `Status::Value` instances create karte
hain. Kuch log is style ko prefer karte hain aur kuch log closures use karna
prefer karte hain. Yeh dono same code mein compile hote hain, is liye woh style
use karein jo aapko zyada clear lage.

### Returning Closures

Closures ko traits ke through represent kiya jata hai, jis ka matlab hai ke aap
closures ko directly return nahi kar sakte. Zyada tar cases mein jahan aap koi
trait return karna chahte hon, aap is ke bajaye us concrete type ko function ki
return value ke taur par use kar sakte hain jo us trait ko implement karti hai.
Lekin, closures ke saath aam tor par aisa nahi kar sakte kyun ke un ki koi
concrete type nahi hoti jise return kiya ja sake; misal ke taur par, agar
closure apne scope se koi values capture karti ho to aap function pointer `fn`
ko return type ke taur par use nahi kar sakte.

Is ke bajaye, aap aam tor par Chapter 10 mein seekhi hui `impl Trait` syntax use
karenge. Aap `Fn`, `FnOnce`, aur `FnMut` ko use karke kisi bhi function type ko
return kar sakte hain. Misal ke taur par, Listing 20-32 ka code bilkul theek
compile hoga.

<Listing number="20-32" caption="Returning a closure from a function using the `impl Trait` syntax">

```rust id="w9f8v2"
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-32/src/lib.rs}}
```

</Listing>

Lekin, jaisa ke humne Chapter 13 ke [“Inferring and Annotating Closure
Types”][closure-types]<!-- ignore --> section mein note kiya tha, har closure
bhi apni ek distinct type hoti hai. Agar aapko multiple functions ke saath kaam
karna ho jin ki signature same ho lekin implementations different hon, to aapko
un ke liye trait object use karna hoga. Consider karein ke agar aap Listing
20-33 mein dikhaye gaye code jaisa code likhte hain to kya hota hai.

<Listing file-name="src/main.rs" number="20-33" caption="Creating a `Vec<T>` of closures defined by functions that return `impl Fn` types">

```rust,ignore,does_not_compile id="5k4d7a"
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-33/src/main.rs}}
```

</Listing>

Yahan hamare paas do functions hain, `returns_closure` aur
`returns_initialized_closure`, jo dono `impl Fn(i32) -> i32` return karte hain.
Notice karein ke jo closures woh return karte hain woh different hain, chahe
woh same type ko implement karte hon. Agar hum isay compile karne ki koshish
karein, to Rust humein batata hai ke yeh kaam nahi karega:

```text id="p6at7m"
{{#include ../listings/ch20-advanced-features/listing-20-33/output.txt}}
```

Error message humein batata hai ke jab bhi hum `impl Trait` return karte hain,
Rust ek unique *opaque type* create karta hai, yani aisi type jis ke andar
Rust hamare liye kya construct karta hai, us ki details hum nahi dekh sakte, aur
na hi hum guess kar sakte hain ke Rust kaunsi type generate karega taa-ke khud
usay likh sakein. Is liye, agarche yeh functions aisi closures return karte hain
jo same trait, `Fn(i32) -> i32`, ko implement karti hain, Rust jo opaque types
har function ke liye generate karta hai woh distinct hoti hain. (Yeh usi tarah
hai jaise Rust distinct async blocks ke liye different concrete types produce
karta hai, chahe un ka output type same ho, jaisa ke humne Chapter 17 ke
[“The `Pin` Type and the `Unpin` Trait”][future-types]<!-- ignore --> section
mein dekha tha.) Humne is problem ka solution ab tak kuch baar dekha hai: Hum
trait object use kar sakte hain, jaisa ke Listing 20-34 mein hai.

<Listing number="20-34" caption="Creating a `Vec<T>` of closures defined by functions that return `Box<dyn Fn>` so that they have the same type">

```rust id="y1w5j9"
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-34/src/main.rs:here}}
```

</Listing>

Yeh code bilkul theek compile hoga. Trait objects ke bare mein mazeed jaanne
ke liye Chapter 18 ke [“Using Trait Objects To Abstract over Shared
Behavior”][trait-objects]<!-- ignore --> section ko refer karein.

Agla topic macros hai!

[advanced-traits]: ch20-02-advanced-traits.html#advanced-traits
[enum-values]: ch06-01-defining-an-enum.html#enum-values
[closure-types]: ch13-01-closures.html#closure-type-inference-and-annotation
[future-types]: ch17-03-more-futures.html
[trait-objects]: ch18-02-trait-objects.html
