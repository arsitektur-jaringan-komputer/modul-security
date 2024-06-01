# Cross-Site Scripting (XSS)

## Disclaimer ⚠
Peringatan: Segala bentuk serangan XSS tanpa izin pada pemilik layanan/sistem dapat dikenakan sanksi pidana yang serius. Segala informasi yang dipaparkan di sini hanya untuk tujuan pembelajaran dan tidak boleh digunakan untuk aktivitas ilegal.

## Introduction to XSS
Cross-site scripting (juga disingkat sebagai XSS) adalah *web vulnerability* yang memungkinkan penyerang untuk mengontrol interaksi pengguna dengan sebuah aplikasi. Ini memungkinkan penyerang untuk melewati *Same Origin Policy*, yang dirancang untuk membatasi situs web antar satu sama lain. Cross-site scripting biasanya memungkinkan *attacker* untuk menyamar sebagai korban, melakukan tindakan apa pun yang dapat dilakukan oleh korban, dan mengakses data korban. Jika korban memiliki akses istimewa dalam aplikasi, maka *attacker* mungkin dapat mengendalikan semua fungsi dan data aplikasi.

## But, how does XSS works?
Cross-site scripting bekerja dengan memanipulasi situs web yang rentan sehingga menjalankan kode JavaScript berbahaya terhadap korban, sehinnga *attacker* dapat mengontrol interaksi korban dengan aplikasi tersebut.

<img src='https://s3.memeshappen.com/memes/lth1gt-ltscriptgt-alertxss-ltscriptgt-.jpg'>

## Type of XSS
Ada tiga tipe XSS yang umum, yaitu:
1. Reflected XSS, di mana kode berbahaya berasal dari *HTTP request* saat ini.
2. Stored XSS, di mana kode berbahaya berasal dari database aplikasi.
3. DOM-based XSS, di mana kerentanan ada dalam kode *client-side* aplikasi.

## Reflected cross-site scripting
<img src="https://assets.website-files.com/5ff66329429d880392f6cba2/60b35d7657e59b65ec41e8b0_Reflected%20XSS%20attacks.png">
Reflected XSS adalah tipe XSS yang paling sederhana. Ini terjadi ketika aplikasi menerima data melalui *HTTP request* dan mengembalikan data tersebut dalam respons (*HTTP response*) langsung dengan cara yang tidak aman.

Contoh sederhana dari kerentanan reflected XSS adalah sebagai berikut:
```html
URL: https://website.com/login?msg=Incorrect+Username+or+Password

Output:
<div class="warning">Incorrect Username or Password</div>
```
Aplikasi tidak melakukan pemrosesan lebih lanjut terhadap data yang dikirim, sehingga *attacker* dapat dengan mudah melakukan serangan seperti ini:
```html
URL: https://website.com/status?message=<script>/*+Malicious+Code+Here+*/</script>

Output:
<div class="warning"><script>/* Malicious Code Here */</script></div>
```
Jika korban mengunjungi URL yang dibuat oleh *attacker*, maka kode berbahaya tersebut akan dieksekusi di browser korban. Jika pada saat itu korban memiliki *session* dalam aplikasi, misalnya korban telah melakukan login ke aplikasi, maka *attacker* dapat melakukan tindakan apa pun dan mengambil data apa pun yang dapat di akses oleh *korban* dalam aplikasi tersebut.

## Stored cross-site scripting

<img src="https://securityzines.com/assets/img/flyers/downloads/intigriti/stored-xss.png">

Stored XSS (juga dikenal sebagai persistent atau second-order XSS) terjadi ketika aplikasi menerima data berbahaya dari sumber yang tidak tepercaya dan menyertakan data tersebut dalam *HTTP response* dengan cara yang tidak aman.

Data tersebut mungkin dikirimkan ke aplikasi melalui *HTTP request*; misalnya, komentar pada postingan blog, *username*, atau lainnya.

Berikut adalah contoh sederhana dari Stored XSS. Aplikasi memiliki fitur yang memungkinkan *user* untuk mengirimkan pesan, yang ditampilkan kepada pengguna lain:
```html
<p>Hello, this is my message!</p>
```

Aplikasi tidak melakukan pemrosesan lebih lanjut terhadap data, sehingga *attacker* dapat dengan mudah mengirim pesan yang mengandung kode berbahaya untuk menyerang *user* lain:
```html
<p>/* Malicious Code Here */</p>
```

## DOM-based cross-site scripting
DOM-based XSS (juga dikenal sebagai DOM XSS) terjadi ketika aplikasi menggunakan beberapa kode JavaScript pada *client-side* yang memproses data dari sumber yang tidak tepercaya dengan cara yang tidak aman, biasanya dengan menulis data kembali ke DOM.

Dalam contoh berikut, sebuah aplikasi menggunakan kode JavaScript untuk membaca *value* dari kolom input dan menulis nilai tersebut ke *HTML element*:
```javascript
var search = document.getElementById('search').value;
var results = document.getElementById('results');
results.innerHTML = 'You searched for: ' + search;
```

