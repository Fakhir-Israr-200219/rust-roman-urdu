<!-- Old headings. Do not remove or links may break. -->

<a id="using-message-passing-to-transfer-data-between-threads"></a>

## Transfer Data Between Threads with Message Passing

Safe concurrency ko ensure karne ke liye ek increasingly popular approach message
passing hai, jahan threads ya actors ek doosre ko data-containing messages bhej kar
communicate karte hain. Yahan idea [the Go language documentation](https://golang.org/doc/effective_go.html#concurrency) ke ek slogan mein diya gaya hai:
“Memory share karke communicate na karein; is ke bajaye, communicate karke memory share karein.”

Message-sending concurrency ko accomplish karne ke liye, Rust ki standard library
channels ki implementation provide karti hai. Ek *channel* ek general programming
concept hai jis ke zariye data ek thread se doosre thread ko bheja jata hai.

Aap programming mein channel ko ek directional channel of water ki tarah imagine kar
sakte hain, jaise koi stream ya river. Agar aap river mein rubber duck jaisi koi cheez
dal dein, to woh pani ke flow ke saath downstream waterway ke end tak chali jayegi.

Ek channel ke do halves hote hain: ek transmitter aur ek receiver. Transmitter half
upstream location hota hai jahan aap rubber duck ko river mein daalte hain, aur receiver
half woh jagah hoti hai jahan rubber duck downstream ja kar pohanchti hai. Aapke code
ka ek hissa transmitter par methods ko us data ke saath call karta hai jo aap send karna
chahte hain, aur doosra hissa receiving end ko aane wale messages ke liye check karta
hai. Channel ko *closed* kaha jata hai agar transmitter ya receiver half mein se koi
ek drop ho jaye.

Yahan hum dheere dheere ek aisa program banayenge jismein ek thread values generate
karke unhein channel ke zariye send karega, aur doosra thread un values ko receive
karke print karega. Feature ko illustrate karne ke liye hum channel use karte hue
threads ke darmiyan simple values send karenge. Jab aap is technique se familiar ho
jayenge, to aap channels ko un tamam threads ke liye use kar sakte hain jinhein ek
doosre ke saath communicate karna ho, jaise ek chat system ya aisa system jahan bohot
se threads calculation ke different parts perform karein aur un parts ko ek thread
ko bhejein jo results ko aggregate kare.

Sab se pehle, Listing 16-6 mein hum ek channel create karenge lekin us ke saath kuch
nahi karenge. Note karein ke yeh abhi compile nahi hoga kyun ke Rust yeh nahi bata
sakta ke hum channel ke zariye kis type ki values send karna chahte hain.

<Listing number="16-6" file-name="src/main.rs" caption="Creating a channel and assigning the two halves to `tx` and `rx`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-06/src/main.rs}}
```

</Listing>

Hum `mpsc::channel` function ko use karke ek naya channel create karte hain; `mpsc` ka
matlab *multiple producer, single consumer* hai. Mukhtasar taur par, Rust ki standard
library jis tarah channels implement karti hai us ka matlab hai ke ek channel ke paas
multiple *sending* ends ho sakte hain jo values produce karte hain, lekin sirf ek
*receiving* end hota hai jo un values ko consume karta hai. Multiple streams ko ek
bari river mein milte hue imagine karein: Kisi bhi stream se bheji gayi har cheez akhir
mein ek hi river mein pohanchegi. Filhaal hum ek single producer se shuru karenge,
lekin jab yeh example kaam karne lagega to hum multiple producers add karenge.

`mpsc::channel` function ek tuple return karta hai, jis ka pehla element sending
end—the transmitter—aur doosra element receiving end—the receiver hota hai.
Bohot se fields mein abbreviations `tx` aur `rx` traditionally *transmitter* aur
*receiver* ke liye respectively use hoti hain, is liye hum apne variables ko isi
tarah name karte hain taake har end ko indicate kiya ja sake. Hum `let` statement
ko ek aise pattern ke saath use kar rahe hain jo tuples ko destructure karta hai;
hum `let` statements mein patterns ke use aur destructuring ke baare mein Chapter 19
mein discuss karenge. Filhaal itna samajh lein ke is tarah `let` statement use karna
`mpsc::channel` se return hone wale tuple ke pieces ko extract karne ka ek convenient
approach hai.

Ab transmitting end ko ek spawned thread mein move karte hain aur us se ek string
send karwate hain taake spawned thread main thread ke saath communicate kar raha ho,
jaisa ke Listing 16-7 mein dikhaya gaya hai. Yeh bilkul aisa hai jaise upstream river
mein rubber duck dalna ya ek thread se doosre thread ko chat message bhejna.

<Listing number="16-7" file-name="src/main.rs" caption='Moving `tx` to a spawned thread and sending `"hi"`'>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-07/src/main.rs}}
```

</Listing>

Dobara, hum naya thread create karne ke liye `thread::spawn` use kar rahe hain aur
phir `move` ko use karke `tx` ko closure mein move kar rahe hain taake spawned thread
`tx` ka owner ho. Spawned thread ke liye transmitter ki ownership hona zaroori hai
taake woh channel ke zariye messages send kar sake.

Transmitter ke paas ek `send` method hai jo woh value leta hai jo hum send karna
chahte hain. `send` method `Result<T, E>` type return karta hai, is liye agar receiver
pehle hi drop ho chuka ho aur value send karne ke liye koi jagah na ho, to send
operation ek error return karega. Is example mein, hum error ki surat mein panic
karne ke liye `unwrap` call kar rahe hain. Lekin real application mein, hum isay
properly handle karenge: Proper error handling ki strategies review karne ke liye
Chapter 9 par wapas jayein.

Listing 16-8 mein, hum main thread mein receiver se value hasil karenge. Yeh bilkul
aisa hai jaise river ke end par pani se rubber duck nikalna ya chat message receive
karna.

<Listing number="16-8" file-name="src/main.rs" caption='Receiving the value `"hi"` in the main thread and printing it'>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-08/src/main.rs}}
```

</Listing>

Receiver ke paas do useful methods hain: `recv` aur `try_recv`. Hum `recv` use kar
rahe hain, jo *receive* ka short form hai, aur yeh main thread ki execution ko block
karega aur tab tak wait karega jab tak channel ke zariye koi value send nahi hoti.
Jab koi value send ho jati hai, `recv` use `Result<T, E>` mein return karega. Jab
transmitter close ho jata hai, `recv` ek error return karega jo signal karta hai ke
ab koi aur values nahi aayengi.

`try_recv` method block nahi karta, balki foran `Result<T, E>` return karta hai:
agar koi message available ho to message ko hold karne wali `Ok` value, aur agar is
waqt koi messages available na hon to `Err` value. `try_recv` use karna us waqt
useful hai jab is thread ko messages ka wait karte hue koi doosra work bhi karna ho:
hum ek loop likh sakte hain jo har kuch der baad `try_recv` call kare, agar koi
message available ho to use handle kare, aur warna dobara check karne tak thori der
ke liye doosra work kare.

Humne is example mein simplicity ke liye `recv` use kiya hai; main thread ke paas
messages ka wait karne ke ilawa koi aur work nahi hai, is liye main thread ko block
karna appropriate hai.

Jab hum Listing 16-8 ka code run karenge, to humein main thread se printed value
nazar aayegi:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
```

