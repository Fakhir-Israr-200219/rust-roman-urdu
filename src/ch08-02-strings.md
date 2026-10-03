## Strings Ke Saath UTF-8 Encoded Text Store Karna

Hum ne Chapter 4 mein strings ke baare mein baat ki thi, lekin ab hum inhein mazeed gehrai mein dekhenge. Naye Rustaceans aam tor par strings par teen reasons ke combination ki wajah se atak jate hain: Rust ka possible errors ko saamne lane ka rujhan, strings ka kai programmers ke khayal se zyada complicated data structure hona, aur UTF-8. Ye factors mil kar us waqt mushkil lag sakte hain jab aap doosri programming languages se aa rahe hon.

Hum strings ko collections ke context mein discuss karte hain kyun ke strings bytes ki ek collection ke taur par implement ki jati hain, saath hi kuch methods hotay hain jo un bytes ko text ke taur par interpret kiye jane par useful functionality provide karte hain. Is section mein hum `String` par un operations ke baare mein baat karenge jo har collection type mein hotay hain, jaise creating, updating, aur reading. Hum ye bhi discuss karenge ke `String` doosri collections se kis tarah different hai, khaas taur par ye ke `String` mein indexing karna is wajah se complicated hai ke log aur computers `String` data ko different tareeqon se interpret karte hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="what-is-a-string"></a>

### Strings Define Karna

Sab se pehle hum ye define karenge ke *string* ki term se hamari kya murad hai. Rust mein core language ke andar sirf ek string type hai, jo string slice `str` hai aur aam tor par apni borrowed form, `&str`, mein nazar aati hai. Chapter 4 mein hum ne string slices ke baare mein baat ki thi, jo kahin aur stored kuch UTF-8 encoded string data ke references hote hain. Misal ke taur par, string literals program ke binary mein store hote hain aur is liye ye string slices hote hain.

`String` type, jo core language mein coded hone ke bajaye Rust ki standard library provide karti hai, ek growable, mutable, owned, UTF-8 encoded string type hai. Jab Rustaceans Rust mein “strings” ka zikr karte hain, to mumkin hai ke woh `String` ya string slice `&str` types mein se kisi ek ki taraf ishara kar rahe hon, na ke sirf in mein se ek type ki taraf. Agarche ye section zyada tar `String` ke baare mein hai, dono types Rust ki standard library mein bohat zyada use hote hain, aur dono `String` aur string slices UTF-8 encoded hote hain.

### Naya String Create Karna

`Vec<T>` ke saath available bohat se wahi operations `String` ke saath bhi available hain, kyun ke `String` asal mein bytes ke ek vector ke around ek wrapper ke taur par implement ki gayi hai, jismein kuch extra guarantees, restrictions, aur capabilities shamil hain. Ek aisi function ki example jo `Vec<T>` aur `String` ke saath ek hi tarah kaam karti hai, `new` function hai jo ek instance create karti hai, jaisa ke Listing 8-11 mein dikhaya gaya hai.

<Listing number="8-11" caption="Ek naya, empty `String` create karna">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-11/src/main.rs:here}}
```

</Listing>

Ye line `s` naam ka ek naya, empty string create karti hai, jismein hum baad mein data load kar sakte hain. Aksar hamare paas kuch initial data hota hai jiske saath hum string shuru karna chahte hain. Is ke liye hum `to_string` method use karte hain, jo har us type par available hota hai jo `Display` trait implement karti hai, jaisa ke string literals karte hain. Listing 8-12 do examples dikhati hai.

<Listing number="8-12" caption="String literal se `String` create karne ke liye `to_string` method use karna">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-12/src/main.rs:here}}
```

</Listing>

Ye code `initial contents` ko contain karne wali string create karta hai.

Hum string literal se `String` create karne ke liye `String::from` function bhi use kar sakte hain. Listing 8-13 ka code Listing 8-12 mein `to_string` use karne wale code ke equivalent hai.

