## Tests Kaise Run Kiye Jate Hain, Isay Control Karna

Jis tarah `cargo run` aapke code ko compile karta hai aur phir resulting binary ko run karta hai, usi tarah `cargo test` aapke code ko test mode mein compile karta hai aur phir resulting test binary ko run karta hai. `cargo test` se produce hone wali binary ka default behavior yeh hai ke woh tamam tests ko parallel mein run karti hai aur test runs ke dauran generate hone wale output ko capture karti hai, jis se output display hone se ruk jata hai aur test results se related output ko read karna aasaan ho jata hai. Lekin aap command line options specify karke is default behavior ko change kar sakte hain.

Kuch command line options `cargo test` ko jate hain, aur kuch resulting test binary ko. In dono types ke arguments ko alag karne ke liye, aap pehle `cargo test` ko jane wale arguments list karte hain, phir separator `--` aur us ke baad woh arguments jo test binary ko jane hain. `cargo test --help` run karne se woh options display hote hain jo aap `cargo test` ke sath use kar sakte hain, aur `cargo test -- --help` run karne se woh options display hote hain jo separator ke baad use kiye ja sakte hain. Yeh options [the “Tests” section of *The `rustc` Book*][tests] mein bhi documented hain.

[tests]: https://doc.rust-lang.org/rustc/tests/index.html

### Tests Ko Parallel Ya Consecutively Run Karna

Jab aap multiple tests run karte hain, to default taur par woh threads ko use karte hue parallel mein run hote hain, jis ka matlab hai ke woh zyada jaldi complete hote hain aur aapko feedback bhi jaldi milta hai. Kyun ke tests ek hi waqt mein run ho rahe hote hain, is liye aapko ensure karna hota hai ke aapke tests ek doosre par ya kisi shared state par depend na karein, jis mein shared environment bhi shamil hai, jaise current working directory ya environment variables.

Misal ke taur par, maan lein ke aapka har test kuch aisa code run karta hai jo disk par *test-output.txt* naam ki ek file create karta hai aur us file mein kuch data write karta hai. Phir har test us file se data read karta hai aur assert karta hai ke file mein ek particular value hai, jo har test mein different hai. Kyun ke tests ek hi waqt mein run hote hain, ek test us waqt file ko overwrite kar sakta hai jab doosra test file mein write karne aur usay read karne ke darmiyan ho. Is surat mein doosra test fail ho jayega, code incorrect hone ki wajah se nahi, balki is liye ke tests ne parallel mein run hote hue ek doosre ke kaam mein interference ki. Is ka ek solution yeh hai ke ensure kiya jaye ke har test different file mein write kare; doosra solution yeh hai ke tests ko ek waqt mein sirf ek run kiya jaye.

Agar aap tests ko parallel mein run nahi karna chahte ya aap use hone wale threads ki tadaad par zyada fine-grained control chahte hain, to aap test binary ko `--test-threads` flag aur un threads ki tadaad bhej sakte hain jinhein aap use karna chahte hain. Neeche di gayi example ko dekhein:

```console id="6n7w0x"
$ cargo test -- --test-threads=1
```

Humne test threads ki tadaad `1` set ki hai, jis se program ko bataya ja raha hai ke woh parallelism use na kare. Tests ko ek thread use karke run karne mein unhein parallel mein run karne ke muqable mein zyada waqt lagega, lekin agar tests shared state use karte hain to woh ek doosre ke kaam mein interference nahi karenge.

### Function Output Dikhana

Default taur par, agar koi test pass ho jaye, to Rust ki test library standard output par print hone wali har cheez ko capture kar leti hai. Misal ke taur par, agar hum test mein `println!` call karein aur test pass ho jaye, to humein terminal mein `println!` ka output nazar nahi aayega; humein sirf woh line nazar aayegi jo batati hai ke test pass ho gaya. Agar test fail ho jaye, to humein standard output par print hone wali cheez failure message ke baqi hisson ke sath nazar aayegi.

Misal ke taur par, Listing 11-10 mein ek simple si function hai jo apne parameter ki value print karti hai aur `10` return karti hai, aur is ke sath ek aisa test hai jo pass hota hai aur ek aisa test hai jo fail hota hai.

<Listing number="11-10" file-name="src/lib.rs" caption="Aise function ke tests jo `println!` call karta hai">

```rust,panics,noplayground id="f8q2mk"
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-10/src/lib.rs}}
```

</Listing>

Jab hum in tests ko `cargo test` ke sath run karte hain, to humein following output nazar aata hai:

```console id="r6k1zt"
{{#include ../listings/ch11-writing-automated-tests/listing-11-10/output.txt}}
```

Note karein ke is output mein kahin bhi `I got the value 4` nazar nahi aata, jo us waqt print hota hai jab pass hone wala test run hota hai. Us output ko capture kar liya gaya hai. Jo test fail hua us ka output, `I got the value 8`, test summary output ke section mein nazar aata hai, jahan test failure ki wajah bhi dikhai jati hai.

