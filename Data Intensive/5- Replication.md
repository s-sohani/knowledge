There are various reasons why you might want to distribute a database across multiple machines:
- Scalability
- Fault tolerance/high availability
- Latency

# Scaling to Higher Load
If you constantly have a high load, it's better to do vertical scaling and have a machine with sufficient resources.
Next, it mentions two methods for vertical scaling: shared-memory architecture and shared-disk architecture.
The shared-memory architecture suggests that you can connect several CPUs, RAM, or disks together and place them under a single operating system.
The shared-disk architecture, on the other hand, suggests that you can have several independent machines, each with its own CPU and RAM, but all working on a shared disk.

The problem with shared-memory architecture is that when you have a machine with twice the resources, first, the costs more than double, and secondly, it can't handle exactly twice the load due to various reasons, including bottlenecks.
The problem with shared-disk architecture is the overhead of locking, which disrupts the system's ability to scale.

# Shared-Nothing Architectures
The opposite of this is the shared-nothing architecture, also known as horizontal scaling. In this architecture, you distribute the data across independent machines or nodes.
It seems that this approach can reduce latency and help address the issue of data loss.

# Replication Versus Partitioning
In the shared-nothing architecture, data is distributed across different nodes.

There are two different approaches for data distribution:
- **Replication:** Keeping multiple identical copies of data on different machines.
- **Partitioning:** Dividing data into smaller parts and storing each part on an independent machine. This is also known as sharding.

# Replication
Replication means having multiple copies of the same data on different machines that are accessible through a network.

### Why Replication is Necessary:
1. **Reducing Latency**: By placing data geographically closer to users, we can reduce latency.
2. **Fault Tolerance**: To ensure that the system remains available even if one copy of the data fails.
3. **Increasing Throughput**: Distributing read requests across multiple machines can increase the system's throughput.

### Challenges in Replication:
The main difficulty in replication is that data is constantly changing, so the copies need to be kept in sync.

### Common Methods for Data Synchronization:
1. **Single-Leader**
2. **Multi-Leader**
3. **Leaderless**

Challenges also arise in choosing between synchronous and asynchronous replication, as well as handling failed replicas.

### Common Solution: Master/Slave or Active/Passive Model
In this model, writes are only performed on the master, while reads can be done from any replica. The master, acting as the leader, must propagate data changes to the other nodes.
This Master/Slave model is a built-in feature of most relational and non-relational databases.

![[Pasted image 20240813180742.png|600]]

### Synchronous vs Asynchronous Replication:
- **Synchronous Replication**: A successful write response is only returned to the client when all slaves have synchronized with the master. This model ensures that no written data is lost if the leader fails. However, it blocks write requests until the data is reflected on all followers. If a follower encounters an issue during this process, the entire write operation may fail.

- **Asynchronous Replication**: In this model, a successful response is returned to the client as soon as the data is written to the master, and data is then propagated to the slaves in parallel. While this ensures that the system remains operational even if all followers fail, there’s a risk that if the leader fails before syncing with the followers, some written data may be lost, making the write operation non-durable despite the client receiving a confirmation.

Typically, a leader-based model is configured to be fully asynchronous. Although the loss of durability is not ideal, this replication model is widely used, especially when there are many followers or they are geographically dispersed.

![[Pasted image 20240813181004.png|600]]

### Adding a New Follower:
When adding a new follower, which node’s data should be copied to the new follower? To avoid downtime, locking the data is not an option.

Steps for copying the latest version of data to the new follower:
1. Take snapshots of the leader’s data at specific time points.
2. Copy the snapshot to the new follower.
3. The new follower requests any remaining data changes that occurred after the snapshot from the leader.
4. Once the new follower has processed the backlog, it announces that it is ready.

# Handling Node Outages
For various reasons, such as node failure, rebooting nodes during maintenance, or installing kernel security patches, one or more nodes might drop out of the replication set. Availability dictates that in such situations, the system should not completely fail or experience downtime. How is availability managed in leader-based architectures?

If a follower experiences a failure or, for any reason, loses connection with the leader for a while, it can use the data-changes log it previously obtained from the leader and stored on its disk to recover itself. It can then request new changes from the leader that occurred after the failure to update its data.

Leader failure is much more complex. In this situation, one of the followers needs to be selected as the new leader. Additionally, all clients need to be reconfigured to send write operations to the new leader, and all followers must also be reconfigured to receive data-logs from the new leader. This process is done automatically in several stages.

While it cannot be determined with absolute certainty, the nodes continuously exchange messages among themselves, and if a node does not respond for a certain period, it is considered dead.

The new leader is selected using one of two methods:

1. Through an election process where a node is chosen by majority vote.
2. The new leader is selected by a controller node.

The best candidate for becoming the leader is the node with the most up-to-date data to minimize data loss. All nodes must update their routing to point to the new leader. If the previous leader resolves its issue and returns, the system must inform it that it is no longer the leader and should be added back as a follower.

