# Python kullanarak Basit Blockchain Uygulaması

##Gereksinimler

`````
Postman Desktop
Python 3
Pycharm
Flask
`````

###Kullanılan Kütüphaneler

`````Python
import datetime
import json
import hashlib
from flask import  Flask, jsonify
`````
- Oluşturulan ve çıkarılan her bir bloğun zaman etiketlenmesi için *datetime* kütüphanesi kullanıldı.
- *hashlib* bır bloğu hashlemek yani şifrelemek için kullanıcağız.
- Mesajları postman üzerinde döndürmek için Flask kütüphanesi içerisinden jsonify kullanacağız.

---

Düğümleri oluşturma oluşturmak için Blaock chain sınıfını oluşturuyoruz. tüm düğümleri bir listesini kaydetmek için ``__init__`` fonksiyonunu değişken oluşturuyoruz.

`````python
class Blockchain:
   def __init__(self):
       self.chain = []
       self.create_blockchain(proof=1, previous_hash='0')
`````

###create_blockchain fonksiyonun oluşturulması 
Düğümün yani zincirin gelişleyebilmesi için bu fonksiyon kullanılacaktır.
3 tane parametresi mevcuttur;
- self
- proof
- previous_hash

>create_blockchain fonksiyonu dictionary türünde blok değişkeni ekler.

key-value şeklinde değişkenler alır:
* **Index:** Düğümün uzunluğunu tespit etmek için kullanıcağız. düğüme erişmek içinde bu değişkeni kullanacağız.
* **Timestamp:** Bloğun oluştuğu şimdiki zamanı alır.
* **Proof:** Çağrıldığında işleve iletilicek bir kanıt değeri alacaktır.
* **Previous_hash:** Önceki bloğun hash değerini alır.
````python
def create_blockchain(self, proof, previous_hash):
   block = {
       'index': len(self.chain) + 1,
       'timestamp': str(datetime.datetime.now()),
       'proof': proof,
       'previous_hash': previous_hash
   }

   self.chain.append(block)
   return block
````
---
###Önceki bloğa erişim
tüm düğümleri alan yeni nbir değişken oluşturulur ve son değer atanır.
````python
def get_previous_block(self):
   last_block = self.chain[-1]
   return last_block
````
---
###İş kanıtı fonksiyonu çalışması
Yeni düğüm oluştururken bu değişkeni kullanmıştık, yani bir blok madenciliği yapmak için yapılan iş kanıtını temsil eder.
iki tane parametre vericez, ```self```, ``previous_proof``.
proof varsayılan olarak False veriyoruz, proofları kontrol ederken bu değişkeni kullanıcağız.
`````python
def proof_of_work(self, previous_proof):
   # miners proof submitted
   new_proof = 1
   # status of proof of work
   check_proof = False
`````
bir while döngüsü kanıt bulana kadar dönücek, program iş kanıtını kontrol etmiş ise while döngüsünden geçer.

`hashlib`değişkeni SHA256 kütüphanesini kullanarak onaltılık basamakta kriptograjik şifreleme yapar.

Algoritma, kullanıcı tarafından sunulan yeni ispatı alır ve onu 2'nin kuvvetine yükseltir, ardından onu önceki ispatın üssünden 2'nin kuvvetine yükseltilmiş olan üssünden çıkarır.
````python
hash_operation = hashlib.sha256(str(new_proof ** 2 - previous_proof ** 2).encode()).hexdigest()
````
