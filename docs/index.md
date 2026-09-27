<div class="lang-fa" markdown="1">

# راه‌اندازی و استقرار NetBSD بر بستر QEMU/KVM

سیستم‌عامل NetBSD به پرتابل بودن (Portability)، طراحی ماژولار و کدهای به شدت تمیز شهرت دارد؛ شعار معروف *"Of course it runs NetBSD"* گواه اجرای آن روی ده‌ها معماری سخت‌افزاری مختلف است. اما در محیط‌های زیرساخت و مجازی‌سازی مدرن لینوکس (بر پایه KVM)، اجرای بهینه و با راندمان بالای NetBSD نیازمند پیکربندی درست شتاب‌دهنده سخت‌افزاری، درایورهای پرسرعت Paravirtualization (مجموعه VirtIO) و مدیریت صحیح کنسول سریال یا گرافیکی است.

در این مستند فنی، مراحل راه‌اندازی NetBSD 10.x روی QEMU/KVM را از ایجاد دیسک و تعریف سوییچ‌های عملکردی تا مراحل نصب و تنظیمات پایه بررسی می‌کنیم.

<img src="images/netbsd.png" alt="NetBSD QEMU Installation" style="width: 100%; max-width: 800px;">

> ### ۱. آماده‌سازی ایمیج دیسک با فرمت qcow2

برای کارایی بهتر در لایه استوریج و بهره‌گیری از قابلیت Thin Provisioning، یک دیسک با فرمت `qcow2` می‌سازیم:

```bash
qemu-img create -f qcow2 netbsd.qcow2 20G
```

<br>

> ### ۲. اجرای ماشین مجازی با سوییچ‌های بهینه‌سازی (KVM + VirtIO)

برای اینکه بیشترین بازدهی I/O و CPU حاصل شود، از درایورهای پاراویرچوال `virtio-net` و `virtio-blk` استفاده می‌کنیم که در کرنل‌های مدرن NetBSD به صورت Native پشتیبانی می‌شوند:

```bash
qemu-system-x86_64 \
    -enable-kvm \
    -m 2048 \
    -smp 2 \
    -cpu host \
    -drive file=netbsd.qcow2,if=virtio,format=qcow2,cache=none \
    -cdrom NetBSD-10.0-amd64.iso \
    -boot d \
    -netdev user,id=net0,hostfwd=tcp::2222-:22 \
    -device virtio-net-pci,netdev=net0 \
    -display default
```

**نکات مهم معماری در این فرمان:**
* **`-cpu host` و `-enable-kvm`:** دسترسی مستقیم به Instruction Setهای پردازنده فیزیکی بدون لایه شبیه‌سازی نرم‌افزاری.
* **`if=virtio` و `virtio-net-pci`:** حذف سربار درایورهای شبیه‌سازی‌شده IDE یا e1000 و ارتباط با باس پرسرعت VirtIO.
* **`hostfwd=tcp::2222-:22`:** مپ کردن پورت SSH ماشین مهمان به پورت ۲۲۲۲ هاست جهت دسترسی ریموت بدون نیاز به بریج شبکه.

<br>

> ### ۳. مراحل نصب از طریق نصاب sysinst

پس از بوت شدن ایمیج، وارد محیط کاربری مبتنی بر کاراکتر (Curses-based) نصاب بومی نت‌بی‌اس یعنی `sysinst` می‌شوید:

1. زبان را روی `a: Installation in English` انتخاب کنید.
2. گزینه `a: Install NetBSD to hard disk` را انتخاب نمایید.
3. درایور دیسک شناسایی‌شده را تایید کنید (به عنوان مثال `ld0` که دیسک VirtIO است).
4. نوع پارتیشن‌بندی دیسک را بر اساس نیاز انتخاب کنید (برای ماشین‌های استاندارد BIOS/MBR گزینه `MBR partitions` یا برای سیستم‌های مبتنی بر UEFI گزینه `GPT`).
5. فایل‌سیستم را روی FFSv2 با پشتیبانی لاگینگ (`WAPBL`) قرار دهید تا در قطعی‌های ناگهانی برق نیازی به اسکن کامل دیسک نباشد.
6. مجموعه بسته‌های پیش‌فرض (`Full installation`) را انتخاب کرده و اجازه دهید استخراج بسته‌ها از روی مدیا انجام گیرد.

<br>

> ### ۴. تنظیمات پس از نصب و آماده‌سازی SSH

پیش از ریبوت، چند گام اساسی در منوی پیکربندی نهایی نصاب پیشنهاد می‌شود:

* **تنظیم گذرواژه ریشه (root password):** یک پسورد امن برای کاربر root تعیین کنید.
* **ایجاد کاربر عملیاتی:** ساخت یک کاربر معمولی و عضویت آن در گروه `wheel` برای انجام امور مدیریتی.
* **فعال‌سازی سرویس sshd:** در صورتی که در مرحله نصب فعال نشد، پس از اولین بوت در `/etc/rc.conf` خط زیر را اضافه کنید:

```sh
sshd=YES
```

همچنین کارت شبکه به عنوان `vioif0` شناخته می‌شود. برای دریافت خودکار IP از سرویس DHCP در `/etc/rc.conf` مقدار زیر را اعمال نمایید:

