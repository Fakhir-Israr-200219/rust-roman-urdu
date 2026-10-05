## Building a Single-Threaded Web Server

Hum ek single-threaded web server ko working banane se shuru karenge. Shuru
karne se pehle, aao web servers build karne mein shamil protocols ka ek quick
overview dekh lete hain. In protocols ki details is book ke scope se bahar hain,
lekin ek brief overview aapko woh information dega jo aapko chahiye.

Web servers mein shamil do main protocols *Hypertext Transfer
Protocol* *(HTTP)* aur *Transmission Control Protocol* *(TCP)* hain. Dono
protocols *request-response* protocols hain, yani ek *client* requests initiate
karta hai aur ek *server* requests ko listen karta hai aur client ko response
provide karta hai. Un requests aur responses ka content protocols ke zariye
define hota hai.

TCP lower-level protocol hai jo is baat ki details describe karta hai ke
information ek server se doosre server tak kaise pohanchti hai, lekin yeh
specify nahi karta ke woh information kya hai. HTTP TCP ke upar build hota hai
aur requests aur responses ke content ko define karta hai. Technically HTTP ko
doosre protocols ke saath use karna possible hai, lekin bohat zyada cases mein
HTTP apna data TCP ke zariye send karta hai. Hum TCP aur HTTP requests aur
responses ke raw bytes ke saath kaam karenge.

### Listening to the TCP Connection

Hamare web server ko TCP connection ko listen karna hoga, is liye yahi woh
pehla hissa hai jis par hum kaam karenge. Standard library ek `std::net`
module provide karti hai jo humein yeh karne deti hai. Aao usual tareeqe se
ek naya project banate hain:

```console
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

Ab shuru karne ke liye Listing 21-1 ka code *src/main.rs* mein enter karein.
Yeh code incoming TCP streams ke liye local address `127.0.0.1:7878` par listen
karega. Jab ise koi incoming stream milegi, yeh `Connection established!`
print karega.

<Listing number="21-1" file-name="src/main.rs" caption="Incoming streams ke liye listen karna aur stream receive hone par ek message print karna">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-01/src/main.rs}}
```

</Listing>

`TcpListener` ko use karke, hum address `127.0.0.1:7878` par TCP connections
ke liye listen kar sakte hain. Address mein colon se pehle wala section ek IP
address hai jo aapke computer ko represent karta hai (yeh har computer par
same hai aur specifically authors ke computer ko represent nahi karta), aur
`7878` port hai. Humne yeh port do reasons ki wajah se choose kiya hai: HTTP
normally is port par accept nahi hota, is liye hamara server aapki machine par
chalne wale kisi doosre web server ke saath conflict karne ka imkaan kam hai,
aur 7878 telephone par *rust* type hota hai.

Is scenario mein `bind` function `new` function ki tarah kaam karta hai, yani
yeh ek naya `TcpListener` instance return karega. Function ko `bind` is liye
kehte hain kyun ke networking mein, listen karne ke liye kisi port se connect
karna “binding to a port” kehlata hai.

`bind` function ek `Result<T, E>` return karta hai, jo indicate karta hai ke
binding fail ho sakti hai, misal ke taur par agar hum apne program ke do
instances run kar dein aur is tarah do programs same port ko listen kar rahe
hon. Kyun ke hum learning purposes ke liye sirf ek basic server likh rahe hain,
hum is tarah ke errors ko handle karne ki fikr nahi karenge; is ke bajaye, agar
errors hoti hain to program ko rokne ke liye hum `unwrap` use karte hain.

`TcpListener` par `incoming` method ek iterator return karta hai jo humein
streams ki ek sequence deta hai (zyada specifically, `TcpStream` type ke
streams). Ek single *stream* client aur server ke darmiyan ek open connection
ko represent karta hai. *Connection* us poore request aur response process ka
naam hai jismein client server se connect karta hai, server response generate
karta hai, aur server connection close karta hai. Is liye, client ne kya send
kiya hai yeh dekhne ke liye hum `TcpStream` se read karenge aur phir client ko
data wapas send karne ke liye stream par apna response write karenge. Overall,
yeh `for` loop har connection ko bari bari process karega aur hamare handle
karne ke liye streams ki ek series produce karega.

