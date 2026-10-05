## Appendix C: Derivable Traits

Book ke mukhtalif maqamat par humne `derive` attribute discuss kiya hai, jo aap
struct ya enum definition par apply kar sakte hain. `derive` attribute aisa
code generate karta hai jo us type par, jis par aapne `derive` syntax lagayi
hai, kisi trait ko uski apni default implementation ke saath implement karta
hai.

Is appendix mein hum standard library ke un tamam traits ka reference provide
karte hain jinhein aap `derive` ke saath use kar sakte hain. Har section mein
yeh cover kiya gaya hai:

* Is trait ko derive karne se kaun se operators aur methods enable honge
* `derive` ki taraf se provide ki gayi trait implementation kya karti hai
* Is trait ko implement karna type ke bare mein kya signify karta hai
* Kin conditions mein aapko trait implement karne ki ijazat hoti hai ya nahi hoti
* Kin operations ke liye is trait ki zaroorat hoti hai

Agar aap `derive` attribute ki taraf se provide kiye gaye behavior se different
behavior chahte hain, to har trait ke liye details jaanne ke liye
[standard library documentation](../std/index.html)<!-- ignore --> dekhein ke
unhein manually kaise implement kiya jaye.

Yahan listed traits hi standard library mein defined woh tamam traits hain
jinhein aap `derive` use karke apni types par implement kar sakte hain. Standard
library mein defined doosre traits ka sensible default behavior nahi hota, is
liye unhein aapko khud us tarah implement karna hota hai jo aapke desired
purpose ke liye munasib ho.

Ek aise trait ki example jo derive nahi kiya ja sakta `Display` hai, jo end
users ke liye formatting handle karta hai. Aapko hamesha is baat par ghaur
karna chahiye ke end user ko kisi type ko display karne ka appropriate tareeqa
kya hona chahiye. Type ke kin parts ko end user ko dekhne ki ijazat honi chahiye?
Kin parts ko woh relevant samjhenge? Data ka kaunsa format unke liye sab se
relevant hoga? Rust compiler ke paas yeh insight nahi hoti, is liye woh aapke
liye appropriate default behavior provide nahi kar sakta.

Is appendix mein diye gaye derivable traits ki list comprehensive nahi hai:
Libraries apne khud ke traits ke liye `derive` implement kar sakti hain, jis ki
wajah se un traits ki list jinke saath aap `derive` use kar sakte hain waqai
open ended hai. `derive` implement karne mein procedural macro ka use hota hai,
jise Chapter 20 ke [“Custom `derive`
Macros”][custom-derive-macros]<!-- ignore --> section mein cover kiya gaya hai.

### `Debug` for Programmer Output

`Debug` trait format strings mein debug formatting enable karta hai, jise aap
`{}` placeholders ke andar `:?` add karke indicate karte hain.

`Debug` trait aapko kisi type ke instances ko debugging purposes ke liye print
karne deta hai, taake aap aur aapki type use karne wale doosre programmers
program ki execution ke kisi particular point par kisi instance ka inspection
kar saken.

Misal ke taur par, `assert_eq!` macro ko use karte waqt `Debug` trait required
hota hai. Agar equality assertion fail ho jaye to yeh macro arguments ke taur
par diye gaye instances ki values print karta hai, taake programmers dekh saken
ke dono instances equal kyun nahi thay.

### `PartialEq` and `Eq` for Equality Comparisons

`PartialEq` trait aapko kisi type ke instances ko equality check karne ke liye
compare karne deta hai aur `==` aur `!=` operators ke use ko enable karta hai.

`PartialEq` ko derive karne se `eq` method implement hota hai. Jab structs par
`PartialEq` derive kiya jata hai, to do instances sirf us waqt equal hotay hain
jab *tamam* fields equal hon, aur agar *koi bhi* field equal na ho to instances
equal nahi hotay. Jab enums par derive kiya jata hai, to har variant khud ke
saath equal hota hai aur doosre variants ke saath equal nahi hota.

