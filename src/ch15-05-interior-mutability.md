## `RefCell<T>` aur Interior Mutability Pattern

*Interior mutability* Rust mein ek design pattern hai jo aapko data ko mutate
karne ki ijazat deta hai, hatta ke jab us data ke immutable references maujood
hon; aam tor par borrowing rules ki wajah se yeh action allowed nahi hota. Data
ko mutate karne ke liye, yeh pattern data structure ke andar `unsafe` code use
karta hai taake Rust ke un usual rules ko bend kiya ja sake jo mutation aur
borrowing ko govern karte hain. Unsafe code compiler ko indicate karta hai ke
hum rules ko manually check kar rahe hain, bajaye iske ke compiler par rely
karke unhein check karwayen; hum Chapter 20 mein unsafe code ko zyada detail
mein discuss karenge.

Hum un types ko use kar sakte hain jo interior mutability pattern use karte
hain sirf tab jab hum yeh ensure kar saken ke borrowing rules runtime par
follow honge, chahe compiler iski guarantee na de sakta ho. Is mein involved
`unsafe` code ko phir ek safe API ke andar wrap kar diya jata hai, aur outer
type ab bhi immutable hota hai.

Aaiye is concept ko `RefCell<T>` type ko dekh kar explore karte hain jo
interior mutability pattern follow karta hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="enforcing-borrowing-rules-at-runtime-with-refcellt"></a>

### Runtime par Borrowing Rules Enforce Karna

`Rc<T>` ke unlike, `RefCell<T>` type us data par single ownership represent
karta hai jo woh hold karta hai. To phir `RefCell<T>` ko `Box<T>` jaise type se
kya different banata hai? Chapter 4 mein seekhe gaye borrowing rules ko yaad
karein:

* Kisi bhi given waqt par aapke paas *ya to* ek mutable reference ho sakta hai
  ya kisi bhi tadaad mein immutable references (lekin dono nahi).
* References hamesha valid hone chahiye.

References aur `Box<T>` ke saath, borrowing rules ke invariants compile time
par enforce kiye jate hain. `RefCell<T>` ke saath, yeh invariants *runtime*
par enforce kiye jate hain. References ke saath, agar aap in rules ko break
karte hain, to aapko compiler error milega. `RefCell<T>` ke saath, agar aap in
rules ko break karte hain, to aapka program panic karega aur exit ho jayega.

Borrowing rules ko compile time par check karne ka faida yeh hai ke errors
development process mein jaldi catch ho jate hain, aur runtime performance par
koi impact nahi hota kyun ke tamam analysis pehle hi complete ho chuka hota
hai. In reasons ki wajah se, majority of cases mein borrowing rules ko compile
time par check karna best choice hai, isi liye yeh Rust ka default hai.

Borrowing rules ko runtime par check karne ka faida yeh hai ke kuch aise
memory-safe scenarios phir allowed ho jate hain jo compile-time checks ki wajah
se disallowed hote. Static analysis, jaise Rust compiler karta hai, inherently
conservative hoti hai. Code ki kuch properties ko code ka analysis karke detect
karna impossible hota hai: Sabse famous example Halting Problem hai, jo is book
ke scope se bahar hai lekin research karne ke liye ek interesting topic hai.

Kyun ke kuch analysis impossible hoti hai, agar Rust compiler ko yaqeen na ho ke
code ownership rules ko comply karta hai, to woh ek correct program ko reject
kar sakta hai; is tarah woh conservative hota hai. Agar Rust kisi incorrect
program ko accept kar le, to users Rust ki di hui guarantees par trust nahi kar
sakenge. Lekin agar Rust kisi correct program ko reject kar de, to programmer
ko inconvenience hogi, lekin kuch catastrophic nahi ho sakta. `RefCell<T>` type
tab useful hota hai jab aapko yaqeen ho ke aapka code borrowing rules follow
karta hai lekin compiler ise samajhne aur guarantee karne mein unable ho.

`Rc<T>` ki tarah, `RefCell<T>` bhi sirf single-threaded scenarios mein use
karne ke liye hai aur agar aap ise multithreaded context mein use karne ki
koshish karenge to compile-time error dega. Hum Chapter 16 mein baat karenge
ke multithreaded program mein `RefCell<T>` ki functionality kaise hasil ki ja
sakti hai.

Yahan `Box<T>`, `Rc<T>`, ya `RefCell<T>` mein se choose karne ki reasons ka
recap hai:

