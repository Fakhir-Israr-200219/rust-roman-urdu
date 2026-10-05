## Pattern Syntax

Is section mein, hum tamam woh syntax jama karte hain jo patterns mein valid hain aur discuss karte hain ke aap har ek ko kyun aur kab use karna chahenge.

### Matching Literals

Jaisa ke aapne Chapter 6 mein dekha, aap patterns ko directly literals ke against match kar sakte hain. Neeche diya gaya code kuch examples deta hai:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-01-literals/src/main.rs:here}}
```

Yeh code `one` print karta hai kyun ke `x` mein value `1` hai. Yeh syntax us waqt useful hai jab aap chahte hain ke aapka code kisi particular concrete value ko receive karne par koi action le.

### Matching Named Variables

Named variables irrefutable patterns hoti hain jo kisi bhi value se match karti hain, aur humne is book mein inhein bohat baar use kiya hai. Lekin jab aap `match`, `if let`, ya `while let` expressions mein named variables use karte hain to ek complication hoti hai. Kyun ke in mein se har tarah ki expression ek naya scope start karti hai, is liye in expressions ke andar pattern ke part ke taur par declare kiye gaye variables un variables ko shadow karenge jinka same name constructs ke bahar hai, jaisa ke tamam variables ke saath hota hai. Listing 19-11 mein hum `x` naam ka ek variable declare karte hain jis ki value `Some(5)` hai aur `y` naam ka ek variable jis ki value `10` hai. Phir hum `x` ki value par ek `match` expression create karte hain. Match arms mein patterns aur end mein `println!` ko dekhein, aur is code ko run karne ya aage parhne se pehle yeh figure out karne ki koshish karein ke code kya print karega.

<Listing number="19-11" file-name="src/main.rs" caption="A `match` expression with an arm that introduces a new variable which shadows an existing variable `y`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-11/src/main.rs:here}}
```

</Listing>

Aaiye step by step dekhte hain ke `match` expression run hone par kya hota hai. Pehle match arm mein pattern `x` ki defined value se match nahi karta, is liye code aage continue karta hai.

Doosre match arm mein pattern `y` naam ka ek naya variable introduce karta hai jo `Some` value ke andar kisi bhi value se match karega. Kyun ke hum `match` expression ke andar ek naye scope mein hain, yeh ek naya `y` variable hai, woh `y` nahi jo humne shuru mein `10` ki value ke saath declare kiya tha. Yeh nayi `y` binding `Some` ke andar kisi bhi value se match karegi, jo ke `x` mein hamare paas hai. Is liye yeh naya `y`, `x` mein `Some` ki inner value ke saath bind hota hai. Woh value `5` hai, is liye us arm ki expression execute hoti hai aur `Matched, y = 5` print karti hai.

Agar `x` ki value `Some(5)` ke bajaye `None` hoti, to pehle do arms mein patterns match nahi karte, is liye value underscore ke saath match ho jati. Humne underscore arm ke pattern mein `x` variable introduce nahi kiya, is liye expression mein `x` ab bhi outer `x` hai jo shadow nahi hua. Is hypothetical case mein, `match` `Default case,
x = None` print karta.

Jab `match` expression complete hoti hai, us ka scope khatam ho jata hai, aur inner `y` ka scope bhi khatam ho jata hai. Aakhri `println!` `at the end: x = Some(5), y = 10` produce karta hai.

