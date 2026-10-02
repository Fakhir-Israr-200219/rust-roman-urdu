## The Slice Type

*Slices* aapko kisi [collection](ch08-00-common-collections.md)<!-- ignore --> mein elements ki ek contiguous sequence ko reference karne dete hain. Slice ek qisam ka reference hota hai, is liye iski ownership nahi hoti.

Yahan ek chhota sa programming problem hai: Ek aisa function likhein jo spaces se separate kiye gaye words ki ek string le aur us string mein milne wala pehla word return kare. Agar function ko string mein koi space na mile, to poori string ek word honi chahiye, is liye poori string return honi chahiye.

> Note: Slices introduce karne ke maqsad ke liye, hum is section mein sirf ASCII assume kar rahe hain; UTF-8 handling ki zyada thorough discussion Chapter 8 ke [“Storing UTF-8 Encoded Text with Strings”][strings]<!-- ignore --> section mein hai.

Aaiye step by step dekhte hain ke slices ko use kiye baghair hum is function ki signature kaise likhenge, taa-ke samajh saken ke slices kis problem ko solve karengi:

```rust,ignore
fn first_word(s: &String) -> ?
```

`first_word` function ka parameter `&String` type ka hai. Humein ownership ki zaroorat nahi, is liye ye theek hai. (Idiomatic Rust mein, functions apne arguments ki ownership tab tak nahi lete jab tak unhein iski zaroorat na ho, aur iski wajahen jaise jaise hum aage barhenge clear hoti jayengi.) Lekin humein return kya karna chahiye? Hamare paas string ke *ek part* ke baare mein baat karne ka koi proper tareeqa nahi hai. Lekin hum word ke end ka index return kar sakte hain, jise ek space indicate karti hai. Aaiye Listing 4-7 mein dikhaye gaye tareeqe se ise try karte hain.

<Listing number="4-7" file-name="src/main.rs" caption="`first_word` function jo `String` parameter mein byte index value return karta hai">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:here}}
```

</Listing>

Kyun ke humein `String` ko element by element traverse karke check karna hai ke koi value space hai ya nahi, hum `as_bytes` method ko use karke apni `String` ko bytes ki ek array mein convert karenge.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:as_bytes}}
```

Is ke baad, hum `iter` method ko use karke bytes ki array par ek iterator create karte hain:

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:iter}}
```

Hum Chapter 13 mein iterators par mazeed detail se baat karenge [Chapter 13][ch13]<!-- ignore -->. Filhaal, itna jaan lein ke `iter` ek aisa method hai jo collection mein mojood har element return karta hai aur `enumerate` `iter` ke result ko wrap karta hai aur har element ko ek tuple ke hisse ke taur par return karta hai. `enumerate` se return hone wale tuple ka pehla element index hota hai aur doosra element element ka reference hota hai. Ye khud index calculate karne ke muqable mein thora zyada convenient hai.

Kyun ke `enumerate` method ek tuple return karta hai, hum us tuple ko destructure karne ke liye patterns use kar sakte hain. Hum Chapter 6 mein patterns par mazeed baat karenge [Chapter 6][ch6]<!-- ignore -->. `for` loop mein hum ek aisa pattern specify karte hain jisme tuple ke index ke liye `i` aur tuple mein mojood single byte ke liye `&item` hai. Kyun ke humein `.iter().enumerate()` se element ka reference milta hai, is liye hum pattern mein `&` use karte hain.

`for` loop ke andar, hum byte literal syntax ko use karke us byte ko search karte hain jo space ko represent karta hai. Agar humein space mil jaye, to hum uski position return kar dete hain. Warna, hum `s.len()` ko use karke string ki length return kar dete hain.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:inside_for}}
```

Ab hamare paas string mein pehle word ke end ka index find karne ka ek tareeqa hai, lekin ek problem hai. Hum apne aap mein ek `usize` return kar rahe hain, lekin ye number sirf `&String` ke context mein meaningful hai. Doosre alfaaz mein, kyun ke ye `String` se separate ek value hai, is baat ki koi guarantee nahi hai ke future mein bhi ye valid rahega. Listing 4-8 mein diye gaye program ko dekhein jo Listing 4-7 ke `first_word` function ko use karta hai.

