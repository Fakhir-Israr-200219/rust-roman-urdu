<!-- Old headings. Do not remove or links may break. -->

<a id="closures-anonymous-functions-that-can-capture-their-environment"></a> <a id="closures-anonymous-functions-that-capture-their-environment"></a>

## Closures

Rust ke closures anonymous functions hote hain jinhein aap kisi variable mein save kar sakte hain ya doosre functions ko arguments ke taur par pass kar sakte hain. Aap closure ko ek jagah create kar sakte hain aur phir us closure ko kahin aur call karke different context mein evaluate kar sakte hain. Functions ke unlike, closures us scope se values capture kar sakte hain jahan woh define kiye gaye hain. Hum demonstrate karenge ke closures ki yeh features code reuse aur behavior customization ki sahulat kaise deti hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-an-abstraction-of-behavior-with-closures"></a> <a id="refactoring-using-functions"></a> <a id="refactoring-with-closures-to-store-code"></a> <a id="capturing-the-environment-with-closures"></a>


### Capturing the Environment

Sab se pehle, hum examine karenge ke closures ko us environment se values capture karne ke liye kaise use kiya ja sakta hai jahan woh define kiye gaye hain, taake unhein baad mein use kiya ja sake. Scenario yeh hai: Har kuch arsay baad, hamari T-shirt company promotion ke taur par apni mailing list mein se kisi ek person ko ek exclusive, limited-edition shirt free mein deti hai. Mailing list par maujood log optionally apne profile mein apna favorite color add kar sakte hain. Agar free shirt ke liye select hone wale person ka favorite color set hai, to unhein usi color ki shirt milti hai. Agar person ne favorite color specify nahi kiya, to unhein woh color diya jata hai jo company ke paas filhaal sab se zyada quantity mein hai.

Isay implement karne ke bohat se tareeqe hain. Is example ke liye, hum `ShirtColor` naam ka ek enum use karenge jismein `Red` aur `Blue` variants honge (simplicity ke liye available colors ki tadaad limited rakhi gayi hai). Company ki inventory ko hum `Inventory` struct se represent karte hain jismein `shirts` naam ka field hai jo `Vec<ShirtColor>` contain karta hai, jo currently stock mein maujood shirt colors ko represent karta hai. `Inventory` par defined `giveaway` method free-shirt winner ki optional shirt color preference leta hai aur woh shirt color return karta hai jo person ko milega. Yeh setup Listing 13-1 mein dikhaya gaya hai.

