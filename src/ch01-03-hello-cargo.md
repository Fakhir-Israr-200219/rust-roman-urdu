## Hello, Cargo!

Cargo Rust ka build system aur package manager hai. Zyada tar Rustaceans apne Rust projects ko manage karne ke liye is tool ko use karte hain, kyun ke Cargo aapke liye bohat se tasks handle karta hai, jaise aapka code build karna, un libraries ko download karna jin par aapka code depend karta hai, aur un libraries ko build karna. (Hum un libraries ko jin ki aapke code ko zaroorat hoti hai *dependencies* kehte hain.)

Sab se simple Rust programs, jaise jo hum ne ab tak likha hai, unki koi dependencies nahi hoti. Agar hum ne “Hello, world!” project ko Cargo ke saath build kiya hota, to ye sirf Cargo ke us hisse ko use karta jo aapka code build karta hai. Jaise jaise aap zyada complex Rust programs likhenge, aap dependencies add karenge, aur agar aap Cargo ko use karke project shuru karte hain, to dependencies add karna bohat aasaan hoga.

Kyun ke Rust projects ki bohat badi majority Cargo use karti hai, is kitab ka baqi hissa ye assume karta hai ke aap bhi Cargo use kar rahe hain. Agar aap ne [“Installation”][installation]<!-- ignore --> section mein discuss kiye gaye official installers use kiye hain, to Cargo Rust ke saath installed aata hai. Agar aap ne Rust kisi doosre tareeqe se install kiya hai, to apne terminal mein following enter karke check karein ke Cargo installed hai ya nahi:

```console
$ cargo --version
```

Agar aapko version number nazar aata hai, to Cargo installed hai! Agar aapko koi error nazar aata hai, jaise `command not found`, to apne installation method ki documentation dekhein aur maloom karein ke Cargo ko separately kaise install karna hai.


### Creating a Project with Cargo

Aaiye Cargo ko use karke ek naya project banate hain aur dekhte hain ke ye hamare original “Hello, world!” project se kis tarah different hai. Wapas apni *projects* directory mein jayein (ya jahan bhi aap ne apna code rakhne ka faisla kiya hai). Phir, kisi bhi operating system par following commands run karein:

```console
$ cargo new hello_cargo
$ cd hello_cargo
```

Pehli command ek nayi directory aur *hello_cargo* naam ka project create karti hai. Hum ne apne project ka naam *hello_cargo* rakha hai, aur Cargo iski files isi naam ki directory mein create karta hai.

*hello_cargo* directory mein jayein aur files ki list dekhein. Aap dekhenge ke Cargo ne hamare liye do files aur ek directory generate ki hai: ek *Cargo.toml* file aur ek *src* directory jiske andar *main.rs* file hai.

Is ne ek nayi Git repository ko bhi ek *.gitignore* file ke saath initialize kiya hai. Agar aap kisi existing Git repository ke andar `cargo new` run karte hain, to Git files generate nahi hongi; aap `cargo new --vcs=git` use karke is behavior ko override kar sakte hain.

> Note: Git ek common version control system hai. Aap `--vcs` flag use karke `cargo new` ko kisi different version control system ya kisi bhi version control system ke baghair use karne ke liye change kar sakte hain. Available options dekhne ke liye `cargo new --help` run karein.

Apne pasand ke text editor mein *Cargo.toml* open karein. Ye Listing 1-2 ke code jaisa nazar aana chahiye.

<Listing number="1-2" file-name="Cargo.toml" caption="`cargo new` ke zariye generate ki gayi *Cargo.toml* ki contents">

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

</Listing>

Ye file [*TOML*][toml]<!-- ignore --> (*Tom’s Obvious, Minimal Language*) format mein hai, jo Cargo ka configuration format hai.

Pehli line, `[package]`, ek section heading hai jo indicate karti hai ke is ke baad wali statements ek package ko configure kar rahi hain. Jaise jaise hum is file mein mazeed information add karenge, hum doosre sections bhi add karenge.

Agli teen lines woh configuration information set karti hain jo Cargo ko aapka program compile karne ke liye chahiye: naam, version, aur Rust ki woh edition jo use karni hai. Hum `edition` key ke baare mein [Appendix E][appendix-e]<!-- ignore --> mein baat karenge.

