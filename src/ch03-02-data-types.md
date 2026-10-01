## Data Types

Rust mein har value ek khaas *data type* ki hoti hai, jo Rust ko batati hai ke kis qisam ka data specify kiya gaya hai taa-ke woh jaan sake ke us data ke saath kaise kaam karna hai. Hum data types ke do subsets dekhenge: scalar aur compound.

Yaad rakhein ke Rust ek *statically typed* language hai, jis ka matlab hai ke compile time par Rust ko tamam variables ki types ka maloom hona zaroori hai. Compiler aam tor par value aur is baat ki bunyaad par infer kar sakta hai ke hum usay kis tarah use kar rahe hain, ke hum kaunsi type use karna chahte hain. Jab kai types possible hon, jaise Chapter 2 ke [“Comparing the Guess to the Secret Number”][comparing-the-guess-to-the-secret-number]<!-- ignore --> section mein `parse` ko use karke hum ne `String` ko numeric type mein convert kiya tha, to humein type annotation add karni hoti hai, jaise:

```rust
let guess: u32 = "42".parse().expect("Not a number!");
```

Agar hum preceding code mein dikhayi gayi `: u32` type annotation add na karein, to Rust following error display karega, jis ka matlab hai ke compiler ko ye jaanne ke liye hum se mazeed information chahiye ke hum kaunsi type use karna chahte hain:

```console
{{#include ../listings/ch03-common-programming-concepts/output-only-01-no-type-annotations/output.txt}}
```

Aap doosri data types ke liye different type annotations dekhenge.

### Scalar Types

Ek *scalar* type ek single value ko represent karti hai. Rust mein chaar primary scalar types hain: integers, floating-point numbers, Booleans, aur characters. Aap inhein doosri programming languages se bhi pehchan sakte hain. Aaiye dekhte hain ke ye Rust mein kaise kaam karti hain.

#### Integer Types

Ek *integer* aisa number hota hai jisme fractional component nahi hota. Hum ne Chapter 2 mein ek integer type, `u32`, use ki thi. Ye type declaration indicate karti hai ke jis value ke saath ye associated hai woh ek unsigned integer honi chahiye (signed integer types `u` ke bajaye `i` se start hoti hain) jo 32 bits ki space leti hai. Table 3-1 Rust mein built-in integer types dikhati hai. Hum integer value ki type declare karne ke liye in mein se koi bhi variant use kar sakte hain.

<span class="caption">Table 3-1: Rust mein Integer Types</span>

| Length                 | Signed  | Unsigned |
| ---------------------- | ------- | -------- |
| 8-bit                  | `i8`    | `u8`     |
| 16-bit                 | `i16`   | `u16`    |
| 32-bit                 | `i32`   | `u32`    |
| 64-bit                 | `i64`   | `u64`    |
| 128-bit                | `i128`  | `u128`   |
| Architecture-dependent | `isize` | `usize`  |

Har variant signed ya unsigned ho sakta hai aur iska ek explicit size hota hai. *Signed* aur *unsigned* se murad ye hai ke kya number negative ho sakta hai—in other words, kya number ke saath sign hona zaroori hai (signed) ya woh hamesha positive hoga aur is liye use sign ke baghair represent kiya ja sakta hai (unsigned). Ye bilkul aisa hai jaise paper par numbers likhna: Jab sign matter karta hai, to number ko plus sign ya minus sign ke saath dikhaya jata hai; lekin jab ye assume karna safe ho ke number positive hai, to use baghair kisi sign ke dikhaya jata hai. Signed numbers ko [two’s complement][twos-complement]<!-- ignore
--> representation use karke store kiya jata hai.

Har signed variant −(2<sup>n − 1</sup>) se 2<sup>n − 1</sup> − 1 tak ke numbers store kar sakta hai, dono limits included hain, jahan *n* us variant ke zariye use kiye jane wale bits ki tadaad hai. Is liye, ek `i8` −(2<sup>7</sup>) se 2<sup>7</sup> − 1 tak ke numbers store kar sakta hai, jo −128 se 127 ke barabar hai. Unsigned variants 0 se 2<sup>n</sup> − 1 tak ke numbers store kar sakte hain, is liye ek `u8` 0 se 2<sup>8</sup> − 1 tak ke numbers store kar sakta hai, jo 0 se 255 ke barabar hai.

Is ke ilawa, `isize` aur `usize` types us computer ke architecture par depend karti hain jis par aapka program run ho raha hai: Agar aap 64-bit architecture par hain to 64 bits, aur agar aap 32-bit architecture par hain to 32 bits.

