PROGRAM ATM_Para_Cekme

// Sabitler
CONST MAX_PIN_DENEME = 3
CONST MIN_CEKIM_TUTARI = 20
CONST TUTAR_KATI = 20
CONST GUNLUK_LIMIT = 5000

// Değişkenler
DEGISKEN pin_deneme_sayisi: INTEGER
DEGISKEN kullanici_pin: STRING
DEGISKEN dogru_pin: STRING
DEGISKEN hesap_bakiye: DECIMAL
DEGISKEN cekilecek_tutar: DECIMAL
DEGISKEN gunluk_cekilen: DECIMAL
DEGISKEN baska_islem: BOOLEAN
DEGISKEN kart_bloke: BOOLEAN

BASLA
    YAZDIR "ATM'ye Hoş Geldiniz"
    YAZDIR "Lütfen kartınızı takınız"
    
    // Kart takma ve okuma
    kart_no = KART_OKU()
    EGER kart_no == NULL VEYA kart_no == GECERSIZ ISE
        YAZDIR "Geçersiz kart! Lütfen tekrar deneyiniz."
        CIK
    SON EGER
    
    // PIN doğrulama
    pin_deneme_sayisi = 0
    kart_bloke = KONTROL_KART_BLOKE(kart_no)
    
    EGER kart_bloke == DOGRU ISE
        YAZDIR "Kartınız bloke edilmiştir. Lütfen bankanızla iletişime geçiniz."
        KART_IAD_ET()
        CIK
    SON EGER
    
    dogru_pin = VERI_TABANINDAN_PIN_GETIR(kart_no)
    
    TEKRARLA
        YAZDIR "Lütfen PIN kodunuzu giriniz:"
        kullanici_pin = GIZLI_GIRIS_AL()
        
        EGER kullanici_pin == dogru_pin ISE
            YAZDIR "PIN doğrulandı"
            CIK_DONGUYU
        DEGILSE
            pin_deneme_sayisi = pin_deneme_sayisi + 1
            kalan_deneme = MAX_PIN_DENEME - pin_deneme_sayisi
            
            EGER pin_deneme_sayisi >= MAX_PIN_DENEME ISE
                YAZDIR "3 hatalı PIN girişi yapıldı!"
                YAZDIR "Kartınız güvenlik nedeniyle bloke edilmiştir."
                KART_BLOKE_ET(kart_no)
                KART_IAD_ET()
                CIK
            DEGILSE
                YAZDIR "Hatalı PIN! Kalan deneme hakkı: " + kalan_deneme
            SON EGER
        SON EGER
    TEKRAR_EDILIR pin_deneme_sayisi < MAX_PIN_DENEME
    
    // Ana işlem döngüsü
    baska_islem = DOGRU
    
    TEKRARLA_SURESI_BOYUNCA (baska_islem == DOGRU)
        
        // Bakiye sorgulama
        hesap_bakiye = BAKIYE_GETIR(kart_no)
        gunluk_cekilen = GUNLUK_CEKILEN_GETIR(kart_no)
        
        YAZDIR "===================="
        YAZDIR "Hesap Bakiyeniz: " + hesap_bakiye + " TL"
        YAZDIR "Bugün Çekilen: " + gunluk_cekilen + " TL"
        YAZDIR "Günlük Limit: " + GUNLUK_LIMIT + " TL"
        YAZDIR "===================="
        
        // Tutar girişi
        gecerli_tutar = YANLIS
        
        TEKRARLA_SURESI_BOYUNCA (gecerli_tutar == YANLIS)
            YAZDIR ""
            YAZDIR "Çekmek istediğiniz tutarı giriniz:"
            YAZDIR "(Not: Tutar " + TUTAR_KATI + " TL'nin katı olmalıdır)"
            cekilecek_tutar = SAYI_GIRIS_AL()
            
            // 20 TL katları kontrolü
            EGER (cekilecek_tutar MOD TUTAR_KATI) != 0 ISE
                YAZDIR "HATA: Tutar " + TUTAR_KATI + " TL'nin katı olmalıdır!"
                DEVAM_ET
            SON EGER
            
            // Minimum tutar kontrolü
            EGER cekilecek_tutar < MIN_CEKIM_TUTARI ISE
                YAZDIR "HATA: Minimum çekim tutarı " + MIN_CEKIM_TUTARI + " TL'dir!"
                DEVAM_ET
            SON EGER
            
            // Bakiye kontrolü
            EGER cekilecek_tutar > hesap_bakiye ISE
                YAZDIR "HATA: Yetersiz bakiye!"
                YAZDIR "Hesabınızdaki bakiye: " + hesap_bakiye + " TL"
                YAZDIR "Başka bir tutar girmek ister misiniz? (E/H)"
                secim = GIRIS_AL()
                EGER secim == "H" VEYA secim == "h" ISE
                    CIK_DONGUYU
                SON EGER
                DEVAM_ET
            SON EGER
            
            // Günlük limit kontrolü
            toplam_cekim = gunluk_cekilen + cekilecek_tutar
            EGER toplam_cekim > GUNLUK_LIMIT ISE
                kalan_limit = GUNLUK_LIMIT - gunluk_cekilen
                YAZDIR "HATA: Günlük çekim limiti aşıldı!"
                YAZDIR "Bugün çekebileceğiniz maksimum tutar: " + kalan_limit + " TL"
                YAZDIR "Başka bir tutar girmek ister misiniz? (E/H)"
                secim = GIRIS_AL()
                EGER secim == "H" VEYA secim == "h" ISE
                    CIK_DONGUYU
                SON EGER
                DEVAM_ET
            SON EGER
            
            // Tüm kontroller başarılı
            gecerli_tutar = DOGRU
            
        SON TEKRARLA
        
        // İşlem onayı
        EGER gecerli_tutar == DOGRU ISE
            YAZDIR ""
            YAZDIR "===================="
            YAZDIR "İŞLEM ÖZETİ"
            YAZDIR "Çekilecek Tutar: " + cekilecek_tutar + " TL"
            YAZDIR "İşlem Sonrası Bakiye: " + (hesap_bakiye - cekilecek_tutar) + " TL"
            YAZDIR "===================="
            YAZDIR "İşlemi onaylıyor musunuz? (E/H)"
            onay = GIRIS_AL()
            
            EGER onay == "E" VEYA onay == "e" ISE
                // Para verme işlemi
                YAZDIR "İşleminiz gerçekleştiriliyor..."
                
                // Veritabanı işlemleri
                BASLA_ISLEM()
                yeni_bakiye = hesap_bakiye - cekilecek_tutar
                BAKIYE_GUNCELLE(kart_no, yeni_bakiye)
                GUNLUK_CEKILEN_GUNCELLE(kart_no, gunluk_cekilen + cekilecek_tutar)
                ISLEM_KAYDI_OLUSTUR(kart_no, cekilecek_tutar, TARIH_SAAT())
                TAMAMLA_ISLEM()
                
                // Para çıkışı simülasyonu
                PARA_VER(cekilecek_tutar)
                YAZDIR "Lütfen paranızı alınız: " + cekilecek_tutar + " TL"
                
                // Fiş çıkarma
                YAZDIR ""
                YAZDIR "Fiş yazdırılıyor..."
                FIS_YAZDIR(kart_no, cekilecek_tutar, yeni_bakiye, TARIH_SAAT())
                YAZDIR "Lütfen fişinizi alınız"
                
                YAZDIR ""
                YAZDIR "İşlem başarıyla tamamlandı!"
            DEGILSE
                YAZDIR "İşlem iptal edildi."
            SON EGER
        SON EGER
        
        // Başka işlem kontrolü
        YAZDIR ""
        YAZDIR "Başka bir işlem yapmak ister misiniz? (E/H)"
        secim = GIRIS_AL()
        
        EGER secim == "H" VEYA secim == "h" ISE
            baska_islem = YANLIS
        SON EGER
        
    SON TEKRARLA
    
    // Çıkış
    YAZDIR ""
    YAZDIR "Lütfen kartınızı alınız"
    KART_IAD_ET()
    YAZDIR "ATM'mizi kullandığınız için teşekkür ederiz!"
    YAZDIR "İyi günler dileriz."
    
