# Programming a Guessing Game

Aaiye ek hands-on project par mil kar kaam karte hue Rust mein dive karte hain! Ye chapter aapko kuch common Rust concepts se introduce karta hai, aur dikhata hai ke aap unhein ek real program mein kaise use kar sakte hain. Aap `let`, `match`, methods, associated functions, external crates, aur bohat kuch seekhenge! Agle chapters mein hum in ideas ko mazeed detail mein explore karenge. Is chapter mein aap sirf fundamentals ki practice karenge.

Hum ek classic beginner programming problem implement karenge: ek guessing game. Ye is tarah kaam karega: Program 1 aur 100 ke darmiyan ek random integer generate karega. Phir ye player ko ek guess enter karne ke liye prompt karega. Guess enter hone ke baad, program batayega ke guess bohat chhota hai ya bohat bara. Agar guess correct hua, to game ek congratulatory message print karega aur exit ho jayega.

## Setting Up a New Project

Ek naya project set up karne ke liye, Chapter 1 mein banayi gayi _projects_ directory mein jayein aur Cargo ko use karke is tarah ek naya project banayein:

```console
$ cargo new guessing_game
$ cd guessing_game
```

Pehli command, `cargo new`, project ka naam (`guessing_game`) pehle argument ke taur par leti hai. Doosri command new project ki directory mein chali jati hai.

Generated _Cargo.toml_ file ko dekhein:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial
rm -rf no-listing-01-cargo-new
cargo new no-listing-01-cargo-new --name guessing_game
cd no-listing-01-cargo-new
cargo run > output.txt 2>&1
cd ../../..
-->

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/Cargo.toml}}
```

Jaisa ke aap ne Chapter 1 mein dekha, `cargo new` aapke liye ek “Hello, world!” program generate karta hai. _src/main.rs_ file ko dekhein:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/src/main.rs}}
```

Ab aaiye is “Hello, world!” program ko compile karein aur `cargo run` command ko use karke isi step mein run karein:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/output.txt}}
```

`run` command us waqt kaam aati hai jab aapko kisi project par rapidly iterate karna ho, jaisa ke hum is game mein karenge, aur next iteration par jane se pehle har iteration ko jaldi se test karna ho.

Ab _src/main.rs_ file ko dobara open karein. Aap tamam code isi file mein likhenge.

## Processing a Guess

Guessing game program ka pehla hissa user se input mangega, us input ko process karega, aur check karega ke input expected form mein hai ya nahi. Shuru karne ke liye, hum player ko ek guess enter karne denge. _src/main.rs_ mein Listing 2-1 ka code enter karein.

<Listing number="2-1" file-name="src/main.rs" caption="Code jo user se guess leta hai aur use print karta hai">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:all}}
```

</Listing>

Is code mein bohat si information hai, is liye aaiye ise line by line dekhte hain. User input hasil karne aur phir result ko output ke taur par print karne ke liye, humein `io` input/output library ko scope mein lana hoga. `io` library standard library se aati hai, jise `std` ke naam se jana jata hai:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:io}}
```

By default, Rust mein standard library mein defined items ka ek set hota hai jise woh har program ke scope mein lata hai. Is set ko _prelude_ kaha jata hai, aur aap is mein shamil tamam cheezen [standard library documentation][prelude] mein dekh sakte hain.

Agar koi type jise aap use karna chahte hain, prelude mein nahi hai, to aapko `use` statement ke zariye us type ko explicitly scope mein lana hota hai. `std::io` library ko use karne se aapko kai useful features milte hain, jin mein user input accept karne ki ability bhi shamil hai.

Jaisa ke aap ne Chapter 1 mein dekha, `main` function program ka entry point hai:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:main}}
```

`fn` syntax ek naya function declare karta hai; parentheses, `()`, indicate karte hain ke koi parameters nahi hain; aur curly bracket, `{`, function ki body ko start karta hai.

Jaisa ke aap ne Chapter 1 mein ye bhi seekha, `println!` ek macro hai jo screen par ek string print karta hai:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print}}
```

Ye code ek prompt print kar raha hai jo batata hai ke game kya hai aur user se input maangta hai.

### Storing Values with Variables

Ab hum user ke input ko store karne ke liye ek _variable_ create karenge, is tarah:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:string}}
```

Ab program interesting ho raha hai! Is chhoti si line mein bohat kuch ho raha hai. Hum variable create karne ke liye `let` statement use karte hain. Ye ek aur example hai:

```rust,ignore
let apples = 5;
```

Ye line `apples` naam ka ek naya variable create karti hai aur ise value `5` ke saath bind karti hai. Rust mein variables by default immutable hote hain, yani ek baar hum variable ko koi value de dein, to woh value change nahi hogi. Hum is concept ko Chapter 3 ke [“Variables and Mutability”][variables-and-mutability]<!-- ignore --> section mein detail se discuss karenge. Kisi variable ko mutable banane ke liye hum variable name se pehle `mut` add karte hain:

```rust,ignore
let apples = 5; // immutable
let mut bananas = 5; // mutable
```

> Note: `//` syntax ek comment start karti hai jo line ke end tak continue hota hai. Rust comments mein mojood har cheez ko ignore karta hai. Hum [Chapter 3][comments]<!-- ignore --> mein comments ke baare mein mazeed detail se discuss karenge.

