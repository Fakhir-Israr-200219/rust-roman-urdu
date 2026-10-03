## Lifetimes Ke Saath References Ko Validate Karna

Lifetimes ek aur qisam ke generic hain jinhein hum pehle se use kar rahe hain. Kisi type ke paas hamara required behavior hai ya nahi, ye ensure karne ke bajaye, lifetimes ye ensure karte hain ke references utni der tak valid rahen jitni der humein unki zaroorat ho.

Ek detail jis par hum ne Chapter 4 ke [“References and Borrowing”][references-and-borrowing]<!-- ignore --> section mein baat nahi ki thi, woh ye hai ke Rust mein har reference ki ek lifetime hoti hai, jo woh scope hota hai jis ke dauran woh reference valid hota hai. Zyada tar waqt lifetimes implicit hoti hain aur infer ki jati hain, bilkul isi tarah jaise zyada tar waqt types infer kiye jate hain. Humein types ko sirf us waqt annotate karna padta hai jab multiple types possible hon. Isi tarah, humein lifetimes ko us waqt annotate karna padta hai jab references ki lifetimes kuch different tareeqon se ek doosre se related ho sakti hon. Rust hum se taqaza karta hai ke generic lifetime parameters ko use karke in relationships ko annotate karein taa-ke ye ensure ho sake ke runtime par use hone wale actual references definitely valid honge.

Lifetimes ko annotate karna aisa concept hai jo zyada tar doosri programming languages mein hota hi nahi, is liye shuru mein ye unfamiliar mehsoos hoga. Agarche hum is chapter mein lifetimes ko poori tarah cover nahi karenge, hum lifetime syntax ke un common tareeqon par baat karenge jin ka aapko saamna ho sakta hai, taa-ke aap is concept se comfortable ho saken.

<!-- Old headings. Do not remove or links may break. -->

<a id="preventing-dangling-references-with-lifetimes"></a>

### Dangling References

Lifetimes ka main maqsad dangling references ko prevent karna hai, jo agar exist karne ki ijazat di jaye, to program ko us data ke bajaye kisi aur data ko reference karne ka sabab ban sakte hain jisay reference karna intended tha. Listing 10-16 mein diye gaye program par ghour karein, jis mein ek outer scope aur ek inner scope hai.

<Listing number="10-16" caption="An attempt to use a reference whose value has gone out of scope">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/src/main.rs}}
```

</Listing>

> Note: Listings 10-16, 10-17, aur 10-23 ke examples mein variables ko declare kiya gaya hai lekin unhein initial value nahi di gayi, is liye variable ka naam outer scope mein mojood hota hai. Pehli nazar mein, ye Rust mein null values na hone ke saath conflict karta hua lag sakta hai. Lekin agar hum kisi variable ko value dene se pehle use karne ki koshish karein, to humein compile-time error milega, jo dikhata hai ke waqai Rust null values ki ijazat nahi deta.

Outer scope `r` naam ka ek variable declare karta hai jis ki koi initial value nahi hai, aur inner scope `x` naam ka ek variable declare karta hai jis ki initial value `5` hai. Inner scope ke andar, hum `r` ki value ko `x` ke reference ke taur par set karne ki koshish karte hain. Phir inner scope khatam ho jata hai, aur hum `r` ki value ko print karne ki koshish karte hain. Ye code compile nahi hoga, kyun ke jis value ko `r` reference kar raha hai woh `r` ko use karne ki koshish se pehle hi scope se bahar chali gayi hai. Yahan error message hai:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/output.txt}}
```

Error message kehta hai ke variable `x` “does not live long enough.” Is ki wajah ye hai ke line 7 par inner scope khatam hone par `x` scope se bahar ho jayega. Lekin `r` ab bhi outer scope ke liye valid hai; kyun ke iska scope zyada bara hai, hum kehte hain ke ye “lives longer.” Agar Rust is code ko kaam karne ki ijazat de deta, to `r` us memory ko reference kar raha hota jo `x` ke scope se bahar hone par deallocate ho chuki hoti, aur `r` ke saath hum jo bhi karne ki koshish karte woh correctly kaam nahi karta. To Rust kaise determine karta hai ke ye code invalid hai? Ye borrow checker ko use karta hai.

