# EditUserDialog Komponens

## Áttekintés

Az `EditUserDialog` komponens a `vetcms` alkalmazás része, pontosabban a `BrowserPresentation` rétegben helyezkedik el. Ez a komponens egy párbeszédablak felületet biztosít a felhasználói adatok szerkesztéséhez.

## Használat

Az `EditUserDialog` komponens a `UserManagement` oldalon belül kerül alkalmazásra, ahol lehetőség van meglévő felhasználók adatainak szerkesztésére. A komponens akkor kerül meghívásra, amikor egy felhasználó a szerkesztést választja.

## Főbb Funkciók

- **Felhasználói adatok szerkesztése**: Lehetőséget biztosít olyan adatok módosítására, mint név, e-mail, telefonszám és cím.
- **Engedély-alapú műveletek**: Biztosítja, hogy csak az arra jogosult felhasználók módosíthassák az adatokat.
- **Jelszókezelés**: Lehetőséget ad a jelszó megváltoztatására, ha a felhasználó rendelkezik a szükséges engedélyekkel.

## Kód Integráció

### A Szerkesztési Ablak Megnyitása

A párbeszédablakot az `OpenEditUserDialog` módszerrel lehet megnyitni, amely betölti a felhasználó adatait és megjeleníti az ablakot.

```c#
private async Task OpenEditUserDialog(UserDto user)
{
    int id = user.Id;
    Console.WriteLine($"Open dialog edit user");

    var simpleUser = await ToastService.ShowIndeterminateProgressToast(LoadEditUserData(id), "Kérjük várjon...", "Felhasználó adatainak betöltése");

    var result = await ShowUserDialog("Felhasználó szerkesztése", simpleUser);

    if (result.Data is not null)
    {
        UserDto? editResult = result.Data as UserDto;
        Console.WriteLine($"Edit user Dialog closed by {editResult?.VisibleName} - Canceled: {result.Cancelled}");
        if (!result.Cancelled)
        {
            await ToastService.ShowIndeterminateProgressToast(SubmitEditUserData(editResult), "Kérjük várjon...", "Felhasználó adatainak mentése");
        }
    }
    else
    {
        Console.WriteLine($"Edit user Dialog closed - Canceled: {result.Cancelled}");
    }
}
```

### A Módosított Adatok Mentése

A `SubmitEditUserData` módszer elküldi a módosított adatokat a szerverre frissítés céljából.

```c#
private async Task SubmitEditUserData(UserDto editedUser)
{
    ModifyUserClientCommand command = new ModifyUserClientCommand()
    {
        UserId = editedUser.Id,
        ModifiedUserDto = editedUser
    };

    await Mediator.Send(command);
    await dataGrid.RefreshDataAsync();
}
```

## Függőségek

- **IDialogService**: Párbeszédablakok megjelenítéséhez.
- **IToastService**: Értesítések ("toast") megjelenítéséhez.
- **IMediator**: Parancsok és lekérdezések közvetítéséhez.

## Engedélyek

A komponens ellenőrzi az engedélyeket, mielőtt bizonyos műveleteket engedélyezne, mint például a jogosultságok módosítása.

## Összegzés

Az `EditUserDialog` komponens a `vetcms` alkalmazás kulcsfontosságú része, amely felhasználóbarát felületet biztosít a felhasználói adatok szerkesztésére, miközben garantálja, hogy csak jogosult felhasználók hajthassanak végre módosításokat.

