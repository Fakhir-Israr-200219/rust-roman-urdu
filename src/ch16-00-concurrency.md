# Fearless Concurrency

Concurrent programming ko safely aur efficiently handle karna Rust ke major goals mein se
ek aur hai. *Concurrent programming*, jismein program ke different parts independently
execute hote hain, aur *parallel programming*, jismein program ke different parts
ek hi waqt mein execute hote hain, increasingly important hoti ja rahi hain kyun ke zyada
computers apne multiple processors ka faida utha rahe hain. Historically, in contexts mein
programming karna mushkil aur errors ka sabab raha hai. Rust is situation ko badalna chahta hai.

Shuru mein, Rust team ka khayal tha ke memory safety ensure karna aur concurrency problems
ko prevent karna do separate challenges hain jinhein different methods se solve karna hoga.
Waqt ke saath, team ne discover kiya ke ownership aur type systems memory safety *aur*
concurrency problems ko manage karne ke liye powerful set of tools hain! Ownership aur type
checking ka faida uthate hue, Rust mein bohot si concurrency errors runtime errors ke bajaye
compile-time errors hoti hain. Is liye, runtime concurrency bug jis exact situation mein
occur hota hai use reproduce karne ki koshish mein bohot waqt lagane ke bajaye, incorrect code
compile hone se inkaar kar dega aur ek error present karega jo problem explain karega. Is ke
result mein, aap apne code ko us waqt fix kar sakte hain jab aap us par kaam kar rahe hon,
bajaye is ke ke mumkin hai production mein ship hone ke baad fix karna pade. Humne Rust ke
is aspect ko *fearless concurrency* ka nickname diya hai. Fearless concurrency aapko aisa
code likhne deti hai jo subtle bugs se free ho aur jise naye bugs introduce kiye baghair
refactor karna easy ho.

> Note: Simplicity ke liye, hum bohot se problems ko zyada precise tareeqe se
> *concurrent* kehne ke bajaye *concurrent* kahenge, jab ke asal mein hum
> *concurrent and/or parallel* keh sakte hain. Is chapter ke liye, jab bhi hum
> *concurrent* use karein to zehni taur par *concurrent and/or parallel* samajh lein.
> Agle chapter mein, jahan yeh distinction zyada important hogi, hum zyada specific honge.

Bohot si languages concurrent problems handle karne ke liye jo solutions offer karti hain
un ke baare mein dogmatic hoti hain. Misal ke taur par, Erlang ke paas message-passing
concurrency ke liye elegant functionality hai lekin threads ke darmiyan state share karne
ke sirf obscure tareeqe hain. Possible solutions mein se sirf ek subset ko support karna
higher-level languages ke liye ek reasonable strategy hai kyun ke higher-level language
kuch control chhor kar abstractions hasil karne ke benefits ka promise karti hai. Lekin
lower-level languages se expect kiya jata hai ke woh kisi bhi given situation mein best
performance wali solution provide karein aur hardware ke upar kam abstractions rakhein.
Is liye, Rust problems ko model karne ke liye variety of tools offer karta hai taake aapki
situation aur requirements ke mutabiq jo tareeqa appropriate ho use kiya ja sake.

Yeh woh topics hain jinhein hum is chapter mein cover karenge:

* Multiple pieces of code ko ek hi waqt mein run karne ke liye threads kaise create karein
* *Message-passing* concurrency, jahan channels threads ke darmiyan messages send karte hain
* *Shared-state* concurrency, jahan multiple threads ko data ke kisi piece tak access hota hai
* `Sync` aur `Send` traits, jo Rust ki concurrency guarantees ko user-defined types ke saath
  saath standard library ki provide ki gayi types tak bhi extend karte hain
