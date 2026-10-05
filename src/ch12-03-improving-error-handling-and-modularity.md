## Refactoring to Improve Modularity and Error Handling

Apne program ko improve karne ke liye, hum chaar problems ko fix karenge jo program ki structure aur potential errors ko handle karne ke tareeqe se related hain. Sab se pehle, hamara `main` function ab do tasks perform karta hai: arguments ko parse karna aur files ko read karna. Jaise jaise hamara program grow karega, `main` function ke handle kiye jane wale separate tasks ki tadaad barhti jayegi. Jaise hi ek function ki responsibilities barhti hain, uske baare mein reasoning karna zyada mushkil ho jata hai, usay test karna mushkil ho jata hai, aur uske kisi ek part ko break kiye baghair usay change karna bhi mushkil ho jata hai. Behtar yeh hai ke functionality ko separate kiya jaye taake har function sirf ek task ke liye responsible ho.

Yeh issue doosri problem se bhi related hai: Agarche `query` aur `file_path` hamare program ke configuration variables hain, lekin `contents` jaise variables program ki logic perform karne ke liye use hote hain. Jitna `main` lamba hota jayega, utne hi zyada variables humein scope mein lane padenge; aur jitne zyada variables scope mein honge, utna hi mushkil hoga ke har variable ke purpose ko track kiya ja sake. Behtar yeh hai ke configuration variables ko ek structure mein group kar diya jaye taake unka purpose clear ho.

Teesri problem yeh hai ke file read karne mein failure hone par error message print karne ke liye humne `expect` use kiya hai, lekin error message sirf `Should have been able to read the file` print karta hai. File read karna kai ways mein fail ho sakta hai: Misal ke taur par, file missing ho sakti hai, ya shayad hamare paas use open karne ki permission na ho. Filhaal, situation chahe jo bhi ho, hum har cheez ke liye same error message print karenge, jo user ko koi information nahi dega!

Chauthi problem yeh hai ke hum error handle karne ke liye `expect` use karte hain, aur agar user hamare program ko sufficient arguments specify kiye baghair run kare, to use Rust ki taraf se `index out of bounds` error milega jo problem ko clearly explain nahi karta. Behtar yeh hoga ke tamam error-handling code ek hi jagah ho taake future maintainers ke paas sirf ek jagah ho jahan woh code ko consult kar saken agar error-handling logic ko change karne ki zaroorat pade. Tamam error-handling code ko ek hi jagah rakhne se yeh bhi ensure hoga ke hum aise messages print kar rahe hon jo hamare end users ke liye meaningful hon.

Aaiye apne project ko refactor karke in chaar problems ko address karte hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="separation-of-concerns-for-binary-projects"></a>

### Separating Concerns in Binary Projects

`main` function ko multiple tasks ki responsibility dene ka organizational problem bohat se binary projects mein common hai. Isi wajah se, bohat se Rust programmers ke liye binary program ke separate concerns ko split karna useful hota hai jab `main` function bara hona shuru ho jata hai. Is process ke following steps hain:

* Apne program ko ek *main.rs* file aur ek *lib.rs* file mein split karein aur apne program ki logic ko *lib.rs* mein move karein.
* Jab tak aapki command line parsing logic chhoti hai, yeh `main` function mein reh sakti hai.
* Jab command line parsing logic complicated hona shuru ho jaye, to ise `main` function se extract karke doosre functions ya types mein move karein.

Is process ke baad `main` function mein jo responsibilities baqi reh jati hain, woh following tak limited honi chahiye:

* Argument values ke saath command line parsing logic ko call karna
* Koi bhi doosri configuration set up karna
* *lib.rs* mein ek `run` function ko call karna
* Agar `run` error return kare to us error ko handle karna

Yeh pattern concerns ko separate karne ke baare mein hai: *main.rs* program ko run karne ko handle karta hai aur *lib.rs* us task ki tamam logic ko handle karta hai jo program perform kar raha hai. Kyun ke aap `main` function ko directly test nahi kar sakte, yeh structure aapko apne program ki tamam logic ko `main` function se bahar move karke test karne deta hai. `main` function mein jo code baqi reh jata hai woh itna chhota hoga ke uski correctness ko sirf use read karke verify kiya ja sakta hai. Aaiye is process ko follow karte hue apne program ko dobara rework karte hain.

