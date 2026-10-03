# Common Collections

Rust ki standard library mein bohat si useful data structures shamil hain jinhein
*collections* kaha jata hai. Zyada tar doosre data types ek specific value ko
represent karte hain, lekin collections multiple values rakh sakti hain. Built-in
array aur tuple types ke baraks, jin data ko ye collections point karti hain woh
heap par store hota hai, jis ka matlab hai ke data ki miktar ka compile time par
maloom hona zaroori nahi hota aur program run hote waqt ye barh ya kam ho sakti
hai. Har qisam ki collection ki apni different capabilities aur costs hoti hain,
aur apni current situation ke liye munasib collection choose karna ek aisi skill
hai jo aap waqt ke saath develop karenge. Is chapter mein hum teen aisi
collections discuss karenge jo Rust programs mein bohat zyada use hoti hain:

* Ek *vector* aapko values ki variable number ko ek doosre ke saath store karne deta hai.
* Ek *string* characters ki ek collection hoti hai. Hum ne pehle `String` type ka zikr kiya hai, lekin is chapter mein hum is ke baare mein detail mein baat karenge.
* Ek *hash map* aapko kisi value ko ek specific key ke saath associate karne deta hai. Ye zyada general data structure jise *map* kaha jata hai, ki ek particular implementation hai.

Standard library ki provide ki hui doosri qisam ki collections ke baare mein
seekhne ke liye, [documentation][collections] dekhein.

Hum discuss karenge ke vectors, strings, aur hash maps ko kaise create aur update
kiya jata hai, aur ye bhi ke in mein se har ek ko kya khaas banata hai.

[collections]: ../std/collections/index.html