<Listing number="8-13" caption="String literal se `String` create karne ke liye `String::from` function use karna">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-13/src/main.rs:here}}
```

</Listing>

Kyun ke strings bohat si cheezon ke liye use hoti hain, hum strings ke liye bohat si different generic APIs use kar sakte hain, jis se humein bohat se options milte hain. In mein se kuch redundant lag sakti hain, lekin har ek ka apna place hai! Is case mein, `String::from` aur `to_string` same kaam karte hain, is liye aap in mein se kisay choose karte hain ye style aur readability ka matter hai.

Yaad rakhein ke strings UTF-8 encoded hoti hain, is liye hum un mein koi bhi properly encoded data include kar sakte hain, jaisa ke Listing 8-14 mein dikhaya gaya hai.

<Listing number="8-14" caption="Strings mein different languages mein greetings store karna">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:here}}
```

</Listing>

Ye tamam valid `String` values hain.

### String Ko Update Karna

Ek `String` size mein grow kar sakti hai aur uske contents change ho sakte hain, bilkul `Vec<T>` ke contents ki tarah, agar aap us mein mazeed data push karein. Is ke ilawa, aap `String` values ko concatenate karne ke liye `+` operator ya `format!` macro bhi aasani se use kar sakte hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="appending-to-a-string-with-push_str-and-push"></a>

#### `push_str` ya `push` Ke Saath Append Karna

Hum `push_str` method ko use karke ek string slice append karte hue `String` ko grow kar sakte hain, jaisa ke Listing 8-15 mein dikhaya gaya hai.

<Listing number="8-15" caption="`push_str` method ko use karke ek string slice ko `String` mein append karna">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-15/src/main.rs:here}}
```

</Listing>

In do lines ke baad, `s` mein `foobar` hoga. `push_str` method ek string slice leti hai kyun ke hum zaroori nahi samajhte ke parameter ki ownership le li jaye. Misal ke taur par, Listing 8-16 ke code mein, hum chahte hain ke `s2` ke contents ko `s1` mein append karne ke baad bhi hum `s2` ko use kar saken.

<Listing number="8-16" caption="Ek `String` mein contents append karne ke baad string slice ko use karna">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-16/src/main.rs:here}}
```

</Listing>

Agar `push_str` method `s2` ki ownership le leti, to hum last line par iski value print nahi kar sakte the. Lekin ye code bilkul hamari expectation ke mutabiq kaam karta hai!

`push` method parameter ke taur par ek single character leti hai aur usay `String` mein add karti hai. Listing 8-17 `push` method ko use karke ek `String` mein letter *l* add karti hai.

<Listing number="8-17" caption="`push` ko use karke `String` value mein ek character add karna">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-17/src/main.rs:here}}
```

</Listing>

Natije ke taur par, `s` mein `lol` hoga.

<!-- Old headings. Do not remove or links may break. -->

<a id="concatenation-with-the--operator-or-the-format-macro"></a>

#### `+` ya `format!` Ke Saath Strings Concatenate Karna

Aksar aap do existing strings ko combine karna chahenge. Is ka ek tareeqa `+` operator use karna hai, jaisa ke Listing 8-18 mein dikhaya gaya hai.

<Listing number="8-18" caption="Do `String` values ko ek nayi `String` value mein combine karne ke liye `+` operator use karna">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-18/src/main.rs:here}}
```

</Listing>

String `s3` mein `Hello, world!` hoga. Addition ke baad `s1` valid kyun nahi rehta, aur hum ne `s2` ke liye reference kyun use kiya, is ka taalluq us method ki signature se hai jo `+` operator use karne par call hoti hai. `+` operator `add` method ko use karta hai, jis ki signature kuch is tarah dikhti hai:

```rust,ignore
fn add(self, s: &str) -> String {
```