Guessing game program ki taraf wapas aate hue, ab aap jaante hain ke `let mut guess` `guess` naam ka ek mutable variable introduce karega. Equal sign (`=`) Rust ko batata hai ke hum ab variable ke saath kisi cheez ko bind karna chahte hain. Equal sign ke right side par woh value hai jiske saath `guess` bind hota hai, jo `String::new` ko call karne ka result hai, ek aisa function jo `String` ka ek naya instance return karta hai. [`String`][string]<!-- ignore --> ek string type hai jo standard library provide karti hai aur jo growable, UTF-8 encoded text hota hai.

`::new` wali line mein `::` syntax indicate karti hai ke `new`, `String` type ka ek associated function hai. Ek _associated function_ woh function hota hai jo kisi type par implement kiya jata hai, is case mein `String` par. Ye `new` function ek nayi, empty string create karta hai. Aapko bohat se types par `new` function milega kyun ke ye kisi qisam ki nayi value banane wale function ke liye ek common naam hai.

Mukammal taur par, `let mut guess = String::new();` line ne ek mutable variable create kiya hai jo filhaal `String` ke ek naye, empty instance ke saath bound hai. Uff!

### Receiving User Input

Yaad karein ke hum ne program ki pehli line mein `use std::io;` ke zariye standard library se input/output functionality include ki thi. Ab hum `io` module se `stdin` function call karenge, jo humein user input handle karne dega:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:read}}
```

Agar hum ne program ke shuru mein `use std::io;` ke zariye `io` module import na kiya hota, to hum phir bhi is function ko use kar sakte the, bas function call ko `std::io::stdin` ki tarah likhna hota. `stdin` function [`std::io::Stdin`][iostdin]<!-- ignore --> ka ek instance return karta hai, jo ek aisi type hai jo aapke terminal ke standard input ke handle ko represent karti hai.

Agli line `.read_line(&mut guess)` standard input handle par [`read_line`][read_line]<!--
ignore --> method ko call karti hai taake user se input hasil kiya ja sake. Hum `read_line` ko argument ke taur par `&mut guess` bhi pass kar rahe hain taake use bataya ja sake ke user input ko kis string mein store karna hai. `read_line` ka poora kaam ye hai ke user standard input mein jo kuch type karta hai, use le kar ek string mein append kar de (uske contents ko overwrite kiye baghair), is liye hum us string ko argument ke taur par pass karte hain. String argument ka mutable hona zaroori hai taake method string ke contents ko change kar sake.

`&` indicate karta hai ke ye argument ek *reference* hai, jo aapko apne code ke multiple parts ko ek hi data tak access dene ka tareeqa deta hai, baghair is data ko memory mein multiple times copy kiye. References ek complex feature hain, aur Rust ke major advantages mein se ek ye hai ke references ko use karna safe aur easy hai. Is program ko complete karne ke liye aapko in tamam details ke baare mein zyada jaanne ki zaroorat nahi hai. Filhaal aapko sirf itna maloom hona chahiye ke variables ki tarah, references bhi by default immutable hote hain. Isi liye, ise mutable banane ke liye aapko `&guess` ke bajaye `&mut guess` likhna hota hai. (Chapter 4 references ko mazeed thoroughly explain karega.)

<!-- Old headings. Do not remove or links may break. -->

<a id="handling-potential-failure-with-the-result-type"></a>

### Handling Potential Failure with `Result`

Hum abhi bhi isi code ki line par kaam kar rahe hain. Ab hum text ki teesri line discuss kar rahe hain, lekin note karein ke ye abhi bhi code ki ek hi logical line ka hissa hai. Agla part ye method hai:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:expect}}
```

Hum is code ko is tarah bhi likh sakte the:

```rust,ignore
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

Lekin ek lambi line ko read karna mushkil hota hai, is liye ise divide karna behtar hai. Jab aap `.method_name()` syntax ke zariye kisi method ko call karte hain, to lambi lines ko break up karne ke liye newline aur doosri whitespace introduce karna aksar behtar hota hai. Ab aaiye discuss karte hain ke ye line kya karti hai.

Jaisa ke pehle mention kiya gaya, `read_line` user jo kuch enter karta hai use us string mein daal deta hai jo hum use pass karte hain, lekin ye ek `Result` value bhi return karta hai. [`Result`][result]<!--
ignore --> ek [*enumeration*][enums]<!-- ignore --> hai, jise aksar *enum* kaha jata hai, jo ek aisi type hai jo multiple possible states mein se kisi ek state mein ho sakti hai. Hum har possible state ko ek *variant* kehte hain.

[Chapter 6][enums]<!-- ignore --> mein enums ko mazeed detail mein cover kiya jayega. In `Result` types ka maqsad error-handling ki information ko encode karna hai.

`Result` ke variants `Ok` aur `Err` hain. `Ok` variant indicate karta hai ke operation successful tha, aur is mein successfully generated value hoti hai. `Err` variant ka matlab hai ke operation fail ho gaya, aur is mein information hoti hai ke operation kis tarah ya kyun fail hua.

`Result` type ki values, doosri tamam types ki values ki tarah, un par defined methods rakhti hain. `Result` ka ek instance [`expect method`][expect]<!-- ignore --> rakhta hai jise aap call kar sakte hain. Agar ye `Result` instance ek `Err` value hai, to `expect` program ko crash kar dega aur woh message display karega jo aap ne `expect` ko argument ke taur par pass kiya tha. Agar `read_line` method ek `Err` return karti hai, to ye mumkin hai ke ye underlying operating system se aane wali kisi error ka result ho. Agar ye `Result` instance ek `Ok` value hai, to `expect` woh return value le lega jo `Ok` ke andar hai aur sirf woh value aapko return karega taake aap use kar saken. Is case mein, woh value user ke input mein bytes ki tadaad hai.

Agar aap `expect` call nahi karte, to program compile ho jayega, lekin aapko ek warning milegi:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-02-without-expect/output.txt}}
```