Ek aisi `match` expression create karne ke liye jo outer `x` aur `y` ki values ko compare kare, bajaye is ke ke ek naya variable introduce kiya jaye jo existing `y` ko shadow kare, humein is ke bajaye ek match guard conditional use karna hoga. Hum baad mein [“Adding Conditionals with Match
Guards”](#adding-conditionals-with-match-guards)<!-- ignore --> section mein match guards ke baare mein baat karenge.

<!-- Old headings. Do not remove or links may break. -->

<a id="multiple-patterns"></a>

### Matching Multiple Patterns

`match` expressions mein, aap `|` syntax ko use karke multiple patterns ko match kar sakte hain, jo pattern *or* operator hai. Misal ke taur par, neeche diye gaye code mein hum `x` ki value ko match arms ke against match karte hain, jin mein se pehle mein ek *or* option hai, yani agar `x` ki value us arm mein di gayi kisi bhi value se match karti hai, to us arm ka code run hoga:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-02-multiple-patterns/src/main.rs:here}}
```

Yeh code `one or two` print karta hai.

### Matching Ranges of Values with `..=`

`..=` syntax humein values ki ek inclusive range ke against match karne ki permission deti hai. Neeche diye gaye code mein, jab koi pattern di gayi range ke andar maujood kisi bhi value se match karta hai, to woh arm execute hoga:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-03-ranges/src/main.rs:here}}
```

Agar `x` `1`, `2`, `3`, `4`, ya `5` hai, to pehla arm match karega. Yeh syntax multiple match values ke liye `|` operator use karne ke muqable mein zyada convenient hai; agar hum `|` use karte, to humein `1 | 2 |
3 | 4 | 5` specify karna padta. Range specify karna bohat chhota hota hai, khaas taur par agar hum, maan lijiye, 1 aur 1,000 ke darmiyan koi bhi number match karna chahte hon!

Compiler compile time par check karta hai ke range empty to nahi hai, aur kyun ke sirf `char` aur numeric values hi woh types hain jin ke liye Rust bata sakta hai ke range empty hai ya nahi, is liye ranges sirf numeric ya `char` values ke saath allowed hain.

Yahan `char` values ki ranges use karne ki ek example hai:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-04-ranges-of-char/src/main.rs:here}}
```

Rust bata sakta hai ke `'c'` pehle pattern ki range ke andar hai aur `early
ASCII letter` print karta hai.

### Destructuring to Break Apart Values

Hum patterns ko structs, enums, aur tuples ko destructure karne ke liye bhi use kar sakte hain taake in values ke mukhtalif parts ko use kiya ja sake. Aaiye har value ko ek ek karke dekhte hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-structs"></a>

#### Structs

Listing 19-12 mein do fields, `x` aur `y`, wala ek `Point` struct dikhaya gaya hai, jise hum `let` statement ke saath ek pattern use karke break apart kar sakte hain.

<Listing number="19-12" file-name="src/main.rs" caption="Destructuring a struct’s fields into separate variables">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-12/src/main.rs}}
```

</Listing>

Yeh code variables `a` aur `b` create karta hai jo `p` struct ke `x` aur `y` fields ki values se match karte hain. Yeh example dikhata hai ke pattern mein variables ke names ka struct ke field names se match karna zaroori nahi hai. Lekin, variables ke names ko field names ke saath match karna common hai taake yeh yaad rakhna aasaan ho ke kaun se variables kis fields se aaye hain. Is common usage ki wajah se, aur kyun ke `let Point { x: x, y: y } = p;` likhne mein bohat duplication hoti hai, Rust mein un patterns ke liye ek shorthand hai jo struct fields se match karti hain: Aapko sirf struct field ka name list karna hota hai, aur pattern se create hone wale variables ke names bhi wohi honge. Listing 19-13 Listing 19-12 ke code ki tarah hi kaam karti hai, lekin `let` pattern mein create hone wale variables `a` aur `b` ke bajaye `x` aur `y` hain.

