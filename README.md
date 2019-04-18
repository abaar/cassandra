# Single & Multi-Node Cassandra on Ubuntu/Bento

## What is Cassandra?

## Catatan
Semua konfigurasi pada percobaan ini ada di `REPOSITORY` ini. 
### Arsitektur
2 Buah Node masing-masing ber ip 192.168.2.2 (cassandra1) dan 192.168.2.3 (cassandra2)

## Jumpto
0. [What is Cassandra?](#what-is-cassandra)
1. [Arsitektur](#arsitektur)
1. [Installasi Single Node](#single-node)
    1. [Install Java](#1-install-java)
    2. [Install Cassandra](#2-install-cassandra)
    3. [Cek Status - Apakah sukses ?](#3-cek-status)
2. [Installasi Multi Node](#multi-node)
    1. [Delete data lama yang mungkin tersimpan](#1-delete-data-yang-mungkin-tersimpan)
    2. [Konfigurasi cassandra-yaml(utama)](#2-konfigurasi-cassandra-yaml)
    3. [Konfigurasi rak](#3-konfigurasi-rak)
    4. [Jalankan Kembali Cassandra](#4-jalankan-kembali-cassandra)
    5. [Apabila Node Belum terdeteksi?](#5-apabila-node-belum-terdeteksi)
3. [Import Data](#import-data)
    1. [Masuk ke Cassandra](#1-masuk-ke-cassandra)
    2. [Buat/Gunakan Keyspace](#2-buat-atau-gunakan-keyspace)
    3. [Buat ColumnFamily](#3-buat-columnfamily)
    4. [Import data ke Cassandra](#4-import-data-ke-cassandra)
4. [CRUD Cassandra](#crud-cassandra)
    1. [Read imported data](#1-read-imported-data)
    2. [Insert Data](#2-insert-data)
    3. [Update Data](#3-update-data)
    4. [Delete Data](#4-delete-data)
5. [Kesimpulan](#kesimpulan)
    
## Single Node
#### 1. Install Java
Karena Cassandra perlu JVM maka kita perlu untuk menginstall Java, dapat dilakukan dengan syntax berikut
```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-set-default
```
Lalu anda bisa mengecek apakah java telah terinstall di node kalian dengan syntax berikut :
```
java -version
```
Kira kira hasilnya akan begini:

![Jre Installed](https://github.com/abaar/cassandra/blob/master/screenshoot/jre%20installed.PNG)


#### 2. Install Cassandra
Kali ini kita akan mencoba install Cassandra versi 2.2, apabila anda ingin menginstall versi terbaru , misalkan versi terbaru adalah 2.3 maka ganti `22x` dengan `23x`.
```
echo "deb http://www.apache.org/dist/cassandra/debian 22x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
echo "deb-src http://www.apache.org/dist/cassandra/debian 22x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
```
Lalu lakukan / tambahkan `signature` berikut agar tidak ada `warning` yang muncul.
```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys A278B781FE4B2BDA
gpg --keyserver pgp.mit.edu --recv-keys F758CE318D77295D
gpg --export --armor F758CE318D77295D | sudo apt-key add -

gpg --keyserver pgp.mit.edu --recv-keys 2B5C1B00
gpg --export --armor 2B5C1B00 | sudo apt-key add -

gpg --keyserver pgp.mit.edu --recv-keys 0353B12C
gpg --export --armor 0353B12C | sudo apt-key add -

sudo apt-get update
```
Lalu lakukan instalasi Cassandra dengan syntax berikut
```
sudo apt-get install cassandra
```

#### 3. Cek Status
Untuk mengecek apakah cassandra berjalan, dapat dilakukan dengan syntax berikut :
```
sudo service cassandra status
```
Syntax diatas akan memunculkan notif `running` seperti gambar dibawah

![Cassandra Running](https://github.com/abaar/cassandra/blob/master/screenshoot/cassandra%20running.PNG)

```
sudo nodetool status
```
Sedangkan syntax diatas akan menampilkan output seperti gambar dibawah apabila anda telah sukses menjalankan Cassandra

![Nodetool Status](https://github.com/abaar/cassandra/blob/master/screenshoot/cassandra%20nodetool-connected.PNG)

Lalu masuk kedalam cassandra dengan synax berikut:
```
cqlsh
```

![Connected to Cassandra](https://github.com/abaar/cassandra/blob/master/screenshoot/cassandra%20cluster-connected.PNG)


## Multi Node
Untuk membuat multi node tentu anda perlu node yang telah di install Cassandra, seperti 2 buah node pada gambar berikut :

![ready to add](https://github.com/abaar/cassandra/blob/master/screenshoot/ready-to-add.PNG)

#### 1. Delete data yang mungkin tersimpan
Sebelum melanjutkan konfigurasi agar Cassandra saling terkoneksi, anda harus menghapus file data di masing-masing node dengan syntax berikut:
```
sudo service cassandra stop
sudo rm -rf /var/lib/cassandra/data/system/*
```

#### 2. Konfigurasi cassandra yaml
Anda perlu mengubah `cassandra.yaml` file konfigurasi yang terletak pada `/etc/cassandra/cassandra.yaml` seperti berikut :
```
.....
cluster_name: 'Test Cluster' #Nama Clustermu, harus sama tiap node!
.....
seed_provider:
  - class_name: org.apache.cassandra.locator.SimpleSeedProvider
    parameters:
         - seeds: "192.168.2.2,192.168.2.3"
         #ip node 1,ip node 2, dan seterusnya
         #ingat, dalam bentuk string yang dipisah ',' antar IP
.....
listen_address: 192.168.2.2 #ip masing-masing node tempat cassandra berjalan , berbeda setiap node
.....
rpc_address: 192.168.2.2 #ip masing-masing node tempat cassandra berjalan , berbeda setiap node
.....
endpoint_snitch: GossipingPropertyFileSnitch
```
Anda dapat melihat contoh konfigurasi pada percobaan saya [disini](https://github.com/abaar/cassandra/blob/master/cassandra.yml) dan [disini](https://github.com/abaar/cassandra/blob/master/cassandra2.yml).

#### 3. Konfigurasi Rak
Pastikan konfigurasi `/etc/cassandra/cassandra-rackdc.properties`  sama di setiap node agar cassandra bisa dijalankan
```
dc=datacenter1 #pastikan isian sama
rack=rack1
```

#### 4. Jalankan Kembali Cassandra
````
sudo service cassandra start
sudo nodetool status
````
Apabila sukses, maka akan muncul kedua node dalam status cluster yang aktif seperti gambar dibawah

![multinode cassandra](https://github.com/abaar/cassandra/blob/master/screenshoot/multinode%20succeed.PNG)

#### 5. Apabila Node belum terdeteksi
Apabila gagal , maka anda perlu memberikan akses node untuk mengakses node lainnya dengan `IPTABLES` dengan menjalankan syntax berikut :
```
sudo iptables -A INPUT -p tcp -s 192.168.2.3 -m multiport --dports 7000,9042 -m state --state NEW,ESTABLISHED -j ACCEPT
#Ganti 192.168.2.3 dengan masing-masing node, contoh kasus saya syntax ini dijalankan di 192.168.2.2
```
Apabila anda belum memiliki `IPTABLES` maka install-lah terlebih dahulu dengan syntax :
```
sudo apt-get install iptables -y
```
## Import Data
#### 1. Masuk ke Cassandra
#### 2. Buat atau Gunakan Keyspace
#### 3. Buat ColumnFamily
#### 4. Import Data Ke Cassandra

## CRUD Cassandra
#### 1. Read Imported Data
#### 2. Insert Data
#### 3. Update Data
#### 4. Delete Data

## Kesimpulan
Kesimpulannya , `Cassandra` merupakan Database No-SQL yang memang memiliki dukungan arsitektur multi-node secara default. Hal ini dapat dilihat dengan mudahnya proses instalasi Cassandra dalam multi-node, tidak seperti MySQL cluster yang lebih _repot_. Anda juga bisa mengakses node lain dengan syntax
```
cqlsh ip-node 9042
```
Seperti gambar dibawah dimana `cassandra2` memiliki ip `192.168.2.3`

![Connect to others](https://github.com/abaar/cassandra/blob/master/screenshoot/connected-cqlsh.PNG)
