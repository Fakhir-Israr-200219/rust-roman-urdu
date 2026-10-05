## Shared-State Concurrency

Message passing concurrency handle karne ka ek acha tareeqa hai, lekin yeh akela tareeqa
nahi hai. Ek aur method yeh ho sakta hai ke multiple threads ek hi shared data ko access
karein. Go language documentation ke slogan ke is hissa ko dobara dekhein: “Memory share
karke communicate na karein.”

Memory share karke communicate karna kaisa hoga? Is ke ilawa, message-passing ke
enthusiasts memory sharing ko use na karne ki caution kyun denge?

Ek tarah se, kisi bhi programming language mein channels single ownership ke similar hote
hain kyun ke jab aap kisi value ko channel ke zariye transfer kar dete hain, to aapko us
value ko dobara use nahi karna chahiye. Shared-memory concurrency multiple ownership ki
tarah hai: Multiple threads ek hi waqt mein same memory location ko access kar sakte hain.
Jaisa ke aapne Chapter 15 mein dekha, jahan smart pointers ne multiple ownership ko
possible banaya, multiple ownership complexity add kar sakti hai kyun ke in different
owners ko manage karna zaroori hota hai. Rust ka type system aur ownership rules is
management ko correctly karne mein bohot madad karte hain. Ek example ke liye, aaiye
mutexes ko dekhte hain, jo shared memory ke liye zyada common concurrency primitives
mein se ek hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-mutexes-to-allow-access-to-data-from-one-thread-at-a-time"></a>

### Controlling Access with Mutexes

*Mutex* *mutual exclusion* ka abbreviation hai, yani mutex kisi bhi given time par sirf
ek thread ko kisi data tak access karne deta hai. Mutex mein data ko access karne ke liye,
thread ko pehle mutex ka lock acquire karne ki request karke signal karna hota hai ke woh
access chahta hai. *Lock* ek data structure hai jo mutex ka hissa hota hai aur yeh track
karta hai ke is waqt data tak exclusive access kis ke paas hai. Is liye, mutex ko locking
system ke zariye apne andar maujood data ko *guarding* karne wala describe kiya jata hai.

Mutexes ka difficult to use hone ka reputation hai kyun ke aapko do rules yaad rakhne hote hain:

1. Data ko use karne se pehle aapko lock acquire karne ki koshish karni zaroori hai.
2. Jab aap mutex ke guard kiye hue data ke saath done ho jayein, to aapko data ko unlock karna
   zaroori hai taake doosre threads lock acquire kar sakein.

Mutex ki ek real-world metaphor ke liye, ek conference mein panel discussion imagine karein
jahan sirf ek microphone ho. Kisi panelist ke bolne se pehle, usay microphone use karne ke
liye request ya signal karna hota hai. Jab usay microphone mil jata hai, to woh jitni der
chahe baat kar sakta hai aur phir microphone us next panelist ko de deta hai jo bolne ki
request karta hai. Agar koi panelist microphone use karne ke baad use hand off karna bhool
jaye, to koi aur bol nahi sakta. Agar shared microphone ki management ghalat ho jaye, to
panel planned tareeqe se kaam nahi karega!

Mutexes ki management ko correctly karna incredibly tricky ho sakta hai, isi liye bohot se
log channels ke baare mein enthusiastic hote hain. Lekin Rust ke type system aur ownership
rules ki wajah se, aap locking aur unlocking ko ghalat nahi kar sakte.

#### The API of `Mutex<T>`

Mutex ko use karne ka example dekhne ke liye, aaiye simplicity ke liye pehle ek
single-threaded context mein mutex use karte hain, jaisa ke Listing 16-12 mein dikhaya
gaya hai.

