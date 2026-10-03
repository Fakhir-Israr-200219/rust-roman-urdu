## `panic!` Ke Saath Unrecoverable Errors

Kabhi kabhi aapke code mein buri situations paida ho jati hain, aur aap unke baare mein kuch nahi kar sakte. Aise cases mein Rust ke paas `panic!` macro hota hai. Amli taur par panic cause karne ke do tareeqe hain: aisa action lena jo hamare code ko panic karwa de (jaise array ke end se aage access karna), ya `panic!` macro ko explicitly call karna. Dono cases mein hum apne program mein panic cause karte hain. Default taur par, ye panics ek failure message print karenge, unwind karenge, stack ko clean up karenge, aur quit kar jayenge. Ek environment variable ke zariye, aap Rust ko panic hone par call stack display karne ke liye bhi keh sakte hain, taa-ke panic ke source ko track down karna aasaan ho.


> ### Panic Ke Response Mein Stack Ko Unwind Karna Ya Abort Karna
>
> Default taur par, jab panic hota hai, program *unwinding* shuru karta hai, jis ka matlab hai ke Rust stack par wapas upar jata hai aur har us function se data ko clean up karta hai jise woh encounter karta hai. Lekin wapas jana aur cleanup karna kaafi kaam hai. Is liye Rust aapko foran *aborting* ka alternative choose karne ki bhi ijazat deta hai, jo cleanup kiye baghair program ko end kar deta hai.
>
> Is ke baad program jis memory ko use kar raha tha, usay operating system ko clean up karna hoga. Agar aapke project mein resultant binary ko jitna mumkin ho sake chhota rakhna zaroori ho, to aap apni *Cargo.toml* file ke munasib `[profile]` sections mein `panic = 'abort'` add karke panic hone par unwinding se aborting par switch kar sakte hain. Misal ke taur par, agar aap release mode mein panic par abort karna chahte hain, to ye add karein:
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

Aaiye ek simple program mein `panic!` call karke dekhte hain:

<Listing file-name="src/main.rs">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-01-panic/src/main.rs}}
```

</Listing>

Jab aap program run karenge, to aapko kuch is tarah ka output nazar aayega:

```console
{{#include ../listings/ch09-error-handling/no-listing-01-panic/output.txt}}
```

`panic!` ki call aakhri do lines mein maujood error message ko cause karti hai. Pehli line hamara panic message aur hamare source code mein woh jagah dikhati hai jahan panic hua: *src/main.rs:2:5* indicate karta hai ke ye hamari *src/main.rs* file ki second line ka fifth character hai.

Is case mein, indicated line hamare apne code ka hissa hai, aur agar hum us line par jayein, to humein `panic!` macro ki call nazar aati hai. Doosre cases mein, `panic!` ki call us code mein ho sakti hai jise hamara code call karta hai, aur error message mein report kiya gaya filename aur line number us doosre code ki jagah hogi jahan `panic!` macro call hui hai, na ke hamare code ki woh line jis ne aakhirkar `panic!` call tak pohanchaya.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-a-panic-backtrace"></a>

Hum un functions ka backtrace use kar sakte hain jahan se `panic!` call aayi hai taa-ke apne code ke us hissa ko figure out kar saken jo problem cause kar raha hai. `panic!` backtrace ko use karne ka tareeqa samajhne ke liye, aaiye ek aur example dekhte hain aur dekhte hain ke us waqt kya hota hai jab `panic!` call hamare code se directly macro call karne ke bajaye hamare code mein kisi bug ki wajah se ek library se aati hai. Listing 9-1 mein aisa code hai jo vector mein valid indexes ki range se bahar kisi index ko access karne ki koshish karta hai.

<Listing number="9-1" file-name="src/main.rs" caption="Vector ke end se aage kisi element ko access karne ki koshish karna, jo `panic!` ki call cause karega">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-01/src/main.rs}}
```

</Listing>

Yahan, hum apne vector ke 100th element ko access karne ki koshish kar rahe hain (jo index 99 par hai kyun ke indexing zero se shuru hoti hai), lekin vector mein sirf teen elements hain. Is situation mein Rust panic karega. `[]` ko use karna ek element return karne ke liye hota hai, lekin agar aap invalid index pass karein, to Rust ke paas yahan return karne ke liye koi aisa element nahi hai jo correct ho.

C mein, data structure ke end se aage read karne ki koshish undefined behavior hoti hai. Aapko memory mein woh kuch bhi mil sakta hai jo data structure mein us element ke corresponding location par ho, chahe woh memory us structure ki na ho. Isay *buffer overread* kaha jata hai aur agar koi attacker index ko is tarah manipulate kar sake ke woh aisa data read kar le jise usay read karne ki permission nahi honi chahiye aur jo data structure ke baad stored ho, to ye security vulnerabilities ka sabab ban sakta hai.

Apne program ko is qisam ki vulnerability se protect karne ke liye, agar aap kisi aise index par element read karne ki koshish karein jo exist nahi karta, to Rust execution ko stop kar deta hai aur continue karne se inkar karta hai. Aaiye ise try karke dekhte hain:

```console
{{#include ../listings/ch09-error-handling/listing-09-01/output.txt}}
```

Ye error hamari *main.rs* ki line 4 ki taraf point karta hai jahan hum `v` mein vector ke index 99 ko access karne ki koshish karte hain.

`note:` line humein batati hai ke hum error cause karne wali exact situation ka backtrace hasil karne ke liye `RUST_BACKTRACE` environment variable set kar sakte hain. Ek *backtrace* un tamam functions ki list hoti hai jinhein is point tak pohanchne ke liye call kiya gaya hai. Rust mein backtraces doosri languages ki tarah kaam karte hain: Backtrace ko read karne ki key ye hai ke top se shuru karein aur tab tak read karein jab tak aapko woh files nazar na aa jayein jo aap ne khud likhi hain. Ye woh jagah hai jahan problem originate hui. Us spot ke upar wali lines woh code hain jise aapke code ne call kiya; neeche wali lines woh code hain jis ne aapke code ko call kiya. Ye before-and-after lines core Rust code, standard library code, ya un crates ko include kar sakti hain jinhein aap use kar rahe hain. Aaiye `RUST_BACKTRACE` environment variable ko `0` ke ilawa kisi bhi value par set karke backtrace hasil karne ki koshish karte hain. Listing 9-2 aapko nazar aane wale output jaisa output dikhati hai.

<!-- manual-regeneration
cd listings/ch09-error-handling/listing-09-01
RUST_BACKTRACE=1 cargo run
copy the backtrace output below
check the backtrace number mentioned in the text below the listing
-->

<Listing number="9-2" caption="`panic!` ki call se generate hone wala backtrace jab `RUST_BACKTRACE` environment variable set ho">

```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
stack backtrace:
   0: rust_begin_unwind
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/std/src/panicking.rs:692:5
   1: core::panicking::panic_fmt
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:75:14
   2: core::panicking::panic_bounds_check
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:273:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:274:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:16:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/alloc/src/vec/mod.rs:3361:9
   6: panic::main
             at ./src/main.rs:4:6
   7: core::ops::function::FnOnce::call_once
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

</Listing>

Ye kaafi zyada output hai! Aapko jo exact output milega woh aapke operating system aur Rust version ke mutabiq different ho sakta hai. Is information ke saath backtraces hasil karne ke liye debug symbols enabled hona zaroori hai. Jab hum `cargo build` ya `cargo run` ko `--release` flag ke baghair use karte hain, to debug symbols default taur par enabled hote hain, jaisa ke yahan hai.

Listing 9-2 ke output mein, backtrace ki line 6 hamare project ki us line ki taraf point karti hai jo problem cause kar rahi hai: *src/main.rs* ki line 4. Agar hum nahi chahte ke hamara program panic kare, to humein apni investigation us location se shuru karni chahiye jis ki taraf hamari likhi hui file ka zikr karne wali pehli line point karti hai. Listing 9-1 mein, jahan hum ne jaan boojh kar aisa code likha tha jo panic kare, panic ko fix karne ka tareeqa ye hai ke vector indexes ki range se bahar kisi element ko request na kiya jaye. Jab future mein aapka code panic kare, to aapko figure out karna hoga ke panic cause karne ke liye code kis values ke saath kya action perform kar raha hai aur us ke bajaye code ko kya karna chahiye.

Hum baad mein is chapter ke [“To `panic!` or Not to `panic!`”][to-panic-or-not-to-panic]<!-- ignore --> section mein `panic!` aur ye discuss karenge ke error conditions ko handle karne ke liye `panic!` ko kab use karna chahiye aur kab nahi. Agley section mein, hum dekhenge ke `Result` ko use karke error se kaise recover kiya jata hai.

[to-panic-or-not-to-panic]: ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic
