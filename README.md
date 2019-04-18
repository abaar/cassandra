# Single & Multi-Node Cassandra on Ubuntu/Bento
## Tools
1. cmder
2. windows command prompt
3. vagrant
4. Git bash

## Catatan
Semua konfigurasi pada percobaan ini ada di `REPOSITORY` ini. 
### What is Cassandra?
Cassandra adalah database NoSQL yang _mirip_ dengan MySQL, Cassandra memiliki `Keyspace` & `ColumnFamily` layaknya MySQL yang memiliki `Database` dan `Table`. Berbeda dengan MongoDB, Cassandra hanya memiliki arsitektur `Masterless` dan replikasi dilakukan secara `ASYNCHRONOUS` dengan teknis penyimpanannya dalam sebuah cache memtable yang mana ketika telah penuh data tersebut di _flush_ kedalam Disk. Sama seperti database NoSQL lainnya, Cassandra dibuat sedemikian rupa sehingga dapat menyokong arsitektur yang terdistribusi secara default.

### Arsitektur
2 Buah Node masing-masing ber ip 192.168.2.2 (cassandra1) dan 192.168.2.3 (cassandra2)

### Dataset
Dataset yang digunakan adalah dataset `googleplaystore` dari [kaggle](https://www.kaggle.com/lava18/google-play-store-apps#googleplaystore_user_reviews.csv). Dataset ini tentang informasi dari beberapa aplikasi yang ada di google play seperti Nama Aplikasi, Rating, Reviews , Ukuran Size , dan lain sebagainya. 

## Jumpto

0. [What is Cassandra?](#what-is-cassandra)
1. [Arsitektur](#arsitektur)
2. [Dataset](#dataset)
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
    1. [Insert Data](#1-insert-data)
    2. [Read imported data](#2-read-imported-data)
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
Langkah pertama yang harus dilakukan adalah masuk kedalam cassandra melalui `CQLSH` , karena disini saya menggunakan arsitektur Multi-Node maka syntax yang diperlukan adalah :
```
cqlsh 'any-ip-node-of-the-cluster' [9042]
```

![Masuk kedalam Cassandra](https://github.com/abaar/cassandra/blob/master/screenshoot/init.PNG)


#### 2. Buat atau Gunakan Keyspace
Selanjutnya, buatlah keyspace, istilahnya keyspace ini mirip dengan database pada MySQL,berikut contoh cara membuatnya :
```
CREATE KEYSPACE IF NOT EXISTS distributed_database WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'datacenter1' : 2 };
#ganti 'distributed_database' dengan nama keyspace lain sesuai kehendak
#class replication ada 2 macam, yang kita gunakan adalah networktopologystrategy dengan replication factor 2
```
Untuk info lebih lanjut mengenai topology & replication pada cassandra dapat dilihat [disini](https://docs.datastax.com/en/cql/3.3/cql/cql_reference/cqlCreateKeyspace.html)

![creating keyspace](https://github.com/abaar/cassandra/blob/master/screenshoot/create%20keyspaces.PNG)

#### 3. Buat ColumnFamily
Lalu, buatlah ColumnFamily, atau dalam MySQL istilahnya adalah tabel, berikut contoh cara membuatnya :
```
create columnfamily application(
    app varchar, 
    category varchar, 
    rating double, 
    reviews varchar, 
    size varchar, 
    installs varchar, 
    type varchar, 
    price_in_dollars varchar, 
    content_rating varchar, 
    genres varchar, 
    last_update varchar, 
    current_ver varchar, 
    android_ver varchar, 
    primary key(app)
    );
# create columnfamily 'names' ('column_name' 'type')
```
Untuk informasi lebih lanjut mengenai tipe data pada CQL Cassandra dapat dilihat [disini](https://www.tutorialspoint.com/cassandra/cassandra_cql_datatypes.htm)

![creating columnfamily](https://github.com/abaar/cassandra/blob/master/screenshoot/create%20columnfamily.PNG)


#### 4. Import Data Ke Cassandra
Setelah itu lakukan import data kedalam table yang telah dibuat dengan syntax
```
COPY application(app,category,rating,reviews,size,installs,type,price_in_dollars,content_rating,genres,last_update,current_ver,android_ver) from 'googleplaystore.csv' with header=true;

#Copy 'columnfamily'('its column') from 'dataset' with [optional parameter]
```
Adapun keterangan optional parameter dapat dilihat [disini](https://docs.datastax.com/en/cql/3.3/cql/cql_reference/cqlshCopy.html)

![import data](https://github.com/abaar/cassandra/blob/master/screenshoot/imported.PNG)




## CRUD Cassandra
Sebelum melakukan CRUD data dalam cassandra , perlu terhubung dengan `CQLSH` dan `Keyspace` (optional) dimana data berada. Caranya
```
USE 'KEYSPACE_NAME'
```

![use keyspace](https://github.com/abaar/cassandra/blob/master/screenshoot/use%20keyspace.PNG)
#### 1. Insert Data
Untuk insert kurang lebih mirip dengan syntax yang ada pada MySQL...
```
insert into application(
    app,
    android_ver, 
    category,
    content_rating,
    current_ver, 
    genres, 
    installs, 
    last_update, 
    price_in_dollars, 
    rating, 
    reviews, 
    size, 
    type
    )values(
        'Sebuah apps 2', 
        '1.1 Up',
        'Asal-asalan', 
        '3+' , 
        '1.1', 
        'Music', 
        '1', 
        'April 9, 2019', 
        'FREE', 
        4.7 ,
        'Mantulll!',
        '10M',
        'Free'
        );
```

![insert](https://github.com/abaar/cassandra/blob/master/screenshoot/insert.PNG)

#### 2. Read Imported Data
```
select * from application where app='Sebuah apps';

#select 'column of columnfamily' from [keyspace].'columnfamily' where 'indexed column'
```
Perlu diingat bahwa untuk `select` data dengan klausa `where` harus menggunakan `Indexed Column` seperti `Primary Key` atau yang lainnya.

![example of select](https://github.com/abaar/cassandra/blob/master/screenshoot/example%20of%20select.PNG)

![SELECT INSERTED](https://github.com/abaar/cassandra/blob/master/screenshoot/select_inserted.PNG)


#### 3. Update Data
Sama seperti Insert , kurang lebih syntax Update yang digunakan pada `CQL Cassandra` sama seperti MySQL
```
update application set last_update = '10 April, 2019' where app='Sebuah apps'

update [keyspace].'columfamily' set 'column' = 'updated_data' where 'indexed_column' = 'data'
```

![update](https://github.com/abaar/cassandra/blob/master/screenshoot/select_updated.PNG)

#### 4. Delete Data
Sama seperti Insert & Update , kurang lebih syntax Delete yang digunakan pada `CQL Cassandra` sama seperti MySQL
```
delete from application where app='Sebuah apps'

#delete from [keyspace].'columnfamily' where 'indexed_column'='data
```

![delete](https://github.com/abaar/cassandra/blob/master/screenshoot/select_deleted.PNG)


## Kesimpulan
Kesimpulannya , `Cassandra` merupakan Database No-SQL yang memang memiliki dukungan arsitektur multi-node secara default. Hal ini dapat dilihat dengan mudahnya proses instalasi Cassandra dalam multi-node, tidak seperti MySQL cluster yang lebih _repot_. Anda juga bisa mengakses node lain dengan syntax
```
cqlsh ip-node 9042
```
Seperti gambar dibawah dimana `cassandra2` memiliki ip `192.168.2.3`

![Connect to others](https://github.com/abaar/cassandra/blob/master/screenshoot/connected-cqlsh.PNG)
