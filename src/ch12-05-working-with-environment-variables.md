## Working with Environment Variables

Hum `minigrep` binary ko ek extra feature add karke improve karenge: case-insensitive searching ka ek option jise user environment variable ke zariye on kar sakta hai. Hum is feature ko command line option bana sakte thay aur users ko har baar jab woh isay apply karna chahein, ise enter karne ki requirement rakh sakte thay, lekin iske bajaye ise environment variable banane se, hum apne users ko environment variable ek baar set karne ki sahulat dete hain aur us terminal session mein unki tamam searches case insensitive ho jati hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="writing-a-failing-test-for-the-case-insensitive-search-function"></a>

### Writing a Failing Test for Case-Insensitive Search

Sab se pehle hum `minigrep` library mein ek naya `search_case_insensitive` function add karte hain jo tab call kiya jayega jab environment variable ki koi value hogi. Hum TDD process ko follow karte rahenge, is liye pehla step dobara ek failing test likhna hai. Hum naye `search_case_insensitive` function ke liye ek naya test add karenge aur apne purane test ka naam `one_result` se change karke `case_sensitive` rakhenge, taake dono tests ke darmiyan differences zyada clear hon, jaisa ke Listing 12-20 mein dikhaya gaya hai.

<Listing number="12-20" file-name="src/lib.rs" caption="Adding a new failing test for the case-insensitive function we’re about to add">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-20/src/lib.rs:here}}
```

</Listing>

Note karein ke humne purane test ke `contents` ko bhi edit kiya hai. Humne text `"Duct tape."` ke saath ek nayi line add ki hai jismein capital *D* hai, jo case-sensitive manner mein search karte waqt query `"duct"` se match nahi honi chahiye. Purane test ko is tarah change karne se yeh ensure karne mein madad milti hai ke hum accidentally case-sensitive search ki us functionality ko break na kar dein jo hum pehle hi implement kar chuke hain. Yeh test ab pass hona chahiye aur jab hum case-insensitive search par kaam karein to bhi pass hota rehna chahiye.

Case-*insensitive* search ke liye naya test `"rUsT"` ko apni query ke taur par use karta hai. `search_case_insensitive` function mein jo hum ab add karne wale hain, query `"rUsT"` ko capital *R* wali `"Rust:"` line se match hona chahiye aur line `"Trust me."` se bhi match hona chahiye, chahe dono ki casing query se different hai. Yeh hamara failing test hai, aur yeh compile nahi hoga kyun ke humne abhi tak `search_case_insensitive` function define nahi kiya. Agar aap chahein to ek skeleton implementation add kar sakte hain jo hamesha ek empty vector return karti ho, bilkul usi tarah jaise humne Listing 12-16 mein `search` function ke liye kiya tha, taake test compile ho aur fail ho.

### Implementing the `search_case_insensitive` Function

`search_case_insensitive` function, jo Listing 12-21 mein dikhaya gaya hai, `search` function jaisa hi lagbhag hoga. Sirf farq yeh hai ke hum `query` aur har `line` ko lowercase karenge, taake input arguments ki casing chahe jo bhi ho, jab hum check karein ke line mein query contain hoti hai ya nahi, dono ki casing same ho.

<Listing number="12-21" file-name="src/lib.rs" caption="Defining the `search_case_insensitive` function to lowercase the query and the line before comparing them">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-21/src/lib.rs:here}}
```

</Listing>

Sab se pehle, hum `query` string ko lowercase karte hain aur usay isi naam ke ek naye variable mein store karte hain, jis se original `query` shadow ho jata hai. `query` par `to_lowercase` call karna is liye zaroori hai taake user ki query `"rust"`, `"RUST"`, `"Rust"`, ya `"rUsT"` ho, hum query ko `"rust"` ki tarah treat karein aur casing se beparwah rahein. `to_lowercase` basic Unicode ko handle karega, lekin yeh 100 percent accurate nahi hoga. Agar hum koi real application likh rahe hote, to hum yahan thoda aur kaam karna chahte, lekin yeh section environment variables ke baare mein hai, Unicode ke baare mein nahi, is liye hum yahan itna hi rakhenge.

Note karein ke `query` ab string slice ke bajaye ek `String` hai kyun ke `to_lowercase` existing data ko reference karne ke bajaye naya data create karta hai. Misal ke taur par, agar query `"rUsT"` hai: us string slice mein hamare use ke liye lowercase `u` ya `t` maujood nahi hai, is liye humein `"rust"` contain karne wali ek nayi `String` allocate karni padti hai. Jab hum ab `query` ko `contains` method mein argument ke taur par pass karte hain, to humein ampersand add karna padta hai kyun ke `contains` ki signature string slice lene ke liye defined hai.

Next, hum har `line` par `to_lowercase` ki call add karte hain taake tamam characters lowercase ho jayein. Ab jab humne `line` aur `query` ko lowercase mein convert kar diya hai, to query ki casing chahe jo bhi ho, hum matches find kar lenge.

Aaiye dekhein ke kya yeh implementation tests pass karti hai:

```console
{{#include ../listings/ch12-an-io-project/listing-12-21/output.txt}}
```

