# The Rust Programming Language — Roman Urdu

Ye repository **"The Rust Programming Language"** kitab ke Roman Urdu tarjume ka source hai.

Ye translation Roman Urdu mein hai, yani Urdu zaban ko Roman/Latin script mein likha gaya hai.

Original kitab [The Rust Programming Language](https://doc.rust-lang.org/book/?utm_source=chatgpt.com) hai.

## Requirements

Kitab ko build karne ke liye [mdBook] darkar hai. Behtar hai ke wohi mdBook version use kiya jaye jo `rust-lang/rust` use karta hai.

mdBook install karne ke liye:

```bash
cargo install mdbook --locked --version <version_num>
```

[mdBook]: https://github.com/rust-lang/mdBook

## Building

Book ko build karne ke liye:

```bash
mdbook build
```

Build hone ke baad output `book` directory mein milega.

Book ko local browser mein dekhne ke liye `book/index.html` open karein.

### Windows — PowerShell

```powershell
Start-Process "firefox.exe" .\book\index.html
```

ya:

```powershell
Start-Process "chrome.exe" .\book\index.html
```

Aap development ke dauran ye command bhi use kar sakte hain:

```bash
mdbook serve
```

Phir browser mein:

```text
http://localhost:3000
```

open karein.

## Contributing

Agar aap is Roman Urdu translation mein contribute karna chahte hain, to repository ko fork karke changes kar sakte hain aur pull request submit kar sakte hain.

Rust Book ke official translation efforts ke baare mein mazeed maloomat ke liye [Translations] label dekhein.

[Translations]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3ATranslations

## Translation

Is project ka maqsad **The Rust Programming Language** ko Roman Urdu readers ke liye accessible banana hai.

Translation ke dauran:

* Rust ke technical terms ko jahan zaroori ho English mein hi rakha jayega.
* Rust code examples ko translate nahi kiya jayega.
* Code blocks aur commands ko original form mein rakha jayega.
* Explanation ko natural aur samajhne mein aasaan Roman Urdu mein translate kiya jayega.
* Technical meaning ko original kitab ke qareeb rakha jayega.

Ye translation **official Rust Book ka replacement nahi** hai. Original English version Rust community ki official reference hai.

## License

Ye project Rust Book ke upstream source aur uski licensing requirements ko follow karta hai.

Original source:

[rust-lang/book](https://github.com/rust-lang/book?utm_source=chatgpt.com)
