BAŞLA

    // Öğrenci Girişi
    YAZDIR "Öğrenci Ders Kayıt Sistemine Hoş Geldiniz"
    YAZDIR "Öğrenci Numaranızı Girin:"
    OKU ogrenci_no
    YAZDIR "Şifrenizi Girin:"
    OKU sifre
    
    giris_basarili = YANLIŞ
    
    EĞER (OgrenciDogrula(ogrenci_no, sifre) == DOĞRU) İSE
        YAZDIR "Giriş Başarılı! Hoş geldiniz."
        giris_basarili = DOĞRU
        
        // Öğrenci bilgilerini al
        ogrenci = OgrenciBilgileriGetir(ogrenci_no)
        YAZDIR "Öğrenci: " + ogrenci.ad + " " + ogrenci.soyad
        YAZDIR "Bölüm: " + ogrenci.bolum
        YAZDIR "GPA: " + ogrenci.gpa
        
    DEĞİLSE
        YAZDIR "Hata: Öğrenci numarası veya şifre yanlış!"
        GİT BAŞLA
    EĞER_SONU
    
    EĞER (giris_basarili == DOĞRU) İSE
        
        // Seçili dersler listesi
        secili_dersler = []
        toplam_kredi = 0
        devam_et = DOĞRU
        
        // Ana Döngü: Ders Ekleme/Çıkarma
        WHILE (devam_et == DOĞRU)
            
            YAZDIR "\n=========================================="
            YAZDIR "DERS KAYIT MENÜsü"
            YAZDIR "=========================================="
            YAZDIR "1. Ders Listesini Görüntüle ve Ders Ekle"
            YAZDIR "2. Seçili Derslerimi Görüntüle"
            YAZDIR "3. Ders Çıkar"
            YAZDIR "4. Kayıt İşlemini Tamamla"
            YAZDIR "Seçiminiz (1-4):"
            OKU menu_secim
            
            EĞER (menu_secim == 1) İSE
                
                // Ders Listesini Görüntüleme Döngüsü
                YAZDIR "\n=========================================="
                YAZDIR "MEVCUT DERSLER"
                YAZDIR "=========================================="
                
                ders_listesi = TumDersleriGetir()
                
                i = 0 İÇİN i < ders_listesi.uzunluk KADAR i++
                    ders = ders_listesi[i]
                    YAZDIR (i+1) + ". " + ders.kod + " - " + ders.ad
                    YAZDIR "   Kredi: " + ders.kredi + 
                           " | Kontenjan: " + ders.mevcut_kontenjan + "/" + ders.max_kontenjan
                    YAZDIR "   Gün-Saat: " + ders.gun + " " + ders.saat
                    YAZDIR "   Hocası: " + ders.ogretim_uyesi
                    
                    EĞER (ders.onkosul_ders != NULL) İSE
                        YAZDIR "   Ön Koşul: " + ders.onkosul_ders
                    EĞER_SONU
                    YAZDIR "   ---"
                DÖNGÜ_SONU
                
                YAZDIR "\nEklemek istediğiniz dersin numarasını girin (0=İptal):"
                OKU ders_secim
                
                EĞER (ders_secim > 0 VE ders_secim <= ders_listesi.uzunluk) İSE
                    
                    secilen_ders = ders_listesi[ders_secim - 1]
                    ders_eklenebilir = DOĞRU
                    hata_mesaji = ""
                    
                    // 1. KONTROL: Kontenjan Kontrolü
                    YAZDIR "\n[Kontrol 1/4] Kontenjan kontrolü yapılıyor..."
                    EĞER (secilen_ders.mevcut_kontenjan >= secilen_ders.max_kontenjan) İSE
                        ders_eklenebilir = YANLIŞ
                        hata_mesaji = "HATA: Ders kontenjanı dolu!"
                        YAZDIR hata_mesaji
                    DEĞİLSE
                        YAZDIR "✓ Kontenjan uygun."
                        
                        // 2. KONTROL: Ön Koşul Dersi Kontrolü
                        YAZDIR "[Kontrol 2/4] Ön koşul kontrolü yapılıyor..."
                        EĞER (secilen_ders.onkosul_ders != NULL) İSE
                            alinan_dersler = OgrenciAlinanDersler(ogrenci_no)
                            onkosul_bulundu = YANLIŞ
                            
                            j = 0 İÇİN j < alinan_dersler.uzunluk KADAR j++
                                EĞER (alinan_dersler[j].kod == secilen_ders.onkosul_ders VE 
                                      alinan_dersler[j].durum == "GEÇTİ") İSE
                                    onkosul_bulundu = DOĞRU
                                    BREAK
                                EĞER_SONU
                            DÖNGÜ_SONU
                            
                            EĞER (onkosul_bulundu == YANLIŞ) İSE
                                ders_eklenebilir = YANLIŞ
                                hata_mesaji = "HATA: Ön koşul dersi (" + 
                                             secilen_ders.onkosul_ders + ") alınmamış!"
                                YAZDIR hata_mesaji
                            DEĞİLSE
                                YAZDIR "✓ Ön koşul dersi alınmış."
                            EĞER_SONU
                        DEĞİLSE
                            YAZDIR "✓ Ön koşul dersi yok."
                        EĞER_SONU
                        
                        EĞER (ders_eklenebilir == DOĞRU) İSE
                            
                            // 3. KONTROL: Zaman Çakışması Kontrolü
                            YAZDIR "[Kontrol 3/4] Zaman çakışması kontrolü yapılıyor..."
                            zaman_cakismasi = YANLIŞ
                            
                            EĞER (secili_dersler.uzunluk > 0) İSE
                                k = 0 İÇİN k < secili_dersler.uzunluk KADAR k++
                                    mevcut_ders = secili_dersler[k]
                                    
                                    EĞER (mevcut_ders.gun == secilen_ders.gun) İSE
                                        // Saat çakışması kontrolü
                                        EĞER (SaatCakismasiVarMi(mevcut_ders.saat, 
                                                                 secilen_ders.saat) == DOĞRU) İSE
                                            zaman_cakismasi = DOĞRU
                                            ders_eklenebilir = YANLIŞ
                                            hata_mesaji = "HATA: " + mevcut_ders.kod + 
                                                         " dersi ile zaman çakışması var!"
                                            YAZDIR hata_mesaji
                                            BREAK
                                        EĞER_SONU
                                    EĞER_SONU
                                DÖNGÜ_SONU
                                
                                EĞER (zaman_cakismasi == YANLIŞ) İSE
                                    YAZDIR "✓ Zaman çakışması yok."
                                EĞER_SONU
                            DEĞİLSE
                                YAZDIR "✓ İlk ders, zaman çakışması kontrolü gerekmiyor."
                            EĞER_SONU
                            
                            EĞER (ders_eklenebilir == DOĞRU) İSE
                                
                                // 4. KONTROL: Kredi Limiti Kontrolü
                                YAZDIR "[Kontrol 4/4] Kredi limiti kontrolü yapılıyor..."
                                yeni_toplam_kredi = toplam_kredi + secilen_ders.kredi
                                
                                EĞER (yeni_toplam_kredi > 35) İSE
                                    ders_eklenebilir = YANLIŞ
                                    hata_mesaji = "HATA: Kredi limiti aşılıyor! " + 
                                                 "(Mevcut: " + toplam_kredi + 
                                                 " + Yeni: " + secilen_ders.kredi + 
                                                 " = " + yeni_toplam_kredi + " > 35)"
                                    YAZDIR hata_mesaji
                                DEĞİLSE
                                    YAZDIR "✓ Kredi limiti uygun."
                                    YAZDIR "  Toplam Kredi: " + yeni_toplam_kredi + "/35"
                                EĞER_SONU
                            EĞER_SONU
                        EĞER_SONU
                    EĞER_SONU
                    
                    // Tüm Kontroller Başarılı ise Dersi Ekle
                    EĞER (ders_eklenebilir == DOĞRU) İSE
                        secili_dersler.EKLE(secilen_ders)
                        toplam_kredi = toplam_kredi + secilen_ders.kredi
                        YAZDIR "\n✓✓✓ BAŞARILI! Ders seçili derslere eklendi. ✓✓✓"
                        YAZDIR "Ders: " + secilen_ders.kod + " - " + secilen_ders.ad
                        YAZDIR "Güncel Toplam Kredi: " + toplam_kredi
                    DEĞİLSE
                        YAZDIR "\n✗✗✗ Ders eklenemedi! ✗✗✗"
                    EĞER_SONU
                    
                DEĞİLSE
                    YAZDIR "Geçersiz seçim!"
                EĞER_SONU
                
            DEĞİLSE EĞER (menu_secim == 2) İSE
                
                // Seçili Dersleri Görüntüle
                YAZDIR "\n=========================================="
                YAZDIR "SEÇİLİ DERSLER"
                YAZDIR "=========================================="
                
                EĞER (secili_dersler.uzunluk == 0) İSE
                    YAZDIR "Henüz ders seçmediniz."
                DEĞİLSE
                    m = 0 İÇİN m < secili_dersler.uzunluk KADAR m++
                        ders = secili_dersler[m]
                        YAZDIR (m+1) + ". " + ders.kod + " - " + ders.ad
                        YAZDIR "   Kredi: " + ders.kredi + 
                               " | Gün-Saat: " + ders.gun + " " + ders.saat
                    DÖNGÜ_SONU
                    YAZDIR "---"
                    YAZDIR "Toplam Kredi: " + toplam_kredi + "/35"
                EĞER_SONU
                
            DEĞİLSE EĞER (menu_secim == 3) İSE
                
                // Ders Çıkarma Döngüsü
                EĞER (secili_dersler.uzunluk == 0) İSE
                    YAZDIR "Çıkarılacak ders bulunmuyor."
                DEĞİLSE
                    YAZDIR "\n=========================================="
                    YAZDIR "DERS ÇIKARMA"
                    YAZDIR "=========================================="
                    
                    n = 0 İÇİN n < secili_dersler.uzunluk KADAR n++
                        YAZDIR (n+1) + ". " + secili_dersler[n].kod + 
                               " - " + secili_dersler[n].ad
                    DÖNGÜ_SONU
                    
                    YAZDIR "Çıkarmak istediğiniz dersin numarasını girin (0=İptal):"
                    OKU cikar_secim
                    
                    EĞER (cikar_secim > 0 VE cikar_secim <= secili_dersler.uzunluk) İSE
                        cikarilan_ders = secili_dersler[cikar_secim - 1]
                        toplam_kredi = toplam_kredi - cikarilan_ders.kredi
                        secili_dersler.SIL(cikar_secim - 1)
                        YAZDIR "✓ Ders çıkarıldı: " + cikarilan_ders.kod
                        YAZDIR "Güncel Toplam Kredi: " + toplam_kredi
                    EĞER_SONU
                EĞER_SONU
                
            DEĞİLSE EĞER (menu_secim == 4) İSE
                
                // Kayıt İşlemini Tamamla
                EĞER (secili_dersler.uzunluk == 0) İSE
                    YAZDIR "Hata: Hiç ders seçmediniz!"
                DEĞİLSE
                    devam_et = YANLIŞ  // Döngüyü sonlandır
                EĞER_SONU
                
            DEĞİLSE
                YAZDIR "Geçersiz seçim! Lütfen 1-4 arası seçim yapın."
            EĞER_SONU
            
        DÖNGÜ_SONU  // Ana ders ekleme/çıkarma döngüsü sonu
        
        // Danışman Onayı Kontrolü
        YAZDIR "\n=========================================="
        YAZDIR "DANISMAN ONAYI KONTROLÜ"
        YAZDIR "=========================================="
        YAZDIR "GPA'nız: " + ogrenci.gpa
        
        danisman_onay_gerekli = YANLIŞ
        
        EĞER (ogrenci.gpa < 2.5) İSE
            danisman_onay_gerekli = DOĞRU
            YAZDIR "⚠ UYARI: GPA'nız 2.5'in altında!"
            YAZDIR "Kayıt işleminiz için danışman onayı gereklidir."
            YAZDIR "Danışmanınıza bildirim gönderiliyor..."
            
            // Danışman onayı simülasyonu
            DanismanaOnayTalebiGonder(ogrenci_no, secili_dersler)
            
            YAZDIR "Danışman onayını bekliyor musunuz? (E/H):"
            OKU onay_bekle
            
            EĞER (onay_bekle == "E" VEYA onay_bekle == "e") İSE
                // Simülasyon için otomatik onay
                YAZDIR "Danışman onayı kontrol ediliyor..."
                danisman_onayi = DanismanOnayKontrol(ogrenci_no)
                
                EĞER (danisman_onayi == DOĞRU) İSE
                    YAZDIR "✓ Danışman kaydınızı onayladı!"
                DEĞİLSE
                    YAZDIR "✗ Danışman kaydınızı onaylamadı veya henüz onaylamadı."
                    YAZDIR "Kayıt işlemi iptal ediliyor."
                    GİT BİTİR
                EĞER_SONU
            DEĞİLSE
                YAZDIR "Kayıt işlemi iptal edildi."
                GİT BİTİR
            EĞER_SONU
        DEĞİLSE
            YAZDIR "✓ GPA'nız 2.5 ve üzeri. Danışman onayı gerekmiyor."
        EĞER_SONU
        
        // Kayıt Özeti Gösterme
        YAZDIR "\n=========================================="
        YAZDIR "KAYIT ÖZETİ"
        YAZDIR "=========================================="
        YAZDIR "Öğrenci: " + ogrenci.ad + " " + ogrenci.soyad
        YAZDIR "Öğrenci No: " + ogrenci_no
        YAZDIR "Bölüm: " + ogrenci.bolum
        YAZDIR "GPA: " + ogrenci.gpa
        YAZDIR "=========================================="
        YAZDIR "SEÇİLİ DERSLER:"
        YAZDIR "=========================================="
        
        p = 0 İÇİN p < secili_dersler.uzunluk KADAR p++
            ders = secili_dersler[p]
            YAZDIR (p+1) + ". " + ders.kod + " - " + ders.ad
            YAZDIR "   Kredi: " + ders.kredi + " | Gün-Saat: " + ders.gun + " " + ders.saat
            YAZDIR "   Hoca: " + ders.ogretim_uyesi
        DÖNGÜ_SONU
        
        YAZDIR "=========================================="
        YAZDIR "TOPLAM KREDİ: " + toplam_kredi + "/35"
        YAZDIR "=========================================="
        
        EĞER (danisman_onay_gerekli == DOĞRU) İSE
            YAZDIR "Danışman Onayı: ONAYLANDI ✓"
        EĞER_SONU
        
        // Son Onay
        YAZDIR "\nKaydınızı onaylıyor musunuz? (E/H):"
        OKU kayit_onay
        
        EĞER (kayit_onay == "E" VEYA kayit_onay == "e") İSE
            
            // Kayıt işlemini tamamla
            kayit_basarili = DersKayitTamamla(ogrenci_no, secili_dersler)
            
            EĞER (kayit_basarili == DOĞRU) İSE
                YAZDIR "\n✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓"
                YAZDIR "KAYIT İŞLEMİ BAŞARIYLA TAMAMLANDI!"
                YAZDIR "✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓"
                YAZDIR "Ders programınız e-posta adresinize gönderilmiştir."
                YAZDIR "İyi çalışmalar dileriz!"
            DEĞİLSE
                YAZDIR "✗ HATA: Kayıt işlemi sırasında bir hata oluştu!"
                YAZDIR "Lütfen öğrenci işleri ile iletişime geçin."
            EĞER_SONU
            
        DEĞİLSE
            YAZDIR "Kayıt işlemi iptal edildi."
            YAZDIR "Sisteme tekrar giriş yaparak kayıt işleminizi yapabilirsiniz."
        EĞER_SONU
        
    EĞER_SONU  // Giriş başarılı kontrolü sonu

