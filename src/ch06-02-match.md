<!-- Old headings. Do not remove or links may break. -->

<a id="the-match-control-flow-operator"></a>

## The `match` Control Flow Construct

Rust mein `match` naam ka ek bohat powerful control flow construct hai jo aapko kisi value ka patterns ki ek series ke saath comparison karne deta hai aur phir is baat ki bunyaad par code execute karta hai ke kaunsa pattern match hota hai. Patterns literal values, variable names, wildcards aur bohat si doosri cheezon se mil kar ban sakte hain; [Chapter 19][ch19-00-patterns]<!-- ignore --> tamam different qisam ke patterns aur unke kaam ko cover karta hai. `match` ki power patterns ki expressiveness aur is fact se aati hai ke compiler confirm karta hai ke tamam possible cases handle kiye gaye hain.

`match` expression ko ek coin-sorting machine ki tarah samjhein: Coins ek track par slide karte hain jisme mukhtalif sizes ke holes hote hain, aur har coin us pehle hole se neeche girta hai jisme woh fit hota hai. Isi tarah, values `match` mein har pattern se guzarti hain, aur jis pehle pattern mein value “fit” hoti hai, value execution ke dauran use hone ke liye us se associated code block mein chali jati hai.

Coins ki baat ho hi rahi hai, to aaiye `match` ki example ke liye inhein use karte hain! Hum ek aisa function likh sakte hain jo ek unknown US coin leta hai aur, counting machine ki tarah, determine karta hai ke woh kaunsa coin hai aur uski value cents mein return karta hai, jaisa ke Listing 6-3 mein dikhaya gaya hai.

<Listing number="6-3" caption="Ek enum aur `match` expression jisme enum ke variants ko patterns ke taur par use kiya gaya hai">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-03/src/main.rs:here}}
```

</Listing>

Aaiye `value_in_cents` function mein `match` ko break down karte hain. Sab se pehle, hum `match` keyword ke baad ek expression list karte hain, jo is case mein `coin` value hai. Ye `if` ke saath use hone wali conditional expression se kaafi similar lagti hai, lekin yahan ek bara difference hai: `if` ke saath condition ka Boolean value mein evaluate hona zaroori hai, lekin yahan ye kisi bhi type ki ho sakti hai. Is example mein `coin` ki type `Coin` enum hai jo hum ne pehli line mein define ki thi.

Is ke baad `match` arms aati hain. Ek arm ke do parts hote hain: ek pattern aur kuch code. Yahan pehli arm ka pattern `Coin::Penny` value hai aur uske baad `=>` operator hai jo pattern aur run hone wale code ko separate karta hai. Is case mein code sirf value `1` hai. Har arm ko aglay arm se comma ke zariye separate kiya jata hai.

Jab `match` expression execute hoti hai, to ye resultant value ka har arm ke pattern ke saath order mein comparison karti hai. Agar koi pattern value se match kar jaye, to us pattern ke saath associated code execute hota hai. Agar woh pattern value se match na kare, to execution aglay arm ki taraf continue karti hai, bilkul coin-sorting machine ki tarah. Hum jitni arms chahein rakh sakte hain: Listing 6-3 mein hamari `match` mein chaar arms hain.

Har arm ke saath associated code ek expression hota hai, aur matching arm mein expression ki resultant value woh value hoti hai jo poori `match` expression ke liye return hoti hai.

Hum aam tor par curly brackets use nahi karte agar match arm ka code chhota ho, jaisa ke Listing 6-3 mein hai jahan har arm sirf ek value return karti hai. Agar aap `match` arm mein multiple lines of code run karna chahte hain, to aapko curly brackets use karne honge, aur phir arm ke baad comma optional hota hai. Misal ke taur par, following code har baar method ko `Coin::Penny` ke saath call karne par “Lucky penny!” print karta hai, lekin phir bhi block ki aakhri value, `1`, return karta hai:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-08-match-arm-multiple-lines/src/main.rs:here}}
```

### Patterns That Bind to Values

`match` arms ka ek aur useful feature ye hai ke ye un values ke parts ko bind kar sakti hain jo pattern se match hoti hain. Isi tarah hum enum variants ke andar se values extract kar sakte hain.

Misal ke taur par, aaiye apne enum variants mein se ek ko change karke uske andar data rakhte hain. 1999 se 2008 tak, United States ne aise quarters mint kiye jin ki ek side par 50 states mein se har state ke liye different design tha. Kisi doosre coin par state designs nahi the, is liye sirf quarters mein ye extra value hoti hai. Hum `Quarter` variant ko change karke is information ko apne `enum` mein add kar sakte hain taa-ke iske andar ek `UsState` value store ho, jaisa ke hum ne Listing 6-4 mein kiya hai.

