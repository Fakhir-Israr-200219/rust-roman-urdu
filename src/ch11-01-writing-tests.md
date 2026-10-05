## Tests kese likhe jate hain

*Tests* Rust ke aise functions hote hain jo verify karte hain ke non-test code expected tareeqe se functioning kar raha hai. Test functions ke bodies aam tor par ye teen actions perform karti hain:

* Jo bhi data ya state zaroori ho, usay set up karein.
* Us code ko run karein jise aap test karna chahte hain.
* Assert karein ke results wohi hain jo aap expect karte hain.

Aaiye un features ko dekhte hain jo Rust specifically tests likhne ke liye provide karta hai aur jo in actions ko perform karne mein madad dete hain. In mein `test` attribute, kuch macros, aur `should_panic` attribute shamil hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="the-anatomy-of-a-test-function"></a>

### Test Functions ko structure karna

Apni sab se simple form mein, Rust mein test ek aisa function hota hai jis par `test` attribute laga hota hai. Attributes Rust code ke pieces ke bare mein metadata hote hain; ek example `derive` attribute hai jo humne Chapter 5 mein structs ke saath use kiya tha. Kisi function ko test function mein badalne ke liye, `fn` se pehle wali line par `#[test]` add karein. Jab aap `cargo test` command ke saath apne tests run karte hain, Rust ek test runner binary build karta hai jo annotated functions ko run karti hai aur report karti hai ke har test function pass hua ya fail.

Jab bhi hum Cargo ke saath ek naya library project banate hain, to hamare liye ek test module automatically generate hota hai jisme ek test function hota hai. Ye module aapko apne tests likhne ke liye ek template deta hai taake jab bhi aap naya project start karein to aapko har baar exact structure aur syntax lookup na karna pade. Aap jitne chahein additional test functions aur jitne chahein test modules add kar sakte hain!

Hum pehle template test ke saath experiment karke tests ke kaam karne ke kuch aspects explore karenge, uske baad asal mein kuch code ko test karenge jo humne khud likha hoga aur assert karenge ke uska behavior correct hai.

Aaiye `adder` naam ka ek naya library project banate hain jo do numbers ko add karega:

```console
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

Aapki `adder` library ki *src/lib.rs* file ka content Listing 11-1 ki tarah dikhna chahiye.

<Listing number="11-1" file-name="src/lib.rs" caption="`cargo new` ke zariye automatically generate hone wala code">

<!-- manual-regeneration
cd listings/ch11-writing-automated-tests
rm -rf listing-11-01
cargo new listing-11-01 --lib --name adder
cd listing-11-01
echo "$ cargo test" > output.txt
RUSTFLAGS="-A unused_variables -A dead_code" RUST_TEST_THREADS=1 cargo test >> output.txt 2>&1
git diff output.txt # commit any relevant changes; discard irrelevant ones
cd ../../.."
-->

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

</Listing>

File ek example `add` function se start hoti hai taake hamare paas test karne ke liye kuch ho.

Abhi ke liye, sirf `it_works` function par focus karte hain. `#[test]` annotation ko note karein: Ye attribute indicate karta hai ke ye ek test function hai, isliye test runner jaanta hai ke is function ko test ke taur par treat karna hai. `tests` module mein hamare paas non-test functions bhi ho sakte hain jo common scenarios set up karne ya common operations perform karne mein help karte hain, isliye hamein hamesha indicate karna hota hai ke kaun se functions tests hain.

Example function body `assert_eq!` macro ko use karti hai taake assert kiya ja sake ke `result`, jisme `add` ko 2 aur 2 ke saath call karne ka result hai, 4 ke equal hai. Ye assertion ek typical test ke format ki example hai. Aaiye ise run karke dekhte hain ke ye test pass hota hai.

`cargo test` command hamare project mein tamam tests run karti hai, jaisa ke Listing 11-2 mein dikhaya gaya hai.

