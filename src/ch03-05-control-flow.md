## Control Flow

Kisi condition ke `true` hone ki surat mein kuch code run karna aur condition ke `true` rehne tak kisi code ko baar baar run karne ki ability, zyada tar programming languages mein basic building blocks hain. Rust code mein execution ke flow ko control karne wale sab se common constructs `if` expressions aur loops hain.

### `if` Expressions

Ek `if` expression aapko conditions ke mutabiq apne code ko branch karne deta hai. Aap ek condition provide karte hain aur phir kehte hain, “Agar ye condition meet hoti hai, to is code block ko run karo. Agar condition meet nahi hoti, to is code block ko run mat karo.”

Apni *projects* directory mein *branches* naam ka ek naya project create karein taa-ke `if` expression ko explore kiya ja sake. *src/main.rs* file mein following enter karein:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/src/main.rs}}
```

Tamam `if` expressions `if` keyword se start hoti hain, jiske baad ek condition hoti hai. Is case mein condition check karti hai ke variable `number` ki value 5 se kam hai ya nahi. Agar condition `true` ho to execute hone wale code block ko hum condition ke foran baad curly brackets ke andar rakhte hain. `if` expressions mein conditions ke saath associated code blocks ko kabhi kabhi *arms* kaha jata hai, bilkul un arms ki tarah jo hum ne Chapter 2 ke [“Comparing the Guess to the Secret Number”][comparing-the-guess-to-the-secret-number]<!--
ignore --> section mein `match` expressions ke saath discuss kiye the.

Optionally, hum ek `else` expression bhi include kar sakte hain, jo hum ne yahan kiya hai, taa-ke agar condition `false` evaluate ho to program ke paas execute karne ke liye ek alternative code block ho. Agar aap `else` expression provide nahi karte aur condition `false` ho, to program sirf `if` block ko skip karega aur code ke aglay hisse ki taraf chala jayega.

Is code ko run karke dekhein; aapko following output nazar aana chahiye:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/output.txt}}
```

Aaiye `number` ki value ko aisi value mein change karke dekhte hain jo condition ko `false` bana de, taa-ke pata chale kya hota hai:

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/src/main.rs:here}}
```

Program ko dobara run karein aur output dekhein:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/output.txt}}
```

Ye bhi note karna zaroori hai ke is code mein condition ka `bool` hona *lazmi* hai. Agar condition `bool` na ho, to humein error milega. Misal ke taur par, following code ko run karke dekhein:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/src/main.rs}}
```

Is baar `if` condition ek value `3` mein evaluate hoti hai, aur Rust error throw karta hai:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/output.txt}}
```

Error indicate karta hai ke Rust ko `bool` ki expectation thi lekin use integer mila. Ruby aur JavaScript jaisi languages ke unlike, Rust automatically non-Boolean types ko Boolean mein convert karne ki koshish nahi karta. Aapko explicit hona hota hai aur `if` ko hamesha condition ke taur par ek Boolean provide karna hota hai. Agar hum chahte hain ke, misal ke taur par, `if` code block sirf us waqt run ho jab number `0` ke barabar na ho, to hum `if` expression ko following mein change kar sakte hain:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-29-if-not-equal-0/src/main.rs}}
```

Is code ko run karne par `number was something other than zero` print hoga.

#### Handling Multiple Conditions with `else if`

Aap `else if` expression mein `if` aur `else` ko combine karke multiple conditions use kar sakte hain. Misal ke taur par:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/src/main.rs}}
```

Is program ke paas chaar possible paths hain jin mein se ye kisi ek par ja sakta hai. Ise run karne ke baad, aapko following output nazar aana chahiye:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/output.txt}}
```

Jab ye program execute hota hai, to ye har `if` expression ko bari bari check karta hai aur pehli aisi body execute karta hai jis ki condition `true` evaluate hoti hai. Note karein ke halaanke 6, 2 se divisible hai, humein `number is divisible by 2` ka output nazar nahi aata, aur na hi `else` block ka `number is not divisible by 4, 3, or 2` text nazar aata hai. Is ki wajah ye hai ke Rust sirf pehli `true` condition ke block ko execute karta hai, aur ek `true` condition milne ke baad baqi conditions ko check bhi nahi karta.

Bohat zyada `else if` expressions use karna aapke code ko clutter kar sakta hai, is liye agar aapke paas ek se zyada hon, to aap apne code ko refactor karna chah sakte hain. Chapter 6 in situations ke liye Rust ke ek powerful branching construct `match` ko describe karta hai.

#### Using `if` in a `let` Statement

Kyun ke `if` ek expression hai, hum ise `let` statement ke right side par use karke uske result ko ek variable ko assign kar sakte hain, jaisa ke Listing 3-2 mein hai.

<Listing number="3-2" file-name="src/main.rs" caption="Ek `if` expression ke result ko ek variable ko assign karna">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-02/src/main.rs}}
```

