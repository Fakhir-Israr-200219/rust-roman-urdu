## Functions

Rust code mein functions bohat zyada use hote hain. Aap language ke sab se important functions mein se ek ko pehle hi dekh chuke hain: `main` function, jo bohat se programs ka entry point hai. Aap `fn` keyword bhi dekh chuke hain, jo aapko naye functions declare karne deta hai.

Rust code function aur variable names ke liye *snake case* ko conventional style ke taur par use karta hai, jisme tamam letters lowercase hote hain aur words ko separate karne ke liye underscores use kiye jate hain. Yahan ek program hai jisme ek example function definition mojood hai:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-16-functions/src/main.rs}}
```

Hum Rust mein `fn` enter karke, uske baad function name aur parentheses ka ek set likh kar function define karte hain. Curly brackets compiler ko batate hain ke function body kahan se start aur kahan khatam hoti hai.

Hum apne define kiye hue kisi bhi function ko uska name enter karke aur uske baad parentheses ka ek set likh kar call kar sakte hain. Kyun ke `another_function` program mein defined hai, is liye ise `main` function ke andar se call kiya ja sakta hai. Note karein ke hum ne source code mein `another_function` ko `main` function ke *baad* define kiya hai; hum ise pehle bhi define kar sakte the. Rust ko is baat se koi farq nahi padta ke aap apne functions kahan define karte hain, bas woh kisi aise scope mein kahin defined hone chahiye jo caller ko nazar aa sakta ho.

Aaiye functions ko mazeed explore karne ke liye *functions* naam ka ek naya binary project shuru karte hain. *src/main.rs* mein `another_function` example rakhein aur use run karein. Aapko following output nazar aana chahiye:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-16-functions/output.txt}}
```

Lines usi order mein execute hoti hain jis order mein woh `main` function mein appear hoti hain. Sab se pehle “Hello, world!” message print hota hai, aur phir `another_function` call hota hai aur uska message print hota hai.

### Parameters

Hum functions ko *parameters* rakhne ke liye define kar sakte hain, jo special variables hote hain aur function ke signature ka hissa hote hain. Jab kisi function mein parameters hote hain, to aap un parameters ke liye concrete values provide kar sakte hain. Technically, in concrete values ko *arguments* kaha jata hai, lekin casual conversation mein log aam tor par *parameter* aur *argument* dono words ko ek doosre ki jagah use karte hain, chahe baat function ki definition mein maujood variables ki ho ya function ko call karte waqt pass ki jane wali concrete values ki.

`another_function` ke is version mein hum ek parameter add karte hain:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/src/main.rs}}
```

Is program ko run karke dekhein; aapko following output milna chahiye:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/output.txt}}
```

`another_function` ki declaration mein `x` naam ka ek parameter hai. `x` ki type `i32` specify ki gayi hai. Jab hum `another_function` mein `5` pass karte hain, to `println!` macro format string mein `x` wale curly brackets ke pair ki jagah `5` rakh deta hai.

Function signatures mein aapko har parameter ki type *lazmi* declare karni hoti hai. Ye Rust ke design mein ek deliberate decision hai: Function definitions mein type annotations require karne ka matlab hai ke compiler ko code mein lagbhag kabhi bhi aapki taraf se doosri jagah type annotations ki zaroorat nahi padti taa-ke woh samajh sake ke aap kis type ki baat kar rahe hain. Agar compiler ko pata ho ke function kin types ki expectation karta hai, to woh zyada helpful error messages bhi de sakta hai.

Multiple parameters define karte waqt, parameter declarations ko commas se separate karein, is tarah:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/src/main.rs}}
```

Ye example `print_labeled_measurement` naam ka ek function create karta hai jisme do parameters hain. Pehle parameter ka naam `value` hai aur ye `i32` hai. Doosre ka naam `unit_label` hai aur iski type `char` hai. Phir function `value` aur `unit_label` dono ko contain karne wala text print karta hai.

Aaiye is code ko run karke dekhte hain. Apne *functions* project ki *src/main.rs* file mein mojood current program ko preceding example se replace karein aur ise `cargo run` ke zariye run karein:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/output.txt}}
```

Kyun ke hum ne function ko `value` ke liye value `5` aur `unit_label` ke liye value `'h'` ke saath call kiya tha, is liye program ke output mein ye values shamil hain.

### Statements and Expressions

Function bodies statements ki ek series par mushtamil hoti hain jo optionally ek expression par khatam ho sakti hain. Ab tak jin functions ko hum ne cover kiya hai, un mein ending expression shamil nahi thi, lekin aap ne ek statement ke hissa ke taur par expression zaroor dekha hai. Kyun ke Rust ek expression-based language hai, is liye is distinction ko samajhna important hai. Doosri languages mein ye distinction isi tarah nahi hoti, is liye aaiye dekhein ke statements aur expressions kya hote hain aur in ke differences function bodies par kaise asar dalte hain.

* *Statements* woh instructions hain jo koi action perform karti hain aur koi value return nahi karti.
* *Expressions* evaluate hokar ek resultant value deti hain.

Aaiye kuch examples dekhte hain.

Hum asal mein statements aur expressions ko pehle hi use kar chuke hain. `let` keyword ke zariye variable create karna aur use ek value assign karna ek statement hai. Listing 3-1 mein, `let y = 6;` ek statement hai.