Filhaal, stream ko handle karne mein hum `unwrap` call kar rahe hain taake agar
stream mein koi errors hon to hamara program terminate ho jaye; agar koi errors
na hon, to program ek message print karta hai. Agli listing mein hum success
case ke liye aur functionality add karenge. `incoming` method se client ke
server se connect hone par errors milne ki wajah yeh hai ke hum asal mein
connections par iterate nahi kar rahe. Is ke bajaye, hum *connection attempts*
par iterate kar rahe hain. Connection kai reasons ki wajah se successful nahi
ho sakta, jin mein se bohat se operating system specific hote hain. Misal ke
taur par, bohat se operating systems simultaneous open connections ki tadaad
par ek limit rakhte hain; is tadaad se zyada new connection attempts ek error
produce karenge jab tak kuch open connections close nahi ho jate.

Aao is code ko run karke dekhte hain! Terminal mein `cargo run` invoke karein aur
phir web browser mein *127.0.0.1:7878* load karein. Browser ko “Connection
reset” jaisa error message dikhna chahiye kyun ke server filhaal koi data
wapas send nahi kar raha. Lekin jab aap apna terminal dekhenge, to aapko kai
messages nazar aane chahiye jo browser ke server se connect hone par print hue
hain!

```text
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

Kabhi kabhi aapko ek browser request ke liye multiple messages print hote hue
nazr aayenge; is ki wajah yeh ho sakti hai ke browser page ke liye ek request
ke saath doosre resources ke liye bhi request kar raha ho, jaise *favicon.ico*
icon jo browser tab mein nazar aata hai.

Yeh bhi ho sakta hai ke browser server se multiple baar connect karne ki koshish
kar raha ho kyun ke server kisi bhi data ke saath respond nahi kar raha. Jab
`stream` scope se bahar chala jata hai aur loop ke end par drop hota hai, to
`drop` implementation ke hisse ke taur par connection close ho jata hai.
Browsers kabhi kabhi closed connections ko retry karke handle karte hain, kyun
ke problem temporary ho sakti hai.

Browsers kabhi kabhi server ke saath multiple connections bhi open karte hain
bina koi requests send kiye, taake agar baad mein woh *do* requests send karein,
to woh requests zyada quickly ho saken. Jab aisa hota hai, hamara server har
connection ko dekhega, chahe us connection par koi requests hon ya na hon.
Chrome-based browsers ke kai versions, misal ke taur par, aisa karte hain; aap
private browsing mode use karke ya koi different browser use karke is
optimization ko disable kar sakte hain.

Important baat yeh hai ke humne successfully ek TCP connection ka handle hasil
kar liya hai!

Yaad rakhein ke jab aap kisi particular version of the code ko run karna
mukammal kar lein to <kbd>ctrl</kbd>-<kbd>C</kbd> press karke program ko stop
kar dein. Phir, har set of code changes karne ke baad `cargo run` command
invoke karke program ko restart karein taake yeh ensure ho ke aap newest code
run kar rahe hain.

### Reading the Request

Aao browser se request read karne ki functionality implement karte hain! Pehle
connection hasil karne aur phir us connection ke saath koi action lene ki
responsibilities ko alag karne ke liye, hum connections ko process karne ke liye
ek naya function start karenge. Is naye `handle_connection` function mein, hum
TCP stream se data read karenge aur use print karenge taake hum browser se send
hone wala data dekh saken. Code ko Listing 21-2 ki tarah change karein.

<Listing number="21-2" file-name="src/main.rs" caption="`TcpStream` se read karna aur data print karna">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-02/src/main.rs}}
```

