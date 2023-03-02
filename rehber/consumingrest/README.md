# Giriş
Bu rehberde Spring ile bir web servisinin nasıl kullanılacağı(consume edileceği) anlatılmaktadır. 

## Genel Bilgiler
http://localhost:8080/api/random adresinde rastgele bir alıntı söz almak için Spring'in `RestTemplate`'ini kullanan bir uygulama oluşturulacaktır.

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
- `git clone https://github.com/spring-guides/gs-consuming-rest.git` komutuyla repo'yu bilgisayarınıza klonlayın.
- cd komutu ile `gs-consuming-rest/initial` dizinine geçin.
- Eğer her şey yolundaysa, [REST kaynağından veri çekmek](#rest-kaynağından-veri-çekmek) başlığına geçiniz.

## Spring initializr ile başlamak
>⚠️ Eğer repo üzerinden devam edecekseniz, bu başlığı atlayabilirsiniz.

Bir Spring projesi başlatmak Spring Initializr online aracını kullanabilirsiniz. Bu araç projeniz için bir zip dosyası üretecektir. Aşağıdaki adımları izleyebilirsiniz:
- `https://start.spring.io` adresine gidiniz. Bu araç bütün gereksinimleri projenize ekleyebilir.
- Gradle veya Maven ile devam edebilirsiniz. Rehberde, dil olarak Java'nın seçildiği kabul edilecektir.
- **Dependencies** sekmesinden **Spring Web** seçiniz.
- **Generate** tuşuna tıklayınız.
- ZIP dosyasını indirin ve kaydedin.

## REST kaynağından veri çekmek
Rehberi tamamladığınızda, RESTful servisini kullanabilen basit bir uygulamayı da tamamlamış olursunuz. Başlamadan önce tüketecek bir REST kaynağına ihtiyacınız olacaktır. Bunun için oluşturulmuş bir servisi https://github.com/spring-guides/quoters'ten klonlayabilirsiniz. Bu uygulamayı başlattığınızda http://localhost:8080/api/random adresini dinleyen bir servis başlatmış olursunuz. Bu adresten servisin üreteceği rastgele alıntı sözleri elde edebilirsiniz. Aynı zamanda  http://localhost:8080/api/ adresinden mevcut bütün alıntı sözleri inceleyebilirsiniz. Son bir kullanım çeşidi olarak, http://localhost:8080/api/1 şeklinde de id'si 1 olan alıntı sözü inceleyebilirsiniz. Örnek bir cevap aşağıdaki şekilde olabilir:
```json
{
   type: "success",
   value: {
      id: 10,
      quote: "Really loving Spring Boot, makes stand alone Spring apps easy."
   }
}
```
Bir REST servisini tarayıcıdan ya da `curl` aracı ile kullanmaktan daha faydalı yolu programatik olarak kullanmaktır. Bu işi mümkün kılabilmek için Spring `RestTemplate` sınıfını sunar. `RestTemplate`, servislerle iletişimi oldukça kolaylaştırır. Dahası, dönen cevabı oluşturduğınız bir sınıfa bile gömebilir.

Dönecek cevabı saklayabilmek için bir _domain_ sınıfı oluşturulmalıdır, bu iş için `Quote` adında bir record sınıfı oluşturulabilir.
```Java
package com.example.consumingrest;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public record Quote(String type, Value value) { }
```
Bu Java record sınıfına `@JsonIgnoreProperties` annotation'u iliştirilmiştir. Amacı, dönen cevap ile sınıfın özellikleri arasında uymayan özelliklerin olması halinde uyumsuz özellikleri ihmal etmesidir.

Dönen veriyi bir objeye direkt olarak gömebilmek için dönen verideki key değeriyle bu değeri tutacak Java değişkeninin isimlerinin aynı olması gereklidir. Aksi durumda `@JsonProperty` annotation ile gerekli düzenleme yapılabilir fakat bu örnekte gerekli olmadığı için kullanılmayacaktır.

Dönen cevap içindeki alıntıyı tutabilmek için bir sınıf daha yaratılmalıdır. `Value` adında bir sınıf yaratılabilir.

```Java
package com.example.consumingrest;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public record Value(Long id, String quote) {}
```

## Uygulamayı tamamlamak
Initializr, `main()` metoduna sahip bir sınıf yaratır. Bu sınıf aşağıdaki gibi gözükmektedir:
```Java
package com.example.consumingrest;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ConsumingRestApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConsumingRestApplication.class, args);
	}
}
```
RESTful kaynağından alıntılar göstermesini sağlamak için `ConsumingRestApplication` sınıfına birkaç şey daha eklemeniz gerekiyor. Eklemeniz gerekenler:
- Bir logger, log gönderebilmek için
- Bir `RestTemplate`, gelen veriyi işlemek için Jackson JSON işlme kütüphanesini kullanır. 
- `CommandLineRunner`, başlangıçta `RestTemplate`'i çalıştırmak için kullanılır. 

Bütün bunları eklediğimizda `ConsumingRestApplication` aşağıdaki gibi görünmelidir:
```Java
package com.example.consumingrest;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
public class ConsumingRestApplication {

	private static final Logger log = LoggerFactory.getLogger(ConsumingRestApplication.class);

	public static void main(String[] args) {
		SpringApplication.run(ConsumingRestApplication.class, args);
	}

	@Bean
	public RestTemplate restTemplate(RestTemplateBuilder builder) {
		return builder.build();
	}

	@Bean
	public CommandLineRunner run(RestTemplate restTemplate) throws Exception {
		return args -> {
			Quote quote = restTemplate.getForObject(
					"http://localhost:8080/api/random", Quote.class);
			log.info(quote.toString());
		};
	}
}
```
## Uygulamayı çalıştırmak
Geliştirdiğiniz yazılımı Maven ya da Gradle ile komut satırı üzerinden çalıştırabilirsiniz. Bunun yanında tüm gerekli sınıflar ve gereksinimleri içinde barındıran bir JAR dosyası da build edebilirsiniz. Çalıştırılabilir jar dosyasına sahip olmak, yazılım geliştirme döngüsü içindeki birçok adımı kolay hale getirir.

Eğer Gradle kullanıyorsanız, uygulamanızı `./gradlew bootRun` komutu ile çalıştırabilirsiniz. JAR dosyası elde etmek için ise aşağıdaki komutu çalıştırabilirsiniz:
```bash
java -jar build/libs/consuming-rest-0.1.0.jar
```
Eğer Maven kullanıyorsanız, `./mvnw spring-boot:run` ile uygulamanızı çalıştırabilirsiniz. JAR dosyası elde etmek için aşağıdaki komutu kullanabilirsiniz:
```bash
java -jar target/libs/consuming-rest-0.1.0.jar
```
Son olarak uygulamanın çalışacağı varsayılan port'u değiştirmek gereklidir çünkü tüketilecek olan servis de 8080 portunda çalışmaktadır. Bu yüzden `main/resources/application.properties` içine `server.port=8090` satırını ekleyebilirsiniz. 

JAR dosyaları yerine WAR dosyası olarak da çıktı alabilirsiniz. 

## Servisi test etmek
Yukarıdaki kod çalışmaya başlandığı zaman terminal üzerinde aşağıdaki gibi bir çıktı görmelisiniz:
```log
2019-08-22 14:06:46.506  INFO 42940 --- [           main] c.e.c.ConsumingRestApplication           : Quote{type='success', value=Value{id=1, quote='Working with Spring Boot is like pair-programming with the Spring developers.'}}
```
## Özet
Tebrikler! Spring Boot'u kullanarak basit bir REST istemcisi geliştirdiniz.