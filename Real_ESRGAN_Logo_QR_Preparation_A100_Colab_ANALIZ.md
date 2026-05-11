# Real-ESRGAN Logo QR Preparation A100 Colab Notebook Analizi

Bu dokuman, `Real_ESRGAN_Logo_QR_Preparation_A100_Colab.ipynb` dosyasini hic bilmeyen biri icin bastan sona aciklar.

Notebook'un amaci, bir logo dosyasini Google Colab ortaminda yukleyip Real-ESRGAN ile buyutmek, kare hale getirmek, arka plan ve keskinlik ayarlarini uygulamak ve QR fusion / sanatsal QR hazirligi icin temiz PNG ciktilari uretmektir.

## Projenin Amaci

Bu notebook bir "son QR uretici" degildir. Yani icine link yazip okunabilir QR kodu dogrudan uretmez.

Bu notebook'un asil gorevi sudur:

1. Musteri logosunu Colab'a yuklemek.
2. Logonun boyut, seffaflik, kontrast ve keskinlik durumunu analiz etmek.
3. Gerekirse Real-ESRGAN ile logoyu kaliteli sekilde buyutmek.
4. Logoyu QR fusion icin uygun kare PNG formatina hazirlamak.
5. Cikti dosyalarini ZIP olarak indirmeye hazir hale getirmek.

Sonucta elde edilen dosyalar, daha sonra baska bir QR fusion, ControlNet, Stable Diffusion, ComfyUI veya benzeri bir gorsel QR uretim pipeline'inda kullanilabilir.

## Notebook Genel Akisi

Notebook su sirayla calisir:

1. Colab ortaminda Real-ESRGAN reposunu kurar.
2. Kullaniciya logo dosyasi yukletir.
3. Yuklenen logoyu analiz eder.
4. Upscale ve hazirlama ayarlarini alir.
5. Real-ESRGAN ile logoyu buyutur.
6. Buyutulmus logoyu kare PNG hale getirir.
7. Ciktilari ZIP olarak indirir.

Huceleri sirayla calistirmak gerekir. Ornegin 5. hucre, 4. hucrede secilen ayarlara ve 2. hucrede yuklenen logoya ihtiyac duyar.

## Ciktilar Nelerdir?

Notebook sonunda `/content/logo_outputs` klasorunde su dosyalar olusur:

### `logo_upscaled.png`

Real-ESRGAN'dan cikan ham buyutulmus logo dosyasidir.

Bu dosya logonun upscale edilmis halidir. Ancak henuz hedef kare boyuta oturtulmus, arka plan uygulanmis veya QR fusion icin final hale getirilmis olmayabilir.

### `logo_prepared_for_qr.png`

QR fusion icin asil kullanilacak hazir logo dosyasidir.

Bu dosyada:

- Logo hedef kare boyuta getirilir.
- `contain_pad` veya `cover_crop` secimine gore kareye yerlestirilir.
- Arka plan secimi uygulanir.
- Keskinlestirme uygulanir.

Genellikle QR fusion pipeline'ina verilecek ana dosya budur.

### `final_readable_art_qr_PLACEHOLDER.png`

Bu dosya gercek final QR degildir. Hazirlanan logo dosyasinin bir kopyasidir.

Adi, sonraki asamada "okunabilir sanatsal QR burada olusacak" anlaminda bir yer tutucudur. Bu notebook QR kodu uretmedigi icin dosya placeholder olarak kaydedilir.

### `settings_used.txt`

Notebook calisirken kullanilan ayarlari kaydeder.

Ornegin secilen model, hedef boyut, arka plan modu, keskinlestirme miktari, tile size gibi bilgiler burada yer alir. Sonradan ayni sonucu tekrar uretmek icin faydalidir.

### `delivery.zip`

Teslim paketi dosyasidir.

Icinde sunlar bulunur:

- `logo_upscaled.png`
- `logo_prepared_for_qr.png`
- `final_readable_art_qr_PLACEHOLDER.png`
- `settings_used.txt`

## 1. Kurulum / Repo Hazirlama

Bu hucre Colab ortaminda Real-ESRGAN'i calisabilir hale getirir.

Kodun yaptiklari:

