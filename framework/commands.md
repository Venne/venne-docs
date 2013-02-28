# Konzolové příkazy (commands)

## Oficiální dokumentace k symfony/console

http://symfony.com/doc/current/components/console.html

## Ukázka kódu

	<?php
	
	/**
	 * This file is part of the Venne:CMS (https://github.com/Venne)
	 *
	 * Copyright (c) 2011, 2012 Josef Kříž (http://www.josef-kriz.cz)
	 *
	 * For the full copyright and license information, please view
	 * the file license.txt that was distributed with this source code.
	 */
	
	namespace ExampleModule\Commands;
	
	use Symfony\Component\Console\Command\Command;
	use Symfony\Component\Console\Input\InputInterface;
	use Symfony\Component\Console\Output\OutputInterface;
	use Symfony\Component\Console\Input\InputArgument;
	use Symfony\Component\Console\Input\InputOption;

	
	class FooCommand extends Command
	{
	
		/** @var FooService */
		protected $fooService;
	
	
		public function __construct(FooService $fooService)
		{
			parent::__construct();
	
			$this->fooService = $fooService;
		}
	
	
		/**
		 * @see Console\Command\Command
		 */
		protected function configure()
		{
			$this
				->setName('example:foo')
				->setDescription('....')
				->setDefinition(array(
				new InputArgument('foo', InputArgument::REQUIRED, '...'),
				new InputArgument('bar', InputArgument::REQUIRED, '...')
			));
		}
	
	
		/**
		 * @see Console\Command\Command
		 */
		protected function execute(InputInterface $input, OutputInterface $output)
		{
			$foo = $input->getArgument('foo');
			$bar = $input->getArgument('bar');

			...
		}
	}

## Registrace do frameworku
	
	services:
		core.exampleCommand:
			class: ExampleModule\Commands\FooCommand
			tags: [command]
	 
