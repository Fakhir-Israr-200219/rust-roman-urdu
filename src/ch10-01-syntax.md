## Generic Data Types

Hum generics ko function signatures ya structs jaise items ki definitions create karne ke liye use karte hain, jinhein hum baad mein bohat se different concrete data types ke saath use kar sakte hain. Sab se pehle, aaiye dekhein ke generics ko use karke functions, structs, enums, aur methods ko kaise define kiya jata hai. Phir, hum discuss karenge ke generics code ki performance ko kaise affect karte hain.


### Function Definitions Mein

Jab hum generics use karne wala function define karte hain, to hum generics ko function ki signature mein us jagah rakhte hain jahan hum aam tor par parameters aur return value ke data types specify karte hain. Aisa karne se hamara code zyada flexible hota hai aur hamare function ko call karne walon ko zyada functionality milti hai, saath hi code duplication se bhi bacha ja sakta hai.

Apne `largest` function ko continue karte hue, Listing 10-4 mein do functions dikhaye gaye hain jo dono ek slice mein sab se bari value find karte hain. Phir hum in dono ko ek single function mein combine karenge jo generics use karta hai.

<Listing number="10-4" file-name="src/main.rs" caption="Two functions that differ only in their names and in the types in their signatures">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-04/src/main.rs:here}}
```

</Listing>

`largest_i32` function woh function hai jo hum ne Listing 10-3 mein extract kiya tha aur jo ek slice mein sab se bara `i32` find karta hai. `largest_char` function ek slice mein sab se bara `char` find karta hai. Dono functions ki bodies mein same code hai, is liye aaiye ek single function mein generic type parameter introduce karke duplication ko khatam karte hain.

Naye single function mein types ko parameterize karne ke liye, humein type parameter ko naam dena hoga, bilkul usi tarah jaise hum function ke value parameters ko naam dete hain. Aap type parameter ke naam ke liye koi bhi identifier use kar sakte hain. Lekin hum `T` use karenge kyun ke convention ke mutabiq, Rust mein type parameter ke naam chhote hote hain, aksar sirf ek letter, aur Rust ki type-naming convention UpperCamelCase hai. *type* ka short form hone ki wajah se, `T` zyada tar Rust programmers ki default choice hai.

Jab hum function ki body mein koi parameter use karte hain, to humein signature mein us parameter ka naam declare karna hota hai taa-ke compiler ko pata ho ke woh naam kya represent karta hai. Isi tarah, jab hum function signature mein type parameter ka naam use karte hain, to use karne se pehle humein type parameter ka naam declare karna hota hai. Generic `largest` function define karne ke liye, hum type name declarations ko angle brackets, `<>`, ke andar function ke naam aur parameter list ke darmiyan rakhte hain, jaise:

```rust,ignore
fn largest<T>(list: &[T]) -> &T {
```

Hum is definition ko is tarah parhte hain: “Function `largest` kisi type `T` ke liye generic hai.” Is function ka ek parameter `list` hai, jo type `T` ki values ka ek slice hai. `largest` function isi type `T` ki ek value ka reference return karega.

Listing 10-5 mein generic data type ko apni signature mein use karne wali combined `largest` function definition dikhayi gayi hai. Listing ye bhi dikhati hai ke hum function ko `i32` values ke slice ya `char` values ke slice ke saath kaise call kar sakte hain. Note karein ke ye code abhi compile nahi hoga.

<Listing number="10-5" file-name="src/main.rs" caption="The `largest` function using generic type parameters; this doesn’t compile yet">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/src/main.rs}}
```

</Listing>

Agar hum abhi is code ko compile karein, to humein ye error milega:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/output.txt}}
```

Help text mein `std::cmp::PartialOrd` ka zikr hai, jo ek trait hai, aur hum next section mein traits ke baare mein baat karenge. Filhal itna samajh lein ke ye error batata hai ke `largest` ki body un tamam possible types ke liye kaam nahi karegi jo `T` ho sakte hain. Kyun ke hum body mein `T` type ki values ka comparison karna chahte hain, is liye hum sirf un types ko use kar sakte hain jin ki values ko order kiya ja sakta ho. Comparisons ko enable karne ke liye, standard library mein `std::cmp::PartialOrd` trait hai jise aap types par implement kar sakte hain (is trait ke baare mein mazeed maloomat ke liye Appendix C dekhein). Listing 10-5 ko fix karne ke liye, hum help text ki suggestion follow kar sakte hain aur `T` ke liye valid types ko sirf un types tak restrict kar sakte hain jo `PartialOrd` implement karte hain. Phir listing compile ho jayegi, kyun ke standard library `i32` aur `char` dono par `PartialOrd` implement karti hai.

### Struct Definitions Mein

Hum `<>` syntax ko use karke aise structs bhi define kar sakte hain jo ek ya zyada fields mein generic type parameter use karte hon. Listing 10-6 mein `Point<T>` struct define kiya gaya hai jo kisi bhi type ki `x` aur `y` coordinate values ko hold karta hai.

<Listing number="10-6" file-name="src/main.rs" caption="A `Point<T>` struct that holds `x` and `y` values of type `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-06/src/main.rs}}
```

</Listing>

Struct definitions mein generics use karne ki syntax function definitions mein use hone wali syntax jaisi hi hai. Sab se pehle, hum type parameter ka naam struct ke naam ke foran baad angle brackets ke andar declare karte hain. Phir, hum struct definition mein generic type ko us jagah use karte hain jahan hum warna concrete data types specify karte.

Note karein ke kyun ke hum ne `Point<T>` ko define karne ke liye sirf ek generic type use kiya hai, ye definition batati hai ke `Point<T>` struct kisi type `T` ke liye generic hai, aur fields `x` aur `y` *dono* usi same type ke hain, chahe woh type koi bhi ho. Agar hum `Point<T>` ka aisa instance create karein jis mein different types ki values hon, jaisa ke Listing 10-7 mein hai, to hamara code compile nahi hoga.

<Listing number="10-7" file-name="src/main.rs" caption="The fields `x` and `y` must be the same type because both have the same generic data type `T`.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/src/main.rs}}
```

