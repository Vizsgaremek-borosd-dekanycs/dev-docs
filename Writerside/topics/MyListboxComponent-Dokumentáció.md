# MyListbox Komponens

## Áttekintés
A `MyListboxComponent` egy testreszabható listadoboz komponens, amely webalkalmazásokhoz készült. Felhasználóbarát felületet biztosít az elemek listájának megjelenítésére és kiválasztására.

## Funkciók
- **Testreszabható megjelenés**: Könnyedén stílusossá tehető az alkalmazás dizájnjához igazítva.
- **Billentyűzetes navigáció**: Támogatja a billentyűzettel történő interakciókat az akadálymentesség érdekében.
- **Dinamikus adatkapcsolás**: Az elemek dinamikus adatkapcsolással tölthetők be.
- **Többszörös kijelölés**: Lehetőség van több elem egyidejű kijelölésére.
- **Eseménykezelés**: Egyedi események az elemek kiválasztásához és a kijelölés megszüntetéséhez.

## Használat
Az alábbi példa bemutatja a `MyListboxComponent` alapvető használatát egy projektben:

```javascript
import MyListboxComponent from 'my-listbox-component';

const elemek = [
    { id: 1, label: 'Elem 1' },
    { id: 2, label: 'Elem 2' },
    { id: 3, label: 'Elem 3' },
];

function App() {
    return (
        <MyListboxComponent items={elemek} />
    );
}
```

## Tulajdonságok
- `items` (Tömb): Az elemek tömbje, amelyek a listadobozban megjelennek. Minden elemnek rendelkeznie kell egy `id` és egy `label` attribútummal.
- `multiSelect` (Boolean): Többszörös kijelölés engedélyezése vagy tiltása. Alapértelmezett érték: `false`.
- `onSelect` (Függvény): Callback függvény, amely egy elem kiválasztásakor hívódik meg.
- `onDeselect` (Függvény): Callback függvény, amely egy elem kijelölésének megszüntetésekor hívódik meg.

## Események
- `onSelect`: Akkor aktiválódik, amikor egy elem kiválasztásra kerül.
- `onDeselect`: Akkor aktiválódik, amikor egy elem kijelölése megszűnik.

## Stílusok
A `MyListboxComponent` megjelenése CSS használatával személyre szabható. Példa:

```css
.my-listbox {
    border: 1px solid #ccc;
    padding: 10px;
}

.my-listbox-item {
    padding: 5px;
    cursor: pointer;
}

.my-listbox-item.selected {
    background-color: #007bff;
    color: white;
}
```