### Borrow Checker

Rust compiler ke paas ek *borrow checker* hota hai jo scopes ko compare karke ye determine karta hai ke tamam borrows valid hain ya nahi. Listing 10-17 mein Listing 10-16 wala hi code diya gaya hai, lekin variables ki lifetimes ko show karne wali annotations ke saath.

<Listing number="10-17" caption="Annotations of the lifetimes of `r` and `x`, named `'a` and `'b`, respectively">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-17/src/main.rs}}
```

</Listing>

Yahan, hum ne `r` ki lifetime ko `'a` aur `x` ki lifetime ko `'b` ke saath annotate kiya hai. Jaisa ke aap dekh sakte hain, inner `'b` block outer `'a` lifetime block se kaafi chhota hai. Compile time par, Rust dono lifetimes ke size ko compare karta hai aur dekhta hai ke `r` ki lifetime `'a` hai, lekin woh aisi memory ko refer karta hai jis ki lifetime `'b` hai. Program reject kar diya jata hai kyun ke `'b`, `'a` se chhota hai: Reference ka subject utni der tak live nahi karta jitni der tak reference live karta hai.

Listing 10-18 code ko fix karti hai taa-ke is mein dangling reference na ho aur ye baghair kisi error ke compile ho jaye.

<Listing number="10-18" caption="A valid reference because the data has a longer lifetime than the reference">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-18/src/main.rs}}
```

</Listing>

Yahan, `x` ki lifetime `'b` hai, jo is case mein `'a` se bari hai. Is ka matlab hai ke `r`, `x` ko reference kar sakta hai kyun ke Rust jaanta hai ke `r` mein mojood reference hamesha us waqt tak valid rahega jab tak `x` valid hai.

Ab jab aap jaante hain ke references ki lifetimes kahan hoti hain aur Rust lifetimes ka analysis karke ye kaise ensure karta hai ke references hamesha valid rahenge, to aaiye function parameters aur return values mein generic lifetimes ko explore karte hain.

### Functions Mein Generic Lifetimes

Hum ek aisa function likhenge jo do string slices mein se zyada lambi string slice return kare. Ye function do string slices lega aur ek single string slice return karega. `longest` function implement karne ke baad, Listing 10-19 mein diya gaya code `The longest string is abcd` print karna chahiye.

<Listing number="10-19" file-name="src/main.rs" caption="A `main` function that calls the `longest` function to find the longer of two string slices">

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-19/src/main.rs}}
```

</Listing>

Note karein ke hum chahte hain ke function string slices le, jo references hain, strings nahi, kyun ke hum nahi chahte ke `longest` function apne parameters ki ownership le. Listing 10-19 mein use kiye gaye parameters hi woh parameters kyun hain jo hum chahte hain, is ke baare mein mazeed discussion ke liye Chapter 4 mein [“String Slices as Parameters”][string-slices-as-parameters]<!-- ignore --> dekhein.

Agar hum `longest` function ko Listing 10-20 mein dikhaye gaye tareeqe se implement karne ki koshish karein, to ye compile nahi hoga.

<Listing number="10-20" file-name="src/main.rs" caption="An implementation of the `longest` function that returns the longer of two string slices but does not yet compile">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/src/main.rs:here}}
```

</Listing>

