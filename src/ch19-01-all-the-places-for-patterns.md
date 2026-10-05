## All the Places Patterns Can Be Used

Patterns Rust mein kai jagahon par nazar aati hain, aur aap inhein bohat baar use kar chuke hain bina yeh realize kiye! Yeh section un tamam jagahon par baat karta hai jahan patterns valid hoti hain.

### `match` Arms

Jaisa ke Chapter 6 mein discuss kiya gaya hai, hum `match` expressions ke arms mein patterns use karte hain. Formally, `match` expressions ko keyword `match`, ek value jiske against match karna ho, aur ek ya zyada match arms se define kiya jata hai. Har match arm mein ek pattern aur ek expression hota hai jo us waqt run hota hai jab value us arm ke pattern se match kare, jaise yahan:

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre><code>match <em>VALUE</em> {
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
}</code></pre>

Misal ke taur par, yahan Listing 6-5 ki `match` expression hai jo variable `x` mein maujood `Option<i32>` value ke against match karti hai:

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

Is `match` expression mein patterns har arrow ke left side par maujood `None` aur `Some(i)` hain.

`match` expressions ke liye ek requirement yeh hai ke woh exhaustive honi chahiye, yani `match` expression mein maujood value ki tamam possibilities ka hisaab hona chahiye. Is baat ko ensure karne ka ek tareeqa yeh hai ke last arm ke liye ek catch-all pattern rakha jaye: Misal ke taur par, kisi bhi value se match karne wala variable name kabhi fail nahi ho sakta aur is tarah baqi tamam cases ko cover kar leta hai.

Khaas pattern `_` kisi bhi cheez se match karega, lekin yeh kabhi kisi variable se bind nahi hota, is liye ise aksar last match arm mein use kiya jata hai. `_` pattern us waqt useful ho sakta hai jab aap kisi aisi value ko ignore karna chahte hon jo specifically specify nahi ki gayi, misal ke taur par. Hum is chapter mein baad mein [“Ignoring Values in a
Pattern”][ignoring-values-in-a-pattern]<!-- ignore --> mein `_` pattern ko zyada detail mein cover karenge.
### `let` Statements

Is chapter se pehle, humne sirf `match` aur `if let` ke saath patterns ko explicitly discuss kiya tha, lekin asal mein humne patterns ko doosri jagahon par bhi use kiya hai, jin mein `let` statements bhi shamil hain. Misal ke taur par, `let` ke saath is simple variable assignment ko dekhein:

```rust
let x = 5;
```

Jab bhi aapne is tarah `let` statement use kiya hai, aap patterns use kar rahe the, halaanke shayad aapko iska ehsaas nahi tha! Zyada formally, ek `let` statement is tarah hota hai:

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre>
<code>let <em>PATTERN</em> = <em>EXPRESSION</em>;</code>
</pre>

`let x = 5;` jaisi statements mein, jahan PATTERN slot mein variable name hota hai, variable name asal mein pattern ki ek bohat simple form hoti hai. Rust expression ko pattern ke against compare karta hai aur jo bhi names us mein milte hain unhein assign karta hai. Is liye `let x = 5;` example mein, `x` ek pattern hai jiska matlab hai “jo yahan match kare usay variable `x` ke saath bind karo.” Kyun ke `x` naam poora pattern hai, is liye yeh pattern effectively kehta hai “value jo bhi ho, har cheez ko variable `x` ke saath bind karo.”

`let` ke pattern-matching aspect ko zyada clearly dekhne ke liye, Listing 19-1 ko dekhein, jo `let` ke saath ek pattern use karke tuple ko destructure karta hai.

