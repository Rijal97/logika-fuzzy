# Praktikum 6 — Fuzzy Rule dan Knowledge Base
## Perancangan Aturan Proposisi IF-THEN, Basis Pengetahuan, dan Evaluasi Firing Strength dengan Python

---

## A. Tujuan Praktikum
Setelah menyelesaikan praktikum ini, mahasiswa diharapkan mampu:
1. Memahami konsep struktur proposisi aturan fuzzy: bagian *Antecedent* (premis/kondisi) dan *Consequent* (konklusi/tindakan).
2. Menyusun basis aturan (*Rule Base*) dan basis pengetahuan (*Knowledge Base*) berdasarkan logika pakar atau karakteristik domain masalah.
3. Menghitung nilai kekuatan pemicu (*Firing Strength* / $\alpha$-predikat) untuk aturan berkondisi majemuk (menggunakan operator AND/OR).
4. Mengimplementasikan struktur data Rule Base dalam bahasa pemrograman Python.
5. Melakukan pengujian evaluasi aturan terhadap berbagai kombinasi input data.
6. Menganalisis kondisi aktivasi aturan (*rule firing*) serta teknik menangani aturan yang tidak aktif.

---

## B. Konsep Dasar

### 1. Struktur Aturan Fuzzy IF-THEN
Aturan fuzzy merupakan bentuk penalaran logis yang digunakan untuk merepresentasikan keputusan berdasarkan kondisi tertentu.
Pada sistem penentuan prioritas pencetakan dokumen, terdapat dua variabel input dan satu variabel output.

Variabel input:

Waktu Tunggu: menunjukkan lamanya dokumen berada dalam antrean pencetakan.
- Cepat
- Sedang
- Lama

Urgensi Dokumen: menunjukkan tingkat kepentingan dokumen untuk segera dicetak.
- Rendah
- Sedang
- Tinggi

Variabel output:
Prioritas Cetak:
- Rendah
- Sedang
- Tinggi
- Kritis

Bentuk umum aturan fuzzy:

IF x is A AND y is B THEN z is C

Contoh aturan:
IF Waktu Tunggu is Lama AND Urgensi is Tinggi THEN Prioritas Cetak is Kritis.

Keterangan:
- Antecedent (Premis): Waktu Tunggu is Lama AND Urgensi is Tinggi.
- Consequent (Kesimpulan): Prioritas Cetak is Kritis.
- x: Variabel Waktu Tunggu.
- y: Variabel Urgensi Dokumen.
- z: Variabel Prioritas Cetak.

Ilustrasi struktur aturan:
          ANTECEDENT / PREMIS
┌──────────────────────────────────────────────┐
│ IF Waktu_Tunggu is LAMA                      │
│ AND Urgensi_Dokumen is TINGGI                │
└──────────────────────────────────────────────┘
                      |
                      v
          CONSEQUENT / KESIMPULAN
             ┌──────────────────────┐
             │ Prioritas_Cetak      │
             │ is KRITIS            │
             └──────────────────────┘

### 2. Nilai Kekuatan Pemicu (*Firing Strength* / $\alpha$-predikat)
Firing Strength merupakan nilai yang menunjukkan seberapa kuat suatu aturan fuzzy diaktifkan berdasarkan derajat keanggotaan input.
Nilai firing strength berada pada rentang 0 sampai 1.

Untuk operator AND menggunakan metode Zadeh Min:
αi = min(μA(x),μB(y))

Untuk operator OR menggunakan metode Zadeh Max:
αi = max(μAx),μB(y))

Keterangan:
- αi : Nilai firing strength aturan ke-i.
- μA(x) : Derajat keanggotaan variabel pertama.
- μB(y) : Derajat keanggotaan variabel kedua.

Semakin besar nilai firing strength, semakin kuat aturan tersebut terpicu.

### 3. Matriks Basis Aturan (*Rule Matrix*)
Pada sistem penentuan prioritas pencetakan dokumen terdapat:
- Variabel Waktu Tunggu memiliki 3 label linguistik.
- Variabel Urgensi Dokumen memiliki 3 label linguistik.
- Variabel Prioritas Cetak memiliki 4 label linguistik.

Jumlah aturan yang diperlukan:
3 × 3 = 9 aturan