<Listing number="19-13" file-name="src/main.rs" caption="Destructuring struct fields using struct field shorthand">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-13/src/main.rs}}
```

</Listing>

Yeh code variables `x` aur `y` create karta hai jo `p` variable ke `x` aur `y` fields se match karte hain. Natija yeh hai ke variables `x` aur `y` mein `p` struct se aane wali values hoti hain.

Hum struct pattern ke part ke taur par variables create karne ke bajaye literal values ke saath bhi destructure kar sakte hain. Is se humein kuch fields ko particular values ke liye test karne aur doosri fields ko destructure karne ke liye variables create karne ki permission milti hai.

Listing 19-14 mein hamare paas ek `match` expression hai jo `Point` values ko teen cases mein separate karti hai: woh points jo directly `x` axis par hotay hain (jo us waqt true hota hai jab `y = 0`), `y` axis par (`x = 0`), ya dono axes mein se kisi par bhi nahi hotay.

<Listing number="19-14" file-name="src/main.rs" caption="Destructuring and matching literal values in one pattern">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-14/src/main.rs:here}}
```

</Listing>

Pehla arm har us point se match karega jo `x` axis par hai, is baat ko specify karke ke `y` field tab match karegi jab us ki value literal `0` se match kare. Pattern phir bhi ek `x` variable create karta hai jise hum is arm ke code mein use kar sakte hain.

Isi tarah, doosra arm `y` axis par maujood har point se match karta hai, is baat ko specify karke ke `x` field tab match karegi jab us ki value `0` ho, aur `y` field ki value ke liye ek variable `y` create karta hai. Teesra arm koi literals specify nahi karta, is liye yeh kisi bhi doosre `Point` se match karta hai aur `x` aur `y` dono fields ke liye variables create karta hai.

Is example mein, value `p` doosre arm se match karti hai kyun ke `x` mein `0` hai, is liye yeh code `On the y axis at 7` print karega.

Yaad rakhein ke `match` expression pehla matching pattern milte hi arms ko check karna rok deti hai, is liye halaanke `Point { x: 0, y: 0 }` `x` axis aur `y` axis dono par hai, yeh code sirf `On the x axis at 0` print karega.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-enums"></a>

#### Enums

Humne is book mein enums ko destructure kiya hai (misal ke taur par, Chapter 6 mein Listing 6-5), lekin humne abhi tak explicitly discuss nahi kiya ke enum ko destructure karne wala pattern us tareeqe se correspond karta hai jis tareeqe se enum ke andar stored data define kiya gaya hai. Misal ke taur par, Listing 19-15 mein hum Listing 6-2 wali `Message` enum ko use karte hain aur aisi `match` likhte hain jisme patterns har inner value ko destructure karenge.

<Listing number="19-15" file-name="src/main.rs" caption="Destructuring enum variants that hold different kinds of values">

```rust id="6v4mqa"
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-15/src/main.rs}}
```

</Listing>

Yeh code `Change color to red 0, green 160, and blue 255` print karega. `msg` ki value change karke dekhein taake doosre arms ka code run hota hua dekh sakein.

Aise enum variants jin mein koi data nahi hota, jaise `Message::Quit`, unki value ko hum mazeed destructure nahi kar sakte. Hum sirf literal `Message::Quit` value ke against match kar sakte hain, aur us pattern mein koi variables nahi hote.

Struct-like enum variants ke liye, jaise `Message::Move`, hum aisa pattern use kar sakte hain jo structs ko match karne ke liye specify kiye gaye pattern ke similar hota hai. Variant name ke baad hum curly brackets rakhte hain aur phir fields ko variables ke saath list karte hain taake hum un pieces ko break apart kar sakein jinhein is arm ke code mein use karna hai. Yahan hum shorthand form use karte hain, jaisa ke Listing 19-13 mein kiya tha.

Tuple-like enum variants ke liye, jaise `Message::Write` jo ek element wala tuple hold karta hai aur `Message::ChangeColor` jo teen elements wala tuple hold karta hai, pattern us pattern ke similar hota hai jo hum tuples ko match karne ke liye specify karte hain. Pattern mein variables ki tadaad us variant mein elements ki tadaad ke barabar honi chahiye jise hum match kar rahe hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-nested-structs-and-enums"></a>

#### Nested Structs and Enums