Is ke bajaye, humein neeche diya gaya error milta hai jo lifetimes ke baare mein baat karta hai:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/output.txt}}
```

Help text ye reveal karta hai ke return type ko ek generic lifetime parameter ki zaroorat hai, kyun ke Rust ye nahi bata sakta ke return kiya jane wala reference `x` ko refer karta hai ya `y` ko. Asal mein, humein bhi nahi pata, kyun ke is function ki body mein `if` block `x` ka reference return karta hai aur `else` block `y` ka reference return karta hai!

Jab hum is function ko define kar rahe hote hain, to humein un concrete values ka pata nahi hota jo is function mein pass ki jayengi, is liye humein nahi pata ke `if` case execute hoga ya `else` case. Humein pass kiye jane wale references ki concrete lifetimes ka bhi pata nahi hota, is liye hum Listings 10-17 aur 10-18 ki tarah scopes ko dekh kar ye determine nahi kar sakte ke jo reference hum return karte hain woh hamesha valid rahega ya nahi. Borrow checker bhi ye determine nahi kar sakta, kyun ke use ye nahi pata ke `x` aur `y` ki lifetimes ka return value ki lifetime ke saath kya relation hai. Is error ko fix karne ke liye, hum generic lifetime parameters add karenge jo references ke darmiyan relationship ko define karte hain taa-ke borrow checker apna analysis perform kar sake.

### Lifetime Annotation Syntax

Lifetime annotations references ki lifetime kitni der tak hoti hai, isay change nahi kartin. Is ke bajaye, ye multiple references ki lifetimes ke darmiyan relationships ko describe karti hain, baghair lifetimes ko affect kiye. Jis tarah functions kisi bhi type ko accept kar sakte hain jab signature mein generic type parameter specify kiya gaya ho, usi tarah functions kisi bhi lifetime wale references ko accept kar sakte hain jab generic lifetime parameter specify kiya gaya ho.

Lifetime annotations ki syntax thori unusual hoti hai: Lifetime parameters ke names apostrophe (`'`) se start hone chahiye aur aam tor par generic types ki tarah lowercase aur bohat short hote hain. Zyada tar log pehli lifetime annotation ke liye `'a` naam use karte hain. Hum lifetime parameter annotations ko reference ke `&` ke baad place karte hain, aur annotation ko reference ke type se separate karne ke liye ek space use karte hain.

Yahan kuch examples hain—ek `i32` ka reference jis mein lifetime parameter nahi hai, ek `i32` ka reference jis mein `'a` naam ka lifetime parameter hai, aur ek mutable `i32` reference jis mein `'a` lifetime bhi hai:

```rust,ignore
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

Sirf ek lifetime annotation ka apne aap mein zyada matlab nahi hota, kyun ke annotations ka maqsad Rust ko ye batana hai ke multiple references ke generic lifetime parameters ek doosre se kis tarah related hain. Aaiye `longest` function ke context mein examine karte hain ke lifetime annotations ek doosre se kis tarah related hoti hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-function-signatures"></a>

### Function Signatures Mein

Function signatures mein lifetime annotations use karne ke liye, humein generic lifetime parameters ko angle brackets ke andar function name aur parameter list ke darmiyan declare karna hota hai, bilkul usi tarah jaise hum ne generic type parameters ke saath kiya tha.

Hum chahte hain ke signature ye constraint express kare: Returned reference us waqt tak valid rahega jab tak dono parameters valid hain. Ye parameters aur return value ki lifetimes ke darmiyan relationship hai. Hum lifetime ko `'a` naam denge aur phir isay har reference mein add karenge, jaisa ke Listing 10-21 mein dikhaya gaya hai.

<Listing number="10-21" file-name="src/main.rs" caption="The `longest` function definition specifying that all the references in the signature must have the same lifetime `'a`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-21/src/main.rs:here}}
```

</Listing>

Ye code compile hona chahiye aur jab hum isay Listing 10-19 ke `main` function ke saath use karein to desired result produce karna chahiye.

Function signature ab Rust ko batati hai ke kisi lifetime `'a` ke liye, function do parameters leta hai, aur dono string slices hain jo kam az kam lifetime `'a` jitni der tak live karte hain. Function signature Rust ko ye bhi batati hai ke function se return hone wali string slice kam az kam lifetime `'a` jitni der tak live karegi. Practical taur par, iska matlab hai ke `longest` function se return hone wale reference ki lifetime, function arguments ki taraf se refer ki jane wali values ki lifetimes mein se chhoti lifetime ke barabar hai. Ye woh relationships hain jinhein hum chahte hain ke Rust is code ka analysis karte waqt use kare.

Yaad rakhein, jab hum is function signature mein lifetime parameters specify karte hain, to hum pass ki jane wali ya return ki jane wali kisi bhi value ki lifetimes ko change nahi kar rahe. Is ke bajaye, hum specify kar rahe hain ke borrow checker un tamam values ko reject kare jo in constraints ko follow nahi kartin. Note karein ke `longest` function ko ye exactly jaanne ki zaroorat nahi hoti ke `x` aur `y` kitni der tak live karenge; sirf itna maloom hona zaroori hai ke koi aisa scope `'a` ki jagah substitute kiya ja sakta hai jo is signature ki conditions ko satisfy karta ho.

