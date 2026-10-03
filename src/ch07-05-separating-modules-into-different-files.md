## Modules Ko Different Files Mein Separate Karna

Ab tak is chapter ki tamam examples mein multiple modules ko ek hi file mein define kiya gaya hai. Jab modules bade ho jate hain, to aap unki definitions ko ek separate file mein move karna chahenge taa-ke code ko navigate karna aasaan ho jaye.

Misal ke taur par, aaiye Listing 7-17 ke code se shuru karte hain jisme multiple restaurant modules the. Hum tamam modules ko crate root file mein define karne ke bajaye unhein files mein extract karenge. Is case mein crate root file *src/lib.rs* hai, lekin ye procedure un binary crates ke saath bhi kaam karta hai jinki crate root file *src/main.rs* hoti hai.

Sab se pehle, hum `front_of_house` module ko apni separate file mein extract karenge. `front_of_house` module ke curly brackets ke andar ka code remove kar dein aur sirf `mod front_of_house;` declaration rehne dein, taa-ke *src/lib.rs* mein woh code ho jo Listing 7-21 mein dikhaya gaya hai. Note karein ke ye tab tak compile nahi hoga jab tak hum Listing 7-22 mein dikhayi gayi *src/front_of_house.rs* file create nahi kar lete.

<Listing number="7-21" file-name="src/lib.rs" caption="`front_of_house` module declare karna jiska body *src/front_of_house.rs* mein hoga">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/lib.rs}}
```

</Listing>

Ab woh code jo curly brackets ke andar tha, *src/front_of_house.rs* naam ki ek nayi file mein rakhein, jaisa ke Listing 7-22 mein dikhaya gaya hai. Compiler is file mein code dhoondhna jaanta hai kyun ke crate root mein usay `front_of_house` name wali module declaration mili thi.

<Listing number="7-22" file-name="src/front_of_house.rs" caption="`front_of_house` module ke andar ki definitions *src/front_of_house.rs* mein">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/front_of_house.rs}}
```

</Listing>

Note karein ke apne module tree mein kisi file ko `mod` declaration ke zariye *sirf ek baar* load karna hota hai. Jab compiler ko pata chal jaye ke file project ka hissa hai (aur ye bhi pata chal jaye ke code module tree mein kahan mojood hai, kyun ke aap ne `mod` statement kahan rakha hai), to aapke project ki doosri files ko loaded file ke code ko refer karne ke liye us jagah ka path use karna chahiye jahan woh declare hui hai, jaisa ke [“Paths for Referring to an Item in the Module Tree”][paths]<!-- ignore --> section mein cover kiya gaya hai. Doosre alfaaz mein, `mod` koi “include” operation *nahi* hai jo aap ne doosri programming languages mein dekha ho.

Ab hum `hosting` module ko apni separate file mein extract karenge. Ye process thora different hai kyun ke `hosting`, root module ka nahi balki `front_of_house` ka child module hai. Hum `hosting` ki file ko ek nayi directory mein rakhenge jo module tree mein uske ancestors ke name par rakhi jayegi; is case mein *src/front_of_house*.

`hosting` ko move karna shuru karne ke liye, hum *src/front_of_house.rs* ko change karke us mein sirf `hosting` module ki declaration rakhenge:

<Listing file-name="src/front_of_house.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house.rs}}
```

</Listing>

Phir, hum ek *src/front_of_house* directory aur ek *hosting.rs* file create karte hain jisme `hosting` module mein ki gayi definitions hongi:

<Listing file-name="src/front_of_house/hosting.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house/hosting.rs}}
```

</Listing>

Agar is ke bajaye hum *hosting.rs* ko *src* directory mein rakh dein, to compiler ye expect karega ke *hosting.rs* ka code crate root mein declare kiye gaye ek `hosting` module mein ho, na ke `front_of_house` module ke child ke taur par declare kiye gaye module mein. Compiler ke ye rules ke kaun si files mein kaun se modules ka code check karna hai, is wajah se directories aur files module tree ke saath zyada closely match karti hain.

> ### Alternate File Paths
>
> Ab tak hum ne sab se idiomatic file paths cover kiye hain jinhein Rust compiler use karta hai, lekin Rust file paths ke ek purane style ko bhi support karta hai. Crate root mein `front_of_house` naam ke module ke liye, compiler module ka code in jagahon par dhoondhega:
>
> * *src/front_of_house.rs* (jise hum ne cover kiya)
> * *src/front_of_house/mod.rs* (purana style, ab bhi supported path)
>
> `front_of_house` ke submodule `hosting` ke liye, compiler module ka code in jagahon par dhoondhega:
>
> * *src/front_of_house/hosting.rs* (jise hum ne cover kiya)
> * *src/front_of_house/hosting/mod.rs* (purana style, ab bhi supported path)
>
> Agar aap same module ke liye dono styles use karein, to aapko compiler error milega. Ek hi project mein different modules ke liye dono styles ko mix karna allowed hai, lekin aapke project ko navigate karne wale logon ke liye ye confusing ho sakta hai.
>
> *mod.rs* naam wali files ko use karne wale style ka main downside ye hai ke aapke project mein bohat si files ka name *mod.rs* ho sakta hai. Jab aap unhein apne editor mein ek hi waqt mein open karte hain, to ye confusing ho sakta hai.

Hum ne har module ke code ko ek separate file mein move kar diya hai, aur module tree wahi hai. `eat_at_restaurant` mein function calls bina kisi modification ke kaam karengi, halaan ke definitions different files mein mojood hain. Ye technique aapko modules ka size barhne par unhein new files mein move karne deti hai.

Ye note karein ke *src/lib.rs* mein `pub use crate::front_of_house::hosting` statement bhi change nahi hua, aur `use` ka is baat par koi asar nahi hota ke crate ke hissa ke taur par kaun si files compile hoti hain. `mod` keyword modules ko declare karta hai, aur Rust us module mein jane wale code ke liye module ke same name wali file mein dekhta hai.

## Summary

Rust aapko ek package ko multiple crates mein aur ek crate ko modules mein split karne deta hai, taa-ke aap ek module mein defined items ko doosre module se refer kar saken. Aap ye absolute ya relative paths specify karke kar sakte hain. In paths ko `use` statement ke saath scope mein laya ja sakta hai, taa-ke us scope mein item ko multiple baar use karte waqt aap shorter path use kar saken. Module code by default private hota hai, lekin aap `pub` keyword add karke definitions ko public bana sakte hain.

Agley chapter mein, hum standard library mein mojood kuch collection data structures dekhenge jinhein aap apne neatly organized code mein use kar sakte hain.

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html

