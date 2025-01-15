# Kérések feldolgozása a kliens oldalon

Ebben a részben a kliens oldal az API-val való kommunikációjára fogunk kitérni

![](../../assets/Client side api command handling.svg)

## Web Interface
- Minden felhasználói interakció a programmal itt kell lefolytatni

## Client Command
- A UI-ből történő kéréseket továbbítja a megfelő helyre a pipeline-on belül
- A szükséges adatok és utasítások tárolandók bennük

## Client Behaviour
- Client Command futtatásának logikája
- Funkciója:
  - Bemeneti adatok ellenőrzése
  - UI frissítése
  - Hibakezelés, felhasználó tájékoztatása

## Client Command Handler
- Client Command utasításának elvégzését biztosítja
- Az API kommunikációnak az eredményét továbbítja a felhasználónak

## API Command
- API utasítások paramétereit tartalmazza, pl: endpoint, HTTP metódus, beérkezett adatok


## API Command Handler
- API Command lefutását biztosítják

## Generic API Command Handler
- API Commandban meghatározott paraméterek alapján továbbítja az utasítást a Backendnek
- Backend válaszát visszaadja Client Command Handler-nek