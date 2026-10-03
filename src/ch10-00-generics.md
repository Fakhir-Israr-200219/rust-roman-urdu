# Generic Types, Traits, and Lifetimes

Har programming language mein concepts ki duplication ko effectively handle karne ke liye tools hote hain. Rust mein aisa hi ek tool *generics* hai: concrete types ya doosri properties ke liye abstract stand-ins. Hum generics ke behavior ya unka doosre generics ke saath relation is baat ko jaane baghair express kar sakte hain ke code compile aur run hone par unki jagah kya hoga.

Functions kisi concrete type jaise `i32` ya `String` ke bajaye kisi generic type ke parameters le sakti hain, bilkul usi tarah jaise woh unknown values ke parameters leti hain taa-ke same code ko multiple concrete values par run kiya ja sake. Asal mein, hum Chapter 6 mein `Option<T>` ke saath, Chapter 8 mein `Vec<T>` aur `HashMap<K, V>` ke saath, aur Chapter 9 mein `Result<T, E>` ke saath generics pehle hi use kar chuke hain. Is chapter mein aap explore karenge ke apne khud ke types, functions, aur methods ko generics ke saath kaise define kiya jata hai!

Sab se pehle, hum review karenge ke code duplication ko kam karne ke liye kisi function ko kaise extract kiya jata hai. Phir hum isi technique ko use karke do aise functions se ek generic function banayenge jo sirf apne parameters ke types mein different hain. Hum ye bhi explain karenge ke struct aur enum definitions mein generic types ko kaise use kiya jata hai.

Phir aap seekhenge ke generic tareeqe se behavior define karne ke liye traits ko kaise use kiya jata hai. Aap traits ko generic types ke saath combine karke kisi generic type ko sirf un types ko accept karne tak constrain kar sakte hain jin mein koi particular behavior ho, bajaye is ke ke woh kisi bhi type ko accept kare.

Aakhir mein, hum *lifetimes* par baat karenge: ye generics ki ek variety hain jo compiler ko is baare mein information deti hain ke references ek doosre se kis tarah related hain. Lifetimes humein borrowed values ke baare mein compiler ko itni information dene deti hain ke woh ye ensure kar sake ke references un situations mein bhi valid honge jahan hamari help ke baghair woh aisa ensure nahi kar sakta.

## Function Extract Karke Duplication Khatam Karna

Generics humein specific types ki jagah ek aisa placeholder use karne dete hain jo multiple types ko represent karta hai, aur is tarah code duplication ko khatam kiya ja sakta hai. Generics ki syntax mein dive karne se pehle, aaiye pehle dekhein ke generic types ko involve kiye baghair duplication ko kaise remove kiya jata hai. Is ke liye hum ek function extract karenge jo specific values ki jagah ek aisa placeholder use karta hai jo multiple values ko represent karta hai. Phir, hum isi technique ko apply karke ek generic function extract karenge! Ye dekh kar ke aise duplicated code ko kaise recognize kiya jata hai jise aap function mein extract kar sakte hain, aap aise duplicated code ko bhi recognize karna shuru kar denge jo generics ko use kar sakta hai.

Hum Listing 10-1 mein diye gaye short program se shuru karenge jo ek list mein sab se bara number find karta hai.

<Listing number="10-1" file-name="src/main.rs" caption="Finding the largest number in a list of numbers">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-01/src/main.rs:here}}
```

</Listing>

Hum integers ki ek list variable `number_list` mein store karte hain aur list ke pehle number ka reference `largest` naam ke variable mein rakhte hain. Phir hum list ke tamam numbers par iterate karte hain, aur agar current number `largest` mein stored number se bara ho, to hum us variable mein reference ko replace kar dete hain. Lekin agar current number ab tak dekhe gaye sab se bare number se chhota ya us ke barabar ho, to variable change nahi hota aur code list ke next number ki taraf chala jata hai. List ke tamam numbers ko consider karne ke baad, `largest` ko sab se bare number ko refer karna chahiye, jo is case mein 100 hai.

Ab humein numbers ki do different lists mein sab se bara number find karne ka task diya gaya hai. Is ke liye hum Listing 10-1 ke code ko duplicate karne aur program mein do different jagahon par same logic use karne ka intekhab kar sakte hain, jaisa ke Listing 10-2 mein dikhaya gaya hai.

<Listing number="10-2" file-name="src/main.rs" caption="Code to find the largest number in *two* lists of numbers">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-02/src/main.rs}}
```

</Listing>

Agarche ye code kaam karta hai, lekin code ko duplicate karna tedious aur error-prone hai. Jab hum is mein change karna chahte hain, to humein multiple places par code ko update karna bhi yaad rakhna padta hai.

Is duplication ko khatam karne ke liye, hum ek aisi abstraction create karenge jo kisi bhi list of integers ko, jo parameter ke taur par pass ki gayi ho, operate karne wala function define karti hai. Ye solution hamare code ko zyada clear banata hai aur humein list mein sab se bara number find karne ke concept ko abstract tareeqe se express karne deta hai.

Listing 10-3 mein, hum sab se bara number find karne wale code ko `largest` naam ke function mein extract karte hain. Phir, hum Listing 10-2 ki dono lists mein sab se bara number find karne ke liye function ko call karte hain. Hum future mein maujood kisi bhi doosri `i32` values ki list par bhi is function ko use kar sakte hain.

<Listing number="10-3" file-name="src/main.rs" caption="Abstracted code to find the largest number in two lists">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-03/src/main.rs:here}}
```

</Listing>

`largest` function ka ek parameter `list` hai, jo `i32` values ke kisi bhi concrete slice ko represent karta hai jise hum function mein pass kar sakte hain. Is ke result ke taur par, jab hum function ko call karte hain, to code un specific values par run hota hai jo hum pass karte hain.

Khulasa ye hai ke Listing 10-2 ke code ko Listing 10-3 mein change karne ke liye hum ne ye steps liye:

1. Duplicate code ko identify kiya.
2. Duplicate code ko function ki body mein extract kiya, aur function signature mein us code ke inputs aur return values specify kiye.
3. Duplicate code ke dono instances ko update karke unki jagah function ko call kiya.

Agla step ye hai ke hum code duplication ko reduce karne ke liye inhi steps ko generics ke saath use karenge. Jis tarah function ki body specific values ke bajaye ek abstract `list` par operate kar sakti hai, usi tarah generics code ko abstract types par operate karne dete hain.

Misal ke taur par, maan lein hamare paas do functions hon: ek jo `i32` values ke slice mein sab se bara item find karta ho aur doosra jo `char` values ke slice mein sab se bara item find karta ho. Hum is duplication ko kaise khatam karenge? Aaiye pata lagate hain!
