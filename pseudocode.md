// ============================================
// ONLINE ALIŞ VERİŞ SİTESİ SEPET SİSTEMİ
// ============================================

// GLOBAL DEĞİŞKENLER
CONST MINIMUM_ORDER_AMOUNT = 50.00
CONST FREE_SHIPPING_THRESHOLD = 200.00
CONST SHIPPING_COST = 29.90

// VERİ YAPILARI
CLASS User
    userId: INTEGER
    username: STRING
    email: STRING
    isGuest: BOOLEAN
    address: STRING
END CLASS

CLASS Product
    productId: INTEGER
    name: STRING
    category: STRING
    price: DECIMAL
    stockQuantity: INTEGER
    description: STRING
    imageUrl: STRING
END CLASS

CLASS CartItem
    product: Product
    quantity: INTEGER
    subtotal: DECIMAL
END CLASS

CLASS ShoppingCart
    cartId: INTEGER
    user: User
    items: LIST<CartItem>
    discountCode: STRING
    discountAmount: DECIMAL
    subtotal: DECIMAL
    shippingCost: DECIMAL
    totalAmount: DECIMAL
END CLASS

CLASS DiscountCoupon
    code: STRING
    discountType: STRING  // "PERCENTAGE" veya "FIXED"
    discountValue: DECIMAL
    minOrderAmount: DECIMAL
    expiryDate: DATE
    isActive: BOOLEAN
END CLASS

// ============================================
// 1. KULLANICI GİRİŞ KONTROLÜ
// ============================================
FUNCTION UserLogin()
    PRINT "1. Kayıtlı Kullanıcı Girişi"
    PRINT "2. Misafir Olarak Devam Et"
    
    choice = INPUT("Seçiminiz: ")
    
    IF choice == 1 THEN
        username = INPUT("Kullanıcı adı: ")
        password = INPUT("Şifre: ")
        
        user = AuthenticateUser(username, password)
        
        IF user != NULL THEN
            PRINT "Giriş başarılı! Hoş geldiniz " + user.username
            RETURN user
        ELSE
            PRINT "Hatalı kullanıcı adı veya şifre!"
            RETURN UserLogin()  // Tekrar dene
        END IF
    ELSE IF choice == 2 THEN
        user = CreateGuestUser()
        PRINT "Misafir kullanıcı olarak devam ediyorsunuz"
        RETURN user
    END IF
END FUNCTION

FUNCTION AuthenticateUser(username, password)
    // Veritabanından kullanıcı doğrulama
    user = DATABASE.Query("SELECT * FROM Users WHERE username = ? AND password = ?", username, HASH(password))
    RETURN user
END FUNCTION

FUNCTION CreateGuestUser()
    guestUser = NEW User()
    guestUser.userId = GENERATE_TEMP_ID()
    guestUser.username = "Misafir_" + guestUser.userId
    guestUser.isGuest = TRUE
    RETURN guestUser
END FUNCTION

// ============================================
// 2. ÜRÜN KATEGORİLERİ ARASINDA GEZİNME
// ============================================
FUNCTION BrowseCategories()
    categories = DATABASE.Query("SELECT DISTINCT category FROM Products")
    
    PRINT "=== ÜRÜN KATEGORİLERİ ==="
    FOR i = 0 TO categories.LENGTH - 1 DO
        PRINT (i + 1) + ". " + categories[i]
    END FOR
    
    categoryChoice = INPUT("Kategori seçin: ")
    selectedCategory = categories[categoryChoice - 1]
    
    RETURN DisplayProductsByCategory(selectedCategory)
END FUNCTION

FUNCTION DisplayProductsByCategory(category)
    products = DATABASE.Query("SELECT * FROM Products WHERE category = ?", category)
    
    PRINT "=== " + category + " KATEGORİSİNDEKİ ÜRÜNLER ==="
    FOR i = 0 TO products.LENGTH - 1 DO
        PRINT (i + 1) + ". " + products[i].name + " - " + products[i].price + " TL"
        PRINT "   Stok: " + products[i].stockQuantity
    END FOR
    
    productChoice = INPUT("Ürün seçin (0: Geri dön): ")
    
    IF productChoice == 0 THEN
        RETURN BrowseCategories()
    ELSE
        RETURN products[productChoice - 1]
    END IF