Rust warning deta hai ke aap ne `read_line` se return hone wali `Result` value ko use nahi kiya, jo indicate karta hai ke program ne possible error ko handle nahi kiya.

Warning ko suppress karne ka sahi tareeqa ye hai ke asal mein error-handling code likha jaye, lekin hamare case mein hum sirf ye chahte hain ke jab koi problem ho to ye program crash ho jaye, is liye hum `expect` use kar sakte hain. Errors se recover karne ke baare mein aap [Chapter
9][recover]<!-- ignore --> mein seekhenge.

### Printing Values with `println!` Placeholders

Closing curly bracket ke ilawa, ab tak ke code mein sirf ek aur line discuss karni baqi hai:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print_guess}}
```

Ye line us string ko print karti hai jisme ab user ka input mojood hai. `{}` curly brackets ka set ek placeholder hai: `{}` ko chhote crab pincers samjhein jo kisi value ko apni jagah par hold karte hain. Jab kisi variable ki value print karni ho, to variable ka naam curly brackets ke andar rakha ja sakta hai. Jab kisi expression ko evaluate karne ka result print karna ho, to format string mein empty curly brackets rakhein, phir format string ke baad comma-separated expressions ki list dein, jo isi order mein har empty curly bracket placeholder mein print ki jayengi. Ek hi `println!` call mein variable aur kisi expression ka result print karna is tarah hoga:

```rust
let x = 5;
let y = 10;

println!("x = {x} and y + 2 = {}", y + 2);
```

Ye code `x = 5 and y + 2 = 12` print karega.

### Testing the First Part

Aaiye guessing game ke pehle part ko test karte hain. Ise `cargo run` ke zariye run karein:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-01/
cargo clean
cargo run
input 6 -->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 6.44s
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

Is point par game ka pehla part complete ho gaya hai: Hum keyboard se input le rahe hain aur phir use print kar rahe hain.


## Generating a Secret Number

Ab humein ek secret number generate karna hai jise user guess karne ki koshish karega. Secret number har baar different hona chahiye taake game ko ek se zyada baar khelna mazedar rahe. Hum 1 aur 100 ke darmiyan ek random number use karenge taake game bohat mushkil na ho. Rust ki standard library mein abhi random number generate karne ki functionality shamil nahi hai. Lekin, Rust team [`rand` crate][randcrate] provide karti hai jisme ye functionality mojood hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-a-crate-to-get-more-functionality"></a>


### Increasing Functionality with a Crate

Yaad rakhein ke crate Rust source code files ka ek collection hota hai. Jo project hum build kar rahe hain woh ek binary crate hai, yani ek executable. `rand` crate ek library crate hai, jisme aisa code hota hai jo doosre programs mein use kiye jane ke liye hota hai aur ise apne taur par execute nahi kiya ja sakta.

External crates ko Cargo jis tarah coordinate karta hai, wahi woh jagah hai jahan Cargo waqai apni strength dikhata hai. Is se pehle ke hum `rand` ko use karne wala code likh saken, humein *Cargo.toml* file ko modify karke `rand` crate ko dependency ke taur par include karna hoga. Ab woh file open karein aur Cargo ki taraf se aapke liye banaye gaye `[dependencies]` section header ke neeche, bottom mein following line add karein. `rand` ko bilkul isi tarah specify karna zaroori hai, isi version number ke saath, warna is tutorial ke code examples kaam nahi kar sakte:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:

* ch01-01-installation.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:8:}}
```

*Cargo.toml* file mein, kisi header ke baad aane wali har cheez us section ka hissa hoti hai, jo tab tak continue hota hai jab tak koi doosra section start na ho. `[dependencies]` mein aap Cargo ko batate hain ke aapka project kin external crates par depend karta hai aur aapko un crates ke kaun se versions chahiye. Is case mein, hum `rand` crate ko semantic version specifier `0.10.1` ke saath specify karte hain. Cargo [Semantic
Versioning][semver]<!-- ignore --> ko samajhta hai (jise kabhi kabhi *SemVer* bhi kaha jata hai), jo version numbers likhne ka ek standard hai. Specifier `0.10.1` asal mein `^0.10.1` ka shorthand hai, jis ka matlab hai koi bhi version jo kam az kam 0.10.1 ho lekin 0.11.0 se neeche ho.

Cargo in versions ko version 0.10.1 ke saath public APIs ke hawale se compatible samajhta hai, aur ye specification ensure karti hai ke aapko latest patch release mile jo is chapter ke code ke saath phir bhi compile ho. Version 0.11.0 ya us se greater kisi bhi version ke baare mein ye guarantee nahi hai ke us ka API wohi hoga jo following examples use karte hain.

