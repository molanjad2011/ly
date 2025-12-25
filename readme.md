# مدیر نمایش Ly

![Ly screenshot](.github/screenshot.png "Ly screenshot")

Ly یک مدیر نمایش سبک با رابط کاربری متنی (مشابه ncurses) برای لینوکس و BSD است که با هدف قابلیت حمل طراحی شده است (مثلاً برای اجرا به systemd نیاز ندارد).

به ما در ماتریکس در [#ly:envs.net](https://matrix.to/#/#ly:envs.net) بپیوندید!

**توجه**: توسعه در [Codeberg](https://codeberg.org/fairyglade/ly) انجام می‌شود و نسخه آینه آن در [GitHub](https://github.com/fairyglade/ly) موجود است.

## وابستگی‌ها

* زمان کامپایل:

  * zig 0.15.x
  * libc
  * pam
  * xcb (اختیاری، به طور پیش‌فرض مورد نیاز؛ برای پشتیبانی از X11 لازم است)
* زمان اجرا (با پیکربندی پیش‌فرض):

  * xorg
  * xorg-xauth
  * shutdown
  * brightnessctl

### دبیان

```
# apt install build-essential libpam0g-dev libxcb-xkb-dev xauth xserver-xorg brightnessctl
```

### فدورا

**هشدار**: ممکن است در فدورا با SELinux به مشکلاتی برخورد کنید. توصیه می‌شود یک قانون برای Ly اضافه کنید، زیرا در حال حاضر ارائه نشده است.

```
# dnf install kernel-devel pam-devel libxcb-devel zig xorg-x11-xauth xorg-x11-server brightnessctl
```

### فری‌بی‌اس‌دی

```
# pkg install ca_root_nss libxcb git xorg xauth
```

## در دسترس بودن

[![Packaging status](https://repology.org/badge/vertical-allrepos/ly-display-manager.svg?exclude_unsupported=1)](https://repology.org/project/ly-display-manager/versions)

## پشتیبانی

Ly با انواع مختلف محیط‌های دسکتاپ و مدیر پنجره‌ها تست شده است که می‌توانید در بخش‌های زیر آن‌ها را بیابید:

[محیط‌های Wayland](#supported-wayland-environments)

[محیط‌های X11](#supported-x11-environments)

لاگ‌ها در `/etc/ly/config.ini` تعریف شده‌اند:

* لاگ جلسه به طور پیش‌فرض در `~/.local/state/ly-session.log` قرار دارد.
* لاگ سیستم به طور پیش‌فرض در `/var/log/ly.log` قرار دارد.

## ساخت دستی

روند ساخت دستی Ly نسبتاً استاندارد است:

```
$ git clone https://codeberg.org/fairyglade/ly.git
$ cd ly
$ zig build
```

پس از ساخت، می‌توانید (اختیاری) Ly را در یک شبیه‌ساز ترمینال تست کنید، اگرچه
احراز هویت **کار نخواهد کرد**:

```
$ zig build run
```

**مهم**: در حالی که می‌توانید Ly را به عنوان روت در یک شبیه‌ساز ترمینال اجرا کنید، **توصیه نمی‌شود**. اگر می‌خواهید Ly را به طور صحیح تست کنید، لطفاً سرویس آن را فعال کرده و سیستم خود را راه‌اندازی مجدد کنید.

بخش‌های بعدی نشان می‌دهند چگونه Ly را برای یک سیستم init خاص نصب کنید. از آنجایی که روند برای همه آن‌ها بسیار مشابه است، دستورات فقط برای بخش اول (که درباره systemd است) به تفصیل آورده شده‌اند.

**توجه**: تمام بخش‌های بعدی فرض می‌کنند که برای سهولت، شما از LightDM استفاده می‌کنید.

### systemd

اکنون می‌توانید Ly را در سیستم خود نصب کنید:

```
# zig build installexe -Dinit_system=systemd
```

**توجه**: پارامتر `init_system` اختیاری است و به طور پیش‌فرض systemd است.

توجه داشته باشید که باید مدیر نمایش فعلی خود را غیرفعال کنید. به عنوان مثال، اگر LightDM مدیر نمایش فعلی است، می‌توانید دستور زیر را اجرا کنید:

```
# systemctl disable lightdm.service
```

سپس، مشابه دستور قبلی، باید سرویس Ly را فعال کنید:

```
# systemctl enable ly@tty2.service
```

**مهم**: از آنجا که Ly در یک TTY اجرا می‌شود، **باید** سرویس TTY که Ly روی آن اجرا می‌شود را غیرفعال کنید، در غیر این صورت مشکلاتی پیش خواهد آمد. برای مثال، برای غیرفعال کردن `getty` در TTY 2، دستور زیر را اجرا کنید:

```
# systemctl disable getty@tty2.service
```

در پلتفرم‌هایی که از systemd-logind برای شروع پویا نمونه‌های `autovt@.service` هنگام تغییر به TTY جدید استفاده می‌کنند، هر نمونه Ly برای TTYها *به جز TTY پیش‌فرض* باید با مکانیزم متفاوتی فعال شود: برای شروع خودکار Ly هنگام تغییر به `tty2`، هیچ واحد `ly` را مستقیماً فعال نکنید، بلکه `autovt@tty2.service` را به `ly@tty2.service` در `/usr/lib/systemd/system/` لینک دهید (به همین ترتیب برای هر TTY دیگر که می‌خواهید Ly روی آن فعال شود).

هدف لینک نمادین، `ly@ttyN.service`، واقعاً وجود ندارد، اما systemd با این حال تشخیص می‌دهد که نمونه‌سازی `autovt@.service` با `%I` برابر `ttyN` اکنون به نمونه‌سازی `ly@.service` با `%I` برابر `ttyN` اشاره می‌کند.

برای مقایسه، `man 5 logind.conf` را ببینید، به ویژه در مورد پارامترهای `NAutoVTs=` و `ReserveVT=`.

در سیستم‌های غیر systemd، می‌توانید TTY مورد نظر Ly را با ویرایش فایل سرویس مربوطه برای پلتفرم خود تغییر دهید.

### OpenRC

```
# zig build installexe -Dinit_system=openrc
# rc-update del lightdm
# rc-update add ly
# rc-update del agetty.tty2
```

**توجه**: به طور خاص در Gentoo، همچنین باید خط مربوط به TTY در /etc/inittab را کامنت کنید.

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
# s6-rc -d change lightdm
# s6-service add default ly-srv
# s6-db-reload
# s6-rc -u change ly-srv
```

برای غیرفعال کردن TTY 2، فایل `/etc/s6/config/tty2.conf` را ویرایش کرده و `SPAWN="no"` را تنظیم کنید.

### dinit

```
# zig build installexe -Dinit_system=dinit
# dinitctl disable lightdm
# dinitctl enable l
```
