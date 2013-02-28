# Tvorba modulu - guestbooku

Následující tutoriál vás seznámí s vytvářením modulů pro Venne:CMS. Bude se věnovat především tomu, jak rozšířit redakční systém o nový typ stránky.


## Definice modulu

[Podrobná dokumentace definice modulů](venne-docs/tree/master/modules/index.md)

Nejprve si připravíme adresářovou strukturu modulu. Může vypadat následovně:

	+- GuestbookModule
	+- Resources
	|   +- config
	|   |   \- config.neon
	|   +- layouts
	|   \- translations
	|       \- guestbook.cs.neon
	+- Module.php
	\- readme.md

Script `Module.php` definuje základní parametry modulu.
Adresář `GuestbookModule` bude sloužit pro zdrojové kódy. Název adresáře je odvozen od namespace modulu.
V hirearchii máme též připraven konfigurační soubor `config.neon`, který využijeme pro registraci služeb do DIC.
Adresář `Resources/translations` bude obsahovat překlady. Více o překladech v [dokumentaci překlady stringů](venne-docs/tree/master/translator.md).

Náš výsledný modul budeme distribuovat jako balíček pro composer, proto v adresářové struktuře vytvoříme ještě soubor `composer.json`.
Zároveň v souboru `Module.php` vytvoříme třídu `GuestbookModule\Module` s následujícím obsahem:

	<?php

	namespace GuestbookModule;

	use Venne;
	use Venne\Module\ComposerModule;

	class Module extends ComposerModule
	{

	}

Všimněme si, že třída dědí z `Venne\Module\ComposerModule`. Díky tomu budeme mít méně práce s definicí modulu, jelikož vše potřebné se načte ze souboru `composer.json`.

Nyní konečně nadefinujeme soubor `composer.json`.

	{
		"name": "venne/guestbook-module",
		"description": "Guestbook module",
		"keywords": ["cms", "nette", "venne", "module"],
		"version": "2.0.0",
		"authors": [
			{
				"name": "Josef Kříž",
				"homepage": "http://josef-kriz.cz"
			}
		],
		"require": {
			"php": ">=5.3.2",
			"venne/cms-module": "2.0.x-dev"
		},
		"autoload": {
			"psr-0": {
				"GuestbookModule": ""
			}
		},
		"extra": {
			"branch-alias": {
				"dev-master": "2.0.x-dev"
			},
			"venne": {
				"installers": [
					"DoctrineModule\\Composer\\Installers\\DoctrineInstaller"
				],
				"configuration": {
					"includes": [
						"%modules.guestbook.path%/Resources/config/config.neon"
					],
					"translator": {
						"dictionaries": [
							"%modules.guestbook.path%/Resources/translations"
						]
					}
				}
			}
		}
	}

Oproti běžným balíčkům zde nesmíme zapomenout na sekci `version`, `autoload` a `extra`.

Sekce `extra.venne.installers` představuje seznam instalátorů, kterými bude modul nainstalován.
Cms systém už obsahuje několik předvytvořených instalátorů, které za nás některé úkoly vyřeší automaticky.
V našem případě chceme využít instalátor, který automaticky připraví databázovou strukturu podle námi nadefinovaných entit.

Sekce `extra.venne.configuration` obsahuje strukturu dat, která se při instalaci nakopíruje do výchozího konfiguračního souboru (`app/config/config.neon`).
Můžeme si všimnout, že po instalaci modulu se do hlavního konfiguračního souboru připojí lokální konfigurační soubor a zaregistruje se adresář s překlady.

Kostru modulu máme připravenou!


## Modelová vrstva

V následující části si vytvoříme entity. Budeme potřebovat jednu entitu představující instanci stránky - guestbooku a jednu entitu reprezentující komentář.

