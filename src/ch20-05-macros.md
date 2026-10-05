## Macros

Humne is book mein `println!` jaisi macros use ki hain, lekin humne abhi tak
poori tarah explore nahi kiya ke macro kya hoti hai aur kaise kaam karti hai.
*Macro* term Rust mein features ki ek family ko refer karti hai—`macro_rules!`
ke saath declarative macros aur teen kinds ki procedural macros:

* Custom `#[derive]` macros jo `derive` attribute ke saath structs aur enums mein
  add hone wale code ko specify karti hain
* Attribute-like macros jo custom attributes define karti hain jinhein kisi bhi
  item par use kiya ja sakta hai
* Function-like macros jo function calls ki tarah nazar aati hain lekin apne
  argument ke taur par diye gaye tokens par operate karti hain

Hum in mein se har ek par bari bari baat karenge, lekin pehle dekhte hain ke
jab hamare paas pehle se functions hain to humein macros ki zaroorat hi kyun hai.

### The Difference Between Macros and Functions

Bunyadi taur par, macros aisa code likhne ka tareeqa hain jo doosra code likhta
hai, jise *metaprogramming* kaha jata hai. Appendix C mein hum `derive`
attribute par discussion karte hain, jo aapke liye mukhtalif traits ki
implementation generate karta hai. Humne poori book mein `println!` aur `vec!`
macros bhi use ki hain. Yeh tamam macros aapke manually likhe hue code se
zyada code produce karne ke liye *expand* hoti hain.

Metaprogramming us code ki quantity ko reduce karne ke liye useful hai jo aapko
likhna aur maintain karna padta hai, jo functions ke roles mein se ek hai.
Lekin macros ke paas kuch additional powers bhi hoti hain jo functions ke paas
nahi hotin.

Function signature mein function ke parameters ki number aur type declare karna
zaroori hota hai. Doosri taraf, macros variable number of parameters le sakti
hain: Hum `println!("hello")` ko ek argument ke saath ya
`println!("hello {}", name)` ko do arguments ke saath call kar sakte hain.
Is ke ilawa, macros compiler ke code ke meaning ko interpret karne se pehle
expand hoti hain, is liye macro, misal ke taur par, kisi given type par trait
implement kar sakti hai. Function aisa nahi kar sakti, kyun ke usay runtime par
call kiya jata hai aur trait ko compile time par implement karna zaroori hota
hai.

Function ke bajaye macro implement karne ka downside yeh hai ke macro
definitions, function definitions ke muqable mein zyada complex hoti hain kyun
ke aap aisa Rust code likh rahe hote hain jo Rust code likhta hai. Is
indirection ki wajah se, macro definitions aam tor par function definitions
ke muqable mein read, understand aur maintain karna zyada mushkil hoti hain.

Macros aur functions ke darmiyan ek aur important difference yeh hai ke aapko
kisi file mein macros ko call karne se *pehle* unhein define karna ya scope mein
lana hota hai, jab ke functions ko aap kahin bhi define kar sakte hain aur
kahin bhi call kar sakte hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="declarative-macros-with-macro_rules-for-general-metaprogramming"></a>

### Declarative Macros for General Metaprogramming

Rust mein macros ki sab se zyada use hone wali form *declarative macro* hai. Inhein
kabhi kabhi “macros by example,” “`macro_rules!` macros,” ya sirf “macros” bhi
kaha jata hai. Apni bunyaad mein, declarative macros aapko Rust ke `match`
expression jaisi koi cheez likhne ki ijazat deti hain. Jaisa ke Chapter 6 mein
discuss kiya gaya hai, `match` expressions control structures hoti hain jo ek
expression leti hain, expression ki resultant value ko patterns ke saath
compare karti hain, aur phir matching pattern ke saath associated code ko run
karti hain. Macros bhi ek value ko particular code ke saath associated patterns
se compare karti hain: is situation mein, value woh literal Rust source code
hota hai jo macro ko pass kiya gaya ho; patterns us source code ki structure ke
saath compare kiye jate hain; aur har pattern ke saath associated code, jab
match ho jaye, macro ko pass kiye gaye code ko replace kar deta hai. Yeh sab
compilation ke dauran hota hai.

