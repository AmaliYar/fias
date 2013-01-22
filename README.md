# Федеральная Информационная Адресная Система (ФИАС)

Для работы с базой потребуются файлы полной БД ФИАС в формате DBF. Скачать их можно по адресу:

    http://fias.nalog.ru/Public/DownloadPage.aspx

Там же можно скачать описание структуры базы.

# Установка и подготовка базы

1. Подключить гем

```
gem 'fias', git: 'https://github.com/evilmartians/kladr.git'
```

2. Скачать и распаковать базу ФИАС в tmp/fias (по умолчанию)

# Импорт структуры данных

Возможны два варианта:

```
rake fias:create_tables [DATABASE_URL=... PREFIX=... PATH=... ONLY=...]
```

Либо:

```
rails g fias:migration [--path=... --prefix=... --only=...]
```

Первый вариант - для использования гема вне рельсов, либо для случая, когда
актуальная база ФИАС будет использоваться только для локального мапинга, и
на продакшн попасть не должна.

В параметре only можно передать имена нужных таблиц, houses - все
таблицы домов.

# Импорт данных

```
rake fias:import PREFIX=fias PATH=tmp/fias ONLY=address_objects
rake fias:import PREFIX=fias PATH=tmp/fias ONLY=houses
```

Первый пример импортирует только адресные объекты (без справочников),
второй - только дома.

Поддерживается импорт в Postgres и SQLite (нужно для :memory: баз)

# Импорт данных в память (для рейк тасок)

```
ActiveRecord::Base.establish_connection(adapter: 'sqlite3', database: ':memory:')
connection = ActiveRecord::Base.connection.raw_connection

wrapper = Fias::DbfWrapper.new('tmp/fias')
importer = Fias::Importer.build(adapter: 'sqlite3', connection: connection)
tables = wrapper.tables(:address_objects) # Или без параметров

ActiveRecord::Schema.define do
  eval(importer.schema(tables))
end

importer.import(tables) do |name, data, index|
  break if index > 300
end
```

# Некоторые замечания про ФИАС

1. ФИАС хранит историю изменений информации адресных объектов. Например,
в ней есть и Ленинград и Санкт-Петербург. Исторические версии это
двунаправленный список.
2. Записи базы данных ФИАС имеют признак актуальности.
3. Ленинград и Санкт-Петербург, хотя, и хранятся в разных записях базы данных,
являются одним и тем же территориальным образованием. Код территориального
образования хранится в поле AOGUID. Из внешних таблиц логично ссылаться
на AOGUID, а не на AOID.

# Работа с данными

Существующие регионы:

```
Fias::AddressObject.actual.leveled(:region).all
```

Подчиненные объекты в регионе (области, районы, столица региона):

```
region = Fias::AddressObject.actual.leveled(:region).first
region.children.actual
```

# TODO

1. Индексы.

# Полезные ссылки

* http://fias.nalog.ru/Public/DownloadPage.aspx
* http://wiki.gis-lab.info/w/%D0%A4%D0%98%D0%90%D0%A1#.D0.A2.D0.B5.D0.BA.D1.81.D1.82.D0.BE.D0.B2.D1.8B.D0.B5_.D1.8D.D0.BB.D0.B5.D0.BC.D0.B5.D0.BD.D1.82.D1.8B_.D0.B0.D0.B4.D1.80.D0.B5.D1.81.D0.B0
* http://basicdata.ru
