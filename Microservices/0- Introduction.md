کتاب هایی که در این دوره آموزش داده میشه:
- DDD
- Microservice Pattern
- Building Microservice 
- Microservice Architecture 
- Monolitic to Microservices
- Event Storming

اول که یک پروژه تعریف میشه خیلی ساده به نظر میاد. کاربران خیلی کمی براش در نظر گرفته میشه. ولی کم کم که پروژه پیش میره به **فیچر هاش اضافه** میشه و ممکنه حتی تعداد کاربرانی که ازش استفاده میکنن زیاد بشه. به طبع اون تعداد توسعه دهنده ها هم افزایش پیدا میکنه که همه باید روی یک کدبیس پروژه رو پیاده سازی کنن. 
ابتدا پروژه به صورت مونولیتیک شروع میکنیم چون که بیلد کردن و دیپلوی کردن راحته. همچنین پیدا کردن باگ هم توش خیلی راحته، کافیه یک برک پوینت بذاریم از ابتدا تا انتهای فرآیند رو میتونیم دیباگ کنیم. یه کپی از اپ ایجاد میکنیم و یک لود بالانسر میذاریم سرش و به راحتی اسکیل میشه. تست کردنش هم با سلنیوم و اینتگرش تست به راحتی انجام میشه. 

ولی چی میشه که میریم سمت میکرو سرویس؟
- فکر میکنیم اپلیکیش کوچیکه ولی به مرور بزرگ میشه که باعث میشه تیم های مختلف برنامه نویسی ایجاد شه به جای دوتا دولوپر.
- توسعه دهندگان زیاد روی سورس کد حجیم و باعث میشه یک توسعه دهنده جدید بخواد کل کد رو بفهمه نمیتونه.
- اینقد فرآیند ها کند و عذاب آور میشه دیگه نمیشه کار رو جلو برد. چون یک کد بیس هست و کد بزرگ هست. و هر جاش دست بزنی اپلیکیشن میترکه. 

پیچیدگی هایی که یک کدبیس بزرگ ایجاد میکنه:
- بسیار بزرگ برای فهم و یادگیری. مثلا پروژه 20 هزار خط کد یک دولوپر بخواد به همش مسلط باشه خیلی سخت میشه
- رفع باگ و ایجاد ویژگی سخته. یک تغییر همه جا تاثیر میذاره 
- کد غیرقابل فهم و اشتباه  در پیاده سازی. هر چیز جدید اضافه کینم جای دیگه خراب میشه
- با هر کاری شرایط بدتر میشه
- فرآیند توسعه خیلی کند میشه. خود IDE به مشکل میخوره و بیلد کردن خیلی طولانی میشه
- فاصله بین انجام تا انتشار چون نیاز به تایید چند تیم هست و ممکنه چند تیم باید یک کار رو همزمان با هم تموم کنن تا یک بیلد صورت بگیره، ماه های زیادی ممکنه وضعیت پروژه غیر قابل انتشار باشه چون افراد مختلف باید پروژه رو به یک سطح یکسان برسونن تا با هم کار کنه ، و فرآیند تست طولانی میشه که باعث میشه عدم همخوانی با چابکی پیش بیاد. 
- لود بالانس کردن هم سخت میشه چون دوتا پروژه بخوایم بیاریم بالا هر کدوم باید منابع سخت افزاری بالایی داشته باشه که شاید حتی خیلی وقتا همه منابع مصرف نشه و از نظر اقتصادی به صرفه نیست. 
- سورس کد قدیمی میشه و باید لایبراری ها رو آپدیت کرد. حالا یه لایبراری بیاد اگر آپدیتش کنیم به هزارجای پروژه آسیب میزنه و دیگه نمیتونیم آپدیت کنیم به مرور پروژه هی قدیمی تر میشه و مجبوره با تکنولوژي و لایبراری قدیمی با هزارتا باگ کار کنه و کم کم نیرو ها به علت بروز نبودن تکنولوژی و لایبراری از شرکت میرن.  کم کم به جایی میرسه که غیر قابل تست و غیر قابل دیپلوی و غیر قابل توسعه جدید میشه. 