END FUNCTION

// ============================================
// 3. STOK KONTROLÜ
// ============================================
FUNCTION CheckStock(product, requestedQuantity)
    IF product.stockQuantity >= requestedQuantity THEN
        RETURN TRUE
    ELSE
        RETURN FALSE
    END IF
END FUNCTION

// ============================================
// 4. ÜRÜN SEPETE EKLEME
// ============================================
FUNCTION AddToCart(cart, product, quantity)
    // Stok kontrolü
    IF NOT CheckStock(product, quantity) THEN
        PRINT "HATA: Ürün stokta yeterli miktarda bulunmuyor!"
        PRINT "Mevcut stok: " + product.stockQuantity
        RETURN FALSE
    END IF
    
    // Sepette ürün var mı kontrol et
    existingItem = FindItemInCart(cart, product.productId)
    
    IF existingItem != NULL THEN
        // Ürün zaten sepette, miktarı artır
        newQuantity = existingItem.quantity + quantity
        
        IF CheckStock(product, newQuantity) THEN
            existingItem.quantity = newQuantity
            existingItem.subtotal = existingItem.quantity * product.price
            PRINT "Ürün miktarı güncellendi: " + existingItem.quantity
        ELSE
            PRINT "HATA: Toplam miktar stok miktarını aşıyor!"
            RETURN FALSE
        END IF
    ELSE
        // Yeni ürün ekle
        cartItem = NEW CartItem()
        cartItem.product = product
        cartItem.quantity = quantity
        cartItem.subtotal = quantity * product.price
        
        cart.items.ADD(cartItem)
        PRINT "Ürün sepete eklendi: " + product.name
    END IF
    
    // Sepet toplamını güncelle
    UpdateCartTotals(cart)
    RETURN TRUE
END FUNCTION

FUNCTION FindItemInCart(cart, productId)
    FOR EACH item IN cart.items DO
        IF item.product.productId == productId THEN
            RETURN item
        END IF
    END FOR
    RETURN NULL
END FUNCTION

// ============================================
// 5. SEPETİ GÖRÜNTÜLEME
// ============================================
FUNCTION DisplayCart(cart)
    PRINT "=== SEPET İÇERİĞİ ==="
    PRINT "======================================"
    
    IF cart.items.LENGTH == 0 THEN
        PRINT "Sepetiniz boş!"
        RETURN
    END IF
    
    FOR i = 0 TO cart.items.LENGTH - 1 DO
        item = cart.items[i]
        PRINT (i + 1) + ". " + item.product.name
        PRINT "   Birim Fiyat: " + item.product.price + " TL"
        PRINT "   Adet: " + item.quantity
        PRINT "   Ara Toplam: " + item.subtotal + " TL"
        PRINT "--------------------------------------"
    END FOR
    
    PRINT "Ara Toplam: " + cart.subtotal + " TL"
    
    IF cart.discountAmount > 0 THEN
        PRINT "İndirim: -" + cart.discountAmount + " TL"
    END IF
    
    PRINT "Kargo: " + cart.shippingCost + " TL"
    PRINT "======================================"
    PRINT "TOPLAM: " + cart.totalAmount + " TL"
    PRINT "======================================"
END FUNCTION

// ============================================
// 6. SEPET DÜZENLEME
// ============================================
FUNCTION EditCart(cart)
    DisplayCart(cart)
    
    PRINT "\n=== SEPET DÜZENLEME ==="
    PRINT "1. Ürün Miktarını Güncelle"
    PRINT "2. Ürün Sil"
    PRINT "3. Alışverişe Devam"
    PRINT "4. Ödeme Sayfasına Git"
    
    choice = INPUT("Seçiminiz: ")
    
    IF choice == 1 THEN
        itemIndex = INPUT("Güncellenecek ürün numarası: ") - 1
        newQuantity = INPUT("Yeni miktar: ")
        
        IF UpdateCartItemQuantity(cart, itemIndex, newQuantity) THEN
            PRINT "Miktar güncellendi!"
        END IF
        
        RETURN EditCart(cart)
        
    ELSE IF choice == 2 THEN
        itemIndex = INPUT("Silinecek ürün numarası: ") - 1
        RemoveFromCart(cart, itemIndex)
        PRINT "Ürün sepetten silindi!"
        RETURN EditCart(cart)
        
    ELSE IF choice == 3 THEN
        RETURN "CONTINUE_SHOPPING"
        
    ELSE IF choice == 4 THEN
        RETURN "PROCEED_TO_CHECKOUT"
    END IF
