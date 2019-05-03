# Ada 2. beadandó (alapötlet)

Egyetemi előadásokat szimulálunk. Egy előadás hossza 15 másodperc lesz. Az előadás elején a tanár belép a katalógusba, generál egy captchát, mely 5 másodpercig érvényes.
A hallgatók addig próbálnak belépni, ameddig nem sikerült, vagy le nem járt a captcha. A hallgató nem tudja, hogy mennyi idő áll rendelkezésére. Hozzunk létre öt hallgatót, akik jelen vannak az előadáson, ezért 80% eséllyel adnak meg jó captchát (mert azért úgy is el lehet rontani), illetve öt hallgatót, akik nincsenek az előadáson, ezért 10% eséllyel adnak meg jó captchát.

Sikertelen belépés esetén (azaz ha már nincs nyitva a katalógus) azonnal távoznak. Sikeres belépés esetén, attól függően, hogy mennyire találják érdekesnek az előadást, valamennyi idő után távoznak (véletlen szám 0-15 mp között), de legkésőbb az óra vége előtt távoznak. 

Az előadónak nem tetszik, hogy a hallgatók csak a katalógusba iratkoznak fel, és elmennek, így mindig, amikor valaki távozik a teremből, szúrópróba szerűen felolvas egy nevet a katalógusból. Ha az adott hallgató nincs ott, a tanár dühösen kihúzza őt a katalógusból.

Az alábbiak szerint valósítsd meg a feladatot.

## Használandó típusok
* A feladat során számos kifejezést érdemes típussal elkódolni az átláthatóság kedvéért. Amiket általánosan használni kell:
* `Captcha`: nagybetűs karakterek hat hosszú listája.
* `Location`: felsorolásos típus, elemei: `Home`, `Lecture`.
* `Attendance_Sheet`: név-hely rekordokat tartalmazó tömbtípus, hossza a diákok száma, 


## Nyomtató védett egység
* Az eseményeket mindig írjuk ki a képernyőre, hogy nyomonkövethető legyen a szimuláció működése.
* A kiírás legyen szálbiztos (ezt biztosítja a védett egység). 

## Captcha generáló védett egység
* Három alprogrammal rendelkezik, ezek az alábbiak:
* `Create`: Új, random captchát generál, ami egy hat hosszú, nagybetűkből álló karakterlánc.
* `Get`: Segítségével lehessen lekérni az így generált captchát.
* `Get_Wrong`: Segítségével lehessen lekérni egy konstans, (általában) hibás captchát.

## Katalógus taszk
* Két belépési ponttal rendelkezik: `Start` és `Join`
* Kezdetben várakozik arra, hogy a tanár elindítsa a `Start` belépési pontját keresztül. A belépési pont paramétere a katalógus futásának időtartama lesz. Ha letelik az idő, azonnal termináljon.
* Ezután folyamatosan várja a hallgatók csatlakozását a `Join` belépési ponton keresztül.
* Ha a hallgató csatlakozik, ellenőrzi a kapott captchát, helyességét egy kimenő paraméterben jelzi a hallgatónak. Csatlakozáskor a hallgató nevére és tartózkodási helyére is szükség van.
* Ha helyes a captcha, hozzáadja a hallgató nevét, helyét egy listához (lásd Tanterem).

## Tanterem védett egység
* A katalógusra feliratkozott hallgatók listáját tartalmazza.
* Négy alprogrammal rendelkezik, ezek a következők:
   * `Add`: Hallgató hozzáadása (helyével együtt)
   * `Leave`: Hallgató távozása, helyét ezzel módosítsuk `Home`-ra. Ha a hallgató a teremben volt, akkor távozásával felkelti a tanárt (becsapja az ajtót), jelzi neki, hogy egy diák elhagyta a termet.
   * `Check_Random`: A tanár hívja meg, ha egy diák elhagyja a termet. Véletlenszerűen kiválaszt egy diákot a katalógusból, ellenőrzi, hogy jelen van-e, ha nincs, törli a katalógusból.
   * `Print_Final`: A katalógusban szereplő nevek kiírása.

## Aszinkron felkeltés
* A tanár felkeltését hajtsuk végre aszinkron módon: ne a tanterem keltse fel a tanárt, hanem egy dinamikusan létrehozott `Door_Slam` ágens taszk. 
* A `Door_Slam` taszknak csupán annyi lesz a feladata, hogy meghívja a tanár belépési pontját.

## Hallgató taszk típus
* Diszkriminánsa egy név és hogy otthon, vagy az előadáson tartózkodik.
* Dinamikusan érkeznek meg az előadásra, vagy lépnek be a rendszerbe. (Főprogramban dinamikus példányosítással.)
* Egy másodpercenként, folyamatosan próbálja a belépést. Ehhez a fent leírt valószínűségekkel jó vagy rossz captchát használ. Ezt a 
  captcha generátortól kérje el.
* Ha sikerül: Vár véletlenszerűen 0-15 másodpercet, de nem többet, mint az óra időtartama, majd távozik (a terem belépési pontján keresztül).
* Ha lejárt a katalógus, kiírja a sikertelen próbálkozást, majd távozik (ilyenkor nem kell meghívni a terem belépési pontját).
* FONTOS: a hallgató nem tudja, hogy meddig van nyitva a katalógus. Próbálkozzon folyamatosan, és vegye észre, ha lejárt.

## Tanár taszk
* Óra elején elindítja a katalógust 5 másodperces időtartammal.
* Ezt követően folyamatosan hallgatja `Student_Left` belépési pontján keresztül, hogy egy hallgató távozott-e, egészen addig, amíg az óra véget nem ér. Ha az óra véget ér (lejár a 15 másodperc), terminál.
* Amikor egy hallgató távozik, a tanterem `Check_Random` eljárásának segítségével kiválaszt egy random nevet a katalógusból, és megkérdezi (a teremben mentett állapotát megnézi) az adott hallgatót, hogy jelen van-e. Ha a hallgató nincs jelen, kihúzza a listából.
* Az óra végén sorolja fel a végleges katalógus tartalmát a tanterem `Print_Final` eljárásának segítségével.

Csak és kizárólag az alábbi fájlokat kell beadni, tömörítve:
* `main.adb`
* `main-captcha_generator.adb`
* `main-catalog.adb`
* `main-classroom.adb`
* `main-printer.adb`
* `main-safe_random.adb`
* `main-student.adb`
* `main-teacher.adb`
* `main-captcha_generator.adb`

Segítség a fájlokba kihelyezéshez [itt található](http://luksan.web.elte.hu/ada/1/index.html#separate).
