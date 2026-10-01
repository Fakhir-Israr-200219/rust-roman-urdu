## Variables and Mutability

Jaisa ke [“Storing Values with Variables”][storing-values-with-variables]<!-- ignore --> section mein mention kiya gaya tha, by default variables immutable hote hain. Ye un bohat se tareeqon mein se ek hai jin ke zariye Rust aapko apna code is tarah likhne ki taraf guide karta hai ke aap Rust ki safety aur easy concurrency ka faida utha saken. Lekin aapke paas apne variables ko mutable banane ka option bhi hota hai. Aaiye explore karte hain ke Rust aapko immutability ko prefer karne ki taraf kyun encourage karta hai aur kab aur kyun aap is se hatna chahenge.

Jab koi variable immutable hota hai, to ek baar koi value kisi name ke saath bind ho jaye, aap us value ko change nahi kar sakte. Is baat ko samajhne ke liye, apni *projects* directory mein `cargo new variables` use karke *variables* naam ka ek naya project banayein.

Phir, apni nayi *variables* directory mein *src/main.rs* open karein aur uske code ko following code se replace karein, jo abhi compile nahi hoga:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/src/main.rs}}
```

Program ko `cargo run` use karke save aur run karein. Aapko immutability error ke baare mein ek error message milna chahiye, jaisa ke is output mein dikhaya gaya hai:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/output.txt}}
```

Ye example dikhata hai ke compiler aapko apne programs mein errors dhoondhne mein kaise help karta hai. Compiler errors frustrating ho sakte hain, lekin asal mein unka sirf ye matlab hota hai ke aapka program abhi safely woh kaam nahi kar raha jo aap us se karwana chahte hain; iska ye matlab *bilkul nahi* ke aap ek achhe programmer nahi hain! Experienced Rustaceans ko bhi compiler errors milte rehte hain.

Aapko error message `` cannot assign twice to immutable variable `x` `` is liye mila kyun ke aap ne immutable `x` variable ko doosri value assign karne ki koshish ki.

Ye important hai ke jab hum kisi aisi value ko change karne ki koshish karein jo immutable designate ki gayi hai, to humein compile-time errors milen, kyun ke yahi situation bugs ka sabab ban sakti hai. Agar hamare code ka ek hissa is assumption par kaam kar raha ho ke koi value kabhi change nahi hogi aur hamare code ka doosra hissa us value ko change kar deta hai, to mumkin hai ke code ka pehla hissa woh kaam na kare jo use karna chahiye tha. Is qisam ke bug ki wajah baad mein track down karna mushkil ho sakta hai, khaas taur par jab code ka doosra hissa value ko sirf *kabhi kabhi* change karta ho. Rust compiler guarantee karta hai ke jab aap state karte hain ke koi value change nahi hogi, to woh waqai change nahi hogi, is liye aapko khud ise track karne ki zaroorat nahi hoti. Is tarah, aapke code ko samajhna aasaan ho jata hai.

Lekin mutability bohat useful ho sakti hai aur code likhna zyada convenient bana sakti hai. Halanke variables by default immutable hote hain, aap variable name ke aage `mut` add karke unhein mutable bana sakte hain, jaisa ke aap ne [Chapter 2][storing-values-with-variables]<!-- ignore --> mein kiya tha. `mut` add karna code ke future readers ko aapka intent bhi convey karta hai, kyun ke is se pata chalta hai ke code ke doosre parts is variable ki value ko change karenge.

Misal ke taur par, aaiye *src/main.rs* ko following code se change karte hain:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/src/main.rs}}
```

Ab jab hum program run karte hain, to humein ye milta hai:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/output.txt}}
```

Jab `mut` use kiya jata hai, to humein `x` ke saath bound value ko `5` se `6` mein change karne ki permission hoti hai. Aakhir mein, mutability use karni hai ya nahi, ye faisla aap par hai aur is baat par depend karta hai ke us particular situation mein aapko kya cheez zyada clear lagti hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="constants"></a>

### Declaring Constants

Immutable variables ki tarah, *constants* bhi aisi values hoti hain jo kisi name ke saath bound hoti hain aur jinhein change karne ki ijazat nahi hoti, lekin constants aur variables ke darmiyan kuch differences hain.

Sab se pehle, aap constants ke saath `mut` use nahi kar sakte. Constants sirf by default immutable nahi hote—woh hamesha immutable hote hain. Aap constants ko `let` keyword ke bajaye `const` keyword use karke declare karte hain, aur value ki type ko *must* annotate karna hota hai. Hum types aur type annotations ko agle section, [“Data Types”][data-types]<!-- ignore -->, mein cover karenge, is liye filhaal details ki fikr na karein. Bas itna yaad rakhein ke aapko type hamesha annotate karni hoti hai.

Constants ko kisi bhi scope mein declare kiya ja sakta hai, jisme global scope bhi shamil hai, jo un values ke liye inhein useful banata hai jin ke baare mein code ke bohat se parts ko maloom hona zaroori hota hai.

Aakhri difference ye hai ke constants ko sirf ek constant expression par set kiya ja sakta hai, kisi aisi value ke result par nahi jo sirf runtime par calculate ki ja sakti ho.