Ab tak, hamare tamam examples structs ya enums ko sirf ek level deep match kar rahe thay, lekin matching nested items par bhi kaam kar sakti hai! Misal ke taur par, hum Listing 19-15 ke code ko refactor karke `ChangeColor` message mein RGB aur HSV colors ko support kar sakte hain, jaisa ke Listing 19-16 mein dikhaya gaya hai.

<Listing number="19-16" caption="Matching on nested enums">

```rust id="o9t2wp"
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-16/src/main.rs}}
```

</Listing>

`match` expression ke pehle arm ka pattern ek `Message::ChangeColor` enum variant se match karta hai jisme `Color::Rgb` variant hota hai; phir, pattern teen inner `i32` values ke saath bind hota hai. Doosre arm ka pattern bhi ek `Message::ChangeColor` enum variant se match karta hai, lekin inner enum `Color::Hsv` se match karta hai. Hum in complex conditions ko ek hi `match` expression mein specify kar sakte hain, halaanke is mein do enums shamil hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-structs-and-tuples"></a>

#### Structs and Tuples

Hum destructuring patterns ko aur bhi complex tareeqon se mix, match aur nest kar sakte hain. Neeche diya gaya example ek complicated destructure dikhata hai jahan hum ek tuple ke andar structs aur tuples ko nest karte hain aur tamam primitive values ko destructure kar lete hain:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-05-destructuring-structs-and-tuples/src/main.rs:here}}
```

Yeh code humein complex types ko un ke component parts mein break karne deta hai taa-ke hum jin values mein interested hain unhein alag se use kar saken.

Patterns ke saath destructuring values ke pieces ko use karne ka ek convenient tareeqa hai, jaise struct ke har field ki value ko ek doosre se alag use karna.

### Ignoring Values in a Pattern

Aap ne dekha hai ke kabhi kabhi pattern mein values ko ignore karna useful hota hai, jaise `match` ke last arm mein, taa-ke ek catch-all mil sake jo asal mein kuch nahi karta lekin baqi tamam possible values ko account karta hai. Pattern mein poori values ya values ke kuch parts ko ignore karne ke kuch tareeqe hain: `_` pattern ko use karna (jise aap dekh chuke hain), kisi doosre pattern ke andar `_` pattern ko use karna, aisa name use karna jo underscore se start hota ho, ya `..` ko use karke value ke baqi parts ko ignore karna. Aaiye explore karte hain ke in mein se har pattern ko kaise aur kyun use karna hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-an-entire-value-with-_"></a>

#### An Entire Value with `_`

Hum underscore ko ek wildcard pattern ke taur par use kar chuke hain jo kisi bhi value se match karega lekin us value ke saath bind nahi hoga. Yeh khaas taur par `match` expression ke last arm ke taur par useful hai, lekin hum ise kisi bhi pattern mein use kar sakte hain, jisme function parameters bhi shamil hain, jaisa ke Listing 19-17 mein dikhaya gaya hai.

<Listing number="19-17" file-name="src/main.rs" caption="Using `_` in a function signature">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-17/src/main.rs}}
```

</Listing>

Yeh code pehle argument ke taur par pass ki gayi value `3` ko poori tarah ignore karega, aur `This code only uses the y parameter: 4` print karega.

Aksar jab aapko kisi particular function parameter ki zaroorat nahi rehti, to aap signature ko change kar dete hain taa-ke us mein unused parameter shamil na ho. Function parameter ko ignore karna khaas taur par un cases mein useful ho sakta hai jab, misal ke taur par, aap kisi trait ko implement kar rahe hon aur aapko ek certain type signature ki zaroorat ho lekin aapki implementation ke function body ko parameters mein se kisi ek ki zaroorat na ho. Is tarah aap unused function parameters ke compiler warning se bach jate hain, jo aapko name use karne par milti.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-parts-of-a-value-with-a-nested-_"></a>

