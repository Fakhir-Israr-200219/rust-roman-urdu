## Module Tree Mein Kisi Item Ko Refer Karne Ke Liye Paths

Rust ko module tree mein kisi item ko dhoondhne ki jagah batane ke liye hum path use karte hain, bilkul usi tarah jaise filesystem navigate karte waqt path use karte hain. Kisi function ko call karne ke liye humein uska path maloom hona zaroori hai.

Path do forms mein se kisi ek form mein ho sakta hai:

* An *absolute path* crate root se shuru hone wala full path hota hai; external crate ke code ke liye absolute path crate name se shuru hota hai, aur current crate ke code ke liye ye literal `crate` se shuru hota hai.
* A *relative path* current module se shuru hota hai aur `self`, `super`, ya current module mein mojood kisi identifier ko use karta hai.

Absolute aur relative dono paths ke baad ek ya zyada identifiers aate hain jo double colons (`::`) se separate hote hain.

Listing 7-1 ki taraf wapas aate hue, maan lein hum `add_to_waitlist` function ko call karna chahte hain. Ye asal mein ye poochne jaisa hai: `add_to_waitlist` function ka path kya hai? Listing 7-3 mein Listing 7-1 mojood hai, lekin kuch modules aur functions remove kar diye gaye hain.

Hum crate root mein define kiye gaye ek naye function, `eat_at_restaurant`, se `add_to_waitlist` function ko call karne ke do tareeqe dikhayenge. Ye paths correct hain, lekin ek aur problem abhi baqi hai jo is example ko as-is compile hone se rokegi. Hum thori dair mein explain karenge ke kyun.

`eat_at_restaurant` function hamari library crate ki public API ka hissa hai, is liye hum isay `pub` keyword ke saath mark karte hain. [`pub` Keyword Ke Saath Paths Expose Karna][pub]<!-- ignore --> section mein hum `pub` ke baare mein mazeed detail mein baat karenge.

<Listing number="7-3" file-name="src/lib.rs" caption="Absolute aur relative paths use karke `add_to_waitlist` function ko call karna">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-03/src/lib.rs}}
```

</Listing>

Pehli baar jab hum `eat_at_restaurant` mein `add_to_waitlist` function ko call karte hain, to hum ek absolute path use karte hain. `add_to_waitlist` function usi crate mein defined hai jisme `eat_at_restaurant` hai, jis ka matlab hai ke hum absolute path shuru karne ke liye `crate` keyword use kar sakte hain. Phir hum har successive module ko include karte hue `add_to_waitlist` tak pohanchte hain. Aap ek aise filesystem ka tasawwur kar sakte hain jiski structure bilkul isi tarah ho: `add_to_waitlist` program ko run karne ke liye hum path `/front_of_house/hosting/add_to_waitlist` specify karte; crate root se shuru karne ke liye `crate` name use karna usi tarah hai jaise apne shell mein filesystem root se shuru karne ke liye `/` use karna.

Doosri baar jab hum `eat_at_restaurant` mein `add_to_waitlist` ko call karte hain, to hum relative path use karte hain. Path `front_of_house` se shuru hota hai, jo module tree mein `eat_at_restaurant` ke same level par defined module ka name hai. Yahan filesystem equivalent path `front_of_house/hosting/add_to_waitlist` use karna hoga. Module name se shuru karne ka matlab hai ke path relative hai.

Relative ya absolute path mein se kisay use karna hai, ye aap apne project ki bunyaad par decide karenge, aur ye is baat par depend karta hai ke aapke liye item ki definition wala code aur us item ko use karne wala code alag alag move hone ka imkaan zyada hai ya dono saath move hone ka. Misal ke taur par, agar hum `front_of_house` module aur `eat_at_restaurant` function ko `customer_experience` naam ke module mein move kar dein, to humein `add_to_waitlist` ka absolute path update karna padega, lekin relative path phir bhi valid rahega. Lekin agar hum `eat_at_restaurant` function ko alag se `dining` naam ke module mein move kar dein, to `add_to_waitlist` call ka absolute path same rahega, lekin relative path ko update karna padega. Aam tor par hamari preference absolute paths specify karna hai, kyun ke zyada imkaan hai ke hum code definitions aur item calls ko ek doosre se independently move karna chahenge.

Aaiye Listing 7-3 ko compile karke dekhte hain ke ye abhi compile kyun nahi hoti! Humein jo errors milte hain woh Listing 7-4 mein dikhaye gaye hain.

<Listing number="7-4" caption="Listing 7-3 ke code ko build karne se milne wale compiler errors">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-03/output.txt}}
```

