---
title: "Asset dosyalarının sonundaki bu şeyler de ne?"
tags: [TIL, Rails]
layout: post
permalink: 2023-09-26-rails-assets-fingerprinting.html
lang: tr
---


## Bu benim en sevdiğim kedi fotoğraflarından birisi
![My favorite cat](/assets/images/eepy-cat.jpeg)

### Bu fotoğrafı Rails'te production'a gecirdigimde fotoğraf URL'ininin şöyle olduğunu farkettim
<br>

```
my-site.com/assets/cuties/eepy-cat-99b7082082fe58d58fe828581d4a47aedf277e1cf0e47a660529d9ca923feece.jpeg
```
<br>

### Asset dosyasının URL'inin sonundaki bu kaos ve rastgele stringler/integerlar nedir?

Bu şeyler `fingerprints` olarak bilinir. Rails bu benzersiz tanımlayıcıları CSS, JS ve image assetlerine uygular.

Rails'in (veya başka bir framework'un) bu methodu kullanmasının temel sebebi, asset dosyalarını cache'leyerek hızlı daha web sayfaları yaratmaktır.

Bilindiği üzere Rails'te Client'tan bir sayfa talep ettiğinizde tarayıcı sunucuya giderek ilgili sayfanın kaynaklarını alır.
Ancak, HTTP/Caching sayesinde her seferinde sayfanın tamamını getirmek yerine tarayıcı ile sunucu arasında çalışır. Parmak izinin bize yardımcı olduğu yer burasıdır.

Tarayıcı, daha önce belirli varlıkları önbelleğe aldıysa, bu assetlerin sürümünü/güncelliğini parmak izleri (fingerprints) aracılığıyla kontrol eder. Sunucudaki mevcut veriler ile tarayıcıdaki önbelleğe alınmış dosyanın kimliğini de parmak izleri ile karşılaştırır ve eğer dosya değiştirilmediyse o asset için yeni bir talepte bulunmaz.

Böylece tarayıcı bant genişliğini daha verimli kullanır.
<br>

### I realized the importance of fingerprinting when trying to pass an image location to the JS file.
Başlangıçta görüntü yolunu JS'ye `data-icon-url-value=assets/cuties/eepy-cat.png` olarak aktardım. Evet, yerel makinemde çalışıyordu :) ancak üretimde başarısız oldu.
Parmak izinin önemini fark ettim ve araştırma sonuçlarını paylaşmak istedim.

Bu yüzden `image_url('cuties/eepy-cat')` kullanmaya devam edin
Parmak izi almak/ön belleğe almak ve varlıklarınıza doğru şekilde erişmek için `src='assets/cuties/eepy-cat'` yerine.
<br>

-----------
<br>

Faydalı kaynaklar:

[Fingerprinting Images to Improve Page Load Speed](https://docs.imgix.com/best-practices/fingerprinting-images-to-improve-page-load-speed)

[What is Fingerprinting and Why Should I Care?](https://guides.rubyonrails.org/asset_pipeline.html#what-is-fingerprinting-and-why-should-i-care-questionmark)
