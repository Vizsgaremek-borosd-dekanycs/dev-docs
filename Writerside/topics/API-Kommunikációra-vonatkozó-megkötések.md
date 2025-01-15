# API Konvenciók

Az alábbiakban összefoglaljuk az API-k használatával kapcsolatos legfontosabb szabályokat és elvárásokat, amelyek biztosítják a rendszer megbízhatóságát, biztonságát és teljesítményét.

> A végpontok dokumentációját az API Végpont dokumentáció tartalmazza [TODO]

{style="note"}
## **Kérés-típusok és HTTP metódusok**
- **GET**: Csak adatok lekérésére szolgál, nem módosíthatja a szerver állapotát.
- **POST**: Új erőforrások létrehozására vagy műveletek kezdeményezésére használható.
- **PUT/PATCH**: Létező erőforrások módosítására.
    - **PUT**: Teljes erőforrás felülírására.
    - **PATCH**: Részleges frissítésre.
- **DELETE**: Erőforrások törlésére.

## Modellek regisztálása

Minden a kommunikációhoz szükséges modelt a SharedModels-en keresztül kell regisztrálni, hogy a kliens és a szerver modul által is könnyen hozzáférhető legyen.
Ezekre a továbbiakban ApiCommand-ként fogunk hivatkozni.

Az ApiCommand recordot tartalmazó fájlt a `SharedModels/Features/[Feature Group]/[hozzá tartozó feature]` mappában kell létrehozni.
A fájlnak tartalmaznia kell a record definíciót és a hozzá tartozó validátort, illetve a választ.

> A validátor gondoskodik a kérés validálásáról, mind kliens mind szerveroldalon a MediatR pipeline és a fluent validation segítségével.

{style="note"}

> A validációra vonatkozó dokumentációt a Fluent Validaiton komponens dokumentációja tartalmazza
{style="tip"}

### Példa: LoginUserCommand
#### Fájl: `SharedModels/Features/IAM/LoginUserApiCommand.cs`

```c#
public record LoginUserApiCommand() : UnauthenticatedApiCommandBase<LoginUserApiCommandResponse>
{
    public string Email { get; init; }
    public string Password { get; init; }

    
    public override string GetApiEndpoint()
    {
        return Path.Join(ApiBaseUrl, "/api/v1/iam/login");
    }

    public override HttpMethodEnum GetApiMethod()
    {
        return HttpMethodEnum.Post;
    }
}

public class LoginUserCommandValidator : AbstractValidator<LoginUserApiCommand>
{
    public LoginUserCommandValidator()
    {
        RuleFor(x => x.Email).NotEmpty().EmailAddress();
        RuleFor(x => x.Password).NotEmpty();
    }
}

public record LoginUserApiCommandResponse : ICommandResult
{
    public bool Success { get; set; }
    public string Message { get; set; }
    public string PermissionSet { get; set; }
    public string AccessToken { get; set; }
    public EntityPermissions GetPermissions()
    {
        return new EntityPermissions(PermissionSet);
    }
}
```
> A példában szereplő `UnauthenticatedApiCommandBase<T>` öröklést az Autentikáció és autorizáció részben fedjük le.

{style="warning"}

## Kérések feldolgozása a szerver oldalon

## **Autentikáció és autorizáció**
- Minden hitelesíteni kívánt API-híváshoz **JWT-tokent** vagy más hitelesítési mechanizmust kell mellékelni.
- A jogosultsági szinteknek megfelelően kell korlátozni a hozzáférést (pl. Admin, Felhasználó, Vendég).


## **Adatformátum**
- A kérések és válaszok alapértelmezett formátuma **JSON**.
- A válaszok mindig tartalmazzanak a következő metaadatokat:
    - **status**: HTTP státuszkód (pl. 200, 404).
    - **message**: Rövid leírás a válasz állapotáról.
    - **data**: A tényleges adatobjektum (ha van).

### Példa válasz:
```json
{
  "status": 200,
  "message": "Sikeres lekérés",
  "data": {
    "userId": 123,
    "username": "johndoe"
  }
}
```

## **Sebességkorlátok (Rate limiting)**
- Egy adott IP-címről érkező hívások száma percenként legfeljebb **100**.
- A sebességkorlát átlépése esetén a válasz:
    - HTTP státuszkód: **429 Too Many Requests**.
    - Üzenet: "Túl sok kérés. Próbálja újra később."

## **Idempotencia**
- Az alábbi HTTP metódusok idempotens működést kell biztosítsanak:
    - **GET**, **PUT**, **DELETE**: Többszöri meghívásuknak nem szabad különböző eredményt okozni.
    - **POST**: Nem idempotens, de biztosítani kell az ismétlődő kérések megfelelő kezelését (pl. duplikáció elkerülése).

