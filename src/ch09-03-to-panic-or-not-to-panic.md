## `panic!` Karna Hai Ya Nahi

To phir aap kaise decide karte hain ke kab `panic!` call karna chahiye aur kab `Result` return karna chahiye? Jab code panic karta hai, to recover karne ka koi tareeqa nahi hota. Aap kisi bhi error situation ke liye `panic!` call kar sakte hain, chahe recover karne ka koi mumkin tareeqa ho ya na ho, lekin phir aap calling code ki taraf se ye decision le rahe hote hain ke ye situation unrecoverable hai. Jab aap `Result` value return karne ka intekhab karte hain, to aap calling code ko options dete hain. Calling code apni situation ke liye munasib tareeqe se recover karne ki koshish kar sakta hai, ya ye decide kar sakta hai ke is case mein `Err` value unrecoverable hai, is liye woh `panic!` call karke aapke recoverable error ko unrecoverable error mein convert kar sakta hai. Is liye, jab aap koi aisa function define kar rahe hon jo fail ho sakta hai, to `Result` return karna ek achha default choice hai.

Examples, prototype code, aur tests jaisi situations mein, `Result` return karne ke bajaye aisa code likhna zyada munasib hota hai jo panic kare. Aaiye explore karte hain ke aisa kyun hai, aur phir un situations par baat karte hain jahan compiler ye nahi bata sakta ke failure impossible hai, lekin aap, ek insan ke taur par, ye jaante hain. Chapter ke aakhir mein kuch general guidelines di jayengi jo aapko library code mein panic karne ya na karne ka faisla karne mein madad karengi.

### Examples, Prototype Code, aur Tests

Jab aap kisi concept ko illustrate karne ke liye example likh rahe hon, to example mein robust error-handling code bhi shamil karna usay kam clear kar sakta hai. Examples mein ye samjha jata hai ke `unwrap` jaisi method ki call, jo panic kar sakti hai, us tareeqe ke liye ek placeholder hai jis ke zariye aap apni application mein errors ko handle karna chahenge, aur ye tareeqa is baat ke mutabiq different ho sakta hai ke aapka baqi code kya kar raha hai.

Isi tarah, jab aap prototyping kar rahe hon aur abhi ye decide karne ke liye tayyar na hon ke errors ko kis tarah handle karna hai, to `unwrap` aur `expect` methods bohat handy hoti hain. Ye aapke code mein clear markers chhor deti hain taa-ke jab aap apne program ko zyada robust banane ke liye tayyar hon, to aapko pata ho ke kahan changes karne hain.

Agar test mein koi method call fail ho jaye, to aap chahenge ke poora test fail ho jaye, chahe woh method test ki ja rahi functionality ka hissa na bhi ho. Kyun ke `panic!` hi woh tareeqa hai jis se kisi test ko failure ke taur par mark kiya jata hai, is liye `unwrap` ya `expect` call karna bilkul wohi hai jo hona chahiye.

<!-- Old headings. Do not remove or links may break. -->

<a id="cases-in-which-you-have-more-information-than-the-compiler"></a>

### Jab Aapke Paas Compiler Se Zyada Information Ho

`expect` ko call karna us waqt bhi munasib hota hai jab aapke paas koi aur logic ho jo ye ensure karta ho ke `Result` mein `Ok` value hogi, lekin compiler us logic ko samajh nahi sakta. Phir bhi aapke paas ek `Result` value hogi jise aapko handle karna zaroori hai: Aap jis operation ko call kar rahe hain, us mein aam taur par fail hone ka possibility ab bhi maujood hota hai, chahe aapki particular situation mein logically failure impossible ho. Agar aap code ko manually inspect karke ye ensure kar sakte hain ke aapke paas kabhi `Err` variant nahi aayega, to `expect` call karna bilkul acceptable hai aur argument text mein ye reason document kar dein ke aapko kyun lagta hai ke `Err` variant kabhi nahi aayega. Yahan ek example hai:

```rust
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-08-unwrap-that-cant-fail/src/main.rs:here}}
```