<Listing number="16-12" file-name="src/main.rs" caption="Exploring the API of `Mutex<T>` in a single-threaded context for simplicity">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-12/src/main.rs}}
```

</Listing>

Bohot si types ki tarah, hum associated function `new` ko use karke ek `Mutex<T>` create
karte hain. Mutex ke andar data ko access karne ke liye, hum lock acquire karne ke liye
`lock` method use karte hain. Yeh call current thread ko block karegi taake jab tak lock
lene ki hamari turn na aaye, woh koi work perform na kar sake.

Agar lock hold karne wala koi doosra thread panic kar jaye to `lock` ki call fail ho sakti
hai. Is situation mein koi bhi kabhi lock hasil nahi kar sakega, is liye humne `unwrap`
choose kiya hai aur agar hum is situation mein hon to is thread ko panic karne denge.

Lock acquire karne ke baad, hum return value ko, jo is case mein `num` naam ki hai, andar
maujood data ke mutable reference ki tarah treat kar sakte hain. Type system ensure karta
hai ke `m` mein value ko use karne se pehle hum lock acquire karein. `m` ki type `i32`
nahi balki `Mutex<i32>` hai, is liye `i32` value ko use karne ke qabil hone ke liye humein
`lock` call karna *must* hai. Hum bhool nahi sakte; type system humein inner `i32` ko
otherwise access nahi karne dega.

`lock` ki call `MutexGuard` naam ki ek type return karti hai, jo `LockResult` mein wrapped
hoti hai aur humne ise `unwrap` ki call ke saath handle kiya hai. `MutexGuard` type inner
data ko point karne ke liye `Deref` implement karti hai; is type ke paas `Drop` implementation
bhi hai jo lock ko automatically release karti hai jab `MutexGuard` scope se bahar chala
jata hai, jo inner scope ke end par hota hai. Is ke result mein, humein lock release karna
bhool jane aur mutex ko doosre threads ke use ke liye block karne ka risk nahi hota kyun ke
lock release automatically hota hai.

Lock ko drop karne ke baad, hum mutex ki value print kar sakte hain aur dekh sakte hain ke
hum inner `i32` ko `6` mein change karne mein kamyab rahe.

<!-- Old headings. Do not remove or links may break. -->

<a id="sharing-a-mutext-between-multiple-threads"></a>

#### Shared Access to `Mutex<T>`

Ab aaiye `Mutex<T>` ko use karke ek value ko multiple threads ke darmiyan share karne ki
koshish karte hain. Hum 10 threads start karenge aur har thread se counter ki value mein 1
increment karwayenge, taake counter 0 se 10 tak chala jaye. Listing 16-13 mein diya gaya
example compiler error dega, aur hum is error ko use karke `Mutex<T>` ko use karne ke baare
mein aur seekhenge aur yeh samjhenge ke Rust humein ise correctly use karne mein kaise
madad karta hai.

<Listing number="16-13" file-name="src/main.rs" caption="Ten threads, each incrementing a counter guarded by a `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-13/src/main.rs}}
```

</Listing>

Hum ek `counter` variable create karte hain jo `Mutex<T>` ke andar ek `i32` hold karta hai,
jaisa ke humne Listing 16-12 mein kiya tha. Is ke baad, hum numbers ki ek range par iterate
karke 10 threads create karte hain. Hum `thread::spawn` use karte hain aur tamam threads ko
same closure dete hain: ek aisi closure jo `counter` ko thread mein move karti hai,
`lock` method ko call karke `Mutex<T>` par lock acquire karti hai, aur phir mutex mein
maujood value mein 1 add karti hai. Jab koi thread apni closure run karna finish karta hai,
to `num` scope se bahar chala jayega aur lock release kar dega taake koi doosra thread ise
acquire kar sake.

Main thread mein, hum tamam join handles collect karte hain. Phir, jaisa ke humne Listing
16-2 mein kiya tha, hum har handle par `join` call karte hain taake ensure ho ke tamam
threads finish ho jayein. Is point par, main thread lock acquire karega aur is program ka
result print karega.

Humne hint diya tha ke yeh example compile nahi hoga. Ab aaiye pata lagate hain ke kyun!

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-13/output.txt}}
```

