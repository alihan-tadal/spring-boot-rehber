# Giriş
Bu rehberde Spring ile bir "Hello, World" web servisinin nasıl geliştirileceği anlatılmaktadır. 

## Genel Bilgiler
Geliştirilecek olan proje, `http://localhost:8080/greeting` adresine gönderilen HTTP GET isteklerini dinler ve bir JSON cevabı verir. Örnek bir cevap aşağıdaki gibi olabilir:
```json
{"id":1,"content":"Hello, World!"}
``` 
Kullanıcı, gönderdiği istekte `name` parametresini özelleştirmeyi tercih ederse cevap buna uygun şekilde üretilecektir. Örnek senaryo aşağıdaki gibi olabilir:

İstek: 
```
http://localhost:8080/greeting?name=Kullanici1
```

Yanıt:
```json
{"id":1,"content":"Merhaba, Kullanici1!"}
```

## Gereksinimler

Rehberi tamamlayabilmeniz için aşağıdakilere ihtiyacınız olacaktır:
- Yaklaşık 15 dakika
- Bir text editörü ya da IDE
- Java 17 veya daha güncel bir versiyonu
- Gradle 7.5+ ya da Maven 3.5+

## Bu rehberi nasıl tamamlayabilirsiniz?

Spring rehberlerinin çoğunda olduğu gibi, bu rehber için de sıfırdan başlamayı tercih edebilir ya da temel aşamaları atlayarak doğruca geliştirmeye başlayabilirsiniz. Her durumda, elinizde çalışan bir kod olacaktır. 

Seçenek 1: Sıfırdan başlamak için, aşağıdaki başlıktan devam ediniz. 