Hum ek hardcoded string ko parse karke `IpAddr` instance create kar rahe hain. Hum dekh sakte hain ke `127.0.0.1` ek valid IP address hai, is liye yahan `expect` use karna acceptable hai. Lekin ek hardcoded, valid string hone se `parse` method ka return type change nahi hota: Humein ab bhi ek `Result` value milti hai, aur compiler ab bhi humein `Result` ko is tarah handle karne par majboor karega jaise `Err` variant ek possibility ho, kyun ke compiler itna smart nahi hai ke ye dekh sake ke ye string hamesha ek valid IP address hai. Agar IP address ki string program mein hardcode hone ke bajaye user ki taraf se aati aur is liye *failure* ka possibility waqai hota, to hum definitely `Result` ko is ke bajaye zyada robust tareeqe se handle karna chahte. Ye assumption mention karna ke IP address hardcoded hai, humein is baat ki yaad dilayega ke agar future mein humein IP address kisi doosre source se hasil karna pade, to `expect` ko better error-handling code se replace karein.

### Error Handling Ke Liye Guidelines

Ye mashwara diya jata hai ke jab mumkin ho ke aapka code kisi bad state mein chala jaye, to aapka code panic kare. Is context mein, *bad state* se murad aisi situation hai jahan koi assumption, guarantee, contract, ya invariant toot gaya ho, jaise jab invalid values, ek doosre se contradictory values, ya missing values aapke code ko pass ki jayein—aur saath hi neeche di gayi conditions mein se ek ya zyada maujood hon:

* Bad state aisi cheez ho jo unexpected ho, bajaye is ke ke woh kabhi kabhi likely hone wali cheez ho, jaise user ka data ko ghalat format mein enter karna.
* Is point ke baad aapke code ko is baat par rely karna ho ke woh bad state mein nahi hai, bajaye is ke ke har step par problem ko check kiya jaye.
* Aapke paas is information ko un types mein encode karne ka koi achha tareeqa na ho jo aap use kar rahe hain. Chapter 18 mein [“Encoding States and Behavior as Types”][encoding]<!-- ignore --> mein hum ek example ke zariye samjhenge ke is se hamari kya murad hai.

Agar koi aapke code ko call karta hai aur aisi values pass karta hai jo sense nahi banatin, to agar mumkin ho to error return karna behtar hai taa-ke library ka user khud decide kar sake ke is situation mein kya karna hai. Lekin aisi situations mein jahan continue karna insecure ya harmful ho sakta hai, behtar choice `panic!` call karna aur aapki library use karne wale person ko unke code mein bug ke baare mein alert karna ho sakti hai taa-ke woh development ke dauran usay fix kar saken. Isi tarah, agar aap external code ko call kar rahe hon jo aapke control mein nahi hai aur woh ek invalid state return karta hai jise aapke paas fix karne ka koi tareeqa nahi, to `panic!` aksar munasib hota hai.

Lekin jab failure expected ho, to `panic!` call karne ke bajaye `Result` return karna zyada munasib hai. Is ki examples mein parser ko malformed data diya jana ya HTTP request ka aisa status return karna shamil hai jo indicate karta ho ke aap rate limit tak pohanch gaye hain. In cases mein `Result` return karna indicate karta hai ke failure ek expected possibility hai aur calling code ko decide karna hoga ke usay kis tarah handle karna hai.

Jab aapka code koi aisa operation perform karta hai jo invalid values ke saath call kiye jane par user ko risk mein daal sakta hai, to aapke code ko pehle verify karna chahiye ke values valid hain aur agar values valid na hon to panic karna chahiye. Ye zyada tar safety reasons ki wajah se hai: Invalid data par operate karne ki koshish aapke code ko vulnerabilities ke liye expose kar sakti hai. Ye main reason hai ke standard library `panic!` call karti hai agar aap out-of-bounds memory access ki koshish karein: Aisi memory ko access karne ki koshish jo current data structure se belong nahi karti, ek common security problem hai. Functions ke paas aksar *contracts* hote hain: Unka behavior sirf us waqt guaranteed hota hai jab inputs particular requirements ko meet karte hon. Jab contract violate ho, to panic karna sense banata hai kyun ke contract violation hamesha caller-side bug ko indicate karta hai, aur ye aisi qisam ki error nahi hai jise aap calling code se explicitly handle karwana chahenge. Asal mein, calling code ke liye recover karne ka koi reasonable tareeqa nahi hota; calling *programmers* ko code fix karna hota hai. Function ke contracts, khaas taur par jab unki violation panic ka sabab banegi, function ki API documentation mein explain kiye jane chahiye.