<Listing number="3-1" file-name="src/main.rs" caption="Ek `main` function declaration jisme ek statement hai">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-01/src/main.rs}}
```

</Listing>

Function definitions bhi statements hoti hain; poora preceding example khud ek statement hai. (Jaisa ke hum thori dair mein dekhenge, function ko call karna statement nahi hota.)

Statements values return nahi karti. Is liye aap `let` statement ko kisi doosre variable ko assign nahi kar sakte, jaisa ke following code karne ki koshish karta hai; aapko ek error milega:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/src/main.rs}}
```

Jab aap ye program run karenge, to aapko jo error milega woh kuch is tarah nazar aayega:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/output.txt}}
```

`let y = 6` statement koi value return nahi karti, is liye `x` ke bind hone ke liye kuch bhi nahi hai. Ye doosri languages, jaise C aur Ruby, mein hone wale behavior se different hai, jahan assignment assignment ki value return karta hai. Un languages mein aap `x = y = 6` likh sakte hain aur `x` aur `y` dono ki value `6` ho sakti hai; Rust mein aisa nahi hai.

Expressions ek value mein evaluate hoti hain aur Rust mein aapke likhe jane wale baqi code ka zyada hissa banati hain. Kisi mathematical operation par gaur karein, jaise `5 + 6`, jo ek expression hai aur value `11` mein evaluate hota hai. Expressions statements ka hissa ho sakti hain: Listing 3-1 mein, statement `let y = 6;` ke andar `6` ek expression hai jo value `6` mein evaluate hota hai. Function ko call karna ek expression hai. Macro ko call karna ek expression hai. Curly brackets ke zariye create kiya gaya naya scope block bhi ek expression hai, misal ke taur par:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-20-blocks-are-expressions/src/main.rs}}
```

Ye expression:

```rust,ignore
{
    let x = 3;
    x + 1
}
```

ek block hai jo is case mein `4` mein evaluate hota hai. Ye value `let` statement ke hissa ke taur par `y` ke saath bind ho jati hai. `x + 1` wali line ke end par semicolon nahi hai, jo un zyada tar lines se different hai jinhein aap ne ab tak dekha hai. Expressions mein ending semicolons shamil nahi hote. Agar aap expression ke end par semicolon add kar dein, to aap use ek statement mein convert kar dete hain, aur phir woh koi value return nahi karega. Is baat ko zehan mein rakhein jab aap agay function return values aur expressions ko explore karein.

### Functions with Return Values

Functions us code ko values return kar sakte hain jo unhein call karta hai. Hum return values ko name nahi dete, lekin arrow (`->`) ke baad unki type declare karna zaroori hai. Rust mein function ki return value, function body ke block mein mojood final expression ki value ke synonymous hoti hai. Aap `return` keyword use karke aur ek value specify karke function se early return kar sakte hain, lekin zyada tar functions last expression ko implicitly return karte hain. Yahan ek example hai ek aise function ka jo ek value return karta hai:

<span class="filename">Filename: src/main.rs</span>

```rust id="m8zz7n"
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/src/main.rs}}
```

`five` function mein koi function calls, macros, ya hatta ke `let` statements bhi nahi hain—sirf number `5` apne aap mein hai. Rust mein ye bilkul valid function hai. Note karein ke function ki return type bhi specify ki gayi hai, `-> i32` ke taur par. Is code ko run karke dekhein; output kuch is tarah hona chahiye:

```console id="xj2l9a"
{{#include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/output.txt}}
```

`five` mein `5` function ki return value hai, isi liye return type `i32` hai. Aaiye isay mazeed detail mein dekhte hain. Do important points hain:

Pehli baat, line `let x = five();` dikhati hai ke hum function ki return value ko ek variable ko initialize karne ke liye use kar rahe hain. Kyun ke `five` function `5` return karta hai, ye line following ke barabar hai:

```rust
let x = 5;
```

Doosri baat, `five` function mein koi parameters nahi hain aur ye return value ki type define karta hai, lekin function ki body mein semicolon ke baghair sirf ek `5` hai kyun ke ye ek expression hai jis ki value hum return karna chahte hain.

Aaiye ek aur example dekhte hain:

<span class="filename">Filename: src/main.rs</span>

```rust id="7x3v1q"
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-22-function-parameter-and-return/src/main.rs}}
```

Is code ko run karne se `The value of x is: 6` print hoga. Lekin agar hum `x + 1` wali line ke end par semicolon laga dein, aur use expression se statement mein change kar dein, to kya hoga?

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile id="kq4z8p"
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/src/main.rs}}
```

Is code ko compile karne se following error produce hoga:

```console id="v9r4hy"
{{#include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/output.txt}}
```

Main error message, `mismatched types`, is code ke core issue ko reveal karta hai. `plus_one` function ki definition kehti hai ke ye ek `i32` return karega, lekin statements kisi value mein evaluate nahi hoti, jise unit type `()` express karta hai. Is liye kuch bhi return nahi hota, jo function definition ke mutabiq nahi hai aur error ka sabab banta hai. Is output mein Rust ek aisa message bhi provide karta hai jo is issue ko rectify karne mein madad kar sakta hai: Ye semicolon remove karne ka suggest karta hai, jo error ko fix kar dega.