</Listing>

Hum `std::io::BufReader` aur `std::io::prelude` ko scope mein laate hain taake
un traits aur types tak access mil sake jo humein stream se read aur us par
write karne dete hain. `main` function mein `for` loop ke andar, connection
banne ka message print karne ke bajaye, ab hum naye `handle_connection`
function ko call karte hain aur `stream` usay pass karte hain.

`handle_connection` function mein, hum ek naya `BufReader` instance create
karte hain jo `stream` ke reference ko wrap karta hai. `BufReader` hamare liye
`std::io::Read` trait ke methods ko calls manage karke buffering add karta hai.

Hum `http_request` naam ka ek variable create karte hain jo browser ki taraf se
hamare server ko bheji gayi request ki lines ko collect karega. Hum `Vec<_>` type
annotation add karke indicate karte hain ke hum in lines ko ek vector mein
collect karna chahte hain.

`BufReader`, `std::io::BufRead` trait ko implement karta hai, jo `lines` method
provide karta hai. `lines` method `Result<String, std::io::Error>` ka ek
iterator return karta hai, jo data ke stream ko har baar newline byte dekhne par
split karta hai. Har `String` hasil karne ke liye hum har `Result` par `map` aur
`unwrap` karte hain. Agar data valid UTF-8 na ho ya stream se read karte waqt
koi problem ho to `Result` mein error ho sakta hai. Dobara, ek production
program ko in errors ko zyada gracefully handle karna chahiye, lekin simplicity
ke liye hum error ki situation mein program ko stop karne ka faisla kar rahe
hain.

Browser ek HTTP request ke end ko do newline characters ek ke baad ek send
karke signal karta hai, is liye stream se ek request hasil karne ke liye hum
lines ko us waqt tak lete hain jab tak humein ek aisi line na mil jaye jo empty
string ho. Jab hum lines ko vector mein collect kar lete hain, to hum unhein
pretty debug formatting use karke print karte hain taake hum dekh saken ke web
browser hamare server ko kya instructions send kar raha hai.