</Listing>

`number` variable ko `if` expression ke result ki bunyaad par ek value ke saath bind kiya jayega. Is code ko run karke dekhein ke kya hota hai:

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-02/output.txt}}
```

Yaad rakhein ke code ke blocks unke andar mojood last expression mein evaluate hote hain, aur numbers apne aap mein bhi expressions hote hain. Is case mein, poore `if` expression ki value is baat par depend karti hai ke kaunsa code block execute hota hai. Is ka matlab hai ke `if` ki har arm se result banne ki potential values ka same type ka hona zaroori hai; Listing 3-2 mein, `if` arm aur `else` arm dono ke results `i32` integers the. Agar types mismatch hon, jaisa ke following example mein hai, to humein ek error milega:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/src/main.rs}}
```

Jab hum is code ko compile karne ki koshish karenge, to humein ek error milega. `if` aur `else` arms ki value types compatible nahi hain, aur Rust exactly indicate karta hai ke program mein problem kahan hai:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/output.txt}}
```

`if` block mein expression ek integer mein evaluate hota hai, aur `else` block mein expression ek string mein evaluate hota hai. Ye kaam nahi karega, kyun ke variables ki ek hi type honi chahiye, aur Rust ko compile time par definitively ye maloom hona zaroori hai ke `number` variable ki type kya hai. `number` ki type maloom hone se compiler verify kar sakta hai ke jab bhi hum `number` ko use karein, uski type valid hai. Agar `number` ki type sirf runtime par determine hoti, to Rust aisa nahi kar pata; compiler zyada complex hota aur code ke baare mein kam guarantees deta agar use har variable ke liye multiple hypothetical types ko track karna padta.

### Repetition with Loops

Aksar kisi code block ko ek se zyada baar execute karna useful hota hai. Is kaam ke liye, Rust kai *loops* provide karta hai, jo loop body ke andar ke code ko end tak execute karte hain aur phir foran dobara shuru se start ho jate hain. Loops ke saath experiment karne ke liye, aaiye *loops* naam ka ek naya project banate hain.

Rust mein teen qisam ke loops hain: `loop`, `while`, aur `for`. Aaiye har ek ko try karte hain.

#### Repeating Code with `loop`

`loop` keyword Rust ko batata hai ke code ke ek block ko baar baar execute karna hai, ya to hamesha ke liye ya jab tak aap explicitly use stop karne ke liye na kahen.

Misal ke taur par, apni *loops* directory mein *src/main.rs* file ko is tarah change karein:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-loop/src/main.rs}}
```

Jab hum is program ko run karenge, to `again!` baar baar continuously print hota rahega jab tak hum program ko manually stop nahi kar dete. Zyada tar terminals keyboard shortcut <kbd>ctrl</kbd>-<kbd>C</kbd> support karte hain, jo continual loop mein phanse hue program ko interrupt karne ke liye use hota hai. Ise try karein:

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-32-loop
cargo run
CTRL-C
-->

