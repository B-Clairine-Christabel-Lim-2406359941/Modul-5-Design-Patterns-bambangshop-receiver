# BambangShop Receiver App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a Rocket web framework skeleton that you can work with.

As this is an Observer Design Pattern tutorial repository, you need to implement a feature: `Notification`.
This feature will receive notifications of creation, promotion, and deletion of a product, when this receiver instance is subscribed to a certain product type.
The notification will be sent using HTTP POST request, so you need to make the receiver endpoint in this project.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Receiver" folder.

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    ROCKET_PORT=8001
    APP_INSTANCE_ROOT_URL=http://localhost:${ROCKET_PORT}
    APP_PUBLISHER_ROOT_URL=http://localhost:8000
    APP_INSTANCE_NAME=Safira Sudrajat
    ```
    Here are the details of each environment variable:
    | variable                | type   | description                                                     |
    |-------------------------|--------|-----------------------------------------------------------------|
    | ROCKET_PORT             | string | Port number that will be listened by this receiver instance.    |
    | APP_INSTANCE_ROOT_URL   | string | URL address where this receiver instance can be accessed.       |
    | APP_PUUBLISHER_ROOT_URL | string | URL address where the publisher instance can be accessed.       |
    | APP_INSTANCE_NAME       | string | Name of this receiver instance, will be shown on notifications. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)
3.  To simulate multiple instances of BambangShop Receiver (as the tutorial mandates you to do so),
    you can open new terminal, then edit `ROCKET_PORT` in `.env` file, then execute another `cargo run`.

    For example, if you want to run 3 (three) instances of BambangShop Receiver at port `8001`, `8002`, and `8003`, you can do these steps:
    -   Edit `ROCKET_PORT` in `.env` to `8001`, then execute `cargo run`.
    -   Open new terminal, edit `ROCKET_PORT` in `.env` to `8002`, then execute `cargo run`.
    -   Open another new terminal, edit `ROCKET_PORT` in `.env` to `8003`, then execute `cargo run`.

## Mandatory Checklists (Subscriber)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create SubscriberRequest model struct.`
    -   [ ] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Notification repository.`
    -   [ ] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 3: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Commit: `Implement receive_notification function in Notification service.`
    -   [ ] Commit: `Implement receive function in Notification controller.`
    -   [ ] Commit: `Implement list_messages function in Notification service.`
    -   [ ] Commit: `Implement list function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections

#### Reflection Subscriber-1

### 1. Penggunaan `RwLock<>` vs `Mutex<>` untuk Sinkronisasi `Vec`
Pada kasus ini, kita menggunakan `RwLock<>` (Read-Write Lock) alih-alih `Mutex<>` (Mutual Exclusion) karena perbedaan cara keduanya menangani akses konkurensi, yang sangat memengaruhi performa aplikasi.

* **`RwLock<>`** memungkinkan *banyak thread* untuk membaca (read) data secara bersamaan, asalkan tidak ada yang sedang menulis. Namun, jika ada thread yang ingin menulis (write/modify), thread tersebut akan mendapatkan akses eksklusif, dan thread pembaca lainnya harus menunggu.
* **`Mutex<>`** sangat ketat; ia hanya mengizinkan *satu thread* saja yang dapat mengakses data pada satu waktu, tidak peduli apakah thread tersebut hanya ingin membaca atau ingin menulis.

**Mengapa ini diperlukan untuk `Vec` Notifications?**
Dalam aplikasi *Receiver*, aksi untuk melihat daftar notifikasi (membaca data) jauh lebih sering terjadi dibandingkan aksi menerima notifikasi baru (menulis/menambah data). Dengan menggunakan `RwLock`, ratusan pengguna dapat melihat halaman notifikasi mereka secara bersamaan tanpa saling memblokir. Jika kita menggunakan `Mutex`, setiap pengguna yang hanya ingin melihat notifikasi harus menunggu giliran satu per satu, yang tentunya akan menciptakan *bottleneck* (kemacetan) dan menurunkan performa aplikasi secara drastis.

### 2. Penggunaan `lazy_static` dan Aturan Variabel Statis di Rust vs Java
Di bahasa pemrograman seperti Java, kita bisa dengan mudah membuat dan memodifikasi variabel global `static`. Sayangnya, kemudahan ini sering kali menjadi sumber masalah *data race* di lingkungan *multi-threading* jika tidak dikelola dengan blok *synchronized* yang benar, karena banyak thread bisa mengubah data tersebut bersamaan secara sembarangan.

Rust dirancang dengan prinsip **fearless concurrency** dan keamanan memori yang ketat di tingkat kompilator (*compile-time*). Oleh karena itu, Rust tidak mengizinkan kita memutasi variabel `static` secara langsung karena secara bawaan hal tersebut tidak aman (*unsafe*) dan pasti memicu *data race*.

