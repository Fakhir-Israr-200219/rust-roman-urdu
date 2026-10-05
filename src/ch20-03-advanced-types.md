## Advanced Types

Rust ke type system mein kuch features hain jin ka humne ab tak zikr to kiya
hai lekin abhi tak un par discussion nahi ki. Hum sab se pehle newtypes par
generally discussion karenge aur dekhenge ke types ke taur par yeh kyun useful
hain. Phir hum type aliases ki taraf jayenge, jo newtypes ke similar ek feature
hai lekin is ke semantics thore different hain. Hum `!` type aur dynamically
sized types par bhi discussion karenge.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-newtype-pattern-for-type-safety-and-abstraction"></a>

### Type Safety and Abstraction with the Newtype Pattern

Yeh section assume karta hai ke aap ne pehle wala section [“Implementing External
Traits with the Newtype Pattern”][newtype]<!-- ignore --> parh liya hai. Newtype
pattern un tasks ke liye bhi useful hai jo humne abhi tak discuss nahi kiye,
jin mein statically enforce karna ke values kabhi confuse na hon aur kisi value
ki units ko indicate karna shamil hai. Aapne Listing 20-16 mein units indicate
karne ke liye newtypes use karne ki ek example dekhi thi: Yaad karein ke
`Millimeters` aur `Meters` structs ne `u32` values ko newtype mein wrap kiya
tha. Agar hum `Millimeters` type ke parameter ke saath ek function likhein, to
hum aisa program compile nahi karwa sakte jo ghalti se us function ko
`Meters` type ki value ya plain `u32` ke saath call karne ki koshish kare.

Hum newtype pattern ko kisi type ki kuch implementation details ko abstract
away karne ke liye bhi use kar sakte hain: New type ek public API expose kar
sakti hai jo private inner type ki API se different ho.

Newtypes internal implementation ko bhi hide kar sakte hain. Misal ke taur par,
hum ek `People` type provide kar sakte hain jo `HashMap<i32, String>` ko wrap
kare aur ek person ki ID ko us ke name ke saath associated store kare. `People`
use karne wala code sirf us public API ke saath interact karega jo hum provide
karte hain, jaise `People` collection mein name string add karne ka method; us
code ko yeh jaanne ki zaroorat nahi hogi ke hum internally names ko `i32` ID
assign karte hain. Newtype pattern implementation details ko hide karne ke
liye encapsulation achieve karne ka ek lightweight tareeqa hai, jis par humne
Chapter 18 ke [“Encapsulation that
Hides Implementation
Details”][encapsulation-that-hides-implementation-details]<!-- ignore -->
section mein discussion ki thi.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-type-synonyms-with-type-aliases"></a>

### Type Synonyms and Type Aliases

Rust existing type ko doosra naam dene ke liye *type alias* declare karne ki
ability provide karta hai. Is ke liye hum `type` keyword use karte hain. Misal
ke taur par, hum `i32` ke liye `Kilometers` alias is tarah create kar sakte hain:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:here}}
```

Ab `Kilometers`, `i32` ka ek *synonym* hai; Listing 20-16 mein banaye gaye
`Millimeters` aur `Meters` types ke unlike, `Kilometers` koi separate, new
type nahi hai. `Kilometers` type rakhne wali values ko bilkul `i32` type ki
values ki tarah treat kiya jayega:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:there}}
```

Kyun ke `Kilometers` aur `i32` same type hain, hum dono types ki values ko add
kar sakte hain aur `Kilometers` values ko un functions mein pass kar sakte hain
jo `i32` parameters lete hain. Lekin is method ko use karne se humein woh
type-checking benefits nahi milte jo pehle discuss kiye gaye newtype pattern se
milte hain. Doosre lafzon mein, agar hum kahin `Kilometers` aur `i32` values ko
mix up kar dein, to compiler humein error nahi dega.

Type synonyms ka main use case repetition ko reduce karna hai. Misal ke taur
par, hamare paas is tarah ka ek lengthy type ho sakta hai:

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