Macro define karne ke liye aap `macro_rules!` construct use karte hain. Aaiye
`macro_rules!` ko use karna samajhte hain aur dekhte hain ke `vec!` macro kaise
defined hai. Chapter 8 mein humne cover kiya tha ke hum particular values ke
saath naya vector create karne ke liye `vec!` macro kaise use kar sakte hain.
Misal ke taur par, following macro teen integers contain karne wala naya vector
create karti hai:

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

Hum `vec!` macro ko do integers ka vector ya paanch string slices ka vector
banane ke liye bhi use kar sakte hain. Hum same kaam karne ke liye function use
nahi kar sakte the kyun ke humein pehle se number ya values ki type ka pata
nahi hota.

Listing 20-35 `vec!` macro ki definition ka ek slightly simplified version
dikhati hai.

<Listing number="20-35" file-name="src/lib.rs" caption="`vec!` macro definition ka ek simplified version">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-35/src/lib.rs}}
```

</Listing>

> Note: Standard library mein `vec!` macro ki actual definition mein pehle se
> sahi amount of memory pre-allocate karne ke liye code shamil hota hai. Woh code
> ek optimization hai jo hum yahan include nahi kar rahe, taake example simple
> rahe.

`#[macro_export]` annotation indicate karti hai ke yeh macro us waqt available
honi chahiye jab woh crate jismein macro defined hai scope mein laya jaye.
Is annotation ke baghair, macro ko scope mein nahi laya ja sakta.

Phir hum macro definition ko `macro_rules!` se start karte hain aur us macro ka
naam likhte hain jo hum define kar rahe hain, *bina* exclamation mark ke. Is
case mein naam `vec` hai, aur is ke baad curly brackets aati hain jo macro
definition ke body ko denote karti hain.

`vec!` body ki structure `match` expression ki structure ke similar hai. Yahan
hamare paas ek arm hai jiska pattern `( $( $x:expr ),* )` hai, is ke baad `=>`
aur is pattern ke saath associated code ka block hai. Agar pattern match ho
jaye, to associated code block emit kiya jayega. Kyun ke is macro mein yahi ek
pattern hai, is liye match karne ka sirf ek valid tareeqa hai; koi bhi doosra
pattern error ka sabab banega. Zyada complex macros mein ek se zyada arms
honge.

Macro definitions mein valid pattern syntax Chapter 19 mein cover ki gayi
pattern syntax se different hoti hai kyun ke macro patterns values ke bajaye
Rust code ki structure ke against match kiye jate hain. Aaiye Listing 20-29 mein
pattern ke pieces ka matlab samajhte hain; complete macro pattern syntax ke
liye [Rust Reference][ref] dekhein.

Sab se pehle, hum poore pattern ko encompass karne ke liye parentheses ka ek
set use karte hain. Hum macro system mein ek variable declare karne ke liye
dollar sign (`$`) use karte hain jo pattern se match hone wala Rust code
contain karega. Dollar sign yeh clear karta hai ke yeh ek macro variable hai,
na ke regular Rust variable. Is ke baad parentheses ka ek set aata hai jo
un values ko capture karta hai jo parentheses ke andar pattern se match hoti
hain,
taake unhein replacement code mein use kiya ja sake. `$()` ke andar
`$x:expr` hai, jo kisi bhi Rust expression se match karta hai aur us expression
ko `$x` naam deta hai.

`$()` ke baad comma indicate karta hai ke literal comma separator character
code ki har instance ke darmiyan hona zaroori hai jo `$()` ke andar diye gaye
code se match karti hai. `*` specify karta hai ke pattern `*` se pehle wali
cheez ki zero ya zyada instances se match karta hai.

Jab hum is macro ko `vec![1, 2, 3];` ke saath call karte hain, to `$x` pattern
teen baar match karta hai, teen expressions `1`, `2`, aur `3` ke saath.