Error message batata hai ke `counter` value loop ki previous iteration mein move ho gayi
thi. Rust humein bata raha hai ke hum `counter` lock ki ownership ko multiple threads mein
move nahi kar sakte. Aaiye Chapter 15 mein discuss kiye gaye multiple-ownership method ko
use karke compiler error fix karte hain.

#### Multiple Ownership with Multiple Threads

Chapter 15 mein, humne smart pointer `Rc<T>` ko use karke ek reference-counted value create
ki aur ek value ko multiple owners diya. Aaiye yahan bhi yahi karte hain aur dekhte hain
ke kya hota hai. Listing 16-14 mein hum `Mutex<T>` ko `Rc<T>` mein wrap karenge aur thread
ko ownership move karne se pehle `Rc<T>` ko clone karenge.

<Listing number="16-14" file-name="src/main.rs" caption="Attempting to use `Rc<T>` to allow multiple threads to own the `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-14/src/main.rs}}
```

</Listing>

Ek baar phir, hum compile karte hain aur humein... different errors milte hain! Compiler
humein bohot kuch sikha raha hai:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-14/output.txt}}
```

Wow, yeh error message bohot zyada wordy hai! Yahan focus karne ke liye important hissa yeh
hai: `` `Rc<Mutex<i32>>` cannot be sent between threads safely ``. Compiler humein yeh bhi
bata raha hai ke kyun: `` the trait `Send` is not implemented for `Rc<Mutex<i32>>` ``.
Hum next section mein `Send` ke baare mein baat karenge: yeh un traits mein se ek hai jo
ensure karta hai ke threads ke saath hum jo types use karte hain woh concurrent situations
mein use ke liye meant hain.

Unfortunately, `Rc<T>` ko threads ke darmiyan share karna safe nahi hai. Jab `Rc<T>` reference
count ko manage karta hai, to har `clone` ki call par count mein add karta hai aur jab har
clone drop hota hai to count mein se subtract karta hai. Lekin yeh koi concurrency primitives
use nahi karta jo ensure karein ke count mein hone wali changes ko kisi doosre thread ke zariye
interrupt na kiya ja sake. Is se wrong counts ho sakte hain—subtle bugs jo aage chal kar
memory leaks ka sabab ban sakte hain ya value ko us waqt drop karwa sakte hain jab hum abhi
us ke saath done na hue hon. Humein aisi type ki zaroorat hai jo bilkul `Rc<T>` ki tarah ho,
lekin reference count mein changes ko thread-safe tareeqe se kare.

#### Atomic Reference Counting with `Arc<T>`

Fortunately, `Arc<T>` ek aisi type hai jo `Rc<T>` ki tarah hai aur concurrent situations mein
use karna safe hai. *A* ka matlab *atomic* hai, yani yeh ek *atomically reference-counted*
type hai. Atomics ek additional kind ka concurrency primitive hain jinhein hum yahan detail
mein cover nahi karenge: zyada details ke liye standard library documentation mein
[`std::sync::atomic`][atomic]<!-- ignore --> dekhein. Is point par, aapko sirf yeh jaanna
zaroori hai ke atomics primitive types ki tarah kaam karte hain lekin threads ke darmiyan
share karna safe hota hai.

Aap phir soch sakte hain ke tamam primitive types atomic kyun nahi hain aur standard library
types ko default taur par `Arc<T>` use karne ke liye implement kyun nahi kiya gaya. Is ki
wajah yeh hai ke thread safety ke saath performance penalty aati hai jo aap sirf us waqt
pay karna chahte hain jab aapko waqai is ki zaroorat ho. Agar aap sirf ek single thread ke
andar values par operations perform kar rahe hain, to aapka code zyada fast run kar sakta
hai agar use atomics ki taraf se provide ki jane wali guarantees enforce na karni parhein.

