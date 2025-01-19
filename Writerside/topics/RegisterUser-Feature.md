# RegisterUser Feature

## Áttekintés

A `RegisterUser` funkció a felhasználói regisztráció kezelését végzi a rendszerben. A funkció több komponensből épül fel, amelyek célja a felhasználói adatok validálása, új felhasználók létrehozása és az információk biztonságos tárolása az adatbázisban. Az alábbiakban részletesen ismertetjük a funkció kulcsfontosságú elemeit és működését.

![](RegisterUser_Feature.png)

---

## Kulcsfontosságú Komponensek

### Kliens Oldali Komponensek

#### `vetcms/src/vetcms.ClientApplication/Features/IAM/RegisterUser/RegisterUserClientCommand.cs`
Ez a fájl tartalmazza a `RegisterUserClientCommand` osztályt, amely a kliens oldali regisztrációs parancsot definiálja.

#### `vetcms/src/vetcms.ClientApplication/Features/IAM/RegisterUser/RegisterUserClientCommandHandler.cs`
A `RegisterUserClientCommandHandler` osztály ebben a fájlban található, amely felelős a kliens oldali regisztrációs parancs kezeléséért és a szerver oldali API-val történő kommunikációért.

#### `vetcms/src/vetcms.BrowserPresentation/Pages/Features/IAM/RegisterPage.razor`
Ez a fájl tartalmazza a regisztrációs oldalt megvalósító Blazor komponenst. Ez biztosítja a felhasználói felületet a regisztrációs adatok beviteléhez és az űrlap beküldéséhez.

---

### Kétoldali Komponensek

#### `vetcms/SharedModels/Features/IAM/RegisterUserApiCommand.cs`
- **Osztályok**:
    - `RegisterUserApiCommand`: Definiálja a felhasználó regisztrációjára vonatkozó API-parancsot. Tulajdonságai: `Email`, `Password`, `PhoneNumber`, `Name`.
    - `RegisterUserApiCommandValidator`: Az API-parancs érvényességét ellenőrző osztály.
    - `RegisterUserApiCommandResponse`: Az API-válasz strukturálására szolgáló osztály.

---

### Szerver Oldali Komponensek

#### `vetcms/src/vetcms.ServerApplication/Features/IAM/RegisterUser/RegisterUserCommandHandler.cs`
Ez a fájl tartalmazza a `RegisterUserCommandHandler` osztályt, amely a `RegisterUserApiCommand` kezelésével új felhasználót hoz létre, és azt a tárolóba helyezi.

#### `vetcms/src/vetcms.ServerApplication/Features/IAM/RegisterUser/RegisterUserController.cs`
Az `IamController` osztály itt található, amely kezeli a felhasználó regisztrációjára vonatkozó HTTP POST kéréseket a `RegisterUser` metóduson keresztül.

#### `vetcms/src/vetcms.ServerApplication/Domain/Entity/User.cs`
A `User` entitás ebben a fájlban van definiálva. Tulajdonságai: `Email`, `PhoneNumber`, `VisibleName`, `Password`.

#### `vetcms/src/vetcms.ServerApplication/Features/IAM/PasswordUtility.cs`
A `PasswordUtility` osztály módszereket kínál a jelszavak hash-elésére és ellenőrzésére.

---

## API Dokumentáció

- **Végpont**: `POST /api/v1/iam/register`

### Kérés

A kérés törzsének a következő formátumú JSON objektumot kell tartalmaznia:

```json
{
  "email": "string",
  "password": "string",
  "phoneNumber": "string",
  "name": "string"
}
```

### Válasz

A válasz JSON objektuma az alábbi tulajdonságokat tartalmazza:

- `success`: Logikai érték, amely jelzi, hogy a regisztráció sikeres volt-e.
- `message`: Szöveges üzenet, amely információt ad a művelet eredményéről (amennyiben hibás).

---

## Bemeneti szabályok

A `RegisterUserApiCommandValidator` osztály a következő érvényesítési szabályokat alkalmazza:

- **Email**: Nem lehet üres, és érvényes e-mail formátummal kell rendelkeznie.
- **Password**: Nem lehet üres.
- **Name**: Nem lehet üres.
- **PhoneNumber**: Pontosan 11 karakter hosszúnak kell lennie.

---

## Hibaüzenetek

Az alábbi hibaüzenetek lehetségesek:

- "Az E-mail cím már foglalt."
- "Email mező nem maradhat üresen, és email formátumban kell lennie, pl.: kallapal@example.hu"
- "Jelszó mező nem lehet üres!"
- "Név mező nem lehet üres!"
- "Helytelen telefonszám!"

---

## Tesztelési Stratégia

A `RegisterUser` funkció tesztelési stratégiája az alábbi forgatókönyvekre terjed ki:

1. **Sikeres felhasználói regisztráció**: Biztosítja, hogy helyes adatok megadása esetén a regisztráció sikeres legyen.
2. **E-mail már létezik**: Ellenőrzi a megfelelő hibakezelést, ha a megadott e-mail cím már használatban van.
3. **Érvénytelen e-mail formátum**: Validálja a hibakezelést érvénytelen e-mail formátum esetén.
4. **Hiányzó adatok**: Biztosítja, hogy a regisztráció megfelelően hibát jelez, ha kötelező mezők hiányoznak.

---

## Architektúra

A `RegisterUser` funkció implementációja több rétegen át terjed, amelyben minden réteg meghatározott komponenseket használ:

### Prezentációs réteg
- **Komponens**: `RegisterPage.razor` (regisztrációs felhasználói felület).

### Kliens alkalmazás réteg
- **Parancs**: `RegisterUserClientCommand.cs`.
- **Parancskezelő**: `RegisterUserClientCommandHandler.cs`.

### Alkalmazási réteg
- **API-parancs**: `RegisterUserApiCommand.cs`.
- **Parancskezelő**: `RegisterUserCommandHandler.cs`.
- **Vezérlő**: `RegisterUserController.cs`.

### Domain réteg
- **Entitás**: `User.cs` (felhasználói entitás).
- **Segédosztály**: `PasswordUtility.cs` (jelszókezelési logika).

### Infrastruktúra réteg

Az infrastruktúra réteg biztosítja az adatok tényleges tárolását és kezelését, valamint az adatbázissal történő interakciókat. Az ebben a rétegben található komponensek a következők:

- **Tároló osztály**:
    - `UserRepository.cs` – Ez az osztály felelős az adatbázis műveletek elvégzéséért, például a felhasználói adatok mentéséért, lekérdezéséért, és egyéb CRUD műveletekért.

- **Adatbázis konfiguráció**: Az adatbázis-kapcsolatokat és a tárolási mechanizmusokat konfiguráló fájlok és osztályok. Ezek biztosítják az összhangot az alkalmazás logikája és az infrastruktúra között.

### Zárás

Az architektúra minden rétege szorosan együttműködik, hogy biztosítsa a felhasználók biztonságos és hatékony regisztrációját a rendszerbe. Az egyes rétegek közötti tiszta szeparáció elősegíti az egyszerű karbantartást, a skálázhatóságot és a hosszú távú fejleszthetőséget.