Nejprve tedy vytvoříme entitu stránky. Umístíme ji do `GuestbookModule/Entities/PageEntity.php`. Obsah může být následující:

	<?php

	namespace GuestbookModule\Entities;

	use Venne;
	use Doctrine\ORM\Mapping as ORM;

	/**
	 * @ORM\Entity(repositoryClass="\DoctrineModule\Repositories\BaseRepository")
	 * @ORM\Table(name="guestbook_page")
	 * @ORM\DiscriminatorEntry(name="guestbookPage")
	 */
	class PageEntity extends \CmsModule\Content\Entities\PageEntity
	{

		/**
		 * @ORM\OneToMany(targetEntity="CommentEntity", mappedBy="page")
		 */
		protected $comments;

		/**
		 * @ORM\Column(type="integer")
		 */
		protected $itemsPerPage = 10;

		public function __construct()
		{
			parent::__construct();

			$this->mainRoute->type = 'Guestbook:Default:default';
		}

		public function setComments($comments)
		{
			$this->comments = $comments;
		}

		public function getComments()
		{
			return $this->comments;
		}

		public function setItemsPerPage($itemsPerPage)
		{
			$this->itemsPerPage = $itemsPerPage;
		}

		public function getItemsPerPage()
		{
			return $this->itemsPerPage;
		}
	}

Součástí entity je vazba `OneToMany` na komentáře. V konstruktoru definujeme typ routy představující jméno presenteru, který bude stránku zobrazovat.
Abychom mohli ovlivnit počet zobrazených příspěvků na stránku, vytvořili jsme si proměnnou `$itemPerPage`.

Nyní přichází na řadu entita komentáře v souboru `GuestbookModule\Entities\CommentEntity`:

	<?php

	namespace GuestbookModule\Entities;

	use Venne;
	use Doctrine\ORM\Mapping as ORM;
	use CmsModule\Security\Entities\UserEntity;

	/**
	 * @ORM\Entity(repositoryClass="\DoctrineModule\Repositories\BaseRepository")
	 * @ORM\Table(name="guestbook_comment")
	 */
	class CommentEntity extends \DoctrineModule\Entities\IdentifiedEntity
	{
		/**
		 * @ORM\Column(type="text")
		 */
		protected $text = '';

		/**
		 * @ORM\ManyToOne(targetEntity="\GuestbookModule\Entities\PageEntity", inversedBy="comments")
		 * @ORM\JoinColumn(onDelete="CASCADE")
		 */
		protected $page;

		/**
		 * @ORM\ManyToOne(targetEntity="\CmsModule\Security\Entities\UserEntity")
		 * @ORM\JoinColumn(onDelete="CASCADE")
		 */
		protected $author;

		/**
		 * @ORM\Column(type="string", nullable=true)
		 */
		protected $authorName = '';

		/**
		 * @ORM\Column(type="datetime")
		 */
		protected $created;

		/**
		 * @ORM\Column(type="datetime", nullable=true)
		 */
		protected $updated;

		public function __construct(PageEntity $pageEntity)
		{
			parent::__construct();

			$this->page = $pageEntity;
			$this->created = new \Nette\DateTime();
		}

		public function setAuthor($author)
		{
			if ($author instanceof UserEntity) {
				$this->author = $author;
				$this->authorName = NULL;
				return;
			} else if (is_string($author) && $author) {
				$this->authorName = $author;
				$this->author = NULL;
				return;
			}
		}

		public function getAuthor()
		{
			return $this->author;
		}

		public function setAuthorName($authorName)
		{
			$this->authorName = $authorName;

			if ($authorName) {
				$this->author = NULL;
			}
		}

		public function getAuthorName()
		{
			return $this->authorName;
		}

		public function setCreated($created)
		{
			$this->created = $created;
		}

		public function getCreated()
		{
			return $this->created;
		}

		public function setPage($page)
		{
			$this->page = $page;
		}

		public function getPage()
		{
			return $this->page;
		}

		public function setText($text)
		{
			$this->text = $text;
		}

		public function getText()
		{
			return $this->text;
		}

		public function setUpdated($updated)
		{
			$this->updated = $updated;
		}

		public function getUpdated()
		{
			return $this->updated;
		}
	}

