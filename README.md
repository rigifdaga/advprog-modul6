Rifda Aulia Nurbahri
2206081660
Adpro B

Commit 1
Fungsi handle_connection beroperasi dengan menerima referensi mutable ke TcpStream. Fungsi ini kemudian membuat BufReader yang melingkupi aliran TCP yang ada, yang memfasilitasi pembacaan data yang efisien dari aliran tersebut.

BufReader berfungsi untuk membaca data dari aliran TCP dengan cara yang efisien. Fungsi ini kemudian membaca setiap baris dari BufReader sampai menemukan baris kosong, yang menandakan akhir dari header permintaan HTTP.

Setelah semua baris telah dibaca, fungsi ini mengumpulkan baris-baris tersebut menjadi vektor, yang mewakili permintaan HTTP yang dibuat oleh klien.

Akhirnya, handle_connection mencetak permintaan HTTP yang telah dikumpulkan untuk dianalisis lebih lanjut. Ini memungkinkan pengguna untuk memeriksa detail permintaan yang diterima oleh server, baik untuk tujuan debugging atau analisis.

Jadi, handle_connection berfungsi dengan membaca aliran TCP masuk baris demi baris sampai menemukan baris kosong yang menandakan akhir dari header permintaan HTTP. Baris-baris yang telah dikumpulkan kemudian dicetak sebagai permintaan HTTP untuk dianalisis lebih lanjut.

Commit 2

Fungsi `handle_connection` yang telah dimodifikasi melakukan hal berikut:
- Mengatur `status_line` untuk menunjukkan bahwa respons HTTP adalah 200 OK, yang berarti permintaan telah berhasil diproses.
- Membaca konten dari file yang bernama `hello.html` dan mengubahnya menjadi string menggunakan `fs::read_to_string()`. Ini mengasumsikan bahwa file `hello.html` berada di direktori yang sama dengan program yang sedang berjalan.
- Menghitung panjang dari string yang berisi konten file.
- Menyiapkan format respons HTTP, yang mencakup baris status, panjang konten, dan konten dari file `hello.html`.
- Menulis respons tersebut kembali ke aliran TCP dan mengirimkannya ke klien menggunakan `write_all()`.

Jadi, fungsi `handle_connection` yang telah dimodifikasi ini membaca konten dari file `hello.html`, membuat respons HTTP dengan status 200 OK, dan mengirim respons tersebut kembali ke klien melalui aliran TCP.

