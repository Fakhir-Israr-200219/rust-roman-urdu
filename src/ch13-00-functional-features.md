# Functional Language Features: Iterators and Closures

Rust ki design ne bohat si existing languages aur techniques se inspiration li hai, aur ek significant influence *functional programming* hai. Functional style mein programming karne mein aksar functions ko values ke taur par use karna shamil hota hai: unhein arguments mein pass karna, doosre functions se return karna, baad mein execution ke liye unhein variables mein assign karna, aur isi tarah doosre tareeqe.

Is chapter mein hum is baat par debate nahi karenge ke functional programming kya hai ya kya nahi hai, balki Rust ke kuch aise features discuss karenge jo un features se milte julte hain jo aksar functional kehlane wali bohat si languages mein maujood hain.

More specifically, hum cover karenge:

* *Closures*, ek function-jaisa construct jise aap variable mein store kar sakte hain
* *Iterators*, elements ki ek series ko process karne ka ek tareeqa
* Chapter 12 ke I/O project ko improve karne ke liye closures aur iterators ko kaise use karein
* Closures aur iterators ki performance (spoiler alert: yeh shayad aapki expectation se zyada fast hain!)

Hum Rust ke kuch doosre features, jaise pattern matching aur enums, ko bhi pehle hi cover kar chuke hain jo functional style se influenced hain. Kyun ke closures aur iterators mein mastery fast, idiomatic Rust code likhne ka ek important hissa hai, hum poora chapter inhi ke liye dedicate kareng