Great! Yeh pass ho gaye. Ab aaiye naye `search_case_insensitive` function ko `run` function se call karte hain. Sab se pehle, hum `Config` struct mein ek configuration option add karenge jo case-sensitive aur case-insensitive search ke darmiyan switch karega. Is field ko add karne se compiler errors aayenge kyun ke hum abhi kahin bhi is field ko initialize nahi kar rahe:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:here}}
```

Humne `ignore_case` field add ki hai jo ek Boolean hold karti hai. Next, humein `run` function ko `ignore_case` field ki value check karne ki zarurat hai aur us value ko use karke decide karna hai ke `search` function ko call karna hai ya `search_case_insensitive` function ko, jaisa ke Listing 12-22 mein dikhaya gaya hai. Yeh abhi bhi compile nahi hoga.

<Listing number="12-22" file-name="src/main.rs" caption="Calling either `search` or `search_case_insensitive` based on the value in `config.ignore_case`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:there}}
```

</Listing>

Aakhir mein, humein environment variable ko check karne ki zarurat hai. Environment variables ke saath kaam karne wale functions standard library ke `env` module mein hain, jo *src/main.rs* ke top par pehle hi scope mein hai. Hum `env` module se `var` function use karenge taake check kar sakein ke `IGNORE_CASE` naam ke environment variable ke liye koi value set ki gayi hai ya nahi, jaisa ke Listing 12-23 mein dikhaya gaya hai.

<Listing number="12-23" file-name="src/main.rs" caption="Checking for any value in an environment variable named `IGNORE_CASE`">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-23/src/main.rs:here}}
```

</Listing>

Yahan, hum ek naya variable, `ignore_case`, create karte hain. Iski value set karne ke liye, hum `env::var` function call karte hain aur usay `IGNORE_CASE` environment variable ka naam pass karte hain. `env::var` function ek `Result` return karta hai jo successful `Ok` variant hoga aur agar environment variable kisi bhi value par set hai to us value ko contain karega. Agar environment variable set nahi hai, to yeh `Err` variant return karega.

Hum `Result` par `is_ok` method use kar rahe hain taake check karein ke environment variable set hai ya nahi, jiska matlab hai ke program ko case-insensitive search karni chahiye. Agar `IGNORE_CASE` environment variable kisi value par set nahi hai, to `is_ok` `false` return karega aur program case-sensitive search perform karega. Humein environment variable ki *value* ki parwah nahi hai, sirf yeh matter karta hai ke woh set hai ya unset, is liye hum `unwrap`, `expect`, ya `Result` par dekhe gaye doosre methods use karne ke bajaye `is_ok` check kar rahe hain.

Hum `ignore_case` variable ki value `Config` instance ko pass karte hain taake `run` function us value ko read kar sake aur decide kar sake ke `search_case_insensitive` ko call karna hai ya `search`, jaisa ke humne Listing 12-22 mein implement kiya hai.

Aaiye isay try karte hain! Sab se pehle, hum apne program ko environment variable set kiye baghair aur `to` query ke saath run karenge, jo har us line se match karni chahiye jo all lowercase mein *to* word contain karti hai:

```console
{{#include ../listings/ch12-an-io-project/listing-12-23/output.txt}}
```

Lagta hai ke yeh ab bhi kaam kar raha hai! Ab aaiye program ko `IGNORE_CASE` ko `1` par set karke usi `to` query ke saath run karte hain:

```console
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

Agar aap PowerShell use kar rahe hain, to aapko environment variable set karne aur program run karne ko separate commands ke taur par karna hoga:

```console
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

Is se `IGNORE_CASE` aapke shell session ke baqi hisson ke liye persist karega. Isay `Remove-Item` cmdlet ke zariye unset kiya ja sakta hai:

```console
PS> Remove-Item Env:IGNORE_CASE
```

Humein aisi lines milni chahiye jo *to* contain karti hon aur jin mein uppercase letters bhi ho sakte hain:

<!-- manual-regeneration
cd listings/ch12-an-io-project/listing-12-23
IGNORE_CASE=1 cargo run -- to poem.txt
can't extract because of the environment variable
-->

```console
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

Excellent, humein *To* contain karne wali lines bhi mil gayi hain! Ab hamara `minigrep` program environment variable ke zariye controlled case-insensitive searching kar sakta hai. Ab aap jaante hain ke command line arguments ya environment variables ke zariye set kiye gaye options ko kaise manage karna hai.

Kuch programs ek hi configuration ke liye arguments *aur* environment variables dono allow karte hain. Aise cases mein, programs decide karte hain ke dono mein se kis ko precedence di jaye. Apni taraf se ek aur exercise ke taur par, case sensitivity ko command line argument ya environment variable ke zariye control karne ki koshish karein. Decide karein ke agar program ko ek ko case sensitive aur doosre ko ignore case ke liye set karke run kiya jaye, to command line argument ya environment variable mein se kis ko precedence milni chahiye.

`std::env` module mein environment variables ke saath deal karne ke liye aur bhi bohat se useful features hain: iski documentation check karein taake dekhein ke kya available hai.
