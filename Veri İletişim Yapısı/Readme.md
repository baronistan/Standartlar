![Blok Diagram](/Veri%20%C4%B0leti%C5%9Fim%20Yap%C4%B1s%C4%B1/Images/Kafka%20Block%20Diagram.jpg)

# Veri İletişim Protokolü [01.02.00]

"STF Tarım" için **Temmuz 2022** ve sonrası geliştirilmekte olan tüm IoT cihazları için kullanılacak olan veri iletişim yapısı ve teknik detayları bu doküman içerisinde tanımlanmıştır. Diğer tüm **STF** projeleri ile birlikte yapılan planlama doğrultusunda tüm cihazların tek bir **EndPoint** üzerinden çalışması uygun görülmüştür. End point üzerinden hangi cihazın hangi şartlarda veri göndereceği ve backend üzerinden bu ayrımın nasıl yapılması gerektiği bu düküman üzerinden tanımlanmıştır. 

## 1. - Cihaz Veri Gönderim ve IoT Bağlantı Yapısı

Geliştirilmekte olan cihazlar bağantı ve veri gönderim yapısı itibari ile değişiklikler göstermektedir. Bazı cihazlarımız elektrik bağlantılı çalışmaktadır ve bu tip cihazlarımız proje doğrultusunda GSM üzerinden sürekli bağlı kalmaktadır. Bunun yanı sıra bazı cihazlarımız da batarya destekli olarak çalışmaktadır bu nedenle uyu-uyan-uyu yapısında veri gönderimi yapmaktadır. Devamlı uyanık olan sistemlerimiz sunucu üzerinden komut alabilir yapıdadır. Uyu-uyan-uyu sistemlerimiz ise tek yönlü veri gönderim yapısına sahiptir.

Bu sistematik ve cihaz özelliği doğrultusunda veri paketi içerisinde yer alan payload segmenti değişiklik gösterebilecektir. Bu nedenle **device** segmenti içerisinde bir komut tanımı yer almaktadır. Bu komut tanımına göre payload segmenti yapısı ilgili bölümde detaylıca anlatılacaktır.

### 1.A. - WeatherStat Veri Gönderim Yapısı

Tarımsal meteoroloji sistemi kurgu gereği tek yönlü veri trafiğiğine sahip bir sistemdir. Bu kurgu bünyesinde aşağıdaki koşullarda IoT veri transferi yapılacaktır. WeatherStat (P101) sistemi uyuyan uyanan bir donanım kurgusuna sahiptir. Bu kurgu gereği sistem donanım olarak belirlenen süre periodlarında (30 dk da bir) uyanarak veri paketi hazırlayacak ve sunucuya veri gönderimi yapacaktır. 

* Cihaz güç altyapısı : Batarya (solar şarjlı)
* Cihaz çalışma sistemi : Uyu-Uyan
* İletişim : Tek yönlü (cihaz --> sunucu)

### 1.B. - PowerStat Veri Gönderim Yapısı

PowerStat sistemi kurgu gereği elektrik panosu üzerine montajlanmaktadır. Bu nedenle güç sorunu bulunmamaktadır. Elektrik olduğu sürece kendisini GSM şebekesine bağlı tutacak elektrik gitmesi durumunda ise elektriğin gittiğini haber ederek GSM i kapatıp uyku moduna geçecektir. Cihaz bağlı durumdayken belirlenen veri tipleri ile gönderim yapacak ve sunucudan veri alabilecektir. 

* Cihaz güç altyapısı : Batarya (220V şarjlı)
* Cihaz çalışma sistemi : Devamlı uyanım (enerji varken)
* İletişim : Çift yönlü (cihaz <--> sunucu)

### 1.C. - FilterStat Veri Gönderim Yapısı
...

## 2. - Veri Paketi Yapısı

Tüm projeler ortak bir veri iletişim altyapısı içerisinde birleştirilmiş ve ortak bir iletişim standardı belirlenmiştir. Bu standart doğrultusunda veri paketi aşağıda tanımlandığı şekilde gönderilecektir.

    Örnek Veri Paketi (WeatherStat)

```json
{
    "Command": "STF:PowerStat.Interrupt",
    "Device": {
        "Info": {
            "ID": "70A11D1D01000026",
            "Temperature": 27.91341,
            "Humidity": 24.7523
        },
        "Power": {
            "Battery": {
                "AC": -0.15,
                "IV": 4.17,
                "SOC": 92.13,
                "Charge": 3
            }
        },
        "IoT": {
            "GSM": {
                "Operator": {
                    "IP": "192.168.0.1",
                    "Code": 28601,
                    "RSSI": 10,
                    "LAC": "855E",
                    "Cell_ID": "BFAB"
                }
            }
        }
    },
    "Payload": {
        "TimeStamp": "2022-03-23  14:18:28",
        "PowerStat": {
            "DeviceStatus": 240,
            "FaultStatus": 500,
            "Pressure": {
                "Min": 6.792786,
                "Max": 6.841798,
                "Avg": 6.801981,
                "Inst": 6.819542,
                "Slope": -0.000488,
                "Offset": 16,
                "R2": 0.724861,
                "DataCount": 48
            },
            "Energy": {
                "Voltage": [222.0329, 220.3512, 223.6668],
                "Current": [0.007388, 0.006879, 0.007192],
                "PowerFactor": -0.15941,
                "Consumption": [60697, 14007, 60697],
                "Frequency": 50.02493
            }
        }
    }
}
```

| Segment | Açıklama                                                                              | Detaylar                                         |
|---------|:--------------------------------------------------------------------------------------|--------------------------------------------------|
| Command | Paketin hangi cihaza ait komut için gönderildiğini tanımlayan komut parametresi.      | [Segment yapısı ve detayları](Command/Readme.md) |
| Device  | Cihaza ait tanımlayıcı bilgilerin yer aldığı segment (tüm cihazlar için aynı yapı).   | [Segment yapısı ve detayları](Device/Readme.md)  |
| Payload | Veri paketine ait ana verinin yer aldığı segment (herbir command için farklı yapıda). | [Segment yapısı ve detayları](Payload/Readme.md) |

## 3. - Cevap Paketi Yapısı

Tüm IoT cihazlarımız sistemi yapı gereği sunuculara veri gönderimi yaptıktan sonra cevap için beklemede kalır. Sunucu tarafından verinin uygun şekilde yakalandığını belirten ibare gelmesi durumunda sistem rutin işlemine devam eder (uyuyan cihazlar tekrar uyku moduna döner). Eğerki onay mesajı gelmemesi durumunda veri kaydını tekrar gönderir (5 deneme yapar ve hala kayıt olmadı ise uyur). Bu nedenle gönderilen veri paketine karşılık olarak aşağıdaki yapıda bir cevap sunucu tarafından dönecektir.


```json
{
	"Event": 200,
}
```

Event değişkeni içerisinde gönderilecek olan kod numaraları HTTP Header tanımları ile aynıdır (aşağıda belirtilmiştir). Bu kod numaraları artabilir olduğu ve duruma göre ekleme yapılabileceği unutulmamalıdır (tabloda belirtilen kodlar sabittir).

| Result  | Açıklama                                    |
|---------|---------------------------------------------|
| 200     | Status Ok : Veri düzgün şekilde kaydedildi. |