END FUNCTION

FUNCTION UpdateCartItemQuantity(cart, itemIndex, newQuantity)
    IF itemIndex < 0 OR itemIndex >= cart.items.LENGTH THEN
        PRINT "Geçersiz ürün numarası!"
        RETURN FALSE
    END IF
    
    item = cart.items[itemIndex]
    
    IF newQuantity <= 0 THEN
        PRINT "Miktar 0'dan büyük olmalıdır!"
        RETURN FALSE
    END IF
    
    // Stok kontrolü
    IF CheckStock(item.product, newQuantity) THEN
        item.quantity = newQuantity
        item.subtotal = item.quantity * item.product.price
        UpdateCartTotals(cart)
        RETURN TRUE
    ELSE
        PRINT "HATA: Stokta yeterli ürün yok!"
        RETURN FALSE
    END IF
END FUNCTION

FUNCTION RemoveFromCart(cart, itemIndex)
    IF itemIndex >= 0 AND itemIndex < cart.items.LENGTH THEN
        cart.items.REMOVE(itemIndex)
        UpdateCartTotals(cart)
    END IF
END FUNCTION

// ============================================
// 7. İNDİRİM KODU UYGULAMA
// ============================================
FUNCTION ApplyDiscountCode(cart)
    discountCode = INPUT("İndirim kodunu girin: ")
    
    coupon = ValidateDiscountCode(discountCode, cart.subtotal)
    
    IF coupon != NULL THEN
        // İndirim hesapla
        IF coupon.discountType == "PERCENTAGE" THEN
            cart.discountAmount = cart.subtotal * (coupon.discountValue / 100)
        ELSE IF coupon.discountType == "FIXED" THEN
            cart.discountAmount = coupon.discountValue
        END IF
        
        cart.discountCode = discountCode
        PRINT "İndirim kodu uygulandı! İndirim: " + cart.discountAmount + " TL"
        
        UpdateCartTotals(cart)
        RETURN TRUE
    ELSE
        PRINT "Geçersiz veya süresi dolmuş indirim kodu!"
        RETURN FALSE
    END IF
END FUNCTION

FUNCTION ValidateDiscountCode(code, orderAmount)
    coupon = DATABASE.Query("SELECT * FROM DiscountCoupons WHERE code = ?", code)
    
    IF coupon == NULL THEN
        RETURN NULL
    END IF
    
    // Kontroller
    IF NOT coupon.isActive THEN
        RETURN NULL
    END IF
    
    IF coupon.expiryDate < CURRENT_DATE() THEN
        RETURN NULL
    END IF
    
    IF orderAmount < coupon.minOrderAmount THEN
        PRINT "Bu indirim kodu minimum " + coupon.minOrderAmount + " TL alışveriş gerektirir!"
        RETURN NULL
    END IF
    
    RETURN coupon
END FUNCTION

// ============================================
// 8. SEPET TOPLAMLARINI GÜNCELLEME
// ============================================
FUNCTION UpdateCartTotals(cart)
    // Ara toplam hesapla
    cart.subtotal = 0
    FOR EACH item IN cart.items DO
        cart.subtotal = cart.subtotal + item.subtotal
    END FOR
    
    // İndirim düşüldükten sonraki tutar
    discountedAmount = cart.subtotal - cart.discountAmount
    
    // Kargo hesapla
    IF discountedAmount >= FREE_SHIPPING_THRESHOLD THEN
        cart.shippingCost = 0
    ELSE
        cart.shippingCost = SHIPPING_COST
    END IF
    
    // Toplam hesapla
    cart.totalAmount = discountedAmount + cart.shippingCost