Ab, kisi bhi code ko change kiye baghair, aaiye project ko build karte hain, jaisa ke Listing 2-2 mein dikhaya gaya hai.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
rm Cargo.lock
cargo clean
cargo build -->

<Listing number="2-2" caption="The output from running `cargo build` after adding the `rand` crate as a dependency">

```console
$ cargo build
    Updating crates.io index
     Locking 8 packages to latest Rust 1.96.0 compatible versions
  Downloaded rand_core v0.10.1
  Downloaded chacha20 v0.10.1
  Downloaded rand v0.10.1
  Downloaded 3 crates (162.9KiB) in 0.59s
   Compiling libc v0.2.186
   Compiling rand_core v0.10.1
   Compiling getrandom v0.4.3
   Compiling cfg-if v1.0.4
   Compiling chacha20 v0.10.1
   Compiling rand v0.10.1
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 2.03s
```

</Listing>

Aapko different version numbers nazar aa sakte hain (lekin SemVer ki wajah se woh sab code ke saath compatible honge!) aur different lines bhi nazar aa sakti hain (jo operating system par depend karti hain), aur lines ka order bhi different ho sakta hai.

Jab hum koi external dependency include karte hain, Cargo us dependency ko jis cheez ki zaroorat hoti hai us ke latest versions ko *registry* se fetch karta hai, jo [Crates.io][cratesio] ke data ki ek copy hai. Crates.io woh jagah hai jahan Rust ecosystem ke log apne open source Rust projects doosron ke use ke liye post karte hain.

Registry update karne ke baad, Cargo `[dependencies]` section ko check karta hai aur un tamam listed crates ko download karta hai jo abhi tak download nahi hue. Is case mein, hum ne sirf `rand` ko dependency ke taur par list kiya, lekin Cargo ne doosre crates bhi download kiye jin par `rand` kaam karne ke liye depend karta hai. Crates download karne ke baad, Rust unhein compile karta hai aur phir available dependencies ke saath project ko compile karta hai.

Agar aap foran dobara `cargo build` run karein aur koi change na karein, to aapko `Finished` line ke ilawa koi output nahi milega. Cargo jaanta hai ke us ne dependencies ko pehle hi download aur compile kar liya hai, aur aap ne apni *Cargo.toml* file mein unke baare mein kuch change nahi kiya. Cargo ye bhi jaanta hai ke aap ne apne code mein koi change nahi kiya, is liye woh use dobara compile nahi karta. Kuch karne ko na hone ki wajah se, woh simply exit ho jata hai.

Agar aap *src/main.rs* file open karein, ek chhoti si change karein, phir use save karke dobara build karein, to aapko sirf do lines ka output nazar aayega:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
touch src/main.rs
cargo build -->

```console
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
```

Ye lines show karti hain ke Cargo sirf *src/main.rs* file mein aapki chhoti si change ke saath build ko update karta hai. Aapki dependencies change nahi hui hain, is liye Cargo jaanta hai ke jo kuch us ne pehle hi download aur compile kiya hai, woh dobara use kiya ja sakta hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="ensuring-reproducible-builds-with-the-cargo-lock-file"></a>

#### Ensuring Reproducible Builds

Cargo ke paas ek aisa mechanism hai jo ye ensure karta hai ke jab bhi aap ya koi aur aapke code ko build kare, to aap har baar wahi artifact dobara build kar saken: Cargo sirf unhi dependency versions ko use karega jinhein aap ne specify kiya hai, jab tak aap khud kuch aur indicate na karein. Misal ke taur par, maan lein ke aglay haftay `rand` crate ka version 0.10.2 release hota hai, aur is version mein ek important bug fix hai, lekin saath hi ek regression bhi hai jo aapke code ko break kar dega. Is situation ko handle karne ke liye, Rust pehli baar `cargo build` run karte waqt *Cargo.lock* file create karta hai, is liye ab hamare paas *guessing_game* directory mein ye file mojood hai.

Jab aap pehli baar project build karte hain, Cargo dependencies ke un tamam versions ka pata lagata hai jo criteria par poore utarte hain aur phir unhein *Cargo.lock* file mein likh deta hai. Jab aap future mein apna project build karenge, Cargo dekhega ke *Cargo.lock* file mojood hai aur versions ka dobara pata lagane ka tamam kaam karne ke bajaye us mein specify kiye gaye versions ko use karega. Is se aapke paas automatically ek reproducible build hota hai. Doosre alfaaz mein, *Cargo.lock* file ki wajah se aapka project 0.10.1 par hi rahega jab tak aap khud explicitly upgrade nahi karte. Kyun ke *Cargo.lock* file reproducible builds ke liye important hai, is liye ise aksar aapke project ke baqi code ke saath source control mein bhi check kiya jata hai.


#### Updating a Crate to Get a New Version

Jab aap *waqai* kisi crate ko update karna chahein, to Cargo `update` command provide karta hai, jo *Cargo.lock* file ko ignore karega aur *Cargo.toml* mein aapki specifications ke mutabiq tamam latest versions ka pata lagayega. Phir Cargo un versions ko *Cargo.lock* file mein likh dega. Is ke ilawa, by default, Cargo sirf un versions ko dekhega jo 0.10.1 se greater aur 0.11.0 se less hon. Agar `rand` crate ne do naye versions 0.10.2 aur 0.999.0 release kiye hon, to agar aap `cargo update` run karein to aapko following output nazar aayega:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
cargo update
assuming there is a new version of rand; otherwise use another update
as a guide to creating the hypothetical output shown here -->

