# PermissionWrapper Komponens

A `PermissionWrapper` egy olyan komponens, amely feltételes tartalom megjelenítésére szolgál a felhasználó jogosultságai alapján. A komponens ellenőrzi, hogy a felhasználó rendelkezik-e a szükséges jogosultságokkal, és csak akkor rendereli a gyermek tartalmat, ha ezek teljesülnek.

## Paraméterek

- **`RequiredPermissions` (`PermissionFlags[]`)**: Egy tömb, amely meghatározza azokat a jogosultságokat, amelyek szükségesek a gyermek tartalom megjelenítéséhez.
- **`ChildContent` (`RenderFragment`)**: Az a tartalom, amely csak akkor kerül megjelenítésre, ha a felhasználó rendelkezik a szükséges jogosultságokkal.

## Használat

A `PermissionWrapper` használatához meg kell adni a szükséges jogosultságokat, valamint azt a tartalmat, amely megjelenik, ha ezek teljesülnek.

```razor
@page "/example"
@using vetcms.ClientApplication.Common.IAM.Commands.CheckPermissionQuery
@inject MediatR.IMediator Mediator

<PermissionWrapper RequiredPermissions="new PermissionFlags[] { PermissionFlags.CAN_ADD_NEW_USERS }">
    <div>
        Ez a tartalom csak azoknak a felhasználóknak látható, akik rendelkeznek a CAN_ADD_NEW_USERS jogosultsággal.
    </div>
</PermissionWrapper>
```

A fenti példában a `PermissionWrapper` belsejében található tartalom csak akkor kerül megjelenítésre, ha a felhasználónak van `CAN_ADD_NEW_USERS` jogosultsága.

## Megvalósítási részletek

A `PermissionWrapper` komponens a `CheckPermissionQuery` parancsot használja a felhasználói jogosultságok ellenőrzésére. A parancsot elküldi a mediatornak, majd az eredmény alapján beállítja a `QueryResult` tulajdonságot. Ha a `QueryResult` értéke `true`, a gyermek tartalom renderelésre kerül.

