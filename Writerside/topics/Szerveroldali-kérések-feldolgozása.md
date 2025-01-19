# Szerveroldali Kérések Feldolgozása

#### Bevezetés

Ez a dokumentum részletes áttekintést nyújt a szerveroldali kérések feldolgozásának működéséről. Bemutatja a rendszer különböző rétegeinek szerepét, az adatáramlás folyamatát, valamint a kérések kezelésében részt vevő komponensek közötti együttműködést.

#### HTTP Kérés

Az `GenericApiCommandHandler` által létrehozott HTTP-kérések az ASP.NET Core API-ban kerülnek feldolgozásra. Ezek a kérések számos információt tartalmaznak, amelyek meghatározzák a művelet végrehajtásának módját és feltételeit. A kérések tartalma az alábbiak szerint alakul:

- **Autorizáció**: A kliensoldali kérések általában tokeneken keresztül igazolják magukat.
- **HTTP-metódusok**: A műveletek típusa, például GET (lekérdezés), POST (létrehozás), PUT (módosítás), vagy DELETE (törlés).
- **Paraméterek és payload**: A metódusok végrehajtásához szükséges adatok.
- **Célútvonal (endpoint)**: Az API azon végpontja, amely a kérés célja.

#### Controller

A szerveroldali folyamat elsődleges eleme a **Controller**, amely felelős a kliensoldalról érkező adatok fogadásáért, előkészítéséért, majd a MediatR pipeline-ba történő továbbításáért. Az adatok továbbítása előtt a **`ApiCommandBaseExtensions.Prepare`** metódus biztosítja a megfelelő előkészítést. Ennek a metódusnak a legfontosabb feladatai a következők:

1. **AuthenticatedApiCommandBase és UnauthenticatedApiCommandBase osztályok szétválasztása**:
    - Az **AuthenticatedApiCommandBase** osztály esetében a metódus kinyeri a felhasználó tokenjét az HTTP kérésből.
    - Az **UnauthenticatedApiCommandBase** osztály adatait további feldolgozás nélkül továbbítja.

2. **Token kezelése**:
    - A megfelelő autentikáció biztosításához a tokeneket a kérésből az alábbi módszerrel vonja ki:

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

#### Server Behaviours

Az előkészített kérések a MediatR pipeline-ba kerülnek, amelyben különböző szerveroldali viselkedési osztályok (behaviours) szűrik és dolgozzák fel azokat. Ezek az osztályok a **`ServerDependencyInitializer`** segítségével kerülnek regisztrálásra, meghatározott sorrendben.

##### Pipeline Regisztráció:

```c#
services.AddMediatR(options =>
{
    options.RegisterServicesFromAssembly(typeof(ServerDependencyInitializer).Assembly);
    options.AddOpenBehavior(typeof(ValidationBehaviour<,>));
    options.AddOpenBehavior(typeof(UserValidationBehavior<,>));
    options.AddOpenBehavior(typeof(PermissionRequirementBehaviour<,>));
});
```

##### Viselkedési osztályok és szerepük:

1. **ValidationBehaviour**:
    - Ez az osztály a **`SharedModels`** gyűjtemény része.
    - Elsődleges feladata az `ApiCommandBase`-ből származtatott objektumok érvényességének ellenőrzése.
    - A `CommandValidator` által végzett validáció alapján a hibás adatok elutasításra kerülnek.

2. **UserValidationBehavior**:
    - Az `AuthenticatedApiCommandBase` osztályt használó objektumokat kezeli.
    - Ellenőrzi, hogy a felhasználói token létezik-e és érvényes-e a kérés feldolgozása során.

3. **PermissionRequirementBehaviour**:
    - Az `AuthenticatedApiCommandBase` osztályból származtatott objektumokat vizsgálja.
    - Megállapítja, hogy a felhasználó rendelkezik-e a szükséges jogosultságokkal az adott művelet végrehajtásához.


#### Command Handler

A Controller által továbbított kérések végrehajtása a **Command Handlerekben** történik. Ezek az osztályok az üzleti logika központjai, ahol a kérések végső feldolgozása történik. A Command Handlerek felelősségi körei a következők:

- Az üzleti szabályok alkalmazása az adott kérésre.
- Interakció más rendszerekkel vagy komponensekkel (például adatbázis-műveletek).
- Az eredmények előkészítése és azok továbbítása az `ApiCommandResponse` osztályon keresztül.