Berikut rancangan matriks aturan:

Waktu Tunggu \ Urgensi   Rendah          Sedang          Tinggi
Cepat                    R1: Rendah      R2: Rendah      R3: Sedang
Sedang                   R4: Rendah      R5: Sedang      R6: Tinggi
Lama                     R7: Sedang      R8: Tinggi      R9: Kritis

Penjelasan logika aturan:
- Dokumen yang baru masuk dan memiliki urgensi rendah mendapatkan prioritas rendah.
- Dokumen dengan urgensi sedang dan waktu tunggu sedang mendapatkan prioritas sedang.
- Dokumen yang sudah lama mengantre dan memiliki urgensi tinggi mendapatkan prioritas kritis.
- Dokumen yang memiliki tingkat urgensi tinggi dapat memperoleh prioritas lebih tinggi meskipun waktu tunggunya belum terlalu lama.

## C. Contoh Perhitungan Manual Firing Strength
Misalkan terdapat dokumen yang akan dicetak dengan data input sebagai berikut.

Nilai keanggotaan Waktu Tunggu:
- Cepat = 0.0
- Sedang = 0.4
- Lama = 0.6

Nilai keanggotaan Urgensi Dokumen:
- Rendah = 0.0
- Sedang = 0.7
- Tinggi = 0.3

Selanjutnya dilakukan perhitungan firing strength menggunakan operator Zadeh Min.

1. **Aturan 5 (R5):**
   $$\textbf{IF } \text{Waktu is Sedang AND Urgensi is Sedang} \textbf{ THEN } \text{Prioritas is Sedang}$$
   $$\alpha_5 = \min(\mu_{\text{Sedang}}(Waktu), \mu_{\text{Sedang}}(Urgensi)) = \min(0.4, 0.7) = \mathbf{0.4}$$

2. **Aturan 6 (R6):**
   $$\textbf{IF } \text{Waktu is Sedang AND Urgensi is Tinggi} \textbf{ THEN } \text{Prioritas is Tinggi}$$
   $$\alpha_6 = \min(\mu_{\text{Sedang}}(Waktu), \mu_{\text{Tinggi}}(Urgensi)) = \min(0.4, 0.3) = \mathbf{0.3}$$

3. **Aturan 8 (R8):**
   $$\textbf{IF } \text{Waktu is Lama AND Urgensi is Sedang} \textbf{ THEN } \text{Prioritas is Tinggi}$$
   $$\alpha_8 = \min(\mu_{\text{Lama}}(Waktu), \mu_{\text{Sedang}}(Urgensi)) = \min(0.6, 0.7) = \mathbf{0.6}$$

4. **Aturan 9 (R9):**
   $$\textbf{IF } \text{Waktu is Lama AND Urgensi is Tinggi} \textbf{ THEN } \text{Prioritas is Kritis}$$
   $$\alpha_9 = \min(\mu_{\text{Lama}}(Waktu), \mu_{\text{Tinggi}}(Urgensi)) = \min(0.6, 0.3) = \mathbf{0.3}$$

Hasil perhitungan:
Aturan            Prioritas Cetak           Firing Strength
R5                Sedang                    0.4
R6                Tinggi                    0.3
R8                Tinggi                    0.6
R9                Kritis                    0.3

Aturan lainnya menghasilkan nilai 
α=0.0 karena setidaknya salah satu derajat keanggotaan pada premisnya bernilai nol.

Dengan demikian, terdapat empat aturan aktif dalam proses penentuan prioritas pencetakan dokumen.

## D. Implementasi dalam Python

Simpan script berikut sebagai `pertemuan6_fuzzy_rule.py`:
Program ini digunakan untuk mendefinisikan basis aturan, menghitung firing strength, dan menampilkan aturan yang aktif.

