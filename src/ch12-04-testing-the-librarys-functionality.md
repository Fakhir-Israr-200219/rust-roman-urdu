<!-- Old headings. Do not remove or links may break. -->

<a id="developing-the-librarys-functionality-with-test-driven-development"></a>

## Adding Functionality with Test-Driven Development

Ab jab hamare paas *src/lib.rs* mein `main` function se separate search logic hai, to hamare code ki core functionality ke liye tests likhna kaafi asaan ho gaya hai. Hum functions ko mukhtalif arguments ke saath directly call kar sakte hain aur return values ko check kar sakte hain, bina command line se apni binary ko call kiye.

Is section mein, hum following test-driven development (TDD) process ko use karte hue `minigrep` program mein searching logic add karenge:

1. Ek aisa test likhein jo fail ho aur use run karein taake yakeen ho jaye ke woh usi reason ki wajah se fail hota hai jiski aap expectation kar rahe hain.
2. Naye test ko pass karwane ke liye sirf itna code likhein ya modify karein jitna zaroori ho.
3. Abhi jo code aapne add ya change kiya hai uski refactor karein aur yakeen karein ke tests pass karte rahen.
4. Step 1 se dobara repeat karein!

Software likhne ke bohat se tareeqon mein se TDD sirf ek tareeqa hai, lekin yeh code design ko drive karne mein madad kar sakta hai. Test ko us code se pehle likhna jo test ko pass karwata hai, poore process ke dauran high test coverage maintain karne mein madad karta hai.

Hum us functionality ki implementation ko test-drive karenge jo asal mein file ke contents mein query string ko search karegi aur un lines ki list produce karegi jo query se match karti hain. Hum yeh functionality `search` naam ke ek function mein add karenge.

### Writing a Failing Test

*src/lib.rs* mein, hum ek `tests` module add karenge jismein ek test function hoga, jaisa ke humne [Chapter 11][ch11-anatomy]<!-- ignore --> mein kiya tha. Test function woh behavior specify karta hai jo hum `search` function se chahte hain: yeh ek query aur search karne ke liye text lega, aur sirf woh lines return karega jo text mein query contain karti hain. Listing 12-15 is test ko dikhati hai.

<Listing number="12-15" file-name="src/lib.rs" caption="Creating a failing test for the `search` function for the functionality we wish we had">

```rust,ignore,does_not_compile id="2xk8sf"
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-15/src/lib.rs:here}}
```

</Listing>

Yeh test `"duct"` string ko search karta hai. Jis text ko hum search kar rahe hain woh teen lines par mushtamil hai, jin mein se sirf ek mein `"duct"` maujood hai (note karein ke opening double quote ke baad backslash Rust ko batata hai ke is string literal ke contents ke start mein newline character na dale). Hum assert karte hain ke `search` function se return hone wali value mein sirf woh line ho jiski humein expectation hai.

Agar hum is test ko run karein, to yeh filhaal fail hoga kyun ke `unimplemented!` macro “not implemented” message ke saath panic karta hai. TDD principles ke mutabiq, hum sirf itna code add karne ka chhota step lenge ke function ko call karte waqt test panic na kare. Iske liye hum `search` function ko aise define karenge ke woh hamesha ek empty vector return kare, jaisa ke Listing 12-16 mein dikhaya gaya hai. Phir test compile ho jana chahiye aur fail hona chahiye kyun ke ek empty vector us vector se match nahi karta jismein `"safe, fast, productive."` line ho.

<Listing number="12-16" file-name="src/lib.rs" caption="Defining just enough of the `search` function so that calling it won’t panic">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-16/src/lib.rs:here}}
```

</Listing>

Ab aaiye discuss karte hain ke humein `search` ke signature mein ek explicit lifetime `'a` define karne aur us lifetime ko `contents` argument aur return value ke saath use karne ki zarurat kyun hai. Yaad karein ke [Chapter 10][ch10-lifetimes]<!-- ignore --> mein lifetime parameters specify karte hain ke kaunsa argument lifetime return value ke lifetime ke saath connected hai. Is case mein, hum indicate karte hain ke returned vector mein aise string slices hone chahiye jo argument `contents` ke slices ko reference karte hon (`query` argument ko nahi).

Doosre lafzon mein, hum Rust ko batate hain ke `search` function se return hone wala data utni der tak live rahega jitni der `search` function mein `contents` argument ke zariye pass kiya gaya data live rahega. Yeh important hai! Jis data ko ek slice *reference* karta hai, woh reference ke valid hone ke liye valid hona chahiye; agar compiler yeh assume kare ke hum `contents` ke bajaye `query` ke string slices bana rahe hain, to woh apni safety checking incorrectly karega.

Agar hum lifetime annotations bhool jayein aur is function ko compile karne ki koshish karein, to humein yeh error milega:

```console
{{#include ../listings/ch12-an-io-project/output-only-02-missing-lifetimes/output.txt}}
```

Rust nahi jaan sakta ke output ke liye humein dono parameters mein se kis parameter ki zarurat hai, is liye humein ise explicitly batana padta hai. Note karein ke help text suggest karta hai ke tamam parameters aur output type ke liye same lifetime parameter specify kiya jaye, jo incorrect hai! Kyun ke `contents` woh parameter hai jo hamara tamam text contain karta hai aur hum us text ke woh parts return karna chahte hain jo match karte hain, is liye hum jaante hain ke `contents` hi woh single parameter hai jise lifetime syntax ke zariye return value ke saath connected hona chahiye.

