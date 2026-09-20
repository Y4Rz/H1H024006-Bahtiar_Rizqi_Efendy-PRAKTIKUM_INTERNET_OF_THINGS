# Modul 2: Konfigurasi Jaringan WiFi pada ESP32

## Percobaan 2A: Konfigurasi Mode Station (STA)

### Program Utama (percobaan2A.ino)


```cpp
#include <WiFi.h>

const char* ssid     = "poco";
const char* password = "9876543210";

const int ledPin = 2;

void setup() {
  Serial.begin(115200);
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  Serial.print("Menghubungkan ke WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println();
  Serial.println("WiFi berhasil terhubung!");
  Serial.print("IP Address  : ");
  Serial.println(WiFi.localIP());
  Serial.print("MAC Address : ");
  Serial.println(WiFi.macAddress());
  Serial.print("RSSI (dBm)  : ");
  Serial.println(WiFi.RSSI());

  digitalWrite(ledPin, HIGH);
}

void loop() {
  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("Status: Terhubung");
  } else {
    Serial.println("Status: Terputus");
    digitalWrite(ledPin, LOW);
  }
  delay(5000);
}
```


### Uraian Program

**Baris 1: Import Library**


```cpp
#include <WiFi.h>
```


Library `WiFi.h` digunakan agar ESP32 dapat menjalankan fungsi-fungsi yang berkaitan dengan pengaturan dan komunikasi jaringan WiFi.

**Baris 3-4: Deklarasi Kredensial WiFi**


```cpp
const char* ssid     = "poco";
const char* password = "9876543210";
```


Bagian ini menyimpan nama jaringan dan password yang akan digunakan ESP32 untuk melakukan koneksi. Tipe `const char*` dipakai karena nilai tersebut bersifat tetap selama program dijalankan.

**Baris 6: Deklarasi Pin LED**


```cpp
const int ledPin = 2;
```


GPIO 2 dimanfaatkan sebagai indikator koneksi. LED berada pada kondisi HIGH ketika ESP32 tersambung ke WiFi dan LOW ketika koneksi terputus.

**Baris 8-11: Inisialisasi Serial dan LED**


```cpp
Serial.begin(115200);
pinMode(ledPin, OUTPUT);
digitalWrite(ledPin, LOW);
```


- `Serial.begin(115200)` membuka komunikasi serial dengan kecepatan 115200 baud agar informasi program dapat dipantau melalui Serial Monitor.
- `pinMode(ledPin, OUTPUT)` menetapkan GPIO 2 sebagai pin keluaran.
- `digitalWrite(ledPin, LOW)` memastikan LED dalam keadaan mati sebelum proses koneksi berhasil.

**Baris 14-15: Konfigurasi Mode Station dan Koneksi**


```cpp
WiFi.mode(WIFI_STA);
WiFi.begin(ssid, password);
```


- `WiFi.mode(WIFI_STA)` memilih mode Station sehingga ESP32 bertindak sebagai client yang bergabung ke jaringan WiFi yang tersedia.
- `WiFi.begin(ssid, password)` menjalankan proses penyambungan menggunakan SSID "poco" dan password "9876543210".

**Baris 17-21: Menunggu Koneksi**


```cpp
Serial.print("Menghubungkan ke WiFi");
while (WiFi.status() != WL_CONNECTED) {
  delay(500);
  Serial.print(".");
}
```


Blok `while` digunakan untuk menunggu hingga koneksi berhasil. Pemeriksaan dilakukan setiap 500 ms; selama status belum `WL_CONNECTED`, karakter titik (`.`) dicetak sebagai tanda proses masih berlangsung. Setelah terhubung, perulangan selesai.

**Baris 24-31: Menampilkan Informasi Jaringan**


```cpp
Serial.println("WiFi berhasil terhubung!");
Serial.print("IP Address  : ");  Serial.println(WiFi.localIP());
Serial.print("MAC Address : ");  Serial.println(WiFi.macAddress());
Serial.print("RSSI (dBm)  : ");  Serial.println(WiFi.RSSI());
```


- `WiFi.localIP()` digunakan untuk mengetahui alamat IP yang diberikan router kepada ESP32 melalui DHCP.
- `WiFi.macAddress()` mengambil MAC address unik milik perangkat ESP32.
- `WiFi.RSSI()` menunjukkan kekuatan sinyal WiFi dalam satuan dBm; nilai yang mendekati 0 menunjukkan sinyal yang lebih kuat.

