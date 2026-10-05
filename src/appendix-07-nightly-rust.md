## Appendix G - How Rust is Made and “Nightly Rust”

Yeh appendix is bare mein hai ke Rust kaise banaya jata hai aur yeh cheez aap
par ek Rust developer ke taur par kaise asar dalti hai.

### Stability Without Stagnation

Ek language ke taur par, Rust aapke code ki stability (mustahkam rehne) ko
*bohat* ahmiyat deta hai. Hum chahte hain ke Rust ek rock-solid foundation ho
jis par aap build kar saken, aur agar cheezen constantly change hoti rahen, to
yeh mumkin nahi hoga. Saath hi, agar hum new features ke saath experiment nahi
kar sakte, to ho sakta hai ke hum important flaws ka pata unke release hone ke
baad lagayen, jab hum cheezon ko mazeed change na kar saken.

Is problem ka hamara solution woh hai jise hum “stability without stagnation”
kehte hain, aur hamara guiding principle yeh hai: aapko stable Rust ke kisi new
version par upgrade karne se kabhi darna nahi chahiye. Har upgrade painless
hona chahiye, lekin saath hi aapke liye new features, kam bugs, aur faster
compile times bhi lekar aana chahiye.

### Choo, Choo! Release Channels and Riding the Trains

Rust development ek *train schedule* par operate karta hai. Yani, tamam
development Rust repository ki main branch mein ki jati hai. Releases software
release train model ko follow karti hain, jo Cisco IOS aur doosre software
projects ne bhi use kiya hai. Rust ke liye teen *release channels* hain:

* Nightly
* Beta
* Stable

Zyada tar Rust developers primarily stable channel use karte hain, lekin jo log
experimental new features ko try karna chahte hain woh nightly ya beta use kar
sakte hain.

Yahan ek example hai ke development aur release process kaise kaam karta hai:
maan lein Rust team Rust 1.5 ke release par kaam kar rahi hai. Yeh release
December of 2015 mein hui thi, lekin yeh humein realistic version numbers
provide karegi. Rust mein ek new feature add hota hai: ek new commit main branch
mein land karta hai. Har raat Rust ka ek new nightly version produce hota hai.
Har din release day hota hai, aur yeh releases hamari release infrastructure
ke zariye automatically create hoti hain. Is liye waqt guzarte ke saath hamari
releases kuch is tarah nazar aati hain, har raat ek:

```text
nightly: * - - * - - *
```

Har six weeks mein, new release prepare karne ka waqt aa jata hai! Rust
repository ki `beta` branch nightly ke liye use hone wali main branch se branch
off hoti hai. Ab do releases hain:

```text
nightly: * - - * - - *
                     |
beta:                *
```

Zyada tar Rust users beta releases ko actively use nahi karte, lekin Rust ko
possible regressions discover karne mein madad dene ke liye apne CI system mein
beta ke against test karte hain. Is dauran, har raat ek nightly release ab bhi
aati rehti hai:

```text
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

Maan lein ke ek regression mil jati hai. Achhi baat hai ke regression ke stable
release mein ghusne se pehle hamare paas beta release ko test karne ke liye kuch
waqt tha! Fix main branch par apply ki jati hai, taake nightly fix ho jaye, aur
phir fix ko `beta` branch par backport kiya jata hai, aur beta ki ek new release
produce ki jati hai:

```text
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

Pehli beta create hone ke six weeks baad, stable release ka waqt aa jata hai!
`stable` branch `beta` branch se produce hoti hai:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |
beta:                * - - - - - - - - *
                                       |
