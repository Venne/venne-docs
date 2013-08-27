# Registrace rout (routes)

Ve výchozím stavu neobsahuje framework žádnou přednastavenou routu. Vše tak zůstává čistě na modulech, které se o své routy musejí postarat sami. Routy se do systému registují jako běžné služby otagované pomocí `route`.



## Registrace do frameworku

```yaml
services:
	foo.route:
		class: Nette\Application\Routers\Route('<presenter>/<action>[/<id>]', 'Homepage:default')
		tags: [route]
```

Pro určení pořadí lze využít systém priorit.

```yaml
services:
	foo.route:
		class: Nette\Application\Routers\Route('<presenter>/<action>[/<id>]', 'Homepage:default')
		tags: [route: [priority: 5]]
```