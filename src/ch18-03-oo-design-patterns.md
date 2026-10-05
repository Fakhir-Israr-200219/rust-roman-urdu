## Implementing an Object-Oriented Design Pattern

*The state pattern* ek object-oriented design pattern hai. Is pattern ka crux yeh hai ke hum internally un states ka ek set define karte hain jo kisi value ki ho sakti hain. States ko *state objects* ke ek set ke zariye represent kiya jata hai, aur value ka behavior us ki state ki bunyaad par change hota hai. Hum ek blog post struct ki example par kaam karenge jis mein apni state ko hold karne ke liye ek field hogi, aur yeh state “draft,” “review,” ya “published” ke set mein se ek state object hogi.

State objects functionality share karte hain: Rust mein, of course, hum objects aur inheritance ke bajaye structs aur traits use karte hain. Har state object apne behavior ka zimmedar hota hai aur yeh govern karta hai ke use kab doosri state mein change hona chahiye. Jo value state object ko hold karti hai, woh states ke different behavior ya states ke darmiyan transition hone ke waqt ke bare mein kuch nahi jaanti.

State pattern use karne ka faida yeh hai ke jab program ki business requirements change hoti hain, to humein state ko hold karne wali value ke code ya value ko use karne wale code ko change karne ki zaroorat nahi hoti. Humein sirf kisi ek state object ke andar ke code ko update karna hota hai taake us ke rules change kiye ja sakein, ya shayad mazeed state objects add kiye ja sakein.

Sab se pehle, hum state pattern ko ek zyada traditional object-oriented tareeqe se implement karenge. Phir, hum ek aisa approach use karenge jo Rust mein thora zyada natural hai. Aaiye state pattern ko use karke blog post workflow ko incrementally implement karte hain.

Final functionality kuch is tarah hogi:

1. Ek blog post ek empty draft ke taur par start hoti hai.
2. Jab draft complete ho jata hai, to post ka review request kiya jata hai.
3. Jab post approve ho jati hai, to woh publish ho jati hai.
4. Sirf published blog posts print karne ke liye content return karti hain taake unapproved posts ghalti se publish na ho sakein.

Post par ki gayi koi bhi doosri attempted change koi effect nahi dalegi. Misal ke taur par, agar hum review request karne se pehle ek draft blog post ko approve karne ki koshish karein, to post unpublished draft hi rehni chahiye.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-traditional-object-oriented-attempt"></a>

### Attempting Traditional Object-Oriented Style

Ek hi problem ko solve karne ke liye code ko structure karne ke infinite tareeqe ho sakte hain, aur har tareeqe ke different trade-offs hote hain. Is section ki implementation zyada traditional object-oriented style ki hai, jo Rust mein likhna mumkin hai, lekin yeh Rust ki kuch strengths ka faida nahi uthati. Baad mein, hum ek different solution demonstrate karenge jo ab bhi object-oriented design pattern use karta hai, lekin is tarah structured hai jo object-oriented experience rakhne wale programmers ko shayad kam familiar lage. Hum dono solutions ko compare karenge taake Rust code ko doosri languages ke code se different tareeqe se design karne ke trade-offs ko experience kar sakein.

Listing 18-11 is workflow ko code ki form mein dikhati hai: Yeh `blog` naam ke library crate mein implement ki jane wali API ke desired use ka ek example hai. Yeh abhi compile nahi hoga kyun ke humne abhi `blog` crate ko implement nahi kiya hai.

