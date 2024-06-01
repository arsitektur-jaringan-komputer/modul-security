# SQL Injection

## Topik Pembelajaran
- [Basic SQL Injection](#introduction-to-sql-injection-and-basic-sql-injection)
- [SQL Injection using Union Method](#sql-injection-using-union-method)
- [Blind SQL Injection](#blind-sql-injection)
- [Time Based SQL Injection](#time-based-sql-injection)
- [Condition Based Blind SQL Injection](#condition-based-blind-sql-injection)
- [SQL Injection Mitigation](#sql-injection-mitigation)

## Mohon Dibaca Sebelum Lanjut
**&#9888;** **Peringatan:** Segala bentuk serangan _SQL Injection_ tanpa izin pada pemilik layanan/sistem dapat dikenakan **sanksi pidana yang serius**. Segala informasi yang dipaparkan di sini **hanya untuk tujuan pembelajaran** dan **tidak boleh** digunakan untuk aktivitas ilegal.


## Introduction to SQL Injection and Basic SQL Injection 
![sqli](https://hackaday.com/wp-content/uploads/2014/04/18mpenleoksq8jpg.jpg)

SQL Injection merupakan serangan **_server side_** yang menggunakan database sebagai utilisasi dari serangannya. Serangan ini biasa terjadi karena kurang baiknya implementasi sanitasi user input. Penyerang menyisipkan bahasa SQL untuk melancarkan serangannya yang dapat berdampak pada: file inclusion, RCE, auth bypass, information disclosure, dan lain-lain. Menurut data di [OWASP top 10 tahun 2021](https://owasp.org/www-project-top-ten/). SQL Injection (yang sekarang dikategorikan sebagai injection) masuk kedalam urutan ke-3 serangan yang banyak terjadi. 

### Basic SQL Injection
Salah satu implementasi SQL Injection yang umum untuk dipelajari pertama kali adalah pada auth bypass. Berikut adalah contoh kode, analisislah sebelum lanjut kebawah:

```php
<?php

require_once('config.php');

if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    
    $username = $_POST['username'];
    $password = $_POST['password'];

    if (empty($username) || empty($password)) {

        echo "Please provide both username and password";

    } else {

        $query = "SELECT * FROM users WHERE username='$username' AND password='$password'";
        $result = mysqli_query($connection, $query);

        if(mysqli_num_rows($result) > 0) {

            $user = mysqli_fetch_assoc($result);
            echo "Welcome, " . $user['name']; 
        } else {
            
            echo "Invalid username or password";
        }
    }
}
?>
```

Apabila kalian belum pernah belajar security sebelumnya, mungkin mengira betnuk kode seperti itu bukanlah masalah yang besar. Namun, apa yang terjadi apabila user memberikan input seperti berikut pada username: ``' or 1=1--``, dan seperti berikut pada password: ``''``, maka script sql yang akan tereksekusi adalah sebagai berikut:
```sql
SELECT * FROM users WHERE username='' or 1=1-- ' AND password=''''
```
terlihat kalau tulisan ``' AND password=''''`` tercomment (tulisannya jadi hijau kalau di format md), dan yang terproses hanyalah sampai ``1=1``. bila kode sql tersebut tereksekusi, maka hasilnya adalah:

1. Database akan mencari username pada table yang namanya `''`, atau yang hasilnya adalah pengecekan boolean 1=1 (yang berarti benar, karena 1 sama dengan 1).
2. Karena hasilnya true, maka database akan mengembalikan row pertama (karena hasilnya tadi 1=1 adalah true, maka database akan mengambilkan row pertama dalam database). Dan karena row pertama biasanya adalah admin, maka database akan mengira bahwa user login sebagai admin, meskipun username dan passwordnya tidak dimasukkan dengan benar.

## SQL Injection using Union Method

### Visible Error Based SQL Injection
Visible Error Based SQL Injection adalah serangan SQL Injection yang apabila diberikan sebuah input yang melanggar struktur query, maka errornya akan tampil keluar. Sebagai contoh, lihat kode berikut:
```js
app.get("/searchcookies", isAuthenticated, async (req, res, next) => {
  cookies = req.query.cookies;

  const query = `SELECT * FROM cookies WHERE flavor = "${cookies}"`;

    pool.query(query, (err, result) => {
      if(err){
        return next(err)
      }

    return res.status(200).render("index", {cookies: result || []})
    });
})
```

Hasil dari query cookie dikembalikan ke frontend sehingga dapat diproses untuk ditampilkan di website. Bila kita analisis, kita bisa mengetahui bahwa kita bisa saja mengescape dari pencarian flavor dan menambahkan karakter yang melanggar struktur query. Inilah yang dimaksud dengan visible SQL Injection.

Sebagai contoh, kita bisa menambahkan karakter ``"``, sehingga struktur query menjadi seperti berikut:

```sql
SELECT * FROM cookies WHERE flavor = """
```

Hasil dari query tersebut akan menampilkan error. Berarti tidak dilakukan sanitasi dan bisa kita eksploitasi lebih lanjut menggunakan tambahan query sql.

Contoh serangan yang bisa kita lakukan adalah menggunakan **union** untuk mendapatkan result lebih dari hasil querynya. Bagaimana caranya?

### Union Method Attack

[UNION](https://www.w3schools.com/sql/sql_union.asp) di SQL merupakan salah satu perintah yang berguna untuk menggabungkan dua atau lebih hasil ``SELECT`` dari query. Sebagai contoh, kalian ingin melakukan query dari 2 tabel ``users`` dan ``informations``, kalian bisa melakukannya seperti berikut:
```sql
SELECT u_name, u_age, u_phone FROM users 
UNION 
SELECT i_title, i_desc, i_time FROM informations
```
hasilnya nanti akan terdapat result query dari query dari SELECT pertama dari table users, dan SELECT kedua dari table informations. Perlu diperhatikan bahwa penggunaan union harus menghasilkan jumlah column yang sama, apabila tidak, maka hasilnya akan error.

Nah, union ini bisa kita manfaatkan untuk melakukan eksploitasi SQL Injection dengan harapan kita bisa mendapatkan hasil yang lebih variatif dari query pertama. Sebagai contoh, suatu database memiliki struktur table seperti berikut:
**Users Table:**

| Field    | Type          | Constraints         |
|----------|---------------|---------------------|
| id       | INTEGER       | PRIMARY KEY, AUTO_INCREMENT |
| username | varchar(255)  | NOT NULL, UNIQUE    |
| password | varchar(255)  | NOT NULL            |

**Cookies Table:**

| Field   | Type          | Constraints         |
|---------|---------------|---------------------|
| id      | INTEGER       | PRIMARY KEY, AUTO_INCREMENT |
| flavor  | varchar(255)  | NOT NULL            |
| name    | varchar(255)  | NOT NULL            |

Berdasarkan kode program yang rentan sebelumnya, kita bisa melakukan pencarian pada table users dengan cara membuat payload seperti ini:

```sql
chocolate chip" union select username, password, null from users-- -
```

kenapa kita perlu menambahkan ``-- -`` pada akhir query, karena kita perlu melakukan commenting sehingga petik 2 di akhir query orisinil dan segala query di belakangnya tidak diikutkan dalam eksekusi perintah SQL. Kira-kira berikut merupakan hasil akhir query dari payload SQL Injection yang kita buat:

```sql
SELECT * FROM cookies WHERE flavor = "chocolate chip" union select username, password, null from users-- -"
```

Dengan payload tersebut, kita bisa mendapatkan konten dari tables users.

## Blind SQL Injection

Oke, sebelumnya kita belajar serangan SQL Injection yang hasilnya terlihat. Lah, memangnya ada serangan SQL Injection yang hasilnya tidak terlihat? Ada dong. Sebenarnya bukan tidak terlihat, tetapi hasil serangannya tidak secara eksplisit ditampilkan ke user. Serangan ini disebut ``Blind SQL Injection``. Apabila kalian sadar, authentication bypass yang kita lakukan di awal modul adalah salah satu contoh dari teknik ini. Dimana kita tidak mendapatkan hasil dari querynya secara eksplisit. Ada 2 teknik yang akan kita gunakan dalam blind SQL Injection, yakni Time Based SQL Injection, dan Condition Based SQL Injection.

### Time Based SQL Injection
Time Based SQL Injection merupakan serangan yang memanfaatkan command [SLEEP()](https://sqlhints.com/2016/10/11/sleep-command-in-sql-server/). Command sleep akan menyebabkan query tertunda dalam rentang waktu tertentu. Teknik ini bisa digunakan untuk memvalidasi apakah hasil query blind kita memberikan result atau tidak. 

Berikut adalah contoh kode flask yang mengembalikan produk berdasarkan ID:
```python
from flask import Flask, request
import sqlite3

app = Flask(__name__)

@app.route('/product')
def get_product():
    product_id = request.args.get('id')

    conn = sqlite3.connect('products.db')
    cursor = conn.cursor()
    
    # Vulnerable query
    query = "SELECT * FROM products WHERE id=" + product_id
    
    cursor.execute(query)
    result = cursor.fetchone()
    
    conn.close()
    
    if result:
        return f"Product found"
    else:
        return "Product not found!"

if __name__ == '__main__':
    app.run(debug=True)
```

Kita bisa melakukan crafting payload sepertit berikut untuk mengeksfiltrasi data pada table lain:
```SQL
chocolate chip" AND IF(SUBSTRING((SELECT password from users WHERE user='admin'),1,1)='a',SLEEP(5),0)-- - 
```
Kita breakdown satu-satu:
1. (cond) AND (cond); Command ini berfungsi untuk melakukan operasi boolean yang akan return true apa bila kondisi sebelah kanan dan kirinya adalah tidak false
2. IF(Cond, if true, if false); Command ini berfungsi untuk pengecekan condition, dimana parameter pertama berfungsi untuk condition yang kita cek, paramter kedua adalah return ketika kondisi true, dan parameter ketiga adalah return ketika kondisi false
3. SUBSTRING(STRINGS, index, length); command ini berfungsi untuk mengambil substring dari dengan index dan panjang tertentu.
4. SLEEP(TIME); Fungsi untuk melakukan sleep query.

Kira-kira berikut hasil query akhirnya:
```SQL
SELECT * FROM cookies WHERE flavour="chocolate chips" AND IF(SUBSTRING((SELECT password from users WHERE user='admin'),1,1)='a',SLEEP(5),0)-- -
```

Apabila hasil dari eksfiltrasi data itu valid, maka query akan disleep selama beberapa detik sebelum selesai. Lakukan teknik ini berungkali untuk setiap lengthnya sehingga kalian bisa mendapatkan semua data yang ada.

Perlu diperhatikan bahwa perintah sql bisa berbeda-beda setiap jenis databasenya, untuk sehingga kalian perlu menyesuaikan perintahnya dengan database yang dimiliki oleh sistem.


### Condition Based Blind SQL Injection

Condition Based Blind SQL Injection ini bisa dilakukan dengan kalian mengetahui apakah hasil dari query tersebut valid atau tidak. Bisa dengan cara apabila hasil query valid, maka ada kondisi yang ter-reflect di website, sebaliknya, maka hasilnya tidak akan tampil.

Lihatlah kode Backend berikut:
```javascript
//Kode Backend
app.get("/searchcookies", isAuthenticated, async (req, res, next) => {
  cookies = req.query.cookies;

  const query = `SELECT DISTINCT * FROM cookies WHERE flavor = "${cookies}"`;

    pool.query(query, (err, result) => {
      if(err){
        return next(err)
      }
    
    if(result){
      result = "The Cookie exists"
    }

    return res.status(200).render("index", {cookies: result || ""})
    });
})
```

Frontend:
```javascript
//Kode Frontend
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cookies</title>
</head>
<body>
    <h1>Check if ur cookies exist:</h1>
    <form id="cookieForm">
        <input type="text" name="flavor" placeholder="chocolate chip">
        <button type="submit">Search</button>
    </form>

    <h1>Here's your result:</h1>
    <ul>
        <li><%= cookies %></li>
    </ul>
</body>
<script>
    document.getElementById('cookieForm').addEventListener('submit', async function(event) {
        event.preventDefault();
        const formData = new FormData(this);
        const jsonData = {};

        formData.forEach((value, key) => {
            jsonData[key] = value;
        });


        window.location.href = `/searchcookies?cookies=${jsonData["flavor"]}`;
    });
</script>
</html>
```

Hasil dari query hanya menunjukkan apakah hasil querynya exist atau tidak tanpa menampilkan hasilnya. Kita dapat mengeksploitasi untuk mendapatkan data lain dengan cara:
```sql
chocolate chip" AND (SELECT SUBSTRING(password,1,1) from users where username = 'admin') = 'c' -- -
```

Sehingga hasil query adalah sebagai berikut:
```sql
SELECT DISTINCT * FROM cookies WHERE flavor = "chocolate chip" AND (SELECT SUBSTRING(password,1,1) from users where username = 'admin') = 'c' -- -"
```

Hasilnya adalah, apabila substring pertama dari password admin sesuai, maka akan muncul string ``The Cookie exist`` di frontend.

### Tips Melakukan Eksfiltrasi Data dengan Blind SQL Injection

Ada beberapa tahapan yang disarankan untuk kalian lakukan pada implementasi blind sql injection agar mempermudah kalian mendapatkan data yang diinginkan:

1. Ketahui jenis database dan versinya.
2. Ketahui tabel-tabel yang ada.
3. Ketahui panjang dari data yang mau kalian ambil. (atau cukup iterasi sampai kembali ke karakter yang pertama kali digunakan untuk menebak)
4. Ekstrak datanya.


### Cheatsheet SQL Injection
[Cheat Sheet by portswigger](https://portswigger.net/web-security/sql-injection/cheat-sheet)

### Automation Tools to Check SQL Injection
[SQLMAP](https://github.com/sqlmapproject/sqlmap)

## SQL Injection Mitigation

SQL Injection merupakan salah satu kelemahan yang memiliki dampak besar tetapi cukup mudah untuk diatasi. Beberapa solusi yang bisa dilakukan adalah:

### Lakukan Sanitasi Pada User Input
Ini merupakan cara manual yang bisa kalian lakukan, sesederhan alakukan sanitasi pada user input, seperti menambahkan escape string dan blacklisting input dari user sehingga string yang masuk ke query akan ditangani selayaknya sebuah string, sehingga tidak tereksekusi.

Contoh fungsi sanitasi yang bisa dipakai pada beberapa bahasa pemrograman:

#### PHP

1. ``mysqli_real_escape_string()`` - Escapes special characters in a string for use in an SQL statement.
2. ``PDO::quote()`` - Quotes a string for use in a query.
3. ``htmlspecialchars()`` - Converts special characters to HTML entities.
4. ``filter_var()`` - Filters a variable with a specified filter (e.g., FILTER_SANITIZE_STRING).
5. ``addslashes()`` - Adds backslashes before certain characters in a string.
6. ``strip_tags()`` - Strips HTML and PHP tags from a string.

#### JavaScript
1. ``encodeURIComponent()`` - Encodes a URI component.
2. ``escape()`` - Encodes a string (deprecated, but still in use).
3. ``DOMPurify.sanitize()`` (with the DOMPurify library) - Sanitizes HTML and prevents XSS.
sanitize-html (npm package) - Library to sanitize HTML.
4. ``validator.escape()`` (with the validator npm package) - Escapes input to make it safe for inclusion in HTML.

#### Python
1. ``str.encode('utf-8', 'ignore')`` - Encodes a string using the specified encoding.
2. ``html.escape()`` - Escapes special characters to HTML-safe sequences.
3. ``cgi.escape()`` - Escapes special characters to HTML-safe sequences (Python 2, deprecated in Python 3).
4. ``urllib.parse.quote()`` - Percent-encodes a string for URL components.
5. ``bleach.clean()`` - Sanitizes an HTML fragment and prevents XSS (using the Bleach library).
6. ``markupsafe.escape()`` - Escapes strings for safe HTML and XML.

#### Golang
1. ``html.EscapeString()`` - Escapes special characters to HTML-safe sequences.
2. ``url.QueryEscape()`` - Escapes a string so it can be safely placed inside a URL query.
3. ``sql.DB.Exec() / sql.DB.Query()`` with parameterized queries - Prevents SQL injection.
4. ``template.HTMLEscapeString()`` - Escapes a string for HTML.
5. ``template.JSEscapeString()`` - Escapes a string for 
JavaScript.


### Gunakan Framework
Framework menjadi cara otomatis kalian dalam melakukan sanitasi user input. ORM pada framework kebanyakan telah melakukan sanitasi pada user input (selama digunakan sesuai fungsi seharusnya). Sehingga kalian tidak perlu repot-repot melakukan blacklisting atau sanitasi. 