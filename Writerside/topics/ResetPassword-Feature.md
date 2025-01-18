# Reset Password Feature

A lehetővé teszi, hogy a felhasználók biztonságosan visszaállítsák jelszavaikat, ha elfelejtették azokat. Ez a funkció magában foglalja a visszaállítási kérelem indítását és egy megerősítő kód segítségével történő hitelesítést.

---

## Megvalósítás Részletei

### Kulcsosztályok

#### Szerver Oldalon
- **BeginResetPasswordCommandHandler**: Jelszó-visszaállítás kezdeményezését kezeli.
- **ConfirmResetPasswordCommandHandler**: Jelszó-visszaállítás megerősítését kezeli.
- **IamController**: API végpontokat definiál a funkcióhoz.
- **MailService**: E-mailek küldését biztosítja az `IMailService` interfészen keresztül.
- **MailgunServiceWrapper**: Mailgun API integráció.
- **PasswordReset**: Jelszó-visszaállítást reprezentáló entitás.

#### Kliens Oldalon
- **ForgottenPasswordEmailMessage.razor**: E-mail emlékeztető oldal implementációja.
- **PasswordReset.razor**: Jelszó-visszaállító oldal.
- **ResetPasswordClientCommand**: Kliens parancs a folyamat kezdeményezésére.
- **ConfirmResetPasswordClientCommand**: Kliens parancs a folyamat megerősítésére.

---

### Jelszó Visszaállítás Folyamata

1. **Kérelem Kezdeményezése**: A felhasználó a `/api/v1/iam/reset-password/begin` végpontot hívja meg.
2. **Kód Generálása**: A rendszer generál egy megerősítő kódot, és e-mailben elküldi.
3. **Megerősítés**: A felhasználó a `/api/v1/iam/reset-password/confirm` végpontot használja a kód és az új jelszó megadásával.
4. **Jelszó Frissítése**: A rendszer érvényesíti a kódot és frissíti a jelszót.

---

## API Végpontok

### Jelszó-visszaállítás Kezdeményezése
- **Végpont**: `/api/v1/iam/reset-password/begin`
- **Módszer**: POST
- **Paraméterek**:
  - `Email` (string, kötelező)
- **Válasz**:
  - `Success` (bool)
  - `Message` (string)

### Jelszó-visszaállítás Megerősítése
- **Végpont**: `/api/v1/iam/reset-password/confirm`
- **Módszer**: POST
- **Paraméterek**:
  - `Email` (string, kötelező)
  - `ConfirmationCode` (string, kötelező)
  - `NewPassword` (string, kötelező)
- **Válasz**:
  - `Success` (bool)
  - `Message` (string)

---

## Bemeneti Szabályok

### Kezdeményezés
- **Email**:
  - Nem lehet üres.
  - Érvényes e-mail cím szükséges.

### Megerősítés
- **Email**:
  - Nem lehet üres.
  - Érvényes e-mail cím szükséges.
- **ConfirmationCode**:
  - 6 karakter hosszú.
- **NewPassword**:
  - Nem lehet üres.

---

## Biztonsági Szempontok

- **Megerősítő Kódok**:
  - Generálás: `Guid.NewGuid().ToString().Substring(0, 6).ToUpper()`.
  - Érvényesség: 30 perc.
- **Jelszótárolás**:
  - `PasswordUtility.HashPassword` használatával.

---

## Tesztelés

### Végpontok Közötti Tesztek
- **Elhelyezkedés**: `vetcms/tests/vetcms.ServerApplicationTests/E2ETests/Features/IAM/ResetPasswordE2ETests.cs`.
- **Fő Tesztesetek**:
  - **BeginResetPassword_ShouldReturnSuccess_WhenUserExists**: Teszteli, hogy a jelszó-visszaállítási kérés sikeres, ha a felhasználó létezik.
  - **BeginResetPassword_ShouldReturnFailure_WhenUserDoesNotExist**: Teszteli, hogy a jelszó-visszaállítási kérés sikertelen, ha a felhasználó nem létezik.
  - **ConfirmResetPassword_ShouldReturnSuccess_WhenCodeIsValid**: Teszteli, hogy a jelszó-visszaállítási kérés sikeres, ha a megerősítő kód érvényes.
  - **ConfirmResetPassword_ShouldReturnFailure_WhenCodeIsInvalid**: Teszteli, hogy a jelszó-visszaállítási kérés sikertelen, ha a megerősítő kód érvénytelen.
  - **BeginResetPassword_ShouldFailValidation_WhenEmailIsInvalid**: Teszteli, hogy a jelszó-visszaállítási kérés nem érvényes, ha az e-mail cím érvénytelen.
  - **ConfirmResetPassword_ShouldFailValidation_WhenEmailIsInvalid**: Teszteli, hogy a jelszó-visszaállítási kérés nem érvényes, ha az e-mail cím érvénytelen.
  - **ConfirmResetPassword_ShouldFailValidation_WhenConfirmationCodeIsInvalid**: Teszteli, hogy a jelszó-visszaállítási kérés nem érvényes, ha a megerősítő kód érvénytelen.

