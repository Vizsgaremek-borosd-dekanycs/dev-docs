# Kivételkezelés az alkalmazásban
A kivételkezelés rétegekre oszlik, ahol a külső rétegek általánosabb hibakezelést végeznek, míg a belső rétegek specifikusabb és célzottabb kivételkezelést biztosítanak.

## 1. réteg: Kezeletlen kivételek kezelése
1. Amennyiben egyik belső kivételkezelő sem tudja elfogni a hibát, végül az `ErrorController` kezeli azt, és strukturált JSON formátumban ad választ a kliensnek.
   ```json
   {
       "error": "UnhandledException",
       "message": "Váratlan hiba történt.",
       "details": "Stack trace részletei..."
   }
   ```
2. A JSON válasz a kliensoldalra kerül visszaküldésre, ahol a `GenericApiCommandHandler` osztály `HandleFailedRequest` metódusa feldolgozza azt.
3. A `HandleFailedRequest` egy `ApiCommandExecutionUnknownException` kivételt dob, amely értesíti a MediatR folyamatot a problémáról.
4. Az `ApiCommandExecutionUnknownException` kivételt a `UnhandledExceptionBehaviour` kezeli a MediatR folyamatban, és a felhasználó számára megjeleníti a `Dialog Service`, amely tartalmazhatja a kivétel stack trace információit is.

## 2. réteg: Kezelt futásidejű kivételek kezelése
Amennyiben a MediatR folyamatban, de még az üzleti logika előtt egy előre definiált, ismert kivétel keletkezik, azt általában az `ApiExceptionFilterAttribute` kezeli. Ezek a kivételek többnyire az alábbi területekre vonatkoznak:
- Általános kérésvalidáció
- Felhasználó hitelesítés
- Engedélyellenőrzés

1. A MediatR folyamat egy kezelt (`CommonApiLogicException`) kivételt dob, amely egy üzenetet és további paramétereket tartalmaz.
2. Az `ApiExceptionFilterAttribute` elkapja ezt a kivételt, és JSON válaszként továbbítja a kliens felé.
   ```json
   {
       "error": "CommonApiLogicException",
       "message": "Validációs hiba történt.",
       "details": "Validáció részletei..."
   }
   ```
3. A kliensoldalon a `GenericApiCommandHandler` `HandleFailedRequest` metódusa feldolgozza a JSON választ.
4. A `HandleFailedRequest` felismeri a kivétel típusát, majd egy `CommonApiLogicException` kivételt dob az `ApiLogicExceptionCode` felsorolt értékei és a részletek felhasználásával.
5. Az `ApiLogicExceptionCode` tartalmazza az adott hibakód leírását.
6. Az `UnhandledExceptionBehaviour` osztály `HandleApiLogicException` metódusa feldolgozza a hibakódot, és a megfelelő intézkedést hajtja végre.
7. Az így generált hibaüzenet a `Dialog Service` segítségével megjelenik a felhasználó számára.

## 3. réteg: Bemeneti adatok validálásának kezelése
A VetCMS architektúrájában az input validáció folyamata egyszerű és átlátható.

1. Minden API parancs válasza implementálja az `IClientCommand` interfészt, amely tartalmaz egy `message` és egy `success` attribútumot.
   ```c#
   public interface IClientCommand
   {
       string Message { get; set; }
       bool Success { get; set; }
   }
   ```
2. Ha validációs hiba lép fel, a `ValidationBehaviour` létrehoz egy `TResponse` példányt, amely a hibákat az `IClientCommand` `message` attribútumába konvertálja, miközben a `success` értékét `false`-ra állítja.
   ```c#
   public class ValidationBehaviour<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
       where TResponse : IClientCommand, new()
   {
       public async Task<TResponse> Handle(TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken cancellationToken)
       {
           var response = await next();
           if (!response.Success)
           {
               response.Message = "Validációs hibák történtek.";
           }
           return response;
       }
   }
   ```
3. A kliensoldali parancskezelő a kapott hibaüzenetet megjeleníti a felhasználónak.
4. A validációs folyamat mind a kliens-, mind a szerveroldalon végrehajtásra kerül, minimalizálva ezzel a szerver terhelését.

## 4. réteg: Üzleti logika futtatása és hibakezelés
Az üzleti logika végrehajtása a megfelelő `Feature` parancskezelőjében történik.

1. Ha a parancskezelőben ismert vagy ismeretlen kivétel fordul elő, a válasz sikerességi státusza `false` lesz.
2. A rendszer a megfelelő üzenetet továbbítja a felhasználónak, amelyből kiderül a hiba oka.
3. Az ilyen jellegű hibák leggyakrabban felhasználói hibákból adódnak.

Ez a struktúra biztosítja, hogy az alkalmazás megbízhatóan kezelje a különböző szintű kivételeket, miközben felhasználóbarát módon tájékoztatja az érintetteket a problémákról.

