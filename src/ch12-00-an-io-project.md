# An I/O Project: Building a Command Line Program

Yeh chapter un bohat si skills ka recap hai jo aap ab tak seekh chuke hain aur standard library ki kuch mazeed features ki exploration bhi hai. Hum ek command line tool build karenge jo file aur command line input/output ke saath interact karega, taake Rust ke un concepts ki practice ki ja sake jo ab aap seekh chuke hain.

Rust ki speed, safety, single binary output, aur cross-platform support ise command line tools banane ke liye ek ideal language banate hain, is liye apne project ke liye hum classic command line search tool `grep` ka apna version banayenge (**g**lobally search a **r**egular **e**xpression and **p**rint). Sab se simple use case mein, `grep` ek specified file mein specified string ko search karta hai. Aisa karne ke liye, `grep` apne arguments ke taur par ek file path aur ek string leta hai. Phir yeh file ko read karta hai, file mein un lines ko find karta hai jin mein argument wali string maujood hoti hai, aur un lines ko print karta hai.

Is process mein, hum yeh bhi dikhayenge ke apne command line tool ko un terminal features ko kaise use karwaya ja sakta hai jo bohat se doosre command line tools use karte hain. Hum ek environment variable ki value read karenge taake user hamare tool ke behavior ko configure kar sake. Hum error messages ko standard output (`stdout`) ke bajaye standard error console stream (`stderr`) par bhi print karenge, taake, misal ke taur par, user successful output ko ek file mein redirect kar sake aur saath hi screen par error messages dekh sake.

Rust community ke ek member, Andrew Gallant, ne pehle hi `grep` ka ek fully featured, bohat fast version create kiya hai, jise `ripgrep` kaha jata hai. Comparison mein, hamara version kaafi simple hoga, lekin yeh chapter aapko woh background knowledge dega jo aapko `ripgrep` jaise real-world project ko samajhne ke liye darkar hai.

Hamara `grep` project ab tak seekhe gaye kai concepts ko combine karega:

* Code ko organize karna ([Chapter 7][ch7]<!-- ignore -->)
* Vectors aur strings use karna ([Chapter 8][ch8]<!-- ignore -->)
* Errors handle karna ([Chapter 9][ch9]<!-- ignore -->)
* Jahan appropriate ho, traits aur lifetimes use karna ([Chapter 10][ch10]<!-- ignore -->)
* Tests likhna ([Chapter 11][ch11]<!-- ignore -->)

Hum closures, iterators, aur trait objects ka bhi mukhtasar introduction denge, jinhein [Chapter 13][ch13]<!-- ignore --> aur [Chapter 18][ch18]<!-- ignore --> detail mein cover karenge.

[ch7]: ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
[ch8]: ch08-00-common-collections.html
[ch9]: ch09-00-error-handling.html
[ch10]: ch10-00-generics.html
[ch11]: ch11-00-testing.html
[ch13]: ch13-00-functional-features.html
[ch18]: ch18-00-oop.html