stable:                                *
```

Hooray! Rust 1.5 complete ho gaya! Lekin hum ek cheez bhool gaye hain: kyun ke
six weeks guzar chuke hain, humein Rust ke *next* version, 1.6 ka ek new beta
bhi chahiye. Is liye `stable` ke `beta` se branch off hone ke baad, `beta` ka
next version dobara `nightly` se branch off hota hai:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

Isay “train model” kaha jata hai kyun ke har six weeks mein ek release “station
se nikalti hai”, lekin stable release ke taur par pohanchne se pehle usay beta
channel ke zariye ek safar karna hota hai.

Rust har six weeks mein, bilkul clockwork ki tarah, release hota hai. Agar aap
Rust ki kisi ek release ki date jaante hain, to aap next release ki date jaan
sakte hain: woh six weeks baad hogi. Har six weeks mein releases scheduled
hone ka ek acha pehlu yeh hai ke next train jald aa rahi hoti hai. Agar koi
feature kisi particular release mein shamil hone se reh jaye, to fikar karne ki
zaroorat nahi: doosri release thore hi waqt mein aa rahi hoti hai! Is se
release deadline ke qareeb possibly unpolished features ko jaldi se shamil
karne ka pressure kam hota hai.

Is process ki wajah se, aap hamesha Rust ki next build ko check out kar sakte
hain aur khud verify kar sakte hain ke us par upgrade karna easy hai: agar beta
release expected tarah se kaam nahi karti, to aap team ko report kar sakte hain
aur next stable release se pehle usay fix karwa sakte hain! Beta release mein
breakage relatively rare hoti hai, lekin `rustc` phir bhi ek software hai, aur
bugs exist karte hain.

### Maintenance time

Rust project sab se recent stable version ko support karta hai. Jab ek new stable
version release hota hai, purana version apni end of life (EOL) par pohanch jata
hai. Is ka matlab hai ke har version ko six weeks tak support kiya jata hai.

### Unstable Features

Is release model ke saath ek aur catch hai: unstable features. Rust ek
technique use karta hai jise “feature flags” kaha jata hai, taake determine kiya
ja sake ke kisi given release mein kaun se features enabled hain. Agar koi new
feature active development ke under hai, to woh main branch mein land hota hai,
aur is liye nightly mein bhi, lekin ek *feature flag* ke peeche. Agar aap,
ek user ke taur par, work-in-progress feature ko try karna chahte hain, to aap
aisa kar sakte hain, lekin aapko Rust ki nightly release use karni hogi aur
apne source code ko appropriate flag ke saath annotate karna hoga taake aap
opt in kar saken.

Agar aap Rust ki beta ya stable release use kar rahe hain, to aap koi feature
flags use nahi kar sakte. Yeh woh key hai jo humein new features ko hamesha ke
liye stable declare karne se pehle unka practical use hasil karne deti hai. Jo
log bleeding edge ko opt into karna chahte hain woh aisa kar sakte hain, aur jo
log rock-solid experience chahte hain woh stable par stick kar sakte hain aur
jaan sakte hain ke unka code break nahi hoga. Stability without stagnation.

Yeh book sirf stable features ke bare mein information contain karti hai, kyun
ke in-progress features abhi bhi change ho rahe hain, aur yaqeenan jab yeh
book likhi gayi thi aur jab woh stable builds mein enabled honge, un dono waqt
ke darmiyan woh different honge. Aap nightly-only features ki documentation
online dhoond sakte hain.

### Rustup and the Role of Rust Nightly

Rustup Rust ke different release channels ke darmiyan global ya per-project
basis par change karna easy banata hai. By default, aapke paas stable Rust
installed hoga. Misal ke taur par nightly install karne ke liye:

```console
$ rustup toolchain install nightly
```

Aap `rustup` ke zariye apne installed tamam *toolchains* (Rust ki releases aur
unke associated components) bhi dekh sakte hain. Yahan aapke authors mein se
ek ke Windows computer par ek example hai:

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

Jaisa ke aap dekh sakte hain, stable toolchain default hai. Zyada tar Rust users
zyada tar waqt stable use karte hain. Aap bhi zyada tar waqt stable use karna
chah sakte hain, lekin kisi specific project par nightly use karna chahte hon,
kyun ke aapko ek cutting-edge feature ki parwah hai. Aisa karne ke liye, aap
us project ki directory mein `rustup override` use karke nightly toolchain ko
woh toolchain set kar sakte hain jo `rustup` ko us directory mein hone par use
karni chahiye:

```console
$ cd ~/projects/needs-nightly
$ rustup override set nightly
```

Ab jab bhi aap *~/projects/needs-nightly* ke andar `rustc` ya `cargo` call
karenge, `rustup` ensure karega ke aap default stable Rust ke bajaye nightly
Rust use kar rahe hain. Yeh tab bohat kaam aata hai jab aapke paas bohat se
Rust projects hon!

### The RFC Process and Teams

To phir aap in new features ke bare mein kaise seekhte hain? Rust ka
development model ek *Request For Comments (RFC) process* follow karta hai.
Agar aap Rust mein koi improvement chahte hain, to aap ek proposal likh sakte
hain, jise RFC kaha jata hai.

Koi bhi shakhs Rust ko improve karne ke liye RFCs likh sakta hai, aur proposals
Rust team review aur discuss karti hai, jo kai topic subteams par mushtamil hai.
Teams ki ek full list [on Rust’s website](https://www.rust-lang.org/governance)
par hai, jis mein project ke har area ke liye teams shamil hain: language
design, compiler implementation, infrastructure, documentation, aur mazeed.
Appropriate team proposal aur comments ko parhti hai, apne comments likhti hai,
aur aakhir mein feature ko accept ya reject karne par consensus hota hai.

Agar feature accept ho jaye, to Rust repository par ek issue open kiya jata hai,
aur koi shakhs usay implement kar sakta hai. Jo shakhs usay implement karta
hai, bohat mumkin hai ke woh wohi shakhs na ho jis ne pehli dafa feature
propose kiya tha! Jab implementation ready ho jati hai, to woh main branch mein
ek feature gate ke peeche land hoti hai, jaisa ke humne [“Unstable
Features”](#unstable-features)<!-- ignore --> section mein discuss kiya hai.

Kuch waqt ke baad, jab nightly releases use karne wale Rust developers ko new
feature try karne ka mauqa mil chuka hota hai, team members feature par
discussion karte hain, dekhte hain ke nightly par woh kaisi rahi, aur decide
karte hain ke usay stable Rust mein shamil hona chahiye ya nahi. Agar decision
aage barhne ka ho, to feature gate remove kar diya jata hai, aur feature ab
stable consider hota hai! Yeh Rust ki new stable release mein trains ke zariye
safar karta hua pohanchta hai.