<Listing number="11-2" caption="Automatically generated test ko run karne ka output">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-01/output.txt}}
```

</Listing>

Cargo ne test ko compile aur run kiya. Hum line `running 1 test` dekhte hain. Agli line generated test function ka naam dikhati hai, jise `tests::it_works` kaha gaya hai, aur ye ke us test ko run karne ka result `ok` hai. Overall summary `test result: ok.` ka matlab hai ke tamam tests pass ho gaye, aur `1 passed; 0 failed` wala hissa un tests ki total tadaad batata hai jo pass ya fail hue.

Kisi test ko ignore kiya ja sakta hai taake woh kisi particular instance mein run na ho; hum isay baad mein is chapter ke [“Ignoring Tests Unless Specifically Requested”][ignoring]<!-- ignore --> section mein cover karenge. Kyun ke humne yahan aisa nahi kiya, summary `0 ignored` dikhati hai. Hum `cargo test` command ko ek argument bhi de sakte hain taake sirf un tests ko run kare jo kisi string ke saath match karte hain; isay *filtering* kaha jata hai, aur hum isay [“Running a Subset of Tests by Name”][subset]<!-- ignore --> section mein cover karenge. Yahan humne run hone wale tests ko filter nahi kiya, isliye summary ke end mein `0 filtered out` dikhata hai.

`0 measured` statistic benchmark tests ke liye hota hai jo performance measure karte hain. Benchmark tests, is waqt jab ye likha gaya hai, sirf nightly Rust mein available hain. Zyada information ke liye [benchmark tests ke documentation][bench] ko dekhein.

Test output ka agla hissa, jo `Doc-tests adder` se start hota hai, kisi bhi documentation tests ke results ke liye hai. Abhi hamare paas koi documentation tests nahi hain, lekin Rust hamari API documentation mein maujood kisi bhi code examples ko compile kar sakta hai. Ye feature aapki docs aur code ko sync mein rakhne mein help karta hai! Hum Chapter 14 ke [“Documentation Comments as Tests”][doc-comments]<!-- ignore --> section mein documentation tests likhne ka tareeqa discuss karenge. Abhi ke liye, hum `Doc-tests` output ko ignore karenge.

Ab hum test ko apni zarooriyat ke mutabiq customize karna shuru karte hain. Sab se pehle, `it_works` function ka naam badal kar koi doosra naam rakh dein, jaise `exploration`, jaisa ke neeche dikhaya gaya hai:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/src/lib.rs}}
```

Phir `cargo test` dobara run karein. Ab output mein `it_works` ke bajaye `exploration` dikhai dega:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/output.txt}}
```

Ab hum ek aur test add karenge, lekin is baar hum aisa test banayenge jo fail ho! Tests tab fail hote hain jab test function ke andar kuch panic karta hai. Har test ek naye thread mein run hota hai, aur jab main thread dekhta hai ke kisi test thread ki execution khatam ho gayi hai, to test ko failed mark kar diya jata hai. Chapter 9 mein humne baat ki thi ke panic karne ka sab se simple tareeqa `panic!` macro ko call karna hai. `another` naam ka function bana kar naya test add karein, taake aapki *src/lib.rs* file Listing 11-3 ki tarah nazar aaye.

<Listing number="11-3" file-name="src/lib.rs" caption="`panic!` macro ko call karne ki wajah se fail hone wala second test add karna">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-03/src/lib.rs}}
```

</Listing>

`cargo test` use karke tests dobara run karein. Output Listing 11-4 ki tarah hona chahiye, jo dikhata hai ke hamara `exploration` test pass hua aur `another` fail hua.

