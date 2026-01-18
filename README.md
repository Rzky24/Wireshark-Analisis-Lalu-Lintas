# Wireshark: Analisis Lalu Lintas

<img width="853" height="334" alt="image" src="https://github.com/user-attachments/assets/7ddd57e8-331b-4ae0-852c-5ab8d595e9c9" />
# Pemindaian Nmap
Pemindaian Nmap
Nmap adalah alat standar industri untuk memetakan jaringan, mengidentifikasi host aktif, dan menemukan layanan. Karena merupakan salah satu alat pemindai jaringan yang paling banyak digunakan, seorang analis keamanan harus mengidentifikasi pola jaringan yang dibuat dengannya. Bagian ini akan membahas identifikasi jenis pemindaian Nmap yang paling umum .

Pemindaian koneksi TCP
Pemindaian SYN
Pemindaian UDP
Penting untuk mengetahui cara kerja pemindaian Nmap untuk mendeteksi aktivitas pemindaian di jaringan. Namun, mustahil untuk memahami detail pemindaian tanpa menggunakan filter yang tepat. Berikut adalah filter dasar untuk menyelidiki perilaku pemindaian Nmap di jaringan. 

Penjelasan singkat tentang flag TCP :

Catatan	Filter Wireshark
Pencarian global.	
tcp
udp
Hanya flag SYN.
Bendera SYN telah diatur. Bit lainnya tidak penting.
tcp.flags == 2
tcp.flags.syn == 1
Hanya bendera ACK.
Bendera ACK telah diatur. Bit lainnya tidak penting.
tcp.flags == 16
tcp.flags.ack == 1
Hanya flag SYN dan ACK.
SYN dan ACK telah diatur. Bit lainnya tidak penting.
tcp.flags == 18
(tcp.flags.syn == 1) and (tcp.flags.ack == 1)
Hanya bendera RST.
Bendera RST telah diatur. Bit lainnya tidak penting.
 
tcp.flags == 4
tcp.flags.reset == 1
Hanya flag RST dan ACK.
RST dan ACK telah diatur. Bit lainnya tidak penting.
tcp.flags == 20
(tcp.flags.reset == 1) and (tcp.flags.ack == 1)
Hanya bendera FIN
Bendera FIN telah diatur. Bit lainnya tidak penting.
tcp.flags == 1
tcp.flags.fin == 1
Pemindaian Koneksi TCP
Secara singkat, TCP Connect Scan:

Mengandalkan jabat tangan tiga arah (perlu menyelesaikan proses jabat tangan).
Biasanya dilakukan dengan nmap -sT perintah.
Digunakan oleh pengguna non-hak istimewa (satu-satunya pilihan untuk pengguna non-root).
Biasanya memiliki ukuran jendela yang lebih besar dari 1024 byte karena permintaan tersebut mengharapkan sejumlah data karena sifat protokolnya.
Buka Port TCP	Buka Port TCP
Port TCP Tertutup
SYN -->
<-- SYN, ACK
ACK -->

Buka port TCP (Hubungkan):
SYN -->
<-- SYN, ACK
ACK -->
RST, ACK -->
SYN -->
<-- RST, ACK
Gambar-gambar di bawah ini menunjukkan proses jabat tangan tiga arah untuk membuka dan menutup port TCP. Gambar dan sampel pcap dipisahkan untuk mempermudah investigasi dan memahami detail setiap kasus.
<img width="1566" height="318" alt="image" src="https://github.com/user-attachments/assets/4000ed92-1c27-4e6e-b871-7c895a5a7aa7" />

Port TCP tertutup (Hubungkan):
<img width="1567" height="256" alt="image" src="https://github.com/user-attachments/assets/a15aaefc-50f8-4fcf-9142-22be4036f71a" />

Gambar-gambar di atas menunjukkan pola-pola dalam lalu lintas yang terisolasi. Namun, tidak selalu mudah untuk menemukan pola-pola tersebut dalam file tangkapan yang besar. Oleh karena itu, analis perlu menggunakan filter generik untuk melihat pola anomali awal, dan kemudian akan lebih mudah untuk fokus pada titik lalu lintas tertentu. Filter yang diberikan menunjukkan pola pemindaian TCP Connect dalam file tangkapan.

tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size > 1024 
<img width="1561" height="415" alt="image" src="https://github.com/user-attachments/assets/19d2b5b7-e57e-4666-8836-e18c2027d15d" />

Pemindaian SYN
TCP SYN Scan secara singkat:

Tidak bergantung pada jabat tangan tiga arah (tidak perlu menyelesaikan proses jabat tangan).
Biasanya dilakukan dengan  nmap -sS perintah.
Digunakan oleh pengguna dengan hak akses istimewa.
Biasanya ukurannya kurang dari atau sama dengan 1024 byte karena permintaan belum selesai dan tidak mengharapkan untuk menerima data.
Buka Port TCP	Tutup Port TCP
SYN -->
<-- SYN,ACK
RST-->
SYN -->
<-- RST,ACK
Buka port TCP (SYN):

Port TCP tertutup (SYN):

Filter yang diberikan menunjukkan  pola pemindaian TCP  SYN dalam file tangkapan.

tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size <= 1024
<img width="1568" height="440" alt="image" src="https://github.com/user-attachments/assets/c597d226-985e-4ea2-a855-5b6b7b16384f" />

Pemindaian UDP
UDP Scan secara singkat:

Tidak memerlukan proses jabat tangan.
Tidak ada permintaan untuk port yang terbuka.
Pesan kesalahan ICMP untuk penutupan port
Biasanya dilakukan dengan  nmap -sU perintah.
Buka Port UDP	Port UDP Tertutup​
Paket UDP -->
Paket UDP -->
Pesan ICMP Tipe 3, Kode 3. (Tujuan tidak dapat dijangkau, port tidak dapat dijangkau)
Port UDP yang tertutup (port nomor 69) dan terbuka (port nomor 68) :

<img width="1568" height="275" alt="image" src="https://github.com/user-attachments/assets/15c05ee7-4c82-4b36-a15f-d057dce0bb3a" />
Gambar di atas menunjukkan bahwa port yang tertutup mengembalikan paket kesalahan ICMP. Sekilas, tidak ada informasi lebih lanjut yang diberikan tentang kesalahan tersebut, jadi bagaimana seorang analis dapat menentukan di mana pesan kesalahan ini berada? Pesan kesalahan ICMP menggunakan permintaan asli sebagai data terenkapsulasi untuk menunjukkan sumber/alasan paket. Setelah Anda memperluas bagian ICMP di panel detail paket, Anda akan melihat data terenkapsulasi dan permintaan asli, seperti yang ditunjukkan pada gambar di bawah ini.

<img width="1565" height="1089" alt="image" src="https://github.com/user-attachments/assets/cbc0111d-4669-4bdd-9409-2f04348369f1" />
Filter yang diberikan menunjukkan pola pemindaian UDP dalam file tangkapan.

icmp.type==3 and icmp.code==3    

