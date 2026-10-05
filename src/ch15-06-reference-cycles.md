## Reference Cycles Memory Leak Kar Sakte Hain

Rust ki memory safety guarantees ki wajah se aisi memory accidentally create karna mushkil hai, lekin
namumkin nahi, jo kabhi clean up na ho (jise *memory leak* kaha jata hai).
Memory leaks ko completely prevent karna Rust ki guarantees mein shamil nahi hai, yani
memory leaks Rust mein memory safe hain. Hum dekh sakte hain ke Rust `Rc<T>` aur
`RefCell<T>` ko use karke memory leaks allow karta hai: Aise references create karna
mumkin hai jahan items ek doosre ko ek cycle mein refer karte hain. Is se memory leaks
create hote hain kyun ke cycle mein har item ka reference count kabhi bhi 0 tak nahi pohanchega,
aur values kabhi drop nahi hongi.

### Reference Cycle Create Karna

Aaiye dekhte hain ke reference cycle kis tarah ho sakti hai aur ise kaise prevent kiya ja sakta hai,
Listing 15-25 mein `List` enum ki definition aur ek `tail` method se shuru karte hue.

<Listing number="15-25" file-name="src/main.rs" caption="Ek cons list definition jo `RefCell<T>` hold karti hai taake hum modify kar saken ke `Cons` variant kis cheez ko refer kar raha hai">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-25/src/main.rs:here}}
```

</Listing>

Hum Listing 15-5 ki `List` definition ka ek aur variation use kar rahe hain. `Cons`
variant ka second element ab `RefCell<Rc<List>>` hai, jis ka matlab hai ke
Listing 15-24 ki tarah `i32` value ko modify karne ki ability rakhne ke bajaye,
hum us `List` value ko modify karna chahte hain jis ki taraf `Cons` variant point kar raha hai.
Hum ek `tail` method bhi add kar rahe hain taake jab hamare paas `Cons` variant ho to
humare liye second item tak access karna convenient ho.

Listing 15-26 mein hum ek `main` function add kar rahe hain jo Listing 15-25 mein
di gayi definitions ko use karta hai. Yeh code `a` mein ek list aur `b` mein ek list
create karta hai jo `a` wali list ko point karti hai. Phir, yeh `a` wali list ko modify
karke use `b` ki taraf point karwata hai, jis se ek reference cycle create hoti hai.
Is process ke mukhtalif points par reference counts kya hain, yeh dikhane ke liye
raaste mein `println!` statements bhi hain.

<Listing number="15-26" file-name="src/main.rs" caption="Do `List` values ki ek reference cycle create karna jo ek doosre ko point karti hain">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-26/src/main.rs:here}}
```

</Listing>

Hum variable `a` mein ek `Rc<List>` instance create karte hain jo ek `List` value
hold karta hai, jismein shuru mein `5, Nil` ki list hoti hai. Phir hum variable `b`
mein ek aur `Rc<List>` instance create karte hain jo ek aur `List` value hold karta hai
jismein value `10` hoti hai aur jo `a` wali list ko point karti hai.

Hum `a` ko modify karte hain taake woh `Nil` ke bajaye `b` ko point kare, jis se ek
cycle create hoti hai. Hum yeh `tail` method ko use karke `a` mein mojood
`RefCell<Rc<List>>` ka ek reference hasil karne se karte hain, jise hum variable
`link` mein rakhte hain. Phir hum `RefCell<Rc<List>>` par `borrow_mut` method use
karke andar ki value ko ek aise `Rc<List>` se, jo `Nil` value hold karta hai, badal kar
`b` mein mojood `Rc<List>` kar dete hain.

