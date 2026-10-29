# A türelmetlen gyógyszerkereső – hibariport

Repó: https://github.com/And-8-And/umfst-bh08-gyogyszerkereso
Készítette: <név>, <csoport>

## 1. hiba
- **Hiba leírása és reprodukálása (pontos gépelési sorrend, időzítés):** Minden egyes leütés külön szerverkérést indít. Reprodukálás: folyamatos gépelés közben (pl. "Aszpirin") a böngésző konzoljában vagy a Performance panelen látszik, hogy minden egyes betű lenyomásakor új hálózati hívás indul.
- **Gyökérok (eseményhurok, Promise-ok időbeli sorrendje):** Az `input` eseményfigyelőben nincs semmilyen késleltetés (debounce). A böngésző azonnal és szinkron módon reagál minden DOM eseményre, ami indokolatlanul túlterheli a hálózatot.
- **Javítás (commit link):** https://github.com/And-8-And/umfst-bh08-gyogyszerkereso/commit/6c9f07c76c9ff4eaf3f4c03cbb550e68838e3e4f (Egy `keresesTimeout` változó bevezetésével és `setTimeout` / `clearTimeout` használatával 300 ms-os Debounce logikát építettünk az eseményfigyelőbe).
- **Hogyan tesztelted (konzolnapló a kérésekről, fake timer-es egységteszt, Performance panel):** A gépelés befejezése után vártam picit, és a konzolba (valamint a Performance panelre) beépített logok alapján már csak 1 darab lekérdezés indult el a teljes szó begépelése után, nem karakterenként.
- **Tanulság / érintett alapelv (A legutolsó kérés nyer; Debounce; Mindig kezeld a Promise elutasítását):** Debounce (késleltetés) elve. Gyakori események (input, scroll) esetén mindig késleltetni és összevonni kell a hálózati kéréseket.

## 2. hiba
- **Hiba leírása és reprodukálása (pontos gépelési sorrend, időzítés):** Gyorsan begépelve az „Algo” szót, a találati listában a végén az „A” betűre illeszkedő 5 találat marad, nem a 2 helyes ("Algo").
- **Gyökérok (eseményhurok, Promise-ok időbeli sorrendje):** Versenyhelyzet (Race Condition). Az "A" kérés lassan tér vissza, az "Algo" pedig gyorsan. Bár az "Algo" kérés indult el később, hamarabb érkezik meg rá a válasz. Később az "A" kérés válasza is befut az eseményhurokba, a Promise `.then()` ága pedig felülírja a helyes, újabb találatokat a régi elavultakkal.
- **Javítás (commit link):** https://github.com/And-8-And/umfst-bh08-gyogyszerkereso/commit/1a08f47c57b42eb024df5b871db104972e5f819b (Bevezettünk egy `utolsoKeres` számlálót. A kérés indításakor elmentjük az azonosítót, a `.then()` blokkban pedig leellenőrizzük, hogy egyezik-e, és csak akkor frissítjük a DOM-ot, ha a kérés még nem évült el).
- **Hogyan tesztelted (konzolnapló a kérésekről, fake timer-es egységteszt, Performance panel):** Gyorsan begépeltem az "Algo" szót. A UI-on megjelent a 2 helyes találat, és nem írta felül a később megérkező, "A"-hoz tartozó hibás válasz.
- **Tanulság / érintett alapelv (A legutolsó kérés nyer; Debounce; Mindig kezeld a Promise elutasítását):** A "legutolsó kérés nyer" (Latest request wins). Aszinkron kéréseknél nem garantált, hogy a kérések indításának sorrendje megegyezik a beérkezésük sorrendjével.

## 3. hiba
- **Hiba leírása és reprodukálása (pontos gépelési sorrend, időzítés):** Az „x” beírására a felület örökre „Keresés…” állapotban marad, a konzolban „Uncaught (in promise) Error” jelenik meg.
- **Gyökérok (eseményhurok, Promise-ok időbeli sorrendje):** A fetch-et egy hagyományos `try...catch` blokk veszi körbe. Ez csak a szinkron kód futását figyeli. A `fakeFetch` aszinkron módon visszatér a Promise-szal (a `try` sikeresen lefut), de a hiba később keletkezik a Promise elutasításakor, amit a szinkron `catch` már nem tud elkapni.
- **Javítás (commit link):** https://github.com/And-8-And/umfst-bh08-gyogyszerkereso/commit/0686751333711c09b945ee45b317344aef0b1284 (A `try...catch` blokkot eltávolítottuk, és a Promise lánc végére egy `.catch(e => ...)` metódust fűztünk az aszinkron hibakezeléshez).
- **Hogyan tesztelted (konzolnapló a kérésekről, fake timer-es egységteszt, Performance panel):** Begépeltem az "x" karaktert. A felületen a "Keresés..." szöveg átváltott a "Hiba: Szerverhiba (503)" üzenetre, a konzolból pedig eltűnt az Uncaught Error.
- **Tanulság / érintett alapelv (A legutolsó kérés nyer; Debounce; Mindig kezeld a Promise elutasítását):** Mindig kezeld a Promise elutasítását. Aszinkron kód hibáit csak `.catch()`-el vagy `async/await` környezetben használt `try/catch`-el lehet elkapni.