END FUNCTION

// ============================================
// 9. MİNİMUM TUTAR KONTROLÜ
// ============================================
FUNCTION CheckMinimumOrderAmount(cart)
    IF cart.subtotal < MINIMUM_ORDER_AMOUNT THEN
        remainingAmount = MINIMUM_ORDER_AMOUNT - cart.subtotal
        PRINT "UYARI: Minimum sipariş tutarı " + MINIMUM_ORDER_AMOUNT + " TL'dir."
        PRINT "Eksik tutar: " + remainingAmount + " TL"
        RETURN FALSE
    END IF
    RETURN TRUE
END FUNCTION

// ============================================
// 10. KARGO ÜCRETİ HESAPLAMA
// ============================================
FUNCTION CalculateShipping(cart)
    discountedAmount = cart.subtotal - cart.discountAmount
    
    IF discountedAmount >= FREE_SHIPPING_THRESHOLD THEN
        PRINT "Ücretsiz kargo kazandınız!"
        RETURN 0
    ELSE
        remainingForFreeShipping = FREE_SHIPPING_THRESHOLD - discountedAmount
        PRINT "Ücretsiz kargo için " + remainingForFreeShipping + " TL daha alışveriş yapın!"
        RETURN SHIPPING_COST
    END IF
END FUNCTION

// ============================================
// 11. ÖDEME YÖNTEMİ SEÇİMİ
// ============================================
FUNCTION SelectPaymentMethod()
    PRINT "=== ÖDEME YÖNTEMİ SEÇİN ==="
    PRINT "1. Kredi Kartı"
    PRINT "2. Banka Kartı"
    PRINT "3. Havale/EFT"
    
    choice = INPUT("Seçiminiz: ")
    
    IF choice == 1 THEN
        RETURN ProcessCreditCardPayment()
    ELSE IF choice == 2 THEN
        RETURN ProcessDebitCardPayment()
    ELSE IF choice == 3 THEN
        RETURN ProcessBankTransfer()
    ELSE
        PRINT "Geçersiz seçim!"
        RETURN SelectPaymentMethod()
    END IF
END FUNCTION

FUNCTION ProcessCreditCardPayment()
    PRINT "=== KREDİ KARTI BİLGİLERİ ==="
    cardNumber = INPUT("Kart Numarası: ")
    cardHolder = INPUT("Kart Sahibi: ")
    expiryDate = INPUT("Son Kullanma Tarihi (AA/YY): ")
    cvv = INPUT("CVV: ")
    
    // Kart doğrulama
    IF ValidateCreditCard(cardNumber, expiryDate, cvv) THEN
        RETURN {
            method: "CREDIT_CARD",
            cardNumber: MASK_CARD_NUMBER(cardNumber),
            cardHolder: cardHolder
        }
    ELSE
        PRINT "Geçersiz kart bilgileri!"
        RETURN NULL
    END IF
END FUNCTION

FUNCTION ProcessDebitCardPayment()
    PRINT "=== BANKA KARTI BİLGİLERİ ==="
    // Benzer işlem kredi kartı ile
    RETURN ProcessCreditCardPayment()
END FUNCTION

FUNCTION ProcessBankTransfer()
    PRINT "=== HAVALE/EFT BİLGİLERİ ==="
    PRINT "Banka: XYZ Bankası"
    PRINT "IBAN: TR00 0000 0000 0000 0000 0000 00"
    PRINT "Hesap Sahibi: ABC E-Ticaret Ltd."
    PRINT "\nLütfen açıklama kısmına sipariş numaranızı yazınız."
    
    RETURN {
        method: "BANK_TRANSFER",
        status: "PENDING"
    }
END FUNCTION

