## Appendix E: Editions

Chapter 1 mein aapne dekha tha ke `cargo new` aapki *Cargo.toml* file mein
edition ke bare mein thora sa metadata add karta hai. Yeh appendix batati hai
ke is ka kya matlab hai!

Rust language aur compiler ka six-week release cycle hai, jis ka matlab hai ke
users ko new features ka ek constant stream milta rehta hai. Doosri
programming languages kam frequently badi changes release karti hain; Rust
chhoti updates zyada frequently release karta hai. Kuch arsay baad, yeh tamam
chhoti changes mil kar kaafi badi ho jati hain. Lekin ek release se doosri
release tak peeche mur kar yeh kehna mushkil ho sakta hai, “Wow, Rust 1.10 aur
Rust 1.31 ke darmiyan Rust mein bohat kuch change ho gaya hai!”

Har taqreeban teen saal baad, Rust team ek nayi Rust *edition* produce karti
hai. Har edition un features ko jo land ho chuke hain ek clear package mein
jama karti hai, jiske saath fully updated documentation aur tooling hoti hai.
New editions usual six-week release process ke part ke taur par ship hoti hain.

Editions different logon ke liye different purposes serve karti hain:

* Active Rust users ke liye, ek new edition incremental changes ko ek
  easy-to-understand package mein jama karti hai.
* Non-users ke liye, ek new edition signal deti hai ke kuch major advancements
  land ho chuki hain, jo Rust ko dobara dekhne ke laayak bana sakti hain.
* Rust develop karne walon ke liye, ek new edition poore project ke liye ek
  rallying point provide karti hai.

Is waqt jab yeh likhi ja rahi hai, chaar Rust editions available hain: Rust
2015, Rust 2018, Rust 2021, aur Rust 2024. Yeh book Rust 2024 edition idioms
ko use karke likhi gayi hai.

*Cargo.toml* mein `edition` key indicate karti hai ke compiler ko aapke code
ke liye kaunsi edition use karni chahiye. Agar key exist nahi karti, to Rust
backward compatibility reasons ki wajah se `2015` ko edition value ke taur par
use karta hai.

Har project default 2015 edition ke ilawa kisi doosri edition ko opt in kar
sakta hai. Editions mein incompatible changes ho sakti hain, jaise ek naya
keyword include karna jo code mein mojood identifiers ke saath conflict karta
ho. Lekin jab tak aap un changes ko opt in nahi karte, aapka code compile hota
rahega, hatta ke jab aap apne use kiye jane wale Rust compiler version ko
upgrade kar lein.

Rust compiler ke tamam versions kisi bhi aisi edition ko support karte hain
jo us compiler ki release se pehle exist karti thi, aur woh kisi bhi supported
editions ke crates ko aapas mein link kar sakte hain. Edition changes sirf is
baat ko affect karti hain ke compiler shuru mein code ko kis tarah parse karta
hai. Is liye, agar aap Rust 2015 use kar rahe hain aur aapki dependencies mein
se ek Rust 2018 use karti hai, to aapka project compile hoga aur us dependency
ko use kar sakega. Is ka ulta situation bhi, jahan aapka project Rust 2018 use
karta hai aur dependency Rust 2015 use karti hai, isi tarah kaam karti hai.

Wazeh taur par: Zyada tar features tamam editions par available honge. Kisi bhi
Rust edition ko use karne wale developers ko new stable releases ke saath
improvements milti rahengi. Lekin kuch cases mein, mainly jab new keywords add
kiye jate hain, kuch new features sirf later editions mein available ho sakte
hain. Agar aap aise features se faida uthana chahte hain to aapko editions
switch karni hongi.

Mazeed details ke liye [*The Rust Edition Guide*][edition-guide] dekhein. Yeh
ek complete book hai jo editions ke darmiyan differences ko enumerate karti
hai aur explain karti hai ke `cargo fix` ke zariye aap apne code ko automatically
new edition par upgrade kaise kar sakte hain.

[edition-guide]: https://doc.rust-lang.org/stable/edition-guide
