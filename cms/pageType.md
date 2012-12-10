# Registrace nového typu stránky

## Registrace do CMS

	services:
	
		pages.pageContent:
			class: CmsModule\Content\ContentType('PagesModule\Entities\PagesEntity')
			setup:
				- addSection('Content', @pages.pageFormFactory)
			tags: [contentType: [name: 'static page']]

### Definice sekcí

Provádí se pomocí `addSection(<Název sekce>, <továrnička komponenty>)`. Továrna může být pro formulář jako v tomto případě, nebo i pro jakoukoliv jinou komponentu.

### Parametry tagu `contentType`

 - **name** - Jméno typu stránky 