<img width="1568" height="375" alt="image" src="https://github.com/user-attachments/assets/08ebd9b3-0d57-41f0-9a50-67366a25a0ce" />
Mendeteksi aktivitas mencurigakan dalam file yang dipecah menjadi beberapa bagian itu mudah dan merupakan cara yang bagus untuk belajar bagaimana fokus pada detail.  Sekarang gunakan "~/Desktop/exercise-pcaps/nmap/Exercise.pcapng"file tersebut untuk mempraktikkan keterampilan Anda terhadap satu file tangkapan tunggal dan jawab pertanyaan di bawah ini!

Jawablah pertanyaan-pertanyaan di bawah in

Berapakah jumlah total pemindaian "TCP Connect"?

1000
ketik di pencarian  (tcp.flags.syn==1) && (tcp.window_size_value ==62727) and tcp.flags.ack == 0  dan akan muncul hasil


Jenis pemindaian apa yang digunakan untuk memindai port TCP 80?

TCP Connect
ketik di pencarian tcp.port==80 akan muncul hasil 


Ada berapa pesan "UDP close port"?
ketik di pencarian : udp - cari protokol icmp bebas nomer berapa pun - cari  code : 3 (Port Unreachable) - klik tahan tarik ke atas pencarian paste kan di pencarian - icmp.code == 3  

jawaban nya : 1083


Port UDP mana dalam rentang port 55-70 yang terbuka?
ketik : udp.port in {55..70}

68

# Keracunan ARP & Man In The Middle!
Serangan ARP Poisoning/Spoofing (juga dikenal sebagai Serangan Man In The Middle)
Protokol ARP , atau Address Resolution Protocol ( ARP ), adalah teknologi yang memungkinkan perangkat untuk mengidentifikasi diri mereka sendiri di jaringan. Serangan Address Resolution Protocol Poisoning (juga dikenal sebagai ARP Spoofing atau Man In The Middle ( MITM )) adalah jenis serangan yang melibatkan pengacauan/manipulasi jaringan dengan mengirimkan paket ARP berbahaya ke gateway default. Tujuan utamanya adalah untuk memanipulasi "tabel alamat IP ke MAC" dan mengendus lalu lintas host target.

Tersedia berbagai macam alat untuk melakukan serangan ARP . Namun, pola pikir penyerang bersifat statis, sehingga serangan semacam itu mudah dideteksi dengan memahami alur kerja protokol ARP dan keterampilan Wireshark.

Analisis ARP secara singkat:

Beroperasi di jaringan lokal
Memungkinkan komunikasi antar alamat MAC.
Bukan protokol yang aman.
Bukan protokol yang dapat dirutekan
Aplikasi ini tidak memiliki fungsi otentikasi.
Pola umum meliputi permintaan & respons, pengumuman, dan paket informasi tambahan.
Sebelum menyelidiki lalu lintas, mari kita tinjau beberapa paket ARP yang sah dan mencurigakan . Permintaan yang sah mirip dengan gambar yang ditampilkan: permintaan siaran yang menanyakan apakah ada host yang tersedia yang menggunakan alamat IP dan balasan dari host yang menggunakan alamat IP tertentu.

Catatan	Filter Wireshark
Pencarian global	
arp
Opsi " ARP " untuk meraih peluang yang mudah didapatkan:

Opcode 1: Permintaan ARP .
Kode operasi 2: Respons ARP .
Hunt: Pemindaian Arp
Hunt: Kemungkinan deteksi keracunan ARP
Hunt: Kemungkinan banjir ARP dari deteksi:
arp.opcode == 1
arp.opcode == 2
arp.dst.hw_mac==00:00:00:00:00:00
arp.duplicate-address-detected or arp.duplicate-address-frame
((arp) && (arp.opcode == 1)) && (arp.src.hw_mac == target-mac-address)
Permintaan ARP

<img width="1568" height="612" alt="image" src="https://github.com/user-attachments/assets/0f06d5d3-2865-48c9-af19-5e3a652f3c6d" />

Balasan ARP

<img width="1568" height="547" alt="image" src="https://github.com/user-attachments/assets/f0dcb3fd-d36c-4c1a-baea-b0e549984eab" />

Situasi mencurigakan berarti adanya dua respons ARP yang berbeda (konflik) untuk alamat IP tertentu. Dalam hal ini, tab informasi ahli Wireshark akan memperingatkan analis. Namun, tab tersebut hanya menampilkan kemunculan kedua dari nilai duplikat untuk menyoroti konflik. Oleh karena itu, mengidentifikasi paket berbahaya dari paket yang sah merupakan tantangan bagi analis. Contoh kasus IP spoofing ditunjukkan pada gambar di bawah ini.

<img width="1568" height="601" alt="image" src="https://github.com/user-attachments/assets/b994238d-b363-4f09-8d6e-add992cfa372" />

Di sini, memahami arsitektur jaringan dan memeriksa lalu lintas dalam jangka waktu tertentu dapat membantu mendeteksi anomali. Sebagai analis, Anda harus mencatat temuan Anda sebelum melanjutkan. Ini akan membantu Anda lebih terorganisir dan memudahkan korelasi temuan selanjutnya. Perhatikan gambar yang diberikan; ada konflik; alamat MAC yang berakhiran "b4" membuat permintaan ARP dengan alamat IP "192.168.1.25", kemudian mengklaim memiliki alamat IP "192.168.1.1".

Catatan	Catatan Deteksi	Temuan
Kemungkinan alamat IP cocok.	1 alamat IP diumumkan dari alamat MAC.	
MAC: 00:0c:29:e2:18:b4
IP: 192.168.1.25
Kemungkinan upaya pemalsuan ARP .	
Terdapat 2 alamat MAC yang mengklaim alamat IP yang sama (192.168.1.1).
Alamat IP "192.168.1.1" kemungkinan merupakan alamat gateway.

MAC1: 50:78:b3:f3:cd:f4
MAC 2: 00:0c:29:e2:18:b4
Kemungkinan upaya pembanjiran oleh ARP .	Alamat MAC yang berakhiran "b4" mengklaim memiliki alamat IP yang berbeda/baru.	
MAC: 00:0c:29:e2:18:b4
IP: 192.168.1.1
Mari terus periksa lalu lintas untuk menemukan anomali lainnya. Perlu dicatat bahwa kasus ini dibagi menjadi beberapa file tangkapan untuk mempermudah penyelidikan

<img width="1570" height="614" alt="image" src="https://github.com/user-attachments/assets/518b7f1e-e586-4f9a-9c04-4fabbe7a2eb4" />

Pada titik ini, jelas terlihat adanya anomali. Seorang analis keamanan tidak dapat mengabaikan banyaknya permintaan ARP . Ini bisa jadi aktivitas berbahaya, pemindaian, atau masalah jaringan. Ada anomali baru; alamat MAC yang berakhiran "b4" membuat beberapa permintaan ARP dengan alamat IP "192.168.1.25". Mari kita fokus pada sumber anomali ini dan memperluas catatan yang telah diambil. 

Catatan	Catatan Deteksi	Temuan
Kemungkinan alamat IP cocok.	1 alamat IP diumumkan dari alamat MAC.	
 

MAC: 00:0c:29:e2:18:b4
IP: 192.168.1.25
 

