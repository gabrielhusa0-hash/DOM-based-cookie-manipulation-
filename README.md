# Lab: DOM-based cookie manipulation

# 📚 Studijní materiál
Tento repozitář slouží jako můj osobní studijní zápisník a přehled řešení laboratoří z platformy PortSwigger Web Security Academy

# --

# 📋 Přehled zranitelnosti

DOM-based cookie manipulation nastává v momentě, kdy klientský JavaScript přijímá uživatelsky řízená data (tzv. Source) a zapisuje je přímo do hodnota cookie záznamu v prohlížeči (Sink) bez dostatečné sanitizace nebo validace.

Pokud aplikace následně čte tato manipulovaná cookie a nevhodně je zpracovává (například vrací jejich obsah do HTML bez ošetření), útočník může přesměrovat uživatele, podvrhnout data nebo vyvolat DOM-based Cross-Site Scripting (DOM XSS).

# 🎯 Cíl laboratoře

Donutit oběť (bot v prostředí PortSwigger), aby navštívila námi vytvořený exploit server, který injektuje škodlivý kód do jejího prohlížeče přes cookie záznam a vyvolá funkci print().

# 🛠️ Mechanika zranitelnosti (Source -> Sink)

Source (Zdroj): window.location.search nebo parametr URL (např. zranitelný parametr na stránce produktu).

Sink (Místo zápisu): document.cookie

Následný Sink (Místo vykonání): Výpis hodnoty z cookie do dokumentu bez enkódování.

# 🚀 Krok za krokem (Exploitation Process)

1. Analýza zdrojového kódu a chování aplikace

Otevři libovolnou stránku produktu na cílovém webu.

V prohlížeči otevři Developer Tools (F12) -> záložka Console nebo Sources.

Při prohlížení produktů si všimni, že aplikace ukládá např. naposledy navštívený produkt do cookie záznamu pomocí klientského skriptu:

document.cookie = "lastViewedProduct=" + window.location.href;


Na hlavní stránce se pak tento odkaz vykresluje zpět do DOMu jako HTML link:

```html
<a href="...">Last viewed product</a>


2. Konstrukce Exploit Payloadu

Potřebujeme vnutit do atributu href vlastní JavaScript, který po načtení okamžitě vykoná funkci print().

Připravíme si payload, který ukončí původní URL hodnotu v cookie a vkládá škodlivé atributy:

```html
"><script>print()</script>


Nebo přes uvozovky v atributu href:
```html
'/><script>print()</script>


3. Vytvoření Exploit Serveru

V PortSwigger prostředí otevři Exploit Server a do sekce Body vlož následující HTML/JS kód:
```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/product?productId=1&'><script>print()</script>" onload="this.src='https://YOUR-LAB-ID.web-security-academy.net/'"></iframe>


# Vysvětlení proč to fungovalo?

iframe nejprve načte stránku produktu s payloadem v URL.

Klientský skript na stránce produktu přečte tuto URL 
```javascript
 (window.location.href) a zapíše ji do document.cookie.

Událost onload následně přesměruje iframe na hlavní stránku (/).

Hlavní stránka přečte nebezpečnou hodnotu z document.cookie a vykreslí ji do DOMu.

Zapsaný 
```html
<script>print()</script> se vykoná a vyvolá okno tisku.

4. Doručení oběti
 (Deliver to Victim)

Vyzkoušej tlačítko View exploit pro ověření, zda se ti v prohlížeči objeví tiskové okno.

Klikni na Deliver exploit to victim.

Počkej na potvrzení o úspěšném vyřešení laboratoře 
(Congratulations, you solved the lab!).

🛡️ Remediace (Jak zranitelnosti zabránit)

Evikce nebezpečného klientského zápisu: Vyhýbejte se přímému zápisu neověřených vstupů z window.location do document.cookie.

Context-aware Encoding: Pokud musíte číst hodnoty z cookie a vkládat je do DOMu, použijte bezpečné API jako textContent místo innerHTML nebo nebezpečné manipulace s href.

Strict Cookie Attributes: Používejte příznaky HttpOnly u citlivých cookies, aby k nim JavaScript na straně klienta neměl vůbec přístup.