V další části zaregistrujeme doctrine repozitáře. Do našeho neon souboru vložíme následující řádky:

	services:
	
		guestbook.commentRepository:
			class: GuestbookModule\Repositories\CommentRepository
			factory: @entityManager::getRepository('GuestbookModule\Entities\CommentEntity')
			tags: [proxy: DoctrineModule\Repositories\BaseRepository]

Tag `repository` zajístí vytvoření služby - repozitáře pracující s námi zadanou entitou. Repozitář pro entitu stránky vytvářet nemusíme, jelikož entita využívá dědění a lze tedy jako repozitář využít službu `@cms.pageRepository`.

Tímto bychom měli mít modelovou vrstvu připravenou k použití.


## Administrační část

V administrační části budeme určitě vyžadovat výpis jednotlivých komentářů i s jejich případnou editací. Naprogramujeme si nejprve továrnu na formulář pro editaci komentáře. Nacházet se bude na adrese `GuestbookModule/Forms`.

	<?php
	
	namespace GuestbookModule\Forms;
	
	use Venne;
	use Venne\Forms\Form;
	use Nette\Security\User;
	use DoctrineModule\Forms\FormFactory;

	class CommentFormFactory extends FormFactory
	{
		
		protected function getControlExtensions()
		{
			return array_merge(parent::getControlExtensions(), array(
				new \FormsModule\ControlExtensions\ControlExtension(),
			));
		}

		public function configure(Form $form)
		{
			$form->addGroup();
			$form->addDateTime('created', 'Created');
	
			$form->addGroup('Author');
			$authorName = $form->addText('authorName', 'Name');
			$author = $form->addManyToOne('author', 'Author');
	
			$authorName
				->addConditionOn($author, $form::FILLED)
				->addRule($form::EQUAL, '')
				->elseCondition()
				->addRule($form::FILLED);
	
			$author
				->addConditionOn($authorName, $form::FILLED)
				->addRule($form::EQUAL, '')
				->elseCondition()
				->addRule($form::FILLED);
	
			$form->addGroup('Text');
			$form->addTextArea('text', 'Text')
				->setRequired(TRUE)
				->getControlPrototype()->attrs['class'][] = 'span12';
	
			$form->addSaveButton('Save');
		}
	}

Následně továrnu zaregistrujeme do modulového konfiguračního souboru:

	services:
	
		guestbook.commentFormFactory:
			class: GuestbookModule\Forms\CommentFormFactory
			setup:
				- injectFactory(@cms.admin.basicFormFactory)

Dále budeme potřebovat formulář pro nastavení počtu komentářů na stránku:

	<?php

	namespace GuestbookModule\Forms;
	
	use Venne;
	use Venne\Forms\Form;
	use DoctrineModule\Forms\FormFactory;

	class PageFormFactory extends FormFactory
	{
		public function configure(Form $form)
		{
			$form->addGroup('Settings');
			$form->addText('itemsPerPage', 'Items per page');
	
			$form->addGroup();
			$form->addSaveButton('Save');
		}
	}

Opět továrnu zaregistrujeme:

	services:

		guestbook.pageFormFactory:
			class: GuestbookModule\Forms\PageFormFactory
			setup:
				- injectFactory(@cms.admin.basicFormFactory)

