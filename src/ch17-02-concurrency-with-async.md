<!-- Old headings. Do not remove or links may break. -->

<a id="concurrency-with-async"></a>

## Applying Concurrency with Async

Is section mein, hum async ko unhi concurrency challenges par apply karenge
jinhein hum ne Chapter 16 mein threads ke saath tackle kiya tha. Kyun ke hum
wahan bohot se key ideas par pehle hi baat kar chuke hain, is section mein hum
threads aur futures ke darmiyan jo different hai us par focus karenge.

Bohot se cases mein, async ko use karke concurrency ke saath kaam karne wali APIs
un APIs se bohot milti julti hain jo threads ko use karne ke liye hoti hain. Doosre
cases mein, yeh kaafi different ho jati hain. Jab threads aur async ke darmiyan APIs
*dekhne mein* similar bhi hon, tab bhi aksar un ka behavior different hota hai—aur
lagbhag hamesha un ki performance characteristics different hoti hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="counting"></a>

### Creating a New Task with `spawn_task`

Chapter 16 ke [“Creating a New Thread with
`spawn`”][thread-spawn]<!-- ignore --> section mein hum ne jo pehla operation tackle kiya tha woh
do separate threads par counting up karna tha. Aaiye async ko use karke bhi wahi kaam karte hain. `trpl` crate ek
`spawn_task` function provide karta hai jo `thread::spawn` API se bohot milta julta hai, aur
ek `sleep` function jo `thread::sleep` API ka async version hai. Hum in dono ko saath use
karke counting example implement kar sakte hain, jaisa ke Listing 17-6 mein dikhaya gaya hai.

<Listing number="17-6" caption="Main task ke kisi aur cheez ko print karte hue ek naya task bana kar ek cheez print karna" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-06/src/main.rs:all}}
```

</Listing>

Apne starting point ke taur par, hum apne `main` function ko `trpl::block_on` ke saath
set up karte hain taake hamara top-level function async ho sake.

> Note: Is point se chapter ke aakhir tak, har example mein `main` ke andar `trpl::block_on`
> ke saath yahi exact wrapping code shamil hoga, is liye hum aksar ise
> `main` ki tarah hi skip kar denge. Apne code mein ise include karna yaad rakhein!

Phir hum us block ke andar do loops likhte hain, jin mein se har ek mein `trpl::sleep`
call hai, jo next message send karne se pehle half second (500 milliseconds) wait karta hai.
Hum ek loop ko `trpl::spawn_task` ke body mein aur doosre ko ek top-level `for` loop mein
rakhte hain. Hum `sleep` calls ke baad ek `await` bhi add karte hain.

Yeh code thread-based implementation ki tarah hi behave karta hai—ismein yeh fact bhi shamil
hai ke jab aap ise apne terminal mein run karenge to aapko messages different order mein
appear hote hue nazar aa sakte hain:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
```

Yeh version jaise hi main async block ke body mein `for` loop finish hota hai, foran ruk jata hai,
kyun ke `spawn_task` se spawn kiya gaya task `main` function ke end hone par shut down ho jata hai.
Agar aap chahte hain ke yeh task poori tarah *complete* hone tak run kare, to aapko first task ke
complete hone ka wait karne ke liye join handle use karna hoga. Threads ke saath, hum ne
thread ke running finish hone tak “block” karne ke liye `join` method use kiya tha. Listing 17-7
mein, hum same kaam karne ke liye `await` use kar sakte hain, kyun ke task handle khud ek future
hai. Is ka `Output` type ek `Result` hai, is liye ise await karne ke baad hum ise unwrap bhi karte hain.