[![Screenshot-2024-03-23-101533.png](https://i.postimg.cc/25CXYbZP/Screenshot-2024-03-23-101533.png)](https://postimg.cc/qNZGX7QL)

Commit 3

1. Saya membuat file `404.html` seperti berikut. 
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Hello!</title>
</head>
<body>
<h1>Oops!</h1>
<p>Sorry, I don't know what you're asking for.</p>
</body>
</html>
``` 

2. Kemudian, saya memodifikasi fungsi `handle_connection` seperti berikut. 
```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    if request_line == "GET / HTTP/1.1" {
        let status_line = "HTTP/1.1 200 OK";
        let contents = fs::read_to_string("hello.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    } else {
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    }
}
```

3. Lalu, saya melakukan *refactor* pada fungsi `handle_connection` seperti berikut.
```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };
    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response = format!(
        "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
    );
    stream.write_all(response.as_bytes()).unwrap();
}
```

4. Berikut adalah tampilan dari `404.html`.

[![Screenshot-2024-03-23-103258.png](https://i.postimg.cc/vZKLq4vr/Screenshot-2024-03-23-103258.png)](https://postimg.cc/TKrDhYF1)

Fungsi `handle_connection` yang dimodifikasi berperan sebagai server HTTP yang sederhana. Ia membaca baris permintaan dari klien dalam permintaan HTTP dan memeriksa apakah itu merupakan permintaan `GET / HTTP/1.1`, yang menandakan permintaan untuk jalur *root* (/). Berdasarkan pemeriksaan ini, fungsi memberikan tanggapan dengan konten yang berbeda:

1. Jika permintaan adalah untuk jalur *root* (GET / HTTP/1.1), fungsi akan memberikan tanggapan dengan baris status `HTTP/1.1 200 OK`. Selanjutnya, ia membaca konten dari file bernama `hello.html`, menghitung panjangnya, membuat tanggapan HTTP yang berisi konten tersebut, dan mengirimkannya kembali ke klien.

2. Jika permintaan bukan untuk jalur *root* (menandakan bahwa sumber daya yang diminta tidak ditemukan), fungsi akan memberikan tanggapan dengan baris status `HTTP/1.1 404 NOT FOUND`. Kemudian, ia membaca konten dari file bernama `404.html`, menghitung panjangnya, membuat tanggapan HTTP yang berisi konten tersebut, dan mengirimkannya kembali ke klien.

Maka dari itu, fungsi `handle_connection` yang telah dimodifikasi ini menanggapi dengan konten yang berbeda berdasarkan permintaan yang diterima. Jika permintaan adalah untuk jalur *root*, server akan memberikan tanggapan dengan `hello.html`. Jika tidak, server akan memberikan tanggapan dengan halaman `404.html` yang menunjukkan bahwa sumber daya yang diminta tidak ditemukan.

Commit 4

1. Fungsi `handle_connection` tetap melakukan pembacaan baris permintaan dari klien seperti sebelumnya.
2. Namun, sekarang fungsi ini menggunakan pemodelan pola untuk memeriksa jenis permintaan yang berbeda.
3. Jika permintaan adalah `GET / HTTP/1.1`, yang menunjukkan permintaan untuk jalur utama (/), maka respons dengan status `HTTP/1.1 200 OK` dan konten dari file `hello.html` akan dikirim.
4. Jika permintaan adalah `GET /sleep HTTP/1.1`, yang menunjukkan permintaan untuk membuat server tidur sebelum memberikan respons, maka server akan beristirahat selama 10 detik sebelum mengirim respons dengan status `HTTP/1.1 200 OK` dan konten dari `hello.html`.
5. Jika permintaan tidak sesuai dengan pola yang ditentukan, server akan mengirim respons dengan status `HTTP/1.1 404 NOT FOUND` dan konten dari file `404.html`.
6. Setelah respons yang sesuai ditentukan, server membaca konten dari file yang diperlukan.
7. Panjang konten dihitung.
8. Respons HTTP dibuat dengan informasi yang diperlukan.
9. Akhirnya, respons dikirim kembali ke klien melalui aliran TCP. Dengan demikian, fungsi server dapat diperluas untuk menangani permintaan khusus di mana server akan beristirahat sejenak sebelum memberikan respons jika permintaan adalah `GET /sleep HTTP/1.1`. Selain itu, server tetap melayani permintaan untuk `hello.html` atau memberikan respons 404 jika sumber daya tidak ditemukan.

Commit 5

1. Selama `Inisialisasi ThreadPool (ThreadPool::new)`, kita memulai *thread pool* dengan jumlah thread pekerja yang ditentukan, membuat channel `(mpsc::channel)` untuk komunikasi antara thread utama dan pekerja, dan membuat vektor untuk menyimpan thread pekerja. Setiap thread pekerja memiliki struktur *Worker* yang menerima pekerjaan melalui channel.

2. Selama `Pembuatan Worker (Worker::new)`, setiap thread pekerja memiliki loop yang tak terbatas yang menunggu pekerjaan. Saat pekerjaan diterima, thread pekerja akan menjalankannya.

3. Selanjutnya, selama `Eksekusi Job (ThreadPool::execute)`, thread utama mengirimkan *job* ke salah satu thread pekerja melalui channel. *Job* tersebut kemudian dieksekusi oleh thread pekerja.

4. Akhirnya, selama proses konkurensi dan komunikasi, `Arc<Mutex<mpsc::Receiver<Job>>>` dibagikan di antara semua thread pekerja. Ini memungkinkan beberapa thread untuk mengakses ujung penerima channel secara aman. Channel `mpsc` (*multi-producer, single-consumer*) memungkinkan pengiriman *job* secara bersamaan dan aman antara thread pekerja.

Jadi, *thread pool* mengelola sejumlah tetap thread pekerja. *Job* diserahkan ke pool dan dieksekusi secara bersamaan oleh thread pekerja, memberikan mekanisme yang sederhana untuk eksekusi tugas secara paralel.

Commit 6

1. Dalam link buku yang telah disediakan sebelumnya, ada saran untuk menggunakan metode `build` alih-alih `new` saat menginisialisasi `ThreadPool`. Alasannya adalah `new` mungkin akan gagal jika jumlah thread yang ditentukan terlalu sedikit.
2. Namun, hal ini belum tentu benar karena diharapkan metode `new` berhasil. Oleh karena itu, lebih efisien menggunakan metode `new` dibandingkan `build` yang mengembalikan `Result`.
3. Dengan menggunakan metode `build`, ketika nilai dikembalikan ke pemanggil, nilai tersebut dapat di-**unwrapped** untuk mendapatkan hasil eksekusi tanpa risiko kesalahan.

Jadi, meskipun ada saran untuk menggunakan `build`, metode `new` memiliki harapan keberhasilan yang tinggi. Oleh karena itu, pemilihan metode harus disesuaikan dengan kebutuhan dan preferensi pengembang.



