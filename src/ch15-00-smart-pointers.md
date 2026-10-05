# Smart Pointers

Pointer ek general concept hai jo aise variable ke liye use hota hai jo memory
mein kisi address ko contain karta hai. Yeh address kisi doosre data ko refer
karta hai, ya “point at” karta hai. Rust mein pointer ki sab se common type
reference hai, jiske baare mein aap Chapter 4 mein seekh chuke hain. References
ko `&` symbol se indicate kiya jata hai aur yeh us value ko borrow karte hain
jise woh point kar rahe hote hain. Data ko refer karne ke ilawa in mein koi
special capabilities nahi hotin, aur in ka koi overhead nahi hota.

*Is ke muqable mein, smart pointers* aisi data structures hain jo pointer ki
tarah act karti hain lekin in ke paas additional metadata aur capabilities
bhi hoti hain. Smart pointers ka concept sirf Rust ke liye unique nahi hai:
Smart pointers C++ mein originate hue aur doosri languages mein bhi maujood
hain. Rust ki standard library mein variety of smart pointers defined hain jo
references ki provided functionality se beyond functionality provide karte
hain. General concept ko explore karne ke liye, hum smart pointers ki kuch
different examples dekhenge, jin mein ek *reference counting* smart pointer
type bhi shamil hai. Yeh pointer aapko data ke multiple owners rakhne ki
ijazat deta hai, kyun ke yeh owners ki tadaad ko track karta hai aur jab koi
owner baqi nahi rehta to data ko clean up karta hai.

Rust mein, ownership aur borrowing ke concept ki wajah se references aur smart
pointers ke darmiyan ek additional difference hai: References sirf data ko
borrow karti hain, jabke bohat se cases mein smart pointers us data ko *own*
karte hain jise woh point karte hain.

Smart pointers aam tor par structs ko use karke implement kiye jate hain.
Ordinary struct ke unlike, smart pointers `Deref` aur `Drop` traits implement
karte hain. `Deref` trait smart pointer struct ke instance ko reference ki tarah
behave karne ki ijazat deta hai, taa-ke aap apna code is tarah likh saken ke
woh references ya smart pointers, dono ke saath kaam kare. `Drop` trait aapko
us code ko customize karne ki ijazat deta hai jo smart pointer ke instance ke
scope se bahar jane par run hota hai. Is chapter mein, hum in dono traits par
discussion karenge aur demonstrate karenge ke yeh smart pointers ke liye kyun
important hain.

Kyun ke smart pointer pattern ek general design pattern hai jo Rust mein
frequently use hota hai, is chapter mein har existing smart pointer cover nahi
kiya jayega. Bohat si libraries ke apne smart pointers hote hain, aur aap apna
smart pointer bhi likh sakte hain. Hum standard library ke sab se common smart
pointers cover karenge:

* `Box<T>`, heap par values allocate karne ke liye
* `Rc<T>`, ek reference counting type jo multiple ownership ko enable karta hai
* `Ref<T>` aur `RefMut<T>`, `RefCell<T>` ke through access kiye jate hain, jo
  borrowing rules ko compile time ke bajaye runtime par enforce karta hai

Is ke ilawa, hum *interior mutability* pattern cover karenge jahan ek immutable
type interior value ko mutate karne ke liye ek API expose karta hai. Hum
reference cycles par bhi discussion karenge: yeh memory ko kaise leak kar sakte
hain aur inhein kaise prevent kiya ja sakta hai.

Aaiye shuru karte hain!