Doosri programming languages mein signature ke andar arguments ko return values ke saath connect karna required nahi hota, lekin waqt ke saath yeh practice asaan hoti jayegi. Aap is example ko Chapter 10 ke [“Validating References with Lifetimes”][validating-references-with-lifetimes]<!-- ignore --> section ke examples ke saath compare karna chah sakte hain.


### Writing Code to Pass the Test

Filhaal, hamara test fail ho raha hai kyun ke hum hamesha ek empty vector return karte hain. Isko fix karne aur `search` ko implement karne ke liye, hamare program ko yeh steps follow karne honge:

1. `contents` ki har line ke through iterate karein.
2. Check karein ke kya line mein hamari query string maujood hai.
3. Agar hai, to usay un values ki list mein add karein jo hum return kar rahe hain.
4. Agar nahi hai, to kuch na karein.
5. Un tamam results ki list return karein jo match karte hain.

Aaiye har step ko samajhte hain, sab se pehle lines ke through iterate karne se shuru karte hain.

#### Iterating Through Lines with the `lines` Method

Rust mein strings ki line-by-line iteration ko handle karne ke liye ek helpful method hai, jiska naam conveniently `lines` hai, aur jo Listing 12-17 mein dikhaye gaye tareeqe se kaam karta hai. Note karein ke yeh abhi compile nahi hoga.

<Listing number="12-17" file-name="src/lib.rs" caption="Iterating through each line in `contents`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-17/src/lib.rs:here}}
```

</Listing>

`lines` method ek iterator return karta hai. Hum [Chapter 13][ch13-iterators]<!-- ignore --> mein iterators ke baare mein detail mein baat karenge. Lekin yaad karein ke aapne [Listing 3-5][ch3-iter]<!-- ignore --> mein iterator ko is tarah use karte hue dekha tha, jahan humne collection ke har item par kuch code run karne ke liye `for` loop ko iterator ke saath use kiya tha.

#### Searching Each Line for the Query

Next, hum check karenge ke kya current line mein hamari query string maujood hai. Khush qismati se, strings mein `contains` naam ka ek helpful method hota hai jo yeh kaam hamare liye karta hai! `search` function mein `contains` method ki call add karein, jaisa ke Listing 12-18 mein dikhaya gaya hai. Note karein ke yeh abhi bhi compile nahi hoga.

<Listing number="12-18" file-name="src/lib.rs" caption="Adding functionality to see whether the line contains the string in `query`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-18/src/lib.rs:here}}
```

</Listing>

Filhaal, hum functionality build kar rahe hain. Code ko compile karne ke liye, humein function signature mein jaisa indicate kiya tha, uske mutabiq body se ek value return karni hogi.

#### Storing Matching Lines

Is function ko complete karne ke liye, humein matching lines ko store karne ka koi tareeqa chahiye jinhein hum return karna chahte hain. Iske liye, hum `for` loop se pehle ek mutable vector bana sakte hain aur vector mein ek `line` ko store karne ke liye `push` method call kar sakte hain. `for` loop ke baad, hum vector return karte hain, jaisa ke Listing 12-19 mein dikhaya gaya hai.

<Listing number="12-19" file-name="src/lib.rs" caption="Storing the lines that match so that we can return them">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:here}}
```

</Listing>

Ab `search` function ko sirf woh lines return karni chahiye jo `query` contain karti hain, aur hamara test pass hona chahiye. Aaiye test run karte hain:

```console
{{#include ../listings/ch12-an-io-project/listing-12-19/output.txt}}
```

Hamara test pass ho gaya, is liye humein pata hai ke yeh kaam karta hai!

Is point par, hum test ko pass rakhte hue `search` function ki implementation ko refactor karne ke opportunities par ghour kar sakte hain, taake wahi functionality maintain rahe. `search` function ka code itna bura nahi hai, lekin yeh iterators ke kuch useful features ka faida nahi uthata. Hum [Chapter 13][ch13-iterators]<!-- ignore --> mein is example ki taraf wapas aayenge, jahan hum iterators ko detail mein explore karenge aur dekhenge ke isay kaise improve kiya ja sakta hai.

Ab poora program kaam karna chahiye! Aaiye isay try karte hain, sab se pehle aise word ke saath jo Emily Dickinson ki poem se exactly ek line return karna chahiye: *frog*.

```console
{{#include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/output.txt}}
```

Cool! Ab aaiye aisa word try karte hain jo multiple lines se match karega, jaise *body*:

```console
{{#include ../listings/ch12-an-io-project/output-only-03-multiple-matches/output.txt}}
```

Aur aakhir mein, aaiye ensure karte hain ke jab hum aise word ko search karein jo poem mein kahin bhi maujood nahi hai, jaise *monomorphization*, to humein koi line na mile:

```console
{{#include ../listings/ch12-an-io-project/output-only-04-no-matches/output.txt}}
```

Excellent! Humne classic tool ka apna mini version build kar liya hai aur applications ko structure karne ke tareeqe ke baare mein bohat kuch seekha hai. Humne file input aur output, lifetimes, testing, aur command line parsing ke baare mein bhi kuch seekha hai.

Is project ko complete karne ke liye, hum briefly demonstrate karenge ke environment variables ke saath kaise kaam karna hai aur standard error par kaise print karna hai; dono cheezen command line programs likhte waqt useful hoti hain.

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[ch11-anatomy]: ch11-01-writing-tests.html#the-anatomy-of-a-test-function
[ch10-lifetimes]: ch10-03-lifetime-syntax.html
[ch3-iter]: ch03-05-control-flow.html#looping-through-a-collection-with-for
[ch13-iterators]: ch13-02-iterators.html
