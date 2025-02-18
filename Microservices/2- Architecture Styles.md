### معماری سیستم چیست؟
معماری سریع اجزا رو تعریف میکنه یا سیستم را تقسیم کنیم به بخش های مختلف و ارتباطات بین آن ها رو تعیین میکنه. به شکل ساده و شماتیک کل سیستم را بیان میکنه. در این تقسیم کردن باید پراپرتی هایی که داریم یا محدودیت هایی که داریم را بیان بکنیم. 
هزینه تولید و نگهداری سیستم را کم میکنه. نیازمندی های مهم سیستم را بیان میکنه. 
>هر سیستم نرم افزاری نیاز داره ساختار ساده داشته باشه در عین حال عمل کرد خوبی داشته باشه.


### ارائه معماری
 1. پاسخ به نگرانی های stackeholder های مختلف. نقشه های مختلف باید داشته باشیم مثلا معماری شبکه یا معماری deployment یا برای توسعه دهنده ها باید معماری جدا داشته باشیم
2.  پاسخ به نیاز های مختلف functional و non functional. برای این کار از 4 + 1 Architectural view استفاده میکنیم. شامل Logical view, Development view, physical view, Process view می شود. 
	- ویو Logical view
		- ارتباط Object oriented decomposition
			- اشیا و ارتباط بین آن ها
			- ویژگی های fUNCTIONAL
			- مشاهده توسط کاربر
	- ویو Process view
		- مجموعه وظایف NonFunctional را در یک فایل یا مربع ارائه میکنیم و هر مربع یک Process هست که با Processهای دیگه ارتباط داره
	- ویو Development View
		- تقسیم کردن اپلیکیشن به کامپوننت از نظر منطقی که توسط برنامه نویس و معمار مورد تعیین و استفاده قرار میگیره
	- ویو physical/deployment view
		- ساختار سخت افزاری به چه شکلی هست و هر ماژول نرم افزاری در کدام بخش سخت افزاری قرار میگیرد. از نظر شبکه و سرور
3. هدف از باکس ها و خطوط بین آن ها چیست؟
4. معرفی توسط philippe krutchen



### سبک معماری یا Architecture Style
سبک های مختلف معماری رو باید آشنا باشیم. ویژگی های سیستم رو لیست کن و از بین معماری هایی که وجود داره یکی رو انتخاب کن و تو اون قالب پیش برو. نیا از صفر یک معماری برای خودت درست کن. ما قراره الگو های معماری میکرو سرویس رو بررسی کنیم. 

#### سبک چند لایه
- اپلیکیشن رو از نظر functionality به لایه های مختلف بشکونیم و هر لایه فقط از لایه پایینی استفاده میکنه و با لایه های دیگه نباید صحبت کنه. 
- اگر یک لایه تغییر کنه فقط روی لایه بالایی تاثیر میذاره
##### ایرادهای این لایه
- ممکنه یه سری لایه ها فقط داده رو رد کنن و هیچ تاثیری روی داده نذارن چون هر لایه فقط با لایه پایینی در ارتباط است. 
- عدم توجه به خروجی های متفاوت. چون ممکنه خروجی وب داشته باشیم یا خروجی موبایل داشته باشیم 
- یک لایه برای کار با داده و اگر چند دیتا اکسس داشته باشیم نمیشه
- وابستگی منطق به دیتابیس و یک تکنولوژی 

#### سبک Domain centric
لایه دامین رو میذاریم وسط کار و بقیه وابستگی های تکنیکالی حول این پیاده سازی میشه و این لایه به هیچی وابستگی نداره. انواع پیاده سازی این سبک به شرح زیر است. 
1. سبک **Hexagonal** Architecture: هسته اصلی و لاجیک رو در وسط پیاده سازی میکنیم. یک سری رابط یا interface برای این لاجیک تعیین میکنیم برای ارتباط با دنیای خارج و هرکی خواست از این ها استفاده کنه باید adapter های این ها رو پیاده سازی کنه و از آنها استفاده کنه.
2. سبک **Onion** Architecture: میگه اپلیکیشن رو بشکون به domain model, domain service, application servic و غیره و برای ارتباط با دنیای بیرون infrastruction  یا user interface باید داشته باشیم بیشتر سعی میکنیم از این نوع استفاده کنیم
3. سبک **Clean** Architecture: شبیه بالاییه ولی اسم هاش رو متفاوت کرده
4. سبک **Microservice** Architecture: هر کامپوننت یک جز مستقل هست که میشه کامل حذف کرد و معادلش رو قرار داد و عمکرد سیستم خللی وارد نشه

