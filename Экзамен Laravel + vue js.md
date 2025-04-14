### Создание проекта Laravel в putty
```bash 
composer create-project --prefer-dist laravel/laravel имя_проекта
```

### Создаём .htaccess в корне
```  
RewriteEngine On  
# Перенаправляем все запросы в папку public  
RewriteRule ^(.*)$ public/$1 [L]  
```

### Создание middleware для управления CORS (если есть проблемы с cors)
```bash  
php artisan make:middleware CorsMiddleware  
```

```php  
#app/Http/Middleware/CorsMiddleware  
<?php  
namespace App\Http\Middleware;  
  
use Closure;  
use Illuminate\Http\Request;  
use Symfony\Component\HttpFoundation\Response;  
  
class CorsMiddleware  
{  
	public function handle(Request $request, Closure $next): Response  
	{  
		$response = $next($request);  
		$response->headers->set('Access-Control-Allow-Origin', '*'); // Разрешить запросы со всех доменов  
		$response->headers->set('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');  
		$response->headers->set('Access-Control-Allow-Headers', 'Content-Type, Authorization');  
		return $response;  
	}  
}  
```
Не забыть подключить middleware в bootstrap/app.php

### Подруб jwt-auth  
```bash  
composer require tymon/jwt-auth  
php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"  
php artisan jwt:secret  
```

### Создаём /frontend в корне. Установка vue, axios, pug, sass
  
```bash  
cd frontend  
npm create vue@latest  
npm install  
npm i axios
npm i -D sass
vue add pug
```

### JWT авторизация. Postman

##### Выдача токена при успешной регистрации

```php
 $payloadParam = [
	'sub' => $id_user,
	'iat' => now()->timestamp,
	'exp' => now()->addDays(3)->timestamp,
	'roles' => $roles
];

$payload = JWTFactory::customClaims($payloadParam)->make();

$token = JWTAuth::encode($payload)->get();
```

Сюда же как получить id последней строки из базы
```php
$id_user = DB::getPdo()->lastInsertId();
```
##### Настройка middlware для protected маршрутов

```php
public function handle(Request $request, Closure $next): Response
    {  
        try{
            $token = JWTAuth::parseToken();
            $payload = $token->getPayload();
        } catch (\Exception $e) {
            return response()->json(['err' => 'Ошибка авторизации: ' . $e->getMessage()], 401);
        }

        $request->attributes->add(["payload"=> $payload]);
        return $next($request);

    }
```
Не забыть подключить middleware на нужные маршруты
##### Получение ролей в самом контроллере
```php
$payload = $request->attributes->get('payload');
$role = $payload->get('roles');
```


### Заметки по vue фронтенду
##### Плавный якорь в vue-router

```js
// после routes[],
scrollBehavior(to, from, savedPosition) {
    if (to.hash) {
      return new Promise((resolve) => {
        setTimeout(() => {
          resolve({
            el: to.hash,  // Используем 'el' вместо 'selector' в Vue Router v4
            behavior: 'smooth',
          });
        }, 100);
      });
    } else if (savedPosition) {
	  return savedPosition;
    } else {
      return { top: 0 }  // Используем 'top' вместо 'x' и 'y' в Vue Router v4
    }

  }
```

##### Получение путей из БД
```js
// vite.config.js
export default defineConfig({
  plugins: [vue(),pug(),vueDevTools()],
  server: {
    proxy: {
      // Прокси для картинок из Laravel /public/data
      '/data': {
        target: 'http://127.0.0.1:8000/', // твой Laravel
        changeOrigin: true,
      }
  }
}
})

// метод в компоненте
getImageSrc(image) {
	return `/data/${image}`;
	// return new URL(`../../assets/images/tours/${image}`, import.meta.url).href;
},
```

#### Валидация формы + сохранение jwt
```js
methods: {
    validateForm() {
      this.errors = {}
      let valid = true  

      if (!/^[А-Яа-яЁёA-Za-z\s\-]{1,200}$/.test(this.form.fio.trim())) {
          this.errors.fio = 'ФИО должно содержать только буквы, пробелы'
          valid = false
      }
	  if (!this.form.email || !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(this.form.email)){
          this.errors.email = 'Некорректный email'
          valid = false
      }

      if (!/^(?:\+7|8)\d{10}$/.test(this.form.tel)) {
        this.errors.tel = 'Телефон должен начинаться с +7 или 8 и содержать ровно 11 цифр';
        valid = false;
      }

      if (!this.form.password || this.form.password.length < 6) {
          this.errors.password = 'Пароль должен быть от 6 символов'
          valid = false
      }
    return valid
    },

    async handleRegister() {
      if (!this.validateForm()) return 
      try {
        const response = await axios.post(import.meta.env.VITE_API_URL + '/register',
        {
          "email": this.form.email,
          "password": this.form.password,
          "fio": this.form.fio,
          "tel":  this.form.tel,
        }
      )
        console.log('Успешно зарегистрирован', response.data)
        localStorage.setItem('token', response.data.access_token)
        this.$router.push('/user-cabinet')
        // Перенаправление или уведомление
      } catch (error) {
        const msg = error.response?.data?.msg || 'Ошибка входа';
        alert(msg);яё
      }
    }
  }
```

#### Кэширование картинок
```js
 async mounted() {
        const cachedTours = sessionStorage.getItem('tours');
        if(cachedTours){
            this.tours = JSON.parse(cachedTours);
        }else {
            this.fetchTours();
        }
    },
```
##### Сортировка(фильтр) на фронте
```js
 methods:{
    sortProducts(){
      const key = this.filter;
      const sortMethods = {
        id: (a,b) => a.id - b.id,
        price: (a,b) => a.price - b.price,
        quantity: (a,b) => a.quantity - b.quantity,
        name: (a,b) => a.name.localeCompare(b.name),
        category: (a,b) => a.category.localeCompare(b.category),
        created_at: (a,b) => new Date(a.created_at) - new Date(b.created_at),
        updated_at: (a,b) => new Date(a.updated_at) - new Date(b.updated_at)
      }
      this.products.sort(sortMethods[key])
    }
  },
  watch: {
    filter(){
      this.sortProducts();
    }
  }
```
