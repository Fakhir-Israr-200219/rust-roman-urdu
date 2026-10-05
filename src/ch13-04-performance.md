<!-- Old headings. Do not remove or links may break. -->

<a id="comparing-performance-loops-vs-iterators"></a>

## Performance in Loops vs. Iterators

Yeh determine karne ke liye ke loops ya iterators mein se kisay use karna chahiye, aapko yeh maloom hona zaroori hai ke kaunsi implementation zyada fast hai: `search` function ka woh version jismein explicit `for` loop hai ya woh version jo iterators use karta hai.

Humne ek benchmark run kiya jismein Sir Arthur Conan Doyle ki *The Adventures of Sherlock Holmes* ka poora content ek `String` mein load kiya aur content mein *the* word ko search kiya. Yahan `for` loop use karne wale `search` ke version aur iterators use karne wale version ke benchmark ke results hain:

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

Dono implementations ki performance similar hai! Hum yahan benchmark code explain nahi karenge, kyun ke maqsad yeh prove karna nahi hai ke dono versions equivalent hain, balki yeh general sense dena hai ke performance ke hawale se yeh dono implementations ek doosre ke muqable mein kaisi hain.

Zyada comprehensive benchmark ke liye, aapko mukhtalif sizes ke mukhtalif texts ko `contents` ke taur par, mukhtalif words ko aur mukhtalif lengths ke words ko `query` ke taur par, aur doosri tamam qisam ki variations ke saath test karna chahiye. Point yeh hai: Iterators, high-level abstraction hone ke bawajood, compile hokar roughly usi code mein convert hote hain jo aapne khud lower-level code ke taur par likha hota. Iterators Rust ki *zero-cost abstractions* mein se ek hain, jiska matlab yeh hai ke abstraction ko use karne se koi additional runtime overhead impose nahi hota. Yeh usi tarah hai jaise Bjarne Stroustrup, jo C++ ke original designer aur implementor hain, apni 2012 ETAPS keynote presentation “Foundations of C++” mein zero-overhead ko define karte hain:

> In general, C++ implementations obey the zero-overhead principle: What you
> don’t use, you don’t pay for. And further: What you do use, you couldn’t hand
> code any better.

Bohat se cases mein, iterators use karne wala Rust code usi assembly mein compile hota hai jo aap khud manually likhte. Optimizations jaise loop unrolling aur array access par bounds checking ko eliminate karna apply hoti hain aur resultant code ko extremely efficient banati hain. Ab jab aap yeh jaante hain, to aap bina kisi khauf ke iterators aur closures use kar sakte hain! Yeh code ko aisa feel karwate hain jaise woh higher level ka ho, lekin aisa karne ke liye koi runtime performance penalty impose nahi karte.

## Summary

Closures aur iterators Rust ke aise features hain jo functional programming language ideas se inspired hain. Yeh low-level performance par high-level ideas ko clearly express karne ki Rust ki capability mein contribute karte hain. Closures aur iterators ki implementations is tarah design ki gayi hain ke runtime performance affect nahi hoti. Yeh Rust ke zero-cost abstractions provide karne ke goal ka hissa hai.

Ab jab humne apne I/O project ki expressiveness ko improve kar liya hai, to aaiye `cargo` ke kuch aur features ko dekhein jo project ko duniya ke saath share karne mein hamari madad karenge.
