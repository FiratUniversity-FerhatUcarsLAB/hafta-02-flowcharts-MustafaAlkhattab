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

// 2. Ürün Kategorileri Arasında Gezinme (Döngü)
tekrar
    ekranYaz("Kategoriler: 1-Elektronik, 2-Giyim, 3-Kitap, 4-Çıkış")
    kategoriSecim ← oku()

    eğer kategoriSecim == 4 ise
        çık
    son

    ürünListesi ← kategoriyeGoreUrunleriGetir(kategoriSecim)
    ekranYaz("Seçili kategorideki ürünler:")
    urunleriListele(ürünListesi)

    // 3. Ürün Sepete Ekleme
    ekranYaz("Sepete eklemek istediğiniz ürün ID’sini girin (0: Ana menü):")
    urunID ← oku()

    eğer urunID != 0 ise
        stok ← stokKontrol(urunID)

        // 4. Stok Kontrolü (Koşul)
        eğer stok > 0 ise
            sepeteEkle(urunID)
            ekranYaz("Ürün sepete eklendi.")
        değilse
            ekranYaz("Stokta yok! Başka ürün seçiniz.")
        son
    son
tekrarSonu

// 5. Sepeti Görüntüleme ve Düzenleme (Döngü)
tekrar
    ekranYaz("Sepetiniz:")
    sepetiGoster()
    ekranYaz("1: Ürün çıkar  2: Miktar değiştir  3: İleri")
    secim ← oku()

    eğer secim == 1 ise
        ekranYaz("Çıkarmak istediğiniz ürün ID’sini girin:")
        id ← oku()
        sepettenCikar(id)
    eğer secim == 2 ise
        ekranYaz("Ürün ID ve yeni miktar girin:")
        id ← oku()
        miktar ← oku()
        miktarGuncelle(id, miktar)
    son
tekrarYap secim != 3

// 6. İndirim Kodu Uygulama (Koşul)
ekranYaz("İndirim kodunuz var mı? (E/H)")
cevap ← oku()
eğer cevap == "E" ise
    ekranYaz("Kodu giriniz:")
    kod ← oku()
    eğer indirimKoduGecerliMi(kod) ise
        indirimiUygula(kod)
        ekranYaz("İndirim uygulandı.")
    değilse
        ekranYaz("Geçersiz kod.")
    son
son

// 7. Minimum 50 TL Kontrolü
toplam ← sepetToplamHesapla()
eğer toplam < 50 ise
    ekranYaz("Sipariş verebilmek için minimum 50 TL alışveriş yapmalısınız.")
    Bitir
son

// 8. Kargo Ücreti Hesaplama (200 TL Üzeri Ücretsiz)
eğer toplam >= 200 ise
    kargoUcreti ← 0
    ekranYaz("Kargo ücretsiz.")
değilse
    kargoUcreti ← 30
    ekranYaz("Kargo ücreti: 30 TL")
son
genelToplam ← toplam + kargoUcreti

// 9. Ödeme Yöntemi Seçimi (Koşul)
ekranYaz("Ödeme yöntemi seçin: 1-Kredi Kartı, 2-Havale, 3-Kapıda Ödeme")
odemeSecim ← oku()

eğer odemeSecim == 1 ise
    ekranYaz("Kredi kartı bilgilerini giriniz.")
eğer odemeSecim == 2 ise
    ekranYaz("Havale için IBAN bilgisi gönderilecek.")
eğer odemeSecim == 3 ise
    ekranYaz("Kapıda ödeme seçildi.")
son

// 10. Sipariş Onayı
ekranYaz("Toplam tutar: " + genelToplam + " TL")
ekranYaz("Siparişi onaylıyor musunuz? (E/H)")
onay ← oku()

eğer onay == "E" ise
    siparisKaydet()
    ekranYaz("Siparişiniz başarıyla oluşturuldu. Teşekkür ederiz!")
değilse
    ekranYaz("Sipariş iptal edildi.")
son

Bitir
