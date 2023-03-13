
## Proje Taslak

```powershell
- py -m venv env
- .\env\Scripts\activate
- pip install djangorestframework
- django-admin startproject main .
- pip install python-decouple
- py manage.py runserver
- py manage.py migrate
- pip freeze > requirements.txt
```

- add rest_framework to INSTALLED_APPS
- create .env and .gitignore (hidden to SECRET_KEY)

```powershell
- py manage.py runserver
- py manage.py migrate 
```

### PostgreSQL setup

- To get Python working with Postgres, you will need to install the “psycopg2” module.
(Python'un Postgres ile çalışmasını sağlamak için “psycopg2” modülünü kurmanız gerekecek.)

```powershell
- pip install psycopg2
- pip install psycopg2-binary # for macOS
- pip freeze > requirements.txt
```

### Install Swagger

- Dokümantasyon için kullanılır, backend de oluşturduğumuz endpointlerin daha doğrusu API nin dokümantasyonu.
- Swagger, 2010 yılında bir girişim tarafından başlatılan açık kaynaklı bir projedir. Amaç, bir framework uygulamaktır.kod ile senkronizasyonu korurken, geliştiricilerin API'leri belgelemesine ve tasarlamasına olanak sağlamaktır.
- Bir API geliştirmek, düzenli ve anlaşılır dokümantasyon gerektirir.
- API'leri Django rest frameworküyle belgelemek ve tasarlamak için gerçek oluşturan drf-yasg kullanacağız.
Bir Django Rest Framework API'sinden Swagger/Open-API 2.0 belirtimleri. belgeler burada bulabilirsin

- drf-yasg -> django rest_framework yet another swagger generator

- installation

```powershell
- pip install drf-yasg
- pip freeze > requirements.txt
```

- add drf-yasg to INSTALLED_APPS -> 'drf_yasg'

- swagger için güncellenmiş urls.py aşağıdaki gibidir. Swagger belgelerinde, bu modeller güncel değildir. urls.py'yi aşağıdaki gibi değiştiriyoruz.

urls.py ->

```py
from django.contrib import admin 
from django.urls import path 
 
# Three modules for swagger:
from rest_framework import permissions 
from drf_yasg.views import get_schema_view 
from drf_yasg import openapi 
 
schema_view = get_schema_view( 
    openapi.Info( 
        title="Taslak Project-Flight Reservation API", 
        default_version="v1",
        description="Taslak Project-Flight Reservation API project provides flight and reservation info", 
        terms_of_service="#", 
        contact=openapi.Contact(email="rafe@clarusway.com"),  # Change e-mail on this line! 
        license=openapi.License(name="BSD License"), 
    ), 
    public=True, 
    permission_classes=[permissions.AllowAny], 
) 
 
urlpatterns = [ 
    path("admin/", admin.site.urls), 
    # Url paths for swagger: 
    path("swagger(<format>\.json|\.yaml)", schema_view.without_ui(cache_timeout=0), name="schema-json"), 
    path("swagger/", schema_view.with_ui("swagger", cache_timeout=0), name="schema-swagger-ui"), 
    path("redoc/", schema_view.with_ui("redoc", cache_timeout=0), name="schema-redoc"), 
]
```


```powershell
- py manage.py runserver
- py manage.py migrate 
```

- swagger ı kurduk, urls.py daki ayarlarını da yaptık, http://127.0.0.1:8000/swagger/  endpointine istek attığımızda swagger ana sayfasını gördük.

### Install Debug Toolbar

- The Django Debug Toolbar is a configurable set of panels that display various debug information about the current request/response and when clicked, display more details about the panel’s content
  https://django-debug-toolbar.readthedocs.io/en/latest/
(Django Hata Ayıklama Araç Çubuğu, çeşitli hata ayıklama bilgilerini görüntüleyen yapılandırılabilir bir panel kümesidir. geçerli istek/yanıt ve tıklandığında panel içeriği hakkında daha fazla ayrıntı görüntüler)

- Install the package

```powershell
- pip install django-debug-toolbar
- pip freeze > requirements.txt
```

- Add "debug_toolbar" to your INSTALLED_APPS setting -> "debug_toolbar"

settings.py ->

```py
INSTALLED_APPS = [
    'django.contrib.staticfiles',
    # 3rd party packages
    # ....
    'debug_toolbar',
]
```

- Add django-debug-toolbar’s URLs to your project’s URLconf
 
urls.py ->
```py
from django.urls import path, include

urlpatterns = [ 
    # ... 
    path('__debug__/', include('debug_toolbar.urls')), 
] 
```

- Add the middleware to the top:

settings.py ->

```py
MIDDLEWARE = [ 
    "debug_toolbar.middleware.DebugToolbarMiddleware", 
    # ... 
] 
```

- Add configuration of internal IPs to "settings.py":

settings.py ->

```py
INTERNAL_IPS = [ 
    "127.0.0.1", 
]
```

- test edelim çalışacak mı?

```powershell
- py manage.py runserver
```

- çalıştı,   http://127.0.0.1:8000/   endpointine istek atınca gelen sayfanın sağ tarafında bir tool DjDT açılıyor.


- create superuser
```powershell
- py manage.py creaetsuperuser
```


### Seperate Dev and Prod Settings (Development ve Product ayarlarını ayırma)

- Şu ana kadar yaptığımız projelerde tüm projede geçerli olan ayarları settings.py altında topluyorduk. Ancak ideal bir ortamda çok tavsiye edilen bir durum değil. 
- Projenin bulunduğu safhaya göre settings.py ayarları değişeceği için, her ortam için ayrı bir settings.py oluşturulur. 
- Dokümanda görüldüğü gibi bir structure oluşturup, 

```text
settings/
    |- __init__.py
    |- base.py
    |- ci.py
    |- local.py
    |- staging.py
    |- production.py
    |- qa.py
```

    Yukarıdaki örnekteki gibi tek bir yerde hepsinin toplanmasından sa ayrı ayrı dosyalar oluşturup, mesela tüm projede geçerli olan ortak ayarları base.py içerisine almış, ci->Continuous integrations, qa->test gibi setting.py ı parçalara ayırıyoruz.
    
    Biz şuanda temel olarak hemen hemen her projede olan developer ve produc ortamını birbirinden ayıracağız.

- main klasörü içerisine settings klasörü create et, içerisine;
  - __init__.py  # burasının bir python package olduğunu gösterir
  - base.py   
  - dev.py   
  - prod.py   

- main içerisindeki settings.py içeriğini, yeni oluşturulan settings/base.py içerisine kopyalıyoruz.

- Artık settings.py dosyası silinebilir. (Biz ismini değiştirdik duruyor.)

- Şimdi artık settings.py daki kodlarımızı ayırmaya başlıyoruz, ->

##### development settings ayarları ->

- dev.py da olacak kodlar ->
  -  base.py dakileri import et, ve şu kodları ekle;
 
```py
from .base import *

THIRD_PARTY_APPS = ["debug_toolbar"] 
 
DEBUG = config("DEBUG") 
 
INSTALLED_APPS += THIRD_PARTY_APPS 
 
THIRD_PARTY_MIDDLEWARE = ["debug_toolbar.middleware.DebugToolbarMiddleware"] 
 
MIDDLEWARE += THIRD_PARTY_MIDDLEWARE 
 
# Database
# https://docs.djangoproject.com/en/4.0/ref/settings/#databases 
DATABASES = { 
    "default": { 
        "ENGINE": "django.db.backends.sqlite3", 
        "NAME": BASE_DIR / "db.sqlite3", 
    } 
} 
 
INTERNAL_IPS = [ 
    "127.0.0.1", 
]
```

  -  DEBUG = config("DEBUG") dediğimiz için base.py daki DEBUG kısmını siliyoruz/yoruma alıyoruz. Artık base.py da DEBUG yok, dev.py a bakıyoruz orada diyor ki config den al bunu, config nereden alıyordu? .env den alıyordu, oraya gidip "DEBUG = True" ekliyoruz. DEBUG ı base den kaldırdık, artık developer dayken config e bakacak orda ne yazıyorsa onu kullanacak.

  -  THIRD_PARTY_APPS = ["debug_toolbar"]  ı dev.py a almış, o yüzden base.py daki INSTALLED_APPS deki debug_toolbar ı kaldırıyoruz. debug_toolbar ne zaman çalışacak? sadece developer dayken çalışacak.
  (INSTALLED_APPS += THIRD_PARTY_APPS komutu ile eklemiş.)
 
  -  debug_toolbar ı dev.py a aldığımıza göre yine kendisi ile birlekte gelen middleware ı vardı yine onu da buraya alıyoruz ve base.py da yoruma alıyoruz.

  -  Default olarak djangoda sql3 kullanılıyordu, o da burada, o zaman base dekini yoruma/siliyoruz. Developer dayken sqlite kullan, onu da burada belirtiyorum.

  -  Yine INTERNAL_IPS i developer dayken kullan diyoruz ve base.py dan siliyoruz/yoruma alıyoruz.
  -  development ayarlarını ayırdık.

##### product settings ayarları ->

- Peki product halindeyken nasıl olacak? 
  - Aynı şekilde base.py daki herşeyi import et,

prod.py ->

```py
from .base import * 
 
DATABASES = { 
    "default": { 
        "ENGINE": "django.db.backends.postgresql_psycopg2", 
        "NAME": config("SQL_DATABASE"), 
        "USER": config("SQL_USER"), 
        "PASSWORD": config("SQL_PASSWORD"), 
        "HOST": config("SQL_HOST"), 
        "PORT": config("SQL_PORT"), 
        "ATOMIC_REQUESTS": True, 
    }
}
```
  
  - product ortamındayken şunu ilave et, bu ne? database. dev deyken sqlite idi, product ta iken bu! Burada ne var? PostgreSQL. Bağlantıyı yapacağız şimdi.
  
  - Buraya bakıyoruz yine configden kullandığı değişkenler var. Bu değişkenleri alıp .env file ımıza ekleyelim.
  
    SQL_DATABASE =  
    SQL_USER =
    SQL_PASSWORD =
    SQL_HOST = 
    SQL_PORT = 
  
  - Artık product taki postgresql server ın bağlantıyı kurabilmesi için gerekli parametreleri .env ye aldık. Şimdi bunları dolduralım. pgAdmin i çalıştırıyoruz, ve flight isminde bir db create ediyoruz, daha sonra .env deki değişkenleri create ettiğimiz db deki değişkenlerle update ediyoruz.
  
    SQL_DATABASE = flight
    SQL_USER = postgres
    SQL_PASSWORD = postgres
    SQL_HOST = localhost
    SQL_PORT = 5432
  
  - Böylelikle development da iken sqlite, product da iken postgresql kullanmasını söyledik.


- Biz prod ve dev diye ayırdık ama bunlardan hangisini kullanacağını nasıl bilecek?

##### __init__.py settings ayarları ->

- Onun için settings/ __init__.py file ına;

```py
from .base import * 


env_name = config("ENV_NAME") 
 
if env_name == "prod": 
 
    from .prod import * 
 
elif env_name == "dev": 
 
    from .dev import *
```

- kodları yazıyoruz. Burada ne demek istiyoruz? 
  - base.py dan herşeyi al, 
  - eğer config deki ENV_NAME i prod ise from .prod import * bunu ekle, 
  - değil config deki ENV_NAME i dev ise from .dev import * bunu ekle diyoruz.
  - Ancak config içindeki ENV_NAME i alıp .env file içinde "ENV_NAME = prod" olarak ekliyoruz.
  - Artık init.py dosyamız ENV_NAME i config den prod (burayı dev/prod olarak manuel değiştirebiliriz.) olarak alıyor, madem prod o zaman, "from .base import *"  bunu zaten almıştı bir de  "from .prod import *" bunu çalıştır diyoruz.



- Şimdi biz daha önce bir migrate yapmıştık ama o sqlite içindi. Artık yeni bir db ye bağladığımız için yine bir migrate yapacağız. 

- Şimdi bizim projemiz production modunda ve postgresql db ye bağlı biz artık migrate ile tablolarımızı postgresql db mizde oluşturabiliriz. 

```powershell
- py manage.py migrate
```

- Artık modellerimizin tablo karşılıklarını postgresql db de oluşturduk.

- Default olarak oluşturduğumuz projelerdeki global settings ayarlarını settings.py da tutuyorduk. Hatta buradaki bir ayarı app içerisinde değiştirebiliyoruz. settings.py da tüm ayarları bir arada tutmaktansa projemizin bulunacağı ortama göre ihtiyacımız olan ayarları parçalıyoruz.
- Developer a ne lazım? sqlite ta çalışması lazım, o zaman developer kullandığı zaman sqlite çalışsın/ayarlar buna göre olsun,  product ortamına alındığı zaman da postgresql e  bağlansın ve ayarlar product ortamına göre olsun.
- Tüm projede geçerli olmasını istediğimiz ayarlar base.py da, sonra ortam değişince değişmesini istediğimiz ayarlarımızı da yine mantıksal olarak ilişki kurduğumuz dosyaya aktardık.

- Development ortamında "DEBUG=True" olsun, product ortamında "DEBUG=False" olsun. Şu an onu yapmadık ama ekleyebiliriz. Şu paketi development ortamıda kullan, product ortamında kullanma gibi...

- product ortamında "DEBUG=False" olsun istiyoruz,
  - 1- dev.py da "DEBUG = config("DEBUG")" şeklinde .env den değil de; prod.py da direkt olarak "DEBUG=False", dev.py da da "DEBUG=True" diye ekleyebiliriz.
  - 2- .env de True yu False yapabiliriz.
  - 3- prod.py da zaten DEBUG = config("DEBUG") gibi bir değişken belirtmeyerek otomatik olarak DEBUG =False olacaktır. Ancak   

- Şu anda prod ortamı seçili ve de prod.py da "DEBUG=False" diye bir şey belirlemedik, base.py da da "DEBUG=True" yu yoruma/sildik, böylelikle otomatik olarak prod ortamında "DEBUG=False" olur. Çünkü bir değer tanımlamadık. Ancak "DEBUG=False" olduğu zaman base.py daki "ALLOWED_HOSTS = []" u bu şekilde bırakamayız, ya "ALLOWED_HOSTS = ["127.0.0.1"]" yazmalıyız ya da uğraşmayayım onunla diyip "ALLOWED_HOSTS = ["*"]" yazacaksınız.

- .env den dev ortamına geçtik, bizden tekrar migrate istedi, ve de tekrar superuser istedi.

```powershell
- py manage.py migrate
- py manage.py createsuperuser
```

### Logging

- logging işlemi restframework ile beraber geliyor olması lazım.

- Python programcıları, kodlarında hızlı ve kullanışlı bir hata ayıklama aracı olarak genellikle print() öğesini kullanır. 
- logging framework (günlük kaydı çerçevesi) kullanmak  bundan biraz daha fazla çaba gerektirir, ancak çok daha zarif ve esnektir. 
- Günlüğe kaydetme, hata ayıklama için yararlı olmanın yanı sıra uygulamanızın durumu ve sağlığı hakkında size daha fazla - ve daha iyi yapılandırılmış - bilgi de sağlayabilir.
- Django, sistem günlük kaydı gerçekleştirmek için Python'un yerleşik günlük modülünü kullanır ve genişletir. 
- Bu modül Python'un kendi belgelerinde ayrıntılı olarak ele alınmıştır; bu bölüm hızlı bir genel bakış sunar.

- Dokümandaki LOGGING kodunu alıp base.py da en sona ekliyoruz.
setting/base.py
```py

LOGGING = { 
    "version": 1, 
    # is set to True then all loggers from the default configuration will be disabled. 
    "disable_existing_loggers": True, 
    # Formatters describe the exact format of that text of a log record.  
    "formatters": { 
        "standard": { 
            "format": "[%(levelname)s] %(asctime)s %(name)s: %(message)s" 
        }, 
        'verbose': { 
            'format': '{levelname} {asctime} {module} {process:d} {thread:d} {message}', 
            'style': '{', 
        }, 
        'simple': { 
            'format': '{levelname} {message}', 
            'style': '{', 
        }, 
    }, 
    # The handler is the engine that determines what happens to each message in a logger. 
    # It describes a particular logging behavior, such as writing a message to the screen,  
    # to a file, or to a network socket. 
    "handlers": { 
        "console": { 
            "class": "logging.StreamHandler", 
            "formatter": "standard", 
            "level": "INFO", 
            "stream": "ext://sys.stdout", 
            }, 
        'file': { 
            'class': 'logging.FileHandler', 
            "formatter": "verbose", 
            'filename': './debug.log', 
            'level': 'INFO', 
        }, 
    }, 
    # A logger is the entry point into the logging system. 
    "loggers": { 
        "django": { 
            "handlers": ["console", 'file'], 
            # log level describes the severity of the messages that the logger will handle.  
            # "level": config("DJANGO_LOG_LEVEL", "INFO"), # .env ye alıyoruz.
            "level": config("DJANGO_LOG_LEVEL"), 
            'propagate': True, 
            # If False, this means that log messages written to django.request  
            # will not be handled by the django logger. 
        }, 
    }, 
} 
```

- Logging ayarlarına baktığımızda baktığımızda formatters, handlers, loggers bölümleri var. linkteki dokümantasyonda mevcut.
  https://docs.djangoproject.com/en/4.0/topics/logging/#logging

- loggers kısmının;
  - level kısmında -> Hangi seviyede log tutacak isek onu belirtiyoruz -> INFO (INFO, DEBUG, WARNING, ERROR, CRITICAL) 
  
  - handlers (topluyor, tutuyor) kısmında -> console, file belirtilmiş. Bunlar da hemen üstte detayları var;
        console; stream çıktıyı düzenli bir akış ile terminale console a veriyor.
            level; INFO
            formatter; hemen biraz üstte standart, verbose, simple olarak belirtilmiş olanlardan standart seçili.
        file; bulunduğu yerdeki ./debug.log isimli bir dosyaya kaydediyor.
            level; INFO
            formatter; hemen biraz üstte belirtilmiş olanlardan verbose seçili.


- level ile -> neleri loglayacak, bunu belirtiyoruz, 
- formatter ile -> loglanan şeylerin ne kadar detayını alacak bunu belirtiyoruz, 
  
- çalıştıryoruz; 
- debug.log isimli bir dosya oluşturukmuş olduğunu ve logging de belirttiğimiz kayıtların olduğunu gördük.

- log işlemi yapıyorsa, her yaptığımız işlemde özellikle INFO belirttiysek, backend de gerçekleşen her işlemi bir yere yazması lazım. Eğer WARNING yazarsak, dosyaya sadece WARNING leri kaydeder. vb.


- "formatters" , "handlers" ,  "loggers" diye yapılarımız var.

- hangi seviyede tutacağım? -> "level": config("DJANGO_LOG_LEVEL", "INFO"),

- neleri tutacağım/handle edeceğim? -> "handlers": ["console", 'file'],
  
- handle ediyorsam nerede saklayacağım? -> 'filename': './debug.log',


- logların level ını .env de tutacağız. Kurarken level olarak 'INFO' belirledik tamam, ama bu ayarı .env de tutarak oradan değiştireceğiz.
    "level": config("DJANGO_LOG_LEVEL") yazıp,
    .env de de -> 
        DJANGO_LOG_LEVEL=INFO  olarak kaydederek log level seçeneğini .env ye taşımış oluruz.

- "level": config("DJANGO_LOG_LEVEL", "INFO"), # .env ye alıyoruz.
  "level": config("DJANGO_LOG_LEVEL"), 




- Herhangi bir hatada veya server ın nasıl çalıştığını izleyebilmek için, bu logları tutuyoruz. Bu loglardaki trafiği takip ederek bazı kararlar alabiliyoruz. Mesela bir endpoint e bir çok istek var, demekki birisi birşeyler deniyor.
- Burada önemli olan husus logları hangi seviyede tutacağız?, neleri handle edeceğiz?, handle ediyorsak nerede saklayacağız? bunları belirtiyoruz.

#### BRD -> business requirement document;
  - Projenin tüm isterlerini, özelliklerini, gereklerini yazdığımız doküman. Projedeki ana kaynak. 
  - Projeye başlamadan önce dizayn kısmında, tasarım kısmında hazırlanır.
  - Software Development Life Cycle (SDLC) İlk aşama gerekliliklerin toplanması, projenin dizayn edilmesi, kurgulanması, isterlerin biraraya getirilmesi.
  - Poje ile ilgili herşey yazar, projenin imkan/özellikleri, hangi tip kullanıcılar olacak?, normal user/admin/başka roller, her kullanıcı rolü için yetkiler, bu dokümanda bulunur.
  
- Projeye başlanacağı zaman Agile kavramından hatırlarsanız proje isterlerinin hepsi task olarak product backlog lara alınıyor. Daha sonra sprintler halinde sprint backlog lara gerekli task lar halinde atılıyor, ve ilgili developer lara assign ediliyor. Böyle devam ediliyor.
- Bir projeye başlarken bu kısımlara zaman ayırıp en ince detayına kadar planlanıp, belirlenmesi gerekiyor.

#### ERD -> Entity Relationship Diagram
  - db tablolarını şekillendirmek, bir diyagrama dökmek. 

#### Swagger -> Document + Test (swagger->ortam, redoc->dokuman)
  - çok kullanılan bir tool, yazdığımız backend api, bunu frontend de bizim takım arkadaşlarımız da kullanabilir, veya dışarıya da servis edebiliriz. Bunun için yani servisin nasıl kullanılacağına dair bir klavuz hazırlanması gerekir.
  - end pointler için doküman oluşturur
  - swagger->ortam, redoc->document
  - Nasıl ekleniyor? ->
    - pip install drf-yasg şeklinde install ettikten sonra, drf_yasg ile third party package ile ekledik,
    - main urls.py da schema_view ve endpointlerini ekledik.

#### Debug_Toolbar -> Documentation 
- Development aşamasında projemizi geliştirirken bize ekstra kabiliyetler kazandırıyor.
- Nasıl ekleniyor? ->
    - pip install django-debug-toolbar şeklinde install ettikten sonra, drf_toolbar ile third party package ile ekledik, daha sonra dev settings ayarlarına aldık.
    - sonra MIDDLEWARE e en üst kısma "debug_toolbar.middleware.DebugToolbarMiddleware" ekledik, daha sonra dev settings ayarlarına aldık.
    - sonra INTERNAL_IPS a da ayarını ekledik, daha sonra dev settings ayarlarına aldık.
    - urls.py da bir de debug_toolbar ın   path('__debug__/', include('debug_toolbar.urls')),     şeklinde bir endpointi var, onu ekliyoruz.
    - En son migrate komutunu çalıştırdık.

#### Logger -> işlem geçmişimizi tutmaya yarayan kullanışlı bir tool.
- settings.py da en alta ekledik, ardından base.py dosyasına aktardık.
- Bu bir şablon şeklinde, tekrardan yazmıyoruz, bu şekilde kullanıyoruz. 
  - En alttan başlıyoruz bakmaya,  
  - loggers da handlers kısmında loglarımızı hem console a yazdır, hem de file oluştur demişiz.
  
  - level kısmına INFO demişiz, hangi level de tutabileceğini belirleyebiliyoruz.
  ( DEBUG: Low level system information for debugging purposes
    INFO: General system information
    WARNING: Information describing a minor problem that has occurred.
    ERROR: Information describing a major problem that has occurred.
    CRITICAL: Information describing a critical problem that has occurred.)

  - belirttiğimiz console ve file ayarlarını da hemen üstte handlers ta configüre ediyoruz, console a yazarken ki ayarlar, file a yazarkenki ayarlar.. Dikkat edilmesi gereken formatter kısmı -> 
    - console da standart, 
    - file da verbose demişiz.
  
  - formatter a da bir yukarıda bakıyoruz, standart, verbose, simple ile yani logların hangi formatta nasıl tutulacağını burada belirliyoruz.
  - belirlediğimiz formatları aşağıda handlers kısmında seçebiliyoruz, handlers ları da bir alttaki loggers kısımda seçebiliyoruz, ikisini de seçmişiz ayrıca  logger seviyesini de level ile seçebiliyoruz.(level ı config ile .env ye aktarabiliriz. Buradan env ortamını değiştirebildiğimiz gibi debug level ını da değiştirebiliriz.)


#### postgreSQL
- pgAdmin de bir db oluşturup, 
- oluşturduğumuz db ayarlarını prod.py kısmına yazıyoruz,
- gizlenmesi gereken verileri config ile .env ye aktarıyoruz.


- base(ortak ayarlar), dev(development ayarları), prod(production ayarları)



- Şimdi bunu taslak olarak kullanıp bu taslak üzerine projeler oluşturabiliriz.
- Bunu repomuza push layıp, 
- repomuza pushladık, repoya girdik, settings, template repository seç, -> yeni bir repository dediğimiz zaman yeni oluşturacağımız repoyu bu taslağa alır ve o şekilde oluşturur. Yani bu taslak üzerinden devam edebilirsiniz, tekrar tekrar paket kurmanıza gerek kalmaz.
