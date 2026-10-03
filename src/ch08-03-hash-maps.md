## Hash Maps Mein Keys Ko Unki Associated Values Ke Saath Store Karna

Hamari common collections mein aakhri collection hash map hai. `HashMap<K, V>` type `_hashing function_` ko use karte hue `K` type ki keys ko `V` type ki values ke saath map karti hai, jo ye determine karta hai ke ye keys aur values memory mein kis tarah place hongi. Bohat si programming languages is qisam ki data structure ko support karti hain, lekin aksar in ke liye different names use karti hain, jaise *hash*, *map*, *object*, *hash table*, *dictionary*, ya *associative array*, aur bhi kai names hain.

Hash maps us waqt useful hoti hain jab aap data ko vector ki tarah index use karke nahi, balki ek aisi key use karke lookup karna chahte hain jo kisi bhi type ki ho sakti hai. Misal ke taur par, ek game mein aap har team ka score ek hash map mein track kar sakte hain, jahan har key team ka naam ho aur values har team ka score hon. Team ka naam de kar aap us ka score retrieve kar sakte hain.

Is section mein hum hash maps ki basic API dekhenge, lekin `HashMap<K, V>` par standard library ki taraf se define kiye gaye functions mein aur bhi bohat si useful cheezen maujood hain. Hamesha ki tarah, mazeed maloomat ke liye standard library ki documentation zaroor dekhein.

### Naya Hash Map Create Karna

Ek empty hash map create karne ka ek tareeqa `new` use karna aur `insert` ke zariye elements add karna hai. Listing 8-20 mein hum do teams ke scores track kar rahe hain, jin ke names *Blue* aur *Yellow* hain. Blue team 10 points ke saath start karti hai, aur Yellow team 50 points ke saath.

<Listing number="8-20" caption="Ek naya hash map create karna aur kuch keys aur values insert karna">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-20/src/main.rs:here}}
```

</Listing>

Note karein ke humein sab se pehle standard library ke collections wale portion se `HashMap` ko `use` karna padta hai. Hamari teen common collections mein se, ye sab se kam use hone wali collection hai, is liye ye un features mein shamil nahi hai jo prelude mein automatically scope mein laaye jate hain. Hash maps ko standard library ki taraf se bhi kam support hasil hai; misal ke taur par, inhein construct karne ke liye koi built-in macro nahi hai.

Vectors ki tarah, hash maps bhi apna data heap par store karti hain. Is `HashMap` mein keys `String` type ki hain aur values `i32` type ki. Vectors ki tarah, hash maps homogeneous hoti hain: tamam keys ka type same hona zaroori hai, aur tamam values ka type bhi same hona zaroori hai.

### Hash Map Mein Values Access Karna

Hum hash map se kisi value ko hasil karne ke liye uski key `get` method ko provide kar sakte hain, jaisa ke Listing 8-21 mein dikhaya gaya hai.

<Listing number="8-21" caption="Hash map mein store ki gayi Blue team ka score access karna">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-21/src/main.rs:here}}
```

</Listing>

Yahan, `score` mein woh value hogi jo Blue team ke saath associated hai, aur result `10` hoga. `get` method ek `Option<&V>` return karti hai; agar hash map mein us key ke liye koi value na ho, to `get` `None` return karegi. Ye program `copied` ko call karke `Option<&i32>` ke bajaye `Option<i32>` hasil karta hai, phir `unwrap_or` use karke agar `scores` mein us key ki koi entry na ho to `score` ko zero set karta hai, aur is tarah `Option` ko handle karta hai.

Hum hash map mein maujood har key-value pair par bhi vectors ki tarah `for` loop use karke iterate kar sakte hain:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-03-iterate-over-hashmap/src/main.rs:here}}
```

Ye code har pair ko kisi bhi arbitrary order mein print karega:

```text
Yellow: 50
Blue: 10
```

<!-- Old headings. Do not remove or links may break. -->

<a id="hash-maps-and-ownership"></a>

### Hash Maps Mein Ownership Manage Karna

Jo types `Copy` trait implement karti hain, jaise `i32`, unki values hash map mein copy ho jati hain. `String` jaisi owned values ke liye, values move ho jati hain aur hash map un values ki owner ban jata hai, jaisa ke Listing 8-22 mein dikhaya gaya hai.

<Listing number="8-22" caption="Ye dikhana ke insert hone ke baad keys aur values hash map ki ownership mein hoti hain">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-22/src/main.rs:here}}
```

</Listing>

