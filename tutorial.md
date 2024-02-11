# Tutorial Laravel MEDSI BEM ITS SELARAS

Di tutorial ini kita akan belajar untuk memakai laravel sebagai framework untuk membuat aplikasi web terutama terkait dengan back-end

## Daftar Isi

1. [Pendahuluan](#pendahuluan)
2. [Instalasi](#instalasi)
3. [Konfigurasi](#konfigurasi)
4. [Penggunaan](#penggunaan)

## Pendahuluan

Tutorial mendalam mengenai laravel dapat dibaca di [dokumentasi resmi laravel](https://laravel.com/docs/10.x)

Untuk dapat memulai menggunakan laravel, anda diharuskan memiliki atau menginstall beberapa aplikasi dan tools di laptop anda

1. PHP Version > 8.0 ([Link](https://www.php.net/downloads.php))
2. Database Server (e.g. MySQL, PostgreSQL, SQLite, SQL Server)
3. Database Management (e.g. phpMyAdmin, MySql Workbench, Tableplus, etc)
4. Composer ([Link](https://getcomposer.org))
5. Text Editor (e.g. Visual Studio Code, PHPStorm, etc)

> Untuk memudahkan anda dapat menyingkat kebutuhan 1,2,3 dengan menggunakan aplikasi [XAMPP](https://www.apachefriends.org/download.html) / [Laragon](https://laragon.org) yang sudah menyediakan PHP, Database Server, dan Database Management untuk windows

## Instalasi

Buka terminal dalam laptop anda, untuk windows dapat menggunakan windows terminal (untuk membukan windows terminal dapat klik kiri pada folder yang diinginkan). Pastikan composer sudah terinstall dengan mengetikkan perintah `composer` di terminal. Jika sudah terinstall, maka composer akan menampilkan daftar perintah yang dapat digunakan.

![alt text](<Screenshot 2024-02-11 at 18.31.34.png>)

1. Arahkan terminal ke folder yang akan anda gunakan untuk menyimpan project laravel anda. (hint: gunakan perintah `cd` untuk berpindah folder)
2. Ketikkan Perintah berikut ini
   ```bash
   composer create-project laravel/laravel nama-project
   ```
3. Tunggu hingga proses instalasi berhasil dilakukan
4. Jika sudah berhasil, buka folder yang telah dibuat dan coba jalankan server laravel.
   ```bash
   cd nama-project
   php artisan serve
   ```
5. Buka browser dan ketikkan `localhost:8000` pada address bar. Apabila tidak ada error maka Laravel sudah terinstall dengan benar

![alt text](<Screenshot 2024-02-11 at 18.33.12.png>)

## Konfigurasi

Di tutorial ini kita akan menggunakan MySQL sebagai database server. Jika anda menggunakan XAMPP maka buka url `localhost/phpmyadmin` pada browser anda.

- Buat database baru dengan nama yang anda inginkan.
- Buka file `.env` pada project laravel anda dan ubah konfigurasi database sesuai dengan konfigurasi database yang anda buat sebelumnya. Misal nama database `tutorial_laravel` maka konfigurasi `.env` akan seperti berikut

  ```env
  DB_CONNECTION=mysql
  DB_HOST=127.0.0.1
  DB_PORT=3306
  DB_DATABASE=tutorial_laravel
  DB_USERNAME=root
  DB_PASSWORD=
  ```

  sesuaikan d dan username dengan yang anda gunakan pada database management. (untuk xampp biasanya username adalah `root` dan password kosong)

- Setelah konfigurasi database selesai, jalankan perintah berikut untuk membuat tabel user dan mencoba konfigurasi yang dibuat

  ```bash
  php artisan migrate
  ```

  ![alt text](<Screenshot 2024-02-11 at 18.25.18.png>)

- Jika tidak ada error, maka konfigurasi database sudah berhasil dilakukan, cek di database management anda apakah sudah ada tabel `users` yang baru saja dibuat.

## Penggunaan

### Model, migration, dan controller

Langkah pertama yang dilakukan di Laravel adalah membuat model, migration, dan controller. Model digunakan untuk mengakses data dari database, migration digunakan untuk membuat tabel di database, dan controller digunakan untuk mengatur alur data dari model ke view.

#### Membuat model, migration, dan controller

```bash
php artisan make:model Post --all
```

command diatas akan membuat model, migration, factory, seeder, policy, controller, and form requests bernama Post

#### Membuat isi tabel pada migration

Buka folder `database/migrations` dan buka file yang baru dibuat oleh command diatas biasanya nama file berupa tahun_bulan_tanggal saat command create migration dibuat. Setelah dibuka maka anda akan melihat kode seperti ini
![alt text](<Screenshot 2024-02-11 at 18.42.57.png>)

Untuk menambahkan isi dari tabel post kita dapat memasukkan kode pada function dibawah ini

```php
public function up()
{
    Schema::create('posts', function (Blueprint $table) {
        $table->id();
        // tambahkan kode disini
        $table->timestamps();
    });
}
```

misal kita akan menambahkan kolom title berupa string, content berupa text dan author berupa user id

```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('content');
    $table->foreignId('author')
    ->constrained('users','id')
    ->onUpdate('cascade')
    ->onDelete('cascade');
    $table->timestamps();
});
```

setelah itu kita dapat menjalankan perintah berikut untuk membuat tabel di database

```bash
php artisan migrate
```

Cek di database apakah tabel sudah dibuat menggunakan phpmyAdmin atau DBMS lainnya
![alt text](<Screenshot 2024-02-11 at 19.08.45.png>)

#### Membuat isi model

Hal pertama yang dapat dilakukan di model adalah menentukan field mana saja yang dapat diisi oleh request.

1. Buka file `app/Models/Post.php`
2. Tambahkan kode berikut

```php
protected $fillable = ['title', 'content', 'author'];
```

Didalam model kita juga dapat menambahkan kode untuk mengatur relasi antar tabel, misalnya kita akan membuat relasi antara tabel post dan user

1. Buka file `app/Models/Post.php`
2. Tambahkan fungsi berikut

```php
public function author(){
    return $this->belongsTo(User::class, 'author','id');
}
```

Kode di atas merupakan model `Post` yang memiliki relasi dengan model `User` dimana `Post` memiliki `author` yang merupakan foreign key dari `User` 3. Buka file `app/Models/User.php` 4. Tambahkan fungsi berikut

```php
public function posts(){
    return $this->hasMany(Post::class);
}
```

Kode di atas merupakan model `User` yang memiliki relasi dengan model `Post` dimana `User` dapat memiliki banyak `Post`

#### Membuat isi controller

Misalkan kita membuat sebuah API untuk store data maka berikut langkah-langkah yang dapat dilakukan

1.  Buka file `app/Http/Controllers/PostController.php`
2.  Tambahkan kode berikut pada function store

```php
public function store(StorePostRequest $request)
    {
       $data = $request->validated();

       if ($errors = $request->get('error')) {
            return response()->json($errors, 422);
       }

       DB::beginTransaction();
         try {
              $post = Post::create($data);
              DB::commit();
              return response()->json($post, 201);
         } catch (\Exception $e) {
              DB::rollBack();
              return response()->json(['message' => 'Failed to create post!'], 500);
         }
    }
```
Pada fungsi di atas terdapat parameter `$request` yang merupakan request yang dikirimkan oleh client. Pada baris kedua kita mengambil data yang telah divalidasi oleh request. Pada baris keempat kita mengecek apakah ada error pada request, jika ada maka kita akan mengembalikan response berupa error 422. Pada baris keenam kita memulai transaksi database, jika terjadi error maka kita akan melakukan rollback pada database, jika tidak maka kita akan melakukan commit pada database.

3. Buka file `app/Http/Requests/StorePostRequest.php`
4. Rubah kode menjadi seperti ini untuk membuat validasi request

```php
class StorePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'max:255'],
            'content' => ['required', 'string'],
            'author' => ['required', 'integer', 'exists:users,id'],
        ];
    }
}
```