**Baris 33: Menyalakan LED Indikator**


```cpp
digitalWrite(ledPin, HIGH);
```


LED dinyalakan untuk memberikan tanda bahwa ESP32 telah berhasil tersambung ke jaringan WiFi.

**Baris 36-44: Loop Pengecekan Status**


```cpp
void loop() {
  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("Status: Terhubung");
  } else {
    Serial.println("Status: Terputus");
    digitalWrite(ledPin, LOW);
  }
  delay(5000);
}
```


Pada fungsi `loop()`, status jaringan diperiksa kembali setiap 5 detik. Jika koneksi masih aktif, Serial Monitor menampilkan "Status: Terhubung". Jika koneksi hilang, LED dimatikan dan pesan "Status: Terputus" ditampilkan.

---

## Percobaan 2B: Konfigurasi Mode Access Point (AP)

### Program Utama (percobaan2B.ino)


```cpp
#include <WiFi.h>

const char* ap_ssid     = "ESP32_AccessPoint";
const char* ap_password = "12345678";

void setup() {
  Serial.begin(115200);

  WiFi.mode(WIFI_AP);
  WiFi.softAP(ap_ssid, ap_password);

  IPAddress apIP = WiFi.softAPIP();
  Serial.println("Access Point aktif!");
  Serial.print("SSID       : ");
  Serial.println(ap_ssid);
  Serial.print("IP Address : ");
  Serial.println(apIP);
}

void loop() {
  int jumlahClient = WiFi.softAPgetStationNum();
  Serial.print("Jumlah perangkat terhubung: ");
  Serial.println(jumlahClient);
  delay(5000);
}
```


### Uraian Program

**Baris 1: Import Library**


```cpp
#include <WiFi.h>
```


Library `WiFi.h` digunakan agar ESP32 dapat menjalankan fungsi-fungsi yang berkaitan dengan pengaturan dan komunikasi jaringan WiFi.

**Baris 3-4: Deklarasi Kredensial Access Point**


```cpp
const char* ap_ssid     = "ESP32_AccessPoint";
const char* ap_password = "12345678";
```


Variabel tersebut menentukan nama dan password jaringan Access Point yang dibuat ESP32. SSID akan terlihat pada daftar jaringan perangkat lain, sedangkan password harus memiliki minimal 8 karakter.

**Baris 6-7: Fungsi setup() dan Inisialisasi Serial**


```cpp
void setup() {
  Serial.begin(115200);
```


Fungsi `setup()` hanya dijalankan satu kali ketika ESP32 mulai bekerja. Perintah `Serial.begin(115200)` membuka komunikasi serial pada baud rate 115200 sehingga keluaran program dapat diamati melalui Serial Monitor.

**Baris 10-11: Konfigurasi Mode Access Point**


```cpp
WiFi.mode(WIFI_AP);
WiFi.softAP(ap_ssid, ap_password);
```


- `WiFi.mode(WIFI_AP)` menetapkan ESP32 sebagai Access Point sehingga perangkat tersebut berfungsi sebagai penyedia jaringan atau hotspot.
- `WiFi.softAP(ap_ssid, ap_password)` membuat Access Point menggunakan SSID "ESP32_AccessPoint" dan password "12345678".

**Baris 13-18: Menampilkan Informasi Access Point**


```cpp
IPAddress apIP = WiFi.softAPIP();
Serial.println("Access Point aktif!");
Serial.print("SSID       : ");  Serial.println(ap_ssid);
Serial.print("IP Address : ");  Serial.println(apIP);
```


- `WiFi.softAPIP()` memperoleh alamat IP bawaan Access Point ESP32, yaitu 192.168.4.1. Alamat ini digunakan ESP32 sebagai server/gateway pada jaringan tersebut.
- SSID dan IP Address kemudian ditampilkan pada Serial Monitor sehingga konfigurasi Access Point dapat diketahui.

**Baris 21-26: Loop Pengecekan Jumlah Client**


```cpp
void loop() {
  int jumlahClient = WiFi.softAPgetStationNum();
  Serial.print("Jumlah perangkat terhubung: ");
  Serial.println(jumlahClient);
  delay(5000);
}
```


