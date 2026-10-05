# Patterns and Matching

Patterns Rust mein ek special syntax hain jo types ke structure, chahe woh complex hon ya simple, ke against matching ke liye use hoti hain. Patterns ko `match` expressions aur doosre constructs ke saath use karne se aap program ke control flow par zyada control hasil karte hain. Ek pattern mein neeche diye gaye elements mein se kuch ka combination hota hai:

* Literals
* Destructured arrays, enums, structs, ya tuples
* Variables
* Wildcards
* Placeholders

Kuch example patterns `x`, `(a, 3)`, aur `Some(Color::Red)` hain. Jin contexts mein patterns valid hoti hain, wahan ye components data ki shape ko describe karte hain. Phir hamara program values ko patterns ke against match karta hai taake determine kiya ja sake ke particular piece of code ko continue run karne ke liye data ki shape correct hai ya nahi.

Pattern ko use karne ke liye, hum usay kisi value ke saath compare karte hain. Agar pattern value se match karti hai, to hum value ke parts ko apne code mein use karte hain. Chapter 6 mein `match` expressions ko yaad karein jo patterns use karti thin, jaise coin-sorting machine ka example. Agar value pattern ki shape mein fit hoti hai, to hum named pieces ko use kar sakte hain. Agar fit nahi hoti, to pattern se associated code run nahi hoga.

Yeh chapter patterns se related tamam cheezon ke liye ek reference hai. Hum patterns ko use karne ki valid places, refutable aur irrefutable patterns ke darmiyan difference, aur mukhtalif kinds of pattern syntax ko cover karenge jo aap dekh sakte hain. Chapter ke end tak, aap patterns ko bohat se concepts ko clear way mein express karne ke liye use karna jaante honge.
