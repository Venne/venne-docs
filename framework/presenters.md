# Presentery

## Ukázka presenteru


	<?php
	
	/**
	 * This file is part of the Venne:CMS (https://github.com/Venne)
	 *
	 * Copyright (c) 2011, 2012 Josef Kříž (http://www.josef-kriz.cz)
	 *
	 * For the full copyright and license information, please view
	 * the file license.txt that was distributed with this source code.
	 */
	
	namespace CmsModule\Administration\Presenters;
	
	use Venne;

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

## Registrace do frameworku

ukázka nastavení pro cestu `Core:Admin:Example`

	services:
	
		core.admin.examplePresenter:
			class: CmsModule\Administration\Presenters\ExamplePresenter
			tags: [presenter] 