Function signatures mein aur poore code mein type annotations ke taur par is
lengthy type ko likhna tiresome aur error-prone ho sakta hai. Sochiye ke ek
project mein Listing 20-25 ki tarah bohot sara code ho.

<Listing number="20-25" caption="Using a long type in many places">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-25/src/main.rs:here}}
```

</Listing>

Ek type alias repetition ko reduce karke is code ko zyada manageable bana deta
hai. Listing 20-26 mein humne verbose type ke liye `Thunk` naam ka ek alias
introduce kiya hai aur type ke tamam uses ko chhote alias `Thunk` se replace
kar sakte hain.

<Listing number="20-26" caption="Introducing a type alias, `Thunk`, to reduce repetition">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-26/src/main.rs:here}}
```

</Listing>

Yeh code read aur write karna kaafi easy hai! Type alias ke liye meaningful
naam choose karna aapke intent ko communicate karne mein bhi help kar sakta hai
(*thunk* aise code ke liye ek word hai jise baad mein evaluate kiya jana ho,
is liye yeh us closure ke liye appropriate naam hai jo stored hota hai).

Type aliases ko `Result<T, E>` type ke saath bhi commonly use kiya jata hai taa-ke
repetition reduce ho. Standard library mein `std::io` module ko consider
karein. I/O operations aksar `Result<T, E>` return karti hain taa-ke un
situations ko handle kiya ja sake jab operations fail ho jayein. Is library mein
`std::io::Error` struct hai jo tamam possible I/O errors ko represent karta
hai. `std::io` ke bohot se functions `Result<T, E>` return karte honge jahan
`E` `std::io::Error` hoga, jaise `Write` trait ke yeh functions:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-05-write-trait/src/lib.rs}}
```

`Result<..., Error>` bohot zyada repeat ho raha hai. Isi liye, `std::io` mein
yeh type alias declaration hai:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:here}}
```

Kyun ke yeh declaration `std::io` module mein hai, hum fully qualified alias
`std::io::Result<T>` use kar sakte hain; yani, ek `Result<T, E>` jismein `E`
ko `std::io::Error` ke taur par fill kiya gaya ho. `Write` trait ki function
signatures is tarah nazar aati hain:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:there}}
```

Type alias do tareeqon se help karta hai: Yeh code ko likhna *aur* easy banata
hai aur yeh humein poore `std::io` mein ek consistent interface deta hai. Kyun
ke yeh ek alias hai, yeh sirf ek aur `Result<T, E>` hai, jis ka matlab hai ke
hum is ke saath `Result<T, E>` par kaam karne wale tamam methods use kar sakte
hain, saath hi `?` operator jaisi special syntax bhi.

### The Never Type That Never Returns

Rust mein `!` naam ka ek special type hai jo type theory ki terminology mein
*empty type* ke taur par jana jata hai kyun ke is ki koi values nahi hotin. Hum
isay *never type* kehna prefer karte hain kyun ke jab koi function kabhi return
nahi karega to yeh return type ki jagah hota hai. Yahan ek example hai:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-07-never-type/src/lib.rs:here}}
```

Is code ko is tarah read kiya jata hai: “function `bar` never return karta
hai.” Jo functions never return karte hain unhein *diverging functions* kaha
jata hai. Hum `!` type ki values create nahi kar sakte, is liye `bar` kabhi
bhi possible taur par return nahi kar sakta.

Lekin aise type ka kya faida hai jis ki values aap kabhi create hi nahi kar
sakte? Listing 2-5 ka code yaad karein, jo number-guessing game ka hissa tha;
humne yahan Listing 20-27 mein us ka thora sa hissa dobara diya hai.

<Listing number="20-27" caption="A `match` with an arm that ends in `continue`">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:ch19}}
```

</Listing>

Us waqt humne is code ki kuch details ko skip kar diya tha. Chapter 6 ke
[“The `match`
Control Flow Construct”][the-match-control-flow-construct]<!-- ignore -->
section mein humne discuss kiya tha ke `match` arms sab ko same type return
karna hota hai. Misal ke taur par, following code kaam nahi karta:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-08-match-arms-different-types/src/main.rs:here}}
```