Aao is code ko try karte hain! Program start karein aur dobara web browser mein
ek request karein. Note karein ke browser mein humein ab bhi ek error page milega,
lekin terminal mein hamare program ka output ab kuch is tarah nazar aayega:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-02
cargo run
make a request to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
]
```

Aapke browser ke mutabiq, output thora different ho sakta hai. Ab jab hum
request data print kar rahe hain, to request ki pehli line mein `GET` ke baad
wale path ko dekh kar hum samajh sakte hain ke ek browser request se humein
multiple connections kyun milti hain. Agar repeated connections sab */* ko
request kar rahi hain, to hum jaante hain ke browser */* ko repeatedly fetch
karne ki koshish kar raha hai kyun ke use hamare program se response nahi mil
raha.

Aao is request data ko break down karte hain taake samajh saken ke browser hamare
program se kya maang raha hai.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-closer-look-at-an-http-request"></a> <a id="looking-closer-at-an-http-request"></a>

### Looking More Closely at an HTTP Request

HTTP ek text-based protocol hai, aur ek request is format mein hoti hai:

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

Pehli line *request line* hoti hai jo is baat ki information rakhti hai ke
client kya request kar raha hai. Request line ka pehla hissa use hone wale
method ko indicate karta hai, jaise `GET` ya `POST`, jo describe karta hai ke
client yeh request kaise kar raha hai. Hamare client ne `GET` request use ki,
jis ka matlab hai ke woh information maang raha hai.

Request line ka agla hissa */* hai, jo *uniform resource
identifier* *(URI)* ko indicate karta hai jise client request kar raha hai: URI
lagbhag, lekin bilkul nahi, *uniform resource locator* *(URL)* ke jaisa hota
hai. URIs aur URLs ke darmiyan farq is chapter mein hamare purposes ke liye
important nahi hai, lekin HTTP spec term *URI* use karti hai, is liye yahan hum
zehni taur par *URI* ko *URL* se substitute kar sakte hain.

Aakhri hissa HTTP version hota hai jo client use karta hai, aur phir request
line CRLF sequence par end hoti hai. (*CRLF* ka matlab *carriage return* aur
*line feed* hai, jo typewriter ke zamane ki terms hain!) CRLF sequence ko
`\r\n` ke taur par bhi likha ja sakta hai, jahan `\r` carriage return hai aur
`\n` line feed hai. *CRLF sequence* request line ko baqi request data se
separate karti hai. Note karein ke jab CRLF print hota hai, to humein `\r\n`
ke bajaye ek new line start hoti hui nazar aati hai.

Ab tak apne program ko run karke jo request line data humne receive kiya hai,
use dekhte hue, hum dekhte hain ke `GET` method hai, */* request URI hai, aur
`HTTP/1.1` version hai.

Request line ke baad, `Host:` se shuru hone wali baqi lines headers hain. `GET`
requests ki koi body nahi hoti.

Kisi different browser se request karne ki koshish karein ya kisi different
address ke liye request karein, jaise *127.0.0.1:7878/test*, taake dekhein ke
request data kaise change hota hai.

Ab jab hum jaante hain ke browser kya maang raha hai, aao kuch data wapas send
karte hain!

### Writing a Response

Hum client ki request ke response mein data send karne ki functionality
implement karenge. Responses ka format yeh hota hai:

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

Pehli line *status line* hoti hai jo response mein use hone wale HTTP version,
request ke result ko summarize karne wala numeric status code, aur status code
ki text description provide karne wala reason phrase contain karti hai. CRLF
sequence ke baad koi bhi headers, ek aur CRLF sequence, aur response ki body
hoti hai.

Yahan ek example response hai jo HTTP version 1.1 use karta hai aur jis mein
200 ka status code, OK reason phrase, koi headers nahi, aur koi body nahi hai:

```text
HTTP/1.1 200 OK\r\n\r\n
```

Status code 200 standard success response hai. Yeh text ek chhota sa successful
HTTP response hai. Aao successful request ke response ke taur par ise stream
mein write karte hain! `handle_connection` function se woh `println!` remove
karein jo request data print kar raha tha aur uski jagah Listing 21-3 ka code
use karein.

<Listing number="21-3" file-name="src/main.rs" caption="Stream mein ek chhota sa successful HTTP response write karna">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-03/src/main.rs:here}}
```

</Listing>

Pehli nayi line `response` variable define karti hai jo success message ka
data hold karta hai. Phir, hum apne `response` par `as_bytes` call karte hain
taake string data ko bytes mein convert kar saken. `stream` par `write_all`
method ek `&[u8]` leti hai aur un bytes ko directly connection ke zariye send
karti hai. Kyun ke `write_all` operation fail ho sakta hai, hum pehle ki tarah
kisi bhi error result par `unwrap` use karte hain. Dobara, ek real application
mein aap yahan error handling add karenge.

In changes ke saath, aao apna code run karein aur ek request karein. Ab hum
terminal par koi data print nahi kar rahe, is liye Cargo ke output ke ilawa
humein koi output nazar nahi aayega. Jab aap web browser mein
*127.0.0.1:7878* load karenge, to aapko error ke bajaye ek blank page milna
chahiye. Aapne abhi handcode karke HTTP request receive karna aur response send
karna seekh liya hai!

### Returning Real HTML

Aao blank page se zyada kuch return karne ki functionality implement karte hain.
Apni project directory ke root mein nayi file *hello.html* create karein, *src*
directory mein nahi. Aap koi bhi HTML input kar sakte hain; Listing 21-4 ek
possibility dikhati hai.

<Listing number="21-4" file-name="hello.html" caption="Response mein return karne ke liye ek sample HTML file">

```html
{{#include ../listings/ch21-web-server/listing-21-05/hello.html}}
```

</Listing>