```console
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.08s
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

Symbol `^C` us jagah ko represent karta hai jahan aap ne <kbd>ctrl</kbd>-<kbd>C</kbd> press kiya.

Aapko `^C` ke baad `again!` print hota hua nazar aa sakta hai ya nahi bhi, ye is baat par depend karta hai ke interrupt signal receive hone ke waqt code loop mein kis point par tha.

Khush qismati se, Rust code ke zariye loop se bahar nikalne ka bhi ek tareeqa provide karta hai. Aap loop ke andar `break` keyword place kar sakte hain taa-ke program ko bataya ja sake ke loop ko kab stop karna hai. Yaad karein ke hum ne Chapter 2 ke [“Quitting After a Correct Guess”][quitting-after-a-correct-guess]<!-- ignore
--> section mein guessing game ke andar ye kiya tha taa-ke user ke correct number guess karke game jeetne par program exit ho jaye.

Hum ne guessing game mein `continue` bhi use kiya tha, jo loop mein program ko batata hai ke loop ki current iteration mein baqi code ko skip karke next iteration par chale jaye.

#### Returning Values from Loops

`loop` ka ek use aisi operation ko dobara try karna hai jis ke fail hone ka aapko andesha ho, jaise ye check karna ke koi thread apna kaam complete kar chuka hai ya nahi. Aapko us operation ka result loop se bahar apne code ke baqi hisson tak pass karne ki bhi zaroorat ho sakti hai. Ye karne ke liye, `break` expression ke baad woh value add kar sakte hain jo aap return karna chahte hain, jab aap loop ko stop karne ke liye `break` use karte hain; woh value loop se bahar return ho jayegi taa-ke aap use kar saken, jaisa ke yahan dikhaya gaya hai:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-33-return-value-from-loop/src/main.rs}}
```

Loop se pehle, hum `counter` naam ka ek variable declare karte hain aur use `0` se initialize karte hain. Phir, hum `result` naam ka ek variable declare karte hain jo loop se return hone wali value ko hold karega. Loop ki har iteration mein, hum `counter` variable mein `1` add karte hain, aur phir check karte hain ke kya `counter` `10` ke barabar hai. Jab ye `10` ho jata hai, to hum `break` keyword ko `counter * 2` ki value ke saath use karte hain. Loop ke baad, hum `result` ko value assign karne wali statement ko end karne ke liye semicolon use karte hain. Aakhir mein, hum `result` ki value print karte hain, jo is case mein `20` hai.

Aap loop ke andar se `return` bhi kar sakte hain. Jabke `break` sirf current loop se bahar nikalta hai, `return` hamesha current function se bahar nikalta hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="loop-labels-to-disambiguate-between-multiple-loops"></a>

#### Disambiguating with Loop Labels

Agar aapke paas loops ke andar loops hon, to us waqt `break` aur `continue` sab se andar wale loop par apply hote hain. Aap optionally kisi loop par ek *loop label* specify kar sakte hain, jise aap baad mein `break` ya `continue` ke saath use karke specify kar sakte hain ke ye keywords innermost loop ke bajaye labeled loop par apply hon. Loop labels ka aaghaz single quote se hona zaroori hai. Yahan do nested loops ki ek example hai:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/src/main.rs}}
```

Outer loop ka label `'counting_up` hai, aur ye 0 se 2 tak count karega. Baghair label wala inner loop 10 se 9 tak countdown karta hai. Pehla `break` jo koi label specify nahi karta, sirf inner loop se bahar niklega. `break 'counting_up;` statement outer loop se bahar niklegi. Ye code following output print karta hai:

```console
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/output.txt}}
```

<!-- Old headings. Do not remove or links may break. -->

<a id="conditional-loops-with-while"></a>

#### Streamlining Conditional Loops with while

Aksar kisi program ko loop ke andar ek condition evaluate karne ki zaroorat hoti hai. Jab tak condition `true` hoti hai, loop run karta hai. Jab condition `true` rehna band kar deti hai, program `break` call karta hai, jo loop ko rok deta hai. `loop`, `if`, `else`, aur `break` ko combine karke is tarah ka behavior implement karna mumkin hai; agar aap chahein to abhi kisi program mein ise try kar sakte hain. Lekin ye pattern itna common hai ke Rust mein is ke liye ek built-in language construct mojood hai, jise `while` loop kaha jata hai. Listing 3-3 mein, hum `while` ko use karke program ko teen baar loop karte hain, har baar countdown karte hue, aur phir loop ke baad ek message print karke exit karte hain.

<Listing number="3-3" file-name="src/main.rs" caption="Jab tak condition `true` evaluate ho, code run karne ke liye `while` loop use karna">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-03/src/main.rs}}
```

</Listing>

Ye construct us nesting ka bohat sa hissa khatam kar deta hai jo `loop`, `if`, `else`, aur `break` use karne ki surat mein zaroori hoti, aur ye zyada clear bhi hai. Jab tak koi condition `true` evaluate hoti hai, code run karta hai; warna, ye loop se exit kar jata hai.

#### Looping Through a Collection with `for`