Data loss during the failover process can also cause issues elsewhere. For example, there was an incident on GitHub where a follower from MySQL with outdated data was selected as the new leader. This resulted in data loss for primary key counters, causing duplicate primary keys to be used from a certain point onward. This issue also affected data in Redis, leading to personal information of some users being displayed to other individuals.
A problematic scenario is when, during failover, two nodes simultaneously think they are the leader. This condition is known as "split brain."
This can lead to data loss or corruption. Some systems handle this by shutting down one of the leaders. However, if this solution is not implemented carefully, it might result in both leaders being shut down.
How long should the timeout be to determine if a node is dead?
A long timeout increases the recovery time, while a short timeout can result in unnecessary failovers. Additionally, temporary load on the system or temporary network issues can increase response times.


# Implementation of Replication Logs
As mentioned in Chapter 3, all write and update operations are recorded in the file using the write-ahead method. This can be applied to mechanisms such as SSTables, LSM-Trees, or B-trees. However, the core concept remains the Write-ahead log.

Each update operation is recorded in several copies, each of which only writes and changes one record. This log is called a logical log.

Mechanisms similar to triggers and stored procedures available in some databases can be used to propagate data changes to other nodes.

For example, you can create a trigger that generates a log upon data changes and writes it somewhere else. Another process can then distribute those logs to the other nodes.

However, this method has higher costs and a greater chance of bugs compared to other methods, but it offers high flexibility.

# Problems with Replication Lag

___
# Persian version
رپلیکیشن یعنی داشتن همزمان چندتا نسخه از یک دیتا روی چندتا ماشین که از طریق شبکه در دسترسی هستند.
چرا رپلیکیشن لازم هست:
۱- بخواهیم دیتا رو از نظر جغرافیایی به کاربران نزدیک تر کنیم و latency رو کم کنیم
۲- سیستم رو فالت تولرنت کنیم تا حتی در صورت فیل شدن یک نسخه از دیتا سیستم در دسترسی باشد.
۳- درخواست های خواندن رو بین ماشین ها پخش کنیم و تروپوت سیستم رو زیاد کنیم.
سختی رپلیکیشن به این هست که دیتا مرتب تغییر میکنه و باید نسخه های آن سینک شوند.
برای یکی کردن نسخه های مختلف دیتا، سه متد متداول وجود داره: 
single-leader
multi-leader
leaderless
چالش های این موضوع مربوط میشن به انتخاب رپلیکیشن سینک و آسینک و به علاوه شیوه هندل کردن رپلیکاهای fail شده 
## Leaders and Followers
سولوشن رایج برای سینک دیتا روی رپلیکیشن، مدل مستر/اسلیو یا همان اکتیو/پسیو است. نوشتن فقط روی مستر انجام میشه و خواندن از همه رپیلیکاها میتواند انجام شود. مستر در نقش لیدر باید تغییرات دیتا را روی بقیه نودها ارسال نماید.
![[Pasted image 20240813180742.png|600]]
این مدل مستر/اسلیو ویژگی built-inاغلب دیتابیس های رابطه ای و غیر رابطه ای میباشد.
## Synchronous Versus Asynchronous Replication
یکی شدن دیتای نودهای اسلیو با مستر دو رویکرد متفاوت سینک و آسینک دارد. مدل سینک زمانی هست که وقتی ریسپانس رایت موفق به کلاینت داده میشود که کلیه ایلیو ها با مستر یکی شده باشند. مدل آسینک یعنی به محض رایت شدن دیتا روی مستر پاسخ موفق به کلاینت برمیگردد و موازی با آن دیتا به اسلیوها  هم منتقل میشود.
![[Pasted image 20240813181004.png|600]]

مدل سینک: وقتی داده روی لیدر نوشته میشه، اول روی تمام فالوور ها منعکس میشه بعد ack رایت به کلاینت برمیگرده. 
مزیت این مدل اینه که اگر لیدر مشکلی براش پیش بیاد امکان نداره داده ی رایت شده ای میس بشه
عیب این روش اینه که درخواست های رایت بلاک میشه تا داده روی تمام فالوور ها منعکشس بشه. اون وسط اگر به فالوور دچار مشکل بشه عملیات رایت کلا فیل میشه. 
به همین دلیل که اگر یکی از فالوورها دچار مشکل بشه کل سیستم دیتا از کار میفته پس مدل سینک عملی نیست. معمولا اگر قراره رپلیکیشن از نوع سینک روی دیتابیس فعال بشه مدل اش اینطوری هست که یکی از فالوور سینک و بقیه آسینک خواهند بود. اگر فالور سینک از دسترس خارج بشه یکی دیگه از فالوور ها نقش سینک رو میپذیرن. به این مدل میگن semi -synchronous.
معمولا مدل leader-based از نوع کاملا آسینک کانفیگ میشود. 
عیب این روش اینه که : در این شرایظ اگر لیدر قبل از انعکاس تغییرات روی فالوورها دچار مشکل بشه دیتاهای رایت شده از یه جا به بعد میس میشن. این یعنی تضمینی نداره که عمل رایت durable باشه، با اینکه به کلاینتی که داده را نوشته کانفرم دادیم.
مزیت این روش اینه که حتی اگر همه فالوورها از کار بیفتن باز هم لیدر میتونه به تنهایی درخواست های رایت  کاربرها رو جواب بده.