Selain masalah keamanan, ada masalah **waktu inisialisasi**. Di Rust, variabel `static` harus memiliki nilai yang sudah diketahui dan dialokasikan saat kode dikompilasi (*compile-time*). Namun, struktur data kompleks seperti `Vec` atau `DashMap` memerlukan alokasi memori dinamis (*heap allocation*) yang hanya bisa dilakukan saat aplikasi sudah berjalan (*runtime*).

Di sinilah **`lazy_static`** (atau `once_cell`) sangat dibutuhkan. *Library* ini menyelesaikan dua masalah tersebut dengan cara:
1. Menunda (*lazy*) inisialisasi variabel statis tersebut sampai ia pertama kali dipanggil atau diakses saat *runtime*.
2. Membungkusnya agar aman diakses oleh banyak *thread* tanpa melanggar aturan keamanan memori Rust.

#### Reflection Subscriber-2
# BambangShop Receiver App - Reflection

**Nama:** Clairine Christabel Lim  
**NPM:** 2406359941  
**Kelas:** B

## Reflection Subscriber-2

### 1. Eksplorasi di Luar Langkah Tutorial (contoh: `src/lib.rs`)
Ya, saya menyempatkan diri untuk mengeksplorasi kode di luar langkah-langkah wajib tutorial, khususnya pada `src/lib.rs` dan struktur modul lainnya.

Dari eksplorasi tersebut, saya belajar tentang praktik terbaik (*best practices*) dalam pemrograman Rust, yaitu pemisahan antara *binary crate* (`src/main.rs`) dan *library crate* (`src/lib.rs`). File `src/main.rs` hanya bertugas sebagai titik masuk aplikasi (menjalankan *server*), sedangkan seluruh logika utama, definisi *struct*, dan pengelolaan *routing* diatur di dalam `src/lib.rs`. Saya juga melihat bagaimana *App State* (status aplikasi) didaftarkan dan dibagikan ke seluruh *handler* API sehingga setiap modul bisa mengakses konfigurasi yang sama tanpa harus membuat ulang *instance*.

### 2. Kemudahan Observer Pattern dan Tantangan Multi-Instance Main App
**Menambahkan lebih banyak *Subscriber*:**
Pola *Observer* membuat penambahan *Subscriber* baru (seperti menjalankan beberapa *instance* dari aplikasi Receiver) menjadi sangat mudah. Hal ini karena adanya **loose coupling** (keterikatan yang longgar). *Main app* (sebagai *Publisher*) tidak perlu tahu implementasi internal dari *Receiver*. Selama *Receiver* menyediakan *endpoint* URL yang valid untuk didaftarkan dan bisa menerima *payload* JSON yang disepakati, *Main app* tinggal menambahkan URL tersebut ke dalam daftarnya dan melakukan *broadcast* saat ada pembaruan.

**Menjalankan lebih dari satu *instance Main app*:**
Jika kita menjalankan beberapa *instance Main app*, penambahan sistem **tidak akan semudah sebelumnya**. Saat ini, daftar *Subscriber* disimpan di dalam memori internal aplikasi (menggunakan variabel statis `DashMap`). Jika ada dua *Main app* yang berjalan, masing-masing akan memiliki `DashMap` (daftar *subscriber*) sendiri yang saling tidak sinkron. Untuk menyelesaikannya dan membuat sistem bisa di-s*cale*, kita tidak bisa lagi menggunakan memori internal. Kita harus menggunakan *Message Broker* eksternal (seperti RabbitMQ, Kafka, atau Redis Pub/Sub) atau *database* terpusat agar semua *instance Main app* berbagi daftar *Subscriber* yang sama.

### 3. Pembuatan Test Mandiri dan Dokumentasi Postman
Ya, saya telah mencoba membuat dan mengeksplorasi dokumentasi *Collection* di Postman. Penggunaan fitur ini sangatlah berguna, baik untuk pengerjaan tutorial maupun untuk *Group Project* nantinya.

Manfaat utama yang saya rasakan:
* **Pengujian yang Konsisten:** Dengan Postman, saya bisa menyimpan berbagai skenario *request* (seperti *payload* yang benar, *payload* yang salah, atau data kosong) dan menjalankannya berulang kali tanpa harus mengetik ulang URL atau JSON *body* secara manual.
* **Kolaborasi Tim:** Dokumentasi API di Postman berfungsi sebagai "kontrak" yang jelas. Pada *Group Project*, tim *Frontend* tidak perlu membaca kode *Backend* Rust saya untuk tahu cara memanggil API. Mereka cukup melihat dokumentasi Postman yang sudah berisi contoh *endpoint*, *header* yang dibutuhkan, dan ekspektasi *response*. Hal ini sangat mempercepat proses integrasi antar tim.