Yahan constant declaration ki ek example hai:

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

Constant ka name `THREE_HOURS_IN_SECONDS` hai, aur iski value 60 (ek minute mein seconds ki tadaad) ko 60 (ek ghante mein minutes ki tadaad) se aur phir 3 (is program mein hum jitne ghanton ko count karna chahte hain) se multiply karne ke result par set ki gayi hai. Constants ke liye Rust ki naming convention ye hai ke tamam letters uppercase hon aur words ke darmiyan underscores hon. Compiler compile time par limited set of operations ko evaluate kar sakta hai, jo humein is value ko 10,800 par set karne ke bajaye aise likhne ka option deta hai jo samajhna aur verify karna aasaan hai. Constants declare karte waqt kaun se operations use kiye ja sakte hain, is ke baare mein mazeed maloomat ke liye [Rust Reference’s section on constant evaluation][const-eval] dekhein.

Constants poore waqt valid rehte hain jab tak program run ho raha hota hai, us scope ke andar jahan unhein declare kiya gaya ho. Ye property constants ko aapke application domain ki un values ke liye useful banati hai jin ke baare mein program ke multiple parts ko maloom hona zaroori ho sakta hai, jaise kisi game mein kisi player ke earn karne ki maximum points ki tadaad, ya light ki speed.

Apne program mein mukhtalif jagahon par use hone wali hardcoded values ko constants ke taur par name dena future mein code maintain karne wale logon ko us value ka meaning samajhne mein madad karta hai. Is ka ye faida bhi hai ke agar future mein hardcoded value ko update karne ki zaroorat pade, to aapke code mein sirf ek jagah hogi jahan aapko change karna hoga.

### Shadowing

Jaisa ke aap ne [Chapter 2][comparing-the-guess-to-the-secret-number]<!-- ignore --> ke guessing game tutorial mein dekha, aap kisi previous variable ke same name ke saath ek naya variable declare kar sakte hain. Rustaceans kehte hain ke pehla variable doosre variable ke zariye *shadowed* ho gaya hai, jis ka matlab hai ke jab aap variable ka name use karenge to compiler doosre variable ko dekhega. Effectively, doosra variable pehle variable ko overshadow kar deta hai aur variable name ke tamam uses ko apne liye le leta hai, jab tak ke woh khud shadowed na ho jaye ya scope khatam na ho jaye. Hum isi variable ka name use karke aur `let` keyword ko dobara use karke variable ko shadow kar sakte hain:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/src/main.rs}}
```

Ye program sab se pehle `x` ko value `5` ke saath bind karta hai. Phir, `let x =` ko dobara use karke ye ek naya variable `x` create karta hai, original value mein `1` add karta hai, jis se `x` ki value `6` ho jati hai. Phir, curly brackets ke zariye create kiye gaye ek inner scope ke andar, teesra `let` statement bhi `x` ko shadow karta hai aur ek naya variable create karta hai, previous value ko `2` se multiply karke `x` ki value `12` kar deta hai. Jab woh scope khatam ho jata hai, to inner shadowing bhi khatam ho jati hai aur `x` dobara `6` ban jata hai. Jab hum is program ko run karte hain, to ye following output dega:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/output.txt}}
```

Shadowing kisi variable ko `mut` mark karne se different hai, kyun ke agar hum galti se `let` keyword use kiye baghair is variable ko dobara assign karne ki koshish karein, to humein compile-time error milega. `let` use karke hum kisi value par kuch transformations perform kar sakte hain aur un transformations ke complete hone ke baad variable ko immutable rehne de sakte hain.

`mut` aur shadowing ke darmiyan doosra difference ye hai ke kyun ke `let` keyword ko dobara use karte waqt hum effectively ek naya variable create kar rahe hote hain, hum value ki type ko change kar sakte hain aur phir bhi wahi name reuse kar sakte hain. Misal ke taur par, maan lein ke hamara program user se poochta hai ke woh kisi text ke darmiyan kitni spaces chahta hai, aur user space characters enter karke jawab deta hai, aur phir hum us input ko ek number ke taur par store karna chahte hain:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-04-shadowing-can-change-types/src/main.rs:here}}
```

Pehla `spaces` variable string type ka hai, aur doosra `spaces` variable number type ka hai. Is tarah shadowing humein `spaces_str` aur `spaces_num` jaise different names sochne ki zaroorat se bacha leti hai; is ke bajaye, hum simple `spaces` name ko reuse kar sakte hain. Lekin agar hum is ke liye `mut` use karne ki koshish karein, jaisa ke yahan dikhaya gaya hai, to humein compile-time error milega:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/src/main.rs:here}}
```

Error kehta hai ke humein variable ki type ko mutate karne ki ijazat nahi hai:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/output.txt}}
```

Ab jab hum explore kar chuke hain ke variables kaise kaam karte hain, to aaiye dekhein ke unki aur kaun si data types ho sakti hain.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[data-types]: ch03-02-data-types.html#data-types
[storing-values-with-variables]: ch02-00-guessing-game-tutorial.html#storing-values-with-variables
[const-eval]: ../reference/const_eval.html

