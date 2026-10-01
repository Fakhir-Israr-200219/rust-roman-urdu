# Introduction

> Note: Is edition ki kitab wahi hai jo [The Rust Programming Language][nsprust] ke naam se print aur ebook format mein [No Starch Press][nsp] se available hai.

[nsprust]: https://nostarch.com/rust-programming-language-3rd-edition
[nsp]: https://nostarch.com/

*The Rust Programming Language* mein khush aamdeed, jo Rust ke baare mein ek introductory kitab hai.
Rust programming language aapko tez aur zyada reliable software likhne mein madad karti hai.
Programming language design mein high-level ergonomics aur low-level control aksar ek doosre ke muqabil hote hain; Rust is conflict ko challenge karta hai.
Powerful technical capacity aur ek great developer experience ke darmiyan balance qayam karke, Rust aapko low-level details (jaise memory usage) ko control karne ka option deta hai, baghair us tamam mushkil ke jo aam tor par is tarah ke control ke saath juri hoti hai.


## Rust Kis Ke Liye Hai

Rust mukhtalif wajah ki bina par bohat se logon ke liye ideal hai. Aaiye kuch sab se aham groups par nazar daalte hain.


### Developers Ki Teams

Rust mukhtalif levels ki systems programming knowledge rakhne wale developers ki bari teams ke darmiyan collaboration ke liye ek productive tool sabit ho rahi hai. Low-level code mein mukhtalif bareek bugs hone ka imkaan hota hai, jinhein zyada tar doosri languages mein sirf extensive testing aur experienced developers ke ehtiyat se kiye gaye code review ke zariye pakra ja sakta hai. Rust mein compiler gatekeeper ka role ada karta hai aur aise mushkil se nazar aane wale bugs, jin mein concurrency bugs bhi shamil hain, wale code ko compile karne se inkar kar deta hai. Compiler ke saath mil kar kaam karte hue, team apna waqt bugs dhoondhne ke bajaye program ki logic par focus karne mein laga sakti hai.

Rust systems programming ki duniya mein contemporary developer tools bhi lata hai:

* Cargo, jo included dependency manager aur build tool hai, dependencies ko add, compile, aur manage karna Rust ecosystem mein aasaan aur consistent bana deta hai.
* `rustfmt` formatting tool developers ke darmiyan consistent coding style ko ensure karta hai.
* Rust Language Server code completion aur inline error messages ke liye integrated development environment (IDE) integration provide karta hai.

Rust ecosystem mein in aur doosre tools ko use karke developers systems-level code likhte hue productive reh sakte hain.


### Students

Rust students aur un logon ke liye hai jo systems concepts ke baare mein seekhne mein interested hain. Rust ko use karte hue, bohat se logon ne operating systems development jaise topics ke baare mein seekha hai. Community bohat welcoming hai aur students ke questions ka jawab dene mein khushi mehsoos karti hai. Is kitab jaisi efforts ke zariye, Rust teams chahti hain ke systems concepts ko zyada logon ke liye, khaas taur par programming mein naye logon ke liye, zyada accessible banaya ja sake.


### Companies

Chhoti aur bari, dono qisam ki hundreds of companies Rust ko production mein mukhtalif tasks ke liye use karti hain, jin mein command line tools, web services, DevOps tooling, embedded devices, audio aur video analysis aur transcoding, cryptocurrencies, bioinformatics, search engines, Internet of Things applications, machine learning, aur hatta ke Firefox web browser ke major parts bhi shamil hain.


### Open Source Developers

Rust un logon ke liye hai jo Rust programming language, community, developer tools, aur libraries build karna chahte hain. Hum chahte hain ke aap Rust language mein contribute karein.


### Jo Log Speed aur Stability Ko Ahmiyat Dete Hain

Rust un logon ke liye hai jo kisi language mein speed aur stability chahte hain. Speed se hamari murad ye hai ke Rust code kitni tezi se run kar sakta hai aur Rust aapko programs kitni tezi se likhne deta hai. Rust compiler ke checks feature additions aur refactoring ke zariye stability ko ensure karte hain. Ye un languages ke brittle legacy code ke baraks hai jin mein is tarah ke checks nahi hote, aur developers aksar us code mein changes karne se darte hain. Zero-cost abstractions—yani higher-level features jo manually likhe gaye code jitni speed se lower-level code mein compile hoti hain—ke liye koshish karte hue, Rust safe code ko fast code banane ki bhi koshish karta hai.

