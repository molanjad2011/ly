# مدیر صفحه نمایش Ly

![Ly screenshot](.github/screenshot.png "Ly screenshot")

Ly یک مدیر صفحه نمایش TUI (ncurses-مانند) سبک وزن برای لینوکس و BSD است.
با در نظر گرفتن قابلیت حمل طراحی شده است (به عنوان مثال برای اجرا به سیستم نیاز ندارد).

به ما در Matrix در [#ly:envs.net](https://matrix.to/#/#ly:envs.net) بپیوندید!

**توجه**: توسعه در [Codeberg] اتفاق می افتد (https://codeberg.org/fairyglade/ly)
با یک آینه در [GitHub](https://github.com/fairyglade/ly).

## وابستگی ها

- زمان کامپایل: 
- زیگ 0.15.x 
- libc 
- پم 
- xcb (اختیاری، به طور پیش فرض لازم است؛ برای پشتیبانی از X11 مورد نیاز است)
- زمان اجرا (با تنظیمات پیش فرض): 
- xorg 
- xorg-xauth 
- خاموش شدن 
- روشناییctl

### دبیان

```
# apt install build-essential libpam0g-dev libxcb-xkb-dev xauth xserver-xorg brightnessctl
```

### فدورا

**هشدار**: ممکن است با SELinux در فدورا با مشکلاتی مواجه شوید.
توصیه می شود یک قانون برای Ly اضافه کنید زیرا در حال حاضر یک قانون ارسال نمی کند.

```
# dnf install kernel-devel pam-devel libxcb-devel zig xorg-x11-xauth xorg-x11-server brightnessctl
```

### FreeBSD

```
# pkg ca_root_nss libxcb git xorg xauth را نصب کنید
```

## در دسترس بودن

[![وضعیت بسته بندی](https://repology.org/badge/vertical-allrepos/ly-display-manager.svg?exclude_unsupported=1)](https://repology.org/project/ly-display-manager/versions)

## پشتیبانی

Ly با طیف گسترده ای از محیط های دسکتاپ و پنجره آزمایش شده است
مدیران، که همه آنها را می توانید در بخش های زیر بیابید:

[محیط‌های Wayland] (#supported-wayland-environments)

[محیط‌های X11] (#supported-x11-environments)

گزارش‌ها با «/etc/ly/config.ini» تعریف می‌شوند:

- گزارش جلسه به طور پیش فرض در «~/.local/state/ly-session.log» قرار دارد.
- گزارش سیستم به طور پیش فرض در «/var/log/ly.log» قرار دارد.

## ساخت دستی

روش ساخت دستی Ly بسیار استاندارد است:

```
$ git clone https://codeberg.org/fairyglade/ly.git
$ cd ly
$ ساخت زیگ
```

پس از ساخت، می توانید (اختیاری) Ly را در یک شبیه ساز ترمینال آزمایش کنید، هرچند
احراز هویت **کار نمی کند**:

```
$ zig build run
```

**مهم**: در حالی که می توانید Ly را در یک شبیه ساز ترمینال به عنوان روت نیز اجرا کنید، اینطور است
**توصیه نمی شود** اگر می خواهید Ly را به درستی آزمایش کنید، لطفاً آن را فعال کنید
سرویس (همانطور که در زیر توضیح داده شده است) و دستگاه خود را راه اندازی مجدد کنید.

بخش های زیر نحوه نصب Ly برای یک سیستم init خاص را نشان می دهد.
از آنجا که رویه برای همه آنها بسیار شبیه است، دستورات فقط این کار را انجام می دهند
برای بخش اول (که در مورد systemd است) به تفصیل توضیح دهید.

**توجه**: همه بخش های زیر فرض می کنند که شما از LightDM برای استفاده می کنید
به خاطر راحتی

### سیستم شده

اکنون می توانید Ly را روی سیستم خود نصب کنید:

```
# zig build installexe -Dinit_system=systemd
```

**نکته**: پارامتر 'init_system' اختیاری است و به طور پیش فرض روی 'systemd' است.

توجه داشته باشید که باید مدیر نمایشگر فعلی خود را نیز غیرفعال کنید. به عنوان مثال،
اگر LightDM مدیر نمایش فعلی باشد، می توانید موارد زیر را اجرا کنید
دستور:

```
# systemctl lightdm.service را غیرفعال کنید
```

سپس، مانند دستور قبلی، باید سرویس Ly را فعال کنید:

```
# systemctl ly@tty2.service را فعال کنید
```

**مهم**: چون Ly در یک TTY اجرا می شود، **باید** سرویس TTY را غیرفعال کنید
که Ly اجرا خواهد شد، در غیر این صورت اتفاقات بدی رخ خواهد داد. به عنوان مثال، برای غیرفعال کردن تخم ریزی «getty» در TTY 2، باید دستور زیر را اجرا کنید:

```
# systemctl getty@tty2.service را غیرفعال کنید
```

در پلتفرم‌هایی که از systemd-logind برای راه‌اندازی پویا نمونه‌های «autovt@.service» استفاده می‌کنند، زمانی که سوئیچ به tty جدید اتفاق می‌افتد، هر نمونه ly برای ttys *به جز tty پیش‌فرض* باید با استفاده از مکانیزم دیگری فعال شود: برای شروع خودکار ly on سوئیچ به «tty2»، هیچ‌یک از «ly» را به‌جای این‌که به‌جای آن، به‌جای آن، به‌جای آن، به‌جای آن، مستقیماً واحد «ly» را فعال نکنید. «ly@tty2.service» در «/usr/lib/systemd/system/» (مشابه هر tty دیگری که می‌خواهید ly در آن را فعال کنید).

هدف پیوند نماد، «ly@ttyN.service»، در واقع وجود ندارد، اما systemd با این وجود تشخیص می‌دهد که نمونه «autovt@.service» با «%I» برابر با «ttyN» اکنون به نمونه‌ای از «ly@.service» با «%I» روی «ttyN» اشاره می‌کند.

با «man 5 logind.conf» مقایسه کنید، به خصوص در مورد پارامترهای «NAutoVTs=» و «ReserveVT=».


در سیستم‌های غیر سیستمی، می‌توانید TTY Ly را با ویرایش متن مربوطه تغییر دهید
فایل سرویس برای پلتفرم شما

### OpenRC

```
# zig build installexe -Dinit_system=openrc
# rc-update del lightdm
# rc-update اضافه کردن ly
# rc-update del agetty.tty2
```

**توجه**: به طور خاص در جنتو، شما همچنین **باید** نظر مناسب را بیان کنید
خط برای TTY در /etc/inittab.

### runit

```
# zig build installexe -Dinit_system=runit
# rm /var/service/lightdm
# ln -s /etc/sv/ly /var/service/
# rm /var/service/agetty-tty2
```

### s6

```
# zig build installexe -Dinit_system=s6
# s6-rc -d تغییر lightdm
# s6-service ly-srv پیش فرض را اضافه کنید
# s6-db-reload
# s6-rc -u تغییر ly-srv
```

برای غیرفعال کردن TTY 2، «/etc/s6/config/tty2.conf» را ویرایش کنید و «SPAWN="no" را تنظیم کنید.

### دینت

```
# zig build installexe -Dinit_system=dinit
# dinitctl lightdm را غیرفعال کنید
# dinitctl فعال کردن l
