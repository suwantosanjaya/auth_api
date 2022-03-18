# Membuat Remote Repository GitHub

Kita lanjut menggunakan git pada Auth API dan mengunggahnya ke GitHub. Kita mulai dengan membuat remote repository baru pada GitHub.

Silakan login GitHub menggunakan akun Anda dan buat repository baru dengan nama auth-api.

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
> git init

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
```
git add .
git commit -m "initial commit"
```