#### Extracting the Argument Parser

Hum arguments ko parse karne ki functionality ko ek function mein extract karenge jise `main` call karega. Listing 12-5 `main` function ka naya start dikhati hai jo ek naye function `parse_config` ko call karta hai, jise hum *src/main.rs* mein define karenge.

<Listing number="12-5" file-name="src/main.rs" caption="Extracting a `parse_config` function from `main`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-05/src/main.rs:here}}
```

</Listing>

Hum ab bhi command line arguments ko ek vector mein collect kar rahe hain, lekin `main` function ke andar index 1 par maujood argument value ko `query` variable aur index 2 par maujood argument value ko `file_path` variable mein assign karne ke bajaye, hum poore vector ko `parse_config` function mein pass karte hain. Phir `parse_config` function woh logic hold karta hai jo determine karta hai ke kaunsa argument kis variable mein jana chahiye aur values ko wapas `main` mein pass karta hai. Hum ab bhi `main` mein `query` aur `file_path` variables create karte hain, lekin ab `main` ke paas yeh responsibility nahi hoti ke command line arguments aur variables ek doosre se kis tarah correspond karte hain.

Yeh rework hamare chhote program ke liye overkill lag sakta hai, lekin hum chhote, incremental steps mein refactoring kar rahe hain. Yeh change karne ke baad, program ko dobara run karein taake verify kiya ja sake ke argument parsing ab bhi kaam kar rahi hai. Apni progress ko frequently check karna achha hota hai, taake jab problems occur hon to unki cause identify karne mein madad mile.

#### Grouping Configuration Values

Hum `parse_config` function ko aur improve karne ke liye ek aur chhota step le sakte hain. Filhaal, hum ek tuple return kar rahe hain, lekin phir hum foran us tuple ko dobara individual parts mein break kar dete hain. Yeh is baat ki nishani hai ke shayad abhi hamare paas sahi abstraction nahi hai.

Ek aur indicator jo dikhata hai ke improvement ki gunjaish hai, woh `parse_config` ka `config` wala hissa hai, jo imply karta hai ke jo do values hum return karte hain woh aapas mein related hain aur dono ek hi configuration value ka hissa hain. Filhaal hum data ke structure mein is meaning ko convey nahi kar rahe, siwaye iske ke dono values ko ek tuple mein group kar diya gaya hai; iske bajaye hum dono values ko ek struct mein rakhenge aur struct ke har field ko ek meaningful name denge. Aisa karne se future mein is code ko maintain karne walon ke liye yeh samajhna asaan hoga ke different values ek doosre se kis tarah related hain aur unka purpose kya hai.

Listing 12-6 `parse_config` function mein ki gayi improvements ko dikhati hai.

<Listing number="12-6" file-name="src/main.rs" caption="Refactoring `parse_config` to return an instance of a `Config` struct">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-06/src/main.rs:here}}
```

</Listing>

Humne `Config` naam ka ek struct add kiya hai jise `query` aur `file_path` naam ke fields rakhne ke liye define kiya gaya hai. Ab `parse_config` ka signature indicate karta hai ke yeh ek `Config` value return karta hai. `parse_config` ke body mein, jahan hum pehle `args` mein maujood `String` values ko reference karne wale string slices return karte the, ab hum `Config` ko owned `String` values contain karne ke liye define karte hain. `main` mein `args` variable argument values ka owner hai aur sirf `parse_config` function ko unhein borrow karne de raha hai, jis ka matlab hai ke agar `Config` `args` ki values ki ownership lene ki koshish kare to hum Rust ke borrowing rules ki khilaf-warzi karenge.