<Listing number="17-7" caption="Task ko completion tak run karne ke liye join handle ke saath `await` use karna" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-07/src/main.rs:handle}}
```

</Listing>

Yeh updated version tab tak run karta hai jab tak *dono* loops finish nahi ho jate:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

Abhi tak, aisa lagta hai ke async aur threads humein similar outcomes dete hain, bas syntax
different hai: join handle par `join` call karne ke bajaye `await` use karna, aur `sleep` calls
ko await karna.

Bara difference yeh hai ke humein yeh kaam karne ke liye ek aur operating system thread
spawn karne ki zaroorat nahi padi. Darasal, humein yahan task spawn karne ki bhi zaroorat nahi.
Kyun ke async blocks anonymous futures mein compile hote hain, hum har loop ko ek async
block mein rakh sakte hain aur runtime ko `trpl::join` function use karke dono ko completion
tak run karwa sakte hain.

Chapter 16 ke [“Waiting for All Threads to Finish”][join-handles]<!-- ignore -->
section mein, hum ne dikhaya tha ke `std::thread::spawn` ko call karne par return hone wale
`JoinHandle` type par `join` method ko kaise use karte hain. `trpl::join` similar hai, lekin
futures ke liye. Jab aap ise do futures dete hain, to yeh ek single new future produce karta hai
jis ka output ek tuple hota hai jo aapki di hui har future ka output contain karta hai, jab
*dono* complete ho jati hain. Is liye, Listing 17-8 mein, hum `fut1` aur `fut2` dono ke
finish hone ka wait karne ke liye `trpl::join` use karte hain. Hum `fut1` aur `fut2` ko
await *nahi* karte, balki `trpl::join` se produce hone wali new future ko await karte hain.
Hum output ko ignore kar dete hain, kyun ke yeh sirf do unit values wala tuple hai.

<Listing number="17-8" caption="Do anonymous futures ko await karne ke liye `trpl::join` use karna" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-08/src/main.rs:join}}
```

</Listing>

Jab hum ise run karte hain, to hum dekhte hain ke dono futures completion tak run hoti hain:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the first task!
hi number 1 from the second task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

Ab aapko har baar bilkul wahi order nazar aayega, jo threads aur Listing 17-7 mein
`trpl::spawn_task` ke saath dekhe gaye behavior se bohot different hai. Is ki wajah yeh hai ke
`trpl::join` function *fair* hai, yani yeh har future ko barabar frequently check karta hai,
un ke darmiyan alternate karta hai, aur agar doosri ready ho to kabhi ek ko aage race nahi karne
deta. Threads ke saath, operating system decide karta hai ke kis thread ko check karna hai aur
use kitni dair run karne dena hai. Async Rust ke saath, runtime decide karta hai ke kis task ko
check karna hai. (Practice mein, details complicated ho jati hain kyun ke ek async runtime
concurrency ko manage karne ke tareeqe ke taur par under the hood operating system threads
use kar sakta hai, is liye fairness guarantee karna runtime ke liye zyada work ho sakta hai—
lekin phir bhi yeh possible hai!) Runtimes ko kisi given operation ke liye fairness guarantee
karna zaroori nahi hota, aur woh aksar different APIs provide karte hain taake aap choose kar
saken ke aap fairness chahte hain ya nahi.

Futures ko await karne ke in variations ko try karein aur dekhein ke yeh kya karte hain:

* Dono mein se kisi ek ya dono loops ke around async block remove kar dein.
* Har async block ko define karne ke foran baad await karein.
* Sirf first loop ko ek async block mein wrap karein, aur second loop ke body ke baad resulting future ko await karein.

Extra challenge ke liye, dekhein ke kya aap code run karne *se pehle* har case mein output
kya hoga yeh figure out kar sakte hain!

<!-- Old headings. Do not remove or links may break. -->

<a id="message-passing"></a> <a id="counting-up-on-two-tasks-using-message-passing"></a>

### Sending Data Between Two Tasks Using Message Passing

Futures ke darmiyan data share karna bhi familiar hoga: hum dobara message passing use karenge,
lekin is baar types aur functions ke async versions ke saath. Hum Chapter 16 ke [“Transfer Data Between Threads
with Message Passing”][message-passing-threads]<!-- ignore --> section mein jo tareeqa use kiya tha
us se thora different path lenge taake thread-based aur futures-based concurrency ke darmiyan
kuch key differences ko illustrate kiya ja sake. Listing 17-9 mein, hum sirf ek single
async block se shuru karenge—*separate task spawn kiye baghair*, jaisa ke hum ne separate thread
spawn kiya tha.

