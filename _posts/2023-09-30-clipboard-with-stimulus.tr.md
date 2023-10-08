---
title: "Stimulus ile Copy to Clipboard"
tags: [Stimulus]
layout: post
permalink: 2023-09-30-clipboard-controller-with-stimulus.html
lang: tr
---

### Bu yazıda Stimulus'un temel bazı özelliklerinden bahsederken, Stimulus ile Rails uygulamasına Copy to Clipboard özelliğinin nasıl eklenebileceğinden bahsedeceğim.

Stimulus, kendisini kısaca mütevazi JavaScript framework'ü olarak tanımlıyor.
View'ın tamamını kontrol etmenizi sağlamaktan ziyade, interaktif DOM aksiyonlarını özel tanımlayıcıları sayesinde kolaylaştıran/karmaşıklığı azaltan bir JS frameworküdür.

Örnek olarak view'ımızda şarkı sözleri olsun.

<script src="https://gist.github.com/safakferhatkaya/210fbcc158f275c2662888489d287062.js?file=_lyricss.html.erb"></script>

Bu HTML kodunun çıktısı şu şekilde olacaktır.

<img src="/assets/images/clipboard-lyrics-html-output.png" loading="lazy" alt="Html kodu ciktiis" width="400"/>

Amacımız kullanıcı kopyalama ikonuna tıklandığında şarkı sözlerinin kullanıcının panosuna (clipboard) kopyalanmasıdır.

Bunu sağlamak için gerekli adımlar şu şekilde sıralanabilir:
1. Stimulus'ta `clipboard_controller.js` oluşturulmalı.
2. Kopyalanacak iceriği, oluşturulan controllerda referans (target) edilmesi sağlanmalı.
3. Icon'a tıklandığında şarkı sözlerinin panoya kopyalanmasını sağlayan method tanımlanmalı.
    <br>Bunun için:
    1. Icon'un `data attribute`'una icona tiklandiginda çalışması üzere, `data-action` attribute'unda `copy` adında bir JS actionı tanımlanmalı.
    2. Kopyalanılacak iceriğin (şarkı sözleri) `clipboard controller`'da referans (target) edilmesi sağlanmalı.
    3. Kopyalama işleminin başarılı olduğunu kullanıcıya feedback verebilmek amacıyla; ikonun yerinde bir süre 'Kopyalandı.' yazmalı ve icon eski haline dönmeli.

HTML tarafında gerekli değişiklikleri uyguladığımızda, kod şu şekilde görünecektir.

<script src="https://gist.github.com/safakferhatkaya/210fbcc158f275c2662888489d287062.js?file=_lyrics.html.erb"></script>

1. Tüm içeriği saran div (wrapper), `data-controller` özelliği ile `clipboard_controller.js`'e bağlanması sağlandı. Böylece Stimulus controller'ın scope'unu tanımlanmış olduk[^1].
2. Icon buton ile wrap edildi, Butona tıklandığında JS controller'da oluşturulacak `copy` actionının çalışmasını sağlayacak.
3. Butonun varsayılan davranışı tıklama (click) olduğu için, yalnızca action'i yazmamız yetecektir. jQuery'deki gibi `click->clipboard#copy` şeklinde bir syntax'a gerek duyulmamaktadır[^2].
4. 2 numaralı hedefi (şarkı sözlerini clipboarda kopyalamak) gerçekleştirmek için şarkı sozlerinin oldugu div'e `text-container` target'i verildi.
5. Iconu wrap eden butona `clipboard-button` target'i tanimlanarak 3.3 hedefini gerçeklestirmek için target hazırlandı.
6. 3.3 hedefini gerçekleştirmek için, clipboard controller'a `notify-message-value` gönderildi, bu sayede lokalize edilmiş bir mesaj ile 'Kopyalamanın başarılı olduğunu' kullanıcıya bildirebileceğiz.

Şimdi, Stimulus tarafında oluşacak targetlarımız ve methodlarımız belli, bunları oluşturabiliriz.

Target'lar temelde querySelector'ler ile çalışan, ilgili tanımlı elementleri bulup erişmemizi, üzerinde işlem yapmamızı kolaylaştıran bir yöntemdir. Stimulus, controller'ın scope'undaki elemanları `queryElement` methodu aracılığıyla data attributelar üzerinden bulur ve kullanmamızı sağlar[^3].


```javascript
// targets

  static targets = ['textContainer', 'clipboardButton']
```

Value tanımlaması esnasında value'nun tipini ve varsayılan değerini vererek daha tutarlı bir çalışma şekli sağlayabiliriz. Örneğin, lokalizasyon tanımlanmadıysa varsayılan olarak İngilizce'yi kullanarak kullanıcıya bildirim yapılmasını sağlarız.