Yeh ek minimal HTML5 document hai jis mein ek heading aur kuch text hai. Jab
request receive ho to server se isay return karne ke liye, hum `handle_connection`
ko Listing 21-5 mein dikhaye gaye tareeqe se modify karenge taake HTML file ko
read kare, use response mein body ke taur par add kare, aur send kare.

<Listing number="21-5" file-name="src/main.rs" caption="*hello.html* ke contents ko response ki body ke taur par send karna">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-05/src/main.rs:here}}
```

</Listing>

Humne `use` statement mein `fs` add kiya hai taake standard library ka
filesystem module scope mein aa jaye. File ke contents ko string mein read
karne ka code aapko familiar lagna chahiye; humne Listing 12-4 mein apne I/O
project ke liye file ke contents read karte waqt isay use kiya tha.

Agla step, hum `format!` use karke file ke contents ko success response ki body
ke taur par add karte hain. Valid HTTP response ensure karne ke liye, hum
`Content-Length` header add karte hain, jo hamari response body ke size par set
hota hai—is case mein, `hello.html` ka size.

Is code ko `cargo run` ke saath run karein aur apne browser mein
*127.0.0.1:7878* load karein; aapko apna HTML rendered nazar aana chahiye!

Filhaal, hum `http_request` mein request data ko ignore kar rahe hain aur bina
kisi condition ke sirf HTML file ke contents wapas send kar rahe hain. Is ka
matlab hai ke agar aap apne browser mein *127.0.0.1:7878/something-else* request
karne ki koshish karein, to aapko phir bhi yahi same HTML response wapas milega.
Is waqt hamara server bohat limited hai aur woh woh kaam nahi karta jo zyada
tar web servers karte hain. Hum request ke mutabiq apne responses ko customize
karna chahte hain aur HTML file sirf */* ke ek well-formed request ke liye wapas
send karna chahte hain.

### Validating the Request and Selectively Responding

Filhaal, hamara web server client ki request chahe jo bhi ho, file mein maujood
HTML return karega. Aao functionality add karte hain jo HTML file return karne
se pehle check kare ke browser */* request kar raha hai, aur agar browser koi
aur cheez request kare to ek error return kare. Is ke liye humein
`handle_connection` ko modify karna hoga, jaisa ke Listing 21-6 mein dikhaya
gaya hai. Yeh naya code received request ke content ko us request ke saath
check karta hai jiske */* ke liye hone ka humein pata hai, aur requests ko
different tareeqe se treat karne ke liye `if` aur `else` blocks add karta hai.

<Listing number="21-6" file-name="src/main.rs" caption="*/ * ke requests ko doosre requests se mukhtalif tareeqe se handle karna">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-06/src/main.rs:here}}
```

</Listing>

Hum sirf HTTP request ki pehli line dekhenge, is liye poori request ko vector
mein read karne ke bajaye, hum `next` call karke iterator ka pehla item hasil
kar rahe hain. Pehla `unwrap` `Option` ko handle karta hai aur agar iterator
mein koi items na hon to program ko stop kar deta hai. Doosra `unwrap` `Result`
ko handle karta hai aur iska effect us `unwrap` jaisa hi hai jo Listing 21-2
mein add kiye gaye `map` mein tha.

Agla step, hum `request_line` ko check karte hain ke kya yeh */* path par GET
request ki request line ke barabar hai. Agar aisa hai, to `if` block hamari HTML
file ke contents return karta hai.

Agar `request_line` */* path par GET request ke barabar *nahi* hai, to iska
matlab hai ke humein koi aur request receive hui hai. Hum doosri tamam requests
ka response dene ke liye `else` block mein ek moment mein code add karenge.

Ab is code ko run karein aur *127.0.0.1:7878* request karein; aapko
*hello.html* mein maujood HTML milna chahiye. Agar aap koi aur request karein,
jaise *127.0.0.1:7878/something-else*, to aapko connection error milega, bilkul
un errors ki tarah jo aapne Listing 21-1 aur Listing 21-2 ka code run karte
waqt dekhe thay.

