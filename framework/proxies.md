# Automaticky generované proxy třídy (proxies)

Využití automaticky generovaných proxy tříd můžeme ocenit například v momentě, kdy potřebujeme použít autowiring dvou odlišných služeb jedné třídy. Typicky například více instancí obecného repozitáře v Doctrine.

## Použití ve frameworku
	
	services:
		cms.layoutRepository:
			class: CmsModule\Content\Repositories\LayoutRepository
			factory: @entityManager::getRepository('CmsModule\Content\Entities\LayoutEntity')
			tags: [proxy: DoctrineModule\Repositories\BaseRepository]

Předchozí ukázka zaregistruje do DIC službu s názvem `@cms.layoutRepository`. Díky tagu `tags` se třída `CmsModule\Content\Repositories\LayoutRepository` generuje automaticky. Vytvořená třída dědí z třídy uvedené v argumentu tagu `proxy`. Nyní lze v presenteru bez problémů využít autowiring.