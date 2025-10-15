// Veri yapıları ve sabitler
CONST MAX_PIN_ATTEMPTS = 3
CONST DAILY_WITHDRAWAL_LIMIT = 10000.00    // örnek TL
CONST PER_TRANSACTION_LIMIT = 5000.00
CONST MIN_DENOMINATION = 5.00              // en küçük banknot değeri

// Hesap kaydı örneği
Account {
  id
  cardNumber
  pinHash
  balance
  dailyWithdrawnAmount   // bugünkü toplam çekim
  isBlocked              // hesap veya kart bloklanmış mı
  lastWithdrawalDate
}

// ATM kasa durumu: her banknot değerinden kaç adet kaldığı
ATMCash {
  map<denomination, quantity>
  totalCash() -> sum(denomination * quantity)
}

// Transaction log
Transaction {
  id
  accountId
  timestamp
  type        // WITHDRAWAL, FAILURE, etc.
  amount
  status      // SUCCESS, FAILED
  details     // hata kodu veya açıklama
}

// Yardımcı fonksiyonlar
function hashPIN(pin) -> hashedPIN
function verifyPIN(pin, storedHash) -> boolean
function getAccountByCard(cardNumber) -> Account | null
function lockATM()   // fiziksel veya yazılımsal kilit
function unlockATM()
function beginDatabaseTransaction()
function commitDatabaseTransaction()
function rollbackDatabaseTransaction()
function logTransaction(tx: Transaction)
function dispenseBanknotes(banknotePlan: map<denomination, quantity>) -> boolean
function printReceipt(account, tx)
function ejectCard()