#### Parts of a Value with a Nested `_`

Hum kisi doosre pattern ke andar bhi `_` use kar sakte hain taa-ke value ke sirf ek part ko ignore kiya ja sake, misal ke taur par, jab hum kisi value ke sirf ek part ko test karna chahte hon lekin baqi parts ka corresponding code mein koi use na ho. Listing 19-18 mein ek setting ki value ko manage karne wala code dikhaya gaya hai. Business requirements yeh hain ke user ko kisi setting ki existing customization ko overwrite karne ki ijazat nahi honi chahiye, lekin user setting ko unset kar sakta hai aur agar woh currently unset hai to usay ek value de sakta hai.

<Listing number="19-18" caption="Using an underscore within patterns that match `Some` variants when we don’t need to use the value inside the `Some`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-18/src/main.rs:here}}
```

</Listing>

Yeh code `Can't overwrite an existing customized value` aur phir `setting is Some(5)` print karega. Pehle match arm mein, humein dono `Some` variants ke andar maujood values se match karne ya unhein use karne ki zaroorat nahi hai, lekin humein us case ko test karna zaroori hai jab `setting_value` aur `new_setting_value` `Some` variant hon. Is case mein, hum `setting_value` ko change na karne ki wajah print karte hain, aur isay change nahi kiya jata.

Baqi tamam cases mein (agar `setting_value` ya `new_setting_value` mein se koi bhi `None` ho), jinhein second arm mein `_` pattern express karta hai, hum `new_setting_value` ko `setting_value` banne ki ijazat dena chahte hain.

Hum ek hi pattern ke andar multiple places par underscores bhi use kar sakte hain taa-ke particular values ko ignore kiya ja sake. Listing 19-19 mein paanch items wale tuple ki second aur fourth values ko ignore karne ki example dikhayi gayi hai.

<Listing number="19-19" caption="Ignoring multiple parts of a tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-19/src/main.rs:here}}
```

</Listing>

Yeh code `Some numbers: 2, 8, 32` print karega, aur values `4` aur `16` ko ignore kar diya jayega.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-an-unused-variable-by-starting-its-name-with-_"></a>

#### An Unused Variable by Starting Its Name with `_`

Agar aap ek variable create karte hain lekin usay kahin bhi use nahi karte, to Rust aam tor par warning dega kyun ke unused variable bug ho sakta hai. Lekin kabhi kabhi aisa variable create karna useful hota hai jise aap abhi use nahi karenge, jaise jab aap prototyping kar rahe hon ya abhi project shuru kar rahe hon. Is situation mein, aap variable ka name underscore se start karke Rust ko bata sakte hain ke unused variable ke bare mein warning na de. Listing 19-20 mein hum do unused variables create karte hain, lekin jab hum is code ko compile karenge, to humein sirf ek ke bare mein warning milni chahiye.

<Listing number="19-20" file-name="src/main.rs" caption="Starting a variable name with an underscore to avoid getting unused variable warnings">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-20/src/main.rs}}
```

</Listing>

Yahan humein variable `y` ko use na karne ke bare mein warning milti hai, lekin `_x` ko use na karne ke bare mein warning nahi milti.

Note karein ke sirf `_` use karne aur underscore se start hone wala name use karne mein ek subtle difference hai. Syntax `_x` ab bhi value ko variable ke saath bind karta hai, jabke `_` bilkul bind nahi karta. Ek aisa case dikhane ke liye jahan yeh distinction matter karti hai, Listing 19-21 humein ek error dega.

<Listing number="19-21" caption="An unused variable starting with an underscore still binds the value, which might take ownership of the value.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-21/src/main.rs:here}}
```

</Listing>

Humein ek error milega kyun ke `s` value ab bhi `_s` mein move ho jayegi, jo humein dobara `s` ko use karne se rokta hai. Lekin sirf underscore ko use karna value ke saath kabhi bind nahi karta. Listing 19-22 bina kisi error ke compile ho jayegi kyun ke `s`, `_` mein move nahi hota.

<Listing number="19-22" caption="Using an underscore does not bind the value.">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-22/src/main.rs:here}}
```