Rust language umeed karti hai ke woh bohat se doosre users ko bhi support kare; yahan jin logon ka zikr kiya gaya hai woh sirf kuch sab se bade stakeholders hain. Overall, Rust ki sab se badi ambition ye hai ke safety *aur* productivity, speed *aur* ergonomics provide karke un trade-offs ko khatam kiya jaye jinhein programmers ne decades se accept kiya hai. Rust ko try karein, aur dekhein ke iski choices aapke liye kaam karti hain ya nahi.


## Ye Kitab Kis Ke Liye Hai

Ye kitab ye assume karti hai ke aap ne kisi doosri programming language mein code likha hua hai, lekin ye koi assumption nahi karti ke woh kaunsi language thi. Hum ne koshish ki hai ke is material ko programming ke mukhtalif backgrounds rakhne wale logon ke liye broadly accessible banaya ja sake. Hum is baat par zyada waqt nahi lagate ke programming *kya* hoti hai ya is ke baare mein kaise sochna chahiye. Agar aap programming mein bilkul naye hain, to aapke liye aisi kitab parhna zyada behtar hoga jo specifically programming ka introduction provide karti ho.


## Is Kitab Ko Kaise Use Karein

Aam tor par, ye kitab ye assume karti hai ke aap isay shuru se aakhir tak sequence mein parh rahe hain. Baad ke chapters pehle chapters mein diye gaye concepts par build karte hain, aur pehle chapters kisi khaas topic ki details mein shayad zyada gehrai se na jayein, lekin baad ke kisi chapter mein us topic ko dobara cover kiya jayega.

Aapko is kitab mein do qisam ke chapters milenge: concept chapters aur project chapters. Concept chapters mein aap Rust ke kisi aspect ke baare mein seekhenge. Project chapters mein hum mil kar chhote programs build karenge, aur ab tak jo kuch aap ne seekha hai usay apply karenge. Chapter 2, Chapter 12, aur Chapter 21 project chapters hain; baqi tamam concept chapters hain.


**Chapter 1** batata hai ke Rust ko kaise install karna hai, “Hello, world!” program kaise likhna hai, aur Cargo, jo Rust ka package manager aur build tool hai, ko kaise use karna hai. **Chapter 2** Rust mein program likhne ka ek hands-on introduction hai, jisme aap ek number-guessing game build karenge. Yahan hum concepts ko high level par cover karte hain, aur baad ke chapters mazeed detail provide karenge. Agar aap foran practical kaam shuru karna chahte hain, to **Chapter 2** is ke liye jagah hai. Agar aap khaas taur par meticulous learner hain jo agay barhne se pehle har detail seekhna pasand karte hain, to aap **Chapter 2** ko skip karke seedha **Chapter 3** par ja sakte hain, jo Rust ke un features ko cover karta hai jo doosri programming languages ke features se milte julte hain; phir jab aap seekhi hui details ko apply karte hue kisi project par kaam karna chahein, to **Chapter 2** par wapas aa sakte hain.


**Chapter 4** mein aap Rust ke ownership system ke baare mein seekhenge. **Chapter 5** structs aur methods par baat karta hai. **Chapter 6** enums, `match` expressions, aur `if let` aur `let...else` control flow constructs ko cover karta hai. Aap structs aur enums ko use karke custom types banayenge.


In **Chapter 7**, aap Rust ke module system aur code ko organize karne ke liye privacy rules aur uski public application programming interface (API) ke baare mein seekhenge. **Chapter 8** standard library ki provide ki gayi kuch common collection data structures par baat karta hai: vectors, strings, aur hash maps. **Chapter 9** Rust ki error-handling philosophy aur techniques ko explore karta hai.


**Chapter 10** generics, traits, aur lifetimes ko detail mein discuss karta hai, jo aapko aisa code define karne ki power dete hain jo multiple types par apply hota hai. **Chapter 11** poori tarah testing ke baare mein hai, jo Rust ki safety guarantees ke bawajood ye ensure karne ke liye zaroori hai ke aapke program ki logic correct hai. **Chapter 12** mein hum `grep` command line tool ki functionality ke ek subset ka apna implementation build karenge, jo files ke andar text search karta hai. Is ke liye hum un bohat se concepts ko use karenge jin par hum ne pichlay chapters mein discussion ki hai.