<Listing number="18-11" file-name="src/main.rs" caption="Code that demonstrates the desired behavior we want our `blog` crate to have">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:all}}
```

</Listing>

Hum user ko `Post::new` ke saath ek nayi draft blog post create karne ki ijazat dena chahte hain. Hum blog post mein text add karne ki bhi ijazat dena chahte hain. Agar hum approval se foran pehle post ka content hasil karne ki koshish karein, to humein koi text nahi milna chahiye kyun ke post abhi draft hai. Humne demonstration ke maqsad ke liye code mein `assert_eq!` add kiya hai. Is ke liye ek excellent unit test yeh hoga ke assert kiya jaye ke draft blog post `content` method se ek empty string return karti hai, lekin hum is example ke liye tests nahi likhenge.

Is ke baad, hum post ke review ki request enable karna chahte hain, aur review ka wait karte waqt `content` ko empty string return karna chahiye. Jab post ko approval mil jaye, to use publish ho jana chahiye, jis ka matlab hai ke `content` call kiye jane par post ka text return hoga.

Notice karein ke crate se hum sirf ek type ke saath interact kar rahe hain, aur woh `Post` type hai. Yeh type state pattern use karega aur ek aisi value hold karega jo teen state objects mein se ek hogi, jo post ki mukhtalif states ko represent karte hain—draft, review, ya published. Ek state se doosri state mein change hona internally `Post` type ke andar manage kiya jayega. States hamare library ke users ke `Post` instance par call kiye gaye methods ke response mein change hoti hain, lekin unhein state changes ko directly manage karne ki zaroorat nahi hoti. Is ke ilawa, users states ke saath koi ghalti nahi kar sakte, jaise review hone se pehle post ko publish kar dena.

<!-- Old headings. Do not remove or links may break. -->

<a id="defining-post-and-creating-a-new-instance-in-the-draft-state"></a>

#### Defining `Post` and Creating a New Instance

Aaiye library ki implementation shuru karte hain! Hum jaante hain ke humein ek public `Post` struct chahiye jo kuch content hold kare, is liye hum struct ki definition aur ek associated public `new` function se shuru karenge jo `Post` ka instance create karega, jaisa ke Listing 18-12 mein dikhaya gaya hai. Hum ek private `State` trait bhi banayenge jo woh behavior define karega jo `Post` ke tamam state objects mein hona zaroori hai.

Phir, `Post` ek private field jiska naam `state` hai, us mein `Option<T>` ke andar `Box<dyn State>` ka trait object hold karega taake state object ko hold kiya ja sake. Thori dair mein aap dekhenge ke `Option<T>` kyun zaroori hai.

<Listing number="18-12" file-name="src/lib.rs" caption="Definition of a `Post` struct and a `new` function that creates a new `Post` instance, a `State` trait, and a `Draft` struct">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-12/src/lib.rs}}
```

</Listing>

`State` trait mukhtalif post states ke darmiyan shared behavior define karta hai. State objects `Draft`, `PendingReview`, aur `Published` hain, aur yeh tamam `State` trait ko implement karenge. Filhal, trait mein koi methods nahi hain, aur hum sirf `Draft` state ko define karne se shuru karenge kyun ke yahi woh state hai jismein hum chahte hain ke post start ho.

Jab hum ek naya `Post` create karte hain, to hum us ke `state` field ko ek `Some` value par set karte hain jo ek `Box` hold karti hai. Yeh `Box` `Draft` struct ke ek naye instance ki taraf point karta hai. Is se yeh ensure hota hai ke jab bhi hum `Post` ka naya instance create karein, woh draft ke taur par start ho. Kyun ke `Post` ka `state` field private hai, is liye `Post` ko kisi doosri state mein create karne ka koi tareeqa nahi hai! `Post::new` function mein, hum `content` field ko ek nayi, empty `String` par set karte hain.

#### Storing the Text of the Post Content

Humne Listing 18-11 mein dekha ke hum `add_text` naam ke ek method ko call karne aur use ek `&str` pass karne ke qabil hona chahte hain, jise phir blog post ke text content ke taur par add kiya jaye. Hum ise `content` field ko `pub` ke taur par expose karne ke bajaye ek method ke taur par implement karte hain, taake baad mein hum ek aisa method implement kar sakein jo control kare ke `content` field ka data kaise read kiya jata hai. `add_text` method kaafi straightforward hai, is liye aaiye Listing 18-13 mein implementation ko `impl Post` block mein add karte hain.

