# Zabezpečení presenterů

## Ukázka

```php
/**
 * @secured
 */
class BasePresenter extends \CmsModule\Presenters\AdminPresenter
{
	...
	/**
	 * @secured(privilege="write")
	 */
	public function handleDelete()
	{
		$this->user->logout(true);
		$this->flashMessage('Item has been deleted.', 'success');
		$this->redirect('this');
	}
	...
	/**
	 * @secured(roles="writer, poster")
	 */
	public function handleDelete()
	{
		$this->user->logout(true);
		$this->flashMessage('Item has been deleted.', 'success');
		$this->redirect('this');
	}
}
```

### Annotace `secured($resource=NULL, $privilege=NULL, $roles=array(), $users=array())` nad třídou

- `$resource` - pokuď není uveden resource, použije se název třídy
- `$privilege` - pokuď není uveden, použije se NULL
- `$roles` - pokuď jsou uvedeny role, ověření se omezí pouze na jejich výčet
- `$users` - pokuď jsou uvedeni uživatelé, ověřuje se omezení pouze na jejich výčet

### Annotace `secured($resource=NULL, $privilege=NULL, $roles=array(), $users=array())` nad `action<Name>` a `handle<Name>`

Bere se v úvahu jen když je anotace `secured` i nad třídou.

- `$resource` - pokuď není uveden resource, použije se z anotace třídy
- `$privilege` - pokuď není uveden, použije se název akce nebo signálu
- `$roles` - pokuď jsou uvedeny role, ověření se omezí pouze na jejich výčet
- `$users` - pokuď jsou uvedeni uživatelé, ověřuje se omezení pouze na jejich výčet