### Integrációs Tesztek
- **Elhelyezkedés**: `vetcms/tests/vetcms.ServerApplicationTests/IntegrationTests/Features/IAM/ResetPasswordIntegrationTests.cs`.
- **Tesztesetek**:
  - **ConfirmResetPassword_ShouldReturnSuccess_WhenCodeIsValid**: Teszteli, hogy a jelszó-visszaállítás sikeres, ha a megerősítő kód érvényes.
  - **ConfirmResetPassword_ShouldReturnFailure_WhenCodeIsInvalid**: Teszteli, hogy a jelszó-visszaállítás sikertelen, ha a megerősítő kód érvénytelen.
  - **BeginResetPassword_ShouldReturnSuccess_WhenUserExists**: Teszteli, hogy a jelszó-visszaállítási kérés sikeres, ha a felhasználó létezik.
  - **BeginResetPassword_ShouldReturnFailure_WhenUserDoesNotExist**: Teszteli, hogy a jelszó-visszaállítási kérés sikertelen, ha a felhasználó nem létezik.
  - **ConfirmResetPassword_ShouldFailValidation_WhenEmailIsInvalid**: Teszteli, hogy a jelszó-visszaállítási kérés nem érvényes, ha az e-mail cím érvénytelen.
  - **ConfirmResetPassword_ShouldFailValidation_WhenConfirmationCodeIsInvalid**: Teszteli, hogy a jelszó-visszaállítási kérés nem érvényes, ha a megerősítő kód érvénytelen.
  - **BeginResetPassword_ShouldFailValidation_WhenEmailIsInvalid**: Teszteli, hogy a jelszó-visszaállítási kérés nem érvényes, ha az e-mail cím érvénytelen.

### Egységtesztek
- **Elhelyezkedés**: `vetcms/tests/vetcms.ServerApplicationTests/UnitTests/Features/IAM/ResetPasswordUnitTests.cs`.
- **Tesztesetek**:
  - **ConfirmResetPassword_ShouldReturnSuccess_WhenCodeIsValid**: Teszteli, hogy a jelszó-visszaállítás sikeres, ha a megerősítő kód érvényes.
  - **ConfirmResetPassword_ShouldReturnFailure_WhenCodeIsInvalid**: Teszteli, hogy a jelszó-visszaállítás sikertelen, ha a megerősítő kód érvénytelen.
  - **BeginResetPassword_ShouldReturnSuccess_WhenUserExists**: Teszteli, hogy a jelszó-visszaállítási kérés sikeres, ha a felhasználó létezik.
  - **BeginResetPassword_ShouldReturnFailure_WhenUserDoesNotExist**: Teszteli, hogy a jelszó-visszaállítási kérés sikertelen, ha a felhasználó nem létezik.
  - **SendPasswordResetEmailAsync_ShouldCallSendEmailAsync**: Teszteli, hogy a `SendPasswordResetEmailAsync` metódus meghívja-e az `IMailDeliveryProviderWrapper` interfész `SendEmailAsync` metódusát.

---

## Architektúra

1. **Prezentációs Réteg**: API végpontok az `IamController` által.
2. **Alkalmazási Réteg**: Parancskezelők az üzleti logikához.
3. **Domain Réteg**: Alapvető logika és entitások.
4. **Infrastruktúra Réteg**: Külső szolgáltatások és adat-hozzáférés.

---

## Kliens Oldali Implementáció


### Folyamat kezdeményezése:
A kliensoldali implementáció során a jelszó-visszaállítás kezdeményezésére szolgáló felület az alábbi főbb komponenseket tartalmazza:

1. **ForgottenPasswordEmailMessage.razor**:
  - Ez az oldal biztosítja a felhasználók számára, hogy megadják az e-mail címüket, amelyre a jelszó-visszaállítási folyamat indításához szükséges értesítés kerül kiküldésre.
  - Az oldal funkcionalitása magában foglalja az űrlap helyes validációját, a megfelelő üzenetek megjelenítését sikeres vagy sikertelen beküldés esetén.

### Folyamat megerősítése:
A jelszó visszaállításának tényleges végrehajtását lehetővé tevő felület az alábbiakban valósul meg:

2. **PasswordReset.razor**:
  - Ez az oldal biztosítja, hogy a felhasználók megadhassák az e-mail címüket, a megerősítő kódot és az új jelszót.
  - Az oldal hitelesíti az összes megadott adatot, és megfelelő vizuális visszajelzéseket nyújt az interakciók során.
  - A folyamat részeként a rendszer validálja a jelszó komplexitási követelményeket és a megerősítő kód érvényességét.

### Kliensparancsok
- **ResetPasswordClientCommand**:
  - A jelszó-visszaállítás kezdeményezését reprezentáló parancs, amely az API végpont meghívásáért felelős a felhasználó által megadott adatokkal.
  - Ez a parancs az ügyfél és a szerver közötti adatkommunikációért felelős.

- **ConfirmResetPasswordClientCommand**:
  - A jelszó-visszaállítás megerősítésére szolgáló kliensparancs, amely a felhasználó új jelszavát és a megerősítő kódot küldi el a szerver számára.
  - Gondoskodik arról, hogy minden adat biztonságosan és megfelelően legyen kezelve.

### Kliensoldali Parancskezelők
- A kliensoldalon implementált parancsokat külön kezelők dolgozzák fel, amelyek biztosítják a hibakezelést, a válaszok megfelelő feldolgozását, valamint a felhasználói élményt javító visszacsatolást.