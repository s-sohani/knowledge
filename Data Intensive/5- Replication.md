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

