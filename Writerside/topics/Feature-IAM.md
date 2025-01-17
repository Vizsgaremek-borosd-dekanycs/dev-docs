# Feature Group:  IAM

## LoginUser Feature
### Általános
A LoginUser Feature lehetővé teszi a felhasználók számára, hogy e-mail címük és jelszavuk megadásával hitelesítsék magukat. Ez a funkció szerver- és kliensoldali komponensek kombinációjával valósul meg a bejelentkezési folyamat kezelésére, a felhasználói hitelesítő adatok érvényesítésére és a hozzáférési tokenek generálására.
[kép: https://docs.google.com/drawings/d/1rMhfcdf2PDWzXor0b5WHI8KAb5k4aMFn9lqtHwS2poE/edit]

### Komponensek
1. **LoginUserCommandHandler.cs**
 - **Hely:** [Forráskód](https://github.com/Vizsgaremek-borosd-dekanycs/Vizsgaremek/blob/ecb02afcc95ace3721e666354ec24d44712eb524/vetcms/src/vetcms.Application/Features/IAM/LoginUser/LoginUserCommandHandler.cs
 ) - **leírás:** Ez az osztály kezeli a bejelentkezési logikát. Lekéri a felhasználót e-mailben, ellenőrzi a jelszót, ellenőrzi a bejelentkezési jogosultságokat, és létrehoz egy hozzáférési tokent, ha a hitelesítő adatok érvényesek.

2. **LoginUserController.cs**
 - **Hely:** [Forráskód](https://github.com/Vizsgaremek-borosd-dekanycs/Vizsgaremek/blob/ecb02afcc95ace3721e666354ec24d44712eb524/vetcms/src/vetcms.Application/Features/IAM/LoginUser/LoginUserController.cs
 ) - **Leírás:** Ez a vezérlő határozza meg a felhasználói bejelentkezés API végpontját. Fogadja a bejelentkezési kérelmet, előkészíti a parancsot, és elküldi a közvetítőnek feldolgozásra.


3. **LoginUserClientCommand.cs**
 - **Hely:** [Forráskód](https://github.com/Vizsgaremek-borosd-dekanycs/Vizsgaremek/blob/ecb02afcc95ace3721e666354ec24d44712eb524/vetcms/vetcms.ClientApplication/Features/IAM/LoginUser/LoginUserClientCommand.cs
 ) - **Description:** Ez az osztály  a felhasználó bejelentkezésének kliensoldali kezdeményezését reprezentálja. Tartalmazza a felhasználónév és a jelszó tulajdonságait.
4. **LoginUserApiCommand.cs**
 - **Hely:** [Forráskód](https://github.com/Vizsgaremek-borosd-dekanycs/Vizsgaremek/blob/ecb02afcc95ace3721e666354ec24d44712eb524/vetcms/SharedModels/Features/IAM/LoginUserApiCommand.cs
 ) - **Leírás:** Ez az osztály határozza meg a login API parancs szerkezetét és érvényesítését. Tartalmazza az e-mail és a jelszó tulajdonságait, az API végpont és a metódus típusának lekérdezésére szolgáló metódusokat, valamint a parancs validálóját.


### API
> /api/v1/iam/login

#### Success
```json
{
    Success : true,
    PermissionSet : "1",
    AccessToken = "jwtbearertoken"
}
```
> HTTP Code 200

#### Invalid Password
```json
{
    Success : false,
    Message : "Hibás bejelentkezési adatok."
}
```
> HTTP Code 200

#### Invalid Email
```json
{
    Success : false,
    Message : "Nem létező felhasználó"
}   
```
> HTTP Code 200

### Tesztelés
> Átlagos tesztlefedettség: 100%

A LoginUser Feature tesztjei különböző forgatókönyvekre terjednek ki:

1. **E2E tesztek**:
    - A sikeres bejelentkezés ellenőrzése érvényes hitelesítő adatokkal.
    - Ellenőrizze a sikertelenséget, ha a hitelesítő adatok érvénytelenek, az e-mail hiányzik, a jelszó hiányzik vagy az e-mail érvénytelen.
    - Az adatbázis beállításának és tisztításának biztosítása minden egyes teszthez.
    - [LoginUserE2ETest.cs](https://github.com/Vizsgaremek-borosd-dekanycs/Vizsgaremek/blob/ecb02afcc95ace3721e666354ec24d44712eb524/vetcms/tests/vetcms.ServerApplicationTests/E2ETests/Features/IAM/LoginUserE2ETest.cs)

2. **Egységtesztek**:
    - Kezelje a nem talált felhasználót, a bejelentkezési engedély hiányát, az érvénytelen jelszót és az érvényes hitelesítő adatokat.
    - [LoginUserCommandHandlerTests.cs](https://github.com/Vizsgaremek-borosd-dekanycs/Vizsgaremek/blob/ecb02afcc95ace3721e666354ec24d44712eb524/vetcms/tests/vetcms.ServerApplicationTests/UnitTests/Features/IAM/LoginUserCommandHandlerTests.cs)

3. **Integrációs tesztek**:
    - Ellenőrzi azokat a forgatókönyveket, amikor a felhasználó nem létezik, érvénytelen jelszó, érvényes hitelesítő adatok és bejelentkezési jog nélküli felhasználó.
    - [LoginUserCommandHandlerIntegrationTests.cs](https://github.com/Vizsgaremek-borosd-dekanycs/Vizsgaremek/blob/ecb02afcc95ace3721e666354ec24d44712eb524/vetcms/tests/vetcms.ServerApplicationTests/IntegrationTests/Features/IAM/LoginUserCommandHandlerIntegrationTests.cs)


## RegisterUser Feature