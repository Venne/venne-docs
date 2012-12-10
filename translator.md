# Užití překladů v Nette
http://doc.nette.org/cs/localization

# Registrace slovníků v `config.neon`

	translator:
		dictionaries:
			- %modules.cms.path%/Resources/translations

# Pravidla pro adresáře slovníku

Adresář (v našem případě `translations`) může obsahovat soubory s překlady v tomto formátu: `<name>.<lang>.<driver>`. Příklady:

- admin.cs.php
- front.de.neon
- contacts.pl.ini

## Formát souborů
V souborech jsou zakódovány pole překladů v dané syntaxi ve tvaru `klíč=>hodnota`.

# Generování překladových souborů

## Extrahování stringů z cesty:

	php www/index.php venne:translator:extract vendor/venne/cms-module/

## Uložení výsledků do souboru

	php www/index.php venne:translator:extract vendor/venne/cms-module/ vendor/venne/cms-module/CmsModule/Resources/translations/test.cs.neon

# Užití překladů

- v šablonách: http://doc.nette.org/cs/default-macros#toc-preklady
- u formulářů: http://doc.nette.org/cs/localization#toc-preklad-formularu 