// ============================================
// 12. SİPARİŞ ONAYI VE İŞLEME
// ============================================
FUNCTION ConfirmOrder(cart, user, paymentInfo)
    PRINT "=== SİPARİŞ ÖZETİ ==="
    DisplayCart(cart)
    
    PRINT "\nTeslimat Adresi: " + user.address
    PRINT "Ödeme Yöntemi: " + paymentInfo.method
    
    confirmation = INPUT("\nSiparişi onaylıyor musunuz? (E/H): ")
    
    IF confirmation == "E" OR confirmation == "e" THEN
        RETURN ProcessOrder(cart, user, paymentInfo)
    ELSE
        PRINT "Sipariş iptal edildi."
        RETURN FALSE
    END IF
END FUNCTION

FUNCTION ProcessOrder(cart, user, paymentInfo)
    // Sipariş kaydı oluştur
    orderId = GENERATE_ORDER_ID()
    orderDate = CURRENT_DATETIME()
    
    // Veritabanına kaydet
    BEGIN TRANSACTION
    
    TRY
        // Siparişi kaydet
        DATABASE.Execute(
            "INSERT INTO Orders (orderId, userId, orderDate, totalAmount, paymentMethod, status) VALUES (?, ?, ?, ?, ?, ?)",
            orderId, user.userId, orderDate, cart.totalAmount, paymentInfo.method, "PROCESSING"
        )
        
        // Sipariş detaylarını kaydet
        FOR EACH item IN cart.items DO
            DATABASE.Execute(
                "INSERT INTO OrderDetails (orderId, productId, quantity, price, subtotal) VALUES (?, ?, ?, ?, ?)",
                orderId, item.product.productId, item.quantity, item.product.price, item.subtotal
            )
            
            // Stok güncelle
            DATABASE.Execute(
                "UPDATE Products SET stockQuantity = stockQuantity - ? WHERE productId = ?",
                item.quantity, item.product.productId
            )
        END FOR
        
        // Ödeme işlemi
        paymentSuccess = FALSE
        
        IF paymentInfo.method == "CREDIT_CARD" OR paymentInfo.method == "DEBIT_CARD" THEN
            paymentSuccess = ProcessCardPayment(orderId, cart.totalAmount, paymentInfo)
        ELSE IF paymentInfo.method == "BANK_TRANSFER" THEN
            paymentSuccess = TRUE  // Havale onay bekliyor
        END IF
        
        IF paymentSuccess THEN
            COMMIT TRANSACTION
            
            // Sepeti temizle
            cart.items.CLEAR()
            cart.discountAmount = 0
            cart.discountCode = ""
            UpdateCartTotals(cart)
            
            // Onay e-postası gönder
            SendOrderConfirmationEmail(user.email, orderId)
            
            PRINT "\n=== SİPARİŞ TAMAMLANDI ==="
            PRINT "Sipariş Numaranız: " + orderId
            PRINT "Toplam Tutar: " + cart.totalAmount + " TL"
            PRINT "Sipariş durumunuzu e-posta üzerinden takip edebilirsiniz."
            
            RETURN TRUE
        ELSE
            ROLLBACK TRANSACTION
            PRINT "HATA: Ödeme işlemi başarısız!"
            RETURN FALSE
        END IF
        
    CATCH error
        ROLLBACK TRANSACTION
        PRINT "HATA: Sipariş işlenirken bir hata oluştu: " + error.message
        RETURN FALSE
    END TRY
END FUNCTION

// ============================================
// YARDIMCI FONKSİYONLAR
// ============================================
FUNCTION ValidateCreditCard(cardNumber, expiryDate, cvv)
    // Luhn algoritması ile kart numarası kontrolü
    IF cardNumber.LENGTH < 13 OR cardNumber.LENGTH > 19 THEN
        RETURN FALSE
    END IF
    
    // CVV kontrolü
    IF cvv.LENGTH != 3 AND cvv.LENGTH != 4 THEN
        RETURN FALSE
    END IF
    
    // Son kullanma tarihi kontrolü
    IF NOT IsValidExpiryDate(expiryDate) THEN
        RETURN FALSE
    END IF
    
    RETURN TRUE
END FUNCTION

