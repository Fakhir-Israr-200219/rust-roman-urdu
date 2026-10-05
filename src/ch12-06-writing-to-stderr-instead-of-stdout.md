<!-- Old headings. Do not remove or links may break. -->

<a id="writing-error-messages-to-standard-error-instead-of-standard-output"></a>

## Redirecting Errors to Standard Error

Filhaal, hum apne tamam output ko terminal par `println!` macro ka use karke likh rahe hain. Aksar terminals mein do qisam ka output hota hai: *standard output* (`stdout`) general information ke liye aur *standard error* (`stderr`) error messages ke liye. Yeh distinction users ko yeh choose karne ki sahulat deti hai ke woh program ke successful output ko ek file mein direct karein, lekin error messages ko phir bhi screen par print karein.

`println!` macro sirf standard output par print karne ki capability rakhta hai, is liye standard error par print karne ke liye humein kuch aur use karna hoga.

### Checking Where Errors Are Written

Sab se pehle, aaiye observe karte hain ke `minigrep` se print hone wala content filhaal standard output mein kaise write ho raha hai, jin error messages ko hum iske bajaye standard error mein write karna chahte hain unhein bhi include karte hue. Hum yeh standard output stream ko ek file par redirect karke karenge, jabke jaan boojh kar ek error cause karenge. Hum standard error stream ko redirect nahi karenge, is liye standard error ko bheja gaya koi bhi content screen par display hota rahega.

Command line programs se expect kiya jata hai ke woh error messages standard error stream ko bhejein taake agar hum standard output stream ko ek file par redirect bhi kar dein, tab bhi hum screen par error messages dekh sakein. Hamara program filhaal theek tarah behave nahi kar raha: hum ab dekhenge ke yeh error message ke output ko bhi ek file mein save kar deta hai!

Is behavior ko demonstrate karne ke liye, hum program ko `>` aur file path *output.txt* ke saath run karenge, jahan hum standard output stream ko redirect karna chahte hain. Hum koi arguments pass nahi karenge, jis se ek error cause hona chahiye:

```console
$ cargo run > output.txt
```

`>` syntax shell ko batata hai ke standard output ke contents ko screen ke bajaye *output.txt* mein write kare. Humein woh error message screen par print hota hua nazar nahi aaya jiski hum expectation kar rahe thay, is liye iska matlab hai ke woh file mein chala gaya hoga. *output.txt* mein yeh content hai:

```text
Problem parsing arguments: not enough arguments
```

Yup, hamara error message standard output par print ho raha hai. Aise error messages ka standard error par print hona zyada useful hai taake sirf successful run ka data hi file mein jaye. Hum isay change karenge.

### Printing Errors to Standard Error

Hum error messages ko print karne ke tareeqe ko change karne ke liye Listing 12-24 mein diye gaye code ko use karenge. Is chapter mein pehle ki gayi refactoring ki wajah se, error messages print karne wala tamam code ek hi function, `main`, mein hai. Standard library `eprintln!` macro provide karti hai jo standard error stream par print karta hai, is liye jahan hum errors print karne ke liye `println!` call kar rahe thay, un dono jagahon ko `eprintln!` use karne ke liye change karte hain.

<Listing number="12-24" file-name="src/main.rs" caption="Writing error messages to standard error instead of standard output using `eprintln!`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-24/src/main.rs:here}}
```

</Listing>

Ab aaiye program ko dobara isi tarah run karte hain, bina kisi arguments ke aur `>` ke zariye standard output ko redirect karte hue:

```console
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

Ab humein error screen par nazar aa raha hai aur *output.txt* mein kuch bhi nahi hai, jo command line programs se hamari expectation ke mutabiq behavior hai.

Ab aaiye program ko dobara aise arguments ke saath run karte hain jo error cause nahi karte lekin phir bhi standard output ko ek file mein redirect karte hain, jaise:

```console
$ cargo run -- to poem.txt > output.txt
```

Humein terminal par koi output nazar nahi aayega, aur *output.txt* mein hamare results honge:

<span class="filename">Filename: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

Yeh demonstrate karta hai ke ab hum successful output ke liye standard output aur error output ke liye standard error ko appropriate taur par use kar rahe hain.

## Summary

Is chapter mein humne ab tak seekhe gaye kuch major concepts ko recap kiya aur dekha ke Rust mein common I/O operations kaise perform kiye jate hain. Command line arguments, files, environment variables, aur errors print karne ke liye `eprintln!` macro ko use karke, ab aap command line applications likhne ke liye tayyar hain. Previous chapters ke concepts ke saath milkar, aapka code achhi tarah organized hoga, data ko appropriate data structures mein effectively store karega, errors ko achhi tarah handle karega, aur achhi tarah tested hoga.

Next, hum Rust ke kuch aise features explore karenge jo functional languages se influenced hain: closures aur iterators.