<Listing number="6-4" caption="Ek `Coin` enum jisme `Quarter` variant ek `UsState` value bhi hold karta hai">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-04/src/main.rs:here}}
```

</Listing>

Tasawwur karein ke ek friend tamam 50 state quarters collect karne ki koshish kar raha hai. Jab hum apni loose change ko coin type ke mutabiq sort karenge, to hum har quarter ke saath associated state ka name bhi batayenge taa-ke agar woh aisa state ho jo hamare friend ke paas nahi hai, to woh use apni collection mein shamil kar sake.

Is code ki `match` expression mein, hum us pattern ke andar `state` naam ka ek variable add karte hain jo `Coin::Quarter` variant ki values se match karta hai. Jab koi `Coin::Quarter` match hota hai, to `state` variable us quarter ke state ki value ke saath bind ho jayega. Phir hum us arm ke code mein `state` ko is tarah use kar sakte hain:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-09-variable-in-pattern/src/main.rs:here}}
```

Agar hum `value_in_cents(Coin::Quarter(UsState::Alaska))` call karein, to `coin` ki value `Coin::Quarter(UsState::Alaska)` hogi. Jab hum is value ka har `match` arm ke saath comparison karte hain, to in mein se koi bhi match nahi hota jab tak hum `Coin::Quarter(state)` tak nahi pohanchte. Us point par, `state` ki binding ki value `UsState::Alaska` hogi. Phir hum `println!` expression mein us binding ko use kar sakte hain aur is tarah `Quarter` ke `Coin` enum variant ke andar mojood state value hasil kar sakte hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="matching-with-optiont"></a>

### The `Option<T>` `match` Pattern

Pichlay section mein hum `Option<T>` ko use karte waqt `Some` case ke andar se inner `T` value hasil karna chahte the; hum `Option<T>` ko `match` use karke bhi handle kar sakte hain, bilkul usi tarah jaise hum ne `Coin` enum ke saath kiya tha! Coins ka comparison karne ke bajaye, hum `Option<T>` ke variants ka comparison karenge, lekin `match` expression jis tarah kaam karti hai woh same rahega.

Maan lein hum ek aisa function likhna chahte hain jo `Option<i32>` leta hai aur, agar uske andar koi value ho, to us value mein 1 add karta hai. Agar andar koi value na ho, to function ko `None` value return karni chahiye aur koi operation perform karne ki koshish nahi karni chahiye.

`match` ki wajah se ye function likhna bohat aasaan hai, aur ye Listing 6-5 jaisa nazar aayega.

<Listing number="6-5" caption="Ek function jo `Option<i32>` par `match` expression use karta hai">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:here}}
```

</Listing>

Aaiye `plus_one` ki pehli execution ko mazeed detail mein examine karte hain. Jab hum `plus_one(five)` call karte hain, to `plus_one` ke body mein `x` variable ki value `Some(5)` hogi. Phir hum is ka comparison har `match` arm ke saath karte hain:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

`Some(5)` value `None` pattern se match nahi karti, is liye hum aglay arm ki taraf continue karte hain:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:second_arm}}
```

Kya `Some(5)` `Some(i)` pattern se match karti hai? Bilkul karti hai! Hamare paas same variant hai. `i`, `Some` ke andar mojood value ke saath bind ho jata hai, is liye `i` ki value `5` ho jati hai. Phir match arm mein mojood code execute hota hai, is liye hum `i` ki value mein 1 add karte hain aur apne total `6` ko andar rakhte hue ek nayi `Some` value create karte hain.

Ab Listing 6-5 mein `plus_one` ki doosri call ko dekhein, jahan `x` `None` hai. Hum `match` mein enter karte hain aur pehle arm ke saath comparison karte hain:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

Ye match karta hai! Add karne ke liye koi value nahi hai, is liye program ruk jata hai aur `=>` ke right side par mojood `None` value return karta hai. Kyun ke pehla arm match ho gaya, kisi doosre arm ka comparison nahi kiya jata.

`match` aur enums ko combine karna bohat si situations mein useful hai. Aap Rust code mein is pattern ko bohat baar dekhenge: enum par `match` karein, uske andar mojood data ke saath ek variable bind karein, aur phir us data ki bunyaad par code execute karein. Shuru mein ye thora tricky lagta hai, lekin jab aap iske aadhi ho jayenge, to aap chahenge ke ye har language mein hota. Ye consistently users ke pasandeeda features mein se ek hai.

### Matches Are Exhaustive

`match` ka ek aur pehlu hai jis par humein baat karni hai: Arms ke patterns ko tamam possibilities ko cover karna zaroori hai. Apne `plus_one` function ke is version ko dekhein, jisme ek bug hai aur ye compile nahi hoga:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/src/main.rs:here}}
```

Hum ne `None` case ko handle nahi kiya, is liye ye code ek bug ka sabab banega. Khush qismati se, ye aisa bug hai jise Rust pakarna jaanti hai. Agar hum is code ko compile karne ki koshish karein, to humein ye error milega:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/output.txt}}
```

