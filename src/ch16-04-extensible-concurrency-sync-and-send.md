<!-- Old headings. Do not remove or links may break. -->

<a id="extensible-concurrency-with-the-sync-and-send-traits"></a> <a id="extensible-concurrency-with-the-send-and-sync-traits"></a>

## Extensible Concurrency with `Send` and `Sync`

Dilchasp baat yeh hai ke ab tak is chapter mein humne concurrency ke jin features ke baare
mein baat ki hai, un mein se almost har ek standard library ka hissa raha hai, language ka
nahi. Concurrency handle karne ke liye aapke options language ya standard library tak
limited nahi hain; aap apne khud ke concurrency features likh sakte hain ya doosron ke
likhe hue features use kar sakte hain.

Lekin, key concurrency concepts mein se jo standard library ke bajaye language mein embedded
hain, woh `std::marker` traits `Send` aur `Sync` hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="allowing-transference-of-ownership-between-threads-with-send"></a>

### Transferring Ownership Between Threads

`Send` marker trait yeh indicate karta hai ke `Send` implement karne wali type ki values ki
ownership threads ke darmiyan transfer ki ja sakti hai. Almost har Rust type `Send`
implement karti hai, lekin kuch exceptions hain, jin mein `Rc<T>` bhi shamil hai: Yeh
`Send` implement nahi kar sakti kyun ke agar aap kisi `Rc<T>` value ko clone karke clone ki
ownership kisi doosre thread ko transfer karne ki koshish karein, to dono threads ek hi waqt
reference count ko update kar sakte hain. Isi wajah se, `Rc<T>` ko single-threaded
situations mein use karne ke liye implement kiya gaya hai jahan aap thread-safe performance
penalty pay nahi karna chahte.

Is liye, Rust ka type system aur trait bounds yeh ensure karte hain ke aap kabhi bhi
galti se kisi `Rc<T>` value ko threads ke darmiyan unsafely send nahi kar sakte. Jab humne
Listing 16-14 mein aisa karne ki koshish ki, to humein error mila
`` the trait `Send` is not implemented for `Rc<Mutex<i32>>` ``. Jab humne `Arc<T>` par
switch kiya, jo `Send` implement karta hai, to code compile ho gaya.

Koi bhi type jo poori tarah `Send` types se composed ho, automatically `Send` ke taur par
bhi marked hoti hai. Almost tamam primitive types `Send` hain, raw pointers ke ilawa,
jin ke baare mein hum Chapter 20 mein baat karenge.

<!-- Old headings. Do not remove or links may break. -->

<a id="allowing-access-from-multiple-threads-with-sync"></a>

### Accessing from Multiple Threads

`Sync` marker trait yeh indicate karta hai ke `Sync` implement karne wali type ko multiple
threads se reference karna safe hai. Doosre lafzon mein, koi bhi type `T` `Sync` implement
karti hai agar `&T` (yani `T` ka immutable reference) `Send` implement karta ho, jis ka
matlab hai ke reference ko safely kisi doosre thread ko send kiya ja sakta hai. `Send` ki
tarah, tamam primitive types `Sync` implement karte hain, aur jo types poori tarah un types
se composed hon jo `Sync` implement karti hain, woh bhi `Sync` implement karti hain.

Smart pointer `Rc<T>` bhi unhi reasons ki wajah se `Sync` implement nahi karta jin ki wajah
se yeh `Send` implement nahi karta. `RefCell<T>` type (jis ke baare mein humne Chapter 15
mein baat ki thi) aur related `Cell<T>` types ki family bhi `Sync` implement nahi karti.
`RefCell<T>` runtime par jo borrow checking perform karta hai woh thread-safe nahi hai. Smart
pointer `Mutex<T>` `Sync` implement karta hai aur multiple threads ke saath access share
karne ke liye use kiya ja sakta hai, jaisa ke aapne [“Shared Access to
`Mutex<T>`”][shared-access]<!-- ignore --> mein dekha.

### Implementing `Send` and `Sync` Manually Is Unsafe

Kyun ke doosri tamam types jo `Send` aur `Sync` traits implement karne wali types se entirely
composed hoti hain, woh bhi automatically `Send` aur `Sync` implement karti hain, is liye
humein in traits ko manually implement karne ki zaroorat nahi hoti. Marker traits hone ki
wajah se, in mein implement karne ke liye koi methods bhi nahi hote. Yeh sirf concurrency
se related invariants ko enforce karne ke liye useful hain.

In traits ko manually implement karne mein unsafe Rust code ko implement karna shamil hota
hai. Hum Chapter 20 mein unsafe Rust code use karne ke baare mein baat karenge; filhaal,
important information yeh hai ke naye concurrent types banana jo `Send` aur `Sync` parts se
nahi bane hote, safety guarantees ko uphold karne ke liye careful thought require karta hai.
In guarantees ke baare mein aur unhein uphold karne ke tareeqe ke liye
[“The Rustonomicon”][nomicon] mein zyada information hai.


## Summary

Is book mein concurrency ke baare mein aapko abhi aur dekhne ko milega: next chapter async
programming par focus karta hai, aur Chapter 21 ka project is chapter ke concepts ko yahan
discuss kiye gaye chhote examples ki nisbat zyada realistic situation mein use karega.

Jaisa ke pehle mention kiya gaya tha, Rust concurrency ko handle karne ka bohot kam hissa
language ka part hai, is liye bohot se concurrency solutions crates ke taur par implement
kiye gaye hain. Yeh standard library ki nisbat zyada tezi se evolve hote hain, is liye
multithreaded situations mein use karne ke liye current, state-of-the-art crates ko online
zaroor search karein.

Rust standard library message passing ke liye channels aur smart pointer types, jaise
`Mutex<T>` aur `Arc<T>`, provide karti hai jo concurrent contexts mein use karna safe hai.
Type system aur borrow checker ensure karte hain ke in solutions ko use karne wala code
data races ya invalid references ke saath end nahi hoga. Jab aapka code compile ho jaye, to
aap itminan rakh sakte hain ke woh multiple threads par khushi se run karega, un mushkil
bugs ki qisam ke baghair jinhein doosri languages mein track down karna common hota hai.
Concurrent programming ab aisa concept nahi raha jis se darne ki zaroorat ho:
Aage barhein aur apne programs ko fearlessly concurrent banayein!

[shared-access]: ch16-03-shared-state.html#shared-access-to-mutext
[nomicon]: ../nomicon/index.html
