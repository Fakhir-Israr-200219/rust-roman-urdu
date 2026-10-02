## Methods

Methods functions ki tarah hoti hain: Hum unhein `fn` keyword aur ek name ke saath declare karte hain, inke parameters aur return value ho sakti hai, aur in mein kuch aisa code hota hai jo method ko kahin aur se call kiye jane par run hota hai. Functions ke baraks, methods ko kisi struct ke context ke andar define kiya jata hai (ya enum ya trait object ke context mein, jin par hum respectively [Chapter 6][enums]<!-- ignore --> aur [Chapter 18][trait-objects]<!-- ignore --> mein baat karenge), aur inka pehla parameter hamesha `self` hota hai, jo us struct ke instance ko represent karta hai jis par method call ki ja rahi hoti hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="defining-methods"></a>

### Method Syntax

Aaiye `Rectangle` instance ko parameter ke taur par lene wale `area` function ko change karke usay `Rectangle` struct par define kiya gaya `area` method banate hain, jaisa ke Listing 5-13 mein dikhaya gaya hai.

<Listing number="5-13" file-name="src/main.rs" caption="`Rectangle` struct par ek `area` method define karna">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-13/src/main.rs}}
```

</Listing>

Function ko `Rectangle` ke context mein define karne ke liye, hum `Rectangle` ke liye ek `impl` (implementation) block shuru karte hain. Is `impl` block ke andar mojood har cheez `Rectangle` type ke saath associated hogi. Phir, hum `area` function ko `impl` ke curly brackets ke andar move karte hain aur signature mein pehle (aur is case mein, sirf) parameter ko aur body ke andar har jagah `self` mein change kar dete hain. `main` mein, jahan hum ne `area` function ko call karke `rect1` ko argument ke taur par pass kiya tha, wahan hum apne `Rectangle` instance par `area` method ko call karne ke liye *method syntax* use kar sakte hain. Method syntax instance ke baad aati hai: Hum ek dot, phir method name, parentheses, aur koi bhi arguments add karte hain.

`area` ki signature mein hum `rectangle: &Rectangle` ke bajaye `&self` use karte hain. `&self` asal mein `self: &Self` ka short form hai. Ek `impl` block ke andar, `Self` us type ka alias hota hai jis ke liye `impl` block hai. Methods ke first parameter ke taur par `Self` type ka `self` naam ka parameter hona zaroori hai, is liye Rust humein first parameter ki jagah sirf `self` ka name use karke ise abbreviate karne deta hai. Note karein ke humein `self` shorthand ke aage `&` phir bhi use karna padta hai taa-ke indicate ho ke ye method `Self` instance ko borrow karti hai, bilkul usi tarah jaise hum ne `rectangle: &Rectangle` mein kiya tha. Methods `self` ki ownership le sakti hain, `self` ko immutably borrow kar sakti hain, jaisa ke hum ne yahan kiya hai, ya `self` ko mutably borrow kar sakti hain, bilkul kisi bhi doosre parameter ki tarah.

Hum ne yahan `&self` ko usi wajah se choose kiya jis wajah se function version mein `&Rectangle` use kiya tha: Hum ownership nahi lena chahte aur sirf struct ka data read karna chahte hain, use write nahi karna. Agar hum method ke kaam ke hissa ke taur par us instance ko change karna chahte hon jis par hum ne method call ki hai, to hum first parameter ke taur par `&mut self` use karenge. Sirf `self` ko first parameter ke taur par use karke instance ki ownership lene wala method rare hota hai; ye technique aam tor par tab use hoti hai jab method `self` ko kisi aur cheez mein transform karti hai aur aap caller ko transformation ke baad original instance use karne se rokna chahte hain.

Functions ke bajaye methods use karne ki main wajah, method syntax provide karne aur har method ki signature mein `self` ki type ko repeat na karne ke ilawa, organization hai. Hum ne woh tamam cheezen jo kisi type ke instance ke saath ki ja sakti hain, ek `impl` block mein rakh di hain, bajaye is ke ke hamare code ke future users ko hamari provide ki hui library mein `Rectangle` ki capabilities ko mukhtalif jagahon par search karna pade.

Note karein ke hum kisi method ko struct ke fields mein se kisi ek ke same name se bhi name de sakte hain. Misal ke taur par, hum `Rectangle` par ek method define kar sakte hain jiska name `width` bhi ho:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-06-method-field-interaction/src/main.rs:here}}
```

