# Refleksi: Milestone 1 - Single threaded web server

## Pendahuluan
Dalam tutorial ini, kita telah mempelajari cara menginisialisasi repository Git, mengatur proyek Rust, dan membuat server web sederhana menggunakan `TcpListener` dari pustaka standar Rust. Proses ini memberikan wawasan tentang bagaimana koneksi TCP dapat digunakan untuk menangani permintaan dari browser.

## Pembelajaran Utama

### 1. Inisialisasi Repository dan Pengelolaan Git
Sebelum mulai mengembangkan server web, langkah pertama adalah menginisialisasi repository Git dan menghubungkannya dengan GitLab atau GitHub. Langkah-langkah yang dilakukan meliputi:
- Menambahkan remote repository.
- Melakukan commit awal.
- Mendorong perubahan ke repository menggunakan `git push`.

Langkah-langkah ini sangat penting dalam pengelolaan kode sumber agar dapat bekerja secara kolaboratif dan memiliki riwayat perubahan yang terdokumentasi dengan baik.

### 2. Membuat Server Web Sederhana
Kode awal yang digunakan:
```rust
use std::net::TcpListener;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    for stream in listener.incoming() {
        let stream = stream.unwrap();
        println!("Connection established!");
    }
}
```
Kode ini mengikat server ke alamat `127.0.0.1:7878` dan menunggu koneksi masuk. Ketika ada koneksi dari browser, program akan mencetak "Connection established!" di terminal.

Saat dijalankan, browser tidak menampilkan output karena server hanya mendeteksi koneksi tanpa menangani permintaan lebih lanjut.

### 3. Menangani Permintaan HTTP
Untuk menangani permintaan dari browser, kode diperbarui dengan menambahkan fungsi `handle_connection`:
```rust
use std::{
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
};

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    for stream in listener.incoming() {
        let stream = stream.unwrap();
        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();
    
    println!("Request: {:#?}", http_request);
}
```
Kode ini membaca permintaan HTTP yang masuk dan mencetaknya di terminal. Dengan demikian, kita dapat melihat detail request yang dikirim oleh browser, termasuk metode HTTP, header, dan informasi lainnya.

### 4. Pemahaman Tentang HTTP Request
Dari hasil eksekusi program, contoh request yang diterima bisa seperti ini:
```
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) Safari/605.1.15",
    "Accept-Language: en-US,en;q=0.9",
]
```
Informasi ini menunjukkan bahwa browser melakukan permintaan `GET` ke root path (`/`) menggunakan protokol HTTP 1.1. Header tambahan seperti `User-Agent` dan `Accept-Language` juga dikirim.

### 5. Tantangan dan Perbaikan
Saat menjalankan program, terdapat peringatan unused variable `stream`. Hal ini dapat diperbaiki dengan mengganti nama variabel menjadi `_stream` untuk menunjukkan bahwa variabel tidak digunakan secara eksplisit. Selain itu, jika server tidak dihentikan dengan `Ctrl + C`, program akan gagal dijalankan kembali karena port masih terikat oleh proses sebelumnya.


# Refleksi: Milestone 2 - Returning HTML

Pada milestone ini, saya belajar bagaimana sebuah program Rust dapat mengembalikan halaman HTML melalui koneksi TCP. Dengan memahami cara kerja fungsi `handle_connection`, saya mendapatkan wawasan tentang dasar-dasar HTTP dan bagaimana server web sederhana dapat dibuat menggunakan Rust.

## **Pemahaman tentang handle_connection**
Fungsi `handle_connection` bertanggung jawab untuk membaca permintaan dari klien dan merespons dengan file HTML yang telah disediakan. Beberapa konsep penting yang saya pelajari dari kode ini meliputi:

1. **Membaca Permintaan HTTP**
   - Program menggunakan `BufReader` untuk membaca data dari `TcpStream`.
   - Menggunakan `.lines()` untuk mendapatkan setiap baris dari permintaan HTTP.
   - Menggunakan `.take_while(|line| !line.is_empty())` untuk hanya membaca bagian header permintaan.

2. **Menyusun Respons HTTP**
   - Respons HTTP terdiri dari:
     - **Status Line**: `HTTP/1.1 200 OK` menandakan bahwa permintaan berhasil.
     - **Header**: `Content-Length` untuk menentukan panjang konten yang dikirim.
     - **Body**: Isi dari `hello.html` yang akan ditampilkan di browser.
   - Menggunakan `fs::read_to_string("hello.html")` untuk membaca file HTML dan menyisipkannya dalam respons.
   - Menggunakan `format!()` untuk membangun respons lengkap sebelum dikirim ke klien menggunakan `stream.write_all(response.as_bytes()).unwrap();`.

## **Kesalahan dan Pembelajaran Penting**
Dalam proses menjalankan kode ini, saya menghadapi beberapa tantangan, di antaranya:

1. **Error karena karakter non-UTF-8 dalam permintaan HTTP**
   - Menggunakan `.unwrap()` pada `.lines()` bisa menyebabkan crash jika permintaan HTTP memiliki karakter non-UTF-8.
   - Solusi: Menggunakan `from_utf8_lossy()` untuk menangani karakter yang tidak valid.

2. **Kesalahan dalam menentukan lokasi file `hello.html`**
   - File harus berada dalam direktori yang sama dengan program agar dapat ditemukan oleh `fs::read_to_string`.

3. **Menjalankan ulang server setelah perubahan kode**
   - Harus memastikan bahwa server lama telah dihentikan sebelum menjalankan `cargo run` kembali.


```markdown
![Commit 2 screen capture](/assets/images/commit2.png)
```

# Refleksi: Milestone 3 - Validating request and selectively responding

Pada milestone ini, saya belajar bagaimana sebuah program Rust dapat mengembalikan halaman HTML melalui koneksi TCP serta melakukan validasi permintaan untuk memberikan respons yang sesuai. Dengan memahami cara kerja fungsi `handle_connection`, saya mendapatkan wawasan tentang dasar-dasar HTTP dan bagaimana server web sederhana dapat dibuat menggunakan Rust.

## **Pemahaman tentang handle_connection**
Fungsi `handle_connection` bertanggung jawab untuk membaca permintaan dari klien dan merespons dengan file HTML yang telah disediakan. Beberapa konsep penting yang saya pelajari dari kode ini meliputi:

1. **Membaca Permintaan HTTP**
   - Program menggunakan `BufReader` untuk membaca data dari `TcpStream`.
   - Menggunakan `.lines()` untuk mendapatkan setiap baris dari permintaan HTTP.
   - Menggunakan `.take_while(|line| !line.is_empty())` untuk hanya membaca bagian header permintaan.

2. **Menyusun Respons HTTP**
   - Respons HTTP terdiri dari:
     - **Status Line**: `HTTP/1.1 200 OK` menandakan bahwa permintaan berhasil.
     - **Header**: `Content-Length` untuk menentukan panjang konten yang dikirim.
     - **Body**: Isi dari `hello.html` yang akan ditampilkan di browser.
   - Menggunakan `fs::read_to_string("hello.html")` untuk membaca file HTML dan menyisipkannya dalam respons.
   - Menggunakan `format!()` untuk membangun respons lengkap sebelum dikirim ke klien menggunakan `stream.write_all(response.as_bytes()).unwrap();`.

## **Menangani Permintaan yang Tidak Valid**
Sebelumnya, server hanya mengembalikan `hello.html` untuk semua permintaan. Namun, dalam aplikasi web yang lebih realistis, server harus dapat merespons dengan halaman "404 Not Found" jika permintaan tidak sesuai dengan halaman yang tersedia.

Untuk itu, saya memodifikasi `handle_connection` agar dapat memeriksa permintaan yang diterima dan memberikan respons yang sesuai:

1. **Mengekstrak Permintaan dari HTTP Request**
   - Menggunakan `.lines().next()` untuk mendapatkan baris pertama dari permintaan HTTP.
   - Mengekstrak path dari request untuk menentukan halaman yang diminta.

2. **Memilih Respons yang Sesuai**
   - Jika path adalah `/`, server mengirimkan `hello.html` dengan status `HTTP/1.1 200 OK`.
   - Jika path bukan yang dikenali, server mengembalikan halaman `404.html` dengan status `HTTP/1.1 404 NOT FOUND`.

## **Mengapa Refactoring Diperlukan?**
Refactoring diperlukan agar kode menjadi lebih terstruktur, mudah dibaca, dan diperluas di masa depan. Beberapa perbaikan yang dilakukan:

1. **Memisahkan logika pembacaan permintaan dari pembuatan respons.**
   - Sebelumnya, semua logika ada dalam satu fungsi besar.
   - Sekarang, ada pemisahan antara membaca request dan menentukan respons.

2. **Menghindari duplikasi kode.**
   - Respons HTTP dibangun dengan format yang serupa, sehingga lebih baik dibuat sebagai fungsi terpisah.

3. **Meningkatkan skalabilitas.**
   - Dengan struktur ini, akan lebih mudah menambahkan halaman baru di masa depan.


```markdown
![Commit 3 screen capture](/assets/images/commit3.png)
```