Aap integer literals ko Table 3-2 mein dikhayi gayi kisi bhi form mein likh sakte hain. Note karein ke woh number literals jo multiple numeric types ho sakte hain, type designate karne ke liye type suffix use kar sakte hain, jaise `57u8`. Number literals mein `_` ko visual separator ke taur par bhi use kiya ja sakta hai taa-ke number ko parhna aasaan ho, jaise `1_000`, jis ki value bilkul wahi hogi jo `1000` specify karne par hoti.

<span class="caption">Table 3-2: Rust mein Integer Literals</span>

| Number literals  | Example       |
| ---------------- | ------------- |
| Decimal          | `98_222`      |
| Hex              | `0xff`        |
| Octal            | `0o77`        |
| Binary           | `0b1111_0000` |
| Byte (`u8` only) | `b'A'`        |

To phir aap kaise jaanenge ke kaunsi integer type use karni hai? Agar aap unsure hain, to Rust ke defaults aam tor par shuru karne ke liye achhi jagah hain: Integer types default taur par `i32` hoti hain. Primary situation jahan aap `isize` ya `usize` use karenge, woh kisi qisam ki collection ko index karte waqt hoti hai.

[twos-complement]: https://en.wikipedia.org/wiki/Two%27s_complement

> ##### Integer Overflow

> Maan lein ke aapke paas `u8` type ka ek variable hai jo 0 aur 255 ke darmiyan values hold kar sakta hai. Agar aap variable ko is range se bahar kisi value, jaise 256, par change karne ki koshish karein, to *integer overflow* hoga, jo do mein se kisi ek behavior ka sabab ban sakta hai. Jab aap debug mode mein compile kar rahe hote hain, Rust integer overflow ke checks include karta hai jo agar ye behavior occur ho to aapke program ko runtime par *panic* karwa dete hain. Rust us situation ke liye *panicking* term use karta hai jab koi program error ke saath exit hota hai; hum Chapter 9 ke [“Unrecoverable Errors with
> `panic!`”][unrecoverable-errors-with-panic]<!-- ignore --> section mein panics ko mazeed detail mein discuss karenge.
>
> Jab aap `--release` flag ke saath release mode mein compile kar rahe hote hain, Rust integer overflow ke aise checks include *nahi* karta jo panics ka sabab bante hain. Is ke bajaye, agar overflow ho, to Rust *two’s complement wrapping* perform karta hai. Mukhtasar taur par, type ki maximum value se zyada values “wrap around” hokar un values ki minimum par chali jati hain jinhein woh type hold kar sakti hai. `u8` ke case mein, value 256, 0 ban jati hai, value 257, 1 ban jati hai, aur isi tarah aage. Program panic nahi karega, lekin variable ki value shayad woh nahi hogi jis ki aap expectation kar rahe the. Integer overflow ke wrapping behavior par rely karna ek error mana jata hai.
>
> Overflow ke possibility ko explicitly handle karne ke liye, aap primitive numeric types ke liye standard library ki taraf se provide kiye gaye in methods ke families use kar sakte hain:
>
> * Har compilation mode mein wrap karne ke liye `wrapping_*` methods use karein, jaise `wrapping_add`.
> * Overflow hone ki surat mein `None` value return karne ke liye `checked_*` methods use karein.
> * Value aur ek Boolean return karne ke liye `overflowing_*` methods use karein jo indicate karta hai ke overflow hua ya nahi.
> * Value ki minimum ya maximum values par saturate karne ke liye `saturating_*` methods use karein.


#### Floating-Point Types

Rust mein *floating-point numbers* ke liye do primitive types bhi hain, jo decimal points wale numbers hote hain. Rust ki floating-point types `f32` aur `f64` hain, jo respectively 32 bits aur 64 bits ki size rakhti hain. Default type `f64` hai kyun ke modern CPUs par ye roughly `f32` jitni hi fast hoti hai, lekin zyada precision provide karti hai. Tamam floating-point types signed hoti hain.

Yahan ek example hai jo floating-point numbers ko action mein dikhata hai:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-06-floating-point/src/main.rs}}
```

Floating-point numbers ko IEEE-754 standard ke mutabiq represent kiya jata hai.


#### Numeric Operations

Rust tamam number types ke liye woh basic mathematical operations support karta hai jin ki aap expectation karte hain: addition, subtraction, multiplication, division, aur remainder. Integer division zero ki taraf truncate hoti hai aur nearest integer tak jati hai. Following code dikhata hai ke aap `let` statement mein har numeric operation ko kaise use karenge:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-07-numeric-operations/src/main.rs}}
```

In statements mein har expression ek mathematical operator use karta hai aur ek single value mein evaluate hota hai, jo phir ek variable ke saath bind kar di jati hai. [Appendix B][appendix_b]<!-- ignore --> mein Rust ki taraf se provide kiye gaye tamam operators ki list mojood hai.

