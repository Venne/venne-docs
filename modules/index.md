# Vytvoření modulu

## Vytvoření modulu příkazem

	php www/index.php venne:module:create [foo]

Příkaz vyžaduje práva zápisu do adresáře `vendor/venne`.

## Definice modulu

Moduly představují balíček rozšíření pro framework, mohou přidávat extenze, šablony, layouty a nejen to. Každý modul musí povinně obsahovat soubor `Module.php` se třídou implementující rozhraní `Venne\Module\IModule`. Abychom takovou třídu nemuseli psát od nuly, můžeme dědit od `Venne\Module\BaseModule` nebo v případě užití composeru můžeme využít třídu `Venne\Module\ComposerModule`, která si načte vše potřebné ze souboru `composer.json`.

Ve většině případu nám postačí vytvořit `Module.php` s obdobným obsahem jako:

	<?php

	namespace ExampleModule;

	use Venne;
	use Venne\Module\ComposerModule;

	class Module extends ComposerModule
	{


	}

A následně v souboru `composer.json`:

	{
		"name":"venne/example-module",

		...

		"require":{
			"venne/cms-module":"2.0.x"
		},
		"autoload":{
			"psr-0":{
				"ExampleModule":""
			}
		},
		"extra":{
			"branch-alias":{
				"dev-master":"2.0.x-dev"
			},
			"venne": {
				"configuration": {
					"parameters": {
						"foo": "bar",
						"bar": "foo"
					},
					"extensions": {
						"example": "ExampleModule\\DI\\ExampleExtension"
					},
					"includes": [
						"%modules.example.path%/Resources/config/config.neon"
					]
				}
			}
		}
	}

### Sekce `autoload`

Jelikož jsou moduly načítány loaderem z composeru, musíme mu říci, jakým způsobem má třídy hledat. Více zde http://getcomposer.org/doc/04-schema.md#autoload

### Sekce `extra`

Obsahuje pokročilé volby. Nás bude zajímat hlavně podsekce `venne`.

#### Subsekce `venne`

##### configuration

Struktura sekce `configuration` se doplní do hlavního konfiguračního souboru `config.neon`.

 - extensions - Umožňuje nám zaregistrovat do frameworku novou extension.
 - includes - Dovoluje includovat další konfigurační soubory.