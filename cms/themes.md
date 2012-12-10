# Tvorba témat
Téma stránky chápeme jako vzhled pro konkrétní web. Může obsahovat definice několika layoutů a každý layout může mít šablony pro přetížení šablon presenterů.


## Konvence
### Hlavní obsah stránky (block #content)
Hlavní obsah stránky je nutné vložit do `<div id="content"> ... </div>`
#### Příklad

	...
	<div id="content">
		{content}
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

* **`/layouts`:** zde budeme přidávat nové layouty
* **`/Resources/public`:** adresář slouží pro materiály k HTML. Zde budeme ukládat css a js soubory, obrázky,...


## Tvorba layoutů
Layouty se nacházejí v adresáři `/layouts` a jeden layout je představován adresářem se stejným jménem a souborem `@layout.latte`, který se v adresáři nachází. (ukázka struktury níže)

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
		{block head}{/block}
	{/head}
	{body}

		<div id="content">
			{content}
		</div>

	{/body}
	</html>