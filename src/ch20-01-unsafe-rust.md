## Unsafe Rust

Ab tak humne jis code par discussion ki hai, us mein Rust ki memory safety guarantees ko compile time par enforce kiya gaya hai. Lekin Rust ke andar ek doosri language bhi chhupi hui hai jo in memory safety guarantees ko enforce nahi karti: Isay *unsafe Rust* kaha jata hai aur yeh bilkul regular Rust ki tarah kaam karti hai, lekin humein extra superpowers deti hai.

Unsafe Rust is liye exist karti hai kyun ke static analysis apni nature mein conservative hoti hai. Jab compiler yeh determine karne ki koshish karta hai ke code guarantees ko uphold karta hai ya nahi, to us ke liye kuch valid programs ko reject karna, kuch invalid programs ko accept karne se behtar hota hai. Agarche code *might* theek ho, lekin agar Rust compiler ke paas itni information nahi hai ke woh confident ho sake, to woh code ko reject kar dega. In cases mein, aap compiler ko unsafe code use karke keh sakte hain, “Mujh par trust karo, mujhe pata hai main kya kar raha hoon.” Lekin warn kar diya jaye ke aap unsafe Rust ko apne risk par use karte hain: Agar aap unsafe code ko ghalat tareeqe se use karein, to memory unsafety ki wajah se problems ho sakti hain, jaise null pointer dereferencing.

Rust ke unsafe alter ego ki ek aur wajah yeh hai ke underlying computer hardware apni nature mein inherently unsafe hai. Agar Rust aapko unsafe operations karne ki ijazat na deta, to aap kuch specific tasks nahi kar sakte. Rust ko aapko low-level systems programming karne ki ability deni hoti hai, jaise operating system ke saath directly interact karna ya hatta ke apna operating system likhna. Low-level systems programming ke saath kaam karna language ke goals mein se ek hai. Aaiye explore karte hain ke hum unsafe Rust ke saath kya kar sakte hain aur usay kaise karna hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="unsafe-superpowers"></a>

### Performing Unsafe Superpowers

Unsafe Rust mein switch karne ke liye `unsafe` keyword use karein aur phir ek naya block start karein jo unsafe code ko hold karta ho. Unsafe Rust mein aap paanch actions perform kar sakte hain jo safe Rust mein nahi kar sakte, jinhein hum *unsafe superpowers* kehte hain. In superpowers mein yeh ability shamil hai ke:

1. Dereference a raw pointer.
2. Call an unsafe function or method.
3. Access or modify a mutable static variable.
4. Implement an unsafe trait.
5. Access fields of `union`s.

Yeh samajhna important hai ke `unsafe` borrow checker ko turn off nahi karta aur na hi Rust ke doosre safety checks ko disable karta hai: Agar aap unsafe code mein reference use karte hain, to woh ab bhi check hoga. `unsafe` keyword sirf aapko in paanch features tak access deta hai jinhein compiler memory safety ke liye check nahi karta. Unsafe block ke andar bhi aapko kuch had tak safety milti rahegi.

Is ke ilawa, `unsafe` ka matlab yeh nahi hai ke block ke andar ka code zaroor dangerous hai ya us mein definitely memory safety problems hongi: Maqsad yeh hai ke programmer ke taur par aap ensure karein ke `unsafe` block ke andar code memory ko valid tareeqe se access karega.

Log ghaltiyan kar sakte hain aur mistakes hongi, lekin in paanch unsafe operations ko `unsafe` se annotated blocks ke andar rakhne ki requirement ki wajah se, aap jaan sakenge ke memory safety se related koi bhi errors `unsafe` block ke andar hi hone chahiye. `unsafe` blocks ko chhota rakhein; baad mein memory bugs investigate karte waqt aap is ke liye thankful honge.

Unsafe code ko jitna mumkin ho isolate karne ke liye, behtar hai ke aise code ko ek safe abstraction ke andar enclose kiya jaye aur ek safe API provide ki jaye, jise hum chapter mein baad mein discuss karenge jab hum unsafe functions aur methods ka jaiza lenge. Standard library ke kuch parts aise safe abstractions ke taur par implement kiye gaye hain jo audited unsafe code ke upar bani hui hain. Unsafe code ko safe abstraction mein wrap karna `unsafe` ke uses ko un tamam places tak leak hone se rokta hai jahan aap ya aapke users unsafe code se implement ki gayi functionality ko use karna chahte hon, kyun ke safe abstraction ko use karna safe hai.