- `/content/Real-ESRGAN` klasorunu proje klasoru olarak belirler.
- `USE_FRESH_CLONE = True` ise eski Real-ESRGAN klasorunu siler.
- Real-ESRGAN GitHub reposunu klonlar.
- Gerekli Python paketlerini kurar.
- Notebook'u Real-ESRGAN klasorune tasir.
- CUDA / GPU var mi kontrol eder.
- Kullanilan GPU adini ve VRAM miktarini yazdirir.

### `USE_FRESH_CLONE`

`True` secilirse:

- Colab icindeki eski Real-ESRGAN klasoru silinir.
- Repo bastan temiz sekilde indirilir.
- Daha temiz ve tekrarlanabilir bir kurulum olur.

`False` secilirse:

- Eger repo zaten varsa silinmez.
- Daha onceki dosyalar kalir.
- Hizli olabilir ama eski dosyalardan kaynakli karisiklik olusabilir.

Yeni baslayan biri icin `True` daha guvenlidir.

### Kurulan paketler

Hucre su paketleri kurar:

- `basicsr`: Real-ESRGAN'in dayandigi altyapi.
- `facexlib`: Yuz iyilestirme araclari icin.
- `gfpgan`: Yuz restorasyon modelleri icin.
- `opencv-python`: Goruntu isleme icin.
- `Pillow`: PNG, JPG gibi gorselleri acmak ve kaydetmek icin.
- `tqdm`: Ilerleme cubuklari icin.

### `torchvision functional_tensor shim`

Bazi yeni `torchvision` surumlerinde `basicsr` eski bir modul bekler. Bu modul yoksa notebook kucuk bir uyumluluk dosyasi olusturur.

Bu kisim kullanicinin ayarlayacagi bir sey degildir. Hata cikmasin diye eklenmis bir tamir adimidir.

## 2. Logo Upload

Bu hucre kullanicidan logo dosyasi ister.

Desteklenen pratik formatlar:

- PNG
- JPG
- JPEG
- WEBP

Kodun yaptiklari:

- `/content/logo_upload` klasorunu olusturur.
- `/content/logo_work` klasorunu olusturur.
- `/content/logo_outputs` klasorunu olusturur.
- Colab dosya yukleme penceresini acar.
- Yuklenen ilk dosyayi kaydeder.
- Dosyanin yolunu `LOGO_PATH` degiskenine yazar.

### Ne yuklemeliyim?

Logo icin en iyi tercih:

- Mumkunse PNG.
- Mumkunse seffaf arka planli PNG.
- Mumkunse yuksek cozunurluklu dosya.
- Cok kucuk, bulanuk veya agir sikistirilmis JPG dosyalardan kacinmak daha iyidir.

Birden fazla dosya yuklenirse notebook sadece ilk yuklenen dosyayi kullanir.

## 3. Logo Analiz

Bu hucre yuklenen logoyu inceler ve ekrana temel bilgiler basar.

Kodun olctugu seyler:

- Dosya formati.
- Renk modu.
- Goruntu boyutu.
- Seffaflik var mi.
- Beyaz piksel orani.
- Kontrast skoru.
- Keskinlik / kenar skoru.
- Olasilik uyarilari.

### `Format`

Dosyanin turunu gosterir. Ornek:

- `PNG`
- `JPEG`
- `WEBP`

### `Mode`

Gorselin renk yapisini gosterir.

Ornekler:

- `RGB`: Normal renkli gorsel, seffaflik yok.
- `RGBA`: Renk + alpha kanali, yani seffaflik olabilir.
- `L`: Gri tonlu gorsel.

QR fusion icin `RGBA` ve seffaf PNG genellikle daha esnek olur.

### `Boyut`

Logonun piksel boyutudur. Ornek:

`512x512`

Kucuk logolar upscale icin daha cok fayda gorur. Cok kucuk logolarda ise detaylar zaten kayip olabilecegi icin buyutme mucize yaratmaz.

### `Seffaflik`

Logoda anlamli alpha / transparency var mi kontrol edilir.

`var` ise:

- Logo arka plansiz olabilir.
- QR tasariminda daha rahat yerlestirilebilir.

`yok` ise:

- Logo muhtemelen beyaz veya solid bir arka plana sahiptir.
- Notebook son hazirlikta arka plan uygulayarak kare PNG uretebilir.

### `Beyaz piksel orani`

Gorselde neredeyse beyaz olan piksellerin oranidir.

