<!-- Old headings. Do not remove or links may break. -->

<a id="streams"></a>

## Streams: Futures in Sequence

Yaad karein ke humne is chapter mein pehle apne async channel ke receiver ko
[“Message Passing”][17-02-messages]<!-- ignore --> section mein kaise use kiya tha. Async
`recv` method waqt ke saath items ki ek sequence produce karta hai. Yeh ek
bohot zyada general pattern ki misaal hai jise *stream* kaha jata hai. Bohot se concepts ko
naturally streams ke taur par represent kiya ja sakta hai: queue mein items ka available hona,
filesystem se data ke chunks ka incrementally pull hona jab poora data set computer ki memory ke
liye bohot bara ho, ya waqt ke saath network par data ka arrive hona.
Kyun ke streams futures hoti hain, hum unhein kisi bhi doosri qisam ke future ke saath use kar sakte hain aur
unhein interesting tareeqon se combine kar sakte hain. Misal ke taur par, hum events ko batch up kar sakte hain
taake bohot zyada network calls trigger na hon, long-running operations ki sequences par timeouts set kar sakte hain,
ya user interface events ko throttle kar sakte hain taake unnecessary work na karna pade.

Humne Chapter 13 mein items ki ek sequence dekhi thi, jab humne
[“The Iterator Trait and the `next` Method”][iterator-trait]<!--
ignore --> section mein `Iterator` trait ko dekha tha, lekin iterators aur
async channel receiver ke darmiyan do differences hain. Pehla difference time ka hai: iterators
synchronous hotay hain, jab ke channel receiver asynchronous hota hai. Doosra difference API ka hai.
`Iterator` ke saath directly kaam karte hue, hum us ke synchronous `next` method ko call karte hain.
Khaas taur par `trpl::Receiver` stream ke saath, humne is ke bajaye asynchronous `recv` method ko
call kiya tha. Is ke ilawa, yeh APIs bohot similar feel hoti hain, aur yeh similarity ittefaq nahi hai.
Stream asynchronous iteration ki tarah hota hai. Lekin jahan `trpl::Receiver` specifically
messages receive karne ka wait karta hai, wahin general-purpose stream API ka scope bohot zyada broad hai:
yeh `Iterator` ki tarah next item provide karta hai, lekin asynchronously.

Rust mein iterators aur streams ke darmiyan similarity ka matlab hai ke hum asal mein
kisi bhi iterator se ek stream create kar sakte hain. Iterator ki tarah, hum stream ke saath
is ke `next` method ko call karke aur phir output ko await karke kaam kar sakte hain, jaisa ke Listing
17-21 mein hai, jo abhi compile nahi hogi.

<Listing number="17-21" caption="Iterator se stream create karna aur is ki values print karna" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-21/src/main.rs:stream}}
```

</Listing>

Hum numbers ki ek array se shuru karte hain, jise hum iterator mein convert karte hain aur phir
tamam values ko double karne ke liye is par `map` call karte hain. Phir hum `trpl::stream_from_iter`
function ko use karke iterator ko stream mein convert karte hain. Is ke baad, hum `while let` loop ke
saath stream mein items ke arrive hone par un par loop karte hain.

Badqismati se, jab hum code ko run karne ki koshish karte hain, to yeh compile nahi hota balki
report karta hai ke koi `next` method available nahi hai:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-21
cargo build
copy only the error output
-->

```text
error[E0599]: no method named `next` found for struct `tokio_stream::iter::Iter` in the current scope
  --> src/main.rs:10:40
   |
10 |         while let Some(value) = stream.next().await {
   |                                        ^^^^
   |
   = help: items from traits can only be used if the trait is in scope
help: following traits which provide `next` are implemented but not in scope; perhaps you want to import one of them
   |
1  + use crate::trpl::StreamExt;
   |
1  + use futures_util::stream::stream::StreamExt;
   |
1  + use std::iter::Iterator;
   |
1  + use std::str::pattern::Searcher;
help: there is a method `try_next` with a similar name
   |
10 |         while let Some(value) = stream.try_next().await {
   |                                        ~~~~~~~~
```

Jaisa ke yeh output explain karta hai, compiler error ki wajah yeh hai ke `next` method ko use karne
ke qabil hone ke liye humein sahi trait ko scope mein lana zaroori hai. Ab tak ki hamari discussion
ko dekhte hue, aap reasonable taur par expect kar sakte hain ke woh trait `Stream` hoga, lekin asal
mein woh `StreamExt` hai. *Extension* ka short form, `Ext`, Rust community mein ek common pattern hai
jo ek trait ko doosre trait ke saath extend karne ke liye use hota hai.

`Stream` trait ek low-level interface define karta hai jo effectively `Iterator` aur `Future` traits
ko combine karta hai. `StreamExt`, `Stream` ke upar higher-level APIs ka ek set provide karta hai,
jis mein `next` method ke saath saath `Iterator` trait ke diye gaye doosre utility methods jaise
methods bhi shamil hain. `Stream` aur `StreamExt` abhi Rust ki standard library ka hissa nahi hain,
lekin ecosystem ki zyada tar crates similar definitions use karti hain.

Compiler error ka fix `trpl::StreamExt` ke liye ek `use` statement add karna hai, jaisa ke Listing
17-22 mein hai.

<Listing number="17-22" caption="Iterator ko stream ki basis ke taur par successfully use karna" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-22/src/main.rs:all}}
```

</Listing>

In tamam pieces ko ek saath rakhne ke baad, yeh code bilkul usi tarah work karta hai jaisa hum chahte hain!
Mazeed yeh ke, ab jab `StreamExt` scope mein hai, hum is ke tamam utility methods ko use kar sakte hain,
bilkul iterators ki tarah.

[17-02-messages]: ch17-02-concurrency-with-async.html#message-passing
[iterator-trait]: ch13-02-iterators.html#the-iterator-trait-and-the-next-method
