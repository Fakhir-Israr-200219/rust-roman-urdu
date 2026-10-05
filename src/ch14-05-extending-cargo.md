## Extending Cargo with Custom Commands

Cargo ko is tarah design kiya gaya hai ke aap us mein naye subcommands add kar
sakte hain bina Cargo ko modify kiye. Agar aapke `$PATH` mein koi binary
`cargo-something` ke naam se maujood hai, to aap `cargo something` run karke
usay aise hi chala sakte hain jaise woh Cargo ka koi subcommand ho. Is tarah
ke custom commands bhi us waqt list hote hain jab aap `cargo --list` run karte
hain. `cargo install` ko use karke extensions install karna aur phir unhein
bilkul built-in Cargo tools ki tarah run kar pana Cargo ke design ka ek
bohat convenient faida hai!

## Summary

Cargo aur [crates.io](https://crates.io/)<!-- ignore --> ke saath code share
karna un cheezon ka hissa hai jo Rust ecosystem ko bohat se different tasks
ke liye useful banati hain. Rust ki standard library chhoti aur stable hai,
lekin crates ko share karna, use karna, aur improve karna aasaan hai, aur yeh
sab language ke development timeline se different timeline par ho sakta hai.
[crates.io](https://crates.io/)<!-- ignore --> par aisa code share karne se
hichkichayein nahi jo aapke liye useful hai; imkaan hai ke woh kisi aur ke liye
bhi useful ho!