* `Rc<T>` same data ke multiple owners enable karta hai; `Box<T>` aur
  `RefCell<T>` ke single owners hote hain.
* `Box<T>` immutable ya mutable borrows allow karta hai jinhein compile time
  par check kiya jata hai; `Rc<T>` sirf immutable borrows allow karta hai jinhein
  compile time par check kiya jata hai; `RefCell<T>` immutable ya mutable
  borrows allow karta hai jinhein runtime par check kiya jata hai.
* Kyun ke `RefCell<T>` mutable borrows allow karta hai jinhein runtime par
  check kiya jata hai, aap `RefCell<T>` ke andar maujood value ko mutate kar
  sakte hain, hatta ke jab `RefCell<T>` khud immutable ho.

Kisi immutable value ke andar maujood value ko mutate karna interior mutability
pattern hai. Aaiye ek aisi situation dekhte hain jahan interior mutability
useful hai aur examine karte hain ke yeh possible kaise hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="interior-mutability-a-mutable-borrow-to-an-immutable-value"></a>

### Interior Mutability Use Karna

Borrowing rules ka ek consequence yeh hai ke jab aapke paas koi immutable value
ho, to aap uska mutable borrow nahi le sakte. Misal ke taur par, yeh code
compile nahi hoga:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/src/main.rs}}
```

Agar aap is code ko compile karne ki koshish karenge, to aapko following error
milega:

```console
{{#include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/output.txt}}
```

Lekin, kuch situations mein kisi value ke liye yeh useful ho sakta hai ke woh
apne methods ke andar khud ko mutate kare, lekin doosre code ko immutable nazar
aaye. Value ke methods ke bahar ka code us value ko mutate nahi kar sakega.
`RefCell<T>` use karna interior mutability ki ability hasil karne ka ek tareeqa
hai, lekin `RefCell<T>` borrowing rules ko completely bypass nahi karta:
compiler mein borrow checker is interior mutability ko allow karta hai, aur
borrowing rules ko runtime par check kiya jata hai. Agar aap rules violate
karte hain, to compiler error ke bajaye aapko `panic!` milega.

Aaiye ek practical example ko step by step dekhte hain jahan hum `RefCell<T>`
use karke ek immutable value ko mutate kar sakte hain aur samajhte hain ke yeh
useful kyun hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-use-case-for-interior-mutability-mock-objects"></a>

#### Mock Objects ke Saath Testing

Kabhi kabhi testing ke dauran programmer ek type ko doosre type ki jagah use
karta hai, taake particular behavior observe kiya ja sake aur assert kiya ja
sake ke woh correctly implement hua hai. Is placeholder type ko *test double*
kehte hain. Isay filmmaking mein stunt double ke sense mein samjhein, jahan ek
person kisi actor ki jagah aa kar ek particularly tricky scene karta hai. Jab
hum tests run kar rahe hote hain to test doubles doosre types ki jagah kaam
karte hain. *Mock objects* test doubles ke specific types hain jo test ke dauran
hone wali cheezon ko record karte hain taake aap assert kar saken ke correct
actions perform hue.

Rust mein doosri languages ki tarah same sense mein objects nahi hote, aur Rust
ki standard library mein mock object functionality built in bhi nahi hai jaisa
ke kuch doosri languages mein hota hai. Lekin aap definitely ek aisa struct
create kar sakte hain jo mock object ke same purposes serve kare.

Yeh woh scenario hai jise hum test karenge: Hum ek library create karenge jo
ek value ko maximum value ke against track karti hai aur current value maximum
value ke kitne qareeb hai is basis par messages send karti hai. Misal ke taur
par, yeh library user ke API calls ki allowed tadaad ke quota ko track karne ke
liye use ki ja sakti hai.

Hamari library sirf yeh functionality provide karegi ke koi value maximum ke
kitne qareeb hai aur kis waqt kya messages hone chahiye. Hamari library ko use
karne wali applications se expect kiya jayega ke woh messages send karne ka
mechanism provide karein: Application message ko directly user ko dikha sakti
hai, email send kar sakti hai, text message send kar sakti hai, ya kuch aur
kar sakti hai. Library ko is detail ka pata hone ki zaroorat nahi hai. Isay
sirf kisi aisi cheez ki zaroorat hai jo humare provide kiye gaye ek trait ko
implement kare, jise `Messenger` kaha gaya hai. Listing 15-20 library ka code
dikhati hai.

<Listing number="15-20" file-name="src/lib.rs" caption="Ek library jo track karti hai ke koi value maximum value ke kitne qareeb hai aur jab value kuch specific levels par ho to warning deti hai">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-20/src/lib.rs}}
```

</Listing>

Is code ka ek important hissa yeh hai ke `Messenger` trait mein `send` naam
ka ek method hai jo `self` ka immutable reference aur message ka text leta hai.
Yeh trait woh interface hai jise hamare mock object ko implement karna hoga taake
mock ko real object ki tarah use kiya ja sake. Doosra important hissa yeh hai
ke hum `LimitTracker` ke `set_value` method ke behavior ko test karna chahte
hain. Hum `value` parameter mein pass ki jane wali value ko change kar sakte
hain, lekin `set_value` hamare liye kuch return nahi karta jis par hum
assertions bana saken. Hum yeh kehna chahte hain ke agar hum ek `LimitTracker`
create karein jo `Messenger` trait ko implement karne wali kisi cheez aur `max`
ki ek particular value ko use karta ho, to jab hum `value` ke liye different
numbers pass karein to messenger ko appropriate messages send karne ke liye
kaha jaye.

Humein ek mock object chahiye jo `send` call karne par email ya text message
send karne ke bajaye sirf un messages ko track kare jo use send karne ke liye
kaha gaya hai. Hum mock object ka ek naya instance create kar sakte hain, mock
object ko use karne wala `LimitTracker` create kar sakte hain,
`LimitTracker` par `set_value` method call kar sakte hain, aur phir check kar
sakte hain ke mock object ke paas woh messages hain jo hum expect karte hain.
Listing 15-21 aisa mock object implement karne ki ek koshish dikhati hai, lekin
borrow checker iski ijazat nahi dega.

<Listing number="15-21" file-name="src/lib.rs" caption="Aisa `MockMessenger` implement karne ki koshish jise borrow checker allow nahi karta">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-21/src/lib.rs:here}}
```

</Listing>

Yeh test code ek `MockMessenger` struct define karta hai jisme `sent_messages`
field hai jo `String` values ke `Vec` ko hold karta hai taake un messages ko
track kiya ja sake jinhein use send karne ke liye kaha gaya hai. Hum `new` naam
ka ek associated function bhi define karte hain taake naye `MockMessenger`
values create karna convenient ho jo messages ki empty list se start hon. Phir
hum `MockMessenger` ke liye `Messenger` trait implement karte hain taake hum
`MockMessenger` ko `LimitTracker` ko de saken. `send` method ki definition mein
hum message ko parameter ke taur par lete hain aur use `MockMessenger` ki
`sent_messages` list mein store kar dete hain.

Test mein hum yeh test kar rahe hain ke jab `LimitTracker` ko `value` ko aisi
value par set karne ke liye kaha jaye jo `max` value ke 75 percent se zyada ho
to kya hota hai. Sabse pehle hum ek naya `MockMessenger` create karte hain jo
messages ki empty list se start hoga. Phir hum ek naya `LimitTracker` create
karte hain aur use naye `MockMessenger` ka reference aur `100` ki `max` value
dete hain. Hum `LimitTracker` par `set_value` method ko `80` ki value ke saath
call karte hain, jo 100 ke 75 percent se zyada hai. Phir hum assert karte hain
ke `MockMessenger` jis messages ki list ko track kar raha hai usmein ab ek
message hona chahiye.

Lekin, is test mein ek problem hai, jaisa ke yahan dikhaya gaya hai:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-21/output.txt}}
```

