# LoginUser Feature

## Általános Leírás

A **LoginUser** funkció lehetővé teszi a felhasználók számára, hogy e-mail címük és jelszavuk megadásával hitelesítsék magukat. Ez a funkció kliens- és szerveroldali komponensek integrációjával biztosítja a bejelentkezési folyamat kezelését, a hitelesítő adatok érvényesítését, valamint a hozzáférési tokenek generálását.


![LoginUser Funkció Áttekintés](LoginUser Feature.png)

## Jogosultsági Követelmények

- **Szükséges jogosultság:** `CAN_LOGIN`

---

## Komponensek

### Kliensoldali Implementáció

1. **LoginUserClientCommand.cs**
    - **Elérhetőség:** `vetcms/vetcms.ClientApplication/Features/IAM/LoginUser/LoginUserClientCommand.cs`
    - **Leírás:** Ez az osztály a felhasználó bejelentkezési folyamatának kliensoldali kezdeményezését valósítja meg. Tartalmazza a felhasználónév és a jelszó tulajdonságait.

### Szerveroldali Implementáció

1. **LoginUserCommandHandler.cs**

    - **Elérhetőség:** `vetcms/src/vetcms.ServerApplication/Features/IAM/LoginUser/LoginUserCommandHandler.cs`
    - **Leírás:** Ez az osztály kezeli a bejelentkezési logikát. Ellenőrzi a felhasználót az e-mail cím alapján, validálja a jelszót, ellenőrzi a jogosultságokat, majd generál egy hozzáférési tokent.

2. **LoginUserController.cs**

    - **Elérhetőség:** `vetcms/src/vetcms.ServerApplication/Features/IAM/LoginUser/LoginUserController.cs`
    - **Leírás:** Ez a vezérlő definiálja a felhasználói bejelentkezés API végpontját, és a beérkező kérelmeket a megfelelő parancskezelőhöz továbbítja.

3. **LoginUserApiCommand.cs**

    - **Elérhetőség:** `vetcms/SharedModels/Features/IAM/LoginUserApiCommand.cs`
    - **Leírás:** Ez az osztály az API parancs szerkezetét határozza meg, beleértve az e-mail és a jelszó tulajdonságokat, valamint a validálási logikát.

---

## API Specifikáció

- **Végpont:** `/api/v1/iam/login`

### Sikeres válasz

```json
{
  "Success": true,
  "PermissionSet": "1",
  "AccessToken": "jwtbearertoken"
}
```

- **HTTP kód:** `200`

### Hibás jelszó

```json
{
  "Success": false,
  "Message": "Hibás bejelentkezési adatok."
}
```

- **HTTP kód:** `200`

### Hibás e-mail cím

```json
{
  "Success": false,
  "Message": "Nem létező felhasználó."
}
```

- **HTTP kód:** `200`

---

## Architekturális Áttekintés

A **LoginUserCommandHandler** osztály a szerveroldalon valósítja meg a bejelentkezési logikát.

### Főbb Feladatok:

- Lekéri a felhasználót az e-mail cím alapján az **IUserRepository** segítségével.
- Ellenőrzi a felhasználói jogosultságokat (pl. `CAN_LOGIN`).
- Érvényesíti a jelszót a **PasswordUtility** osztállyal.
- Amennyiben minden ellenőrzés sikeres, generál egy JWT hozzáférési tokent az **IAuthenticationCommon** interfész által.

### Integráció Más Rendszerekkel

1. **Adatbázis**:
    - A felhasználói adatokat az `IUserRepository` interfész kezeli.
2. **Jogosultságkezelés**:
    - Az engedélyek a `PermissionFlags` alapján az `EntityPermissions` segítségével ellenőrizhetők.
3. **Token Generálás**:
    - A hozzáférési tokent az `IAuthenticationCommon` interfész generálja.
4. **Hiba- és Kivételkezelés**:
    - Az érvénytelen felhasználók vagy hibás jelszavak esetén megfelelő válaszüzenetet küld vissza az API-nak.
5. **Jelszó ellenőrzés**:
   - A `PasswordUtility` osztály a jelszavak biztonságos kezelésére szolgál.
   - A `VerifyPassword` metódus összehasonlítja a felhasználó által megadott jelszót a tárolt, sózott és hashelt jelszóval.

---

## Tesztelés

### Tesztlefedettség

- **Átlagos lefedettség:** 100%

### Végponttól Végpontig Tesztek (E2E)

A `LoginUser` funkció E2E tesztjei az alábbi helyen találhatók: `vetcms/tests/vetcms.ServerApplicationTests/E2ETests/Features/IAM/LoginUserE2ETest.cs`

1. **Sikeres bejelentkezés érvényes hitelesítő adatokkal.**
2. **Sikertelen bejelentkezés érvénytelen hitelesítő adatokkal.**
3. **Sikertelen bejelentkezés hiányzó e-mail cím esetén.**
4. **Sikertelen bejelentkezés hiányzó jelszó esetén.**
5. **Sikertelen bejelentkezés érvénytelen e-mail cím esetén.**

### Egységtesztek

Az egységtesztek helye: `vetcms/tests/vetcms.ServerApplicationTests/UnitTests/Features/IAM/LoginUserCommandHandlerTests.cs`


1. **Felhasználó nem található az adatbázisban.**
2. **Felhasználó nem rendelkezik bejelentkezési jogosultsággal.**
3. **Érvénytelen jelszó esetén sikertelen válasz.**
4. **Érvényes hitelesítő adatok esetén sikeres válasz.**

### Integrációs Tesztek

Az integrációs tesztek helye: `vetcms/tests/vetcms.ServerApplicationTests/IntegrationTests/Features/IAM/LoginUserCommandHandlerIntegrationTests.cs`

1. **Felhasználó nem létezik az adatbázisban.**
2. **Érvénytelen jelszó esetén sikertelen válasz.**
3. **Érvényes hitelesítő adatok esetén sikeres válasz.**
4. **Felhasználó nem rendelkezik bejelentkezési jogosultsággal.**