Jika *attacker* dapat mengontrol *value* kolom input, mereka dapat dengan mudah membuat *value* berbahaya yang menyebabkan kode tersebut dieksekusi:
```html
<div id="results">
You searched for: <img src=1 onerror='/* Malicious Code Here */'>
</div>
```

Dalam kasus yang umum, kolom input akan diisi dari bagian *HTTP request*, seperti parameter *query* URL, memungkinkan *attacker* untuk mengirimkan serangan menggunakan URL yang berbahaya, dengan cara yang sama seperti *reflected XSS*.

## Impact
Dampak dari XSS umumnya bergantung pada konteks aplikasi,  fungsionalitas dan data, serta status user/akun yang berhasil dikontrol. Sebagai contoh:
1. Dalam aplikasi di mana semua pengguna anonim dan semua informasi publik, dampaknya biasanya minimal.
2. Dalam aplikasi yang menyimpan data sensitif, seperti transaksi perbankan, email, atau catatan kesehatan, dampaknya biasanya serius.
3. Jika user/akun yang berhasil dikontrol memiliki hak istimewa yang tinggi dalam suatu aplikasi, maka dampaknya umumnya kritis, karena hal tersebut dapat memungkinkan penyerang untuk mengambil kendali penuh atas aplikasi tersebut dan mengontrol semua user/akun serta data mereka.

## Cara mencegah serangan XSS
Mencegah cross-site scripting adalah hal yang mudah dalam beberapa kasus tetapi bisa jauh lebih sulit tergantung pada kompleksitas aplikasi dan cara menangani data yang dapat dikontrol pengguna.

Secara umum, secara efektif mencegah kerentanan XSS kemungkinan akan melibatkan kombinasi tindakan berikut:
 
1. Memfilter input saat diterima. Segala input yang berasal dari pengguna harus difilter seketat mungkin berdasarkan apa yang diharapkan.
2. Meng-*encode* data pada *output/response*. Jika aplikasi ingin menyeterkan *user input* dalam *HTTP response*, maka perlu dilakukan encode output untuk mencegahnya diinterpretasikan sebagai konten yang aktif.
3. Gunakan *HTTP response header* yang sesuai. Untuk mencegah XSS dalam *HTTP response* yang memang tidak ditujukan untuk berisi kode HTML atau JavaScript, kita dapat menggunakan header "Content-Type" dan "X-Content-Type-Options" untuk memastikan bahwa browser menginterpretasikan respons sesuai dengan yang kita maksudkan.
4. Content Security Policy (CSP).# Cross-Site Scripting (XSS)

## Disclaimer ⚠
Peringatan: Segala bentuk serangan XSS tanpa izin pada pemilik layanan/sistem dapat dikenakan sanksi pidana yang serius. Segala informasi yang dipaparkan di sini hanya untuk tujuan pembelajaran dan tidak boleh digunakan untuk aktivitas ilegal.

## Introduction to XSS
Cross-site scripting (juga disingkat sebagai XSS) adalah *web vulnerability* yang memungkinkan penyerang untuk mengontrol interaksi pengguna dengan sebuah aplikasi. Ini memungkinkan penyerang untuk melewati *Same Origin Policy*, yang dirancang untuk membatasi situs web antar satu sama lain. Cross-site scripting biasanya memungkinkan *attacker* untuk menyamar sebagai korban, melakukan tindakan apa pun yang dapat dilakukan oleh korban, dan mengakses data korban. Jika korban memiliki akses istimewa dalam aplikasi, maka *attacker* mungkin dapat mengendalikan semua fungsi dan data aplikasi.

## But, how does XSS works?
Cross-site scripting bekerja dengan memanipulasi situs web yang rentan sehingga menjalankan kode JavaScript berbahaya terhadap korban, sehinnga *attacker* dapat mengontrol interaksi korban dengan aplikasi tersebut.

<img src='https://portswigger.net/web-security/images/cross-site-scripting.svg'>

## Type of XSS
Ada tiga tipe XSS yang umum, yaitu:
1. Reflected XSS, di mana kode berbahaya berasal dari *HTTP request* saat ini.
2. Stored XSS, di mana kode berbahaya berasal dari database aplikasi.
3. DOM-based XSS, di mana kerentanan ada dalam kode *client-side* aplikasi.

## Reflected cross-site scripting
Reflected XSS adalah tipe XSS yang paling sederhana. Ini terjadi ketika aplikasi menerima data melalui *HTTP request* dan mengembalikan data tersebut dalam respons (*HTTP response*) langsung dengan cara yang tidak aman.