Functions mein lifetimes ko annotate karte waqt, annotations function signature mein jati hain, function body mein nahi. Lifetime annotations function ke contract ka hissa ban jati hain, bilkul signature mein types ki tarah. Function signatures mein lifetime contract hone se Rust compiler ke liye analysis zyada simple ho jata hai. Agar function ko annotate karne ke tareeqe mein ya usay call karne ke tareeqe mein koi problem ho, to compiler errors hamare code ke relevant part aur constraints ki taraf zyada precisely point kar sakti hain. Agar is ke bajaye Rust compiler lifetimes ke relationships ke baare mein hamari intention ke zyada inferences karta, to compiler shayad sirf hamare code ke us use ki taraf point kar pata jo problem ki asal wajah se kai steps door hota.

Jab hum `longest` ko concrete references pass karte hain, to `'a` ke liye substitute ki jane wali concrete lifetime, `x` ke scope ka woh hissa hota hai jo `y` ke scope ke saath overlap karta hai. Doosre alfaaz mein, generic lifetime `'a` ko woh concrete lifetime milegi jo `x` aur `y` ki lifetimes mein se chhoti lifetime ke barabar hai. Kyun ke hum ne returned reference ko bhi isi lifetime parameter `'a` ke saath annotate kiya hai, is liye returned reference bhi `x` aur `y` ki lifetimes mein se chhoti lifetime jitni der tak valid rahega.

Aaiye dekhte hain ke mukhtalif concrete lifetimes wale references pass karke lifetime annotations `longest` function ko kis tarah restrict karti hain. Listing 10-22 ek seedha example hai.

<Listing number="10-22" file-name="src/main.rs" caption="Using the `longest` function with references to `String` values that have different concrete lifetimes">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-22/src/main.rs:here}}
```

</Listing>

Is example mein, `string1` outer scope ke end tak valid hai, `string2` inner scope ke end tak valid hai, aur `result` kisi aisi cheez ko reference karta hai jo inner scope ke end tak valid hai. Is code ko run karein aur aap dekhenge ke borrow checker isay approve karta hai; ye compile hoga aur `The longest string is long string is long` print karega.

Ab ek aisa example try karte hain jo dikhata hai ke `result` mein reference ki lifetime dono arguments mein se chhoti lifetime honi chahiye. Hum `result` variable ki declaration ko inner scope ke bahar move karenge, lekin `result` variable ko value assign karna usi scope ke andar rakhenge jahan `string2` hai. Phir hum `result` ko use karne wali `println!` ko inner scope ke bahar, inner scope khatam hone ke baad move karenge. Listing 10-23 ka code compile nahi hoga.

<Listing number="10-23" file-name="src/main.rs" caption="Attempting to use `result` after `string2` has gone out of scope">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/src/main.rs:here}}
```

</Listing>

Jab hum is code ko compile karne ki koshish karte hain, to humein ye error milta hai:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/output.txt}}
```

Error dikhata hai ke `println!` statement ke liye `result` ko valid rakhne ke liye, `string2` ko outer scope ke end tak valid rehna hoga. Rust ye is liye jaanta hai kyun ke hum ne function parameters aur return values ki lifetimes ko same lifetime parameter `'a` use karke annotate kiya hai.

Insaan ke taur par, hum is code ko dekh kar samajh sakte hain ke `string1`, `string2` se lambi hai aur is liye `result` mein `string1` ka reference hoga. Kyun ke `string1` abhi scope se bahar nahi gayi, is liye `string1` ka reference `println!` statement ke liye ab bhi valid hoga. Lekin compiler ye nahi dekh sakta ke is case mein reference valid hai. Hum ne Rust ko bataya hai ke `longest` function se return hone wale reference ki lifetime, pass kiye gaye references ki lifetimes mein se chhoti lifetime ke barabar hai. Is liye borrow checker Listing 10-23 ke code ko is possibility ki wajah se disallow karta hai ke is mein invalid reference ho sakta hai.

