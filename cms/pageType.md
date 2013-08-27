# Tvorba vlastních typů front-end stránek

- [Minimální podoba entit](#minimální-podoba-entit)
- [Příprava presenterů](#příprava-presenterů)
- [Příprava administračních sekcí](#příprava-administračních-sekcí)
- [Registrace do CMS](#registrace-do-cms)
- [Řízení oprávnění](#Řízení-oprávnění)
- [Speciální stránky](#speciální-stránky)

---

Venne:CMS umožňuje vytvářet vlastní typy stránek. Typem může být například jednoduchá statická stránka textu s jednou URL adresou, ale i složitý rezervační systém rozkládající se přes desítky URL adres. Každý typ stránky se navíc může na webu vyskytovat hned několikrát (až na některé výjimky jako třeba chybové stránky).

Modelová část typu stránky se skládá ze dvou základních entit. První dědící z `CmsModule\Content\Entities\ExtendedPageEntity` reprezentuje data typu stránky. Například u blogu by bylo dobré poznamenat si počet příspěvků, které se mají zobrazit ve výpisu. Dále pak bude zapotřebí vytvořit minimálně jednoho potomka třídy `CmsModule\Content\Entities\ExtendedRouteEntity`. Tato entita obsahuje data pro jednu konkrétní URL, která spadají pod celý typ stránky. Složitější typy stránek mohou mít nadefinováno hned několik typů rout. Jednoduchý blog bude mít jeden typ routy pro výpis a druhý typ pro jednotlivé články.

Z předchozího odstavce je zřejmé, že instance typu stránky je tvořena právě jednou instancí `ExtendedPageEntity` a nejméně jednou instancí třídy `ExtendedRouteEntity`. Platí zde vztah `oneToMany` - jedna stránka může mít několik rout. Aby bylo možné určit, která routa ve stránce je hlavní, entity obsahují ještě jednu vazbu `mainExtendedRoute` a to typu `oneToOne`.

Možná už někoho napadlo, proč entita stránky i routy je označovaná jako `rozšířená`. Je to proto, že ve skutečnosti mají obě entity uvnitř sebe schovány ještě jednu entitu. `ExtendedRouteEntity` v sobě obsahuje `RouteEntity` a `ExtendedPageEntity` má `PageEntity`. Tyto "pomocné" entity jsou pro celý systém společné a slouží k uchování dat, které jsou potřeba pro veškeré typy stránek. O jejich inicializaci se není nutné starat, provede se sama. Toto řešení supluje chování dědičnosti v Doctrime ORM, ovšem na rozdíl od klasického postupu přetěžování entit se zde nemnoží počet JOINů v SQL dotazu. Při větším množství typů stránek by razantně upadal výkon aplikace.

Je dobrým zvykem nové typy stránek soustředit do namespace `<$moduleName>Module\Pages\<$pageType>`;



## Minimální podoba entit

```php
use CmsModule\Content\Entities\ExtendedPageEntity;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity(repositoryClass="\CmsModule\Content\Repositories\PageRepository")
 * @ORM\Table(name="myPage")
 */
class PageEntity extends ExtendedPageEntity
{

}

use CmsModule\Content\Entities\ExtendedRouteEntity;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity(repositoryClass="\DoctrineModule\Repositories\BaseRepository")
 * @ORM\Table(name="myRoute")
 */
class RouteEntity extends ExtendedRouteEntity
{

}
```

Aby systém rozpoznal, jaká routa patří k jaké stránce, existuje v každé routě metoda `getPageName()` (ve stránce `getMainRouteName()`), která vrací název vazební třídy. Pokud jsou dodrženy konvence pojmenovávání rout a stránek, tyto metody lze ignorovat.


### Konvence pro pojmenovávání entit

1. Entita stránky: `PageEntity`.
2. Entita hlavní routy stránky: `RouteEntity`.
3. Ostatní routy: `<$name>Entity`.
4. Společný namespace.

V případě upřednostnění jiné konvence je potřeba přetížit metody `getPageName()` a `getMainRouteName()`.



## Příprava presenterů

Každá routa (`RouteExtendedEntity`) definuje, pomocí kterého presenteru se dané URL zobrazí. Konkrétně jméno presenteru definuje metoda `getPresenterName()`. Ta ve výchozí implementaci vrací presenter s korespondujícím jménem:

- `CmsModule\Pages\Tags\TagEntity` => `Cms:Pages:Tags:Tag:default`
- `GuestbookModule\Pages\Guestbook\RouteEntity` => `Guestbook:Pages:Guestbook:Route:default`

Presentery musejí dědit ze třídy `CmsModule\Content\Presenters\PagePresenter`. Pro stránky s výpisem položek lze využít jako předka připravený presenter `CmsModule\Content\Presenters\ItemsPresenter`.

```php
use CmsModule\Content\Presenters\PagePresenter;

class RoutePresenter extends PagePresenter
{

}
```

V presenteru existuje několik důležitých metod pro přístup k entitám:

```php
$presenter->extendedPage;	// entita stránky
$presenter->page;			// základní entita stránky
$presenter->extendedRoute;	// entita routy
$presenter->route;			// základní entita routy
$presenter->language;		// entita jazyka, může obsahovat NULL = pro všechny jazyky
```

Přepnutí jazyka se provede následujícím způsobem:

```php
$presenter->changeLanguage('en');
```

Pokud aktuální stránka v novém jazyce neexistuje, metoda se pokusí vyhledat nejbližší stránku na vyšší URL.




### Šablona presenteru

Aby šablony presenterů byly snadno modifikovatelné pomocí jiných modulů, je doporučeno oddělit definice jednotlivých bloků do samostatného souboru. Bloky stránky pojmenováváme s prefixem `page-` a hlavní blok stránky pojmenováváme `page-content`. Šablona pro stránku s přihlášením může vypadat následovně:

```php
{block content}

{includeblock @cmsModule/@login.latte}
{include #page-content}
```

Šablona pouze načte bloky z externího souboru `@cmsModule/@login.latte` a poté zobrazí hlavní blok. Podoba externího souboru s bloky je následující:

```php
{define #page-content}

	{if !$presenter->user->isLoggedIn()}
		{control form}
	{/if}

{/define}
```

V této ukázce se definuje pouze jeden blok (`#page-content`), nic však nebrání využít více bloku, pokud si o to komplexnost stránky žádá.


### Užitečné widgety

V šabloně presenteru lze často využít několik widgetů, které ulehčí vykreslení informací o routách.

- `{control item [$route]}` - Vykreslí textový obsah routy.
- `{control itemList $routes}` - vykreslí seznam rout. Vhodné pro výpis se stránkováním.
- `{control itemInfo [$route]}` - vykreslí informační lištu pro danou routu.
- `{control author [$route]}` - vykreslí informace o autorovi dané routy.

Vykreslení stránky uživatele se provede takto:

```php
{control item $presenter->getUser()->identity}
```



## Příprava administračních sekcí

Každá instance typu stránky má svoji administrační část. Ta je rozdělena do záložek - sekcí. Každá sekce je reprezentována komponentou, která má za úkol editovat nějaká data stránky. Touto komponentou může být jak jednoduchý formulář, tak i složitý editační grid nebo jiná komponenta dědící ze třídy `CmsModule\Content\SectionControl`.

Pokud je jako komponenta zvolen formulář, jako data obdrží přímo instanci třídy `ExtendedPageEntity`. Formulář může vypadat například takto:

```php
use DoctrineModule\Forms\FormFactory;
use Venne\Forms\Form;

class MenuFormFactory extends FormFactory
{
	public function configure(Form $form)
	{
		$form->addGroup('Settings');
		$form->addText('itemsPerPage', 'Items per page');

		$form->addGroup();
		$form->addSaveButton('Save');
	}
}
```

Jestliže bude komponenta sekce potomkem `CmsModule\Content\SectionControl`, k entitě stránky lze přistoupit pomocí metody `getEntity()`.

```php
$sectionControl->entity;	// entita editované stránky
```



## Registrace do CMS

	services:

		example.pages.exampleContent:
			class: CmsModule\Content\ContentType('ExampleModule\Pages\Example\PageEntity')
			setup:
				- addSection('Content', @example.pageFormFactory)
			tags: [contentType: 'example page']


### Definice sekcí

Při editaci stránky v administraci může být správa rozdělena do záložek - sekcí. Registrace sekce se provádí pomocí `addSection(<Název sekce>, <továrnička komponenty>)`. Továrna může být pro formulář jako v tomto případě, nebo i pro jakoukoliv jinou komponentu.



## Řízení oprávnění

Některé weby dovolují mazat příspěvky v diskuzích jejich autorům, jiné tuto pravomoc udělují pouze administrátorům. Venne:CMS nabízí jednoduchou cestu, jak přenechat nastavení oprávnění autorovi webu, aniž by musel zasahovat do kódu aplikace. Totéž platí i pro administrační část.


### definice privilegií

V první řadě je třeba nadefinovat `privilegie` v entitě `ExtendedPageEntity`. Metody k tomu určené mohou vypadat následovně:

```php
	...
	public function getPrivileges()
	{
		return parent::getPrivileges() + array(
			self::PRIVILEGE_EDIT_OWN => 'edit own comments',
			self::PRIVILEGE_DELETE_OWN => 'delete own comments',
			self::PRIVILEGE_EDIT => 'edit comments from all authors',
			self::PRIVILEGE_DELETE => 'delete comments from all authors',
		);
	}

	/**
     * @return array
     */
    public function getAdminPrivileges()
    {
		return parent::getAdminPrivileges() + array(
			...
		);
	}
    ...
```

V administraci stránky se automaticky vygeneruje formulář pro zadávání práv podle výše uvedených hodnot.


### Kontrola oprávnění

V presenteru můžeme oprávnění kontrolovat pomocí metody `$presenter->isAllowed(...)`.

Obdobně v šabloně lze využít `{if $presenter->isAllowed(...)}...{/if}`.



## Speciální stránky

Jeden typ speciální stránky se může na celém webu vyskytovat maximálně jednou. Speciální stránkou může být například stránka pro přihlášení, stránka profilu přihlášeného uživatele,... . Aby nový typ stránky byl brán jako speciální, musí entita `ExtendedPageEntity` obsahovat metodu `getSpecial()` vracející název speciální stránky.

```php
	protected function getSpecial()
	{
		return 'login';
	}
```

Velkou výhodou speciálních stránek je, že se na ně dobře odkazuje.

```php
$this->redirect('Route', array('special' => 'login'));
```

```php
{link Route special => 'login'}
```

Jestliže chceme otestovat, zda-li existuje instance speciální stránky, můžeme otestovat odkaz pomocí makra `ifLinkExists`.

```php
<li n:ifLinkExists="Route special => 'login'"><a n:link>{_'Sign in'}</a></li>
```