**Chapter 13** closures aur iterators ko explore karta hai: Rust ke aise features jo functional programming languages se aaye hain. **Chapter 14** mein hum Cargo ko mazeed detail mein examine karenge aur doosron ke saath apni libraries share karne ke best practices par baat karenge. **Chapter 15** standard library ke provide kiye gaye smart pointers aur unki functionality ko enable karne wale traits par discussion karta hai.


In **Chapter 16**, hum concurrent programming ke mukhtalif models ko samjhenge aur baat karenge ke Rust aapko multiple threads mein be-khauf programming karne mein kaise madad karta hai. **Chapter 17** mein hum isi bunyaad ko aage barhate hue Rust ke async aur await syntax ke saath tasks, futures, aur streams ko explore karenge, aur us lightweight concurrency model ko samjhenge jo ye enable karte hain.


**Chapter 18** dekhta hai ke Rust ke idioms un object-oriented programming principles ke muqable mein kaise hain jin se aap shayad waqif hain. **Chapter 19** patterns aur pattern matching ka ek reference hai, jo Rust programs mein ideas ko express karne ke powerful tareeqe hain. **Chapter 20** mein advanced topics ka ek smorgasbord shamil hai jin mein unsafe Rust, macros, aur lifetimes, traits, types, functions, aur closures ke baare mein mazeed maloomat shamil hai.


In **Chapter 21**, hum ek project complete karenge jisme hum ek low-level multithreaded web server implement karenge!

Aakhir mein, kuch appendixes mein language ke baare mein useful information zyada reference-like format mein di gayi hai. **Appendix A** Rust ke keywords ko cover karta hai, **Appendix B** Rust ke operators aur symbols ko cover karta hai, **Appendix C** standard library ki taraf se provide kiye gaye derivable traits ko cover karta hai, **Appendix D** kuch useful development tools ko cover karta hai, aur **Appendix E** Rust editions ko explain karta hai. **Appendix F** mein aap kitab ki translations dhoond sakte hain, aur **Appendix G** mein hum cover karenge ke Rust kaise banaya jata hai aur nightly Rust kya hai.

Is kitab ko parhne ka koi ghalat tareeqa nahi hai: Agar aap aage ke chapters par jump karna chahte hain, to bilkul karein! Agar aapko kisi cheez mein confusion ho to aapko pehle ke chapters par wapas jana par sakta hai. Lekin jo tareeqa aapke liye kaam kare, wahi karein.


<span id="ferris"></span>

Rust seekhne ke process ka ek important hissa ye seekhna hai ke compiler jo error
messages display karta hai, unhein kaise read karna hai: Ye aapko working code ki taraf guide karenge. Isi liye, hum bohat se aise examples provide karenge jo compile nahi hote, aur har situation mein compiler jo error message show karega, woh bhi saath diya jayega. Ye baat yaad rakhein ke agar aap koi random example enter karke run karte hain, to ho sakta hai ke woh compile na ho! Is baat ko zaroor dekhein ke aap jis example ko run karne ki koshish kar rahe hain, kya woh error dene ke liye hai ya nahi. Zyada tar situations mein, hum aapko us code ke correct version tak le jayenge jo compile nahi hota. Ferris bhi aapko aisa code pehchanne mein madad karega jo work karne ke liye nahi hai:


| Ferris                                                                                                           | Meaning                                          |
| ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| <img src="img/ferris/does_not_compile.svg" class="ferris-explain" alt="Ferris with a question mark"/>            | Ye code compile nahi hota!                      |
| <img src="img/ferris/panics.svg" class="ferris-explain" alt="Ferris throwing up their hands"/>                   | Ye code panic karta hai!                                |
| <img src="img/ferris/not_desired_behavior.svg" class="ferris-explain" alt="Ferris with one claw up, shrugging"/> | Ye code desired behavior produce nahi karta. |

Zyada tar situations mein, hum aapko us code ke correct version tak le jayenge jo compile nahi hota.

## Source Code

Is kitab ko generate karne wali source files [GitHub][book] par mil sakti hain.

[book]: https://github.com/rust-lang/book/tree/main/src