Aaiye baari baari se in paanch unsafe superpowers mein se har ek ko dekhte hain. Hum kuch aisi abstractions bhi dekhenge jo unsafe code ke liye ek safe interface provide karti hain.

### Dereferencing a Raw Pointer

Chapter 4 mein, [“Dangling References”][dangling-references]<!-- ignore
--> section mein, humne mention kiya tha ke compiler ensure karta hai ke references hamesha valid hon. Unsafe Rust mein do naye types hote hain jinhein *raw pointers* kaha jata hai jo references se milte-julte hain. References ki tarah, raw pointers immutable ya mutable ho sakte hain aur respectively `*const T` aur `*mut T` ke taur par likhe jate hain. Asterisk dereference operator nahi hai; yeh type name ka hissa hai. Raw pointers ke context mein, *immutable* ka matlab hai ke dereference hone ke baad pointer ko directly assign nahi kiya ja sakta.

References aur smart pointers se mukhtalif, raw pointers:

* Borrowing rules ko ignore kar sakte hain, kyun ke ek hi location par immutable aur mutable pointers dono, ya multiple mutable pointers rakhna allowed hai
* Valid memory ki taraf point karne ki guarantee nahi hoti
* Null ho sakte hain
* Koi automatic cleanup implement nahi karte

Rust ko in guarantees ko enforce karne se opt out karke, aap guaranteed safety ko greater performance ya kisi doosri language ya hardware ke saath interface karne ki ability ke badle mein give up kar sakte hain, jahan Rust ki guarantees apply nahi hotin.

Listing 20-1 dikhati hai ke immutable aur mutable raw pointer kaise create kiya jata hai.

<Listing number="20-1" caption="Creating raw pointers with the raw borrow operators">

```rust id="7n3wpa"
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-01/src/main.rs:here}}
```

</Listing>

Notice karein ke hum is code mein `unsafe` keyword include nahi karte. Hum safe code mein raw pointers create kar sakte hain; hum sirf unsafe block ke bahar raw pointers ko dereference nahi kar sakte, jaisa ke aap thori der mein dekhenge.

Humne raw borrow operators ko use karke raw pointers create kiye hain: `&raw const num` ek `*const i32` immutable raw pointer create karta hai, aur `&raw mut num` ek `*mut i32` mutable raw pointer create karta hai. Kyun ke humne inhein directly ek local variable se create kiya hai, hum jaante hain ke yeh particular raw pointers valid hain, lekin hum har raw pointer ke bare mein yeh assumption nahi kar sakte.

Is baat ko demonstrate karne ke liye, ab hum ek aisa raw pointer create karenge jis ki validity ke bare mein hum itne certain nahi ho sakte, aur raw borrow operator use karne ke bajaye value ko cast karne ke liye keyword `as` use karenge. Listing 20-2 dikhati hai ke memory mein kisi arbitrary location ke liye raw pointer kaise create kiya jata hai. Arbitrary memory ko use karna undefined hai: Us address par data ho bhi sakta hai aur nahi bhi, compiler code ko is tarah optimize kar sakta hai ke koi memory access hi na ho, ya program segmentation fault ke saath terminate ho sakta hai. Aam tor par, is tarah ka code likhne ki koi achi wajah nahi hoti, khaas taur par un cases mein jahan aap is ke bajaye raw borrow operator use kar sakte hain, lekin yeh possible hai.

<Listing number="20-2" caption="Creating a raw pointer to an arbitrary memory address">

```rust id="a5l5i4"
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-02/src/main.rs:here}}
```

</Listing>

Yaad rakhein ke hum safe code mein raw pointers create kar sakte hain, lekin hum raw pointers ko dereference karke unke pointed-to data ko read nahi kar sakte. Listing 20-3 mein, hum ek raw pointer par dereference operator `*` use karte hain jo `unsafe` block require karta hai.

<Listing number="20-3" caption="Dereferencing raw pointers within an `unsafe` block">

```rust id="p9o6ls"
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-03/src/main.rs:here}}
```

