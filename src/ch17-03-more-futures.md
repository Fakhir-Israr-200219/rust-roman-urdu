
<!-- Old headings. Do not remove or links may break. -->

<a id="yielding"></a>

### Yielding Control to the Runtime

[“Our First Async Program”][async-program]<!-- ignore -->
section se yaad karein ke har await point par, Rust runtime ko yeh mauqa deta hai ke agar jis
future ko await kiya ja raha hai woh ready nahi hai, to woh task ko pause karke kisi doosre task
par switch kar sake. Is ka ulta bhi true hai: Rust *sirf* await point par async blocks ko pause
karta hai aur control runtime ko wapas deta hai. Await points ke darmiyan sab kuch synchronous
hota hai.

Is ka matlab hai ke agar aap kisi async block mein await point ke baghair bohot sara kaam karte
hain, to woh future kisi doosre future ko progress karne se rok dega. Aap kabhi kabhi isay yeh
kehte hue sun sakte hain ke ek future doosre futures ko *starve* kar raha hai. Kuch cases mein,
yeh koi bara masla nahi hota. Lekin agar aap kisi qisam ka expensive setup ya long-running
work kar rahe hain, ya aapke paas koi aisa future hai jo kisi particular task ko indefinitely
karta rahega, to aapko sochna hoga ke runtime ko control kab aur kahan wapas dena hai.

Starvation problem ko illustrate karne ke liye aaiye ek long-running operation ko simulate
karte hain, phir explore karte hain ke isay solve kaise kiya ja sakta hai. Listing 17-14 ek
`slow` function introduce karti hai.

<Listing number="17-14" caption="Slow operations ko simulate karne ke liye `thread::sleep` use karna" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-14/src/main.rs:slow}}
```

</Listing>

Yeh code `trpl::sleep` ke bajaye `std::thread::sleep` use karta hai taake `slow` ko call karna
current thread ko kuch milliseconds ke liye block kare. Hum `slow` ko un real-world operations
ki jagah use kar sakte hain jo long-running aur blocking dono hoti hain.

Listing 17-15 mein, hum `slow` ko CPU-bound work ki is qisam ko futures ke ek pair mein emulate
karne ke liye use karte hain.

<Listing number="17-15" caption="Slow operations ko simulate karne ke liye `slow` function ko call karna" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-15/src/main.rs:slow-futures}}
```

</Listing>

Har future bohot sari slow operations carry out karne ke *baad* hi control runtime ko wapas
deta hai. Agar aap yeh code run karein, to aapko yeh output nazar aayega:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-15/
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'a' ran for 10ms
'a' ran for 20ms
'b' started.
'b' ran for 75ms
'b' ran for 10ms
'b' ran for 15ms
'b' ran for 350ms
'a' finished.
```

Listing 17-5 ki tarah, jahan humne do URLs fetch karne wale futures ko race karne ke liye
`trpl::select` use kiya tha, `select` usi waqt finish hota hai jab `a` complete ho jata hai.
Lekin dono futures mein `slow` calls ke darmiyan koi interleaving nahi hoti. `a` future apna
tamam kaam karta hai jab tak `trpl::sleep` call ko await nahi kiya jata, phir `b` future apna
tamam kaam karta hai jab tak uski apni `trpl::sleep` call ko await nahi kiya jata, aur aakhir
mein `a` future complete ho jata hai. Dono futures ko unke slow tasks ke darmiyan progress
karne dene ke liye, humein await points chahiye taake hum control runtime ko wapas de saken.
Is ka matlab hai ke humein koi aisi cheez chahiye jise hum await kar saken!

Hum Listing 17-15 mein is qisam ka handoff pehle se hota hua dekh sakte hain: agar hum `a`
future ke end par `trpl::sleep` ko remove kar dein, to woh `b` future ke *bilkul bhi* run kiye
baghair complete ho jayega. Aaiye `trpl::sleep` function ko operations ko progress karne se
switch off karne dene ke liye starting point ke taur par use karne ki koshish karte hain, jaisa
ke Listing 17-16 mein dikhaya gaya hai.

<Listing number="17-16" caption="Operations ko progress karne se switch off karne dene ke liye `trpl::sleep` use karna" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-16/src/main.rs:here}}
```