Hum `MockMessenger` ko messages track karne ke liye modify nahi kar sakte, kyun
ke `send` method `self` ka immutable reference leta hai. Hum error text mein
di gayi suggestion ko bhi use nahi kar sakte ke `impl` method aur trait
definition dono mein `&mut self` use karein. Hum sirf testing ki wajah se
`Messenger` trait ko change nahi karna chahte. Iske bajaye, humein koi aisa
tareeqa dhoondna hoga jo hamare existing design ke saath hamare test code ko
correctly work karne de.

Yeh woh situation hai jahan interior mutability help kar sakti hai! Hum
`sent_messages` ko `RefCell<T>` ke andar store karenge, aur phir `send` method
`sent_messages` ko modify karke un messages ko store kar sakega jo humne dekhe
hain. Listing 15-22 dikhati hai ke yeh kaisa nazar aata hai.

<Listing number="15-22" file-name="src/lib.rs" caption="`RefCell<T>` ko use karke inner value ko mutate karna jabke outer value ko immutable maana jata hai">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-22/src/lib.rs:here}}
```

</Listing>

`sent_messages` field ab `Vec<String>` ke bajaye `RefCell<Vec<String>>` type
ka hai. `new` function mein hum empty vector ke around ek naya
`RefCell<Vec<String>>` instance create karte hain.

`send` method ki implementation ke liye, pehla parameter ab bhi `self` ka
immutable borrow hai, jo trait definition se match karta hai. Hum
`self.sent_messages` mein `RefCell<Vec<String>>` par `borrow_mut` call karte
hain taake `RefCell<Vec<String>>` ke andar maujood value, yani vector, ka
mutable reference mil sake. Phir hum vector ke mutable reference par `push`
call kar sakte hain taake test ke dauran send kiye gaye messages ko track kiya
ja sake.

Aakhri change jo humein karna hai woh assertion mein hai: Inner vector mein
kitne items hain yeh dekhne ke liye hum `RefCell<Vec<String>>` par `borrow`
call karte hain taake vector ka immutable reference mil sake.

Ab jab aapne dekha hai ke `RefCell<T>` ko kaise use karna hai, aaiye detail
mein dekhte hain ke yeh kaise kaam karta hai!

<!-- Old headings. Do not remove or links may break. -->

<a id="keeping-track-of-borrows-at-runtime-with-refcellt"></a>

#### Runtime par Borrows ko Track Karna

Jab hum immutable aur mutable references create karte hain, to respectively `&` aur `&mut`
syntax use karte hain. `RefCell<T>` ke saath hum `borrow` aur `borrow_mut`
methods use karte hain, jo `RefCell<T>` ki safe API ka hissa hain. `borrow`
method smart pointer type `Ref<T>` return karta hai, aur `borrow_mut`
smart pointer type `RefMut<T>` return karta hai. Dono types `Deref` ko implement karte hain, is liye hum
inhein regular references ki tarah treat kar sakte hain.

`RefCell<T>` track karta hai ke is waqt kitne `Ref<T>` aur `RefMut<T>` smart
pointers active hain. Har baar jab hum `borrow` call karte hain, `RefCell<T>`
active immutable borrows ki count ko 1 se increase karta hai. Jab koi `Ref<T>`
value scope se bahar chali jati hai, to immutable borrows ki count 1 se kam ho jati hai. Bilkul
compile-time borrowing rules ki tarah, `RefCell<T>` humein kisi bhi waqt kai immutable
borrows ya ek mutable borrow rakhne deta hai.

Agar hum in rules ko violate karne ki koshish karein, to references ke saath hone wale compiler error ke bajaye,
`RefCell<T>` ki implementation runtime par panic karegi. Listing 15-23 mein
Listing 15-22 mein `send` ki implementation ki ek modification dikhayi gayi hai. Hum jaan-boojh kar ek hi
scope mein do mutable borrows active create karne ki koshish kar rahe hain, taake illustrate kiya ja sake ke
`RefCell<T>` humein runtime par aisa karne se rokta hai.

<Listing number="15-23" file-name="src/lib.rs" caption="Ek hi scope mein do mutable references create karna taake dekha ja sake ke `RefCell<T>` panic karega">

```rust,ignore,panics
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-23/src/lib.rs:here}}
```

</Listing>

Hum `borrow_mut` se return hone wale `RefMut<T>` smart pointer ke liye
`one_borrow` naam ka variable create karte hain. Phir, hum isi tarah
`two_borrow` variable mein ek aur mutable borrow create karte hain. Is se ek hi scope mein do mutable references ho jate hain,
jo allowed nahi hai. Jab hum apni library ke tests run karte hain, Listing
15-23 ka code bina kisi error ke compile ho jayega, lekin test fail ho jayega:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-23/output.txt}}
```

