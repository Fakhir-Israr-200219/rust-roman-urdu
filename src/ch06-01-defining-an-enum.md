## Enum Define Karna

Jahan structs aapko related fields aur data ko ek saath group karne ka tareeqa deti hain, jaise ek `Rectangle` jisme uski `width` aur `height` hoti hain, wahin enums aapko ye kehne ka tareeqa deti hain ke koi value mumkin values ke ek set mein se kisi ek value hai. Misal ke taur par, hum kehna chah sakte hain ke `Rectangle` mumkin shapes ke ek set mein se ek hai, jisme `Circle` aur `Triangle` bhi shamil hain. Is ke liye, Rust humein in possibilities ko enum ke taur par encode karne deti hai.

Aaiye ek aisi situation dekhte hain jise hum code mein express karna chah sakte hain aur samajhte hain ke is case mein enums kyun useful aur structs ke muqable mein zyada munasib hain. Maan lein humein IP addresses ke saath kaam karna hai. Filhaal IP addresses ke liye do major standards use hote hain: version four aur version six. Kyun ke yehi woh possibilities hain jo hamara program kisi IP address ke liye encounter karega, hum tamam mumkin variants ko *enumerate* kar sakte hain, aur isi wajah se iska naam enumeration pada hai.

Koi bhi IP address ya to version four address ho sakta hai ya version six address, lekin ek hi waqt mein dono nahi ho sakta. IP addresses ki ye property enum data structure ko munasib banati hai, kyun ke ek enum value sirf apne variants mein se ek ho sakti hai. Version four aur version six addresses bunyadi taur par phir bhi IP addresses hi hain, is liye jab code aisi situations handle kar raha ho jo kisi bhi qisam ke IP address par apply hoti hain, to unhein ek hi type ke taur par treat kiya jana chahiye.

Hum is concept ko code mein `IpAddrKind` enumeration define karke aur IP address ke mumkin kinds, `V4` aur `V6`, list karke express kar sakte hain. Ye enum ke variants hain:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:def}}
```

Ab `IpAddrKind` ek custom data type hai jise hum apne code mein doosri jagahon par use kar sakte hain.

### Enum Values

Hum `IpAddrKind` ke dono variants ke instances is tarah create kar sakte hain:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:instance}}
```

Note karein ke enum ke variants uske identifier ke andar namespaced hote hain, aur hum dono ko separate karne ke liye double colon use karte hain. Ye useful hai kyun ke ab dono values `IpAddrKind::V4` aur `IpAddrKind::V6` ek hi type ki hain: `IpAddrKind`. Ab hum, misal ke taur par, ek aisa function define kar sakte hain jo kisi bhi `IpAddrKind` ko leta ho:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn}}
```

Aur hum is function ko kisi bhi variant ke saath call kar sakte hain:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn_call}}
```

Enums ko use karne ke aur bhi zyada faide hain. Apni IP address type ke baare mein mazeed sochte hue, filhaal hamare paas actual IP address *data* store karne ka koi tareeqa nahi hai; humein sirf ye maloom hai ke ye kis *kind* ka hai. Chapter 5 mein structs ke baare mein abhi seekhne ke baad, aap shayad is problem ko structs ke zariye solve karne ki koshish karein, jaisa ke Listing 6-1 mein dikhaya gaya hai.

<Listing number="6-1" caption="Ek IP address ke data aur `IpAddrKind` variant ko `struct` ke zariye store karna">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-01/src/main.rs:here}}
```

</Listing>

Yahan hum ne `IpAddr` naam ki ek struct define ki hai jisme do fields hain: ek `kind` field jo `IpAddrKind` type ki hai (yani woh enum jo hum ne pehle define ki thi), aur ek `address` field jo `String` type ki hai. Hamare paas is struct ke do instances hain. Pehla `home` hai, aur iske `kind` ki value `IpAddrKind::V4` hai, jiske saath address data `127.0.0.1` associated hai. Doosra instance `loopback` hai. Iske `kind` ki value `IpAddrKind` ka doosra variant, `V6`, hai, aur iske saath `::1` address associated hai. Hum ne `kind` aur `address` values ko ek saath bundle karne ke liye struct use ki hai, is liye ab variant value ke saath associated hai.

Lekin isi concept ko sirf ek enum use karke represent karna zyada concise hai: Struct ke andar enum rakhne ke bajaye, hum data ko directly har enum variant mein rakh sakte hain. `IpAddr` enum ki ye nayi definition kehti hai ke `V4` aur `V6` dono variants ke saath `String` values associated hongi:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-02-enum-with-data/src/main.rs:here}}
```