</Listing>

Error messages kehte hain ke `hosting` module private hai. Doosre alfaaz mein, `hosting` module aur `add_to_waitlist` function ke liye hamare paths correct hain, lekin Rust humein inhein use karne nahi dega kyun ke in private sections tak hamari access nahi hai. Rust mein tamam items (functions, methods, structs, enums, modules, aur constants) by default parent modules ke liye private hote hain. Agar aap kisi item jaise function ya struct ko private banana chahte hain, to aap usay ek module mein rakhte hain.

Parent module mein mojood items child modules ke andar private items ko use nahi kar sakte, lekin child modules mein mojood items apne ancestor modules ke items ko use kar sakte hain. Iski wajah ye hai ke child modules apni implementation details ko wrap aur hide karte hain, lekin child modules us context ko dekh sakte hain jisme woh define kiye gaye hain. Apni metaphor ko continue karte hue, privacy rules ko restaurant ke back office ki tarah samjhein: Wahan jo kuch hota hai woh restaurant ke customers ke liye private hota hai, lekin office managers us restaurant mein sab kuch dekh aur kar sakte hain jise woh operate karte hain.

Rust ne module system ko is tarah function karne ke liye is liye design kiya taa-ke inner implementation details ko hide karna default ho. Is tarah aapko maloom hota hai ke inner code ke kaun se parts aap outer code ko break kiye baghair change kar sakte hain. Lekin Rust aapko `pub` keyword use karke child modules ke code ke inner parts ko outer ancestor modules ke liye expose karne ka option bhi deta hai, taa-ke kisi item ko public banaya ja sake.

### `pub` Keyword Ke Saath Paths Expose Karna

Aaiye Listing 7-4 mein diye gaye error ki taraf wapas aate hain jis ne humein bataya tha ke `hosting` module private hai. Hum chahte hain ke parent module mein mojood `eat_at_restaurant` function ko child module mein mojood `add_to_waitlist` function tak access hasil ho, is liye hum `hosting` module ko `pub` keyword ke saath mark karte hain, jaisa ke Listing 7-5 mein dikhaya gaya hai.

<Listing number="7-5" file-name="src/lib.rs" caption="`hosting` module ko `pub` declare karna taa-ke ise `eat_at_restaurant` se use kiya ja sake">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-05/src/lib.rs:here}}
```

</Listing>

Badqismati se, Listing 7-5 ka code ab bhi compiler errors produce karta hai, jaisa ke Listing 7-6 mein dikhaya gaya hai.

<Listing number="7-6" caption="Listing 7-5 ke code ko build karne se milne wale compiler errors">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-05/output.txt}}
```

</Listing>

Kya hua? `mod hosting` ke aage `pub` keyword add karne se module public ho jata hai. Is change ke saath, agar hum `front_of_house` tak access kar sakte hain, to hum `hosting` tak bhi access kar sakte hain. Lekin `hosting` ke *contents* ab bhi private hain; module ko public banane se uske contents public nahi ho jate. Module par `pub` keyword sirf itna karta hai ke uske ancestor modules mein mojood code us module ko refer kar sakta hai, uske inner code ko access nahi kar sakta. Kyun ke modules containers hote hain, is liye sirf module ko public banane se hum bohat kuch nahi kar sakte; humein ek qadam aur aage ja kar module ke andar mojood ek ya zyada items ko bhi public banane ka intekhab karna hoga.

Listing 7-6 mein errors kehte hain ke `add_to_waitlist` function private hai. Privacy rules modules ke saath saath structs, enums, functions, aur methods par bhi apply hote hain.

Aaiye `add_to_waitlist` function ki definition se pehle `pub` keyword add karke ise bhi public banate hain, jaisa ke Listing 7-7 mein hai.

