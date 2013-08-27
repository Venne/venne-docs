# Tvorba elementů

- [Minimální podoba entit](#minimální-podoba-entit)
- [Příprava komponenty](#příprava-komponenty)
- [Registrace do CMS](#registrace-do-cms)

Elementy jsou speciální widgety, které se dají editovat přímo na frontendu a slouží jako konfigurovatelné miniaplikace. Elementem můžou být ankety, editovatelné pole a podobně.



## Minimální podoba entit

Obdobně jako u typů stránek se zde vyskytuje tzv. rozšířená entita (`ExtendedElementEntity`), která obsahuje společnou entitu (`ElementEntity`) společnou pro všechny elementy.

Rozšířená entita může vypadat následovně:

```php
use CmsModule\Content\Elements\ExtendedElementEntity;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity(repositoryClass="\DoctrineModule\Repositories\BaseRepository")
 * @ORM\Table(name="textElement")
 */
class TextEntity extends ExtendedElementEntity
{
	/**
	 * @ORM\Column(type="text")
	 */
	protected $text = '';

	public function setText($text)
	{
		$this->text = $text;
	}

	public function getText()
	{
		return $this->text;
	}
}
```



## Příprava komponenty

Komponenta je potomkem třídy `CmsModule\Content\Elements\BaseElement`. Jednoduchá ukázka:

```php
use CmsModule\Content\Elements\BaseElement;

class TextElement extends BaseElement
{
	public function renderDefault()
	{
		echo 'Hello world!!';
	}

	public function renderSetup()
	{
		echo 'There is nothing to configure.';
	}
}
```

#### Metody

- `renderDefault()` - volá se při renderování elementu na frontend
- `renderSetup()` - volá se při renderování nastavení elementu. 

## Registrace do CMS

	factories:
		foo.barElement:
			class: FooModule\Elements\BarElement
			tags: [element: foo]

## Použití v šablonách

### Makro `{element $name $id}`

	{element foo panel}
	{element foo 10}   

### Zobrazení nastavení přímo v šabloně

	{element foo:setup}
