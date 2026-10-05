## Futures and the Async Syntax

Rust mein asynchronous programming ke key elements *futures* aur Rust ke `async` aur
`await` keywords hain.

Ek *future* ek aisi value hai jo abhi ready na ho sakti hai lekin future mein kisi point par
ready ho jayegi. (Yahi concept bohot si languages mein nazar aata hai, kabhi kabhi doosre
names ke under, jaise *task* ya *promise*.) Rust `Future` trait ko ek building block ke
taur par provide karta hai taake different async operations ko different data structures
ke saath implement kiya ja sake, lekin ek common interface ke saath. Rust mein, futures
woh types hain jo `Future` trait implement karti hain. Har future apni progress ke baare
mein information hold karti hai aur yeh bhi ke “ready” hone ka kya matlab hai.

Aap `async` keyword ko blocks aur functions par apply karke specify kar sakte hain ke unhein
interrupt aur resume kiya ja sakta hai. Async block ya async function ke andar, aap
`await` keyword ko *await a future* ke liye use kar sakte hain (yani us ke ready hone ka
wait karne ke liye). Async block ya function ke andar koi bhi point jahan aap future ko
await karte hain, ek potential spot hota hai jahan woh block ya function pause aur resume
ho sakta hai. Future ke saath check karne ke process ko ke us ki value ab available hai ya
nahi, *polling* kehte hain.

Kuch doosri languages, jaise C# aur JavaScript, async programming ke liye `async` aur
`await` keywords bhi use karti hain. Agar aap in languages se familiar hain, to aap notice
kar sakte hain ke Rust syntax ko handle karne mein kuch significant differences hain. Is
ki good reason hai, jaisa ke hum dekhenge!

Async Rust likhte waqt, hum zyada tar waqt `async` aur `await` keywords use karte hain. Rust
inhein `Future` trait ko use karne wale equivalent code mein compile karta hai, bilkul usi
tarah jaise yeh `for` loops ko `Iterator` trait ko use karne wale equivalent code mein
compile karta hai. Kyun ke Rust `Future` trait provide karta hai, though, jab aapko zaroorat
ho to aap ise apni data types ke liye bhi implement kar sakte hain. Bohot se functions jinhein
hum is chapter mein dekhenge, aisi types return karte hain jin ki apni `Future`
implementations hoti hain. Hum chapter ke end mein trait ki definition par wapas aayenge
aur is baat mein aur detail mein jayenge ke yeh kaise kaam karta hai, lekin abhi itni detail
hamare liye aage barhne ke liye kaafi hai.

Yeh sab thora abstract mehsoos ho sakta hai, is liye aaiye apna pehla async program likhte
hain: ek chhota sa web scraper. Hum command line se do URLs pass karenge, dono ko
concurrently fetch karenge, aur jo pehle finish hoga us ka result return karenge. Is example
mein kaafi new syntax hogi, lekin fikr na karein—jaise jaise hum aage barhenge, hum har woh
cheez explain karenge jo aapko jaanne ki zaroorat hai.

## Our First Async Program

Is chapter mein focus async seekhne par rakhne ke liye, na ke ecosystem ke different parts ko ek saath handle karne par, hum ne `trpl` crate (`trpl` “The Rust
Programming Language” ka short form hai) create kiya hai. Yeh un tamam types, traits, aur functions ko re-export karta hai jin ki aapko zaroorat hogi, zyada tar [`futures`][futures-crate]<!-- ignore --> aur
[`tokio`][tokio]<!-- ignore --> crates se. `futures` crate Rust mein async code ke liye experimentation ka official home hai, aur asal mein yahin `Future`
trait ko originally design kiya gaya tha. Tokio aaj Rust mein sab se zyada widely used async runtime hai, khaas taur par web applications ke liye. Wahan aur bhi
great runtimes maujood hain, aur mumkin hai ke woh aapke purposes ke liye zyada suitable hon. Hum `trpl` ke andar `tokio`
crate ko use karte hain kyun ke yeh well tested aur widely used hai.

