# Automaticky generované proxy třídy (proxies)

Využití automaticky generovaných proxy tříd můžeme ocenit například v momentě, kdy potřebujeme použít autowiring dvou odlišných služeb jedné třídy. Typicky například více instancí obecného repozitáře v Doctrine.

## Použití ve frameworku
	
	services:
		cms.layoutRepository:
			class: CmsModule\Content\Repositories\LayoutRepository
			factory: @entityManager::getRepository('CmsModule\Content\Entities\LayoutEntity')
			tags: [proxy: DoctrineModule\Repositories\BaseRepository]

Předchozí ukázka zaregistruje do DIC službu s názvem `@cms.layoutRepository`. Třída `CmsModule\Content\Repositories\LayoutRepository` v našem projektu neexistuje. Díky tagu `proxy` je však automaticky vygenerována a poděděna ze třídy `BaseRepository`. Výhodou je, že jsme nemuseli napsat ani řádku kódu a zároveň jsme nepřišli o autowiring, který lze využít následujícím způsobem:

		...
		public function __construct(LayoutRepository $layoutRepository)
		{
			parent::__construct();
			$this->layoutRepository = $layoutRepository;
		}
		...

Aby se takto vygenerovaná třída napovídala ve vašem oblíbeném IDE, je třeba přegenerovat DIC, což se ve vývojovém prostředí provede automaticky po dalším požadavku na libovolnou stránku. V produkci je třeba navíc promazat cache.