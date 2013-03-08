# Konfigurace projektu (configuration)

Ve Venne frameworku je možné používat různé konfigurace pro různá prostředí. Ve výchozím stavu se jméno prostředí odvozuje z proměnné `$_SERVER['SERVER_NAME']`.

## Typy konfiguračních souborů

Více typů konfiguračních souborů vyplynulo z nutnosti konfigurace rozdílných vrstev aplikace.

### Konfigurace sandboxu `sandbox.php`

V první části je nutné definovat cesty k adresářům. Ve výchozím sandboxu se tato konfigurace nachází `/app/sandbox.php`. Díky takto oddělené konfiguraci můžeme na jedné instanci frameworku provozovat hned několik webů obdobného charakteru. Každý web potom využije vlastní `sandbox.php` s jinými cestami. PHP syntaxe se využívá z důvodu co nejrychlejší interpretace.

Spuštění aplikace se provádí následujícím kódem:

	$configurator = new \Venne\Config\Configurator(dirname(__DIR__) . '/app', $loader);
	$configurator->enableDebugger();
	$configurator->enableLoader();
	$configurator->getContainer()->application->run();

První parametr v konstruktoru třídy `Configurator` udává cestu do adresáře, kde se nachází konfigurační soubor `sandbox.php`.


### Konfigurace prostředí `%configDir%/settings.php`

Konfigurace opět využívá syntaxi PHP. Využívá se primárně pro definici modulů a pro nastavení debuggeru.

#### Výčet souborů

* `%configDir%/settings.php` - povinný, obsahuje konfiguraci pro všechny typy prostředí
* `%configDir%/settings_{$environment}.php` - nepovinný, slouží pro konfiguraci konkrétního prostředí

#### Nastavení debug módu

Není-li uvedeno jinak, debug mód se detekuje automaticky. Pokuď z nějakého důvodu chceme napevno uvést mód, doplníme do konfigurace následující:

	'debugMode' => false, // případně true

Framework navíc poskytuje zajímavou možnost povolit debug mód jen po http authentifikaci, což se může hodit pro ladění v provozu.

	'debugModeLogin' => array(
		'name' => 'user',
		'password' => '1234',
	),

Aktivace debug módu se provede zasláním `$_GET['debugMode']` argumentu. Tedy například `http://mycompany.com/?debugMode=1`.


### Konfigurace aplikace `%configDir%/config.neon`

Jedná se o stejný konfigurační soubor na jaký jsme zvyklí z Nette.

#### Výčet souborů

* `%configDir%/config.neon` - povinný, obsahuje konfiguraci pro všechny typy prostředí
* `%configDir%/config_{$environment}.neon` - nepovinný, slouží pro konfiguraci konkrétního prostředí