<Listing number="11-4" caption="Jab ek test pass aur ek test fail ho to test results">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-03/output.txt}}
```

</Listing>

<!-- manual-regeneration
rg panicked listings/ch11-writing-automated-tests/listing-11-03/output.txt
check the line number of the panic matches the line number in the following paragraph
 -->

`ok` ke bajaye, line `test tests::another` `FAILED` show karti hai. Individual results aur summary ke darmiyan do naye sections appear hote hain: Pehla har test failure ki detailed reason display karta hai. Is case mein hamein details milti hain ke `tests::another` fail hua kyun ke *src/lib.rs* file ki line 17 par `Make this test fail` message ke saath panic hua. Agla section sirf un tamam failing tests ke names list karta hai, jo tab useful hota hai jab bohat saare tests aur bohat saara detailed failing test output ho. Hum kisi failing test ke naam ko use karke sirf usi test ko run kar sakte hain taake usay zyada easily debug kiya ja sake; hum [“Controlling How Tests Are Run”][controlling-how-tests-are-run]<!-- ignore --> section mein tests run karne ke tareeqon ke bare mein mazeed baat karenge.

Summary line end mein display hoti hai: Overall, hamara test result `FAILED` hai. Hamara ek test pass hua aur ek test fail hua.

Ab jab aapne different scenarios mein test results kaise dikhte hain ye dekh liya hai, to aaiye `panic!` ke ilawa kuch aur macros dekhte hain jo tests mein useful hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="checking-results-with-the-assert-macro"></a>

### `assert!` ke saath Results ko Check karna

Standard library ki taraf se provide kiya gaya `assert!` macro us waqt useful hota hai jab aap ye ensure karna chahte hain ke test mein koi condition `true` evaluate ho. Hum `assert!` macro ko ek aisa argument dete hain jo Boolean mein evaluate hota hai. Agar value `true` ho, to kuch nahi hota aur test pass ho jata hai. Agar value `false` ho, to `assert!` macro `panic!` ko call karta hai taake test fail ho jaye. `assert!` macro use karne se hamein check karne mein madad milti hai ke hamara code usi tarah function kar raha hai jis tarah hum chahte hain.

Chapter 5 mein, Listing 5-15 mein, humne `Rectangle` struct aur uska `can_hold` method use kiya tha, jinhein yahan Listing 11-5 mein dobara diya gaya hai. Aaiye is code ko *src/lib.rs* file mein rakhte hain, phir `assert!` macro ko use karke iske liye kuch tests likhte hain.

<Listing number="11-5" file-name="src/lib.rs" caption="Chapter 5 ka `Rectangle` struct aur uska `can_hold` method">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-05/src/lib.rs}}
```

</Listing>

`can_hold` method ek Boolean return karta hai, jis ka matlab hai ke ye `assert!` macro ke liye perfect use case hai. Listing 11-6 mein, hum ek aisa test likhte hain jo `can_hold` method ko exercise karta hai. Is ke liye hum ek `Rectangle` instance create karte hain jis ki width 8 aur height 7 hai aur assert karte hain ke ye ek doosre `Rectangle` instance ko hold kar sakta hai jis ki width 5 aur height 1 hai.

<Listing number="11-6" file-name="src/lib.rs" caption="`can_hold` ke liye ek test jo check karta hai ke kya bara rectangle waqai chhote rectangle ko hold kar sakta hai">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-06/src/lib.rs:here}}
```

</Listing>

`tests` module ke andar `use super::*;` line ko note karein. `tests` module ek regular module hai jo unhi usual visibility rules ko follow karta hai jinhein humne Chapter 7 ke [“Paths for Referring to an Item in the Module Tree”][paths-for-referring-to-an-item-in-the-module-tree]<!-- ignore --> section mein cover kiya tha. Kyun ke `tests` module ek inner module hai, hamein outer module ke andar maujood us code ko inner module ke scope mein lana hota hai jise hum test kar rahe hain. Yahan hum glob use karte hain, isliye outer module mein hum jo kuch bhi define karte hain woh `tests` module ke liye available hota hai.

Humne apne test ka naam `larger_can_hold_smaller` rakha hai, aur humne woh do `Rectangle` instances create kiye hain jin ki hamein zaroorat hai. Phir humne `assert!` macro ko call kiya aur usay `larger.can_hold(&smaller)` call karne ka result pass kiya. Is expression ka result `true` hona chahiye, isliye hamara test pass hona chahiye. Aaiye dekhte hain!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-06/output.txt}}
```