Notice karein ke code `already borrowed:
BorrowMutError` message ke saath panic hua. Isi tarah `RefCell<T>` runtime par borrowing
rules ki violations ko handle karta hai.

Borrowing errors ko compile time ke bajaye runtime par catch karne ka choice, jaisa
humne yahan kiya hai, iska matlab hai ke aap apne code mein mistakes development
process mein baad mein discover kar sakte hain: mumkin hai ke aapka code production mein
deploy hone tak bhi nahi. Iske ilawa, aapke code ko ek chhota sa runtime performance penalty
bhi dena padega, kyun ke borrows ko compile time ke bajaye runtime par track kiya ja raha hai.
Lekin `RefCell<T>` use karne se ek mock object likhna possible ho jata hai jo
immutable values ki ijazat wale context mein use karte waqt apne andar
changes kar sakta hai aur un messages ko track kar sakta hai jo usne dekhe hain. Aap
`RefCell<T>` ko iske trade-offs ke bawajood use kar sakte hain taake regular references
ke muqable mein zyada functionality hasil ki ja sake.

<!-- Old headings. Do not remove or links may break. -->

<a id="having-multiple-owners-of-mutable-data-by-combining-rc-t-and-ref-cell-t"></a> <a id="allowing-multiple-owners-of-mutable-data-with-rct-and-refcellt"></a>

