# Vlastní Latte makra (macros)

## Ukázka kódu

	class CssMacro extends \Nette\Latte\Macros\MacroSet
	{

		public static function filter(\Nette\Latte\MacroNode $node, $writer)
		{
			$path = $node->tokenizer->fetchWord();
			$params = $writer->formatArray();

			return ('$control->getPresenter()->getContext()->getService("assets.assetManager")->addStylesheet("' . $path . '", ' . $params . '); ');
		}

		public static function install(\Nette\Latte\Compiler $compiler)
		{
			$me = new static($compiler);
			$me->addMacro('css', array($me, "filter"));
		}

	}

## Registrace do frameworku

	factories:
		cssMacro:
			factory: AssetsModule\Macros\CssMacro::install(%compiler%)
			parameters: [compiler]
			tags: [macro]
