# laravel-bog
Course Laravel In Sun-Asterisk Summer 2020

# Installation

## Installing Laravel

###### Via Composer Create-Project

** Create project laravel by composer:**
```bash
composer create-project --prefer-dist laravel/laravel name_project
```

** Local Development Server **
(access project and execute command below)
```bar
php artisan serve
```
## Configuration

**Public Directory**

- In public directory is your web server's document/ web roor.
- The index.php in public directory serves as front controller for all HTTP request entering your application

**Configuaration Files**
- All configuration files for the Laravel framework are in the config directory

**Directory Permissions**
- Configure some permissions in  storage and the bootstrap/cache directories 
- It should be writable by your web server or Laravel will not run.

**Application Key**
- Set your application key to a random string.
- If the application key is not set, your user sessions and other encrypted data will not be secure!
- If you installed Laravel via Composer or the Laravel installer, this key has already been set for you by
```bar
php artisan key:generate
```
- Typically, this string should be 32 characters long. The key can be set in the .env environment file. If you have not copied the .env.example file to a new file named .env, you should do that now. 

**Additional Configuration**

- You may wish to review the `config/app.php` file and its documentation
- Configure a few additional components of Laravel, such as: Cache, Database, Sesscion

## Web Server Configuration

#### Directory Configuration
`Laravel should always be served out of the root of the "web directory" configured for your web server. You should not attempt to serve a Laravel application out of a subdirectory of the "web directory". Attempting to do so could expose sensitive files present within your application`

#### Pretty URLs
###### Apache

- Laravel includes a `public/.htaccess` file that is used to provide URLs without the `index.php` front controller in the path

- Before serving Laravel with Apache, be sure to enable the `mod_rewrite` module so the `.htaccess` file will be honored by the server.

- If the `.htaccess` file that ships with Laravel does not work with your Apache installation, try this alternative:

```bar
Options +FollowSymLinks -Indexes
RewriteEngine On

RewriteCond %{HTTP:Authorization} .
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
```

#### Nginx

- If you are using Nginx, the following directive in your site configuration will direct all requests to the `index.php` front controller

```bar
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```