</Listing>

Yahan hum `width` method ko is tarah define kar rahe hain ke agar instance ke `width` field ki value `0` se greater ho to ye `true` return kare aur agar value `0` ho to `false` return kare: Hum kisi method ke andar same name wale field ko kisi bhi purpose ke liye use kar sakte hain. `main` mein, jab hum `rect1.width` ke baad parentheses lagate hain, to Rust samajhta hai ke hamara matlab `width` method hai. Jab hum parentheses use nahi karte, to Rust samajhta hai ke hamara matlab `width` field hai.

Aksar, lekin hamesha nahi, jab hum kisi method ko field ke same name se name dete hain to hum chahte hain ke woh sirf field ki value return kare aur koi doosra kaam na kare. Is tarah ke methods ko *getters* kaha jata hai, aur Rust inhein struct fields ke liye automatically implement nahi karta, jaisa ke kuch doosri languages karti hain. Getters useful hote hain kyun ke aap field ko private aur method ko public rakh sakte hain aur is tarah us field ko type ki public API ke hissa ke taur par read-only access provide kar sakte hain. Hum [Chapter 7][public]<!-- ignore --> mein public aur private kya hote hain aur kisi field ya method ko public ya private kaise designate karna hai, is par baat karenge.

> ### `->` Operator Kahan Hai?
>
> C aur C++ mein methods call karne ke liye do different operators use hote hain: Agar aap object par directly method call kar rahe hon to `.` use karte hain aur agar aap object ke pointer par method call kar rahe hon aur pehle pointer ko dereference karna zaroori ho to `->` use karte hain. Doosre alfaaz mein, agar `object` ek pointer hai, to `object->something()` kuch had tak `(*object).something()` ke jaisa hai.
>
> Rust mein `->` operator ka koi equivalent nahi hai; is ke bajaye, Rust mein *automatic referencing and dereferencing* naam ka ek feature hai. Methods ko call karna Rust mein un chand jagahon mein se ek hai jahan ye behavior hota hai.
>
> Ye is tarah kaam karta hai: Jab aap `object.something()` ke zariye method call karte hain, to Rust automatically `&`, `&mut`, ya `*` add karta hai taa-ke `object` method ki signature ke mutabiq ho. Doosre alfaaz mein, following dono same hain:
>
> <!-- CAN'T EXTRACT SEE BUG https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> Pehla wala kaafi clean lagta hai. Ye automatic referencing behavior is liye kaam karta hai kyun ke methods ke paas ek clear receiver hota hai—`self` ki type. Receiver aur method ke name ko dekh kar Rust definitively figure out kar sakta hai ke method read kar rahi hai (`&self`), mutate kar rahi hai (`&mut self`), ya consume kar rahi hai (`self`). Ye fact ke Rust method receivers ke liye borrowing ko implicit bana deta hai, ownership ko practical taur par ergonomic banane ka ek bara hissa hai.

### Methods with More Parameters

Aaiye `Rectangle` struct par ek second method implement karke methods ko use karne ki practice karte hain. Is baar hum chahte hain ke `Rectangle` ka ek instance kisi doosre `Rectangle` instance ko le aur agar doosra `Rectangle` poori tarah `self` (pehle `Rectangle`) ke andar fit ho sakta ho to `true` return kare; warna `false` return kare. Yani, ek baar hum `can_hold` method define kar lein, to hum Listing 5-14 mein dikhaya gaya program likhne ke qabil hona chahte hain.

<Listing number="5-14" file-name="src/main.rs" caption="Abhi tak na likhe gaye `can_hold` method ko use karna">

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-14/src/main.rs}}
```

</Listing>

Expected output following jaisi hogi kyun ke `rect2` ki dono dimensions `rect1` ki dimensions se chhoti hain, lekin `rect3`, `rect1` se zyada wide hai:

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

Humein maloom hai ke hum ek method define karna chahte hain, is liye ye `impl Rectangle` block ke andar hogi. Method ka name `can_hold` hoga, aur ye doosre `Rectangle` ka immutable borrow parameter ke taur par legi. Method ko call karne wale code ko dekh kar hum bata sakte hain ke parameter ki type kya hogi: `rect1.can_hold(&rect2)` mein `&rect2` pass kiya ja raha hai, jo `rect2`, yani `Rectangle` ke ek instance, ka immutable borrow hai. Ye sense banata hai kyun ke humein sirf `rect2` ko read karna hai (write nahi karna, warna humein mutable borrow ki zaroorat hoti), aur hum chahte hain ke `main`, `rect2` ki ownership retain kare taa-ke `can_hold` method ko call karne ke baad hum use dobara use kar saken. `can_hold` ki return value Boolean hogi, aur implementation ye check karegi ke `self` ki width aur height respectively doosre `Rectangle` ki width aur height se greater hain ya nahi. Aaiye Listing 5-13 ke `impl` block mein naya `can_hold` method add karte hain, jaisa ke Listing 5-15 mein dikhaya gaya hai.

<Listing number="5-15" file-name="src/main.rs" caption="`Rectangle` par `can_hold` method implement karna jo doosre `Rectangle` instance ko parameter ke taur par leta hai">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-15/src/main.rs:here}}
```

