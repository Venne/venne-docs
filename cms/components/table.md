# TableControl

Komponentra pro správu tabulek

## Ukázka

		public function createComponentTable()
		{
			$table = new \CmsModule\Components\Table\TableControl;
			$table->setRepository($this->languageRepository);
			$table->setPaginator(10);
			$table->enableSorter();
	
			// forms
			$form = $table->addForm($this->form);
	
			// navbar
			$table->addButtonCreate('create', 'Create new', $form, 'file');
	
			// columns
			$table->addColumn('name', 'Name', '50%');
			$table->addColumn('alias', 'Alias', '20%');
			$table->addColumn('short', 'Short', '30%');
	
			// actions
			$table->addActionEdit('edit', 'Edit', $form);
			$table->addActionDelete('delete', 'Delete');
	
			// global actions
			$table->setGlobalAction($table['delete']);
	
			return $table;
		}