```sh
ifconfig_vioif0="dhcp"
```

<br>

> ### ۵. بوت نهایی بدون مدیای نصب

پس از اتمام مراحل، ماشین را خاموش کرده و برای بوت‌های دائمی، مدیا را جدا کرده و سیستم را بالا بیاورید:

```bash
qemu-system-x86_64 \
    -enable-kvm \
    -m 2048 \
    -smp 2 \
    -cpu host \
    -drive file=netbsd.qcow2,if=virtio,format=qcow2,cache=none \
    -netdev user,id=net0,hostfwd=tcp::2222-:22 \
    -device virtio-net-pci,netdev=net0 \
    -nographic
```

> **نکته برای محیط‌های هدلس سروری:** با قرار دادن پرچم `-nographic` و هدایت کنسول NetBSD به `com0` در فایل `/boot.cfg`، می‌توانید مستقیماً ترمینال NetBSD را درون TTY شل لینوکسی خود بدون نیاز به پنجره گرافیکی در اختیار داشته باشید.

</div>

<div class="lang-en" markdown="1">

# Deploying and Running NetBSD on QEMU/KVM

The NetBSD operating system is renowned for its portability, modular design, and clean codebase; its famous slogan *"Of course it runs NetBSD"* bears testament to its ability to run across dozens of hardware architectures. In modern Linux infrastructure and virtualization environments (based on KVM), achieving optimal performance with NetBSD requires proper configuration of hardware acceleration, high-throughput paravirtualization drivers (the VirtIO suite), and appropriate serial or graphical console management.

In this technical guide, we walk through setting up NetBSD 10.x on QEMU/KVM—from creating disk images and configuring performance flags to completing installation and essential post-install setup.

<img src="images/dfly.png" alt="NetBSD QEMU Installation" style="width: 100%; max-width: 800px;">

> ### 1. Preparing the Disk Image (qcow2 Format)

For optimal storage performance and to leverage thin provisioning capabilities, create a virtual disk in `qcow2` format:

```bash
qemu-img create -f qcow2 netbsd.qcow2 20G
```

<br>

> ### 2. Launching the Virtual Machine with Optimized Flags (KVM + VirtIO)

To achieve maximum I/O and CPU throughput, utilize the `virtio-net` and `virtio-blk` paravirtualized drivers, which are natively supported in modern NetBSD kernels:

```bash
qemu-system-x86_64 \
    -enable-kvm \
    -m 2048 \
    -smp 2 \
    -cpu host \
    -drive file=netbsd.qcow2,if=virtio,format=qcow2,cache=none \
    -cdrom NetBSD-10.0-amd64.iso \
    -boot d \
    -netdev user,id=net0,hostfwd=tcp::2222-:22 \
    -device virtio-net-pci,netdev=net0 \
    -display default
```

**Key architectural flags in this command:**
* **`-cpu host` and `-enable-kvm`:** Direct pass-through of physical CPU instruction sets without software emulation overhead.
* **`if=virtio` and `virtio-net-pci`:** Bypasses emulation layers of legacy IDE or e1000 adapters, utilizing high-speed VirtIO bus communication.
* **`hostfwd=tcp::2222-:22`:** Forwards host port 2222 to guest SSH port 22 for convenient remote access without requiring network bridge setup.

<br>

> ### 3. Installation Steps via sysinst

Once the ISO boots, you enter the curses-based NetBSD installer, `sysinst`:

1. Select language: `a: Installation in English`.
2. Choose `a: Install NetBSD to hard disk`.
3. Confirm the detected disk device (typically `ld0` for the VirtIO block device).
4. Select partitioning scheme according to your firmware (`MBR partitions` for standard BIOS or `GPT` for UEFI).
5. Format the filesystem as FFSv2 with logging support enabled (`WAPBL`) to prevent lengthy fsck scans upon unexpected shutdowns.
6. Choose the default package set (`Full installation`) and proceed with extracting installation sets from the install media.

<br>

> ### 4. Post-Installation & SSH Configuration

Before rebooting, perform these essential steps in the post-install configuration menu:

* **Set root password:** Choose a strong password for administrative access.
* **Add an operational user:** Create an unprivileged user and add them to the `wheel` group for administrative tasks.
* **Enable SSH daemon (`sshd`):** If not enabled during install, append to `/etc/rc.conf`:

```sh
sshd=YES
```

The VirtIO network interface is recognized as `vioif0`. Configure dynamic IP allocation via DHCP in `/etc/rc.conf`:

```sh
ifconfig_vioif0="dhcp"
```

<br>

> ### 5. Final Boot Without Installation Media

After installation completes, shut down the VM, detach the installation media, and boot permanently from disk:

```bash
qemu-system-x86_64 \
    -enable-kvm \
    -m 2048 \
    -smp 2 \
    -cpu host \
    -drive file=netbsd.qcow2,if=virtio,format=qcow2,cache=none \
    -netdev user,id=net0,hostfwd=tcp::2222-:22 \
    -device virtio-net-pci,netdev=net0 \
    -nographic
```

> **Tip for headless server setups:** Passing the `-nographic` flag along with redirecting NetBSD's console to `com0` in `/boot.cfg` allows full serial console interaction right inside your host Linux shell TTY without any graphic window dependencies.

</div>