Hum `String` data ko manage karne ke kai tareeqe apna sakte hain; lekin sab se asaan, agarche kuch had tak inefficient, tareeqa values par `clone` method call karna hai. Is se data ki ek complete copy banegi jise `Config` instance own karega, jo string data ka reference store karne ke muqable mein zyada time aur memory lega. Lekin data ko clone karna hamare code ko kaafi straightforward bhi bana deta hai kyun ke humein references ki lifetimes manage nahi karni padtin; is situation mein simplicity hasil karne ke liye thori performance sacrifice karna ek worthwhile trade-off hai.

> ### The Trade-Offs of Using `clone`
>
> Bohat se Rustaceans mein `clone` ko ownership problems fix karne ke liye use karne se bachne ka rujhan hota hai, kyun ke iski runtime cost hoti hai. [Chapter 13][ch13]<!-- ignore --> mein, aap seekhenge ke is tarah ki situation mein zyada efficient methods kaise use kiye jate hain. Lekin filhaal, progress continue rakhne ke liye kuch strings ko copy karna theek hai, kyun ke aap yeh copies sirf ek baar banayenge aur aapka file path aur query string bohat chhote hain. Aisa working program rakhna jo thora inefficient ho, is baat se behtar hai ke aap apni first pass mein code ko hyperoptimize karne ki koshish karein. Jaise jaise aap Rust ke saath zyada experienced hote jayenge, sab se efficient solution ke saath start karna asaan ho jayega, lekin filhaal `clone` call karna bilkul acceptable hai.

Humne `main` ko update kiya hai taake woh `parse_config` se return hone wale `Config` instance ko `config` naam ke variable mein rakhe, aur humne pehle separate `query` aur `file_path` variables ko use karne wale code ko bhi update kiya hai taake ab woh `Config` struct ke fields ko use kare.

Ab hamara code zyada clearly convey karta hai ke `query` aur `file_path` aapas mein related hain aur unka purpose program ke kaam karne ke tareeqe ko configure karna hai. Jo bhi code in values ko use karta hai, woh janta hai ke unhein unke purpose ke naam par rakhe gaye fields mein `config` instance ke andar find karna hai.

#### Creating a Constructor for `Config`

Ab tak, humne command line arguments ko parse karne ki logic ko `main` se extract karke `parse_config` function mein rakh diya hai. Aisa karne se humein yeh samajhne mein madad mili ke `query` aur `file_path` values aapas mein related hain, aur yeh relationship hamare code mein convey honi chahiye. Phir humne `Config` struct add kiya taake `query` aur `file_path` ke related purpose ko name kiya ja sake aur `parse_config` function se values ke names ko struct field names ke taur par return kiya ja sake.

Ab, kyun ke `parse_config` function ka purpose `Config` instance create karna hai, hum `parse_config` ko ek plain function se change karke `new` naam ka function bana sakte hain jo `Config` struct ke saath associated ho. Yeh change karne se code zyada idiomatic ho jayega. Hum standard library mein types ke instances, jaise `String`, ko `String::new` call karke create kar sakte hain. Isi tarah, `parse_config` ko `Config` ke saath associated `new` function mein change karke, hum `Config` ke instances `Config::new` call karke create kar sakenge. Listing 12-7 un changes ko dikhati hai jo humein karne ki zarurat hai.