Ye pass hota hai! Ab ek aur test add karte hain, is baar assert karte hue ke ek chhota rectangle ek bare rectangle ko hold nahi kar sakta:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/src/lib.rs:here}}
```

Kyun ke is case mein `can_hold` function ka correct result `false` hai, hamein is result ko `assert!` macro ko pass karne se pehle negate karna hoga. Is tarah hamara test tab pass hoga jab `can_hold` `false` return kare:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/output.txt}}
```

Do tests jo pass ho gaye! Ab dekhte hain ke jab hum apne code mein ek bug introduce karte hain to hamare test results par kya asar hota hai. Hum `can_hold` method ki implementation ko change karenge aur widths ko compare karte waqt greater-than sign (`>`) ko less-than sign (`<`) se replace kar denge:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/src/lib.rs:here}}
```

Ab tests run karne se ye output milta hai:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/output.txt}}
```

Hamare tests ne bug pakar liya! Kyun ke `larger.width` `8` hai aur `smaller.width` `5` hai, `can_hold` mein widths ka comparison ab `false` return karta hai: 8, 5 se less nahi hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="testing-equality-with-the-assert_eq-and-assert_ne-macros"></a>

### `assert_eq!` aur `assert_ne!` ke Sath Equality Test Karna

Functionality ko verify karne ka ek common tareeqa yeh hai ke test kiye ja rahe code ke result aur us value ke darmiyan equality ko test kiya jaye jo hum expect karte hain ke code return karega. Aap yeh kaam `assert!` macro ko use karke aur `==` operator wala expression pass karke kar sakte hain. Lekin yeh itna common test hai ke standard library is test ko zyada asaani se perform karne ke liye macros ki ek pair—`assert_eq!` aur `assert_ne!`—provide karti hai. Yeh macros respectively do arguments ko equality ya inequality ke liye compare karte hain. Agar assertion fail ho jaye to yeh dono values ko bhi print karte hain, jis se yeh samajhna aasaan ho jata hai ke test *kyun* fail hua; is ke baraks, `assert!` macro sirf yeh indicate karta hai ke `==` expression se `false` value mili, aur un values ko print nahi karta jo is `false` value ka sabab bani thin.

Listing 11-7 mein, hum `add_two` naam ka ek function likhte hain jo apne parameter mein `2` add karta hai, aur phir `assert_eq!` macro ko use karke is function ko test karte hain.

<Listing number="11-7" file-name="src/lib.rs" caption="`assert_eq!` macro ko use karke function `add_two` ko test karna">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-07/src/lib.rs}}
```

</Listing>

Chaliye check karte hain ke yeh pass hota hai!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-07/output.txt}}
```

Hum `result` naam ka ek variable create karte hain jo `add_two(2)` call karne ka result hold karta hai. Phir, hum `result` aur `4` ko `assert_eq!` macro ke arguments ke taur par pass karte hain. Is test ke liye output line `test tests::it_adds_two ... ok` hai, aur `ok` text indicate karta hai ke hamara test pass ho gaya!

Ab apne code mein ek bug introduce karte hain taake dekh sakein ke jab `assert_eq!` fail hota hai to woh kaisa nazar aata hai. `add_two` function ki implementation ko change karke is ke bajaye `3` add karein:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/src/lib.rs:here}}
```

Dobara tests run karein:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/output.txt}}
```

Hamare test ne bug pakar liya! `tests::it_adds_two` test fail ho gaya, aur message humein batata hai ke jo assertion fail hui woh `left == right` thi aur `left` aur `right` ki values kya hain. Yeh message humein debugging shuru karne mein madad karta hai: `left` argument, jahan hamare paas `add_two(2)` call karne ka result tha, `5` tha, jabke `right` argument `4` tha. Aap imagine kar sakte hain ke jab hamare paas bohat saare tests chal rahe hon to yeh khaas taur par kitna helpful hoga.

