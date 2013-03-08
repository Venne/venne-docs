# Tvorba témat, layoutů a šablon
Téma stránky chápeme jako vzhled pro konkrétní web. Může obsahovat definice několika layoutů a každý layout může mít šablony pro přetížení šablon presenterů.


## Konvence
### Hlavní obsah stránky (block #content)
Hlavní obsah stránky se nachází v bloku `content`.
#### Příklad

	...
	<div id="content">
		{block #content}
	</div>
	...

### AJAX
Ajaxové odkazy a další tagy, které volají ajaxové požadavky označujeme `class="ajax"`.

### Lightbox
Odkazy, které vyvolávají popup okénka označujeme `rel="lightbox"`.


## Téma jako modul
V prvé řadě je třeba si uvědomit, že i layouty a šablony mohou vyžadovat funkčnost z ostatních modulů. Mohou tedy být závislé na ostatních modulech. Proto jsou témata přímo součástí modulů. 


## Vytvoření tématu
Pro každý nový projekt je doporučeno vytvořit nový modul, který bude obsahovat vše, co s projektem souvisí. V tomto případě ho využijeme na vlastní téma. Jak vytvořit nový modul se dočtete v sekci [Definice modulu](modules-howto).

Nyní máme modul připraven. Přidáme do něj tyto adresáře:

* **`/Resources/layouts`:** zde budeme přidávat nové layouty
* **`/Resources/public`:** adresář slouží pro materiály k HTML. Zde budeme ukládat css a js soubory, obrázky,...


## Tvorba layoutů
Layouty se nacházejí v adresáři `/Resources/layouts` a jeden layout je představován adresářem se stejným jménem a souborem `@layout.latte`, který se v adresáři nachází. (ukázka struktury níže)

Každý layout má možnost ovlivňovat šablony jiných modulů. Lze tak ovlivnit HTML kód jiného modulu.

### Ukázka adresářové struktury modulu s hlavní stránkou a stránkou pro text 

* layouts
	* main
		* @layout.latte
	* default
		* @layout.latte
		* NavigationControl.latte
		* Blog
			* List.default.latte
	* LoginControl.latte

Defaultní layout navíc přetěžuje šablonu pro modul blog a jeho výpis příspěvků a control pro navigaci.

### Ukázka jednoduchého layoutu

	<!DOCTYPE html>
	<html>
	{head}
		{js @CoreModule/js/jquery/jquery-1.6.4.min.js}
		{js @CoreModule/js/jquery/jquery-ui-1.8.16.custom.min.js}
		{js @CoreModule/js/jquery.nette.js}
		{js @CoreModule/js/netteForms.js}
		{js @CoreModule/js/jquery.ajaxform.js}
		{js @CoreModule/js/venne.js}
		{css @CoreModule/css/jquery-ui-1.8.16.custom.css, media=>'screen, projection, tv'}
		
		{css @ProjectModule/venne/css/style.css, media=>'screen, projection, tv'}
		{block #head}{/block}
	{/head}
	{body}

		<div id="content">
			<h1>{include #title}</h1>

			{control flashMessage}
			{include #content}
		</div>

	{/body}
	</html>

### Povinné bloky

Každá šablona musí povinně definovat několik bloků, které pak můžeme využít v layoutech. Jedná se o tyto bloky:

* **{block #title}{/block}** - blok pro hlavní nadpis stránky
* **{block #content}{/content}** - blok pro hlavní obsah stránky

### Volitelné bloky

Navíc je možné využít i volitelné bloky

* **{block #head}{/block}** - blok v části hlavičky. Díky němu můžeme do hlavičky propagovat další kód.


### Dvouúrovňový layout

V rozsáhlejších webových aplikací budeme jistě potřebovat několik obdobných šablon lišících se jen v několik drobnostech. Abychom zbytečně neduplikovali šablonový kód, rozdělíme si šablonu do více vrstev. Adresářová struktura může být následující:

* layouts
	* main
		* @layout.latte
	* default
		* @layout.latte
	* @layout.latte

V souboru `layouts/@layout.latte` budeme mít kostru webu a v níže zanořených layoutech pak jen úpravy. Podoba souborů může být například tato:

#### `layouts/@layout.latte`

	<!DOCTYPE html>
	<html>
	{head}
		{css @ProjectModule/venne/css/style.css, media=>'screen, projection, tv'}
		{block #head}{/block}
	{/head}
	{body}

		<h1>{include #title}</h1>

		<div id="content">
			{include #layout}
		</div>

	{/body}
	</html>

#### `layouts/default/@layout.latte`

	{extends ../@layout.latte}

	{block #layout}

	{include #content}

#### `layouts/main/@layout.latte`

	{extends ../@layout.latte}

	{block #layout}

	<div class="container">
		<div class="row">
			<div class="span6">
				{include #content}
			</div>
			<div class="span6">
				{element textarea}
			</div>
		</div>
	</div>