<Listing number="18-13" file-name="src/lib.rs" caption="Implementing the `add_text` method to add text to a post’s `content`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-13/src/lib.rs:here}}
```

</Listing>

`add_text` method `self` ka ek mutable reference leta hai kyun ke hum us `Post` instance ko change kar rahe hain jis par hum `add_text` call kar rahe hain. Phir hum `content` mein maujood `String` par `push_str` call karte hain aur saved `content` mein add karne ke liye `text` argument pass karte hain. Yeh behavior is baat par depend nahi karta ke post kis state mein hai, is liye yeh state pattern ka hissa nahi hai. `add_text` method `state` field ke saath bilkul interact nahi karta, lekin yeh us behavior ka hissa hai jise hum support karna chahte hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="ensuring-the-content-of-a-draft-post-is-empty"></a>

#### Ensuring That the Content of a Draft Post Is Empty

Ab jab humne `add_text` call karke apni post mein kuch content add kar diya hai, tab bhi hum chahte hain ke `content` method ek empty string slice return kare kyun ke post abhi draft state mein hai, jaisa ke Listing 18-11 mein pehle `assert_eq!` se dikhaya gaya hai. Filhal, aaiye `content` method ko sab se simple tareeqe se implement karte hain jo is requirement ko poora kare: hamesha ek empty string slice return karna. Baad mein hum ise change karenge jab hum post ki state change karne ki ability implement kar lenge taake use publish kiya ja sake. Abhi tak, posts sirf draft state mein ho sakti hain, is liye post ka content hamesha empty hona chahiye. Listing 18-14 is placeholder implementation ko dikhati hai.

<Listing number="18-14" file-name="src/lib.rs" caption="Adding a placeholder implementation for the `content` method on `Post` that always returns an empty string slice">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-14/src/lib.rs:here}}
```

</Listing>

Is `content` method ko add karne ke baad, Listing 18-11 mein pehle `assert_eq!` tak sab kuch intended tareeqe se kaam karta hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="requesting-a-review-of-the-post-changes-its-state"></a> <a id="requesting-a-review-changes-the-posts-state"></a>

#### Requesting a Review, Which Changes the Post’s State

Ab humein post ke review ki request karne ki functionality add karni hai, jo us ki state ko `Draft` se `PendingReview` mein change karni chahiye. Listing 18-15 is code ko dikhati hai.

<Listing number="18-15" file-name="src/lib.rs" caption="Implementing `request_review` methods on `Post` and the `State` trait">

```rust,noplayground id="p9g2rx"
{{#rustdoc_include ../listings/ch18-oop/listing-18-15/src/lib.rs:here}}
```

</Listing>

Hum `Post` ko `request_review` naam ka ek public method dete hain jo `self` ka mutable reference lega. Phir, hum `Post` ki current state par ek internal `request_review` method call karte hain, aur yeh doosra `request_review` method current state ko consume karta hai aur ek new state return karta hai.

Hum `State` trait mein `request_review` method add karte hain; ab woh tamam types jo is trait ko implement karte hain, unhein `request_review` method bhi implement karna hoga. Note karein ke method ke first parameter ke taur par `self`, `&self`, ya `&mut self` rakhne ke bajaye, humare paas `self: Box<Self>` hai. Is syntax ka matlab hai ke yeh method sirf us waqt valid hai jab ise us `Box` par call kiya jaye jo is type ko hold kar raha ho. Yeh syntax `Box<Self>` ki ownership leta hai, jis se old state invalid ho jati hai aur `Post` ki state value ek new state mein transform ho sakti hai.

