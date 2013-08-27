# Presentery



## Ukázka presenteru

```php
namespace CmsModule\Administration\Presenters;

class ExamplePresenter extends BasePresenter
{

	/** @var ExampleFormFactory */
	protected $formFactory;

	public function injectFormFactory(ExampleFormFactory $formFactory)
	{
		$this->formFactory = $formFactory;
	}
	...

}
```



## Registrace do frameworku

ukázka nastavení pro cestu `Core:Admin:Example`

	factories:
	
		core.admin.examplePresenter:
			class: CmsModule\Administration\Presenters\ExamplePresenter
			tags: [presenter] 
