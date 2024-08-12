# فیچر لیست کرولر[](https://docs.mci.dev/display/PF/Crawler#Crawler-فیچرلیستکرولر)

پیدا کردن Outlink ها از HomePage.

به صورت multi thread بتونه درخواست بزنه و صفحات جدید رو بگیره.

الگوریتم ReCrawl خوبی داشته باشه. مثلا ساده ترین الگوریتم این میتونه باشه که HomePage رو سریع تر و عمق های پایین تر را دیرتر ReCrawl کنه.

روی کوبر بیاد بالا و بشه scale کرد. 

بتونیم بهش proxy بدیم تا از چند ip درخواست بزنه.

بتونه url ها رو standard کنه تا یک url با فرمت های مختلفت رو چند بار پردازش و ذخیره نکنیم.

توی دیتابیس ttl داشته باشه و بعد از مدتی دیتا پاک شه.

قابلیت Render کردن صفحات single page رو داشته باشه.

برای محاسبه next fetch باید diff محتوا رو بگیریم 

بشه راحت روش develop کرد.

# ابزارهای OpenSource[](https://docs.mci.dev/display/PF/Crawler#Crawler-ابزارهایOpenSource)

|   |   |   |   |   |
|---|---|---|---|---|
|## apache storm[](https://docs.mci.dev/display/PF/Crawler#Crawler-apachestorm)|Java|879|3 months||
|## Scrapy[](https://docs.mci.dev/display/PF/Crawler#Crawler-Scrapy)|Python|51.9k|3 weeks|Should define parser to find next link|
|~~Heritrix~~|Java|2.8k|3 months|Working with UI|
|## Colly[](https://docs.mci.dev/display/PF/Crawler#Crawler-Colly)|Go|22.8k|2 months||
|~~ache~~|Java|445|1 year||
|~~PySpider~~|Python|16.4k|4 years||
|~~Nutch~~|Java|2.9k|4 months||
|~~WebCrawler~~|Java|3.1K|1 year||
|webMagic|Java|11.4k|1 month|not easy syntax|
|~~hakrawler~~|Go|4.3k|2 years||
|## crawlee[](https://docs.mci.dev/display/PF/Crawler#Crawler-crawlee)|JavaScript|13.9k|2 months|use **[playwright](https://github.com/microsoft/playwright)** to open browser|
|## crawl4ai[](https://docs.mci.dev/display/PF/Crawler#Crawler-crawl4ai)|Python|1.6k|yesterday||
|~~crawlergo~~|Go|2.8k|1 year|`use chrome headless`|
