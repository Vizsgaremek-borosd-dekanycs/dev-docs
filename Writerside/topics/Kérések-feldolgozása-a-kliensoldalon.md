# Kliensoldali Kérések Feldolgozása

#### **Bevezetés**

Ez a dokumentum átfogó bemutatást nyújt a kliensoldali kérések feldolgozásáról, részletezve a rendszer különböző rétegeinek működését és szerepét. A cél az, hogy érthetően és átláthatóan szemléltesse, hogyan történik a kliensoldali komponensek és a háttérrendszer közötti kommunikáció, miközben biztosított az adatfeldolgozás hatékonysága és megbízhatósága.

![](../../assets/Client%20side%20api%20command%20handling.png)


#### **Webes Felület**

A webes felület a rendszer felhasználói interakcióinak központi helyszíne, amely lehetővé teszi a rendszer különböző funkcióihoz való hozzáférést. Bár a webes felület csupán korlátozott betekintést biztosít a rendszer architektúrájába, kulcsfontosságú szerepe van a felhasználói műveletek továbbításában a "Client Command" komponensek felé.

**Kapcsolódó fájl:**  
`Pages\Features\IAM\LoginPage.razor`

**Példa kódrészlet egy bejelentkezési műveletről:**

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

#### **Hibakezelési Mechanizmus**

A hibakezelési mechanizmus célja, hogy mind a kliens-, mind a szerveroldalon előforduló problémák esetén a felhasználó világos és egyértelmű visszajelzést kapjon. A hibák kezelésére különböző szinteken történik a feldolgozás, amelyek az alábbiak szerint oszlanak meg:

- **ClientCommandHandler:** A felhasználói műveletek során keletkező hibák kezeléséért felelős.
- **ApiCommand:** A szerveroldali hibák begyűjtése és továbbítása a kliensoldalra a megfelelő válasz formájában.
- **GenericApiCommandHandler:** Az API-val való kommunikáció során felmerülő hibák kezelésére szolgál, különös tekintettel a hibakódok feldolgozására és értelmezésére.
- **UnhandledExceptionBehaviour:** Ez a réteg felel az ismeretlen vagy nem kezelt hibák feldolgozásáért, minimalizálva ezzel a rendszer összeomlásának kockázatát.

Mindezek működését támogatja a `DialogService`, amely a felhasználói értesítések megjelenítését végzi. Ez a modul a hibakezelési mechanizmus kulcsfontosságú eleme, mivel segít a pontos és részletes visszajelzések biztosításában.


#### **Client Command**

A „Client Command” osztályok felelősek a webes felületről érkező adatok feldolgozásának első lépéseiért. Ezek az osztályok előkészítik az adatokat, majd a „Client Command Bus” segítségével továbbítják azokat a háttérrendszer felé. Minden „Client Command” implementálja az `IClientCommand<bool>` interfészt, amely biztosítja a feldolgozási folyamat egységességét és megbízhatóságát.


#### **Client Behaviour**

A „Client Behaviour” osztályok a feldolgozási pipeline első szakaszában kapnak szerepet. Ezek a modulok végzik az elsődleges adatellenőrzéseket, és lehetőséget biztosítanak a kliensoldali kérések elfogására, módosítására, vagy szükség esetén azok elutasítására.

A legfontosabb viselkedési osztályok a következők:
- **UnhandledExceptionBehaviour:** Ez az osztály felelős az ismeretlen vagy váratlan hibák kezeléséért. Elsődlegesen az `IClientCommand<T>` implementációkat figyeli.
- **ValidationBehaviour:** Ez az osztály az `ApiCommandBase<T>` objektumokat validálja, és a nem megfelelő formátumú bemeneteket hibaüzenettel visszautasítja.

Ezek az osztályok biztosítják, hogy a pipeline szigorúan betartsa az adatfeldolgozás sorrendjét, és csak validált adatok kerüljenek a következő feldolgozási szakaszba.


#### **Client Command Handler**

A kliensoldali kéréseket a `ClientCommandHandler` kezeli, amely a MediatR keretrendszer segítségével továbbítja a kéréseket. Ez az osztály ellenőrzi az adatokat, és elvégzi azokat a műveleteket, amelyek nem érintik közvetlenül a háttérrendszert. Emellett feldolgozza az API-tól érkező válaszokat, és gondoskodik azok megfelelő kezeléséről a webes felület számára.


#### **API Command**

Az API-kommunikáció alapját az API Command osztályok jelentik. Ezek az osztályok tartalmazzák az API végpontjainak eléréséhez szükséges paramétereket, mint például a kérés célját (endpoint), a HTTP-módszert, valamint az átadni kívánt adatokat.

**Példa egy API végpontra vonatkozó implementációra:**

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


#### **API Command Handler**

Az API Command Handler-ek a `GenericApiCommandHandler` osztályból származtatják működésüket. Ezek az osztályok végzik az egyes API-végpontokhoz kapcsolódó egyedi implementációk kialakítását és kezelését. Az API kommunikáció során felmerülő műveleteket a `ClientCommandHandler` segítségével hívják meg.


#### **Generic API Command Handler**

A „Generic API Command Handler” osztály a rendszer központi eleme az API-val történő kommunikáció során. Ez az osztály végrehajtja az összes „ApiCommand” által definiált HTTP-műveletet, majd a kapott válaszokat feldolgozza. Szerepe elengedhetetlen az adatok pontos továbbítása és a felhasználók értesítése érdekében.