## **Hibakezelés**
- A szerver által visszaadott hibakódoknak az alábbi kategóriákba kell tartozniuk:
    - **4xx**: Kliensoldali hibák (pl. 400 - Hibás kérés, 401 - Jogosulatlan hozzáférés).
    - **5xx**: Szerveroldali hibák (pl. 500 - Szerverhiba).
- A válaszok tartalmazzák a hiba leírását:
```json
{
  "status": 404,
  "message": "A kért erőforrás nem található."
}
```

## **Titkosítás**
- Minden adatforgalmat **HTTPS** felett kell bonyolítani, hogy biztosítsuk az adatok titkosítását.

## **API Verziózás**
- A stabilitás érdekében az API-kat verziózni kell:
    - Verzió formátuma: `/v1/`, `/v2/` stb.
    - A különböző verziók egymástól függetlenül érhetők el, hogy a meglévő implementációk kompatibilisek maradjanak.

## **Végpontok elnevezésére vonatkozó megkötések**
A végpontok elnevezésében követendő szabályok a következők:

- **RESTful konvenciók**:
    - A végpontok nevei az erőforrások nevét tükrözzék (többes szám használatával, pl. `/users` a felhasználókhoz).
    - Az URL-ek ne tartalmazzanak igéket, mivel a műveleteket a HTTP metódusok (GET, POST, stb.) határozzák meg.

- **Konzisztens névhasználat**:
    - Minden végpont angol nyelven legyen megadva.
    - Használjunk **kötőjelet (`-`)** a több szóból álló nevek elválasztására (pl. `/user-profiles`).

- **Hierarchikus felépítés**:
    - Az erőforrások közötti kapcsolatokat az URL struktúrában tükrözzük:
        - Példa: Egy felhasználó rendelései: `/users/{userId}/orders`.

- **Idempotens és szűkített végpontok**:
    - Az erőforrások részhalmazait elérő végpontokat alvégpontként adjuk meg:
        - Példa: Egy specifikus rendelés lekérése: `/orders/{orderId}`.

- **Verziószám az URL-ben**:
    - Az API verzióját az URL első szegmensében tüntessük fel:
        - Példa: `/v1/users`, `/v2/orders`.

- **Speciális műveletek megkülönböztetése**:
    - Ha az alap CRUD (Create, Read, Update, Delete) műveleteken kívüli speciális funkciókat kell támogatni, külön alvégpontot használjunk:
        - Példa: `/users/{userId}/activate` egy felhasználó aktiválására.

### Példák végpontok kialakítására:
| HTTP Metódus | Végpont               | Funkció                                |
|--------------|-----------------------|----------------------------------------|
| GET          | `/v1/users`          | Minden felhasználó lekérése            |
| POST         | `/v1/users`          | Új felhasználó létrehozása             |
| GET          | `/v1/users/{userId}` | Egy konkrét felhasználó adatainak lekérése |
| PUT          | `/v1/users/{userId}` | Egy felhasználó adatainak frissítése   |
| DELETE       | `/v1/users/{userId}` | Egy felhasználó törlése                |

## **Tesztlefedettségre vonatkozó követelmények**
Az API megbízhatóságának, stabilitásának és helyes működésének biztosítása érdekében az alábbi tesztlefedettségi irányelveket kell követni:

### **Unit tesztek**
- **Cél**: Az API-t támogató üzleti logika és funkciók helyességének ellenőrzése izolált környezetben.
- **Minimum lefedettség**: Az API featureök legalább **90%-os lefedettsége** unit tesztekkel.
- **Tesztelendő komponensek**:
    - Validációs logikák.
    - Egyedi üzleti szabályok.

### **Integrációs tesztek**
- **Cél**: A komponens funkcionalitásának és az általa használt további komponensekkel való együttműködésének a tesztelése
- **Tesztelendő elemek**:
    - Adatbázis-interakciók (CRUD műveletek).

### **End To End (E2E) tesztek**
- **Cél**: Az API különböző komponenseinek (adatbázis, külső szolgáltatások, stb.) együttműködésének ellenőrzése.
- **Tesztelendő elemek**:
    - Végpontok teljes adatfolyamatainak ellenőrzése.
    - Külső API-k hívásainak szimulálása és visszatérési értékeik kezelése.
- **Példa**: Egy új rendelés létrehozásának tesztje, beleértve az adatbázisba történő mentést és a külső fizetési szolgáltatóval való interakciót.

### **Automatizált tesztelés**
- Az összes unit, integrációs és E2E tesztet automatizálni kell, és integrálni kell a CI/CD folyamatba.
- Az API minden módosításakor a teljes tesztcsomagnak automatikusan le kell futnia.

### **Tesztlefedettségi riport**
- A fejlesztési ciklus végén készül