```console
$ cargo update
    Updating crates.io index
     Locking 1 package to latest Rust 1.96.0 compatible version
    Updating rand v0.10.1 -> v0.10.2 (available: v0.999.0)
```

Cargo 0.999.0 release ko ignore karta hai. Is point par, aap apni *Cargo.lock* file mein bhi ek change notice karenge, jo indicate karta hai ke ab aap `rand` crate ka version 0.10.2 use kar rahe hain. `rand` version 0.999.0 ya 0.999.*x* series ka koi bhi version use karne ke liye, aapko *Cargo.toml* file ko is tarah update karna hoga (asal mein ye change na karein kyun ke following examples assume karte hain ke aap `rand` 0.10 use kar rahe hain):

```toml
[dependencies]
rand = "0.999.0"
```

Agli baar jab aap `cargo build` run karenge, Cargo available crates ki registry ko update karega aur aapki specify ki hui nayi version ke mutabiq aapki `rand` requirements ka dobara jaiza lega.

[Cargo][doccargo]<!-- ignore --> aur [its
ecosystem][doccratesio]<!-- ignore --> ke baare mein kehne ko bohat kuch aur hai, jise hum Chapter 14 mein discuss karenge, lekin filhaal aapko itna hi jaanne ki zaroorat hai. Cargo libraries ko reuse karna bohat aasaan bana deta hai, is liye Rustaceans aise chhote projects likh sakte hain jo multiple packages se mil kar assemble kiye jate hain.


### Generating a Random Number

Aaiye `rand` ko use karke woh number generate karna shuru karte hain jise guess kiya jayega. Agla step *src/main.rs* ko update karna hai, jaisa ke Listing 2-3 mein dikhaya gaya hai.

<Listing number="2-3" file-name="src/main.rs" caption="Random number generate karne ke liye code add karna">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:all}}
```

</Listing>

Sab se pehle, hum line `use rand::prelude::*;` add karte hain. `prelude` module mein `rand` crate ke sab se zyada commonly used parts shamil hote hain, aur `use` un items ko hamare program ke scope mein available kar deta hai.

Is ke baad, hum darmiyan mein do lines add kar rahe hain. Pehli line mein, hum `rand::rng` function call karte hain jo humein woh particular random number generator deta hai jise hum use karne wale hain: ek aisa generator jo current thread of execution ke liye local hota hai aur operating system ke zariye seeded hota hai. Phir, hum random number generator par `random_range` method call karte hain. Ye method `RngExt` trait ke zariye define kiya gaya hai jo `rand::prelude` module ka hissa hai, jise hum `use
rand::prelude::*;` statement ke zariye scope mein laaye hain. `random_range` method ek range expression ko argument ke taur par leti hai aur us range mein ek random number generate karti hai. Yahan hum jis qisam ka range expression use kar rahe hain woh `start..=end` ki form mein hota hai aur lower aur upper dono bounds ko include karta hai, is liye humein 1 aur 100 ke darmiyan number mangne ke liye `1..=100` specify karna hoga.

> Note: Aap khud se ye nahi jaan sakenge ke kisi crate se kya cheez scope mein lani hai aur kaun se methods aur functions call karne hain, is liye har crate ke saath uske use karne ki instructions wali documentation hoti hai. Cargo ka ek aur neat feature ye hai ke `cargo doc --open` command run karne se aapki tamam dependencies ki provide ki hui documentation locally build ho jayegi aur browser mein open ho jayegi. Agar aap `rand` crate ki doosri functionality mein interested hain, to misal ke taur par `cargo doc --open` run karein aur left side par sidebar mein `rand` par click karein.

Doosri nayi line secret number ko print karti hai. Program develop karte waqt ye useful hai kyun ke is se hum ise test kar sakte hain, lekin final version mein hum ise delete kar denge. Agar program start hote hi answer print kar de to ye zyada game nahi rahega!

Program ko kuch baar run karke dekhein:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-03/
cargo run
4
cargo run
5
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4

$ cargo run
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

Aapko different random numbers milne chahiye, aur woh tamam 1 aur 100 ke darmiyan hone chahiye. Agar aapko warnings milti hain, to unhein ignore karna safe hai. Agar aapko errors milte hain, to check karein ke aapke *Cargo.toml* mein `rand = "0.10.1"` mojood hai, kyun ke `rand` ke future versions ka API different ho sakta hai, lekin `0.10` series ka koi bhi version is chapter ke code ke saath kaam karna chahiye.


## Comparing the Guess to the Secret Number

Ab jab hamare paas user input aur ek random number hai, to hum in dono ka comparison kar sakte hain. Ye step Listing 2-4 mein dikhaya gaya hai. Note karein ke ye code abhi compile nahi hoga, jaisa ke hum explain karenge.

<Listing number="2-4" file-name="src/main.rs" caption="Do numbers ko compare karne ke possible return values ko handle karne wala code">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-04/src/main.rs:here}}
```