Hum variables `field_name` aur `field_value` ko us waqt use nahi kar sakte jab woh `insert` call ke zariye hash map mein move ho chuke hon.

Agar hum hash map mein values ke references insert karein, to values hash map mein move nahi hongi. Jin values ki taraf references point karte hain, unka kam az kam utni dair valid rehna zaroori hai jitni dair hash map valid hai. Hum Chapter 10 mein [“Validating References with
Lifetimes”][validating-references-with-lifetimes]<!-- ignore --> mein in issues ke baare mein mazeed baat karenge.

### Hash Map Ko Update Karna

Key aur value pairs ki tadaad barh sakti hai, lekin har unique key ke saath ek waqt mein sirf ek value associated ho sakti hai (lekin iska ulta zaroori nahi: misal ke taur par, Blue team aur Yellow team dono ki value `10` ho sakti hai jo `scores` hash map mein stored ho).

Jab aap hash map mein data change karna chahte hain, to aapko ye decide karna hota hai ke us situation ko kis tarah handle karna hai jab kisi key ke saath pehle se ek value assigned ho. Aap purani value ko nayi value se replace kar sakte hain aur purani value ko bilkul ignore kar sakte hain. Aap purani value ko rakh kar nayi value ko ignore kar sakte hain, aur sirf us waqt nayi value add kar sakte hain jab key ke saath pehle se koi value *assigned na ho*. Ya aap purani aur nayi value ko combine kar sakte hain. Aaiye dekhein ke in mein se har ek kaise kiya jata hai!

#### Value Ko Overwrite Karna

Agar hum hash map mein ek key aur value insert karein aur phir usi key ko kisi different value ke saath dobara insert karein, to us key ke saath associated value replace ho jayegi. Halanke Listing 8-23 mein code `insert` ko do baar call karta hai, hash map mein sirf ek key-value pair hoga, kyun ke dono baar hum Blue team ki key ke liye value insert kar rahe hain.

<Listing number="8-23" caption="Kisi specific key ke saath stored value ko replace karna">

```rust id="9a4k2m"
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-23/src/main.rs:here}}
```

</Listing>

Ye code `{"Blue": 25}` print karega. Original value `10` overwrite ho chuki hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="only-inserting-a-value-if-the-key-has-no-value"></a>

#### Key Mojood Na Hone Ki Surat Mein Hi Key Aur Value Add Karna

Aksar kisi specific key ke liye hash map mein pehle se value mojood hai ya nahi, ye check karna common hota hai aur phir us ke mutabiq ye actions liye jate hain: Agar key hash map mein mojood ho, to existing value ko jaisa hai waisa hi rehna chahiye; agar key mojood na ho, to us key ko aur uski value ko insert karna chahiye.

Hash maps mein is ke liye ek special API hoti hai jise `entry` kaha jata hai, jo us key ko parameter ke taur par leti hai jise aap check karna chahte hain. `entry` method ki return value ek enum hoti hai jise `Entry` kaha jata hai, jo represent karti hai ke koi value mojood ho bhi sakti hai aur nahi bhi. Maan lein ke hum check karna chahte hain ke Yellow team ki key ke saath koi value associated hai ya nahi. Agar nahi hai, to hum value `50` insert karna chahte hain, aur Blue team ke liye bhi aisa hi karna chahte hain. `entry` API ko use karte hue, code Listing 8-24 ki tarah dikhta hai.

<Listing number="8-24" caption="Sirf us waqt insert karne ke liye `entry` method use karna jab key ke saath pehle se koi value mojood na ho">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-24/src/main.rs:here}}
```

</Listing>

`Entry` par `or_insert` method is tarah define ki gayi hai ke agar corresponding `Entry` key mojood ho, to ye us key ki value ka mutable reference return karti hai, aur agar key mojood na ho, to ye parameter ko is key ki nayi value ke taur par insert karti hai aur nayi value ka mutable reference return karti hai. Ye technique khud logic likhne ke muqable mein kaafi clean hai aur, is ke ilawa, borrow checker ke saath bhi zyada achhi tarah kaam karti hai.

Listing 8-24 ka code run karne par `{"Yellow": 50, "Blue": 10}` print hoga. `entry` ki pehli call Yellow team ki key ko value `50` ke saath insert karegi kyun ke Yellow team ke paas pehle se koi value nahi hai. `entry` ki doosri call hash map ko change nahi karegi, kyun ke Blue team ke paas pehle se value `10` hai.


#### Purani Value Ki Bina Par Value Ko Update Karna

Hash maps ka ek aur common use case ye hai ke kisi key ki value ko lookup kiya jaye aur phir purani value ki bina par usay update kiya jaye. Misal ke taur par, Listing 8-25 mein aisa code dikhaya gaya hai jo kisi text mein har word ke appear hone ki tadaad count karta hai. Hum words ko keys ke taur par use karne wala hash map use karte hain aur value ko increment karke track karte hain ke hum ne us word ko kitni baar dekha hai. Agar hum ne kisi word ko pehli baar dekha ho, to hum pehle value `0` insert karenge.

<Listing number="8-25" caption="Words aur counts store karne wale hash map ko use karke words ke occurrences count karna">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-25/src/main.rs:here}}
```

