# Modul 4 — Komunikasi dan Pertukaran Data

## Deskripsi Praktikum

Praktikum ini mempelajari pertukaran informasi dua arah pada perangkat IoT menggunakan ESP8266 dan protokol MQTT. Pada Percobaan 4A, perangkat menerima pesan melalui mekanisme *subscribe*, membaca data berformat JSON, lalu menggunakan perintah tersebut untuk mengatur LED. Percobaan 4B menggabungkan pengiriman data sensor dengan penerimaan instruksi dalam satu program. Pengiriman data dilakukan secara berkala menggunakan pendekatan non-blocking melalui `millis()` agar perangkat tetap bisa menangani pesan masuk.

## Alat dan Bahan

- Board ESP8266 DevKit / NodeMCU
- Sensor DHT11
- LED dan resistor 220 Ω
- Breadboard serta kabel jumper
- Kabel USB yang sesuai dengan board
- Komputer dengan Arduino IDE dan board support ESP8266
- Library `PubSubClient`, `ArduinoJson`, dan `DHT sensor library`
- Koneksi Wi-Fi dengan akses internet
- Aplikasi MQTT client, misalnya MQTT Explorer atau HiveMQ WebSocket Client
- Broker MQTT `broker.hivemq.com`, port `1883`

## Library yang Digunakan

| Library | Kegunaan |
|---|---|
| `ESP8266WiFi.h` | Menghubungkan board ESP8266 ke jaringan Wi-Fi. |
| `PubSubClient.h` | Menangani koneksi serta komunikasi MQTT melalui mekanisme publish dan subscribe. |
| `ArduinoJson.h` | Membantu mengubah data menjadi JSON dan membaca pesan JSON yang diterima. |
| `DHT.h` | Membaca informasi suhu dan kelembapan dari sensor DHT11 pada Percobaan 4B. |

> **Keamanan:** jangan mengunggah kata sandi Wi-Fi yang aktif ke repository publik. Ganti kredensial di dalam kode dengan placeholder sebelum melakukan commit, lalu masukkan kredensial lokal hanya saat pengujian.

---

## Percobaan 4A — Menerima JSON melalui MQTT untuk Mengendalikan LED

### Tujuan

Memahami cara ESP8266 menerima pesan dari topic MQTT, memproses isi JSON, dan menjalankan perintah untuk mengubah kondisi LED. Program yang dilampirkan juga mendukung pengaturan kecerahan LED menggunakan PWM.

### Konfigurasi Pin

| Komponen | Pin ESP8266 |
|---|---|
| LED melalui resistor | GPIO 14 (D5) |
| Ground LED | GND |

**Catatan:** pada kode yang diberikan, nilai `ledPin` adalah `14`, yaitu GPIO 14/D5. Komentar di dekat deklarasi pin menyebut D2/GPIO 4 sehingga tidak konsisten dengan nilai program. Pastikan kabel LED terpasang pada pin yang sesuai dengan nilai `ledPin`, atau sesuaikan kode dan rangkaian.

### Fungsi Penting pada Kode

| Fungsi / Kode | Penjelasan |
|---|---|
| `WiFi.begin(ssid, password)` | Memulai koneksi ke jaringan Wi-Fi dengan kredensial yang ditentukan. |
| `WiFi.status()` | Memeriksa status jaringan dan menunjukkan apakah perangkat sudah tersambung. |
| `PubSubClient client(espClient)` | Membuat objek client MQTT dengan koneksi Wi-Fi sebagai jalur komunikasinya. |
| `client.setServer(mqttServer, mqttPort)` | Menentukan alamat broker MQTT dan port yang digunakan. |
| `client.setCallback(callback)` | Menetapkan fungsi yang menangani pesan masuk dari topic yang diikuti. |
| `callback(topic, payload, length)` | Mengolah pesan masuk, menggabungkan payload menjadi teks, membaca JSON, dan mengatur LED sesuai perintah. |
| `client.connect(clientId.c_str())` | Menghubungkan client ke broker menggunakan Client ID. |
| `client.subscribe(topicPerintah)` | Mendaftarkan ESP8266 untuk menerima pesan dari topic perintah. |
| `client.connected()` | Memeriksa apakah koneksi ke broker masih aktif. |
| `client.loop()` | Memproses pesan MQTT yang masuk serta menjaga koneksi tetap berjalan. |
| `deserializeJson(doc, pesan)` | Mengubah teks JSON menjadi objek yang dapat dibaca program. |
| `doc["perintah"]` | Mengambil nilai dari field `perintah` di dalam JSON. |
| `analogWrite(ledPin, intensitas)` | Mengatur keluaran PWM untuk menentukan tingkat kecerahan LED. |
| `constrain(intensitas, 0, 1023)` | Memastikan nilai PWM berada dalam rentang yang diperbolehkan oleh program. |

