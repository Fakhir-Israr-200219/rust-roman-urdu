## Using Threads to Run Code Simultaneously

Zyada tar current operating systems mein, execute hone wale program ka code ek
*process* mein run hota hai, aur operating system ek hi waqt mein multiple
processes ko manage karta hai. Ek program ke andar bhi aap independent parts rakh
sakte hain jo simultaneously run hote hain. Woh features jo in independent parts ko
run karte hain *threads* kehlate hain. Misal ke taur par, ek web server ke paas
multiple threads ho sakte hain taake woh ek hi waqt mein ek se zyada requests ka
response de sake.

Apne program ki computation ko multiple threads mein split karna taake multiple
tasks ek hi waqt mein run hon, performance ko improve kar sakta hai, lekin is se
complexity bhi add hoti hai. Kyun ke threads simultaneously run ho sakte hain,
is liye is baat ki koi inherent guarantee nahi hoti ke different threads par aapke
code ke parts kis order mein run honge. Is se problems paida ho sakti hain, jaise:

* Race conditions, jismein threads data ya resources ko ek inconsistent order mein
  access kar rahe hote hain
* Deadlocks, jismein do threads ek doosre ka wait kar rahe hote hain, jis ki wajah se dono
  threads continue nahi kar pate
* Aise bugs jo sirf kuch situations mein hote hain aur jinhein reliably reproduce
  aur fix karna mushkil hota hai

Rust threads use karne ke negative effects ko kam karne ki koshish karta hai, lekin
multithreaded context mein programming ke liye phir bhi careful thought ki zarurat
hoti hai aur code structure aise programs se different hota hai jo ek single
thread mein run karte hain.

Programming languages threads ko kuch different ways mein implement karti hain, aur bohot
se operating systems ek API provide karte hain jise programming language new
threads create karne ke liye call kar sakti hai. Rust standard library threads ki
*1:1* model of thread implementation use karti hai, jismein ek program har ek
language thread ke liye ek operating system thread use karta hai. Aise crates bhi
maujood hain jo threading ke doosre models implement karte hain aur 1:1 model ke
muqable mein different trade-offs offer karte hain. (Rust ka async system, jise hum
next chapter mein dekhenge, concurrency ke liye ek aur approach provide karta hai.)

### Creating a New Thread with `spawn`

Ek naya thread create karne ke liye, hum `thread::spawn` function ko call karte hain aur
use ek closure pass karte hain (humne Chapter 13 mein closures ke baare mein baat ki thi)
jismein woh code hota hai jo hum naye thread mein run karna chahte hain. Listing 16-1 mein
di gayi example ek main thread se kuch text print karti hai aur ek naye thread se doosra
text print karti hai.

<Listing number="16-1" file-name="src/main.rs" caption="Creating a new thread to print one thing while the main thread prints something else">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-01/src/main.rs}}
```

</Listing>

Note karein ke jab Rust program ka main thread complete ho jata hai, to saare spawned threads
shutdown ho jate hain, chahe unhon ne running complete ki ho ya nahi. Is program ka output
har baar thora different ho sakta hai, lekin yeh kuch is tarah nazar aayega:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

`thread::sleep` ko call karna thread ko thori der ke liye apni execution rokne par majboor karta
hai, jis se kisi doosre thread ko run karne ka mauqa milta hai. Threads shayad bari bari run
karein, lekin is ki guarantee nahi hai: Yeh is baat par depend karta hai ke aapka operating system
threads ko kis tarah schedule karta hai. Is run mein, main thread ne pehle print kiya, halaan ke
spawned thread ka print statement code mein pehle nazar aata hai. Aur halaan ke humne spawned
thread ko `i` ke `9` tak print karne ke liye kaha tha, woh main thread ke shutdown hone se pehle
sirf `5` tak hi pohanch saka.

Agar aap yeh code run karein aur sirf main thread ka output dekhein, ya koi overlap nazar na aaye,
to ranges mein numbers ko barhane ki koshish karein taake operating system ko threads ke darmiyan
switch karne ke zyada opportunities mil sakein.

<!-- Old headings. Do not remove or links may break. -->

<a id="waiting-for-all-threads-to-finish-using-join-handles"></a>

### Waiting for All Threads to Finish

Listing 16-1 ka code aksar main thread ke end hone ki wajah se spawned thread ko waqt se pehle
na sirf rok deta hai, balki kyun ke threads kis order mein run honge is ki koi guarantee nahi,
hum yeh bhi guarantee nahi kar sakte ke spawned thread ko run karne ka mauqa milega bhi ya nahi!

Hum spawned thread ke run na karne ya waqt se pehle end ho jane ki problem ko `thread::spawn`
ki return value ko ek variable mein save karke fix kar sakte hain. `thread::spawn` ka return type
`JoinHandle<T>` hai. Ek `JoinHandle<T>` ek owned value hai jo, jab hum is par `join` method
call karte hain, to apne thread ke finish hone ka wait karegi. Listing 16-2 dikhati hai ke
Listing 16-1 mein create kiye gaye thread ke `JoinHandle<T>` ko kaise use karein aur `join`
ko kaise call karein taake `main` exit hone se pehle spawned thread finish ho jaye.

<Listing number="16-2" file-name="src/main.rs" caption="Saving a `JoinHandle<T>` from `thread::spawn` to guarantee the thread is run to completion">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-02/src/main.rs}}
```