<Listing number="7-7" file-name="src/lib.rs" caption="`mod hosting` aur `fn add_to_waitlist` mein `pub` keyword add karne se hum `eat_at_restaurant` se function ko call kar sakte hain.">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-07/src/lib.rs:here}}
```

</Listing>

Ab code compile ho jayega! Ye samajhne ke liye ke `pub` keyword add karne se privacy rules ke hawale se `eat_at_restaurant` mein in paths ko use karne ki ijazat kyun milti hai, aaiye absolute aur relative paths ko dekhte hain.

Absolute path mein hum `crate` se shuru karte hain, jo hamare crate ke module tree ka root hai. `front_of_house` module crate root mein defined hai. Agarche `front_of_house` public nahi hai, kyun ke `eat_at_restaurant` function usi module mein defined hai jisme `front_of_house` hai (yani `eat_at_restaurant` aur `front_of_house` siblings hain), is liye hum `eat_at_restaurant` se `front_of_house` ko refer kar sakte hain. Is ke baad `hosting` module hai jo `pub` se marked hai. Hum `hosting` ke parent module ko access kar sakte hain, is liye hum `hosting` ko bhi access kar sakte hain. Aakhir mein, `add_to_waitlist` function `pub` se marked hai, aur hum uske parent module ko access kar sakte hain, is liye ye function call kaam karti hai!

Relative path mein logic absolute path jaisa hi hai, siwaye pehle step ke: Crate root se shuru hone ke bajaye, path `front_of_house` se shuru hota hai. `front_of_house` module usi module ke andar defined hai jisme `eat_at_restaurant` hai, is liye us module se shuru hone wala relative path jahan `eat_at_restaurant` defined hai, kaam karta hai. Phir, kyun ke `hosting` aur `add_to_waitlist` dono `pub` se marked hain, path ka baqi hissa kaam karta hai, aur ye function call valid hai!

Agar aap apni library crate ko share karne ka plan rakhte hain taa-ke doosre projects aapka code use kar saken, to aapki public API aapke crate ke users ke saath aapka contract hoti hai jo determine karta hai ke woh aapke code ke saath kis tarah interact kar sakte hain. Aapki public API mein changes ko manage karne ke hawale se bohat si considerations hain taa-ke logon ke liye aapke crate par depend karna aasaan ho. Ye considerations is book ke scope se bahar hain; agar aap is topic mein interested hain, to [Rust API Guidelines][api-guidelines] dekhein.

> #### Binary aur Library Wale Packages Ke Liye Best Practices
>
> Hum ne mention kiya tha ke ek package mein *src/main.rs* binary crate root ke saath saath *src/lib.rs* library crate root bhi ho sakta hai, aur dono crates ke paas by default package ka name hoga. Aam tor par, jin packages mein library aur binary crate dono hote hain, unke binary crate mein sirf itna code hota hai jo ek executable ko start kare aur library crate mein define kiye gaye code ko call kare. Is se doosre projects package ki provide ki hui zyada se zyada functionality se faida utha sakte hain, kyun ke library crate ka code share kiya ja sakta hai.
>
> Module tree ko *src/lib.rs* mein define kiya jana chahiye. Phir, kisi bhi public item ko binary crate mein package ke name se paths shuru karke use kiya ja sakta hai. Binary crate library crate ka user ban jata hai, bilkul usi tarah jaise koi completely external crate library crate ko use karega: Ye sirf public API ko use kar sakta hai. Ye aapko ek achhi API design karne mein madad karta hai; aap sirf author hi nahi, balki ek client bhi hain!
>
> [Chapter 12][ch12]<!-- ignore --> mein, hum is organizational practice ko ek command line program ke saath demonstrate karenge jo binary crate aur library crate dono par mushtamil hoga.

### Relative Paths Ko `super` Se Shuru Karna

Hum relative paths bana sakte hain jo current module ya crate root ke bajaye parent module se shuru hote hain. Is ke liye path ke start mein `super` use karte hain. Ye filesystem path ko `..` syntax ke saath shuru karne jaisa hai, jiska matlab parent directory mein jana hota hai. `super` use karne se hum us item ko refer kar sakte hain jiske baare mein humein pata hai ke woh parent module mein hai. Ye module tree ko rearrange karna aasaan bana sakta hai jab module ka parent ke saath close relation ho, lekin mumkin ho ke future mein parent ko module tree mein kisi aur jagah move kar diya jaye.

Listing 7-8 mein diye gaye code ko dekhein, jo us situation ko model karta hai jahan ek chef ghalat order ko theek karta hai aur khud usay customer ke paas le kar jata hai. `back_of_house` module mein defined `fix_incorrect_order` function, `super` se shuru hone wala `deliver_order` ka path specify karke parent module mein defined `deliver_order` function ko call karta hai.

<Listing number="7-8" file-name="src/lib.rs" caption="`super` se shuru hone wale relative path ka use karke function ko call karna">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-08/src/lib.rs}}
```

</Listing>