<Listing number="12-7" file-name="src/main.rs" caption="Changing `parse_config` into `Config::new`">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-07/src/main.rs:here}}
```

</Listing>

Humne `main` mein us jagah ko update kiya hai jahan hum `parse_config` call kar rahe the, taake ab `Config::new` call kiya jaye. Humne `parse_config` ka name `new` kar diya hai aur ise ek `impl` block ke andar move kar diya hai, jo `new` function ko `Config` ke saath associate karta hai. Is code ko dobara compile karke dekhein taake yakeen ho jaye ke yeh kaam karta hai.

### Fixing the Error Handling

Ab hum apni error handling ko fix karne par kaam karenge. Yaad karein ke `args` vector mein index 1 ya index 2 par maujood values ko access karne ki koshish karne par program panic kar jayega agar vector mein teen se kam items hon. Program ko bina kisi arguments ke run karke dekhein; yeh kuch is tarah nazar aayega:

```console
{{#include ../listings/ch12-an-io-project/listing-12-07/output.txt}}
```

`index out of bounds: the len is 1 but the index is 1` wali line ek error message hai jo programmers ke liye intended hai. Yeh hamare end users ko yeh samajhne mein madad nahi karega ke unhein iske bajaye kya karna chahiye. Aaiye ab ise fix karte hain.

#### Improving the Error Message

Listing 12-8 mein, hum `new` function mein ek check add karte hain jo verify karega ke index 1 aur index 2 ko access karne se pehle slice ki length kaafi hai. Agar slice ki length kaafi nahi hai, to program panic karega aur ek behtar error message display karega.

<Listing number="12-8" file-name="src/main.rs" caption="Adding a check for the number of arguments">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-08/src/main.rs:here}}
```

</Listing>

Yeh code [Listing 9-13 mein likhe gaye `Guess::new` function][ch9-custom-types]<!-- ignore --> jaisa hai, jahan humne `panic!` call kiya tha jab `value` argument valid values ki range se bahar tha. Yahan values ki range check karne ke bajaye, hum yeh check kar rahe hain ke `args` ki length kam az kam `3` hai aur function ka baqi hissa is assumption ke under operate kar sakta hai ke yeh condition puri hoti hai. Agar `args` mein teen se kam items hon, to yeh condition `true` hogi, aur hum program ko foran end karne ke liye `panic!` macro call karenge.

`new` mein in chand extra lines of code ke saath, aaiye program ko dobara bina kisi arguments ke run karte hain taake dekhein ke ab error kaisa nazar aata hai:

```console
{{#include ../listings/ch12-an-io-project/listing-12-08/output.txt}}
```

Yeh output behtar hai: Ab hamare paas ek reasonable error message hai. Lekin is mein kuch extra information bhi hai jo hum apne users ko nahi dena chahte. Shayad jo technique humne Listing 9-13 mein use ki thi, woh yahan use karne ke liye best nahi hai: `panic!` ki call programming problem ke liye usage problem ki nisbat zyada appropriate hai, [jaisa ke Chapter 9 mein discuss kiya gaya hai][ch9-error-guidelines]<!-- ignore -->. Iske bajaye, hum woh doosri technique use karenge jo aapne Chapter 9 mein seekhi thi—ek [`Result` return karna][ch9-result]<!-- ignore --> jo ya to success ya error ko indicate karta hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="returning-a-result-from-new-instead-of-calling-panic"></a>

#### Returning a `Result` Instead of Calling `panic!`

Iske bajaye hum ek `Result` value return kar sakte hain jo successful case mein ek `Config` instance contain karegi aur error case mein problem ko describe karegi. Hum function ka name bhi `new` se change karke `build` karenge kyun ke bohat se programmers `new` functions se expect karte hain ke woh kabhi fail nahi hote. Jab `Config::build` `main` ko communicate kar raha hoga, to hum `Result` type ko yeh signal dene ke liye use kar sakte hain ke koi problem hui hai. Phir hum `main` ko change kar sakte hain taake woh `Err` variant ko hamare users ke liye zyada practical error mein convert kare, bina us surrounding text ke jo `panic!` ki call `thread 'main'` aur `RUST_BACKTRACE` ke baare mein generate karti hai.

Listing 12-9 un changes ko dikhati hai jo humein ab `Config::build` kehlane wale function ki return value aur function ke us body mein karne hain jo `Result` return karegi. Note karein ke jab tak hum `main` ko bhi update nahi karte, yeh compile nahi hoga, jo hum next listing mein karenge.

<Listing number="12-9" file-name="src/main.rs" caption="Returning a `Result` from `Config::build`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-09/src/main.rs:here}}
```

</Listing>

Hamara `build` function ek `Result` return karta hai jismein successful case mein ek `Config` instance aur error case mein ek string literal hota hai. Hamari error values hamesha aise string literals hongi jin ki `'static` lifetime hogi.

Humne function ki body mein do changes kiye hain: User jab kaafi arguments pass nahi karta to `panic!` call karne ke bajaye, ab hum ek `Err` value return karte hain, aur humne `Config` return value ko ek `Ok` mein wrap kar diya hai. Yeh changes function ko uske naye type signature ke mutabiq bana dete hain.

`Config::build` se ek `Err` value return karne se `main` function ko `build` function se return hone wali `Result` value ko handle karne aur error case mein process ko zyada cleanly exit karne ka mauqa milta hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="calling-confignew-and-handling-errors"></a>

#### Calling `Config::build` and Handling Errors

Error case ko handle karne aur user-friendly message print karne ke liye, humein `main` ko update karna hoga taake woh `Config::build` se return hone wali `Result` ko handle kare, jaisa ke Listing 12-10 mein dikhaya gaya hai. Hum command line tool ko nonzero error code ke saath exit karne ki responsibility bhi `panic!` se le lenge aur iske bajaye ise khud implement karenge. Nonzero exit status ek convention hai jo us process ko signal karta hai jis ne hamare program ko call kiya ke program error state ke saath exit hua hai.

<Listing number="12-10" file-name="src/main.rs" caption="Exiting with an error code if building a `Config` fails">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-10/src/main.rs:here}}
```

</Listing>

Is listing mein, humne ek aisa method use kiya hai jise humne abhi tak detail mein cover nahi kiya: `unwrap_or_else`, jo standard library ki taraf se `Result<T, E>` par defined hai. `unwrap_or_else` ko use karne se humein kuch custom, non-`panic!` error handling define karne ka mauqa milta hai. Agar `Result` ek `Ok` value hai, to is method ka behavior `unwrap` jaisa hai: yeh woh inner value return karta hai jise `Ok` wrap kar raha hai. Lekin agar value ek `Err` value hai, to yeh method closure ke andar maujood code ko call karta hai, jo ek anonymous function hai jise hum define karke `unwrap_or_else` ko argument ke taur par pass karte hain. Hum closures ko [Chapter 13][ch13]<!-- ignore --> mein zyada detail mein cover karenge. Filhaal, aapko sirf itna jaanne ki zarurat hai ke `unwrap_or_else` `Err` ki inner value ko, jo is case mein static string `"not enough arguments"` hai jo humne Listing 12-9 mein add ki thi, argument `err` mein hamari closure ko pass karega jo vertical pipes ke darmiyan nazar aata hai. Phir closure ke andar ka code run hone par `err` value ko use kar sakta hai.

Humne ek nayi `use` line add ki hai taake standard library se `process` ko scope mein laya ja sake. Error case mein run hone wali closure ka code sirf do lines ka hai: hum `err` value ko print karte hain aur phir `process::exit` call karte hain. `process::exit` function program ko foran rok dega aur woh number return karega jo exit status code ke taur par pass kiya gaya tha. Yeh Listing 12-8 mein use ki gayi `panic!`-based handling jaisa hai, lekin ab humein woh tamam extra output nahi milta. Aaiye ise try karte hain:

```console
{{#include ../listings/ch12-an-io-project/listing-12-10/output.txt}}
```

Great! Yeh output hamare users ke liye kaafi zyada friendly hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="extracting-logic-from-the-main-function"></a>

### Extracting Logic from `main`

Ab jab hum configuration parsing ki refactoring complete kar chuke hain, to aaiye program ki logic ki taraf aate hain. Jaisa ke humne [“Separating Concerns in Binary Projects”](#separation-of-concerns-for-binary-projects)<!-- ignore --> mein bataya tha, hum ek `run` naam ka function extract karenge jo `main` function mein currently maujood tamam logic ko hold karega jo configuration set up karne ya errors handle karne se related nahi hai. Jab hum complete kar lenge, to `main` function concise aur inspection ke zariye verify karna asaan hoga, aur hum baqi tamam logic ke liye tests likh sakenge.

Listing 12-11 `run` function ko extract karne ki chhoti, incremental improvement dikhati hai.

<Listing number="12-11" file-name="src/main.rs" caption="Extracting a `run` function containing the rest of the program logic">

```rust,ignore id="9uhjv7"
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-11/src/main.rs:here}}
```

</Listing>

Ab `run` function mein `main` ki baqi tamam logic maujood hai, jo file read karne se start hoti hai. `run` function `Config` instance ko ek argument ke taur par leta hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="returning-errors-from-the-run-function"></a>

#### Returning Errors from `run`

Ab jab baqi program logic ko `run` function mein separate kar diya gaya hai, hum error handling ko improve kar sakte hain, jaisa ke humne Listing 12-9 mein `Config::build` ke saath kiya tha. Program ko `expect` call karke panic karne dene ke bajaye, `run` function jab kuch ghalat hoga to ek `Result<T, E>` return karega. Is se humein errors handle karne ki logic ko `main` mein ek user-friendly tareeqe se aur zyada consolidate karne ka mauqa milega. Listing 12-12 un changes ko dikhati hai jo humein `run` ke signature aur body mein karne hain.

<Listing number="12-12" file-name="src/main.rs" caption="Changing the `run` function to return `Result`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-12/src/main.rs:here}}
```

