# Rails 4.0 INTERESTING PARTS

http://habrahabr.ru/company/engineyard/blog/170473/
http://guides.rubyonrails.org/4_0_release_notes.html
https://blog.engineyard.com/2013/rails-4-changes
https://blog.engineyard.com/2013/new-in-rails-4


##RUBY 1.9.3 MINIMUM 
Ruby 2.0 preferred; 1.9.3+ required


##Больше нет vendor/plugins
юзайте bundler


##Hash-based и методы dynamic finder

```ruby
User.find(:first, :conditions => { … })
```
- раньше такой код выдавал deprecation warning, теперь эксепшен
(Чтобы добавить эту функциональность, придётся установить gem activerecord-deprecated_finders


##ActiveRecord::SessionStore
 - убрали, используйте gem activerecord-session_store
 а вообще лучше храните саму сессию с конфидениальной инфой в memcache, а айдишник сессии - в куки
 
 
##ActiveResource
так же удалён, юзаем gem activeresource


##Обзёрверы, кэширование страниц и экшенов

- всё это было удалено, вместо них теперь кэширование фрагментов 
(см. <a href="https://github.com/BrVer/rails_presentation/blob/master/README.MD#user-content-11-Кэширование" 
target="_blank">https://github.com/BrVer/rails_presentation/blob/master/README.MD#user-content-11-Кэширование</a>)


##Dalli вместо memcache-client

говорят он шустрее и позволяет использовать параллелизм


##New Default Test Locations

app/models -> test/models (было test/units)
app/helpers -> test/helpers (было test/units/helpers)
app/controllers -> test/controllers (было test/functional)
app/mailers -> test/mailers (было test/functional)


## update -> HTTP PATCH 
было PUT, PATCH более правильно, т.к. PUT, должен содержать полную презентацию объекта, который нужно изменить, 
а PATCH предназначен для того, чтобы пересылать кусочки информации об объекте на сервере без его полного описания.

Для того, чтобы осталась обратная совместимость, бывший метод PUT также будет направлен к “update” в Rails 4.0, 
чтобы существующие APIs могли продолжить свою нормальную работу.

##Threadsafe по умолчанию


##Postgres H-Store

ActiveRecord в Rails 4 поддерживает расширение PostgreSQL hstore. 
Это расширение для базы данных позволяет создавать новый тип данных в колонках PostgreSQL, 
который называется ‘hstore’ и представляет собой string-only хэш. 
Для большинства намерений и целей это похоже на сериализацию данных в текстовом столбце, но то, что это является теперь 
нативным типом данных, даёт нам солидную прибавку производительности и возможность посылать прямые запросы к этому 
типу данных. Это означает, что у вашего приложения будет частично non-schematic база данных без необходимости 
использования MongoDB, CouchDB, Riak или другого non-schematic datastore.
Hstore расширение, хоть и не является полномасштабным non-schematic решением, его интеграция в Rails будет прекрасным 
дополнением для многих приложений. Бывают ситуации, когда большинство ваших данных нормализировано для конкретной 
модели, но на некоторую информацию нужно ссылаться на лету. Вот где hstore может по-настоящему себя проявить.


##Array, MACADDR, INET, CIDR

ActiveRecord вместе с Rails 4 поддерживают в PostgreSQL следующие типы данных: Array, MACADDR, INET и CIDR. 
Хотя они обычно хранятся как строки в большинстве баз данных, PostgreSQL предостовляет поддержку нативных типов данных 
для каждого из них.
Типы данных INET и CIDR преобразуются в инстансы класса IPAddr, массивы остаются массивами, 
а MAC адреса на запросы возвращаются строками.

##Live Streaming

фактически позволяет транслировать контент в браузер подсоединённых пользователей напрямую с сервера с помощью 
Server-Sent Events. И тот факт, что браузер имеет решающее значение для этого процесса должны намекнуть вам об 
очевидной проблеме: Internet Explorer это еще не поддерживает. Это может быть исправлено в будущих версияю IE, 
однако на данный момент live streaming не работает в браузере Microsoft’а.
Live Streaming в Rails 4 также открывает множество интересных возможностей для вашего приложения. 
Наибольшую пользу это принесёт приложениям, использующим информацию, получаемую в реальном времени: биржевые сводки и 
финансовые приложения, игры и т.д.

подробнее:
http://en.wikipedia.org/wiki/Server-sent_events - про SSE
http://www.sitepoint.com/streaming-with-rails-4/
http://tenderlovemaking.com/2012/07/30/is-it-live.html


## strong params вместо attr_accessible

```ruby
def recipe_params
  params.require(:recipe).permit(:name, :ingredients, :directions)
end
```

## Default Headers

```ruby
config.action_dispatch.default_headers = {
'X-Frame-Options' => 'SAMEORIGIN',
'X-XSS-Protection' => '1; mode=block',
'X-Content-Type-Options' => 'nosniff'
}
```

Вы также можете установить и другие хэдеры по дефолту
подробнее: http://edgeguides.rubyonrails.org/security.html#default-headers


##Encrypted Cookies

новая session_store опция под названием ‘encrypted_cookie_store’, стоит по дэфолту, юзает sha