BİTİR


// ============================================
// FONKSİYON TANIMLARI
// ============================================

FONKSİYON OgrenciDogrula(ogrenci_no, sifre)
    // Veritabanında öğrenci kontrolü
    DÖNÜT (ogrenci_no VE sifre doğru mu?)
FONKSİYON_SONU

FONKSİYON OgrenciBilgileriGetir(ogrenci_no)
    // Öğrenci bilgilerini getir (ad, soyad, bölüm, gpa)
    DÖNÜT ogrenci_bilgileri
FONKSİYON_SONU

FONKSİYON TumDersleriGetir()
    // Tüm dersleri veritabanından getir
    DÖNÜT ders_listesi
FONKSİYON_SONU

FONKSİYON OgrenciAlinanDersler(ogrenci_no)
    // Öğrencinin daha önce aldığı dersleri getir
    DÖNÜT alinan_dersler
FONKSİYON_SONU

FONKSİYON SaatCakismasiVarMi(saat1, saat2)
    // İki ders saati arasında çakışma kontrolü
    // Örnek: "09:00-10:50" ve "10:00-11:50" çakışır
    DÖNÜT (çakışma var mı?)
FONKSİYON_SONU

FONKSİYON DanismanaOnayTalebiGonder(ogrenci_no, dersler)
    // Danışmana e-posta/bildirim gönder
    // Danışman panelinde onay bekleyen kayıt oluştur
FONKSİYON_SONU

FONKSİYON DanismanOnayKontrol(ogrenci_no)
    // Danışmanın onay durumunu kontrol et
    DÖNÜT (onaylandı mı?)
FONKSİYON_SONU

FONKSİYON DersKayitTamamla(ogrenci_no, dersler)
    // Dersleri veritabanına kaydet
    // Kontenjanları güncelle
    // Öğrenciye e-posta gönder
    DÖNÜT (kayıt başarılı mı?)
FONKSİYON_SONU