</Listing>

Pointer create karne se koi harm nahi hota; sirf us waqt jab hum us value ko access karne ki koshish karte hain jis ki taraf woh point karta hai, tab hum ek invalid value ke saath deal kar sakte hain.

Yeh bhi note karein ke Listings 20-1 aur 20-3 mein, humne `*const i32` aur `*mut
i32` raw pointers create kiye jo dono ek hi memory location ki taraf point kar rahe thay, jahan `num` store hai. Agar hum is ke bajaye `num` ke liye ek immutable aur ek mutable reference create karne ki koshish karte, to code compile nahi hota kyun ke Rust ke ownership rules ek mutable reference ko usi waqt kisi bhi immutable references ke saath allow nahi karte. Raw pointers ke saath, hum ek hi location ke liye mutable pointer aur immutable pointer create kar sakte hain aur mutable pointer ke zariye data change kar sakte hain, jis se potentially data race create ho sakta hai. Ehtiyat karein!

In tamam dangers ke bawajood, aap raw pointers kabhi use kyun karenge? Ek major use case C code ke saath interfacing hai, jaisa ke aap next section mein dekhenge. Ek aur case safe abstractions build karna hai jinhein borrow checker samajh nahi pata. Hum unsafe functions introduce karenge aur phir safe abstraction ki ek example dekhenge jo unsafe code use karti hai.

### Calling an Unsafe Function or Method

Doosri type ki operation jo aap unsafe block mein perform kar sakte hain, woh unsafe functions ko call karna hai. Unsafe functions aur methods bilkul regular functions aur methods ki tarah nazar aate hain, lekin unki definition ke baqi hisson se pehle extra `unsafe` hota hai. Is context mein `unsafe` keyword indicate karta hai ke function ki kuch requirements hain jinhein humein is function ko call karte waqt uphold karna hota hai, kyun ke Rust guarantee nahi kar sakta ke humne in requirements ko meet kiya hai. Kisi unsafe function ko `unsafe` block ke andar call karke, hum keh rahe hote hain ke humne is function ki documentation parhi hai aur hum function ke contracts ko uphold karne ki responsibility lete hain.

Yahan `dangerous` naam ka ek unsafe function hai jo apne body mein kuch nahi karta:

```rust id="r7d2kp"
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-01-unsafe-fn/src/main.rs:here}}
```

Humein `dangerous` function ko ek separate `unsafe` block ke andar call karna zaroori hai. Agar hum `unsafe` block ke baghair `dangerous` ko call karne ki koshish karein, to humein ek error milega:

```console id="q1v8mx"
{{#include ../listings/ch20-advanced-features/output-only-01-missing-unsafe/output.txt}}
```

`unsafe` block ke saath, hum Rust ko assert kar rahe hote hain ke humne function ki documentation parhi hai, hum samajhte hain ke isay properly kaise use karna hai, aur humne verify kar liya hai ke hum function ke contract ko fulfill kar rahe hain.

Unsafe function ke body mein unsafe operations perform karne ke liye bhi aapko `unsafe` block use karna zaroori hai, bilkul regular function ke andar ki tarah, aur agar aap bhool jayein to compiler aapko warning dega. Is se humein `unsafe` blocks ko jitna mumkin ho chhota rakhne mein madad milti hai, kyun ke unsafe operations poore function body mein zaroori nahi hote.

#### Creating a Safe Abstraction over Unsafe Code

Sirf is wajah se ke kisi function mein unsafe code hai, yeh zaroori nahi ke hum poore function ko `unsafe` mark karein. Asal mein, unsafe code ko safe function mein wrap karna ek common abstraction hai. Misal ke taur par, standard library ke `split_at_mut` function ka jaiza lete hain, jise kuch unsafe code ki zaroorat hoti hai. Hum dekhenge ke hum isay kaise implement kar sakte hain. Yeh safe method mutable slices par defined hai: Yeh ek slice leta hai aur argument ke taur par diye gaye index par slice ko split karke usay do slices mein bana deta hai. Listing 20-4 dikhati hai ke `split_at_mut` ko kaise use karna hai.

