<!-- Old headings. Do not remove or links may break. -->

<a id="using-trait-objects-that-allow-for-values-of-different-types"></a>

## Using Trait Objects to Abstract over Shared Behavior

Chapter 8 mein humne mention kiya tha ke vectors ki ek limitation yeh hai ke woh sirf ek hi type ke elements store kar sakte hain. Humne Listing 8-9 mein ek workaround create kiya tha jahan humne ek `SpreadsheetCell` enum define kiya tha jis mein integers, floats, aur text ko hold karne ke liye variants thay. Is ka matlab tha ke hum har cell mein different types ka data store kar sakte thay aur phir bhi hamare paas cells ki ek row ko represent karne wala vector hota. Jab hamare interchangeable items fixed types ka set hon jinhein hum code compile hone ke waqt jaante hon, to yeh bilkul theek solution hai.

Lekin kabhi kabhi hum chahte hain ke hamari library ka user un types ke set ko extend kar sake jo kisi particular situation mein valid hain. Yeh dikhane ke liye ke hum yeh kaise achieve kar sakte hain, hum ek example graphical user interface (GUI) tool create karenge jo items ki ek list ke through iterate karta hai, aur har item par `draw` method call karke use screen par draw karta hai—aam tor par GUI tools mein use hone wali technique. Hum `gui` naam ka ek library crate create karenge jo GUI library ka structure contain karega. Is crate mein logon ke use ke liye kuch types shamil ho sakti hain, jaise `Button` ya `TextField`. Is ke ilawa, `gui` users apne khud ke aise types create karna chahenge jinhein draw kiya ja sake: Misal ke taur par, ek programmer `Image` add kar sakta hai, aur doosra `SelectBox` add kar sakta hai.

Library likhte waqt, hum tamam un types ko nahi jaan sakte aur define nahi kar sakte jo doosre programmers create karna chahenge. Lekin hum yeh jaante hain ke `gui` ko different types ki bohot si values ka record rakhna hoga, aur in differently typed values mein se har ek par `draw` method call karna hoga. Isay yeh jaanne ki zaroorat nahi hai ke jab hum `draw` method call karenge to exactly kya hoga, bas itna pata hona chahiye ke value par woh method available hoga jise hum call kar sakte hain.

Inheritance wali language mein aisa karne ke liye, hum `Component` naam ki ek class define kar sakte thay jis par `draw` naam ka method hota. Doosri classes, jaise `Button`, `Image`, aur `SelectBox`, `Component` se inherit kartin aur is tarah `draw` method bhi inherit kar letin. Woh har ek `draw` method ko override karke apna custom behavior define kar sakti thin, lekin framework tamam types ko `Component` instances ke taur par treat kar sakta tha aur un par `draw` call kar sakta tha. Lekin kyun ke Rust mein inheritance nahi hai, humein `gui` library ko structure karne ke liye koi doosra tareeqa chahiye jo users ko library ke saath compatible naye types create karne ki ijazat de.

### Defining a Trait for Common Behavior

`gui` mein jo behavior hum chahte hain usay implement karne ke liye, hum `Draw` naam ka ek trait define karenge jis mein `draw` naam ka ek method hoga. Phir, hum ek aisa vector define kar sakte hain jo ek trait object leta hai. Ek *trait object* ek taraf us type ke instance ki taraf point karta hai jo hamare specified trait ko implement karta hai aur doosri taraf ek aisi table ki taraf jo runtime par us type ke trait methods ko lookup karne ke liye use hoti hai. Hum trait object ko kisi qisam ka pointer, jaise reference ya `Box<T>` smart pointer, phir `dyn` keyword, aur us ke baad relevant trait specify karke create karte hain. (Hum is baat ki wajah ke trait objects ko pointer use karna kyun zaroori hai, [“Dynamically Sized Types and the `Sized` Trait”][dynamically-sized]<!-- ignore --> mein Chapter 20 mein discuss karenge.) Hum generic ya concrete type ki jagah trait objects ko use kar sakte hain. Jahan bhi hum trait object use karte hain, Rust ka type system compile time par ensure karega ke us context mein use hone wali koi bhi value trait object ke trait ko implement karti ho. Natijatan, humein compile time par tamam possible types ko jaanne ki zaroorat nahi hoti.

Humne mention kiya hai ke Rust mein hum structs aur enums ko “objects” kehne se parhez karte hain taake unhein doosri languages ke objects se distinguish kiya ja sake. Ek struct ya enum mein, struct fields ka data aur `impl` blocks mein behavior alag alag hote hain, jabke doosri languages mein data aur behavior ko mila kar ek concept banaya jata hai jise aksar object kaha jata hai. Trait objects doosri languages ke objects se is liye different hain ke hum trait object mein data add nahi kar sakte. Trait objects doosri languages ke objects ki tarah generally useful nahi hote: Un ka specific purpose common behavior ke across abstraction allow karna hai.

Listing 18-3 dikhati hai ke ek `Draw` trait ko ek `draw` method ke saath kis tarah define kiya jata hai.