</Listing>

Is example mein, jab hum `x` ko integer value `5` assign karte hain, to hum compiler ko batate hain ke `Point<T>` ke is instance ke liye generic type `T` ek integer hoga. Phir, jab hum `y` ke liye `4.0` specify karte hain, jise hum ne `x` ke same type ka define kiya hai, to humein is tarah ka type mismatch error milega:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/output.txt}}
```

Aisa `Point` struct define karne ke liye jismein `x` aur `y` dono generics hon lekin different types ho sakte hon, hum multiple generic type parameters use kar sakte hain. Misal ke taur par, Listing 10-8 mein hum `Point` ki definition ko change karke usay types `T` aur `U` ke liye generic karte hain, jahan `x` ka type `T` aur `y` ka type `U` hai.

<Listing number="10-8" file-name="src/main.rs" caption="A `Point<T, U>` generic over two types so that `x` and `y` can be values of different types">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-08/src/main.rs}}
```

</Listing>

Ab `Point` ke dikhaye gaye tamam instances allowed hain! Aap kisi definition mein jitne chahein generic type parameters use kar sakte hain, lekin kuch se zyada use karne se aapka code parhna mushkil ho jata hai. Agar aapko apne code mein bohat saare generic types ki zaroorat mehsoos ho rahi hai, to ye is baat ka ishara ho sakta hai ke aapke code ko chhote pieces mein restructure karne ki zaroorat hai.

### Enum Definitions Mein

Jis tarah hum ne structs ke saath kiya tha, usi tarah hum enums ko bhi define kar sakte hain taa-ke unke variants mein generic data types hold kiye ja saken. Aaiye standard library ki taraf se provide ki jane wali `Option<T>` enum ko dobara dekhte hain, jise hum ne Chapter 6 mein use kiya tha:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