Seçenek 2: 
Temel aşamaları atlamak için:
- `git clone https://github.com/spring-guides/gs-rest-service.git` komutuyla repo'yu bilgisayarınıza klonlayın.
- cd komutu ile `gs-rest-service/initial` dizinine geçin.
- Eğer her şey yolundaysa, [Resource Representation Class Oluşturma](#resource-representation-class-oluşturma) başlığına geçiniz.

## Spring initializr ile başlamak
>⚠️ Eğer repo üzerinden devam edecekseniz, bu başlığı atlayabilirsiniz.

Bir Spring projesi başlatmak Spring Initializr online aracını kullanabilirsiniz. Bu araç projeniz için bir zip dosyası üretecektir. Aşağıdaki adımları izleyebilirsiniz:
- `https://start.spring.io` adresine gidiniz. Bu araç bütün gereksinimleri projenize ekleyebilir.
- Gradle veya Maven ile devam edebilirsiniz. Rehberde, dil olarak Java'nın seçildiği kabul edilecektir.
- **Dependencies** sekmesinden **Spring Web** seçiniz.
- **Generate** tuşuna tıklayınız.
- ZIP dosyasını indirin ve kaydedin.

## Resource representation class oluşturmak
Projenin temel kurlumunu tamamladıysanız, web servisinizi oluşturabilirsiniz. Servisin, kullanıcı ile gerçekleştirmesini beklediğiniz etkileşimleri düşünerek başlayabilirsiniz.

Servis, `/greeting` adresine gelen `GET` isteklerini kabul edecektir. Bu isteğe zorunlu olmayan bir `name` parametresi de eklenebilir. Bu istek kullanıcıya durum kodu olarak `200 OK` ve bununla birlikte karşılama mesajını içeren JSON döndürmelidir. Cevap aşağıdaki gibi olabilir:
```json
{
    "id": 1,
    "content": "Hello, World!"
}
```
Cevaptaki `id` alanı karşılamayı eşsiz biçimde tanımlayan bir alandır, karşılamanın kimliğidir. `content` alanı karşılamanın metin karşılığını içerir.

Bu karşılamayı oluşturmak için, bir "resource representation class" yazmakla başlayın. Bunu, id ve content verisi saklayan bir Java record oluşturarak yapabilirsiniz. Kodu, yaratacağınız `Greeting.java` dosyası içinde yazabilirsiniz. 
```Java
package com.example.restservice;

public record Greeting(long id, String content) {}
```

## Resource controller oluşturmak
Spring'de, RESTful web servislerine gelen HTTP istekleri bir controller tarafından kontrol edilir. Bu bileşenler `@RestController` annotator'u ile belirtilir. Bu örnekte, gelen istekleri yönetecek olan `GreetingController` aşağıdaki gibi yazılmıştır. Kodu, yaratacağınız `GreetingController.java` içine yazabilirsiniz. 
```Java
package com.exmaple.restservice;

import java.util.concurrent.atomic.AtomicLong;

import org.springframework.bind.annotation.GetMapping;
import org.springframework.bind.annotation.RestController;
import org.springframework.bind.annotation.RequestParam;

@RestController
public class GreetingController() {
    private static final String template = "Hello, %s!";
    private final AtomicLong counter = new AtomicLong();

    @GetMapping("/greeting")
    public Greeting greeting(@RequestParam(value = "name", defaultValue="World") String name) {
        return new Greeting(counter.incrementAndGet(), String.format(template, name))
    }
}
```

Bu controller basittir ancak aslında arkada birçok işlem döndürmektedir. `@GetMapping` annotation'u `/greeting`'e gelen HTTP GET isteklerinin `greeting()` metoduna yönlendirme işlemini yapar. 
> `@GetMapping` bu işi yapabilen tek annotation değildir, benzer işleri `@PostMapping` ve ikisinin de türediği `@RequestMapping` annotation'ları da yapabilir.

`®RequestParam`, istek içinde tercihe bağlı gelebilecek olan `name` parametresini `greeting()` metodunun parametresi olan `name`'e gömer. Eğer istekte `name` parametresi belirtilmemiş ise, `defaultValue` değer yine burada tanımlanır. Örnekte bu default değer `"World"` kelimesidir.

`greeting()` metodu, bir `Greeting` objesi yaratır ve bu objeyi döndürür. `counter` değeri, greeting metodu her çalıştığında artacak ve döndürülen Greeting objesi içine bir özellik olarak verilecektir. 

Normal bir HTTP cevabı ile RESTful cevabı arazındaki temel fark, cevap body'sinin nasıl oluşturulduğudur. Web sayfasını sunucu tarafında hazırlayıp cevabı bir HTML dosyası içine gömerek göndermek yerine, RESTful servis bir `Greeting` objesi oluşturup bu objeyi HTTP üzerinden JSON olarak gönderir. 

Bu kod, `@RestController` annotation içerdiği için, gelen isteklere view yerine bir nesne döndürür. Bu farklı yollarla sağlanabilir ancak en kestirme yol budur. Alternatif olarak `@Controller` ve `@ResponseBody` beraber kullanılabilirdi.

Cevap hazırlanırken, Greeting objesi bir JSON'a dönüştürülür. Spring'in HTTP mesaj dönüştürücüsü desteği sayesinde bu iş otomatik olarak gerçekleştirilir. **Jackson 2** classpath üzerinde olduğu için Spring'in `MappingJackson2HttpMessageConverter` aracı Greeting --> JSON dönüşümünü bahsedildiği gibi, otomatik gerçekleştirir.

`@SpringBootApplication` annotation'u ise aşağıdaki annotation'ların toplamda yaptığı işleri yapar:
- `@Configuration`: Sınıfı, uygulama bağlamı için _bean_ tanımlarının kaynağı olarak etiketler.
- `@EnableAutoConfiguration`: Bu annotation, Spring Boot'un otomatik yapılandırmasını etkinleştirir. Bu, uygulamanın ihtiyaç duyduğu Spring bileşenlerinin otomatik olarak yapılandırılmasını sağlar. 
- `@ComponentScan`: Spring'e `com/example` altındaki diğer komponentleri, konfigürasyonları ve servisleri aramasını söyler.


Geleneksel Java web uygulamaları geliştirirken, XML dosyaları ve web.xml dosyaları gibi çeşitli yapılandırma dosyaları kullanmak gerekiyordu. Bu dosyalar, uygulamanın hangi bileşenlerinin kullanılacağını ve nasıl yapılandırılacağını belirtmek için kullanılırdı. Bu yapılandırma dosyalarının oluşturulması ve yönetimi sıkıcı ve hatalara neden olabilirdi.
Ancak, Spring Boot'un kullanımı, bu yapılandırma dosyalarına ihtiyaç duymadan, sadece Java kodu kullanarak web uygulaması geliştirmenize olanak sağlar. Bu, uygulamanın başlatılması `SpringApplication.run()` metodu ile gerçekleştirilir.

## JAR dosyası build etmek
Geliştirdiğiniz yazılımı Maven ya da Gradle ile komut satırı üzerinden çalıştırabilirsiniz. Bunun yanında tüm gerekli sınıflar ve gereksinimleri içinde barındıran bir JAR dosyası da build edebilirsiniz. Çalıştırılabilir jar dosyasına sahip olmak, yazılım geliştirme döngüsü içindeki birçok adımı kolay hale getirir.

Eğer Gradle kullanıyorsanız, uygulamanızı `./gradlew bootRun` komutu ile çalıştırabilirsiniz. JAR dosyası elde etmek için ise aşağıdaki komutu çalıştırabilirsiniz:
```bash
java -jar build/libs/rest-service-0.1.0.jar
```
Eğer Maven kullanıyorsanız, `./mvnw spring-boot:run` ile uygulamanızı çalıştırabilirsiniz. JAR dosyası elde etmek için aşağıdaki komutu kullanabilirsiniz:
```bash
java -jar target/libs/rest-service-0.1.0.jar
```

JAR dosyaları yerine WAR dosyası olarak da çıktı alabilirsiniz. 

## Servisi test etmek
Servis çalıştığı için `http://localhost:8080/greeting` adresine yaptığınız isteklere cevap alabilirsiniz. Aynı servisi, bir `name` parametresi belirterek de kullanabilirsiniz. Örneğin: `http://localhost:8080/greeting?name=User`.
Yaptığınız her istekte, gelen cevaptaki `id` alanının değiştiğini gözlemleyebilirsiniz. Bu, counter'i yöneten `GreetingController` ile çalıştığınızı kanıtlar. 

## Özet
Tebrikler, Spring ile bir RESTful web servisi geliştirdiniz. 
