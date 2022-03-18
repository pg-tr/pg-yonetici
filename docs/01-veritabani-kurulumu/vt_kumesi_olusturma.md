---
title: "Veritabanı Kümesi Oluşturma"
layout: default
parent: Veritabanı Kurulumu
nav_order: 3
---

## Veritabanı Kümesi Oluşturma

Veritabı kümesi oluşturma işlemine ilk olarak disk üzerinde bir veritabanı depolama alanı oluşturularak başlanır. PostgreSQL'de buna veritabanı kümesi ( database cluster ) denilirken SQL standardı, katalog kümesi terimini kullanır. Veritabanı kümesi, çalışan bir veritabanı sunucusunun tek bir instance'ı tarafından yönetilen veritabanları koleksiyonudur. Veritabanı kümesi başlatıldıktan sonra, yardımcı programlar, kullanıcılar ve üçüncü parti uygulamalar tarafından kullanılmak üzere varsayılan `postgres` isimli veritabanı beraberinde gelir. Başlatma sırasında her kümede oluşturulan bir başka veritabanına da `template1` veritabanıdır. İsminden de anlaşılacağı üzere `template1` oluşturulacak veritabanları için şablon olarak kullanılacaktır, operasyonel işler için kullanılmamalıdır.

Dosya sistemi açısından bir veritabanı kümesi tüm verilerin depolanacağı tek bir dizindir. Buna veri dizini ( *data directory* ) veya veri alanı denilir. Verilerin nerede depolayacağı tamamen size bağlıdır. Varsayılan olmamakla birlikte `/usr/local/pgsql/data` ve `/var/lib/pgsql/data` konumları popülerdir. Veri dizini ve küme alt yapısı, PostgreSQL ile gelen [`initdb`](https://www.postgresql.org/docs/current/app-initdb.html) programı kullanılarak oluşturulur. (initialize)

{% include important.html content="Önceden paketlenmiş bir PostgreSQL sürümünü kullanıyorsanız, bu sürüm veri dizininin nereye konumlandırılacağına dair belirli bir kurala sahip olabilir, veri dizinini oluşturmak için bir betik de sağlayabilir. Bu durumda `initdb`'yi doğrudan çalıştırmak yerine sağlanan betiği kullanmalısınız. Ayrıntılar için paket düzeyindeki belgelere bakınız." %}

Veritabanı kümesini manuel olarak başlatmak için initdb'yi veritabanı kümesi konumunu `-D` parametresiyle belirterek çalıştırın,

```bash
$ initdb -D /usr/local/pgsql/data
```

{% include callout.html content=" Bu komutu önceki bölümde anlatılan PostgreSQL kullanıcısı hesabında çalıştırmalınız. Ayrıca, `-D` parametresine alternatif olarak `PGDATA` ortam değişkeni ayarlanabilir." type="info" %}

Alternatif olarak, initdb'yi `pg_ctl` programını kullanarak şu şekilde çalıştırılabilir:

```bash
$ pg_ctl -D /usr/local/pgsql/data initdb
```

Verilen dizin yoksa, initdb belirtilen dizini yaratmaya çalışır. Fakat, initdb'nin üst dizine yazma izni bulunmadığında bu işlem başarısız olacaktır. PostgreSQL kullanıcısının sadece veri dizinine değil aynı zamanda onun ana dizinine de sahip olması gerekir. İstenen üst dizin yoksa, root ayrıcalıklarını kullanarak önce bunu oluşturmalı ve gerekli yetkileri ayarlamalısınız. Dolayısıyla süreç şu şekilde olur:

```bash
root# mkdir /usr/local/pgsql
root# chown postgres /usr/local/pgsql
root# su postgres
postgres$ initdb -D /usr/local/pgsql/data
```

{% include callout.html content=" Mevcut bir veri dizinin bulunması ve dosyalar içermesi durumunda initdb çalışmaz. Bu, yanlışlıkla varolan bir kurulumun üzerine yazılmasını önlemek için alınan bir önlemdir." type="info" %}

Veri dizini, veri tabanında depolanan tüm verileri içerdiğinden yetkisiz erişime karşı korunmalıdır. `initdb`, PostgreSQL kullanıcısı dışındaki herkesin erişim izinlerini iptal eder. İsteğe bağlı olarak grup erişim izinleri iptal edilmeyebilir. Grup erişimi etkinleştirildiğinde salt okunurdur. Böylece, küme sahibiyle aynı gruptaki ayrıcalıksız bir kullanıcı küme verilerinin yedeğini alabilir veya yalnızca okuma erişimi gerektiren işlemleri gerçekleştirebilir.

Mevcut kümede grup erişimini etkinleştirilme veya devre dışı bırakma işlemi, kümenin kapatılmasını ve PostgreSQL'i yeniden başlatmadan önce tüm dizinlerde ve dosyalarda uygun erişim modunun ayarlanmasını gerektirir. Aksi takdirde, veri dizininde karışık modlar bulunabilir. Sadece küme sahibi erişimine izin verirken gerekli modlar dizinlerde `0700` ve dosyalarda `0600` şeklindedir. Kümelerin grup tarafından okunmasına izin vermek için uygun modlar, dizinlerde `0750` ve dosyalarda `0640`'tır.

Dizin içeriklerini güvene almış olsak da, varsayılan istemci kimlik doğrulama kurulumu yerel kullanıcıların veritabanına bağlanmasına, hatta veritabanı süper kullanıcısı olmasına izin verir. Yerel kullanıcılara güvenmiyorsanız, veritabanı süper kullanıcısına parola atamak için initdb'nin `-W`, `--pwprompt` veya `--pwfile` parametreleri kullanılır. Ayrıca, varsayılan `trust` kimlik doğrulama modunu `-A md5` veya `-A password` ile değiştirin veya `pg_hba.conf` dosyasını üzerinden belirleyin. Bağlantı kısıtlamak için diğer yaklaşımlar ise `peer` kimlik doğrulaması ve dosya sistemi izinleri kullanmaktır.

`initdb` veritabanı kümesi için varsayılan yerel (locale) ayarı da başlatır. Yerel ayarları ortamdan alır ve başlatılmış veritabanına uygular. Yeni veritabanı oluşturulurken şablon veritabanlarındaki sıralama düzeni kullanılır. Bu düzen drop ve recreate edilmeden değiştirilemez. Farklı sıralama düzenine sahip veritabanları oluşturmak mümkündür.

`initdb` ayrıca veritabanı kümesi için varsayılan karakter seti kodlamasını da ayarlar. Bu, yerel ayarlarla eşleşecek şekilde seçilmelidir. Ayrıntılar için bkz. [Karakter Seti Desteği](https://www.postgresql.org/docs/current/multibyte.html)

Karakter seti sıralaması için işletim sisteminin collation kütüphanesi kullanır. Bu, indekslerde depolanan anahtarların sırasını kontrol eder. Bu nedenle bir küme; snapshot restore, binary streaming replikasyon, farklı işletim sistemi veya işletim sistemi yükseltmesi yoluyla uyumsuz bir collation kütüphanesine geçemez.

## İkincil Dosya Sistemlerinin Kullanımı

Çoğu kurulum, veritabanı kümelerini, makinenin "root" birimi dışındaki dosya sistemlerinde oluşturur. PostgreSQL kullanıcısının sahip olduğu mount-point dizini içinde oluşturacağınız dizinde veri dizinini oluşturulmalıdır. Bu şekilde `pg_upgrade` gibi işlemlerde izin sorunları önlenir.

## Dosya Sistemleri

POSIX semantiğine sahip her dosya sistemi PostgreSQL için kullanılabilir. Kullanıcılar; satıcı desteği, performans ve aşinalık gibi çeşitli nedenlerden farklı dosya sistemlerini tercih edebilirler. Diğer tüm şeyler eşitken, yalnızca dosya sistemlerini değiştirerek veya dosya sistemi konfigürasyon değişiklikleri yaparak büyük performans veya davranış değişiklikleri beklenmemelidir.

**Kaynak:**

[1]. [PostgreSQL Documentation](https://www.postgresql.org/docs/current/creating-cluster.html)

{% include links.html %}