`fix_incorrect_order` function `back_of_house` module mein hai, is liye hum `super` ko use karke `back_of_house` ke parent module mein ja sakte hain, jo is case mein `crate`, yani root hai. Wahan se hum `deliver_order` ko dhoondte hain aur use mil jata hai. Kamyabi! Hamara khayal hai ke `back_of_house` module aur `deliver_order` function ke darmiyan ye relation barqarar rehne ka imkaan hai aur agar hum crate ke module tree ko reorganize karne ka faisla karein to ye dono saath move honge. Is liye hum ne `super` use kiya taa-ke agar future mein ye code kisi doosre module mein move kiya jaye to humein code ko update karne ke liye kam jagahon par changes karne padhein.

### Structs aur Enums Ko Public Banana

Hum structs aur enums ko public designate karne ke liye bhi `pub` use kar sakte hain, lekin structs aur enums ke saath `pub` ke usage mein kuch extra details hain. Agar hum struct definition se pehle `pub` use karein, to hum struct ko public bana dete hain, lekin struct ke fields phir bhi private rahenge. Hum har field ko case-by-case basis par public ya private rakh sakte hain. Listing 7-9 mein hum ne ek public `back_of_house::Breakfast` struct define kiya hai jisme `toast` field public hai, jabke `seasonal_fruit` field private hai. Ye restaurant ki us situation ko model karta hai jahan customer meal ke saath milne wali bread ka type choose kar sakta hai, lekin chef decide karta hai ke meal ke saath kaunsa fruit diya jaye, jo is baat par depend karta hai ke season mein aur stock mein kya available hai. Available fruit jaldi jaldi change hota rehta hai, is liye customers fruit choose nahi kar sakte aur na hi ye dekh sakte hain ke unhein kaunsa fruit milega.

<Listing number="7-9" file-name="src/lib.rs" caption="Ek struct jisme kuch public fields aur kuch private fields hain">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-09/src/lib.rs}}
```

</Listing>

Kyun ke `back_of_house::Breakfast` struct mein `toast` field public hai, `eat_at_restaurant` mein hum dot notation use karke `toast` field mein value likh aur usay read kar sakte hain. Notice karein ke hum `eat_at_restaurant` mein `seasonal_fruit` field ko use nahi kar sakte, kyun ke `seasonal_fruit` private hai. `seasonal_fruit` field ki value ko modify karne wali line ko uncomment karke dekhein ke aapko kaunsa error milta hai!

Ye bhi note karein ke kyun ke `back_of_house::Breakfast` mein ek private field hai, is liye struct ko ek public associated function provide karni hoti hai jo `Breakfast` ka instance construct kare (yahan hum ne isay `summer` naam diya hai). Agar `Breakfast` mein aisi function na hoti, to hum `eat_at_restaurant` mein `Breakfast` ka instance create nahi kar sakte the, kyun ke hum `eat_at_restaurant` mein private `seasonal_fruit` field ki value set nahi kar sakte.

Is ke baraks, agar hum kisi enum ko public banate hain, to uski tamam variants bhi public ho jati hain. Humein sirf `enum` keyword se pehle `pub` likhne ki zaroorat hoti hai, jaisa ke Listing 7-10 mein dikhaya gaya hai.

<Listing number="7-10" file-name="src/lib.rs" caption="Enum ko public designate karne se uski tamam variants public ho jati hain.">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-10/src/lib.rs}}
```

</Listing>

Kyun ke hum ne `Appetizer` enum ko public banaya hai, is liye hum `eat_at_restaurant` mein `Soup` aur `Salad` variants ko use kar sakte hain.

Enums us waqt tak zyada useful nahi hote jab tak unki variants public na hon; har case mein tamam enum variants ko `pub` se annotate karna annoying hota, is liye enum variants ka default public hona hai. Structs aksar apne fields ko public kiye baghair bhi useful hote hain, is liye struct fields general rule follow karte hain ke har cheez by default private hoti hai jab tak usay `pub` ke saath annotate na kiya jaye.

`pub` se related ek aur situation hai jise hum ne abhi tak cover nahi kiya, aur woh hamare module system ka aakhri feature hai: `use` keyword. Pehle hum `use` ko khud cover karenge, aur phir dikhayenge ke `pub` aur `use` ko kis tarah combine kiya jata hai.

[pub]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[api-guidelines]: https://rust-lang.github.io/api-guidelines/
[ch12]: ch12-00-an-io-project.html
