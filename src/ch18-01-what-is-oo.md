## Characteristics of Object-Oriented Languages

Programming community mein is baat par koi consensus nahi hai ke kisi language ko object oriented samajhne ke liye us mein kaun se features hona zaroori hain. Rust bohot se programming paradigms se influenced hai, jin mein OOP bhi shamil hai; misal ke taur par, humne Chapter 13 mein functional programming se aane wale features explore kiye thay. Yeh kaha ja sakta hai ke OOP languages kuch common characteristics share karti hain—yani, objects, encapsulation, aur inheritance. Aaiye dekhein ke in mein se har characteristic ka kya matlab hai aur kya Rust ise support karta hai.

### Objects Contain Data and Behavior

Erich Gamma, Richard Helm, Ralph Johnson, aur John Vlissides (Addison-Wesley, 1994) ki book *Design Patterns: Elements of Reusable Object-Oriented Software*, jise aam tor par *The Gang of Four* book kaha jata hai, object-oriented design patterns ka ek catalog hai. Yeh OOP ko is tarah define karti hai:

> Object-oriented programs objects se mil kar bante hain. Ek **object** data aur us data par operate karne wale procedures, dono ko package karta hai. In procedures ko aam tor par **methods** ya **operations** kaha jata hai.

Is definition ko use karte hue, Rust object oriented hai: Structs aur enums mein data hota hai, aur `impl` blocks structs aur enums par methods provide karte hain. Halanke methods wale structs aur enums ko *objects* nahi kaha jata, lekin Gang of Four ki objects wali definition ke mutabiq, yeh wohi functionality provide karte hain.

### Encapsulation That Hides Implementation Details

OOP ke saath aam tor par associate kiya jane wala ek aur aspect *encapsulation* ka idea hai, jis ka matlab hai ke kisi object ki implementation details us object ko use karne wale code ke liye accessible nahi hotin. Is liye, object ke saath interact karne ka sirf ek tareeqa us ki public API ke zariye hota hai; object ko use karne wala code object ke internals tak pohanch kar data ya behavior ko directly change nahi kar sakta. Is se programmer ko yeh sahulat milti hai ke woh object ke internals ko change aur refactor kar sake, baghair is ke ke us code ko change karna pade jo object ko use karta hai.

Humne Chapter 7 mein discuss kiya tha ke encapsulation ko kis tarah control kiya jata hai: Hum `pub` keyword ko use karke decide kar sakte hain ke hamare code mein kaun se modules, types, functions, aur methods public hone chahiye, aur default tor par baqi sab kuch private hota hai. Misal ke taur par, hum ek struct `AveragedCollection` define kar sakte hain jis mein ek field `i32` values ke vector ko contain karti hai. Struct mein ek aisi field bhi ho sakti hai jo vector ki values ka average contain kare, jis ka matlab hai ke jab bhi kisi ko average ki zaroorat ho, us waqt average calculate karna zaroori nahi hoga. Doosre lafzon mein, `AveragedCollection` hamare liye calculated average ko cache karega. Listing 18-1 mein `AveragedCollection` struct ki definition hai.

<Listing number="18-1" file-name="src/lib.rs" caption="An `AveragedCollection` struct that maintains a list of integers and the average of the items in the collection">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-01/src/lib.rs}}
```

</Listing>

Struct ko `pub` mark kiya gaya hai taake doosra code ise use kar sake, lekin struct ke andar ki fields private rehti hain. Is case mein yeh important hai kyun ke hum yeh ensure karna chahte hain ke jab bhi list mein koi value add ya remove ho, average bhi update ho. Hum yeh struct par `add`, `remove`, aur `average` methods implement karke karte hain, jaisa ke Listing 18-2 mein dikhaya gaya hai.

<Listing number="18-2" file-name="src/lib.rs" caption="Implementations of the public methods `add`, `remove`, and `average` on `AveragedCollection`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-02/src/lib.rs:here}}
```

</Listing>

Public methods `add`, `remove`, aur `average` hi `AveragedCollection` ke kisi instance mein data ko access ya modify karne ke tareeqe hain. Jab `add` method ko use karke `list` mein koi item add kiya jata hai ya `remove` ko use karke remove kiya jata hai, to har call ki implementation private `update_average` method ko call karti hai, jo `average` field ko bhi update karne ka kaam karti hai.

Hum `list` aur `average` fields ko private rakhte hain taake external code ke paas `list` field mein directly items add ya remove karne ka koi tareeqa na ho; warna jab `list` change hoti, to `average` field aur `list` ke darmiyan consistency khatam ho sakti thi. `average` method `average` field ki value return karta hai, jis se external code `average` ko read kar sakta hai lekin modify nahi kar sakta.