<Listing number="13-1" file-name="src/main.rs" caption="Shirt company giveaway situation">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-01/src/main.rs}}
```

</Listing>

`main` mein define ki gayi `store` mein is limited-edition promotion ke liye distribute karne ko do blue shirts aur ek red shirt baqi hain. Hum `giveaway` method ko ek aise user ke liye call karte hain jiski preference red shirt hai aur ek aise user ke liye jiski koi preference nahi hai.

Dobara, is code ko bohat se tareeqon se implement kiya ja sakta hai, aur yahan, closures par focus karne ke liye, hum un concepts tak hi limited rahe hain jo aap pehle hi seekh chuke hain, siwaye `giveaway` method ke body ke jo ek closure use karti hai. `giveaway` method mein, hum user preference ko `Option<ShirtColor>` type ke parameter ke taur par lete hain aur `user_preference` par `unwrap_or_else` method call karte hain. [`Option<T>` par `unwrap_or_else` method][unwrap-or-else]<!-- ignore --> standard library mein defined hai. Yeh ek argument leta hai: ek aisa closure jismein koi arguments nahi hote aur jo ek value `T` return karta hai (is case mein `Option<T>` ke `Some` variant mein stored same type `ShirtColor`). Agar `Option<T>` `Some` variant hai, to `unwrap_or_else` `Some` ke andar se value return karta hai. Agar `Option<T>` `None` variant hai, to `unwrap_or_else` closure ko call karta hai aur closure se return hone wali value return karta hai.

Hum `unwrap_or_else` ko argument ke taur par closure expression `|| self.most_stocked()` specify karte hain. Yeh ek aisa closure hai jo khud koi parameters nahi leta (agar closure ke parameters hote, to woh dono vertical pipes ke darmiyan appear hote). Closure ki body `self.most_stocked()` ko call karti hai. Hum yahan closure define kar rahe hain, aur `unwrap_or_else` ki implementation closure ko baad mein evaluate karegi agar result ki zarurat hui.

Is code ko run karne par yeh print hota hai:

```console
{{#include ../listings/ch13-functional-features/listing-13-01/output.txt}}
```

Yahan ek interesting aspect yeh hai ke humne ek aisa closure pass kiya hai jo current `Inventory` instance par `self.most_stocked()` call karta hai. Standard library ko `Inventory` ya `ShirtColor` types ke baare mein kuch bhi jaanne ki zarurat nahi thi jo humne define kiye hain, na hi us logic ke baare mein jo hum is scenario mein use karna chahte hain. Closure `self` `Inventory` instance ka ek immutable reference capture karta hai aur usay humare specify kiye gaye code ke saath `unwrap_or_else` method ko pass karta hai. Doosri taraf, functions apne environment ko is tarah capture karne ke qabil nahi hote.

<!-- Old headings. Do not remove or links may break. -->

<a id="closure-type-inference-and-annotation"></a>

### Inferring and Annotating Closure Types

Functions aur closures ke darmiyan aur bhi differences hain. Closures ko aam tor par parameters ke types ya return value ko annotate karne ki zarurat nahi hoti, jaisa ke `fn` functions mein hota hai. Functions par type annotations is liye required hoti hain kyun ke types ek explicit interface ka hissa hoti hain jo aapke users ke saamne expose hota hai. Is interface ko rigidly define karna is baat ko ensure karne ke liye important hai ke sab log is baat par agree karein ke function kis types ki values use karta hai aur return karta hai. Doosri taraf, closures ko is tarah ke exposed interface mein use nahi kiya jata: woh variables mein store hote hain aur unhein name kiye baghair use kiya jata hai, aur hamari library ke users ke saamne expose nahi kiya jata.

Closures aam tor par short hote hain aur sirf ek narrow context mein relevant hote hain, kisi bhi arbitrary scenario mein nahi. In limited contexts ke andar, compiler parameters aur return type ke types infer kar sakta hai, bilkul usi tarah jaise woh aksar variables ke types infer kar sakta hai (kuch rare cases hote hain jahan compiler ko closure type annotations ki bhi zarurat hoti hai).

Variables ki tarah, agar hum explicitness aur clarity barhana chahein to type annotations add kar sakte hain, lekin iski cost yeh hai ke code strictly necessary se zyada verbose ho jata hai. Closure ke types ko annotate karna Listing 13-2 mein dikhayi gayi definition jaisa hoga. Is example mein, hum closure ko define karke ek variable mein store kar rahe hain, bajaye iske ke closure ko usi jagah define karein jahan hum usay argument ke taur par pass karte hain, jaisa ke humne Listing 13-1 mein kiya tha.

<Listing number="13-2" file-name="src/main.rs" caption="Adding optional type annotations of the parameter and return value types in the closure">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-02/src/main.rs:here}}
```

</Listing>

Type annotations add hone ke baad, closures ka syntax functions ke syntax se zyada similar nazar aata hai. Yahan, comparison ke liye, hum ek aisa function define karte hain jo apne parameter mein 1 add karta hai aur ek aisa closure jo same behavior rakhta hai. Relevant parts ko align karne ke liye humne kuch spaces add ki hain. Yeh illustrate karta hai ke closure syntax function syntax jaisa hai, siwaye pipes ke use aur is baat ke ke syntax ka kitna hissa optional hai:

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

Pehli line function definition dikhati hai aur doosri line fully annotated closure definition dikhati hai. Teesri line mein, hum closure definition se type annotations remove kar dete hain. Chauthi line mein, hum brackets remove kar dete hain, jo optional hain kyun ke closure body mein sirf ek expression hai. Yeh tamam valid definitions hain jo call kiye jane par same behavior produce karengi. `add_one_v3` aur `add_one_v4` lines ko compile karne ke liye closures ko evaluate karna zaroori hai kyun ke types unke usage se infer honge. Yeh isi tarah hai jaise `let v = Vec::new();` ko Rust ke liye type infer karne ke liye ya to type annotations ki zarurat hoti hai ya phir `Vec` mein kisi type ki values insert karni hoti hain.

Closure definitions ke liye, compiler unke har parameter aur unki return value ke liye ek concrete type infer karega. Misal ke taur par, Listing 13-3 ek short closure ki definition dikhati hai jo sirf woh value return karta hai jo usay parameter ke taur par milti hai. Yeh closure is example ke maqsad ke ilawa zyada useful nahi hai. Note karein ke humne definition mein koi type annotations add nahi ki hain. Kyun ke koi type annotations nahi hain, hum closure ko kisi bhi type ke saath call kar sakte hain, jo humne yahan pehli baar `String` ke saath kiya hai. Agar phir hum `example_closure` ko integer ke saath call karne ki koshish karein, to humein ek error milega.

<Listing number="13-3" file-name="src/main.rs" caption="Attempting to call a closure whose types are inferred with two different types">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-03/src/main.rs:here}}
```

</Listing>

Compiler humein yeh error deta hai:

```console
{{#include ../listings/ch13-functional-features/listing-13-03/output.txt}}
```

Pehli baar jab hum `example_closure` ko `String` value ke saath call karte hain, compiler `x` ke type aur closure ki return type ko `String` infer karta hai. Phir yeh types `example_closure` mein closure ke liye lock ho jati hain, aur jab hum isi closure ke saath next time koi different type use karne ki koshish karte hain, to humein type error milta hai.

### Capturing References or Moving Ownership

Closures apne environment se values ko teen tareeqon se capture kar sakte hain, jo directly un teen tareeqon se match karte hain jin se ek function parameter le sakta hai: immutably borrow karna, mutably borrow karna, aur ownership lena. Closure in mein se kis tareeqe ko use karega, iska faisla is baat ki bunyaad par hota hai ke function ki body captured values ke saath kya karti hai.

Listing 13-4 mein, hum ek aisa closure define karte hain jo `list` naam ke vector ka immutable reference capture karta hai kyun ke isay value ko print karne ke liye sirf immutable reference ki zarurat hai.

<Listing number="13-4" file-name="src/main.rs" caption="Defining and calling a closure that captures an immutable reference">

```rust id="8l4xpj"
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-04/src/main.rs}}
```

</Listing>

Yeh example yeh bhi illustrate karta hai ke ek variable closure definition se bind ho sakta hai, aur hum baad mein variable name aur parentheses use karke closure ko call kar sakte hain, bilkul aise jaise variable name function ka naam ho.

Kyun ke hum ek hi waqt mein `list` ke multiple immutable references rakh sakte hain, is liye `list` closure definition se pehle wale code mein, closure definition ke baad lekin closure call hone se pehle, aur closure call hone ke baad bhi accessible rehta hai. Yeh code compile aur run hota hai aur yeh print karta hai:

```console id="xw2f2p"
{{#include ../listings/ch13-functional-features/listing-13-04/output.txt}}
```

Next, Listing 13-5 mein, hum closure body ko change karte hain taake yeh `list` vector mein ek element add kare. Ab closure ek mutable reference capture karta hai.

<Listing number="13-5" file-name="src/main.rs" caption="Defining and calling a closure that captures a mutable reference">

```rust id="7kqf6n"
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-05/src/main.rs}}
```

</Listing>

Yeh code compile aur run hota hai aur yeh print karta hai:

```console id="j9xk8s"
{{#include ../listings/ch13-functional-features/listing-13-05/output.txt}}
```

Note karein ke ab `borrows_mutably` closure ki definition aur call ke darmiyan koi `println!` nahi hai: Jab `borrows_mutably` define hota hai, to yeh `list` ka mutable reference capture karta hai. Closure ko call karne ke baad hum usay dobara use nahi karte, is liye mutable borrow end ho jata hai. Closure definition aur closure call ke darmiyan, print karne ke liye immutable borrow allowed nahi hai, kyun ke jab mutable borrow hota hai to koi doosra borrow allowed nahi hota. Wahan `println!` add karke dekhein ke aapko kya error message milta hai!

Agar aap closure ko force karna chahte hain ke woh environment mein use hone wali values ki ownership le, halaan ke closure ki body ko strictly ownership ki zarurat nahi hai, to aap parameter list se pehle `move` keyword use kar sakte hain.

Yeh technique zyada tar tab useful hoti hai jab kisi closure ko ek new thread ko pass kiya jata hai taake data ko move kiya ja sake aur woh new thread ke owned data mein ho. Hum threads aur unhein use karne ki wajah ke baare mein Chapter 16 mein detail se discuss karenge jab hum concurrency ki baat karenge, lekin filhaal, aaiye briefly ek new thread spawn karne ko explore karte hain jahan closure ko `move` keyword ki zarurat hoti hai. Listing 13-6 mein Listing 13-4 ko modify karke vector ko main thread ke bajaye ek new thread mein print kiya gaya hai.

<Listing number="13-6" file-name="src/main.rs" caption="Using `move` to force the closure for the thread to take ownership of `list`">

```rust id="l0s5bn"
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-06/src/main.rs}}
```

</Listing>

Hum ek new thread spawn karte hain aur thread ko ek closure argument ke taur par dete hain jise run karna hai. Closure body list ko print karti hai. Listing 13-4 mein, closure ne `list` ko sirf immutable reference ke zariye capture kiya tha kyun ke usay print karne ke liye `list` tak itni hi access ki zarurat thi. Is example mein, halaan ke closure body ko ab bhi sirf immutable reference ki zarurat hai, humein specify karna padta hai ke `list` ko closure mein move kiya jana chahiye, closure definition ke start mein `move` keyword rakh kar. Agar main thread new thread par `join` call karne se pehle mazeed operations karta, to new thread main thread ke baqi kaam complete hone se pehle finish ho sakta tha, ya main thread pehle finish ho sakta tha. Agar main thread `list` ki ownership maintain karta lekin new thread se pehle end ho jata aur `list` ko drop kar deta, to thread mein maujood immutable reference invalid ho jata. Is liye compiler require karta hai ke `list` ko new thread ko diye gaye closure mein move kiya jaye taake reference valid rahe. `move` keyword remove karke ya closure define hone ke baad main thread mein `list` ko use karke dekhein ke aapko compiler se kaun se errors milte hain!

<!-- Old headings. Do not remove or links may break. -->

<a id="storing-closures-using-generic-parameters-and-the-fn-traits"></a> <a id="limitations-of-the-cacher-implementation"></a> <a id="moving-captured-values-out-of-the-closure-and-the-fn-traits"></a> <a id="moving-captured-values-out-of-closures-and-the-fn-traits"></a>

### Moving Captured Values Out of Closures

Jab ek closure us environment se kisi reference ko capture kar leta hai ya kisi value ki ownership capture kar leta hai jahan closure define kiya gaya hai (aur is tarah yeh affect hota hai ke closure mein kya move *into* hota hai), to closure ki body ka code yeh define karta hai ke baad mein jab closure evaluate kiya jata hai to references ya values ke saath kya hota hai (aur is tarah yeh affect hota hai ke closure se kya move *out of* hota hai).

Closure body in mein se koi bhi kaam kar sakti hai: captured value ko closure se bahar move karna, captured value ko mutate karna, na value ko move karna aur na mutate karna, ya shuru se environment se kuch bhi capture na karna.

Closure jis tarah environment se values ko capture aur handle karta hai, woh affect karta hai ke closure kaun se traits implement karta hai, aur traits woh tareeqa hain jin se functions aur structs specify kar sakte hain ke woh kis qisam ke closures use kar sakte hain. Closures automatically in teen `Fn` traits mein se ek, do, ya teeno implement karenge, additive fashion mein, jo is baat par depend karta hai ke closure ki body values ko kis tarah handle karti hai:

* `FnOnce` un closures par apply hota hai jinhein sirf ek baar call kiya ja sakta hai. Tamam closures kam az kam is trait ko implement karte hain kyun ke tamam closures ko call kiya ja sakta hai. Aisa closure jo captured values ko apni body se bahar move karta hai, sirf `FnOnce` implement karega aur doosre `Fn` traits mein se koi nahi, kyun ke usay sirf ek baar call kiya ja sakta hai.
* `FnMut` un closures par apply hota hai jo captured values ko apni body se bahar move nahi karte lekin captured values ko mutate kar sakte hain. In closures ko ek se zyada baar call kiya ja sakta hai.
* `Fn` un closures par apply hota hai jo captured values ko apni body se bahar move nahi karte aur captured values ko mutate bhi nahi karte, aur un closures par bhi jo apne environment se kuch bhi capture nahi karte. In closures ko apne environment ko mutate kiye baghair ek se zyada baar call kiya ja sakta hai, jo un cases mein important hai jahan ek closure ko multiple times concurrently call karna ho.

Aaiye `Option<T>` par `unwrap_or_else` method ki definition dekhte hain jo humne Listing 13-1 mein use ki thi:

```rust,ignore
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