Standard library mein, aap `add` ko generics aur associated types ko use karke defined dekhenge. Yahan hum ne concrete types substitute kiye hain, jo us waqt hota hai jab hum is method ko `String` values ke saath call karte hain. Hum Chapter 10 mein generics ke baare mein discuss karenge. Ye signature humein `+` operator ke tricky parts ko samajhne ke liye zaroori clues deti hai.

Sab se pehle, `s2` ke saath `&` hai, jis ka matlab hai ke hum second string ka reference first string mein add kar rahe hain. Ye `add` function ke `s` parameter ki wajah se hai: Hum sirf ek string slice ko `String` mein add kar sakte hain; hum do `String` values ko directly ek doosre mein add nahi kar sakte. Lekin rukiye—`&s2` ki type `&String` hai, `&str` nahi, jaisa ke `add` ke second parameter mein specify kiya gaya hai. To phir Listing 8-18 compile kyun hoti hai?

Hum `add` ko call karte waqt `&s2` use kar sakte hain kyun ke compiler `&String` argument ko `&str` mein coerce kar sakta hai. Jab hum `add` method call karte hain, Rust deref coercion use karta hai, jo yahan `&s2` ko `&s2[..]` mein convert kar deta hai. Hum Chapter 15 mein deref coercion ko mazeed detail mein discuss karenge. Kyun ke `add` `s` parameter ki ownership nahi leti, is operation ke baad bhi `s2` ek valid `String` rahega.

Doosra, hum signature mein dekh sakte hain ke `add` `self` ki ownership leti hai kyun ke `self` ke saath `&` nahi hai. Is ka matlab hai ke Listing 8-18 mein `s1`, `add` call mein move ho jayega aur us ke baad valid nahi rahega. Is liye, agarche `let s3 = s1 + &s2;` dekhne mein aisa lagta hai ke ye dono strings ko copy karke ek nayi string create karega, asal mein ye statement `s1` ki ownership le leta hai, `s2` ke contents ki ek copy append karta hai, aur phir result ki ownership return karta hai. Doosre alfaaz mein, ye dekhne mein aisa lagta hai ke bohat si copies ban rahi hain, lekin aisa nahi hai; implementation copying se zyada efficient hai.

Agar humein multiple strings ko concatenate karna ho, to `+` operator ka behavior unwieldy ho jata hai:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-01-concat-multiple-strings/src/main.rs:here}}
```

Is point par, `s` `tic-tac-toe` hoga. Tamam `+` aur `"` characters ki wajah se ye samajhna mushkil ho jata hai ke kya ho raha hai. Strings ko zyada complicated tareeqe se combine karne ke liye, hum is ke bajaye `format!` macro use kar sakte hain:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-02-format/src/main.rs:here}}
```

Ye code bhi `s` ko `tic-tac-toe` set karta hai. `format!` macro `println!` ki tarah kaam karta hai, lekin output ko screen par print karne ke bajaye, ye contents wali ek `String` return karta hai. `format!` use karne wala code parhne mein kaafi aasaan hai, aur `format!` macro se generate hone wala code references use karta hai, is liye ye call apne kisi bhi parameter ki ownership nahi leti.

### Strings Mein Indexing Karna

Bohat si doosri programming languages mein string ke individual characters ko index ke zariye access karna ek valid aur common operation hai. Lekin agar aap Rust mein indexing syntax ko use karke `String` ke kisi part ko access karne ki koshish karein, to aapko error milega. Listing 8-19 mein diye gaye invalid code ko dekhein.

<Listing number="8-19" caption="`String` ke saath indexing syntax use karne ki koshish karna">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-19/src/main.rs:here}}
```

</Listing>

Ye code neeche diya gaya error produce karega:

```console
{{#include ../listings/ch08-common-collections/listing-08-19/output.txt}}
```

Error khud sab kuch bata deta hai: Rust strings indexing ko support nahi karti. Lekin kyun nahi? Is sawal ka jawab dene ke liye, humein discuss karna hoga ke Rust strings ko memory mein kis tarah store karta hai.

#### Internal Representation