Note karein ke kuch languages aur test frameworks mein equality assertion functions ke parameters ko `expected` aur `actual` kaha jata hai, aur arguments ko specify karne ka order matter karta hai. Lekin Rust mein, inhein `left` aur `right` kaha jata hai, aur jis value ki hum expectation rakhte hain aur jo value code produce karta hai unhein specify karne ka order matter nahi karta. Hum is test mein assertion ko `assert_eq!(4, result)` bhi likh sakte hain, jis ka result isi failure message mein hoga jo ``assertion `left == right` failed`` display karta hai.

`assert_ne!` macro tab pass hoga jab jo do values hum ise dete hain woh equal na hon, aur jab woh equal hon to yeh fail hoga. Yeh macro un cases ke liye sab se zyada useful hai jab humein yaqeen na ho ke koi value *kya* hogi, lekin humein pata ho ke woh value definitely *kya nahi honi chahiye*. Misal ke taur par, agar hum kisi aise function ko test kar rahe hon jo apne input ko kisi na kisi tareeqe se change karna guaranteed karta hai, lekin input kis tareeqe se change hoga yeh is baat par depend karta ho ke hum apne tests week ke kis din run karte hain, to assert karne ke liye behtareen cheez yeh ho sakti hai ke function ka output input ke equal na ho.

Andar se, `assert_eq!` aur `assert_ne!` macros respectively `==` aur `!=` operators ko use karte hain. Jab assertions fail hoti hain, to yeh macros apne arguments ko debug formatting ka use karke print karte hain, jis ka matlab hai ke jin values ko compare kiya ja raha hai unhein `PartialEq` aur `Debug` traits implement karna hoga. Tamam primitive types aur standard library ke zyada tar types in traits ko implement karte hain. Jo structs aur enums aap khud define karte hain, un types ki equality assert karne ke liye aapko `PartialEq` implement karna hoga. Assertion fail hone par values ko print karne ke liye aapko `Debug` bhi implement karna hoga. Jaisa ke Chapter 5 mein Listing 5-12 mein mention kiya gaya tha, dono traits derivable traits hain, is liye aam tor par yeh sirf itna seedha hota hai ke apni struct ya enum definition mein `#[derive(PartialEq, Debug)]` annotation add kar dein. In traits aur doosre derivable traits ke baare mein mazeed details ke liye Appendix C, [“Derivable Traits,”][derivable-traits]<!-- ignore --> dekhein.

### Custom Failure Messages Add Karna

Aap `assert!`, `assert_eq!`, aur `assert_ne!` macros ko optional arguments ke taur par custom message bhi de sakte hain, jo failure message ke sath print kiya jayega. Required arguments ke baad specify kiye gaye tamam arguments `format!` macro ko pass kiye jate hain (`format!` ke baare mein Chapter 8 mein [“Concatenating with `+` or `format!`”][concatenating]<!--
ignore --> mein discuss kiya gaya hai), is liye aap aisa format string pass kar sakte hain jis mein `{}` placeholders aur un placeholders mein rakhne ke liye values shamil hon. Custom messages is baat ko document karne ke liye useful hote hain ke assertion ka matlab kya hai; jab test fail hota hai, to aapko behtar idea hota hai ke code mein problem kya hai.

Misal ke taur par, maan lein ke hamare paas ek function hai jo logon ko unke naam se greet karta hai aur hum test karna chahte hain ke jo naam hum function mein pass karte hain woh output mein appear hota hai:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-05-greeter/src/lib.rs}}
```

Is program ke requirements par abhi tak agreement nahi hua, aur humein kaafi yaqeen hai ke greeting ke beginning mein `Hello` text change ho jayega. Humne decide kiya ke requirements change hone par humein test ko update na karna pade, is liye `greeting` function se return hone wali value ke sath exact equality check karne ke bajaye, hum sirf yeh assert karenge ke output mein input parameter ka text maujood hai.

Ab `greeting` ko change karke `name` ko exclude karte hue is code mein ek bug introduce karte hain, taake dekhein ke default test failure kaisa nazar aata hai:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/src/lib.rs:here}}
```