Yuksekse:

- Logo beyaz arka plan uzerinde olabilir.
- Seffaf olmayan bir PNG/JPG olabilir.

### `Kontrast skoru`

Logodaki acik-koyu farkini kabaca olcer.

Dusuk kontrast:

- QR fusion okunabilirligini zorlastirabilir.
- Logo detaylari QR desenine karisabilir.

Yuksek kontrast:

- Sekiller daha net ayrilir.
- QR ile birlestirme daha kontrollu olabilir.

### `Keskinlik/kenar skoru`

Gorseldeki kenar yogunlugunu kabaca olcer.

Dusukse:

- Logo yumusak veya bulaniktir.
- `sharpen_after` ayarini biraz yuksek kullanmak faydali olabilir.

### Uyarilar

Notebook su durumlarda uyarilar verir:

- Logo 512 pikselden kucukse.
- Logo seffaf degilse.
- Beyaz arka plan agirlikli gorunuyorsa.
- Kontrast dusukse.
- Goruntu bulaniksaysa.
- Kucuk detay veya yazi bozulma riski tasiyorsa.

Bu uyarilar hata degildir. Sadece karar vermeye yardimci olur.

## 4. Ayarlar

Bu hucre notebook'un en onemli kontrol panelidir.

Burada sececeginiz ayarlar, Real-ESRGAN'in nasil calisacagini ve final PNG'nin nasil hazirlanacagini belirler.

## `upscale_enabled`

Degerler:

- `True`
- `False`

`True` secilirse:

- Logo Real-ESRGAN ile buyutulur.
- Kalite artisi ve detay toparlama hedeflenir.

`False` secilirse:

- Real-ESRGAN calismaz.
- Yuklenen logo dogrudan sonraki hazirlama adimina verilir.

Ne zaman `True`?

- Logo kucukse.
- Logo bulaniksa.
- Final cikti buyuk olmalisa.
- QR fusion icin daha temiz kaynak isteniyorsa.

Ne zaman `False`?

- Logo zaten cok kaliteli ve buyukse.
- Sadece kare PNG hazirlamak istiyorsaniz.
- Hizi onemsiyorsaniz.

Varsayilan: `True`

## `upscale_model`

Secenekler:

- `realesrgan-x4plus-anime`
- `realesrgan-x4plus`
- `realesrgan-x2plus`
- `realesr-general-x4v3`

Bu ayar hangi Real-ESRGAN modelinin kullanilacagini belirler.

### `realesrgan-x4plus-anime`

Notebook'un varsayilan modelidir.

En uygun oldugu gorseller:

- Logo
- Ikon
- Maskot
- Cizim
- Flat illustration
- Anime / cartoon tarzi gorsel
- Keskin renk bloklari olan tasarimlar

Logo isleri icin genellikle en guvenli secim budur.

### `realesrgan-x4plus`

Daha gercekci gorseller icin uygundur.

En uygun oldugu gorseller:

- Fotograf
- Urun gorseli
- Gercekci poster
- Doku iceren logo
- Fotografik arka planli marka gorseli

Eger logo fotograf gibi gorunuyorsa bu model denenebilir.

### `realesrgan-x2plus`

2 kat buyutme icin uygundur.

Ne zaman kullanilir?

- Logo zaten orta veya buyuk boyuttaysa.
- 4x buyutme fazla yapay gorunuyorsa.
- Daha yumusak ve az agresif bir buyutme isteniyorsa.

### `realesr-general-x4v3`

Genel amacli 4x modeldir.

Avantaji:

- `denoise_strength` ayarini destekler.
- JPEG sikistirma izi, noise veya kirli goruntulerde faydali olabilir.

Ne zaman kullanilir?

- Logo JPG'den geliyorsa.
- Kenarlarda kirlenme varsa.
- Gorselde noise veya artifact gorunuyorsa.

## `upscale_factor`

Secenekler:

- `2`
- `4`

Bu ayar Real-ESRGAN'in gorseli kac kat buyutecegini belirler.

`2` secilirse:

- Gorsel 2 kat buyur.
- Daha hafif islem yapilir.
- Zaten buyuk logolarda mantiklidir.

`4` secilirse:

- Gorsel 4 kat buyur.
- Kucuk logolar icin daha uygundur.
- Varsayilan ve QR hazirligi icin genellikle tercih edilen secimdir.