<Listing number="4-8" file-name="src/main.rs" caption="`first_word` function ko call karne ka result store karna aur phir `String` ke contents ko change karna">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-08/src/main.rs:here}}
```

</Listing>

Ye program baghair kisi error ke compile hota hai aur agar hum `s.clear()` call karne ke baad `word` ko use karein tab bhi compile hoga. Kyun ke `word` ka `s` ki state ke saath koi connection nahi hai, `word` mein ab bhi value `5` mojood hai. Hum is value `5` ko variable `s` ke saath use karke pehla word extract karne ki koshish kar sakte hain, lekin ye ek bug hoga kyun ke `word` mein `5` save karne ke baad `s` ke contents change ho chuke hain.

`word` mein index ke `s` ke data ke saath out of sync hone ki fikr karna tedious aur error-prone hai! Agar hum `second_word` function likhein to in indices ko manage karna aur bhi brittle ho jayega. Iski signature kuch is tarah dikhni padegi:

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

Ab hum starting *aur* ending index ko track kar rahe hain, aur hamare paas aur bhi zyada aisi values hain jo kisi particular state mein data se calculate ki gayi hain lekin us state ke saath bilkul tied nahi hain. Hamare paas teen unrelated variables idhar udhar mojood hain jinhein sync mein rakhna zaroori hai.

Khush qismati se, Rust ke paas is problem ka solution hai: string slices.

### String Slices

Ek *string slice* `String` ke elements ki ek contiguous sequence ka reference hota hai, aur ye is tarah nazar aata hai:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-17-slice/src/main.rs:here}}
```

Poori `String` ke reference ke bajaye, `hello` `String` ke ek portion ka reference hai, jise extra `[0..5]` bit specify karti hai. Hum square brackets ke andar ek range specify karke slices create karte hain, yani `[starting_index..ending_index]`, jahan *`starting_index`* slice ki pehli position hoti hai aur *`ending_index`* slice ki aakhri position se ek zyada hota hai. Internally, slice data structure starting position aur slice ki length store karta hai, jo *`ending_index`* minus *`starting_index`* ke barabar hoti hai. Is liye, `let world = &s[6..11];` ke case mein, `world` ek aisa slice hoga jo `s` ke index 6 par mojood byte ka pointer aur `5` ki length value rakhta hai.

Figure 4-7 isay ek diagram mein dikhati hai.

<img alt="Three tables: a table representing the stack data of s, which points
to the byte at index 0 in a table of the string data &quot;hello world&quot; on
the heap. The third table represents the stack data of the slice world, which
has a length value of 5 and points to byte 6 of the heap data table."
src="img/trpl04-07.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-7: `String` ke ek hissa ko refer karta hua string slice</span>

Rust ki `..` range syntax ke saath, agar aap index 0 se start karna chahte hain, to aap do periods se pehle wali value ko hata sakte hain. Doosre alfaaz mein, ye dono barabar hain:

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

Isi tarah, agar aapke slice mein `String` ka aakhri byte shamil hai, to aap trailing number ko hata sakte hain. Is ka matlab hai ke ye dono barabar hain:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

Aap dono values ko bhi hata sakte hain taa-ke poori string ka slice le saken. Is liye, ye dono barabar hain:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> Note: String slice ke range indices ka valid UTF-8 character boundaries par hona zaroori hai. Agar aap kisi multibyte character ke darmiyan string slice create karne ki koshish karein, to aapka program error ke saath exit ho jayega.