### Mutable Data ke Multiple Owners Allow Karna

`RefCell<T>` ko use karne ka ek common tareeqa ise `Rc<T>` ke combination mein
use karna hai. Yaad rakhein ke `Rc<T>` aapko kisi data ke multiple owners rakhne deta hai,
lekin woh us data tak sirf immutable access deta hai. Agar aapke paas ek `Rc<T>` ho
jo `RefCell<T>` ko hold karta ho, to aapko aisi value mil sakti hai jiske multiple owners
bhi hon *aur* jise aap mutate bhi kar sakte hon!

Misal ke taur par, Listing 15-18 mein cons list ki example yaad karein jahan humne
`Rc<T>` use karke multiple lists ko kisi doosri list ki ownership share karne di thi.
Kyun ke `Rc<T>` sirf immutable values hold karta hai, is liye ek baar values create
karne ke baad hum list ki kisi bhi value ko change nahi kar sakte. Aaiye `RefCell<T>` ki
values change karne ki ability ko bhi add karte hain. Listing 15-24 dikhati hai ke
`Cons` definition mein `RefCell<T>` use karke hum tamam lists mein stored
value ko modify kar sakte hain.

<Listing number="15-24" file-name="src/main.rs" caption="`Rc<RefCell<i32>>` ko use karke ek aisi `List` create karna jise hum mutate kar sakte hain">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-24/src/main.rs}}
```

</Listing>

Hum ek aisi value create karte hain jo `Rc<RefCell<i32>>` ka instance hai aur use
`value` naam ke variable mein store karte hain taake baad mein hum directly us tak access
kar saken. Phir, hum `a` mein ek `List` create karte hain jismein ek `Cons` variant
`value` ko hold karta hai. Humein `value` ko clone karna padta hai taake `a` aur `value`
dono inner `5` value ki ownership rakh saken, bajaye iske ke ownership `value` se `a` ko
transfer ho jaye ya `a`, `value` se borrow kare.

Hum list `a` ko ek `Rc<T>` mein wrap karte hain taake jab hum lists `b` aur `c` create karein,
to dono `a` ko refer kar saken, bilkul usi tarah jaise humne Listing 15-18 mein kiya tha.

Lists `a`, `b`, aur `c` create karne ke baad, hum `value` mein mojood value mein 10 add karna
chahte hain. Hum yeh `value` par `borrow_mut` call karke karte hain, jo Chapter 5 mein
[“Where’s the `->`
Operator?”][wheres-the---operator]<!-- ignore --> mein discuss ki gayi automatic dereferencing feature ko use karke
`Rc<T>` ko dereference kar ke andar mojood `RefCell<T>` value tak pohanchta hai. `borrow_mut`
method ek `RefMut<T>` smart pointer return karta hai, aur hum us par dereference operator
use karke inner value ko change kar dete hain.

Jab hum `a`, `b`, aur `c` ko print karte hain, to hum dekh sakte hain ke un sab ke paas
`5` ke bajaye modified value `15` hai:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-24/output.txt}}
```

Yeh technique kaafi neat hai! `RefCell<T>` ko use karke hamare paas outwardly
immutable `List` value hoti hai. Lekin hum `RefCell<T>` ke un methods ko use kar sakte hain jo
uski interior mutability tak access provide karte hain, taake zaroorat padne par hum apne
data ko modify kar saken. Borrowing rules ki runtime checks humein data races se protect
karti hain, aur kabhi kabhi hamari data structures mein is flexibility ke liye thodi si
speed sacrifice karna worth it hota hai. Note karein ke `RefCell<T>` multithreaded code ke liye kaam nahi karta!
`Mutex<T>` `RefCell<T>` ka thread-safe version hai, aur hum Chapter 16 mein
`Mutex<T>` discuss karenge.

[wheres-the---operator]: ch05-03-method-syntax.html#wheres-the---operator