Kuch cases mein, `trpl` original APIs ka naam bhi change karta hai ya unhein wrap karta hai taake aapki tawajjo un details par rahe jo is chapter se relevant hain. Agar aap samajhna chahte hain ke
crate kya karta hai, to hum aapko [is ke source code][crate-source] ko check out karne ki encourage karte hain.
Aap dekh sakenge ke har re-export kis crate se aata hai, aur hum ne extensive comments chhode hain jo explain karte hain ke crate kya karta hai.

`hello-async` naam ka ek naya binary project create karein aur `trpl` crate ko ek
dependency ke taur par add karein:

```console
$ cargo new hello-async
$ cd hello-async
$ cargo add trpl
```

Ab hum `trpl` ki taraf se provide kiye gaye different pieces ko use karke apna pehla async
program likh sakte hain. Hum ek chhota sa command line tool banayenge jo do web pages
fetch karega, dono mein se `<title>` element nikalega, aur us page ka title print karega jo
is poore process ko sab se pehle complete karega.


### Defining the page_title Function

Aaiye ek aisa function likhne se shuru karte hain jo ek page URL ko parameter ke taur par leta hai, us ke liye
request karta hai, aur `<title>` element ka text return karta hai (Listing
17-1 dekhein).

<Listing number="17-1" file-name="src/main.rs" caption="HTML page se title element hasil karne ke liye ek async function define karna">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-01/src/main.rs:all}}
```

</Listing>

Sab se pehle, hum `page_title` naam ka ek function define karte hain aur use `async`
keyword se mark karte hain. Phir hum `trpl::get` function ko use karke jo bhi URL pass
kiya gaya hai use fetch karte hain aur response ko await karne ke liye `await` keyword add
karte hain. `response` ka text hasil karne ke liye, hum us ka `text` method call karte hain
aur ek baar phir `await` keyword ke saath use await karte hain. Yeh dono steps asynchronous
hain. `get` function ke liye, humein server ke response ka pehla hissa wapas bhejne tak wait
karna hota hai, jis mein HTTP headers, cookies, waghera shamil honge aur jo response body se
alag deliver kiya ja sakta hai. Khaas taur par agar body bohot badi ho, to us ke poore
aane mein kuch waqt lag sakta hai. Kyun ke humein response ke *poore* aane ka wait karna
hota hai, `text` method bhi async hai.

Humein in dono futures ko explicitly await karna padta hai, kyun ke Rust mein futures
*lazy* hoti hain: jab tak aap `await` keyword ke saath un se aisa karne ko nahi kehte,
woh kuch nahi kartin. (Darasal, agar aap future ko use na karein to Rust compiler warning
show karega.) Yeh aapko Chapter 13 ke [“Processing a Series of
Items with Iterators”][iterators-lazy]<!-- ignore --> section mein iterators ki discussion yaad dila sakta hai.
Iterators kuch nahi kartin jab tak aap un ka `next` method call na karein—chahe directly ya
`for` loops ya `map` jaise methods ko use karke jo under the hood `next` use karte hain.
Isi tarah, futures kuch nahi kartin jab tak aap explicitly un se aisa karne ko na kahen. Yeh
laziness Rust ko async code ko us waqt tak run karne se bachane deti hai jab tak waqai is ki
zaroorat na ho.

> Note: Yeh us behavior se different hai jo hum ne Chapter 16 ke [“Creating a New Thread with spawn”][thread-spawn]<!-- ignore -->
> section mein `thread::spawn` use karte waqt dekha tha, jahan hum ne doosre thread ko jo closure
> pass kiya tha woh foran run hona shuru ho gaya tha. Yeh is baat se bhi different hai ke bohot si
> doosri languages async ko kaise approach karti hain. Lekin Rust ke liye apni
> performance guarantees provide karne ke qabil hona important hai, bilkul waise hi jaise
> iterators ke saath hai.

Jab hamare paas `response_text` aa jata hai, to hum `Html::parse` ko use karke ise `Html`
type ke ek instance mein parse kar sakte hain. Ab hamare paas raw string ke bajaye ek aisa
data type hai jise hum HTML ke saath ek richer data structure ki surat mein kaam karne ke
liye use kar sakte hain. Khaas taur par, hum `select_first` method ko use karke diye gaye CSS
selector ki pehli instance dhoond sakte hain. String `"title"` pass karne se, agar document mein
maujood ho, humein pehla `<title>` element mil jayega. Kyun ke mumkin hai ke koi matching
element na ho, `select_first` ek `Option<ElementRef>` return karta hai. Aakhir mein, hum
`Option::map` method ko use karte hain, jo humein `Option` ke andar item maujood hone ki surat
mein us ke saath kaam karne deta hai, aur agar item maujood na ho to kuch nahi karta. (Hum yahan
`match` expression bhi use kar sakte the, lekin `map` zyada idiomatic hai.) `map` ko diye gaye
function ke body mein, hum `title` par `inner_html` call karke us ka content hasil karte hain,
jo ek `String` hai. Jab sab kuch complete ho jata hai, hamare paas ek `Option<String>` hota hai.

Ghaur karein ke Rust ka `await` keyword us expression ke *baad* aata hai jise aap await kar rahe
hote hain, us se pehle nahi. Yani, yeh ek *postfix* keyword hai. Agar aap ne doosri languages mein
`async` use kiya hai to yeh aapke liye mukhtalif ho sakta hai, lekin Rust mein is se
methods ki chains ke saath kaam karna kaafi behtar ho jata hai. Is ke nateejay mein, hum
`page_title` ke body ko `trpl::get` aur `text` function calls ko chain karke aur un ke darmiyan
`await` rakh kar change kar sakte hain, jaisa ke Listing 17-2 mein dikhaya gaya hai.

<Listing number="17-2" file-name="src/main.rs" caption="`await` keyword ke saath chaining">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-02/src/main.rs:chaining}}
```

