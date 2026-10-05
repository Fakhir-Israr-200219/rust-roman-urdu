<!-- Old headings. Do not remove or links may break. -->

<a id="turning-our-single-threaded-server-into-a-multithreaded-server"></a>
<a id="from-single-threaded-to-multithreaded-server"></a>

## From a Single-Threaded to a Multithreaded Server

Filhaal, server har request ko bari bari process karta hai, yani pehli connection
ki processing complete hone tak yeh doosri connection ko process nahi karega.
Agar server ko zyada se zyada requests receive hoti rahein, to yeh serial
execution dheere dheere kam optimal hoti jayegi. Agar server ko aisi request
receive ho jo process hone mein kaafi waqt leti hai, to us ke baad aane wali
requests ko us long request ke complete hone tak wait karna padega, chahe nayi
requests ko jaldi process kiya ja sakta ho. Humein isay fix karna hoga, lekin
pehle hum problem ko action mein dekhenge.

<!-- Old headings. Do not remove or links may break. -->

<a id="simulating-a-slow-request-in-the-current-server-implementation"></a>

### Simulating a Slow Request

Hum dekhenge ke slowly processing request hamari current server implementation
par ki jane wali doosri requests ko kaise affect kar sakti hai. Listing 21-10
*/sleep* ki request ko ek simulated slow response ke saath handle karne ko
implement karti hai, jo response dene se pehle server ko paanch seconds ke liye
sleep karwayegi.

<Listing number="21-10" file-name="src/main.rs" caption="Paanch seconds ke liye sleep karke ek slow request ko simulate karna">

```rust,no_run id="q7f2km"
{{#rustdoc_include ../listings/ch21-web-server/listing-21-10/src/main.rs:here}}
```

</Listing>

Ab hum `if` se `match` par switch ho gaye hain kyun ke ab hamare paas teen
cases hain. Humein `request_line` ki slice par explicitly match karna hoga taake
string literal values ke against pattern-match kar saken; `match`, equality
method ki tarah automatic referencing aur dereferencing nahi karta.

Pehla arm Listing 21-9 ke `if` block jaisa hi hai. Doosra arm */sleep* ki
request se match karta hai. Jab yeh request receive hoti hai, to successful
HTML page render karne se pehle server paanch seconds ke liye sleep karega.
Teesra arm Listing 21-9 ke `else` block jaisa hi hai.

Aap dekh sakte hain ke hamara server kitna primitive hai: Real libraries
multiple requests ki recognition ko bohat kam verbose tareeqe se handle
karengi!

`cargo run` use karke server start karein. Phir, do browser windows open karein:
ek *http://127.0.0.1:7878* ke liye aur doosri
*http://127.0.0.1:7878/sleep* ke liye. Agar aap pehle ki tarah */* URI ko kuch
baar enter karein, to aap dekhenge ke yeh quickly respond karta hai. Lekin agar
aap */sleep* enter karein aur phir */* load karein, to aap dekhenge ke */*
load hone se pehle `sleep` ke apne poore paanch seconds sleep karne ka wait
karta hai.

Hum requests ko slow request ke peeche backup hone se bachane ke liye multiple
techniques use kar sakte hain, jin mein async ka use bhi shamil hai jaisa ke
humne Chapter 17 mein kiya tha; jo technique hum implement karenge woh thread
pool hai.

### Improving Throughput with a Thread Pool

Ek *thread pool* spawned threads ka ek group hota hai jo kisi task ko handle
karne ke liye ready aur waiting hota hai. Jab program ko koi naya task receive
hota hai, to woh pool ke threads mein se ek thread ko task assign karta hai, aur
woh thread task ko process karega. Pool mein baqi threads kisi bhi doosre task
ko handle karne ke liye available rehte hain jo pehle thread ke processing ke
dauran aata hai. Jab pehla thread apne task ki processing complete kar leta hai,
to use idle threads ke pool mein wapas kar diya jata hai, jahan woh naye task ko
handle karne ke liye ready hota hai. Thread pool aapko connections ko
concurrently process karne deta hai, jis se aapke server ka throughput barhta
hai.

Hum pool mein threads ki tadaad ko ek chhoti tadaad tak limit karenge taake
hum DoS attacks se protect rahen; agar humara program har incoming request ke
liye ek naya thread create karta rahe, to hamare server par 10 million requests
karne wala koi shakhs hamare server ke tamam resources use karke tabahi macha
sakta hai aur requests ki processing ko bilkul rok sakta hai.

Is liye unlimited threads spawn karne ke bajaye, hum pool mein threads ki ek
fixed tadaad waiting mein rakhenge. Jo requests aayengi unhein processing ke
liye pool mein bheja jayega. Pool incoming requests ki ek queue maintain karega.
Pool ka har thread is queue se ek request pop karega, request ko handle karega,
aur phir queue se ek aur request maangega. Is design ke saath, hum ek waqt mein
*`N`* requests tak concurrently process kar sakte hain, jahan *`N`* threads ki
tadaad hai. Agar har thread ek long-running request ka response de raha ho, to
us ke baad aane wali requests ab bhi queue mein jama ho sakti hain, lekin humne
us point tak pohanchne se pehle handle ki ja sakne wali long-running requests
ki tadaad barha di hai.

