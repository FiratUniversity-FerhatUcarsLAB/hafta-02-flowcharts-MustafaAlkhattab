// --- 2. SİSTEM BAŞLANGIÇ VE AKTİVASYON KONTROLÜ ---
EKRANA_YAZ("Akıllı Ev Güvenlik Sistemi Başlatılıyor...")
// Kullanıcı girişinden/Uygulamadan SistemAktif değerini al (Örn: PIN ile)
EKRANA_YAZ("Sistemi aktif etmek için 'A' girin, çıkmak için 'C'.")
GİRİŞ KOMUT

EĞER KOMUT EŞİT "A" İSE
  SistemAktif = DOĞRU
  EKRANA_YAZ("Güvenlik Sistemi AKTİF edildi.")
DEĞİLSE
  EKRANA_YAZ("Sistem pasif kaldı. İyi günler.")
  GERİ_DÖN // Fonksiyondan çık
SON_EĞER

// --- 3. SÜREKLİ SENSÖR OKUMA DÖNGÜSÜ ---
SÜREKLİ_DÖNGÜ // Sistem çalıştığı sürece sürekli döner

  // --- 4. SİSTEM AKTİF Mİ KONTROLÜ ---
  EĞER SistemAktif EŞİT DOĞRU İSE

    // --- 5. SENSÖR OKUMALARI VE GÜNCELLEMELER ---
    // Sensörleri oku ve durumları güncelle
    HAREKET_VAR = HAREKET_SENSÖRÜ_OKU()
    KAPI_PENCERE_AÇIK = KAPI_PENCERE_SENSÖRÜ_OKU()
    EvSahibiEvdeMi = KONUM_VE_GİRİŞ_KONTROLÜ() // Ev sahibi evde/uzakta mı?

    // --- 6. OLAY TESPİTİ VE KARAR VERME ---

    EĞER HAREKET_VAR EŞİT DOĞRU VEYA KAPI_PENCERE_AÇIK EŞİT DOĞRU İSE

      EKRANA_YAZ("TEHLİKE TESPİT EDİLDİ: Hareket veya Açık Giriş Algılandı!")

      // --- 7. YANLIŞ ALARM KONTROLÜ (Ev Sahibi Evde mi?) ---
      EĞER EvSahibiEvdeMi EŞİT DOĞRU İSE
        // Eğer ev sahibi evdeyken hareket algılanırsa alarm seviyesi düşük olur.
        EKRANA_YAZ("Ev sahibi evde. Yanlış alarm ihtimali yüksek.")
        AlarmSeviyesi = 1 // Düşük: Yalnızca bilgilendirme
        Kamera_Aktivasyon(KISA) // Kısa kayıt başlat

        // Alarm Sıfırlama Mekanizması: Ev sahibinden onay iste
        EKRANA_YAZ("Yanlış alarmı sıfırlamak için onay kodu girin veya HİÇBİR ŞEY YAPMAYIN.")
        // 5 saniye bekleme süresi tanınır
        GİRİŞ ONAY_KODU // Kullanıcı uygulamadan ya da panelden kod girer

        EĞER ONAY_KODU EŞİT "SIFIRLA" İSE
          AlarmSeviyesi = 0
          EKRANA_YAZ("Alarm sıfırlandı. Yanlış alarm.")
          DEVAM_ET // Döngünün başına dön
        SON_EĞER

      DEĞİLSE // Ev sahibi evde değil, alarm ciddi!
        EKRANA_YAZ("Ev sahibi evde değil. Yüksek güvenlik ihlali!")

        // --- 8. ALARM SEVİYESİ BELİRLEME ---
        EĞER HAREKET_VAR EŞİT DOĞRU VE KAPI_PENCERE_AÇIK EŞİT DOĞRU İSE
          AlarmSeviyesi = 3 // Yüksek: Hem hareket hem giriş ihlali
        DEĞİLSE
          AlarmSeviyesi = 2 // Orta: Tek başına hareket veya giriş ihlali
        SON_EĞER

        // --- 9. KAMERA AKTİVASYONU ---
        Kamera_Aktivasyon(UZUN_KAYIT) // Uzun, sürekli kayıt başlat

        // --- 10. BİLDİRİM GÖNDERME ---
        Bildirim_Gonder(AlarmSeviyesi, "Müdahale Gerekiyor!")

      SON_EĞER // Ev Sahibi Kontrolü Sonu

    DEĞİLSE
      AlarmSeviyesi = 0
      EKRANA_YAZ("Durum Normal. Bekleniyor...")
    SON_EĞER // Olay Tespiti Sonu

  DEĞİLSE // SistemAktif EŞİT YANLIŞ ise
    EKRANA_YAZ("Sistem pasif durumda. Sensörler kontrol edilmiyor.")
  SON_EĞER

  // --- 11. BEKLE VE TEKRAR KONTROL ET DÖNGÜSÜ ---
  BEKLE(BEKLEME_SÜRESİ_SN) // 5 saniye bekle
SON_SÜREKLİ_DÖNGÜ