#### سبک ماکروسرویس
که در ادامه روش صحبت میکنم.
### سرویس چیست
- نرم افزار مستقل 
- قابل انتشار به تنهایی
- پیاده سازی کارکردی ارزشمند
- دربرگرفته شده با API
- معماری داخلی مستقل با دیتابیس مستقل 
- خاصیت LOOSE COUPLE
- کتابخانه اشتراکی ندارن
- اهمیت اندازه در محدوده سرویس ها، مثلا اندازه و ویژگی های هر سرویس چقد باشه و کامپوننت رو از کجا بشکونیم تا بشه یک سرویس


### شروع توسعه مبتنی بر Microservice
تشخیص نیازمندی ها و عملکردها. تشخیص و درک نیازمندیهای سیستم. چه آبجکت ها و چه ارتباطاتی بین انها هست. یکی از این روش ها **event storming** هست. ایجاد domain model میکنیم. اصلی ترین نیازمندی ها که در **سند نیازمندی** آمده. نامگذاری مطابق با نام  entity ها در اسناد نیازمندی
مثلا سند نیازمندی رو که بهمون دادن از توش **کلید واژه های عملکردی** رو میکشیم بیرون و **ارتباط بین آن ها** رو رسم میکنیم. **فانکشنالیتی ها** هم لیست میکنیم به فارسی. مرحله بعد اینکه که جزئیات رو لیست کنیم که شامل این است که:
- چه کارهایی انجام می شود و ورودی مورد نیاز چیست؟
- بعد از انجام کار چه مقداری بازگشت داده می شود؟
- پیش شرط ها چیست؟
- پس شرط ها چیست؟
#### تقسیم به سرویس ها
با توجه به فرآیندی که در اپلیکیشن در حال انجام هست سرویس ها رو تشخیص میدیم و از هم جدا میکنی. چه کاری در سیستم داره انجام میشه و با توجه به جواب این سوال سرویس ها رو جدا میکنیم. با این کار با توجه به **ساختار سازمانی** اومدیم سرویس ها رو تشخیص دادیم برای مثال
با توجه به کارمندان و واحد هایی که واقعا تو سازمان داره کار میکنه بیایم سرویس ها رو جدا کنیم مثلا واحد حساب داری یا واحد ثبت نام مشتریان یا بخش وفادار سازی مشتریان یا مدیریت پشتیبانی مشتریان یا ... روش دوم استفاده از Domain driven design هست که دوره ای جدا داره.
 >نکته: سرویس هایی باید جدا کنیم که اصل تک مسئولیت درش رعایت شده باش
نکته: یک موضوع باید باعث تغییر سرویس بشه یا هیچ چیز منجر به تغییر سرویس نشه که بهش میگن common colusion
#### تعیین API ها
هر کدام از functionality را به یک api باید map بکنیم. علاوه بر این باید برای رفع پیش شرط و پس شرط ها بین کامپوننت ها ارتباط برقرار کنیم. 
موانعی که در این راه داریم 
عدم امکان انجام ماشینی

#### مشکلات رایج
- کندی شبکه در معماری میکرو سرویس. در صورتی که معماری مونو شبکه برامون چالش نبود و همه چی از رم و سی پی رو رد میشد ولی الان از شبکه رد میشه 
- ارتباطات در سیستم مهمه چون خیلی از کار ها رو به صورت سینک انجام میدیم و اگر یکی کند بشه همه کند میشن
- مشکلات داده موقع نوشتن داده transactional و موقع خوندن داده ممکنه نیاز شه از چند سرویس مختلف داده دریافت کنیم و برامون مهم هست که همه آنها با هم همخوانی داشته باشه