<Listing number="19-1" caption="Using a pattern to destructure a tuple and create three variables at once">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-01/src/main.rs:here}}
```

</Listing>

Yahan hum ek tuple ko ek pattern ke against match karte hain. Rust value `(1, 2, 3)` ko pattern `(x, y, z)` ke saath compare karta hai aur dekhta hai ke value pattern se match karti hai—yani, yeh dekhta hai ke dono mein elements ki tadaad same hai—is liye Rust `1` ko `x` ke saath, `2` ko `y` ke saath, aur `3` ko `z` ke saath bind karta hai. Aap is tuple pattern ko is tarah soch sakte hain ke is ke andar teen individual variable patterns nested hain.

Agar pattern mein elements ki tadaad tuple mein elements ki tadaad se match na kare, to overall type match nahi karega aur humein compiler error milega. Misal ke taur par, Listing 19-2 mein teen elements wale tuple ko do variables mein destructure karne ki koshish dikhayi gayi hai, jo kaam nahi karegi.

<Listing number="19-2" caption="Incorrectly constructing a pattern whose variables don’t match the number of elements in the tuple">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-02/src/main.rs:here}}
```

</Listing>

Is code ko compile karne ki koshish ka result yeh type error hai:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-02/output.txt}}
```

Error ko fix karne ke liye, hum tuple mein ek ya zyada values ko `_` ya `..` use karke ignore kar sakte hain, jaisa ke hum [“Ignoring Values in a
Pattern”][ignoring-values-in-a-pattern]<!-- ignore --> section mein dekhenge. Agar problem yeh hai ke pattern mein variables bohat zyada hain, to solution yeh hai ke variables remove karke types ko match kiya jaye, taake variables ki tadaad tuple mein elements ki tadaad ke barabar ho.

### Conditional `if let` Expressions

Chapter 6 mein humne discuss kiya tha ke `if let` expressions ko mainly us waqt use kiya jata hai jab hum equivalent `match` ko shorter way mein likhna chahte hain jo sirf ek case ko match karta hai. Optionally, `if let` ke saath ek corresponding `else` bhi ho sakta hai jisme woh code run hota hai jab `if let` ka pattern match nahi karta.

Listing 19-3 dikhati hai ke `if let`, `else if`, aur `else if let` expressions ko mix and match karna bhi possible hai. Is tarah humein `match` expression se zyada flexibility milti hai, jisme hum patterns ke saath compare karne ke liye sirf ek value express kar sakte hain. Saath hi, Rust yeh require nahi karta ke `if let`, `else if`, aur `else if let` arms ki ek series mein conditions ek doosre se related hon.

Listing 19-3 mein code kai conditions ke liye ek ke baad ek checks ke basis par determine karta hai ke aapke background ko kis color ka banana hai. Is example ke liye, humne hardcoded values ke saath variables create kiye hain jo kisi real program mein user input se receive ki ja sakti hain.

<Listing number="19-3" file-name="src/main.rs" caption="Mixing `if let`, `else if`, `else if let`, and `else`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-03/src/main.rs}}
```

</Listing>

Agar user apna favorite color specify karta hai, to woh color background ke liye use hota hai. Agar koi favorite color specify nahi kiya gaya aur aaj Tuesday hai, to background color green hota hai. Warna, agar user apni age ko string ke taur par specify karta hai aur hum usay successfully number ke taur par parse kar sakte hain, to number ki value ke mutabiq color purple ya orange hota hai. Agar in mein se koi bhi condition apply na ho, to background color blue hota hai.

Yeh conditional structure humein complex requirements ko support karne deta hai. Yahan jo hardcoded values hamare paas hain, unke saath yeh example `Using purple as the background color` print karega.

Aap dekh sakte hain ke `if let` naye variables bhi introduce kar sakta hai jo existing variables ko shadow karte hain, bilkul usi tarah jaise `match` arms kar sakti hain: line `if let Ok(age) = age` ek naya `age` variable introduce karti hai jisme `Ok` variant ke andar wali value hoti hai, aur yeh existing `age` variable ko shadow karta hai. Is ka matlab hai ke humein `if age > 30` condition ko us block ke andar rakhna zaroori hai: Hum in dono conditions ko `if let Ok(age) = age && age > 30` mein combine nahi kar sakte. Jis naye `age` ko hum 30 ke saath compare karna chahte hain, woh tab tak valid nahi hota jab tak curly bracket ke saath naya scope start na ho.