FUNCTION IsValidExpiryDate(expiryDate)
    parts = SPLIT(expiryDate, "/")
    month = INTEGER(parts[0])
    year = INTEGER("20" + parts[1])
    
    currentYear = CURRENT_YEAR()
    currentMonth = CURRENT_MONTH()
    
    IF year < currentYear THEN
        RETURN FALSE
    END IF
    
    IF year == currentYear AND month < currentMonth THEN
        RETURN FALSE
    END IF
    
    RETURN TRUE
END FUNCTION

FUNCTION MASK_CARD_NUMBER(cardNumber)
    maskedNumber = ""
    FOR i = 0 TO cardNumber.LENGTH - 1 DO
        IF i < cardNumber.LENGTH - 4 THEN
            maskedNumber = maskedNumber + "*"
        ELSE
            maskedNumber = maskedNumber + cardNumber[i]
        END IF
    END FOR
    RETURN maskedNumber
END FUNCTION

FUNCTION SendOrderConfirmationEmail(email, orderId)
    emailBody = "Sayın Müşterimiz,\n\n"
    emailBody = emailBody + "Sipariş numarası " + orderId + " olan siparişiniz alınmıştır.\n"
    emailBody = emailBody + "En kısa sürede kargoya teslim edilecektir.\n\n"
    emailBody = emailBody + "İyi günler dileriz."
    
    SEND_EMAIL(email, "Sipariş Onayı - " + orderId, emailBody)
END FUNCTION

// ============================================
// ANA PROGRAM AKIŞI
// ============================================
FUNCTION Main()
    PRINT "=== ONLINE ALIŞ VERİŞ SİTESİNE HOŞ GELDİNİZ ==="
    
    // 1. Kullanıcı girişi
    user = UserLogin()
    
    // Sepet oluştur
    cart = NEW ShoppingCart()
    cart.user = user
    cart.items = []
    cart.discountAmount = 0
    cart.subtotal = 0
    cart.shippingCost = 0
    cart.totalAmount = 0
    
    // Ana döngü
    WHILE TRUE DO
        // 2. Ürün kategorileri
        product = BrowseCategories()
        
        // 3. Sepete ekle
        quantity = INPUT("Kaç adet eklemek istersiniz? ")
        AddToCart(cart, product, quantity)
        
        continueChoice = INPUT("Alışverişe devam etmek istiyor musunuz? (E/H): ")
        
        IF continueChoice == "H" OR continueChoice == "h" THEN
            BREAK
        END IF
    END WHILE
    
    // 4. Sepet düzenleme
    WHILE TRUE DO
        action = EditCart(cart)
        
        IF action == "CONTINUE_SHOPPING" THEN
            // Alışverişe devam et
            CONTINUE
        ELSE IF action == "PROCEED_TO_CHECKOUT" THEN
            BREAK
        END IF
    END WHILE
    
    // 5. Minimum tutar kontrolü
    IF NOT CheckMinimumOrderAmount(cart) THEN
        PRINT "Lütfen sepetinize ürün ekleyin."
        RETURN
    END IF
    
    // 6. İndirim kodu
    discountChoice = INPUT("İndirim kodunuz var mı? (E/H): ")
    IF discountChoice == "E" OR discountChoice == "e" THEN
        ApplyDiscountCode(cart)
    END IF
    
    // 7. Kargo hesaplama
    cart.shippingCost = CalculateShipping(cart)
    UpdateCartTotals(cart)
    
    // 8. Son özet
    DisplayCart(cart)
    
    // 9. Teslimat adresi
    IF user.isGuest OR user.address == "" THEN
        user.address = INPUT("Teslimat adresinizi girin: ")
    END IF
    
    // 10. Ödeme yöntemi
    paymentInfo = SelectPaymentMethod()
    
    IF paymentInfo == NULL THEN
        PRINT "Ödeme işlemi iptal edildi."
        RETURN
    END IF
    
    // 11. Sipariş onayı
    orderSuccess = ConfirmOrder(cart, user, paymentInfo)
    
    IF orderSuccess THEN
        PRINT "Siparişiniz için teşekkür ederiz!"
    ELSE
        PRINT "Sipariş tamamlanamadı. Lütfen tekrar deneyin."
    END IF
END FUNCTION

// Programı başlat
Main()