<Listing number="17-9" caption="Ek async channel create karna aur is ke dono halves ko `tx` aur `rx` assign karna" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-09/src/main.rs:channel}}
```

</Listing>

Yahan, hum `trpl::channel` use karte hain, jo multiple-producer, single-consumer channel API
ka async version hai jo hum ne Chapter 16 mein threads ke saath use kiya tha. API ka async
version thread-based version se sirf thora different hai: yeh immutable receiver `rx` ke bajaye
mutable receiver use karta hai, aur is ka `recv` method value ko directly produce karne ke
bajaye ek aisi future produce karta hai jise humein await karna hota hai.
Ab hum sender se receiver ko messages bhej sakte hain. Ghaur karein ke humein na to separate
thread spawn karna hai aur na hi task; humein sirf `rx.recv` call ko await karna hai.

`std::mpsc::channel` mein synchronous `Receiver::recv` method tab tak block karta hai jab tak
use koi message receive na ho. `trpl::Receiver::recv` aisa nahi karta, kyun ke yeh async hai.
Block karne ke bajaye, yeh control runtime ko wapas de deta hai jab tak ya to koi message
receive ho ya channel ka send side close ho jaye. Is ke baraks, hum `send` call ko await nahi
karte, kyun ke yeh block nahi karta. Is ki zaroorat bhi nahi, kyun ke jis channel mein hum
send kar rahe hain woh unbounded hai.

> Note: Kyun ke yeh tamam async code ek `trpl::block_on` call ke andar ek
> async block mein run hota hai, is ke andar ki har cheez blocking se bach sakti hai. Lekin
> is ke *bahar* ka code `block_on` function ke return hone ka wait karte hue block hoga.
> `trpl::block_on` function ka poora point yahi hai: yeh aapko *choose* karne deta hai ke
> async code ke kisi set par kahan block karna hai, aur is tarah yeh bhi choose karne deta hai
> ke sync aur async code ke darmiyan kahan transition karna hai.

Is example ke baare mein do cheezen notice karein. Pehli, message foran arrive ho jayega.
Doosri, halanke hum yahan ek future use kar rahe hain, abhi koi concurrency nahi hai.
Listing mein sab kuch sequence mein hota hai, bilkul usi tarah jaise agar futures involve hi
na hotin.

Aaiye pehle part ko address karte hain aur messages ki ek series send karte hain aur un ke
darmiyan sleep karte hain, jaisa ke Listing 17-10 mein dikhaya gaya hai.

<!-- We cannot test this one because it never stops! -->

<Listing number="17-10" caption="Async channel par multiple messages send aur receive karna aur har message ke darmiyan `await` ke saath sleep karna" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-10/src/main.rs:many-messages}}
```

</Listing>

Messages send karne ke saath saath, humein unhein receive bhi karna hoga. Is case mein,
kyun ke humein pata hai ke kitne messages aa rahe hain, hum `rx.recv().await` ko chaar baar
call karke manually yeh kar sakte hain. Real world mein, however, hum aam tor par messages ki
kisi *unknown* tadaad ka wait kar rahe honge, is liye humein tab tak wait karte rehna hoga jab
tak hum determine na kar lein ke ab aur messages nahi hain.

Listing 16-10 mein, hum ne synchronous channel se receive hone wale tamam items ko process
karne ke liye ek `for` loop use kiya tha. Lekin Rust ke paas abhi tak
*asynchronously produced* items ki series ke saath `for` loop use karne ka koi tareeqa nahi
hai, is liye humein ek aisa loop use karna hoga jo hum ne pehle nahi dekha: `while let`
conditional loop. Yeh `if let` construct ka loop version hai jis ko hum ne Chapter 6 ke
[“Concise Control Flow with `if
let` and `let...else`”][if-let]<!-- ignore --> section mein dekha tha. Yeh loop tab tak
execute hota rahega jab tak is mein specify kiya gaya pattern value se match karta rahe.

`rx.recv` call ek future produce karti hai, jise hum await karte hain. Runtime future ko tab
tak pause karega jab tak woh ready nahi ho jati. Jab koi message arrive hota hai, future
jitni baar koi message arrive hoga utni baar `Some(message)` mein resolve hogi. Jab channel
close ho jata hai, chahe *koi* messages aaye hon ya na aaye hon, future is ke bajaye
`None` mein resolve hogi, jo indicate karta hai ke ab koi aur values nahi hain aur is liye
humein polling rok deni chahiye—yani awaiting rok deni chahiye.

