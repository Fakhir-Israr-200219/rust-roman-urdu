## Hello, World!

Ab jab aap Rust install kar chuke hain, to ab waqt hai ke aap apna pehla Rust program likhein.
Nayi language seekhte waqt riwayati tor par ek chhota sa program likha jata hai jo screen par `Hello, world!` text print karta hai, is liye hum yahan bhi aisa hi karenge!

> Note: Ye kitab assume karti hai ke aapko command line se basic familiarity hai. Rust aapke editing ya tooling ke hawale se koi khaas requirement nahi rakhta aur na hi is baat ki ke aapka code kahan maujood hai, is liye agar aap command line ke bajaye IDE use karna pasand karte hain, to bejhijak apna favorite IDE use karein. Aaj kal bohat se IDEs mein Rust ke liye kisi na kisi level ki support mojood hai; details ke liye IDE ki documentation dekhein. Rust team `rust-analyzer` ke zariye behtareen IDE support enable karne par focus kar rahi hai. Mazeed details ke liye [Appendix D][devtools]<!-- ignore --> dekhein.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-project-directory"></a>


### Project Directory Setup

Aap sab se pehle ek directory banayenge jahan aap apna Rust code rakhenge. Rust ke liye is baat se koi farq nahi padta ke aapka code kahan maujood hai, lekin is kitab ki exercises aur projects ke liye hum suggest karte hain ke aap apni home directory mein ek *projects* directory banayein aur apne tamam projects ko usi mein rakhein.

Ek terminal open karein aur *projects* directory aur *projects* directory ke andar “Hello, world!” project ke liye ek directory banane ke liye following commands enter karein.

Linux, macOS, aur Windows par PowerShell ke liye ye enter karein:

```console
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

Windows CMD ke liye ye enter karein:

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

<!-- Old headings. Do not remove or links may break. -->

<a id="writing-and-running-a-rust-program"></a>

### Rust Program Basics

Ab ek nayi source file banayein aur uska naam *main.rs* rakhein. Rust files hamesha *.rs* extension par khatam hoti hain. Agar aapki filename mein ek se zyada words hain, to convention ye hai ke unhein separate karne ke liye underscore use kiya jaye. Misal ke taur par, *helloworld.rs* ke bajaye *hello_world.rs* use karein.

Ab jo *main.rs* file aap ne banayi hai, use open karein aur Listing 1-1 mein diya gaya code enter karein.

<Listing number="1-1" file-name="main.rs" caption="Ek program jo `Hello, world!` print karta hai">

```rust
fn main() {
    println!("Hello, world!");
}
```

</Listing>

File save karein aur apni terminal window mein *~/projects/hello_world* directory par wapas jayein. Linux ya macOS par file ko compile aur run karne ke liye following commands enter karein:

```console
$ rustc main.rs
$ ./main
Hello, world!
```

Windows par `./main` ke bajaye `.\main` command enter karein:

```powershell
> rustc main.rs
> .\main
Hello, world!
```

Aapka operating system chahe koi bhi ho, string `Hello, world!` terminal mein print honi chahiye. Agar aapko ye output nazar nahi aata, to help hasil karne ke tareeqon ke liye Installation section ke [“Troubleshooting”][troubleshooting]<!-- ignore --> part ki taraf wapas jayein.

Agar `Hello, world!` print ho gaya, to mubarak ho! Aap ne officially ek Rust program likh liya hai. Is ka matlab hai ke ab aap Rust programmer hain—khush aamdeed!

<!-- Old headings. Do not remove or links may break. -->

<a id="anatomy-of-a-rust-program"></a>

### The Anatomy of a Rust Program

Aaiye is “Hello, world!” program ko detail mein review karte hain. Puzzle ka pehla hissa ye hai:

```rust
fn main() {

}
```

Ye lines `main` naam ka ek function define karti hain. `main` function khaas hai: Har executable Rust program mein hamesha sab se pehle isi ka code run hota hai. Yahan pehli line `main` naam ka ek function declare karti hai jiske koi parameters nahi hain aur jo kuch return nahi karta. Agar parameters hote, to woh parentheses (`()`) ke andar diye jate.

Function body ko `{}` ke andar rakha jata hai. Rust tamam function bodies ke gird curly brackets ko lazmi rakhta hai. Opening curly bracket ko function declaration wali hi line par rakhna aur darmiyan mein ek space dena achhi style hai.

> Note: Agar aap Rust projects mein ek standard style follow karna chahte hain, to apne code ko ek khaas style mein format karne ke liye `rustfmt` naam ka automatic formatter tool use kar sakte hain (`rustfmt` ke baare mein mazeed [Appendix D][devtools]<!-- ignore --> mein). Rust team ne is tool ko standard Rust distribution mein `rustc` ki tarah shamil kiya hai, is liye ye aapke computer par pehle se installed hona chahiye!

`main` function ki body mein following code hai:

```rust
println!("Hello, world!");
```

Ye line is chhote se program ka tamam kaam karti hai: Ye screen par text print karti hai. Yahan teen important details hain jin par gaur karna chahiye.

Pehli baat, `println!` ek Rust macro ko call karta hai. Agar ye ek function ko call karta, to ise `println` ke taur par likha jata (`!` ke baghair). Rust macros aisa code likhne ka tareeqa hain jo Rust syntax ko extend karne ke liye code generate karta hai, aur hum [Chapter 20][ch20-macros]<!-- ignore --> mein in par mazeed detail se baat karenge. Filhaal aapko sirf itna maloom hona chahiye ke `!` use karne ka matlab hai ke aap ek normal function ke bajaye macro ko call kar rahe hain, aur macros hamesha functions jaise same rules follow nahi karte.

Doosri baat, aap `"Hello, world!"` string dekh rahe hain. Hum is string ko `println!` ke argument ke taur par pass karte hain, aur string screen par print ho jati hai.

Teesri baat, hum line ko semicolon (`;`) par khatam karte hain, jo indicate karta hai ke ye expression khatam ho gaya hai aur agla expression shuru hone ke liye tayyar hai. Rust code ki zyada tar lines semicolon par khatam hoti hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="compiling-and-running-are-separate-steps"></a>

### Compilation and Execution

Aap ne abhi ek naya banaya hua program run kiya hai, to aaiye process ke har step ko examine karte hain.

Rust program run karne se pehle, aapko Rust compiler ka use karke ise compile karna hota hai. Is ke liye `rustc` command enter karein aur usay apni source file ka naam dein, jaise:

```console
$ rustc main.rs
```

Agar aapka background C ya C++ mein hai, to aap notice karenge ke ye `gcc` ya `clang` ke jaisa hai. Successfully compile hone ke baad, Rust ek binary executable output karta hai.

Linux, macOS, aur Windows par PowerShell mein, aap apni shell mein `ls` command enter karke executable dekh sakte hain:

```console
$ ls
main  main.rs
```

Linux aur macOS par aapko do files nazar aayengi. Windows par PowerShell use karte hue, aapko wohi teen files nazar aayengi jo CMD use karte hue nazar aati hain. Windows par CMD ke saath aap following enter karenge:

```cmd
> dir /B %= the /B option says to only show the file names =%
main.exe
main.pdb
main.rs
```

Ye source code file ko *.rs* extension ke saath, executable file ko (*main.exe* Windows par, lekin baqi tamam platforms par *main*), aur Windows use karte waqt *.pdb* extension wali debugging information contain karne wali file ko show karta hai. Yahan se aap *main* ya *main.exe* file ko is tarah run karte hain:

```console
$ ./main # or .\main on Windows
```

Agar aapka *main.rs* aapka “Hello, world!” program hai, to ye line aapke terminal mein `Hello, world!` print karegi.

Agar aap dynamic language, jaise Ruby, Python, ya JavaScript se zyada waqif hain, to shayad aap program ko compile aur run karne ko separate steps ke taur par karne ke aadi na hon. Rust ek *ahead-of-time compiled* language hai, jis ka matlab hai ke aap ek program ko compile karke uska executable kisi doosre shakhs ko de sakte hain, aur woh Rust install kiye baghair bhi ise run kar sakta hai. Agar aap kisi ko *.rb*, *.py*, ya *.js* file dein, to unhein respectively Ruby, Python, ya JavaScript ki implementation installed honi zaroori hai. Lekin un languages mein aapko apne program ko compile aur run karne ke liye sirf ek command ki zaroorat hoti hai. Language design mein har cheez ek trade-off hoti hai.

Simple programs ke liye sirf `rustc` se compile karna theek hai, lekin jaise jaise aapka project bara hota jata hai, aap tamam options ko manage karna aur apne code ko share karna aasaan banana chahenge. Ab hum aapko Cargo tool se introduce karenge, jo aapko real-world Rust programs likhne mein madad karega.

[troubleshooting]: ch01-01-installation.html#troubleshooting
[devtools]: appendix-04-useful-development-tools.html
[ch20-macros]: ch20-05-macros.html

