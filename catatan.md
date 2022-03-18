# Menyiapkan Proyek
Awali dengan menyiapkan kebutuhan untuk proyek, database, dan testing supaya kita dapat fokus menulis kode tanpa terganggu proses konfigurasi dan persiapan lain nantinya. Sudah siap? Yuk, langsung simak uraiannya di bawah ini.

Sebelum membuat proyek, pastikan komputer Anda sudah terpasang PostgreSQL dan psql-cli untuk mengakses Postgres melalui Terminal

# Membuat Proyek Node.js
Silakan buat proyek baru dengan nama auth-api, kemudian inisialisasi proyek dengan perintah:

> `npm init -y`

Pastikan berkas `package.json` sudah sesuai dg yang diinstall

Selanjutnya pasang beberapa package dependencies yang dibutuhkan melalui perintah berikut:

> `npm install @hapi/hapi @hapi/jwt bcrypt dotenv nanoid pg`

Kemudian pasang juga beberapa package dev dependencies melalui perintah berikut:

> `npm install @types/jest eslint jest node-pg-migrate nodemon --save-dev`

Setelah proses instalasi dependencies selesai, pastikan seluruh package yang dipasang muncul pada properti dependencies dan devDependecies di package.json.

20210805113751b8dfb911430f0c6a7bde44fe2d87cfe4.jpeg

Selanjutnya, jika Anda ingin menggunakan ESLint pada proyek kali ini, silakan konfigurasi ESLint dan pilih style guide yang ingin diadopsi melalui perintah:

> `npx eslint --init`

Jika Anda tidak ingin menggunakan ESLint, hiraukan perintah tersebut.

Setelah proses pemasangan dependencies selesai, kita akan mengatur script runner pada package.json. Silakan buka berkas package.json, kemudian buat runner script seperti di bawah ini:

```json
"scripts": {
    "start": "node src/app.js",
    "start:dev": "nodemon src/app.js",
    "test": "jest --setupFiles dotenv/config",
    "test:watch": "jest --watchAll --coverage --setupFiles dotenv/config",
    "migrate": "node-pg-migrate",
    "migrate:test": "node-pg-migrate -f config/database/test.json"
  },
```

Berikut penjelasan dari fungsi masing-masing runner script:

> start → Menjalankan aplikasi tahap production.
start:dev → Menjalankan aplikasi tahap development menggunakan nodemon.
test → Menjalankan pengujian di sisi production. Runner ini akan kita gunakan nanti ketika deployment.
test:watch → Menjalankan pengujian setiap kali ada perubahan berkas. Runner digunakan saat development untuk memastikan setiap kode yang ditulis lolos pengujian. Di sana Anda juga akan melihat tambahan parameter --setupFiles yang diberi nilai dotenv/config. Itu berarti Jest akan menggunakan dotenv setiap kali pengujian dijalankan.
migrate → Menjalankan migration pada database yang digunakan oleh aplikasi.
migrate:test → Menjalankan migration pada database yang digunakan untuk proses pengujian.
Sudah paham fungsi dari runner script yang dibuat? Jangan lupa untuk menyimpan perubahan pada berkas package.json ya.

Terakhir, kita siapkan struktur dasar dari proyek Auth API. Silakan buat struktur folder dan berkas app.js seperti ini:

20210805113941d632693856cb4f70d706f4f8c07c616e.jpeg

Mantap! Sampai di sini dulu untuk persiapan proyek Node.js-nya karena kita akan lanjut untuk mempersiapkan kebutuhan database terlebih dahulu.


# Menyiapkan Kebutuhan Database
Saat ini kita akan membuat dua database yang digunakan untuk aplikasi dan proses pengujian. Mengapa memerlukan dua database? Hal tersebut karena pada proses pengujian nantinya akan menyentuh hingga level integrasi dengan database. Proses pengujian akan benar-benar menyimpan data di database walaupun data tersebut hanya sebatas data dumi (dummy). Oleh karena itu, praktik terbaiknya adalah membuat dua database supaya database utama (yang digunakan aplikasi) tidak terganggu proses pengujian.

Silakan masuk ke Postgres Database menggunakan akun postgres (root account) melalui psql dengan perintah berikut:

> `psql --username postgres`

Hasilnya:

2021080511411197fbf91c5aa98cf7a04751505d4c4d80.jpeg

Kemudian buat dua database baru dengan nama authapi dan authapi_test. Gunakan perintah berikut:

```sql
CREATE DATABASE authapi; 
CREATE DATABASE authapi_test;
```

Hasilnya:

20210805114144e19eead7f1df33659197feea30963f7c.jpeg

Jika Anda ingin mendelegasikan pengelolaan database ke user yang lain, Anda bisa gunakan perintah berikut:

```sql
GRANT ALL PRIVILEGES ON DATABASE authapi, authapi_test TO <user>;
```

Ubah <user> dengan user Postgres yang Anda miliki. Contoh, jika Anda memiliki user dengan nama developer, hasilnya menjadi seperti di bawah ini:

