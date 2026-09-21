# Jawaban Pertanyaan Praktikum & Analisis Modul 3

---

## 3.5.4 Percobaan 3A: Komunikasi Data Menggunakan HTTP

### Pertanyaan 1

**Gambarkan diagram alur (flowchart) proses pengiriman data melalui HTTP POST pada program di atas!**

### Jawaban

Alur proses program dapat digambarkan sebagai berikut:

```mermaid
flowchart TD
    A([Mulai]) --> B[Inisialisasi Serial & WiFi]
    B --> C{WiFi Terhubung?}
    C -- Tidak --> D[Tunggu 500 ms dan cetak "."]
    D --> C
    C -- Ya --> E[Masuk ke loop]
    E --> F{WiFi masih terhubung?}
    F -- Ya --> G[Inisialisasi HTTPClient]
    G --> H[Set URL dan Header JSON]
    H --> I[Buat data JSON suhu & kelembaban]
    I --> J[Serialisasi JSON menjadi String]
    J --> K[Kirim HTTP POST]
    K --> L{Response Code > 0?}
    L -- Ya --> M[Tampilkan Response Code & Response Body]
    L -- Tidak --> N[Tampilkan kode error]
    M --> O[http.end()]
    N --> O
    F -- Tidak --> P[Tunggu]
    O --> Q[Delay 10 detik]
    P --> Q
    Q --> E
```

Secara umum, ESP8266 terlebih dahulu melakukan koneksi ke jaringan WiFi. Setelah koneksi tersedia, program membentuk data sensor dalam format JSON, mengirimkannya menggunakan metode HTTP POST, kemudian membaca respons dari server. Proses tersebut diulang dengan interval 10 detik.

---

### Pertanyaan 2

**Apa fungsi dari perintah `http.addHeader("Content-Type", "application/json")` pada program tersebut?**

### Jawaban

Perintah tersebut digunakan untuk memberi informasi kepada server mengenai jenis data yang terdapat pada body HTTP request. Nilai `application/json` menunjukkan bahwa isi data yang dikirim oleh client menggunakan format JSON.

Dengan demikian, server dapat mengenali dan memproses isi request sebagai data JSON, bukan sebagai format data lainnya.

---

### Pertanyaan 3

**Jelaskan arti dari kode response HTTP 200 dan sebutkan salah satu contoh kode response HTTP lain beserta artinya!**

### Jawaban

HTTP response code **200** menunjukkan bahwa request yang dikirim oleh client telah berhasil diproses oleh server.

Salah satu contoh kode lainnya adalah:

- **404 Not Found** — server tidak menemukan resource atau endpoint yang diminta oleh client.

Pada hasil pengamatan praktikum, seluruh pengiriman memperoleh response code `200`, sehingga data JSON berhasil diterima oleh endpoint pengujian.

---

### Pertanyaan 4

**Modifikasi program agar ESP32 dapat mengirimkan data tambahan berupa waktu (dalam milidetik sejak dinyalakan menggunakan `millis()`) ke dalam JSON yang dikirim, dan berikan penjelasan di setiap baris kode yang ditambahkan dalam bentuk README.md!**

### Jawaban

Modifikasi dilakukan dengan menambahkan nilai `millis()` sebagai pasangan key-value baru pada objek JSON. Nilai tersebut menunjukkan waktu yang telah berlalu sejak perangkat mulai menjalankan program.

#### Kode Modifikasi

```cpp
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <ArduinoJson.h>

const char* ssid     = "poco";
const char* password = "9876543210";
const char* serverUrl = "https://httpbin.org/post"; // endpoint uji HTTP POST

WiFiClientSecure wifiClient;

void setup() {
  Serial.begin(115200);
  wifiClient.setInsecure();
  WiFi.begin(ssid, password);
  Serial.print("Menghubungkan ke WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();
  Serial.println("WiFi berhasil terhubung!");
}

void loop() {
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;
    http.begin(wifiClient, serverUrl);
    http.addHeader("Content-Type", "application/json");

    // Membuat objek data sensor dalam format JSON
    JsonDocument doc;
    doc["suhu"] = 28.5;
    doc["kelembaban"] = 65.0;

    // Menambahkan waktu sejak perangkat mulai berjalan
    doc["waktu"] = millis();

    String requestBody;
    serializeJson(doc, requestBody);

    Serial.print("Mengirim data: ");
    Serial.println(requestBody);

    // Mengirim data melalui HTTP POST
    int httpResponseCode = http.POST(requestBody);

    if (httpResponseCode > 0) {
      Serial.print("Kode Response HTTP: ");
      Serial.println(httpResponseCode);
      Serial.println("Isi Response:");
      Serial.println(http.getString());
    } else {
      Serial.print("Pengiriman gagal, kode error: ");
      Serial.println(httpResponseCode);
    }
    http.end();
  }
  delay(10000);
}
```