Rust jaanti hai ke hum ne har possible case ko cover nahi kiya aur ye bhi jaanti hai ke hum kaunsa pattern bhool gaye hain! Rust mein `match` *exhaustive* hoti hain: Code ko valid hone ke liye humein har aakhri possibility ko exhaust karna zaroori hai. Khaas taur par `Option<T>` ke case mein, jab Rust humein `None` case ko explicitly handle karna bhoolne se rokta hai, to ye humein ye assume karne se protect karta hai ke hamare paas ek value hai jabke ho sakta hai wahan null ho, aur is tarah pehle discuss ki gayi billion-dollar mistake ko impossible bana deta hai.

### Catch-All Patterns and the `_` Placeholder

Enums ko use karte hue, hum kuch khaas values ke liye special actions bhi le sakte hain, lekin baqi tamam values ke liye ek default action le sakte hain. Tasawwur karein ke hum ek game implement kar rahe hain jahan agar dice roll par 3 aaye, to aapka player move nahi karta balki ek fancy nayi hat hasil karta hai. Agar 7 aaye, to aapka player ek fancy hat kho deta hai. Baqi tamam values ke liye, aapka player game board par utni spaces move karta hai jitna number roll hua hai. Yahan ek `match` hai jo is logic ko implement karti hai, jisme dice roll ko random value ke bajaye hardcode kiya gaya hai, aur baqi tamam logic ko aise functions ke zariye represent kiya gaya hai jin ke bodies nahi hain kyun ke is example mein unhein actually implement karna scope se bahar hai:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-15-binding-catchall/src/main.rs:here}}
```

Pehli do arms ke patterns literal values `3` aur `7` hain. Aakhri arm ke liye, jo baqi tamam possible values ko cover karti hai, pattern woh variable hai jise hum ne `other` name dene ka faisla kiya hai. `other` arm ke liye run hone wala code is variable ko `move_player` function mein pass karke use karta hai.

Ye code compile ho jata hai, halaanke hum ne un tamam possible values ko list nahi kiya jo ek `u8` rakh sakta hai, kyun ke aakhri pattern un tamam values se match karega jo specifically list nahi ki gayi hain. Ye catch-all pattern is requirement ko poora karta hai ke `match` exhaustive honi chahiye. Note karein ke humein catch-all arm ko aakhir mein rakhna hota hai kyun ke patterns ko order mein evaluate kiya jata hai. Agar hum catch-all arm ko pehle rakh dete, to baqi arms kabhi run hi na hotin, is liye agar hum catch-all ke baad arms add karein to Rust humein warning dega!

Rust mein ek aisa pattern bhi hai jo hum us waqt use kar sakte hain jab humein catch-all chahiye ho lekin hum catch-all pattern mein value ko *use* nahi karna chahte: `_` ek special pattern hai jo kisi bhi value se match karta hai aur us value ke saath bind nahi hota. Ye Rust ko batata hai ke hum is value ko use nahi karne wale, is liye Rust humein unused variable ke baare mein warning nahi dega.

Aaiye game ke rules change karte hain: Ab agar aap 3 ya 7 ke ilawa kuch bhi roll karein, to aapko dobara roll karna hoga. Ab humein catch-all value ko use karne ki zaroorat nahi hai, is liye hum apne code ko `other` naam ke variable ke bajaye `_` use karne ke liye change kar sakte hain:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-16-underscore-catchall/src/main.rs:here}}
```

Ye example bhi exhaustiveness ki requirement ko poora karta hai kyun ke hum aakhri arm mein baqi tamam values ko explicitly ignore kar rahe hain; hum ne kuch bhi nahi bhoola.

Aakhir mein, hum game ke rules ko ek baar aur change karenge taa-ke agar aap 3 ya 7 ke ilawa kuch bhi roll karein, to aapki turn par aur kuch na ho. Hum `_` arm ke saath code ke taur par unit value (empty tuple type jiska hum ne [“The Tuple Type”][tuples]<!-- ignore --> section mein zikr kiya tha) use karke isay express kar sakte hain:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-17-underscore-unit/src/main.rs:here}}
```

Yahan hum Rust ko explicitly bata rahe hain ke hum kisi aisi doosri value ko use nahi karne wale jo kisi earlier arm mein pattern se match nahi hui, aur hum is case mein koi code run nahi karna chahte.

Patterns aur matching ke baare mein mazeed maloomat hum [Chapter 19][ch19-00-patterns]<!-- ignore --> mein cover karenge. Filhaal, hum `if let` syntax ki taraf barhte hain, jo un situations mein useful ho sakti hai jahan `match` expression kuch zyada wordy ho jati hai.

[tuples]: ch03-02-data-types.html#the-tuple-type
[ch19-00-patterns]: ch19-00-patterns.html