SON


// Yardımcı Fonksiyonlar

FONKSIYON KART_OKU(): STRING
BASLA
    kart_no = KART_OKUYUCUDAN_OKU()
    GER kart_no
SON FONKSIYON

FONKSIYON KONTROL_KART_BLOKE(kart_no: STRING): BOOLEAN
BASLA
    durum = VERI_TABANINDAN_DURUM_GETIR(kart_no)
    GER (durum == "BLOKE")
SON FONKSIYON

FONKSIYON KART_BLOKE_ET(kart_no: STRING)
BASLA
    VERI_TABANINDA_DURUM_GUNCELLE(kart_no, "BLOKE")
    LOG_KAYDI_OLUSTUR("Kart bloke edildi: " + kart_no)
SON FONKSIYON

FONKSIYON BAKIYE_GETIR(kart_no: STRING): DECIMAL
BASLA
    bakiye = VERI_TABANINDAN_BAKIYE_GETIR(kart_no)
    GER bakiye
SON FONKSIYON

FONKSIYON GUNLUK_CEKILEN_GETIR(kart_no: STRING): DECIMAL
BASLA
    bugun = BUGUN_TARIH()
    cekilen = VERI_TABANINDAN_GUNLUK_TOPLAM_GETIR(kart_no, bugun)
    GER cekilen
SON FONKSIYON

FONKSIYON FIS_YAZDIR(kart_no: STRING, tutar: DECIMAL, kalan_bakiye: DECIMAL, tarih_saat: STRING)
BASLA
    YAZDIR "================================"
    YAZDIR "         ATM FİŞİ              "
    YAZDIR "================================"
    YAZDIR "Tarih/Saat: " + tarih_saat
    YAZDIR "Kart No: ****" + SON_4_HANE(kart_no)
    YAZDIR "İşlem: Para Çekme"
    YAZDIR "Tutar: " + tutar + " TL"
    YAZDIR "Kalan Bakiye: " + kalan_bakiye + " TL"
    YAZDIR "================================"
    YAZDIR "   Bizi tercih ettiğiniz için  "
    YAZDIR "        teşekkür ederiz         "
    YAZDIR "================================"
SON FONKSIYON

SON PROGRAM
