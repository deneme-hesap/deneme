# Integration Worker Service

Bu proje, çeşitli veri kaynaklarından veri toplayan, dönüştüren ve bu verileri Elasticsearch'e gönderen bir veri entegrasyon pipeline'ını içermektedir. Ayrıca, belirli durumlarda bildirim gönderme işlevi de bulunmaktadır.


## Veri Kaynakları

- **JDBC**
- **JSON**
- **XML SOAP**
- **CSV FILE**
- **S3**

## Logstasg.conf Yapısı

### İNPUT
Bu yapılandırma, belirli bir URL'ye düzenli olarak POST istekleri gönderir, gelen yanıtları alır ve bu yanıtları Logstash pipeline'ında işlemek üzere giriş olarak kullanır.

```
input {
  http_poller {
    urls => {
      test1 => "https://jsonplaceholder.typicode.com/todos"
    }
    request_timeout => 60
    schedule => { "every" => "24h" }
    tags => ["get_users_to_elastic"]
    codec => "plain"
  }
}
```
#### Temel Parametreler
- **request_timeout**
request_timeout, isteğin zaman aşımına uğramadan önce bekleyeceği süreyi (saniye cinsinden) belirtir.

- **schedule**
isteklerin ne sıklıkta yapılacağını belirler. 

- **tags**
olaylara eklenen etiketlerdir. Bu etiket filtreleme ve çıktılama aşamalarında olayları tanımlamak için kullanılabilir.(elasticsearch'e basma veya loglama vb.)

- **codec**
verilerin nasıl kodlanacağını veya kodlarının çözüleceğini belirtir. 


### FİLTER
Logstash'in verileri işleme ve dönüştürme aşamasını tanımlar. Bu yapı, gelen verileri filtreler, dönüştürür ve zenginleştirir. Farklı pipeline'lardan gelen veriler varsa bunların kontrolünü gerçekleştirip her biri için ayrı ayrı işlemler gerçekleştirebilir

```
filter {
    if "get_data_to_elastic" in [tags] {
        ruby {
            path => "/src/stores.rb"
        }
    }
}
```

Bu yapı, belirli bir koşulun sağlandığı durumlarda ek işlemler yapabilmek için kullanışlıdır. Bu durumda, "get_data_to_elastic" etiketine sahip olan veriler için özel bir Ruby betiği çalıştırılarak ek işlevsellik sağlanabilir.

### OUTPUT
Logstash'in filter aşamasında eklenen tagların kontrolü yapılarak, işlenmiş verileri nereye ve nasıl göndereceğini tanımlar. Tag eklenmek zorunda değildir

#### TEMEL PARAMETRELER
- **HOSTS**
Elasticsearch kümesine bağlantı kurulacak sunucu adreslerini belirtir

- **index**
- Verilerin gönderileceği indeksin adını belirtir.

- **document_id**
Belgenin kimliğini belirler. Her belgenin benzersiz bir kimliğe sahip olmasını sağlar.

- **doc_as_upsert**
- Belirtilen belge kimliği zaten mevcutsa, belgeyi güncellemek için kullanılır değilse yeni bir belge oluşturur.

- **action**
- Belgelerin Elasticsearch'te nasıl işleneceğini belirtir.