Old state ko consume karne ke liye, `request_review` method ko state value ki ownership leni zaroori hai. Yahin `Post` ke `state` field mein maujood `Option` kaam aata hai: Hum `state` field mein se `Some` value lene ke liye `take` method call karte hain aur us ki jagah `None` chhor dete hain kyun ke Rust humein structs mein unpopulated fields rakhne nahi deta. Is se hum `Post` se `state` value ko borrow karne ke bajaye move kar sakte hain. Phir, hum post ki `state` value ko is operation ke result par set karenge.

Ownership hasil karne ke liye humein `state` ko temporarily `None` par set karna hota hai, bajaye is ke ke hum ise directly is tarah ke code se set karein: `self.state = self.state.request_review();`. Is se yeh ensure hota hai ke jab hum old state ko new state mein transform kar chuke hon, to `Post` old `state` value ko use nahi kar sakta.

`Draft` par `request_review` method ek naye `PendingReview` struct ka naya, boxed instance return karta hai, jo us state ko represent karta hai jab post review ka wait kar rahi hoti hai. `PendingReview` struct bhi `request_review` method implement karta hai, lekin koi transformation nahi karta. Is ke bajaye, yeh khud ko return karta hai kyun ke jab hum kisi aisi post par review ki request karein jo pehle hi `PendingReview` state mein ho, to use `PendingReview` state mein hi rehna chahiye.

Ab hum state pattern ke faide dekhna shuru kar sakte hain: `Post` par `request_review` method ki implementation us ki `state` value chahe jo bhi ho, same rehti hai. Har state apne rules ki khud zimmedar hoti hai.

Hum `Post` par `content` method ko filhal waise hi rehne denge, jo ek empty string slice return karta hai. Ab humare paas `PendingReview` state ke saath-saath `Draft` state mein bhi ek `Post` ho sakti hai, lekin hum `PendingReview` state mein bhi wahi behavior chahte hain. Listing 18-11 ab doosre `assert_eq!` call tak kaam karti hai!

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-the-approve-method-that-changes-the-behavior-of-content"></a> <a id="adding-approve-to-change-the-behavior-of-content"></a>

#### Adding `approve` to Change `content`'s Behavior

`approve` method `request_review` method ki tarah hi hoga: Yeh `state` ko us value par set karega jo current state batati hai ke approval milne par us ki state kya honi chahiye, jaisa ke Listing 18-16 mein dikhaya gaya hai.

<Listing number="18-16" file-name="src/lib.rs" caption="Implementing the `approve` method on `Post` and the `State` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-16/src/lib.rs:here}}
```

</Listing>

Hum `State` trait mein `approve` method add karte hain aur ek naya struct add karte hain jo `State` ko implement karta hai, yani `Published` state.

Jis tarah `PendingReview` par `request_review` kaam karta hai, usi tarah agar hum `Draft` par `approve` method call karein, to iska koi effect nahi hoga kyun ke `approve` `self` return karega. Jab hum `PendingReview` par `approve` call karte hain, to yeh `Published` struct ka ek naya, boxed instance return karta hai. `Published` struct `State` trait ko implement karta hai, aur `request_review` method aur `approve` method dono ke liye yeh khud ko return karta hai kyun ke un cases mein post ko `Published` state mein hi rehna chahiye.

Ab humein `Post` par `content` method ko update karna hoga. Hum chahte hain ke `content` se return hone wali value `Post` ki current state par depend kare, is liye hum `Post` ko apni `state` par defined `content` method ko delegate karwayenge, jaisa ke Listing 18-17 mein dikhaya gaya hai.

<Listing number="18-17" file-name="src/lib.rs" caption="Updating the `content` method on `Post` to delegate to a `content` method on `State`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-17/src/lib.rs:here}}
```

</Listing>

Kyun ke goal yeh hai ke in tamam rules ko un structs ke andar rakha jaye jo `State` ko implement karte hain, hum `state` mein maujood value par `content` method call karte hain aur post instance (yani `self`) ko argument ke taur par pass karte hain. Phir, hum woh value return karte hain jo `state` value par `content` method use karne se return hoti hai.