#### The Boolean Type

Jaisa ke zyada tar doosri programming languages mein hota hai, Rust mein Boolean type ki do possible values hoti hain: `true` aur `false`. Booleans ki size ek byte hoti hai. Rust mein Boolean type ko `bool` use karke specify kiya jata hai. Misal ke taur par:

<span class="filename">Filename: src/main.rs</span>

```rust id="4trq9f"
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-08-boolean/src/main.rs}}
```

Boolean values ko use karne ka main tareeqa conditionals ke zariye hai, jaise ek `if` expression. Hum [“Control Flow”][control-flow]<!-- ignore --> section mein cover karenge ke Rust mein `if` expressions kaise kaam karte hain.

#### The Character Type

Rust ka `char` type language ka sab se primitive alphabetic type hai. Yahan `char` values declare karne ki kuch examples hain:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-09-char/src/main.rs}}
```

Note karein ke hum `char` literals ko single quotation marks ke saath specify karte hain, jabke string literals double quotation marks use karte hain. Rust ka `char` type 4 bytes ki size rakhta hai aur ek Unicode scalar value ko represent karta hai, jis ka matlab hai ke ye sirf ASCII se kahin zyada represent kar sakta hai. Accented letters; Chinese, Japanese, aur Korean characters; emojis; aur zero-width spaces sab Rust mein valid `char` values hain. Unicode scalar values `U+0000` se `U+D7FF` aur `U+E000` se `U+10FFFF` tak, dono limits samet, hoti hain. Lekin, Unicode mein “character” asal mein koi proper concept nahi hai, is liye aapki human intuition ke mutabiq jo “character” hai, zaroori nahi ke woh Rust mein `char` ke concept se match kare. Hum Chapter 8 mein [“Storing UTF-8 Encoded Text with Strings”][strings]<!-- ignore --> mein is topic ko detail se discuss karenge.

### Compound Types

*Compound types* multiple values ko ek hi type mein group kar sakti hain. Rust mein do primitive compound types hain: tuples aur arrays.

#### The Tuple Type

Ek *tuple* mukhtalif types ki kai values ko ek compound type mein group karne ka ek general tareeqa hai. Tuples ki length fixed hoti hai: Ek baar declare ho jayein, to unki size na barh sakti hai aur na kam ho sakti hai.

Hum parentheses ke andar comma-separated values ki list likh kar tuple create karte hain. Tuple mein har position ki ek type hoti hai, aur tuple mein different values ki types ka same hona zaroori nahi hai. Is example mein hum ne optional type annotations add ki hain:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-10-tuples/src/main.rs}}
```

Variable `tup` poore tuple ke saath bind hota hai kyun ke tuple ko ek single compound element consider kiya jata hai. Tuple ki individual values hasil karne ke liye hum pattern matching ko use karke tuple value ko destructure kar sakte hain, is tarah:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-11-destructuring-tuples/src/main.rs}}
```

Ye program sab se pehle ek tuple create karta hai aur use variable `tup` ke saath bind karta hai. Phir ye `let` ke saath ek pattern use karta hai taake `tup` ko lekar use teen separate variables, `x`, `y`, aur `z` mein convert kar de. Isay *destructuring* kaha jata hai kyun ke ye single tuple ko teen parts mein break karta hai. Aakhir mein, program `y` ki value print karta hai, jo `6.4` hai.

Hum tuple ke kisi element ko directly bhi access kar sakte hain, value ke index ke baad period (`.`) use karke. Misal ke taur par:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-12-tuple-indexing/src/main.rs}}
```

Ye program tuple `x` create karta hai aur phir unke respective indices ko use karke tuple ke har element ko access karta hai. Zyada tar programming languages ki tarah, tuple mein pehla index 0 hota hai.

Bina kisi value wale tuple ka ek khaas naam *unit* hai. Ye value aur iski corresponding type dono `()` likhe jate hain aur ek empty value ya empty return type ko represent karte hain. Expressions agar koi doosri value return nahi karte, to implicitly unit value return karte hain.

#### The Array Type

Multiple values ki collection rakhne ka ek aur tareeqa *array* hai. Tuple ke baraks, array ka har element same type ka hona chahiye. Kuch doosri languages ke arrays ke baraks, Rust mein arrays ki length fixed hoti hai.