In tamam maloomat ko zehan mein rakhte hue, aaiye `first_word` ko dobara likhte hain taa-ke ye ek slice return kare. “String slice” ko represent karne wali type `&str` likhi jati hai:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-18-first-word-slice/src/main.rs:here}}
```

</Listing>

Humein word ke end ka index usi tarah milta hai jis tarah Listing 4-7 mein mila tha, yani space ke pehle occurrence ko dhoondh kar. Jab humein space mil jati hai, to hum string ke start aur space ke index ko starting aur ending indices ke taur par use karke ek string slice return kar dete hain.

Ab jab hum `first_word` ko call karte hain, to humein ek single value milti hai jo underlying data ke saath tied hoti hai. Ye value slice ke starting point ke ek reference aur slice mein elements ki tadaad se mil kar banti hai.

`second_word` function ke liye bhi slice return karna kaam karega:

```rust,ignore
fn second_word(s: &String) -> &str {
```

Ab hamare paas ek straightforward API hai jise ghalat use karna kaafi mushkil hai, kyun ke compiler ensure karega ke `String` ke andar references valid rahen. Listing 4-8 mein program wala bug yaad karein, jab humne pehle word ke end ka index hasil kiya tha aur phir string ko clear kar diya tha, jis se hamara index invalid ho gaya tha? Woh code logically incorrect tha lekin foran koi error nazar nahi aaya. Agar hum emptied string ke saath first word ke index ko use karne ki koshish karte rehte, to problems baad mein saamne aatin. Slices is bug ko impossible bana deti hain aur humein bohat pehle hi bata deti hain ke hamare code mein problem hai. `first_word` ka slice version use karne se compile-time error milega:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/src/main.rs:here}}
```

</Listing>

Yahan compiler error hai:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/output.txt}}
```

Borrowing rules se yaad karein ke agar hamare paas kisi cheez ka immutable reference hai, to hum us ka mutable reference bhi nahi le sakte. Kyun ke `clear` ko `String` ko truncate karne ki zaroorat hoti hai, is liye use mutable reference lena padta hai. `clear` ki call ke baad `println!` `word` mein mojood reference ko use karta hai, is liye us point par immutable reference ab bhi active hona zaroori hai. Rust `clear` ke mutable reference aur `word` ke immutable reference ko ek hi waqt mein exist karne ki ijazat nahi deta, aur compilation fail ho jati hai. Rust ne na sirf hamari API ko use karna aasaan bana diya hai, balki compile time par errors ki ek poori category ko bhi eliminate kar diya hai!

<!-- Old headings. Do not remove or links may break. -->

<a id="string-literals-are-slices"></a>

#### String Literals as Slices

Yaad karein ke hum ne baat ki thi ke string literals binary ke andar store hoti hain. Ab jab hum slices ke baare mein jaante hain, to hum string literals ko properly samajh sakte hain:

```rust
let s = "Hello, world!";
```

Yahan `s` ki type `&str` hai: Ye binary ke us specific point ki taraf point karne wala slice hai. Isi wajah se string literals immutable bhi hoti hain; `&str` ek immutable reference hai.

#### String Slices as Parameters

Ye jaanna ke aap literals aur `String` values ke slices le sakte hain, humein `first_word` mein ek aur improvement ki taraf le jata hai, aur woh hai iski signature:

```rust,ignore
fn first_word(s: &String) -> &str {
```

Ek zyada experienced Rustacean is signature ko use karega jo Listing 4-9 mein dikhayi gayi hai, kyun ke is se hum same function ko `&String` values aur `&str` values dono par use kar sakte hain.

<Listing number="4-9" caption="`s` parameter ki type ke liye string slice use karke `first_word` function ko behtar banana">

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:here}}
```

</Listing>

Agar hamare paas string slice ho, to hum use directly pass kar sakte hain. Agar hamare paas `String` ho, to hum `String` ka slice ya `String` ka reference pass kar sakte hain. Ye flexibility deref coercions ka faida uthati hai, jo ek aisa feature hai jise hum Chapter 15 ke [“Using Deref Coercions in Functions and Methods”][deref-coercions]<!--
ignore --> section mein cover karenge.

`String` ke reference ke bajaye string slice lene ke liye function define karna hamari API ko zyada general aur useful bana deta hai, baghair kisi functionality ko lose kiye:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:usage}}
```

</Listing>

### Other Slices

String slices, jaisa ke aap andaza laga sakte hain, strings ke liye specific hoti hain. Lekin ek zyada general slice type bhi hoti hai. Is array par gaur karein:

```rust
let a = [1, 2, 3, 4, 5];
```

Jis tarah hum string ke kisi part ko refer karna chah sakte hain, isi tarah hum array ke kisi part ko bhi refer karna chah sakte hain. Hum ye is tarah karenge:

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

Is slice ki type `&[i32]` hai. Ye string slices ki tarah hi kaam karti hai, yani pehle element ka reference aur ek length store karti hai. Aap is qisam ki slice ko har tarah ki doosri collections ke liye use karenge. Hum Chapter 8 mein vectors ke baare mein baat karte waqt in collections ko detail mein discuss karenge.


## Summary

Ownership, borrowing, aur slices ke concepts Rust programs mein compile time par memory safety ko ensure karte hain. Rust language aapko apni memory usage par usi tarah control deti hai jaise doosri systems programming languages deti hain. Lekin data ka owner automatically us data ko clean up kar deta hai jab owner scope se bahar chala jata hai, jis ka matlab hai ke is control ko hasil karne ke liye aapko extra code likhne aur debug karne ki zaroorat nahi padti.

Ownership Rust ke bohat se doosre parts ke kaam karne ke tareeqe ko affect karti hai, is liye kitab ke baqi hisson mein hum in concepts par mazeed baat karenge. Aaiye Chapter 5 ki taraf chalte hain aur dekhte hain ke data ke mukhtalif pieces ko ek `struct` mein kis tarah ek saath group kiya jata hai.

[ch13]: ch13-02-iterators.html
[ch6]: ch06-02-match.html#patterns-that-bind-to-values
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[deref-coercions]: ch15-02-deref.html#using-deref-coercions-in-functions-and-methods