Kemungkinan upaya pemalsuan ARP .	
 

2 alamat MAC mengklaim alamat IP yang sama (192.168.1.1).
Alamat IP "192.168.1.1" adalah alamat gateway yang mungkin.
 

MAC1: 50:78:b3:f3:cd:f4
MAC 2: 00:0c:29:e2:18:b4
Kemungkinan upaya pemalsuan ARP .	Alamat MAC yang berakhiran "b4" mengklaim memiliki alamat IP yang berbeda/baru.	
MAC: 00:0c:29:e2:18:b4
IP: 192.168.1.1
Kemungkinan upaya pembanjiran oleh ARP .	Alamat MAC yang berakhiran "b4" membuat beberapa permintaan ARP terhadap berbagai alamat IP.	
MAC: 00:0c:29:e2:18:b4
IP: 192.168.1.xxx
Sampai saat ini, jelas bahwa alamat MAC yang berakhiran "b4" memiliki alamat IP "192.168.1.25" dan membuat permintaan ARP yang mencurigakan terhadap berbagai alamat IP. Alamat tersebut juga mengklaim memiliki kemungkinan alamat gateway. Mari kita fokus pada protokol lain dan temukan refleksi anomali ini di bagian selanjutnya dari rentang waktu tersebut.

<img width="1568" height="318" alt="image" src="https://github.com/user-attachments/assets/ff853ded-953d-4fae-be77-07ad2adf018c" />

Terdapat lalu lintas HTTP, dan semuanya tampak normal pada tingkat IP, sehingga tidak ada informasi yang terkait dengan temuan kita sebelumnya. Mari kita tambahkan alamat MAC sebagai kolom di panel daftar paket untuk mengungkap komunikasi di balik alamat IP tersebut.

<img width="1568" height="325" alt="image" src="https://github.com/user-attachments/assets/d9efcfbe-d071-4302-ba51-74ecaba96997" />

Satu anomali lagi! Alamat MAC yang berakhiran "b4" adalah tujuan dari semua paket HTTP! Jelas terlihat bahwa ada serangan MITM, dan penyerangnya adalah host dengan alamat MAC yang berakhiran "b4". Semua lalu lintas yang terkait dengan alamat IP "192.168.1.12" diteruskan ke host berbahaya tersebut. Mari kita rangkum temuan-temuan tersebut sebelum menyimpulkan investigasi.

Catatan	Catatan Deteksi	Temuan
IP cocok dengan MAC.	3 kecocokan alamat IP dengan alamat MAC. 	
MAC: 00:0c:29:e2:18:b4 = IP: 192.168.1.25
MAC: 50:78:b3:f3:cd:f4 = IP: 192.1681.1
MAC: 00:0c:29:98:c7:a8 = IP: 192.168.1.12
Penyerang	Penyerang menciptakan gangguan dengan paket ARP .	
MAC: 00:0c:29:e2:18:b4 = IP: 192.168.1.25
Router/gateway	Alamat gateway.	
MAC: 50:78:b3:f3:cd:f4 = IP: 192.1681.1
Korban	Pelaku mengendus semua lalu lintas data korban.	
MAC: 50:78:b3:f3:cd:f4 = IP: 192.1681.12
Mendeteksi potongan-potongan informasi ini dalam file tangkapan data yang besar merupakan tantangan. Namun, dalam kasus nyata, Anda tidak akan memiliki "data yang disesuaikan" yang siap untuk diselidiki. Oleh karena itu, Anda perlu memiliki pola pikir, pengetahuan, dan keterampilan menggunakan alat analisis untuk menyaring dan mendeteksi anomali.

Catatan: Dalam analisis lalu lintas, selalu ada solusi alternatif yang tersedia. Jenis solusi dan pendekatannya bergantung pada tingkat pengetahuan dan keterampilan analis serta sumber data yang tersedia.

Mendeteksi aktivitas mencurigakan dalam file yang dipecah menjadi beberapa bagian itu mudah dan merupakan cara yang bagus untuk belajar bagaimana fokus pada detail. Sekarang gunakan "~ /Desktop/exercise-pcaps/arp/Exercise"file tersebut untuk mempraktikkan keterampilan Anda terhadap satu file tangkapan tunggal dan jawab pertanyaan di bawah ini!

Jawablah pertanyaan-pertanyaan di bawah ini. 
pastikan dulu  berada di file : "~ /Desktop/exercise-pcaps/arp/Exercise" ( wireshark )

Berapakah jumlah permintaan ARP yang dibuat oleh penyerang?
ketik di pencarian : eth.src == 00:0c:29:e2:18:b4 and arp.opcode == 1
jawabaan lihat display bawah  : 284

Berapakah jumlah paket HTTP yang diterima oleh penyerang?
ketik di pencarian : eth.dst == 00:0c:29:e2:18:b4 and http
jawaban : 90

Berapakah jumlah entri nama pengguna dan kata sandi yang diendus?
KETIK DIN PENCARIAN : http.host == testphp.vulnweb.com and http.request.method == "POST"
JAWABAN : 6


Apa kata sandi untuk "Client986"?
masih dengan query pencarian yang sama : http.host == testphp.vulnweb.com and http.request.method == "POST" 
1668	354.682038726	192.168.1.12	44.228.249.3	HTTP	728	POST /userinfo.php HTTP/1.1  (application/x-www-form-urlencoded)
daN LIHAT DI detail http
Form item: "pass" = "clientnothere!"

jawaban clientnothere!


Apa komentar yang diberikan oleh "Client354"?
masih dengan query pencarian yang sama : http.host == testphp.vulnweb.com and http.request.method == "POST" 
2320	618.814163954	192.168.1.12	44.228.249.3	HTTP	787	POST /comment.php HTTP/1.1  (application/x-www-form-urlencoded)
HTML Form URL Encoded: application/x-www-form-urlencoded    dan lihat
Form item: "comment" = "Nice work!"

jawaban : Nice work!

# Mengidentifikasi Host: DHCP, NetBIOS, dan Kerberos
Mengidentifikasi Inang
Saat menyelidiki aktivitas peretasan atau infeksi malware, seorang analis keamanan harus mengetahui cara mengidentifikasi host di jaringan selain dari pencocokan alamat IP dan MAC. Salah satu metode terbaik adalah mengidentifikasi host dan pengguna di jaringan untuk menentukan titik awal investigasi dan membuat daftar host dan pengguna yang terkait dengan lalu lintas/aktivitas berbahaya.

Biasanya, jaringan perusahaan menggunakan pola yang telah ditentukan sebelumnya untuk memberi nama pengguna dan host. Meskipun hal ini memudahkan untuk mengetahui dan mengikuti inventaris, ada sisi baik dan buruknya. Sisi baiknya adalah akan mudah untuk mengidentifikasi pengguna atau host hanya dengan melihat namanya. Sisi buruknya adalah pola tersebut akan mudah ditiru dan dibiarkan ada di jaringan perusahaan oleh pihak yang tidak bertanggung jawab. Ada banyak solusi untuk menghindari aktivitas semacam ini, tetapi bagi seorang analis keamanan, memiliki keterampilan identifikasi host dan pengguna tetap sangat penting.