Jab hum is code ko run karte hain, aur filhaal aakhri `println!` ko commented out
rakhte hain, to humein yeh output milega:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-26/output.txt}}
```

`a` aur `b` dono mein mojood `Rc<List>` instances ka reference count `a` wali list ko
`b` ki taraf point karne ke baad 2 ho jata hai. `main` ke end par Rust variable `b`
ko drop karta hai, jis se `b` wale `Rc<List>` instance ka reference count 2 se 1 ho jata hai.
Is point par `Rc<List>` ke paas heap par mojood memory drop nahi hogi kyun ke iska
reference count 1 hai, 0 nahi. Phir Rust `a` ko drop karta hai, jis se `a` wale
`Rc<List>` instance ka reference count bhi 2 se 1 ho jata hai. Is instance ki memory
bhi drop nahi ki ja sakti, kyun ke doosra `Rc<List>` instance abhi bhi ise refer kar raha hai.
List ke liye allocate ki gayi memory hamesha ke liye uncollected rahegi. Is reference
cycle ko visualize karne ke liye humne Figure 15-4 mein diagram create kiya hai.

<img alt="Ek rectangle jis par 'a' label hai aur jo ek aise rectangle ki taraf point karta hai jisme integer 5 hai. Ek rectangle jis par 'b' label hai aur jo ek aise rectangle ki taraf point karta hai jisme integer 10 hai. 5 wala rectangle 10 wale rectangle ki taraf point karta hai, aur 10 wala rectangle wapas 5 wale rectangle ki taraf point karta hai, jis se ek cycle create hoti hai." src="img/trpl15-04.svg" class="center" />

<span class="caption">Figure 15-4: Lists `a` aur `b` ki ek reference cycle
jo ek doosre ko point karti hain</span>

Agar aap aakhri `println!` ko uncomment karke program run karein, to Rust is cycle ko
print karne ki koshish karega, jahan `a`, `b` ko point karta hai, jo `a` ko point karta hai,
aur isi tarah aage, jab tak stack overflow nahi ho jata.

Ek real-world program ke muqable mein, is example mein reference cycle create karne ke
consequences bohot serious nahi hain: Reference cycle create karne ke foran baad hi
program end ho jata hai. Lekin agar koi zyada complex program cycle mein bohot saari
memory allocate kare aur use lambe waqt tak hold karke rakhe, to program apni zaroorat se
zyada memory use karega aur system ko overwhelm kar sakta hai, jis ki wajah se available
memory khatam ho sakti hai.

Reference cycles create karna aasaan nahi hai, lekin yeh namumkin bhi nahi hai.
Agar aapke paas `RefCell<T>` values hain jo `Rc<T>` values ya isi tarah ke nested
combinations of types with interior mutability aur reference counting contain karti hain,
to aapko ensure karna hoga ke aap cycles create na karein; aap Rust par inhein catch karne
ke liye rely nahi kar sakte. Reference cycle create karna aapke program mein ek logic bug
hoga, jise minimize karne ke liye aapko automated tests, code reviews, aur doosri software
development practices use karni chahiye.

Reference cycles avoid karne ka ek aur solution yeh hai ke apni data structures ko
reorganize kiya jaye taake kuch references ownership express karein aur kuch references
ownership express na karein. Is ke result mein aapke paas kuch ownership relationships
aur kuch non-ownership relationships se bani hui cycles ho sakti hain, aur sirf ownership
relationships is baat par effect dalti hain ke koi value drop ki ja sakti hai ya nahi.
Listing 15-25 mein hum hamesha chahte hain ke `Cons` variants apni list ki ownership
rakhein, is liye data structure ko reorganize karna possible nahi hai. Aaiye parent nodes
aur child nodes se bani hui graphs ki example dekhein taake samajh saken ke reference cycles
ko prevent karne ke liye non-ownership relationships kab appropriate hoti hain.

<!-- Old headings. Do not remove or links may break. -->

<a id="preventing-reference-cycles-turning-an-rct-into-a-weakt"></a>

### `Weak<T>` ko Use Karke Reference Cycles Prevent Karna

Ab tak humne demonstrate kiya hai ke `Rc::clone` call karne se kisi
`Rc<T>` instance ka `strong_count` increase hota hai, aur `Rc<T>` instance tabhi clean
up hota hai jab uska `strong_count` 0 ho. Aap `Rc::downgrade` call karke aur
`Rc<T>` ka reference pass karke `Rc<T>` instance ke andar mojood value ka weak reference
bhi create kar sakte hain. *Strong references* woh references hain jin ke zariye aap
`Rc<T>` instance ki ownership share kar sakte hain. *Weak references* ownership
relationship express nahi karte, aur inki count is baat par effect nahi karti ke
`Rc<T>` instance kab clean up hoga. Yeh reference cycle cause nahi karenge, kyun ke
kisi bhi aisi cycle mein jismein kuch weak references shamil hon, involved values ka
strong reference count 0 hote hi cycle break ho jayegi.

Jab aap `Rc::downgrade` call karte hain, to aapko `Weak<T>` type ka smart pointer milta hai.
`Rc<T>` instance mein `strong_count` ko 1 se increase karne ke bajaye, `Rc::downgrade`
call karne se `weak_count` 1 se increase hota hai. `Rc<T>` type `weak_count` ko
track karne ke liye use karta hai ke kitne `Weak<T>` references exist karte hain,
bilkul `strong_count` ki tarah. Farq yeh hai ke `Rc<T>` instance ko clean up hone ke
liye `weak_count` ka 0 hona zaroori nahi hai.

Kyun ke jis value ko `Weak<T>` reference karta hai woh shayad drop ho chuki ho, is liye
`Weak<T>` jis value ki taraf point kar raha hai uske saath kuch bhi karne ke liye aapko
pehle ensure karna hoga ke woh value abhi exist karti hai. Is ke liye `Weak<T>` instance
par `upgrade` method call karein, jo `Option<Rc<T>>` return karega. Agar `Rc<T>` value
abhi drop nahi hui to aapko `Some` ka result milega aur agar `Rc<T>` value drop ho chuki
hai to `None` ka result milega. Kyun ke `upgrade` ek `Option<Rc<T>>` return karta hai,
Rust ensure karega ke `Some` aur `None` dono cases handle kiye jayen, aur koi invalid
pointer nahi hoga.

Misal ke taur par, aisi list use karne ke bajaye jiske items sirf next item ke baare mein
jaante hain, hum ek tree create karenge jiske items apne child items *aur* parent items
ke baare mein jaante hon.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-tree-data-structure-a-node-with-child-nodes"></a>

#### Tree Data Structure Create Karna

Shuru karne ke liye, hum aisa tree build karenge jiske nodes apne child nodes ke baare mein
jaante hon. Hum `Node` naam ka ek struct create karenge jo apni `i32` value ke saath-saath
apne child `Node` values ke references bhi hold karega:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:here}}
```