Aap kisi collection, jaise array, ke elements par loop karne ke liye `while` construct use karna choose kar sakte hain. Misal ke taur par, Listing 3-4 mein loop array `a` ke har element ko print karta hai.

<Listing number="3-4" file-name="src/main.rs" caption="`while` loop ko use karke collection ke har element par loop karna">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-04/src/main.rs}}
```

</Listing>

Yahan code array ke elements ke through count up karta hai. Ye index `0` se start hota hai aur tab tak loop karta hai jab tak array ke final index tak nahi pohanch jata (yani jab `index < 5` mazeed `true` nahi rehta). Is code ko run karne se array ka har element print hoga:

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-04/output.txt}}
```

Terminal mein tamam paanch array values nazar aati hain, jaisa ke expected tha. Halanke kisi point par `index` ki value `5` tak pohanch jayegi, loop array se sixth value fetch karne ki koshish karne se pehle hi execute hona band kar deta hai.

Lekin, ye approach error-prone hai; agar index ki value ya test condition ghalat ho, to hum program ko panic karwa sakte hain. Misal ke taur par, agar aap `a` array ki definition ko change karke is mein chaar elements kar dein lekin condition ko `while index < 4` par update karna bhool jayein, to code panic karega. Ye slow bhi hai, kyun ke compiler loop ki har iteration mein ye conditional check perform karne ke liye runtime code add karta hai ke index array ki bounds ke andar hai ya nahi.

Is ke muqable mein zyada concise alternative ke taur par, aap `for` loop use kar sakte hain aur collection ke har item ke liye kuch code execute kar sakte hain. `for` loop Listing 3-5 ke code jaisa nazar aata hai.

<Listing number="3-5" file-name="src/main.rs" caption="`for` loop ko use karke collection ke har element par loop karna">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-05/src/main.rs}}
```

</Listing>

Jab hum is code ko run karenge, to humein Listing 3-4 jaisa hi output nazar aayega. Is se bhi zyada important baat ye hai ke ab hum ne code ki safety barha di hai aur un bugs ke chances ko khatam kar diya hai jo array ke end se aage jane ya itna aage na jane ke sabab ho sakte hain ke kuch items miss ho jayein. `for` loops se generate hone wala machine code zyada efficient bhi ho sakta hai, kyun ke har iteration mein index ko array ki length ke saath compare karne ki zaroorat nahi hoti.

`for` loop use karte hue, agar aap array mein values ki number change kar dein, to aapko kisi doosre code ko change karna yaad rakhne ki zaroorat nahi hogi, jaisa ke Listing 3-4 mein use kiye gaye method ke saath hota.

`for` loops ki safety aur conciseness unhein Rust mein sab se zyada commonly used loop construct banati hai. Hatta ke un situations mein bhi jahan aap kisi code ko ek khaas number of times run karna chahte hain, jaise Listing 3-3 mein `while` loop use karne wale countdown example mein, zyada tar Rustaceans `for` loop use karenge. Is ke liye tareeqa ye hai ke standard library ki taraf se provide ki gayi `Range` ko use kiya jaye, jo ek number se start karke doosre number se pehle tak sequence mein tamam numbers generate karti hai.

Yahan `for` loop aur ek aur method, `rev`, ko use karte hue countdown is tarah nazar aayega, jis ke baare mein hum ne abhi tak baat nahi ki; `rev` range ko reverse karne ke liye use hota hai:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-34-for-range/src/main.rs}}
```

Ye code thora zyada behtar lagta hai, hai na?


## Summary

Aap ne kar liya! Ye ek kaafi bara chapter tha: Aap ne variables, scalar aur compound data types, functions, comments, `if` expressions, aur loops ke baare mein seekha! Is chapter mein discuss kiye gaye concepts ki practice karne ke liye, aise programs banane ki koshish karein jo following kaam karein:

* Fahrenheit aur Celsius ke darmiyan temperatures convert karein.
* *n*th Fibonacci number generate karein.
* Christmas carol “The Twelve Days of Christmas” ke lyrics print karein, aur song mein hone wali repetition ka faida uthayein.

Jab aap agay barhne ke liye ready hon, to hum Rust ke ek aise concept ke baare mein baat karenge jo aam tor par doosri programming languages mein mojood nahi hota: ownership.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[quitting-after-a-correct-guess]: ch02-00-guessing-game-tutorial.html#quitting-after-a-correct-guess