</Listing>

Handle par `join` call karna currently running thread ko us waqt tak block kar deta hai jab tak
handle se represented thread terminate nahi ho jata. *Blocking* ka matlab hai ke us thread ko
work perform karne ya exit hone se rok diya jata hai. Kyun ke humne `join` ki call main thread ke
`for` loop ke baad rakhi hai, Listing 16-2 ko run karne par output kuch is tarah hona chahiye:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

Dono threads bari bari continue karte rehte hain, lekin main thread `handle.join()` ki call ki
wajah se wait karta hai aur tab tak end nahi hota jab tak spawned thread finish na ho jaye.

Lekin ab dekhte hain ke agar hum `main` ke `for` loop se pehle `handle.join()` ko move kar dein,
jaise ke yahan hai:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/no-listing-01-join-too-early/src/main.rs}}
```

</Listing>

Main thread spawned thread ke finish hone ka wait karega aur us ke baad apna `for` loop run karega,
is liye output ab interleaved nahi hoga, jaisa ke yahan dikhaya gaya hai:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

Choti choti details, jaise ke `join` ko kahan call kiya jata hai, is baat par asar daal sakti hain
ke aapke threads ek hi waqt mein run karte hain ya nahi.

### Using `move` Closures with Threads

Hum aksar `thread::spawn` ko pass kiye jane wale closures ke saath `move` keyword use
karenge kyun ke phir closure environment se un values ki ownership le lega jinhein woh
use karta hai, aur is tarah un values ki ownership ek thread se doosre thread ko transfer
ho jayegi. Chapter 13 mein [“Capturing References or Moving Ownership”][capture]<!-- ignore
--> ke context mein humne closures ke hawale se `move` discuss kiya tha. Ab hum
`move` aur `thread::spawn` ke darmiyan interaction par zyada focus karenge.

Listing 16-1 mein note karein ke jo closure hum `thread::spawn` ko pass karte hain woh koi
arguments nahi leta: Hum spawned thread ke code mein main thread ka koi data use nahi kar
rahe. Main thread se data ko spawned thread mein use karne ke liye, spawned thread ke closure
ko un values ko capture karna hoga jin ki use usay zarurat hai. Listing 16-3 main thread mein
ek vector create karne aur use spawned thread mein use karne ki koshish dikhati hai. Lekin,
jaisa ke aap ek lamhe mein dekhenge, yeh abhi kaam nahi karega.

<Listing number="16-3" file-name="src/main.rs" caption="Attempting to use a vector created by the main thread in another thread">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-03/src/main.rs}}
```

</Listing>