Yeh technique web server ke throughput ko improve karne ke kai tareeqon mein se
sirf ek hai. Doosre options jinhein aap explore kar sakte hain, fork/join model,
single-threaded async I/O model, aur multithreaded async I/O model hain. Agar
aap is topic mein interested hain, to aap doosre solutions ke bare mein aur
parh sakte hain aur unhein implement karne ki koshish kar sakte hain; Rust jaisi
low-level language ke saath yeh tamam options possible hain.

Thread pool implement karna shuru karne se pehle, aao baat karte hain ke pool
ka use karna kaisa dikhna chahiye. Jab aap code design karne ki koshish kar rahe
hote hain, to pehle client interface likhna aapke design ko guide karne mein
madad kar sakta hai. Code ki API ko is tarah structure karein ke aap use jis
tareeqe se call karna chahte hain, woh structure us mein maujood ho; phir us
structure ke andar functionality implement karein, bajaye is ke ke pehle
functionality implement karein aur us ke baad public API design karein.

Chapter 12 ke project mein jis tarah humne test-driven development use kiya
tha, usi tarah yahan hum compiler-driven development use karenge. Hum woh code
likhenge jo un functions ko call karta hai jinhein hum chahte hain, aur phir
hum compiler ke errors ko dekhenge taake determine kar saken ke code ko kaam
karwane ke liye humein agla kya change karna chahiye. Lekin is se pehle, hum
us technique ko explore karenge jise hum starting point ke taur par use nahi
karne wale.

<!-- Old headings. Do not remove or links may break. -->

<a id="code-structure-if-we-could-spawn-a-thread-for-each-request"></a>

#### Spawning a Thread for Each Request

Sab se pehle, aao explore karte hain ke agar hamara code har connection ke liye
ek naya thread create karta to woh kaisa nazar aa sakta tha. Jaisa ke pehle
mention kiya gaya hai, potentially unlimited number of threads spawn karne ke
problems ki wajah se yeh hamara final plan nahi hai, lekin yeh pehle ek working
multithreaded server banane ke liye ek starting point hai. Phir hum ek
improvement ke taur par thread pool add karenge, aur dono solutions ka
comparison karna aasaan hoga.

Listing 21-11 `main` mein ki jane wali changes dikhati hai taake `for` loop ke
andar har stream ko handle karne ke liye ek naya thread spawn kiya ja sake.

<Listing number="21-11" file-name="src/main.rs" caption="Har stream ke liye ek naya thread spawn karna">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-11/src/main.rs:here}}
```

</Listing>

Jaisa ke aapne Chapter 16 mein seekha, `thread::spawn` ek naya thread create
karta hai aur phir closure mein maujood code ko naye thread mein run karta hai.
Agar aap is code ko run karein aur apne browser mein */sleep* load karein, phir
do aur browser tabs mein */* load karein, to aap waqai dekhenge ke */* ki
requests ko */sleep* ke finish hone ka wait nahi karna padega. Lekin, jaisa ke
humne mention kiya, aakhir mein yeh system ko overwhelm kar dega kyun ke aap
bina kisi limit ke naye threads banate rahenge.

Aapko Chapter 17 se yeh bhi yaad ho sakta hai ke yeh bilkul woh situation hai
jahan async aur await waqai shine karte hain! Is baat ko yaad rakhein jab hum
thread pool build karte hain aur sochte hain ke async ke saath cheezein kis tarah
different ya same nazar aayengi.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-similar-interface-for-a-finite-number-of-threads"></a>

#### Creating a Finite Number of Threads

Hum chahte hain ke hamara thread pool ek similar, familiar tareeqe se kaam kare
taake threads se thread pool par switch karne ke liye hamari API ko use karne
wale code mein bohat zyada changes na karne paren. Listing 21-12 `ThreadPool`
struct ka woh hypothetical interface dikhati hai jise hum `thread::spawn` ki
jagah use karna chahte hain.

<Listing number="21-12" file-name="src/main.rs" caption="Hamara ideal `ThreadPool` interface">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-12/src/main.rs:here}}
```

</Listing>

Hum `ThreadPool::new` ko ek naya thread pool create karne ke liye use karte
hain jisme threads ki configurable tadaad hoti hai, is case mein four. Phir
`for` loop mein, `pool.execute` ka interface `thread::spawn` jaisa hai, is
maayne mein ke yeh ek closure leta hai jise pool har stream ke liye run kare.
Humein `pool.execute` ko is tarah implement karna hai ke yeh closure ko le aur
use pool ke ek thread ko run karne ke liye de. Yeh code abhi compile nahi hoga,
lekin hum phir bhi ise try karenge taake compiler humein guide kar sake ke ise
theek karne ke liye humein kya change karna hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="building-the-threadpool-struct-using-compiler-driven-development"></a>

