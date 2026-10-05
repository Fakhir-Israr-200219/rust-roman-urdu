## Customizing Builds with Release Profiles

Rust mein, *release profiles* pehle se defined aur customizable profiles hain jin mein different configurations hoti hain jo programmer ko code compile karne ke mukhtalif options par zyada control deti hain. Har profile doosri profiles se independently configured hoti hai.

Cargo ke do main profiles hain: `dev` profile jise Cargo tab use karta hai jab aap `cargo
build` run karte hain, aur `release` profile jise Cargo tab use karta hai jab aap `cargo build --release` run karte hain. `dev` profile development ke liye achhe defaults ke saath defined hai, aur `release` profile release builds ke liye achhe defaults rakhti hai.

Yeh profile names shayad aapko apni builds ke output se familiar hon:

<!-- manual-regeneration
anywhere, run:
cargo build
cargo build --release
and ensure output below is accurate
-->

```console
$ cargo build
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
$ cargo build --release
    Finished `release` profile [optimized] target(s) in 0.32s
```

`dev` aur `release` compiler ke zariye use ki jane wali yeh different profiles hain.

Cargo ke paas har profile ke liye default settings hoti hain jo tab apply hoti hain jab aapne project ki *Cargo.toml* file mein explicitly koi `[profile.*]` section add nahi kiya hota. Kisi bhi profile ko customize karne ke liye `[profile.*]` sections add karke, aap default settings ke kisi bhi subset ko override kar sakte hain. Misal ke taur par, `dev` aur `release` profiles ke liye `opt-level` setting ki default values yeh hain:

<span class="filename">Filename: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

`opt-level` setting yeh control karti hai ke Rust aapke code par kitni optimizations apply karega, jiska range 0 se 3 tak hai. Zyada optimizations apply karne se compiling time barhta hai, is liye agar aap development mein hain aur apne code ko frequently compile kar rahe hain, to aap kam optimizations chahenge taake code jaldi compile ho, chahe resultant code slower run kare. Is liye `dev` ka default `opt-level` `0` hai. Jab aap apne code ko release karne ke liye ready hon, to compiling mein zyada time spend karna behtar hai. Aap release mode mein sirf ek baar compile karenge, lekin compiled program ko bohat dafa run karenge, is liye release mode zyada compile time ko aise code ke saath trade karta hai jo faster run hota hai. Isi wajah se `release` profile ka default `opt-level` `3` hai.

Aap *Cargo.toml* mein us setting ke liye different value add karke default setting ko override kar sakte hain. Misal ke taur par, agar hum development profile mein optimization level 1 use karna chahte hain, to hum apne project ki *Cargo.toml* file mein yeh do lines add kar sakte hain:

<span class="filename">Filename: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 1
```

Yeh code default setting `0` ko override karta hai. Ab jab hum `cargo build` run karenge, Cargo `dev` profile ke defaults ke saath `opt-level` ke liye hamari customization use karega. Kyun ke humne `opt-level` ko `1` set kiya hai, Cargo default ke muqable mein zyada optimizations apply karega, lekin release build jitni nahi.

Har profile ke configuration options aur defaults ki complete list ke liye [Cargo ki documentation](https://doc.rust-lang.org/cargo/reference/profiles.html) dekhein.