Aakhri line, `[dependencies]`, ek aise section ki shuruaat hai jahan aap apne project ki dependencies list karenge. Rust mein code ke packages ko *crates* kaha jata hai. Humein is project ke liye kisi doosre crate ki zaroorat nahi hogi, lekin Chapter 2 ke pehle project mein humein crates ki zaroorat hogi, is liye hum us waqt is dependencies section ko use karenge.

Ab *src/main.rs* open karein aur dekhein:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo ne aapke liye “Hello, world!” program generate kiya hai, bilkul us program ki tarah jo hum ne Listing 1-1 mein likha tha! Abhi tak hamare project aur Cargo ke generate kiye hue project ke darmiyan farq sirf ye hai ke Cargo ne code ko *src* directory mein rakha hai, aur hamare paas top directory mein ek *Cargo.toml* configuration file hai.

Cargo expect karta hai ke aapki source files *src* directory ke andar maujood hon. Top-level project directory sirf README files, license information, configuration files, aur kisi bhi doosri cheez ke liye hai jo aapke code se related na ho. Cargo use karne se aapko apne projects ko organize karne mein madad milti hai. Har cheez ke liye ek jagah hai, aur har cheez apni jagah par hai.

Agar aap ne aisa project shuru kiya jo Cargo use nahi karta, jaise hum ne “Hello, world!” project ke saath kiya tha, to aap use aise project mein convert kar sakte hain jo Cargo use karta ho. Project code ko *src* directory mein move karein aur ek munasib *Cargo.toml* file create karein. Us *Cargo.toml* file ko hasil karne ka ek aasaan tareeqa `cargo init` run karna hai, jo ise aapke liye automatically create kar dega.


### Building and Running a Cargo Project

Ab dekhte hain ke “Hello, world!” program ko Cargo ke saath build aur run karne par kya different hota hai! Apni *hello_cargo* directory se following command enter karke apna project build karein:

```console
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

Ye command current directory ke bajaye *target/debug/hello_cargo* mein executable file create karti hai (ya Windows par *target\debug\hello_cargo.exe*). Kyun ke default build ek debug build hota hai, Cargo binary ko *debug* naam ki directory mein rakhta hai. Aap executable ko is command ke saath run kar sakte hain:

```console
$ ./target/debug/hello_cargo # or .\target\debug\hello_cargo.exe on Windows
Hello, world!
```

Agar sab kuch theek raha, to `Hello, world!` terminal mein print hona chahiye. Pehli baar `cargo build` run karne se Cargo top level par ek nayi file bhi create karta hai: *Cargo.lock*. Ye file aapke project mein dependencies ke exact versions ka record rakhti hai. Is project mein koi dependencies nahi hain, is liye ye file kaafi sparse hai. Aapko is file ko kabhi manually change karne ki zaroorat nahi hogi; Cargo iske contents ko aapke liye manage karta hai.

Hum ne abhi `cargo build` ke saath project build kiya aur `./target/debug/hello_cargo` ke saath ise run kiya, lekin hum `cargo run` bhi use kar sakte hain, jo code ko compile karke resultant executable ko ek hi command mein run karta hai:

```console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

`cargo run` use karna is baat se zyada convenient hai ke aapko `cargo build` run karna aur phir binary ka poora path yaad rakhna pade, is liye zyada tar developers `cargo run` use karte hain.

Notice karein ke is baar humein aisa output nazar nahi aaya jo indicate kare ke Cargo `hello_cargo` ko compile kar raha hai. Cargo ne samajh liya ke files mein koi change nahi hua, is liye us ne project ko dobara build nahi kiya, sirf binary run kar di. Agar aap ne apne source code mein modification ki hoti, to Cargo use run karne se pehle project ko dobara build karta, aur aapko ye output nazar aata:

```console
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Cargo `cargo check` naam ki ek command bhi provide karta hai. Ye command aapke code ko quickly check karti hai taake ye ensure ho ke code compile hota hai, lekin executable produce nahi karti:

```console
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

