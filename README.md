# Python kullanarak Basit Blockchain Uygulaması

## Gereksinimler

`````
Postman Desktop
Python 3
Pycharm
Flask
`````

### Kullanılan Kütüphaneler

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

### create_blockchain fonksiyonun oluşturulması 
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
### Önceki bloğa erişim
tüm düğümleri alan yeni nbir değişken oluşturulur ve son değer atanır.
````python
def get_previous_block(self):
   last_block = self.chain[-1]
   return last_block
````
---
### İş kanıtı fonksiyonu çalışması
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

### Gerekli kontrollerin yapılması
`hash_operation` fonksiyonunda hash değişkeninin ilk 4 karakterini sıfıra eşit olup olmadığını kontrol edilicek.

Eğer **True** dönerse kanıtı kontrol ettik ve geçerli anlamındadır.

Eğer **False** dönerse tekrar denemesi için new_proof değerini 1 arttırırız (neden anlamadım??) ve ardında yeni kanıt döndürülür.

````python
def proof_of_work(self, previous_proof):
   # miners proof submitted
   new_proof = 1
   # status of proof of work
   check_proof = False
   while check_proof is False:
       # problem and algorithm based off the previous proof and new proof
       hash_operation = hashlib.sha256(str(new_proof ** 2 - previous_proof ** 2).encode()).hexdigest()
       # check miners solution to problem, by using miners proof in cryptographic encryption
       # if miners proof results in 4 leading zero's in the hash operation, then:
       if hash_operation[:4] == '0000':
           check_proof = True
       else:
           # if miners solution is wrong, give mine another chance until correct
           new_proof += 1
   return new_proof
````
#### Peki neden 0 sıfır olması gerekiyor?
Önde ne kadar sıfıra  ihtiyac duyarsak işlem yapamız o kadar zorlaşıcaktır. Bu örnekte 4 adet sıfır yeterli olacaktır.

Ayrıca, `hash_operation`simetrik olmamalıdır, yani işlem ttersine çevrildiği zaman aynı sonuclar gelmeyecektir. Örneğin, a+b == b+a ama a-b !=b-a.

### Hash Oluşturma
````python
# generate a hash of an entire block
def hash(self, block):
   encoded_block = json.dumps(block, sort_keys=True).encode()
   return hashlib.sha256(encoded_block).hexdigest()
````

### Blok Zincirinin Gecerliliğini Kontrol Etme
Son bloğu değişkene atarız
````python
while block_index < len(chain):
block = chain[block_index]
````
Sonrasında ise mevcut bloğun önceki blok ile hash alanlarını benzer olup olmadığını kontrol ederiz.
````python
if block["previous_hash"] != self.hash(previous_block):
   return False
````
Önceki bloğun ve şuanki bloğun hash değerleri bir değişkene atanır.
Ardındanilk dört karakterin sıfıra eşit olup olmadığını kontrol edilir. 

````python
# check if the blockchain is valid
def is_chain_valid(self, chain):
   # get the first block in the chain and it serves as the previous block
   previous_block = chain[0]
   # an index of the blocks in the chain for iteration
   block_index = 1
   while block_index < len(chain):
       # get the current block
       block = chain[block_index]
       # check if the current block link to previous block has is the same as the hash of the previous block
       if block["previous_hash"] != self.hash(previous_block):
           return False

       # get the previous proof from the previous block
       previous_proof = previous_block['proof']

       # get the current proof from the current block
       current_proof = block['proof']

       # run the proof data through the algorithm
       hash_operation = hashlib.sha256(str(current_proof ** 2 - previous_proof ** 2).encode()).hexdigest()
       # check if hash operation is invalid
       if hash_operation[:4] != '0000':
           return False
       # set the previous block to the current block after running validation on current block
       previous_block = block
       block_index += 1
   return True
````
### Blok zincirinde madencilik 
Flask kullanarak bir ve uygulaması oluşturuyoruz.
````python
app = Flask(__name__)
````
Blockcahin sınıfından bir nesne oluşturuyoruz.
````python
blockchain = Blockchain()
````

GET methodunu kullanarak bir Flask rotası oluşturuyoruz.
```python
@app.route('/mine_block', methods=['GET'])
def mine_block():
```

mine_block içerisinde bir blok oluşturmak için ihtiyacımız olan verileri getiriyoruz.
- önceki blok
- önceki blogun kanıtı (proof)
- iş kanıtı (the proof of work)
- önceki hash kodu

````python
previous_block = blockchain.get_previous_block()
previous_proof = previous_block['proof']
proof = blockchain.proof_of_work(previous_proof)
previous_hash = blockchain.hash(previous_block)
````

yeni bir blok değişkenine  bir blok oluşturuyoruz.
`````python
block = blockchain.create_blockchain(proof, previous_hash)
`````

geri dönücek json kodunu yazıyoruz.
````python
response = {'message': 'Block mined!',
           'index': block['index'],
           'timestamp': block['timestamp'],
           'proof': block['proof'],
           'previous_hash': block['previous_hash']}
return jsonify(response)
````

Aynı şekilde GET methodu kullanarak bütün zinciri ve zincir uzunluğunu getiriyoruz.
````python
@app.route('/get_chain', methods=['GET'])
def get_chain():
   response = {'chain': blockchain.chain,
               'length': len(blockchain.chain)}
   return jsonify(response), 200
````

### Sonuç
port

![port](https://github.com/HasanBeratSoke/simple_blockchain/blob/main/port.png)

mine_block

![mine](https://github.com/HasanBeratSoke/simple_blockchain/blob/main/mine_block.png)

get_chain

![get](https://github.com/HasanBeratSoke/simple_blockchain/blob/main/get_chain.png)

validation

![valid](https://github.com/HasanBeratSoke/simple_blockchain/blob/main/valıd.png)