`PartialEq` trait required hota hai, misal ke taur par, `assert_eq!` macro ke
saath, jise kisi type ke do instances ko equality ke liye compare karne ke
qabil hona zaroori hai.

`Eq` trait ka koi method nahi hota. Is ka purpose yeh signal karna hai ke
annotated type ki har value ke liye, woh value khud ke barabar hai. `Eq` trait
sirf un types par apply kiya ja sakta hai jo `PartialEq` bhi implement karti
hon, halanke `PartialEq` implement karne wali tamam types `Eq` implement nahi
kar sakti. Is ki ek example floating-point number types hain: floating-point
numbers ki implementation ke mutabiq not-a-number (`NaN`) value ke do
instances ek doosre ke barabar nahi hotay.

Ek example jahan `Eq` required hota hai `HashMap<K, V>` ki keys hain, taake
`HashMap<K, V>` yeh bata sake ke do keys same hain ya nahi.

### `PartialOrd` and `Ord` for Ordering Comparisons

`PartialOrd` trait aapko sorting purposes ke liye kisi type ke instances ko
compare karne deta hai. Jo type `PartialOrd` implement karti hai usay `<`, `>`,
`<=`, aur `>=` operators ke saath use kiya ja sakta hai. Aap `PartialOrd` trait
sirf un types par apply kar sakte hain jo `PartialEq` bhi implement karti hon.

`PartialOrd` ko derive karne se `partial_cmp` method implement hota hai, jo ek
`Option<Ordering>` return karta hai aur jab di gayi values koi ordering produce
na karein to `None` return karega. Aisi value ki ek example jo ordering
produce nahi karti, halanke us type ki zyada tar values ko compare kiya ja
sakta hai, floating-point `NaN` value hai. `partial_cmp` ko kisi bhi
floating-point number aur `NaN` floating-point value ke saath call karne par
`None` return hoga.

Jab structs par derive kiya jata hai, `PartialOrd` do instances ko struct
definition mein fields ke appear hone ke order ke mutabiq har field ki value
compare karke compare karta hai. Jab enums par derive kiya jata hai, enum
definition mein pehle declare kiye gaye variants ko baad mein listed variants
se less than consider kiya jata hai.

`PartialOrd` trait required hota hai, misal ke taur par, `rand` crate ke
`gen_range` method ke liye, jo range expression mein specify ki gayi range ke
andar ek random value generate karta hai.

`Ord` trait aapko yeh jaanne deta hai ke annotated type ki kisi bhi do values ke
liye ek valid ordering mojood hogi. `Ord` trait `cmp` method implement karta
hai, jo `Option<Ordering>` ke bajaye `Ordering` return karta hai, kyun ke ek
valid ordering hamesha possible hogi. Aap `Ord` trait sirf un types par apply
kar sakte hain jo `PartialOrd` aur `Eq` dono implement karti hon (`Eq` ke liye
`PartialEq` required hai). Structs aur enums par derive kiye jane par `cmp`
bilkul usi tarah behave karta hai jis tarah `PartialOrd` ke saath `partial_cmp`
ki derived implementation karti hai.

Ek example jahan `Ord` required hota hai `BTreeSet<T>` mein values store karna
hai, jo ek aisa data structure hai jo values ke sort order ki bunyaad par data
store karta hai.

### `Clone` and `Copy` for Duplicating Values

`Clone` trait aapko kisi value ki deep copy explicitly create karne deta hai,
aur duplication ka process arbitrary code run karne aur heap data copy karne par
mushtamil ho sakta hai. `Clone` ke bare mein mazeed information ke liye
Chapter 4 ka [“Variables and Data Interacting with
Clone”][variables-and-data-interacting-with-clone]<!-- ignore --> section
dekhein.

`Clone` ko derive karne se `clone` method implement hota hai, jo jab poori type
ke liye implement hota hai to type ke har part par `clone` call karta hai. Is ka
matlab hai ke type ke tamam fields ya values ka bhi `Clone` implement karna
zaroori hai taake `Clone` derive kiya ja sake.