Mazeed experiments design karne ki koshish karein jin mein `longest` function ko pass kiye jane wale references ki values aur lifetimes, aur returned reference ke use ko vary kiya jaye. Compile karne se pehle hypotheses banayein ke aapke experiments borrow checker ko pass karenge ya nahi; phir check karein ke aap sahi thay ya nahi!

<!-- Old headings. Do not remove or links may break. -->

<a id="thinking-in-terms-of-lifetimes"></a>

### Relationships

Aapko lifetime parameters kis tarah specify karne ki zaroorat hai, ye is baat par depend karta hai ke aapka function kya kar raha hai. Misal ke taur par, agar hum `longest` function ki implementation ko is tarah change kar dein ke woh hamesha longest string slice ke bajaye pehla parameter return kare, to humein `y` parameter par lifetime specify karne ki zaroorat nahi hogi. Neeche diya gaya code compile ho jayega:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-08-only-one-reference-with-lifetime/src/main.rs:here}}
```

</Listing>

Hum ne parameter `x` aur return type ke liye lifetime parameter `'a` specify kiya hai, lekin parameter `y` ke liye nahi, kyun ke `y` ki lifetime ka `x` ya return value ki lifetime ke saath koi relationship nahi hai.

Jab kisi function se reference return kiya jata hai, to return type ke liye lifetime parameter ko parameters mein se kisi ek ke lifetime parameter se match karna zaroori hota hai. Agar return kiya gaya reference parameters mein se kisi ek ko *refer* nahi karta, to usay is function ke andar create ki gayi kisi value ko refer karna hoga. Lekin ye ek dangling reference hoga kyun ke function ke end par woh value scope se bahar chali jayegi. `longest` function ki is attempted implementation par ghour karein jo compile nahi hogi:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/src/main.rs:here}}
```

</Listing>

Yahan, agarche hum ne return type ke liye lifetime parameter `'a` specify kiya hai, ye implementation compile hone mein fail hogi kyun ke return value ki lifetime ka parameters ki lifetimes ke saath bilkul koi relationship nahi hai. Yahan woh error message hai jo humein milta hai:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/output.txt}}
```

Problem ye hai ke `result`, `longest` function ke end par scope se bahar chala jata hai aur clean up ho jata hai. Hum function se `result` ka reference bhi return karne ki koshish kar rahe hain. Aisa koi tareeqa nahi hai ke hum lifetime parameters specify karke is dangling reference ko change kar saken, aur Rust humein dangling reference create karne ki ijazat nahi deta. Is case mein, behtareen fix ye hoga ke reference ke bajaye owned data type return kiya jaye, taa-ke phir calling function value ko clean up karne ki zimmedari uthaye.

Aakhirkar, lifetime syntax ka maqsad functions ke mukhtalif parameters aur return values ki lifetimes ko aapas mein connect karna hai. Jab ye connect ho jati hain, Rust ke paas memory-safe operations ko allow karne aur dangling pointers create karne wali ya kisi doosre tareeqe se memory safety violate karne wali operations ko disallow karne ke liye kaafi information hoti hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-struct-definitions"></a>

### Struct Definitions Mein

Ab tak, hum ne jo structs define kiye hain, woh tamam owned types hold karte hain. Hum aise structs bhi define kar sakte hain jo references hold karein, lekin is case mein humein struct ki definition mein har reference par lifetime annotation add karni hogi. Listing 10-24 mein `ImportantExcerpt` naam ka ek struct hai jo ek string slice hold karta hai.

<Listing number="10-24" file-name="src/main.rs" caption="A struct that holds a reference, requiring a lifetime annotation">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-24/src/main.rs}}
```

</Listing>

Is struct mein sirf ek field `part` hai jo ek string slice hold karti hai, jo ek reference hai. Generic data types ki tarah, hum generic lifetime parameter ka naam struct ke naam ke baad angle brackets ke andar declare karte hain taa-ke hum struct definition ki body mein lifetime parameter ko use kar saken. Is annotation ka matlab hai ke `ImportantExcerpt` ka koi instance us reference se zyada der tak exist nahi kar sakta jo woh apne `part` field mein hold karta hai.