Ab code ke is arm ke saath associated body mein pattern ko dekhte hain:
`$()*` ke andar `temp_vec.push()` pattern mein `$()` se match hone wale har
part ke liye zero ya zyada baar generate hota hai, yeh is baat par depend karta
hai ke pattern kitni baar match hota hai. `$x` ko har matched expression se
replace kiya jata hai. Jab hum is macro ko `vec![1, 2, 3];` ke saath call
karte hain, to generated code jo is macro call ko replace karega, following
hoga:

```rust,ignore
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

Humne ek aisi macro define ki hai jo kisi bhi type ke kisi bhi number of
arguments le sakti hai aur specified elements ko contain karne wala vector
create karne ke liye code generate kar sakti hai.

Macros likhne ke bare mein mazeed seekhne ke liye online documentation ya
doosre resources dekhein, jaise [“The Little Book of Rust Macros”][tlborm],
jise Daniel Keep ne start kiya tha aur Lukas Wirth ne continue kiya.

### Procedural Macros for Generating Code from Attributes

Macros ki doosri form procedural macro hai, jo zyada function ki tarah kaam
karti hai (aur procedure ki ek type hai). *Procedural macros* input ke taur par
kuch code accept karti hain, us code par operate karti hain, aur output ke
taur par kuch code produce karti hain, bajaye is ke ke patterns ke against
match karein aur code ko doosre code se replace karein, jaisa ke declarative
macros karti hain. Procedural macros ki teen kinds custom `derive`,
attribute-like, aur function-like hain, aur yeh sab ek similar tareeqe se kaam
karti hain.

Procedural macros create karte waqt, un ki definitions apne alag crate mein
honi chahiye jismein ek special crate type ho. Yeh complex technical reasons ki
wajah se hai jinhein hum umeed karte hain ke future mein khatam kar denge.
Listing 20-36 mein hum dikhate hain ke procedural macro kaise define ki jati
hai, jahan `some_attribute` ek specific macro variety ke use ke liye
placeholder hai.

<Listing number="20-36" file-name="src/lib.rs" caption="Procedural macro define karne ki ek example">

```rust,ignore
use proc_macro::TokenStream;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

</Listing>

Jo function procedural macro ko define karta hai woh input ke taur par
`TokenStream` leta hai aur output ke taur par `TokenStream` produce karta hai.
`TokenStream` type `proc_macro` crate se define hoti hai jo Rust ke saath
included hai aur tokens ki ek sequence ko represent karti hai. Yeh macro ka
core hai: woh source code jis par macro operate kar rahi hoti hai input
`TokenStream` ko constitute karta hai, aur woh code jo macro produce karti hai
output `TokenStream` hota hai. Function ke saath ek attribute bhi attached hota
hai jo specify karta hai ke hum kis type ki procedural macro create kar rahe
hain. Hum ek hi crate mein procedural macros ki multiple kinds rakh sakte hain.

Aaiye procedural macros ki different kinds ko dekhte hain. Hum custom `derive`
macro se start karenge aur phir un chhoti differences ko explain karenge jo
doosri forms ko different banati hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="how-to-write-a-custom-derive-macro"></a>

### Custom `derive` Macros

Aaiye `hello_macro` naam ka ek crate create karte hain jo `HelloMacro` naam ka
ek trait define karta hai jismein `hello_macro` naam ka ek associated function
hai. Apne users se har type ke liye `HelloMacro` trait implement karwane ke
bajaye, hum ek procedural macro provide karenge taake users apne type ko
`#[derive(HelloMacro)]` se annotate karke `hello_macro` function ki default
implementation hasil kar saken. Default implementation `Hello, Macro! My name is
TypeName!` print karegi, jahan `TypeName` us type ka naam hai jis par yeh trait
define kiya gaya hai. Doosre alfaaz mein, hum aisa crate likhenge jo kisi doosre
programmer ko apne crate ko use karte hue Listing 20-37 jaisa code likhne ki
ijazat dega.

