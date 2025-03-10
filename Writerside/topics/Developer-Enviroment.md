# Fejlesztői környezet

## Fejlesztőkörnyezet Hardvere

Ez a dokumentáció a fejlesztőkörnyezet hardveres követelményeit tartalmazza.

- **Processzor:** Intel Core i7-9700K
- **Memória:** 32 GB DDR4 RAM
- **Tároló:** 1 TB NVMe SSD
- **Grafikus kártya:** NVIDIA GeForce RTX 2070
- **Operációs rendszer:** Windows 10 Pro

Ez a konfiguráció biztosítja a szükséges teljesítményt a fejlesztési feladatokhoz, beleértve a nagyobb projektek kezelését és a virtuális gépek futtatását is.

## Fejlesztés során használt szoftverek

### Fejlesztői környezet
- **Visual Studio 2022:** A projekt fő fejlesztői környezete, amely támogatja a .NET 8.0 alkalmazások fejlesztését.
- **Visual Studio Code:** Könnyűsúlyú kódszerkesztő, amelyet kiegészítő eszközként használnak.

### Programozási nyelv
- **C#:** A projekt fő programozási nyelve. A C# nyelvet választottuk, mert erőteljes és sokoldalú, jól támogatott a .NET ökoszisztémában, és kiváló eszközöket biztosít a modern webes és asztali alkalmazások fejlesztéséhez. Emellett a C# nyelv szintaxisa tiszta és könnyen olvasható, ami elősegíti a kód karbantarthatóságát és a csapatmunkát.

### Verziókezelés
- **Git:** Verziókezelő rendszer, amely lehetővé teszi a kód változásainak nyomon követését és kezelését.
- **GitHub:** Távoli tárhely a kód tárolására és csapatmunkára.

### Adatbázis
- **Microsoft SQL Server:** Az alkalmazás fő adatbázis-kezelő rendszere.
- **SQLite:** Könnyűsúlyú adatbázis-kezelő rendszer, amelyet teszteléshez és fejlesztéshez használnak.

### Futtatási környezet
- **.NET 8.0:** A projekt keretrendszere, amely biztosítja az alkalmazás futtatásához szükséges környezetet.

### Egyéb eszközök
- **Swagger:** API dokumentáció és tesztelési eszköz.
- **MediatR:** Mediátor minta implementáció, amelyet a kérések és parancsok kezelésére használnak.
- **AutoMapper:** Objektumok közötti leképezés automatizálására szolgáló eszköz.
- **FluentValidation:** Validációs keretrendszer, amelyet az adatok érvényesítésére használnak.

### Tesztelés
- **xUnit:** Egységteszt keretrendszer.
- **Moq:** Mocking keretrendszer, amelyet a tesztek izolálására használnak.
- **coverlet:** Kódlefedettség mérésére szolgáló eszköz.