202108051142219725713878cda8ea7649fb3f1b4c7727.jpeg

TAMBAHAN BUAT GANTI PASSWORD DI POSTGRES:
> `ALTER USER developer WITH PASSWORD 'supersecretpassword'`


Pembuatan database selesai. Silakan tulis perintah exit untuk keluar dari psql.

Selanjutnya, buat berkas `.env` pada root proyek dan tulis environment variable untuk nilai konfigurasi database seperti ini:

```Ruby
# POSTGRES
PGHOST=localhost
PGUSER=developer
PGDATABASE=authapi
PGPASSWORD=supersecretpassword
PGPORT=5432

# POSTGRES TEST
PGHOST_TEST=localhost
PGUSER_TEST=developer
PGDATABASE_TEST=authapi_test
PGPASSWORD_TEST=supersecretpassword
PGPORT_TEST=5432
```

Sesuaikan nilai PGUSER dan PGPASSWORD dengan kredensial user Postgres yang Anda miliki. Setelah itu, simpan perubahan pada berkas `.env`.

Selanjutnya, buat berkas `test.json` di dalam folder baru dengan alamat `config/database`.

20210805114351860f9a3b45d81950697346df13e3d17f.jpeg

Kemudian dalam berkas tersebut, tulis konfigurasi seperti di bawah ini:

```json
{
  "user": "developer",
  "password": "supersecretpassword",
  "host": "localhost",
  "port": 5432,
  "database": "authapi_test"
}
```

Berkas konfigurasi di atas akan digunakan oleh runner script `migrate:test`. Supaya proses migration dapat mengarah ke database testing, `migrate:test` tidak boleh menggunakan default configuration melainkan harus mengarahkan konfigurasinya ke berkas tersebut. Paham kan maksudnya? Silakan simpan berkas `test.json` dan kita lanjut dengan membuat tabel yang dibutuhkan pada database.

Auth API membutuhkan 2 tabel yakni tabel `users` dan `authentications`. Tabel users digunakan untuk menyimpan data pengguna (user) yang terdaftar pada Auth API. Sedangkan tabel authentications digunakan untuk menyimpan refresh token user yang terautentikasi agar dapat memperbarui autentikasinya. Jadi, seperti apa bentuk kedua tabel yang akan kita buat? Berikut struktur tabelnya.



Tabel: `users`
|	Nama Kolom	|	Tipe Data	|	Constraint |
| ------	|	-----	|	-----	|
|	id	|	VARCHAR(50)	|	PRIMARY KEY	|
|	username |	VARCHAR(50) |	NOT NULL, UNIQUE |
|	password |	TEXT |	NOT NULL |
|	fullname |	TEXT |	NOT NULL |


Tabel: `authentications`
|	Nama Kolom	|	Tipe Data	|	Constraint	|
| --- | --- | --- |
|token|TEXT|NOT NULL|

Setelah mengetahui struktur dari tabel yang akan dibuat, saatnya sekarang kita membuat tabel tersebut pada database aplikasi maupun testing melalui proses migration.

Pertama, kita buat tabel users terlebih dahulu. Silakan buat migrations baru dengan nama `create table users` melalui perintah berikut:

> `npm run migrate create "create table users"`

Pastikan berkas migrations baru terbuat.

2021080511495609374f7f5d89f94215048dd4e2262736.jpeg

Buka berkas migrations tersebut, kemudian tulis kode dalam membuat tabel users seperti ini:

```js
/* eslint-disable camelcase */
exports.up = (pgm) => {
  pgm.createTable('users', {
    id: {
      type: 'VARCHAR(50)',
      primaryKey: true,
    },
    username: {
      type: 'VARCHAR(50)',
      notNull: true,
      unique: true,
    },
    password: {
      type: 'TEXT',
      notNull: true,
    },
    fullname: {
      type: 'TEXT',
      notNull: true,
    },
  });
};

exports.down = (pgm) => {
  pgm.dropTable('users');
};
```

Simpan berkas migrations tersebut dan kita lanjutkan dengan membuat berkas migrations kedua untuk tabel `authentications`.

Silakan buat berkas migration dengan nama `create table authentications` melalui perintah berikut:

> `npm run migrate create "create table authentications"`

Pastikan berkas migrations baru terbuat:

2021080511524228c95fe63a43594b29b3fb98687a805a.jpeg

Buka berkas migration dan tulis kode dalam membuat tabel authentications.

```js
/* eslint-disable camelcase */

exports.up = (pgm) => {
  pgm.createTable('authentications', {
    token: {
      type: 'TEXT',
      notNull: true,
    },
  });
};

exports.down = (pgm) => {
  pgm.dropTable('authentications');
};
```

Simpan berkas migration tersebut. Selanjutnya kita akan mengeksekusi migration pada database aplikasi dan testing.

Silakan eksekusi kedua migration pada database aplikasi dengan perintah:

> `npm run migrate up`

Hasilnya:

20210805115348f3a5f0b67d22aafbbaad3706ec11cfad.jpeg

Eksekusi juga migrations untuk database testing dengan perintah:
> `npm run migrate:test up`

Hasilnya:

20210805115428ce7ecc99b7b440573c78c47186071981.jpeg

Mantap! Sekarang database `authap`i dan `authapi_test` memiliki tabel `users` dan `authentications` di dalamnya.

Selanjutnya, kita buat pool connection yang dibutuhkan dalam mengakses postgres database. Silakan buat berkas `pool.js` di dalam folder baru dengan lokasi `Infrastructures/database/postgres`.

20210805115511a5b931950c66d1b2f517f10562e86862.jpeg

Kemudian di dalam berkas pool.js tuliskan kode berikut:

```js
/* istanbul ignore file */
const { Pool } = require('pg');

const testConfig = {
  host: process.env.PGHOST_TEST,
  port: process.env.PGPORT_TEST,
  user: process.env.PGUSER_TEST,
  password: process.env.PGPASSWORD_TEST,
  database: process.env.PGDATABASE_TEST,
};

const pool = process.env.NODE_ENV === 'test' ? new Pool(testConfig) : new Pool();

module.exports = pool;
```

Komentar kode `istanbul ignore file` berfungsi untuk memberitahu `Jest` bahwa berkas ini (`pool.js`) tidak perlu diuji. Berkas ini akan diabaikan oleh jest dan tidak akan masuk ke dalam laporan coverage test.

Connection pool digunakan untuk mengakses database aplikasi maupun pengujian. Nilai dari konfigurasi connection pool ditentukan oleh nilai environment variable `NODE_ENV`. Jika nilai `NODE_ENV` tersebut `“test”`, connection pool akan menggunakan database testing. Selain itu, connection pool akan menggunakan database utama.

Sebagai informasi, ketika node process dijalankan oleh Jest (itu berarti menjalankan pengujian), environment variable `NODE_ENV` secara otomatis akan bernilai `“test”`. Jadi, kita tidak perlu mendefinisikan nilai `NODE_ENV` secara manual.

Simpan perubahan pada berkas `pool.js` dan kebutuhan database sudah selesai.


# Menyiapkan Kebutuhan Testing
Setelah kebutuhan database sudah siap, sekarang kita lanjut menyiapkan beberapa kebutuhan untuk pengujian nanti. Kebutuhannya adalah membuat `class Helper` yang berfungsi untuk mengelola dan membersihkan database pengujian selama kebutuhan testing nantinya.

Silakan buat folder baru bernama `tests` di *root proyek*. Kemudian buat dua berkas JavaScript baru di dalamnya dengan nama `UsersTableTestHelper.js` dan `AuthenticationsTableTestHelper.js`.

20210805115738bb38805621f81b71b619bc385127e452.jpeg

Di dalam kedua berkas tersebut, tuliskan kode berikut:

UsersTableTestHelper.js
```js
/* istanbul ignore file */
const pool = require('../src/Infrastructures/database/postgres/pool');

const UsersTableTestHelper = {
  async addUser({
    id = 'user-123', username = 'dicoding', password = 'secret', fullname = 'Dicoding Indonesia',
  }) {
    const query = {
      text: 'INSERT INTO users VALUES($1, $2, $3, $4)',
      values: [id, username, password, fullname],
    };

    await pool.query(query);
  },

  async findUsersById(id) {
    const query = {
      text: 'SELECT * FROM users WHERE id = $1',
      values: [id],
    };

    const result = await pool.query(query);
    return result.rows;
  },

  async cleanTable() {
    await pool.query('TRUNCATE TABLE users');
  },
};

module.exports = UsersTableTestHelper;
```

AuthenticationsTableTestHelper.js
```js
/* istanbul ignore file */
const pool = require('../src/Infrastructures/database/postgres/pool');
 
const AuthenticationsTableTestHelper = {
  async addToken(token) {
    const query = {
      text: 'INSERT INTO authentications VALUES($1)',
      values: [token],
    };
 
    await pool.query(query);
  },
 
  async findToken(token) {
    const query = {
      text: 'SELECT token FROM authentications WHERE token = $1',
      values: [token],
    };
 
    const result = await pool.query(query);
 
    return result.rows;
  },
  async cleanTable() {
    await pool.query('TRUNCATE TABLE authentications');
  },
};
 
module.exports = AuthenticationsTableTestHelper;
```

Ingat, kode di atas hanya digunakan untuk membantu proses pengujian saja, tidak digunakan sebagai kode production. Jadi Anda tidak perlu khawatir bila kode tidak dituliskan dengan TDD, karena memang kode tersebut tidak perlu diuji.

Simpan perubahan kedua berkas tersebut. Kini persiapan kebutuhan pengujian sudah selesai.






# Membuat Service Locator
> `npm install instances-container`

