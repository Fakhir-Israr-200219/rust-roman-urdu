<!-- Old headings. Do not remove or links may break. -->

<a id="managing-growing-projects-with-packages-crates-and-modules"></a>

# Packages, Crates, and Modules

Jab aap bade programs likhte hain, to apne code ko organize karna dheere dheere aur zyada important hota jayega. Related functionality ko group karke aur different features wale code ko separate karke, aap ye wazeh kar sakenge ke kisi particular feature ko implement karne wala code kahan milega aur kisi feature ke kaam karne ke tareeqe ko change karne ke liye kahan jana hoga.

Ab tak hum ne jo programs likhe hain woh ek file mein ek module ke andar rahe hain. Jaise jaise project grow hota hai, aapko code ko pehle multiple modules aur phir multiple files mein split karke organize karna chahiye. Ek package mein multiple binary crates aur optionally ek library crate ho sakta hai. Jaise jaise package grow hota hai, aap uske kuch parts ko separate crates mein extract kar sakte hain jo external dependencies ban jate hain. Ye chapter in tamam techniques ko cover karta hai. Bohat bade projects ke liye jo aapas mein related packages ke ek set par mushtamil hon aur saath saath evolve karte hon, Cargo workspaces provide karta hai, jinhein hum Chapter 14 mein [“Cargo Workspaces”][workspaces]<!-- ignore --> mein cover karenge.

Hum implementation details ko encapsulate karne ke baare mein bhi baat karenge, jo aapko higher level par code reuse karne deta hai: Ek baar aap ne koi operation implement kar liya, to doosra code aapke code ko uske public interface ke zariye call kar sakta hai, baghair ye jaane ke ke implementation kis tarah kaam karti hai. Aap jis tarah code likhte hain woh define karta hai ke doosre code ke use karne ke liye kaun se parts public hain aur kaun se parts private implementation details hain jinhein aap future mein change karne ka haq apne paas rakhte hain. Ye un details ki quantity ko limit karne ka ek aur tareeqa hai jo aapko apne zehan mein rakhni padti hain.

Ek related concept scope hai: Woh nested context jisme code likha jata hai, us mein names ka ek set hota hai jo “in scope” define kiye jate hain. Code ko read, write, aur compile karte waqt, programmers aur compilers ko ye jaanna zaroori hota hai ke kisi particular jagah par koi particular name kisi variable, function, struct, enum, module, constant, ya kisi aur item ko refer karta hai aur us item ka kya matlab hai. Aap scopes create kar sakte hain aur change kar sakte hain ke kaun se names scope mein hain aur kaun se scope se bahar. Aap ek hi scope mein same name ke do items nahi rakh sakte; name conflicts ko resolve karne ke liye tools available hain.

Rust mein kai aise features hain jo aapko apne code ki organization manage karne dete hain, jin mein ye bhi shamil hai ke kaun si details expose ki jati hain, kaun si details private hoti hain, aur aapke programs ke har scope mein kaun se names mojood hote hain. In features ko kabhi kabhi collectively *module system* kaha jata hai, aur in mein ye shamil hain:

* **Packages**: Cargo ka ek feature jo aapko crates build, test, aur share karne deta hai
* **Crates**: Modules ka ek tree jo library ya executable produce karta hai
* **Modules and use**: Aapko paths ki organization, scope, aur privacy control karne dete hain
* **Paths**: Kisi item, jaise struct, function, ya module ko name karne ka ek tareeqa

Is chapter mein hum in tamam features ko cover karenge, discuss karenge ke ye aapas mein kis tarah interact karte hain, aur explain karenge ke inhein scope manage karne ke liye kis tarah use kiya jata hai. Chapter ke end tak, aapko module system ki solid understanding honi chahiye aur aap scopes ke saath ek pro ki tarah kaam karne ke qabil hone chahiye!

[workspaces]: ch14-03-cargo-workspaces.html