Hum `Option` par `as_ref` method call karte hain kyun ke hum `Option` ke andar maujood value ka reference chahte hain, value ki ownership nahi. Kyun ke `state` ek `Option<Box<dyn State>>` hai, jab hum `as_ref` call karte hain to `Option<&Box<dyn State>>` return hota hai. Agar hum `as_ref` call na karein, to humein error milega kyun ke function parameter ke borrowed `&self` se hum `state` ko move nahi kar sakte.

Phir hum `unwrap` method call karte hain, jo hum jaante hain kabhi panic nahi karega kyun ke hum jaante hain ke `Post` ke methods yeh ensure karte hain ke un methods ke complete hone par `state` mein hamesha ek `Some` value hogi. Yeh un cases mein se ek hai jin par humne Chapter 9 ke [“When You Have More Information Than the Compiler”][more-info-than-rustc]<!-- ignore --> section mein baat ki thi, jab hum jaante hain ke `None` value kabhi possible nahi hoti, chahe compiler yeh samajhne ke qabil na ho.

Is point par, jab hum `&Box<dyn State>` par `content` call karte hain, to `&` aur `Box` par deref coercion apply hoga, jis se aakhir mein `content` method us type par call hoga jo `State` trait ko implement karta hai. Is ka matlab hai ke humein `State` trait definition mein `content` add karna hoga, aur yahin hum yeh logic rakhenge ke hamare paas jo state hai us ki bunyaad par kaunsa content return karna hai, jaisa ke Listing 18-18 mein dikhaya gaya hai.