Contoh sederhana dari kerentanan reflected XSS adalah sebagai berikut:
```html
URL: https://website.com/login?msg=Incorrect+Username+or+Password

Output:
<div class="warning">Incorrect Username or Password</div>
```
Aplikasi tidak melakukan pemrosesan lebih lanjut terhadap data yang dikirim, sehingga *attacker* dapat dengan mudah melakukan serangan seperti ini:
```html
URL: https://website.com/status?message=<script>/*+Malicious+Code+Here+*/</script>

Output:
<div class="warning"><script>/* Malicious Code Here */</script></div>
```
Jika korban mengunjungi URL yang dibuat oleh *attacker*, maka kode berbahaya tersebut akan dieksekusi di browser korban. Jika pada saat itu korban memiliki *session* dalam aplikasi, misalnya korban telah melakukan login ke aplikasi, maka *attacker* dapat melakukan tindakan apa pun dan mengambil data apa pun yang dapat di akses oleh *korban* dalam aplikasi tersebut.

## Stored cross-site scripting
Stored XSS (juga dikenal sebagai persistent atau second-order XSS) terjadi ketika aplikasi menerima data berbahaya dari sumber yang tidak tepercaya dan menyertakan data tersebut dalam *HTTP response* dengan cara yang tidak aman.

Data tersebut mungkin dikirimkan ke aplikasi melalui *HTTP request*; misalnya, komentar pada postingan blog, *username*, atau lainnya.

Berikut adalah contoh sederhana dari Stored XSS. Aplikasi memiliki fitur yang memungkinkan *user* untuk mengirimkan pesan, yang ditampilkan kepada pengguna lain:
```html
<p>Hello, this is my message!</p>
```

Aplikasi tidak melakukan pemrosesan lebih lanjut terhadap data, sehingga *attacker* dapat dengan mudah mengirim pesan yang mengandung kode berbahaya untuk menyerang *user* lain:
```html
<p>/* Malicious Code Here */</p>
```

## DOM-based cross-site scripting
DOM-based XSS (juga dikenal sebagai DOM XSS) terjadi ketika aplikasi menggunakan beberapa kode JavaScript pada *client-side* yang memproses data dari sumber yang tidak tepercaya dengan cara yang tidak aman, biasanya dengan menulis data kembali ke DOM.

Dalam contoh berikut, sebuah aplikasi menggunakan kode JavaScript untuk membaca *value* dari kolom input dan menulis nilai tersebut ke *HTML element*:
```javascript
var search = document.getElementById('search').value;
var results = document.getElementById('results');
results.innerHTML = 'You searched for: ' + search;
```

Jika *attacker* dapat mengontrol *value* kolom input, mereka dapat dengan mudah membuat *value* berbahaya yang menyebabkan kode tersebut dieksekusi:
```html
<div id="results">
You searched for: <img src=1 onerror='/* Malicious Code Here */'>
</div>
```

Dalam kasus yang umum, kolom input akan diisi dari bagian *HTTP request*, seperti parameter *query* URL, memungkinkan *attacker* untuk mengirimkan serangan menggunakan URL yang berbahaya, dengan cara yang sama seperti *reflected XSS*.

## Impact
Dampak dari XSS umumnya bergantung pada konteks aplikasi,  fungsionalitas dan data, serta status user/akun yang berhasil dikontrol. Sebagai contoh:
1. Dalam aplikasi di mana semua pengguna anonim dan semua informasi publik, dampaknya biasanya minimal.
2. Dalam aplikasi yang menyimpan data sensitif, seperti transaksi perbankan, email, atau catatan kesehatan, dampaknya biasanya serius.
3. Jika user/akun yang berhasil dikontrol memiliki hak istimewa yang tinggi dalam suatu aplikasi, maka dampaknya umumnya kritis, karena hal tersebut dapat memungkinkan penyerang untuk mengambil kendali penuh atas aplikasi tersebut dan mengontrol semua user/akun serta data mereka.

## Cara mencegah serangan XSS
Mencegah cross-site scripting adalah hal yang mudah dalam beberapa kasus tetapi bisa jauh lebih sulit tergantung pada kompleksitas aplikasi dan cara menangani data yang dapat dikontrol pengguna.

Secara umum, secara efektif mencegah kerentanan XSS kemungkinan akan melibatkan kombinasi tindakan berikut:
 
1. Memfilter input saat diterima. Segala input yang berasal dari pengguna harus difilter seketat mungkin berdasarkan apa yang diharapkan.
2. Meng-*encode* data pada *output/response*. Jika aplikasi ingin menyeterkan *user input* dalam *HTTP response*, maka perlu dilakukan encode output untuk mencegahnya diinterpretasikan sebagai konten yang aktif.
3. Gunakan *HTTP response header* yang sesuai. Untuk mencegah XSS dalam *HTTP response* yang memang tidak ditujukan untuk berisi kode HTML atau JavaScript, kita dapat menggunakan header "Content-Type" dan "X-Content-Type-Options" untuk memastikan bahwa browser menginterpretasikan respons sesuai dengan yang kita maksudkan.
4. Content Security Policy (CSP).