### Alur Kerja Program

1. ESP8266 memulai koneksi Wi-Fi dan menunggu sampai jaringan tersambung.
2. Program mengatur broker MQTT dan mendaftarkan fungsi `callback()`.
3. Perangkat mencoba terhubung ke broker. Setelah berhasil, ESP8266 melakukan subscribe pada topic perintah.
4. Ketika pesan diterima, fungsi `callback()` menggabungkan byte payload menjadi teks.
5. Pesan diproses menggunakan `deserializeJson()`. Jika format JSON tidak valid, program menampilkan kesalahan pada Serial Monitor dan menghentikan pemrosesan pesan tersebut.
6. Jika JSON berhasil dibaca, program mengambil nilai `perintah`.
7. Perintah `ON` menyalakan LED dan menggunakan nilai `intensitas` untuk menentukan kecerahan. Jika field intensitas tidak tersedia, nilai bawaan yang digunakan adalah 1023.
8. Nilai intensitas dibatasi antara 0 dan 1023. Perintah `OFF` memberikan nilai PWM 0 sehingga LED mati.

### Broker, Topic, dan Format Pesan

```text
Broker : broker.hivemq.com
Port   : 1883
Topic  : unsoed/tk245004/AB3/perintah
```

Contoh payload untuk menyalakan LED dengan kecerahan penuh:

```json
{ "perintah": "ON", "intensitas": 1023 }
```

Contoh payload untuk menyalakan LED dengan tingkat kecerahan yang lebih rendah:

```json
{ "perintah": "ON", "intensitas": 800 }
```

Untuk mematikan LED, kirim:

```json
{ "perintah": "OFF" }
```

### Jawaban Pertanyaan Percobaan 4A

**1. Bagaimana urutan pemrosesan pesan pada callback?**

Pesan diterima dari topic langganan, kemudian `callback()` mengubah payload menjadi teks. Program mencoba membaca teks tersebut sebagai JSON. Jika terjadi kesalahan, pesan error ditampilkan dan proses dihentikan. Jika berhasil, nilai `perintah` diperiksa untuk menentukan tindakan terhadap LED.

**2. Apa yang terjadi jika pesan bukan JSON yang valid?**

Fungsi `deserializeJson()` menghasilkan error. Program menampilkan informasi kesalahan pada Serial Monitor dan menjalankan `return`, sehingga isi pesan tersebut tidak dipakai untuk mengubah kondisi LED.

**3. Mengapa subscribe dilakukan setelah koneksi MQTT berhasil?**

Subscribe membutuhkan koneksi aktif ke broker. Karena itu, `client.subscribe()` dipanggil setelah `client.connect()` berhasil. Dengan begitu, perangkat dapat mendaftarkan topic saat pertama kali terhubung maupun ketika perlu melakukan koneksi ulang.

**4. Bagaimana PWM digunakan untuk mengatur kecerahan LED?**

Program mengambil nilai `intensitas` dari JSON. Nilai tersebut dibatasi dalam rentang 0–1023, lalu diterapkan menggunakan `analogWrite()`. Nilai 0 membuat LED mati, sedangkan nilai 1023 memberikan keluaran maksimum. Jika intensitas tidak disertakan pada perintah `ON`, program memakai nilai default 1023.

### Catatan Kendala

Catat kendala yang benar-benar ditemui saat pengujian, seperti topic yang tidak cocok, format JSON yang salah, koneksi broker terputus, atau LED tidak merespons. Jika tidak ada masalah, tuliskan bahwa pengujian berjalan tanpa kendala berarti.

---

## Percobaan 4B — Publish dan Subscribe dalam Satu Sistem

### Tujuan