<Listing number="20-4" caption="Using the safe `split_at_mut` function">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-04/src/main.rs:here}}
```

</Listing>

Hum is function ko sirf safe Rust use karke implement nahi kar sakte. Ek attempt Listing 20-5 jaisa ho sakta hai, jo compile nahi hoga. Simplicity ke liye, hum `split_at_mut` ko method ke bajaye function ke taur par implement karenge aur generic type `T` ke bajaye sirf `i32` values ke slices ke liye implement karenge.

<Listing number="20-5" caption="An attempted implementation of `split_at_mut` using only safe Rust">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-05/src/main.rs:here}}
```

</Listing>

Yeh function sab se pehle slice ki total length hasil karta hai. Phir, yeh check karke ke index length se less than ya equal hai, yeh assert karta hai ke parameter ke taur par diya gaya index slice ke andar hai. Is assertion ka matlab hai ke agar hum slice ko split karne ke liye length se greater index pass karein, to function us index ko use karne ki koshish se pehle panic karega.

Phir, hum tuple mein do mutable slices return karte hain: ek original slice ke start se `mid` index tak aur doosra `mid` se slice ke end tak.

Jab hum Listing 20-5 ke code ko compile karne ki koshish karte hain, to humein ek error milega:

```console
{{#include ../listings/ch20-advanced-features/listing-20-05/output.txt}}
```

Rust ka borrow checker yeh nahi samajh sakta ke hum slice ke different parts ko borrow kar rahe hain; woh sirf itna jaanta hai ke hum ek hi slice se do baar borrow kar rahe hain. Slice ke different parts ko borrow karna fundamentally theek hai kyun ke dono slices overlap nahi karte, lekin Rust itna smart nahi hai ke yeh baat jaan sake. Jab humein pata ho ke code theek hai, lekin Rust ko pata nahi, to yeh unsafe code use karne ka waqt hai.

Listing 20-6 dikhati hai ke `split_at_mut` ki implementation ko kaam karwane ke liye `unsafe` block, raw pointer, aur unsafe functions ki kuch calls ko kaise use kiya jata hai.

<Listing number="20-6" caption="Using unsafe code in the implementation of the `split_at_mut` function">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-06/src/main.rs:here}}
```

</Listing>

Chapter 4 ke [“The Slice Type”][the-slice-type]<!-- ignore --> section se yaad karein ke slice kisi data ka pointer aur slice ki length hota hai. Hum slice ki length hasil karne ke liye `len` method use karte hain aur slice ke raw pointer ko access karne ke liye `as_mut_ptr` method use karte hain. Is case mein, kyun ke hamare paas `i32` values ka mutable slice hai, `as_mut_ptr` `*mut i32` type ka raw pointer return karta hai, jise humne `ptr` variable mein store kiya hai.

Hum yeh assertion barqarar rakhte hain ke `mid` index slice ke andar hai. Phir hum unsafe code tak pohanchte hain: `slice::from_raw_parts_mut` function ek raw pointer aur length leta hai aur ek slice create karta hai. Hum is function ko use karke ek aisa slice create karte hain jo `ptr` se start hota hai aur `mid` items long hota hai. Phir, hum `ptr` par `add` method ko `mid` ko argument ke taur par dekar call karte hain taa-ke ek aisa raw pointer hasil ho jo `mid` se start hota hai, aur hum us pointer aur `mid` ke baad remaining items ki tadaad ko length ke taur par use karke ek slice create karte hain.

Function `slice::from_raw_parts_mut` unsafe hai kyun ke yeh ek raw pointer leta hai aur is baat par trust karna padta hai ke yeh pointer valid hai. Raw pointers par `add` method bhi unsafe hai kyun ke isay trust karna padta hai ke offset location bhi ek valid pointer hai. Is liye humein `slice::from_raw_parts_mut` aur `add` ki calls ke around `unsafe` block rakhna pada taa-ke hum unhein call kar saken. Code ko dekh kar aur yeh assertion add karke ke `mid` less than ya equal to `len` hona zaroori hai, hum bata sakte hain ke `unsafe` block ke andar use hone wale tamam raw pointers slice ke andar data ke valid pointers honge. Yeh `unsafe` ka ek acceptable aur appropriate use hai.

Note karein ke humein resultant `split_at_mut` function ko `unsafe` mark karne ki zaroorat nahi hai, aur hum is function ko safe Rust se call kar sakte hain. Humne unsafe code ke liye ek safe abstraction create ki hai, aisi function implementation ke saath jo `unsafe` code ko safe tareeqe se use karti hai, kyun ke yeh sirf us data se valid pointers create karti hai jis tak is function ko access hasil hai.

Is ke baraks, Listing 20-7 mein `slice::from_raw_parts_mut` ka use likely crash karega jab slice ko use kiya jayega. Yeh code ek arbitrary memory location leta hai aur 10,000 items long slice create karta hai.

<Listing number="20-7" caption="Creating a slice from an arbitrary memory location">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-07/src/main.rs:here}}
```

