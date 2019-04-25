# MobWeb jegyzet

## 2. Android alapok, Activity, BackStack
### Az Android platform felépítése
![Android platform felépítése](https://developer.android.com/guide/platform/images/android-stack_2x.png)

- A **piros** színnel jelölt rész a Linux kernel, amely tartalmazza:
	- A hardver által kezelendő eszközök meghajtó programjait
	- Memória kezelés, a folyamatok ütemezése és az alacsony fogyasztást elősegítő teljesítmény-kezelés
- A **türkiz** réteg a hardver abszterkciós szint, mely a hardver vezérlést köti össze a szoftveres réteggel
- A Linux kernel felett futnak a **lila** részben található programkönyvtárak/szolgáltatások, mint például: libc, SSL, SQLite (C/C++)
- Részben ezekre épül a **sárga** egységben található virtuális gép
	- Nem kompatibilis a Sun virtuális gépével
	- Nem .class állományok vannak, hanem .dex (Dalvik Executable) formátumú fájlok.
	- A Java csak mint nyelv jelenik meg
- A **zöld** színnel jelölt részekben már csak Java forrást találunk
>A Java forrást a virtuális gép futtatja, ez adja az Android lényegét:
>- Látható és tapintható operációs rendszert
>- Futó programokat

- A legfelső **kék** részben láthatók az OS-en elérhető alapértelmezett alkalmazások

### Szoftverfejlesztési eszközök Android platformra

- __Android SDK (Software Development Kit):__
	- Fejlesztő eszközök
	- Emulátor kezelő
- __Android NDK (Native Development Kit):__
	- Lehetővé teszi natív kód futtatását
- __Android ADK (Accessory Development Kit):__
	- Támogatás Android kiegészítő eszközök gyárásához (egészségügyi eszközök, időjárás kiegészítő eszközök stb..) 

### A fordítás menete (forrás-> .apk)
![fordítás](https://i.loli.net/2018/11/04/5bddccef4f76f.png)

### Az Android .apk állomány
- Tömörített állomány, mely tipikusan a következő tartalommal rendelkezik:
	- META-INF könyvtár:
		- Cert.RSA: alkalmazás tanúsítvány
		- MANIFEST.MF: meta információk
		- CERT.SF: erőforrások listája és SHA-1 hash értékük
	- Res könyvtár: erőforrásokat tartalmazza
	- AndroidManfest.xml: név, verzió, jogosultság, könyvtárak
	- classes.dex: lefordított osztályok
	- resources.arcs

### Az Android alkalmazások felépítése
- __Komponensek:__
	- Activity
	- Service
	- Content Provider
	- Broadcast Receiver

Minden komponensnek különböző szerepe van az alkalmazáson belül.
Bármelyik komponens önállóan aktiválódhat.
Akár gy másik alkalmazás is aktiválhatja az egyes komponenseket.

Az alkalmazást leíró (_manifest_) állománynak deklarálnia kell a következőket:
- Alkalmazás komponensek listája
- Szükséges min. Android verzió
- Szükséges hardware konfig

A nem forráskód jellegő erőforrásoknak (képek, szövegek, nézetek, stb.) rendelkezésre kell állnia.

#### Activity-k
- Különálló nézet, saját UI-al
- Például:
	- Emlékeztető alkalmazás
	- 3 Activity: ToDo lista, új ToDo felvitele, ToDo részletek
- Független Activity-k, de együtt alkotják az alkalmazást
- Más alkalmazásból is indítható az Acitivity, például:
	- Kamera alkalmazás el tudja indítani az új ToDo felvitele Activity-t és a képet hozzá rendeli az emlékeztetőhöz
- Az `android.app.Activity` osztályból származik le
#### Service-k
- Egy hosszabb ideig háttérben futó feladatot jelképez
- Nincs felhasználói felülete
- Más komponens elindíthatja, vagy csatlakozhat hozzá vezérlés céljából
- Az `andorid.app.Service` osztályból kell öröklődnie
#### Content provider-ek
- Feladata egy megosztott adatforrás kezelése
- Tárolódhat: fájlrendszerben, SQLite db-ben , web-en, vagy egyéb perzisztens adattárban
- Content provider-en keresztül más alkalmazások hozzáférhetnek az adatokhoz.
- Az `android.app.ContentProvider` osztályból származik le
#### Broadcast receiver-ek
- A rendszer szintű eseményekre reagál
- Az alkalmazás is indíthat saját "broadcast"-ot
- Nem rendelkeznek saját felülettel, inkább elindítanak egy másik komponenst
- Az `android.content.BroadcastReceiver` osztályból származik le

#### Manifest állomány
- Alkalmazás leíró
- XML
- Alkalmazás telepítésekor ellenőrződik 
- Tartalma:
	- java package (*egyedi id-ként szolgál*)
	- engedélyek
	- min API szint
	- hw és sw funkciók, amit az app használ
	- külső API könyvtárak

#### Alkalmazás erőforrások
- Pl.: képek, hanganyagok, stb
- XML-ben definiált felületek: elrendezés, animáció, menü, stílus, szín
- res/drawable/logo.png -> R.drawable.logo
- R.java
---
### Acitivity
- Tipikusan egy képernyő
- leginkább ablakként képzelhető el
- lehet: teljes képernyős vagy pop-up
- Az egy appban lévő Activity-k lazán csatoltak
- Általában létezik ey _fő_ Activity, ahonnan a többi elérhető.
- Bármelyik Activity indíthat újabbakat

#### Activity életciklus modell
![lifecycle](https://s3-us-west-1.amazonaws.com/nearsoft-com-media/uploads/2018/01/android-activity-lifecycle-01.png)

Paused, vagy Stopped áll. a rendszer bármikor leállíthatja az Activity-t memória-felszabadítás céljából

#### Activity skeleton:

```java
public class ExampleActivity extends Activity {
	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState); 
		// Most jön létre az Activity
	}
	@Override
	protected void onStart() {
		 super.onStart();
		 // Most válik láthatóvá az Activity
	} 
	@Override
	protected void onResume() {
		 super.onResume(); 
		 // Láthatóvá vált az Activity 
	}

	@Override
	protected void onPause() { 
		super.onPause(); 
		// Másik Activity veszi át a focus-t 
		// (ez az Activity most kerül „Paused” állapotba)
	}
	@Override
	protected void onStop() {
	    super.onStop();
	    // Az Activity már nem látható 
	    // (most már „Stopped” állapotban van)
	}
	@Override
	protected void onDestroy() {
	    super.onDestroy(); 
		// Az Activity meg fog semmisülni 
	}
}
```
#### Életciklus callback függvények
- Amikor az Activity állapotot vált, megfelelő callback függvények hívódnak meg
- **Mindig meg kell hívni az ős osztály implementációját is (pl. super.onCreate();)**
- A rendszer felelőssége meghívni ezeket a fv-eket

Függvények:
- `onCreate()` - Activity létrejön és beállítja a megfelelő állapotokat.
- `onDestroy()` - Minden még lefoglalt erőforrás felszabadítása
- `onStart()` - Az Activity láható.
- `onStop()` - Az Activity nem látható
	- _Az Activity álete során többször válthat látható és nem látható állapotok között._
- `onRestart()` - `onStop()` után újraindítása után hívódik meg az `onStart()` előtt
- `onResume()` - Az Activity láthatóvá válik és előtérben van.
- `onPause()` - Az Activity háttérbe kerül, de valamennyire látszik a háttérben.

#### Activity váltás
- Callback fv-ek meghívási sorrendje:
	- A Activity `onPause()` fv-e.
	- B Activity `onCreate()`, `onStart()` és `onResume()` fv-e (B Activity-n van már a fókusz).
	- A Activity `onStop()` fv-e, mivel már nem látható.
> Ha B szeretne valamit, amit még az A ír be azt A-nak még az `onPause()` fb-ben kell megtörténjen.

	
#### Activity Back Stack

- Több Activity egy appban.
- A rendszer az Activity-ket egy ún. Back Stack-en tárolja.
- Az előtérben lévő Activity van a BackStack tetején
- Váltáskor eggyel lejjebb kerül a stacken.
- LIFO
![BackStack](https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2011/07/diagram_backstack1.png)

---
### Felhasználói felület alapok

#### Android felhasználói felület felépítése

- Minden elem a View-ból származik le
- Layout-ok:
	- ViewGroup leszármazottak
	- **ViewGroup extends View**
- ViewGroup-ok egymásba ágyazhatók

**Súlyozás**
- Megadható egy layout teljes súly értéke
- Elemek súly értéke is megadható és az alapján töltődik ki a layout

**Példa**
![súlyozás példa](https://i.loli.net/2018/11/04/5bddf8073f5b9.png)

## 3. Felhasználói felület, Layout-ok, LayoutInflater

### Legfontosabb fogalmak
- Képernyő méret
	- fizikai képátló
	- 4 kategória: _small_, _normal_, _large_, és _extra large_
- Képernyő sűrűség (dpi - dots per incs)
- Orientáció: A képernyő orientációja a felhasználó nézőpontjából
- Felbontás
- Sűrűség független pixel (dp)
	- Virtuális pixel egység
	- px = dp * (dpi / 160)
	- 1 dp = 1 px, ha dpi = 160
- erőforrás minősítő: erőforrás könyvtár utáni postfix jelölő, pl values**-hu** vagy layout**-large**)

### Erőforrás típusok
#### Gyakran használt erőforrások

- Drawable
- Hangok, videók
- Felhasználói felület leíró
- Animáció
- Stílusok, témák
- Szöveges erőforrások
- raw állományok

#### Szöveges erőforrások
- res/values/strings.xml
- Többnyelvűség miatt
- Paraméterezhetőség:
	- < string name="timeFormat"> %1$d minutes ago </ string>
	- használat:
		- Context.getString(R.strings.timeFormat, 14)


### Felhasználói felületi elemek
#### View hierarchia
![View hierarchia](https://i.loli.net/2018/11/04/5bde10db6c70e.png)

#### Dinamikus UI kezelés -LayoutInflater
- feladata
	- XML-ben összeállított felületi elemek példányosítása

Használata: 
```java
LayoutInflater layoutInflater = (LayoutInflater) 
getBaseContext().getSystemService(LAYOUT_INFLATER_SERVICE);
View myView = layoutInflater.inflate(R.layout.activity_main, null);
layoutLin.addView(myView);
```

## 4. Felugró ablakok, Stílusok, Animációk, Grafikai erőforrások, Fragmentek

### Felugró ablakok
- Activity felugró ablakban
	- Manifest állományba: <br/> `android:theme="@android:style/Theme.Dialog"`
- PopupWindow
	- PopupWindow objektum létrehozása és layout beállítása
- AlertDialog
	- AlertDialog.Builder
- Toast
	- `Toast.makeText( Context, getResources().getString(R.string.toast_info), Toast.LENGTH_LONG).show();`
- SnackBar

**PopUp**
```java
LayoutInflater layoutInflater = (LayoutInflater) getBaseContext()
	.getSystemService(LAYOUT_INFLATER_SERVICE);
View popupView = layoutInflater.inflate(R.layout.popuptest, null);
final PopupWindow popupWindow = new PopupWindow(popupView,
LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT);

Button btnDismiss = (Button) popupView.findViewById(R.id.btnPopUpOk);
btnDismiss.setOnClickListener(new Button.OnClickListener() {
	public void onClick(View v) {
		popupWindow.dismiss();
	}
});
popupWindow.showAsDropDown(btnShow, 50, -30);
```

**AlertDialog**

```java
private void showAlertMessage(final String aMessage) {
	AlertDialog.Builder alertbox = new AlertDialog.Builder(this);
	alertbox.setMessage(aMessage);
	alertbox.setNeutralButton("Ok",
		new DialogInterface.OnClickListener() {
			public void onClick(DialogInterface arg0, int arg1) {
				// Ok kiválasztva
			}
	});
	alertbox.show();
}
```


**SnackBar**
- képernyő alján megjelenő információs sáv
	- Toast helyett
	- Egyedi akció
![](https://i.loli.net/2018/11/04/5bde16279665b.png)

```java
Snackbar.make(findViewById(android.R.id.content),
	"Demo message", Snackbar.LENGTH_LONG)
		.setAction("Undo", mOnClickListener)
		.setActionTextColor(Color.RED).show();
```

### Stílusok, témák

#### Stílusok készítése
- Stílus file: res/values/styles.xml
	```xml
	<style name="ExampleStyle">
		<item name="android:textSize">22sp</item>
		<item name="android:textColor">#0000EE</item>
	</style>
	```
- használata
	```xml
	<TextView
		android:id="@+id/tvHello"
		android:text="@string/hello_world"
		style="@style/ExampleStyle" />
	```

#### Saját témák használata
- definíció: 
	```xml
	<style name="CustomTheme" parent="android:Theme">
		<item name="android:windowTitleSize">50dip</item>
		<item name="android:textColor">#000000</item>
		<item name="android:windowBackground">@color/white_color</item>
	</style>
	```
- A témák öröklődhetnek egymásból
- Manifestben állítható be:
	```xml
	<activity android:label="" android:name=".MainActivity"
		android:screenOrientation="portrait"
		android:theme="@style/CustomTheme"/>
	```

### Grafikai erőforrások xml-ben

- Grafikai elemek, hátterek is leírhatók XML-ben

### Animációk
- Animációk támogatása
	- XML erőforrás *(res/anim)*
	- Programkód
- Layout Animáció
	- Scale
	- Rotate
	- Translate
	- Alpha
- Három fő típus:
	- Tween animáció
	- Frame animáció
	- Property animator


- Tween animáció erőforrás
	```xml
	<?xml version="1.0" encoding="utf-8"?>
	<set xmlns:android="http://schemas.android.com/apk/res/android"
		android:shareInterpolator="false">
		<scale
			android:interpolator="@android:anim/accelerate_interpolator"
			android:fromXScale="0.0"
			android:toXScale="1.0"
			android:fromYScale="0.0"
			android:toYScale="1.0"
			android:pivotX="50%"
			android:pivotY="50%"
			android:duration="1000" />
		<alpha android:fromAlpha="0.0"
			android:toAlpha="1.0"
			android:duration="5000"/>
		<rotate
			android:interpolator="@android:anim/accelerate_interpolator"
			android:fromDegrees="0.0"
			android:toDegrees="360"
			android:pivotX="50%"
			android:pivotY="50%"
			android:duration="5000" />
	</set>
	```

- Tween animáció lejátszása
```java
@Override
public void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	setContentView(R.layout.main);
	Button btn = (Button)findViewById(R.id.button1);
	btn.setOnClickListener(new OnClickListener() {
		public void onClick(View v) {
			Toast.makeText(ActivityUI.this, getResources().getString(R.string.toast_info), Toast.LENGTH_LONG).show();
		}
	});
	Animation showAnim = AnimationUtils.loadAnimation(this,R.anim.btnanim);
	btn.startAnimation(showAnim);
}
```

### Élő háttérkép, widget

### Élő háttérkép
- Android 2.1-től
- Animált, interaktív háttér
- Hozzáférés a platform API-aihoz
- Pl:
	- Animált tájkép
	- Google térkép

#### Megvalósítása
- Leinkább egy Service-hez hasonló
- `OnCreateEngie()`:
	- WallpaperService.Engine létrehozása
	- Ez felelős az életciklusért és a rajzolásért

#### Eseménykezelés
-`onVisibilityChanged()`:
	- Ha háttérbe kerül, függesszünk fel mindent
-`onOffsetsChanged()`:
	- Home screen-ek közötti váltás
-`onTouchEvent()`
-`onCommand()`:
	- Más app küldhet utasítást a wp-nek, de jelenleg csak a Home app tud.


### Widget
- Kis alkalmazások, melyek tipikkusan a Home Screen-e ágyazódnak
- Periodikusan frissülnek


### Fragment-ek

#### Fragment API
- Fragment: A képrenyő egy nagyobb részéért felelős, és/vagy háttérben munkát végző objektumok
- Hasznosság: több funkció egy képernyőn
- Egy Fragment mindig egy Activity-hez csatoltan jelenik meg
- Az életciklusaik nagyrészt megegyeznek
![Fragment](https://developer.android.com/images/fragment_lifecycle.png)

#### Csatolása
- Statikusan 
	- Activity layout-ban <fragment .../> tag
-Dinamikusan
	- Futás közben tölti be a megfelelő Fragment-eket, adott ViewGroup-okba
	- Fragment-Tranzakciókkal módosítható

#### FragmentManager
- activity.getFragmentManager()
- FragmentTransaction indítása
- Aktív Fragment-ek  közt keres
	- Tag alapján
	- ID alapján
- Fragment-stack-et menedzseli

#### FragmentTransaction osztály
- Ezen kersztül módosíthatók az aktív Fragment-ek
- A FragmentManager `.beginTransaction()` metódusával indítható
- Fontosabb műveletek:
	- `add(...)` / `.remove(...)` / `.replace(...)`
		- Fragment példányok le- és felcsatolása az adott Activity-re
	- `.commit()`
		- Tranzakció végrehajtása
	- `.show(...)` / `.hide(...)`
		- Fragment példány elrejtése / újra megjelenítése
	- `.setTransition(...)` / `.setCustomAnimations(...)`
		- A tranzakció végrehajtásakor lejátszandó animáció beállítása
	- `.addToBackStack(...)`
		- Rákerüljön-e a FragmentTransaction backstack-re a tranzakció?
	- `.commit()`
		- Tranzakció végrehajtása

#### FragmentTransaction példa:
- Fragment kicserélése:
```java
FragmentManager fm = getSupportFragmentManager();

FragmentTransaction ft = fm.beginTransaction();
ft.replace(R.id.FragmentContent,
	DetailsFragment.newInstance(selIndex),
	DetailsFragment.TAG);
ft.commit();
```
- Fragment hozzáadása, a tranzakciót a backstack-re téve:
```java
FragmentManager fm = getSupportFragmentManager();
FragmentTransaction ft = fm.beginTransaction();
ft.setCustomAnimations(R.anim.slide_in_top,
	R.anim.slide_out_right);
ft.add(R.id.FragmentContent,
	DetailsFragment.newInstance(position),
	DetailsFragmenr.TAG);
ft.addToBackStack(null);
ft.commit();
```

#### Fragment kommunikáció
- Egy Fregment-nek egységbezártnak kell lennie -> közvetett kommunikáció
	- Az Activity közvetít

![Fragment comm](https://i.loli.net/2018/11/05/5bdf368009562.png)


#### DialogFragment
- Egy Fragment dialógusként is megjelenhet
- így egy dialógus megkapja a Fragment életciklust
- a DialogFragment-ek is rákerülhetnek a BackStack-re
-  `.onCreateDialog()`
	- Ez a metódus is visszatérhet a megjelenítendő Dialog-gal
- `.onCreateView()`
	- `.onCreateDialog()` helyett
	- Tetszőleges megjeleníthető tartalom

## 5. Runtime permission, Perzisztens adattárolás, ContentProvider

### Android Engedélyek (Permission modell) 

- Célja
	- Veszélyes/kritikus/személyes adatokat érintő műveletekhez engedélyre van szükség a felhasználótól.
- Új permission modell 6-os Androidtól felfele:
	- Runtime permission
- Permission meglétének ellenőrzése:
	`int permissionCheck = ContextCompat.checkSelfPermission(thisActivity, Manifest.permission.WRITE_CALENDAR);`
- Permission kérés Activity-ben:
	- Megvizsgálni, hogy megvan-e
	- Elmagyarázni hogy mire kell
	- Eredmény kezelés
#### Permission típusok
- Manifestben továbbra is érdemes fellsorolni
	- Normal -> azonnal
	- Veszélyes -> prompt
- Engedélyeket vissza lehet kérni
#### Permission kérés példa

```java 
public static final int REQUEST_CODE_LOCATION_PERMISSION = 401;

public void requestNeededPermission() {
	if (ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
		if (ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.ACCESS_FINE_LOCATION)) {
			// aszinkron módon magyarázat megjelenítése dialógusban,
			// majd újra kérés manuálisan
		}
		ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.ACCESS_FINE_LOCATION}, REQUEST_CODE_LOCATION_PERMISSION);
	} else {
		// megkaptuk az engedélyt, indíthatjuk a kívánt műveletet
	}
}
```
```java
@Override
public void onRequestPermissionsResult(int requestCode, String permissions[], int[] grantResults) {
	switch (requestCode) {
		case REQUEST_CODE_LOCATION_PERMISSION: {
			// Ha mégsem (vissza gomb) lett választva a
			// kéréskor akkor üres a tömb
			if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
				// Megkaptuk az engedélyt, indítható a művelet
				myLocationManager.startLocationMonitoring();
			} else {
				// Nem kaptuk meg az engedélyt,
				// üzenet jelzése a felhasználónak
			}
		return;
		}	
	}
}
```
- Külső könyvtár permission kérésre
	- [https://github.com/hotchemi/PermissionsDispatcher](https://github.com/hotchemi/PermissionsDispatcher)

![PermissionsDispatcher](https://i.loli.net/2018/11/05/5bdf56ee03da1.png)


#### Legjobb gyakorlatok permission kezelésre 
- Használjuk a lehetőleg a rendszer beépített szolgáltatásait (pl. kamera)!
- CSak a szükséges engedélyeket kérjük el
- Ne terheljük túl a felhasználót kérésekkel!
	- Csak akkor kérjük el az engedélyt, amikor szükség van rá
- Magyarázzuk el a felhasználónak, hogy miért van szükség az adott engedélyre!
- Teszteljük mindkét permission kezelést


### Perzisztens adattárolás

#### Bevezetés

- Androidon minden igényre van beépített megoldás: 
	- **SharedPrefences:** alaptípusok kulcs-érték párokban
	- **Privát lemezterület:** nem publikus adatok tárolása a fájlrendserben
	- **SD kártya:** nagy méretű adatok tárolás, nyilvánosan hozzáférhető
	- **SQLite adatbázis:** strukturált adatok tárolására
	- **Hálózat:** szerveren vagy felhőben tárolt adatok

### SQLite

- Relációs adatbáziskezelő
- Strukturált adatok tárolására ez a legalkalmasabb
- Alapból nincs ORM, nekünk kell a sémát meghatározni és megírni a query-ket
- külső ORM osztálykönyvtárak
- *érdemes minden táblában elsődleges kulcsot definiálni*
	- autoincrement
	- Ahhoz, hogy ContentProvider-rel ki tudjuk ajánlani, ill. UI elemeket Adapterrel feltölteni, **kötelező egy ilyen oszlop**, melynek neve: "_id"


#### Jellemzői
- Standard relációs adatbázis szolgáltatások:
	- SQL szintax
	- Tranzakciók
	- Prepared statement
- Támogatott oszlop típusok:
	- TEXT *(String)*
	- INTEGER *(long)*
	- REAL *(double)*
- File rendszer elérés miatt lassú lehet
- Érdemes aszinkron módon végrehajtani

### Alap SQLite API

- Adatbázis megnyitásához az `SQLiteOpenHelper` használható:
```java
public class DictionaryOpenHelper extends SQLiteOpenHelper {
	
	private static final int DATABASE_VERSION = 2;
	private static final String DICTIONARY_TABLE_NAME = "dictionary";
	private static final String DICTIONARY_TABLE_CREATE = "CREATE TABLE " + DICTIONARY_TABLE_NAME + " (" + KEY_WORD + " TEXT, " + KEY_DEFINITION + " TEXT);";
	
	DictionaryOpenHelper(Context context) {
		super(context, DATABASE_NAME, null, DATABASE_VERSION);	
	}

	@Override
	public void onCreate(SQLiteDatabase db) {
		db.execSQL(DICTIONARY_TABLE_CREATE);	
	}
}
```
#### SQLite Használata:
- Pldányosítjuk a saját `SQLiteOpenHelper` osztályunkat
- A példányon meghívjuk a `.getWriteableDatabase()` vagy `.getReadableDatabase()` metódust, amik egy `SQLiteDatabase` objektumot adnak vissaz. Ez a teljes db-t reprezentálja .
- Ezen az obj-on futtathatunk query-ket
	- használható az `SQLiteQueryBuilder` segédosztály
- Minden query egy Cursor obj-mal tér vissza, ami az eredmény adathalmazra mutat
	- do-while ciklussal végigmenni az adathalmazon.
	- 

### Shared Preferences
- Alaptípusok tárolása kulcs-érték párokként *~Dictionary*
- Fájlban tárolódik, de ezt elfedi az OS
- Létrehozáskor beállítható a láthatósága:
	- **MODE_PRIVATE:** csak saját app látja
	- **MODE_WORLD_READABLE:** csak a saját appunk írhatja, bárki olvashatja
	- **MODE_WORLD_WRITABLE:** bárki írhatja és olvashatja
- Ideális primitív adatok tárolására pl:
	- Default beállítások értékei
	- UI állapot
	- Settings-ben megjelenő beállítások
- Több lehet, a nevük különbözteti meg őket.
	- `getSharedPreferences(String name, int mode);`
	-  Ha nem létezik akkor létrehozódik
- nem kötelező elnevezni, ha csak egy van
#### Preferences framework
- Android által biztosított XML alapú keretrendszer saját Beállítások képernyő létrehozására
	- ugyanúgy fog kinézni mint az alap Beállítások alkalmazás


### File kezelés
#### Fájlkezelés Androidon
- Ugyanaz mint sima Java esetén
- Néhány specialitás Android környezetben
- Két lemezterület
	- Internal: védett csak az alkalmazás érheti el
	- External: felhasználó által is írható-olvasható terület
- *External torage* is lehet belső memóriában, ha nincs SD kártya a készülékben


#### Internal storage
- `openFileOutput(String filename, int mode)`
	- Támogatott Módok:
		- Context:MODE_PRIVATE: alapértelmezett, felülírja, ha már van benn valami
		- Context.MODE_APPEND: hozzáfűz
- Fájl olvasása ugyanígy:
	- `openFileInput(String filename)`
	- Byte-okat olvas `read()` methodussal
- Cache használata:
	- `getCacheDir()` visszaad egy File objektumot, ami a cache könyvtárra mutat
	- Ezen belül létrehozhatunk cache fájlokat

#### Statickus fájlok egy alkalmazáshoz
- Szükséges lehet statikusan fájlokat linkelni:
	- Bármi ami fájl, de nem illik a res könyvtár mappáiba
- Fejlesztéskor a **res/raw** mappába kell rakni
- Read-only lesz telepítés után, nem tudjuk utólag módosítani
- Olvasásuk futásidőben:
	```java
	Resources resources = getResources();
	InputStream inStream = resources.openRawResource(R.raw.myfile);
	```

#### Nyilványos lemezterület

- Lehet akár SD kártyán, akár belső memóriában
- Bárki által írható, olvasható
- Amikor a telefon mountolódik, a fájlok Read-Onlyvá válnak

```java
String state = Environment.getExternalStorageState();
// sokféle állapotban lehet, nekünk kettő fontos:
if (state.equals(Environment.MEDIA_MOUNTED)) {
	// Olvashatjuk és írhatjuk a külső tárat
} else if (state.equals(Environment.MEDIA_MOUNTED_READ_ONLY)) {
	// Csak olvasni tudjuk
} else {
	// Valami más állapotban van, se olvasni,
	// se írni nem tudjuk
}
```

- Fájlok elérése a nyilvános tárhelyen v2.2-től:
	`File filesDir = getExternalFilesDir(int type);`
- type: megadhatjuk milyen típusú fájlok könyvtárát akarjuk használni, például:
	- null: root
	- DIRECTORY_MUSIC:
	- DIRECTORY_PICTURES:
	- DIRECTORY_RINGTONES
	- DIRECTORY_DOWNLOADS
	- DIRECTORY_DCIM
	- DIRECTORY_MOVIES
- **andorid.permission.WRITE_EXTERNAL_STORAGE**

### ContentProvider
- Olyan mechanizmus, ami
	- Elérési réteget biztosít strukturált adatokhoz
	- Elfedi az adat tényleges tárolási módját 
	- Adatvédelemet biztosít
	- processzek közti adatmegosztásra is alkalmas

#### Adattárolás
- Konkrét adattárolási struktúra/módszer az alkalmazásra bízva
- Ami szükséges: a megfelelő interfész megvalósítása
	- Leszármaztatás az absztrakt ContentProvider osztályból és a kötelező metódusok implementálása 
- legegyszerűbb: SQLite

#### ContentProvider elérése
- **ContentResolver**:
	- Csak ez tudja lekérdezni a Content Providert
	- IPC (InterProcess Communication) az Android által megoldva
	- singleton Content Provider -> ezt éri el az összes Resolver
	- Műveletek:
		- SELECT: `getContentResolver().query(...)`
		- INSERT: `getContentResolver().insert(...)`
		- UPDATE: `getContentResolver().update(...)`
		- DELETE: `getContentResolver().delete(...)`

#### CONTENT_URI
- A azonosítja a Content Provider-t, és aon belül a táblát
- Pl. **UserDicitionary.Words.CONTENT_URI = content://user_dictionary/words**
- Felépítése:
	- **content://** - **séma**, ez mindig jelen van, ebből tudja a rendszer hogy ez egy Content URI
	- **user_dictionary - "authority"**, azonosítja a Providert, globálisan egyedinek kell lennie
	- **words - "path",** az adattábla neve


## 6. RecyclerView, Intent

### Listák kezelése

- ListActivity / ListFragment
- Tartalmaz egy ListView-t -> `getListView()`
- Beállítható egy ListAdapter, mely a lista elemeinek tárolásáért és megjelenítésért felelős
	- Tartalmazza az elemeket  (objektumok listája)
	- Felelős
		- hozzáadásért
		- elérésért
		- törlésért
	- A `getView(int position, View converView, ViewGroup parent)` fv felüldefiniálásával megadhatjuk, hogy a lista egy sorában egy elem hogyan renderelődjön
	- A converView egy korábbi lista sor, ami felhasználható az új sor létrehozására 

#### ListAdapter példa

```java
public class MyContactsAdapter extends ArrayAdapter {
	private List myContacts;
	private Context context;
	public MyContactsAdapter(Context context, int textViewResourceId,
List contacts) {
		super(context, textViewResourceId, data);
		this.context = context;
		this.myContacts = contacts;
	}
	
	@Override
	public View getView(int position, View convertView, ViewGroup parent) {
		View v = convertView;
		if(v == null) {
			LayoutInflater inflater = LayoutInflater.from(context);
			v = inflater.inflate(R.layout.layout_list_item, parent);
		}
		TextView txtName = (TextView)v.findViewById(R.id.txtName);
		TextView txtMail = (TextView)v.findViewById(R.id.txtMail);
		MyContact entry = myContacts.get(position);
		txtName.setText(entry.getName());
		txtMail.setText(entry.getMail());
		return v;
	}
}
```

getView(...) használata - ViewHolder-rel (gyors!) 

```java
@Override
public View getView(int position, View convertView, ViewGroup parent) {
	View v = convertView;
	if(v == null) {
		LayoutInflater inflater = LayoutInflater.from(mContext);
		v = inflater.inflate(R.layout.layout_list_item, parent);
		ViewHolder holder = new ViewHolder();
		holder.txtName = (TextView)v.findViewById(R.id.txtName);
		holder.txtMail = (TextView)v.findViewById(R.id.txtMail);
		v.setTag(holder);
	}
	Contact entry = mList.get(position);
	if(entry != null) {
		ViewHolder holder = (ViewHolder)v.getTag();
		holder.txtName.setText(entry.getName());
		holder.txtMail.setText(entry.getMail());
	}
	return v;
}

static class ViewHolder {
	TextView txtName;
	TextView txtMail;
}
```

#### ViewHolder pattern előnyei

- Statikus `ViewHolder` objektum, cache támogatás
- Nincs folyamatos `findViewById(...) hívás
- Gyors!

### RecyclerView
- A ListView fejlett és flexibilis változata
- ViewHolder minta kikényszerítése
- Hatékony elem újrafelhasználás
- Fejlesztés:
	- ListView, és Adapter mellett egy LayoutManager-t is kell készíteni
- A LayoutManager feladata a `findViewById(...)` felesleges sokszori hívásának elkerülése


#### RecyclerView.Adapter< ViewHolder > 
**- Inicializálás, konstruktor**
```java
private final List<Todo> todos = new ArrayList<Todo>();
private Context context;

public TodoAdapter(Context context) {
	this.context = context;
	for (int j = 0; j < 20; j++) {
		todos.add(new Todo("Todo " + j, false));
	}
}
```
- **InitViewHolder**
```java
@Override
public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
	View rowView = LayoutInflater.from(parent.getContext())
		.inflate(R.layout.todo_list_row, parent, false);
	return new ViewHolder(rowView);
}
```

**- ViewHolder Implementáció**

```java
public static class ViewHolder extends RecyclerView.ViewHolder {
	private final TextView tvTodo;
	private final CheckBox cbDone;
	
	public ViewHolder(View itemView) {
		super(itemView);
		tvTodo = (TextView) itemView.findViewById(R.id.tvTodo);
		cbDone = (CheckBox) itemView.findViewById(R.id.cbDone);
	}
}
```
**- ViewHolder bindig**

```java
@Override
public void onBindViewHolder(ViewHolder holder, final int position) {
	final Todo todo = todos.get(position);
	holder.tvTodo.setText(todo.getTodo());
	holder.cbDone.setChecked(todo.isDone());
	holder.cbDone.setOnClickListener(new View.OnClickListener() {
		@Override
		public void onClick(View v) {
			Toast.makeText(context, "Checkbox "+position+" clicked!",
				Toast.LENGTH_SHORT).show();
		}
	});
	
	holder.itemView.setTag(todo);
}
```
**- Elemek száma, hozzáadás, törlés**
```java
@Override
public int getItemCount() {
	return todos.size();
}

public void removeTodo(int position) {
	todos.remove(position);
	notifyItemRemoved(position);
}

public void addTodo(Todo todo) {
	todos.add(0, todo);
	notifyDataSetChanged();
	//notifyItemInserted(0);
}
```


### Intent
#### Lazán csatolt komponensek és köztük a kommunikáció
WTH? 
- Appok külön ART VM példányokban *(sandbox)*
- Kritikus műveletekhez engedély szükséges
- App = komponensek halmaza
A komponensek akár alkalmazások között is kommunikálhatnak egymással **(!)**
- Két komponens között: __Intent__ _(pl.: következő képernyőre lépés 
- Egy komponensből mindenki másnak: __Broadcast Intent__  _(pl.: akku alacsony)_
### **Intent átadása: Mindig az Android runtime-on keresztül!**
- **Intent típusok:**
	- Explicit Intent:
		- konkrétan meg van nevezve a cél komponens
	- Implicit intent:
		- a végrehajtandó feladat kerül leírásra
- **Intent részei:**
	- **Címzett komponens osztályneve**
	- **Akció** (az elvárt vagy megtörtént esemény)
	- __Adat__
	- __Kategória__
	- __Extrák__ (saját kulcs-érték párok, amiket át akarunk adni a címzettnek) 
	- **Kapcsolók** *(Flags)*

#### Explicit Intent
- Az Intent-ben kitöltjük a címzett komponens nevét
- Ha van már példány adott komponensből akkor az folytatódik és nem hozódik létre új.
#### Implicit Intent
- Nem tudjuk a címzett komponens nevét csak az akciót
- Pl.:
	```java
	Intent i = new Intent(Intent.ACTION_DIAL, Uri.parse(„tel:0630-123-4567”));
	startActivity(i);
	```
#### Intent Filter
- Lehetséges a saját alkalmazásunk funkcióinak kiajánlása mások számára
	- Az androidban beépítve vannak ilyenk, ld. Intent Action (pl. ACTION_CALL, ACTION_IMAGE_CAPTURE)
	- Az AndroidManifest-ben kell deklarálni
	- Ha nincs Intent filter beállíta, akkor a komponens kizárólag explicit intetnet képes fogadni

## 7. BradcastReceiver, Http communication, Service

### BroadcastReceiver

#### Broadcast események
- Rendszerszintű eseményekre fel lehet iratkozni - Broadcast üzenet
- Az Intent alkalmas arra, hogy leírja az eseményt
- Sok beépített Broadcast Intent, lehet egyedi is
![Bradcast Intents](https://i.loli.net/2018/11/06/5be09ca9964fe.png)

#### Broadcast események
- Nem csak az Android, hanem appok is dobhatnak Bradcast Intentet
	- Telephony service küldi az ACTION_PHONE_STATE_CHANGED Broadcast Intentet, ha a mobilhálózat csatlakozás megváltozott
	- Saját appunkból is dobhatunk a `sendBroadcast(String action)` metódussal
	- Ez is Intent, lehet Extra és Data része

#### Broadcast intentek elkapása
- Broadcast Receiver nevű komponens segítségével
	- Kódból vagy manifestben kell regisztrálni
	- Intent filterr állíthatjuk be, hogy milyen Intent esetén aktivizálódjon
- nem Activity, nincs UI
- Azonban képes Activity-t indítani
- Használata: BradcastReceiver osztályból származtatunk, és felüldefiniáljuk az `onReceive()` metódust, majd intent-filter
![anyád](https://i.loli.net/2018/11/06/5be09de14ebd3.png)

#### Broadcast intentek kezelése
Broadcast tobbdobásának megakadályozása:
- `abortBradcast()`
Pl. ha a fulhallgató média gombjait kell kezelni és nem akarjuk, hogy a zenelejátszó is megkapja a Broadcast-ot

#### Alkalmazáskomponens indítása Boot után
- Néha olyan szolgáltatásokra van szükség, amelyek mindig futnak a készüléken
- Van lehetőség feliratkozni a *"boot fejeződött"* eseményre
	- BroadcastReceiver efiniálása Manifest-ben
	- android.intent.action.BOOT_COMPLETED
- `onReceive()` fv-ben elindíthatjuk a megfelelő komponenst

### Hálózati kommunikáció

- "Rövid" távú kommunikáció
	- NFC
	- Infra
	- Bluetooth
- "Hosszabb" távú / internet alapú kommunikáció
	- UDP
	- TCP/IP, Socket, ServerSocket
	- HTTP (REST)
	
#### Bluetooth
- Bluetooth készülékek felderítése
- Helyi Bluetooth adaptere-ek lekérdezése a párosított eszközökhöz
- RFCOMM csatorna kiépítése
- Adattovábbítás
- Fontosabb osztályok
	- android.bluetooth csomag

#### Legfontosabb Bluetooth osztályok 
- *BluetoothAdapter*: felderítés, párosítások lekérdezése ... 
- *BluetoothDevice*: távoli Bluetooth eszközt jelképezi
- *BletoothSocket/BluetoothServerSocket*
- *BluetoothClass*: eszköz tulajdonságai (Read-Only)
- *BluetoothProfile*: profil jellemzői

### HTTP(S) kommunikáció

####Android platformon
- HTTP methods
	- GET, POST, PUT, DELETE
- Teljes körű HTTPS támogatás és cerificate import lehetőség
- REST komm. támogatás

#### HTTP kapcsolatok kezelése
- Szükséges engedély:
	`< uses-permission android:name= "android.permission.INTERNET" />`
-  ==Új szálban kell megvalósítani a hálózati kommunikáció hívást!!==
-  Ellenőrizzük a HTTP válasz kódot: 
- HTTP REST
- Ügyeljünk az alapos hibakezelésre

#### HTTP GET példa

```java
private void httpGet(String urlAddr) {
	BufferedReader reader = null;
	try {
		URL url = new URL(“http://mysrver.com/api/getitems”);
		HttpURLConnection conn = (HttpURLConnection) url.openConnection();
		reader = new BufferedReader(new InputStreamReader(conn.getInputStream()));
		String line = "";
		while ((line = reader.readLine()) != null) {
			System.out.println(line);
		}
	} catch (IOException e) {
		e.printStackTrace();
	} finally {
		if (reader != null) {
			try {
				reader.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
}
```

#### HTTP POST példa

```java
private void httpPost(String urlAddr, byte[] postBody) {
	…
	OutputStream os = null;
	try {
		URL url = new URL(urlAddr);
		HttpURLConnection conn = (HttpURLConnection) url.openConnection();
		conn.setRequestMethod(„POST”);
		conn.setDoOutput(true);
		conn.setUseCaches(false);
		os = conn.getOutputStream();
		os.write(content);
		os.flush();
		…
	} catch (IOException e) {
		e.printStackTrace();
	} finally {
		…
		if (os!= null) {
			try {
				os.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
}
```

#### Timeout értékek beállítása
- Fontos, hogy faszán legyen kezelve
- Timeout a kapcsolat megnyitásra
- Timeout az eredmény kiolvasására

#### Header paraméterek beállítása
 - Egyszerű Header beállítása:
```java
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestProperty(”[KEY]”,”[VALUE”);
```
- Cookie beállítása:
```java
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestProperty(”Cookie”,”sessionid=abc;age=15”);
```
- Összetett példa: 
```java
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestProperty(„Content-Type”,”application/json”);
conn.setRequestProperty(”Cookie”,”sessionid=abc;age=15”);
```


### Szálak

#### UI módosítása más szálból
- Indításkor OS létrehoz egy *main* szálat (UI száál) 
- Sokáig tartó műveletek blokkolják a UI-t -> új szálba
- **Az Android a UI-t csak a fő szálból engedi módosítani!**
- megoldások: 
	- Activity.runOnUiThread(Runnable)  
	- View.post(Runnable) 
	- View.postDelayed(Runnable, long) 
	- Handler 
	- AsyncTask 
	- LocalBroadcast 
	- Külső libek, pl. EventBus v. Otto 
	- Retrofit

#### AsyncTask példa

```java
public class AsyncTaskUploadVote extends AsyncTask<String, Void, String> {
	private Context context = null;
	public AsyncTaskUploadVote(Context context) {
		this.context = context;
	}
	
	@Override
	protected void onPreExecute() {
		// UI szálon fut
	}

	@Override
	protected String doInBackground(String... params) {
		String result = null;
		// Hálózati kommunikácó és a válasz mentése a result-ba
		return result;
	}

	@Override
	protected void onPostExecute(String result) {
		// LocalBroadcast küldése
		Intent intent = new Intent("REST_RESULT");
		intent.putExtra("response", result);
		LocalBroadcastManager.getInstance(context).sendBroadcast(intent);
	}
}
```
- AsyncTask indítása:
```java
new AsyncTaskWeather(getApplicationContext()).execute(
	"http://api.openweathermap.org/data/2.5/weather?q=London,uk");
```
- LocalBoradcast feliratkozás/leiratkozás pl. Activity-ben vagy Fragment-ben:
```java
@Override
protected void onResume() {
	super.onResume();
	LocalBroadcastManager.getInstance(this).registerReceiver(mMessageReceiver, newIntentFilter("REST_RESULT"));
}

@Override
protected void onPause() {
	super.onPause();
	LocalBroadcastManager.getInstance(this).unregisterReceiver(mMessageReceiver);
}

private BroadcastReceiver mMessageReceiver = new BroadcastReceiver() {
	@Override
	public void onReceive(Context context, Intent intent) {
		String rawWeather = intent.getStringExtra("response");
		Toast.makeText(MainActivity.this, rawWeather, Toast.LENGTH_LONG).show();
	}
}
```
#### Tipikus üzenet formátumok
- CSV, TSV
- XML	
- JSON

#### JSON feldolgozás
- JSONObject
	- JSON objektumok parse-olása
	- Elemek elérhetősége a kulcs megadásával:
		- `getString(String name) `
		- `getJSONObject(String name)`
		- `getJSONArray(Stringname)`
	- JSON objektum létrehozása String-ből vagy Map-ból
- JSONArray:
	- elemek lekérdezése index alapján
	- van hossza
	- Létrehozás pl *Collection*-ból

### Push notificaiton
- Sokszok szükség lehet rá, hogy a szerver tájékoztassa amobil klienseket valamilyen eseményről
- Jelenlegi eszökünk:
	- Poll-ozás
		- Lassú és nem real-time
- Fordítottan: valahogy a szervernek kéne tájékoztatni a klienseket
- Megoldás GCM (Google Cloud Messaging) 
- Szerver és kliens oldali implementáció szükséges

#### Android GCM/FCM
- Cél: szerver -> kliens kommunikáció
- Rövid üzenetek (~4kb) a kliensek értesítésére
- __Ingyenes__
- GCM/FCM komponensek:
	- Android készülék
	- App server
	- GCM/FCM cloud

### Service

#### Bemutatás
- Háttérben futó feladatot lát el
- Felhasználói beavatkozás nélkül működik
- No UI
- Más komponensek/alkalmazások számára is szolgáltathat funkciókat
- Bármilyen alkalmazáskomponens elindíthatja és **akkor is futva maradhat, amikor a hívó** (pl. Activity) **megáll**, tipikusan ha más alkalmazásra váltunk.
- Példák:
	- Zenelejátszó
	- torrent
	- GPS pozíció követés
	- stb..

#### Típusok
- Kétféle
	- Komponensből indított / Started
	- Kapcsolt / Bound
![](https://i.loli.net/2018/11/06/5be0c391d08e0.png)


1. Komponensből indított (**Started**):
	- Valamilyen app komponens (tipikusan Activity) elindítja a `startService(intent)` metódussal
	- A Service akkor is fut, amikor a komponens már megsemmisült.
		- *A hívó megállítása nem törli a Service-t is!*
	-	Általában az í indítottak **egy feladatot hajtanak végre**, (pl: egy fájl letöltése) 
	-	Csak a hívónak van rá referenciája
	-	Ha végzett, **magát kell leállítania** a `stopSelf()` metódussal.

#### Service megvalósítás
- Service osztályból származunk
- Megvalósítjuk a megfelelő callback-eket
	- Ha *Started*-ként (is) akarjuk használni: 
		- `onStartCommand()` implementálása
	- Ha *Bound*-ként (is) akarjuk használni:
		- `onBind()` implementálása

![Service lifecycle](https://developer.android.com/images/service_lifecycle.png)

#### Műveletek vérehajtása Service-n belül
- A Service alapértelmezetten a processze fő szálában fut, **nem kap külön szálat!!**
- Erőforrás igényes műveletek esetén "kézzel" kell új szálat indítanunk, pl.:
	- CPU intenzív feladatok (pl. titkosítás, zene lejátszás) 
	- Hálózati komm.
	- stb...
- Ellenkező esetben 5mp után ugyanúgy megjelenik az *Application Not Responding* ablak, mint Activity esetén