Lekin apne tamam functions mein bohat saari error checks rakhna verbose aur annoying hoga. Khush qismati se, aap Rust ke type system (aur is tarah compiler ki taraf se ki jane wali type checking) ko use karke bohat se checks khud karwa sakte hain. Agar aapke function mein parameter ke taur par koi particular type hai, to aap apne code ki logic ko is yaqeen ke saath aage barha sakte hain ke compiler ne pehle hi ensure kar diya hai ke aapke paas ek valid value hai. Misal ke taur par, agar aapke paas `Option` ke bajaye koi type ho, to aapka program *something* hone ki tawaqqo karta hai, *nothing* ki nahi. Phir aapke code ko `Some` aur `None` variants ke liye do cases handle karne ki zaroorat nahi hoti: Is ke paas sirf ek case hoga jahan definitely ek value mojood hogi. Aapke function ko nothing pass karne ki koshish karne wala code compile hi nahi hoga, is liye aapke function ko runtime par us case ko check karne ki zaroorat nahi hogi. Ek aur example unsigned integer type jaise `u32` ko use karna hai, jo ensure karta hai ke parameter kabhi negative nahi hoga.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-custom-types-for-validation"></a>

### Validation Ke Liye Custom Types

Aaiye Rust ke type system ko use karke valid value ensure karne ke idea ko ek qadam aur aage le jate hain aur validation ke liye ek custom type create karne ko dekhte hain. Chapter 2 ke guessing game ko yaad karein, jismein hamare code ne user se 1 aur 100 ke darmiyan ek number guess karne ko kaha tha. Hum ne secret number ke saath check karne se pehle kabhi ye validate nahi kiya tha ke user ka guess un numbers ke darmiyan hai; hum ne sirf ye validate kiya tha ke guess positive hai. Is case mein consequences bohat serious nahi the: Hamara “Too high” ya “Too low” ka output phir bhi correct hota. Lekin ye ek useful enhancement hoti ke user ko valid guesses ki taraf guide kiya jaye aur jab user range se bahar ka number guess kare to us waqt different behavior ho, bajaye is ke ke user, misal ke taur par, letters type kar de.

Isay karne ka ek tareeqa ye hoga ke guess ko sirf `u32` ke bajaye `i32` ke taur par parse kiya jaye taa-ke potentially negative numbers ki bhi ijazat ho, aur phir check add kiya jaye ke number range ke andar hai, jaise:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-09-guess-out-of-range/src/main.rs:here}}
```

</Listing>

`if` expression check karti hai ke hamari value range se bahar hai ya nahi, user ko problem ke baare mein batati hai, aur loop ki next iteration start karne ke liye `continue` call karti hai aur ek aur guess mangti hai. `if` expression ke baad, hum `guess` aur secret number ke darmiyan comparisons is yaqeen ke saath aage barha sakte hain ke `guess` 1 aur 100 ke darmiyan hai.

Lekin ye ideal solution nahi hai: Agar ye bilkul critical hota ke program sirf 1 aur 100 ke darmiyan values par operate kare, aur bohat se functions ki ye requirement hoti, to har function mein is tarah ka check rakhna tedious hota (aur performance ko bhi impact kar sakta tha).

Is ke bajaye, hum ek dedicated module mein ek naya type bana sakte hain aur validations ko ek function mein rakh sakte hain jo type ka instance create kare, bajaye is ke ke validations ko har jagah repeat kiya jaye. Is tarah, functions ke liye apni signatures mein naye type ko use karna aur unhein milne wali values ko confidence ke saath use karna safe hota hai. Listing 9-13 ek tareeqa dikhati hai jisse `Guess` type define kiya ja sakta hai jo sirf us waqt `Guess` ka instance create karega jab `new` function ko 1 aur 100 ke darmiyan koi value mile.

<Listing number="9-13" caption="A `Guess` type that will only continue with values between 1 and 100" file-name="src/guessing_game.rs">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-13/src/guessing_game.rs}}
```