#### Building `ThreadPool` Using Compiler-Driven Development

Listing 21-12 mein changes *src/main.rs* mein karein, aur phir `cargo check` se
milne wale compiler errors ko apni development ko guide karne dein. Humein jo
pehla error milta hai woh yeh hai:

```console
{{#include ../listings/ch21-web-server/listing-21-12/output.txt}}
```

Great! Yeh error humein batata hai ke humein `ThreadPool` type ya module ki
zaroorat hai, to ab hum ise build karenge. Hamari `ThreadPool` implementation
is baat se independent hogi ke hamara web server kis tarah ka kaam kar raha hai.
Is liye, aao `hello` crate ko binary crate se library crate mein switch karte
hain taake is mein hamari `ThreadPool` implementation rakhi ja sake. Library
crate mein change hone ke baad, hum separate thread pool library ko kisi bhi
aise kaam ke liye bhi use kar sakte hain jo hum thread pool ka use karke karna
chahte hain, sirf web requests serve karne ke liye nahi.

Ek *src/lib.rs* file create karein jismein abhi ke liye `ThreadPool` struct ki
sab se simple definition ho jo hum bana sakte hain:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/lib.rs}}
```

</Listing>

Phir, *main.rs* file ko edit karein taake library crate se `ThreadPool` ko scope
mein laya ja sake. Is ke liye *src/main.rs* ke top par yeh code add karein:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/main.rs:here}}
```

</Listing>

Yeh code abhi bhi kaam nahi karega, lekin aao ise dobara check karte hain taake
agla error mil sake jise humein address karna hai:

```console
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/output.txt}}
```

