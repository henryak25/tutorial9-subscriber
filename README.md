1. What is amqp?
    - AMQP (Advanced Message Queuing Protocol) adalah open protocol yang digunakan untuk komunikasi message antar aplikasi atau sistem. Dalam konteks code pada main.rs, amqp://guest:guest@localhost:5672 adalah URI untuk koneksi ke server RabbitMQ menggunakan AMQP. RabbitMQ adalah message broker yang mendukung AMQP, memungkinkan aplikasi untuk mengirim dan menerima message secara asynchronous.

2. What does it mean? guest:guest@localhost:5672 , what is the first guest, and what is the second guest, and what is localhost:5672 is for?
    - amqp://guest:guest@localhost:5672 adalah URI untuk koneksi ke server RabbitMQ menggunakan AMQP. guest:guest adalah pasangan username dan password yang akan digunakan untuk authentication ke RabbitMQ, dimana guest pertama adalah username dan guest kedua adalah password. Pada RabbitMQ, username dan password yang default juga memang guest. localhost berarti host tempat dimana RabbitMQ berjalan, dalam hal ini berarti mengacu pada komputer dimana aplikasinya berjalan. 5672 adalah port yang digunakan oleh RabbitMQ untuk komunikasi menggunakan protokol AMQP. port 5672 adalah port default yang digunakan oleh RabbitMQ.

<br>

### Simulation Slow Subscriber
![slowsubscriber](slowsubscriber.png)
Explanation: <br>
Pada grafik Queued Messages, terlihat adanya spike yang cukup signifikan hingga mencapai angka 20. Hal ini terjadi karena saya menjalankan cargo run pada publisher sebanyak 4 kali secara berurutan dalam waktu singkat. Ketika publisher mengirimkan pesan dengan kecepatan yang lebih tinggi dibanding kemampuan subscriber dalam mengonsumsi pesan, maka akan terjadi penumpukan pesan di queue. Pesan-pesan yang belum sempat diproses oleh subscriber tidak langsung hilang, melainkan disimpan sementara oleh RabbitMQ dalam message queue untuk menunggu giliran diproses. Hal tersebutlah yang menyebabkan terjadinya peningkatan tajam pada jumlah pesan yang mengantre. Lonjakan ini akan menurun secara bertahap seiring dengan subscriber mulai memproses satu per satu pesan dari queue, sampai akhirnya queue kembali kosong jika semua pesan telah dikonsumsi.

<br><br><br><br>

### Reflection and Running at least three subscribers
![graphmulti](graphmulti.png)
![terminal1](terminal1.png)
![terminal2](terminal2.png)
![terminal3](terminal3.png)
<br>
Explanation: <br>
Saya menjalankan cargo run pada publisher sebanyak 7 kali. Hasilnya, ketika diamati di terminal, pesan-pesan yang dikirim berhasil terbagi cukup merata ke tiga subscriber, masing-masing menangani sekitar 10 hingga 11 pesan. Hal ini menunjukkan bahwa dengan menambahkan lebih banyak subscriber, proses konsumsi pesan menjadi jauh lebih efisien dan cepat.<br>
Pada graf queued messages, kita juga dapat melihat bahwa spike jumlah pesan yang sebelumnya tinggi kini menurun dengan signifikan. Penurunan ini terjadi karena proses pengambilan pesan dari antrean kini dilakukan secara paralel oleh tiga subscriber, bukan hanya satu, sehingga antrean dapat dikosongkan lebih cepat.<br>
Ketika publisher mengirim banyak pesan ke dalam antrean, setiap subscriber menerima bagian pesan yang unik dan tidak saling tumpang tindih. Setiap subscriber bertindak seperti instance aplikasi tersendiri, mengambil pesan secara independen. Setelah sebuah pesan diambil oleh salah satu subscriber, pesan tersebut akan langsung dihapus dari antrean dan tidak bisa diakses oleh subscriber lainnya. 