<Listing number="18-18" file-name="src/lib.rs" caption="Adding the `content` method to the `State` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-18/src/lib.rs:here}}
```

</Listing>

Hum `content` method ke liye ek default implementation add karte hain jo ek empty string slice return karti hai. Is ka matlab hai ke humein `Draft` aur `PendingReview` structs par `content` implement karne ki zaroorat nahi hai. `Published` struct `content` method ko override karega aur `post.content` mein maujood value return karega. Convenient hone ke bawajood, `State` par `content` method ka hona jo `Post` ka content determine karta hai, `State` ki responsibility aur `Post` ki responsibility ke darmiyan boundaries ko dhundla kar raha hai.

Note karein ke humein is method par lifetime annotations ki zaroorat hai, jaisa ke humne Chapter 10 mein discuss kiya tha. Hum `post` ka ek reference argument ke taur par le rahe hain aur us `post` ke ek part ka reference return kar rahe hain, is liye returned reference ki lifetime `post` argument ki lifetime se related hai.

Aur hum done hain—ab Listing 18-11 ka tamam code kaam karta hai! Humne blog post workflow ke rules ke saath state pattern implement kar liya hai. Rules se related logic `Post` mein har jagah scattered hone ke bajaye state objects mein rehta hai.

> ### Why Not An Enum?
>
> Shayad aap soch rahe honge ke humne mukhtalif possible post states ko variants ke taur par rakhne wale enum ko kyun use nahi kiya. Yeh yaqeenan ek possible solution hai; ise try karein aur final results ko compare karein taake dekhein ke aap ko kaunsa zyada pasand hai! Enum use karne ka ek nuqsan yeh hai ke har woh jagah jahan enum ki value check ki jati hai, wahan har possible variant ko handle karne ke liye ek `match` expression ya isi tarah ki koi cheez zaroori hogi. Yeh is trait object solution ke muqable mein zyada repetitive ho sakta hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="trade-offs-of-the-state-pattern"></a>

#### Evaluating the State Pattern

Humne dikhaya hai ke Rust different kinds ke behavior ko encapsulate karne ke liye object-oriented state pattern implement karne ke qabil hai, jo post ko har state mein hona chahiye. `Post` par methods ko mukhtalif behaviors ke bare mein kuch pata nahi hota. Jis tarah humne code ko organize kiya hai, us ki wajah se humein yeh jaanne ke liye sirf ek jagah dekhni hoti hai ke published post kis tarah ke different behaviors kar sakti hai: `Published` struct par `State` trait ki implementation.

Agar hum ek alternative implementation banate jo state pattern use nahi karti, to hum is ke bajaye `Post` par methods mein, ya hatta ke `main` code mein `match` expressions use kar sakte hain jo post ki state check karta hai aur un jagahon par behavior change karta hai. Is ka matlab hota ke post ke published state mein hone ke tamam implications ko samajhne ke liye humein kai jagahon par dekhna padta.

State pattern ke saath, `Post` methods aur woh jagah jahan hum `Post` ko use karte hain, unhein `match` expressions ki zaroorat nahi hoti, aur ek new state add karne ke liye humein sirf ek new struct add karna hota hai aur us ek struct par trait methods ko ek hi jagah implement karna hota hai.

State pattern use karne wali implementation mein mazeed functionality add karna easy hai. State pattern use karne wale code ko maintain karne ki simplicity dekhne ke liye, in mein se kuch suggestions try karein:

* Ek `reject` method add karein jo post ki state ko `PendingReview` se wapas `Draft` mein change kare.
* State ko `Published` mein change karne se pehle `approve` ko do calls ki zaroorat rakhein.
* Users ko sirf us waqt text content add karne ki ijazat dein jab post `Draft` state mein ho. Hint: state object ko is baat ka zimmedar banayein ke content mein kya change ho sakta hai, lekin `Post` ko modify karne ka zimmedar na banayein.

State pattern ka ek downside yeh hai ke, kyun ke states hi states ke darmiyan transitions implement karti hain, kuch states ek doosri ke saath coupled hoti hain. Agar hum `PendingReview` aur `Published` ke darmiyan ek aur state add karein, jaise `Scheduled`, to humein `PendingReview` ke code ko change karna padega taake woh us ke bajaye `Scheduled` mein transition kare. Agar `PendingReview` ko new state add hone par change karne ki zaroorat na hoti to kam kaam hota, lekin is ka matlab hota ke kisi doosre design pattern par switch karna.

Ek aur downside yeh hai ke humne kuch logic duplicate kiya hai. Kuch duplication ko khatam karne ke liye hum `State` trait par `request_review` aur `approve` methods ke default implementations banane ki koshish kar sakte hain jo `self` return karein. Lekin yeh kaam nahi karega: Jab `State` ko trait object ke taur par use kiya jata hai, to trait ko yeh exactly nahi pata hota ke concrete `self` kya hoga, is liye return type compile time par known nahi hota. (Yeh pehle mention kiye gaye dyn compatibility rules mein se ek hai.)

Doosri duplication mein `Post` par `request_review` aur `approve` methods ki similar implementations shamil hain. Dono methods `Post` ke `state` field ke saath `Option::take` use karte hain, aur agar `state` `Some` ho, to woh wrapped value ki usi method ki implementation ko delegate karte hain aur `state` field ki new value ko result par set kar dete hain. Agar `Post` par bohat se methods hote jo isi pattern ko follow karte, to hum repetition ko khatam karne ke liye ek macro define karne par ghour kar sakte hain (Chapter 20 ke [“Macros”][macros]<!-- ignore --> section ko dekhein).

Object-oriented languages ke liye jis tarah state pattern define kiya gaya hai, bilkul usi tarah implement karke hum Rust ki strengths ka utna poora faida nahi utha rahe jitna hum utha sakte hain. Aaiye kuch aisi changes dekhein jo hum `blog` crate mein kar sakte hain aur jo invalid states aur transitions ko compile-time errors mein badal sakti hain.


### Encoding States and Behavior as Types

Hum aap ko dikhayenge ke different set of trade-offs hasil karne ke liye state pattern ko dobara kaise socha ja sakta hai. States aur transitions ko completely encapsulate karne ke bajaye, taake outside code ko in ke bare mein koi knowledge na ho, hum states ko different types mein encode karenge. Natijatan, Rust ka type-checking system draft posts ko wahan use karne ki koshish ko rok dega jahan sirf published posts allowed hain, aur compiler error issue karega.

Aaiye Listing 18-11 mein `main` ke pehle part par ghour karte hain:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:here}}
```