با وجود اینکه که لطمه خوردن durability خیلی جالب نیست با این حال این مدل رپلیکیشن  خیلی کاربردی هست به خصوص وقتی تعداد فالوورها زیادن یا از نظر جغرافیایی خیلی از هم دورن. 

## Setting Up New Followers
موقع اضافه شدن فالوور جدید دیتای کدوم نود رو باید روی فالوور تازه وارد کپی کنیم؟ چون نمیخواهیم دان-تایم داشته باشیم راه حل لاک کردن دیتا رو انتخاب نمیکنیم.
روش کپی کرپن آخرین نسخه دیتا روی فالوور تازه وارد:
۱- در یه تایم-پوینت های خاصی از دیتای لیدر اسنپ شات میگیرم
۲- کپی کردن اسنپ شات روی فالوور تازه وارد
۳- فالور تازه وارد مابقی تغییرات دیتا که بعد از اسنپ شات رخ داده است را هم از لیدر تقاضا میکند
۴- وقتی فالوور تازه وارد backlog ها رو پردازش کرد به بقیه اعلام میکنه که آمادس

## Handling Node Outages
به هر دلیلی از جمله خرابی نود، ریبوت کردن نودها در فرایند maintenance و نصب پچ های امنیتی کرنل ممکن هست نود یا نودهایی از مجموعه replication خارج بشن. availability میگه که در این شرایط سیستم سیستم نباید کلا از کار بیفته و نباید دان-تایم داشته باشیم. availability در معماری های leader-based چگونه انجام میشه؟
### Follower failure: Catch-up recovery
اگر یه فلوور دچار failure بشه و یا به هر دلیلی یه مدتی ارتباط اش با لیدر قطع بشه برای ریکاور کردن خودش میتونه از لاگ مربوط به data-changes که قبلا از لیدر گرفته و روی دیسک خودش نگه داری کرده بوده استفاده بکنه. تغییرات جدید که از زمان failure به بعد روی لیدر درج شده را هم از لیدر درخواست بدهد و به این صورت دیتای خود را به روز نماید.
### Leader failure: Failover
فیل شدن لیدر خیلی پیچیده تره. در این شرایط باید یکی از فالوورها به عنوان لیدر انتخاب بشه. به علاوه تمام کلاینت ها باید مجددا کانفیگ بشن که ازین به بعد عملیات رایت رو روی لیدر جدید ارسال کنند و کلیه فالوورها هم باید مجددا کانفیگ بشن تا دیتا-لاگ ها رو از این به بعد از لیدر جدید دریافت کنند. این پروسه به صورت اوتوماتیک در چند مرحله انجام میشه.
این موضوع رو با قاطعیت که نمیشه تشخیص داد ولی برای تشخیص اش تمام نودها مرتب بین خودشون مسیج رد و بدل میکنن و اگر هر نودی یه مدت پاسخ نده، مرده به حساب میاد.
انتخاب لیدر جدید به یکی از دو روش: 
۱- طی پروسه election یک نود با اکثریت آرا
۲ - لیدر جدید توسط نود کنترلر انتخاب میشه
بهترین کاندید برای لیدر شدن نودی هست که به روز ترین دیتا را دارد تا کمتری دیتالاس اتفاق بیفته
همه باید مسیریابی شون رو به سمت لیدر جدید تغییر بدن. لیدر قبلی هم هروقت مشکل اش حل شد و برگشت باید سیستم اونو مطملع کنه که دیگه لیدر نیست و باید در قالب فالوور به سیستم اضافه بشه.
اگر مدل آسینک داریم استفاده میکنیم به احتمال زیاد لیدر جدید دیتای به روزی نداره. و در فرایند Failover یه سری از رایت ها عملا از بین میرن.  مشکل دیگری که اینجا ممکنه پیش بیاد اینه که لیدر قبلی دوباره به سیستم برگرده (مشکل دقیقا کجاست؟ اینکه همزمان دوتا لیدر داریم؟ چطوری بهش بگیم که تو دیگه لیدر نیستی؟ تغییراتی که فقط خودش ازشون خبر داشته و به کسی اطلاع نداده بود رو چیکار کنیم؟). راه حل اینه که ایشون باید تغییراتی که به کلاستر اعلام نکرده را discard کند. این هم باعث میشه durability سیستم میره زیر سوال.
پریدن برخی دیتاها در فرایند failover یه جای دیگه هم مشکل ساز میشه. جایی که اثر دیتا توی سایر دیتاسیستم ها هم منعکس میشه. مثلا یه اینسیدنتی تو گیت-هاب رخ داد، بکی از فالوورهایی از mySQL که خیلی دیتاش قدیمی بود به عنوان لیدر جدید انتخاب شد. دیتای کانترهای مربوط به primary keys پریدن. این موضوع باعث شد از یه جا به بعد  primary keys تکراری استفاده بشن. این primary keys در ردیس هم نشسته بودن. این موضوع باعث شد اطلاعات شخصی برخی کاربرها به افراد دیگری نمایش داده بشه.
یک سناریوی مشکل دار اینه که در پروسه failover دوتا نود همزمان فکر میکنند که لیدر هستند. به این شرایط میگن split brain.
ممکنه دیتا lost  یا خراب بشه. برخی سیستمها در این شرایط یکی از لیدرها رو shutdown میکنند. همین راه حل اگر با دقت انجام نشه ممکنه هر دوتا لیدر shutdown بشن.
تایم اوت درست چقدر باشه تا بگیم یه نود مرده است؟
تایم اوت زیاد مدت زمان ریکاوری رو طولانی تر میکنه. 
تام اوت کم تعداد failovers. ها رو الکی زیاد میکنه.
از طرفی لود موقتی روی سیستم یا مشکلات موقتی نتوورکیresponse-time ها  رو زیاد میکنه.