</Listing>

Sab se pehle, hum ek aur `use` statement add karte hain, jo standard library se `std::cmp::Ordering` naam ki type ko scope mein lata hai. `Ordering` type bhi ek enum hai aur iske variants `Less`, `Greater`, aur `Equal` hain. Ye teen possible outcomes hain jo do values ko compare karne par hasil ho sakte hain.

Phir, hum bottom par paanch nayi lines add karte hain jo `Ordering` type ko use karti hain. `cmp` method do values ko compare karta hai aur ise kisi bhi aisi cheez par call kiya ja sakta hai jise compare kiya ja sakta ho. Ye us cheez ka reference leta hai jiske saath aap compare karna chahte hain: yahan ye `guess` ko `secret_number` ke saath compare kar raha hai. Phir, ye `Ordering` enum ka ek variant return karta hai jise hum `use` statement ke zariye scope mein laaye hain. Hum ek [`match`][match]<!-- ignore --> expression use karte hain taake `guess` aur `secret_number` ki values ke saath `cmp` call se jo `Ordering` variant return hua hai, uski bunyaad par decide kar saken ke agay kya karna hai.

Ek `match` expression *arms* se mil kar banta hai. Ek arm mein ek *pattern* hota hai jiske saath match karna hota hai, aur woh code hota hai jo us waqt run hona chahiye jab `match` ko di gayi value us arm ke pattern ke saath match kare. Rust `match` ko di gayi value leta hai aur baari baari har arm ke pattern ko check karta hai. Patterns aur `match` construct Rust ke powerful features hain: Ye aapko mukhtalif situations express karne dete hain jin ka aapka code saamna kar sakta hai, aur ye ensure karte hain ke aap un tamam situations ko handle karein. In features ko Chapter 6 aur Chapter 19 mein respectively detail se cover kiya jayega.

Aaiye yahan use kiye gaye `match` expression ke saath ek example ko step by step dekhte hain. Maan lein ke user ne 50 guess kiya hai aur is baar randomly generated secret number 38 hai.

Jab code 50 ko 38 ke saath compare karta hai, to `cmp` method `Ordering::Greater` return karega kyun ke 50, 38 se greater hai. `match` expression ko `Ordering::Greater` value milti hai aur woh har arm ke pattern ko check karna shuru karta hai. Ye pehle arm ke pattern, `Ordering::Less`, ko dekhta hai aur samajhta hai ke `Ordering::Greater` value `Ordering::Less` ke saath match nahi karti, is liye woh us arm ke code ko ignore karta hai aur next arm par chala jata hai. Agle arm ka pattern `Ordering::Greater` hai, jo `Ordering::Greater` ke saath *match* karta hai! Is arm ka associated code execute hoga aur screen par `Too big!` print karega. `match` expression pehle successful match ke baad khatam ho jata hai, is liye is situation mein woh last arm ko check nahi karega.

Lekin Listing 2-4 ka code abhi compile nahi hoga. Aaiye ise try karte hain:

<!--
The error numbers in this output should be that of the code **WITHOUT** the
anchor or snip comments
-->

```console
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-04/output.txt}}
```

Error ka core ye hai ke *mismatched types* hain. Rust ke paas ek strong, static type system hai. Lekin is mein type inference bhi hai. Jab hum ne `let mut guess = String::new()` likha, to Rust ye infer kar saka ke `guess` ek `String` hona chahiye aur humein type likhne ki zaroorat nahi padi. Doosri taraf, `secret_number` ek number type hai. Rust ki kuch number types 1 aur 100 ke darmiyan value rakh sakti hain: `i32`, jo 32-bit number hai; `u32`, jo unsigned 32-bit number hai; `i64`, jo 64-bit number hai; aur doosri types bhi. Agar otherwise specify na kiya jaye, to Rust default taur par `i32` use karta hai, jo `secret_number` ki type hai jab tak aap kahin aur type information add na karein jo Rust ko koi different numerical type infer karne par majboor kare. Error ki wajah ye hai ke Rust ek string aur number type ko compare nahi kar sakta.

Aakhir mein, hum `String` ko, jo program input ke taur par read karta hai, ek number type mein convert karna chahte hain taake hum uska secret number ke saath numerical comparison kar saken. Hum `main` function ki body mein ye line add karke aisa karte hain:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/src/main.rs:here}}
```

Line ye hai:

```rust,ignore
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

Hum `guess` naam ka ek variable create karte hain. Lekin rukiye, kya program mein pehle se `guess` naam ka variable nahi hai? Hai, lekin Rust humein pichli `guess` value ko ek nayi value ke saath shadow karne ki ijazat deta hai. *Shadowing* humein `guess` variable name ko dobara use karne deta hai, bajaye is ke ke hum do unique variables create karne par majboor hon, jaise `guess_str` aur `guess`, misal ke taur par. Hum isay [Chapter 3][shadowing]<!-- ignore --> mein mazeed detail se cover karenge, lekin filhaal itna jaan lein ke ye feature aksar us waqt use hota hai jab aap kisi value ko ek type se doosri type mein convert karna chahte hain.

