# Tvorba elementů

## Interface `CmsModule\Content\IElement`

Každý element musí implementovat rozhraní z `IElement`.

## Třída `CmsModule\Content\Elements\BaseElement`

Pro snadnější užití můžeme použít už předpřipravenou třídu `BaseElement`. 

### Ukázka

	class TextElement extends BaseElement
	{
	
		public function render()
		{
			echo 'Hello world!!';
		}
	
	
		public function renderSetup()
		{
			echo 'There is nothing to configure.';
		}
	
	}

#### Metody

- `render()` - volá se při renderování elementu na frontend
- `renderSetup()` - volá se při renderování nastavení elementu. 

## Registrace do CMS

	factories:
		foo.barElement:
			class: FooModule\Elements\BarElement
			tags: [element: foo]

## Použití v šablonách

### Makro `{element $name $id = NULL}`

	{element foo}
	{element foo 10}   

#### Argumenty
- `name` - povinný, jméno elementu k zobrazení
- `id` - integer, nepovinný, v případě vynechání se použije inkrementovaná hodnota od 0.

#### Zobrazení nastavení přímo v šabloně

	{element foo:setup}   

 