Ab ye definition aapko zyada samajh aani chahiye. Jaisa ke aap dekh sakte hain, `Option<T>` enum type `T` ke liye generic hai aur is ke do variants hain: `Some`, jo type `T` ki ek value hold karta hai, aur `None` variant jo koi value hold nahi karta. `Option<T>` enum ko use karke hum optional value ke abstract concept ko express kar sakte hain, aur kyun ke `Option<T>` generic hai, hum is abstraction ko is baat se beparwah ho kar use kar sakte hain ke optional value ka type kya hai.

Enums multiple generic types bhi use kar sakte hain. `Result` enum ki definition jo hum ne Chapter 9 mein use ki thi, iska ek example hai:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`Result` enum do types, `T` aur `E`, ke liye generic hai aur is ke do variants hain: `Ok`, jo type `T` ki ek value hold karta hai, aur `Err`, jo type `E` ki ek value hold karta hai. Ye definition `Result` enum ko har us jagah use karna convenient banati hai jahan hamare paas koi aisa operation ho jo successful ho sakta hai (kisi type `T` ki value return kare) ya fail ho sakta hai (kisi type `E` ki error return kare). Asal mein, Listing 9-3 mein file open karne ke liye hum ne isi cheez ko use kiya tha, jahan file successfully open hone par `T` ko type `std::fs::File` se fill kiya gaya tha aur file open karne mein problems hone par `E` ko type `std::io::Error` se fill kiya gaya tha.

Jab aap apne code mein aisi situations ko recognize karein jahan multiple struct ya enum definitions sirf un values ke types mein different hon jinhein woh hold karti hain, to aap is ke bajaye generic types use karke duplication se bach sakte hain.


### Method Definitions Mein

Hum structs aur enums par methods implement kar sakte hain (jaisa ke hum ne Chapter 5 mein kiya tha) aur unki definitions mein generic types bhi use kar sakte hain. Listing 10-9 mein woh `Point<T>` struct dikhaya gaya hai jo hum ne Listing 10-6 mein define kiya tha, aur jis par `x` naam ka method implement kiya gaya hai.

<Listing number="10-9" file-name="src/main.rs" caption="Implementing a method named `x` on the `Point<T>` struct that will return a reference to the `x` field of type `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-09/src/main.rs}}
```

</Listing>

Yahan, hum ne `Point<T>` par `x` naam ka ek method define kiya hai jo field `x` mein mojood data ka reference return karta hai.

Note karein ke humein `impl` ke foran baad `T` declare karna hota hai taa-ke hum `T` ko use karke ye specify kar saken ke hum type `Point<T>` par methods implement kar rahe hain. `T` ko `impl` ke baad generic type ke taur par declare karke, Rust identify kar sakta hai ke `Point` mein angle brackets ke andar wala type ek generic type hai, na ke concrete type. Hum is generic parameter ke liye struct definition mein declare kiye gaye generic parameter se different naam choose kar sakte the, lekin same naam use karna conventional hai. Agar aap `impl` ke andar aisa method likhte hain jo ek generic type declare karta hai, to woh method type ke har instance par define hoga, chahe aakhir mein generic type ki jagah koi bhi concrete type substitute ho.

Hum type par methods define karte waqt generic types par constraints bhi specify kar sakte hain. Misal ke taur par, hum methods ko sirf `Point<f32>` instances par implement kar sakte hain, bajaye `Point<T>` instances par jo kisi bhi generic type ke hon. Listing 10-10 mein hum concrete type `f32` use karte hain, jis ka matlab hai ke hum `impl` ke baad koi types declare nahi karte.

<Listing number="10-10" file-name="src/main.rs" caption="An `impl` block that only applies to a struct with a particular concrete type for the generic type parameter `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-10/src/main.rs:here}}
```

</Listing>

Is code ka matlab hai ke type `Point<f32>` ke paas `distance_from_origin` method hoga; `Point<T>` ke doosre instances, jahan `T` ka type `f32` nahi hai, unke liye ye method defined nahi hoga. Ye method measure karta hai ke hamara point coordinates (0.0, 0.0) wale point se kitna door hai aur un mathematical operations ko use karta hai jo sirf floating-point types ke liye available hain.

Struct definition mein generic type parameters hamesha un generic type parameters jaise nahi hote jo aap usi struct ke method signatures mein use karte hain. Listing 10-11 mein example ko zyada clear banane ke liye `Point` struct ke liye generic types `X1` aur `Y1`, aur `mixup` method signature ke liye `X2` aur `Y2` use kiye gaye hain. Ye method ek naya `Point` instance create karta hai jis mein `self` `Point` (type `X1`) se `x` value aur pass kiye gaye `Point` se `y` value (type `Y2`) hoti hai.

<Listing number="10-11" file-name="src/main.rs" caption="A method that uses generic types that are different from its struct’s definition">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-11/src/main.rs}}
```