# Implementation of Replication Logs
### Statement-based replication
کلیه عملیات INSERT, UPDATE و DELETE در قالب SQL statement روی فالوور ها منعکس میشه.
فانکشن هایی مثل NOW و RAND اگر در SQL statement استفاده بشه مقادیر مختلف روی replicaها مختلف خواهد نشست.
اگر در کوئری SQL یک ستوی autoincrementing داره آپدیت میشه order های مختلف روی رپلیکاهای مختلف باعث تولید دیتای متفاوت روی اونها میشه.
اگر کوئری SQL یه side-effect ای داره مثل تریگر شدن یه چابی، stored procedures یا هر چیزی، ممکنه روی رپلیکاهای متفاوت اثرات جانبی مختلفی رو در پی داشته باشه.
راه حل اینکه همه این موارد لیدر کلیه این فانکشن های غیر قطعی رو اجرا کنه و مقادیر مشخص جایگزین شون بکنه تو کوئری. بعد بفرسته به فالوور ها.
### Write-ahead log (WAL) shipping
به همون روشی که تو فصل ۳ گفته شد، تمام عملیات رایت و آپدیت داخل فایل به روش write-ahead نوشته میشه. حالا ممکنه روی ان مکانیزم هایی مثل SSTables یا LSM-Trees یا B-tree هم اعمال بشه. ولی کلیت قضیه همون Write-ahead log هست.
### Logical (row-based) log replication
هر کوپری آپدیت میکشنه به چندین کوپری که هر کدوم فقط یه رکورد رو انتخام مینه و اونو تغییر میده. به این لاگ میگن logical log .
### Trigger-based replication
از مکانیزم هایی شبیه riggers و stored procedures که روی برخی دیتابیس ها وجود دارند هم میشه برای انعکلاس تغییرات دیتا روی نودهای دیگه استفاده کرد.

میشه یه تریگر نوشت که به محض تغییر یک دیتا یه لاگی تولید کنه و یه جای دیگه بنوسه. یه پراسس دیگه هم بذاریم که اون لاگ ها رو به اطلاع بقیه نودها برسونه.

البته هزینه و احتمال باگ در این روش بیشتر از بقیه روش ها است. اما خب flexibility بالایی داره.

# Problems with Replication Lag
قبلا هم گفتیم که چرا دنبال replication هستیم! دلایلش در این پاراگراف هایلایت شدن!
اگر در یک اپلیکیشن تعداد خواندن ها زیاد و درصدی کمی امکان تغییر دیتا وجود داره روش تک لیدری خوب جواب میده. چون فقط یک لیدر داریم که به تنهایی نوشتن ها رو هندل میکنه.
معماری read-scaling یعنی اینکه اگر خواندن ها زیاد شد فالوور ها رو زیاد میکنیم. 
گفیم که کلا هم بیخیال روش سنکرون میشیم چون نوشتن ها رو خیلی کند میکنه و اگر یه فالوور از کار بیفته کاربر نمیتونه  اوکی بگیره برای نوشتن هاش.
در مدل تک لیدری آسنکرون هم ممکنه کاربر دیتای کهنه بخونه از یک فالووری که هنوز دیتای به روز بهش نرسیده! درحالی که اگر اون دیتا رو از روی لیدر بخونه جواب متفاوتی میگیره. 

اگر کمی صبر کنه تا تغییرات روی همه فالورها منعکس بشه مشکل حل میشه. یعنی اینجا مساله eventual consistency  داریم.

مساله اصلی اینه چقدر باید صبر کنیم تا مطمئن بشیم consistency برقرار شده!
پس یه تاخیری داریم بین زمانی که داده رو کاربر روی لیدر مینویسه تا اینکه اون تغییر روی همه فالوورها منعکس بشه. ظاهرا به این تاخیر میگن replication-lag .
اگر لگ زیاد باشه دیگه تبدیل میشه به یک مشکل اساسی. در ادامه چندتا سناریوی مشکل دار رو با هم میبینیم.

# Reading Your Own Writes
مساله replication lag ممکنه Reading Your Own Writes رو مختل کنه. کاربر یه چیزی رو روی پروفایلش تغییر بده و سیو کنه. چند دیقه بعد رفرش کنه و اون تغییر رو نبینه. چون درخواست خوندن رسیده به فالووری که نسبت به لیدر lag داره.
![[Pasted image 20240821202639.png|600]]

چیزی که تو این شرایط لازم داریم read-after-write consistency هست. ممکنه تغییر یک کاربر رو کاربران دیگه نبینه تا یه مدتی، ولی خود اون کاربر باید بتونه ببینه.

چیکار کنیم این اتفاق بیفته؟ در ادامه ...