</Listing>

Hum is arbitrary location par maujood memory ke owner nahi hain, aur is baat ki koi guarantee nahi hai ke jo slice yeh code create karta hai us mein valid `i32` values hain. `values` ko ek valid slice samajh kar use karne ki koshish undefined behavior ka sabab banti hai.

#### Using `extern` Functions to Call External Code

Kabhi kabhi aapke Rust code ko kisi doosri language mein likhe gaye code ke saath interact karne ki zaroorat ho sakti hai. Is ke liye, Rust ke paas `extern` keyword hai jo *Foreign Function Interface (FFI)* ki creation aur use ko facilitate karta hai, jo ek aisa tareeqa hai jiske zariye ek programming language functions ko define kar sakti hai aur kisi different (foreign) programming language ko un functions ko call karne ki ability de sakti hai.

Listing 20-8 demonstrate karti hai ke C standard library ke `abs` function ke saath integration kaise set up ki jati hai. `extern` blocks ke andar declare kiye gaye functions ko Rust code se call karna generally unsafe hota hai, is liye `extern` blocks ko bhi `unsafe` mark karna zaroori hai. Is ki wajah yeh hai ke doosri languages Rust ke rules aur guarantees enforce nahi kartin, aur Rust unhein check nahi kar sakta, is liye safety ensure karne ki responsibility programmer par hoti hai.

<Listing number="20-8" file-name="src/main.rs" caption="Declaring and calling an `extern` function defined in another language">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-08/src/main.rs}}
```

</Listing>

`unsafe extern "C"` block ke andar, hum un external functions ke names aur signatures list karte hain jinhein hum kisi doosri language se call karna chahte hain. `"C"` wala hissa define karta hai ke external function kaunsa *application binary interface (ABI)* use karta hai: ABI define karta hai ke assembly level par function ko kaise call kiya jata hai. `"C"` ABI sab se common hai aur C programming language ke ABI ko follow karta hai. Rust jin tamam ABIs ko support karta hai unke bare mein information [the Rust Reference][ABI] mein available hai.

`unsafe extern` block ke andar declare kiya gaya har item implicitly unsafe hota hai. Lekin kuch FFI functions *safe* hotay hain. Misal ke taur par, C ki standard library ka `abs` function memory safety ke hawale se koi considerations nahi rakhta, aur hum jaante hain ke isay kisi bhi `i32` ke saath call kiya ja sakta hai. Aise cases mein, hum `safe` keyword use karke keh sakte hain ke yeh specific function call karne ke liye safe hai, chahe yeh `unsafe extern` block ke andar ho. Jab hum yeh change kar dete hain, to isay call karne ke liye ab `unsafe` block ki zaroorat nahi rehti, jaisa ke Listing 20-9 mein dikhaya gaya hai.

<Listing number="20-9" file-name="src/main.rs" caption="Explicitly marking a function as `safe` within an `unsafe extern` block and calling it safely">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-09/src/main.rs}}
```

</Listing>

Kisi function ko `safe` mark karna inherently usay safe nahi bana deta! Is ke bajaye, yeh us promise ki tarah hai jo aap Rust se kar rahe hote hain ke yeh safe hai. Yeh ensure karna ab bhi aapki responsibility hai ke woh promise poora ho!

#### Calling Rust Functions from Other Languages