Protokol yang dapat digunakan dalam identifikasi Host dan Pengguna:

Lalu lintas Dynamic Host Configuration Protocol ( DHCP )
Lalu lintas NetBIOS (NBNS) 
Lalu lintas Kerberos
Analisis DHCP
 Protokol  DHCP , atau Dynamic Host Configuration  Protocol (  DHCP ) ,  adalah teknologi yang bertanggung jawab untuk mengelola penetapan alamat IP otomatis dan parameter komunikasi yangdibutuhkan . 

Ringkasan investigasi DHCP :

Catatan	Filter Wireshark
Pencarian global.	
dhcpataubootp
Memfilter opsi paket DHCP yang tepat sangat penting untuk menemukan peristiwa yang diminati. 

Paket "Permintaan DHCP" berisi informasi nama host.
Paket "DHCP ACK" mewakili permintaan yang diterima.
Paket "DHCP NAK" mewakili permintaan yang ditolak.
Karena sifat protokolnya, hanya "Opsi 53" (tipe permintaan) yang memiliki nilai statis yang telah ditentukan sebelumnya. Anda harus memfilter tipe paket terlebih dahulu, lalu Anda dapat memfilter opsi lainnya dengan "menerapkan sebagai kolom" atau menggunakan filter lanjutan seperti "berisi" dan "cocok".

Meminta:dhcp.option.dhcp == 3
ACK:dhcp.option.dhcp == 5
NAK:dhcp.option.dhcp == 6
Opsi " Permintaan DHCP " untuk mendapatkan akses yang mudah:

Opsi 12: Nama host.
Opsi 50: Alamat IP yang diminta.
Opsi 51: Waktu sewa IP yang diminta.
Opsi 61: Alamat MAC klien.
dhcp.option.hostname contains "keyword"
Opsi " DHCP ACK" untuk mendapatkan akses yang mudah:

Opsi 15: Nama domain.
Opsi 51: Waktu sewa IP yang ditetapkan.
dhcp.option.domain_name contains "keyword"
Opsi " DHCP NAK" untuk meraih peluang yang mudah didapatkan:

Opsi 56: Pesan (rincian/alasan penolakan).
Karena pesan tersebut bisa unik tergantung pada kasus/situasi, disarankan untuk membaca pesan tersebut daripada menyaringnya. Dengan demikian, analis dapat membuat hipotesis/hasil yang lebih andal dengan memahami keadaan kejadian tersebut.

<img width="1502" height="1068" alt="image" src="https://github.com/user-attachments/assets/2bc750d8-d8f8-4ebe-8ca6-84c7255323a4" />

Analisis NetBIOS (NBNS)
NetBIOS  atau  Network Basic  Input /  Output System  adalah teknologi yang memungkinkan aplikasi pada host yang berbeda untuk berkomunikasi satu sama lain .

Ringkasan investigasi NBNS:

Catatan	Filter Wireshark
Pencarian global.	
nbns
Opsi "NBNS"  untuk meraih peluang yang mudah didapatkan:

Pertanyaan: Detail pertanyaan.
Detail kueri dapat berisi "nama, Waktu hidup (TTL) dan detail alamat IP"
nbns.name contains "keyword"

<img width="1502" height="1068" alt="image" src="https://github.com/user-attachments/assets/b06f2874-01b7-43f9-a156-5830d710bdcb" />

Analisis Kerberos
Kerberos  adalah layanan otentikasi default untuk domain Microsoft Windows. Layanan ini bertanggung jawab untuk mengotentikasi permintaan layanan antara dua atau lebih komputer melalui jaringan yang tidak tepercaya. Tujuan utamanya adalah untuk membuktikan identitas secara aman.

Ringkasan investigasi Kerberos:

Catatan	Filter Wireshark
Pencarian global.	
kerberos
Pencarian akun pengguna:

CNameString: Nama pengguna.
Catatan:  Beberapa paket mungkin memberikan informasi nama host di kolom ini. Untuk menghindari kebingungan ini, saring nilai "$" . Nilai yang diakhiri dengan "$" adalah nama host, dan yang tanpa tanda "$" adalah nama pengguna.

kerberos.CNameString contains "keyword" 
kerberos.CNameString and !(kerberos.CNameString contains "$" )
Opsi " Kerberos " untuk meraih peluang yang mudah didapatkan:

pvno: Versi protokol.
realm: Nama domain untuk tiket yang dihasilkan.
sname: Nama layanan dan domain untuk tiket yang dihasilkan.
Alamat: Alamat IP klien dan nama NetBIOS.
Catatan: informasi "alamat" hanya tersedia dalam paket permintaan.

kerberos.pvno == 5
kerberos.realm contains ".org" 
kerberos.SNameString == "krbtg"

<img width="1502" height="1068" alt="image" src="https://github.com/user-attachments/assets/3e1de5f9-bf50-44e5-b864-92eb8a82f749" />

Mendeteksi aktivitas mencurigakan dalam file yang dipecah menjadi bagian-bagian kecil (chunked files) itu mudah dan merupakan cara yang bagus untuk belajar bagaimana fokus pada detail.  Sekarang gunakan "~/Desktop/exercise-pcaps/dhcp-netbios-kerberos/dhcp-netbios.pcap"file tersebut untuk menyelesaikan pertanyaan 1 sampai 3 dan  "~/Desktop/exercise-pcaps/dhcp-netbios-kerberos/**kerberos.pcap"untuk menyelesaikan pertanyaan 4 dan 5.

Jawablah pertanyaan-pertanyaan di bawah ini.

Berapakah alamat MAC dari host "Galaxy A30"?

9a:81:41:cb:96:6c



Berapa banyak permintaan registrasi NetBIOS yang dimiliki oleh workstation "LIVALJM"?

16



Host mana yang meminta alamat IP "172.16.13.85"?
Galaxy-A12


Berapakah alamat IP pengguna "u5"? (Masukkan alamat dalam format yang sudah di-defang.)

10[.]1[.]12[.]2


Apa nama host dari host yang tersedia dalam paket Kerberos?
xp1$