Kyun ke humne struct `AveragedCollection` ki implementation details ko encapsulate kiya hai, is liye hum future mein is ke aspects, jaise data structure, ko aasani se change kar sakte hain. Misal ke taur par, hum `list` field ke liye `Vec<i32>` ke bajaye `HashSet<i32>` use kar sakte hain. Jab tak public methods `add`, `remove`, aur `average` ke signatures same rehte hain, `AveragedCollection` ko use karne wale code ko change karne ki zaroorat nahi hogi. Agar hum `list` ko public kar dete, to zaroori nahi ke aisa hi hota: `HashSet<i32>` aur `Vec<i32>` mein items add aur remove karne ke liye different methods hain, is liye agar external code `list` ko directly modify kar raha hota to us code ko likely change karna padta.

Agar encapsulation kisi language ko object oriented samajhne ke liye required aspect hai, to Rust is requirement ko meet karta hai. Code ke different parts ke liye `pub` use karne ya na karne ka option implementation details ki encapsulation ko possible banata hai.

### Inheritance as a Type System and as Code Sharing

*Inheritance* ek mechanism hai jis ke zariye koi object kisi doosre object ki definition se elements inherit kar sakta hai, aur is tarah parent object ka data aur behavior hasil kar leta hai, baghair is ke ke aapko unhein dobara define karna pade.

Agar kisi language ko object oriented hone ke liye inheritance ka hona zaroori ho, to Rust aisi language nahi hai. Aisa koi tareeqa nahi hai ke kisi macro ko use kiye baghair ek struct ko is tarah define kiya ja sake ke woh parent struct ki fields aur method implementations inherit kar le.

Lekin agar aap apne programming toolbox mein inheritance rakhne ke aadhi hain, to aap Rust mein doosre solutions use kar sakte hain, jo is baat par depend karte hain ke aapne shuru mein inheritance ko use karne ka faisla kyun kiya tha.

Aap do main reasons ki wajah se inheritance choose karenge. Ek code ke reuse ke liye hai: Aap ek particular behavior ko ek type ke liye implement kar sakte hain, aur inheritance aapko us implementation ko kisi different type ke liye reuse karne deti hai. Rust code mein aap default trait method implementations ko use karke limited way mein yeh kar sakte hain, jaisa ke aapne Listing 10-14 mein dekha tha jab humne `Summary` trait par `summarize` method ki default implementation add ki thi. `Summary` trait ko implement karne wale kisi bhi type par `summarize` method available hoga, baghair kisi aur code ke. Yeh kuch had tak aisa hai jaise kisi parent class mein kisi method ki implementation ho aur inheriting child class par bhi us method ki implementation available ho. Hum `Summary` trait ko implement karte waqt `summarize` method ki default implementation ko override bhi kar sakte hain, jo kuch had tak aisa hai jaise child class parent class se inherited kisi method ki implementation ko override kar rahi ho.

Inheritance ko use karne ki doosri wajah type system se related hai: taake child type ko unhi jagahon par use kiya ja sake jahan parent type ko use kiya ja sakta hai. Isay *polymorphism* bhi kaha jata hai, jis ka matlab hai ke agar multiple objects kuch certain characteristics share karte hon, to aap runtime par unhein ek doosre ke substitute ke taur par use kar sakte hain.

> ### Polymorphism
>
> Bohot se logon ke liye, polymorphism inheritance ka synonym hai. Lekin asal mein yeh ek zyada general concept hai jo aise code ko refer karta hai jo multiple types ke data ke saath kaam kar sakta hai. Inheritance ke case mein, woh types aam tor par subclasses hoti hain.
>
> Rust is ke bajaye different possible types ko abstract karne ke liye generics aur yeh impose karne ke liye ke un types ko kya provide karna zaroori hai, trait bounds use karta hai. Isay kabhi kabhi *bounded parametric polymorphism* kaha jata hai.

Inheritance offer na karke Rust ne trade-offs ka ek different set choose kiya hai. Inheritance mein aksar zaroorat se zyada code share hone ka risk hota hai. Subclasses ko hamesha apni parent class ki tamam characteristics share nahi karni chahiye, lekin inheritance ke saath aisa hoga. Is se program ka design kam flexible ho sakta hai. Is se yeh possibility bhi paida hoti hai ke subclasses par aise methods call kiye jayein jo sense nahi banate ya errors cause karte hain kyun ke woh methods subclass par apply hi nahi hote. Is ke ilawa, kuch languages sirf *single inheritance* allow karti hain (yani, ek subclass sirf ek class se inherit kar sakti hai), jo program ke design ki flexibility ko aur restrict karta hai.

In reasons ki wajah se, Rust runtime par polymorphism achieve karne ke liye inheritance ke bajaye trait objects use karne ka different approach leta hai. Aaiye dekhein ke trait objects kis tarah work karte hain.