</Listing>

Yahan humne teen significant changes kiye hain. Sab se pehle, humne `run` function ka return type `Result<(), Box<dyn Error>>` kar diya hai. Pehle yeh function unit type, `()`, return karta tha, aur hum ise `Ok` case mein return hone wali value ke taur par rakhte hain.

Error type ke liye, humne trait object `Box<dyn Error>` use kiya hai (aur top par `use` statement ke zariye `std::error::Error` ko scope mein laya hai). Hum trait objects ko [Chapter 18][ch18]<!-- ignore --> mein cover karenge. Filhaal, bas itna jaan lein ke `Box<dyn Error>` ka matlab hai ke function ek aisi type return karega jo `Error` trait ko implement karti hai, lekin humein yeh specify karne ki zarurat nahi ke return value ki particular type kya hogi. Is se humein aise error values return karne ki flexibility milti hai jo different error cases mein different types ki ho sakti hain. `dyn` keyword *dynamic* ka short form hai.

Doosra, humne `expect` ki call ko hata kar `?` operator use kiya hai, jaisa ke humne [Chapter 9][ch9-question-mark]<!-- ignore --> mein discuss kiya tha. Error par `panic!` karne ke bajaye, `?` current function se error value return kar dega taake caller use handle kar sake.

Teesra, ab `run` function success case mein ek `Ok` value return karta hai. Humne signature mein `run` function ka success type `()` declare kiya hai, jis ka matlab hai ke humein unit type value ko `Ok` value mein wrap karna hoga. Yeh `Ok(())` syntax shuru mein thora ajeeb lag sakta hai. Lekin `()` ko is tarah use karna yeh indicate karne ka idiomatic tareeqa hai ke hum `run` ko sirf uske side effects ke liye call kar rahe hain; yeh koi aisi value return nahi karta jiski humein zarurat ho.

