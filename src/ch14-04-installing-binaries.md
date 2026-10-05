<!-- Old headings. Do not remove or links may break. -->

<a id="installing-binaries-from-cratesio-with-cargo-install"></a>

## Installing Binaries with `cargo install`

`cargo install` command aapko binary crates ko locally install karke use karne
ki ijazat deta hai. Is ka maqsad system packages ko replace karna nahi hai;
balki yeh Rust developers ke liye un tools ko install karne ka ek convenient
tareeqa hai jo doosre logon ne [crates.io](https://crates.io/)<!-- ignore -->
par share kiye hain. Note karein ke aap sirf un packages ko install kar sakte
hain jin mein binary targets hon. Ek *binary target* woh runnable program hota
hai jo us waqt create hota hai jab crate mein *src/main.rs* file ho ya binary
ke taur par specify ki gayi koi doosri file ho. Is ke muqable mein ek library
target apne aap runnable nahi hota, lekin doosre programs ke andar include
karne ke liye suitable hota hai. Aam tor par, crates ki README file mein yeh
information hoti hai ke crate ek library hai, binary target rakhta hai, ya
dono rakhta hai.

`cargo install` se install ki gayi tamam binaries installation root ke *bin*
folder mein store hoti hain. Agar aapne Rust *rustup.rs* ko use karke install
kiya hai aur aapke paas koi custom configurations nahi hain, to yeh directory
*$HOME/.cargo/bin* hogi. Ensure karein ke yeh directory aapke `$PATH` mein
maujood ho taa-ke aap `cargo install` se install kiye gaye programs ko run kar
saken.

Misal ke taur par, Chapter 12 mein humne mention kiya tha ke files search karne
ke liye `grep` tool ki ek Rust implementation hai jise `ripgrep` kaha jata hai.
`ripgrep` ko install karne ke liye hum following run kar sakte hain:

<!-- manual-regeneration
cargo install something you don't have, copy relevant output below
-->

```console
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v14.1.1
  Downloaded 1 crate (213.6 KB) in 0.40s
  Installing ripgrep v14.1.1
--snip--
   Compiling grep v0.3.2
    Finished `release` profile [optimized + debuginfo] target(s) in 6.73s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v14.1.1` (executable `rg`)
```

Output ki second-to-last line installed binary ki location aur name dikhati hai,
jo `ripgrep` ke case mein `rg` hai. Jab tak installation directory aapke
`$PATH` mein hai, jaisa ke pehle mention kiya gaya hai, aap phir
`rg --help` run kar sakte hain aur files search karne ke liye ek zyada fast,
Rustier tool use karna shuru kar sakte hain!
