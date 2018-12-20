# Cake3 CookieAuth
A simple Cake3 plugin to authenticate users with Cookies. This plugin is based on the awesome plugin [Xety/Cake3-Cookieauth](https://github.com/Xety/Cake3-CookieAuth) but with an option to allow empty passwords. It has also been fixed for CakePHP 3.7

## Requirements
* CakePHP 3.X

## Installation
Run : `composer require rubyan/cake3-cookieauth:1.*`
Or add it in your `composer.json`:
``` php
"require": {
	"rubyan/cake3-cookieauth": "1.*"
},
```

## Usage
In your `config/bootstrap.php` add :
``` php
Plugin::load('Xety/Cake3CookieAuth');
```

In your `AppController` :
``` php
public $components = [
	'Cookie',
	'Auth' => [
		'authenticate' => [
			'Form',
			'Xety/Cake3CookieAuth.Cookie'
		]
	]
			
];
```

In your `AppController`, in the `beforeFilter` action :
``` php
public function beforeFilter(Event $event) {
	//Automaticaly Login.
	if (!$this->Auth->user() && $this->Cookie->read('CookieAuth')) {

		$user = $this->Auth->identify();
		if ($user) {
			$this->Auth->setUser($user);
		} else {
			$this->Cookie->delete('CookieAuth');
		}
	}
}

//If you want to update some fields, like the last_login_date, or last_login_ip, just do :
public function beforeFilter(Event $event) {
	//Automaticaly Login.
	if (!$this->Auth->user() && $this->Cookie->read('CookieAuth')) {
		$this->loadModel('Users');

		$user = $this->Auth->identify();
		if ($user) {
			$this->Auth->setUser($user);

			$user = $this->Users->newEntity($user);
			$user->isNew(false);
			
			//Last login date
			$user->last_login = new Time();
			//Last login IP
			$user->last_login_ip = $this->request->clientIp();
			//etc...

			$this->Users->save($user);
		} else {
			$this->Cookie->delete('CookieAuth');
		}
	}
}
```

In your `login` action, after `$this->Auth->setUser($user);` :
``` php
//It will write Cookie without RememberMe checkbox
$this->Cookie->configKey('CookieAuth', [
	'expires' => '+1 year',
	'httpOnly' => true
]);
$this->Cookie->write('CookieAuth', [
	'username' => $this->request->data('username'),
	'password' => $this->request->data('password')
]);


//If you want use a RememberMe checkbox in your form :
//In your view
echo $this->Form->checkbox('remember_me');

//In the login action :
if($this->request->data('remember_me')) {
	$this->Cookie->configKey('CookieAuth', [
		'expires' => '+1 year',
		'httpOnly' => true
	]);
	$this->Cookie->write('CookieAuth', [
		'username' => $this->request->data('username'),
		'password' => $this->request->data('password')
	]);
}
```

If you use LDAP for authentication you don't want to store the password obviously.
You can set the password to null when writing the cookie.
``` php
	$this->Cookie->write('CookieAuth', [
		'username' => $this->request->data('username'),
		'password' => null
	]);
```

## Contribute
[Follow this guide to contribute](https://github.com/Xety/Cake3-CookieAuth/blob/master/CONTRIBUTING.md)
