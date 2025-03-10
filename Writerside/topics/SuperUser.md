# Szuper Felhasználó Funkcionalitás

## Áttekintés

Az alkalmazás Szuper Felhasználó funkciója lehetővé teszi egy speciális, magasabb jogosultságokkal rendelkező felhasználó létrehozását és kezelését. Ez a felhasználó lehetővé teszi az üzemeltetésért felelős illetékes számára, hogy az első indításkor konfiguálja az alkalmazást.

> Fontos, hogy a Super User csak az első indításhoz készült. Élő üzemeltetéshez nem megfelelő biztonságú, kérem amint végzett az alkalmazás konfigurálásával, deaktiválja a funkcionalitást, és törölje az adatbázisból, vagy módosítsa a jelszavát.

## Konfiguráció

A Szuper Felhasználó az `appsettings.json` fájlon keresztül van konfigurálva. A releváns rész a következő:

```json
"SuperUser": {
  "Enabled": true,
  "Email": "su@su.su",
  "Password": "32154"
}
```

- `Enabled`: Egy logikai érték, amely jelzi, hogy a Szuper Felhasználó funkció engedélyezve van-e.
- `Email`: A Szuper Felhasználó email címe.
- `Password`: A Szuper Felhasználó jelszava.

## Inicializálás

A Szuper Felhasználó az alkalmazás indításakor inicializálódik. Ezt a `SuperUserInitializer` osztály kezeli. Ha a Szuper Felhasználó nem létezik az adatbázisban, akkor a megadott email címmel és jelszóval létrehozza. Ha a Szuper Felhasználó már létezik, akkor frissíti a jogosultságait.

## Jogosultságok

A Szuper Felhasználó alapértelmezés szerint minden elérhető jogosultságot megkap. Ez magában foglalja, de nem korlátozódik a következőkre:

- CAN_LOGIN
- CAN_ADD_NEW_USERS
- CAN_DELETE_USERS
- CAN_MODIFY_OTHER_USER
- CAN_ASSIGN_PERMISSIONS

## Használat

A Szuper Felhasználó funkció használatához:

1. Győződjön meg arról, hogy a Szuper Felhasználó engedélyezve van a konfigurációs fájlban.
2. Indítsa el az alkalmazást. A Szuper Felhasználó automatikusan létrejön vagy frissül.
3. Jelentkezzen be a Szuper Felhasználó email címével és jelszavával.

A Szuper Felhasználó ezután bármilyen adminisztratív feladatot elvégezhet az alkalmazásban.

## Biztonság

Fontos, hogy a Szuper Felhasználó hitelesítő adatait biztonságban tartsa. Ne ossza meg a Szuper Felhasználó email címét és jelszavát illetéktelen személyekkel. Rendszeresen frissítse a Szuper Felhasználó jelszavát a biztonság fenntartása érdekében.

> Fontos, hogy a Super User csak az első indításhoz készült. Élő üzemeltetéshez nem megfelelő biztonságú, kérem amint végzett az alkalmazás konfigurálásával, deaktiválja a funkcionalitást, és törölje az adatbázisból, vagy módosítsa a jelszavát.