اگر کاربری داره دیتایی رو میخونه که ممکنه خودش تغییر داده باشه، درخواست خوندنش برسه به دست لیدر. در غیر این صورت برسه به دست فالوورها.

برای این کار باید بتونی تشخیص بدی کیا ممکنه در ارتباط با چه دیتایی تغییر دهنده باشن!

یه مثال ساده اطلاعات پروفایل هست که حتما توسط خود کاربرها تغییر میکنه.
در مورد دیتایی که ممکنه توسط هر کاربری تغییر کنن روش قبلی کارساز نیست.

بعد از هر بار که یک آپدیت به سمت لیدر ارسال میشه تا یک مدت زمان بعدش مثلا تا یک دقیقه کلیه درخواست های خواندن به سمت لیدر ارسال بشن.

یا حتی میشه لگ روی فالوورهای مختلف مانیتور بشه. فالوورهایی که لپ شون بیشتر از یک دقیقه هستن دیگه روشون کوئری نزنیم.


کلاینت timestamp آخرین آپدیت خودشو یادش نگه داره. اینطوری میشه مشکل لپ رو حل کرد که وقتی کلاینت یه دیتا رو خوند از روی یه نود، timestamp داده خوانده شده رو با timestamp ای که یادش نگه داشته مقایسه میکنه تا بدونه اون نود ایا لگ داره یا نه.

اگر نودها دیتا روی دیتاسنتر های مختلف پخش باشن یه معضل دیگه هم اینه که وقتی کلاینت به این نتیجه رسید که باید داده ای رو از لیدر بخونه نه از فالوور ها، تازه باید تشخیص یده که لیدر دقیقا روی کدوم دیتاسنتر قرارداره.
اگر کاربر دیتای خودش رو همزمان از روی چندتا دیوایس (مثلا یک موبایل و یک سیستم دیگه با یک یوزر و چند کلاینت) بخواد ببینه که دیگه هیچی!! مشکل بزرگتر میشه : cross-device read-after-write consistency
اینجوری اصلا کلاینت هم سخت میتونه یادش نگه داره که آخرین تایمی که یه دیتا تغییر کرده کی بوده. اون timestamp روی یه دیواس یه چیزه روی دیوایس دیگه یه چیز دیگس.
اگر نودهای دیتامون روی چندین دیتاسنتر پخش هستن و در عین حال کاربر همزمان داره از چندتا دیواس اطلاعاتی که خودش تغییر داده رو مشاهده میکنه دوباره مشکل پیچیده تر میشه. نمیشه تضمین کرد درخواست های خواندن هر دیوایس روی همان دیتاسنتر هدایت بشه.

اگر رویکردمون این هست که بعد از به نوشتن تا یه مدت از لیدر دیتامون رو بخونیم اونوقت تو این سناریو همه دیوایس های کاربر تا یه مدت بعد از رایت کردن باید از یه دیتا سنتر (حاوی لیدر) دیتاشون رو بخونن. 

# Monotonic Reads
فقط که مشکل اوناج نیست که کاربر میخواد دیتایی که خودش نوشته رو بلافاصله بخونه. فرض کنید کاربر اول یه دیتایی رو نوشته. کاربر دوم میخاود اونو بخونه. اگر یکبار درخواست کنه و دیتای به روز دریافت کنه طوری نیست. اگر درخواست خواندنش رو چندبار تکرار کنه و هربار جواب متفاوت از فالوور متفاوت بگیرم. اول تغییرات جدید دیتا رو ببینه. رفرش کنه بعدش تغییرات رو دیگه نبینه. دوباره رفرش کنه تغییرات رو ببینه :D 

اصطلاحا دچار moving backward in time اتفاق میفته.

در واقع Monotonic Reads  راه حلی برای همین آنومالی هست.

یک راه برای برقراری Monotonic Reads اینه که هر کاربری درخواست های خواندن اش همیشه به یک نود ثابت برسه.

مثلا نود از روی نام کاربری انتخاب بشه نه همینطوری رندوم.



![[Pasted image 20240821202810.png|600]]

# Consistent Prefix Reads

اینجا در مورد یه آنومالی دیگه داره صحبت میکنه:
اینکه دو نفر با هم مکالمه میکنند. نفر سوم جملات مکالمه این دو رو داره پس و پیش میشنوه.

در واقع Consistent Prefix Reads میگه نباید این آنومالی رخ بده.


اگر دیتاهایی که ادامه همدیگه هستن باید در یک پارتیشن درج بشن تا این مشکل پیش نیاد.

![[Pasted image 20240821202840.png|600]]

# Solutions for Replication Lag
در سیستم های eventually consistent باید دید چقدر Replication Lag کار خراب کن هست؟

هندل کردنش سمت اپلیکیشن خیلی پیچیدگی داره. بهتره سمت دیتابیس انجام بشه.

میشه گفت transaction ها یه راه حل برای این مساله هستن. ولی فقط سینگل نود هستیم transaction ها کمک کننده هستن. 

در مورد دیتاسورس های توزیع شده transaction ها  مزاحم سرسخت پرفورمنس و availability, هستن. در این سیستم ها باید eventual consistency را پذیرفت.

