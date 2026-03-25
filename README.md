#  Sistem Rekap Absensi Kelas | Java OOP
 
Aplikasi berbasis Java untuk mencatat dan merekap kehadiran siswa per sesi pelajaran. Guru memasukkan data diri dan mata pelajaran, lalu menginput daftar siswa beserta status kehadiran masing-masing. Hasil rekap ditampilkan di konsol dan dapat diekspor ke file CSV berformat tabel.
 
---
 
##  Deskripsi Kasus
 
Sistem ini mensimulasikan proses absensi manual yang biasa dilakukan guru di kelas, namun diotomatisasi melalui program Java. Setiap sesi absensi mencatat:
 
- **Satu guru** beserta mata pelajaran yang diajarkan
- **Banyak siswa** dengan NIS unik dan status kehadiran: `HADIR`, `IZIN`, `SAKIT`, atau `ALFA`
- **Rekap akhir** berupa rangkuman jumlah per status dan opsi ekspor ke CSV
 
---
 
##  Class Diagram

![Output](ClssDiagram.png)

---
 
##  Kode Program Java

```java
import java.util.*;
import java.io.*;

abstract class Orang {
    protected int id;
    protected String nama;

    public Orang(int id, String nama) {
        this.id = id;
        this.nama = nama;
    }

    public int getId() { return id; }
    public String getNama() { return nama; }
    public abstract String getRole();
}

enum Keterangan {
    HADIR, IZIN, SAKIT, ALFA
}

// guru yang ngajar di kelas, satu guru per sesi absensi
class Guru extends Orang {
    private String mapel;

    public Guru(String nama, String mapel) {
        super(1, nama);
        this.mapel = mapel;
    }

    public String getMapel() { return mapel; }

    @Override
    public String getRole() { return "Guru"; }
}

// data siswa per hari
class Siswa extends Orang {
    private int nis;
    private Keterangan status;

    public Siswa(String nama, int nis, Keterangan status) {
        super(2, nama);
        this.nis = nis;
        this.status = status;
    }

    public int getNis() { return nis; }
    public Keterangan getStatus() { return status; }

    @Override
    public String getRole() { return "Siswa"; }
}

class Validator {
    // nama boleh pakai titik/koma buat gelar, misal "Ir. Soekarno"
    public static boolean cekNama(String n) {
        return n != null && n.matches("[A-Za-z\\s.,'-]+") && n.length() >= 2;
    }

    // NIS sekolah umumnya 5-10 digit
    public static boolean cekNis(String n) {
        return n != null && n.matches("[0-9]{5,10}");
    }
}

class RekapAbsensi {
    private Guru guru;
    private List<Siswa> daftarSiswa = new ArrayList<>();

    public void setGuru(Guru g) { this.guru = g; }

    public void tambahSiswa(Siswa s) { daftarSiswa.add(s); }

    public void tampilRekap() {
        System.out.println("\n=== REKAP ABSENSI ===");
        System.out.println("Guru : " + guru.getNama() + " | Mapel: " + guru.getMapel());
        System.out.println("------------------------------------------------");

        int hadir = 0, izin = 0, sakit = 0, alfa = 0;

        for (Siswa s : daftarSiswa) {
            System.out.printf("%-20s | NIS: %-8d | %s%n",
                s.getNama(), s.getNis(), s.getStatus());

            switch (s.getStatus()) {
                case HADIR: hadir++; break;
                case IZIN:  izin++;  break;
                case SAKIT: sakit++; break;
                case ALFA:  alfa++;  break;
            }
        }

        System.out.println("------------------------------------------------");
        System.out.printf("Hadir: %d | Izin: %d | Sakit: %d | Alfa: %d%n",
            hadir, izin, sakit, alfa);
    }

    public void exportCSV(String namaFile) {
        try (PrintWriter pw = new PrintWriter(new FileWriter(namaFile))) {
            String garis = "+-------+----+--------------------+-----------+------------+";
            pw.println(garis);
            pw.printf("| %-5s | %-2s | %-18s | %-9s | %-10s |%n",
                "Role", "ID", "Nama", "NIS/Mapel", "Status");
            pw.println(garis);

            pw.printf("| %-5s | %-2d | %-18s | %-9s | %-10s |%n",
                "Guru", guru.getId(), guru.getNama(), guru.getMapel(), "-");

            for (Siswa s : daftarSiswa) {
                pw.printf("| %-5s | %-2d | %-18s | %-9d | %-10s |%n",
                    "Siswa", s.getId(), s.getNama(), s.getNis(), s.getStatus());
            }

            pw.println(garis);
            System.out.println("Rekap berhasil disimpan ke: " + namaFile);

        } catch (IOException e) {
            System.out.println("Gagal simpan file — pastikan folder tujuan bisa diakses: " + e.getMessage());
        }
    }
}

public class AbsensiApp {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        RekapAbsensi rekap = new RekapAbsensi();
        HashSet<Integer> nisYangSudahAda = new HashSet<>();

        System.out.println("=== INPUT DATA GURU ===");

        String namaGuru;
        while (true) {
            System.out.print("Nama guru: ");
            namaGuru = sc.nextLine().trim();
            if (Validator.cekNama(namaGuru)) break;
            System.out.println("Nama tidak valid — minimal 2 karakter, hanya huruf.");
        }

        System.out.print("Mata pelajaran: ");
        String mapel = sc.nextLine().trim();
        rekap.setGuru(new Guru(namaGuru, mapel));

        System.out.println("\n=== INPUT DATA SISWA ===");

        while (true) {
            String nama;
            while (true) {
                System.out.print("Nama siswa: ");
                nama = sc.nextLine().trim();
                if (Validator.cekNama(nama)) break;
                System.out.println("Nama tidak valid — hanya huruf dan tanda baca umum.");
            }

            int nis;
            while (true) {
                System.out.print("NIS (5-10 digit): ");
                String inputNis = sc.nextLine().trim();

                if (!Validator.cekNis(inputNis)) {
                    System.out.println("NIS harus 5-10 digit angka.");
                    continue;
                }

                nis = Integer.parseInt(inputNis);
                if (nisYangSudahAda.contains(nis)) {
                    System.out.println("NIS " + nis + " sudah terdaftar di sesi ini.");
                    continue;
                }

                nisYangSudahAda.add(nis);
                break;
            }

            Keterangan statusHadir;
            while (true) {
                System.out.print("Status (HADIR/IZIN/SAKIT/ALFA): ");
                String input = sc.nextLine().trim().toUpperCase();
                try {
                    statusHadir = Keterangan.valueOf(input);
                    break;
                } catch (IllegalArgumentException e) {
                    System.out.println("Status tidak dikenali — pilih salah satu: HADIR, IZIN, SAKIT, ALFA.");
                }
            }

            rekap.tambahSiswa(new Siswa(nama, nis, statusHadir));

            System.out.print("Tambah siswa lagi? (y/n): ");
            if (!sc.nextLine().trim().equalsIgnoreCase("y")) break;
        }

        rekap.tampilRekap();

        System.out.print("\nSimpan ke CSV? (y/n): ");
        if (sc.nextLine().trim().equalsIgnoreCase("y")) {
            rekap.exportCSV("rekap_absensi.csv");
        }

        sc.close();
    }
}
```