Aaiye apne example par wapas aate hain: `Arc<T>` aur `Rc<T>` ka API same hai, is liye hum
`use` line, `new` ki call, aur `clone` ki call ko change karke apne program ko fix karte hain.
Listing 16-15 mein code finally compile aur run hoga.

<Listing number="16-15" file-name="src/main.rs" caption="Using an `Arc<T>` to wrap the `Mutex<T>` to be able to share ownership across multiple threads">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-15/src/main.rs}}
```

</Listing>

Yeh code following output print karega:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Result: 10
```

Humne kar liya! Humne 0 se 10 tak count kiya, jo shayad bohot impressive na lage, lekin is
ne humein `Mutex<T>` aur thread safety ke baare mein bohot kuch sikhaya. Aap is program ki
structure ko sirf counter increment karne se zyada complicated operations perform karne ke
liye bhi use kar sakte hain. Is strategy ko use karke, aap ek calculation ko independent
parts mein divide kar sakte hain, un parts ko threads ke darmiyan split kar sakte hain, aur
phir `Mutex<T>` ko use karke har thread ko apne part ke saath final result ko update karne
de sakte hain.

Note karein ke agar aap simple numerical operations kar rahe hain, to `Mutex<T>` types se
simpler types bhi hain jo standard library ke [`std::sync::atomic` module][atomic]<!-- ignore -->
mein provide kiye gaye hain. Yeh types primitive types ko safe, concurrent, atomic access
provide karte hain. Humne is example ke liye primitive type ke saath `Mutex<T>` use karna
choose kiya taake hum `Mutex<T>` ke kaam karne ke tareeqe par concentrate kar sakein.

<!-- Old headings. Do not remove or links may break. -->

<a id="similarities-between-refcelltrct-and-mutextarct"></a>

### Comparing `RefCell<T>`/`Rc<T>` and `Mutex<T>`/`Arc<T>`

Aap ne shayad notice kiya ho ke `counter` immutable hai lekin hum is ke andar ki value ka
mutable reference hasil kar sakte hain; iska matlab hai ke `Mutex<T>` interior mutability
provide karta hai, bilkul `Cell` family ki tarah. Jis tarah humne Chapter 15 mein
`RefCell<T>` ko use karke `Rc<T>` ke andar maujood contents ko mutate karne ki ijazat hasil
ki thi, usi tarah hum `Mutex<T>` ko use karke `Arc<T>` ke andar maujood contents ko mutate
karte hain.

Ek aur detail note karne wali yeh hai ke jab aap `Mutex<T>` use karte hain to Rust aapko har
tarah ke logic errors se protect nahi kar sakta. Chapter 15 se yaad karein ke `Rc<T>` ko
use karne ke saath reference cycles create hone ka risk tha, jahan do `Rc<T>` values ek
doosre ko refer karti hain, jis ki wajah se memory leaks hotay hain. Isi tarah, `Mutex<T>`
ke saath *deadlocks* create hone ka risk hota hai. Yeh tab occur hotay hain jab kisi
operation ko do resources ko lock karne ki zaroorat ho aur do threads mein se har ek ne
ek lock acquire kar liya ho, jis ki wajah se dono hamesha ke liye ek doosre ka wait karte
rehte hain. Agar aap deadlocks mein interested hain, to aisa Rust program create karne ki
koshish karein jis mein deadlock ho; phir kisi bhi language mein mutexes ke liye deadlock
mitigation strategies ko research karein aur unhein Rust mein implement karne ki koshish
karein. `Mutex<T>` aur `MutexGuard` ke liye standard library API documentation useful
information provide karti hai.

Hum is chapter ko `Send` aur `Sync` traits ke baare mein baat karke aur yeh samajh kar
complete karenge ke hum inhein custom types ke saath kaise use kar sakte hain.

[atomic]: ../std/sync/atomic/index.html