</Listing>

Is ke saath, hum ne successfully apna pehla async function likh liya hai! Is se pehle ke hum
`main` mein ise call karne ke liye kuch code add karein, aaiye jo hum ne likha hai aur is ka
kya matlab hai us ke baare mein thora aur baat karte hain.

Jab Rust `async` keyword ke saath marked ek *block* dekhta hai, to woh ise ek unique,
anonymous data type mein compile karta hai jo `Future` trait implement karta hai. Jab Rust
`async` se marked ek *function* dekhta hai, to woh ise ek non-async function mein compile
karta hai jis ka body ek async block hota hai. Async function ka return type us anonymous
data type ka type hota hai jo compiler us async block ke liye create karta hai.

Is liye, `async fn` likhna aise function ko likhne ke equivalent hai jo return type ka ek
*future* return karta hai. Compiler ke liye, Listing 17-1 mein `async fn page_title` jaisi
function definition roughly ek non-async function ke equivalent hai jo is tarah define ki
gayi ho:

```rust
# extern crate trpl; // required for mdbook test
use std::future::Future;
use trpl::Html;

fn page_title(url: &str) -> impl Future<Output = Option<String>> {
    async move {
        let text = trpl::get(url).await.text().await;
        Html::parse(&text)
            .select_first("title")
            .map(|title| title.inner_html())
    }
}
```

Aaiye transformed version ke har part ko samajhte hain:

* Is mein `impl Trait` syntax use hota hai jis par hum ne Chapter 10 mein [“Traits as Parameters”][impl-trait]<!-- ignore --> section mein baat ki thi.
* Returned value `Future` trait implement karti hai jismein `Output` ki ek associated type hoti hai. Ghaur karein ke `Output` type `Option<String>` hai, jo `page_title` ke `async fn` version ke original return type ke barabar hai.
* Original function ke body mein call kiya gaya tamam code ek `async move` block ke andar wrap hai. Yaad rakhein ke blocks expressions hotay hain. Yeh poora block function se return hone wala expression hai.
* Yeh async block `Option<String>` type ki value produce karta hai, jaisa ke abhi describe kiya gaya hai. Yeh value return type mein `Output` type se match karti hai. Yeh bilkul doosre blocks ki tarah hai jo aap ne dekhe hain.
* Naya function body ek `async move` block hai kyun ke yeh `url` parameter ko use karta hai. (Hum chapter mein baad mein `async` aur `async move` ke darmiyan farq ke baare mein kaafi zyada baat karenge.)

Ab hum `main` mein `page_title` ko call kar sakte hain.

<!-- Old headings. Do not remove or links may break. -->

<a id ="determining-a-single-pages-title"></a>

### Executing an Async Function with a Runtime

Shuru mein, hum ek single page ka title hasil karenge, jaisa ke Listing 17-3 mein dikhaya gaya hai.
Badqismati se, yeh code abhi compile nahi hota.

<Listing number="17-3" file-name="src/main.rs" caption="User ki taraf se diye gaye argument ke saath `main` se `page_title` function ko call karna">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-03/src/main.rs:main}}
```

</Listing>

Hum wohi pattern follow karte hain jo hum ne Chapter 12 ke
[“Accepting Command Line Arguments”][cli-args]<!-- ignore --> section mein command line arguments hasil karne ke liye use kiya tha.
Phir hum URL argument ko `page_title` mein pass karte hain aur result ko await karte hain.
Kyun ke future se produce hone wali value ek `Option<String>` hai, hum ek `match`
expression use karke different messages print karte hain, taake is baat ko account mein
rakha ja sake ke page mein `<title>` tha ya nahi.

Sirf async functions ya blocks mein hi hum `await` keyword use kar sakte hain,
aur Rust humein special `main` function ko `async` mark karne nahi deta.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-03
cargo build
copy just the compiler error
-->

```text
error[E0752]: `main` function is not allowed to be `async`
 --> src/main.rs:6:1
  |