#### Penjelasan Bagian yang Ditambahkan

| Bagian Kode | Penjelasan |
|---|---|
| `doc["waktu"] = millis();` | Menambahkan pasangan `waktu` ke objek JSON. Nilainya berasal dari `millis()` dan menunjukkan jumlah milidetik yang telah berlalu sejak perangkat mulai menjalankan program. |
| `serializeJson(doc, requestBody);` | Mengubah objek JSON yang sekarang sudah memiliki data suhu, kelembaban, dan waktu menjadi bentuk teks untuk dikirim melalui HTTP POST. |

Contoh struktur data setelah modifikasi menjadi:

```json
{"suhu":28.5,"kelembaban":65.0,"waktu":12345}
```

Nilai `waktu` akan berubah sesuai lama perangkat telah berjalan.

---

## 3.6.4 Percobaan 3B: Komunikasi MQTT

### Pertanyaan 1

**Apa fungsi dari topic pada protokol MQTT, dan mengapa topic yang digunakan perlu dibuat unik?**

### Jawaban

Topic berfungsi sebagai alamat atau jalur yang digunakan untuk mengelompokkan pesan pada sistem MQTT. Publisher mengirimkan data ke topic tertentu, sedangkan subscriber menerima pesan dengan melakukan subscribe pada topic yang sama.

Topic sebaiknya dibuat unik agar data dari satu kelompok atau perangkat tidak tercampur dengan data milik kelompok atau perangkat lain. Pada praktikum ini digunakan topic:

`unsoed/tk245004/nim10/sensor`

---

### Pertanyaan 2

**Jelaskan fungsi dari perintah `client.loop()` yang dipanggil pada setiap iterasi `loop()`!**

### Jawaban

`client.loop()` digunakan untuk menjaga komunikasi antara perangkat dengan broker MQTT tetap berjalan. Perintah ini membantu library PubSubClient memproses komunikasi MQTT yang sedang berlangsung.

Pemanggilan `client.loop()` secara berkala diperlukan agar koneksi dan pertukaran pesan dengan broker dapat terus ditangani oleh ESP8266.

---

### Pertanyaan 3

**Apa yang akan terjadi apabila koneksi ke broker MQTT terputus di tengah program berjalan?**

### Jawaban

Apabila koneksi dengan broker terputus, kondisi `client.connected()` akan menjadi tidak terpenuhi. Program kemudian menjalankan fungsi `hubungkanMQTT()` untuk mencoba membuat koneksi kembali.

Fungsi tersebut akan terus mencoba menghubungkan perangkat ke broker. Jika percobaan belum berhasil, program menampilkan status kegagalan dan menunggu selama 2 detik sebelum mencoba kembali.

Dengan mekanisme tersebut, perangkat dapat melakukan pemulihan koneksi secara otomatis tanpa harus menjalankan ulang program secara manual.

---

### Program Percobaan 3B

Kode yang digunakan pada percobaan MQTT adalah sebagai berikut:

```cpp
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>

const char* ssid     = "poco";
const char* password = "9876543210";

const char* mqttServer = "broker.hivemq.com";
const int mqttPort = 1883;
const char* mqttTopic = "unsoed/tk245004/nim10/sensor";

WiFiClient espClient;
PubSubClient client(espClient);

void hubungkanWiFi() {
  WiFi.begin(ssid, password);
  Serial.print("Menghubungkan ke WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi berhasil terhubung!");
}

void hubungkanMQTT() {
  while (!client.connected()) {
    Serial.print("Menghubungkan ke broker MQTT...");
    String clientId = "ESP32Client-" + String(random(0xffff), HEX);
    if (client.connect(clientId.c_str())) {
      Serial.println("berhasil terhubung!");
    } else {
      Serial.print("gagal, rc=");
      Serial.print(client.state());
      Serial.println(" coba lagi dalam 2 detik");
      delay(2000);
    }
  }
}

void setup() {
  Serial.begin(115200);
  hubungkanWiFi();
  client.setServer(mqttServer, mqttPort);
}

void loop() {
  if (!client.connected()) {
    hubungkanMQTT();
  }
  client.loop();

  // Membuat data sensor dalam format JSON
  JsonDocument doc;
  doc["suhu"] = 28.5;
  doc["kelembaban"] = 65.0;

  char buffer[128];
  serializeJson(doc, buffer);

  // Mempublikasikan data ke topic MQTT
  client.publish(mqttTopic, buffer);
  Serial.print("Data terkirim ke topic ");
  Serial.print(mqttTopic);
  Serial.print(": ");
  Serial.println(buffer);

  delay(5000); // publish data setiap 5 detik
}

```

---

# 3.7 Pertanyaan Analisis

## Pertanyaan 1