---

## Screenshot Output

Output Terminal :
![Output1](Output1.png)

Output CSV :
![Output2](Output2.png)

---

## Prinsip OOP yang Diterapkan
 
### 1. Abstraksi (Abstraction)
Class `Orang` bersifat `abstract`, tidak bisa diinstansiasi langsung, hanya menyediakan kerangka dasar berupa atribut `id` dan `nama`, serta method abstrak `getRole()`. Ini memodelkan konsep "orang" secara umum tanpa terikat ke peran spesifik.
 
```java
abstract class Orang {
    public abstract String getRole();
}
```
 
### 2. Pewarisan (Inheritance)
`Guru` dan `Siswa` mewarisi class `Orang`, sehingga tidak perlu mendefinisikan ulang atribut `id` dan `nama`. Setiap subclass hanya menambahkan atribut yang relevan dengan perannya masing-masing . `mapel` untuk guru, `nis` dan `status` untuk siswa.
 
```java
class Guru extends Orang { ... }
class Siswa extends Orang { ... }
```
 
### 3. Polimorfisme (Polymorphism)
Method `getRole()` di-override di setiap subclass untuk mengembalikan nilai yang berbeda. Kalau nanti ada peran baru (misal `StafTU`), cukup tambah subclass baru tanpa ubah logika yang sudah ada.
 
```java
// Guru
public String getRole() { return "Guru"; }
 
// Siswa
public String getRole() { return "Siswa"; }
```
 
### 4. Enkapsulasi (Encapsulation)
Semua atribut di setiap class bersifat `private` atau `protected`, cuma bisa diakses lewat method getter. Mencegah perubahan data secara langsung dari luar class.
 
```java
class Siswa extends Orang {
    private int nis;
    private Keterangan status;
 
    public int getNis() { return nis; }
    public Keterangan getStatus() { return status; }
}
```
 
### 5. Enum sebagai Type Safety
`Keterangan` didefinisikan sebagai `enum`, bukan `String` biasa, sehingga status kehadiran terbatas hanya pada nilai yang valid. Input selain `HADIR`, `IZIN`, `SAKIT`, `ALFA` langsung ditolak oleh `Keterangan.valueOf()` dan memunculkan `IllegalArgumentException`.
 
```java
enum Keterangan { HADIR, IZIN, SAKIT, ALFA }
```

---
 
## Keunikan yang Membedakan
 
### 1. Validasi NIS Berbasis Panjang Digit
Validator tidak hanya mengecek apakah input berupa angka, tapi juga memvalidasi panjangnya **5 sampai 10 digit** yang sesuai format NIS nyata di sekolah-sekolah Indonesia.
 
```java
public static boolean cekNis(String n) {
    return n != null && n.matches("[0-9]{5,10}");
}
```
 
### 2. Validasi Nama Mendukung Gelar
Regex validasi nama mendukung karakter titik dan koma, sehingga nama seperti `Drs. Kevin` atau `S.Pd., M.M.` tetap diterima.
 
```java
public static boolean cekNama(String n) {
    return n != null && n.matches("[A-Za-z\\s.,'-]+") && n.length() >= 2;
}
```
 
### 3. Deteksi NIS Duplikat dalam Satu Sesi
Menggunakan `HashSet<Integer>` untuk melacak NIS yang sudah diinput. Jika NIS yang sama dimasukkan dua kali, sistem langsung menolak dan menampilkan pesan spesifik, mencegah data siswa ganda dalam satu sesi absensi.
 
```java
if (nisYangSudahAda.contains(nis)) {
    System.out.println("NIS " + nis + " sudah terdaftar di sesi ini.");
}
```
 
### 4. Export CSV Berformat Tabel ASCII
Output CSV tidak hanya berupa baris data mentah, tapi diformat sebagai **tabel ASCII** dengan garis pembatas dan alignment kolom yang rapi bisa langsung dibaca tanpa perlu membuka di spreadsheet.
 
### 5. Pesan Error yang Kontekstual
Setiap error message menyebutkan alasan spesifik dan panduan perbaikan, bukan sekadar "input tidak valid". Termasuk mencetak `e.getMessage()` saat IOException agar pengguna tahu penyebab gagalnya ekspor file.
 
---