Is code mein `guess` ka type ek integer *aur* ek string hona padega, aur Rust
require karta hai ke `guess` ka sirf ek type ho. To `continue` kya return
karta hai? Listing 20-27 mein humein ek arm se `u32` return karne aur doosre
arm ko `continue` par end karne ki permission kaise mili?

Jaisa ke aapne shayad guess kiya, `continue` ki value `!` hai. Yani, jab Rust
`guess` ka type calculate karta hai, to woh dono match arms ko dekhta hai,
pehle mein `u32` value aur doosre mein `!` value hoti hai. Kyun ke `!` ki
kabhi koi value nahi ho sakti, Rust decide karta hai ke `guess` ka type
`u32` hai.

Is behavior ko describe karne ka formal tareeqa yeh hai ke `!` type ki
expressions ko kisi bhi doosre type mein coerce kiya ja sakta hai. Humein is
`match` arm ko `continue` ke saath end karne ki permission is liye hai kyun ke
`continue` koi value return nahi karta; is ke bajaye, yeh control ko loop ke
top par wapas le jata hai, is liye `Err` case mein hum `guess` ko kabhi koi
value assign nahi karte.

Never type `panic!` macro ke saath bhi useful hai. `Option<T>` values par value
produce karne ya is definition ke saath panic karne ke liye hum jis `unwrap`
function ko call karte hain, usay yaad karein:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-09-unwrap-definition/src/lib.rs:here}}
```

Is code mein bhi wahi cheez hoti hai jo Listing 20-27 ke `match` mein hui thi:
Rust dekhta hai ke `val` ka type `T` hai aur `panic!` ka type `!` hai, is liye
overall `match` expression ka result `T` hai. Yeh code is liye kaam karta hai
kyun ke `panic!` koi value produce nahi karta; yeh program ko end kar deta hai.
`None` case mein, hum `unwrap` se koi value return nahi karenge, is liye yeh
code valid hai.

Ek final expression jis ka type `!` hota hai, woh ek loop hai:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-10-loop-returns-never/src/main.rs:here}}
```

Yahan loop kabhi end nahi hota, is liye `!` expression ki value hai. Lekin
agar hum `break` include karte to yeh true nahi hota, kyun ke loop `break`
tak pohanchne par terminate ho jata.


### Dynamically Sized Types and the `Sized` Trait

Rust ko apni types ke bare mein kuch details ka pata hona zaroori hai, jaise ke
kisi particular type ki value ke liye kitni space allocate karni hai. Is ki wajah
se is ke type system ka ek hissa shuru mein thora confusing lagta hai:
*dynamically sized types* ka concept. Kabhi kabhi inhein *DSTs* ya *unsized
types* bhi kaha jata hai, aur yeh humein aisa code likhne dete hain jo un
values ke saath kaam karta hai jin ka size hum sirf runtime par jaan sakte hain.