# Multi-Leader Replication
در مدل چند لیدری، چندتا لیدر درخواست رایت را به طور موازی میپذیرند. هر لیدر بعد از نوشتن داده، اون تغییر رو روی کلیه نودها منعکس میکند.

# Use Cases for Multi-Leader Replication
### Multi-datacenter operation
مدل چند لیدری:
در هر دیتاسنتر یه لیدر داریم با یه تعداد فالوور. بین دیتاسنترها لیدرها باهم تغییرات دیتا رو رد و بدل میکنند. هر لیدر هم فالوورهای داخل اون دیتاسنتر رو از تغییرات دیتا باخبر میکنه
![[Pasted image 20240821202941.png|600]]

در مدل چند لیدری عمل رایت کردن دیتا هم سرعت میگیره. چون درخواستش به نزدیک ترین دیتاسنتر ارسال میشه. حتی انعکاس تغییرات روی فالوور ها هم سرعت بهتری داره. چون داخل نتوورک یک دیتاسنتر داره انجام میشه.
در مدل چند لیدری اگر یک دیتاسنتر دچار مشکل بشه، دیگه هزینه هایی که در مدل تک لیدری میدادیم رو نمیدیم مثلا leader election رو دیگه نداریم.
ارتباط بین دیتاسنترها در شبکه عمومی انجام میشه که قابل اعتماد نیست. مشکلاتی که شبکه عمومی ایجاد میکنه مدل تک لیدری رو بیشتر مختل میکنه. چونعملیات رایت که به صورت سنکرون باید انجام بشن محبورن روی این نتوورک پابلیک رد و بدل بشن.
یکی از مشکلات مدل چند لیدری نوشتن همزمان یک دیتا در چند دیتاسنتر هست که نیاز به conflict resolution دارد.

### Clients with offline operation
یکی از شرایطی که مدل چند لیدری کمک کننده هست اینه که بخواهیم کلاینت ها حتی در شرایطی که اینترنت ندارند هم بتوانند با اپلیکیشن کار کنند. مثل کلندر که روی گوشی ما هست. در این مدل هر گوشی یه دیتابیس همراه خودش داره که یه لیدر حساب میشه. کلندری که ما روی دیوایس دیگر خودمون داریم هم لیدر خودشو داره.

این سناریو همون مدل مولتی لیدر هست که دیوایس های ما حکم همون دیتاسنترها رو دارند.
### Collaborative editing

ابزارهایی شبیه google-Doc که امکان Real-time collaborative editing ر. فراهم میکنند هم دقیقا شبیه سناریوی قبل هستند. همون کلندر که ما روی دیوایس های مختلف مون نصب میکنیم. هر تغییری که هر کاربر در داکیومنت ایجاد میکنه سریعا روی نسخه بقیه کاربرها منعکس میشه.
 در این مدل ایپلیکیشن ها برای جلوگیری از کانفلیکت میشه از روش لاک کردن استفاده کرد. وقتی یوزر میخاد اقدام به تغییر داکیومنت بکنه اول باید اونو لاک کنه. تا تغییرات اش رو کامیت نکنه بقیه کاربرها نیمتونن اونو تغییر بدن. وقتی لاک کردن میاد وسط میشه معادل تک لیدری.
 برای بهبود همروندی میشه اندازه تغییرات هر کاربر رو انقدر کوچک بگیریم مثلا هر تغییر در حد یک کلید از کیبورد. به علاوه لاک هم نمیذاریم. با این تغییرات دوباره میشیم شبیه مدل چند لیدری و چالش conflict resolution میاد وسط.
# Handling Write Conflicts
یک مثال از مشکل conflict در مدل چند لیدری. 
مثلا همزمان روی یک سند گوگل داک دو نفر دارن کار میکنن. 


![[Pasted image 20240821203132.png|600]]

### Synchronous versus asynchronous conflict detection
در مدل تک لیدری نوشتن های همزمان روی یک دیتا با لاک هندل میشه و کانفلیکت اتفاق نمی افته. اما در مدل چند لیدری ممکنه یه موقع هایی یه کانفلیکت هایی تشخیص داده بشه ولی دیگه برای ریزالور کردنش دیر باشه.
میشه کانفلیکت رو به صورت سینک هم تشخیص داد. اینطوری که هنوز به کاربری که داده را مینویسد، اوکی نداده دیتا را روی بقیه لیدر ها منعکس کنیم. اگر کانفلیکت داشت به یوزر اوکی ندیم.
با این راه حل یکی از مزیت های  مدل چند لیدری رو از دست میدیم اونم بهبود پرفورمنس عملیات نوشتن هست.

### Conflict avoidance
حل مشکل کانفلیکت ها به روش پیش گیری: هر رکورد مشخص فقط روی یک لیدر قابلیت نوشتنو تغییر کردن داشته باشه.
مثلا دیتاهای هر کاربر روی دیتاسنتری که از نظر جغرافیایی بهش نزدیک تره فقط میتونه رایت بشه. اینجوری از منظر هر کاربر مدل مون تک لیدری هست.
اینطوری اگر یک دیتاسنتر از کار بیفته یا مثلا لوکیشن کاربر تغییر بکنه لازمه لیدر مربوط به رایت های اون کاربر تغییر کنه. در این شرایط دوباره خطر کانفیلک ها بالا میره.

