## Putting It All Together: Futures, Tasks, and Threads

Jaisa ke humne [Chapter 16][ch16]<!-- ignore --> mein dekha, threads concurrency ke liye ek approach provide karte hain. Is chapter mein humne ek aur approach dekhi hai: futures aur streams ke saath async use karna. Agar aap soch rahe hain ke ek method ko doosre par kab choose karna chahiye, to jawab hai: yeh depend karta hai! Aur bohot se cases mein choice threads *ya* async nahi, balkay threads *aur* async hoti hai.

Bohot se operating systems ne decades se threading-based concurrency models provide kiye hue hain, aur natije ke taur par bohot si programming languages unhein support karti hain. Lekin in models ke apne tradeoffs hain. Bohot se operating systems par har thread ke liye kaafi memory use hoti hai. Threads tabhi ek option hain jab aapka operating system aur hardware unhein support karein. Mainstream desktop aur mobile computers ke unlike, kuch embedded systems mein bilkul OS nahi hota, is liye un mein threads bhi nahi hote.

Async model tradeoffs ka ek different—aur aakhirkar complementary—set provide karta hai. Async model mein concurrent operations ke liye apne threads ki zaroorat nahi hoti. Is ke bajaye, woh tasks par run kar sakte hain, jaise humne streams section mein synchronous function se work start karne ke liye `trpl::spawn_task` use kiya tha. Task thread ke similar hota hai, lekin operating system ke bajaye library-level code: runtime, usay manage karta hai.

Threads spawn karne aur tasks spawn karne ki APIs itni similar hone ki ek wajah hai. Threads synchronous operations ke sets ke liye ek boundary ki tarah act karte hain; concurrency threads *ke darmiyan* possible hoti hai. Tasks *asynchronous* operations ke sets ke liye ek boundary ki tarah act karte hain; concurrency tasks *ke darmiyan* aur tasks *ke andar* dono possible hoti hai, kyun ke ek task apne body mein futures ke darmiyan switch kar sakta hai. Aakhir mein, futures Rust ki concurrency ki sab se granular unit hain, aur har future doosre futures ke ek tree ko represent kar sakta hai. Runtime—specifically, us ka executor—tasks ko manage karta hai, aur tasks futures ko manage karte hain. Is hawale se, tasks lightweight, runtime-managed threads ke similar hain jin mein kuch additional capabilities hoti hain jo operating system ke bajaye runtime ke managed hone se aati hain.

Is ka matlab yeh nahi ke async tasks hamesha threads se behtar hain (ya is ke baraks). Threads ke saath concurrency kuch ways mein `async` ke saath concurrency se simpler programming model hai. Yeh ek strength ya weakness ho sakti hai. Threads kuch had tak “fire and forget” hote hain; un ka future ke barabar koi native equivalent nahi hota, is liye woh simply completion tak run karte hain aur sirf operating system khud hi unhein interrupt kar sakta hai.

Aur yeh pata chalta hai ke threads aur tasks aksar ek saath bohot achi tarah work karte hain, kyun ke tasks ko (kam az kam kuch runtimes mein) threads ke darmiyan move kiya ja sakta hai. Darasal, under the hood, jis runtime ko hum use kar rahe hain—jis mein `spawn_blocking` aur `spawn_task` functions bhi shamil hain—woh default tor par multithreaded hai! Bohot se runtimes *work stealing* naam ka approach use karte hain taake threads ke current utilization ki bunyaad par tasks ko transparently threads ke darmiyan move kiya ja sake aur system ki overall performance improve ho. Is approach ke liye asal mein threads *aur* tasks, aur is liye futures, ki zaroorat hoti hai.

Jab aap soch rahe hon ke kab kaunsa method use karna hai, to in rules of thumb ko consider karein:

* Agar work *very parallelizable* hai (yani CPU-bound), jaise data ke ek bunch ko process karna jahan har part ko separately process kiya ja sakta hai, to threads behtar choice hain.
* Agar work *very concurrent* hai (yani I/O-bound), jaise bohot se different sources se messages handle karna jo different intervals ya different rates par aa sakte hain, to async behtar choice hai.

Aur agar aapko parallelism aur concurrency dono ki zaroorat hai, to aapko threads aur async mein se ek choose karne ki zaroorat nahi. Aap unhein freely ek saath use kar sakte hain aur har ek ko woh role de sakte hain jis ke liye woh sab se suitable hai. Misal ke taur par, Listing 17-25 real-world Rust code mein is tarah ke mix ki ek fairly common example dikhati hai.

<Listing number="17-25" caption="Sending messages with blocking code in a thread and awaiting the messages in an async block" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-25/src/main.rs:all}}
```

</Listing>

Hum ek async channel create karke shuru karte hain, phir ek thread spawn karte hain jo `move` keyword use karke channel ke sender side ki ownership le leta hai. Thread ke andar hum 1 se 10 tak numbers send karte hain, har ek ke darmiyan ek second ke liye sleep karte hue. Aakhir mein, hum ek async block ke zariye create kiye gaye future ko `trpl::block_on` ko pass karke run karte hain, bilkul waise hi jaise humne poore chapter mein kiya hai. Us future mein hum un messages ko await karte hain, bilkul un doosre message-passing examples ki tarah jo humne dekhe hain.

Chapter ke shuru mein jis scenario se humne baat start ki thi, us ki taraf wapas aate hue, imagine karein ke aap video encoding tasks ka ek set dedicated thread use karke run kar rahe hain (kyun ke video encoding compute-bound hai), lekin UI ko in operations ke complete hone ki notification ek async channel ke zariye de rahe hain. Real-world use cases mein is tarah ke combinations ki countless examples hain.

## Summary

Yeh is book mein concurrency ke hawale se aapki aakhri mulaqat nahi hai. [Chapter 21][ch21]<!-- ignore --> ka project in concepts ko yahan discuss kiye gaye simpler examples se zyada realistic situation mein apply karega aur threading versus tasks aur futures ke saath problem-solving ka zyada direct comparison karega.

Chahe aap in approaches mein se kaunsa bhi choose karein, Rust aapko safe, fast, concurrent code likhne ke liye zaroori tools deta hai—chahe woh high-throughput web server ke liye ho ya embedded operating system ke liye.

Next, hum is baat par discuss karenge ke jaise jaise aapke Rust programs bigger hote jate hain, problems ko model karne aur solutions ko structure karne ke idiomatic tareeqe kya hain. Is ke ilawa, hum discuss karenge ke Rust ke idioms un idioms se kis tarah related hain jin se aap object-oriented programming se familiar ho sakte hain.

[ch16]: ch16-00-concurrency.html
[combining-futures]: ch17-03-more-futures.html#building-our-own-async-abstractions
[streams]: ch17-04-streams.html#composing-streams
[ch21]: ch21-00-final-project-a-web-server.html
