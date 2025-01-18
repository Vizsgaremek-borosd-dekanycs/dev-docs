# Kérések feldolgozása a kliensoldalon

## Bevezetés

Ez a dokumentum részletesen ismerteti a kliensoldali kérések feldolgozásának folyamatát, amely magában foglalja a különböző rétegek működését, valamint azok szerepköreit. A cél, hogy áttekinthető legyen, hogyan kommunikálnak a kliensoldali komponensek a háttérrendszerrel a hatékony és megbízható adatfeldolgozás érdekében.

## Webes felület

- A parancsok végrehajtása a felhasználók számára elérhető webes felületen kezdődik.
- Ez a réteg biztosítja a rendszerrel való interakció elsődleges pontját, ugyanakkor a legkevesebb rálátással rendelkezik a teljes architektúra működésére.
- A felhasználói műveletek a funkciónak megfelelő "Client Command"-ba kerülnek továbbításra, a MediatR keretrendszer alkalmazásával.

**Fájl:** `\Pages\Features\IAM\LoginPage.razor`

```c#
private async void Login()
{
    LoginUserClientCommand userClientCommand = new LoginUserClientCommand(DialogService)
    {
        Username = username,
        Password = password
    };
    await Mediator.Send(userClientCommand);
}
```

### Hibakezelés

- Az alkalmazás hibakezelési folyamata biztosítja, hogy mind a kliens-, mind a szerveroldali problémákról értesítést kapjon a felhasználó a **DialogService** segítségével.
- A hiba szintjétől függ a hibakezelés helye.
  - `ClientCommandHandler`: olyan problémákból, ami a felhasználónak későbbi
  - `ApiCommand`: validálásnál az `ApiCommandValidator` (fleunt validation) segítségével
  - `CommandHandler`: adatbázis adataival kapcsolatos problémák pl: nem lehet egy íméllel két fiókot létrehozni
  - `GenericApiCommandHandler`: API kommunikációs hibáknál
  - `UnhandledExceptionBehaviour`:  ismeretlen hiba esetén, ha más hibakezelő nem tudta kezelni
- A "Client Command" osztályok paraméterként kapják meg a szükséges hibakezelési információkat, ezáltal lehetővé téve a pontos és célzott visszajelzéseket.

## Client Command

- A Client Command osztályok feladata a beérkező adatok előkészítése és a "Communication Bus"-on való továbbítása.
- Minden Client Command implementálja az `IClientCommand<bool>` interfészt, amely biztosítja a pipeline folyamatosságát és megfelelő működését.

## Client Behaviour

- Az elsődleges adatellenőrzéseket a Client Behaviour osztályok végzik. Csak az `IClientCommand<T>` interfészből származtatott osztályokkal dolgoznak.
- Ezek az osztályok biztosítják a kérések megfelelő továbbítását a pipeline következő szakaszára. Amennyiben egy adott szakasz nem képes kezelni a beérkező adatokat, a rendszer értesítést küld a felhasználónak. Sikeres vizsgálat esetén a feldolgozás tovább folytatódik.

## Client Command Handler

- A `ClientCommand` objektumok a Client Command Handler segítségével kerülnek továbbításra az API irányába.
- Ez az osztály elvégzi a backendre nem vonatkozó ellenőrzéseket, és a kapott API válaszokat feldolgozza, majd a felhasználó számára továbbítja azokat.

## API Command

- Az API-val való kommunikációhoz szükséges paraméterek, mint például az endpointok és a HTTP metódusok, az API Command osztályokban kerülnek definiálásra.
- Az osztályok gondoskodnak az adatok validálásáról és a válaszok megfelelő kezeléséről is.

```c#
public override string GetApiEndpoint()
{
    return Path.Join(ApiBaseUrl, "/api/v1/iam/register");
}

public override HttpMethodEnum GetApiMethod()
{
    return HttpMethodEnum.Post;
}
```

## API Command Handler

- Az API Command Handler osztályok felelősek az adatbázisból lekért adatok összevetéséért, lekérdezéséért vagy feltöltéséért.
- Az adatokat ideiglenesen helyben tárolja, amíg az API nem kap közvetlen utasítást a megfelelő művelet végrehajtására.
- A bemeneti adatok megfelelőségét ellenőrzi, és az API válaszait az `ApiCommandResponse` osztály dolgozza fel, amely aztán továbbítja az eredményeket a `ClientCommandHandler` felé.

## Generic API Command Handler

- A MediatR keretrendszer lehetőséget ad arra, hogy minden funkcióhoz egyedi `ApiCommandHandler`-t hozzunk létre. Ezek az osztályok a `GenericApiCommandHandler`-ből származnak.
- A Generic API Command Handler végrehajtja az `ApiCommand` osztályokban meghatározott műveleteket.
- Az osztályok felelősek a végrehajtás eredményéről szóló tájékoztatás biztosításáért a felhasználók számára.