Yahan `main` function `ImportantExcerpt` struct ka ek instance create karta hai jo `novel` variable ki owned `String` ke pehle sentence ka reference hold karta hai. `novel` mein mojood data `ImportantExcerpt` instance create hone se pehle se exist karta hai. Is ke ilawa, `novel` ka scope `ImportantExcerpt` ke scope se baad mein khatam hota hai, is liye `ImportantExcerpt` instance mein mojood reference valid hai.

### Lifetime Elision

Aap ne seekha hai ke har reference ki ek lifetime hoti hai aur references use karne wale functions ya structs ke liye lifetime parameters specify karna zaroori hota hai. Lekin Listing 4-9 mein hamare paas ek aisa function tha, jo Listing 10-25 mein dobara diya gaya hai, jo lifetime annotations ke baghair compile ho gaya tha.

<Listing number="10-25" file-name="src/lib.rs" caption="A function we defined in Listing 4-9 that compiled without lifetime annotations, even though the parameter and return type are references">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-25/src/main.rs:here}}
```

</Listing>

Ye function lifetime annotations ke baghair compile hone ki wajah historical hai: Rust ke early versions (pre-1.0) mein ye code compile nahi hota, kyun ke har reference ke liye explicit lifetime zaroori hoti thi. Us waqt function signature is tarah likhi jati:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Bohat saara Rust code likhne ke baad, Rust team ne mehsoos kiya ke Rust programmers khaas situations mein ek hi lifetime annotations ko baar baar likh rahe the. Ye situations predictable thin aur kuch deterministic patterns ko follow karti thin. Developers ne in patterns ko compiler ke code mein program kar diya taa-ke borrow checker in situations mein lifetimes infer kar sake aur explicit annotations ki zaroorat na ho.

Rust ki history ka ye hissa relevant hai kyun ke mumkin hai ke future mein mazeed deterministic patterns samne aayein aur compiler mein add kiye jayein. Future mein shayad aur bhi kam lifetime annotations ki zaroorat ho.

Rust ke references ke analysis mein programmed patterns ko *lifetime elision rules* kaha jata hai. Ye programmers ke liye follow karne ke rules nahi hain; balki ye kuch specific cases ka set hain jinhein compiler consider karega, aur agar aapka code in cases mein fit hota hai, to aapko lifetimes explicitly likhne ki zaroorat nahi hoti.

Elision rules complete inference provide nahi kartin. Agar Rust ke rules apply karne ke baad bhi references ki lifetimes ke baare mein ambiguity ho, to compiler ye guess nahi karega ke baqi references ki lifetime kya honi chahiye. Guess karne ke bajaye, compiler aapko ek error dega jise aap lifetime annotations add karke resolve kar sakte hain.

Function ya method parameters par lifetimes ko *input lifetimes* kaha jata hai, aur return values par lifetimes ko *output lifetimes* kaha jata hai.

Jab explicit annotations na hon, to compiler references ki lifetimes figure out karne ke liye teen rules use karta hai. Pehla rule input lifetimes par apply hota hai, aur doosra aur teesra rule output lifetimes par apply hote hain. Agar compiler teenon rules ke end tak pohanch jaye aur phir bhi aise references mojood hon jin ki lifetimes woh figure out nahi kar sakta, to compiler error ke saath ruk jayega. Ye rules `fn` definitions ke saath saath `impl` blocks par bhi apply hote hain.

Pehla rule ye hai ke compiler har aise parameter ko ek lifetime parameter assign karta hai jo ek reference ho. Doosre alfaaz mein, ek parameter wale function ko ek lifetime parameter milta hai: `fn foo<'a>(x: &'a i32)`; do parameters wale function ko do separate lifetime parameters milte hain: `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`; aur isi tarah aage.

Doosra rule ye hai ke agar exactly ek input lifetime parameter ho, to woh lifetime tamam output lifetime parameters ko assign kar di jati hai: `fn foo<'a>(x: &'a i32) -> &'a i32`.

Teesra rule ye hai ke agar multiple input lifetime parameters hon, lekin un mein se ek `&self` ya `&mut self` ho kyun ke ye ek method hai, to `self` ki lifetime tamam output lifetime parameters ko assign kar di jati hai. Ye teesra rule methods ko parhna aur likhna kaafi behtar bana deta hai kyun ke kam symbols ki zaroorat hoti hai.