<Listing number="18-3" file-name="src/lib.rs" caption="Definition of the `Draw` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-03/src/lib.rs}}
```

</Listing>

Yeh syntax Chapter 10 mein traits define karne ke hawale se hamari discussions se familiar lagna chahiye. Is ke baad kuch naya syntax aata hai: Listing 18-4 `Screen` naam ka ek struct define karti hai jo `components` naam ka ek vector hold karta hai. Yeh vector `Box<dyn Draw>` type ka hai, jo ek trait object hai; yeh `Box` ke andar kisi bhi aise type ke liye stand-in hai jo `Draw` trait ko implement karta hai.

<Listing number="18-4" file-name="src/lib.rs" caption="Definition of the `Screen` struct with a `components` field holding a vector of trait objects that implement the `Draw` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-04/src/lib.rs:here}}
```

</Listing>

`Screen` struct par, hum `run` naam ka ek method define karenge jo apne har `components` par `draw` method call karega, jaisa ke Listing 18-5 mein dikhaya gaya hai.

<Listing number="18-5" file-name="src/lib.rs" caption="A `run` method on `Screen` that calls the `draw` method on each component">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-05/src/lib.rs:here}}
```

</Listing>

Yeh us struct ko define karne se different tareeqe se kaam karta hai jo trait bounds ke saath ek generic type parameter use karta hai. Generic type parameter ko ek waqt mein sirf ek concrete type se substitute kiya ja sakta hai, jabke trait objects runtime par multiple concrete types ko trait object ke liye fill in karne ki ijazat dete hain. Misal ke taur par, hum `Screen` struct ko generic type aur trait bound ke saath define kar sakte thay, jaisa ke Listing 18-6 mein hai.

<Listing number="18-6" file-name="src/lib.rs" caption="An alternate implementation of the `Screen` struct and its `run` method using generics and trait bounds">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-06/src/lib.rs:here}}
```

</Listing>

Yeh humein ek aise `Screen` instance tak restrict karta hai jis mein tamam components ki list ya to `Button` type ki ho ya tamam `TextField` type ki. Agar aapke paas hamesha homogeneous collections hi hon gi, to generics aur trait bounds use karna preferable hai kyun ke definitions ko compile time par concrete types ko use karne ke liye monomorphized kiya jayega.

Doosri taraf, trait objects use karne wale method ke saath, ek `Screen` instance aisa `Vec<T>` hold kar sakta hai jis mein `Box<Button>` ke saath `Box<TextField>` bhi ho. Aaiye dekhein ke yeh kis tarah kaam karta hai, aur phir hum runtime performance par is ke implications par baat karenge.

### Implementing the Trait

Ab hum kuch aise types add karenge jo `Draw` trait ko implement karte hain. Hum `Button` type provide karenge. Dobara, asal mein GUI library implement karna is book ke scope se bahar hai, is liye `draw` method ke body mein koi useful implementation nahi hogi. Yeh imagine karne ke liye ke implementation kaisi nazar aa sakti hai, ek `Button` struct mein `width`, `height`, aur `label` ke liye fields ho sakti hain, jaisa ke Listing 18-7 mein dikhaya gaya hai.

<Listing number="18-7" file-name="src/lib.rs" caption="A `Button` struct that implements the `Draw` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-07/src/lib.rs:here}}
```

</Listing>

`Button` par `width`, `height`, aur `label` fields doosre components ki fields se different hongi; misal ke taur par, ek `TextField` type mein wohi fields aur ek `placeholder` field bhi ho sakti hai. Har woh type jise hum screen par draw karna chahte hain `Draw` trait ko implement karega, lekin particular type ko kis tarah draw karna hai yeh define karne ke liye `draw` method mein different code use karega, jaisa ke yahan `Button` mein hai (jaisa ke mention kiya gaya hai, actual GUI code ke baghair). Misal ke taur par, `Button` type mein ek additional `impl` block ho sakta hai jis mein user ke button par click karne par hone wale behavior se related methods hon. Is qisam ke methods `TextField` jaise types par apply nahi honge.

Agar hamari library use karne wala koi shakhs `SelectBox` struct implement karne ka faisla karta hai jis mein `width`, `height`, aur `options` fields hon, to woh `SelectBox` type par bhi `Draw` trait implement karega, jaisa ke Listing 18-8 mein dikhaya gaya hai.

<Listing number="18-8" file-name="src/main.rs" caption="Another crate using `gui` and implementing the `Draw` trait on a `SelectBox` struct">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-08/src/main.rs:here}}
```

</Listing>

Hamari library ka user ab apna `main` function likh sakta hai taake ek `Screen` instance create kare. `Screen` instance mein woh ek `SelectBox` aur ek `Button` add kar sakta hai, har ek ko `Box<T>` mein rakh kar trait object bana sakta hai. Phir woh `Screen` instance par `run` method call kar sakta hai, jo har component par `draw` call karega. Listing 18-9 is implementation ko dikhati hai.