Menggabungkan pengiriman data sensor dan penerimaan perintah melalui MQTT dalam satu perangkat. ESP8266 mengirim pembacaan suhu dari DHT11 secara berkala dan tetap memproses instruksi kendali yang diterima dari broker.

### Konfigurasi Pin

| Komponen | Pin ESP8266 |
|---|---|
| DHT11 — VCC | 3.3V |
| DHT11 — DATA | GPIO 4 |
| DHT11 — GND | GND |
| LED melalui resistor | GPIO 26 |
| Ground LED | GND |
| Buzzer (+), jika memakai versi modifikasi | GPIO 27 |
| Buzzer (−), jika memakai versi modifikasi | GND |

### Fungsi Penting pada Program

| Fungsi / Kode | Penjelasan |
|---|---|
| `DHT dht(DHTPIN, DHTTYPE)` | Membuat objek sensor berdasarkan pin dan tipe sensor yang digunakan. |
| `dht.begin()` | Menginisialisasi sensor DHT11. |
| `dht.readTemperature()` | Mengambil data suhu dalam satuan Celsius. |
| `isnan(suhu)` | Memeriksa apakah hasil pembacaan suhu tidak valid. |
| `millis()` | Memberikan waktu sejak board mulai berjalan dan mendukung penjadwalan tanpa menghentikan program. |
| `serializeJson(doc, buffer)` | Mengubah objek JSON menjadi teks yang siap dikirim. |
| `client.publish(topic, buffer)` | Mengirim data sensor ke topic MQTT yang ditetapkan. |
| `client.subscribe(topic)` | Mendaftarkan topic agar perangkat menerima perintah dari broker. |
| `client.loop()` | Memeriksa pesan masuk secara berulang dan membantu menjaga koneksi MQTT tetap aktif. |

### Mekanisme Pengiriman Data

Program menggunakan `waktuTerakhirPublish` bersama `intervalPublish` untuk menentukan waktu pengiriman data berikutnya. Interval yang dicantumkan pada dokumentasi sumber adalah 5000 milidetik atau lima detik. Pada setiap putaran `loop()`, program memeriksa selisih waktu menggunakan `millis()`. Jika interval sudah tercapai, data suhu dibaca, diperiksa validitasnya, diubah ke JSON, kemudian dikirim ke topic data.

Pendekatan ini disebut non-blocking karena program tidak harus berhenti selama menunggu interval pengiriman. Dengan demikian, pemanggilan `client.loop()` tetap dapat berlangsung dan pesan kendali dapat diproses.

### Broker, Topic, dan Contoh Pesan

```text
Broker         : broker.hivemq.com
Port           : 1883
Topic Data     : unsoed/tk245004/AB3/data
Topic Perintah : unsoed/tk245004/AB3/perintah
```

Contoh data suhu yang dikirim perangkat:

```json
{ "suhu": 28.5 }
```

Contoh pesan kendali yang diterima perangkat:

```json
{ "perintah": "ON" }
{ "perintah": "OFF" }
```

### Modifikasi 4B — Mengendalikan LED dan Buzzer melalui Topic Terpisah

Pada versi modifikasi yang dijelaskan di dokumen sumber, buzzer memperoleh topic kendali tersendiri. Dengan pemisahan ini, callback dapat menentukan aktuator yang harus diubah berdasarkan topic asal pesan.

```text
Topic LED    : unsoed/tk245004/AB3/perintah
Topic Buzzer : unsoed/tk245004/AB3/perintah/buzzer
```

Masing-masing topic menggunakan payload JSON yang sama, yaitu `{"perintah":"ON"}` untuk mengaktifkan aktuator dan `{"perintah":"OFF"}` untuk menonaktifkannya. Program harus melakukan subscribe pada kedua topic setelah berhasil terhubung ke broker. Pin buzzer juga harus diatur sebagai output dan diberi kondisi awal yang sesuai.

### Jawaban Pertanyaan Percobaan 4B

**1. Mengapa `delay()` sebaiknya tidak digunakan terlalu lama?**

Saat `delay()` berjalan, program tidak dapat menjalankan bagian lain seperti pemrosesan pesan melalui `client.loop()`. Akibatnya, perintah yang datang dapat terlambat ditangani dan koneksi MQTT berisiko tidak terpelihara dengan baik.

**2. Bagaimana `millis()` membantu menjalankan proses non-blocking?**