</Listing>

Ye code `{"world": 2, "hello": 1, "wonderful": 1}` print karega. Aapko yehi key-value pairs different order mein print hote hue nazar aa sakte hain: [“Hash Map Mein Values Access Karna”][access]<!-- ignore --> se yaad karein ke hash map par iterate karna arbitrary order mein hota hai.

`split_whitespace` method `text` mein mojood value ke subslices par ek iterator return karti hai, jo whitespace se separate hote hain. `or_insert` method specified key ki value ka mutable reference (`&mut V`) return karti hai. Yahan hum is mutable reference ko `count` variable mein store karte hain, is liye us value ko assign karne ke liye humein pehle asterisk (`*`) use karke `count` ko dereference karna padta hai. Mutable reference `for` loop ke end par scope se bahar nikal jata hai, is liye borrowing rules ki wajah se ye tamam changes safe aur allowed hain.

### Hashing Functions

Default taur par, `HashMap` ek hashing function use karta hai jise *SipHash* kaha jata hai, jo hash tables se related denial-of-service (DoS) attacks ke muqable mein resistance provide kar sakta hai[^siphash]<!-- ignore -->. Ye sab se fast hashing algorithm nahi hai, lekin performance mein is kami ke badle jo behtar security milti hai, woh is trade-off ko worth it banati hai. Agar aap apne code ko profile karein aur pata chale ke default hash function aapke purposes ke liye bohat slow hai, to aap different hasher specify karke kisi aur function par switch kar sakte hain. Ek *hasher* woh type hoti hai jo `BuildHasher` trait implement karti hai. Hum traits aur unhein implement karne ke tareeqe ke baare mein [Chapter 10][traits]<!-- ignore --> mein baat karenge. Zaroori nahi ke aap apna hasher bilkul scratch se implement karein; [crates.io](https://crates.io/)<!-- ignore --> par doosre Rust users ki share ki hui libraries mojood hain jo bohat se common hashing algorithms ko implement karne wale hashers provide karti hain.

[^siphash]: https://en.wikipedia.org/wiki/SipHash

## Summary

Vectors, strings, aur hash maps programs mein us waqt bohat si zaroori functionality provide karti hain jab aapko data store, access, aur modify karna ho. Ab aapko in exercises ko solve karne ke liye tayyar hona chahiye:

1. Integers ki ek list di gayi ho, to vector use karke list ka median (jab sorted ho, to beech wali position ki value) aur mode (jo value sab se zyada baar appear hoti hai; yahan hash map helpful hoga) return karein.
2. Strings ko Pig Latin mein convert karein. Har word ka pehla consonant word ke end par move kiya jata hai aur *ay* add kiya jata hai, is liye *first* se *irst-fay* ban jata hai. Jo words vowel se start hote hain, unke end par *hay* add karein (*apple* se *apple-hay* ban jata hai). UTF-8 encoding ke details ko zehan mein rakhein!
3. Hash map aur vectors ko use karke ek text interface create karein jo user ko company ke kisi department mein employees ke names add karne de; misal ke taur par, “Add Sally to Engineering” ya “Add Amir to Sales.” Phir user ko kisi department ke tamam logon ki list ya company mein department ke hisaab se tamam logon ki list retrieve karne dein, aur list ko alphabetically sort karein.

Standard library ki API documentation mein vectors, strings, aur hash maps ke paas mojood woh methods describe kiye gaye hain jo in exercises ke liye helpful honge!

Ab hum zyada complex programs ki taraf barh rahe hain jahan operations fail ho sakti hain, is liye error handling discuss karne ka ye perfect waqt hai. Hum agley chapter mein ye karenge!

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[access]: #accessing-values-in-a-hash-map
[traits]: ch10-02-traits.html