Agar hum passing tests ke liye bhi printed values dekhna chahte hain, to hum Rust ko `--show-output` ke sath successful tests ka output bhi dikhane ke liye keh sakte hain:

```console
$ cargo test -- --show-output
```

Jab hum Listing 11-10 ke tests ko `--show-output` flag ke sath dobara run karte hain, to humein following output nazar aata hai:

```console id="k3v9pa"
{{#include ../listings/ch11-writing-automated-tests/output-only-01-show-output/output.txt}}
```

### Naam Ke Zariye Tests Ke Ek Subset Ko Run Karna

Kabhi kabhi poori test suite ko run karne mein kaafi waqt lag sakta hai. Agar aap code ke kisi particular area par kaam kar rahe hain, to aap shayad sirf un tests ko run karna chahein jo us code se related hain. Aap `cargo test` ko un test(s) ka naam ya naam dekar choose kar sakte hain jinhein aap argument ke taur par run karna chahte hain.

Tests ke ek subset ko run karne ka tareeqa demonstrate karne ke liye, hum pehle apne `add_two` function ke liye teen tests create karenge, jaisa ke Listing 11-11 mein dikhaya gaya hai, aur phir choose karenge ke kaun se tests run karne hain.

<Listing number="11-11" file-name="src/lib.rs" caption="Teen tests jin ke teen different names hain">

```rust,noplayground id="q7m2cx"
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-11/src/lib.rs}}
```

</Listing>

Agar hum koi arguments pass kiye baghair tests run karein, jaisa ke humne pehle dekha tha, to tamam tests parallel mein run honge:

```console id="n5x8wr"
{{#include ../listings/ch11-writing-automated-tests/listing-11-11/output.txt}}
```

#### Single Tests Run Karna

Hum kisi bhi test function ka naam `cargo test` ko dekar sirf usi test ko run kar sakte hain:

```console id="q2k7mf"
{{#include ../listings/ch11-writing-automated-tests/output-only-02-single-test/output.txt}}
```

Sirf `one_hundred` naam wala test run hua; baqi dono tests is naam se match nahi hue. Test output humein yeh batata hai ke aur bhi tests thay jo run nahi hue, aur end mein `2 filtered out` display hota hai.

Hum is tareeqe se multiple tests ke names specify nahi kar sakte; `cargo test` ko di gayi sirf pehli value use ki jayegi. Lekin multiple tests run karne ka ek tareeqa maujood hai.

#### Multiple Tests Run Karne Ke Liye Filtering

Hum test name ka sirf ek hissa specify kar sakte hain, aur jis bhi test ka naam us value se match karega woh run kiya jayega. Misal ke taur par, kyun ke hamare do tests ke names mein `add` shamil hai, hum `cargo test add` run karke un dono tests ko run kar sakte hain:

```console id="w6j3qa"
{{#include ../listings/ch11-writing-automated-tests/output-only-03-multiple-tests/output.txt}}
```

Is command ne un tamam tests ko run kiya jin ke naam mein `add` tha aur `one_hundred` naam wale test ko filter out kar diya. Yeh bhi note karein ke jis module mein koi test hota hai, woh module bhi test ke naam ka hissa ban jata hai, is liye hum module ke naam par filtering karke us module ke tamam tests run kar sakte hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-some-tests-unless-specifically-requested"></a>

### Tests Ko Ignore Karna Jab Tak Khaas Taur Par Request Na Ki Jaye

Kabhi kabhi kuch specific tests ko execute karne mein bohat zyada waqt lag sakta hai, is liye aap `cargo test` ke zyada tar runs ke dauran unhein exclude karna chah sakte hain. Un tamam tests ko arguments ke taur par list karne ke bajaye jinhein aap run karna chahte hain, aap time-consuming tests par `ignore` attribute laga kar unhein exclude kar sakte hain, jaisa ke yahan dikhaya gaya hai:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground id="j6t4xp"
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/src/lib.rs:here}}
```

`#[test]` ke baad hum us test ke liye `#[ignore]` line add karte hain jise hum exclude karna chahte hain. Ab jab hum apne tests run karte hain, `it_works` run hota hai, lekin `expensive_test` nahi:

```console id="c8m2vz"
{{#include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/output.txt}}
```

`expensive_test` function ko `ignored` ke taur par list kiya gaya hai. Agar hum sirf ignored tests ko run karna chahte hain, to hum `cargo test -- --ignored` use kar sakte hain:

```console id="p5r9kw"
{{#include ../listings/ch11-writing-automated-tests/output-only-04-running-ignored/output.txt}}
```

Yeh control karke ke kaun se tests run hon, aap ensure kar sakte hain ke aapke `cargo test` ke results jaldi return hon. Jab aap aisi stage par hon jahan `ignored` tests ke results check karna munasib ho aur aapke paas results ka intezar karne ka waqt ho, to aap is ke bajaye `cargo test -- --ignored` run kar sakte hain. Agar aap tamam tests run karna chahte hain, chahe woh ignored hon ya na hon, to aap `cargo test -- --include-ignored` run kar sakte hain.