Ek `String`, `Vec<u8>` ke around ek wrapper hoti hai. Aaiye Listing 8-14 se hamari kuch properly encoded UTF-8 example strings ko dekhein. Sab se pehle, ye wali:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:spanish}}
```

Is case mein, `len` `4` hoga, jis ka matlab hai ke string `"Hola"` ko store karne wala vector 4 bytes lamba hai. UTF-8 mein encode hone par in mein se har letter 1 byte leta hai. Lekin neeche wali line aapko surprise kar sakti hai (note karein ke ye string capital Cyrillic letter *Ze* se shuru hoti hai, number 3 se nahi):

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:russian}}
```

Agar aapse poocha jaye ke string kitni lambi hai, to aap shayad 12 kahenge. Asal mein, Rust ka jawab 24 hai: Ye “Здравствуйте” ko UTF-8 mein encode karne ke liye required bytes ki tadaad hai, kyun ke is string mein har Unicode scalar value 2 bytes ki storage leti hai. Is liye, string ke bytes mein ek index hamesha kisi valid Unicode scalar value ke saath correspond nahi karega. Is baat ko samajhne ke liye, is invalid Rust code ko dekhein:

```rust,ignore,does_not_compile
let hello = "Здравствуйте";
let answer = &hello[0];
```

Aap pehle se jaante hain ke `answer`, first letter `З` nahi hoga. Jab `З` ko UTF-8 mein encode kiya jata hai, to iska first byte `208` aur second `151` hota hai, is liye dekhne mein aisa lag sakta hai ke `answer` asal mein `208` hona chahiye, lekin `208` apne aap mein ek valid character nahi hai. Agar koi user is string ka first letter maange, to `208` return karna shayad woh nahi hoga jo user chahta hai; lekin byte index 0 par Rust ke paas yahi ek data hai. Users aam tor par byte value return nahi chahte, chahe string mein sirf Latin letters hi kyun na hon: Agar `&"hi"[0]` valid code hota jo byte value return karta, to ye `h` ke bajaye `104` return karta.

Is liye jawab ye hai ke unexpected value return karne aur aise bugs paida karne se bachne ke liye jo foran discover na bhi hon, Rust is code ko bilkul compile nahi karta aur development process ke shuru mein hi misunderstandings ko rok deta hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="bytes-and-scalar-values-and-grapheme-clusters-oh-my"></a>

#### Bytes, Scalar Values, aur Grapheme Clusters

UTF-8 ke baare mein ek aur point ye hai ke Rust ke perspective se strings ko dekhne ke asal mein teen relevant tareeqe hain: bytes, scalar values, aur grapheme clusters (jo us cheez ke sab se qareeb hain jise hum *letters* kehte hain).

Agar hum Devanagari script mein likhe hue Hindi word “नमस्ते” ko dekhein, to ye `u8` values ke ek vector ki surat mein is tarah store hota hai:

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

Ye 18 bytes hain aur ye woh tareeqa hai jis se computers aakhirkar is data ko store karte hain. Agar hum inhein Unicode scalar values ke taur par dekhein, jo Rust ki `char` type hoti hain, to ye bytes is tarah nazar aati hain:

```text
['न', 'म', 'स', '्', 'त', 'े']
```

Yahan chhe `char` values hain, lekin fourth aur sixth letters nahi hain: Ye diacritics hain jo apne aap mein meaningful nahi hote. Aakhir mein, agar hum inhein grapheme clusters ke taur par dekhein, to humein woh chaar letters milenge jinhein koi person Hindi word ko banane wale letters kahega:

```text
["न", "म", "स्", "ते"]
```

Rust raw string data ko interpret karne ke different tareeqe provide karta hai jo computers store karte hain, taa-ke har program apni zaroorat ke mutabiq interpretation choose kar sake, chahe data kisi bhi human language mein ho.