Yeh error indicate karta hai ke ab humein `ThreadPool` ke liye `new` naam ka
ek associated function create karna hai. Humein yeh bhi pata hai ke `new` mein
ek parameter hona chahiye jo argument ke taur par `4` accept kar sake aur ek
`ThreadPool` instance return karna chahiye. Aao sab se simple `new` function
implement karte hain jismein yeh characteristics hon:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/src/lib.rs}}
```

</Listing>

We chose `usize` as the type of the `size` parameter because we know that a
negative number of threads doesn’t make any sense. We also know we’ll use this
`4` as the number of elements in a collection of threads, which is what the
`usize` type is for, as discussed in the [“Integer Types”][integer-types]<!--
ignore --> section in Chapter 3.

Humne `size` parameter ke type ke liye `usize` choose kiya kyun ke hum jaante
hain ke threads ki negative tadaad ka koi maani nahi hai. Hum yeh bhi jaante
hain ke hum is `4` ko threads ke collection mein elements ki tadaad ke taur par
use karenge, aur `usize` type isi kaam ke liye hai, jaisa ke Chapter 3 ke
[“Integer Types”][integer-types]<!-- ignore --> section mein discuss kiya gaya
hai.

Aao code ko dobara check karte hain:

```console
{{#include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/output.txt}}
```

Ab error is liye aa raha hai kyun ke hamare paas `ThreadPool` par `execute`
method nahi hai. [“Creating a Finite Number of
Threads”](#creating-a-finite-number-of-threads)<!-- ignore --> section se yaad
karein ke humne decide kiya tha ke hamare thread pool ka interface
`thread::spawn` ke similar hona chahiye. Is ke ilawa, hum `execute` function ko
is tarah implement karenge ke yeh di gayi closure ko le aur use pool ke kisi
idle thread ko run karne ke liye de.

Hum `ThreadPool` par `execute` method define karenge jo ek closure ko parameter
ke taur par lega. Chapter 13 ke [“Moving Captured Values Out of
Closures”][moving-out-of-closures]<!-- ignore --> se yaad karein ke hum closures
ko parameters ke taur par teen different traits ke saath le sakte hain: `Fn`,
`FnMut`, aur `FnOnce`. Humein decide karna hoga ke yahan kis qisam ki closure
use karni hai. Hum jaante hain ke aakhir mein hum standard library ki
`thread::spawn` implementation ke similar kuch karenge, is liye hum dekh sakte
hain ke `thread::spawn` ke signature mein uske parameter par kaun se bounds
hain. Documentation humein yeh dikhati hai:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

Yahan `F` type parameter woh hai jis mein humein dilchaspi hai; `T` type
parameter return value se related hai, aur humein us ki fikr nahi. Hum dekh
sakte hain ke `spawn` `F` par trait bound ke taur par `FnOnce` use karta hai.
Yeh shayad wohi hai jo hum bhi chahte hain, kyun ke aakhir mein hum `execute`
mein milne wale argument ko `spawn` ko pass karenge. Hum is baat par mazeed
confident ho sakte hain ke `FnOnce` woh trait hai jo humein use karna chahiye,
kyun ke request ko run karne wala thread us request ki closure ko sirf ek baar
execute karega, jo `FnOnce` mein `Once` se match karta hai.

`F` type parameter par `Send` ka trait bound aur `'static` ka lifetime bound
bhi hai, jo hamari situation mein useful hain: Humein closure ko ek thread se
doosre thread mein transfer karne ke liye `Send` ki zaroorat hai aur `'static`
is liye chahiye kyun ke humein nahi pata ke thread ko execute hone mein kitna
waqt lagega. Aao `ThreadPool` par ek `execute` method create karte hain jo in
bounds ke saath `F` type ka generic parameter lega:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-03-define-execute/src/lib.rs:here}}
```

</Listing>

Hum ab bhi `FnOnce` ke baad `()` use karte hain kyun ke yeh `FnOnce` aisi
closure ko represent karta hai jo koi parameter nahi leti aur unit type `()`
return karti hai. Bilkul function definitions ki tarah, return type ko
signature se omit kiya ja sakta hai, lekin agar hamare paas koi parameters na
bhi hon, phir bhi humein parentheses ki zaroorat hoti hai.

Dobara, yeh `execute` method ki sab se simple implementation hai: Yeh kuch
nahi karti, lekin hum sirf apne code ko compile karne ki koshish kar rahe hain.
Aao ise dobara check karte hain:

```console
{{#include ../listings/ch21-web-server/no-listing-03-define-execute/output.txt}}
```

Yeh compile ho jata hai! Lekin note karein ke agar aap `cargo run` try karein
aur browser mein request karein, to aapko browser mein wohi errors nazar aayenge
jo chapter ke shuru mein dekhe thay. Hamari library abhi tak `execute` ko pass
ki gayi closure ko actually call nahi kar rahi!

> Note: Strict compilers wali languages, jaise Haskell aur Rust, ke bare mein aap
> ek saying sun sakte hain: “If the code compiles, it works.” Lekin yeh saying
> universally true nahi hai. Hamara project compile ho jata hai, lekin yeh
> bilkul kuch nahi karta! Agar hum ek real, complete project build kar rahe
> hote, to yeh unit tests likhna shuru karne ka acha waqt hota taake check kiya
> ja sake ke code compile bhi hota hai *aur* us ka woh behavior bhi hai jo hum
> chahte hain.

Ghor karein: Agar hum closure ke bajaye kisi future ko execute karne wale hote,
to yahan kya different hota?

#### Validating the Number of Threads in `new`

Hum `new` aur `execute` ke parameters ke saath abhi kuch nahi kar rahe. Aao in
functions ke bodies ko us behavior ke saath implement karte hain jo hum chahte
hain. Shuru mein, aao `new` ke bare mein sochte hain. Pehle humne `size`
parameter ke liye ek unsigned type choose kiya tha kyun ke negative number of
threads wala pool koi maani nahi rakhta. Lekin zero threads wala pool bhi koi
maani nahi rakhta, jab ke zero ek bilkul valid `usize` hai. Hum `ThreadPool`
instance return karne se pehle yeh check karne ke liye code add karenge ke
`size` zero se greater ho, aur agar program ko zero receive ho to hum `assert!`
macro use karke program ko panic karwa denge, jaisa ke Listing 21-13 mein
dikhaya gaya hai.

<Listing number="21-13" file-name="src/lib.rs" caption="`size` zero hone par panic karne ke liye `ThreadPool::new` ko implement karna">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-13/src/lib.rs:here}}
```

</Listing>

Humne apne `ThreadPool` ke liye doc comments ke saath kuch documentation bhi
add ki hai. Note karein ke humne achi documentation practices follow karte
hue ek section add kiya hai jo un situations ko clearly batata hai jin mein
hamara function panic kar sakta hai, jaisa ke Chapter 14 mein discuss kiya gaya
hai. `cargo doc --open` run karne ki koshish karein aur `ThreadPool` struct par
click karein taake dekhein ke `new` ke liye generated docs kaise nazar aate hain!

Yahan `assert!` macro add karne ke bajaye, hum `new` ko `build` mein change kar
sakte thay aur ek `Result` return kar sakte thay, bilkul usi tarah jaisa humne
Listing 12-9 mein I/O project ke `Config::build` ke saath kiya tha. Lekin humne
is case mein decide kiya hai ke bina kisi thread ke thread pool create karne ki
koshish ek unrecoverable error honi chahiye. Agar aap ambitious feel kar rahe
hain, to `build` naam ka ek function neeche diye gaye signature ke saath likhne
ki koshish karein taake `new` function ke saath iska comparison kar saken:

```rust,ignore
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### Creating Space to Store the Threads

Ab jab hamare paas yeh jaanne ka tareeqa hai ke hamare paas pool mein store karne
ke liye threads ki valid tadaad hai, hum un threads ko create kar sakte hain aur
struct return karne se pehle unhein `ThreadPool` struct mein store kar sakte
hain. Lekin hum kisi thread ko “store” kaise karein? Aao `thread::spawn` ke
signature ko ek baar phir dekhte hain:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

`spawn` function ek `JoinHandle<T>` return karta hai, jahan `T` us type ko
represent karta hai jo closure return karti hai. Aao `JoinHandle` ko bhi use
karne ki koshish karte hain aur dekhte hain kya hota hai. Hamare case mein,
thread pool ko jo closures pass ki jayengi woh connection ko handle karengi aur
kuch return nahi karengi, is liye `T` unit type `()` hoga.

Listing 21-14 ka code compile ho jayega, lekin yeh abhi koi threads create
nahi karta. Humne `ThreadPool` ki definition ko change karke is mein
`thread::JoinHandle<()>` instances ka ek vector hold karwaya hai, vector ko
`size` ki capacity ke saath initialize kiya hai, ek `for` loop set up kiya hai
jo threads create karne ke liye kuch code run karega, aur ek `ThreadPool`
instance return kiya hai jo unhein contain karta hai.

<Listing number="21-14" file-name="src/lib.rs" caption="`ThreadPool` ke liye threads ko hold karne wala vector create karna">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-14/src/lib.rs:here}}
```

</Listing>

Humne library crate mein `std::thread` ko scope mein laya hai kyun ke hum
`ThreadPool` mein vector ke items ke type ke taur par `thread::JoinHandle` use
kar rahe hain.

Jab ek valid size receive hota hai, to hamara `ThreadPool` ek naya vector
create karta hai jo `size` items hold kar sakta hai. `with_capacity` function
`Vec::new` jaisa hi kaam karta hai lekin ek important difference ke saath:
Yeh vector mein pehle se space allocate kar deta hai. Kyun ke hum jaante hain
ke humein vector mein `size` elements store karne hain, is allocation ko shuru
mein hi kar dena `Vec::new` use karne ke muqable mein thoda zyada efficient hai,
jo elements insert hone ke saath khud resize hota rehta hai.

Jab aap dobara `cargo check` run karenge, to yeh succeed hona chahiye.

<!-- Old headings. Do not remove or links may break. -->

<a id ="a-worker-struct-responsible-for-sending-code-from-the-threadpool-to-a-thread"></a>

#### Sending Code from the `ThreadPool` to a Thread

Humne Listing 21-14 ke `for` loop mein threads create karne ke hawale se ek
comment chhoda tha. Yahan hum dekhenge ke hum asal mein threads kaise create
karte hain. Standard library threads create karne ke liye `thread::spawn`
provide karti hai, aur `thread::spawn` yeh expect karta hai ke thread create
hote hi use kuch code diya jaye jo thread ko run karna hai. Lekin hamare case
mein, hum threads create karna chahte hain aur unhein us code ke liye *wait*
karwana chahte hain jo hum baad mein bhejenge. Standard library ki threads ki
implementation mein aisa karne ka koi tareeqa shamil nahi hai; humein ise
manually implement karna hoga.

Hum `ThreadPool` aur un threads ke darmiyan ek naya data structure introduce
karke is behavior ko implement karenge jo is naye behavior ko manage karega.
Hum is data structure ko *Worker* kahenge, jo pooling implementations mein ek
common term hai. `Worker` us code ko pick karta hai jise run karna hota hai aur
apne thread mein us code ko run karta hai.

Restaurant ki kitchen mein kaam karne wale logon ke bare mein sochein: Workers
customers ki taraf se orders aane tak wait karte hain, aur phir un orders ko
lene aur unhein poora karne ke zimmedar hote hain.

Thread pool mein `JoinHandle<()>` instances ka vector store karne ke bajaye,
hum `Worker` struct ke instances store karenge. Har `Worker` ek single
`JoinHandle<()>` instance store karega. Phir hum `Worker` par ek method
implement karenge jo run kiye jane wale code ki ek closure lega aur use pehle
se running thread ko execution ke liye bhej dega. Hum har `Worker` ko ek `id`
bhi denge taake logging ya debugging ke waqt pool mein maujood different
`Worker` instances ke darmiyan farq kar saken.

Jab hum `ThreadPool` create karenge to naya process yeh hoga. Is tarah
`Worker` setup karne ke baad hum woh code implement karenge jo closure ko
thread tak bhejta hai:

1. Ek `Worker` struct define karein jo ek `id` aur ek `JoinHandle<()>` hold kare.
2. `ThreadPool` ko change karein taake woh `Worker` instances ka vector hold kare.
3. Ek `Worker::new` function define karein jo ek `id` number leta hai aur ek
   `Worker` instance return karta hai jo `id` aur ek empty closure ke saath
   spawned thread hold karta hai.
4. `ThreadPool::new` mein `for` loop counter ko use karke ek `id` generate karein,
   us `id` ke saath ek naya `Worker` create karein, aur `Worker` ko vector mein
   store karein.

Agar aap challenge ke liye tayyar hain, to Listing 21-15 ke code ko dekhne se
pehle in changes ko khud implement karne ki koshish karein.

Ready? Yeh rahi Listing 21-15, jismein upar diye gaye modifications karne ka
ek tareeqa dikhaya gaya hai.

<Listing number="21-15" file-name="src/lib.rs" caption="Directly threads hold karne ke bajaye `ThreadPool` ko `Worker` instances hold karne ke liye modify karna">

```rust,noplayground id="h1t2r3"
{{#rustdoc_include ../listings/ch21-web-server/listing-21-15/src/lib.rs:here}}
```

</Listing>

Humne `ThreadPool` ke field ka naam `threads` se badal kar `workers` kar diya
hai kyun ke ab yeh `JoinHandle<()>` instances ke bajaye `Worker` instances hold
kar raha hai. Hum `for` loop mein counter ko `Worker::new` ke argument ke
taur par use karte hain, aur har naye `Worker` ko `workers` naam ke vector mein
store karte hain.

External code (jaise *src/main.rs* mein hamara server) ko `ThreadPool` ke
andar `Worker` struct use karne ki implementation details jaanne ki zaroorat
nahi hai, is liye hum `Worker` struct aur uske `new` function ko private rakhte
hain. `Worker::new` function use diya gaya `id` use karta hai aur ek
`JoinHandle<()>` instance store karta hai jo ek empty closure ke zariye naya
thread spawn karke create kiya jata hai.

> Note: Agar operating system thread create nahi kar sakta kyun ke system
> resources kafi nahi hain, to `thread::spawn` panic karega. Is se hamara poora
> server panic kar jayega, chahe kuch threads create karne mein kamyabi hi kyun
> na hui ho. Simplicity ki khatir, yeh behavior theek hai, lekin production
> thread pool implementation mein aap shayad
> [`std::thread::Builder`][builder]<!-- ignore --> aur uske
> [`spawn`][builder-spawn]<!-- ignore --> method ko use karna chahenge jo
> `Result` return karta hai.

Yeh code compile ho jayega aur `ThreadPool::new` ko argument ke taur par
specify ki gayi tadaad ke mutabiq `Worker` instances store karega. Lekin hum
*abhi bhi* `execute` mein milne wali closure ko process nahi kar rahe. Aao ab
dekhein ke yeh kaise kiya jata hai.

#### Sending Requests to Threads via Channels

Agla problem jise hum tackle karenge yeh hai ke `thread::spawn` ko di jane wali
closures bilkul kuch nahi kartin. Filhal, humein `execute` method mein woh
closure milti hai jise hum execute karna chahte hain. Lekin jab hum
`ThreadPool` ki creation ke dauran har `Worker` create karte hain, to humein
`thread::spawn` ko run karne ke liye ek closure deni hoti hai.

Hum chahte hain ke jo `Worker` structs humne abhi create kiye hain, woh
`ThreadPool` mein rakhi hui ek queue se run kiya jane wala code fetch karein aur
us code ko apne thread ko run karne ke liye bhejein.

Chapter 16 mein jin channels ke bare mein humne seekha tha—do threads ke darmiyan
communicate karne ka ek simple tareeqa—woh is use case ke liye perfect honge.
Hum ek channel ko jobs ki queue ke taur par use karenge, aur `execute`
`ThreadPool` se ek job ko `Worker` instances ko bhejega, jo us job ko apne
thread ko bhejenge. Yeh hai plan:

1. `ThreadPool` ek channel create karega aur sender ko apne paas rakhega.
2. Har `Worker` receiver ko apne paas rakhega.
3. Hum ek naya `Job` struct create karenge jo un closures ko hold karega jinhein
   hum channel ke through bhejna chahte hain.
4. `execute` method us job ko jise woh execute karna chahta hai sender ke through
   bhejega.
5. Apne thread mein `Worker` apne receiver par loop karega aur jo bhi jobs use
   receive hongi unki closures ko execute karega.

Aao `ThreadPool::new` mein ek channel create karne aur sender ko `ThreadPool`
instance mein hold karne se shuru karte hain, jaisa ke Listing 21-16 mein
dikhaya gaya hai. `Job` struct filhal kuch hold nahi karta, lekin yeh un items
ka type hoga jinhein hum channel ke through bhejenge.

<Listing number="21-16" file-name="src/lib.rs" caption="`Job` instances ko transmit karne wale channel ke sender ko store karne ke liye `ThreadPool` ko modify karna">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-16/src/lib.rs:here}}
```

</Listing>

`ThreadPool::new` mein hum apna naya channel create karte hain aur pool sender
ko hold karta hai. Yeh successfully compile ho jayega.

Aao thread pool ke channel create karte waqt channel ka receiver har `Worker`
mein pass karne ki koshish karte hain. Hum jaante hain ke hum receiver ko us
thread mein use karna chahte hain jo `Worker` instances spawn karte hain, is
liye hum closure ke andar `receiver` parameter ko reference karenge. Listing
21-17 ka code abhi poori tarah compile nahi hoga.

<Listing number="21-17" file-name="src/lib.rs" caption="Receiver ko har `Worker` mein pass karna">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-17/src/lib.rs:here}}
```

