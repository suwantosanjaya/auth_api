# Membuat Remote Repository GitHub

Kita lanjut menggunakan git pada Auth API dan mengunggahnya ke GitHub. Kita mulai dengan membuat remote repository baru pada GitHub.

Silakan login GitHub menggunakan akun Anda dan buat repository baru dengan nama auth_api.

Setelah berhasil di buat maka akan muncul seperti berikut:

…or create a new repository on the command line
```
echo "# auth_api" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/suwantosanjaya/auth_api.git
git push -u origin main
```

…or push an existing repository from the command line
```
git remote add origin https://github.com/suwantosanjaya/auth_api.git
git branch -M main
git push -u origin main
```

Sekarang kita akan pasang git pada proyek Auth API untuk membuat local repository. Silakan buka kembali proyek Auth API pada VSCode. Kemudian pasang git pada proyek menggunakan perintah:

```bash
git init
```

**Catatan**: Jika gagal, pastikan Anda sudah memasang git pada komputer.

Selanjutnya buat berkas `.gitignore` pada proyek. Tulislah beberapa folder dan file penampung kredensial yang tidak perlu dikelola oleh git pada proyek. Silakan tulis kode berikut:
```
node_modules
coverage
.env
auth-api-app-server.pem
package-lock.json
config/database/test.json
```

Setelah itu masukkan seluruh berkas (selain yang berada di .gitignore) ke stage change dan commit dengan pesan “initial commit” melalui kode berikut:
```bash
git add .
git commit -m "initial commit"
```

Selanjutnya tambahkan remote repository yang sudah kita buat di GitHub pada local repository dengan perintah:

```bash
git remote add origin <remote repository URL>
```

Silakan ganti <remote repository URL> dengan URL url remote repository GitHub yang Anda miliki. Contoh:

```bash
git remote add origin https://github.com/https://github.com/suwantosanjaya/auth_api.git
```

Setelah itu, push branch master (branch utama) pada local repository ke remote repository dengan perintah:

```bash
git push origin master
```

Proses ini membutuhkan authorization pada akun GitHub. Untuk Linux dan macOS, Anda bisa menuliskan username dan password github secara langsung pada Terminal. Namun, jika menggunakan Windows, Anda bisa login melalui GitHub desktop.

Silakan kembali ke halaman remote repository GitHub pada browser. Refresh halaman tersebut dan Anda akan melihat source code proyek di sana.



## Membuat CI Pipeline menggunakan GitHub Action

Mari kita membuat CI Pipeline menggunakan GitHub Action terlebih dahulu. Caranya yaitu dengan membuat berkas .yml di dalam folder `.github/workflows` repository. Dalam berkas tersebut, definisikan kebutuhan dan juga alur kerja (workflow) untuk menjalankan sebuah aksi. Satu berkas .yml akan dibaca sebagai satu buah aksi.

Silakan buka kembali proyek Auth API dan buat berkas baru bernama `ci.yml` pada folder `.github/workflows` **(buat folder baru)**.

Di dalamnya, tulis kode berikut:

- [ci.yml](https://www.dicoding.com/academies/276/tutorials/19067?from=19062#tab1-code1)

```yaml
name: Continuous Integration 
on:   pull_request:    
	branches:      
		- master 

jobs:  
	test:   
    	runs-on: ubuntu-latest
    	
    	strategy:
    		matrix:
    			node-version: [14.x]        
    			# See supported Node.js release schedule at https://nodejs.org/en/about/releases/     
    	
    	steps:    
    		- uses: actions/checkout@v2    
    		- name: Use Node.js ${{ matrix.node-version }}      
    		  uses: actions/setup-node@v2      
    		  with:        
    		  	node-version: ${{ matrix.node-version }}    
    		 - name: npm install migrate and test      
    		   run: |        
    		   	npm install        
    		   	npm run migrate up        
    		   	npm run test      
    		   env:        
    		   	CI: true        
    		   	PGHOST_TEST: ${{ secrets.PGHOST_TEST }}        
    		   	PGUSER_TEST: ${{ secrets.PGUSER_TEST }}        
    		   	PGDATABASE_TEST: ${{ secrets.PGDATABASE_TEST }}        
    		   	PGPASSWORD_TEST: ${{ secrets.PGPASSWORD_TEST }}        
    		   	PGPORT_TEST: ${{ secrets.PGPORT_TEST }}        
    		   	PGHOST: ${{ secrets.PGHOST_TEST }}        
    		   	PGUSER: ${{ secrets.PGUSER_TEST }}        
    		   	PGDATABASE: ${{ secrets.PGDATABASE_TEST }}        
    		   	PGPASSWORD: ${{ secrets.PGPASSWORD_TEST }}        
    		   	PGPORT: ${{ secrets.PGPORT_TEST }}        
    		   	ACCESS_TOKEN_KEY: ${{ secrets.ACCESS_TOKEN_KEY }}        
    		   	REFRESH_TOKEN_KEY: ${{ secrets.REFRESH_TOKEN_KEY }}
```

> **Catatan:**
> Pastikan spasi dan tab sama dengan yang ada di modul karena dalam bahasa YAML hal ini sangat berpengaruh. Jika Anda belum familier dengan bahasa YAML, silakan pelajari secara singkat pada tautan berikut: [Learn yaml in Y Minutes](https://learnxinyminutes.com/docs/yaml/).



GitHub Actions (dan kebanyakan tools CI lainnya) menggunakan format YAML untuk mendefinisikan alur dari sebuah actions. Format YAML cocok untuk digunakan karena alur kerja yang didefinisikan relatif mudah untuk dibaca.

Kami tidak akan menjelaskan secara detail bagaimana kode GitHub Actions ditulis, tetapi hanya bagian-bagian penting saja. Pada bagian terluar berkas `ci.yml` terdapat properti **name**, **on**, dan **jobs**. Berikut penjelasan dari masing-masing properti tersebut:

- **name**: merupakan nama dari actions. Karena ini CI pipeline, maka namanya adalah Continuous Integrations.
- **on**: merupakan event trigger atau event yang dapat men-trigger kapan actions harus berjalan. Di properti ini, kita mendefinisikan **pull_request** pada branch **master** sebagai event trigger.
- **jobs**: merupakan properti yang mendefinisikan pekerjaan yang dilakukan ketika event trigger dibangkitkan. Satu GitHub Actions dapat terdiri dari beberapa jobs. Namun pada contoh kali ini, kita hanya mendefinisikan satu jobs yaitu **test**. Di dalam job test kita definisikan alur dan cara kerjanya, seperti mendefinisikan plugin yang digunakan, scripts yang perlu dijalankan, dan menetapkan nilai-nilai environment yang diperlukan.

Anda juga mungkin menyadari beberapa nilai environment dituliskan dengan memanfaatkan nilai dari properti milik secrets. Yups, nantinya seluruh nilai environment akan kita simpan pada *repository secrets*.

Bila Anda ingin mengetahui lebih dalam bagaimana menuliskan workflow pada GitHub Actions, kami sangat merekomendasikan untuk membaca dokumentasi yang diberikan langsung oleh GitHub mengenai [GitHub Actions](https://docs.github.com/en/actions). Oke, mari kita lanjutkan.

Simpan perubahan pada berkas tersebut, kemudian commit dan push perubahannya dengan perintah:

```bash
git add .
git commit -m "add ci action"
git push origin master
```

Setelah berhasil push maka repository GitHub auth-api sudah memiliki actions CI yang ter-trigger bila ada pull request pada branch master.

Sebelum mencoba pull request, kita perlu menyiapkan environment variable yang dibutuhkan oleh proses CI terlebih dahulu. Karena nilai environment variable tersebut bersifat rahasia atau sensitif, maka simpan nilainya pada repository secrets.

Silakan buka halaman repository GitHub proyek Auth API Anda. Kemudian klik tab **Settings**.

Kemudian pilih menu **Secrets** pada panel samping kiri halaman setting.

Setelah itu klik tombol **New repository secret** untuk membuat repository secret baru.

Selanjutnya Anda akan diarahkan ke halaman input secret.

Pada kolom input **Name** isi dengan nama environment variable. Kemudian untuk input Value, isi dengan nilai environment variable. Contoh name **PGHOST_TEST** dengan nilai **auth-api.csadc8kpllpa.ap-southeast-1.rds.amazonaws.com** (sesuaikan dengan host/endpoint RDS server Anda):

Untuk menambah dan menyimpan nilai secrets, silakan klik tombol **Add secret**.

Pastikan nilai secrets yang baru saja kita tambahkan muncul pada Repository secrets.



Silakan tambahkan repository secret lain dengan nilai-nilai berikut:

- PGUSER_TEST: **postgres** (sesuaikan dengan kredensial Anda)
- PGPASSWORD_TEST: **supersecret** (sesuaikan dengan kredensial Anda)
- PGDATABASE_TEST: **authapi_test**
- PGPORT_TEST: **5432**
- PGHOST: **auth-api.csadc8kpllpa.ap-southeast-1.rds.amazonaws.com** (sesuaikan dengan host/endpoint RDS server Anda)
- PGUSER: **postgres** (sesuaikan dengan kredensial Anda)
- PGDATABASE: **authapi_test**
- PGPASSWORD: **supersecret** (sesuaikan dengan kredensial Anda)
- PGPORT: **5**432
- ACCESS_TOKEN_KEY: (access token key Anda)
- REFRESH_TOKEN_KEY: (refresh token key Anda)

**Catatan:** Untuk nilai **PGDATABASE**, tetap gunakan database testing (**authapi_test**). Hal tersebut karena sejatinya proses CI tidak perlu bersentuhan dengan database production. Nilai **PGDATABASE** pada proses CI digunakan dalam proses migration database menggunakan perintah **npm run migrate**.



Sekarang, kita bisa coba trigger CI dengan membuat pull request ke branch master. Caranya:

- Buat branch baru di lokal repository,
- Push branch baru ke remote repository.
- Pull request branch baru ke branch master.

Yuk kita mulai. Silakan buka kembali proyek Auth API pada VSCode dan anggaplah kita membuat fitur baru yakni “say hello world”. Jadi, buat branch baru dengan nama “feature-say-hello-world” dengan perintah berikut:

```bash
git checkout -b feature-say-hello-world
```

**Catatan**: perintah tersebut digunakan untuk membuat sekaligus berpindah ke branch **feature-say-hello-world**.

Hasilnya:



Selanjutnya buat fitur say hello world pada Endpoint **GET** /. Seperti biasa, kita tulis testingnya terlebih dahulu. Tambahkan skenario testing pada kode yang diberi tanda tebal.

- [createServer.test.js](https://www.dicoding.com/academies/276/tutorials/19067?from=19062#tab2-code1)



```
const pool = require('../../database/postgres/pool');const UsersTableTestHelper = require('../../../../tests/UsersTableTestHelper');const AuthenticationsTableTestHelper = require('../../../../tests/AuthenticationsTableTestHelper');const injections = require('../../injections');const createServer = require('../createServer'); describe('HTTP server', () => {  afterAll(async () => {    await pool.end();  });   afterEach(async () => {    await UsersTableTestHelper.cleanTable();    await AuthenticationsTableTestHelper.cleanTable();  });   it('should response 404 when request unregistered route', async () => {    // Arrange    const server = await createServer({});     // Action    const response = await server.inject({      method: 'GET',      url: '/unregisteredRoute',    });     // Assert    expect(response.statusCode).toEqual(404);  });   describe('when GET /', () => {    it('should return 200 and hello world', async () => {      // Arrange      const server = await createServer({});      // Action      const response = await server.inject({        method: 'GET',        url: '/',      });      // Assert      const responseJson = JSON.parse(response.payload);      expect(response.statusCode).toEqual(200);      expect(responseJson.value).toEqual('Hello world!');    });  });   // Skenario testing lain ...});
```

Jalankan testingnya dan hasilnya akan merah:

[![202108072200233417857a0fab720a4d3a4cbff3220adc.jpeg](https://d17ivq9b7rppb3.cloudfront.net/original/academy/202108072200233417857a0fab720a4d3a4cbff3220adc.jpeg)](https://www.dicoding.com/academies/276/tutorials/19067?from=19062#)

Sebelum memperbaiki pengujiannya, alangkah baiknya kita coba dulu pull request dengan skenario gagal seperti ini. Hal tersebut supaya kita mengetahui apa yang terjadi jika Anda atau anggota tim melakukan pull request dengan pengujian yang gagal.

Silakan commit dan push perubahan ke branch feature-say-hello-world ke remote repository dengan menggunakan perintah:



```
git add .git commit -m "add say hello world test case"git push origin feature-say-hello-world 
```

Hasilnya:

[![202108072200248091d4ddd509f222cb8d2149a44c4ea6.jpeg](https://d17ivq9b7rppb3.cloudfront.net/original/academy/202108072200248091d4ddd509f222cb8d2149a44c4ea6.jpeg)](https://www.dicoding.com/academies/276/tutorials/19067?from=19062#)

Sekarang kembali ke halaman GitHub repository Auth API Anda. Seharusnya terdapat branch baru yaitu feature-say-hello-world.

[![20210807220206529fbfc7943e4e5042a201368d168c9e.jpeg](https://d17ivq9b7rppb3.cloudfront.net/original/academy/20210807220206529fbfc7943e4e5042a201368d168c9e.jpeg)](https://www.dicoding.com/academies/276/tutorials/19067?from=19062#)

Buat pull request dengan klik tombol **View all branches**.

[![202108072202057bfd803c4b298191085d3f1b28d4fed3.jpeg](https://d17ivq9b7rppb3.cloudfront.net/original/academy/202108072202057bfd803c4b298191085d3f1b28d4fed3.jpeg)](https://www.dicoding.com/academies/276/tutorials/19067?from=19062#)

Kemudian pada halaman daftar branches, buat pull request baru dengan klik tombol **New pull request** dari branch **feature-say-hello-world**.

[![20210807220206b46d9c215c703b214012bee4a7cc2a3e.jpeg](https://d17ivq9b7rppb3.cloudfront.net/original/academy/20210807220206b46d9c215c703b214012bee4a7cc2a3e.jpeg)](https://www.dicoding.com/academies/276/tutorials/19067?from=19062#)

Secara default pull request akan dilakukan dari branch feature-say-hello-world ke branch master (utama).

[![20210807220206366fbde85285f7fa631cbc6a3ba1966d.jpeg](https://d17ivq9b7rppb3.cloudfront.net/original/academy/20210807220206366fbde85285f7fa631cbc6a3ba1966d.jpeg)](https://www.dicoding.com/academies/276/tutorials/19067?from=19062#)

Biarkan konfigurasi tetap seperti default dan klik **Create pull request** untuk mulai membuat pull request.

[![2021080722020655225d98b709e542adce4c00d7b1ed90.jpeg](https://d17ivq9b7rppb3.cloudfront.net/original/academy/2021080722020655225d98b709e542adce4c00d7b1ed90.jpeg)](https://www.dicoding.com/academies/276/tutorials/19067?from=19062#)

Setelah pull request berhasil dibuat, maka actions CI akan berjalan seperti pada gambar di atas.

Karena kita melakukan pull request dengan hasil pengujian yang gagal, maka CI pun akan memberitahukan bahwa proses pengujiannya gagal.

[![20210807220204f8226d5d27f17c185c709054c1b5f899.jpeg](https://d17ivq9b7rppb3.cloudfront.net/original/academy/20210807220204f8226d5d27f17c185c709054c1b5f899.jpeg)](https://www.dicoding.com/academies/276/tutorials/19067?from=19062#)

Dari sini kita tahu bahwa kode yang di PR oleh Anda atau tim tidaklah benar. Detail kesalahan yang terjadi dapat dilihat dengan klik tautan **Details**.

[![2021080722020652245622913b546dce0ced93b81f4f80.jpeg](https://d17ivq9b7rppb3.cloudfront.net/original/academy/2021080722020652245622913b546dce0ced93b81f4f80.jpeg)](https://www.dicoding.com/academies/276/tutorials/19067?from=19062#)

Kembali ke halaman pull request. Karena proses pengujian pada CI gagal, reviewer bisa langsung close atau menolak pull request dengan klik tombol **Close pull request**.

[![20210807220205ec26f05540b107f90483bc1e28dfdb5f.jpeg](https://d17ivq9b7rppb3.cloudfront.net/original/academy/20210807220205ec26f05540b107f90483bc1e28dfdb5f.jpeg)](https://www.dicoding.com/academies/276/tutorials/19067?from=19062#)

Sekarang, kita coba perbaiki dengan membuat pengujiannya hijau.

Silakan buka kembali VSCode dan buat pengujiannya menjadi hijau dengan menuliskan kode yang diberi tanda tebal pada fungsi createServer:

- [createServer.js](https://www.dicoding.com/academies/276/tutorials/19067?from=19062#tab3-code1)



```
const Hapi = require('@hapi/hapi');const ClientError = require('../../Commons/exceptions/ClientError');const DomainErrorTranslator = require('../../Commons/exceptions/DomainErrorTranslator');const users = require('../../Interfaces/http/api/users');const authentications = require('../../Interfaces/http/api/authentications'); const createServer = async (injections) => {  const server = Hapi.server({    host: process.env.HOST,    port: process.env.PORT,  });   await server.register([    {      plugin: users,      options: { injections },    },    {      plugin: authentications,      options: { injections },    },  ]);   server.route({    method: 'GET',    path: '/',    handler: () => ({      value: 'Hello world!',    }),  });   server.ext('onPreResponse', (request, h) => {    // mendapatkan konteks response dari request    const { response } = request;     if (response instanceof Error) {      // bila response tersebut error, tangani sesuai kebutuhan      const translatedError = DomainErrorTranslator.translate(response);       // penanganan client error secara internal.      if (translatedError instanceof ClientError) {        const newResponse = h.response({          status: 'fail',          message: translatedError.message,        });        newResponse.code(translatedError.statusCode);        return newResponse;      }       // mempertahankan penanganan client error oleh hapi secara native, seperti 404, etc.      if (!translatedError.isServer) {        return h.continue;      }       // penanganan server error sesuai kebutuhan      const newResponse = h.response({        status: 'error',        message: 'terjadi kegagalan pada server kami',      });      newResponse.code(500);      return newResponse;    }     // jika bukan error, lanjutkan dengan response sebelumnya (tanpa terintervensi)    return h.continue;  });   return server;}; module.exports = createServer;
```

Jalankan testing dan hasilnya akan hijau:

[![202108072213501646697b56983a8060c131a22cf81269.jpeg](https://d17ivq9b7rppb3.cloudfront.net/original/academy/202108072213501646697b56983a8060c131a22cf81269.jpeg)](https://www.dicoding.com/academies/276/tutorials/19067?from=19062#)

Kita commit perubahan dan push kembali branch **feature-say-hello-world** ke remote repository menggunakan perintah:



```
git add .git commit -m "pass say hello world test case"git push origin feature-say-hello-world
```

Hasilnya:

[![202108072213501b0c3055a7d97bfd7680d5790dcf3bf0.jpeg](https://d17ivq9b7rppb3.cloudfront.net/original/academy/202108072213501b0c3055a7d97bfd7680d5790dcf3bf0.jpeg)](https://www.dicoding.com/academies/276/tutorials/19067?from=19062#)

Kembali ke GitHub Repository. Sekarang kita buat pull request lagi dengan cara yang sama yaitu *View all branches* -> *New pull request* pada branch *feature-say-hello-world*.

[![2021080722135102a2641e22d549ff9f6d64207501494b.jpeg](https://d17ivq9b7rppb3.cloudfront.net/original/academy/2021080722135102a2641e22d549ff9f6d64207501494b.jpeg)](https://www.dicoding.com/academies/276/tutorials/19067?from=19062#)

Klik **Create pull request** untuk mulai membuat pull request baru.

Setelah pull request berhasil dibuat, tunggu proses CI hingga selesai dan lolos.

[![20210807222049c3b4c5592f82b3303588c95a5dd1f12f.jpeg](https://d17ivq9b7rppb3.cloudfront.net/original/academy/20210807222049c3b4c5592f82b3303588c95a5dd1f12f.jpeg)](https://www.dicoding.com/academies/276/tutorials/19067?from=19062#)

Proses CI selesai dan lulus. Sehingga, reviewer bisa merge pull request dengan aman. Namun, sebelum melakukan merge pull request, kita buat dulu CD pipeline agar perubahan kode langsung bisa di-deploy secara otomatis ke server EC2.