Hum data ko directly enum ke har variant ke saath attach kar dete hain, is liye kisi extra struct ki zaroorat nahi rehti. Yahan enums ke kaam karne ke tareeqe ki ek aur detail dekhna bhi aasaan hai: Har enum variant ka woh name jo hum define karte hain, ek aisa function bhi ban jata hai jo enum ka ek instance construct karta hai. Yani, `IpAddr::V4()` ek function call hai jo ek `String` argument leti hai aur `IpAddr` type ka ek instance return karti hai. Enum define karne ke result mein humein ye constructor function automatically mil jata hai.

Struct ke bajaye enum use karne ka ek aur faida hai: Har variant ke saath different types aur different amount ka associated data ho sakta hai. Version four IP addresses mein hamesha chaar numeric components honge jin ki values 0 aur 255 ke darmiyan hongi. Agar hum `V4` addresses ko chaar `u8` values ke taur par store karna chahte hon, lekin `V6` addresses ko ek `String` value ke taur par express karna chahte hon, to hum struct ke saath aisa nahi kar sakte. Enums is situation ko aasani se handle karti hain:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-03-variants-with-different-data/src/main.rs:here}}
```

Hum ne version four aur version six IP addresses ko store karne ke liye data structures define karne ke kai different tareeqe dekhe hain. Lekin, jaisa ke pata chalta hai, IP addresses ko store karna aur ye encode karna ke woh kis kind ke hain itna common hai ke [standard library mein ek definition mojood hai jise hum use kar sakte hain!][IpAddr]<!-- ignore --> Aaiye dekhte hain ke standard library `IpAddr` ko kaise define karti hai. Is mein bilkul wohi enum aur variants hain jo hum ne define aur use kiye hain, lekin ye address data ko variants ke andar do different structs ki form mein embed karti hai, jo har variant ke liye differently define kiye gaye hain:

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

Ye code illustrate karta hai ke aap enum variant ke andar kisi bhi qisam ka data rakh sakte hain: misal ke taur par strings, numeric types, ya structs. Aap ek aur enum bhi include kar sakte hain! Is ke ilawa, standard library ki types aksar us se zyada complicated nahi hoti jo aap khud create kar sakte hain.

Note karein ke halanke standard library mein `IpAddr` ki ek definition mojood hai, hum phir bhi apni definition create aur use kar sakte hain baghair kisi conflict ke, kyun ke hum ne standard library ki definition ko apne scope mein nahi laya. Types ko scope mein lane ke baare mein hum Chapter 7 mein mazeed baat karenge.

Aaiye Listing 6-2 mein enum ki ek aur example dekhte hain: Is mein iske variants ke andar bohat mukhtalif types embedded hain.

<Listing number="6-2" caption="Ek `Message` enum jiske variants mein different amounts aur types ki values store hoti hain">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

</Listing>

Is enum mein different types ke chaar variants hain:

* `Quit`: Is ke saath koi data associated nahi hai.
* `Move`: Is mein named fields hain, bilkul struct ki tarah.
* `Write`: Is mein ek `String` shamil hai.
* `ChangeColor`: Is mein teen `i32` values shamil hain.

Listing 6-2 mein diye gaye variants jaisi enum define karna different qisam ki struct definitions define karne ke similar hai, siwaye is ke ke enum `struct` keyword use nahi karti aur tamam variants ko `Message` type ke andar ek saath group kiya jata hai. Following structs wohi data hold kar sakti hain jo pichli enum ke variants hold karte hain:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-04-structs-similar-to-message-enum/src/main.rs:here}}
```

Lekin agar hum different structs use karte, jin mein se har ek ki apni type hoti, to hum in mein se kisi bhi qisam ke message ko lene wala function utni aasani se define nahi kar pate jitni aasani se Listing 6-2 mein define ki gayi `Message` enum ke saath kar sakte hain, kyun ke `Message` ek single type hai.