Is test ko run karne par yeh result milta hai:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/output.txt}}
```

Yeh result sirf itna indicate karta hai ke assertion fail hui aur yeh batata hai ke assertion kis line par hai. Zyada useful failure message `greeting` function se milne wali value ko print karega. Chaliye ek custom failure message add karte hain jo ek format string par mushtamil ho aur us mein actual value se placeholder fill kiya gaya ho jo humein `greeting` function se mili:

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/src/lib.rs:here}}
```

Ab jab hum test run karenge, to humein zyada informative error message milega:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/output.txt}}
```

Hum test output mein woh value dekh sakte hain jo humein waqai mili thi, jo humein yeh debug karne mein madad karegi ke asal mein kya hua, bajaye is ke ke hum sirf yeh dekhein ke hum kya hone ki expectation kar rahe thay.

### `should_panic` ke Sath Panics Check Karna

Return values ko check karne ke ilawa, yeh check karna bhi important hai ke hamara code error conditions ko us tarah handle karta hai jis tarah hum expect karte hain. Misal ke taur par, Chapter 9 mein Listing 9-13 mein banaye gaye `Guess` type ko dekhein. `Guess` ko use karne wala doosra code is guarantee par depend karta hai ke `Guess` instances mein sirf 1 aur 100 ke darmiyan values hongi. Hum ek aisa test likh sakte hain jo ensure kare ke is range se bahar ki value ke sath `Guess` instance create karne ki koshish panic karti hai.

Hum yeh apne test function mein `should_panic` attribute add karke karte hain. Agar function ke andar ka code panic karta hai to test pass hota hai; agar function ke andar ka code panic nahi karta to test fail hota hai.

Listing 11-8 mein ek aisa test dikhaya gaya hai jo check karta hai ke `Guess::new` ki error conditions tab hoti hain jab hum unki expectation karte hain.

<Listing number="11-8" file-name="src/lib.rs" caption="Yeh test karna ke koi condition `panic!` cause karegi">

```rust,noplayground id="f2c7la"
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-08/src/lib.rs}}
```

</Listing>

Hum `#[should_panic]` attribute ko `#[test]` attribute ke baad aur jis test function par yeh apply hota hai us se pehle place karte hain. Chaliye dekhte hain jab yeh test pass hota hai to result kya hota hai:

```console id="s4p2ko"
{{#include ../listings/ch11-writing-automated-tests/listing-11-08/output.txt}}
```

Sab theek lag raha hai! Ab apne code mein ek bug introduce karte hain, `new` function se woh condition remove karke jo value 100 se greater hone par panic karti hai:

```rust,not_desired_behavior,noplayground id="gj6qne"
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/src/lib.rs:here}}
```

Jab hum Listing 11-8 mein diye gaye test ko run karenge, to yeh fail ho jayega:

```console id="q3b8xy"
{{#include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/output.txt}}
```

Is case mein humein bohat helpful message nahi milta, lekin jab hum test function ko dekhte hain, to humein nazar aata hai ke is par `#[should_panic]` annotation laga hua hai. Jo failure humein mila hai us ka matlab hai ke test function ke andar ke code ne panic cause nahi ki.

`should_panic` ko use karne wale tests imprecise ho sakte hain. `should_panic` test tab bhi pass ho jayega agar test kisi aisi doosri wajah se panic kare jis ki hum expectation nahi kar rahe thay. `should_panic` tests ko zyada precise banane ke liye, hum `should_panic` attribute mein optional `expected` parameter add kar sakte hain. Test harness ensure karega ke failure message mein diya gaya text maujood ho. Misal ke taur par, Listing 11-9 mein `Guess` ke modified code ko dekhein, jahan `new` function is baat ke mutabiq different messages ke sath panic karta hai ke value bohat chhoti hai ya bohat bari.

