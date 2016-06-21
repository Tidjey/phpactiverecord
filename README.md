# PHP ActiveRecord - Version 1.3 #

Судя по тому, что phpactiverecord развиватеся очень медленно и неохотно принимают pull-request, даже с важными изменениями, пришлось создать свою ветку

В оригинале с ветками вообще неразбериха 1.2 намного отстала от 1.1, а 1.1 от мастера

Этот форк из ветки 1.1, так как в ней есть возможность создавать свои правила с учетом создания, изменения или удаления сущности

Но постараюсь вносить в него изменения из мастера

## Отличия
- Все фиксы из ветки мастер, актуальность отслеживается
- Более удобный вывод логов - https://github.com/jpfuentes2/php-activerecord/pull/444/files
- Добавлен класс для работы с Memcached
- Добавлены сценарии, примерно как в yii 1.1, валидация будет срабатывать только для определенного сценария
- Добавлена поддержка проверка длины UTF-8 строк, в оригинале методы валидации неверно определяют длину строки
- Добавлен новый метод валидации validates_existence_of для проверки существования записи в таблицах

### Пример использования сценариев

####В моделе

```php
static $validates_presence_of = [
  ['reply', 'message' => 'Необходимо заполнить поле ответ', 'scenario' => 'reply'],
];
```

####В контроллере
```php
$guest = new Guestbook();
$guest->scenario = 'reply';
$guest->text = 'Текст сообщения';
$guest->reply = 'Ответ';
$guest->save();
```
Если передано свойство scenario = имя сценария, то при валидации данных будет проверено с учетом данного сценария, в противном случае правило валидации будет проигнорировано

```php
$guest = new Guestbook();
$guest->text = 'Текст сообщения';
$guest->save();
```

Можно передавать массив сценариев: `'scenario' => ['reply', 'answer']`

### Пример использования валидации validates_existence_of
```php
static $validates_existence_of = [
  ['parent_id', 'in' => 'Category:id', 'presence' => true, 'allow_empty' => true, 'message' => 'Родительская категория не найдена']
];

- in - указывает на модель:поле
- presence - должна ли присутствовать запись или нет, по умолчанию true
- allow_empty - разрешено ли передавать нулевое значение, если передан ноль, то валидация не срабатывает
- message - пользовательское описание ошибки
```

### Вид лога
```
[info] 0.002s -- SHOW COLUMNS FROM `users` [июн 21 15:21:47]
[info] 0.001s -- SELECT * FROM `users` WHERE `id`='1' LIMIT 0,1 [июн 21 15:21:48]
[info] 0.001s -- SHOW COLUMNS FROM `forums` [июн 21 15:21:48]
[info] 0.001s -- SELECT * FROM `forums` WHERE parent_id = '0' ORDER BY sort [июн 21 15:21:48]
[info] 0.001s -- SHOW COLUMNS FROM `topics` [июн 21 15:21:48]
[info] 0.000s -- SELECT count(*) as count, forum_id FROM `topics` WHERE `forum_id` IN('1','2','3','4') GROUP BY forum_id [июн 21 15:21:48]
[info] 0.000s -- SELECT * FROM `forums` WHERE `parent_id` IN('1','2','3','4') ORDER BY sort DESC [июн 21 15:21:48]
[info] 0.001s -- SHOW COLUMNS FROM `posts` [июн 21 15:21:48]
[info] 0.001s -- SELECT count(*) as count, forum_id FROM `posts` WHERE `forum_id` IN('15','14','13','12','11','10','9','8','7','6','5') GROUP BY forum_id [июн 21 15:21:48]
[info] 0.000s -- SELECT count(*) as count, forum_id FROM `topics` WHERE `forum_id` IN('15','14','13','12','11','10','9','8','7','6','5') GROUP BY forum_id [июн 21 15:21:48]
[info] 0.000s -- SHOW COLUMNS FROM `topics` [июн 21 15:21:48]
[info] 0.000s -- SELECT * FROM `topics` WHERE `id` IN('7',NULL,NULL,NULL) [июн 21 15:21:48]
```
Показывается время выполнения каждого запроса и подставлены все значения в плейсхолдеры


## Introduction ##
A brief summarization of what ActiveRecord is:

> Active record is an approach to access data in a database. A database table or view is wrapped into a class,
> thus an object instance is tied to a single row in the table. After creation of an object, a new row is added to
> the table upon save. Any object loaded gets its information from the database; when an object is updated, the
> corresponding row in the table is also updated. The wrapper class implements accessor methods or properties for
> each column in the table or view.

More details can be found [here](http://en.wikipedia.org/wiki/Active_record_pattern).

This implementation is inspired and thus borrows heavily from Ruby on Rails' ActiveRecord.
We have tried to maintain their conventions while deviating mainly because of convenience or necessity.
Of course, there are some differences which will be obvious to the user if they are familiar with rails.

## Minimum Requirements ##

- PHP 5.3+
- PDO driver for your respective database

## Supported Databases ##

- MySQL
- SQLite
- PostgreSQL
- Oracle

## Features ##

- Finder methods
- Dynamic finder methods
- Writer methods
- Relationships
- Validations
- Callbacks
- Serializations (json/xml)
- Transactions
- Support for multiple adapters
- Miscellaneous options such as: aliased/protected/accessible attributes

## Installation ##

Setup is very easy and straight-forward. There are essentially only three configuration points you must concern yourself with:

1. Setting the model autoload directory.
2. Configuring your database connections.
3. Setting the database connection to use for your environment.

Example:

```php
ActiveRecord\Config::initialize(function($cfg)
{
    $cfg->set_model_directories(array(
      '/path/to/your/model_directory',
      '/some/other/path/to/your/model_directory'
    ));
    $cfg->set_connections(
      array(
        'development' => 'mysql://username:password@localhost/development_database_name',
        'test' => 'mysql://username:password@localhost/test_database_name',
        'production' => 'mysql://username:password@localhost/production_database_name'
      )
    );
});
```

Alternatively (w/o the 5.3 closure):

```php
$cfg = ActiveRecord\Config::instance();
$cfg->set_model_directory('/path/to/your/model_directory');
$cfg->set_connections(
  array(
    'development' => 'mysql://username:password@localhost/development_database_name',
    'test' => 'mysql://username:password@localhost/test_database_name',
    'production' => 'mysql://username:password@localhost/production_database_name'
  )
);
```

PHP ActiveRecord will default to use your development database. For testing or production, you simply set the default
connection according to your current environment ('test' or 'production'):

```php
ActiveRecord\Config::initialize(function($cfg)
{
  $cfg->set_default_connection(your_environment);
});
```

Once you have configured these three settings you are done. ActiveRecord takes care of the rest for you.
It does not require that you map your table schema to yaml/xml files. It will query the database for this information and
cache it so that it does not make multiple calls to the database for a single schema.

## Basic CRUD ##

### Retrieve ###
These are your basic methods to find and retrieve records from your database.
See the *Finders* section for more details.

```php
$post = Post::find(1);
echo $post->title; # 'My first blog post!!'
echo $post->author_id; # 5

# also the same since it is the first record in the db
$post = Post::first();

# finding using dynamic finders
$post = Post::find_by_name('The Decider');
$post = Post::find_by_name_and_id('The Bridge Builder',100);
$post = Post::find_by_name_or_id('The Bridge Builder',100);

# finding using a conditions array
$posts = Post::find('all',array('conditions' => array('name=? or id > ?','The Bridge Builder',100)));
```

### Create ###
Here we create a new post by instantiating a new object and then invoking the save() method.

```php
$post = new Post();
$post->title = 'My first blog post!!';
$post->author_id = 5;
$post->save();
# INSERT INTO `posts` (title,author_id) VALUES('My first blog post!!', 5)
```

### Update ###
To update you would just need to find a record first and then change one of its attributes.
It keeps an array of attributes that are "dirty" (that have been modified) and so our
sql will only update the fields modified.

```php
$post = Post::find(1);
echo $post->title; # 'My first blog post!!'
$post->title = 'Some real title';
$post->save();
# UPDATE `posts` SET title='Some real title' WHERE id=1

$post->title = 'New real title';
$post->author_id = 1;
$post->save();
# UPDATE `posts` SET title='New real title', author_id=1 WHERE id=1
```

### Delete ###
Deleting a record will not *destroy* the object. This means that it will call sql to delete
the record in your database but you can still use the object if you need to.

```php
$post = Post::find(1);
$post->delete();
# DELETE FROM `posts` WHERE id=1
echo $post->title; # 'New real title'
```
