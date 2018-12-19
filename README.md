# TCC - Infrastructure as a Code

**Infrastructure as a Code is ?** Infrastruktur sebagai Kode (IaC) adalah manajemen infrastruktur (jaringan, mesin virtual, load balancer, dan topologi koneksi) dalam model deskriptif, menggunakan versi yang sama seperti yang digunakan tim DevOps untuk kode sumber. Seperti prinsip bahwa kode sumber yang sama menghasilkan biner yang sama, model IaC menghasilkan lingkungan yang sama setiap kali itu diterapkan. IaC adalah praktik utama DevOps dan digunakan bersamaan dengan pengiriman berkelanjutan.

Infrastruktur sebagai Kode berevolusi untuk memecahkan masalah penyimpangan lingkungan dalam pipa rilis. Tanpa IaC, tim harus mempertahankan pengaturan lingkungan penyebaran individu. Seiring waktu, setiap lingkungan menjadi keping salju, yaitu, konfigurasi unik yang tidak dapat direproduksi secara otomatis. Inkonsistensi di antara lingkungan menyebabkan masalah selama penerapan. Dengan kepingan salju, administrasi dan pemeliharaan infrastruktur melibatkan proses manual yang sulit dilacak dan menyebabkan kesalahan.

Untuk menerapkan IAC ada beberapa tools yang dapat digunakan. salah satunya adalah [Salt Stack](https://www.saltstack.com/). Disini saya akan menggunakan Salt Stack untuk sedikit mendemokan Infrastructure as a Code.

**Apa itu Salt ?** 
Salt adalah pendekatan baru untuk manajemen infrastruktur yang dibangun di atas bus komunikasi yang dinamis. Garam dapat digunakan untuk orkestrasi berbasis data, eksekusi jarak jauh untuk infrastruktur apa pun, pengelolaan konfigurasi untuk tumpukan aplikasi apa pun, dan banyak lagi

Disini saya akan mencoba menggunakan Salt untuk membuat Formula menggunakan Vim, Apache web server, PHP, da Git

Pertama saya melakukan Installasi SaltStack, mengikuti instruksi dari dokumentasi disini:
https://docs.saltstack.com/en/latest/topics/installation/index.html

Setelah Installasi, dapat dimulai untuk menjalankan SaltStack.
Formula Salt secara default adalah yaml teks yang disimpan pada direktori :
>/srv/salt/

Pastikan vim sudah terinstall jika belum, untuk menginstallnya dapat dilakukan seperti berikut : 
>/srv/salt/vim.sls

Tambahkan teks berikut ke vim.sls :
> - vim:
>   - pkg:
>       - installed

Baris pertama disebut Deklarasi ID ;  vim adalah untuk nama paket.

Baris kedua disebut Deklarasi State atau State Declaration. Ini mengacu pada State Salt khusus yang akan saya manfaatkan. Dalam contoh ini saya menggunakan status "pkg".

Baris ketiga disebut Deklarasi Fungsi . Ini mengacu pada nama fungsi di dalam modul negara yang akan kita install.

Sekarang kita dapat menerapkan status ini ke server :
> salt 'minion01' state.sls vim

Minion1 adalah nama server yang saya gunakan.

Selanjutnya saya akan menginstal server web Apache dan PHP pada saat yang bersamaan. Buat file bernama "webserver.sls" 
> vi /srv/salt/webserver.sls

tambahkan paket berikut ke "webserver.sls"
webserver_stuff:
>  pkg:
>    - installed
>    - pkgs:
>      - apache2
>      - php5
>      - php5-mysql

Argumen "- pkgs:". berarti setiap item dalam daftar di bawah "- pkgs:" akan diteruskan bersama ke manajer paket OS untuk dipasang bersama. Ini berarti hanya satu panggilan ke "apt" atau "yum" yang akan terjadi. Jika kita memiliki daftar paket yang besar, ini adalah cara yang paling efisien untuk menginstalnya.

Selanjutnya jalankan perintah dibawah untuk menerapkan :
> salt 'minion01' state.sls webserver

Selanjutnya saya akan menginstall Git, Caranya adalah seperti berikut :
> vi /srv/salt/git.sls

kemudian isikan paket kedalamnya 

> - git:
>   - pkg:
>   - installed

Dan lagi, Untuk menerapkan konfigurasi baru ini jalankan lagi perintah :
>salt 'minion01' state.sls git

Jika kita ingin menerapkan masing-masing konfigurasi ini pada saat yang sama kita dapat mengeksekusi hal berikut: 
>salt 'minion01' state.sls vim,webserver,git

Selanjutnya, setelah semua paket diinstall, kita bisa mengecek highstate dari paket yg telah kita buat. "highstate" adalah cara Salt untuk secara dinamis menentukan Formula Salt yang harus diterapkan pada minion tertentu. 
Untuk menjalankanya digunakan perintah dibawah ini :
> salt 'minion01' state.highstate

Perintah diatas akan menjalankan Minion untuk mengunduh dan memeriksa file dari Salt Master yang disebut "highstate". Secara default file ini ditemukan di Salt Master di /srv/salt/top.sls
tampilanya adalah seperti berikut :

>base:
>  - '*':
>    - vim
>  - 'minion*':
>    - git
>    - webserver
> - 'minion02':
>    - mongodb

Dari hasil top.sls ini dapat dilihat disini kita berada pada enviroment "base". ini adalah enviroment default. Ketika Minion diperintahkan untuk mengeksekusi highstate seperti yang dilakukan sebelumnya, Minion akan meminta top.sls dari master dan mencari formula yang cocok. '*' Adalah wildcard dan berarti semua Minion harus menerapkan daftar rumus di bawahnya; dalam hal ini hanya formula "vim". Baris keempat, 'minion *', juga cocok dengan minion saya. Jadi itu berarti Minion saya akan menerapkan formula "git" dan "webserver". Minion saya tidak cocok dengan 'minion02' sehingga minion saya tidak akan mencoba menerapkan rumus "mongodb".

Dalam dalam Demo ini minion hanya mencocokkan pada ID atau nama minion dengan globbing standar. minion juga dapat mencocokkan pada pcre, alamat ip dan rentang alamat ip, dan berbagai hal lainnya. ketentuanya dapat dilihat di sini.
https://docs.saltstack.com/en/latest/topics/targeting/compound.html