Aap executable kyun nahi chahenge? Aksar `cargo check`, `cargo build` se kaafi zyada fast hota hai kyun ke ye executable produce karne wala step skip kar deta hai. Agar aap code likhte waqt continuously apna kaam check kar rahe hain, to `cargo check` use karne se ye pata chalne ka process tez ho jayega ke aapka project ab bhi compile ho raha hai ya nahi! Isi liye bohat se Rustaceans apna program likhte hue periodically `cargo check` run karte hain taake ensure kar saken ke ye compile hota hai. Phir jab executable use karne ke liye tayyar hote hain, to `cargo build` run karte hain.

Aaiye ab tak Cargo ke baare mein jo kuch hum ne seekha hai us ka recap karte hain:

* Hum `cargo new` use karke project create kar sakte hain.
* Hum `cargo build` use karke project build kar sakte hain.
* Hum `cargo run` use karke ek hi step mein project ko build aur run kar sakte hain.
* Hum `cargo check` use karke binary produce kiye baghair project build kar sakte hain taake errors check kiye ja saken.
* Build ka result hamare code wali same directory mein save karne ke bajaye, Cargo ise *target/debug* directory mein store karta hai.

Cargo use karne ka ek additional faida ye hai ke commands same rehti hain, chahe aap kisi bhi operating system par kaam kar rahe hon. Is liye ab se hum Linux aur macOS ke muqable mein Windows ke liye specific instructions provide nahi karenge.

### Building for Release

Jab aapka project aakhirkar release ke liye tayyar ho, to aap optimizations ke saath compile karne ke liye `cargo build --release` use kar sakte hain. Ye command *target/release* mein executable create karegi, *target/debug* mein nahi. Optimizations aapke Rust code ko tez run karne deti hain, lekin inhein enable karne se aapke program ko compile hone mein zyada waqt lagta hai. Isi liye do different profiles hain: ek development ke liye, jab aap jaldi aur frequently rebuild karna chahte hain, aur doosra final program build karne ke liye jo aap user ko denge, jise baar baar rebuild nahi kiya jayega aur jo mumkin ho utni tezi se run karega. Agar aap apne code ke running time ko benchmark kar rahe hain, to zaroor `cargo build --release` run karein aur *target/release* mein maujood executable ke saath benchmark karein.

<!-- Old headings. Do not remove or links may break. -->

<a id="cargo-as-convention"></a>


### Leveraging Cargo’s Conventions

Simple projects ke saath, sirf `rustc` use karne ke muqable mein Cargo zyada value provide nahi karta, lekin jaise jaise aapke programs zyada complex hote jayenge, ye apni usefulness sabit karega. Jab programs multiple files tak grow karte hain ya kisi dependency ki zaroorat hoti hai, to build ko coordinate karne ka kaam Cargo par chhor dena kaafi aasaan hota hai.

Chahe `hello_cargo` project simple hai, lekin ab ye bohat se aise real tooling ko use karta hai jo aap apne Rust career ke baqi hisson mein use karenge. Asal mein, kisi bhi existing project par kaam karne ke liye aap code ko Git ke zariye checkout karne, us project ki directory mein jane, aur build karne ke liye following commands use kar sakte hain:

```console id="1m8xg4"
$ git clone example.org/someproject
$ cd someproject
$ cargo build
```

Cargo ke baare mein mazeed information ke liye [is ki documentation][cargo] dekhein.


## Summary

Aap apne Rust journey ki bohat achhi shuruat kar chuke hain! Is chapter mein aap ne seekha ke:

* `rustup` ko use karke Rust ka latest stable version install karna.
* Naye Rust version par update karna.
* Locally installed documentation ko open karna.
* `rustc` ko directly use karke “Hello, world!” program likhna aur run karna.
* Cargo ke conventions ko use karke naya project create aur run karna.

Ab ye ek zyada substantial program build karne ka acha waqt hai, taa-ke aap Rust code ko read aur write karne ke aadi ho sakein. Is liye Chapter 2 mein hum ek guessing game program build karenge. Agar aap pehle ye seekhna chahte hain ke common programming concepts Rust mein kaise kaam karte hain, to Chapter 3 dekhein aur phir Chapter 2 par wapas aayein.

[installation]: ch01-01-installation.html#installation
[toml]: https://toml.io
[appendix-e]: appendix-05-editions.html
[cargo]: https://doc.rust-lang.org/cargo/