## 4. hiba
- **Hiba leírása és reprodukálása (pontos gépelési sorrend, időzítés):** A „Találatok listaáron összesen” eredménye mindig 0 lej, bármennyi elem is van a listában.
- **Gyökérok (eseményhurok, Promise-ok időbeli sorrendje):** Az `osszesit` függvényben a `forEach` ciklus szinkron módon iterál. Elindítja a Promise-okat, de nem várja meg azok (`fakePrice`) feloldását. A kód azonnal továbblép a DOM frissítésre, és kiírja a kezdőértéket (0). A háttérben később befejeződő aszinkron műveletek a semmibe frissítik a változót.
- **Javítás (commit link):** https://github.com/And-8-And/umfst-bh08-gyogyszerkereso/commit/6fe825349f765cbe8492e9357da239964af557c3 (A `nevek.forEach(...)` ciklust kicseréltük egy `for (const n of nevek)` ciklusra, ami már képes megvárni az iteráción belüli `await` kulcsszóra érkező válaszokat).
- **Hogyan tesztelted (konzolnapló a kérésekről, fake timer-es egységteszt, Performance panel):** Rákerestem az "Algo" szóra. Pár száz milliszekundum késleltetés után az összeg mező a helyes (0-tól eltérő) értékre frissült.
- **Tanulság / érintett alapelv (A legutolsó kérés nyer; Debounce; Mindig kezeld a Promise elutasítását):**  A tömbmetódusok (`forEach`, `map`) visszahívásai nem várják meg az aszinkron kódokat. Ha sorrendiséget és bevárást akarunk, `for...of` ciklust vagy `Promise.all()`-t kell alkalmazni.

## 5. hiba
- **Hiba leírása és reprodukálása (pontos gépelési sorrend, időzítés):** Az „Automatikus frissítés” gombot többször megnyomva egyre sűrűbben villog a lista, a CPU-terhelés nő, és az ismétlődés leállíthatatlan.
- **Gyökérok (eseményhurok, Promise-ok időbeli sorrendje):** Minden gombnyomás egy teljesen új `setInterval` folyamatot indít el. Mivel ezek azonosítóit nem mentjük el, nem is töröljük őket, így az időzítők egymásra halmozódnak a memóriában (memóriaszivárgás) és mindegyik párhuzamosan indítja a hálózati kéréseket.
- **Javítás (commit link):** https://github.com/And-8-And/umfst-bh08-gyogyszerkereso/commit/1b4ed0a20a3b287190175a032dacab501f2f8b4d (Egy globális `frissitInterval` változót vezettünk be. Kattintáskor a `clearInterval` segítségével először mindig megállítjuk az előző folyamatot, és csak utána indítunk újat).
- **Hogyan tesztelted (konzolnapló a kérésekről, fake timer-es egységteszt, Performance panel):** Gyorsan megnyomtam 10-szer a frissítés gombot. A hálózati forgalomban továbbra is csak pontosan 2 másodpercenként indult 1 darab kérés, nem volt halmozódás vagy villogás.
- **Tanulság / érintett alapelv (A legutolsó kérés nyer; Debounce; Mindig kezeld a Promise elutasítását):** Erőforrás-menedzsment (Memory leak elkerülése). Időzítőket és eseményfigyelőket új példányosítás előtt mindig le kell állítani vagy takarítani kell.

## Összegzés
- Mit tanultam: Az aszinkron JavaScript legkomplexebb problémáit: az események megfelelő késleltetését, a versenyhelyzetek (race conditions) elkerülését, az aszinkron hibakezelést és a memóriaszivárgások kivédését.
- Melyik alapelv sérült a leggyakrabban: Az eseményhurok (Event Loop) működésének meg nem értése: a kód hibásan szinkron végrehajtást feltételezett olyan helyeken, ahol az időzítés aszinkron és párhuzamos volt.