Hum is naye variable ko expression `guess.trim().parse()` ke saath bind karte hain. Expression mein `guess` original `guess` variable ko refer karta hai jisme input string ke taur par mojood tha. `String` instance par `trim` method shuru aur aakhir mein mojood kisi bhi whitespace ko eliminate kar deta hai, jo humein string ko `u32` mein convert karne se pehle karna zaroori hai, kyun ke `u32` mein sirf numerical data ho sakta hai. User ko apna guess input karne ke liye <kbd>enter</kbd> press karna zaroori hai taake `read_line` satisfy ho, aur is se string mein ek newline character add ho jata hai. Misal ke taur par, agar user <kbd>5</kbd> type karke <kbd>enter</kbd> press karta hai, to `guess` is tarah nazar aata hai: `5\n`. `\n` “newline” ko represent karta hai. (Windows par, <kbd>enter</kbd> press karne se carriage return aur newline, `\r\n`, result hota hai.) `trim` method `\n` ya `\r\n` ko eliminate kar deta hai, jis ka result sirf `5` hota hai.

Strings par [`parse method`][parse]<!-- ignore --> ek string ko kisi doosri type mein convert karta hai. Yahan, hum ise string ko number mein convert karne ke liye use karte hain. Humein `let guess: u32` use karke Rust ko exact number type batani hoti hai jo hum chahte hain. `guess` ke baad colon (`:`) Rust ko batata hai ke hum variable ki type annotate karenge. Rust mein kuch built-in number types hain; yahan nazar aane wala `u32` ek unsigned, 32-bit integer hai. Chhoti positive number ke liye ye ek achha default choice hai. Aap [Chapter 3][integers]<!-- ignore --> mein doosri number types ke baare mein seekhenge.

Is ke ilawa, is example program mein `u32` annotation aur `secret_number` ke saath comparison ka matlab hai ke Rust infer kar lega ke `secret_number` bhi `u32` hona chahiye. To ab comparison ek hi type ki do values ke darmiyan hoga!

`parse` method sirf un characters par kaam karega jinhein logically numbers mein convert kiya ja sakta hai aur is liye ye aasani se errors cause kar sakta hai. Misal ke taur par, agar string mein `A👍%` hota, to use number mein convert karne ka koi tareeqa nahi hota. Kyun ke ye fail ho sakta hai, `parse` method ek `Result` type return karta hai, bilkul `read_line` method ki tarah, jise pehle [“Handling Potential Failure with
`Result`”](#handling-potential-failure-with-result)<!-- ignore --> mein discuss kiya gaya tha. Hum is `Result` ko bhi `expect` method use karke usi tarah handle karenge. Agar `parse` ek `Err` `Result` variant return karta hai kyun ke woh string se number create nahi kar saka, to `expect` call game ko crash kar dega aur woh message print karega jo hum ne use diya hai. Agar `parse` string ko successfully number mein convert kar sakta hai, to ye `Result` ka `Ok` variant return karega, aur `expect` `Ok` value se woh number return karega jo humein chahiye.

Ab aaiye program ko run karte hain:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/
touch src/main.rs
cargo run
  76
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.26s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

Nice! Guess se pehle spaces add hone ke bawajood, program ne phir bhi sahi tarah figure out kar liya ke user ne 76 guess kiya tha. Different kinds of input ke saath different behavior verify karne ke liye program ko kuch baar run karein: Number ko correctly guess karein, aisa number guess karein jo bohat high ho, aur aisa number guess karein jo bohat low ho.

Ab game ka zyada tar hissa kaam kar raha hai, lekin user sirf ek guess kar sakta hai. Aaiye ek loop add karke isay change karte hain!


## Allowing Multiple Guesses with Looping

`loop` keyword ek infinite loop create karta hai. Hum users ko number guess karne ke mazeed chances dene ke liye ek loop add karenge:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-04-looping/src/main.rs:here}}
```

Jaisa ke aap dekh sakte hain, hum ne guess input prompt se onward sab kuch ek loop ke andar move kar diya hai. Loop ke andar wali lines ko mazeed chaar spaces se indent karna zaroori hai aur phir program ko dobara run karein. Ab program hamesha ek aur guess maangta rahega, jo asal mein ek naya problem introduce karta hai. Aisa lagta hai ke user quit hi nahi kar sakta!

User keyboard shortcut <kbd>ctrl</kbd>-<kbd>C</kbd> use karke hamesha program ko interrupt kar sakta hai. Lekin is insatiable monster se bachne ka ek aur tareeqa hai, jaisa ke [“Comparing the Guess to the Secret Number”](#comparing-the-guess-to-the-secret-number)<!-- ignore --> mein `parse` ki discussion ke dauran mention kiya gaya tha: Agar user non-number answer enter kare, to program crash ho jayega. Hum is cheez ka faida utha kar user ko quit karne ki permission de sakte hain, jaisa ke yahan dikhaya gaya hai:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-04-looping/
touch src/main.rs
cargo run
(too small guess)
(too big guess)
(correct guess)
quit
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.23s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit

thread 'main' (6694925) panicked at src/main.rs:28:47:
Please type a number!: ParseIntError { kind: InvalidDigit }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

`quit` type karne se game quit ho jayega, lekin jaisa ke aap notice karenge, kisi bhi doosre non-number input ko enter karne se bhi aisa hi hoga. Kam az kam kehne ke liye, ye ideal nahi hai; hum chahte hain ke jab correct number guess ho jaye to game bhi ruk jaye.


### Quitting After a Correct Guess

Aaiye `break` statement add karke game ko is tarah program karte hain ke user ke jeetne par game quit ho jaye:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-05-quitting/src/main.rs:here}}
```

