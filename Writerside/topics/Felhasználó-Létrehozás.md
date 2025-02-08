# Felhasználó Létrehozása

## **Áttekintés**
A `CreateUser` funkció a `vetcms` rendszerben egy strukturált és biztonságos folyamatot követ az új felhasználók létrehozásához. Ez a dokumentáció részletesen bemutatja a folyamat lépéseit, az API végpontok használatát, valamint a validációs és tesztelési eljárásokat.

---

## **Folyamatleírás**

### **API Kérés**
- A folyamat egy `POST` kérés küldésével indul a következő végpontra:
  - **URL:** `/api/v1/iam/user`
  - **Törzs:** `CreateUserApiCommand` objektum az új felhasználó adataival.

### **Validáció**
- A `CreateUserApiCommandValidator` ellenőrzi a bemeneti adatokat:
  - Minden szükséges mező jelenléte és megfelelő formázása biztosított.

### **Parancs Kezelése**
- A `CreateUserCommandHandler` végzi a kérelem feldolgozását:
  - Ellenőrzi, hogy létezik-e már a felhasználó az `IUserRepository` segítségével.
  - Ha az e-mail cím már használatban van, hibaüzenetet ad vissza.

### **Felhasználó Létrehozása**
- Ha az e-mail cím nem foglalt, új `User` entitás jön létre.
- Az új felhasználó alapértelmezett jogosultságokkal rendelkezik, kivéve a `CAN_LOGIN` jogot, amelyet később aktiválhat.

### **Adatbázis Műveletek**
- Az `IUserRepository` segítségével a rendszer elmenti az új entitást az adatbázisba.

### **Első Bejelentkezés és Hitelesítés**
- **Első hitelesítési kód generálása:** A `FirstTimeAuthenticationCode` létrejön.
- **Tárolás:** A kódot az `IFirstTimeAuthenticationCodeRepository` menti.

### **E-mail Értesítés**
- Az `IMailService` egy e-mailt küld az új felhasználónak az első bejelentkezési kóddal.
- Az e-mail tartalmaz egy linket a kezdeti jelszó beállításához.

### **E-mail Link Aktiválása**
- A felhasználó az e-mailben kapott link segítségével beállítja jelszavát és aktiválja a fiókját.

### **Rendszerválasz**
- A rendszer visszaküld egy `CreateUserApiCommandResponse` választ:
  - **Siker esetén:** Az objektum `Success = true` értéket tartalmaz és megerősíti a felhasználó létrehozását.
  - **Hiba esetén:** Az objektum `Success = false` értéket tartalmaz egy hibaüzenettel.

---

## **Jogosultságkezelés**

### **Szükséges Jogosultságok**
- **Jogosultság neve:** `CAN_CREATE_USER`
- **Leírás:** Új felhasználók létrehozásának engedélyezése.
- **Szerepkörök:** Adminisztratív és felhasználókezelési jogokkal rendelkező szerepkörök.

---

## **API Végpontok**

### **POST /api/v1/iam/user**
**Kérés:**
- **URL:** `/api/v1/iam/user`
- **Módszer:** `POST`
- **Törzs:** `CreateUserApiCommand` objektum.

**Válasz:**
- **Siker:** `CreateUserApiCommandResponse` (`Success = true`).
- **Hiba:** `CreateUserApiCommandResponse` (`Success = false`).

---

## **Tesztelés**

### **Egységtesztek**
1. **Valid adatokkal történő felhasználó létrehozás**
  - Input: Helyes adatokkal kitöltött `CreateUserApiCommand`
  - Elvárt eredmény: `Success = true`, felhasználó létrejön.

2. **Hiányzó mezők kezelése**
  - Input: Hiányzó e-mail cím vagy jelszó
  - Elvárt eredmény: `Success = false`, megfelelő hibaüzenet visszatérése.

### **Integrációs Tesztek**
1. **Felhasználó létrehozása és adatbázis ellenőrzés**
  - Input: Érvényes `CreateUserApiCommand`
  - Elvárt eredmény: Felhasználó megjelenik az `IUserRepository` adatbázisában.

2. **Értesítő e-mail sikeres elküldése**
  - Input: Érvényes `CreateUserApiCommand`
  - Elvárt eredmény: Az `IMailService` sikeresen elküldi az aktiváló e-mailt.

### **Végponttól Végpontig (E2E) Tesztek**
1. **Teljes létrehozási folyamat tesztelése**
  - Input: Új felhasználó regisztrációja API hívással
  - Elvárt eredmény: Felhasználó létrejön, e-mailt kap, és sikeresen beállíthatja jelszavát.

---

## **Összegzés**
A `vetcms` rendszer `CreateUser` funkciója egy átfogó és biztonságos megoldást kínál az új felhasználók létrehozására. A folyamat biztosítja az adatok validálását, a jogosultságkezelést és az értesítési mechanizmusokat, hogy az új felhasználók zökkenőmentesen és biztonságosan csatlakozhassanak a rendszerhez.

---