Hum chahte hain ke ek `Node` apne children ka owner ho, aur hum is ownership ko variables ke
saath share karna chahte hain taake hum tree ke har `Node` ko directly access kar saken. Is ke
liye, hum `Vec<T>` ke items ko `Rc<Node>` type ki values define karte hain. Hum yeh bhi chahte hain
ke hum modify kar saken ke kaun se nodes kisi doosre node ke children hain, is liye `children`
mein `Vec<Rc<Node>>` ke around ek `RefCell<T>` hai.

Ab hum apni struct definition ko use karenge aur ek `leaf` naam ka `Node` instance create
karेंगे jis ki value `3` hai aur koi children nahi hain, aur ek aur instance `branch` naam ka
create karenge jis ki value `5` hai aur `leaf` uske children mein se ek hai, jaisa ke Listing
15-27 mein dikhaya gaya hai.

<Listing number="15-27" file-name="src/main.rs" caption="Bina children wale `leaf` node aur `leaf` ko apne children mein rakhne wale `branch` node ko create karna">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:there}}
```

</Listing>

Hum `leaf` mein mojood `Rc<Node>` ko clone karke usay `branch` mein store karte hain, jis ka
matlab hai ke `leaf` mein mojood `Node` ke ab do owners hain: `leaf` aur `branch`. Hum
`branch.children` ke zariye `branch` se `leaf` tak ja sakte hain, lekin `leaf` se `branch` tak
jaane ka koi tareeqa nahi hai. Is ki wajah yeh hai ke `leaf` ka `branch` ke saath koi reference
nahi hai aur woh nahi jaanta ke dono related hain. Hum chahte hain ke `leaf` ko pata ho ke
`branch` uska parent hai. Ab hum yeh karenge.

#### Child se Uske Parent ka Reference Add Karna

Child node ko apne parent ke baare mein aware karne ke liye, humein apni `Node` struct
definition mein ek `parent` field add karni hogi. Mushkil yeh decide karne mein hai ke
`parent` ka type kya hona chahiye. Hum jaante hain ke ismein `Rc<T>` nahi ho sakta, kyun ke
is se `leaf.parent` ka `branch` ki taraf point karna aur `branch.children` ka `leaf` ki
taraf point karna ek reference cycle create karega, jis ki wajah se unki `strong_count`
values kabhi 0 nahi hongi.

In relationships ko ek doosre tareeqe se dekhein to parent node ko apne children ka owner
hona chahiye: Agar parent node drop ho jaye, to uske child nodes bhi drop ho jane chahiye.
Lekin child ko apne parent ka owner nahi hona chahiye: Agar hum child node ko drop karein,
to parent phir bhi exist karna chahiye. Yeh weak references ke liye ek case hai!

Is liye, `Rc<T>` ke bajaye, hum `parent` ka type `Weak<T>` use karenge,
specifically `RefCell<Weak<Node>>`. Ab hamari `Node` struct definition kuch is tarah
dikhai degi:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:here}}
```