Ab aao Listing 21-7 ka code `else` block mein add karte hain taake status code
404 ke saath ek response return ki ja sake, jo signal karta hai ke request ka
content nahi mila. Hum browser mein render hone ke liye kuch HTML bhi return
karengi jo end user ko response ke bare mein indicate karegi.

<Listing number="21-7" file-name="src/main.rs" caption="*/ * ke ilawa kuch bhi request kiye jane par status code 404 aur error page ke saath response dena">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-07/src/main.rs:here}}
```

</Listing>

Yahan, hamari response mein status code 404 aur reason phrase `NOT FOUND` ke
saath ek status line hai. Response ki body file *404.html* mein maujood HTML
hogi. Error page ke liye aapko *hello.html* ke saath *404.html* file create
karni hogi; dobara, aap koi bhi HTML use kar sakte hain, ya Listing 21-8 mein
di gayi example HTML use kar sakte hain.

<Listing number="21-8" file-name="404.html" caption="Kisi bhi 404 response ke saath wapas bhejne ke liye sample page content">

```html
{{#include ../listings/ch21-web-server/listing-21-07/404.html}}
```

</Listing>

In changes ke saath apne server ko dobara run karein. *127.0.0.1:7878* request
karne par *hello.html* ke contents return hone chahiye, aur koi bhi doosri
request, jaise *127.0.0.1:7878/foo*, ko *404.html* ka error HTML return karna
chahiye.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-touch-of-refactoring"></a>

### Refactoring

Filhaal, `if` aur `else` blocks mein bohat repetition hai: Dono files read kar
rahe hain aur files ke contents ko stream mein write kar rahe hain. Sirf status
line aur filename mein farq hai. Aao status line aur filename ki values ko
variables mein assign karne wali separate `if` aur `else` lines mein in
differences ko nikaal kar code ko zyada concise banate hain; phir hum un
variables ko file read karne aur response write karne wale code mein
unconditionally use kar sakte hain. Listing 21-9 large `if` aur `else` blocks
ko replace karne ke baad resultant code dikhati hai.

<Listing number="21-9" file-name="src/main.rs" caption="`if` aur `else` blocks ko refactor karna taake un mein sirf woh code rahe jo dono cases ke darmiyan different hai">

```rust,no_run id="3g8x4m"
{{#rustdoc_include ../listings/ch21-web-server/listing-21-09/src/main.rs:here}}
```

</Listing>

Ab `if` aur `else` blocks sirf tuple mein status line aur filename ki
appropriate values return karte hain; phir hum destructuring use karke in dono
values ko `let` statement mein ek pattern ke zariye `status_line` aur `filename`
mein assign karte hain, jaisa ke Chapter 19 mein discuss kiya gaya tha.

Pehle duplicate kiya gaya code ab `if` aur `else` blocks ke bahar hai aur
`status_line` aur `filename` variables ko use karta hai. Is se dono cases ke
darmiyan farq dekhna aasaan ho jata hai, aur iska matlab hai ke agar hum file
reading aur response writing ke tareeqe ko change karna chahein, to code ko
update karne ke liye hamare paas sirf ek jagah hai. Listing 21-9 mein code ka
behavior Listing 21-7 ke code jaisa hi hoga.

Awesome! Ab hamare paas taqreeban 40 lines of Rust code mein ek simple web
server hai jo ek request ka response content ke page ke saath deta hai aur
baqi tamam requests ka response 404 ke saath deta hai.

Filhaal, hamara server ek single thread mein run karta hai, yani yeh ek waqt
mein sirf ek request serve kar sakta hai. Aao kuch slow requests simulate karke
dekhein ke yeh problem kaise ban sakti hai. Phir hum isay fix karenge taake
hamara server ek waqt mein multiple requests handle kar sake.