`while let` loop yeh sab ek saath laata hai. Agar `rx.recv().await` call karne ka result
`Some(message)` hai, to humein message tak access mil jata hai aur hum ise loop body mein
use kar sakte hain, bilkul waise hi jaise hum `if let` ke saath kar sakte the. Agar result
`None` ho, to loop end ho jata hai. Har baar jab loop complete hota hai, yeh dobara await
point par pohanchta hai, is liye runtime ise dobara pause karta hai jab tak koi aur message
arrive na ho.

Ab code successfully tamam messages send aur receive karta hai.
Badqismati se, abhi bhi kuch problems hain. Ek baat yeh hai ke messages half-second intervals
par arrive nahi karte. Woh sab ek saath, program start hone ke 2 seconds (2,000 milliseconds)
baad arrive hote hain. Doosri baat, yeh program bhi kabhi exit nahi karta! Is ke bajaye, yeh
new messages ka hamesha wait karta rehta hai. Aapko ise <kbd>ctrl</kbd>-<kbd>C</kbd> use karke
band karna hoga.

#### Code Within One Async Block Executes Linearly

Aaiye yeh examine karne se shuru karte hain ke messages har ek ke darmiyan delay ke saath aane ke
bajaye, poore delay ke baad sab ek saath kyun aate hain. Kisi given async
block ke andar, code mein `await` keywords jis order mein appear hote hain, program run
hone par unhein bhi usi order mein execute kiya jata hai.

Listing 17-10 mein sirf ek async block hai, is liye is ke andar sab kuch
linearly run hota hai. Abhi bhi koi concurrency nahi hai. Tamam `tx.send` calls
execute hoti hain, aur un ke darmiyan tamam `trpl::sleep` calls aur un se associated await
points hote hain. Sirf us ke baad `while let` loop ko `recv` calls par maujood kisi bhi
`await` points se guzarne ka mauqa milta hai.

Jo behavior hum chahte hain, jahan har message ke darmiyan sleep delay ho, use hasil karne ke
liye humein `tx` aur `rx` operations ko apne apne async blocks mein rakhna hoga, jaisa ke
Listing 17-11 mein dikhaya gaya hai. Phir runtime `trpl::join` ko use karke in mein se har ek
ko separately execute kar sakta hai, bilkul Listing 17-8 ki tarah. Ek baar phir, hum
`trpl::join` call karne ke result ko await karte hain, individual futures ko nahi. Agar hum
individual futures ko sequence mein await karte, to hum dobara sequential flow mein pohanch
jate—bilkul wohi cheez jo hum *nahi* karna chahte.

<!-- We cannot test this one because it never stops! -->

<Listing number="17-11" caption="`send` aur `recv` ko apne apne `async` blocks mein separate karna aur un blocks ki futures ko await karna" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-11/src/main.rs:futures}}
```

</Listing>

Listing 17-11 ke updated code ke saath, messages har 500-millisecond interval par print hote
hain, bajaye is ke ke 2 seconds ke baad sab ek saath jaldi jaldi print hon.

#### Moving Ownership Into an Async Block

Program ab bhi kabhi exit nahi karta, lekin is ki wajah yeh hai ke `while let` loop
`trpl::join` ke saath kis tarah interact karta hai:

* `trpl::join` se return hone wala future sirf us waqt complete hota hai jab usay
  diye gaye *dono* futures complete ho chuke hon.
* `tx_fut` future us waqt complete hota hai jab `vals` mein maujood last
  message send karne ke baad sleep karna complete ho jata hai.
* `rx_fut` future us waqt tak complete nahi hoga jab tak `while let` loop end na ho.
* `while let` loop us waqt tak end nahi hoga jab tak `rx.recv` ko await karne se `None` na mil jaye.
* `rx.recv` ko await karne se `None` sirf us waqt return hoga jab channel ka doosra end
  close ho jaye.
* Channel sirf us waqt close hoga jab hum `rx.close` call karein ya sender side,
  `tx`, drop ho jaye.
* Hum kahin bhi `rx.close` call nahi karte, aur `tx` tab tak drop nahi hoga jab tak
  `trpl::block_on` ko pass kiya gaya outermost async block end na ho jaye.
* Block end nahi ho sakta kyun ke woh `trpl::join` ke complete hone par blocked hai,
  jo humein dobara is list ke top par le aata hai.

Abhi, woh async block jahan hum messages send karte hain sirf `tx` ko *borrow* karta hai
kyun ke message send karne ke liye ownership ki zaroorat nahi hoti, lekin agar hum `tx` ko
us async block mein *move* kar saken, to woh block end hote hi drop ho jayega. Chapter 13 ke
[“Capturing References or Moving Ownership”][capture-or-move]<!-- ignore -->
section mein aapne seekha tha ke closures ke saath `move` keyword kaise use karte hain,
aur Chapter 16 ke [“Using `move` Closures with
Threads”][move-threads]<!-- ignore --> section mein, humne dekha tha ke threads ke
saath kaam karte waqt humein aksar data ko closures mein move karna padta hai. Yehi basic
dynamics async blocks par bhi apply hoti hain, is liye `move` keyword async blocks ke saath
bilkul usi tarah kaam karta hai jaise closures ke saath karta hai.

Listing 17-12 mein, hum messages send karne ke liye use hone wale block ko `async` se
`async move` mein change karte hain.

<Listing number="17-12" caption="Listing 17-11 ke code ki ek revision jo complete hone par sahi tarah shutdown ho jati hai" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-12/src/main.rs:with-move}}
```