Aaiye pretend karte hain ke hum compiler hain. Hum in rules ko apply karke Listing 10-25 mein `first_word` function ke signature mein references ki lifetimes figure out karenge. Signature references ke saath kisi lifetime ke baghair start hoti hai:

```rust,ignore
fn first_word(s: &str) -> &str {
```

Phir compiler pehla rule apply karta hai, jo specify karta hai ke har parameter ko apni lifetime milegi. Hum hamesha ki tarah isay `'a` kahenge, to ab signature ye hai:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &str {
```

Doosra rule apply hota hai kyun ke exactly ek input lifetime mojood hai. Doosra rule specify karta hai ke ek input parameter ki lifetime output lifetime ko assign kar di jaye, to ab signature ye hai:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Ab is function signature mein tamam references ki lifetimes mojood hain, aur compiler programmer ko is function signature mein lifetimes annotate karne ki zaroorat ke baghair apna analysis continue kar sakta hai.

Aaiye ek aur example dekhte hain, is baar `longest` function ko use karte hue, jis ke saath jab hum ne Listing 10-20 mein kaam shuru kiya tha to koi lifetime parameters nahi the:

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
```

Aaiye pehla rule apply karte hain: Har parameter ko apni lifetime milti hai. Is baar hamare paas ek ke bajaye do parameters hain, is liye hamare paas do lifetimes hongi:

```rust,ignore
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

Aap dekh sakte hain ke doosra rule apply nahi hota, kyun ke ek se zyada input lifetimes hain. Teesra rule bhi apply nahi hota, kyun ke `longest` ek function hai, method nahi, is liye koi bhi parameter `self` nahi hai. Teenon rules ko apply karne ke baad bhi hum ye figure out nahi kar sake ke return type ki lifetime kya hai. Isi wajah se Listing 10-20 mein code compile karne ki koshish par humein error mila: Compiler ne lifetime elision rules ko apply kiya, lekin phir bhi signature mein references ki tamam lifetimes figure out nahi kar saka.

Kyun ke teesra rule asal mein sirf method signatures par apply hota hai, is liye ab hum isi context mein lifetimes dekhenge taa-ke samajh saken ke teesre rule ki wajah se humein method signatures mein aksar lifetimes annotate karne ki zaroorat kyun nahi padti.

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-method-definitions"></a>

### Method Definitions Mein

Jab hum lifetimes wale struct par methods implement karte hain, to hum wohi syntax use karte hain jo generic type parameters ke liye use hoti hai, jaisa ke Listing 10-11 mein dikhaya gaya hai. Hum lifetime parameters ko kahan declare aur use karte hain, ye is baat par depend karta hai ke woh struct fields se related hain ya method parameters aur return values se.

Struct fields ke liye lifetime names ko hamesha `impl` keyword ke baad declare karna hota hai aur phir struct ke naam ke baad use karna hota hai, kyun ke ye lifetimes struct ke type ka hissa hoti hain.

`impl` block ke andar method signatures mein references struct ke fields mein mojood references ki lifetime ke saath tied ho sakti hain, ya woh independent bhi ho sakti hain. Is ke ilawa, lifetime elision rules aksar is tarah kaam karti hain ke method signatures mein lifetime annotations ki zaroorat nahi padti. Aaiye `ImportantExcerpt` naam ke struct ko use karte hue kuch examples dekhte hain jo hum ne Listing 10-24 mein define kiya tha.

Sab se pehle, hum `level` naam ka ek method use karenge jis ka sirf ek parameter `self` ka reference hai aur jis ki return value ek `i32` hai, jo kisi cheez ka reference nahi hai:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:1st}}
```

`impl` ke baad lifetime parameter declaration aur type name ke baad us ka use zaroori hai, lekin pehle elision rule ki wajah se humein `self` ke reference ki lifetime ko annotate karne ki zaroorat nahi hai.

Yahan ek example hai jahan teesra lifetime elision rule apply hota hai:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:3rd}}
```

Do input lifetimes hain, is liye Rust pehla lifetime elision rule apply karta hai aur `&self` aur `announcement` dono ko apni apni lifetimes deta hai. Phir, kyun ke parameters mein se ek `&self` hai, return type ko `&self` ki lifetime mil jati hai, aur tamam lifetimes account ho jati hain.

### The Static Lifetime

Ek khaas lifetime jis par humein baat karni chahiye woh `'static` hai, jo ye denote karti hai ke affected reference *program ki poori duration* tak live kar sakta hai. Tamam string literals ki lifetime `'static` hoti hai, jise hum is tarah annotate kar sakte hain:

```rust
let s: &'static str = "I have a static lifetime.";
```

Is string ka text seedha program ki binary mein store hota hai, jo hamesha available hoti hai. Is liye, tamam string literals ki lifetime `'static` hoti hai.

