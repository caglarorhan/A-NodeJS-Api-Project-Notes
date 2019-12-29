**NodeJS Api Projesi**
- Proje sinema film ve yonetmen kayitlari uzerine
- Express generator kullanilarak proje taslagi olusturuldu
- Mongoose kuruldu
- Route lar olusturuldu
- get ve post request testleri icin Postman kullaniliyor
- Tum olasi route (url) lar bir tablo icinde dokuman olarak yazildi, boylece tum endpointleri bunlara bakarak yazilabilir hale getirildi, bu tabloda tablo basliklari route, http verb (method), post body, description
- bodyParser kullanilarak post datasi parse ediliyor, bunun icin gerekli bodyParser middleware leri ayarlaniyor
- Gelen data parse edildikten sonra mongoDB ye kaydedilebilmek icin Models altindaki ilgili model (movie) import edilir
- Movie modelinden bir degiskene instance olusturulup atanir.
- bu degiskende ilgili alanlar obje olarak doldurulup .save() metodu ile kaydedilir. Dogrudan request.body de kullanilabilir. body de nesnedir. Gerekli herhangi bir alan eksik olursa hata alinir.
- kayit sirasinda Promise yapisi kullanilir (try-catch ile), bunun icin once global tanimlamasi yapilmalidir. mongoose.Promise = global.Promise ile
- Tum endpointlerin islemleri tek tek kodlaniyor, kayitlar,  listelemeler, silme ve guncellemeler vb...
- Front-end tarafindan bagimsiz olarak sadece postman ile serverside tarafi kodlanip en son (tum cevpalar json olarak donuyor) istenilen herhangi bir yontemle/framework ile front-end tarafi yazilabilir.
- guncelleme isleminde guncelleme bittiginde ilgili kaydin eski hali gosteirilyor, yeni hali icin new:true parametresi ayri bir obje icinde gonderilmeli
- MongoDB semalarini (Model) olusturuken type, default, maxlength ve minlength validasyonlari kullanilabilir.
- Yetkilendirme icin JWT (JSON Web Token) kullanilabilir. JWT, RFC 7519 isimli standarti kullanir.
- Password gibi alanlarin hash edilmesi icin bcrypt modulunu kullaniriz.

**JWT Nasil Kullanilir**
- jsonwebtoken kurulur
    `npm install jsonwebtoken --save`
- giris icin (authenticate) bir route olusturulur
- giris formundan (post ile) geldigi varsayilan veriler bu route ile kabil edilir
- user ve pass uzerinden db de kontroller yapilir, (password `bcrypt.compare` ile pass kontrol edilir)
- Pass dogru use token olusturulur
    - api icin gizli bir ifade tespit edilir ve sayfaya dahil edilir.
    - app.js icinde `app.set('api_secret_key', config.api_secret_key)` ile atamasi yapilir 
    - bu set edilen deger diger sayfalarda `app.get('api_secret_key')` ile alinip kullanilir    
    - gerekli tum sayfalara `const jwt = require('jsonwebtoken')` ile api dahil edilir
        - `const token = jwt.sign(payload, req.app.get('api_secret_key'),{ expiresIn: 720} //12 saat` ;
        - boylece token olusuturuldu 
    - diger sayfalarda bu token kontrolu icin bir middleware yazilir
    - middleware sayfasina `verify-token.js` diyelim. 
    - Once bu sayfaya  `jsonwebtoken` dahil edelim
    - Sayfada tokenin ulasma sekillerinin hepsini alabilmeliyiz, bunlar request header icinde (`req.headers['x-access-token']`), post bodysi icerisinde (`req.body.token`) veya get requesti uzerinde (`req.query.token`) gelebilir.
    - `const token = req.header['x-access-token] || req.body.token || req.query.token ;` ile token degeri alinir
    - Token hic gelmiyorsa hata mesaji ile token gelmedigi bildirilir
    - jwt'nin verify metodu ile token dogru mu kontrol edilir, dogru deilse hata mesaji dondurulur
    - verify metodunun ilk parametresi token, ikincisi secretkey, ucuncusu de callback fonksiyonu
    - callback fonksiyonu 2 parametre alir, birincisi olusursa hata `err` digeri de decode edilmis bilgi `decoded`
    - Eger token bos degilse 
    -       jwt.verify(
                        token,
                        req.app.get('api_secret_key'),
                         (err,decoded)=>{
                                        //requeste decoded eklenir
                                        req.decoded = decoded;
                                        // sonraki middleware e ilerlenir
                                        next();
                                        }
                       )
    - Bundan sonra ana dosyamizda (app.js ya da index.js ) require edilip dahil edilir
        - `const verifyToken = require('./middleware/verify-token');`
    - Sonra `app.use()` ile middleware olarak kullanilir    
        - `app.use('/api', verifyToken);` olarak ustlerde eklendiginde altindaki tum middlewarelerden once devreye girer ve kullanilabilir
    - burada decoded bizim sagladigimiz ve kullaniciyi tanimakta kullandigimiz payload'umuzdur, genelde kullanici adi gibi gizliligi cok onemli olmayan bilgileri barindirir.
    
**JWT Hakkinda**
- Tamamen tek basina bir yetkilendirme yontemi olarak kullanilmamalidir.
- Mumkun oldugunca local (standart cookie veya local storage) de saklanmamali HTTP-cookie icinde saklanmalidir
- Iletisimde kesinlikle HTTP degil HTTPS kullanilmalidir. Aksi taktirde ortadaki adam saldirilarinda token ele gecerse kullanici yerine her turlu girisimde bulunabilirler.
- Mumkun olan durumlarda mutlaka ikinci bir dogrulama yontemi uygulanmalidir. (Google two-factor Authenticate, SMS ile dogrulama vb.))
- Token ele gecerse senaryolarinda, tokenleri sifirlama, kullanicidan sifre degisikligi isteme, ele gecmenin belirlenmesi icin AI destekli (TensorFlow) davranissal takip yapilmalidir
- Secret Key mumkun oldugunca buyuk ve karisik olmali ve kendisi de hash lenmelidir.
- Tokenlere mutlaka omur verilmelidir (expire time).
- JWT stateless dir. Bir oturum yonetimi olarak dusunulmemelidir.

   


  
 