</Listing>

Hum ab bhi `Post::new` ka use karke new posts ko draft state mein create karne aur post ke content mein text add karne ki ability enable karte hain. Lekin draft post par `content` method rakhne ke bajaye jo ek empty string return karta hai, hum ise is tarah banayenge ke draft posts mein `content` method bilkul nahi hoga. Is tarah, agar hum draft post ka content hasil karne ki koshish karein, to humein compiler error milega jo batayega ke yeh method exist nahi karta. Natije ke taur par, production mein draft post ke content ko ghalti se display karna hamare liye impossible hoga kyun ke woh code compile hi nahi hoga. Listing 18-19 ek `Post` struct aur `DraftPost` struct ki definition, aur dono par methods dikhati hai.

<Listing number="18-19" file-name="src/lib.rs" caption="A `Post` with a `content` method and a `DraftPost` without a `content` method">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-19/src/lib.rs}}
```

</Listing>

`Post` aur `DraftPost` dono structs mein ek private `content` field hai jo blog post ka text store karti hai. Ab structs mein `state` field nahi hai kyun ke hum state ki encoding ko structs ke types mein move kar rahe hain. `Post` struct ek published post ko represent karega, aur is mein ek `content` method hai jo `content` return karta hai.

Humare paas ab bhi `Post::new` function hai, lekin `Post` ka instance return karne ke bajaye, yeh `DraftPost` ka instance return karta hai. Kyun ke `content` private hai aur aisa koi function nahi hai jo `Post` return karta ho, is waqt `Post` ka instance create karna mumkin nahi hai.

`DraftPost` struct mein ek `add_text` method hai, is liye hum pehle ki tarah `content` mein text add kar sakte hain, lekin note karein ke `DraftPost` mein `content` method defined nahi hai! Ab program yeh ensure karta hai ke tamam posts draft posts ke taur par start hon, aur draft posts ka content display ke liye available nahi hota. In constraints se bachne ki koi bhi koshish compiler error ka sabab banegi.

<!-- Old headings. Do not remove or links may break. -->

<a id="implementing-transitions-as-transformations-into-different-types"></a>

To phir, humein published post kaise milegi? Hum is rule ko enforce karna chahte hain ke kisi draft post ko publish hone se pehle review aur approval se guzarna zaroori hai. Pending review state mein maujood post ko bhi abhi koi content display nahi karna chahiye. Aaiye ek aur struct, `PendingReviewPost`, add karke in constraints ko implement karte hain, `DraftPost` par `request_review` method define karte hain jo ek `PendingReviewPost` return kare, aur `PendingReviewPost` par ek `approve` method define karte hain jo ek `Post` return kare, jaisa ke Listing 18-20 mein dikhaya gaya hai.

<Listing number="18-20" file-name="src/lib.rs" caption="A `PendingReviewPost` that gets created by calling `request_review` on `DraftPost` and an `approve` method that turns a `PendingReviewPost` into a published `Post`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-20/src/lib.rs:here}}
```

</Listing>