<Listing number="11-9" file-name="src/lib.rs" caption="Aise `panic!` ko test karna jis ke panic message mein specified substring shamil ho">

```rust,noplayground id="v4m0zn"
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-09/src/lib.rs:here}}
```

</Listing>

Yeh test pass hoga kyun ke `should_panic` attribute ke `expected` parameter mein jo value humne rakhi hai woh us message ka ek substring hai jis ke sath `Guess::new` function panic karta hai. Hum poora panic message bhi specify kar sakte thay jis ki hum expectation karte hain; is case mein woh `Guess value must be less than or equal to 100, got 200` hota. Aap kya specify karte hain, yeh is baat par depend karta hai ke panic message ka kitna hissa unique ya dynamic hai aur aap apne test ko kitna precise rakhna chahte hain. Is case mein, panic message ka ek substring itna kaafi hai ke ensure kiya ja sake ke test function ka code `else if value > 100` case ko execute karta hai.

Yeh dekhne ke liye ke `expected` message wale `should_panic` test ke fail hone par kya hota hai, ek baar phir apne code mein bug introduce karte hain aur `if value < 1` aur `else if value > 100` blocks ke bodies ko aapas mein swap kar dete hain:

```rust,ignore,not_desired_behavior id="x8c4nr"
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/src/lib.rs:here}}
```

Is baar jab hum `should_panic` test run karenge, to yeh fail ho jayega:

```console id="m7v9pa"
{{#include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/output.txt}}
```

Failure message indicate karta hai ke yeh test waqai panic hua, jaisa ke hum expect kar rahe thay, lekin panic message mein expected string `less than or equal to 100` shamil nahi thi. Is case mein jo panic message humein mila woh `Guess value must be greater than or equal to 1, got 200` tha. Ab hum yeh pata lagana shuru kar sakte hain ke hamare code mein bug kahan hai!

### Tests mein `Result<T, E>` Use Karna

Ab tak hamare tamam tests fail hone par panic karte hain. Hum aise tests bhi likh sakte hain jo `Result<T, E>` use karte hain! Yahan Listing 11-1 ka test hai, jise `Result<T, E>` use karne aur panic karne ke bajaye `Err` return karne ke liye rewrite kiya gaya hai:

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-10-result-in-tests/src/lib.rs:here}}
```

`it_works` function ka return type ab `Result<(), String>` hai. Function ke body mein, `assert_eq!` macro call karne ke bajaye, test pass hone par hum `Ok(())` return karte hain aur test fail hone par andar ek `String` ke sath `Err` return karte hain.

Tests ko is tarah likhna ke woh `Result<T, E>` return karein, aapko tests ke body mein question mark operator use karne ki sahulat deta hai, jo aise tests likhne ka convenient tareeqa ho sakta hai jo tab fail hone chahiye jab unke andar koi bhi operation `Err` variant return kare.

Aap `Result<T, E>` use karne wale tests par `#[should_panic]` annotation use nahi kar sakte. Yeh assert karne ke liye ke koi operation `Err` variant return karta hai, `Result<T, E>` value par question mark operator *use na karein*. Is ke bajaye, `assert!(value.is_err())` use karein.

Ab jab aap tests likhne ke kai tareeqe jaante hain, to chaliye dekhte hain ke jab hum apne tests run karte hain to kya ho raha hota hai aur `cargo test` ke sath hum kaun se different options use kar sakte hain.

[concatenating]: ch08-02-strings.html#concatenating-with--or-format
[bench]: ../unstable-book/library-features/test.html
[ignoring]: ch11-02-running-tests.html#ignoring-tests-unless-specifically-requested
[subset]: ch11-02-running-tests.html#running-a-subset-of-tests-by-name
[controlling-how-tests-are-run]: ch11-02-running-tests.html#controlling-how-tests-are-run
[derivable-traits]: appendix-03-derivable-traits.html
[doc-comments]: ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests
[paths-for-referring-to-an-item-in-the-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