Aaiye `str` naam ke ek dynamically sized type ki details mein jate hain, jise
hum poori book mein use karte aa rahe hain. Ji haan, `&str` nahi, balki sirf
`str` apne aap mein ek DST hai. Bohot se cases mein, jaise jab user ki enter ki
hui text ko store karna ho, hum yeh nahi jaan sakte ke string kitni lambi hogi
jab tak runtime na aa jaye. Is ka matlab hai ke hum `str` type ka variable
create nahi kar sakte, na hi hum `str` type ka argument le sakte hain.
Following code ko consider karein, jo kaam nahi karta:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-11-cant-create-str/src/main.rs:here}}
```

Rust ko kisi particular type ki kisi bhi value ke liye yeh pata hona zaroori hai
ke kitni memory allocate karni hai, aur kisi type ki tamam values ko memory ki
same amount use karni hoti hai. Agar Rust humein yeh code likhne deta, to in
dono `str` values ko same amount of space use karni padti. Lekin in ki lengths
different hain: `s1` ko 12 bytes ki storage chahiye aur `s2` ko 15. Isi liye
dynamically sized type ko hold karne wala variable create karna possible nahi
hai.

To hum kya karte hain? Is case mein, aap jawab pehle se jaante hain: Hum
`s1` aur `s2` ki type `str` ke bajaye string slice (`&str`) rakhte hain. Chapter
4 ke [“String Slices”][string-slices]<!-- ignore --> section se yaad karein ke
slice data structure sirf slice ki starting position aur length store karta hai.
Is liye, agarche `&T` ek single value hai jo us memory address ko store karti
hai jahan `T` located hai, string slice *do* values hain: `str` ka address aur
us ki length. Is tarah, hum compile time par string slice value ka size jaan
sakte hain: yeh `usize` ki length ka do guna hota hai. Yani, string slice ka
size humein hamesha pata hota hai, chahe jis string ko woh refer karta hai woh
kitni bhi lambi ho. Generally, Rust mein dynamically sized types ko isi tarah
use kiya jata hai: Un ke paas extra metadata ka ek hissa hota hai jo dynamic
information ka size store karta hai. Dynamically sized types ka golden rule yeh
hai ke humein dynamically sized types ki values ko hamesha kisi na kisi type ke
pointer ke peeche rakhna chahiye.

Hum `str` ko har tarah ke pointers ke saath combine kar sakte hain: misal ke
taur par, `Box<str>` ya `Rc<str>`. Asal mein, aap isay pehle bhi dekh chuke
hain, lekin ek different dynamically sized type ke saath: traits. Har trait ek
dynamically sized type hai jise hum trait ke naam ko use karke refer kar sakte
hain. Chapter 18 ke [“Using Trait Objects to Abstract over
Shared Behavior”][using-trait-objects-to-abstract-over-shared-behavior]<!--
ignore --> section mein humne mention kiya tha ke traits ko trait objects ke
taur par use karne ke liye humein unhein kisi pointer ke peeche rakhna hota hai,
jaise `&dyn Trait` ya `Box<dyn Trait>` (`Rc<dyn Trait>` bhi kaam karega).

DSTs ke saath kaam karne ke liye, Rust `Sized` trait provide karta hai taa-ke
yeh determine kiya ja sake ke kisi type ka size compile time par known hai ya
nahi. Yeh trait har us cheez ke liye automatically implement hota hai jis ka
size compile time par known hota hai. Is ke ilawa, Rust har generic function
mein implicitly `Sized` par ek bound add karta hai. Yani, generic function
definition jaise yeh:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-12-generic-fn-definition/src/lib.rs}}
```

asal mein is tarah treat hoti hai jaise humne yeh likha ho:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-13-generic-implicit-sized-bound/src/lib.rs}}
```

Default taur par, generic functions sirf un types par kaam karenge jin ka size
compile time par known ho. Lekin, aap following special syntax use karke is
restriction ko relax kar sakte hain:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-14-generic-maybe-sized/src/lib.rs}}
```

`?Sized` par trait bound ka matlab hai “`T` `Sized` ho bhi sakta hai aur nahi bhi,”
aur yeh notation is default ko override karta hai ke generic types ka size
compile time par known hona chahiye. Is meaning ke saath `?Trait` syntax sirf
`Sized` ke liye available hai, kisi doosre trait ke liye nahi.

Yeh bhi note karein ke humne `t` parameter ki type `T` se badal kar `&T` kar di
hai. Kyun ke type `Sized` na bhi ho sakti hai, humein isay kisi na kisi type ke
pointer ke peeche use karna hoga. Is case mein, humne ek reference choose kiya
hai.

Agla topic functions aur closures hain!

[encapsulation-that-hides-implementation-details]: ch18-01-what-is-oo.html#encapsulation-that-hides-implementation-details
[string-slices]: ch04-03-slices.html#string-slices
[the-match-control-flow-construct]: ch06-02-match.html#the-match-control-flow-construct
[using-trait-objects-to-abstract-over-shared-behavior]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[newtype]: ch20-02-advanced-traits.html#implementing-external-traits-with-the-newtype-pattern
