---
title: "Stimulus ile Panoya Kopyalama"
tags: [Stimulus]
layout: post
permalink: 2023-09-30-clipboard-controller-with-stimulus.html
lang: tr
---

### Bu yazıda Stimulus'un temel bazı özelliklerinden bahsederken, Stimulus ile Rails uygulamasına 'Panoya Kopyalama' özelliğinin nasıl eklenebileceğinden bahsedilecek.

Örnek olarak view'ımızda şarkı sözleri olsun.

<script src="https://gist.github.com/safakferhatkaya/210fbcc158f275c2662888489d287062.js?file=_lyricss.html.erb"></script>

Bu HTML kodunun çıktısı şu şekilde olacaktır.

<img src="/assets/images/clipboard-lyrics-html-output.png" loading="lazy" alt="Html kodu ciktiis" width="400"/>

Amacımız kullanıcı kopyalama ikonuna tıklandığında şarkı sözlerinin kullanıcının panosuna (clipboard'a) kopyalanmasını sağlamak.

İkon'a tıklandığında şarkı sözlerinin panoya kopyalanmasını sağlamak için gerekli adımlar şu şekilde sıralanabilir:
1. Stimulus'ta `clipboard_controller.js` oluşturulmalı.
2. Kopyalanacak içeriğin, oluşturulan controllera referans (target) edilmesi sağlanmalı.
3. İkon'un `data attribute`'una ikona tıklandığında çalışması üzere, `data-action` attribute'unda `copy` adında bir JS methodu tanımlanmalı.
4. Kopyalanılacak içeriğin (şarkı sözleri) `clipboard controller`'da referans (target) edilmesi sağlanmalı.
5. Kopyalama işleminin başarılı olduğunu kullanıcıya bildirebilmek (feedback) amacıyla; ikonun konumunda bir süreliğine 'Kopyalandı.' yazmalı ve ikon eski haline dönmeli.

HTML tarafında gerekli değişiklikleri uyguladığımızda, kod şu şekilde görünecektir.

<script src="https://gist.github.com/safakferhatkaya/210fbcc158f275c2662888489d287062.js?file=_lyrics.html.erb"></script>

1. Tüm içeriği saran div (wrapper), `data-controller` özelliği ile `clipboard_controller.js`'e bağlanması sağlandı. Böylece Stimulus controller'ın scope'u tanımlanmış oldu[^1].
2. İkon buton ile wrap edildi, Butona tıklandığında JS controller'da oluşturulacak `copy` methodunun çalışmasını sağlandı.
3. Butonun varsayılan davranışı tıklama (click) olduğu için, yalnızca action'i yazmamız yetecektir. jQuery'deki gibi `click->clipboard#copy` şeklinde bir syntax'a gerek duyulmamaktadır[^2].
4. 2 numaralı hedefi (şarkı sözlerini controller'a target olarak tanımlamak) gerçekleştirmek için şarkı sozlerinin oldugu div'e `text-container` target'i verildi.
5. İkonu wrap eden butona `clipboard-button` target'i tanimlanarak 5 numaralı hedefi gerçeklestirmek için target hazırlandı.
6. 5 numaralı hedefini gerçekleştirmek için, clipboard controller'a `notify-message-value` gönderildi, bu sayede lokalize edilmiş bir mesaj ile 'Kopyalamanın başarılı olduğunu' kullanıcıya bildirilebilecek.

Şimdi, Stimulus tarafında oluşacak targetlarımız ve methodlarımız belli, bunları oluşturabiliriz.

Target'lar temelde querySelector'ler ile çalışan, ilgili tanımlı elementleri bulup erişmemizi, üzerinde işlem yapmamızı kolaylaştıran bir yöntemdir. Stimulus, controller'ın scope'undaki elemanları `queryElement` methodu aracılığıyla data attributelar üzerinden bulur ve kullanabilmemize olanak sağlamaktadır[^3].


```javascript
// targets

  static targets = ['textContainer', 'clipboardButton']
```

Value tanımlaması esnasında value'nun tipini ve varsayılan değerini vererek daha tutarlı bir çalışma şekli sağlayabiliriz. Örneğin, lokalizasyon tanımlanmadıysa varsayılan olarak İngilizce'yi kullanarak kullanıcıya bildirim yapılması sağlanabilir.


```javascript
// values

  static values = {
    notifyMessage: {
            type:    String,
            default: 'Copied!' // value if not defined as data attr
        }
  }
```

Yukarıda dikkat edilmesi gereken bir diğer husus, HTML'de kebab-case olarak tanımlanan `notify-message` değişkeninin Stimulus'ta camelCase'e dönüşmesidir. Naming convention'a dikkat edilmesi gerekmektedir.[^4]

Methodlar:


Öncelikle `copy` methodumuzla başlayalım ve bu method'un clipboard ikonuna tıklandığında çalışacağını hatırlayalım. Targetlara `this.[targetName]Target` şeklinde erişilmektedir ve bu methodda `textContainerTarget`'ın `innerText`'i clipboard'a yazılmıştır (kopyalanmıştır).


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
Şu an aslında controllerda pnaoya kopyalama işlemi yapılabilmektedir. Fakat durumun başarılı olduğunu kullanıcıya ifade etmek için bazı eklemeler yapmamız gerekiyor.. `succesMessage`'ı henüz kullanmadık.

Yapacağımız işlem aslında basitçe `notifyUI` methodu aracılığıyla kopyalama ikonunun içeriğini değiştirip, bir süre sonra eski haline döndürülmesidir.

Panoya kopyalama işlemi async bir işlem oldugu için `navigator.clipboard.writeText(text).then(notifyUI)` şeklinde ekrana feedback veren bir method tanımlamamız gerekmektedir.
Bunu sağlamak için `setTimeout` methodunu kullanmak gereklidir.

Bunun için öncelikle state'i (yani eski haline dönüşünü) sağlayabilmek için Stimulus'un callback actionlarindan olan `connect()` methodunu kullanalım ve innerHTML'ın eski halini hafızaya alalım.
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

Özetle: Yukarıdaki kod parçası, kopyalama işlemi başarılı olduğunda kullanıcıya bildirim vermek ve ikonu eski haline döndürmek için kullanılmaktadır. notifyUI methodu, `navigator.clipboard.writeText(text)` işleminin ardından çalışır. İlk olarak, ikonun içeriğini "Copied!" veya `successMessageValue` ile değiştirmektedir, böylece kullanıcıya başarılı bir kopyalama işlemi olduğunu bildirir. Ardından, `setTimeout` kullanarak 2 saniye sonra ikonun orijinal halini geri almasını sağlamaktadır.

<img src="/assets/images/notifyUI-method.gif" loading="lazy" alt="NotifyUI methodunun çalışma şekli" width="400"/>

Elimizdeki tüm bu methodları birleştirdiğimizde `clipboard_controller.js` dosyasının son hali aşağıdaki gibi olacaktır.

<script src="https://gist.github.com/safakferhatkaya/210fbcc158f275c2662888489d287062.js?file=clipboard_controller.js"></script>

Bu yazıda 'copy to clipboard' fonksiyonalitesi sağlamak için ilgili controller'ın oluşturulması üzerinden temel Stimulus yöntemleri üzerine konuştuk.
<br>

-----------
<br>
Faydalı kaynaklar:


[^1]: [Stimulus'ta Scope Tanımı](https://github.com/hotwired/stimulus/blob/8cbca6db3b1b2ddb384deb3dd98397d3609d25a0/src/core/controller.ts#L48)
[^2]: [Stimulus'ta varsayılan actionlar](https://stimulus.hotwired.dev/reference/actions#event-shorthand)
[^3]: [Stimulus queryElements methodu](https://github.com/hotwired/stimulus/blob/8cbca6db3b1b2ddb384deb3dd98397d3609d25a0/src/core/scope.ts#L43)
[^4]: [Naming conventions](https://gist.github.com/sadeghbarati/4650e4c4e2f25b79a60d937e15cc7665#naming-conventions)
[^5]: [Stimulus Lifecycle Callbacks](https://stimulus.hotwired.dev/reference/lifecycle-callbacks)