<Listing number="18-9" file-name="src/main.rs" caption="Using trait objects to store values of different types that implement the same trait">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-09/src/main.rs:here}}
```

</Listing>

Jab humne library likhi thi, humein yeh nahi pata tha ke koi `SelectBox` type add kar sakta hai, lekin hamari `Screen` implementation naye type ke saath bhi operate kar saki aur use draw kar saki kyun ke `SelectBox` `Draw` trait ko implement karta hai, jis ka matlab hai ke woh `draw` method ko implement karta hai.

Yeh concept—ke kisi value ki concrete type ke bajaye sirf is baat se concerned hona ke woh kin messages ka response deti hai—dynamically typed languages mein *duck typing* ke concept se milta julta hai: Agar woh duck ki tarah chalta hai aur duck ki tarah awaaz nikalta hai, to woh duck hi hona chahiye! Listing 18-5 mein `Screen` ke `run` ki implementation mein, `run` ko yeh jaanne ki zaroorat nahi ke har component ki concrete type kya hai. Yeh check nahi karta ke component `Button` ya `SelectBox` ka instance hai; yeh bas component par `draw` method call karta hai. `components` vector mein values ki type ke taur par `Box<dyn Draw>` specify karke, humne `Screen` ko is tarah define kiya hai ke use aisi values chahiye jin par hum `draw` method call kar sakte hain.

Trait objects aur Rust ke type system ko use karke duck typing ke similar code likhne ka faida yeh hai ke humein runtime par kabhi check nahi karna padta ke koi value particular method ko implement karti hai ya nahi, aur na hi is baat ki fikr karni padti hai ke agar koi value method implement na karti ho aur hum phir bhi usay call karein to errors milenge. Agar values un traits ko implement nahi kartin jin ki trait objects ko zaroorat hai, to Rust hamare code ko compile nahi karega.

Misal ke taur par, Listing 18-10 dikhati hai ke agar hum `String` ko component ke taur par use karke ek `Screen` create karne ki koshish karein to kya hota hai.

<Listing number="18-10" file-name="src/main.rs" caption="Attempting to use a type that doesn’t implement the trait object’s trait">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-10/src/main.rs}}
```

</Listing>

Humein yeh error milega kyun ke `String` `Draw` trait ko implement nahi karta:

```console
{{#include ../listings/ch18-oop/listing-18-10/output.txt}}
```

Yeh error humein batata hai ke ya to hum `Screen` ko koi aisi cheez pass kar rahe hain jo hum pass nahi karna chahte thay aur is liye humein koi different type pass karni chahiye, ya phir humein `String` par `Draw` implement karna chahiye taake `Screen` us par `draw` call kar sake.

<!-- Old headings. Do not remove or links may break. -->

<a id="trait-objects-perform-dynamic-dispatch"></a>

### Performing Dynamic Dispatch

Chapter 10 mein [“Performance of Code Using
Generics”][performance-of-code-using-generics]<!-- ignore --> mein generics par
compiler ke zariye perform kiye jane wale monomorphization process ke hawale se
hamari discussion yaad karein: Compiler har concrete type ke liye functions aur
methods ki nongeneric implementations generate karta hai jise hum generic type
parameter ki jagah use karte hain. Monomorphization se resulting code *static
dispatch* kar raha hota hai, jo us waqt hota hai jab compiler compile time par
jaanta hai ke aap kaunsa method call kar rahe hain. Yeh *dynamic dispatch* ke
baraks hai, jahan compiler compile time par yeh nahi bata sakta ke aap kaunsa
method call kar rahe hain. Dynamic dispatch ke cases mein compiler aisa code
emit karta hai jo runtime par yeh jaanta hoga ke kaunsa method call karna hai.

Jab hum trait objects use karte hain, Rust ko dynamic dispatch use karna padta
hai. Compiler un tamam types ko nahi jaanta jo trait objects ko use karne wale
code ke saath use kiye ja sakte hain, is liye use yeh nahi pata hota ke kis type
par implement kiye gaye kaun se method ko call karna hai. Is ke bajaye, runtime
par Rust trait object ke andar maujood pointers ko use karta hai taake yeh pata
chal sake ke kaunsa method call karna hai. Yeh lookup ek runtime cost incur
karta hai jo static dispatch ke saath nahi hoti. Dynamic dispatch compiler ko
method ke code ko inline karne ka option bhi nahi deta, jo baaz optimizations ko
bhi prevent karta hai, aur Rust ke paas kuch rules hain ke aap dynamic dispatch
ko kahan use kar sakte hain aur kahan nahi, jinhein *dyn compatibility* kaha
jata hai. Yeh rules is discussion ke scope se bahar hain, lekin aap
[in the reference][dyn-compatibility]<!-- ignore --> mein in ke bare mein mazeed
parh sakte hain. Lekin humein Listing 18-5 mein likhe gaye code mein extra
flexibility mili aur hum Listing 18-9 mein is flexibility ko support karne mein
able hue, is liye yeh ek trade-off hai jis par ghour karna chahiye.

[performance-of-code-using-generics]: ch10-01-syntax.html#performance-of-code-using-generics
[dynamically-sized]: ch20-03-advanced-types.html#dynamically-sized-types-and-the-sized-trait
[dyn-compatibility]: https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility
