# Lab: DOM-based cookie manipulation

## 📚 Studijní materiál

Tento repozitář slouží jako můj osobní studijní zápisník a přehled řešení laboratoří z platformy **PortSwigger Web Security Academy**.

---

## 📋 Přehled zranitelnosti

DOM-based cookie manipulation nastává v momentě, kdy klientský JavaScript přijímá uživatelsky řízená data (tzv. **Source**) a zapisuje je přímo do hodnoty cookie v prohlížeči (**Sink**) bez dostatečné sanitizace nebo validace.

Pokud aplikace následně čte  manipulovanou cookie a nevhodně ji zpracovává (například vrací její obsah do HTML bez ošetření), útočník může podvrhnout data nebo vyvolat DOM-based Cross-Site Scripting (DOM XSS).

## 🎯 Cíl laboratoře

Donutit oběť  aby navštívila námi vytvořený exploit server, který manipuluje s hodnotou cookie v jejím prohlížeči a následně vyvolá funkci `print()`.

## 🛠️ Mechanika zranitelnosti (Source → Sink)

- **Source (Zdroj):** `window.location.search` nebo jiný parametr URL.
- **Sink (Místo zápisu):** `document.cookie`.
- **Následný Sink (Místo vykonání):** Vložení hodnoty cookie do dokumentu bez bezpečného ošetření.

---

## 🚀 Krok za krokem (Exploitation Process)

### 1. Analýza zdrojového kódu a chování aplikace

Otevři libovolnou stránku produktu na cílovém webu.

V prohlížeči otevři Developer Tools (`F12`) → záložku Console nebo Sources.

Při prohlížení produktů si všimni, že aplikace ukládá například naposledy navštívený produkt do cookie pomocí klientského skriptu:

```javascript
document.cookie = "lastViewedProduct=" + window.location.href;
```

Na hlavní stránce se pak tento odkaz vykresluje zpět do DOM jako HTML odkaz.

### 2. Konstrukce Exploit Payloadu

Potřebujeme dostat vlastní JavaScript do hodnoty, která se později vykreslí v atributu `href`.

Ukázka testovacího payloadu:

```html
"><script>print()</script>
```

Alternativní varianta:

```html
'/><script>print()</script>
```

### 3. Vytvoření Exploit Serveru

V prostředí PortSwigger otevři **Exploit Server**.

Do sekce **Body** vlož exploit HTML/JavaScript odpovídající struktuře labu.

Před použitím nahraď `YOUR-LAB-ID` skutečnou adresou své laboratoře.

```html
<iframe
    src="https://YOUR-LAB-ID.web-security-academy.net/product?productId=1&"
    onload="this.src='https://YOUR-LAB-ID.web-security-academy.net/'">
</iframe>
```

### Proč to fungovalo?

1. `iframe` nejprve načte stránku produktu s upravenou URL.
2. Klientský skript přečte aktuální adresu pomocí `window.location.href`.
3. Načtená URL se uloží do `document.cookie`.
4. Událost `onload` následně přesměruje iframe na hlavní stránku.
5. Hlavní stránka přečte hodnotu z cookie a vykreslí ji do DOM.
6. Pokud je hodnota vložena do HTML bez bezpečného ošetření, může dojít ke spuštění vloženého JavaScriptu a zavolání funkce `print()`.

### 4. Doručení oběti (Deliver to Victim)

1. Vyzkoušej tlačítko **View exploit** a ověř, zda se v prohlížeči objeví tiskové okno.
2. Klikni na **Deliver exploit to victim**.
3. Počkej na potvrzení úspěšného vyřešení laboratoře:

`Congratulations, you solved the lab!`

--

## 🔍 Technické vysvětlení – Proč to fungovalo?

Aplikace ukládala celou URL produktu do cookie pomocí JavaScriptu, aniž by kontrolovala její obsah. Při návratu na hlavní stránku pak hodnotu cookie znovu vložila do HTML.

Tím vzniklo propojení mezi uživatelským vstupem (URL), cookie a DOM. Protože aplikace nepracovala s uloženou hodnotou bezpečně, prohlížeč ji interpretoval jako HTML, nikoliv jako obyčejný text.

Právě toho jsem využil. Upravená URL se uložila do cookie a při následném vykreslení na hlavní stránce došlo ke spuštění vloženého JavaScriptu `print()`.