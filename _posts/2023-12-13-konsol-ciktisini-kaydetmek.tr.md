---
title: "Terminal Çıktılarını Dosyaya Kaydetme, &>"
tags: [TIL, Notes, Bash]
layout: post
permalink: 2023-12-13-terminal-ciktisini-kaydetmek.html
lang: tr
---

Örneğin, `bin/rails test` komutunu koşturduğunuzda terminalinize sığmayacak kadar çok hata fırlatan bir test ortamınız var, bu sebepten çalıştırdığınız komutun çıktısını ne tam inceleyebiliyorsunuz ne de saklayabiliyorsunuz,
veya takım arkadaşınızla bir log'u derli toplu şekilde paylaşmak istiyorsunuz..
<br>
<br>
Bash'ın `&>` operatörünü kullanarak çıktıyı ayrı bir dosyaya yazabilirsiniz.

```bash
bin/rails test:all &> output.log
```

Bu satırda, `bin/rails test:all` komutu çalıştır ve çıktısını output.log dosyasını oluşturup, bu dosyaya yazar.

### örnek, git status komutunun çıktısını status.log dosyasına yazmak:

<img src="/assets/images/log-run.png" alt="output redirecting kullanımı"/>
<img src="/assets/images/status-log.png" alt="output redirecting ciktisi"/>
<br><br>

#### Özet: &> operatörü, terminaldeki stdout (standart çıktı) ve stderr (standart hata) çiktilarini bir dosyaya yönlendirmenizi/yazmanızı sağlayan bash komutudur.


<br>

-----------
<br>
Faydalı kaynaklar:


[Output redirecting](https://www.gnu.org/software/bash/manual/html_node/Redirections.html#Redirecting-Output)

