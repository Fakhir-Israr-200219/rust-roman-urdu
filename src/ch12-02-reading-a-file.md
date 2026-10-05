## Reading a File

Ab hum `file_path` argument mein specified file ko read karne ki functionality add karenge. Sab se pehle, humein ise test karne ke liye ek sample file ki zaroorat hai: Hum ek aisi file use karenge jismein multiple lines par thori si text ho aur kuch repeated words hon. Listing 12-3 mein Emily Dickinson ki ek poem hai jo is kaam ke liye achhi rahegi! Apne project ke root level par *poem.txt* naam ki ek file create karein, aur us mein poem “I’m Nobody! Who are you?” enter karein.

<Listing number="12-3" file-name="poem.txt" caption="A poem by Emily Dickinson makes a good test case.">

```text
{{#include ../listings/ch12-an-io-project/listing-12-03/poem.txt}}
```

</Listing>

Text ko place karne ke baad, *src/main.rs* ko edit karein aur file ko read karne ke liye code add karein, jaisa ke Listing 12-4 mein dikhaya gaya hai.

<Listing number="12-4" file-name="src/main.rs" caption="Reading the contents of the file specified by the second argument">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/src/main.rs:here}}
```

</Listing>

Sab se pehle, hum `use` statement ke zariye standard library ka ek relevant hissa scope mein laate hain: Files ko handle karne ke liye humein `std::fs` ki zaroorat hai.

`main` mein, naya statement `fs::read_to_string` `file_path` ko leta hai, us file ko open karta hai, aur `std::io::Result<String>` type ki ek value return karta hai jo file ke contents ko contain karti hai.

Iske baad, hum dobara ek temporary `println!` statement add karte hain jo file read hone ke baad `contents` ki value ko print karta hai, taake hum check kar saken ke program ab tak sahi kaam kar raha hai.

Aaiye is code ko pehle command line argument ke taur par kisi bhi string ke saath (kyun ke humne abhi searching wala hissa implement nahi kiya) aur doosre argument ke taur par *poem.txt* file ke saath run karte hain:

```console
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/output.txt}}
```

Great! Code ne file ke contents ko read kiya aur phir print kar diya. Lekin code mein kuch flaws hain. Filhaal, `main` function ki multiple responsibilities hain: Aam taur par, functions zyada clear aur maintain karne mein aasaan hote hain agar har function sirf ek idea ke liye responsible ho. Doosra problem yeh hai ke hum errors ko utni achhi tarah handle nahi kar rahe jitna hum kar sakte hain. Program abhi chhota hai, is liye yeh flaws koi bara problem nahi hain, lekin jaise jaise program grow karega, inhein cleanly fix karna mushkil hoga. Program develop karte waqt early stage par refactoring shuru karna ek achhi practice hai, kyun ke chhoti miktar mein code ko refactor karna kaafi aasaan hota hai. Ab hum yahi karenge.