Hum `extern` ko ek aisa interface create karne ke liye bhi use kar sakte hain jo doosri languages ko Rust functions call karne ki ability deta hai. Pura `extern` block create karne ke bajaye, hum `extern` keyword add karte hain aur relevant function ke `fn` keyword se bilkul pehle use kiya jane wala ABI specify karte hain. Humein `#[unsafe(no_mangle)]` annotation bhi add karna hota hai taa-ke Rust compiler ko bataya ja sake ke is function ke name ko mangle na kare. *Mangling* us waqt hoti hai jab compiler hamare diye hue function name ko ek different name mein change karta hai jismein compilation process ke doosre parts ke consume karne ke liye zyada information hoti hai, lekin jo insaan ke liye kam readable hota hai. Har programming language ka compiler names ko thora different tareeqe se mangle karta hai, is liye kisi Rust function ko doosri languages ke zariye nameable banane ke liye humein Rust compiler ki name mangling disable karni hoti hai. Yeh unsafe hai kyun ke built-in mangling ke baghair libraries ke darmiyan name collisions ho sakti hain, is liye yeh hamari responsibility hai ke hum jo name choose karein woh mangling ke baghair export karne ke liye safe ho.

Neeche diye gaye example mein, hum `call_from_c` function ko C code se accessible banate hain, jab isay shared library mein compile karke C se link kiya jaye:

```
#[unsafe(no_mangle)]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```

`extern` ka yeh usage sirf attribute mein `unsafe` require karta hai, `extern` block par nahi.

### Accessing or Modifying a Mutable Static Variable

Is book mein, humne abhi tak global variables ke bare mein baat nahi ki, jinhein Rust support karta hai lekin jo Rust ke ownership rules ke saath problematic ho sakte hain. Agar do threads ek hi mutable global variable ko access kar rahe hon, to is se data race ho sakti hai.

Rust mein, global variables ko *static* variables kaha jata hai. Listing 20-10 ek static variable ki example declaration aur use dikhati hai jismein value ke taur par ek string slice hai.

<Listing number="20-10" file-name="src/main.rs" caption="Defining and using an immutable static variable">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-10/src/main.rs}}
```

</Listing>

Static variables constants ke similar hotay hain, jin par humne Chapter 3 ke [“Declaring Constants”][constants]<!-- ignore --> section mein baat ki thi. Static variables ke names convention ke mutabiq `SCREAMING_SNAKE_CASE` mein hotay hain. Static variables sirf un references ko store kar sakte hain jinka lifetime `'static` ho, jis ka matlab hai ke Rust compiler lifetime ka pata laga sakta hai aur humein usay explicitly annotate karne ki zaroorat nahi hoti. Immutable static variable ko access karna safe hai.

Constants aur immutable static variables ke darmiyan ek subtle difference yeh hai ke static variable mein values ki memory mein ek fixed address hoti hai. Value ko use karne par hamesha wahi data access hota hai. Doosri taraf, constants ko ijazat hoti hai ke jab bhi unhein use kiya jaye to woh apna data duplicate kar dein. Ek aur difference yeh hai ke static variables mutable ho sakte hain. Mutable static variables ko access aur modify karna *unsafe* hai. Listing 20-11 dikhati hai ke `COUNTER` naam ke mutable static variable ko kaise declare, access, aur modify karna hai.

<Listing number="20-11" file-name="src/main.rs" caption="Reading from or writing to a mutable static variable is unsafe.">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-11/src/main.rs}}
```

</Listing>

Regular variables ki tarah, hum `mut` keyword use karke mutability specify karte hain. Koi bhi code jo `COUNTER` se read ya us mein write karta hai, `unsafe` block ke andar hona zaroori hai. Listing 20-11 ka code compile hota hai aur `COUNTER: 3` print karta hai, jaisa ke hum expect karte hain, kyun ke yeh single threaded hai. Agar multiple threads `COUNTER` ko access karein, to likely data races hongi, is liye yeh undefined behavior hai. Is wajah se, humein poore function ko `unsafe` mark karna aur safety limitation ko document karna zaroori hai taa-ke function ko call karne wala har shakhs jaane ke woh safely kya kar sakta hai aur kya nahi.

Jab bhi hum ek unsafe function likhte hain, to `SAFETY` se shuru hone wala comment likhna idiomatic hai jo explain kare ke caller ko function ko safely call karne ke liye kya karna zaroori hai. Isi tarah, jab bhi hum koi unsafe operation perform karte hain, to `SAFETY` se shuru hone wala comment likhna idiomatic hai taa-ke explain kiya ja sake ke safety rules ko kaise uphold kiya ja raha hai.

Is ke ilawa, compiler by default compiler lint ke zariye mutable static variable ke references create karne ki kisi bhi koshish ko deny kar dega. Aapko ya to `#[allow(static_mut_refs)]` annotation add karke us lint ki protections se explicitly opt out karna hoga, ya mutable static variable ko raw borrow operators mein se kisi ek ke zariye create kiye gaye raw pointer ke through access karna hoga. Is mein woh cases bhi shamil hain jahan reference invisibly create hota hai, jaise is code listing mein `println!` mein use hone par. Mutable static variables ke references ko raw pointers ke zariye create karna zaroori banane se unhein use karne ke safety requirements zyada obvious ho jati hain.