Hum array mein values ko square brackets ke andar comma-separated list ki surat mein likhte hain:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-13-arrays/src/main.rs}}
```

Arrays us waqt useful hoti hain jab aap chahte hain ke aapka data stack par allocate ho, bilkul un doosri types ki tarah jinhein hum ne ab tak dekha hai, na ke heap par (hum [Chapter 4][stack-and-heap]<!-- ignore --> mein stack aur heap ke baare mein mazeed detail se discuss karenge), ya jab aap ye ensure karna chahte hain ke aapke paas hamesha elements ki ek fixed number ho. Lekin array vector type jitni flexible nahi hoti. Vector ek similar collection type hai jo standard library provide karti hai aur jis ki size barhne ya kam hone ki ijazat hoti hai kyun ke iska content heap par rehta hai. Agar aap unsure hain ke array use karni hai ya vector, to chances hain ke aapko vector use karni chahiye. [Chapter 8][vectors]<!-- ignore --> mein vectors ko mazeed detail mein discuss kiya gaya hai.

Lekin arrays us waqt zyada useful hoti hain jab aap jaante hon ke elements ki number change nahi hogi. Misal ke taur par, agar aap kisi program mein months ke names use kar rahe hon, to aap shayad vector ke bajaye array use karenge kyun ke aap jaante hain ke is mein hamesha 12 elements honge:

```rust
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

Aap array ki type square brackets mein har element ki type, ek semicolon, aur phir array mein elements ki number likh kar specify karte hain, is tarah:

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

Yahan `i32` har element ki type hai. Semicolon ke baad number `5` indicate karta hai ke array mein paanch elements hain.

Aap array ko is tarah bhi initialize kar sakte hain ke uske har element mein same value ho: Pehle initial value specify karein, phir semicolon, aur uske baad square brackets mein array ki length likhein, jaisa ke yahan dikhaya gaya hai:

```rust
let a = [3; 5];
```

`a` naam ki array mein `5` elements honge aur shuru mein sab ki value `3` set hogi. Ye `let a = [3, 3, 3, 3, 3];` likhne ke barabar hai, lekin zyada concise tareeqe se.

<!-- Old headings. Do not remove or links may break. -->

<a id="accessing-array-elements"></a>

#### Array Element Access

Ek array known, fixed size ki memory ka ek single chunk hoti hai jo stack par allocate ki ja sakti hai. Aap indexing ko use karke array ke elements ko access kar sakte hain, is tarah:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-14-array-indexing/src/main.rs}}
```

Is example mein `first` naam ka variable value `1` hasil karega kyun ke ye array ke index `[0]` par mojood value hai. `second` naam ka variable array ke index `[1]` se value `2` hasil karega.

#### Invalid Array Element Access

Aaiye dekhte hain ke agar aap array ke end se aage kisi element ko access karne ki koshish karein to kya hota hai. Maan lein ke aap Chapter 2 ke guessing game ki tarah ye code run karte hain, jo user se array ka index hasil karta hai:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,panics
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access/src/main.rs}}
```

Ye code successfully compile ho jata hai. Agar aap is code ko `cargo run` ke zariye run karein aur `0`, `1`, `2`, `3`, ya `4` enter karein, to program array mein us index par mojood corresponding value print karega. Lekin agar aap array ke end se aage koi number enter karein, jaise `10`, to aapko kuch is tarah ka output nazar aayega:

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access
cargo run
10
-->

```console
thread 'main' panicked at src/main.rs:19:19:
index out of bounds: the len is 5 but the index is 10
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Program ko indexing operation mein invalid value use karne ke point par runtime error ka saamna hua. Program error message ke saath exit ho gaya aur final `println!` statement execute nahi ki. Jab aap indexing use karke kisi element ko access karne ki koshish karte hain, Rust check karta hai ke aap ne jo index specify kiya hai woh array ki length se chhota hai. Agar index length se greater ya us ke barabar ho, to Rust panic karega. Ye check runtime par hona zaroori hai, khaas taur par is case mein, kyun ke compiler ye jaan hi nahi sakta ke user baad mein code run karte waqt kaunsi value enter karega.

Ye Rust ke memory safety principles ki ek example hai jo action mein nazar aati hai. Bohat si low-level languages mein is qisam ka check nahi kiya jata, aur jab aap incorrect index provide karte hain, to invalid memory access ki ja sakti hai. Rust aapko is qisam ki error se foran exit karke protect karta hai, bajaye is ke ke memory access allow ki jaye aur program continue karta rahe. Chapter 9 mein Rust ke error handling aur is baat par mazeed discussion hai ke aap readable, safe code kaise likh sakte hain jo na panic kare aur na invalid memory access allow kare.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[twos-complement]: https://en.wikipedia.org/wiki/Two%27s_complement
[control-flow]: ch03-05-control-flow.html#control-flow
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[stack-and-heap]: ch04-01-what-is-ownership.html#the-stack-and-the-heap
[vectors]: ch08-01-vectors.html
[unrecoverable-errors-with-panic]: ch09-01-unrecoverable-errors-with-panic.html
[appendix_b]: appendix-02-operators.md