# Lalu Lintas Terowongan: DNS dan ICMP
Tunnelling Traffic: ICMP and DNS
Traffic tunnelling is (also known as "port forwarding" transferring the data/resources in a secure method to network segments and zones. It can be used for "internet to private networks" and "private networks to internet" flow/direction. There is an encapsulation process to hide the data, so the transferred data appear natural for the case, but it contains private data packets and transfers them to the final destination securely.

Tunnelling provides anonymity and traffic security. Therefore it is highly used by enterprise networks. However, as it gives a significant level of data encryption, attackers use tunnelling to bypass security perimeters using the standard and trusted protocols used in everyday traffic like ICMP and DNS. Therefore, for a security analyst, it is crucial to have the ability to spot ICMP and DNS anomalies.

ICMP Analysis
Internet Control Message Protocol (ICMP) is designed for diagnosing and reporting network communication issues. It is highly used in error reporting and testing. As it is a trusted network layer protocol, sometimes it is used for denial of service (DoS) attacks; also, adversaries use it in data exfiltration and C2 tunnelling activities.

ICMP analysis in a nutshell:

Usually, ICMP tunnelling attacks are anomalies appearing/starting after a malware execution or vulnerability exploitation. As the ICMP packets can transfer an additional data payload, adversaries use this section to exfiltrate data and establish a C2 connection. It could be a TCP, HTTP or SSH data. As the ICMP protocols provide a great opportunity to carry extra data, it also has disadvantages. Most enterprise networks block custom packets or require administrator privileges to create custom ICMP packets.

A large volume of ICMP traffic or anomalous packet sizes are indicators of ICMP tunnelling. Still, the adversaries could create custom packets that match the regular ICMP packet size (64 bytes), so it is still cumbersome to detect these tunnelling activities. However, a security analyst should know the normal and the abnormal to spot the possible anomaly and escalate it for further analysis.

Notes	Wireshark filters
Global search	
icmp
"ICMP" options for grabbing the low-hanging fruits:

Packet length.
ICMP destination addresses.
Encapsulated protocol signs in ICMP payload.
data.len > 64 and icmp

Analisis DNS
Sistem Nama Domain (DNS) dirancang untuk menerjemahkan/mengonversi alamat domain IP ke alamat IP. DNS juga dikenal sebagai buku telepon internet. Karena merupakan bagian penting dari layanan web, DNS umum digunakan dan dipercaya, sehingga sering diabaikan. Akibatnya, pihak yang tidak bertanggung jawab menggunakannya dalam eksfiltrasi data dan aktivitas C2.

Analisis DNS secara singkat:

Mirip dengan terowongan ICMP, serangan DNS adalah anomali yang muncul/dimulai setelah eksekusi malware atau eksploitasi kerentanan. Penyerang membuat (atau sudah memiliki) alamat domain dan mengkonfigurasinya sebagai saluran C2. Malware atau perintah yang dieksekusi setelah eksploitasi mengirimkan kueri DNS ke server C2. Namun, kueri ini lebih panjang daripada kueri DNS standar dan dirancang untuk alamat subdomain.  Sayangnya, alamat subdomain ini bukanlah alamat sebenarnya; melainkan perintah yang dikodekan seperti yang ditunjukkan di bawah ini:

"encoded-commands.maliciousdomain.com"

Ketika permintaan ini dialihkan ke server C2, server mengirimkan perintah berbahaya yang sebenarnya ke host. Karena permintaan DNS merupakan bagian alami dari aktivitas jaringan, paket-paket ini berpotensi tidak terdeteksi oleh perimeter jaringan. Seorang analis keamanan harus mengetahui cara menyelidiki panjang paket DNS dan alamat target untuk menemukan anomali ini.

Catatan	Filter Wireshark
Pencarian global	
dns
Opsi " DNS " untuk meraih peluang yang mudah didapatkan:

Panjang kueri.
Nama-nama anomali dan tidak lazim dalam alamat DNS .
Alamat DNS panjang dengan alamat subdomain yang dikodekan.
Pola yang sudah dikenal seperti dnscat dan dns2tcp.
Analisis statistik seperti volume permintaan DNS yang anomali untuk target tertentu.
!mdns: Nonaktifkan kueri perangkat tautan lokal.

dns contains "dnscat"
dns.qry.name.len > 15 and !mdns
 <img width="1503" height="1069" alt="image" src="https://github.com/user-attachments/assets/13735ce0-91bf-4ee1-b466-34bbf2f32b0b" />

Mendeteksi aktivitas mencurigakan dalam file yang dipecah menjadi bagian-bagian kecil (chunked files) itu mudah dan merupakan cara yang bagus untuk belajar bagaimana fokus pada detail. Sekarang gunakan file latihan untuk mempraktikkan keterampilan Anda terhadap satu file tangkapan tunggal dan jawab pertanyaan di bawah ini!

Jawablah pertanyaan-pertanyaan di bawah ini.

Gunakan file "Desktop/exercise-pcaps/dns-icmp/icmp-tunnel.pcap". Selidiki paket-paket anomali tersebut. Protokol apa yang digunakan dalam tunneling ICMP?

SSH


Gunakan file "Desktop/exercise-pcaps/dns-icmp/dns.pcap". Selidiki paket-paket anomali tersebut. Apa alamat domain utama yang mencurigakan yang menerima permintaan DNS anomali? (Masukkan alamat dalam format yang sudah diubah.)

dataexfil[.]com

Analisis Protokol Teks Jelas: FTP
Analisis Protokol Teks Jelas
Menganalisis jejak protokol teks biasa terdengar mudah, tetapi ketika tiba saatnya untuk menganalisis jejak jaringan besar untuk analisis dan respons insiden, situasinya berubah. Analisis yang tepat lebih dari sekadar mengikuti aliran data dan membaca data teks biasa. Bagi seorang analis keamanan, penting untuk membuat statistik dan hasil utama dari proses investigasi. Seperti yang disebutkan sebelumnya di awal seri ruang Wireshark, analis harus memiliki pengetahuan jaringan dan keterampilan alat yang diperlukan untuk mencapai hal ini. Mari kita simulasikan investigasi protokol teks biasa dengan Wireshark!

Analisis FTP
File Transfer Protocol ( FTP ) dirancang untuk mentransfer file dengan mudah, sehingga berfokus pada kesederhanaan daripada keamanan. Akibatnya, penggunaan protokol ini di lingkungan yang tidak aman dapat menimbulkan masalah keamanan seperti:

Serangan MITM
Pencurian kredensial dan akses tanpa izin
Phishing
Penyebaran malware
Eksfiltrasi data
Analisis FTP secara singkat:

Catatan	Filter Wireshark
Pencarian global	
ftp
Opsi " FTP " untuk meraih peluang yang mudah didapatkan:

Seri x1x: Tanggapan terhadap permintaan informasi.
Seri x2x: Pesan koneksi.
Seri x3x: Pesan otentikasi.
Catatan: "200" berarti perintah berhasil.

---
Opsi seri "x1x" untuk meraih peluang yang mudah didapatkan:

211: Status sistem.
212: Status direktori.
213: Status berkas
ftp.response.code == 211 
Opsi seri "x2x" untuk meraih peluang yang mudah didapatkan:

220: Layanan siap.
227: Memasuki mode pasif.
228: Mode pasif panjang.
229: Mode pasif diperpanjang.
ftp.response.code == 227
Opsi seri "x3x" untuk meraih peluang yang mudah didapatkan:

230: Login pengguna.
231: Pengguna keluar.
331: Nama pengguna valid.
430: Nama pengguna atau kata sandi tidak valid
530: Tidak dapat masuk, kata sandi tidak valid.
ftp.response.code == 230
Perintah " FTP " untuk meraih hasil yang mudah didapatkan:

PENGGUNA: Nama pengguna.
PASS: Kata Sandi.
CWD: Direktori kerja saat ini.
DAFTAR: Daftar.
ftp.request.command == "USER"
ftp.request.command == "PASS"
ftp.request.arg == "password"
Contoh penggunaan tingkat lanjut untuk meraih peluang yang mudah didapatkan:

Sinyal serangan brute force: Daftar upaya login yang gagal.
Sinyal serangan brute force: Cantumkan nama pengguna target.
Sinyal semprotan kata sandi: Daftar target untuk kata sandi statis.
ftp.response.code == 530
(ftp.response.code == 530) and (ftp.response.arg contains "username")
(ftp.request.command == "PASS" ) and (ftp.request.arg == "password")
<img width="1503" height="1069" alt="image" src="https://github.com/user-attachments/assets/f6f399ef-a4b0-4b59-93f6-e418e39dff75" />
Mendeteksi aktivitas mencurigakan dalam file yang dipecah menjadi bagian-bagian kecil (chunked files) itu mudah dan merupakan cara yang bagus untuk belajar bagaimana fokus pada detail.  Sekarang gunakan  "~/Desktop/exercise-pcaps/ftp/ftp.pcap"file tersebut untuk mempraktikkan keterampilan Anda dan jawab pertanyaan-pertanyaan di bawah ini!

Jawablah pertanyaan-pertanyaan di bawah ini.

Ada berapa kali upaya login yang gagal?
737



Berapakah ukuran file yang diakses oleh akun "ftp"?
39424



Pihak lawan mengunggah sebuah dokumen ke server FTP. Apa nama filenya?
resume.doc


Pihak penyerang mencoba memberikan flag khusus untuk mengubah izin eksekusi file yang diunggah. Apa perintah yang digunakan oleh pihak penyerang?
CHMOD 777

# Analisis Protokol Teks Jelas: HTTP
Analisis HTTP
Hypertext Transfer Protocol ( HTTP ) adalah protokol berbasis teks biasa, berbasis permintaan-respons dan klien-server. Ini adalah jenis aktivitas jaringan standar untuk meminta/menyajikan halaman web, dan secara default, tidak diblokir oleh perimeter jaringan apa pun. Karena tidak terenkripsi dan menjadi tulang punggung lalu lintas web, HTTP adalah salah satu protokol yang wajib diketahui dalam analisis lalu lintas. Serangan berikut dapat dideteksi dengan bantuan analisis HTTP :

Halaman phishing
Serangan web
Eksfiltrasi data
Lalu lintas perintah dan kendali ( C2 )
Analisis HTTP secara singkat:

Catatan	Filter Wireshark
Pencarian global

Catatan:  HTTP2 adalah revisi dari protokol HTTP untuk kinerja dan keamanan yang lebih baik. Protokol ini mendukung transfer data biner dan multiplexing permintaan & respons.

http
http2
 " Metode Permintaan HTTP " untuk meraih peluang yang mudah didapatkan:

MENDAPATKAN
POS
Permintaan: Daftar semua permintaan
http.request.method == "GET"
http.request.method == "POST"
http.request
" Kode Status Respons HTTP  " untuk meraih peluang yang mudah didapatkan:

200 OK: Permintaan berhasil.
301 Moved Permanently: Sumber daya dipindahkan ke URL/jalur baru (secara permanen).
302 Dipindahkan Sementara: Sumber daya dipindahkan ke URL/jalur baru (sementara).
400 Permintaan Buruk: Server tidak memahami permintaan tersebut.
401 Tidak Diizinkan: URL memerlukan otorisasi (login, dll.).
403 Terlarang: Tidak dapat mengakses URL yang diminta. 
404 Not Found: Server tidak dapat menemukan URL yang diminta.
405 Metode Tidak Diizinkan: Metode yang digunakan tidak sesuai atau diblokir.
408 Request Timeout:   Permintaan membutuhkan waktu lebih lama daripada waktu tunggu server.
500 Internal Server Error: Permintaan tidak selesai, kesalahan tak terduga.
503 Layanan Tidak Tersedia: Permintaan tidak selesai, server atau layanan sedang down.
http.response.code == 200
http.response.code == 401
http.response.code == 403
http.response.code == 404
http.response.code == 405
http.response.code == 503
" Parameter HTTP  " untuk meraih peluang yang mudah didapatkan:

Agen pengguna: Identifikasi peramban dan sistem operasi ke aplikasi server web.
URI Permintaan: Menunjuk ke sumber daya yang diminta dari server.
URI lengkap: Informasi URI lengkap.
*URI: Pengidentifikasi Sumber Daya Seragam.

http.user_agent contains "nmap"
http.request.uri contains "admin"
http.request.full_uri contains "admin"
" Parameter HTTP " untuk meraih peluang yang mudah didapatkan:

Server: Nama layanan server.
Host: Nama host server
Koneksi: Status koneksi.
Data teks berbasis baris: Data teks biasa yang disediakan oleh server.
Formulir HTML yang Dikodekan URL: Informasi formulir web.
http.server contains "apache"
http.host contains "keyword"
http.host == "keyword"
http.connection == "Keep-Alive"
data-text-lines contains "keyword"
Analisis Agen Pengguna
Karena pihak musuh menggunakan teknik canggih untuk melakukan serangan, mereka mencoba meninggalkan jejak yang mirip dengan lalu lintas alami melalui protokol yang dikenal dan tepercaya. Bagi seorang analis keamanan, penting untuk menemukan tanda-tanda anomali pada bagian-bagian paket. Bidang "user-agent" adalah salah satu sumber daya yang bagus untuk menemukan anomali dalam lalu lintas HTTP. Dalam beberapa kasus, pihak musuh berhasil memodifikasi data user-agent, yang bisa terlihat sangat alami. Seorang analis keamanan tidak dapat hanya mengandalkan bidang user-agent untuk menemukan anomali. Jangan pernah memasukkan user-agent ke daftar putih, meskipun terlihat alami. Deteksi/perburuan anomali/ancaman berbasis user-agent adalah sumber data tambahan untuk diperiksa dan berguna ketika ada anomali yang jelas. Jika Anda tidak yakin tentang suatu nilai, Anda dapat melakukan pencarian web untuk memvalidasi temuan Anda dengan informasi user-agent default dan normal ( contoh situs ).

Analisis User Agent secara singkat:

Catatan	Filter Wireshark
Pencarian global.	
http.user_agent
Hasil penelitian untuk meraih peluang yang mudah didapatkan:

Informasi user agent yang berbeda dari host yang sama dalam waktu singkat.
Informasi agen pengguna non-standar dan kustom.
Perbedaan ejaan yang halus. ("Mozilla" tidak sama dengan "Mozlilla" atau "Mozlila")
Informasi alat audit seperti Nmap, Nikto, Wfuzz, dan sqlmap di kolom user agent.
Data payload di kolom user agent.
(http.user_agent contains "sqlmap") or (http.user_agent contains "Nmap") or (http.user_agent contains "Wfuzz") or (http.user_agent contains "Nikto")
<img width="1668" height="1069" alt="image" src="https://github.com/user-attachments/assets/9caa69a5-07e8-4d58-a0f0-191bc4b1ec6d" />

Analisis Log4j
Investigasi yang tepat dimulai dengan riset awal tentang ancaman dan anomali yang akan diburu. Mari kita tinjau hal-hal yang diketahui tentang serangan "Log4j" sebelum menjalankan Wireshark.

Analisis kerentanan Log4j secara singkat:

Catatan	Filter Wireshark
Hasil penelitian  untuk meraih peluang yang mudah didapatkan:

Serangan dimulai dengan permintaan "POST"
Terdapat pola teks biasa yang dikenal: " jndi:ldap " dan " Exploit.class ".
http.request.method == "POST"
(ip contains "jndi") or ( ip contains "Exploit")
(frame contains "jndi") or ( frame contains "Exploit")
(http.user_agent contains "$") or (http.user_agent contains "==")
<img width="1668" height="1069" alt="image" src="https://github.com/user-attachments/assets/cf07b3a8-d151-4ec9-894b-495d46a05393" />
Mendeteksi aktivitas mencurigakan dalam file yang dipecah menjadi bagian-bagian kecil (chunked files) itu mudah dan merupakan cara yang bagus untuk belajar bagaimana fokus pada detail.  Sekarang gunakan "~/Desktop/exercise-pcaps/http/user-agent.pcap"file tersebut untuk menjawab pertanyaan 1 hingga 2 dan "~/Desktop/exercise-pcaps/http/http.pcapng"file lainnya untuk menjawab pertanyaan 3 hingga 4.

Jawablah pertanyaan-pertanyaan di bawah ini.
Selidiki agen pengguna. Berapa jumlah tipe "agen pengguna" yang anomali?

6


Berapakah nomor paket dengan perbedaan ejaan yang halus pada kolom agen pengguna?

52

Temukan fase awal serangan "Log4j". Berapa nomor paketnya?

444


Temukan fase awal serangan "Log4j" dan dekode perintah base64. Berapa alamat IP yang dihubungi oleh penyerang? (Masukkan alamat dalam format yang sudah dienkripsi dan kecualikan "{}".)

62[.]210[.]130[.]250

Analisis Protokol Terenkripsi: Mendekripsi HTTPS
Mendekripsi Lalu Lintas HTTPS
Saat menyelidiki lalu lintas web, analis sering menemukan lalu lintas terenkripsi. Hal ini disebabkan oleh penggunaan protokol Hypertext Transfer Protocol Secure (HTTPS) untuk meningkatkan keamanan terhadap serangan spoofing, sniffing, dan intersepsi. HTTPS menggunakan protokol TLS untuk mengenkripsi komunikasi, sehingga tidak mungkin untuk mendekripsi lalu lintas dan melihat data yang ditransfer tanpa memiliki pasangan kunci enkripsi/dekripsi. Karena protokol ini memberikan tingkat keamanan yang baik untuk mentransmisikan data sensitif, penyerang dan situs web berbahaya juga menggunakan HTTPS. Oleh karena itu, seorang analis keamanan harus mengetahui cara menggunakan file kunci untuk mendekripsi lalu lintas terenkripsi dan menyelidiki aktivitas lalu lintas.

Paket-paket tersebut akan muncul dalam warna yang berbeda karena lalu lintas HTTP dienkripsi. Selain itu, detail protokol dan informasi (alamat URL sebenarnya dan data yang dikembalikan dari server) tidak akan terlihat sepenuhnya. Gambar pertama di bawah ini menunjukkan paket HTTP yang dienkripsi dengan protokol TLS .

Informasi tambahan untuk HTTPS:

Catatan	Filter Wireshark
"Parameter HTTPS"  untuk meraih peluang yang mudah didapatkan:

Permintaan: Daftar semua permintaan
TLS : Pencarian TLS global
Permintaan Klien TLS
Respons Server TLS
Protokol Penemuan Layanan Sederhana Lokal (SSDP)
Catatan:  SSDP adalah protokol jaringan yang menyediakan pengiklanan dan penemuan layanan jaringan.

http.request
tls
tls.handshake.type == 1
tls.handshake.type == 2
ssdp

<img width="1669" height="518" alt="image" src="https://github.com/user-attachments/assets/f4405af1-8b17-4879-a270-d4d2544dca06" />
Mirip dengan proses jabat tangan tiga arah TCP, protokol TLS juga memiliki proses jabat tangan. Dua langkah pertama berisi pesan "Client Hello" dan "Server Hello". Filter yang diberikan menunjukkan paket hello awal dalam file tangkapan. Filter ini berguna untuk mengetahui alamat IP mana yang terlibat dalam jabat tangan TLS.

Halo Klien:(http.request or tls.handshake.type == 1) and !(ssdp) 
Halo Server: (http.request or tls.handshake.type == 2) and !(ssdp)  
<img width="1668" height="1069" alt="image" src="https://github.com/user-attachments/assets/fdedcaeb-baf8-4939-a531-fbbabe09cb0b" />
<img width="1668" height="1068" alt="image" src="https://github.com/user-attachments/assets/c03fd081-c145-45e4-8028-7e14606e4f6a" />

File log kunci enkripsi adalah file teks yang berisi pasangan kunci unik untuk mendekripsi sesi lalu lintas terenkripsi. Pasangan kunci ini dibuat secara otomatis (per sesi) ketika koneksi dibuat dengan halaman web yang mendukung SSL/ TLS . Karena semua proses ini dilakukan di browser, Anda perlu mengkonfigurasi sistem Anda dan menggunakan browser yang sesuai (Chrome dan Firefox mendukung ini) untuk menyimpan nilai-nilai ini sebagai file log kunci. Untuk melakukan ini, Anda perlu mengatur variabel lingkungan dan membuat SSLKEYLOGFILE, dan browser akan menyimpan kunci ke file ini saat Anda menjelajahi web. Pasangan kunci SSL/ TLS dibuat per sesi pada saat koneksi, jadi penting untuk menyimpan kunci selama pengambilan lalu lintas. Jika tidak, tidak mungkin untuk membuat/menghasilkan file log kunci yang sesuai untuk mendekripsi lalu lintas yang diambil. Anda dapat menggunakan menu "klik kanan" atau menu "Edit --> Preferensi --> Protokol --> TLS " untuk menambahkan/menghapus file log kunci.

Menambahkan file log penting dengan menu "klik kanan":
<img width="1656" height="1033" alt="image" src="https://github.com/user-attachments/assets/16d3167e-10ca-47d0-887d-73259f8dccb0" />
Menambahkan file log penting melalui "Edit --> Preferensi --> Protokol --> TLS "
<img width="1656" height="1033" alt="image" src="https://github.com/user-attachments/assets/ec4619be-2418-49b8-8eef-08d6111f1b7e" />
Melihat lalu lintas dengan/tanpa file log kunci:
<img width="1815" height="1161" alt="image" src="https://github.com/user-attachments/assets/d065525e-13cd-4413-acec-4a8b3e4ccbba" />

Gambar di atas menunjukkan bahwa detail lalu lintas terlihat setelah menggunakan file log kunci. Perhatikan bahwa panel detail paket dan byte menyediakan data dalam berbagai format untuk investigasi. Info header yang didekompresi dan detail paket HTTP2 tersedia setelah mendekripsi lalu lintas. Tergantung pada detail paket, Anda juga dapat memiliki format data berikut:

Bingkai
TLS yang telah didekripsi
Header yang telah didekompresi
TCP yang disusun ulang
SSL yang telah dirakit ulang
Mendeteksi aktivitas mencurigakan dalam file yang dipecah menjadi bagian-bagian kecil (chunked files) itu mudah dan merupakan cara yang bagus untuk belajar bagaimana fokus pada detail. Sekarang gunakan "Desktop/exercise-pcaps/https/Exercise.pcap"file tersebut untuk mempraktikkan keterampilan Anda dan jawab pertanyaan-pertanyaan di bawah ini!

Jawablah pertanyaan-pertanyaan di bawah ini.
Berapakah nomor frame dari pesan "Client Hello" yang dikirim ke "accounts.google.com"?

16


Dekripsi lalu lintas menggunakan file "KeysLogFile.txt". Berapa jumlah paket HTTP2?

115


Buka Frame 322. Apa header otoritas dari paket HTTP2?  (Masukkan alamat dalam format tanpa karakter khusus.)

safebrowsing[.]googleapis[.]com

Selidiki paket yang telah didekripsi dan temukan flag-nya! Apakah flag tersebut?

FLAG{THM-PACKETMASTER}

# Bonus: Buru Kredensial Teks Biasa!
Bonus: Buru Kredensial Teks Biasa!
Sampai di sini, kita telah membahas cara memeriksa paket untuk kondisi spesifik dan mendeteksi anomali. Seperti yang disebutkan di ruang pertama, Wireshark bukanlah IDS , tetapi memberikan saran untuk beberapa kasus di bawah informasi ahli. Namun, terkadang anomali mereplikasi lalu lintas yang sah, sehingga deteksi menjadi lebih sulit. Misalnya, dalam kasus perburuan kredensial teks biasa, tidak mudah untuk mendeteksi beberapa input kredensial dan memutuskan apakah itu serangan brute-force atau pengguna biasa yang salah mengetik kredensial mereka.

Karena semuanya disajikan pada tingkat paket, sulit untuk melihat beberapa entri nama pengguna/kata sandi secara sekilas. Waktu deteksi akan berkurang ketika seorang analis dapat melihat entri kredensial sebagai sebuah daftar. Wireshark memiliki fitur tersebut untuk membantu analis yang ingin mencari entri kredensial dalam bentuk teks biasa.

Beberapa dissector Wireshark ( FTP , HTTP , IMAP , POP, dan SMTP ) diprogram untuk mengekstrak kata sandi teks biasa dari file tangkapan. Anda dapat melihat kredensial yang terdeteksi menggunakan  menu "Tools --> Credentials"  . Fitur ini hanya berfungsi setelah versi Wireshark tertentu (v3.1 dan yang lebih baru). Karena fitur ini hanya berfungsi dengan protokol tertentu, disarankan untuk melakukan pemeriksaan manual dan tidak sepenuhnya bergantung pada fitur ini untuk menentukan apakah ada kredensial teks biasa dalam lalu lintas.

Setelah Anda menggunakan fitur ini, jendela baru akan terbuka dan menampilkan kredensial yang terdeteksi. Jendela tersebut akan menampilkan nomor paket, protokol, nama pengguna, dan informasi tambahan. Jendela ini dapat diklik; mengklik nomor paket akan memilih paket yang berisi kata sandi, dan mengklik nama pengguna akan memilih paket yang berisi informasi nama pengguna. Bagian tambahan akan menampilkan nomor paket yang berisi nama pengguna.
<img width="853" height="334" alt="image" src="https://github.com/user-attachments/assets/f81f97fe-4805-46e3-b375-cd3eebc881f5" />
Jawablah pertanyaan-pertanyaan di bawah ini.
Gunakan file "Desktop/exercise-pcaps/bonus/Bonus-exercise.pcap". Berapakah nomor paket kredensial yang menggunakan "HTTP Basic Auth"?

237

Berapakah nomor paket tempat "kata sandi kosong" dikirimkan?

170

Bonus: Hasil yang Dapat Ditindaklanjuti!
Bonus: Hasil yang Dapat Ditindaklanjuti!
Anda telah menyelidiki lalu lintas, mendeteksi anomali, dan membuat catatan untuk penyelidikan lebih lanjut. Apa selanjutnya? Tidak setiap investigasi kasus dilakukan oleh tim yang terdiri dari banyak orang. Sebagai analis keamanan, akan ada beberapa kasus di mana Anda perlu menemukan anomali, mengidentifikasi sumbernya, dan mengambil tindakan. Wireshark bukan hanya tentang detail paket; ia dapat membantu Anda membuat aturan firewall yang siap diimplementasikan hanya dengan beberapa klik. Anda dapat membuat aturan firewall dengan menggunakan  menu "Tools --> Firewall ACL Rules"  . Setelah Anda menggunakan fitur ini, jendela baru akan terbuka dan menyediakan kombinasi aturan (berdasarkan IP, port, dan alamat MAC) untuk berbagai tujuan. Perhatikan bahwa aturan ini dibuat untuk diimplementasikan pada antarmuka firewall eksternal .

Saat ini, Wireshark dapat membuat aturan untuk:

Netfilter (iptables)
Cisco IOS (standar/diperluas)
Filter IP (ipfilter)
IPFirewall (ipfw)
Filter paket (pf)
Firewall Windows (netsh format baru/lama)
<img width="879" height="422" alt="image" src="https://github.com/user-attachments/assets/50d465ab-47e9-4b01-8b76-8821c8251c74" />

Jawablah pertanyaan-pertanyaan di bawah ini.
Gunakan file "Desktop/exercise-pcaps/bonus/Bonus-exercise.pcap". Pilih nomor paket 99. Buat aturan untuk "IPFirewall (ipfw)". Apa aturan untuk "menolak alamat IPv4 sumber"?

add deny ip from 10.121.70.151 to any in


Pilih nomor paket 231. Buat aturan "IPFirewall". Apa aturan untuk "mengizinkan alamat MAC tujuan"?

add allow MAC 00:d0:59:aa:af:80 any in

Kesimpulan
Selamat! Anda baru saja menyelesaikan ruangan "Wireshark: Analisis Lalu Lintas".

Di ruangan ini, kita telah membahas cara menggunakan Wireshark untuk mendeteksi anomali dan menyelidiki peristiwa yang menarik pada tingkat paket. Sekarang, kami mengundang Anda untuk menyelesaikan ruangan tantangan Wireshark:  Carnage , Warzone 1 , dan Warzone 2 .

Wireshark adalah alat yang bagus untuk memulai investigasi keamanan jaringan. Namun, itu saja tidak cukup untuk menghentikan ancaman. Seorang analis keamanan harus memiliki pengetahuan IDS / IPS dan keterampilan alat yang lebih luas untuk mendeteksi dan mencegah anomali dan ancaman. Karena serangan semakin canggih secara konsisten, penggunaan berbagai alat dan strategi deteksi menjadi suatu kebutuhan. Ruang-ruang berikut akan membantu Anda melangkah maju dalam analisis lalu lintas jaringan dan deteksi anomali/ancaman.

Penambang Jaringan
Mendengus
Tantangan Menghirup - Dasar-Dasarnya
Tantangan Mengendus - Serangan Langsung
Zeek
Latihan Zeek
Meluap





 


 

