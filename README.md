yii2-sharding
============================

Компонент для работы Active Record моделей c шардированной базой данных.

Установка
-------------------
 
Установка с помощью пакета composer `"kllakk/yii2-sharding": "*"`

Пример использования
------------

Сперва сконфигурируйте компонент `sharding` по примеру:

```php
 'sharding' => [
            'class' => 'kllakk\sharding\Connection',
            'shard' => [
                'profile' => [
                    'coordinator' => 'coordinator',
                    'db' => ['db1', 'db2']
                ]
            ]
        ],
```
Где `shard` - это массив разных типов шардирования, где ключами выступают имена этих типов. Для каждого типа необходимо задать 
название компонента координатора `coordinator` и массив названий компонентов баз данных `db`, которые участвуют в данном виде шардинга.

Затем сконфигурируйте и настройте компонент [coordinator](https://github.com/kllakk/yii2-coordinator) для данного типа разделения
данных.

Для использования необходимо наследовать вашу Active Record модель от класса `kllakk\sharding\ActiveRecord` и реализовать
обязательные методы:

```php
    public static function primaryKey() {
        return ['id'];
    }

    public static function shardingColumn() {
        return 'id';
    }

    public static function shardingType() {
        return 'profile';
    }
    
    public function attributeLabels() {
        return [
            'id' => 'ID',
            'title' => 'Название',
            'description' => 'Описание',
        ];
    }
```
Которые возвращают соотвественно первичный ключ, ключ шардирования, тип шардирования и атрибуты. Затем вы можете работать с этой
моделью как с обычной Active Record, но с некоторыми ограничениями из-за специфики шардинга 
(в методы all, one и другие у Active Query теперь необходимо передавать не сам компонент нужной базы, а его название, группировка не будем работать,
если для запроса придется пройти несколько шардов). 

Выполнение SQL запросов
------------

Выполнение запросов напрямую осуществляется немного иначе. В `createCommand` первым параметров передается название компонентов db, 
вторым параметров передается массив sql запросов, где ключами массива выступает название компонента db соотвествующему этому запросу,
а значениями сам запрос.

```php
$sharding->createCommand(['db1'], ['db1' => 'SELECT * FROM test']);
```
