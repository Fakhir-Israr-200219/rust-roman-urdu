## Comments

Tamam programmers apne code ko samajhne mein aasaan banane ki koshish karte hain, lekin kabhi kabhi extra explanation dena zaroori hota hai. In situations mein, programmers apne source code mein *comments* likhte hain jinhein compiler ignore karta hai, lekin jo log source code parh rahe hote hain unke liye useful ho sakte hain.

Yahan ek simple comment hai:

```rust
// hello, world
```

Rust mein idiomatic comment style do slashes se comment start karti hai, aur comment line ke end tak continue hota hai. Aise comments jo ek single line se zyada extend hon, unke liye aapko har line par `//` include karna hoga, is tarah:

```rust
// So we're doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what's going on.
```

Comments ko code wali lines ke end par bhi place kiya ja sakta hai:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```

Lekin zyada tar aap unhein is format mein dekhenge, jahan comment us code ke upar ek separate line par hota hai jis ke baare mein woh explanation de raha hota hai:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```

Rust mein ek aur qisam ka comment bhi hota hai, documentation comments, jinhein hum Chapter 14 ke [“Publishing a Crate to Crates.io”][publishing]<!-- ignore --> section mein discuss karenge.

[publishing]: ch14-02-publishing-to-crates-io.html
