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