</Listing>

Yeh code bilkul theek kaam karta hai kyun ke hum `s` ko kisi cheez ke saath bind hi nahi karte; yeh move nahi hota.

<a id="ignoring-remaining-parts-of-a-value-with-"></a>

#### Remaining Parts of a Value with `..`

Aisi values jin mein bohat se parts hon, hum `..` syntax ko use karke specific parts ko use aur baqi ko ignore kar sakte hain, jis se har ignored value ke liye underscores list karne ki zaroorat nahi rehti. `..` pattern value ke un tamam parts ko ignore karta hai jinhein humne pattern ke baqi hisson mein explicitly match nahi kiya. Listing 19-23 mein hamare paas ek `Point` struct hai jo three-dimensional space mein ek coordinate hold karta hai. `match` expression mein hum sirf `x` coordinate par operate karna chahte hain aur `y` aur `z` fields ki values ko ignore karna chahte hain.

<Listing number="19-23" caption="Ignoring all fields of a `Point` except for `x` by using `..`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-23/src/main.rs:here}}
```

</Listing>

Hum `x` value ko list karte hain aur phir sirf `..` pattern include kar dete hain. Yeh `y: _` aur `z: _` ko list karne ki zaroorat se zyada quick hai, khaas taur par jab hum aise structs ke saath kaam kar rahe hon jin mein bohat se fields hon aur sirf ek ya do fields relevant hon.

Syntax `..` utni values ke liye expand hoga jitni isay zaroorat hogi. Listing 19-24 dikhati hai ke tuple ke saath `..` ko kaise use karna hai.

<Listing number="19-24" file-name="src/main.rs" caption="Matching only the first and last values in a tuple and ignoring all other values">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-24/src/main.rs}}
```

</Listing>

Is code mein, pehli aur aakhri values ko `first` aur `last` ke saath match kiya jata hai. `..` darmiyan ki tamam cheezon se match karega aur unhein ignore kar dega.

Lekin, `..` ka use unambiguous hona zaroori hai. Agar yeh clear na ho ke kaunsi values matching ke liye intended hain aur kaunsi ignore ki jani chahiye, to Rust humein error dega. Listing 19-25 mein `..` ko ambiguous tareeqe se use karne ki example dikhayi gayi hai, is liye yeh compile nahi hogi.

<Listing number="19-25" file-name="src/main.rs" caption="An attempt to use `..` in an ambiguous way">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-25/src/main.rs}}
```

</Listing>

Jab hum is example ko compile karte hain, to humein yeh error milta hai:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-25/output.txt}}
```

Rust ke liye yeh determine karna impossible hai ke `second` ke saath kisi value ko match karne se pehle tuple mein kitni values ko ignore karna hai aur us ke baad mazeed kitni values ko ignore karna hai. Yeh code is ka matlab ho sakta hai ke hum `2` ko ignore karna chahte hain, `second` ko `4` ke saath bind karna chahte hain, aur phir `8`, `16`, aur `32` ko ignore karna chahte hain; ya phir hum `2` aur `4` ko ignore karna, `second` ko `8` ke saath bind karna, aur phir `16` aur `32` ko ignore karna chahte hain; aur isi tarah aur bhi possibilities ho sakti hain. Variable name `second` Rust ke liye kuch special meaning nahi rakhta, is liye humein compiler error milta hai kyun ke is tarah do places par `..` use karna ambiguous hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="extra-conditionals-with-match-guards"></a>

### Adding Conditionals with Match Guards