`request_review` aur `approve` methods `self` ki ownership lete hain, is tarah `DraftPost` aur `PendingReviewPost` instances ko consume karte hain aur respectively unhein `PendingReviewPost` aur ek published `Post` mein transform karte hain. Is tarah, `request_review` call karne ke baad hamare paas koi lingering `DraftPost` instances nahi rahenge, aur isi tarah aage bhi. `PendingReviewPost` struct par `content` method defined nahi hai, is liye us ka content read karne ki koshish se bhi compiler error milega, bilkul `DraftPost` ki tarah. Kyun ke published `Post` instance hasil karne ka, jismein `content` method defined hai, sirf ek tareeqa `PendingReviewPost` par `approve` method call karna hai, aur `PendingReviewPost` hasil karne ka sirf ek tareeqa `DraftPost` par `request_review` method call karna hai, is liye ab humne blog post workflow ko type system mein encode kar diya hai.

Lekin humein `main` mein bhi kuch choti changes karni hongi. `request_review` aur `approve` methods un par call kiye gaye struct ko modify karne ke bajaye new instances return karte hain, is liye humein returned instances ko save karne ke liye mazeed `let post =` shadowing assignments add karni hongi. Hum draft aur pending review posts ke contents ke bare mein assertions ko empty strings bhi nahi rakh sakte, aur na hi humein un ki zaroorat hai: Ab hum aisa code compile nahi kar sakte jo in states mein posts ke content ko use karne ki koshish kare. `main` mein updated code Listing 18-21 mein dikhaya gaya hai.

<Listing number="18-21" file-name="src/main.rs" caption="Modifications to `main` to use the new implementation of the blog post workflow">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-21/src/main.rs}}
```

</Listing>

`post` ko reassign karne ke liye jo changes humein `main` mein karni pari, un ka matlab hai ke yeh implementation ab object-oriented state pattern ko bilkul follow nahi karti: States ke darmiyan transformations ab entirely `Post` implementation ke andar encapsulated nahi hain. Lekin humein is ka faida yeh mila hai ke type system aur compile time par hone wali type checking ki wajah se invalid states ab impossible hain! Is se yeh ensure hota hai ke kuch bugs, jaise unpublished post ke content ka display hona, production tak pohanchne se pehle hi discover ho jayenge.

Is section ke shuru mein `blog` crate ke liye jo tasks suggest kiye gaye thay, unhein Listing 18-21 ke baad wali `blog` crate par try karein aur dekhein ke aap is version ke code ke design ke bare mein kya sochte hain. Note karein ke kuch tasks is design mein pehle hi complete ho sakte hain.

Humne dekha ke Rust object-oriented design patterns implement karne ke qabil hone ke bawajood, doosre patterns bhi Rust mein available hain, jaise state ko type system mein encode karna. In patterns ke different trade-offs hain. Agarche aap object-oriented patterns se bohat familiar ho sakte hain, lekin Rust ke features ka faida uthane ke liye problem ko dobara sochna aise benefits provide kar sakta hai, jaise kuch bugs ko compile time par hi prevent karna. Rust mein object-oriented patterns hamesha best solution nahi honge kyun ke kuch features, jaise ownership, Rust mein hain jo object-oriented languages mein nahi hain.


## Summary

Is chapter ko parhne ke baad, chahe aap Rust ko object-oriented language samjhein ya na samjhein, ab aap jaante hain ke Rust mein kuch object-oriented features hasil karne ke liye trait objects ko use kiya ja sakta hai. Dynamic dispatch aapke code ko thori runtime performance ke badle kuch flexibility de sakta hai. Aap is flexibility ko object-oriented patterns implement karne ke liye use kar sakte hain jo aapke code ki maintainability mein madad kar sakte hain. Rust mein ownership jaise doosre features bhi hain jo object-oriented languages mein nahi hain. Rust ki strengths ka faida uthane ke liye object-oriented pattern hamesha best tareeqa nahi hoga, lekin yeh ek available option hai.

Ab hum patterns dekhenge, jo Rust ke ek aur feature hain jo bohat zyada flexibility enable karte hain. Humne poori book mein inhein briefly dekha hai lekin abhi tak in ki full capability nahi dekhi. Aaiye chalte hain!

[more-info-than-rustc]: ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
[macros]: ch20-05-macros.html#macros