</Listing>

Humne har `slow` call ke darmiyan await points ke saath `trpl::sleep` calls add ki hain.
Ab dono futures ka work interleaved hai:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-16
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'b' started.
'b' ran for 75ms
'a' ran for 10ms
'b' ran for 10ms
'a' ran for 20ms
'b' ran for 15ms
'a' finished.
```

`a` future ab bhi `b` ko control hand off karne se pehle kuch der run karta hai, kyun ke woh
`trpl::sleep` ko call karne se pehle `slow` ko call karta hai, lekin us ke baad futures har
baar ek doosre ke saath switch karte hain jab un mein se koi await point tak pohanchta hai.
Is case mein, humne yeh har `slow` call ke baad kiya hai, lekin hum work ko apni zaroorat ke
mutabiq kisi bhi tarah break up kar sakte hain.

Lekin hum yahan asal mein *sleep* nahi karna chahte: hum jitni tezi se ho sake progress karna
chahte hain. Humein sirf control runtime ko wapas dena hai. Hum yeh directly `trpl::yield_now`
function ko use karke kar sakte hain. Listing 17-17 mein, hum un tamam `trpl::sleep` calls ko
`trpl::yield_now` se replace karte hain.

<Listing number="17-17" caption="Operations ko progress karne se switch off karne dene ke liye `yield_now` use karna" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-17/src/main.rs:yields}}
```

</Listing>

Yeh code actual intent ke baare mein zyada clear hai aur `sleep` use karne ke muqable mein
significantly faster bhi ho sakta hai, kyun ke `sleep` jaise timers ki aksar is baat par limits
hoti hain ke woh kitni granular ho sakti hain. Misal ke taur par, `sleep` ka jo version hum use
kar rahe hain, woh hamesha kam az kam ek millisecond ke liye sleep karega, chahe hum usay
one nanosecond ki `Duration` dein. Dobara, modern computers *fast* hain: woh ek millisecond
mein bohot kuch kar sakte hain!

Is ka matlab hai ke async compute-bound tasks ke liye bhi useful ho sakta hai, yeh is baat par
depend karta hai ke aapka program aur kya kar raha hai, kyun ke yeh program ke different parts
ke darmiyan relationships ko structure karne ke liye ek useful tool provide karta hai (lekin
async state machine ke overhead ki cost ke saath). Yeh *cooperative multitasking* ki ek form
hai, jahan har future ke paas await points ke zariye yeh determine karne ki power hoti hai ke
woh control kab hand over kare. Is liye har future ki yeh responsibility bhi hoti hai ke woh
bohot der tak blocking na kare. Kuch Rust-based embedded operating systems mein, multitasking
ki *sirf* yahi qisam hoti hai!

Real-world code mein, aap aam tor par har single line par function calls ko await points ke
saath alternate nahi karenge, bilkul obvious hai. Is tarah control yield karna relatively
inexpensive hai, lekin free nahi hai. Bohot se cases mein, compute-bound task ko break up
karne ki koshish usay significantly slower bana sakti hai, is liye kabhi kabhi *overall*
performance ke liye operation ko thori der block hone dena behtar hota hai. Hamesha measure
karein taake pata chal sake ke aapke code ke actual performance bottlenecks kya hain. Lekin
underlying dynamic ko zehan mein rakhna important hai, khaas taur par agar aapko *serial* mein
bohot sara work hota hua nazar aa raha ho jab ke aap expect kar rahe thay ke woh concurrently
hoga!

### Building Our Own Async Abstractions

Hum futures ko ek doosre ke saath compose karke naye patterns bhi create kar sakte hain. Misal ke taur par, hum
pehle se maujood async building blocks ko use karke ek `timeout` function bana sakte hain. Jab hum complete
kar lenge, to result mein hamare paas ek aur building block hoga jise hum mazeed async abstractions create
karne ke liye use kar sakenge.

Listing 17-18 dikhati hai ke hum expect karenge ke yeh `timeout` ek slow
future ke saath kaise work kare.