Ek *match guard* ek additional `if` condition hoti hai jo `match` arm mein pattern ke baad specify ki jati hai aur arm ke choose hone ke liye us ka bhi match hona zaroori hota hai. Match guards un complex ideas ko express karne ke liye useful hain jinhein sirf pattern ke zariye express nahi kiya ja sakta. Lekin note karein ke yeh sirf `match` expressions mein available hain, `if let` ya `while let` expressions mein nahi.

Condition pattern mein create kiye gaye variables ko use kar sakti hai. Listing 19-26 mein ek `match` dikhaya gaya hai jahan pehle arm mein `Some(x)` pattern hai aur saath hi `if x % 2 == 0` ka match guard hai (jo `true` hoga agar number even ho).

<Listing number="19-26" caption="Adding a match guard to a pattern">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-26/src/main.rs:here}}
```

</Listing>

Yeh example `The number 4 is even` print karega. Jab `num` ko pehle arm ke pattern ke saath compare kiya jata hai, to yeh match karta hai kyun ke `Some(4)`, `Some(x)` se match karta hai. Phir match guard check karta hai ke `x` ko 2 se divide karne par remainder 0 ke barabar hai ya nahi, aur kyun ke aisa hai, pehla arm select ho jata hai.

Agar `num` is ke bajaye `Some(5)` hota, to pehle arm ka match guard `false` hota kyun ke 5 ko 2 se divide karne par remainder 1 hota hai, jo 0 ke barabar nahi hai. Rust phir doosre arm par chala jata, jo match karta kyun ke doosre arm mein match guard nahi hai aur is liye woh kisi bhi `Some` variant se match karta hai.

`if x % 2 == 0` condition ko pattern ke andar express karne ka koi tareeqa nahi hai, is liye match guard humein is logic ko express karne ki ability deta hai. Is additional expressiveness ka downside yeh hai ke jab match guard expressions shamil hon to compiler exhaustiveness check karne ki koshish nahi karta.

Listing 19-11 ko discuss karte waqt, humne mention kiya tha ke apne pattern-shadowing problem ko solve karne ke liye hum match guards use kar sakte hain. Yaad karein ke humne `match` expression ke andar pattern mein ek naya variable create kiya tha, bahar maujood variable ko use karne ke bajaye. Is naye variable ki wajah se hum outer variable ki value ke against test nahi kar sakte thay. Listing 19-27 dikhati hai ke is problem ko fix karne ke liye hum match guard ko kaise use kar sakte hain.

<Listing number="19-27" file-name="src/main.rs" caption="Using a match guard to test for equality with an outer variable">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-27/src/main.rs}}
```

</Listing>

Ab yeh code `Default case, x = Some(5)` print karega. Doosre match arm ka pattern koi naya variable `y` introduce nahi karta jo outer `y` ko shadow kare, jis ka matlab hai ke hum match guard mein outer `y` ko use kar sakte hain. Pattern ko `Some(y)` specify karne ke bajaye, jo outer `y` ko shadow karta, hum `Some(n)` specify karte hain. Is se ek naya variable `n` create hota hai jo kisi cheez ko shadow nahi karta kyun ke `match` ke bahar koi `n` variable nahi hai.

Match guard `if n == y` pattern nahi hai aur is liye yeh koi naye variables introduce nahi karta. Yeh `y` outer `y` hi hai, na ke koi naya `y` jo usay shadow kar raha ho, aur hum `n` ko `y` ke saath compare karke aisi value search kar sakte hain jo outer `y` ke same value rakhti ho.

Aap match guard mein multiple patterns specify karne ke liye *or* operator `|` bhi use kar sakte hain; match guard ki condition tamam patterns par apply hogi. Listing 19-28 `|` use karne wale pattern ko match guard ke saath combine karte waqt precedence dikhati hai. Is example ka important part yeh hai ke `if y` match guard `4`, `5`, *aur* `6` par apply hota hai, halaanke dekhne mein aisa lag sakta hai ke `if y` sirf `6` par apply hota hai.