Nyní nám už chybí jenom komponenta, která za nás vyřeší editaci komentářů. Jelikož se bude jednat o komponentu spravující stránku, jako předka komponenty využijeme třídu `CmsModule\Content\SectionControl`. Třídu umístíme do `GuestbookModule/Components`.

	<?php

	namespace GuestbookModule\Components;
	
	use GuestbookModule\Repositories\CommentRepository;
	use CmsModule\Content\SectionControl;
	use GuestbookModule\Forms\CommentFormFactory;

	class TableControl extends SectionControl
	{

		protected $commentRepository;

		protected $commentFormFactory;

		public function __construct(CommentRepository $commentRepository, CommentFormFactory $commentFormFactory)
		{
			parent::__construct();

			$this->commentRepository = $commentRepository;
			$this->commentFormFactory = $commentFormFactory;
		}
	
		protected function createComponentTable()
		{
			$table = new \CmsModule\Components\Table\TableControl;
			$table->setTemplateConfigurator($this->templateConfigurator);
			$table->setRepository($this->commentRepository);
	
			$pageId = $this->entity->id;
			$table->setDql(function ($sql) use ($pageId) {
				$sql = $sql->andWhere('a.page = :page')->setParameter('page', $pageId);
				return $sql;
			});
	
			// forms
			$repository = $this->commentRepository;
			$entity = $this->entity;
			$form = $table->addForm($this->commentFormFactory, 'Comment', function () use ($repository, $entity) {
				return $repository->createNew(array($entity));
			}, \CmsModule\Components\Table\Form::TYPE_LARGE);
	
			// navbar
			$table->addButtonCreate('create', 'Create new', $form, 'file');
	
			$table->addColumn('text', 'Text')
				->setWidth('35%')
				->setSortable(TRUE)
				->setFilter();
			$table->addColumn('author', 'Author')
				->setWidth('25%')
				->setCallback(function ($entity) {
					return $entity->author ? $entity->author : $entity->authorName;
				});
			$table->addColumn('created', 'Created', \CmsModule\Components\Table\TableControl::TYPE_DATE_TIME)
				->setWidth('20%')
				->setSortable(TRUE);
			$table->addColumn('updated', 'Updated', \CmsModule\Components\Table\TableControl::TYPE_DATE_TIME)
				->setWidth('20%')
				->setSortable(TRUE);
	
			$table->addActionEdit('edit', 'Edit', $form);
			$table->addActionDelete('delete', 'Delete');
	
			// global actions
			$table->setGlobalAction($table['delete']);
	
			return $table;
		}
	
		public function render()
		{
			$this['table']->render();
		}
	}

Naše komponenta obsahuje subkomponentu pro editaci tabulek. V první části továrničky zadáváme repozitář, se kterým komponenta bude pracovat a následně upravíme defaultní DQL dotaz na databázi. Jelikož chceme v tabulce jenom ty komentáře, které patří k námi editované instanci stránky, přidáme omezující podmínku. V následující části nastavujeme formulář, který bude záznamy editovat. Jedná se o námi dříve připravený `@guestbook.commentFormFactory`. V další části nakonfigurujeme sloupečky a akce. Metoda `render` pouze vyrenderuje subkomponentu. Díky tomu nebudeme potřebovat žádnou šablonu.

Na závěr jako tradičně komponentu zaregistrujeme do DIC, tentokrát do sekce `factories`:

	factories:
		guestbook.tableControl:
			class: GuestbookModule\Components\TableControl
			tags: [component]

Nyní máme vše nachystané, abychom nový typ stránky mohli zaregistrovat do redakčního systému. Opět cesta povede přes několik řádků v lokálním neon souboru:

	services:
	
		guestbook.guestbookContent:
			class: CmsModule\Content\ContentType('GuestbookModule\Entities\PageEntity')
			setup:
				- addSection('Content', @guestbook.tableControlFactory)
				- addSection('Settings', @guestbook.pageFormFactory)
			tags: [contentType: [name: 'guestbook']]

První řádek definuje entitu reprezentující stránku. V následující sekci `setup` můžeme nadefinovat sekce administrační části. Naše aplikace bude vyžadovat dvě sekce. První `content`, na které budeme provádět editaci komentářů pomocí komponenty `@guestbook.tableControlFactory`. Druhá `settings`, kde bude formulář pro nastavení počtu komentářů na stránku. Aby systém věděl, že se jedná o definici typu stránky, nesmíme zapomenout na tag `contentType` a na jméno nového typu stránky.

Nyní bychom měli mít backend připraven k použití. Zbývá tedy poslední část.

## Frontend návštěvní knihy