Jab aap is code ko run karenge, to yeh compile ho jayega lekin ek warning display karega:

```console
{{#include ../listings/ch12-an-io-project/listing-12-12/output.txt}}
```

Rust humein batata hai ke hamare code ne `Result` value ko ignore kar diya hai aur `Result` value yeh indicate kar sakti hai ke koi error occur hua hai. Lekin hum yeh check nahi kar rahe ke error hua hai ya nahi, aur compiler humein yaad dilata hai ke shayad humein yahan kuch error-handling code rakhna tha! Aaiye ab is problem ko theek karte hain.

#### Handling Errors Returned from `run` in `main`

Hum errors ko check karenge aur unhein ek aisi technique ke zariye handle karenge jo Listing 12-10 mein `Config::build` ke saath use ki gayi technique jaisi hai, lekin is mein ek halka sa difference hai:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore id="g8x0qa"
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-01-handling-errors-in-main/src/main.rs:here}}
```

Hum `if let` ko `unwrap_or_else` ke bajaye use karte hain taake check kiya ja sake ke `run` ek `Err` value return karta hai ya nahi, aur agar karta hai to `process::exit(1)` call kiya ja sake. `run` koi aisi value return nahi karta jise hum `unwrap` karna chahte hon, jis tarah `Config::build` `Config` instance return karta hai. Kyun ke success case mein `run` `()` return karta hai, hum sirf error detect karne mein interested hain, is liye humein `unwrap_or_else` ki zarurat nahi hai jo unwrapped value return kare, jo sirf `()` hoti.

Dono cases mein `if let` aur `unwrap_or_else` functions ki bodies same hain: hum error print karte hain aur exit karte hain.

### Splitting Code into a Library Crate

Hamara `minigrep` project ab tak achha lag raha hai! Ab hum *src/main.rs* file ko split karenge aur kuch code ko *src/lib.rs* file mein rakhenge. Is tarah, hum code ko test kar sakenge aur hamare paas kam responsibilities wali *src/main.rs* file hogi.

Aaiye text search karne ki responsibility wale code ko *src/main.rs* ke bajaye *src/lib.rs* mein define karte hain, jo humein (ya hamari `minigrep` library ko use karne wale kisi bhi doosre shakhs ko) searching function ko hamari `minigrep` binary se zyada contexts mein call karne dega.

Sab se pehle, aaiye *src/lib.rs* mein `search` function ka signature define karte hain, jaisa ke Listing 12-13 mein dikhaya gaya hai, aur iski body mein `unimplemented!` macro call karte hain. Jab hum implementation fill karenge to hum signature ko zyada detail mein explain karenge.

<Listing number="12-13" file-name="src/lib.rs" caption="Defining the `search` function in *src/lib.rs*">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-13/src/lib.rs}}
```