<Listing number="19-28" caption="Combining multiple patterns with a match guard">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-28/src/main.rs:here}}
```

</Listing>

Match condition kehti hai ke arm sirf tab match karta hai jab `x` ki value `4`, `5`, ya `6` ke barabar ho *aur* `y` `true` ho. Jab yeh code run hota hai, to pehle arm ka pattern match karta hai kyun ke `x` `4` hai, lekin match guard `if y` `false` hai, is liye pehla arm choose nahi hota. Code doosre arm par chala jata hai, jo match karta hai, aur yeh program `no` print karta hai. Is ki wajah yeh hai ke `if` condition poore pattern `4 | 5 | 6` par apply hoti hai, sirf aakhri value `6` par nahi. Doosre lafzon mein, match guard ki pattern ke relation mein precedence is tarah behave karti hai:

```text
(4 | 5 | 6) if y => ...
```

is ke bajaye:

```text
4 | 5 | (6 if y) => ...
```

Code run karne ke baad precedence ka behavior clear ho jata hai: Agar match guard sirf `|` operator se specify ki gayi values ki list ki aakhri value par apply hota, to arm match kar jata, aur program `yes` print karta.

<!-- Old headings. Do not remove or links may break. -->

<a id="-bindings"></a>

### Using `@` Bindings

*At* operator `@` humein ek aisa variable create karne deta hai jo ek value ko hold karta hai, saath hi hum us value ko pattern match ke liye test bhi kar rahe hote hain. Listing 19-29 mein, hum test karna chahte hain ke `Message::Hello` ka `id` field range `3..=7` ke andar hai. Hum value ko `id` naam ke variable ke saath bind bhi karna chahte hain taa-ke hum isay arm se associated code mein use kar saken.

<Listing number="19-29" caption="Using `@` to bind to a value in a pattern while also testing it">

```rust id="v4l0nz"
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-29/src/main.rs:here}}
```

</Listing>

Yeh example `Found an id in range: 5` print karega. Range `3..=7` se pehle `id @` specify karke, hum range se match hone wali kisi bhi value ko `id` naam ke variable mein capture kar rahe hain aur saath hi test bhi kar rahe hain ke value range pattern se match karti hai.

Doosre arm mein, jahan pattern mein sirf ek range specified hai, arm se associated code ke paas actual `id` field ki value ko contain karne wala koi variable nahi hai. `id` field ki value 10, 11, ya 12 ho sakti thi, lekin us pattern ke saath wala code nahi jaanta ke in mein se kaunsi hai. Pattern code `id` field se value ko use nahi kar sakta kyun ke humne `id` value ko kisi variable mein save nahi kiya.

Aakhri arm mein, jahan humne range ke baghair ek variable specify kiya hai, wahan arm ke code mein use karne ke liye value `id` naam ke variable mein available hai. Is ki wajah yeh hai ke humne struct field shorthand syntax use ki hai. Lekin humne is arm mein `id` field ki value par koi test apply nahi kiya, jaisa ke pehle do arms mein kiya tha: Koi bhi value is pattern se match karegi.

`@` use karne se humein ek hi pattern ke andar kisi value ko test karne aur usay variable mein save karne ki ability milti hai.

## Summary

Rust ke patterns different kinds of data ke darmiyan farq karne mein bohat useful hain. Jab inhein `match` expressions ke saath use kiya jata hai, Rust ensure karta hai ke aapke patterns har possible value ko cover karte hon, warna aapka program compile nahi hoga. `let` statements aur function parameters mein patterns in constructs ko zyada useful banate hain, kyun ke yeh values ko chhote parts mein destructure karne aur un parts ko variables ke saath assign karne ki ability dete hain. Hum apni needs ke mutabiq simple ya complex patterns create kar sakte hain.

Ab, book ke penultimate chapter ke liye, hum Rust ke mukhtalif features ke kuch advanced aspects dekhenge.

