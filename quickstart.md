# Quickstart

Rychlý úvod do redakčního systému Venne:CMS.



## Instalace

### Krok 1 - Stažení

Z Githubu stáhneme aktuální verzi CMS. K dostání jsou dva druhy sestavení - klasické a ws (without symlinks). Varianta ws se hodí pro souborové systémy a hostingy, které nepodporují symlinky. Například hostingy s FTP apod. Pokud se symlinky nemáte problém, rozhodně využijte klasické sestavení.

Připravené balíčky: [https://github.com/Venne/cms-module/releases]

Jestliže přeferujete ruční sestavení, spusťte následující příkazy:

```bash
composer create-project venne/sandbox:2.0.x-dev myApp && cd myApp
composer require venne/cms-module:2.1.x [--prefer-dist]
php www/index.php venne:module:update
php www/index.php venne:module:install cms [--noconfirm]
```


### Krok 2 - Instalace

Redakční systém spusťte v prohlížeči. Systém automaticky nastartuje instalační proces.


### Krok 3 - Aktivate dalších modulů

Předem sestavené balíčky už obsahují škálu modulů, máte je rovnou připraveny k aktivaci. Pokud jste instalovali projekt pomocí composeru, musíte další moduly stáhnout:

```bash
composer require venne/sample-module:2.0.x [--prefer-dist]      # Download module
php www/index.php venne:module:update                           # Update local database of modules
```

Obdobně budete postupovat při instalaci vlastních modulů, které si pro CMS vytvoříte.

V tuto chvíli máte moduly staženy a připojeny do systému. Chybí už jen modul nainstalovat:

```php
php www/index.php venne:module:install sample                   # Install module
```

Poslední krok můžete provést přímo v administraci CMS v sekci "Správa modulů".



## Tvorba témat, layoutů a šablon

Téma stránky chápeme jako vzhled pro konkrétní web. Může obsahovat definice několika layoutů a každý layout může mít šablony pro přetížení šablon presenterů nebo komponent.

Přetěžování šablon znamená, že se k vykreslení nepoužije výchozí šablona, ale vlastní - upravená pro konkrétní layout.

Pro každý nový projekt je doporučeno vytvořit nový modul, který bude obsahovat vše, co s projektem souvisí. V tomto případě ho využijeme na vlastní téma. V připravených sestaveních je k tomu připraven modul `site` umístění `app/modules/site-module`.


### Adresářová struktura layoutů

V první řadě je třeba si uvědomit, že i layouty a šablony mohou vyžadovat funkce z ostatních modulů. Mohou tedy být závislé na ostatních modulech. Proto jsou témata přímo součástí modulů a nikoliv sandboxu.

Šablony layoutů, presenterů a komponent se nacházejí na cestě `<modulePath>/Resources/layouts`.

- Resources/layouts
	- main
		- @layout.latte
	- default
		- @layout.latte
		- NavigationControl.latte
		- @blog.route.latte
	- LoginControl.latte
	- @layout.latte
	- @sitemap.route.latte


### Šablona layoutu

Tato šablona obsahuje základní kostru webové stránky - layout. Jméno layoutu je určeno jménem adresáře, ve kterém je umístěn. Například layout `main` bude definován v souboru `Resources/layouts/main/@layout.latte`. Jedna webová stránka může využívat hned několik layoutů, které budou obsahovat společný základ (hlavičku, patičku,...) a lišit se budou jen v uspořádání vnitřní části. V tomto případě je výhodné použít víceúrovňový layout a společnou kostru webu definovat v souboru `Resources/layouts/@layout.latte`.


#### Konkrétní ukázka řešení výceúrovňového layoutu

Hlavní kostra layoutu (`Resources\layouts\@layout.latte`):

```php
<!DOCTYPE html>
<html>
{head}
...
{/head}

{body}
	{control breadcrumb}

	<h1>{block #title}{$presenter->route->name|firstUpper}{/block}</h1>

	<div class="row">
		{include #layout}
	</div>
	...
{/body}
</html>
```

Layout `main` (`Resources\layouts\main\@layout.latte`):

```php
{extends ../@layout.latte}

{block #layout}
<div class="span12">

	{control flashMessage}
	{block #contentTop}{/block}
	{block #content}{include #page-content}{/block}
	{block #contentBottom}{/block}

	<hr />

	<small class="muted">
		{control itemInfo $presenter->route}
	</small>
</div>
```


#### Konvence

1. Textový obsah se vkládá pomocí `{block #content}{include #page-content}{/block}`.
2. Víceúrovňový layout se řeší pomocí bloku `#layout`.
3. Hlavní nadpis stránky se vkládá do bloku `#title`.
4. Hlavička webu obsahuje blok `#head`.


### Šablona presenteru

Jména souborů šablon presenterů frontendových stránek mají tvar `Resources/layouts/@<type>.<presenterName>[.<view>].latte`. Tedy například `@text.route.latte`. Jestliže chceme změnit HTML kód nějaké frontend stránky, zkopírujeme soubor s originální šablonou do adresáře s tématem, případně přímo ke konkrétnímu layoutu. 


### Šablona komponent

Následující pravidla platí pro komponenty dědící ze třídy `CmsModule\Content\Control`, tedy převážně pro widgety, elementy a podobně.

Tyto třídy mají své originální šablony uloženy vedle souboru s PHP kódem. Tedy například `NavigationControl.php` má šablonu v souboru `NavigationControl.latte`.

Chceme-li ve vlastním layoutu pozměnit vzhled komponenty, opět využijeme následující priority ve vyhledávání šablony:

- `<$presenterTemplatePath>/<$controlName>Control.<$variant>.latte`
- `<$presenterTemplatePath>/<$controlName>Control.latte`
- `<$layoutPath>/<$controlName>Control.<$variant>.latte`
- `<$layoutPath>/<$controlName>Control.latte`
- `<$presenterTemplatePath>/../<$controlName>Control.<$variant>.latte`
- `<$presenterTemplatePath>/../<$controlName>Control.latte`
- `<$layoutPath>/../<$controlName>Control.<$variant>.latte`
- `<$layoutPath>/../<$controlName>Control.latte`
- `./<$class>.latte`


#### Varianty šablon

Vzhledem k tomu, že komponenta může být na rozdíl od presenteru na stránce vykreslena několikrát, můžeme se setkat s požadavkem, aby stejná komponenta byla na stránce vykreslena pokaždé s jinou šablonou. Toho lze dosáhnout snadno. Třída `CmsModule\Content\Control` nabízí nastavení `varianty` zobrazení pomocí následující syntaxe:

```php
{control navigation config => [variant => 'foo']}
```

Šablona se dohledává dle výše zmíněného pořadí.


### Předpřipravené widgety

Venne:CMS obsahuje několik předpřipravených widgetů, které lze v šablonách využít. Jestliže bychom chtěli upravit HTML kód některého widgetu, překopírujeme originální šablonu do aktuálního tématu, případně ke konkrétnímu layoutu.

```php
{control navigation 0, 2, FALSE, $rootEntity} # navigace
{control breadcrumb}                          # drobečková navigace
{control tagCloud}                            # vykreslení mraku tagů
{control flashMessage}                        # výpis zpráv
{control head}                                # HTML hlavička
{control languageswitch}                      # přepínač jazyků
{control login}                               # login form
{control search}                              # pole pro vyhledávání
{control item $presenter->route}              # vykreslení obsahu routy
{control itemInfo $presenter->route}          # vykreslení informací o routě
{control itemThumbnail $presenter->route}     # ikona routy
{control itemList $routes}                    # vykreslení seznamu rout
{control author $presenter->route}            # zobrazení autora routy
```