</Listing>

Humne kuch chhoti aur straightforward changes ki hain: Hum receiver ko
`Worker::new` mein pass karte hain, aur phir use closure ke andar use karte hain.

Jab hum is code ko check karne ki koshish karte hain, to humein yeh error milta
hai:

```console
{{#include ../listings/ch21-web-server/listing-21-17/output.txt}}
```

Code `receiver` ko multiple `Worker` instances mein pass karne ki koshish kar
raha hai. Yeh kaam nahi karega, jaisa ke aapko Chapter 16 se yaad hoga: Rust
jo channel implementation provide karta hai woh multiple *producer*, single
*consumer* hai. Is ka matlab hai ke hum is code ko theek karne ke liye consuming
end of the channel ko bas clone nahi kar sakte. Hum yeh bhi nahi chahte ke ek
message ko multiple consumers ko multiple baar bheja jaye; hum messages ki ek
list chahte hain jismein multiple `Worker` instances hon, taake har message
sirf ek baar process ho.

Is ke ilawa, channel queue se ek job lene mein `receiver` ko mutate karna shamil
hai, is liye threads ko `receiver` ko safely share aur modify karne ka tareeqa
chahiye; warna humein race conditions mil sakti hain (jaisa ke Chapter 16 mein
cover kiya gaya hai).