Aap error messages mein `'static` lifetime use karne ki suggestions dekh sakte hain. Lekin kisi reference ke liye `'static` ko lifetime specify karne se pehle, ye sochein ke kya aapke paas jo reference hai woh waqai aapke program ki poori lifetime tak live karta hai, aur kya aap chahte bhi hain ke woh aisa kare. Zyada tar waqt, `'static` lifetime suggest karne wala error message dangling reference create karne ki koshish ya available lifetimes ke mismatch ki wajah se hota hai. Aise cases mein solution un problems ko fix karna hai, `'static` lifetime specify karna nahi.

<!-- Old headings. Do not remove or links may break. -->

<a id="generic-type-parameters-trait-bounds-and-lifetimes-together"></a>

## Generic Type Parameters, Trait Bounds, and Lifetimes

Aaiye mukhtasar taur par dekhein ke ek hi function mein generic type parameters, trait bounds, aur lifetimes ko specify karne ka syntax kya hota hai!

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-11-generics-traits-and-lifetimes/src/main.rs:here}}
```

Ye Listing 10-21 ka `longest` function hai jo do string slices mein se zyada lambi string slice return karta hai. Lekin ab is mein `ann` naam ka ek extra parameter hai jo generic type `T` ka hai, jise koi bhi aisa type fill kar sakta hai jo `Display` trait ko implement karta ho, jaisa ke `where` clause mein specify kiya gaya hai. Is extra parameter ko `{}` ka use karte hue print kiya jayega, isi liye `Display` trait bound zaroori hai. Kyun ke lifetimes bhi generic ki ek type hain, is liye lifetime parameter `'a` aur generic type parameter `T` ki declarations function name ke baad angle brackets ke andar ek hi list mein hoti hain.

## Summary

Hum ne is chapter mein bohat kuch cover kiya! Ab jab aap generic type parameters, traits aur trait bounds, aur generic lifetime parameters ke baare mein jaante hain, aap aisa code likhne ke liye tayyar hain jo repetition ke baghair kai mukhtalif situations mein kaam karta hai. Generic type parameters aapko code ko mukhtalif types par apply karne dete hain. Traits aur trait bounds ye ensure karte hain ke types generic hone ke bawajood, un mein woh behavior hoga jis ki code ko zaroorat hai. Aap ne seekha ke lifetime annotations kaise use karni hain taa-ke ye flexible code kisi dangling references ke saath kaam na kare. Aur ye tamam analysis compile time par hota hai, is liye runtime performance par koi asar nahi padta!

Yaqeen karein ya na karein, is chapter mein discuss kiye gaye topics par seekhne ke liye abhi bohat kuch baqi hai: Chapter 18 trait objects discuss karta hai, jo traits ko use karne ka ek aur tareeqa hain. Lifetime annotations se mutalliq mazeed complex scenarios bhi hain jin ki aapko sirf bohat advanced scenarios mein zaroorat padegi; un ke liye aapko [Rust Reference][reference] parhna chahiye. Lekin aglay chapter mein, aap Rust mein tests likhna seekhenge taa-ke aap yaqeen kar saken ke aapka code usi tarah kaam kar raha hai jis tarah usay karna chahiye.

[references-and-borrowing]: ch04-02-references-and-borrowing.html#references-and-borrowing
[string-slices-as-parameters]: ch04-03-slices.html#string-slices-as-parameters
[reference]: ../reference/trait-bounds.html