6 | async fn main() {
  | ^^^^^^^^^^^^^^^ `main` function is not allowed to be `async`
```

`main` ko `async` mark na karne ki wajah yeh hai ke async code ko ek *runtime* ki
zaroorat hoti hai: ek Rust crate jo asynchronous code ko execute karne ki details ko
manage karta hai. Kisi program ka `main` function ek runtime ko *initialize* kar sakta hai,
lekin woh khud runtime *nahi* hota_. (Hum thori der mein dekhenge ke aisa kyun hai.)
Har Rust program jo async code execute karta hai, us mein kam az kam ek aisi jagah hoti hai
jahan woh ek runtime set up karta hai jo futures ko execute karta hai.

Zyada tar languages jo async ko support karti hain, ek runtime bundle karti hain, lekin Rust
aisa nahi karta. Is ke bajaye, bohot se different async runtimes available hain, jin mein
se har ek different tradeoffs karta hai jo us use case ke liye suitable hote hain jise woh
target karta hai. Misal ke taur par, bohot zyada throughput wala web server jismein bohot se
CPU cores aur RAM ki bari miktar ho, us ki needs ek aise microcontroller se bohot different
hoti hain jismein single core, RAM ki chhoti miktar, aur heap allocation ki koi ability na ho.
Jo crates in runtimes ko provide karte hain, woh aksar common functionality ke async
versions bhi provide karte hain, jaise file ya network I/O.

Yahan, aur is chapter ke baqi tamam hisson mein, hum `trpl` crate se `block_on`
function use karenge, jo ek future ko argument ke taur par leta hai aur current thread ko
tab tak block karta hai jab tak yeh future completion tak run na ho jaye. Background mein,
`block_on` ko call karne se `tokio` crate ko use karke ek runtime set up hota hai jo pass ki
gayi future ko run karta hai (`trpl` crate ka `block_on` behavior doosre runtime crates ke
`block_on` functions ke similar hai). Jab future complete ho jati hai,
`block_on` woh value return karta hai jo future ne produce ki hoti hai.

Hum `page_title` se return hone wali future ko directly `block_on` mein pass kar sakte the aur,
jab woh complete ho jati, to resulting `Option<String>` par match kar sakte the, jaisa hum ne
Listing 17-3 mein karne ki koshish ki thi. Lekin chapter ke zyada tar examples mein (aur
real world ke zyada tar async code mein), hum sirf ek async function call se zyada kaam
kar rahe honge, is liye is ke bajaye hum ek `async` block pass karenge aur `page_title` call
ke result ko explicitly await karenge, jaisa ke Listing 17-4 mein hai.

<Listing number="17-4" caption="`trpl::block_on` ke saath ek async block ko await karna" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook test does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-04/src/main.rs:run}}
```

</Listing>

Jab hum is code ko run karte hain, to humein wohi behavior milta hai jis ki hum ne shuru mein
tawaqqo ki thi:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-04
cargo build # skip all the build noise
cargo run -- "https://www.rust-lang.org"
# copy the output here
-->

```console
$ cargo run -- "https://www.rust-lang.org"
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/async_await 'https://www.rust-lang.org'`
The title for https://www.rust-lang.org was
            Rust Programming Language
```

Phew—akhirkaar hamare paas kuch working async code hai! Lekin is se pehle ke hum do sites
ko ek doosre ke muqable mein race karne wala code add karein, aaiye thori dair ke liye dobara
futures ke kaam karne ke tareeqe par tawajjo dein.

Har *await point*—yani har woh jagah jahan code `await` keyword use karta hai—ek aisi jagah
ko represent karta hai jahan control runtime ko wapas hand over kiya jata hai. Is ko kaam
karne ke liye, Rust ko async block mein shamil state ka track rakhna hota hai, taake runtime
kisi aur work ko start kar sake aur phir jab woh pehla work dobara advance karne ke liye ready
ho to wapas aa sake. Yeh ek invisible state machine hai, bilkul aisa jaise aap ne har await
point par current state ko save karne ke liye is tarah ka enum likha ho:

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-state-machine/src/lib.rs:enum}}
```

Har state ke darmiyan transition karne ke liye code manually likhna, however, tedious aur
error-prone hota, khaas taur par jab baad mein code mein aur functionality aur zyada states
add karni hon. Khush qismati se, Rust compiler async code ke liye state machine ke data
structures ko automatically create aur manage karta hai. Data structures ke around normal
borrowing aur ownership rules sab ab bhi apply hote hain, aur khushi ki baat hai ke compiler
unhein check karna bhi hamare liye handle karta hai aur useful error messages provide karta
hai. Hum chapter mein baad mein in mein se kuch par kaam karenge.

Aakhirkaar, kisi na kisi cheez ko is state machine ko execute karna hota hai, aur woh cheez
runtime hai. (Isi liye runtimes ko dekhte waqt aapko *executors* ka zikr mil sakta hai:
executor runtime ka woh hissa hai jo async code ko execute karne ka zimmedar hota hai.)

Ab aap dekh sakte hain ke compiler ne Listing 17-3 mein humein `main` ko khud ek async
function banane se kyun roka. Agar `main` ek async function hota, to kisi aur cheez ko
`main` se return hone wali future ke state machine ko manage karna padta, lekin `main`
program ka starting point hai! Is ke bajaye, hum ne `main` mein `trpl::block_on` function
call kiya taake ek runtime set up ho aur `async` block se return hone wali future ko tab tak
run kare jab tak woh complete na ho jaye.

> Note: Kuch runtimes macros provide karte hain taake aap *async `main` function* likh saken.
> Yeh macros `async fn main() { ... }` ko rewrite karke ek normal `fn
> main` bana dete hain, jo wohi kaam karta hai jo hum ne Listing 17-4 mein manually kiya:
> ek aisa function call karna jo future ko completion tak run karta hai, jis tarah
> `trpl::block_on` karta hai.

Ab aaiye in tamam pieces ko ek saath rakhte hain aur dekhte hain ke hum concurrent code
kaise likh sakte hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="racing-our-two-urls-against-each-other"></a>

### Racing Two URLs Against Each Other Concurrently

Listing 17-5 mein, hum `page_title` ko command line se pass kiye gaye do different URLs ke
saath call karte hain aur unhein race karte hain, is tarah ke jo future sab se pehle finish
hoti hai use select kar lete hain.

<Listing number="17-5" caption="Do URLs ke liye `page_title` ko call karke dekhna ke kaunsa pehle return karta hai" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-05/src/main.rs:all}}
```

</Listing>

Hum user ki taraf se diye gaye har URL ke liye `page_title` ko call karne se shuru karte hain. Hum resulting futures ko
`title_fut_1` aur `title_fut_2` ke naam se save karte hain. Yaad rakhein, abhi yeh
kuch nahi kartin, kyun ke futures lazy hoti hain aur hum ne abhi tak unhein await nahi kiya.
Phir hum futures ko `trpl::select` mein pass karte hain, jo ek value return karta hai jo
batati hai ke us ko pass ki gayi futures mein se kaunsi sab se pehle finish hoti hai.

> Note: Under the hood, `trpl::select` ek zyada general `select`
> function par built hai jo `futures` crate mein define hai. `futures` crate ka `select`
> function bohot si aisi cheezen kar sakta hai jo `trpl::select` function nahi kar sakta, lekin
> is mein kuch additional complexity bhi hai jise hum filhaal skip kar sakte hain.

Dono mein se koi bhi future legitimately “win” kar sakti hai, is liye `Result` return karna
meaningful nahi hota. Is ke bajaye, `trpl::select` ek aisa type return karta hai jo hum ne
abhi tak nahi dekha, `trpl::Either`. `Either` type kuch had tak `Result` ke similar hai, is
sense mein ke is ke do cases hote hain. Lekin `Result` ke unlike, `Either` mein success ya
failure ka koi notion built-in nahi hota. Is ke bajaye, yeh “ek ya doosra” indicate karne ke
liye `Left` aur `Right` use karta hai:

```rust
enum Either<A, B> {
    Left(A),
    Right(B),
}
```

`select` function pehli argument win karne par us future ke output ke saath `Left` return
karta hai, aur agar *doosri* future argument win kare to doosri future argument ke output ke
saath `Right` return karta hai. Yeh us order se match karta hai jis mein arguments function
ko call karte waqt appear hote hain: pehli argument doosri argument ke left mein hoti hai.

Hum `page_title` ko bhi update karte hain taake woh wahi URL return kare jo usay pass kiya
gaya tha. Is tarah, agar jo page pehle return karta hai us mein koi `<title>` na ho jise hum
resolve kar saken, to hum phir bhi ek meaningful message print kar sakte hain. Is information
ke available hone ke saath, hum apne `println!` output ko update karke yeh indicate karte
hain ke kaunsa URL pehle finish hua aur us URL par web page ka `<title>` kya hai, agar koi
hai.

Ab aap ne ek chhota sa working web scraper bana liya hai! Kuch URLs choose karein aur command
line tool run karein. Aap discover kar sakte hain ke kuch sites consistently doosri sites se
faster hoti hain, jab ke doosre cases mein faster site run se run change hoti rehti hai. Is se
bhi zyada important baat yeh hai ke aap ne futures ke saath kaam karne ki basics seekh li hain,
is liye ab hum aur gehrai mein ja sakte hain ke async ke saath hum kya kar sakte hain.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
[iterators-lazy]: ch13-02-iterators.html
[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[cli-args]: ch12-01-accepting-command-line-arguments.html

<!-- TODO: map source link version to version of Rust? -->

[crate-source]: https://github.com/rust-lang/book/tree/main/packages/trpl
[futures-crate]: https://crates.io/crates/futures
[tokio]: https://tokio.rs