Rust humein `String` mein index karke character hasil karne ki ijazat na dene ki ek aakhri wajah ye hai ke indexing operations se hamesha constant time (O(1)) lene ki expectation hoti hai. Lekin `String` ke saath is performance ki guarantee dena mumkin nahi hai, kyun ke Rust ko ye determine karne ke liye ke kitne valid characters maujood hain, beginning se index tak contents ke through walk karna padega.

### Strings Ko Slice Karna

String mein indexing karna aksar ek bura idea hota hai kyun ke ye clear nahi hota ke string-indexing operation ka return type kya hona chahiye: ek byte value, ek character, ek grapheme cluster, ya ek string slice. Is liye, agar aapko waqai string slices create karne ke liye indices use karne ki zaroorat ho, to Rust aapse zyada specific hone ko kehta hai.

`[]` ko ek single number ke saath indexing karne ke bajaye, aap `[]` ko ek range ke saath use karke particular bytes par mushtamil string slice create kar sakte hain:

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

Yahan, `s` ek `&str` hoga jismein string ke pehle 4 bytes honge. Pehle hum ne mention kiya tha ke in mein se har character 2 bytes ka tha, jis ka matlab hai ke `s` `Зд` hoga.

Agar hum kisi character ke bytes ke sirf ek hisse ko slice karne ki koshish karein, jaise `&hello[0..1]`, to Rust runtime par panic karega, bilkul usi tarah jaise vector mein kisi invalid index ko access kiya jaye:

```console
{{#include ../listings/ch08-common-collections/output-only-01-not-char-boundary/output.txt}}
```

Ranges ke saath string slices create karte waqt aapko ehtiyat karni chahiye, kyun ke aisa karne se aapka program crash ho sakta hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="methods-for-iterating-over-strings"></a>

### Strings Par Iterate Karna

Strings ke pieces par kaam karne ka behtareen tareeqa ye hai ke aap clear taur par decide karein ke aap characters chahte hain ya bytes. Individual Unicode scalar values ke liye `chars` method use karein. “Зд” par `chars` call karne se do `char` type ki values separate hokar return hoti hain, aur aap har element ko access karne ke liye result par iterate kar sakte hain:

```rust
for c in "Зд".chars() {
    println!("{c}");
}
```

Ye code ye output print karega:

```text
З
д
```

Is ke bajaye, `bytes` method har raw byte return karta hai, jo aapke domain ke liye munasib ho sakta hai:

```rust
for b in "Зд".bytes() {
    println!("{b}");
}
```

Ye code is string ko banane wale 4 bytes print karega:

```text
208
151
208
180
```

Lekin ye zaroor yaad rakhein ke valid Unicode scalar values 1 se zyada bytes par mushtamil ho sakti hain.

Strings se grapheme clusters hasil karna, jaisa ke Devanagari script ke saath hota hai, complex hai, is liye ye functionality standard library provide nahi karti. Agar aapko ye functionality chahiye, to [crates.io](https://crates.io/)<!-- ignore --> par crates available hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="strings-are-not-so-simple"></a>

### Strings Ki Complexities Ko Handle Karna

Khulasa ye hai ke strings complicated hoti hain. Different programming languages programmer ke saamne is complexity ko present karne ke liye different choices karti hain. Rust ne tamam Rust programs ke liye `String` data ko correctly handle karne ko default behavior banane ka faisla kiya hai, jis ka matlab hai ke programmers ko shuru se hi UTF-8 data ko handle karne ke baare mein zyada sochna padta hai. Ye trade-off strings ki complexity ko doosri programming languages ke muqable mein zyada zahir karta hai, lekin ye aapko apne development life cycle ke baad ke stages mein non-ASCII characters se related errors handle karne se bachata hai.

Achhi baat ye hai ke standard library `String` aur `&str` types par based bohat si functionality provide karti hai jo in complex situations ko correctly handle karne mein madad karti hai. Useful methods ki documentation zaroor dekhein, jaise string mein search karne ke liye `contains` aur string ke kisi hisse ko doosri string se replace karne ke liye `replace`.

Ab thori kam complex cheez ki taraf chalte hain: hash maps!