// Ana akış
procedure ATM_WithdrawFlow(cardNumber):
  atmLock = false
  account = getAccountByCard(cardNumber)
  if account == null:
    display("Tanımlanamayan kart. Lütfen bankanızla iletişime geçin.")
    ejectCard()
    return

  if account.isBlocked:
    display("Kartınız blokeli. Banka ile iletişime geçiniz.")
    ejectCard()
    return

  // PIN doğrulama - MAX_PIN_ATTEMPTS hakkı
  attempts = 0
  authenticated = false
  while attempts < MAX_PIN_ATTEMPTS and not authenticated:
    inputPIN = promptUser("PIN giriniz:")
    if verifyPIN(inputPIN, account.pinHash):
      authenticated = true
    else:
      attempts += 1
      remaining = MAX_PIN_ATTEMPTS - attempts
      display("Hatalı PIN. Kalan deneme hakkı: " + remaining)
  if not authenticated:
    // güvenlik: blokla
    account.isBlocked = true
    updateAccount(account) // veri tabanına yaz
    logTransaction(Transaction{accountId: account.id, timestamp: now(), type: "LOCK", status: "FAILED", details: "PIN attempts exceeded"})
    display("Kartınız güvenlik nedeniyle bloklandı.")
    ejectCard()
    return

  // İşlem seçimi (burada sadece para çekme)
  choice = promptUser("1: Para Çekme, 2: İptal")
  if choice != 1:
    display("İşlem iptal edildi.")
    ejectCard()
    return

  // Tutar alma ve ön kontroller
  requestedAmount = promptUserAmount("Çekmek istediğiniz tutarı giriniz:")
  if requestedAmount <= 0:
    display("Geçersiz tutar.")
    ejectCard()
    return

  // Yuvarlama / banknot uyumu kontrolü
  if requestedAmount % MIN_DENOMINATION != 0:
    display("Lütfen " + MIN_DENOMINATION + " TL katı olarak giriniz.")
    ejectCard()
    return

  if requestedAmount > PER_TRANSACTION_LIMIT:
    display("İşlem limiti aşıldı. Maksimum: " + PER_TRANSACTION_LIMIT)
    ejectCard()
    return

  // Günlük limit kontrolü
  // Eğer tarih değişmişse dailyWithdrawnAmount sıfırlanır
  if account.lastWithdrawalDate != today():
    account.dailyWithdrawnAmount = 0
    account.lastWithdrawalDate = today()

  if account.dailyWithdrawnAmount + requestedAmount > DAILY_WITHDRAWAL_LIMIT:
    display("Günlük limit aşıldı.")
    ejectCard()
    return

  // Bakiye kontrolü
  if requestedAmount > account.balance:
    display("Yetersiz bakiye.")
    ejectCard()
    return

  // ATM nakit kontrolu
  atmCash = getATMCashStatus()
  if requestedAmount > atmCash.totalCash():
    display("ATM'de yeterli nakit yok.")
    ejectCard()
    return

  // Denominasyon planı üretme (greedy veya DP)
  banknotePlan = computeBanknotePlan(requestedAmount, atmCash)
  if banknotePlan == null:
    // ATM kombinasyon ile veremiyor -> öner: en yakın alt tutarı teklif et
    altPlan = findClosestLowerAmountPlan(requestedAmount, atmCash)
    if altPlan == null:
      display("ATM istenen miktarı banknot kombinasyonu nedeniyle veremiyor.")
      ejectCard()
      return
    else:
      display("İstenen tutar kombinasyon nedeniyle verilemiyor. "+ altPlan.amount +" TL çekmek ister misiniz? (E/H)")
      userChoice = promptUserYesNo()
      if userChoice == NO:
        display("İşlem iptal edildi.")
        ejectCard()
        return
      else:
        requestedAmount = altPlan.amount
        banknotePlan = altPlan.banknotePlan

  // Kritik bölüm: veri tabanı ve kasa güncellemesi atomik olmalı
  beginDatabaseTransaction()
  try:
    // Lock atm cash container ve account record (concurrency)
    lockResource("atm_cash")
    lockResource("account_" + account.id)

    // Son kontroller: eş zamanlarda bakiye değişmiş olabilir
    freshAccount = getAccountById(account.id)
    if requestedAmount > freshAccount.balance:
      rollbackDatabaseTransaction()
      unlockResource("atm_cash")
      unlockResource("account_" + account.id)
      display("İşlem başarısız: güncel bakiyeniz yetersiz.")
      ejectCard()
      return

    // ATM kasa miktarını güncelle
    success = reduceATMCash(banknotePlan)
    if not success:
      rollbackDatabaseTransaction()
      unlockResource("atm_cash")
      unlockResource("account_" + account.id)
      display("ATM para verme hatası. Lütfen başka ATM deneyiniz.")
      ejectCard()
      return

    // Hesap bakiyesini güncelle
    freshAccount.balance = freshAccount.balance - requestedAmount
    freshAccount.dailyWithdrawnAmount = freshAccount.dailyWithdrawnAmount + requestedAmount
    updateAccount(freshAccount)

    // Transaction log
    tx = Transaction{accountId: freshAccount.id, timestamp: now(), type: "WITHDRAWAL", amount: requestedAmount, status: "PENDING"}
    txId = createTransactionLog(tx)

    // Para fiziksel olarak verilir
    dispenseOK = dispenseBanknotes(banknotePlan)
    if not dispenseOK:
      // para verilmezse rollback: ATM kasa ve hesap eski haline dönmeli
      rollbackDatabaseTransaction()
      // ayrıca eğer kasa mecazi olarak azaltıldıysa onu restore et
      restoreATMCash(banknotePlan)
      unlockResource("atm_cash")
      unlockResource("account_" + account.id)
      display("Para verilemedi. İşlem iptal edildi.")
      // Log failure
      logTransaction(Transaction{accountId: freshAccount.id, timestamp: now(), type: "WITHDRAWAL", amount: requestedAmount, status: "FAILED", details: "Dispense failure"})
      ejectCard()
      return

    // Başarılıysa commit et
    commitDatabaseTransaction()
    unlockResource("atm_cash")
    unlockResource("account_" + account.id)

    // Güncelleme sonrası success log
    logTransaction(Transaction{accountId: freshAccount.id, timestamp: now(), type: "WITHDRAWAL", amount: requestedAmount, status: "SUCCESS", details: "Dispensed: " + stringify(banknotePlan)})

    display("Lütfen nakitinizi alınız: " + requestedAmount + " TL")
    // Makbuz isteği
    if promptUserYesNo("Makbuz ister misiniz? (E/H)"):
      printReceipt(freshAccount, txId)

    ejectCard()
    return

  catch Exception e:
    rollbackDatabaseTransaction()
    unlockResource("atm_cash")
    unlockResource("account_" + account.id)
    logTransaction(Transaction{accountId: account.id, timestamp: now(), type: "WITHDRAWAL", amount: requestedAmount, status: "FAILED", details: "Exception: " + e.message})
    display("Beklenmeyen hata. Lütfen bankanızla iletişime geçin.")
    ejectCard()
    return