Closure `v` ko use karta hai, is liye woh `v` ko capture karega aur use closure ke
environment ka hissa bana dega. Kyun ke `thread::spawn` is closure ko ek naye thread mein
run karta hai, humein `v` ko us naye thread ke andar access kar sakna chahiye. Lekin jab hum
is example ko compile karte hain, to humein yeh error milta hai:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-03/output.txt}}
```

Rust *infer* karta hai ke `v` ko kis tarah capture karna hai, aur kyun ke `println!` ko sirf
`v` ke reference ki zarurat hai, closure `v` ko borrow karne ki koshish karta hai. Lekin yahan
ek problem hai: Rust yeh nahi bata sakta ke spawned thread kitni der tak run karega, is liye
use yeh maloom nahi ke `v` ka reference hamesha valid rahega ya nahi.

Listing 16-4 ek aisi situation provide karti hai jahan `v` ka reference valid na rehne ka
imkaan zyada hai.

<Listing number="16-4" file-name="src/main.rs" caption="A thread with a closure that attempts to capture a reference to `v` from a main thread that drops `v`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-04/src/main.rs}}
```

</Listing>

Agar Rust humein yeh code run karne deta, to ek possibility yeh hoti ke spawned thread ko
foran background mein bhej diya jata aur woh bilkul run hi na karta. Spawned thread ke andar
`v` ka reference hai, lekin main thread foran `v` ko drop kar deta hai, `drop` function ko use
karte hue jis par humne Chapter 15 mein baat ki thi. Phir, jab spawned thread execute karna
shuru karta hai, `v` ab valid nahi rehta, is liye us ka reference bhi invalid hota hai. Oh no!

Listing 16-3 mein compiler error ko fix karne ke liye, hum error message ke mashware ko use
kar sakte hain:

<!-- manual-regeneration
after automatic regeneration, look at listings/ch16-fearless-concurrency/listing-16-03/output.txt and copy the relevant part
-->

```text
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

Closure se pehle `move` keyword add karke, hum closure ko un values ki ownership lene par majboor
karte hain jinhein woh use kar raha hai, bajaye is ke ke Rust khud infer kare ke use values ko
borrow karna chahiye. Listing 16-3 mein Listing 16-5 ke mutabiq ki gayi yeh modification compile
aur run hogi, jaisa hum chahte hain.

<Listing number="16-5" file-name="src/main.rs" caption="Using the `move` keyword to force a closure to take ownership of the values it uses">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-05/src/main.rs}}
```

</Listing>

Humein yeh khayal aa sakta hai ke Listing 16-4 ke code ko bhi isi tarah fix karne ki koshish
karein, jahan main thread ne `drop` call kiya tha, aur `move` closure use karein. Lekin yeh fix
kaam nahi karega kyun ke Listing 16-4 jo karne ki koshish kar rahi hai woh ek different reason
ki wajah se disallowed hai. Agar hum closure mein `move` add kar dein, to hum `v` ko closure ke
environment mein move kar denge, aur phir hum main thread mein us par `drop` call nahi kar
sakeinge. Is ke bajaye humein yeh compiler error milega:

```console
{{#include ../listings/ch16-fearless-concurrency/output-only-01-move-drop/output.txt}}
```

Rust ke ownership rules ne humein ek baar phir bacha liya! Listing 16-3 ke code se humein
error is liye mila kyun ke Rust conservative ho kar sirf thread ke liye `v` ko borrow kar raha
tha, jis ka matlab tha ke main thread theoretically spawned thread ke reference ko invalid
kar sakta tha. Rust ko yeh batakar ke `v` ki ownership spawned thread ko move karni hai, hum
Rust ko guarantee kar rahe hain ke main thread ab `v` ko use nahi karega. Agar hum Listing 16-4
ko bhi isi tarah change karte hain, to phir main thread mein `v` ko use karne ki koshish par hum
ownership rules ki violation kar rahe honge. `move` keyword Rust ke conservative default of
borrowing ko override karta hai; yeh humein ownership rules ki violation karne nahi deta.

Ab jab humne cover kar liya hai ke threads kya hain aur thread API ke provide kiye gaye methods
kya hain, to chaliye kuch aisi situations dekhte hain jahan hum threads ko use kar sakte hain.

[capture]: ch13-01-closures.html#capturing-references-or-moving-ownership