</Listing>

`main` mein, hum ne ek `Point` define kiya hai jis mein `x` ke liye `i32` (value `5` ke saath) aur `y` ke liye `f64` (value `10.4` ke saath) hai. Variable `p2` ek `Point` struct hai jis mein `x` ke liye string slice (value `"Hello"` ke saath) aur `y` ke liye `char` (value `c` ke saath) hai. `p1` par `p2` ko argument ke taur par de kar `mixup` call karne se humein `p3` milta hai, jis mein `x` ke liye `i32` hoga kyun ke `x`, `p1` se aaya hai. Variable `p3` mein `y` ke liye `char` hoga kyun ke `y`, `p2` se aaya hai. `println!` macro ki call `p3.x = 5, p3.y = c` print karegi.

Is example ka maqsad ek aisi situation demonstrate karna hai jahan kuch generic parameters `impl` ke saath declare kiye jate hain aur kuch method definition ke saath. Yahan, generic parameters `X1` aur `Y1` ko `impl` ke baad declare kiya gaya hai kyun ke ye struct definition ke saath hain. Generic parameters `X2` aur `Y2` ko `fn mixup` ke baad declare kiya gaya hai kyun ke ye sirf method ke liye relevant hain.

### Generics Use Karne Wale Code Ki Performance

Aap shayad soch rahe hon ke generic type parameters use karne par runtime mein koi cost hoti hai ya nahi. Achhi baat ye hai ke generic types use karne se aapka program concrete types ke saath run hone wale program ke muqable mein zara bhi slow nahi hota.

Rust compile time par generics use karne wale code ki *monomorphization* perform karke ye hasil karta hai. *Monomorphization* woh process hai jismein compile hote waqt use kiye jane wale concrete types ko fill karke generic code ko specific code mein convert kiya jata hai. Is process mein compiler un steps ke bilkul opposite steps perform karta hai jo hum ne Listing 10-5 mein generic function create karne ke liye use kiye the: Compiler un tamam jagahon ko dekhta hai jahan generic code call kiya gaya hai aur un concrete types ke liye code generate karta hai jin ke saath generic code call kiya gaya hai.

Aaiye dekhte hain ke standard library ki generic `Option<T>` enum ko use karke ye kaise kaam karta hai:

```rust
let integer = Some(5);
let float = Some(5.0);
```

Jab Rust is code ko compile karta hai, to woh monomorphization perform karta hai. Is process ke dauran, compiler un values ko read karta hai jo `Option<T>` instances mein use hui hain aur `Option<T>` ki do qisam identify karta hai: Ek `i32` hai aur doosra `f64`. Is tarah, ye `Option<T>` ki generic definition ko `i32` aur `f64` ke liye specialized do definitions mein expand karta hai, aur generic definition ko in specific definitions se replace kar deta hai.

Code ka monomorphized version kuch is tarah nazar aata hai (illustration ke liye compiler un names se different names use karta hai jo hum yahan use kar rahe hain):

<Listing file-name="src/main.rs">

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

</Listing>

Generic `Option<T>` ko compiler ki taraf se create ki gayi specific definitions se replace kar diya jata hai. Kyun ke Rust generic code ko aise code mein compile karta hai jo har instance mein type ko specify karta hai, is liye generics use karne ki koi runtime cost nahi hoti. Jab code run hota hai, to woh bilkul usi tarah perform karta hai jaise hum ne har definition ko manually duplicate kiya hota. Monomorphization ka process Rust ke generics ko runtime par extremely efficient banata hai.

