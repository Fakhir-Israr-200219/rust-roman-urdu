## Graceful Shutdown and Cleanup

Listing 21-20 ka code thread pool ke use ke zariye requests ko asynchronously
respond kar raha hai, jaisa ke hum chahte thay. Humein `workers`, `id`, aur
`thread` fields ke bare mein kuch warnings milti hain jinhein hum direct tareeqe
se use nahi kar rahe, jo humein yaad dilati hain ke hum abhi kuch bhi clean up
nahi kar rahe. Jab hum main thread ko rokne ke liye kam elegant <kbd>ctrl</kbd>-<kbd>C</kbd> method use karte hain, to baqi tamam threads bhi
foran stop ho jate hain, chahe woh kisi request ko serve karne ke darmiyan hi
kyun na hon.

Ab hum `Drop` trait ko implement karenge taake pool ke har thread par `join`
call kiya ja sake aur woh close hone se pehle jin requests par kaam kar rahe
hain unhein complete kar saken. Phir hum threads ko yeh batane ka tareeqa
implement karenge ke unhein naye requests accept karna band karke shut down
ho jana chahiye. Is code ko action mein dekhne ke liye, hum apne server ko
modify karenge taake woh gracefully apne thread pool ko shut down karne se
pehle sirf do requests accept kare.

Ek baat notice karni hai: Is mein se koi bhi cheez code ke un parts ko affect
nahi karti jo closures ko execute karne ka kaam handle karte hain, is liye agar
hum async runtime ke liye thread pool use kar rahe hote to yahan sab kuch same
hota.

### Implementing the `Drop` Trait on `ThreadPool`

Aao apne thread pool par `Drop` implement karne se shuru karte hain. Jab pool
drop hota hai, hamare tamam threads ko `join` karna chahiye taake yeh ensure ho
ke woh apna kaam complete kar len. Listing 21-22 `Drop` implementation ki ek
pehli koshish dikhati hai; yeh code abhi poori tarah kaam nahi karega.

