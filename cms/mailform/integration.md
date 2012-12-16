# Integrace mailformu do vlastního modulu

## Definice v entitě

	/**
	 * @var \MailformModule\Entities\MailformEntity
	 * @OneToOne(targetEntity="MailformModule\Entities\MailformEntity", cascade={"all"})
	 */
	protected $mailform;

### Inicializace asociace

	public function __construct()
	{
		parent::__construct();

		...
		$this->mailform = new MailformEntity();
		...
	}

## Formulář s editací mailformu

	class FormFactory extends FormFactory
	{
		
		protected function getControlExtensions()
		{
			return array_merge(parent::getControlExtensions(), array(
				new \CmsModule\Content\ControlExtension(),
				new \FormsModule\ControlExtensions\ControlExtension(),
				new \MailformModule\Forms\ControlExtensions\MailformExtension(),
			));
		}
		
		public function configure(Venne\Forms\Form $form)
		{
			...
			$form->addMailform('mailform');
			...
		}

	}

## Třída `Mailform\Components\MailControl`

Tato komponenta představuje front-end mailformu. Po odeslání formuláře dojde k automatickému rozeslání mailů podle konfigurace v `MailformEntity`. 

V našich aplikací budeme nejšastěji používat předpřipravenou službu `@mailform.mailControl`, která jako parametr v konstruktoru dostává entitu `MailformEntity`.

### API komponenty

* `$onSuccess[] -> function(MailControl)` - událost po úspěšném odeslání formuláře a e-mailů.
* `$onSendMessage[] -> function(MailControl, Message)` - událost před odesláním e-mailu. Slouží pro dodatečné úpravy odesílaného e-mailu.
* `$onSendCopyMessage[] -> function(MailControl, Message)` - událost před odesláním kopie e-mailu odesílateli. Slouží pro dodatečné úpravy odesílaného e-mailu.

### Ukázka vložení mailformu do presenteru

	class FooPresenter extends \CmsModule\Content\Presenters\PagePresenter
	{
	
		/** @var Callback */
		protected $mailConotrolFactory;
		
		public function injectFormFactory(Callback $mailConotrolFactory)
		{
			$this->mailConotrolFactory = $mailConotrolFactory;
		}
		
		public function createComponentOrderForm()
		{
			$control = $this->mailConotrolFactory->invoke($this->page->mailform); # první argument představuje entitu mailformu
			$control->onSuccess[] = $this->formSuccess;
			return $control;
		}

		public function formSuccess(MailControl $control)
		{
			$this->flashMessage('Message has been sent', 'success');
			$this->redirect('this');
		}
	
	}
	
###