Enums aur structs ke darmiyan ek aur similarity hai: Jis tarah hum `impl` use karke structs par methods define kar sakte hain, usi tarah hum enums par bhi methods define kar sakte hain. Yahan `call` naam ka ek method hai jo hum apni `Message` enum par define kar sakte hain:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-05-methods-on-enums/src/main.rs:here}}
```

Method ka body `self` ko use karega taa-ke woh value hasil ki ja sake jis par hum ne method call kiya tha. Is example mein, hum ne ek variable `m` create kiya hai jis ki value `Message::Write(String::from("hello"))` hai, aur jab `m.call()` run hoga to `call` method ke body mein `self` ki value yahi hogi.

Aaiye ab standard library mein mojood ek aur enum ko dekhte hain jo bohat common aur useful hai: `Option`.

<!-- Old headings. Do not remove or links may break. -->

<a id="the-option-enum-and-its-advantages-over-null-values"></a>

### The `Option` Enum

Is section mein hum `Option` ka case study explore karenge, jo standard library ki taraf se define ki gayi ek aur enum hai. `Option` type ek bohat common situation ko encode karti hai jisme koi value ya to kuch ho sakti hai, ya phir kuch bhi nahi ho sakti.

Misal ke taur par, agar aap ek non-empty list mein pehli item request karein, to aapko ek value milegi. Agar aap ek empty list mein pehli item request karein, to aapko kuch nahi milega. Is concept ko type system ke hawale se express karne ka matlab hai ke compiler check kar sakta hai ke aap ne tamam un cases ko handle kiya hai jinhein aapko handle karna chahiye; ye functionality un bugs ko prevent kar sakti hai jo doosri programming languages mein bohat common hain.

Programming language design ke baare mein aksar ye socha jata hai ke aap kaun se features include karte hain, lekin jin features ko aap exclude karte hain woh bhi important hote hain. Rust mein woh `null` feature nahi hai jo bohat si doosri languages mein hota hai. *Null* ek aisi value hai jo ye mean karti hai ke wahan koi value mojood nahi hai. Null wali languages mein variables hamesha do states mein se kisi ek mein ho sakte hain: null ya not-null.

2009 ki apni presentation “Null References: The Billion Dollar Mistake” mein, Tony Hoare, jo null ke inventor hain, ne kaha:

> Main ise apni billion-dollar mistake kehta hoon. Us waqt, main object-oriented language mein references ke liye pehla comprehensive type system design kar raha tha. Mera goal ye ensure karna tha ke references ka tamam use bilkul safe ho, aur checking compiler ke zariye automatically perform ho. Lekin main null reference shamil karne ke temptation ko resist nahi kar saka, sirf is liye ke ise implement karna bohat aasaan tha. Is ki wajah se be-shumar errors, vulnerabilities, aur system crashes hue hain, jin ki wajah se pichlay chalis saalon mein shayad ek billion dollars ka dard aur nuqsan hua hai.

Null values ke saath problem ye hai ke agar aap null value ko not-null value ki tarah use karne ki koshish karein, to aapko kisi qisam ka error milega. Kyun ke ye null ya not-null property har jagah mojood hoti hai, is qisam ki ghalti karna bohat aasaan hai.

Lekin jo concept null express karne ki koshish karta hai, woh phir bhi useful hai: Null ek aisi value hai jo kisi wajah se filhaal invalid ya absent hai.

Masla asal mein concept ke saath nahi, balki uski particular implementation ke saath hai. Isi liye Rust mein nulls nahi hain, lekin Rust mein ek enum hai jo value ke present ya absent hone ke concept ko encode kar sakti hai. Ye enum `Option<T>` hai, aur ye [standard library ki taraf se define][option]<!-- ignore --> ki gayi hai:

```rust
enum Option<T> {
    None,
    Some(T),
}
```

`Option<T>` enum itni useful hai ke ye prelude mein bhi included hai; aapko ise explicitly scope mein lane ki zaroorat nahi hai. Is ke variants bhi prelude mein included hain: Aap `Option::` prefix ke baghair directly `Some` aur `None` use kar sakte hain. `Option<T>` enum phir bhi ek regular enum hi hai, aur `Some(T)` aur `None` ab bhi `Option<T>` type ke variants hain.

`<T>` syntax Rust ka ek feature hai jis ke baare mein hum ne abhi tak baat nahi ki. Ye ek generic type parameter hai, aur hum Chapter 10 mein generics ko mazeed detail mein cover karenge. Filhaal aapko sirf itna maloom hona chahiye ke `<T>` ka matlab hai ke `Option` enum ka `Some` variant kisi bhi type ka ek piece of data hold kar sakta hai, aur `T` ki jagah use hone wali har concrete type overall `Option<T>` type ko ek different type bana deti hai. Yahan `Option` values ko number types aur char types hold karne ke liye use karne ki kuch examples hain:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-06-option-examples/src/main.rs:here}}
```