Chapter 16 mein discuss kiye gaye thread-safe smart pointers ko yaad karein:
multiple threads ke darmiyan ownership share karne aur threads ko value mutate
karne ki permission dene ke liye humein `Arc<Mutex<T>>` use karna hoga. `Arc`
multiple `Worker` instances ko receiver own karne dega, aur `Mutex` ensure karega
ke ek waqt mein sirf ek `Worker` receiver se job le. Listing 21-18 mein woh
changes dikhaye gaye hain jo humein karne hain.

<Listing number="21-18" file-name="src/lib.rs" caption="`Arc` aur `Mutex` ka use karke `Worker` instances ke darmiyan receiver share karna">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-18/src/lib.rs:here}}
```

</Listing>

`ThreadPool::new` mein hum receiver ko ek `Arc` aur `Mutex` ke andar rakhte
hain. Har naye `Worker` ke liye hum `Arc` ko clone karte hain taake reference
count barhe aur `Worker` instances receiver ki ownership share kar saken.

In changes ke saath, code compile ho jata hai! Hum manzil ke qareeb pohanch
rahe hain!

#### Implementing the `execute` Method

Aakhir mein `ThreadPool` par `execute` method ko implement karte hain. Hum
`Job` ko bhi struct se change karke ek trait object ke liye type alias banayenge
jo us closure ke type ko hold karega jo `execute` receive karta hai. Jaisa ke
Chapter 20 ke [“Type Synonyms and Type
Aliases”][type-aliases]<!-- ignore --> section mein discuss kiya gaya hai,
type aliases humein long types ko use karne mein aasani ke liye chhota karne
dete hain. Listing 21-19 dekhein.

<Listing number="21-19" file-name="src/lib.rs" caption="Har closure ko hold karne wale `Box` ke liye `Job` type alias create karna aur phir job ko channel ke through bhejna">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-19/src/lib.rs:here}}
```