</Listing>

Note karein ke *src/guessing_game.rs* mein ye code ek module declaration `mod guessing_game;` add karne par depend karta hai jo *src/lib.rs* mein hona chahiye, lekin hum ne yahan nahi dikhaya. Is naye module ki file ke andar, hum `Guess` naam ka ek struct define karte hain jis mein `value` naam ka field hai jo ek `i32` hold karta hai. Yahin number store kiya jayega.

Phir, hum `Guess` par `new` naam ka ek associated function implement karte hain jo `Guess` values ke instances create karta hai. `new` function ko is tarah define kiya gaya hai ke is ka ek parameter `value` hai jo `i32` type ka hai aur ye ek `Guess` return karta hai. `new` function ki body mein code `value` ko test karta hai taa-ke ye ensure kiya ja sake ke ye 1 aur 100 ke darmiyan hai. Agar `value` ye test pass nahi karta, to hum `panic!` call karte hain, jo calling code likhne wale programmer ko alert karega ke unke code mein ek bug hai jise unhein fix karna hai, kyun ke is range se bahar `value` ke saath `Guess` create karna us contract ko violate karega jis par `Guess::new` rely kar raha hai. Woh conditions jin mein `Guess::new` panic kar sakta hai, uski public-facing API documentation mein discuss ki jani chahiye; hum Chapter 14 mein API documentation mein `panic!` ke possibility ko indicate karne wali documentation conventions ko cover karenge. Agar `value` test pass kar leta hai, to hum ek naya `Guess` create karte hain jis ka `value` field `value` parameter par set hota hai aur `Guess` return kar dete hain.

Agla step ye hai ke hum `value` naam ka ek method implement karte hain jo `self` ko borrow karta hai, is ka koi aur parameter nahi hota, aur ye ek `i32` return karta hai. Is qisam ke method ko kabhi kabhi *getter* kaha jata hai kyun ke is ka maqsad apne fields se kuch data hasil karke usay return karna hota hai. Ye public method is liye zaroori hai kyun ke `Guess` struct ka `value` field private hai. Ye important hai ke `value` field private ho taa-ke `Guess` struct ko use karne wale code ko `value` directly set karne ki ijazat na ho: `guessing_game` module ke bahar ka code *must* `Guess` ka instance create karne ke liye `Guess::new` function use kare, aur is tarah ye ensure ho ke `Guess` ke paas aisi `value` hone ka koi tareeqa na ho jise `Guess::new` function ki conditions ke zariye check na kiya gaya ho.

Ek aisa function jis ka parameter ya return value sirf 1 aur 100 ke darmiyan numbers ho, phir apni signature mein ye declare kar sakta hai ke woh `i32` ke bajaye `Guess leta ya return karta hai aur usay apni body mein koi additional checks karne ki zaroorat nahi hogi.

## Summary

Rust ke error-handling features is tarah design kiye gaye hain ke aap zyada robust code likh saken. `panic!` macro signal karta hai ke aapka program aisi state mein hai jise woh handle nahi kar sakta, aur aapko process ko stop karne deta hai bajaye is ke ke invalid ya incorrect values ke saath aage barhne ki koshish ki jaye. `Result` enum Rust ke type system ko use karke indicate karta hai ke operations is tarah fail ho sakti hain ke aapka code recover kar sake. Aap `Result` ko use karke apne code ko call karne wale code ko bhi bata sakte hain ke usay potential success ya failure ko handle karna hoga. `panic!` aur `Result` ko appropriate situations mein use karna inevitable problems ka saamna karte hue aapke code ko zyada reliable banayega.

Ab jab aap dekh chuke hain ke standard library `Option` aur `Result` enums ke saath generics ko useful tareeqon se kaise use karti hai, to hum baat karenge ke generics kaise kaam karte hain aur aap unhein apne code mein kaise use kar sakte hain.

[encoding]: ch18-03-oo-design-patterns.html#encoding-states-and-behav