Ek node apne parent node ko refer kar sakega, lekin apne parent ka owner nahi hoga. Listing
15-28 mein hum `main` ko update karte hain taake is nayi definition ko use kiya ja sake aur
`leaf` node ke paas apne parent, `branch`, ko refer karne ka tareeqa ho.

<Listing number="15-28" file-name="src/main.rs" caption="Ek `leaf` node jismein apne parent node, `branch`, ka weak reference hai">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:there}}
```

</Listing>

`leaf` node create karna Listing 15-27 jaisa hi hai, sirf `parent` field mein farq hai:
`leaf` shuru mein bina parent ke hota hai, is liye hum ek naya, empty `Weak<Node>` reference
instance create karte hain.

Is point par, jab hum `upgrade` method use karke `leaf` ke parent ka reference hasil karne ki
koshish karte hain, to humein `None` value milti hai. Hum isay pehle `println!` statement ke
output mein dekh sakte hain:

```text
leaf parent = None
```

Jab hum `branch` node create karte hain, to iske `parent` field mein bhi ek naya
`Weak<Node>` reference hoga kyun ke `branch` ka koi parent node nahi hai. Hamare paas ab bhi
`branch` ke children mein se ek ke taur par `leaf` hai. Jab hamare paas `branch` mein
`Node` instance aa jata hai, to hum `leaf` ko modify kar sakte hain taake uske paas apne
parent ka ek `Weak<Node>` reference ho. Hum `leaf` ke `parent` field mein mojood
`RefCell<Weak<Node>>` par `borrow_mut` method use karte hain, aur phir `branch` mein
mojood `Rc<Node>` se `branch` ka `Weak<Node>` reference create karne ke liye
`Rc::downgrade` function use karte hain.

Jab hum dobara `leaf` ka parent print karte hain, to is baar humein `branch` ko hold karne
wala ek `Some` variant milega: Ab `leaf` apne parent ko access kar sakta hai! Jab hum
`leaf` ko print karte hain, to hum us cycle se bhi bach jate hain jo aakhir mein stack
overflow par khatam hoti thi, jaisa ke Listing 15-26 mein hua tha; `Weak<Node>` references
`(Weak)` ke taur par print hote hain:

```text
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

Infinite output ka na hona indicate karta hai ke is code ne reference cycle create nahi ki.
Hum is baat ko `Rc::strong_count` aur `Rc::weak_count` call karne se milne wali values ko
dekh kar bhi samajh sakte hain.

#### `strong_count` aur `weak_count` mein Changes ko Visualize Karna

