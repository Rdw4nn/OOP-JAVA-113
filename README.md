#  Sistem Rekap Absensi Kelas — Java OOP
 
Aplikasi berbasis Java untuk mencatat dan merekap kehadiran siswa per sesi pelajaran. Guru memasukkan data diri dan mata pelajaran, lalu menginput daftar siswa beserta status kehadiran masing-masing. Hasil rekap ditampilkan di konsol dan dapat diekspor ke file CSV berformat tabel.
 
---
 
##  Deskripsi Kasus
 
Sistem ini mensimulasikan proses absensi manual yang biasa dilakukan guru di kelas, namun diotomatisasi melalui program Java. Setiap sesi absensi mencatat:
 
- **Satu guru** beserta mata pelajaran yang diajarkan
- **Banyak siswa** dengan NIS unik dan status kehadiran: `HADIR`, `IZIN`, `SAKIT`, atau `ALFA`
- **Rekap akhir** berupa rangkuman jumlah per status dan opsi ekspor ke CSV
 
---
 
##  Class Diagram

![Output](https://github.com/Rdw4nn/test/blob/main/kebutuhan/ClssDiagram.png)

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







