# Általános információk

## Projekt Adatok

|                  |                                               |                                                                   |
|------------------|-----------------------------------------------|-------------------------------------------------------------------|
| **Projekt neve** | Állatorovosi nyilvántartó rendszer            |                                                                   |
| **Téma**         | Backend és Fronted                            | A szoftvernek várhatóan nagyobb kitettsége ezen két tantárgy felé |
| **Témavezető**   | Méhes József                                  | A szoftvernek várhatóan nagyobb kitettsége ezen két tantárgy felé |
| **Fő platform**  | Web                                           |                                                                   |
| **Osztály**      | 13T-II                                        |                                                                   |
| **Csoporttagok** | Dékány Csaba                                  | Boros Dániel                                                      |

## Projekt szükségességének indoklása

**Az állatorvosi nyilvántartó rendszer létrehozásának indoklása és egyedísége**

Az állatorvosi nyilvántartó rendszer elkészítésének egyik fő indoka az volt, hogy a meglévő rendszerek gyakran elavultak, nehezen kezelhetők, vagy nem teljeskörűen elégítik ki az állatorvosok és pácienseik igényeit. A digitalizáció terjedésével egyre nagyobb igény mutatkozott egy olyan rendszerre, amely gyors, biztonságos és könnyen elérhető bármilyen eszközről.

**A rendszer főbb előnyei és egyedísége:**

1. **Felhasználóbarát kezelőfelület** – Intuitív és letisztult design, amely lehetővé teszi a gyors navigálást, minimalizálva a betanulási időt.

2. **Rugalmas páciensnyilvántartás** – Az állatorvosok egyedi igényeihez igazodó, testreszabható adatbázis, amely lehetővé teszi a részletes betegkartonok vezetését, a kezelések és vizsgálatok pontos dokumentálását.

3. **Felhasználási rugalmasság** – A rendszer asztali gépről, tabletről és mobiltelefonról is elérhető, ezáltal az állatorvosok a rendelőn kívül is hatékonyan tudnak dolgozni.

4. **Papírmentes dokumentáció** – Minden kapcsolódó dokumentumok digitális formában történő kezelése csökkenti a papírfelhasználást, gyorsabb visszakereshetőséget biztosít, és csökkenti az elveszett dokumentumok kockázatát.

5. **Testreszabható funkcionalitás** – A rendszer lehetőséget biztosít egyéni beállításokra, így az állatorvosok saját igényeik szerint alakíthatják ki a használati módot.


## Felhasznált függőségek

| Függőség | Használat oka |
|------------|----------------|
| FluentValidation | Erősen típusos validációs szabályok létrehozásához. |
| MediatR | A mediátor minta megvalósításához kérések és válaszok kezelésére. |
| Microsoft.AspNetCore.Components.WebAssembly | Blazor WebAssembly alkalmazások építéséhez. |
| Microsoft.AspNetCore.Components.WebAssembly.DevServer | Fejlesztői szervert biztosít Blazor WebAssembly alkalmazásokhoz. |
| Microsoft.FluentUI.AspNetCore.Components | Fluent UI komponensek integrálásához ASP.NET Core alkalmazásokban. |
| Microsoft.FluentUI.AspNetCore.Components.Icons | Fluent UI ikonokat biztosít ASP.NET Core alkalmazásokhoz. |
| Blazored.LocalStorage | Egyszerűsíti a helyi tároló használatát Blazor alkalmazásokban. |
| Microsoft.AspNetCore.Http.Abstractions | HTTP absztrakciókat biztosít ASP.NET Core-hoz. |
| Microsoft.Extensions.DependencyInjection.Abstractions | Függőség befecskendezési absztrakciókat biztosít ASP.NET Core-hoz. |
| Microsoft.AspNetCore.Mvc.Versioning | Lehetővé teszi az ASP.NET Core MVC API-k verziózását. |
| Microsoft.EntityFrameworkCore | ORM adatbázis-hozzáféréshez. |
| Microsoft.EntityFrameworkCore.Design | Tervezési idejű eszközök Entity Framework Core-hoz. |
| Microsoft.EntityFrameworkCore.InMemory | Memóriabeli adatbázis-szolgáltató teszteléshez Entity Framework Core-ral. |
| Microsoft.EntityFrameworkCore.Sqlite | SQLite adatbázis-szolgáltató Entity Framework Core-hoz. |
| Microsoft.EntityFrameworkCore.SqlServer | SQL Server adatbázis-szolgáltató Entity Framework Core-hoz. |
| Microsoft.EntityFrameworkCore.Tools | Eszközök Entity Framework Core-hoz. |
| RestSharp | Egyszerűsíti a HTTP kéréseket .NET-ben. |
| Swashbuckle.AspNetCore | API dokumentáció generálása Swagger-rel. |
| coverlet.collector | Kódlefedettségi adatok gyűjtése .NET projektekhez. |
| Microsoft.AspNetCore.Mvc.Testing | Infrastruktúrát biztosít ASP.NET Core MVC alkalmazások teszteléséhez. |
| Microsoft.NET.Test.Sdk | Teszt SDK .NET projektekhez. |
| Moq | Könyvtár mock objektumok létrehozásához egységtesztekben. |
| xunit | Egységteszt keretrendszer. |
| xunit.runner.visualstudio | Visual Studio futtatókörnyezet xUnit tesztekhez. |