**Uraikan hasil tugas pada praktikum yang telah dilakukan pada setiap percobaan!**

### Jawaban

### Percobaan 3A — HTTP

Pada percobaan HTTP, perangkat berhasil mengirimkan data dalam format JSON menuju endpoint `httpbin.org/post` menggunakan metode POST. Berdasarkan data pengamatan, pengiriman dilakukan pada interval 10 detik mulai dari waktu 0 sampai 90 detik.

Seluruh 10 kali pengiriman memperoleh **HTTP Response Code 200** dan status pengiriman tercatat **Berhasil**. Response body juga menunjukkan kembali data JSON yang dikirimkan, sehingga data dapat diverifikasi telah diterima oleh server.

### Percobaan 3B — MQTT

Pada percobaan MQTT, perangkat berhasil terhubung ke broker dan mempublikasikan data JSON ke topic `unsoed/tk245004/nim10/sensor`. Data tersebut kemudian diterima oleh subscriber dengan isi JSON yang sama.

Berdasarkan tabel pengamatan, seluruh 10 kali pengiriman pada rentang 0 hingga 90 detik berstatus **Terhubung** dan **Berhasil**, serta data yang diterima subscriber sesuai dengan data yang dipublikasikan. 

---

## Pertanyaan 2

**Bandingkan besar overhead data dan pola komunikasi antara protokol HTTP dan MQTT berdasarkan hasil percobaan yang telah dilakukan!**

### Jawaban

Perbedaan utama keduanya terletak pada mekanisme komunikasi yang digunakan.

| Aspek | HTTP | MQTT |
|---|---|---|
| Pola komunikasi | Request-response | Publish-subscribe |
| Perantara | Client berkomunikasi langsung dengan server/endpoint | Menggunakan broker MQTT |
| Pengiriman data | Client membuat request setiap kali data dikirim | Publisher mengirim pesan ke topic |
| Koneksi | Tidak harus dipertahankan secara terus-menerus | Dirancang untuk koneksi yang tetap terjaga |
| Overhead | Relatif lebih besar karena terdapat HTTP request/response dan header | Relatif kecil sehingga lebih efisien untuk komunikasi IoT |
| Cocok untuk | Pengiriman data periodik atau komunikasi berbasis web | Pertukaran data sensor secara kontinu |

Pada percobaan, HTTP melakukan pengiriman menuju endpoint menggunakan request POST dan menerima response dari server. Sementara itu, MQTT mengirim data ke broker melalui topic dan data tersebut diterima oleh subscriber.

---

## Pertanyaan 3

**Untuk skenario pengiriman data sensor secara terus-menerus setiap beberapa detik dalam jangka waktu lama, protokol manakah (HTTP atau MQTT) yang lebih sesuai digunakan? Jelaskan alasannya!**

### Jawaban

Untuk pola pengiriman data sensor secara terus-menerus setiap beberapa detik dalam waktu yang panjang, **MQTT lebih sesuai** berdasarkan karakteristik yang dijelaskan pada modul.

MQTT menggunakan pola publish-subscribe dan dirancang sebagai protokol komunikasi ringan dengan overhead yang relatif kecil. Selain itu, MQTT mendukung koneksi yang tetap terjaga sehingga sesuai untuk pertukaran data secara kontinu.

Sebaliknya, HTTP menggunakan pola request-response dan memiliki overhead header yang relatif lebih besar. HTTP lebih sesuai ketika perangkat perlu melakukan komunikasi periodik tanpa harus mempertahankan koneksi secara terus-menerus.

---

## Pertanyaan 4

**Bagaimana peran format JSON dalam mendukung interoperabilitas data antara perangkat IoT dan berbagai platform/aplikasi yang berbeda?**

### Jawaban

JSON berperan sebagai format pertukaran data yang sederhana, ringan, dan mudah dibaca oleh manusia maupun mesin. Struktur data JSON menggunakan pasangan **key-value**, sehingga informasi dapat disusun secara jelas.

Dalam praktikum ini, data seperti suhu dan kelembaban dikemas dalam JSON sebelum dikirim menggunakan HTTP maupun MQTT. Karena JSON merupakan format berbasis teks yang umum digunakan, data yang berasal dari ESP8266 dapat lebih mudah diproses oleh server, broker, aplikasi MQTT client, maupun platform IoT lain yang mendukung format tersebut.

Contoh data:

```json
{"suhu":28.5,"kelembaban":65}
```

Penggunaan format yang sama pada berbagai sisi komunikasi membantu perangkat pengirim dan penerima memahami struktur data yang dipertukarkan.

---

# Data Pengamatan Modul 3

## Percobaan 3A — Komunikasi Data Menggunakan HTTP

### Tabel 1. Hasil Pengiriman Data melalui HTTP POST