</Listing>

Jab hum is code ko Listing 5-14 ke `main` function ke saath run karenge, to humein desired output milegi. Methods `self` parameter ke baad signature mein multiple parameters le sakti hain, aur ye parameters bilkul functions ke parameters ki tarah kaam karte hain.

### Associated Functions

`impl` block ke andar define kiye gaye tamam functions ko *associated functions* kaha jata hai kyun ke ye `impl` ke baad diye gaye type ke saath associated hote hain. Hum aise associated functions bhi define kar sakte hain jin ka first parameter `self` nahi hota (aur is wajah se ye methods nahi hoti), kyun ke inhein kaam karne ke liye type ke kisi instance ki zaroorat nahi hoti. Hum pehle hi aisa ek function use kar chuke hain: `String` type par define kiya gaya `String::from` function.

Aise associated functions jo methods nahi hoti, aksar constructors ke liye use ki jati hain jo struct ka ek naya instance return karti hain. Inhein aksar `new` kaha jata hai, lekin `new` koi special name nahi hai aur na hi ye language mein built-in hai. Misal ke taur par, hum `square` naam ka ek associated function provide karne ka faisla kar sakte hain jo ek dimension parameter le aur usi ko width aur height dono ke taur par use kare, is tarah square `Rectangle` create karna aasaan ho jayega, bajaye is ke ke humein same value ko do baar specify karna pade:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-03-associated-functions/src/main.rs:here}}
```

Return type aur function ki body mein `Self` keywords us type ke aliases hain jo `impl` keyword ke baad mojood hai, jo is case mein `Rectangle` hai.

Is associated function ko call karne ke liye, hum struct name ke saath `::` syntax use karte hain; `let sq = Rectangle::square(3);` ek example hai. Ye function struct ke namespace mein hota hai: `::` syntax associated functions aur modules ki taraf se create kiye gaye namespaces dono ke liye use hoti hai. Hum modules ke baare mein [Chapter 7][modules]<!-- ignore --> mein baat karenge.

### Multiple `impl` Blocks

Har struct ke multiple `impl` blocks ho sakte hain. Misal ke taur par, Listing 5-15, Listing 5-16 mein dikhaye gaye code ke equivalent hai, jisme har method apne alag `impl` block mein hai.

<Listing number="5-16" caption="Multiple `impl` blocks ko use karke Listing 5-15 ko dobara likhna">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-16/src/main.rs:here}}
```

</Listing>

Yahan in methods ko multiple `impl` blocks mein separate karne ki koi wajah nahi hai, lekin ye valid syntax hai. Chapter 10 mein, jahan hum generic types aur traits ke baare mein discuss karenge, hum ek aisi situation dekhenge jahan multiple `impl` blocks useful hote hain.

## Summary

Structs aapko aisi custom types create karne deti hain jo aapke domain ke liye meaningful hoti hain. Structs ko use karke aap related data ke pieces ko ek doosre ke saath connected rakh sakte hain aur har piece ko name de sakte hain, jis se aapka code clear hota hai. `impl` blocks mein aap apni type ke saath associated functions define kar sakte hain, aur methods associated function ki ek qisam hoti hain jo aapko ye specify karne deti hain ke aapke structs ke instances ka behavior kya hoga.

Lekin structs hi custom types create karne ka eklauta tareeqa nahi hain: Aaiye Rust ke enum feature ki taraf chalte hain aur apne toolbox mein ek aur tool shamil karte hain.

[enums]: ch06-00-enums.html
[trait-objects]: ch18-02-trait-objects.md
[public]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html