Yaad rakhein ke `T` generic type hai jo `Option` ke `Some` variant mein value ke type ko represent karta hai. Woh type `T` `unwrap_or_else` function ka return type bhi hai: Misal ke taur par, jo code `Option<String>` par `unwrap_or_else` call karta hai, usay ek `String` milega.

Next, note karein ke `unwrap_or_else` function ke paas additional generic type parameter `F` hai. `F` type `f` naam ke parameter ka type hai, jo woh closure hai jo hum `unwrap_or_else` ko call karte waqt provide karte hain.

Generic type `F` par specified trait bound `FnOnce() -> T` hai, jis ka matlab hai ke `F` ko ek baar call kiya ja sakna chahiye, koi arguments nahi lene chahiye, aur ek `T` return karna chahiye. Trait bound mein `FnOnce` use karna is constraint ko express karta hai ke `unwrap_or_else`, `f` ko ek se zyada baar call nahi karega. `unwrap_or_else` ki body mein hum dekh sakte hain ke agar `Option` `Some` hai to `f` call nahi hoga. Agar `Option` `None` hai to `f` ek baar call hoga. Kyun ke tamam closures `FnOnce` implement karte hain, `unwrap_or_else` teenon qisam ke closures ko accept karta hai aur jitna flexible ho sakta hai utna flexible hai.