| No. | Waktu Pengiriman (s) | Data JSON yang Dikirim | HTTP Response Code | Response Body | Status Pengiriman |
| :---: | :---: | :---: | :---: | :---: | :---: |
| 1 | 0 | `{"suhu":28.5,"kelembaban":65}` | `200` | `data:"{\"suhu\":28.5,\"kelembaban\":65}"` | Berhasil |
| 2 | 10 | `{"suhu":28.5,"kelembaban":65}` | `200` | `data:"{\"suhu\":28.5,\"kelembaban\":65}"` | Berhasil |
| 3 | 20 | `{"suhu":28.5,"kelembaban":65}` | `200` | `data:"{\"suhu\":28.5,\"kelembaban\":65}"` | Berhasil |
| 4 | 30 | `{"suhu":28.5,"kelembaban":65}` | `200` | `data:"{\"suhu\":28.5,\"kelembaban\":65}"` | Berhasil |
| 5 | 40 | `{"suhu":28.5,"kelembaban":65}` | `200` | `data:"{\"suhu\":28.5,\"kelembaban\":65}"` | Berhasil |
| 6 | 50 | `{"suhu":28.5,"kelembaban":65}` | `200` | `data:"{\"suhu\":28.5,\"kelembaban\":65}"` | Berhasil |
| 7 | 60 | `{"suhu":28.5,"kelembaban":65}` | `200` | `data:"{\"suhu\":28.5,\"kelembaban\":65}"` | Berhasil |
| 8 | 70 | `{"suhu":28.5,"kelembaban":65}` | `200` | `data:"{\"suhu\":28.5,\"kelembaban\":65}"` | Berhasil |
| 9 | 80 | `{"suhu":28.5,"kelembaban":65}` | `200` | `data:"{\"suhu\":28.5,\"kelembaban\":65}"` | Berhasil |
| 10 | 90 | `{"suhu":28.5,"kelembaban":65}` | `200` | `data:"{\"suhu\":28.5,\"kelembaban\":65}"` | Berhasil |

**Keterangan:**
- **Data JSON yang Dikirim:** misalnya `{"suhu":28.5,"kelembaban":65.0}`
- **HTTP Response Code:** misalnya `200`
- **Response Body:** salin bagian response yang menunjukkan data diterima server.
- **Status Pengiriman:** berdasarkan hasil pengamatan, berhasil atau gagal.

---

---

## Percobaan 3B — Komunikasi MQTT

### Tabel 2. Hasil Komunikasi Data melalui MQTT

| No. | Waktu Pengiriman (s) | Status Broker | Topic | Data JSON yang Dipublish | Data Diterima Subscriber | Status Pengiriman |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| 1 | 0 | Terhubung | `unsoed/tk245004/nim10/sensor` | `{"suhu":28.5,"kelembaban":65}` | `{"suhu":28.5,"kelembaban":65}` | Berhasil |
| 2 | 10 | Terhubung | `unsoed/tk245004/nim10/sensor` | `{"suhu":28.5,"kelembaban":65}` | `{"suhu":28.5,"kelembaban":65}` | Berhasil |
| 3 | 20 | Terhubung | `unsoed/tk245004/nim10/sensor` | `{"suhu":28.5,"kelembaban":65}` | `{"suhu":28.5,"kelembaban":65}` | Berhasil |
| 4 | 30 | Terhubung | `unsoed/tk245004/nim10/sensor` | `{"suhu":28.5,"kelembaban":65}` | `{"suhu":28.5,"kelembaban":65}` | Berhasil |
| 5 | 40 | Terhubung | `unsoed/tk245004/nim10/sensor` | `{"suhu":28.5,"kelembaban":65}` | `{"suhu":28.5,"kelembaban":65}` | Berhasil |
| 6 | 50 | Terhubung | `unsoed/tk245004/nim10/sensor` | `{"suhu":28.5,"kelembaban":65}` | `{"suhu":28.5,"kelembaban":65}` | Berhasil |
| 7 | 60 | Terhubung | `unsoed/tk245004/nim10/sensor` | `{"suhu":28.5,"kelembaban":65}` | `{"suhu":28.5,"kelembaban":65}` | Berhasil |
| 8 | 70 | Terhubung | `unsoed/tk245004/nim10/sensor` | `{"suhu":28.5,"kelembaban":65}` | `{"suhu":28.5,"kelembaban":65}` | Berhasil |
| 9 | 80 | Terhubung | `unsoed/tk245004/nim10/sensor` | `{"suhu":28.5,"kelembaban":65}` | `{"suhu":28.5,"kelembaban":65}` | Berhasil |
| 10 | 90 | Terhubung | `unsoed/tk245004/nim10/sensor` | `{"suhu":28.5,"kelembaban":65}` | `{"suhu":28.5,"kelembaban":65}` | Berhasil |