</Listing>

`execute` mein milne wali closure ko use karke ek naya `Job` instance create
karne ke baad, hum us job ko channel ke sending end ke through bhejte hain.
Agar sending fail ho jaye to hum `send` par `unwrap` call kar rahe hain. Aisa,
misal ke taur par, tab ho sakta hai jab hum apne tamam threads ko execute karne
se rok dein, jis ka matlab hoga ke receiving end ne naye messages receive karna
band kar diya hai. Filhal, hum apne threads ko execute karna band nahi kar sakte:
Jab tak pool exist karta hai, hamare threads execute karte rehte hain.
Hum `unwrap` is liye use karte hain kyun ke hum jaante hain ke failure case
nahi hoga, lekin compiler yeh nahi jaanta.

Lekin hum abhi poori tarah done nahi hue! `Worker` mein, `thread::spawn` ko di
gayi hamari closure abhi bhi channel ke receiving end ko sirf *reference* karti
hai. Is ke bajaye, humein closure ko hamesha ke liye loop karwana hai, taake
woh channel ke receiving end se job maangti rahe aur jab koi job mile to use run
kare. Aao `Worker::new` mein Listing 21-20 mein dikhaya gaya change karte hain.

<Listing number="21-20" file-name="src/lib.rs" caption="`Worker` instance ke thread mein jobs receive karna aur execute karna">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-20/src/lib.rs:here}}
```

</Listing>

Yahan, hum sab se pehle `receiver` par `lock` call karte hain taake mutex ko
acquire karein, aur phir kisi bhi errors par panic karne ke liye `unwrap` call
karte hain. Lock acquire karna fail ho sakta hai agar mutex *poisoned* state
mein ho, jo tab ho sakta hai jab koi doosra thread lock ko release karne ke
bajaye hold karte hue panic kar jaye. Is situation mein, is thread ko panic
karwane ke liye `unwrap` call karna sahi action hai. Agar aap chahein to is
`unwrap` ko aise error message ke saath `expect` mein change kar sakte hain jo
aapko meaningful lage.

Agar humein mutex ka lock mil jata hai, to hum channel se `Job` receive karne
ke liye `recv` call karte hain. Yahan ek final `unwrap` bhi errors se aage barh
jata hai, jo tab occur ho sakte hain jab sender ko hold karne wala thread shut
down ho gaya ho, bilkul usi tarah jaise receiver shut down hone par `send`
method `Err` return karta hai.

`recv` call block karti hai, is liye agar abhi koi job nahi hai to current
thread wait karega jab tak koi job available nahi ho jati. `Mutex<T>` ensure
karta hai ke ek waqt mein sirf ek `Worker` thread job request karne ki koshish
kar raha ho.

Ab hamara thread pool working state mein hai! Ise `cargo run` dein aur kuch
requests karein:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-20
cargo run
make some requests to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field `workers` is never read
 --> src/lib.rs:7:5
  |
6 | pub struct ThreadPool {
  |            ---------- field in this struct
7 |     workers: Vec<Worker>,
  |     ^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: fields `id` and `thread` are never read
  --> src/lib.rs:48:5
   |
47 | struct Worker {
   |        ------ fields in this struct
48 |     id: usize,
   |     ^^
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^
   
warning: `hello` (lib) generated 2 warnings
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 4.91s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```

