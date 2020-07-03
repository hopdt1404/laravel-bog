#Directory Structure

#####**#Introduction**
#####**#The Root Directory**
  




## Directory struct of Laravel application
Directory | Include 

`/app`| Include controller, model, views and assets: contain your code

`/public` | Store file css, javascrip, images, ... and start file `index.php`

`/verdor` | Contain code of third party, plugin install for application
`/app/config/`| Contain files configure when runing application as: database, session, etc.

`/app/config/app.php` | Configure setup layer application as: timezone, locale, modedug, unique key encode

`/app/config/auth.php` | Drive authen

`/app/config/cache.php` | Cache data and response request faster

`/app/database/migrations/` | Migration folder include php classes allow Laravel update Schema database but still keep all versions of databse when synchronized. Migration files create by Artisan tool

`/app/database/seeds/` | Contain php files allow Artisan put into(đưa vào) table with data suggestions

`/app/models/` | Contain model files of application

`/app/views/` | Folder contain html files use by controller or route

`/app/lang/` | Folder contain langluage use for pagination(phân trang) and authen from user with default language (common is Ennlish )

`/app/start/` | Contain config related(liên quan) to Artisan tool, context local and gloal

`/app/storage` | Folder store tempo files of Laravel services as: sesscion, cache, template view compile. This Folder can write by web server. Laravel control manage this folder and we uncessary intervence(can thiệp)

`/app/routes.php` | Route file in your application, it save all route to inform with Laravel how to connect when have request.

`/app/filter.php` | File use to limmit area access deny of web