### Converging toward a consistent state
حل مشکل کانفلیکت ها به روش پیش گیری: هر رکورد مشخص فقط روی یک لیدر قابلیت نوشتنو تغییر کردن داشته باشه.
مثلا دیتاهای هر کاربر روی دیتاسنتری که از نظر جغرافیایی بهش نزدیک تره فقط میتونه رایت بشه. اینجوری از منظر هر کاربر مدل مون تک لیدری هست.
اینطوری اگر یک دیتاسنتر از کار بیفته یا مثلا لوکیشن کاربر تغییر بکنه لازمه لیدر مربوط به رایت های اون کاربر تغییر کنه. در این شرایط دوباره خطر کانفیلک ها بالا میره.
### Converging toward a consistent state
در مدل تک لیدری تمام کوپری های آپدیت کردنی به ترتیب و تولی روی دیتا اعمل میشن.
اما در روش چند لیدری، هر لیدر تغییرات دیتا رو ممکنه در ترتیب متفاوت نسبت به لیدر دیگه دریافت کنه. اینجوری سیستم به یک وضعیت نامتناقض میرسه (inconsistent)
چون تقدم و تاخر رایت ها در مدل چند لیدری مشخص نیست میان به هر درخواست رایت یه یونیک آیدی اختصاص میدن به منزله timestamp یا UUID که بشه به اون درخواستی که بزرگترین یونیک آیدی رو داره اصالت داد به عنوان winner. 
اگر از timestamp استفاده بشه که به این روش میگن last write wins (LWW). روش رایجی هست ولی خطر data-loss وجود داره.
به هر رپلیکا یه یونیک آیدی بدیم و رایت هایی که روی رپلیکای با بالاترین آیدی نوشته شدن رو به عنوان winner انتخاب کنیم. این روش هم با خطر data loss مواجه هست.
تغییرات همزمان رو با هم merge کنیم. مثلا در مثال قبل عنوان داکیومنت بشود B/C
کانفلیکت ها در یک جایی ثبت بشن تا بعدا ریزالو بشن. مثلا با کمک کاربر مربوطه.

### Custom conflict resolution logic
متناسب با منطق اپلیکیشن میشه روش خاصی برای ریزالور کردن کانفلیکت ها انتخاب و برنامه آن را بنویسیم. حتی میشه متناسب با مواقع رایت و رید منطق متفاوتی برای ریزالو کردن کانفلیکت انتخاب کرد.
مثلا دیتابیس Bucardo بهت اجازه میده که با زبان Perl یه تیکه کد بنویسی و بگی هرزمان موقع نوشتن کانفلیکتی تشخیص داده شد این برنامه در بک گراند اجاره بشه و کانفیلیکت رو ریزالو کنه.
یا مثلا یه کدی نوشته بشه که هروقت موقع خوندن داده ای کانفلیکتی شناسایی بشه مقادیر مختلف شناسایی شده به اون تیکه کد پاس داده بشه تا اون کد در مورد دیتای ولید تصمیم گیری بکنه.

### What is a conflict?
گاهی تشخیص کانفلیکت خیلی هم کار راحتی نیست.
منظور شیوه اتصال لیدرها به همدیگه برای propagate کردن رایت ها هستش

# Multi-Leader Replication Topologies
لیدرها در قالبی شبیه درخت با هم سینک میشن
هر لیدر آپدیت ها رو فقط از یک لیدر تحویل میگیره. و همه آپدیت ها رو فقط به یک لیدر اطلاع میده.


![[Pasted image 20240821203437.png|600]]


امکان داره تغییرات به طور بی نهایت در مدل حلقه ای و درختی بچرخه. برای جلوگیری از آن میشه به هر لیدر یه کد یونیک داد و هر عمل رایت ای مشخص باشه که توسط چه لیدری ثبت شده. اگر لیدری به آپدیتی رسید که با کد خودش ثبت شده اونو ایگنور کنه.
یکی از مشکلات مدل حلقه ای و درختی اینه که اگر یکی از لیدرها از کار بیفته سیستم دچار اختلال میشه.
در مدل توپولوژی all-to-all هم یه مشکلاتی وجود داره. مثلا برخی ارتباطها سرعت بهتر دارن. اینطوری باعث میشه مسیج های برخی رپلیکاها از بقیه سبقت بگیره و ترتیب به هم بخوره

در شکل زیر نوشتن ها غیر همزمان هستن ولی اثر دومی سریعتر از نوشتن اولی به لیدر رسیده

![[Pasted image 20240821203530.png|600]]

برای حل این مشکل تقدم و تاخیر اختصاص دادن timestamp به مسیج های آپدیت کافی نیست. چون لزوما کلاک ماشین های مختلف یا هم یکسان نیست. 

راه حل پیشنهادی version vectors نام دارد:

# Leaderless Replication
برخی دیتابیس اه رویکری متفاوت به نام leaderless رو انتخاب کردن.
In some leaderless implementations, the client directly sends its writes to several rep‐
licas, while in others, a coordinator node does this on behalf of the client.

# Writing to the Database When a Node Is Down
![[Pasted image 20240821203626.png|600]]

