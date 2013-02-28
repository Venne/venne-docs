# Widgety

Widgety představují vylepšené komponenty, které nevyžadují továrničky v presenterech. Místo nich se využívá továrnička definovaná v DIC. 

## Ukázka widgetu


	<?php
	
	namespace ExampleModule\Components;
	
	use CmsModule\Content\Control;
	
	class NavigationControl extends Control
	{
	
		public function render()
		{
			...
		}
	
	}

## Registrace do frameworku

Ukázka registrace widgetu do frameworku pod jménem `navigation`.

	factories:
	
		example.navigationControl:
			class: ExampleModule\Components\NavigationControl
			tags: [widget: navigation]
			
## Použití v šabloně

	{control navigation}
	
### Priorita volání továrniček

1. továrna v presenteru - `createComponent<name>`
2. továrna v DIC s tagem - tags: [widget: <name>]