Success! Ab hamare paas ek thread pool hai jo connections ko asynchronously
execute karta hai. Kabhi bhi four se zyada threads create nahi hote, is liye
agar server ko bohat zyada requests receive hon to hamara system overload nahi
hoga. Agar hum */sleep* par request karein, to server doosri requests ko kisi
doosre thread se run karke serve kar sakega.

> Note: Agar aap ek hi waqt mein multiple browser windows mein */sleep* open
> karein, to woh paanch-second intervals mein ek ek karke load ho sakti hain.
> Kuch web browsers caching reasons ki wajah se ek hi request ke multiple
> instances ko sequentially execute karte hain. Yeh limitation hamare web
> server ki wajah se nahi hai.

Ab yeh acha waqt hai ke ruk kar socha jaye ke Listings 21-18, 21-19, aur 21-20
mein code kis tarah different hota agar hum kiye jane wale work ke liye closure
ke bajaye futures use kar rahe hote. Kaun se types change hote? Method
signatures kis tarah different hotay, agar bilkul different hotay? Code ke kaun
se parts same rehte?

Chapter 17 aur Chapter 19 mein `while let` loop seekhne ke baad, aap shayad
soch rahe hon ke humne `Worker` thread ka code Listing 21-21 mein dikhaye gaye
tareeqe se kyun nahi likha.

<Listing number="21-21" file-name="src/lib.rs" caption="`while let` use karke `Worker::new` ki ek alternative implementation">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-21/src/lib.rs:here}}
```

</Listing>

Yeh code compile aur run hota hai lekin desired threading behavior produce
nahi karta: Ek slow request phir bhi doosri requests ko process hone ke liye
wait karne par majboor karegi. Is ki wajah kuch subtle hai: `Mutex` struct mein
koi public `unlock` method nahi hai kyun ke lock ki ownership us
`MutexGuard<T>` ki lifetime par based hoti hai jo `lock` method
`LockResult<MutexGuard<T>>` ke andar return karta hai. Compile time par borrow
checker phir yeh rule enforce kar sakta hai ke `Mutex` se guarded resource ko
tab tak access nahi kiya ja sakta jab tak hamare paas lock na ho. Lekin agar hum
`MutexGuard<T>` ki lifetime ka khayal na rakhein, to yeh implementation lock ko
zaroorat se zyada der tak hold karne ka sabab bhi ban sakti hai.

Listing 21-20 mein `let job =
receiver.lock().unwrap().recv().unwrap();` use karne wala code is liye kaam
karta hai kyun ke `let` ke saath, equal sign ke right-hand side wali expression
mein use hone wali tamam temporary values `let` statement ke khatam hote hi
drop ho jati hain. Lekin `while let` (aur `if let` aur `match`) associated
block ke end tak temporary values ko drop nahi karte. Listing 21-21 mein lock
`job()` call ki poori duration tak held rehta hai, jis ka matlab hai ke doosre
`Worker` instances jobs receive nahi kar sakte.

[type-aliases]: ch20-03-advanced-types.html#type-synonyms-and-type-aliases
[integer-types]: ch03-02-data-types.html#integer-types
[moving-out-of-closures]: ch13-01-closures.html#moving-captured-values-out-of-closures
[builder]: ../std/thread/struct.Builder.html
[builder-spawn]: ../std/thread/struct.Builder.html#method.spawn
