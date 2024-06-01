## Directory Traversal Vulnerabilities

### Deskripsi

Directory Traversal merupakan kerentanan dimana aplikasi web memperbolehkan client untuk mengakses file yang tidak seharusnya diakses dan tidak sewajarnya diakses melalui interface web diluar root directory website. Sebagai contoh, client dapat mengakses informsi/dokumen pada server yang seharusnya tidak ditampilkan di website.

### Contoh:

Kode php dan html dibawah, berfungsi sebagai pengubah warna background dari website kita dengan memberikan kode php berdasarkan warna yang dipilih pada parameter COLOR di GET request.
<br>
<br>

![image](https://github.com/arsitektur-jaringan-komputer/Modul-Web-App-Security/assets/100863813/82ed8a67-ee1b-4b92-958f-5291d4c0b395)

<br>
<br>

Menurut anda, apa yang akan anda lakukan sebagai penyerang untuk dapat mengakses file lain yang ada pada server?

### Cara Mengidentifikasi Kerentanan Directory Traversal

- Identifikasi request parameter yang dapat dimanipulasi
- Lakukan percobaan dengan memasukkan payload supaya website memuat informasi yang tidak seharusnya bisa diakses
- Lihat error

### Contoh serangan Directory Traversal

Input yang tidak tersanitasi dan cara menampilkan file dengan cara yang kurang baik dapat menyebabkan munculnya kelemahan Directory Traversal, sebagai contoh pada kasus di DVWA berikut

![File Inclusion Section](https://github.com/arsitektur-jaringan-komputer/Modul-Web-App-Security/blob/master/src/FileInclusionSection.png?raw=true)

bila kita lihat dari kode sumbernya:

![Alt text](https://github.com/arsitektur-jaringan-komputer/Modul-Web-App-Security/blob/master/src/FI_Source_Code.png?raw=true)

Input parameter 'page' dari user tidak disanitasi dengan baik, sehingga apabila kita memasukkan input seperti:

```
../../../../../etc/passwd
```

Akan menampilkan file seperti berikut:

![Alt text](https://github.com/arsitektur-jaringan-komputer/Modul-Web-App-Security/blob/master/src/DirTraverse.png?raw=true)

File tersebut menampilkan siapa saja user yang beroperasi dalam sistem, terlihat tidak terlalu berbahaya bukan? Bagaimana apabila kasusnya kita ganti menjadi seperti ini:

- contoh kalian menyimpan file berupa catatan pribadi perusahaan atau informasi rahasia seperti pada sebuah file email_pass.txt di server seperti berikut:

```
user                 pass
admin@gmail.com   Djum4nt0sup3r
```

- Apabila hacker bisa menemukan lokasi dari file tersebut dan menggunakan cara diatas

```
/var/www/dvwa/email_pass.txt
```
Hasilnya bisa seperti berikut:

![Cred Leak](https://github.com/arsitektur-jaringan-komputer/Modul-Web-App-Security/blob/master/src/DirTravCredLeak.png?raw=true)