Not: Model adinda `x4` gecse bile komutta `-s` ile secilen olcek ayrica verilir. En temiz kullanimda model ve faktor uyumlu secilmelidir.

## `target_size`

Secenekler:

- `2048`
- `3072`
- `4096`

Bu ayar final hazir PNG'nin kare boyutunu belirler.

`2048` secilirse:

- Daha kucuk dosya olusur.
- Daha hizli islenir.
- Basit QR fusion denemeleri icin yeterli olabilir.

`3072` secilirse:

- Notebook'un varsayilanidir.
- Kalite ve dosya boyutu arasinda dengeli secimdir.
- A100 GPU icin mantikli bir orta-yuksek kalite ayaridir.

`4096` secilirse:

- Daha buyuk ve detayli cikti verir.
- Dosya boyutu artar.
- Sonraki isleme adimlari daha yavas olabilir.

QR fusion icin genellikle `3072` iyi bir baslangic degeridir.

## `fit_mode`

Secenekler:

- `contain_pad`
- `cover_crop`

Bu ayar logonun kare alana nasil yerlestirilecegini belirler.

### `contain_pad`

Logonun tamami korunur.

Ne olur?

- Logo hedef kare icine sigdirilir.
- Hicbir yeri kesilmez.
- Bos kalan alan arka planla doldurulur.

Ne zaman kullanilir?

- Logonun hic kirpilmeyip tam gorunmesi gerekiyorsa.
- Marka logolarinda guvenli secim isteniyorsa.
- QR fusion oncesi temiz bir kaynak hazirlanacaksa.

Varsayilan ve en guvenli secim: `contain_pad`

### `cover_crop`

Kare alan tamamen doldurulur.

Ne olur?

- Logo kare alani kaplayacak kadar buyutulur.
- Kenarlardan kirpma olabilir.

Ne zaman kullanilir?

- Gorsel kareyi tamamen doldursun isteniyorsa.
- Kenarlardan kesilme sorun degilse.
- Logo degil de poster / desen / arka plan gibi bir gorsel kullaniliyorsa.

Logo islerinde dikkatli kullanilmalidir; marka yazisi veya ikon kenardan kesilebilir.

## `background_mode`

Secenekler:

- `white`
- `keep_transparent`
- `custom_color`

Bu ayar final kare PNG'nin arka planini belirler.

### `white`

Arka plan beyaz olur.

Ne zaman kullanilir?

- QR fusion araclari beyaz zemin bekliyorsa.
- Logo seffaf degilse.
- Daha temiz ve klasik bir kaynak isteniyorsa.

Varsayilan: `white`

### `keep_transparent`

Seffaflik korunur.

Ne zaman kullanilir?

- Sonraki pipeline seffaf PNG ile calisacaksa.
- Logoyu baska bir arka plana oturtacaksaniz.
- Arka plani daha sonra belirlemek istiyorsaniz.

Dikkat: Bazi QR fusion araclari seffaf PNG'yi beklenmedik yorumlayabilir. Bu durumda beyaz veya ozel renk daha guvenli olabilir.

### `custom_color`

Arka plan kullanicinin verdigi renge boyanir.

Bu secimde `custom_background_color` kullanilir.

Ne zaman kullanilir?

- Marka rengine uygun zemin isteniyorsa.
- Beyaz yerine farkli bir duz renk gerekiyorsa.
- QR tasariminin genel renk dili onceden belliyse.

## `custom_background_color`

Ornek:

`#FFFFFF`

Bu ayar sadece `background_mode = "custom_color"` secildiginde anlamlidir.

Format mutlaka `#RRGGBB` olmalidir.

Dogru ornekler:

- `#FFFFFF` beyaz
- `#000000` siyah
- `#FF0000` kirmizi
- `#1A73E8` mavi

Yanlis ornekler:

- `white`
- `fff`
- `#FFF`
- `255,255,255`

Yanlis format girilirse notebook hata verir:

`custom_background_color #RRGGBB formatinda olmali.`

## `denoise_strength`

Aralik:

- `0`
- `1`

Bu ayar sadece `realesr-general-x4v3` modeli secildiginde komuta eklenir.

`0` secilirse:

- Gurultu temizleme cok az veya yoktur.
- Orijinal detay daha fazla korunur.

`0.2` secilirse:

- Hafif temizlik yapar.
- Varsayilan degerdir.

`0.5` ve uzeri secilirse:

- Daha guclu temizlik yapar.
- JPEG izi ve noise azalabilir.
- Ama logo detaylari fazla yumusayabilir.

Logo icin genellikle `0.2` gibi dusuk degerler daha guvenlidir.

## `sharpen_after`

Aralik:

- `1`
- `1.8`

Bu ayar final hazir gorsele keskinlik uygular.

`1` secilirse:

- Ek keskinlestirme yapilmaz.

`1.1 - 1.3` arasi:

- Hafif ve genellikle guvenli keskinlestirme yapar.
- Logo kenarlarini toparlar.

`1.5 - 1.8` arasi:

- Daha sert keskinlik verir.
- Kenarlarda yapaylik veya halo olusabilir.

Varsayilan: `1.2`

Logo ve QR fusion icin genellikle `1.1`, `1.2`, `1.3` arasi iyi baslangictir.

## `tile_size`

Secenekler:

- `0`
- `512`
- `768`
- `1024`

Real-ESRGAN buyuk gorselleri islerken goruntuyu parcalara bolebilir. Bu parcalara tile denir.

### `0`

Tile kullanmaz.

Ne olur?

- Goruntu tek parca islenir.
- Kalite ve sure acisindan iyi olabilir.
- Ama VRAM yetmezse hata verebilir.

A100 gibi guclu GPU'da denenebilir ama cok buyuk dosyalarda risklidir.

### `512`

Daha kucuk parcalar kullanir.

Ne zaman kullanilir?

- VRAM hatasi aliniyorsa.
- Buyuk dosya isleniyorsa.
- Daha guvenli calisma isteniyorsa.

### `768`

Varsayilan secimdir.

Ne zaman kullanilir?

- A100 GPU icin dengeli ve guvenli ayar isteniyorsa.
- Hiz ve bellek kullanimi dengelensin isteniyorsa.

### `1024`

Daha buyuk parcalar kullanir.

Ne zaman kullanilir?

- VRAM yeterliyse.
- Daha hizli islem denenmek isteniyorsa.

VRAM hatasi alirsaniz `tile_size` degerini dusurun.

## `output_format`

Notebook'ta sadece `png` secenegi vardir.

Neden PNG?

- Logo icin kayipsizdir.
- Seffaflik destekler.
- QR fusion icin daha temiz kaynak verir.

JPG bu is icin ideal degildir cunku sikistirma izleri ve kenar kirlenmesi olusturabilir.

## `MODEL_MAP`

Bu sozluk, kullanicinin sectigi kolay model adini Real-ESRGAN'in bekledigi model adina cevirir.

Ornek:

`realesrgan-x4plus-anime` secilirse, komutta `RealESRGAN_x4plus_anime_6B` kullanilir.

Bu kisim kullanicinin degistirmesi gereken bir ayar degildir. Notebook'un model adlarini dogru komuta cevirmesi icindir.

## 5. Real-ESRGAN Upscale Calistir

Bu hucre logoyu gercekten buyuten hucredir.

Kodun yaptiklari:

1. Yuklenen logoyu `RGBA` PNG formatina cevirir.
2. `/content/logo_work/esr_input.png` olarak kaydeder.
3. `upscale_enabled` true ise Real-ESRGAN komutunu olusturur.
4. Secilen modeli, upscale faktorunu, tile ayarini ve cikti klasorunu komuta ekler.
5. Komutu calistirir.
6. Olusan `_upscaled.png` dosyasini bulur.
7. Ciktiyi ekranda 512x512 onizleme olarak gosterir.

### Calistirilan komutun anlami

Komut kabaca sunu yapar:

```bash
python inference_realesrgan.py \
  -n MODEL_ADI \
  -i INPUT_DOSYASI \
  -o OUTPUT_KLASORU \
  -s UPSCALE_FACTOR \
  --suffix upscaled \
  --ext png \
  --alpha_upsampler realesrgan \
  -t TILE_SIZE \
  --tile_pad 16 \
  --gpu-id 0
```

### `-n`

Kullanilacak Real-ESRGAN modelidir.

### `-i`

Input dosyasidir. Bu notebook'ta:

`/content/logo_work/esr_input.png`

### `-o`

Output klasorudur. Bu notebook'ta:

`/content/logo_esrgan_result`

### `-s`

Buyutme katsayisidir. `2` veya `4` olabilir.

### `--suffix upscaled`

Cikti dosyasinin adina `_upscaled` ekler.

### `--ext png`

Ciktiyi PNG olarak kaydeder.

### `--alpha_upsampler realesrgan`

Seffaflik kanali varsa onu da Real-ESRGAN mantigiyla buyutmesini soyler.

Logo PNG'lerinde bu onemlidir.

### `-t`

Tile size ayaridir.

### `--tile_pad 16`

Tile parcalari arasinda kenar izi olusmasini azaltmak icin ekstra bindirme alani kullanir.

### `--gpu-id 0`

Ilk GPU'yu kullanir.

Colab'da genelde tek GPU oldugu icin `0` dogrudur.

## 6. QR Fusion Icin Kare Hazirla, Arka Plan ve Sharpen Uygula

Bu hucre, Real-ESRGAN'dan gelen dosyayi QR fusion icin son hazir hale getirir.

Kodun yaptiklari:

1. Cikti klasorunu olusturur.
2. `hex_to_rgba` fonksiyonunu tanimlar.
3. `fit_square` fonksiyonunu tanimlar.
4. Arka plan rengini belirler.
5. Upscale edilmis logoyu acar.
6. Logoyu hedef kare boyuta yerlestirir.
7. Gerekirse solid arka plan uygular.
8. Keskinlestirme uygular.
9. Dosyalari kaydeder.
10. Ayarlari `settings_used.txt` dosyasina yazar.
11. Teslim ZIP'i olusturur.
12. Final hazir gorseli ekranda gosterir.

### `hex_to_rgba(value)`

Bu fonksiyon `#RRGGBB` formatindaki rengi RGBA formatina cevirir.

Ornek:

`#FFFFFF` su degere cevrilir:

`(255, 255, 255, 255)`

Son `255`, rengin tamamen opak oldugu anlamina gelir.

### `fit_square(im, size, mode, bg)`

Bu fonksiyon logoyu kare hale getirir.

`mode = cover_crop` ise:

- `ImageOps.fit` kullanir.
- Gorsel kareyi doldurur.
- Kenarlardan kirpabilir.

`mode = contain_pad` ise:

- Yeni kare bir canvas olusturur.
- Logoyu oranini bozmadan bu canvas icine sigdirir.
- Bos alanlari arka plan rengiyle doldurur.

### Arka plan nasil uygulanir?

`background_mode = white` ise:

- Arka plan `(255, 255, 255, 255)` olur.

`background_mode = custom_color` ise:

- `custom_background_color` degeri RGBA'ya cevrilir.

`background_mode = keep_transparent` ise:

- Arka plan `(0, 0, 0, 0)` olur.
- Yani tamamen seffaf kalir.

### Solid arka plan donusumu

Eger `background_mode` `keep_transparent` degilse notebook son gorseli opak hale getirir.

Yani beyaz veya ozel renk secildiyse:

- Logo bu arka planla birlestirilir.
- Sonra tekrar RGBA olarak kaydedilir.
- Ama goruntu artik gorunur olarak seffaf degildir.

### Keskinlestirme

`sharpen_after > 1` ise:

- Gorsel RGB'ye cevrilir.
- Pillow `ImageEnhance.Sharpness` ile keskinlestirilir.
- Eger seffaflik korunacaksa alpha kanali geri eklenir.

Bu adim, Real-ESRGAN sonrasi logonun kenarlarini biraz daha net hale getirmek icindir.

## 7. ZIP Indir

Bu son hucre Colab'dan `delivery.zip` dosyasini indirir.

Kod:

```python
from google.colab import files
files.download('/content/logo_outputs/delivery.zip')
```

Bu hucre calistiginda tarayici indirme penceresi acar.

## Model Secim Notlari

Notebook sonunda kisa bir Markdown notu vardir.

Bu notlar model secimini ozetler:

- Flat logo, ikon, maskot, cizim icin `realesrgan-x4plus-anime`.
- Fotograf, urun gorseli, gercekci logo icin `realesrgan-x4plus`.
- Zaten buyuk logo icin `realesrgan-x2plus`.
- Noise veya JPEG artifact varsa `realesr-general-x4v3`.