</Listing>

Humne function definition par `pub` keyword use kiya hai taake `search` ko hamare library crate ke public API ke hissa ke taur par designate kiya ja sake. Ab hamare paas ek library crate hai jise hum apne binary crate se use kar sakte hain aur jise hum test kar sakte hain!

Ab humein *src/lib.rs* mein define kiye gaye code ko binary crate mein *src/main.rs* ke scope mein lana hai aur use call karna hai, jaisa ke Listing 12-14 mein dikhaya gaya hai.

<Listing number="12-14" file-name="src/main.rs" caption="Using the `minigrep` library crate’s `search` function in *src/main.rs*">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-14/src/main.rs:here}}
```

</Listing>

Hum `use minigrep::search` line add karte hain taake library crate se `search` function ko binary crate ke scope mein laya ja sake. Phir, `run` function mein file ke contents ko print karne ke bajaye, hum `search` function ko call karte hain aur `config.query` value aur `contents` ko arguments ke taur par pass karte hain. Phir, `run` `for` loop use karke `search` se return hone wali har us line ko print karega jo query se match karti hai. Yeh `main` function mein maujood un `println!` calls ko remove karne ka bhi achha waqt hai jo query aur file path display karti thin, taake hamara program sirf search results print kare (agar koi errors occur na hon).

Note karein ke search function printing shuru hone se pehle tamam results ko ek vector mein collect karega aur phir us vector ko return karega. Bari files mein search karte waqt yeh implementation results display karne mein slow ho sakti hai, kyun ke results milte hi print nahi kiye jate; hum Chapter 13 mein iterators ko use karke isay fix karne ke ek possible tareeqe par baat karenge.

Whew! Yeh kaafi kaam tha, lekin humne khud ko future mein success ke liye set up kar diya hai. Ab errors handle karna kaafi asaan hai, aur humne code ko zyada modular bana diya hai. Yahan se aage hamara lagbhag tamam kaam *src/lib.rs* mein hoga.

Aaiye is nayi hasil hui modularity ka faida uthate hain aur woh kaam karte hain jo purane code ke saath mushkil hota, lekin naye code ke saath asaan hai: hum kuch tests likhenge!

[ch13]: ch13-00-functional-features.html
[ch9-custom-types]: ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation
[ch9-error-guidelines]: ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling
[ch9-result]: ch09-02-recoverable-errors-with-result.html
[ch18]: ch18-00-oop.html

