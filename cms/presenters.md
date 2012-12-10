# Registrace presenteru do administrace

## Registrace do CMS

	services:
	
		cms.admin.languagePresenter:
			class: CmsModule\Administration\Presenters\LanguagePresenter(@cms.languageRepository, ..., @cms.languageFormFactory)
			tags: [presenter, administration: [
				link: 'Cms:Admin:Language:'
				category: 'Website'
				name: 'Language settings'
				description: 'Manage website languages, aliases,...'
				priority: 150
			]]

### Parametry tagu `administration`

 - **link** - Odkaz na presenter
 - **category** - Jméno kategorie (Položky v hlavní navigaci)
 - **name** - Název položky v navigaci
 - **description** - Popisek položky v navigaci
 - **priority** - Určuje pořadí prvku v kategorii