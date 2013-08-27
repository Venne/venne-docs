# Třída `Venne\Forms\Form`

Tato třída rozšiřuje Nette formulář o několik funkcí

## Datový objekt `data`

Mimo klasických metod `setValues()` a `getValues()` tu zde máme i `setData()` a `getData()`. To nám umožňuje přiřazovat k formuláři objekt, který bude formulář editovat. Využití najdeme zejména u entit. Formulář umí editovat ty data, na které máme ovladač - `mapper` (viz `mappers`). 

## Mapper

Jedná se o ovladač, který edituje datový objekt. Pro vytvoření vlastního mapperu stačí implementovat rozhraní `Venne\Forms\IMapper`.

## Události

- `onSuccess` - odeslání je validní
- `onError` - odeslání je chybné
- `onValidate` - při validaci
- `onAttached` - při připojení k předkovi
- `onSave` - při ukládání do datového objektu (viz `data`), provádí se pouze tehdy, pokud je formulář validní, navíc může formulář ještě invalidovat
- `onLoad` - při načítání z datového objektu (viz `data`)

## Rozšíření o další typy inputů

Abychom dostali do formuláře své vlastní nové typy inputu, stačí jednoduše implementovat rozhraní `Venne\Forms\IControlExtension`.

### Ukázka

	<?php
	class DoctrineExtension extends Object implements IControlExtension
	{

		protected $form;

		public function setForm(Form $form)
		{
			$this->form = $form;
		}

		public function getControls()
		{
			return array(
				'one', 'two'
			);
		}

		public function addOne($name)
		{
			return $this->form[$name] = new MyOne;
		}


		public function addTwo($name, $containerFactory, $entityFactory = NULL)
		{
			return $this->form[$name] = new MyTwo($containerFactory, $entityFactory);
		}
	}

# Tvorba znovupoužitelných formulářů

## Třída `Venne\Forms\FormFactory`

Usnadňuje nám vytváření znovupoužitelných továrniček.

### Ukázka

	<?php

	namespace CmsModule\Content\Forms;
	
	use Venne;
	use Venne\Forms\FormFactory;
	use Venne\Application\UI\Form;
	use DoctrineModule\Forms\Mappers\EntityMapper;
	use DoctrineModule\ORM\BaseRepository;

	class RoutesFormFactory extends FormFactory
	{

		protected $mapper;
	
		protected $repository;

		public function __construct(EntityMapper $mapper, BaseRepository $repository)
		{
			$this->mapper = $mapper;
			$this->repository = $repository;
		}
		
		protected function getMapper()
		{
			return $this->mapper;
		}
		
		protected function getControlExtensions()
		{
			return array(
				new \DoctrineModule\Forms\ControlExtensions\DoctrineExtension(),
			);
		}
	
		protected function configure(Form $form)
		{
			$form->addMany('routes', function(\Nette\Forms\Container $container)
			{
				$container->setCurrentGroup($container->getForm()->addGroup('Route' . $container->data->url));
				$container->addText('title', 'Title');
				$container->addText('keywords', 'Keywords');
				$container->addText('description', 'Description');
				$container->addText('author', 'Author');
				$container->addSelect('robots', 'Robots', \CmsModule\Content\Entities\RouteEntity::$robotsValues);
				$container->addCheckbox('copyLayoutFromParent', 'Layout from parent');
			});
			$form->setCurrentGroup();
			$form->addSubmit('_submit', 'Save');
		}
	
		public function handleSave(Form $form)
		{
			try {
				$this->repository->save($form->data);
			} catch (\DoctrineModule\ORM\SqlException $e) {
				if ($e->getCode() == 23000) {
					$form->addError("Role is not unique");
				} else {
					throw $e;
				}
			} catch (\Nette\InvalidArgumentException $e) {
				$form->addError($e->getMessage());
			}
		}
	}

### Metody

- `__construct()` - použijeme pro podání požadovaných služeb
- `injectFactory()` - nastavujeme zvenčí. Slouží pro podání továrny výchozího formuláře. V Nette bychom nastavovali službu `@nette.basicForm`.
- `getMapper()` - definuje mapper pro formulář. Lze vynechat.
- `getControlExtensions()` - definuje třídy implementující `IControlExtension` pro nové formulářové prvky. Lze vynechat.
- `configure(Form $form)` - konfiguruje podobu formuláře.
- `handleSuccess($form)`
- `handleError($form)`
- `handleValidate($form)`
- `handleSave($form)`
- `handleLoad($form)`
- `handleAttached($form)`

### Registrace do DIC

	services:
		cms.websiteFormFactory:
			class: CmsModule\Forms\WebsiteFormFactory
			setup:
				- injectFactory(@nette.basicFormFactory)

## Použití v presenteru

	class LanguagePresenter extends BasePresenter
	{
		
		...
		protected $form;

		public function injectForm(\CmsModule\Forms\LanguageFormFactory $form)
		{
			$this->form = $form;
		}
	
		public function createComponentForm()
		{
			$repository = $this->languageRepository;
			$entity = $repository->createNew();
	
			$form = $this->form->invoke($entity);
			$form->onSuccess[] = $this->formSuccess;
			return $form;
		}
	
		public function formSuccess()
		{
			$this->flashMessage('Entity has been saved.', 'success');
	
			if (!$this->isAjax()) {
				$this->redirect('this');
			}
		}
	} 