`if let` expressions use karne ka downside yeh hai ke compiler exhaustiveness check nahi karta, jabke `match` expressions ke saath karta hai. Agar hum last `else` block ko omit kar dein aur is tarah kuch cases ko handle na kar paayein, to compiler humein possible logic bug ke baare mein alert nahi karega.

### `while let` Conditional Loops

`if let` ki construction se milti-julti, `while let` conditional loop ek `while` loop ko us waqt tak run hone deta hai jab tak koi pattern match hota rahe. Listing 19-4 mein hum ek `while let` loop dikhate hain jo threads ke darmiyan bheje gaye messages ka wait karta hai, lekin is case mein `Option` ke bajaye `Result` ko check karta hai.

<Listing number="19-4" caption="Using a `while let` loop to print values for as long as `rx.recv()` returns `Ok`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-04/src/main.rs:here}}
```

</Listing>

Yeh example `1`, `2`, aur phir `3` print karta hai. `recv` method receiver side of the channel se pehla message leti hai aur ek `Ok(value)` return karti hai. Jab humne Chapter 16 mein pehli baar `recv` dekha tha, to humne error ko directly unwrap kiya tha, ya `for` loop ko iterator ke taur par use karke is ke saath interact kiya tha. Lekin, jaisa ke Listing 19-4 dikhati hai, hum `while let` bhi use kar sakte hain, kyun ke jab tak sender exist karta hai, har baar koi message arrive hone par `recv` method ek `Ok` return karti hai, aur phir sender side disconnect hone ke baad ek `Err` produce karti hai.

### `for` Loops

Ek `for` loop mein, keyword `for` ke foran baad aane wali value ek pattern hoti hai. Misal ke taur par, `for x in y` mein `x` pattern hai. Listing 19-5 dikhati hai ke `for` loop mein pattern ko use karke tuple ko destructure, yani usay parts mein break apart, kaise kiya ja sakta hai.

<Listing number="19-5" caption="Using a pattern in a `for` loop to destructure a tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-05/src/main.rs:here}}
```

</Listing>

Listing 19-5 ka code yeh output print karega:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-05/output.txt}}
```

Hum `enumerate` method ko use karke ek iterator ko adapt karte hain taake woh ek value aur us value ka index produce kare, jo ek tuple mein rakhe jate hain. Produce hone wali pehli value tuple `(0, 'a')` hai. Jab is value ko pattern `(index, value)` ke saath match kiya jata hai, to `index` `0` aur `value` `'a'` hoga, aur output ki pehli line print hogi.

### Function Parameters

Function parameters bhi patterns ho sakte hain. Listing 19-6 ka code, jo `foo` naam ka ek function declare karta hai aur jo `i32` type ka ek parameter `x` leta hai, ab tak aapko familiar lagna chahiye.

<Listing number="19-6" caption="A function signature using patterns in the parameters">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-06/src/main.rs:here}}
```

</Listing>

`x` wala part ek pattern hai! Jaisa humne `let` ke saath kiya tha, hum function ke arguments mein ek tuple ko pattern ke saath match kar sakte hain. Listing 19-7 tuple ki values ko split karti hai jab hum usay function ko pass karte hain.

<Listing number="19-7" file-name="src/main.rs" caption="A function with parameters that destructure a tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-07/src/main.rs}}
```

</Listing>

Yeh code `Current location: (3, 5)` print karta hai. Values `&(3, 5)` pattern `&(x, y)` se match karti hain, is liye `x` ki value `3` aur `y` ki value `5` hai.

Hum closure parameter lists mein bhi isi tarah patterns use kar sakte hain jis tarah function parameter lists mein karte hain, kyun ke closures functions ke similar hoti hain, jaisa ke Chapter 13 mein discuss kiya gaya hai.

Is point par, aapne patterns ko use karne ke kai tareeqe dekhe hain, lekin har jagah jahan hum patterns use kar sakte hain wahan patterns ek jaisa kaam nahi karti. Kuch jagahon par patterns ka irrefutable hona zaroori hota hai; doosri circumstances mein woh refutable ho sakti hain. Ab hum in dono concepts ko discuss karenge.

[ignoring-values-in-a-pattern]: ch19-03-pattern-syntax.html#ignoring-values-in-a-pattern
