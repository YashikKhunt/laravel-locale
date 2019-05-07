# How to set Locale in Laravel (Language)
แก้ไฟล์ **database/create_users_table.php** เพิ่ม
```php
$table->string('lang')->default('en');
```
เพื่อทำการเก็บข้อมูลภาษาของ User

```$ php artisan make:middleware LanguageMiddleware```&nbsp;&nbsp;แล้วแก้ไขไฟล์
```php
<?php

namespace App\Http\Middleware;

use Closure;

class LanguageMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        app()->setLocale(auth()->user()->lang);
        return $next($request);
    }
}
```

---
เข้า **app/Http/Kernel.php** หา $routeMiddleware แล้วเพิ่ม
```php
'lang' => \App\Http\Middleware\LanguageMiddleware::class,
```

---
แก้ไขไฟล์ **routes/web.php** เพิ่ม
```php
Route::group(['middleware' => ['lang']], function(){

}
```
หลังจากนั้นนำ Route (get) ที่ต้องมีการเปลี่ยนภาษาทั้งหมดไปใส่ไว้ใน Route ข้างบน เช่น
```php
Route::group(['middleware' => ['lang']], function(){
    Route::get('/home', 'HomeController@index')->name('home');
    Route::post('/home/upload', 'HomeController@upload')->name('home.upload');

    Route::get('/2fa', 'PasswordSecurityController@show2faForm')->middleware('auth');
});
```

---
# การทำ Form เปลี่ยนภาษา
```$ php artisan make:controller LanguageController```&nbsp;&nbsp;เพิ่ม Function
```php
public function changeLanguage(Request $request){
    auth()->user()->lang = $request->locale;
    auth()->user()->save();
    return back();
}
```
แก้ไข Route เพิ่ม
```php
Route::post('/changeLanguage', 'LanguageController@changeLanguage')->name('changeLanguage');
```

การสร้าง Form
```html
<form method="POST" action="{{ route('changeLanguage') }}">
    @csrf
    <label>Language</label>
    <select name="locale">
        <option value="en">EN</option>
        <option value="th">TH</option>
    </select>
    <button class="btn btn-inline">Change</button>
</form>
```

---
การแสดงข้อความในภาษาต่าง ๆ กัน ใช้
```html
{{ __('texts.welcome') }}
```
การทำงานของ Code ข้างบนคือ จะเข้าไปที่ resources/lang แล้วมันจะหา Folder ภาษาให้เองแล้วก็จะหาไฟล์ชื่อ texts.php ซึ่งใน texts.php จะมี return เป็น array, มันจะหาค่า welcome ใน array

ตัวอย่าง texts.php
```php
<?php
return [
    'welcome' => 'Hello',
    'choose-language' => 'Choose Language',
    'drop-files' => 'Drag files here to upload or click here.',
    'authen-complete' => 'Authen complete',
];
```

ถ้าจะทำเป็นภาษาไทยก็ต้องสร้าง Folder ชื่อ th ใน resources/lang แล้วสร้างไฟล์ที่ต้องการใน Folder 'th'  
ตัวอย่าง texts.php ใน Folder 'th'
```php
<?php
return [
    'welcome' => 'สวัสดี',
    'choose-language' => 'เลือกภาษา',
    'drop-files' => 'ลากไฟล์มาวางหรือกดเลือกไฟล์เพื่ออัพโหลด',
    'authen-complete' => 'ยืนยัน 2FA สำเร็จ',
];
```

Source: [Youtube](https://www.youtube.com/watch?v=O1l-tzyylVY) มีการปรับเปลี่ยนมาจาก Video นิดหน่อย