Version numbers are used to determine
which value is newer

دیتااستورهای Dynamo-style به اونایی میگه که یه مکانیزیمی دارن که دیتاهای out-of-date رو شناسایی کنند و اصلاح شون کنن. 
حالا یا با روش read-repair
وقتی چندتا خوندن موازی انجام میشه اگر در جواب ها دیتای یکی از رپلیکاها قدیمی باشه همونجا اصلاح میشه
یا با روش Anti-entropy process
یه پراسس در بکگراند همواره داره اجرا میشه و دنبال دیتاهای قدیمی در بین یپلیکاها میگرده و اصلاح میکنه

### Quorums for reading and writing
چند نفر از کل replica ها در جواب نوشتن اوکی بدن کافیه؟
در مدل بدون-لیدر درخواست خواند و نوشتن همواره به صورت موازی به تمام رپلیکاها ارسال میشه. و تعبیر کتاب از حروف n-w-r اینه:
n:
تعداد کل نودهای رپلیکا (اعم از سالم و ناسالم)
w:
 تعداد اوکی هایی که لازمه موقع نوشتن از سیستم بگیریم تا بتونیم فرض کنیم کارمون انجام شده
r:
تعداد خواندن های صحیحی که موقع خوندن از تک تک نودها باید اتفاق بیفته تا در مجموع داده صحیح گیرمون بیاد


![[Pasted image 20240821203710.png|600]]

# Detecting Concurrent Writes
دیتابیس های مدل Dynamo-style ( همونا که مکانیزیم هایی دارن که دیتاهای درژن قدیمی رو شناسایی و اصلاح کنند)‌اجازه میدن چندتا کلاینت همزمان و به طور موازی درخواست های رایت روی یک داده داشته باشن. اینجوری حتی اگر کوآروم های خیلی دقیقی هم انتخاب کنیم باز هم امکان conflict وجود داره. بالاخره هر لینکی از شبکه تاخیر متفاوتی ممکنه داشته باشه یا یه بخشی از شبکه ممکنه اختلال داشته باشه

توجه به سناریوی عکس  - نوشتن کاملا همزمان

![[Pasted image 20240821203746.png|600]]

اگر هر لیدر هر آپدیتی رو به محض دریافت اپلای کنه حتما خرابی دیتا خواهیم داشت
انتخاب راه حل برای مساله
آخرین آپدیت رو فقط قبول کنیم و بقیه رو رد کنیم.  ولی از کجا بفهمیم آخرین آپدیت کدوم هست؟
اگر برای تشخیص آخرین آپدیت از روش timestamp استفاده کنیم اسم راه حل مون last write wins (LWW) نامیده میشه
مشکلات روش last write wins:
کاربر میاد روی n تا لیدر رایت میکنه. از w تا لیدر اوکی میگیره. اما نهایتا فقط یکی از اون نوشتن ها ثبت میشه و بقیه discard میشن. کلاینت هم از این قضیه خبر نداره.
این روش حتی ممکنه نوشتن های غیر همزمان رو هم discard کنه
تنها وقتی میشه به روش last write wins اعتماد کرد که کلیدها رو immutable انتخاب کنیم. مثلا کاساندرا از UUID به عنوان کلید استفاده میکنه. یونیک هست و هیچ وقت هم قرار نیست تغییر کنه.

چی شد؟ مگه در عملیات  x = 1  کلیدمون ایکس نبود؟؟ یعنی دیگه ایکس نباشه و یه ussd باشه؟؟؟؟
 از کجا بفهمیم دو تا عمل آپدیت واقعا همزمان بودن؟
 کلا تو این چندتا پاراگرافت داره توضیح میده دنبال این هستیم که همزمانی واقعی رو تشخیص بدیم.
تو این مساله تایم اصلا کمک نمیکنه چون سیستم توزیع شده است و کلاک ها لزوما عین هم نیستن.

تنها در صورتی میشه گفت دوتا عمل راست واقعا یکسان بودن که هر دو دقیقا هر دو تاش موقع رخدادن از اونیکی بی خبر باشه.

# Capturing the happens-before relationship

اینجا داره سعی میکنه سناریوی شکل ۵-۱۳ رو یه جوری توضیح بده که نوشتن دوتا کاربر روی یک دیتا داره رو قراره خراب کنه. تو این سناریو دیتابیس اصلا قرار نیست مقدارهای چندتا کلاینت رو مرج کنه. هرچی میدن میگیره set میکنه

![[Pasted image 20240821203913.png|600]]

![[Pasted image 20240821203926.png|600]]

اینجا اشاره میکنه به روش  versioning که باهاش مشکل حل میشه

# Merging concurrently written values
### Version vectors
اگر در سناریوی شکل 5-13 چندتا لیدر داشتیم! اونوقت آیا یه ورژن کفایت میکنه؟
در مدلی که چندتا رپلیکا داریم باید باید هر کلید، به ازای هر رپلیکا یه ورژن نگه داریم. این کالکشن از ورژن ها رو میگن version vectors .
تو این مدل وقتی دیتایی توسط کلاینت خونده میشه اون vector هم همراه دیتا میاد. موقع نوشتن هم اون vector همراه دیتا میرسه دست نودها.