Program membandingkan waktu saat ini dengan waktu publish terakhir. Jika selisihnya sudah mencapai interval yang ditentukan, data sensor dikirim dan waktu terakhir diperbarui. Selama menunggu, program tetap menjalankan proses lain, termasuk pemeriksaan pesan MQTT.

**3. Apa dampaknya jika `client.loop()` terlalu jarang dijalankan?**

Perangkat akan lebih lambat memproses pesan yang diterima. Selain itu, karena fungsi tersebut turut menjaga komunikasi dengan broker, pemanggilan yang terlalu jarang dapat mengganggu koneksi MQTT.

**4. Bagaimana menambahkan kendali buzzer?**

Buat topic khusus buzzer, atur pin buzzer sebagai output, dan daftarkan topic tersebut menggunakan `client.subscribe()` setelah koneksi broker berhasil. Di callback, periksa topic asal pesan sebelum menentukan apakah perintah dijalankan pada LED atau buzzer.

---

## Pertanyaan Analisis

### 1. Uraikan hasil setiap percobaan.

Pada Percobaan 4A, ESP8266 menerima perintah JSON melalui MQTT dan menggunakannya untuk mengendalikan LED, termasuk mengatur intensitas PWM jika nilai tersebut disertakan. Pada Percobaan 4B, perangkat mengirim data suhu secara berkala sekaligus menerima perintah kendali. Sesuaikan uraian ini dengan hasil yang benar-benar terlihat selama pengujian.

### 2. Apa perbedaan publish saja dengan publish dan subscribe?

Publish saja memungkinkan perangkat mengirim data ke broker untuk dipantau. Jika subscribe juga digunakan, perangkat dapat menerima instruksi dari aplikasi atau perangkat lain. Dengan demikian, komunikasi dua arah mendukung pemantauan sekaligus kendali jarak jauh.

### 3. Mengapa `millis()` lebih sesuai daripada `delay()` untuk komunikasi dua arah?

`delay()` menghentikan eksekusi program selama periode tertentu, sehingga penanganan pesan dapat tertunda. `millis()` memungkinkan program memeriksa interval waktu tanpa menghentikan keseluruhan proses. Hal ini membantu pengiriman data sensor tetap terjadwal sambil menjaga penerimaan perintah berjalan responsif.

### 4. Sebutkan contoh penggunaan komunikasi dua arah pada sistem IoT.

Sistem rumah pintar dapat mengirimkan informasi suhu ruangan ke dashboard dan menerima perintah dari pengguna untuk mengaktifkan atau mematikan kipas. Dibandingkan sistem yang hanya mengirim data, komunikasi dua arah memungkinkan pengguna mengambil tindakan dari jarak jauh berdasarkan informasi yang diterima.

---

## Kesimpulan

1. ESP8266 dapat menerima pesan MQTT melalui topic yang telah didaftarkan dan memprosesnya menggunakan fungsi callback.
2. Data JSON memungkinkan perangkat membaca perintah seperti `ON` dan `OFF` untuk mengendalikan aktuator.
3. PWM dapat digunakan untuk mengatur tingkat kecerahan LED melalui nilai intensitas.
4. Publish dan subscribe dapat digabungkan sehingga perangkat mengirim data sensor sekaligus menerima instruksi.
5. `millis()` mendukung pengiriman data berkala tanpa memblokir seluruh program seperti penggunaan `delay()` yang panjang.
6. Pemisahan topic data dan topic perintah membantu mengelola komunikasi, terutama ketika sistem memiliki lebih dari satu aktuator.

---

## Dokumentasi

Tambahkan bukti pelaksanaan praktikum ke repository, misalnya:

- **Flowchart:** diagram proses penerimaan dan pemrosesan pesan pada callback.
- **Foto rangkaian:** foto board ESP8266 beserta sensor dan aktuator yang digunakan.
- **Bukti perintah OFF:** tangkapan layar aplikasi MQTT dan/atau Serial Monitor ketika LED dimatikan.
- **Bukti perintah ON:** tangkapan layar ketika LED berhasil dinyalakan.
- **Hasil Percobaan 4B:** tangkapan layar data suhu yang dikirim serta bukti respons perangkat terhadap perintah kendali.

Pastikan nama file gambar dan tautan dokumentasi sesuai dengan file yang benar-benar ada di repository.
