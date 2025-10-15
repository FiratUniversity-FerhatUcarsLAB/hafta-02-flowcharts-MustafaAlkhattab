Başla

// 1. Kullanıcı Giriş Kontrolü
ekranYaz("Kullanıcı adı giriniz:")
kullaniciAdi ← oku()
ekranYaz("Şifre giriniz:")
sifre ← oku()

eğer (girisKontrol(kullaniciAdi, sifre) == FALSE) ise
    ekranYaz("Giriş başarısız! Kullanıcı adı veya şifre hatalı.")
    Bitir
değilse
    ekranYaz("Giriş başarılı, hoş geldiniz " + kullaniciAdi)
son

// 2. Hasta Kimlik Doğrulama (TC No)
ekranYaz("Lütfen TC Kimlik Numaranızı giriniz:")
tc ← oku()

eğer (tcDogruMu(tc) == FALSE) ise
    ekranYaz("Geçersiz TC Kimlik Numarası!")
    Bitir
değilse
    ekranYaz("Kimlik doğrulandı.")
son

// 3. İşlem Seçimi
tekrar
    ekranYaz("İşlem Seçiniz:")
    ekranYaz("1 - Randevu Al")
    ekranYaz("2 - Tahlil Sonucu Gör")
    ekranYaz("3 - Çıkış")
    secim ← oku()

    eğer secim == 1 ise
        // --------------------- RANDEVU MODÜLÜ ---------------------
        ekranYaz("Randevu Modülü")

        // Poliklinik Seçimi
        poliklinikler ← ["Dahiliye", "Kardiyoloji", "Göz", "Ortopedi"]
        ekranYaz("Poliklinik Seçiniz:")
        listele(poliklinikler)
        poliSecim ← oku()

        // Doktor Listesi ve Uygun Saatler (Döngü)
        doktorListesi ← doktorlariGetir(poliSecim)
        ekranYaz("Mevcut Doktorlar:")
        listele(doktorListesi)
        doktorSecim ← oku()

        ekranYaz("Uygun Saatler:")
        uygunSaatler ← saatleriGetir(doktorSecim)
        listele(uygunSaatler)
        saatSecim ← oku()

        eğer (randevuMusaitMi(doktorSecim, saatSecim)) ise
            ekranYaz("Randevu alındı: " + doktorSecim + " - " + saatSecim)
            ekranYaz("SMS ile bilgilendirme gönderiliyor...")
            smsGonder(tc, "Randevunuz oluşturuldu.")
        değilse
            ekranYaz("Seçilen saat dolu. Lütfen başka saat seçiniz.")
        son

    eğer secim == 2 ise
        // --------------------- TAHLİL MODÜLÜ ---------------------
        ekranYaz("Tahlil Sonuçları Modülü")

        // Tahlil Var mı Kontrolü
        eğer (tahlilVarMi(tc) == FALSE) ise
            ekranYaz("Henüz kayıtlı bir tahliliniz yok.")
        değilse
            // Sonuç Hazır mı Kontrolü
            eğer (tahlilSonucuHazirMi(tc) == TRUE) ise
                ekranYaz("Tahlil Sonuçlarınız:")
                sonucGoster(tc)

                ekranYaz("Sonuçları PDF olarak indirmek ister misiniz? (E/H)")
                cevap ← oku()
                eğer cevap == "E" ise
                    pdfIndir(tc)
                    ekranYaz("Sonuçlar PDF olarak indirildi.")
                son
            değilse
                ekranYaz("Tahlil sonuçlarınız henüz hazırlanıyor, lütfen daha sonra tekrar deneyin.")
            son
        son

    eğer secim == 3 ise
        ekranYaz("Sistemden çıkılıyo