<Listing number="21-22" file-name="src/lib.rs" caption="Thread pool ke scope se bahar jane par har thread ko join karna">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-22/src/lib.rs:here}}
```

</Listing>

Sab se pehle, hum thread pool ke har `worker` ke through loop karte hain. Hum
yahan `&mut` use karte hain kyun ke `self` ek mutable reference hai, aur humein
`worker` ko bhi mutate karne ke qabil hona zaroori hai. Har `worker` ke liye,
hum ek message print karte hain jo batata hai ke yeh particular `Worker`
instance shut down ho raha hai, aur phir hum us `Worker` instance ke thread par
`join` call karte hain. Agar `join` ki call fail ho jaye, to hum `unwrap` use
karke Rust ko panic karwate hain aur ek ungraceful shutdown mein chale jate
hain.

Yeh woh error hai jo humein is code ko compile karne par milta hai:

```console
{{#include ../listings/ch21-web-server/listing-21-22/output.txt}}
```

Error humein batata hai ke hum `join` call nahi kar sakte kyun ke hamare paas
har `worker` ka sirf mutable borrow hai aur `join` apne argument ki ownership
leta hai. Is issue ko solve karne ke liye, humein thread ko us `Worker`
instance se move karna hoga jo `thread` ka owner hai taake `join` thread ko
consume kar sake. Is ka ek tareeqa wohi approach lena hai jo humne Listing
18-15 mein li thi. Agar `Worker` ke paas
`Option<thread::JoinHandle<()>>` hota, to hum `Option` par `take` method call
karke value ko `Some` variant se bahar move kar sakte thay aur uski jagah
`None` variant chhor sakte thay. Doosre alfaaz mein, jo `Worker` running ho
uske `thread` mein `Some` variant hota, aur jab hum kisi `Worker` ko clean up
karna chahte, to hum `Some` ko `None` se replace kar dete taake `Worker` ke
paas run karne ke liye koi thread na rahe.

Lekin yeh sirf us waqt samne aata jab `Worker` ko drop kiya ja raha ho. Is ke
badle mein, humein har us jagah
`Option<thread::JoinHandle<()>>` se deal karna padta jahan hum
`worker.thread` access karte. Idiomatic Rust mein `Option` kaafi use hota hai,
lekin jab aap kisi aisi cheez ko workaround ke taur par `Option` mein wrap
karte hain jo aap jaante hain ke hamesha present hogi, to alternative approaches
talash karna acha idea hota hai taake aapka code cleaner aur kam error-prone
rahe.

Is case mein ek better alternative mojood hai: `Vec::drain` method. Yeh ek
range parameter accept karta hai taake specify kiya ja sake ke vector se kaun
se items remove karne hain, aur in items ka ek iterator return karta hai.
`..` range syntax pass karne se vector ki *har* value remove ho jayegi.

Is liye, humein `ThreadPool` ki `drop` implementation ko is tarah update karna
hoga:

<Listing file-name="src/lib.rs">

```rust
{{#rustdoc_include ../listings/ch21-web-server/no-listing-04-update-drop-definition/src/lib.rs:here}}
```

</Listing>

Yeh compiler error ko resolve karta hai aur hamare code mein kisi aur change ki
zaroorat nahi hoti. Note karein ke kyun ke drop panic ke waqt bhi call kiya ja
sakta hai, `unwrap` bhi panic kar sakta hai aur double panic ka sabab ban sakta
hai, jo foran program ko crash karta hai aur progress mein maujood tamam
cleanup ko khatam kar deta hai. Example program ke liye yeh theek hai, lekin
production code ke liye iski recommendation nahi ki jati.

### Signaling to the Threads to Stop Listening for Jobs

Humne jo tamam changes kiye hain, unke saath hamara code bina kisi warning ke
compile ho jata hai. Lekin buri khabar yeh hai ke yeh code abhi tak us tarah
function nahi karta jaisa hum chahte hain. Key `Worker` instances ke threads
ki chalne wali closures ke logic mein hai: Is waqt hum `join` call karte hain,
lekin is se threads shut down nahi honge, kyun ke woh jobs dhoondne ke liye
hamesha `loop` karte rehte hain. Agar hum apni current `drop` implementation
ke saath `ThreadPool` ko drop karne ki koshish karein, to main thread hamesha ke
liye block ho jayega aur pehle thread ke finish hone ka wait karega.

Is problem ko fix karne ke liye, humein `ThreadPool` ki `drop` implementation
mein ek change aur phir `Worker` loop mein ek change karna hoga.

Sab se pehle, hum `ThreadPool` ki `drop` implementation ko change karenge taake
threads ke finish hone ka wait karne se pehle explicitly `sender` ko drop kiya
ja sake. Listing 21-23 `ThreadPool` mein `sender` ko explicitly drop karne ke
liye ki jane wali changes dikhati hai. Thread ke muqable mein, yahan humein
waqai `Option` use karne ki zaroorat hai taake `Option::take` ke zariye
`sender` ko `ThreadPool` se move kiya ja sake.

<Listing number="21-23" file-name="src/lib.rs" caption="`Worker` threads ko join karne se pehle `sender` ko explicitly drop karna">

```rust,noplayground,not_desired_behavior id="k3j7pd"
{{#rustdoc_include ../listings/ch21-web-server/listing-21-23/src/lib.rs:here}}
```

</Listing>

`sender` ko drop karne se channel close ho jata hai, jo indicate karta hai ke
ab aur messages send nahi kiye jayenge. Jab aisa hota hai, to woh tamam
`recv` calls jo `Worker` instances infinite loop mein karte hain, ek error
return karengi. Listing 21-24 mein hum `Worker` loop ko change karte hain taake
us situation mein woh gracefully loop se bahar nikal jaye, jis ka matlab hai
ke jab `ThreadPool` ki `drop` implementation un par `join` call karegi to
threads finish ho jayenge.

<Listing number="21-24" file-name="src/lib.rs" caption="`recv` ke error return karne par loop se gracefully bahar nikalna">

```rust,noplayground id="v8q2lm"
{{#rustdoc_include ../listings/ch21-web-server/listing-21-24/src/lib.rs:here}}
```

</Listing>

Is code ko action mein dekhne ke liye, aao `main` ko modify karte hain taake
server ko gracefully shut down karne se pehle sirf do requests accept kare,
jaisa ke Listing 21-25 mein dikhaya gaya hai.

<Listing number="21-25" file-name="src/main.rs" caption="Loop se bahar nikal kar do requests serve karne ke baad server ko shut down karna">

```rust,ignore id="n4f8qs"
{{#rustdoc_include ../listings/ch21-web-server/listing-21-25/src/main.rs:here}}
```

</Listing>

Aap nahi chahenge ke real-world web server sirf do requests serve karne ke
baad shut down ho jaye. Yeh code sirf yeh demonstrate karta hai ke graceful
shutdown aur cleanup working order mein hain.

`take` method `Iterator` trait mein defined hai aur iteration ko zyada se zyada
pehle do items tak limit karta hai. `main` ke end par `ThreadPool` scope se bahar
chala jayega, aur `drop` implementation run hogi.

Server ko `cargo run` ke saath start karein aur teen requests karein. Teesri
request ko error hona chahiye, aur apne terminal mein aapko is ke similar output
nazar aana chahiye:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-25
cargo run
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
third request will error because server will have shut down
copy output below
Can't automate because the output depends on making requests
-->

```console id="p0a6nc"
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

Aapko `Worker` IDs aur print hone wale messages ka order different nazar aa
sakta hai. Hum messages se dekh sakte hain ke yeh code kaise kaam karta hai:
`Worker` instances 0 aur 3 ko pehli do requests mili. Server ne doosri
connection ke baad connections accept karna band kar diya, aur `ThreadPool` ki
`Drop` implementation `Worker 3` ke apna job start karne se bhi pehle execute
hona shuru ho jati hai. `sender` ko drop karne se tamam `Worker` instances
disconnect ho jate hain aur unhein shut down hone ka signal mil jata hai.
`Worker` instances disconnect hone par har ek ek message print karta hai, aur
phir thread pool har `Worker` thread ke finish hone ka wait karne ke liye
`join` call karta hai.

Is particular execution ka ek interesting aspect notice karein: `ThreadPool` ne
`sender` ko drop kiya, aur kisi bhi `Worker` ko error receive hone se pehle humne
`Worker 0` ko join karne ki koshish ki. `Worker 0` ko abhi tak `recv` se error
nahi mila tha, is liye main thread block ho gaya aur `Worker 0` ke finish hone ka
wait karne laga. Isi dauran, `Worker 3` ko ek job mili aur phir tamam threads ko
error receive hua. Jab `Worker 0` finish hua, to main thread ne baqi `Worker`
instances ke finish hone ka wait kiya. Us waqt woh sab apne loops se bahar
nikal chuke thay aur stop ho gaye thay.

Congrats! Ab humne apna project complete kar liya hai; hamare paas ek basic web
server hai jo asynchronously respond karne ke liye thread pool use karta hai.
Hum server ka graceful shutdown perform kar sakte hain, jo pool ke tamam
threads ko clean up karta hai.

Reference ke liye yahan complete code hai:

<Listing file-name="src/main.rs">

```rust,ignore id="z6q4mt"
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/main.rs}}
```

</Listing>

<Listing file-name="src/lib.rs">

```rust,noplayground id="r2x8vf"
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/lib.rs}}
```

</Listing>

Hum yahan aur bhi kaam kar sakte hain! Agar aap is project ko enhance karna
continue karna chahte hain, to yahan kuch ideas hain:

* `ThreadPool` aur uske public methods ke liye mazeed documentation add karein.
* Library ki functionality ke tests add karein.
* `unwrap` calls ko zyada robust error handling mein change karein.
* `ThreadPool` ko web requests serve karne ke ilawa kisi aur task ke liye use
  karein.
* [crates.io](https://crates.io/) par ek thread pool crate dhoondein aur us
  crate ko use karke ek similar web server implement karein. Phir uski API aur
  robustness ka hamare implement kiye hue thread pool ke saath comparison
  karein.

## Summary

Shabash! Aap book ke end tak pohanch gaye hain! Hum aapka shukriya ada karna
chahte hain ke aapne Rust ke is safar mein hamara saath diya. Ab aap apne Rust
projects implement karne aur doosre logon ke projects mein help karne ke liye
tayyar hain. Yeh baat zehan mein rakhein ke doosre Rustaceans ki ek welcoming
community mojood hai jo aapke Rust ke safar mein aane wale kisi bhi challenge
mein aapki madad karna chahegi.

