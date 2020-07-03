#**Configuration**
#######Introduction
#######Environment Configuration
    #######Environment Variable Types
	#######Retrieving Environment Configuration
	#######Determining The Current Environment
	#######Hiding Environment Variables From Debug Pages
#######Accessing Configuration Values
#######Configuration Caching
#######Maintenance Mode

## Introduction

All of the configuration files for the Laravel framework are stored in the `config` directory.

## Environment Confiuration

*
- Hữu ích khi có sự khác biệt configuration khi application chạy.
VD: Sự khác biệt giữa cache tại local và cache ở production server

*
- Laravel sử dụng một thư viện DotEnv PHP của Vance Lucas.
- Thư mục gốc của application chứa một file `.env.example`
- Nếu install Laravel thông qua Composer, file `.env.example` sẽ được renamed thành `.env`. Với các trường hợp còn lại thì nên rename file 

*
- Mỗi developer/server thường có một environment khác nhau nên file `.env` thường không được commit 
- Và cũng đầy rủi do khi những thông tin nhạy cảm (sensitive) bị lộ và có thể bị khai thác và giành quyền truy cập source control repositor.
- Tuy nhiên thì ta vẫn có thể tạo file riêng `env.testing`
- Khi chạy PHPUnit tests hoặc thực thi Artisan với option --env=tesing thì file sẽ bị ghi đè thành `.env`

### Environment Variable Types
- Tất cả variable ở trong file  `.env` là các `parsed as strings`
- Với enviroment variable chứa khoẳng trắng thì đặt trong dấu nháy kép

```
APP_NAME="My Application"
```

### Retrieving Environment Configuration

- Tất cả các variable listed will be load into the $_ENV PHP super-global variable.
- Tuy nhiên ta có thể sử dụng `env` để lấy dữ liệu từ các biến trong configuration files. 

```ruby
'debug' => env('APP_DEBUG', false),
```

- Second value passed to the `env` function is `default value`.
This value will be used if no enviroment variable exits for given key

### Determining The Current Environment



- Link: https://laravel.com/docs/7.x/configuration
- Position: The current application environment is determined via the APP_ENV variable from your .env file. You may access this value via the environment method on the App facade