</Listing>

Jab hum *is* version of the code ko run karte hain, to last message send aur receive hone
ke baad yeh gracefully shutdown ho jata hai. Ab dekhte hain ke ek se zyada future se data
send karne ke liye kya change karna hoga.

#### Joining a Number of Futures with the `join!` Macro

Yeh async channel bhi ek multiple-producer channel hai, is liye agar hum multiple futures se
messages send karna chahein to `tx` par `clone` call kar sakte hain, jaisa ke Listing
17-13 mein dikhaya gaya hai.

<Listing number="17-13" caption="Async blocks ke saath multiple producers use karna" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-13/src/main.rs:here}}
```

</Listing>

Sab se pehle, hum `tx` ko clone karte hain, jis se pehle async block ke bahar `tx1` create hota hai. Hum
`tx1` ko us block mein move karte hain, bilkul usi tarah jaise pehle `tx` ke saath kiya tha. Phir, baad mein,
hum original `tx` ko ek *new* async block mein move karte hain, jahan hum thore zyada slow delay ke saath
mazeed messages send karte hain. Ittefaq se hum is new async block ko messages receive karne wale async
block ke baad rakhte hain, lekin isay us se pehle bhi bilkul isi tarah rakha ja sakta tha. Key yeh hai ke
futures kis order mein await kiye jate hain, na ke yeh ke woh kis order mein create kiye jate hain.

Messages send karne wale dono async blocks ka `async move` blocks hona zaroori hai taake `tx` aur `tx1`
dono un blocks ke finish hone par drop ho jayein. Warna hum dobara usi infinite loop mein phans jayenge
jahan se humne shuru kiya tha.

Aakhir mein, hum additional future ko handle karne ke liye `trpl::join` se `trpl::join!` par switch karte
hain: `join!` macro arbitrary number of futures ko await karta hai jab humein compile time par futures ki
number maloom ho. Is chapter mein baad mein hum unknown number of futures ki collection ko await karne par
baat karenge.

Ab humein dono sending futures se tamam messages nazar aate hain, aur kyun ke sending
futures message send karne ke baad thore different delays use karte hain, messages bhi un
different intervals par receive hote hain:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
received 'hi'
received 'more'
received 'from'
received 'the'
received 'messages'
received 'future'
received 'for'
received 'you'
```

Humne explore kiya hai ke message passing ko use karke futures ke darmiyan data kaise send kiya jata hai,
async block ke andar code sequentially kaise run hota hai, ownership ko async block mein kaise move kiya jata hai,
aur multiple futures ko kaise join kiya jata hai. Ab, aaiye discuss karte hain ke runtime ko yeh batana ke woh
kisi doosre task par switch kar sakta hai, kaise aur kyun kiya jata hai.

[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[join-handles]: ch16-01-threads.html#waiting-for-all-threads-to-finish
[message-passing-threads]: ch16-02-message-passing.html
[if-let]: ch06-03-if-let.html
[capture-or-move]: ch13-01-closures.html#capturing-references-or-moving-ownership
[move-threads]: ch16-01-threads.html#using-move-closures-with-threads