<Listing number="20-37" file-name="src/main.rs" caption="Woh code jo hamare crate ka user hamari procedural macro use karte hue likh sakega">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-37/src/main.rs}}
```

</Listing>

Jab hum kaam mukammal kar lenge, to yeh code `Hello, Macro! My name is
Pancakes!` print karega. Pehla step ek naya library crate banana hai, is tarah:

```console
$ cargo new hello_macro --lib
```

Ab Listing 20-38 mein hum `HelloMacro` trait aur is ke associated function ko
define karenge.

<Listing file-name="src/lib.rs" number="20-38" caption="Ek simple trait jise hum `derive` macro ke saath use karenge">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-38/hello_macro/src/lib.rs}}
```

</Listing>

Hamare paas ek trait aur us ka function hai. Is point par, hamare crate ka user
desired functionality hasil karne ke liye trait ko khud implement kar sakta hai,
jaisa ke Listing 20-39 mein hai.

<Listing number="20-39" file-name="src/main.rs" caption="Agar users `HelloMacro` trait ki manual implementation likhein to yeh kaisa nazar aayega">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-39/pancakes/src/main.rs}}
```

</Listing>

Lekin unhein har us type ke liye implementation block likna padega jise woh
`hello_macro` ke saath use karna chahte hain; hum unhein yeh kaam khud karne
se bachana chahte hain.

Is ke ilawa, hum abhi `hello_macro` function ko aisi default implementation
provide nahi kar sakte jo us type ka naam print kare jis par trait implement
kiya gaya hai: Rust mein reflection capabilities nahi hain, is liye woh runtime
par type ka naam lookup nahi kar sakta. Humein compile time par code generate
karne ke liye ek macro ki zaroorat hai.

Agla step procedural macro define karna hai. Is writing ke waqt, procedural
macros ka apne alag crate mein hona zaroori hai. Eventually, yeh restriction
lift ki ja sakti hai. Crates aur macro crates ko structure karne ka convention
yeh hai: `foo` naam ke crate ke liye, custom `derive` procedural macro crate ko
`foo_derive` kaha jata hai. Aaiye apne `hello_macro` project ke andar
`hello_macro_derive` naam ka ek naya crate start karte hain:

```console
$ cargo new hello_macro_derive --lib
```

Hamare dono crates closely related hain, is liye hum procedural macro crate ko
apne `hello_macro` crate ki directory ke andar create karte hain. Agar hum
`hello_macro` mein trait definition change karte hain, to humein
`hello_macro_derive` mein procedural macro ki implementation bhi change karni
padegi. Dono crates ko separately publish karna hoga, aur in crates ko use
karne wale programmers ko dono ko dependencies ke taur par add karke dono ko
scope mein lana hoga. Is ke bajaye hum `hello_macro` crate ko
`hello_macro_derive` ko dependency ke taur par use karwa sakte hain aur
procedural macro code ko re-export kar sakte hain. Lekin jis tareeqe se humne
project ko structure kiya hai, us se programmers `hello_macro` ko tab bhi use
kar sakte hain jab woh `derive` functionality nahi chahte.

Humein `hello_macro_derive` crate ko procedural macro crate ke taur par
declare karna hoga. Humein `syn` aur `quote` crates ki functionality bhi
chahiye hogi, jaisa ke aap thori der mein dekhenge, is liye humein inhein
dependencies ke taur par add karna hoga. `hello_macro_derive` ki *Cargo.toml*
file mein following add karein:

<Listing file-name="hello_macro_derive/Cargo.toml">

```toml
{{#include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/Cargo.toml:6:12}}
```

</Listing>

Procedural macro define karna start karne ke liye, Listing 20-40 ka code
`hello_macro_derive` crate ki *src/lib.rs* file mein place karein. Note karein
ke yeh code tab tak compile nahi hoga jab tak hum `impl_hello_macro` function
ki definition add nahi karte.

<Listing number="20-40" file-name="hello_macro_derive/src/lib.rs" caption="Woh code jis ki zyada tar procedural macro crates ko Rust code process karne ke liye zaroorat hogi">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/src/lib.rs}}
```

</Listing>

Notice karein ke humne code ko `hello_macro_derive` function mein split kiya hai,
jo `TokenStream` ko parse karne ke liye responsible hai, aur
`impl_hello_macro` function mein, jo syntax tree ko transform karne ke liye
responsible hai: Is se procedural macro likhna zyada convenient ho jata hai.
Outer function ka code (`hello_macro_derive` is case mein) lagbhag har
procedural macro crate mein same hoga jo aap dekhen ya create karein. Inner
function ke body mein jo code aap specify karte hain (`impl_hello_macro` is
case mein), woh aapki procedural macro ke purpose ke mutabiq different hoga.

Humne teen naye crates introduce kiye hain: `proc_macro`, [`syn`][syn]<!-- ignore
-->, aur [`quote`][quote]<!-- ignore -->. `proc_macro` crate Rust ke saath
aata hai, is liye humein usay *Cargo.toml* mein dependencies mein add karne ki
zaroorat nahi thi. `proc_macro` crate compiler ki API hai jo humein apne code
se Rust code ko read aur manipulate karne ki ijazat deti hai.

`syn` crate Rust code ko ek string se parse karke ek data structure mein convert
karta hai jis par hum operations perform kar sakte hain. `quote` crate `syn`
data structures ko dobara Rust code mein convert karta hai. Yeh crates kisi bhi
qism ke Rust code ko parse karna kaafi simple bana dete hain jise hum handle
karna chahte hon: Rust code ke liye ek full parser likhna koi simple task nahi
hai.

`hello_macro_derive` function us waqt call hoga jab hamari library ka user kisi
type par `#[derive(HelloMacro)]` specify karega. Yeh is liye possible hai kyun
ke humne yahan `hello_macro_derive` function ko `proc_macro_derive` se
annotate kiya hai aur `HelloMacro` naam specify kiya hai, jo hamare trait name
se match karta hai; yeh convention zyada tar procedural macros follow karti
hain.

`hello_macro_derive` function sab se pehle `input` ko `TokenStream` se ek aise
data structure mein convert karta hai jise hum phir interpret kar sakte hain
aur jis par operations perform kar sakte hain. Yahin `syn` kaam aata hai.
`syn` ka `parse` function ek `TokenStream` leta hai aur ek `DeriveInput` struct
return karta hai jo parsed Rust code ko represent karta hai. Listing 20-41
`DeriveInput` struct ke relevant parts dikhati hai jo `struct Pancakes;`
string ko parse karne se humein milte hain.

<Listing number="20-41" caption="Listing 20-37 mein macro ka attribute rakhne wale code ko parse karne par milne wala `DeriveInput` instance">

```rust,ignore
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

</Listing>

Is struct ke fields dikhate hain ke jo Rust code humne parse kiya hai woh ek
unit struct hai jiska `ident` (*identifier*, yani naam) `Pancakes` hai. Is
struct mein mukhtalif qisam ke tamam Rust code ko describe karne ke liye mazeed
fields bhi hain; mazeed information ke liye [`syn` documentation for
`DeriveInput`][syn-docs] dekhein.

Jald hi hum `impl_hello_macro` function define karenge, jahan hum woh naya Rust
code build karenge jise hum include karna chahte hain. Lekin us se pehle note
karein ke hamari `derive` macro ka output bhi ek `TokenStream` hai. Returned
`TokenStream` us code mein add hota hai jo hamare crate users likhte hain, is
liye jab woh apna crate compile karte hain, to unhein modified `TokenStream`
mein hamari provide ki hui additional functionality mil jati hai.

Aapne shayad notice kiya ho ke hum yahan `unwrap` call kar rahe hain taake agar
`syn::parse` function ki call fail ho jaye to `hello_macro_derive` function
panic kare. Hamari procedural macro ke liye errors par panic karna zaroori hai
kyun ke `proc_macro_derive` functions ko procedural macro API ke mutabiq
`Result` ke bajaye `TokenStream` return karna hota hai. Humne is example ko
`unwrap` use karke simple rakha hai; production code mein, `panic!` ya `expect`
use karke jo ghalat hua hai us ke bare mein zyada specific error messages
provide karne chahiye.

Ab hamare paas annotated Rust code ko `TokenStream` se `DeriveInput` instance
mein convert karne wala code hai, to aaiye woh code generate karte hain jo
annotated type par `HelloMacro` trait ko implement karta hai, jaisa ke Listing
20-42 mein dikhaya gaya hai.

<Listing number="20-42" file-name="hello_macro_derive/src/lib.rs" caption="Parsed Rust code ko use karke `HelloMacro` trait implement karna">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-42/hello_macro/hello_macro_derive/src/lib.rs:here}}
```

</Listing>

Hum `ast.ident` ko use karke annotated type ke name (identifier) ko contain karne
wala `Ident` struct instance hasil karte hain. Listing 20-41 ka struct dikhata
hai ke jab hum Listing 20-37 ke code par `impl_hello_macro` function run karte
hain, to jo `ident` humein milega us mein `ident` field ki value `"Pancakes"`
hogi. Is liye, Listing 20-42 mein `name` variable ek `Ident` struct instance
contain karega jo print kiye jane par string `"Pancakes"` hoga, yani Listing
20-37 mein struct ka naam.

`quote!` macro humein woh Rust code define karne deti hai jo hum return karna
chahte hain. Compiler `quote!` macro ki direct execution ke result se kuch
different expect karta hai, is liye humein isay `TokenStream` mein convert karna
hoga. Hum `into` method call karke yeh karte hain, jo is intermediate
representation ko consume karta hai aur required `TokenStream` type ki value
return karta hai.

`quote!` macro kuch bohat useful templating mechanics bhi provide karti hai:
hum `#name` enter kar sakte hain, aur `quote!` isay `name` variable mein maujood
value se replace kar degi. Aap regular macros ki tarah kuch repetition bhi kar
sakte hain. Thorough introduction ke liye [the `quote` crate’s docs][quote-docs]
dekhein.

Hum chahte hain ke hamari procedural macro us type ke liye `HelloMacro` trait ki
implementation generate kare jis par user ne annotation lagaya hai, jo hum
`#name` use karke hasil kar sakte hain. Trait implementation mein ek function
`hello_macro` hai, jiske body mein woh functionality hai jo hum provide karna
chahte hain: `Hello, Macro! My name is` print karna aur phir annotated type ka
naam print karna.

Yahan use hone wali `stringify!` macro Rust mein built-in hai. Yeh ek Rust
expression leti hai, jaise `1 + 2`, aur compile time par expression ko ek
string literal mein convert kar deti hai, jaise `"1 + 2"`. Yeh `format!` ya
`println!` se different hai, jo expressions ko evaluate karti hain aur phir
result ko ek `String` mein convert karti hain. Is baat ka possibility hai ke
`#name` input aisa expression ho jise literally print karna ho, is liye hum
`stringify!` use karte hain. `stringify!` use karna ek allocation ko bhi save
karta hai kyun ke `#name` ko compile time par string literal mein convert kar
diya jata hai.

Is point par, `cargo build` ko `hello_macro` aur `hello_macro_derive` dono mein
successfully complete ho jana chahiye. Aaiye in crates ko Listing 20-37 ke code
ke saath connect karte hain taake procedural macro ko action mein dekh saken!
Apni *projects* directory mein `cargo new pancakes` use karke ek naya binary
project create karein. Humein `pancakes` crate ki *Cargo.toml* mein
`hello_macro` aur `hello_macro_derive` ko dependencies ke taur par add karna
hoga. Agar aap apne `hello_macro` aur `hello_macro_derive` ke versions ko
[crates.io](https://crates.io/)<!-- ignore --> par publish kar rahe hain, to yeh
regular dependencies hongi; agar nahi, to aap inhein `path` dependencies ke
taur par is tarah specify kar sakte hain:

```toml
{{#include ../listings/ch20-advanced-features/no-listing-21-pancakes/pancakes/Cargo.toml:6:8}}
```

Listing 20-37 ka code *src/main.rs* mein rakhein aur `cargo run` run karein:
Isay `Hello, Macro! My name is Pancakes!` print karna chahiye. Procedural macro
se aane wali `HelloMacro` trait ki implementation ko `pancakes` crate ke
implement kiye baghair include kar liya gaya; `#[derive(HelloMacro)]` ne trait
implementation add kar di.

Ab aaiye explore karte hain ke doosri kinds ki procedural macros custom
`derive` macros se kis tarah different hain.

### Attribute-Like Macros

Attribute-like macros custom `derive` macros se similar hoti hain, lekin `derive`
attribute ke liye code generate karne ke bajaye, yeh aapko naye attributes
create karne ki ijazat deti hain. Yeh zyada flexible bhi hoti hain: `derive`
sirf structs aur enums ke saath kaam karta hai; attributes doosre items par bhi
apply kiye ja sakte hain, jaise functions. Yahan attribute-like macro ko use
karne ki ek example hai. Maan lein ke aapke paas `route` naam ka ek attribute
hai jo web application framework use karte waqt functions ko annotate karta
hai:

```rust,ignore
#[route(GET, "/")]
fn index() {
```

Yeh `#[route]` attribute framework ki taraf se ek procedural macro ke taur par
define kiya jayega. Macro definition function ki signature is tarah nazar
aayegi:

```rust,ignore
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

Yahan hamare paas `TokenStream` type ke do parameters hain. Pehla attribute ke
contents ke liye hai: `GET, "/"` wala hissa. Doosra us item ke body ke liye hai
jis ke saath attribute attached hai: is case mein `fn index() {}` aur function
ki baqi body.

Is ke ilawa, attribute-like macros custom `derive` macros ki tarah hi kaam
karti hain: Aap `proc-macro` crate type ke saath ek crate create karte hain aur
ek aisa function implement karte hain jo aapka required code generate karta
hai!

### Function-Like Macros

Function-like macros aisi macros define karti hain jo function calls ki tarah
nazar aati hain. `macro_rules!` macros ki tarah, yeh functions se zyada
flexible hoti hain; misal ke taur par, yeh unknown number of arguments le sakti
hain. Lekin `macro_rules!` macros ko sirf us match-like syntax ko use karke
define kiya ja sakta hai jis par humne pehle [“Declarative Macros for General
Metaprogramming”][decl]<!-- ignore --> section mein discussion ki thi.
Function-like macros ek `TokenStream` parameter leti hain, aur un ki definition
us `TokenStream` ko Rust code ke zariye manipulate karti hai, bilkul usi tarah
jaise procedural macros ki doosri do types karti hain. Function-like macro ki
ek example `sql!` macro hai jise is tarah call kiya ja sakta hai:

```rust,ignore id="7e1z6q"
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

Yeh macro apne andar maujood SQL statement ko parse karegi aur check karegi ke
woh syntactically correct hai, jo `macro_rules!` macro ke muqable mein kaafi
zyada complex processing hai. `sql!` macro ko is tarah define kiya jayega:

```rust,ignore id="m3j9x2"
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

Yeh definition custom `derive` macro ki signature se similar hai: Humein
parentheses ke andar maujood tokens receive hote hain aur hum woh code return
karte hain jo hum generate karna chahte hain.

## Summary

Whew! Ab aapke toolbox mein Rust ke kuch aise features hain jinhein aap shayad
aksar use nahi karenge, lekin aapko pata hoga ke bohat particular situations
mein yeh available hain. Humne kai complex topics introduce kiye hain taake jab
aap error message ki suggestions mein ya doosre logon ke code mein inhein
dekhein, to aap in concepts aur syntax ko recognize kar saken. Is chapter ko
ek reference ke taur par use karein jo aapko solutions tak guide kare.

Ab hum poori book mein discuss ki gayi har cheez ko practice mein laayenge aur
ek aur project karenge!

[ref]: ../reference/macros-by-example.html
[tlborm]: https://veykril.github.io/tlborm/
[syn]: https://crates.io/crates/syn
[quote]: https://crates.io/crates/quote
[syn-docs]: https://docs.rs/syn/2.0/syn/struct.DeriveInput.html
[quote-docs]: https://docs.rs/quote
[decl]: #declarative-macros-with-macro_rules-for-general-metaprogramming
