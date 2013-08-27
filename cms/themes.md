# Tvorba témat, layoutů a šablon

- [Adresářová struktura layoutů](#adresářová-struktura-layoutů)
- [Šablona layoutu](#Šablona-layoutu)
- [Šablona presenteru](#Šablona-presenteru)
- [Šablona komponent](#Šablona-komponent)

---

Téma stránky chápeme jako vzhled pro konkrétní web. Může obsahovat definice několika layoutů a každý layout může mít šablony pro přetížení šablon presenterů nebo komponent.

Přetěžování šablon znamená, že se k vykreslení nepoužije výchozí šablona, ale vlastní - upravená pro konkrétní layout.

Pro každý nový projekt je doporučeno vytvořit nový modul, který bude obsahovat vše, co s projektem souvisí. V tomto případě ho využijeme na vlastní téma. Jak vytvořit nový modul se dočtete v sekci [Definice modulu](/framework/modules.md).



## Adresářová struktura layoutů

V prvé řadě je třeba si uvědomit, že i layouty a šablony mohou vyžadovat funkce z ostatních modulů. Mohou tedy být závislé na ostatních modulech. Proto jsou témata přímo součástí modulů a nikoliv sandboxu.

Šablony layoutů, presenterů a komponent se nacházejí na cestě `Resources/layouts`.

- Resources/layouts
	- main
		- @layout.latte
	- default
		- @layout.latte
		- NavigationControl.latte
		- Blog.List.default.latte
	- LoginControl.latte
	- @layout.latte
	- @sitemap.latte



## Šablona layoutu

Tato šablona obsahuje základní kostru webové stránky - layout. Jméno layoutu je určeno jménem adresáře, ve kterém je umístěn. Například layout `main` bude definován v souboru `Resources/layouts/main/@layout.latte`. Jedna webová stránka může využívat hned několik layoutů, které budou obsahovat společný základ (hlavičku, patičku,...) a lišit se budou jen v uspořádání vnitřní části. V tomto případě je výhodné použít víceúrovňový layout a společnou kostru webu definovat v souboru `Resources/layouts/@layout.latte`.


### Konkrétní ukázka řešení výceúrovňového layoutu

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
	{include #content}
	{block #contentBottom}{/block}

	<hr />

	<small class="muted">
		{control itemInfo $presenter->route}
	</small>
</div>
```

### Konvence

1. Textový obsah se vkládá pomocí `{include #content}`.
2. Víceúrovňový layout se řeší pomocí bloku `#layout`.
3. Hlavní nadpis stránky se vkládá do bloku `#title`.
4. Hlavička webu obsahuje blok `#head`.



## Šablona presenteru

Šablony presenterů jsou standardně umístěny vedle presenterů, nejčastěji na relativní adrese `./templates`. Pokud chceme docílit toho, aby se v určitém layoutu použila jiná šablona presenteru, stačí vedle souboru layoutu vytvořit soubor ve formátu `<$presenter>.<$action>.latte`, tedy konkrétně třeba `Resources/layouts/main/Cms.Pages.Text.Route.default.latte`. Jestliže bychom chtěli upravit šablonu presenteru pro všechny layouty v jednom modulu, cesta by byla o úroveň výše, tedy `Resources/layouts/Cms.Pages.Text.Route.default.latte`.

Šablona presenteru se detekuje dle existence cílového souboru v tomto pořadí.

- `<$layoutPath>/<$module>.<$presenter>.<$action>.latte`
- `<$layoutPath>/../<$module>.<$presenter>.<$action>.latte`
- `./templates/<$module>.<$presenter>/<$action>.latte`
- `./templates/<$module>.<$presenter>.<$action>.latte`


### Rozdělení šablony presenterů do bloků

Jestliže celá šablona presenteru je tvořena jedním blokem (se jménem `#content`), přináší to jednu nevýhodu. Když bude potřeba takovou šablonu upravit přetížením, musí se celý blok předefinovat znovu. Z toho důvodu se vyplatí šablonu presenteru rozebrat na více bloků, ty uložit do zvláštního souboru a tento soubor v šabloně presenteru načítat a vykreslit jeho hlavní blok. Následující ukázka zobrazuje, jak by řešení mohlo vypadat.

Šablona presenteru:

```php
{block content}

{includeblock @blogModule/@page.route.latte}
{include #page-content}
```

Šablona načte bloky z externího souboru a následně vykreslí blok `#page-content`. Je dobré všechny bloky pojmenovávat s prefixem `page-`, aby nedošlo ke kolizi s bloky v šabloně. Výchozí blok pak jménem `#page-content`.

Šablona s definicemi bloků by mohla vypadat například takto (`Resources/layouts/@page.route.latte`):

```php
{define #page-content}
	{include #page-items items => $control->getItemsBuilder()->orderBy('r.released', 'DESC')->orderBy('r.created', 'DESC')->getQuery()->getResult()}
{/define}

{define #page-items}
	<ul n:inner-foreach="$items as $item">
		{include #page-items-item item => $item}
	</ul>
{/define}

{define #page-items-item}
	<li>{$item->name}</li>
{/define}
```

Nyní, kdyby bylo zapotřebí upravit šablonu presenteru - například přidáním tříd k elementu `li`, vytvořil by se soubor `Resources/<$layout>/Blog.Pages.Blog.Route.default.latte` s obsahem:

```php
{block content}
	{includeblock @blogModule/@page.route.latte}
	{include #page-content}
{/block}

{define #page-items-item}
	<li class="{$item->class}">{$item->name}</li>
{/define}
```



## Šablona komponent

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


### Varianty šablon

Vzhledem k tomu, že komponenta může být na rozdíl od presenteru na stránce vykreslena několikrát, můžeme se setkat s požadavkem, aby stejná komponenta byla na stránce vykreslena pokaždé s jinou šablonou. Toho lze dosáhnout snadno. Třída `CmsModule\Content\Control` nabízí nastavení `varianty` zobrazení pomocí následující syntaxe:

```php
{control navigation config => [variant => 'foo']}
```

Šablona se dohledává dle výše zmíněného pořadí.