> Note: Agar jo hum karna chahte hain us mein environment se kisi value ko capture karna required nahi hai, to jahan humein aisi cheez chahiye jo `Fn` traits mein se kisi ek ko implement karti ho, wahan hum closure ke bajaye kisi function ka naam use kar sakte hain. Misal ke taur par, `Option<Vec<T>>` value par hum `unwrap_or_else(Vec::new)` call kar sakte hain taake agar value `None` ho to ek naya, empty vector mil jaye. Compiler automatically function definition ke liye applicable `Fn` traits mein se jo bhi trait ho usay implement karta hai.

Ab standard library method `sort_by_key` ko dekhte hain, jo slices par defined hai, taake samajh sakein ke yeh `unwrap_or_else` se kis tarah different hai aur `sort_by_key` trait bound ke liye `FnOnce` ke bajaye `FnMut` kyun use karta hai. Closure ko ek argument milta hai jo consider kiye ja rahe slice ke current item ka reference hota hai, aur yeh `K` type ki ek value return karta hai jise order kiya ja sakta hai. Yeh function tab useful hai jab aap kisi slice ko har item ke kisi particular attribute ki bunyaad par sort karna chahte hain. Listing 13-7 mein, hamare paas `Rectangle` instances ki ek list hai, aur hum `sort_by_key` ko use karke unhein unke `width` attribute ki bunyaad par low se high order mein arrange karte hain.