```python

# ==========================================================
# 1. KELAS STRUKTUR ATURAN FUZZY
# SISTEM PENENTUAN PRIORITAS PENCETAKAN DOKUMEN
# ==========================================================

class FuzzyRule:
    def __init__(self, rule_id, antecedent, consequent,
                 operator='AND'):

        self.rule_id = rule_id
        self.antecedent = antecedent
        self.consequent = consequent
        self.operator = operator.upper()

    def hitung_firing_strength(self, derajat_fuzzifikasi):

        nilai_derajat = []

        for var_nama, label_nama in self.antecedent.items():

            deg = derajat_fuzzifikasi[var_nama].get(
                label_nama, 0.0
            )

            nilai_derajat.append(deg)

        if not nilai_derajat:
            return 0.0

        if self.operator == 'AND':
            return min(nilai_derajat)

        elif self.operator == 'OR':
            return max(nilai_derajat)

        else:
            raise ValueError(
                f"Operator {self.operator} tidak dikenali."
            )

    def __str__(self):

        syarat = f" {self.operator} ".join(
            [f"{var} is {lbl}"
             for var, lbl in self.antecedent.items()]
        )

        hasil = ", ".join(
            [f"{var} is {lbl}"
             for var, lbl in self.consequent.items()]
        )

        return (
            f"[{self.rule_id}] IF {syarat} THEN {hasil}"
        )


# ==========================================================
# 2. INISIALISASI BASIS ATURAN (RULE BASE)
# ==========================================================

rule_base = [

    FuzzyRule("R1",
        {"waktu": "Cepat", "urgensi": "Rendah"},
        {"prioritas": "Rendah"}),

    FuzzyRule("R2",
        {"waktu": "Cepat", "urgensi": "Sedang"},
        {"prioritas": "Rendah"}),

    FuzzyRule("R3",
        {"waktu": "Cepat", "urgensi": "Tinggi"},
        {"prioritas": "Sedang"}),

    FuzzyRule("R4",
        {"waktu": "Sedang", "urgensi": "Rendah"},
        {"prioritas": "Rendah"}),

    FuzzyRule("R5",
        {"waktu": "Sedang", "urgensi": "Sedang"},
        {"prioritas": "Sedang"}),

    FuzzyRule("R6",
        {"waktu": "Sedang", "urgensi": "Tinggi"},
        {"prioritas": "Tinggi"}),

    FuzzyRule("R7",
        {"waktu": "Lama", "urgensi": "Rendah"},
        {"prioritas": "Sedang"}),

    FuzzyRule("R8",
        {"waktu": "Lama", "urgensi": "Sedang"},
        {"prioritas": "Tinggi"}),

    FuzzyRule("R9",
        {"waktu": "Lama", "urgensi": "Tinggi"},
        {"prioritas": "Kritis"}),
]


# ==========================================================
# 3. DATA INPUT HASIL FUZZIFIKASI
# ==========================================================

derajat_input_sampel = {

    "waktu": {
        "Cepat": 0.0,
        "Sedang": 0.4,
        "Lama": 0.6
    },

    "urgensi": {
        "Rendah": 0.0,
        "Sedang": 0.7,
        "Tinggi": 0.3
    }

}


# ==========================================================
# 4. EVALUASI ATURAN FUZZY
# ==========================================================

print("=" * 80)
print("SISTEM PENENTUAN PRIORITAS PENCETAKAN DOKUMEN")
print("=" * 80)

aturan_aktif = []

for rule in rule_base:

    alpha = rule.hitung_firing_strength(
        derajat_input_sampel
    )

    output_var = list(rule.consequent.keys())[0]
    output_lbl = rule.consequent[output_var]

    if alpha > 0:

        status = f"AKTIF (alpha = {alpha:.2f})"

        aturan_aktif.append(
            (rule.rule_id, output_lbl, alpha)
        )

    else:

        status = "NONAKTIF"

    print(f"{rule} --> {status}")


# ==========================================================
# 5. RINGKASAN ATURAN YANG TERPICU
# ==========================================================

print("\n" + "=" * 80)
print("RINGKASAN ATURAN AKTIF (FIRING RULES)")
print("=" * 80)

for r_id, konklusi, alpha in aturan_aktif:

    print(
        f"{r_id:<4} : Prioritas Cetak = "
        f"{konklusi:<8} dengan alpha = {alpha:.2f}"
    )


# ==========================================================
# 6. MENENTUKAN PRIORITAS DENGAN NILAI ALPHA TERBESAR
# ==========================================================

if aturan_aktif:

    hasil_tertinggi = max(
        aturan_aktif,
        key=lambda item: item[2]
    )

    print("\n" + "=" * 80)
    print("ATURAN DENGAN FIRING STRENGTH TERBESAR")
    print("=" * 80)

    print("Aturan       :", hasil_tertinggi[0])
    print("Prioritas    :", hasil_tertinggi[1])
    print("Nilai Alpha  :", hasil_tertinggi[2])

else:

    print("Tidak ada aturan yang aktif.")

---

**Contoh hasil keluaran program:**

================================================================================
SISTEM PENENTUAN PRIORITAS PENCETAKAN DOKUMEN
================================================================================

[R1] IF waktu is Cepat AND urgensi is Rendah
THEN prioritas is Rendah --> NONAKTIF

[R2] IF waktu is Cepat AND urgensi is Sedang
THEN prioritas is Rendah --> NONAKTIF

[R3] IF waktu is Cepat AND urgensi is Tinggi
THEN prioritas is Sedang --> NONAKTIF

[R4] IF waktu is Sedang AND urgensi is Rendah
THEN prioritas is Rendah --> NONAKTIF

[R5] IF waktu is Sedang AND urgensi is Sedang
THEN prioritas is Sedang --> AKTIF (alpha = 0.40)

[R6] IF waktu is Sedang AND urgensi is Tinggi
THEN prioritas is Tinggi --> AKTIF (alpha = 0.30)

[R7] IF waktu is Lama AND urgensi is Rendah
THEN prioritas is Sedang --> NONAKTIF

[R8] IF waktu is Lama AND urgensi is Sedang
THEN prioritas is Tinggi --> AKTIF (alpha = 0.60)

[R9] IF waktu is Lama AND urgensi is Tinggi
THEN prioritas is Kritis --> AKTIF (alpha = 0.30)

================================================================================
RINGKASAN ATURAN AKTIF (FIRING RULES)
================================================================================

R5   : Prioritas Cetak = Sedang   dengan alpha = 0.40
R6   : Prioritas Cetak = Tinggi   dengan alpha = 0.30
R8   : Prioritas Cetak = Tinggi   dengan alpha = 0.60
R9   : Prioritas Cetak = Kritis   dengan alpha = 0.30

================================================================================
ATURAN DENGAN FIRING STRENGTH TERBESAR
================================================================================

Aturan       : R8
Prioritas    : Tinggi
Nilai Alpha  : 0.6

## E. Skenario Pengujian Basis Aturan

Lakukan pengujian basis aturan untuk tiga skenario operasional pencetakan dokumen berikut.

**Skenario A — Dokumen Baru Masuk dan Tidak Mendesak**

Deskripsi kasus:
Terdapat dokumen baru yang masuk ke antrean pencetakan. Dokumen tersebut belum lama menunggu dan memiliki tingkat urgensi rendah.

Fuzzifikasi input:
- Waktu Tunggu:
  - Cepat = 1.0
  - Sedang = 0.0
  - Lama = 0.0
- Urgensi:
  - Rendah = 0.8
  - Sedang = 0.2
  - Tinggi = 0.0

Aturan yang terpicu:
Aturan             Perhitungan              Alpha
R1                 min(1.0, 0.8)            0.8
R2                 min(1.0, 0.2)            0.2

Kesimpulan:
Dokumen mendapatkan prioritas rendah karena baru masuk antrean dan tingkat urgensinya rendah.

**Skenario B — Dokumen Sudah Menunggu dan Mendesak**

Deskripsi kasus:
Terdapat dokumen yang sudah berada dalam antrean cukup lama. Dokumen tersebut memiliki tingkat urgensi sedang hingga tinggi.

Fuzzifikasi input:
- Waktu Tunggu:
  - Cepat = 0.0
  - Sedang = 0.7
  - Lama = 0.3
- Urgensi:
  - Rendah = 0.0
  - Sedang = 0.4
  - Tinggi = 0.6

Aturan yang terpicu:
Aturan             Perhitungan            Alpha
R5                 min(0.7, 0.4)          0.4
R6                 min(0.7, 0.6)          0.6
R8                 min(0.3, 0.4)          0.3
R9                 min(0.3, 0.6)          0.3

Kesimpulan:
Dokumen memiliki beberapa aturan aktif. Aturan R6 memiliki firing strength terbesar, yaitu 0.6, dengan konsekuen prioritas tinggi.

**Skenario C — Dokumen Sangat Penting dan Lama Mengantre**

Deskripsi kasus:
Terdapat dokumen penting yang telah lama berada dalam antrean pencetakan. Dokumen tersebut harus segera dicetak.

Fuzzifikasi input:
- Waktu Tunggu:
  - Cepat = 0.0
  - Sedang = 0.0
  - Lama = 1.0
- Urgensi:
  - Rendah = 0.0
  - Sedang = 0.0
  - Tinggi = 1.0

Aturan yang terpicu:
Aturan             Perhitungan           Alpha
R9                 min(1.0, 1.0)         1.0

Kesimpulan:
Dokumen mendapatkan prioritas kritis karena telah lama menunggu dan memiliki tingkat urgensi tinggi.

Ringkasan Pengujian
Skenario            Aturan Aktif          Prioritas Dominan
A                   R1, R2                Rendah
B                   R5, R6, R8, R9        Tinggi
C                   R9                    Kritis

## F. Tugas Praktikum 6

### Kasus: Sistem Penentuan Prioritas Pencetakan Dokumen
Sebuah tempat layanan pencetakan dokumen memiliki banyak permintaan pencetakan dari pelanggan. Setiap dokumen memiliki tingkat urgensi dan jumlah halaman yang berbeda. Oleh karena itu, diperlukan sistem berbasis logika fuzzy untuk membantu menentukan prioritas pencetakan dokumen agar dokumen yang lebih penting dapat diproses terlebih dahulu.
Dalam praktikum ini digunakan dua variabel input.

1. Variabel Waktu Tunggu
Memiliki tiga label linguistik:
- Cepat
- Sedang
- Lama

2. Variabel Urgensi Dokumen
Memiliki tiga label linguistik:
- Rendah
- Sedang
- Tinggi

Variabel output: Prioritas Cetak
Memiliki empat label linguistik:
- Rendah
- Sedang
- Tinggi
- Kritis

Instruksi Praktikum
1. Rancang tabel matriks basis aturan sebanyak 9 aturan (3 × 3) berdasarkan hubungan waktu tunggu dan urgensi dokumen.
2. Buat program Python untuk mendefinisikan seluruh aturan fuzzy tersebut.
3. Implementasikan fungsi perhitungan Firing Strength menggunakan operator AND dan metode Zadeh Min.
4. Jalankan pengujian evaluasi aturan dengan data input berikut.

Data input pengujian:
- Waktu Tunggu:
  - Cepat = 0.2
  - Sedang = 0.8
  - Lama = 0.0
- Urgensi Dokumen:
  - Rendah = 0.0
  - Sedang = 0.5
  - Tinggi = 0.5

Perhitungan Firing Strength
Aturan yang akan aktif adalah:

R5:
IF Waktu Tunggu is Sedang AND Urgensi is Sedang THEN Prioritas is Sedang.
α5 = min(0.8,0.5)=0.5

R6:
IF Waktu Tunggu is Sedang AND Urgensi is Tinggi THEN Prioritas is Tinggi.
α6 = min(0.8,0.5)=0.5

Aturan lainnya memiliki firing strength sebesar 0 karena setidaknya salah satu kondisi memiliki derajat keanggotaan nol.

Hasil yang Diharapkan
Aturan              Kondisi                Prioritas           Alpha
R5                  Sedang AND Sedang      Sedang              0.5
R6                  Sedang AND Tinggi      Tinggi              0.5

Dengan demikian, terdapat dua aturan aktif, yaitu R5 dan R6.
Kedua aturan memiliki firing strength yang sama, yaitu 0.5. Artinya, sistem menghasilkan dua kemungkinan konsekuen dengan tingkat aktivasi yang sama.

Pertanyaan tugas:
1. Mengapa R5 dan R6 menjadi aturan yang aktif?
2. Apa pengaruh tingkat urgensi dokumen terhadap prioritas pencetakan?
3. Bagaimana sistem menentukan keputusan jika terdapat dua aturan aktif dengan nilai alpha yang sama?
4. Mengapa dokumen yang sudah lama mengantre perlu mendapatkan perhatian lebih?
5. Bagaimana cara mengembangkan sistem agar dapat menentukan urutan pencetakan berdasarkan skor prioritas numerik?

## G. Pertanyaan Analisis

1. Apa yang akan terjadi pada sistem inferensi fuzzy jika suatu kombinasi input tidak memicu satu pun aturan (αi=0 untuk semua aturan)? Bagaimana cara mencegah kondisi tersebut?

Jawaban:
Jika tidak ada aturan yang aktif, sistem tidak dapat menghasilkan keputusan berdasarkan basis aturan yang tersedia. Hal ini dapat menyebabkan sistem tidak mampu menentukan prioritas pencetakan dokumen.
Cara mencegahnya adalah dengan memastikan seluruh kemungkinan kombinasi input sudah tercakup dalam basis aturan (completeness of rule base). Selain itu, dapat ditambahkan aturan default sebagai penanganan jika tidak terdapat aturan yang aktif.

2. Jika dua aturan yang aktif menghasilkan konsekuen yang sama, misalnya R6 dan R8 sama-sama menghasilkan Prioritas Tinggi, bagaimana kedua nilai alpha digabungkan pada tahap agregasi?

Jawaban:
Dalam metode inferensi Mamdani, nilai keanggotaan konsekuen dari setiap aturan dapat digabungkan menggunakan operator maksimum (MAX).

Contohnya:
- R6 menghasilkan prioritas tinggi dengan alpha 0.3.
- R8 menghasilkan prioritas tinggi dengan alpha 0.6.

Maka hasil agregasinya:
μTinggi = max(0.3,0.6)
μTinggi = 0.6

Dengan demikian, derajat aktivasi prioritas tinggi setelah agregasi adalah 0.6.

3. Mengapa urutan aturan (Rule Ordering) pada basis aturan fuzzy tidak memengaruhi hasil akhir sistem? Jelaskan perbedaannya dengan struktur if-elif-else!

Jawaban:
Dalam sistem fuzzy, setiap aturan dievaluasi secara independen berdasarkan derajat keanggotaan input. Semua aturan dapat aktif secara bersamaan dengan nilai firing strength yang berbeda.
Oleh karena itu, perubahan urutan aturan tidak mengubah nilai firing strength maupun hasil agregasi selama seluruh aturan tetap dievaluasi.
Hal ini berbeda dengan struktur pemrograman if-elif-else, di mana kondisi akan diperiksa secara berurutan dan ketika satu kondisi terpenuhi, kondisi berikutnya tidak lagi diperiksa.
Sementara itu, sistem fuzzy memungkinkan beberapa aturan menghasilkan keputusan secara bersamaan.

## H. Ketentuan & Pengumpulan
1. Mahasiswa mengumpulkan script Python `Modul-2/Praktikum6_<NIM>.py`.
2. Commit git: `feat(modul2): implementasi basis aturan dan firing strength praktikum 6`.

---

## I. Kesimpulan
Berdasarkan praktikum yang telah dilakukan, dapat disimpulkan bahwa sistem penentuan prioritas pencetakan dokumen menggunakan metode fuzzy dapat membantu menentukan tingkat kepentingan dokumen berdasarkan waktu tunggu dan urgensi dokumen.
Basis aturan fuzzy (Rule Base) digunakan untuk merepresentasikan pengetahuan manusia ke dalam bentuk aturan IF-THEN. Setiap aturan memiliki bagian antecedent sebagai kondisi dan consequent sebagai hasil keputusan.
Nilai Firing Strength atau α-predikat digunakan untuk mengetahui seberapa kuat suatu aturan diaktifkan berdasarkan derajat keanggotaan input. Perhitungan dilakukan menggunakan operator AND (MIN) atau OR (MAX).
Melalui implementasi Python, setiap aturan dapat dievaluasi secara independen sehingga beberapa aturan dapat aktif secara bersamaan. Hasil evaluasi tersebut kemudian digunakan dalam proses agregasi untuk menghasilkan keputusan prioritas pencetakan dokumen.
Dengan adanya sistem ini, proses pencetakan dokumen diharapkan menjadi lebih terstruktur, sehingga dokumen dengan tingkat urgensi tinggi dan waktu tunggu yang lama dapat memperoleh perhatian lebih dalam antrean pencetakan.