Ek example jahan `Clone` required hota hai slice par `to_vec` method call karna
hai. Slice un type instances ki ownership nahi rakhti jinhein woh contain
karti hai, lekin `to_vec` se return hone wale vector ko apne instances ki
ownership rakhni hogi, is liye `to_vec` har item par `clone` call karta hai.
Is tarah, slice mein stored type ka `Clone` implement karna zaroori hai.

`Copy` trait aapko kisi value ko sirf stack par stored bits copy karke duplicate
karne deta hai; is ke liye arbitrary code ki zaroorat nahi hoti. `Copy` ke bare
mein mazeed information ke liye Chapter 4 ka [“Stack-Only Data:
Copy”][stack-only-data-copy]<!-- ignore --> section dekhein.

`Copy` trait koi methods define nahi karta, taake programmers un methods ko
overload karke is assumption ki khilaf warzi na kar saken ke koi arbitrary code
run nahi ho raha. Is tarah tamam programmers yeh assume kar sakte hain ke kisi
value ko copy karna bohat fast hoga.

Aap `Copy` ko kisi bhi aisi type par derive kar sakte hain jis ke tamam parts
`Copy` implement karte hon. Jo type `Copy` implement karti hai us ka `Clone`
bhi implement karna zaroori hai, kyun ke `Copy` implement karne wali type ke
paas `Clone` ki ek trivial implementation hoti hai jo `Copy` jaisa hi kaam
karti hai.

`Copy` trait ki zaroorat kam hi padti hai; `Copy` implement karne wali types ke
liye optimizations available hoti hain, jis ka matlab hai ke aapko `clone`
call karne ki zaroorat nahi hoti, aur code zyada concise ho jata hai.

`Copy` ke saath jo kuch possible hai woh `Clone` ke saath bhi accomplish kiya ja
sakta hai, lekin code slower ho sakta hai ya kuch jagahon par `clone` use karna
par sakta hai.

### `Hash` for Mapping a Value to a Value of Fixed Size

`Hash` trait aapko arbitrary size ki kisi type ke instance ko ek hash function
use karke fixed size ki value mein map karne deta hai. `Hash` ko derive karne
se `hash` method implement hota hai. `hash` method ki derived implementation
type ke har part par `hash` call karne ke results ko combine karti hai, jis ka
matlab hai ke derive `Hash` karne ke liye tamam fields ya values ka bhi `Hash`
implement karna zaroori hai.

Ek example jahan `Hash` required hota hai `HashMap<K, V>` mein keys store karna
hai, taake data efficiently store kiya ja sake.

### `Default` for Default Values

`Default` trait aapko kisi type ki default value create karne deta hai. `Default`
ko derive karne se `default` function implement hota hai. `default` function ki
derived implementation type ke har part par `default` function call karti hai,
jis ka matlab hai ke type ke tamam fields ya values ka bhi `Default` implement
karna zaroori hai taake `Default` derive kiya ja sake.

`Default::default` function aam tor par Chapter 5 ke [“Creating Instances from
Other Instances with Struct Update
Syntax”][creating-instances-from-other-instances-with-struct-update-syntax]<!--
ignore --> section mein discuss ki gayi struct update syntax ke saath use hota
hai. Aap struct ke kuch fields ko customize kar sakte hain aur phir
`..Default::default()` use karke baqi fields ke liye default value set aur use
kar sakte hain.

`Default` trait required hota hai jab aap `Option<T>` instances par
`unwrap_or_default` method use karte hain, misal ke taur par. Agar `Option<T>`
`None` ho, to `unwrap_or_default` us `T` type ke liye `Default::default` ka
result return karega jo `Option<T>` mein stored hai.

[creating-instances-from-other-instances-with-struct-update-syntax]: ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax
[stack-only-data-copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
[variables-and-data-interacting-with-clone]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-clone
[custom-derive-macros]: ch20-05-macros.html#custom-derive-macros