Burada dikkat edilmesi gereken bir diğer husus, HTML'de kebab-case olarak tanımlanan `notify-message` değişkeninin Stimulus'ta camelCase'e dönüşmesidir. Naming convention'a dikkat edilmesi gerekmektedir.[^4]
```javascript
// values

  static values = {
    notifyMessage: {
            type:    String,
            default: 'Copied!' // value if not defined as data attr
        }
  }
```
Öncelikle `copy` methodumuzla başlayalım ve bu method'un clipboard iconuna tıklandığında çalışacağını hatırlayalım. Targetlara `this.[targetName]Target` şeklinde erişilmektedir ve bu methodda `textContainerTarget`'ın `innerText`'i clipboard'a yazılmıştır (kopyalanmıştır).


```javascript
  copy(event) {
    event.preventDefault()
    event.stopPropagation()

    const text = this.textContainerTarget.innerText
    navigator.clipboard.writeText(text)
  }
```

Elimizdekileri birleştirirsek:

```javascript
import { Controller } from "@hotwired/stimulus";

export default class extends Controller {
   static values = {
      succesMessage: {
              type:    String,
              default: 'Copied!'
          }
   }

  static targets = ['textContainer', 'clipboardButton']

  copy(event) {
    event.preventDefault()
    event.stopPropagation()

    const text = this.textContainerTarget.innerText
    navigator.clipboard.writeText(text)
  }
}

```
Şu an aslında clipboard'a kopyalama işlemi tamamlandı. Fakat durumun başarılı olduğunu kullanıcıya ifade etmek için bazı eklemeler yapmamız gerekiyor.. `succesMessage`'ı henüz kullanmadık.

Clipboard'a kopyalama işlemi async bir işlem oldugu için `navigator.clipboard.writeText(text).then(notifyUI)` şeklinde ekrana feedback veren bir method tanımlamamız gerekmektedir.

Yapacağımız işlem aslında basitçe `notifyUI` methodu aracılığıyla clipboard iconunun içeriğini değiştirip, bir süre sonra eski haline dondürmek.
Bunu sağlamak için `setTimeout` methodu makul görünmektedir.
Bunun için oncelikle state'i yani eski haline dönüşünü, sağlayabilmek için Stimulus'un callback actionlarindan olan `connect()` methodunu kullanalım ve innerHTML'ın eski halini hafızaya alalım.
`connect` methodu controller DOM'a her bağlandığında çalışmaktadır. [^5]

```javascript
  connect() {
    this.clipboardButtonInitialInnerHTML = this.clipboardButtonTarget.innerHTML
  }
```

`notifyUI` methodunu ekleyelim.

```javascript
  notifyUI() {
    this.clipboardButtonTarget.innerHTML = this.notifyMessageValue
    setTimeout(() => {
      this.clipboardButtonTarget.innerHTML = this.clipboardButtonInitialInnerHTML
    }, 2000)
  }
```

Yukarıdaki kod, kopyalama işlemi başarılı olduğunda kullanıcıya bildirim vermek ve ikonu eski haline döndürmek için kullanılır. notifyUI methodu, `navigator.clipboard.writeText(text)` işleminin ardından çalışır. İlk olarak, iconun içeriğini "Copied!" veya successMessageValue ile değiştirir, böylece kullanıcıya başarılı bir kopyalama işlemi olduğunu bildirir. Daha sonra, setTimeout kullanarak 2 saniye sonra iconun orijinal içeriğini geri almasını sağlar.

<img src="/assets/images/notifyUI-method.gif" loading="lazy" alt="NotifyUI methodunun çalışma şekli" width="400"/>

Elimizdeki tüm bu methodları birleştirdigimizde `clipboard_controller.js` dosyasının son hali aşağıdaki gibi olacaktır.

<script src="https://gist.github.com/safakferhatkaya/210fbcc158f275c2662888489d287062.js?file=clipboard_controller.js"></script>

Bu yazıda 'copy to clipboard' fonksiyonalitesi sağlamak için ilgili controller'ın oluşturulması üzerinden temel Stimulus yöntemleri üzerine konuştuk. Umarım faydalı bir yazı olmuştur.
-----------
<br>

Faydalı kaynaklar:


[^1]: [Stimulus'ta Scope Tanımı](https://github.com/hotwired/stimulus/blob/8cbca6db3b1b2ddb384deb3dd98397d3609d25a0/src/core/controller.ts#L48)
[^2]: [Stimulus'ta varsayılan actionlar](https://stimulus.hotwired.dev/reference/actions#event-shorthand)
[^3]: [Stimulus queryElements methodu](https://github.com/hotwired/stimulus/blob/8cbca6db3b1b2ddb384deb3dd98397d3609d25a0/src/core/scope.ts#L43)
[^4]: [Naming conventions](https://gist.github.com/sadeghbarati/4650e4c4e2f25b79a60d937e15cc7665#naming-conventions)
[^5]: [Stimulus Lifecycle Callbacks](https://stimulus.hotwired.dev/reference/lifecycle-callbacks)
