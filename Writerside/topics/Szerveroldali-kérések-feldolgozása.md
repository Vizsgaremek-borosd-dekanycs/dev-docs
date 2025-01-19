# Szerveroldali Kérések Feldolgozása

---

### **Bevezetés**

Ez a dokumentum részletesen bemutatja a szerveroldali kérések feldolgozásának teljes folyamatát. Ismerteti a rendszer különböző rétegeinek szerepét, valamint a kommunikáció és adatkezelés lépéseit az alkalmazás belső működése során.

---

### **HTTP Kérések Felépítése**

Az általános HTTP-kérések az `GenericApiCommandHandler` komponensen keresztül indulnak el, majd eljutnak az ASP.NET Core API-hoz. A kérések az alábbi adatokat tartalmazzák:

- **Autorizáció:** Tokenek formájában biztosított jogosultságok.
- **HTTP-metódusok:** Az alkalmazás a GET, POST, PUT és DELETE módszereket támogatja.
- **Metódusokhoz kapcsolódó adatok:** Paraméterek és payload adatok, amelyek meghatározzák a műveleteket.
- **Célútvonal:** Az endpoint, amely meghatározza a kérés irányát.

Fontos megjegyezni, hogy a `GenericApiCommandHandler` a kérés előkészítéséért felel, nem pedig az egyéni válaszok kezeléséért.

---

### **Controller Működése**

A Controller osztály feladata a beérkezett adatok előkészítése, majd ezek továbbítása a MediatR pipeline-on keresztül a Command Handler részére. Az adatok feldolgozása előtt az `ApiCommandBaseExtensions.Prepare` metódus hajtja végre az alábbi lépéseket:

- **Autentikált Kérések:** Az `AuthenticatedApiCommandBase` objektumok esetében kinyeri a felhasználó tokenjét (Bearer token) a headerből, majd hozzárendeli azt az adott parancshoz.
- **Nem Autentikált Kérések:** Az `UnauthenticatedApiCommandBase` objektumokat továbbítja módosítás nélkül.

A Controller réteg biztosítja az adatok helyes formázását, mielőtt azokat továbbítaná a pipeline felé. Az előkészített adatok garantálják a folyamat további lépéseinek sikeres végrehajtását.

Példa kódrészlet az eljáráshoz:

```c#
namespace vetcms.ServerApplication.Common.Abstractions.Api
{
    internal static class ApiCommandBaseExtensions
    {
        public static AuthenticatedApiCommandBase<T> Prepare<T>(this AuthenticatedApiCommandBase<T> command, HttpRequest request)
            where T : ICommandResult
        {
            string token = Utility.ExtractBearerToken(request);
            command.BearerToken = token;

            return command;
        }

        public static UnauthenticatedApiCommandBase<T> Prepare<T>(this UnauthenticatedApiCommandBase<T> command, HttpRequest request)
            where T : ICommandResult
        {
            return command;
        }
    }
}
```

---

### **MediatR Behaviour Osztályok a Szerveroldalon**

Az előkészített kérések a MediatR pipeline-ba kerülnek, ahol a szerveroldali "viselkedés" osztályokon haladnak át. Ezek az osztályok a `ServerDependencyInitializer` osztályban vannak regisztrálva, a működési sorrend figyelembevételével. A sorrend különösen fontos, mert meghatározza a viselkedések hatásának sorrendiségét, és biztosítja az ellenőrzési lépések helyes végrehajtását.

Az alábbi kódrészlet mutatja a pipeline konfigurálását:

```c#
services.AddMediatR(options =>
{
    options.RegisterServicesFromAssembly(typeof(ServerDependencyInitializer).Assembly);
    options.AddOpenBehavior(typeof(ValidationBehaviour<,>));
    options.AddOpenBehavior(typeof(UserValidationBehavior<,>));
    options.AddOpenBehavior(typeof(PermissionRequirementBehaviour<,>));
});
```

Az egyes osztályok szerepe és futási sorrendje:

1. **ValidationBehaviour:**
   - Helye: `SharedModels`.
   - Feladata: Az `ApiCommandBase` osztály leszármazottainak validálása, a hibás bemenetek visszautasítása a `CommandValidator` segítségével.

2. **UserValidationBehavior:**
   - Helye: `ServerApplication`.
   - Feladata: Az `AuthenticatedApiCommandBase` objektumok kezelése, amelynek során ellenőrzi a felhasználó token létezését és érvényességét.

3. **PermissionRequirementBehaviour:**
   - Helye: `ServerApplication`.
   - Feladata: Ellenőrzi, hogy az adott felhasználó rendelkezik-e a szükséges jogosultságokkal az adott művelet elvégzéséhez.

---

### **Command Handler**

A Controller osztályok által készített parancsok feldolgozása a Command Handler komponensekben történik. Ezek az osztályok hajtják végre az üzleti logikát, miközben interakcióba lépnek további rendszerkomponensekkel, például adatbázisokkal vagy külső szolgáltatásokkal.

A feldolgozás eredménye az `ApiCommandResponse` objektumon keresztül kerül vissza a klienshez, biztosítva a megfelelő válaszformátumot és státuszinformációkat. Az `ApiCommandResponse` objektum központi szerepet játszik az eredmények reprezentálásában, megkönnyítve a hibák kezelését és az eredmények vizualizációját.

---

Ez a dokumentum átfogó képet nyújt a szerveroldali kérések kezeléséről, kiemelve a működésüket segítő kulcsfontosságú mechanizmusokat és komponenseket. Gyakorlati alkalmazásként javasolt a pipeline viselkedésének rendszeres tesztelése és a kulcskomponensek naprakészen tartása a biztonságos és hatékony működés érdekében.