<Listing number="13-7" file-name="src/main.rs" caption="Using `sort_by_key` to order rectangles by width">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-07/src/main.rs}}
```

</Listing>

Yeh code print karta hai:

```console
{{#include ../listings/ch13-functional-features/listing-13-07/output.txt}}
```

`sort_by_key` ko `FnMut` closure lene ke liye define karne ki wajah yeh hai ke yeh closure ko multiple times call karta hai: slice ke har item ke liye ek baar. Closure `|r| r.width` apne environment se kuch capture, mutate, ya move out nahi karta, is liye yeh trait bound ki requirements ko meet karta hai.

Is ke baraks, Listing 13-8 ek aise closure ka example dikhati hai jo sirf `FnOnce` trait implement karta hai, kyun ke yeh environment se ek value ko move karta hai. Compiler humein is closure ko `sort_by_key` ke saath use karne nahi dega.

<Listing number="13-8" file-name="src/main.rs" caption="Attempting to use an `FnOnce` closure with `sort_by_key`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-08/src/main.rs}}
```

</Listing>

Yeh ek banawati, complicated tareeqa hai (jo kaam nahi karta) `sort_by_key` ke closure ko `list` ko sort karte waqt kitni baar call karne ki counting karne ki koshish ka. Yeh code `value` ko push karke counting karne ki koshish karta hai—`value` closure ke environment se ek `String` hai—`sort_operations` vector mein. Closure `value` ko capture karta hai aur phir `value` ki ownership `sort_operations` vector ko transfer karke `value` ko closure se bahar move kar deta hai. Is closure ko ek baar call kiya ja sakta hai; ise doosri baar call karne ki koshish kaam nahi karegi, kyun ke `value` ab environment mein maujood nahi hoga jise dobara `sort_operations` mein push kiya ja sake! Is liye yeh closure sirf `FnOnce` implement karta hai. Jab hum is code ko compile karne ki koshish karte hain, to humein yeh error milta hai ke `value` ko closure se bahar move nahi kiya ja sakta kyun ke closure ko `FnMut` implement karna zaroori hai:

```console
{{#include ../listings/ch13-functional-features/listing-13-08/output.txt}}
```

Error closure body ki us line ki taraf point karta hai jo `value` ko environment se bahar move karti hai. Isay fix karne ke liye, humein closure body ko is tarah change karna hoga ke yeh environment se values ko bahar move na kare. Environment mein ek counter rakhna aur closure body mein uski value ko increment karna, closure ko kitni baar call kiya gaya hai iski counting ka zyada straightforward tareeqa hai. Listing 13-9 mein closure `sort_by_key` ke saath kaam karta hai kyun ke yeh sirf `num_sort_operations` counter ka mutable reference capture karta hai aur is liye isay ek se zyada baar call kiya ja sakta hai.

<Listing number="13-9" file-name="src/main.rs" caption="Using an `FnMut` closure with `sort_by_key` is allowed.">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-09/src/main.rs}}
```

</Listing>

`Fn` traits un functions ya types ko define ya use karte waqt important hain jo closures ka use karte hain. Agle section mein, hum iterators discuss karenge. Bohat se iterator methods closure arguments lete hain, is liye aage barhte hue closure ki in details ko zehan mein rakhein!

[unwrap-or-else]: ../std/option/enum.Option.html#method.unwrap_or_else
