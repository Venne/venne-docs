# Přílohy v entitách

## Definice v entitě

### OneToOne

	/**
	 * @var \CmsModule\Content\Entities\FileEntity
	 * @ORM\OneToOne(targetEntity="\CmsModule\Content\Entities\FileEntity", cascade={"all"}, orphanRemoval=true)
	 * @ORM\JoinColumn(onDelete="SET NULL")
	 */
	protected $photo;

### OneToMany

	/**
	 * @var \CmsModule\Content\Entities\FileEntity[]
	 * @ORM\ManyToMany(targetEntity="\CmsModule\Content\Entities\FileEntity", cascade={"all"}, orphanRemoval=true)
	 * @ORM\JoinTable(
	 *      joinColumns={@ORM\JoinColumn(referencedColumnName="id")},
	 *      inverseJoinColumns={@ORM\JoinColumn(referencedColumnName="id", unique=true)}
	 *      )
	 **/
	protected $photos;

### Nastavení parenta

	/**
	 * @param \CmsModule\Content\Entities\FileEntity $photo
	 */
	public function setPhoto(\CmsModule\Content\Entities\FileEntity $photo = NULL)
	{
		$this->photo = $photo;

		if ($this->photo) {
			$this->photo->setParent($this->page->getDir());
			$this->photo->setInvisible(true);
		}
	}

## Formulář s editací přílohy

	class FormFactory extends \Venne\Forms\FormFactory
	{
		public function configure(Form $form)
		{
			... 
			$form->addFileEntityInput('photo', 'Photo');
			 ...
			$form->addFileEntityInput('photos', 'Photos');
			...
		}
		protected function getControlExtensions()
		{
			return array(
				new \DoctrineModule\Forms\ControlExtensions\DoctrineExtension(),
				new \CmsModule\Content\Forms\ControlExtensions\ControlExtension(),
				new \FormsModule\ControlExtensions\ControlExtension(),
			);
		}
	} 