<Listing number="17-18" caption="Time limit ke saath slow operation ko run karne ke liye hamare imagined `timeout` ko use karna" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-18/src/main.rs:here}}
```

</Listing>

Aaiye isay implement karte hain! Shuru karne ke liye, aaiye `timeout` ke API ke baare mein sochte hain:

* Isay khud ek async function hona chahiye taake hum isay await kar saken.
* Is ka pehla parameter ek future hona chahiye jise run karna hai. Hum isay generic bana sakte hain taake
  yeh kisi bhi future ke saath work kar sake.
* Is ka doosra parameter wait karne ka maximum time hoga. Agar hum `Duration` use karein, to isay
  `trpl::sleep` ko pass karna easy hoga.
* Isay ek `Result` return karna chahiye. Agar future successfully complete ho jata hai, to `Result` future
  ki produced value ke saath `Ok` hoga. Agar timeout pehle elapse ho jata hai, to `Result` us duration ke
  saath `Err` hoga jitni der timeout ne wait kiya.

Listing 17-19 mein yeh declaration dikhayi gayi hai.

<!-- This is not tested because it intentionally does not compile. -->

<Listing number="17-19" caption="`timeout` ki signature define karna" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-19/src/main.rs:declaration}}
```

</Listing>

Yeh hamare types ke goals ko satisfy karta hai. Ab aaiye us *behavior* ke baare mein sochte hain jo
humein chahiye: hum passed-in future ko duration ke against race karna chahte hain. Hum duration se ek
timer future banane ke liye `trpl::sleep` use kar sakte hain, aur us timer ko caller ke passed-in future
ke saath run karne ke liye `trpl::select` use kar sakte hain.

Listing 17-20 mein, hum `trpl::select` ko await karne ke result par match karke `timeout` implement karte hain.

<Listing number="17-20" caption="`select` aur `sleep` ke saath `timeout` define karna" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-20/src/main.rs:implementation}}
```

</Listing>

`trpl::select` ki implementation fair nahi hai: yeh hamesha arguments ko usi order mein poll karti hai jis
order mein woh pass kiye jate hain (doosri `select` implementations randomly choose karengi ke pehle kis
argument ko poll karna hai). Is liye, hum `future_to_try` ko `select` ko sab se pehle pass karte hain taake
usay complete hone ka mauqa mile, chahe `max_time` bohot short duration hi kyun na ho. Agar `future_to_try`
pehle finish hota hai, to `select`, `future_to_try` ke output ke saath `Left` return karega. Agar `timer`
pehle finish hota hai, to `select` timer ke output `()` ke saath `Right` return karega.

Agar `future_to_try` successfully complete hota hai aur humein `Left(output)` milta hai, to hum
`Ok(output)` return karte hain. Agar is ke bajaye sleep timer elapse ho jata hai aur humein `Right(())`
milta hai, to hum `_` ke zariye `()` ko ignore karte hain aur is ke bajaye `Err(max_time)` return karte hain.

Is ke saath, hamare paas do doosre async helpers se bana hua ek working `timeout` hai. Agar hum apna code
run karein, to yeh timeout ke baad failure mode print karega:

```text
Failed after 2 seconds
```

Kyun ke futures doosre futures ke saath compose ho sakte hain, aap chhote async building blocks ko use
karke bohot powerful tools build kar sakte hain. Misal ke taur par, aap isi approach ko timeouts ko
retries ke saath combine karne ke liye use kar sakte hain, aur phir unhein network calls jaisi operations
ke saath use kar sakte hain (jaise Listing 17-5 mein).

Practice mein, aap aam tor par directly `async` aur `await` ke saath kaam karenge, aur secondary taur par
`select` jaise functions aur `join!` macro jaise macros ko use karenge taake yeh control kiya ja sake ke
outermost futures kaise execute hote hain.

Ab humne ek hi waqt mein multiple futures ke saath kaam karne ke kai tareeqe dekhe hain. Agay, hum dekhenge
ke *streams* ke saath waqt ke saath sequence mein multiple futures ke saath kaise kaam kiya ja sakta hai.

[async-program]: ch17-01-futures-and-syntax.html#our-first-async-program