Globally accessible mutable data ke saath yeh ensure karna mushkil hota hai ke koi data races na hon, isi liye Rust mutable static variables ko unsafe consider karta hai. Jahan mumkin ho, behtar hai ke hum Chapter 16 mein discuss ki gayi concurrency techniques aur thread-safe smart pointers use karein taa-ke compiler check kar sake ke different threads se data access safely kiya ja raha hai.

### Implementing an Unsafe Trait

Hum `unsafe` ko unsafe trait ko implement karne ke liye use kar sakte hain. Ek trait tab unsafe hota hai jab uske kam az kam ek method mein koi aisa invariant ho jise compiler verify nahi kar sakta. Hum `trait` se pehle `unsafe` keyword add karke declare karte hain ke koi trait `unsafe` hai aur trait ki implementation ko bhi `unsafe` mark karte hain, jaisa ke Listing 20-12 mein dikhaya gaya hai.

<Listing number="20-12" caption="Defining and implementing an unsafe trait">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-12/src/main.rs:here}}
```

</Listing>

`unsafe impl` use karke, hum promise kar rahe hote hain ke hum un invariants ko uphold karenge jinhein compiler verify nahi kar sakta.

Misal ke taur par, Chapter 16 ke [“Extensible Concurrency with `Send` and `Sync`”][send-and-sync]<!-- ignore --> section mein discuss kiye gaye `Send` aur `Sync` marker traits ko yaad karein: Compiler in traits ko automatically implement karta hai agar hamare types poori tarah un doosre types se composed hon jo `Send` aur `Sync` implement karte hain. Agar hum aisa type implement karte hain jo kisi aise type ko contain karta hai jo `Send` ya `Sync` implement nahi karta, jaise raw pointers, aur hum us type ko `Send` ya `Sync` ke taur par mark karna chahte hain, to humein `unsafe` use karna hoga. Rust verify nahi kar sakta ke hamara type is guarantee ko uphold karta hai ke isay safely threads ke darmiyan send kiya ja sakta hai ya multiple threads se access kiya ja sakta hai; is liye humein yeh checks manually karne aur `unsafe` ke zariye is baat ko indicate karne ki zaroorat hoti hai.

### Accessing Fields of a Union

Final action jo sirf `unsafe` ke saath kaam karta hai, woh union ke fields ko access karna hai. Ek *union* `struct` ke similar hota hai, lekin kisi particular instance mein ek waqt mein sirf ek declared field use hota hai. Unions ka primary use C code mein unions ke saath interface karna hai. Union fields ko access karna unsafe hai kyun ke Rust guarantee nahi kar sakta ke is waqt union instance mein kis type ka data store hai. Aap unions ke bare mein mazeed [the Rust Reference][unions] mein jaan sakte hain.

### Using Miri to Check Unsafe Code

Jab aap unsafe code likh rahe hon, to aap yeh check karna chahenge ke jo aapne likha hai woh waqai safe aur correct hai. Is ka ek behtareen tareeqa Miri use karna hai, jo undefined behavior detect karne ke liye ek official Rust tool hai. Borrow checker ek *static* tool hai jo compile time par kaam karta hai, jab ke Miri ek *dynamic* tool hai jo runtime par kaam karta hai. Yeh aapke code ko aapka program, ya uski test suite, run karke check karta hai aur detect karta hai jab aap un rules ki violation karte hain jinhein yeh Rust ke kaam karne ke tareeqe ke bare mein samajhta hai.

Miri ko use karne ke liye Rust ka nightly build required hai (jis par hum [Appendix G: How Rust is Made and “Nightly Rust”][nightly]<!-- ignore --> mein mazeed baat karte hain). Aap `rustup +nightly component add miri` type karke Rust ka nightly version aur Miri tool dono install kar sakte hain. Is se aapke project mein use hone wale Rust ke version mein koi change nahi hota; yeh sirf tool ko aapke system mein add karta hai taa-ke jab aap chahein isay use kar saken. Aap `cargo +nightly miri run` ya `cargo +nightly miri test` type karke kisi project par Miri chala sakte hain.

Is ki usefulness ki ek example ke liye, dekhein ke jab hum isay Listing 20-7 par run karte hain to kya hota hai.

```console
{{#include ../listings/ch20-advanced-features/listing-20-07/output.txt}}
```

Miri humein correctly warn karta hai ke hum ek integer ko pointer mein cast kar rahe hain, jo problem ho sakti hai, lekin Miri determine nahi kar sakta ke problem exist karti hai ya nahi kyun ke usay nahi pata ke pointer originate kahan se hua. Phir, Miri ek error return karta hai jahan Listing 20-7 mein undefined behavior hai kyun ke hamare paas ek dangling pointer hai. Miri ki wajah se, ab humein pata hai ke undefined behavior ka risk hai, aur hum soch sakte hain ke code ko safe kaise banaya jaye. Kuch cases mein, Miri errors ko fix karne ke tareeqe ke bare mein recommendations bhi de sakta hai.

Miri har woh cheez catch nahi karta jo unsafe code likhte waqt aap se ghalat ho sakti hai. Miri ek dynamic analysis tool hai, is liye yeh sirf un code ke problems catch karta hai jo waqai run hota hai. Is ka matlab hai ke aapko apne likhe hue unsafe code ke bare mein apna confidence barhane ke liye isay achhi testing techniques ke saath use karna hoga. Miri aapke code ke unsound hone ke har mumkin tareeqe ko bhi cover nahi karta.

Doosre alfaaz mein: Agar Miri koi problem *catch* karta hai, to aap jaante hain ke ek bug hai, lekin sirf is wajah se ke Miri koi bug *catch nahi* karta, yeh matlab nahi ke koi problem nahi hai. Lekin yeh bohat kuch catch kar sakta hai. Is chapter mein unsafe code ki doosri examples par bhi isay run karke dekhein aur dekhein ke yeh kya kehta hai!

Aap [its GitHub repository][miri] par Miri ke bare mein mazeed jaan sakte hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="when-to-use-unsafe-code"></a>

### Using Unsafe Code Correctly

Abhi discuss ki gayi paanch superpowers mein se kisi ek ko use karne ke liye `unsafe` use karna ghalat nahi hai aur na hi isay na-pasand kiya jata hai, lekin `unsafe` code ko correctly implement karna zyada tricky hota hai kyun ke compiler memory safety ko uphold karne mein madad nahi kar sakta. Jab aapke paas `unsafe` code use karne ki koi wajah ho, to aap isay use kar sakte hain, aur explicit `unsafe` annotation hone ki wajah se jab problems occur hon to unke source ko track down karna aasaan ho jata hai. Jab bhi aap unsafe code likhein, aap Miri ko use karke is baat par zyada confident ho sakte hain ke aapka likha hua code Rust ke rules ko uphold karta hai.

Unsafe Rust ke saath effectively kaam karne ki bohat zyada detailed exploration ke liye, `unsafe` ke liye Rust ki official guide, [The Rustonomicon][nomicon], parhein.

[dangling-references]: ch04-02-references-and-borrowing.html#dangling-references
[ABI]: ../reference/items/external-blocks.html#abi
[constants]: ch03-01-variables-and-mutability.html#declaring-constants
[send-and-sync]: ch16-04-extensible-concurrency-sync-and-send.html
[the-slice-type]: ch04-03-slices.html#the-slice-type
[unions]: ../reference/items/unions.html
[miri]: https://github.com/rust-lang/miri
[editions]: appendix-05-editions.html
[nightly]: appendix-07-nightly-rust.html
[nomicon]: https://doc.rust-lang.org/nomicon/
