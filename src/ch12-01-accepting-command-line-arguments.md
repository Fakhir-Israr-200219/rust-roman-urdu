## Accepting Command Line Arguments

Aaiye, hamesha ki tarah, `cargo new` ke saath ek naya project create karte hain. Hum apne project ko `minigrep` kahenge taake ise `grep` tool se alag rakha ja sake jo shayad aapke system par pehle se maujood ho:

```console
$ cargo new minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

Pehla task `minigrep` ko uske do command line arguments accept karwana hai: file path aur search karne ke liye ek string. Yani, hum apne program ko `cargo run` ke saath run karne, do hyphens se yeh indicate karne ke qabil banana chahte hain ke following arguments `cargo` ke bajaye hamare program ke liye hain, phir search karne ke liye ek string aur us file ka path dena chahte hain jismein search karni hai, is tarah:

```console
$ cargo run -- searchstring example-filename.txt
```

Filhaal, `cargo new` se generate hua program un arguments ko process nahi kar sakta jo hum use dete hain. [crates.io](https://crates.io/) par kuch existing libraries hain jo command line arguments accept karne wala program likhne mein madad kar sakti hain, lekin kyun ke aap abhi yeh concept seekh rahe hain, aaiye is capability ko khud implement karte hain.

### Reading the Argument Values

`minigrep` ko command line arguments ki woh values read karne ke qabil banane ke liye jo hum usay pass karte hain, humein Rust ki standard library mein provide ki gayi `std::env::args` function ki zaroorat hogi. Yeh function `minigrep` ko pass kiye gaye command line arguments ka ek iterator return karta hai. Hum iterators ko [Chapter 13][ch13]<!-- ignore
--> mein poori tarah cover karenge. Filhaal, aapko iterators ke baare mein sirf do details jaanne ki zaroorat hai: Iterators values ki ek series produce karte hain, aur hum iterator par `collect` method call karke use ek collection, jaise vector, mein convert kar sakte hain, jo un tamam elements ko contain karta hai jo iterator produce karta hai.

Listing 12-1 mein diya gaya code aapke `minigrep` program ko usay pass kiye gaye kisi bhi command line arguments ko read karne aur phir un values ko ek vector mein collect karne ki ijazat deta hai.

<Listing number="12-1" file-name="src/main.rs" caption="Collecting the command line arguments into a vector and printing them">

```rust
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-01/src/main.rs}}
```

</Listing>

Sab se pehle, hum `use` statement ke zariye `std::env` module ko scope mein laate hain taake hum iski `args` function ko use kar saken. Note karein ke `std::env::args` function do levels ke modules ke andar nested hai. Jaisa ke humne [Chapter
7][ch7-idiomatic-use]<!-- ignore --> mein discuss kiya tha, jab desired function ek se zyada modules ke andar nested ho, to humne function ke bajaye parent module ko scope mein lane ka intekhab kiya hai. Aisa karne se hum `std::env` ki doosri functions ko bhi aasani se use kar sakte hain. Yeh `use std::env::args` add karne aur phir function ko sirf `args` ke naam se call karne ke muqable mein kam ambiguous bhi hai, kyun ke `args` ko aasani se kisi aisi function samjha ja sakta hai jo current module mein defined ho.


> ### The `args` Function and Invalid Unicode
>
> Note karein ke agar kisi argument mein invalid Unicode ho to `std::env::args` panic karega. Agar aapke program ko invalid Unicode par mushtamil arguments accept karne ki zaroorat ho, to iske bajaye `std::env::args_os` use karein. Yeh function ek aisa iterator return karta hai jo `String` values ke bajaye `OsString` values produce karta hai. Humne yahan simplicity ke liye `std::env::args` use karne ka intekhab kiya hai, kyun ke `OsString` values platform ke mutabiq different hoti hain aur `String` values ke muqable mein inke saath kaam karna zyada complex hota hai.

`main` ki pehli line mein hum `env::args` call karte hain, aur foran `collect` ko use karke iterator ko ek aise vector mein convert kar dete hain jo iterator se produce hone wali tamam values ko contain karta hai. Hum `collect` function ko bohat qisam ki collections create karne ke liye use kar sakte hain, is liye hum `args` ki type ko explicitly annotate karte hain taake specify ho ke hum strings ka ek vector chahte hain. Agarche Rust mein types ko annotate karne ki zaroorat bohat kam hi padti hai, `collect` un functions mein se ek hai jise aapko aksar annotate karna padta hai, kyun ke Rust yeh infer karne ke qabil nahi hota ke aap kis qisam ki collection chahte hain.

Aakhir mein, hum debug macro ko use karke vector ko print karte hain. Aaiye pehle code ko bina arguments ke aur phir do arguments ke saath run karke dekhte hain:

```console id="e3q6nt"
{{#include ../listings/ch12-an-io-project/listing-12-01/output.txt}}
```

```console id="k0q6dz"
{{#include ../listings/ch12-an-io-project/output-only-01-with-args/output.txt}}
```

Note karein ke vector ki pehli value `"target/debug/minigrep"` hai, jo hamari binary ka naam hai. Yeh C mein arguments list ke behavior se match karta hai, jo programs ko us naam ko use karne deta hai jis naam se unhein execution ke waqt invoke kiya gaya tha. Program name tak access hona aksar convenient hota hai, agar aap ise messages mein print karna chahte hon ya is baat ki bunyaad par program ka behavior change karna chahte hon ke program ko invoke karne ke liye kaunsa command line alias use kiya gaya tha. Lekin is chapter ke maqsad ke liye, hum ise ignore karenge aur sirf woh do arguments save karenge jin ki humein zaroorat hai.

### Saving the Argument Values in Variables

Program filhaal command line arguments ke taur par specify ki gayi values ko access kar sakta hai. Ab humein dono arguments ki values ko variables mein save karna hai taake hum program ke baqi hisson mein in values ko use kar saken. Hum yeh Listing 12-2 mein karte hain.

<Listing number="12-2" file-name="src/main.rs" caption="Creating variables to hold the query argument and file path argument">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-02/src/main.rs}}
```

</Listing>

Jaisa ke humne vector ko print karte waqt dekha, program ka naam `args[0]` par vector ki pehli value occupy karta hai, is liye hum arguments ko index 1 se start kar rahe hain. `minigrep` jo pehla argument leta hai woh woh string hai jise hum search kar rahe hain, is liye hum pehle argument ka ek reference `query` variable mein rakhte hain. Doosra argument file path hoga, is liye hum doosre argument ka ek reference `file_path` variable mein rakhte hain.

Hum filhaal in variables ki values ko print karte hain taake yeh prove ho sake ke code hamari expectation ke mutabiq kaam kar raha hai. Aaiye is program ko dobara `test` aur `sample.txt` arguments ke saath run karte hain:

```console
{{#include ../listings/ch12-an-io-project/listing-12-02/output.txt}}
```

Great, program kaam kar raha hai! Jin arguments ki humein zaroorat hai unki values sahi variables mein save ho rahi hain. Baad mein hum kuch potential erroneous situations ko handle karne ke liye error handling add karenge, jaise jab user koi arguments provide na kare; filhaal hum us situation ko ignore karenge aur iske bajaye file-reading capabilities add karne par kaam karenge.

[ch13]: ch13-00-functional-features.html
[ch7-idiomatic-use]: ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths
