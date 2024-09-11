#### Bacula. Установка и настройка в операционной системе Debian Bookworm
`Bacula — это набор компьютерных программ, позволяющий системному администратору управлять резервным копированием, восстановлением и проверкой компьютерных данных в сети компьютеров разных типов. Bacula также может работать полностью на одном компьютере и может выполнять резервное копирование на различные типы носителей, включая ленту и диск.`
[Источник](https://www.bacula.org/what-is-bacula/)

_Bacula_, в отличие от _Bareos_ присутствует в стандартных репозиториях _Debian_, поэтому для установки воспользуемся пакетным менеджером _Apt_:
```
apt install bacula
```

> [!NOTE]
> При установке метапакета _Bacula_ устанавливаются следующий перечень пакетов: 
> ```bacula-bscan bacula-client bacula-common bacula-common-pgsql bacula-console bacula-director bacula-director-pgsql bacula-fd bacula-sd bacula-server bsd-mailx dbconfig-common dbconfig-pgsql exim4-base exim4-config exim4-daemon-light libcommon-sense-perl
> libgnutls-dane0 libjson-perl libjson-xs-perl libllvm14 liblockfile1 libpq5 libtypes-serialiser-perl libunbound8 mt-st mtx postgresql postgresql-15 postgresql-client postgresql-client-15 postgresql-client-common postgresql-common sysstat```. 
> По крайней мере, один пакет  из данного списка, а имненно _bacula-director-pgsq_, потребует интерактивной настройки.

![Настройка пакета _bacula-director-pgsql_](20240910-01.png)

Чтобы избежать такого поведения, необходимо перед запуском `apt install` задать переменную **_DEBIAN_FRONTEND=noninteractive_** таким образом: `export DEBIAN_FRONTEND=noninteractive`. Тогда установщик при настройке _Bacula Catalog_ назначит параметры по умолчанию:
```# Generic catalog service
  Catalog {
  Name = MyCatalog
  dbname = "bacula"; DB Address = "localhost"; dbuser = "bacula"; dbpassword = "Password"
}
```
где __Password__ - произвольный пароль, созданный инсталлятором.