`some_number` ki type `Option<i32>` hai. `some_char` ki type `Option<char>` hai, jo ek different type hai. Rust in types ko infer kar sakta hai kyun ke hum ne `Some` variant ke andar ek value specify ki hai. `absent_number` ke liye Rust hum se overall `Option` type annotate karne ka taqaza karta hai: Compiler sirf `None` value ko dekh kar ye infer nahi kar sakta ke corresponding `Some` variant kis type ki value hold karega. Yahan hum Rust ko batate hain ke hum chahte hain `absent_number` ki type `Option<i32>` ho.

Jab hamare paas `Some` value hoti hai, to humein maloom hota hai ke ek value present hai, aur woh value `Some` ke andar held hoti hai. Jab hamare paas `None` value hoti hai, to ek maayne mein iska matlab null jaisa hi hai: Hamare paas ek valid value nahi hai. To phir `Option<T>` rakhna null rakhne se behtar kyun hai?

Mukhtasar jawab ye hai ke `Option<T>` aur `T` (jahan `T` koi bhi type ho sakti hai) different types hain, is liye compiler humein `Option<T>`value ko aise use nahi karne dega jaise woh definitely ek valid value ho. Misal ke taur par, ye code compile nahi hoga, kyun ke ye ek`i8`ko`Option<i8>` ke saath add karne ki koshish kar raha hai:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/src/main.rs:here}}
```

Agar hum ye code run karein, to humein is tarah ka error message milega:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/output.txt}}
```

Intense! Asal mein, is error message ka matlab hai ke Rust ko samajh nahi aa raha ke ek `i8` aur ek `Option<i8>` ko kaise add kiya jaye, kyun ke ye different types hain. Jab Rust mein hamare paas `i8` jaisi type ki value hoti hai, to compiler ensure karega ke hamare paas hamesha ek valid value ho. Hum us value ko use karne se pehle null check kiye baghair confidence ke saath aage barh sakte hain. Sirf us waqt jab hamare paas `Option<i8>` (ya jo bhi type ki value hum use kar rahe hon) ho, humein is baat ki fikr karni hoti hai ke shayad koi value mojood na ho, aur compiler ensure karega ke value ko use karne se pehle hum us case ko handle karein.

Doosre alfaaz mein, `Option<T>` ke saath `T` operations perform karne se pehle aapko `Option<T>` ko `T` mein convert karna hota hai. Aam tor par, ye null ke saath hone wale sab se common issues mein se ek ko pakarne mein madad karta hai: Ye assume kar lena ke koi cheez null nahi hai jabke asal mein woh null ho.

Not-null value ko ghalat tareeqe se assume karne ke risk ko khatam karna aapko apne code ke baare mein zyada confident hone mein madad karta hai. Agar aap aisi value rakhna chahte hain jo mumkin taur par null ho sakti hai, to aapko explicitly opt in karna hota hai aur us value ki type `Option<T>` banani hoti hai. Phir, jab aap us value ko use karte hain, to aapko explicitly us case ko handle karna hota hai jab value null ho. Jahan bhi kisi value ki type `Option<T>` nahi hai, aap *safely* assume kar sakte hain ke value null nahi hai. Rust ke liye ye ek deliberate design decision tha taa-ke null ki pervasiveness ko limit kiya ja sake aur Rust code ki safety ko barhaya ja sake.

To phir jab aapke paas `Option<T>` type ki value ho, to aap `Some` variant ke andar se `T` value kaise hasil karte hain taa-ke aap us value ko use kar saken? `Option<T>` enum mein bohat badi tadaad mein methods hain jo mukhtalif situations mein useful hoti hain; aap inhein [iski documentation][docs]<!-- ignore --> mein dekh sakte hain. `Option<T>` ke methods se waqif hona aapke Rust ke safar mein bohat useful hoga.

Aam tor par, `Option<T>` value ko use karne ke liye aap aisa code rakhna chahenge jo har variant ko handle kare. Aapke paas kuch aisa code hona chahiye jo sirf us waqt run ho jab aapke paas `Some(T)` value ho, aur ye code andar mojood `T` ko use kar sake. Aapke paas kuch doosra code bhi hona chahiye jo sirf us waqt run ho jab aapke paas `None` value ho, aur us code ke paas koi `T` value available nahi hoti. `match` expression ek control flow construct hai jo enums ke saath use kiye jane par bilkul yahi kaam karti hai: Ye is baat ke mutabiq different code run karti hai ke enum ka kaunsa variant uske paas hai, aur woh code matching value ke andar mojood data ko use kar sakta hai.

[IpAddr]: ../std/net/enum.IpAddr.html
[option]: ../std/option/enum.Option.html
[docs]: ../std/option/enum.Option.html