Perfect!

<!-- Old headings. Do not remove or links may break. -->

<a id="channels-and-ownership-transference"></a>

### Transferring Ownership Through Channels

Ownership rules message sending mein vital role play karte hain kyun ke yeh aapko
safe, concurrent code likhne mein help karte hain. Concurrent programming mein
errors ko prevent karna Rust ke tamam programs mein ownership ke baare mein sochne
ka ek faida hai. Aaiye ek experiment karte hain taake dekhein ke channels aur
ownership problems ko prevent karne ke liye kis tarah mil kar kaam karte hain:
hum spawned thread mein `val` value ko channel ke zariye send karne ke *baad*
use karne ki koshish karenge. Listing 16-9 ke code ko compile karne ki koshish
karein taake dekhein ke yeh code allowed kyun nahi hai.

<Listing number="16-9" file-name="src/main.rs" caption="Attempting to use `val` after we’ve sent it down the channel">

```rust,ignore,does_not_compile id="r8w2hx"
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-09/src/main.rs}}
```

</Listing>

Yahan hum `tx.send` ke zariye `val` ko channel ke through send karne ke baad
use print karne ki koshish karte hain. Is ki permission dena ek bad idea hoga:
jab value kisi doosre thread ko send ho jati hai, to woh thread humare dobara
value ko use karne ki koshish karne se pehle usay modify ya drop kar sakta hai.
Mumkin hai ke doosre thread ki modifications inconsistent ya nonexistent data
ki wajah se errors ya unexpected results cause karein. Lekin Rust humein error
deta hai agar hum Listing 16-9 ke code ko compile karne ki koshish karein:

```console id="s3v5xe"
{{#include ../listings/ch16-fearless-concurrency/listing-16-09/output.txt}}
```

Hamari concurrency mistake ne compile-time error paida kar diya hai. `send` function
apne parameter ki ownership le leta hai, aur jab value move hoti hai to receiver
us ki ownership le leta hai. Is se hum value ko send karne ke baad accidentally
dobara use karne se ruk jate hain; ownership system check karta hai ke sab kuch
theek hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="sending-multiple-values-and-seeing-the-receiver-waiting"></a>

### Sending Multiple Values

Listing 16-8 ka code compile aur run hua, lekin is ne humein clearly yeh nahi dikhaya ke
do separate threads channel ke zariye ek doosre se baat kar rahe the.

Listing 16-10 mein humne kuch modifications ki hain jo prove karengi ke Listing 16-8 ka code
concurrently run ho raha hai: Ab spawned thread multiple messages send karega aur har message
ke darmiyan ek second ke liye pause karega.

<Listing number="16-10" file-name="src/main.rs" caption="Sending multiple messages and pausing between each one">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-10/src/main.rs}}
```

</Listing>

Is baar, spawned thread ke paas strings ka ek vector hai jise hum main thread ko send karna
chahte hain. Hum in par iterate karte hain, har ek ko individually send karte hain, aur
`thread::sleep` function ko one second ki `Duration` value ke saath call karke har message
ke darmiyan pause karte hain.

Main thread mein, hum ab `recv` function ko explicitly call nahi kar rahe:
Is ke bajaye, hum `rx` ko ek iterator ki tarah treat kar rahe hain. Har received value ke
liye, hum use print kar rahe hain. Jab channel close ho jata hai, to iteration end ho jayegi.

Listing 16-10 ka code run karte waqt, aapko har line ke darmiyan one-second pause ke saath
following output nazar aana chahiye:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: from
Got: the
Got: thread
```

Kyun ke main thread ke `for` loop mein aisa koi code nahi hai jo pause ya delay karta ho,
hum bata sakte hain ke main thread spawned thread se values receive karne ka wait kar raha hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-multiple-producers-by-cloning-the-transmitter"></a>

### Creating Multiple Producers

Pehle humne mention kiya tha ke `mpsc`, *multiple producer, single consumer* ka acronym
hai. Ab `mpsc` ko use karte hain aur Listing 16-10 ke code ko expand karke multiple
threads create karte hain jo sab ek hi receiver ko values send karte hain. Hum yeh
transmitter ko clone karke kar sakte hain, jaisa ke Listing 16-11 mein dikhaya gaya hai.

<Listing number="16-11" file-name="src/main.rs" caption="Sending multiple messages from multiple producers">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-11/src/main.rs:here}}
```

</Listing>

Is baar, pehla spawned thread create karne se pehle, hum transmitter par `clone` call
karte hain. Is se humein ek naya transmitter milega jise hum pehle spawned thread ko
pass kar sakte hain. Hum original transmitter ko doosre spawned thread ko pass karte hain.
Is se humare paas do threads ho jate hain, jin mein se har ek different messages ek hi
receiver ko send karta hai.

Jab aap code run karenge, to aapka output kuch is tarah nazar aana chahiye:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

Aapko values kisi doosre order mein bhi nazar aa sakti hain, jo aapke system par depend
karta hai. Yehi cheez concurrency ko interesting hone ke saath saath difficult bhi
banati hai. Agar aap `thread::sleep` ke saath experiment karein aur different threads
mein ise mukhtalif values dein, to har run zyada nondeterministic hoga aur har baar
different output create karega.

Ab jab humne dekh liya hai ke channels kis tarah kaam karte hain, to chaliye concurrency
ke ek different method ko dekhte hain.