A100 icin `tile_size=768` guvenli varsayilan olarak secilmistir.

## Hangi Durumda Hangi Ayari Secmeliyim?

### Klasik logo / ikon hazirlamak istiyorsam

Onerilen ayarlar:

```text
upscale_enabled = True
upscale_model = realesrgan-x4plus-anime
upscale_factor = 4
target_size = 3072
fit_mode = contain_pad
background_mode = white
sharpen_after = 1.2
tile_size = 768
```

Bu, notebook'un varsayilan ve en guvenli akisine yakindir.

### Seffaf PNG olarak kullanmak istiyorsam

Onerilen ayar:

```text
background_mode = keep_transparent
```

Bu durumda logo arka plansiz kalir. Sonraki tasarim aracinda baska bir zemine oturtabilirsiniz.

### Marka rengiyle arka plan vermek istiyorsam

Onerilen ayarlar:

```text
background_mode = custom_color
custom_background_color = #MARKA_RENGI
```

Ornek:

```text
custom_background_color = #101820
```

Renk mutlaka `#RRGGBB` formatinda yazilmalidir.

### JPG, bozuk veya kirli bir logo geldiyse

Onerilen ayarlar:

```text
upscale_model = realesr-general-x4v3
denoise_strength = 0.2
```

Gurultu cok fazlaysa `0.3` veya `0.4` denenebilir. Fazla yuksek deger logoyu yumusatabilir.

### Logo zaten buyuk ve kaliteliyse

Onerilen seceneklerden biri:

```text
upscale_enabled = False
```

veya:

```text
upscale_model = realesrgan-x2plus
upscale_factor = 2
```

Ilk secenek hic upscale yapmaz. Ikinci secenek daha hafif bir buyutme yapar.

### VRAM hatasi alirsam

`tile_size` degerini dusurun:

```text
tile_size = 512
```

Hala hata varsa daha kucuk input kullanmak veya `target_size` degerini dusurmek gerekebilir.

### Daha buyuk final kalite istersem

```text
target_size = 4096
```

Ama bu dosya boyutunu ve sonraki islem suresini artirir.

### Logo kirpiliyor veya eksik gorunuyorsa

```text
fit_mode = contain_pad
```

Bu secim logonun tamamini korur.

### Logo kareyi tamamen doldursun istiyorsam

```text
fit_mode = cover_crop
```

Ama logo kenarlardan kesilebilir. Marka logolarinda dikkatli kullanilmalidir.

## Yeni Baslayanlar Icin En Basit Kullanim

1. Notebook'u Colab'da acin.
2. Runtime tipini GPU yapin.
3. Mumkunse A100 GPU kullanin.
4. 1. hucreyi calistirin.
5. 2. hucrede logonuzu yukleyin.
6. 3. hucrede analizi okuyun.
7. Emin degilseniz 4. hucrede varsayilan ayarlari bozmayin.
8. 5. hucreyi calistirip Real-ESRGAN upscale sonucunu gorun.
9. 6. hucreyi calistirip QR icin hazir PNG'yi uretin.
10. 7. hucreyle ZIP dosyasini indirin.

## Bu Notebook Ne Degildir?

Bu notebook:

- QR kodun linkini uretmez.
- QR matrix / modullerini olusturmaz.
- Okunabilirlik testi yapmaz.
- Stable Diffusion veya ControlNet calistirmaz.
- Son sanatsal QR tasarimini uretmez.

Bu notebook sadece logo tarafini temiz ve kaliteli hale getirir.

## Sonuc

`Real_ESRGAN_Logo_QR_Preparation_A100_Colab.ipynb`, logo dosyasini QR fusion surecine hazirlamak icin tasarlanmis bir Colab notebook'udur.

En onemli final cikti:

```text
/content/logo_outputs/logo_prepared_for_qr.png
```

Bu dosya, logonun Real-ESRGAN ile buyutulmus, kare boyuta oturtulmus, arka plan ve keskinlik ayarlari uygulanmis halidir.

Teslim icin en pratik dosya:

```text
/content/logo_outputs/delivery.zip
```

Bu ZIP, hem hazir gorselleri hem de kullanilan ayarlari bir arada tasir.
