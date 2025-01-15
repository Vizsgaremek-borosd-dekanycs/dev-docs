# Autentikáció és Autorizáció


## Bevezetés

Az autentikáció és az autorizáció kulcsfontosságú elemei a biztonságos informatikai rendszerek működésének. Az autentikáció során igazoljuk a felhasználó vagy egy rendszer azonosságát, míg az autorizáció meghatározza, hogy az adott entitás milyen műveletek végrehajtására jogosult. Ezen folyamatok integrálása képes biztosítani a rendszerek integritását, adataink védelmét és a műveletek megfelelő szintű szabályozását.

Ez a dokumentum bemutatja, hogyan érvényesíthetők az autentikációs és autorizációs elvek egy modern rendszerben. Külön kitérünk az API-parancsok kezelésére, a jogosultságok meghatározására, és azok gyakorlati alkalmazására.

[kép: https://docs.google.com/drawings/d/1bJSLzSSpIYbzLFG3vfwcmRSNhXIdJg1tVHnGF88dNWo/edit]

---

## Authenticated API Command

Minden API-parancsot regisztrálni kell a `SharedModels` modulban, hogy az mind a kliens-, mind a szerveroldali projektből elérhető legyen. Ez a regisztráció biztosítja, hogy az API-parancsok a megfelelő végrehajtási pipeline-ba kerüljenek.

Az autorizálni kívánt API-parancsnak implementálnia kell az `AuthenticatedApiCommandBase<TResponse>` osztályt. Ez garantálja, hogy a parancs a megfelelő hitelesítési és autorizációs folyamatokban vegyen részt.

### GetRequiredPermissions()
Az API-parancs végrehajtásához szükséges jogosultságok meghatározását végzi. Az alábbiakban egy konkrét példa szerepel:

```c#
public record AssignUserPermissionApiCommand : AuthenticatedApiCommandBase<AssignUserPermissionApiCommandResponse>
{
    public int Id { get; init; }
    public string PermissionSet { get; set; }

    public EntityPermissions GetPermissions()
    {
        return new EntityPermissions(PermissionSet);
    }

    public override string GetApiEndpoint()
    {
        return Path.Join(ApiBaseUrl, "/api/v1/iam/assign-permission");
    }

    public override HttpMethodEnum GetApiMethod()
    {
        return HttpMethodEnum.Put;
    }

    public override PermissionFlags[] GetRequiredPermissions()
    {
        return [PermissionFlags.CAN_ASSIGN_PERMISSIONS];
    }
}
```

---

## Input Validation
Az input validáció biztosítja, hogy a parancshoz tartozó adatokat megfelelő szabályok szerint kezeljük. Az ilyen szabályokat ugyanabban a fájlban kell regisztrálni, amelyben az API-parancs található.

```c#
public class AssignUserPermissionApiCommandValidator : AbstractValidator<AssignUserPermissionApiCommand>
{
    public AssignUserPermissionApiCommandValidator()
    {
        RuleFor(x => x.Id).NotEmpty();
    }
}
```

---

## JWT Validation

A JSON Web Token (JWT) validálása elengedhetetlen a hitelesítési folyamatban. A JWT biztosítja, hogy a felhasználók és a rendszerek között zajló kommunikáció megfelelően hitelesített és biztonságos legyen.

[kép: https://docs.google.com/drawings/d/13XkQAiiwz9wBqBG0p9inySpUsFH0HwIbohKoreI985A/edit]

---

## Jogosultság Rendszer
A jogosultságok ellenőrzése biztosítja, hogy a felhasználók csak az általuk jogosult feladatokat hajthassák végre az alkalmazáson belül. Ez elengedhetetlen a rendszer biztonságának és integritásának fenntartásához.

### Általános Működés
A jogosultsági rendszer bináris „FLAG” alapú struktúra szerint működik. Minden FLAG egy jogosultságot jelöl, amelyeket a `PermissionFlags` enum definiál a `SharedModels` modulban.

Az enum elemeihez bináris helyértékek tartoznak. Az alábbi példa bemutatja a működést:

```c#
public enum PermissionFlags
{
    CAN_LOGIN, // 0.
    CAN_ASSIGN_PERMISSIONS, // 1.
    CAN_LIST_USERS, // 2.
    [...]
}
```

| Név           | CAN_LIST_USERS | CAN_ASSIGN_PERMISSIONS | CAN_LOGIN |
|----------------|----------------|------------------------|-----------|
| Helyérték    | 2              | 1                      | 0         |
| Bináris érték | 0              | 1                      | 1         |

Ebben a konfigurációban a felhasználó rendelkezik az engedélyek hozzárendelésének és a bejelentkezés lehetőségével, de nem tudja listázni a felhasználókat.

Az adatbázisban a jogosultságokat szöveges formában tároljuk a felhasználói tábla részeként. Ezeket a jogosultságokat az `EntityPermissions` osztály segítségével kezelhetjük.

---

### Jogosultság Ellenőrzése

A felhasználó jogosultságainak ellenőrzését a `PermissionRequirementBehaviour` biztosítja, amely a parancsok futtatásáért felel.

Az alábbi kódrészlet szemlélteti a jogosultságok lekérésének folyamatát:

```c#
namespace vetcms.ServerApplication.Domain.Entity
{
    public class User : AuditedEntity
    {
        [Key]
        public int Id { get; set; }

        [...]

        public string PermissionSet { get; private set; } = new EntityPermissions().AddFlag(PermissionFlags.CAN_LOGIN).ToString();

        public EntityPermissions GetPermissions()
        {
            return new EntityPermissions(PermissionSet);
        }

        [...]
    }
}
```

---

### Jogosultság Felülírása

A jogosultságok módosításához az `EntityPermissions` megfelelő metódusai állnak rendelkezésre. Az alábbi példa részletezi ezeket:

```c#
namespace vetcms.SharedModels.Common.IAM.Authorization
{
    public class EntityPermissions
    {
        [...]
        public EntityPermissions ClearAllFlags()
        {
            [...]
        }

        public EntityPermissions RemoveFlag(PermissionFlags flag)
        {
            [...]
        }

        public EntityPermissions AddFlag(PermissionFlags flag)
        {
            [...]
        }

        public EntityPermissions AddFlag(params PermissionFlags[] flags)
        {
            [...]
        }

        public EntityPermissions MergePermissions(EntityPermissions other)
        {
            [...]
        }
    }
}
```

A módosított jogosultságokat a felhasználó entitásban kell felülírni a `User.OverwritePermissions` metódussal:

```c#
namespace vetcms.ServerApplication.Domain.Entity
{
    public class User : AuditedEntity
    {
        [...]

        public string PermissionSet { get; private set; } = new EntityPermissions().AddFlag(PermissionFlags.CAN_LOGIN).ToString();

        [...]

        public User OverwritePermissions(EntityPermissions newPermissions)
        {
            PermissionSet = newPermissions.ToString();
            return this;    
        }
    }
}
```

# Bejelentkezés