Aaiye dekhte hain ke `Rc<Node>` instances ki `strong_count` aur `weak_count` values kis tarah
change hoti hain, ek naya inner scope create karke aur `branch` ki creation ko us scope mein
move karke. Aisa karne se hum dekh sakte hain ke `branch` create hone aur phir scope se bahar
nikalne par drop hone ke waqt kya hota hai. Yeh modifications Listing 15-29 mein dikhayi gayi hain.

<Listing number="15-29" file-name="src/main.rs" caption="Ek inner scope mein `branch` create karna aur strong aur weak reference counts ko examine karna">

```rust id="r4x9kp"
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-29/src/main.rs:here}}
```

</Listing>

`leaf` create hone ke baad, uske `Rc<Node>` ka strong count 1 aur weak count 0 hota hai. Inner
scope mein hum `branch` create karte hain aur use `leaf` ke saath associate karte hain. Is point
par jab hum counts print karte hain, to `branch` mein mojood `Rc<Node>` ka strong count 1 aur
weak count 1 hoga (`leaf.parent` ke `Weak<Node>` ke saath `branch` ki taraf point karne ki wajah
se). Jab hum `leaf` mein counts print karenge, to hum dekhenge ke iska strong count 2 ho gaya hai
kyun ke `branch` ke paas ab `leaf` ke `Rc<Node>` ka ek clone `branch.children` mein stored hai,
lekin iska weak count ab bhi 0 hoga.

Jab inner scope khatam hota hai, `branch` scope se bahar chala jata hai aur `Rc<Node>` ka strong
count 0 ho jata hai, is liye iska `Node` drop ho jata hai. `leaf.parent` se aane wale weak count
1 ka `Node` ke drop hone par koi asar nahi hota, is liye humein koi memory leaks nahi milte!

Agar hum scope khatam hone ke baad `leaf` ke parent ko access karne ki koshish karein, to humein
dobara `None` milega. Program ke end par, `leaf` mein mojood `Rc<Node>` ka strong count 1 aur
weak count 0 hota hai kyun ke ab variable `leaf` dobara `Rc<Node>` ka sirf ek reference hai.

Counts ko manage karne aur values ko drop karne wali tamam logic `Rc<T>` aur `Weak<T>` mein aur
unke `Drop` trait ki implementations mein built-in hai. `Node` ki definition mein yeh specify
karke ke child se parent ka relationship ek `Weak<T>` reference hona chahiye, aap parent nodes ko
child nodes ki taraf aur child nodes ko parent nodes ki taraf point karwa sakte hain, bina
reference cycle aur memory leaks create kiye.

## Summary

Is chapter mein humne dekha ke smart pointers ko kaise use kiya ja sakta hai taake
un guarantees aur trade-offs ko provide kiya ja sake jo Rust regular references ke
saath default taur par nahi deta. `Box<T>` type ka size known hota hai aur yeh heap par
allocate kiye gaye data ki taraf point karta hai. `Rc<T>` type heap par mojood data ke
references ki tadaad ko track karta hai taake data ke multiple owners ho saken.
`RefCell<T>` type apni interior mutability ke saath humein aisi type deta hai jise hum
us waqt use kar sakte hain jab humein immutable type chahiye ho lekin us type ki inner
value ko change karna ho; yeh borrowing rules ko compile time ke bajaye runtime par
enforce bhi karta hai.

Humne `Deref` aur `Drop` traits par bhi baat ki, jo smart pointers ki bohot si
functionality ko enable karte hain. Humne reference cycles ko explore kiya jo memory
leaks ka sabab ban sakti hain, aur yeh bhi dekha ke `Weak<T>` ko use karke unhein kaise
prevent kiya ja sakta hai.

Agar is chapter ne aapki interest barha di hai aur aap apne smart pointers implement
karna chahte hain, to mazeed useful information ke liye [“The Rustonomicon”][nomicon]
dekhein.

Agla chapter mein hum Rust mein concurrency ke baare mein baat karenge. Aap kuch naye
smart pointers ke baare mein bhi seekhenge.

[nomicon]: ../nomicon/index.html
