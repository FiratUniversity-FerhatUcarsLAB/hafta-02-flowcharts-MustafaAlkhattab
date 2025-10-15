Başla

// 1. Öğrenci Girişi
ekranYaz("Öğrenci numarasını giriniz:")
ogrenciNo ← oku()
ekranYaz("Şifreyi giriniz:")
sifre ← oku()

eğer (girisKontrol(ogrenciNo, sifre) == FALSE) ise
    ekranYaz("Hatalı kullanıcı bilgileri! Giriş başarısız.")
    Bitir
değilse
    ekranYaz("Giriş başarılı, hoş geldiniz " + ogrenciNo)
son

// 2. Ders Listesini Görüntüleme
dersListesi ← dersleriGetir()
ekranYaz("Mevcut dersler:")
listele(dersListesi)

// 3. Ders Ekleme/Çıkarma Döngüsü
toplamKredi ← 0
secilenDersler ← []

tekrar
    ekranYaz("Bir işlem seçiniz:")
    ekranYaz("1 - Ders ekle")
    ekranYaz("2 - Ders çıkar")
    ekranYaz("3 - Kayıt özetini görüntüle ve onayla")
    secim ← oku()

    eğer (secim == 1) ise
        ekranYaz("Eklemek istediğiniz ders kodunu giriniz:")
        dersKod ← oku()
        ders ← dersiBul(dersKod)

        // --- Kontenjan kontrolü ---
        eğer (kontenjanMusaitMi(ders) == FALSE) ise
            ekranYaz("Bu dersin kontenjanı dolu!")
            devamEt
        son

        // --- Ön koşul kontrolü ---
        eğer (onKosulTamamMi(ogrenciNo, ders) == FALSE) ise
            ekranYaz("Bu dersi alabilmek için gerekli ön koşul derslerini tamamlamadınız!")
            devamEt
        son

        // --- Zaman çakışması kontrolü ---
        eğer (zamanCakisiyorMu(secilenDersler, ders) == TRUE) ise
            ekranYaz("Zaman çakışması tespit edildi! Bu ders mevcut programla çakışıyor.")
            devamEt
        son

        // --- Kredi limiti kontrolü ---
        eğer (toplamKredi + ders.kredi > 35) ise
            ekranYaz("Kredi limiti aşıldı! Maksimum 35 krediye izin veriliyor.")
            devamEt
        son

        // Ders ekleme işlemi
        secilenDersler ← secilenDersler + [ders]
        toplamKredi ← toplamKredi + ders.kredi
        ekranYaz("Ders başarıyla eklendi: " + ders.ad)

    eğer (secim == 2) ise
        ekranYaz("Çıkarmak istediğiniz ders kodunu giriniz:")
        dersKod ← oku()

        eğer (dersListedeMi(secilenDersler, dersKod) == FALSE) ise
            ekranYaz("Bu ders seçili değil!")
        değilse
            ders ← dersiBul(dersKod)
            secilenDersler ← secilenDersler - [ders]
            toplamKredi ← toplamKredi - ders.kredi
            ekranYaz("Ders başarıyla çıkarıldı: " + ders.ad)
        son

    eğer (secim == 3) ise
        // --- Kayıt Özeti ---
        ekranYaz("Kayıt özeti:")
        listele(secilenDersler)
        ekranYaz("Toplam kredi: " + toplamKredi)

        // Danışman onayı kontrolü
        gpa ← ogrenciGPA(ogrenciNo)
        eğer (gpa < 2.5) ise
            ekranYaz("GPA < 2.5. Danışman onayı gereklidir.")
            ekranYaz("Danışman onayı alındı mı? (E/H)")
            onay ← oku()
            eğer (onay == "H") ise
                ekranYaz("Danışman onayı olmadan kayıt tamamlanamaz.")
                devamEt
            son
        son

        // Kayıt onayı
        ekranYaz("Kaydı onaylamak istiyor musunuz? (E/H)")
        cevap ← oku()
        eğer (cevap == "E") ise
            kaydet(secilenDersler, ogrenciNo)
            ekranYaz("Kayıt başarıyla tamamlandı.")
            Bitir
        değilse
            ekranYaz("Kayıt iptal edildi.")
            Bitir
        son
    son

    ekranYaz("Başka işlem yapmak ister misiniz? (E/H)")
    devam ← oku()
tekrarYap (devam == "E")

ekranYaz("Sistemden çıkılıyor...")
Bitir
