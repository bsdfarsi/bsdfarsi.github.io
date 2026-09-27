<div class="lang-fa" markdown="1">

# تحلیل فلسفی و حقوقی: تقابل لایسنس BSD و GPL

در پارادایم متن‌باز، مفهوم «آزادی» یک واژه واحد با دو تفسیر کاملاً متضاد است. این تضاد فلسفی و حقوقی، خود را در قالب دو لایسنس بومی دنیای سیستم‌عامل یعنی BSD (Permissive) و GPL (Copyleft) نشان می‌دهد. درک این تفاوت برای مهندسین سیستم و معماران زیرساخت پیش از انتخاب چیدمان ابزارها حیاتی است.

### فلسفه آزادی: آزادی کاربر در برابر آزادی نرم‌افزار

بزرگ‌ترین مرز جدایی این دو مجوز، تعریف آن‌ها از مرزهای حقوقی توسعه است:

* **مجوز بی‌اس‌دی (آزادی مطلق کاربر):** فلسفه برکلی بر این باور است که اگر نرم‌افزار آزاد است، کاربر باید حق داشته باشد هر کاری خواست با آن انجام دهد؛ حتی متن‌بسته کردن آن! لایسنس BSD عملاً هیچ محدودیتی اعمال نمی‌کند. شما می‌توانید سورس‌کد را بردارید، تغییر دهید، آن را با کدهای انحصاری (Proprietary) خود ترکیب کنید و محصول را بدون انتشار سورس‌کد جدید بفروشید. تنها شرط، حفظ نام پدیدآورنده اولیه است.
* **مجوز جی‌پی‌ال (آزادی مشروط نرم‌افزار):** فلسفه ریچارد استالمن و FSF معتقد است نرم‌افزار باید برای همیشه آزاد بماند، حتی به قیمت محدود کردن آزادی کاربر. GPL یک مجوز **Copyleft** یا اصطلاحاً «ویروسی» است. طبق قانون GPL، اگر شما از کدی با این لایسنس در پروژه خود استفاده کنید، کل پروژه نهایی شما تبدیل به یک اثر مشتق‌شده (Derivative Work) می‌شود و **مجبورید** سورس‌کد نهایی خود را نیز تحت لایسنس GPL به صورت عمومی منتشر کنید.

</div>

<div class="lang-en" markdown="1">

# Philosophical and Legal Analysis: BSD vs GPL License

Within open-source computing, the term "freedom" carries two fundamentally divergent interpretations. This philosophical and legal divide is manifested through two historic operating system license traditions: BSD (Permissive) and GPL (Copyleft). Grasping this distinction is critical for systems engineers and infrastructure architects when selecting tech stacks.

### The Philosophy of Freedom: User Freedom vs Software Freedom

The central point of divergence lies in how each license defines legal boundaries for downstream software distribution:

* **BSD License (Unrestricted User Freedom):** The Berkeley philosophy posits that if software is truly free, the user must possess absolute liberty to do whatever they choose with it—including incorporating it into closed-source, commercial products. The BSD license imposes virtually zero restrictions. You may inspect, modify, fork, and combine the source with proprietary codebases and distribute binary products without releasing modifications. The primary requirement is retaining the original copyright notice and disclaimer.
* **GPL License (Conditional Software Freedom):** Founded on the philosophy of Richard Stallman and the Free Software Foundation (FSF), the GPL dictates that software must remain perpetually free, even if that necessitates restricting user liberties. The GPL operates as a **Copyleft** (reciprocal) license. By legal design, if you link or incorporate GPL-licensed code into your application, the entire resultant work is classified as a derivative work—requiring you to publish the complete source code under the same GPL license upon distribution.

</div>