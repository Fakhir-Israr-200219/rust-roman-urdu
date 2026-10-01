## Installation

Pehla step Rust ko install karna hai. Hum Rust ko `rustup` ke zariye download karenge, jo Rust versions aur un se related tools ko manage karne ke liye ek command line tool hai. Download ke liye aapko internet connection ki zaroorat hogi.

> Note: Agar aap kisi wajah se `rustup` use nahi karna chahte, to mazeed options ke liye [Other Rust Installation Methods page][otherinstall] dekhein.

Neeche diye gaye steps Rust compiler ka latest stable version install karte hain. Rust ki stability guarantees ye ensure karti hain ke kitab mein diye gaye tamam examples jo compile hote hain, woh newer Rust versions ke saath bhi compile hote rahenge. Rust aksar error messages aur warnings ko improve karta rehta hai, is liye versions ke darmiyan output mein thora farq ho sakta hai. Doosre alfaaz mein, in steps ko use karke jo bhi newer, stable version of Rust aap install karenge, woh is kitab ke content ke saath expected tareeqe se kaam karega.


> ### Command Line Notation
>
> Is chapter mein aur poori kitab mein, hum kuch commands dikhayenge jo terminal mein use ki jati hain. Jo lines aapko terminal mein enter karni hain, woh sab `$` se start hoti hain. Aapko `$` character type karne ki zaroorat nahi hai; ye command line prompt hai jo har command ke start ko indicate karta hai. Jo lines `$` se start nahi hoti, woh aam tor par previous command ka output show karti hain. Is ke ilawa, PowerShell-specific examples mein `$` ke bajaye `>` use kiya jayega.


### Installing `rustup` on Linux or macOS

Agar aap Linux ya macOS use kar rahe hain, to terminal open karein aur following command enter karein:

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

Ye command ek script download karti hai aur `rustup` tool ki installation start karti hai, jo Rust ka latest stable version install karta hai. Aapse aapka password poocha ja sakta hai. Agar installation successful ho, to following line appear hogi:

```text
Rust is installed now. Great!
```

Aapko ek *linker* ki bhi zaroorat hogi, jo ek aisa program hai jise Rust apne compiled outputs ko ek file mein join karne ke liye use karta hai. Mumkin hai ke aapke paas pehle se hi ek linker ho. Agar aapko linker errors milte hain, to aapko C compiler install karna chahiye, jisme aam tor par linker bhi shamil hota hai. C compiler is liye bhi useful hai kyun ke kuch common Rust packages C code par depend karte hain aur unhein C compiler ki zaroorat hogi.

macOS par aap C compiler following command run karke hasil kar sakte hain:

```console
$ xcode-select --install
```

Linux users ko aam tor par apni distribution ki documentation ke mutabiq GCC ya Clang install karna chahiye. Misal ke taur par, agar aap Ubuntu use karte hain, to aap `build-essential` package install kar sakte hain.


### Installing `rustup` on Windows

Windows par [https://www.rust-lang.org/tools/install][install]<!-- ignore
--> par jayein aur Rust install karne ke liye instructions follow karein. Installation ke kisi stage par aapse Visual Studio install karne ke liye kaha jayega. Ye ek linker aur woh native libraries provide karta hai jo programs compile karne ke liye zaroori hoti hain. Agar aapko is step ke liye mazeed madad chahiye, to
[https://rust-lang.github.io/rustup/installation/windows-msvc.html][msvc]<!--
ignore --> dekhein.


Is kitab ka baqi hissa aisi commands use karta hai jo *cmd.exe* aur PowerShell dono mein kaam karti hain. Agar koi specific differences honge, to hum batayenge ke kaunsi command use karni hai.

### Troubleshooting

Ye check karne ke liye ke Rust sahi tarah install hua hai ya nahi, ek shell open karein aur ye line enter karein:

```console
$ rustc --version
```

Aapko latest released stable version ka version number, commit hash, aur commit date following format mein nazar aani chahiye:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

Agar aapko ye information nazar aa rahi hai, to aap ne Rust successfully install kar liya hai! Agar ye information nazar nahi aa rahi, to check karein ke Rust aapke `%PATH%` system variable mein following tareeqe se maujood hai.

Windows CMD mein, use karein:

```console
> echo %PATH%
```

PowerShell mein, use karein:

```powershell
> echo $env:Path
```

Linux aur macOS mein, use karein:

```console
$ echo $PATH
```

Agar ye sab theek hai aur phir bhi Rust kaam nahi kar raha, to aap help hasil karne ke liye kai jagahon se rabta kar sakte hain. Doosre Rustaceans se contact karne ka tareeqa (Rustaceans hamara apne liye use kiya jane wala ek mazaahiya nickname hai) [the community page][community] par dekhein.

### Updating and Uninstalling

Jab Rust `rustup` ke zariye install ho jaye, to naye release kiye gaye version par update karna aasaan hai. Apni shell se following update script run karein:

```console
$ rustup update
```

Rust aur `rustup` ko uninstall karne ke liye, apni shell se following uninstall script run karein:

```console
$ rustup self uninstall
```

<!-- Old headings. Do not remove or links may break. -->

<a id="local-documentation"></a>

### Reading the Local Documentation

Rust ki installation mein documentation ki ek local copy bhi shamil hoti hai, taake aap ise offline parh saken. Local documentation ko apne browser mein kholne ke liye `rustup doc` run karein.

Jab bhi koi type ya function standard library provide karti ho aur aapko yaqeen na ho ke woh kya karta hai ya use kaise karna hai, to maloomat hasil karne ke liye application programming interface (API) documentation use karein!

<!-- Old headings. Do not remove or links may break. -->

<a id="text-editors-and-integrated-development-environments"></a>

### Using Text Editors and IDEs

Ye kitab is baat ke baare mein koi assumption nahi karti ke aap Rust code likhne ke liye kaun se tools use karte hain. Lagbhag koi bhi text editor kaam kar dega! Lekin, bohat se text editors aur integrated development environments (IDEs) mein Rust ke liye built-in support mojood hai. Aap Rust website ke [tools page][tools] par editors aur IDEs ki ek kaafi current list hamesha dekh sakte hain.


### Working Offline with This Book

Kayi examples mein hum standard library ke ilawa Rust packages use karenge. In examples par kaam karne ke liye aapke paas ya to internet connection hona chahiye ya aapne un dependencies ko pehle se download kiya hua hona chahiye. Dependencies ko pehle se download karne ke liye aap following commands run kar sakte hain. (`cargo` kya hai aur in mein se har command kya karti hai, hum baad mein detail mein explain karenge.)

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:

* ch02-00-guessing-game-tutorial.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

```console
$ cargo new get-dependencies
$ cd get-dependencies
$ cargo add rand@0.10.1 trpl@0.2.0
```

Ye packages ke downloads ko cache kar dega, is liye baad mein aapko inhein dobara download karne ki zaroorat nahi hogi. Jab aap ye command run kar chuke hon, to aapko `get-dependencies` folder rakhne ki zaroorat nahi hai. Agar aapne ye command run kar li hai, to kitab ke baqi hisson mein tamam `cargo` commands ke saath `--offline` flag use kar sakte hain, taake network use karne ki koshish karne ke bajaye ye cached versions use karein.

[otherinstall]: https://forge.rust-lang.org/infra/other-installation-methods.html
[install]: https://www.rust-lang.org/tools/install
[msvc]: https://rust-lang.github.io/rustup/installation/windows-msvc.html
[community]: https://www.rust-lang.org/community
[tools]: https://www.rust-lang.org/tools