`You win!` ke baad `break` line add karne se, jab user secret number ko sahi guess karta hai to program loop se exit kar jata hai. Loop se exit karne ka matlab program se exit karna bhi hai, kyun ke loop `main` ka aakhri hissa hai.

### Handling Invalid Input

Game ke behavior ko mazeed refine karne ke liye, jab user non-number input enter kare to program ko crash karne ke bajaye, aaiye game ko non-number ko ignore karne dein taa-ke user guessing jaari rakh sake. Hum ye us line ko alter karke kar sakte hain jahan `guess` ko `String` se `u32` mein convert kiya jata hai, jaisa ke Listing 2-5 mein dikhaya gaya hai.

<Listing number="2-5" file-name="src/main.rs" caption="Non-number guess ko ignore karna aur program ko crash karne ke bajaye doosra guess maangna">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:here}}
```

</Listing>

Hum `expect` call se `match` expression par switch karte hain taa-ke error par crash karne ke bajaye error ko handle kiya ja sake. Yaad rakhein ke `parse` ek `Result` type return karta hai aur `Result` ek enum hai jisme `Ok` aur `Err` variants hote hain. Hum yahan `match` expression use kar rahe hain, bilkul usi tarah jaise hum ne `cmp` method ke `Ordering` result ke saath kiya tha.

Agar `parse` string ko successfully number mein convert kar sake, to ye ek `Ok` value return karega jisme resultant number hoga. Ye `Ok` value pehle arm ke pattern se match karegi, aur `match` expression sirf woh `num` value return karega jo `parse` ne produce ki aur `Ok` value ke andar rakhi. Woh number seedha us jagah aa jayega jahan humein chahiye, yani naye `guess` variable mein jo hum create kar rahe hain.

Agar `parse` string ko number mein convert *nahi* kar sakta, to ye ek `Err` value return karega jisme error ke baare mein mazeed information hogi. `Err` value pehle `match` arm ke `Ok(num)` pattern se match nahi karti, lekin doosre arm ke `Err(_)` pattern se match karti hai. Underscore, `_`, ek catch-all value hai; is example mein hum keh rahe hain ke hum tamam `Err` values se match karna chahte hain, chahe unke andar koi bhi information ho. Is liye program doosre arm ka code, `continue`, execute karega, jo program ko `loop` ki next iteration par jane aur doosra guess maangne ke liye kehta hai. Is tarah, effectively, program un tamam errors ko ignore kar deta hai jin ka `parse` ko saamna ho sakta hai!

Ab program mein sab kuch expected tareeqe se kaam karna chahiye. Aaiye ise try karte hain:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-05/
cargo run
(too small guess)
(too big guess)
foo
(correct guess)
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

Awesome! Sirf ek chhoti si final tweak ke saath hum guessing game complete kar denge. Yaad karein ke program abhi bhi secret number print kar raha hai. Testing ke liye ye achha tha, lekin is se game ka maza kharab ho jata hai. Aaiye secret number output karne wale `println!` ko delete kar dein. Listing 2-6 final code dikhati hai.

<Listing number="2-6" file-name="src/main.rs" caption="Mukammal guessing game ka code">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-06/src/main.rs}}
```

</Listing>

Is point par, aap ne successfully guessing game build kar liya hai. Mubarak ho!

## Summary

Ye project ek hands-on tareeqa tha jis ke zariye aapko Rust ke bohat se naye concepts se introduce kiya gaya: `let`, `match`, functions, external crates ka use, aur bohat kuch. Agle kuch chapters mein aap in concepts ke baare mein mazeed detail se seekhenge. Chapter 3 un concepts ko cover karta hai jo zyada tar programming languages mein mojood hote hain, jaise variables, data types, aur functions, aur dikhata hai ke inhein Rust mein kaise use kiya jata hai. Chapter 4 ownership ko explore karta hai, jo ek aisa feature hai jo Rust ko doosri languages se different banata hai. Chapter 5 structs aur method syntax par baat karta hai, aur Chapter 6 explain karta hai ke enums kaise kaam karte hain.

[prelude]: ../std/prelude/index.html
[variables-and-mutability]: ch03-01-variables-and-mutability.html#variables-and-mutability
[comments]: ch03-04-comments.html
[string]: ../std/string/struct.String.html
[iostdin]: ../std/io/struct.Stdin.html
[read_line]: ../std/io/struct.Stdin.html#method.read_line
[result]: ../std/result/enum.Result.html
[enums]: ch06-00-enums.html
[expect]: ../std/result/enum.Result.html#method.expect
[recover]: ch09-02-recoverable-errors-with-result.html
[randcrate]: https://crates.io/crates/rand
[semver]: http://semver.org
[cratesio]: https://crates.io/
[doccargo]: https://doc.rust-lang.org/cargo/
[doccratesio]: https://doc.rust-lang.org/cargo/reference/publishing.html
[match]: ch06-02-match.html
[shadowing]: ch03-01-variables-and-mutability.html#shadowing
[parse]: ../std/primitive.str.html#method.parse
[integers]: ch03-02-data-types.html#integer-types