- `WiFi.softAPgetStationNum()` digunakan untuk mengetahui jumlah client yang sedang tersambung ke Access Point ESP32.
- Hasil penghitungannya ditampilkan setiap 5 detik pada Serial Monitor, sehingga perubahan jumlah perangkat yang terhubung dapat diamati.

---

## Hasil Pengamatan

### Percobaan 2A: Konfigurasi Mode Station (STA)

#### Tabel 1. Hasil Pemantauan Koneksi WiFi Mode Station (STA)


| No. | Waktu (s) | Status Koneksi | IP Address | MAC Address | RSSI (dBm) | Status LED |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| 1 | 0 | Terhubung | 10.30.129.194 | 88:57:21:8D:7D:B8 | -55 | HIGH |
| 2 | 5 | Terhubung | 10.30.129.194 | 88:57:21:8D:7D:B8 | -45 | HIGH |
| 3 | 10 | Terhubung | 10.30.129.194 | 88:57:21:8D:7D:B8 | -54 | HIGH |
| 4 | 15 | Terhubung | 10.30.129.194 | 88:57:21:8D:7D:B8 | -55 | HIGH |
| 5 | 20 | Terhubung | 10.30.129.194 | 88:57:21:8D:7D:B8 | -52 | HIGH |
| 6 | 25 | Terhubung | 10.30.129.194 | 88:57:21:8D:7D:B8 | -55 | HIGH |
| 7 | 30 | Terhubung | 10.30.129.194 | 88:57:21:8D:7D:B8 | -50 | HIGH |
| 8 | 35 | Terhubung | 10.30.129.194 | 88:57:21:8D:7D:B8 | -39 | HIGH |
| 9 | 40 | Terhubung | 10.30.129.194 | 88:57:21:8D:7D:B8 | -49 | HIGH |
| 10 | 45 | Terhubung | 10.30.129.194 | 88:57:21:8D:7D:B8 | -51 | HIGH |


#### Tabel 2. Hasil Pengujian Mode Station (STA)


| No. | Kondisi Pengujian | SSID | Password | Status Koneksi | Output Serial Monitor | Keterangan |
| :---: | :--- | :---: | :---: | :---: | :---: | :---: |
| 1 | Kredensial Benar | poco | 9876543210 | Terhubung | WiFi connected | Berhasil terkoneksi |
| 2 | Password Salah | poco | 9876543210x | Tidak Terhubung | WiFi connection failed | Password tidak valid |
| 3 | SSID Salah | pocox | 9876543210 | Tidak Terhubung | WiFi connection failed | SSID tidak ditemukan |


---

### Percobaan 2B: Konfigurasi Mode Access Point (AP)

#### Tabel 3. Hasil Pengamatan Access Point (AP)


| No. | Parameter | Nilai Konfigurasi | Hasil Pengamatan |
| :---: | :--- | :--- | :--- |
| 1 | SSID | ESP32_AccessPoint | ESP32_AccessPoint |
| 2 | Password | 12345678 | Behasil digunakan |
| 3 | IP Address AP | 192.168.4.1 | 192.168.4.1 |
| 4 | Status Access Point | Aktif | Aktif |
| 5 | SSID terdeteksi pada client | Ya | Ya |
| 6 | Perangkat berhasil terhubung | Ya | Ya, 2 smartphone |


#### Tabel 4. Hasil Pemantauan Client yang Terhubung ke AP


| No. | Waktu (s) | Jumlah Client | Perangkat yang terhubung | Keterangan |
| :---: | :---: | :---: | :--- | :--- |
| 1 | 0 | 0 | - | Belum ada client |
| 2 | 5 | 0 | - | Belum ada client |
| 3 | 10 | 1 | 1 | Terhubung 1 client |
| 4 | 15 | 1 | 1 | Terhubung 1 client |
| 5 | 20 | 2 | 2 | Terhubung 2 client |
| 6 | 25 | 2 | 2 | Terhubung 2 client |
| 7 | 30 | 2 | 2 | Terhubung 2 client |
| 8 | 35 | 2 | 2 | Terhubung 2 client |
| 9 | 40 | 2 | 2 | Terhubung 2 client |
| 10 | 45 | 1 | 1 | Terhubung 1 client |

