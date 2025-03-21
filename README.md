# Refleksi: Membangun Web Server Sederhana dengan Rust

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
