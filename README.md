# Single & Multi-Node Cassandra on Ubuntu/Bento

## Pre - Requisite

## Jumpto

## Single Node
### Requirement
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
Lalu lakukan / tambahkan `signature` berikut agar tidak ada `warning` yang muncul, sebenarnya bisa ditinggalkan dengan risiko ditanggun sendiri. Karena saya pribadi tidak memasukkannya
```
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
