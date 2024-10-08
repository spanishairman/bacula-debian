## Bacula. Установка и настройка в операционной системе Debian Bookworm
_Bacula_ — это набор компьютерных программ, позволяющий системному администратору управлять резервным копированием, восстановлением и проверкой компьютерных данных в сети компьютеров разных типов. _Bacula_ также может работать полностью на одном компьютере и может выполнять резервное копирование на различные типы носителей, включая ленту и диск.

[Источник](https://www.bacula.org/what-is-bacula/)
### Компоненты Bacula

  -  **_Bacula Director_** - центральная управляющая программа для всех остальных демонов. Он планирует и контролирует все операции резервного копирования, восстановления, проверки и архивирования. Системный администратор использует _Bacula Director_ для планирования резервного копирования и восстановления файлов. Директор работает как демон (или служба) в фоновом режиме.
  -  **_The Bacula Console_** - это программа, которая позволяет администратору или пользователю общаться с _Bacula Director_. Он запускается в окне консоли (интерфейс TTY).
  -  **_Bacula File Daemon_** - это программа, которая должна быть установлена на каждом (клиентском) компьютере, для которого необходимо создать резервную копию. По запросу Директора _Bacula_ он находит файлы для резервного копирования и отправляет их (их данные) в демон хранения Bacula.
  -  **_Bacula Storage Daemon_** - По запросу от _Bacula Director_, _Storage Daemon_ отвечает за прием данных от _Bacula File Daemon_ и сохранение атрибутов и данных файла на физическом носителе или томах резервного копирования. В случае запроса на восстановление он отвечает за поиск данных и отправку их демону файлов _Bacula_. В вашей среде может быть несколько демонов _Bacula Storage_, каждый из которых контролируется одним и тем же _Bacula Director_. Службы хранилища работают как демон на компьютере с устройством резервного копирования (например, на ленточном накопителе).
  -  **_Catalog_** - Службы каталога состоят из программ, отвечающих за поддержание файловых индексов и баз данных томов для всех резервных копий файлов. Службы каталога позволяют системному администратору или пользователю быстро найти и восстановить любой нужный файл. Службы каталога устанавливают _Bacula_ отдельно от простых программ резервного копирования, таких как _tar_ и _bru_, поскольку в Каталоге ведется запись всех используемых томов, всех выполненных заданий и всех сохраненных файлов, что обеспечивает эффективное восстановление и управление томами. В настоящее время _Bacula_ поддерживает три разные базы данных: _MySQL_, _PostgreSQL_ и _SQLite_, одну из которых необходимо выбрать при сборке _Bacula_.

![Bacula](bacula.png)

Сетевые взаимодействия между компонентами _Bacula_. 

Для корректной работы, нам потребуется открыть некоторые порты на межсетевом экране, если такой имеется в нашей сети. Следующий список поможет настроить правила файервола:
```
Console       -> Director:9101
Director      -> Storage Daemon:9103
Director      -> File Daemon:9102
File Daemon   -> Storage Daemon:9103
```

Плюсы _Bacula_:
  - [x] Высокая масштабируемость системы. Действительно, вы можете использовать несколько ресурсов _Storage Daemon_, находящихся на разных хостах, дисковых кластерах, системах хранения данных и т.д. Ваши службы _Bacula Storage Daemon_ могут даже размещаться в разных городах и регионах, что даёт повышенную отказоустойчивость от внешних воздействий, в том числе природного характера. Тоже самое можно сказать и о ресурсах _Catalog_;
  - [x] Хранение информации о ваших файлах, задачах, пулах и томах в СУБД (MySQL, PostgreSQL, SQLite);
  - [x] Возможность получения доступа к данным и восстановления информации даже в случае недоступности _Bacula Catalog_ (см. _bscan_, _bextract_);
  - [x] Доступная и подробная [документация от сообщества](https://www.bacula.org/documentation/)
  - [x] Большое количество настроек, включая сжатие, хеширование, настройки лимитов и ограничений, автоматическую маркировку томов, поддержку _VSS_ в системах Win32;
  - [x] Система оповещения о событиях - как в журналах, так и по электронной почте или в консоли управления Директором;
  - [x] Кроссплатформенность.

### Термины
  - **_Client_** - в терминологии _Bacula_ слово _Client_ относится к резервируемому компьютеру и является синонимом Файловых служб или Файлового демона, и довольно часто его называют _File Daemon_. Клиент определяется в ресурсе файла конфигурации;
  - **_Directive_** - термин _Directive_ используется для обозначения оператора или записи в ресурсе в файле конфигурации, который определяет один конкретный параметр. Например, директива _Name_ определяет имя ресурса;
  - **_File Attributes_** -  это вся информация, необходимая для идентификации файла и все его свойства, такие как размер, дата создания, дата изменения, разрешения и т.д. Обычно атрибуты полностью обрабатываются _Bacula_, так что пользователю никогда не нужно беспокоиться о них. Атрибуты не содержат данные файла.
  - **_FileSet_** - ресурс, содержащийся в файле конфигурации, который определяет файлы для резервного копирования. Он состоит из списка включенных файлов или каталогов, списка исключенных файлов и способа хранения файла (сжатие, шифрование, подписи);
  - **_Full_** - полная резервная копия, включает в себя все файлы и каталоги, которые необходимо сохранить;
  - **_Differential_** - разностная резервная копия включает все файлы, измененные с момента начала последнего полного сохранения. Обратите внимание, что другие программы резервного копирования могут определять это по другому;
  - **_Incremental_** - инкрементное резервное копирование включает все файлы, измененные с момента запуска последнего полного, разностного или инкрементного резервного копирования. Обычно он указывается в директиве _Level_ в определении ресурса _Job_ или в ресурсе _Schedule_;
  - **_Resource_** - ресурс является частью файла конфигурации, который определяет конкретную единицу информации, доступную для _Bacula_. Он состоит из нескольких директив (отдельных операторов конфигурации). Например, ресурс задания определяет все свойства конкретного задания: имя, расписание, пул томов, тип резервного копирования, уровень резервного копирования, и т.д.;
  - **_Job_** - задание в _Bacula_ - это ресурс конфигурации, который определяет работу, которую _Bacula_ должна выполнить для резервного копирования или восстановления конкретного клиента. Он состоит из типа (резервное копирование, восстановление, проверка и т.д.), Уровня (полный, дифференциальный, инкрементный и т.д.), Набора файлов и хранилища, для которого необходимо создать резервные копии файлов (устройство хранения, пул носителей);
  - **_JobDefs_** - необязательный ресурс для предоставления значений по умолчанию для ресурсов _Job_. Т.о. - _JobDefs_ представляет собой шаблон со значениями, которые часто используются в различных ресурсах _Job_, позволяющий сократить ваши настройки и облегчить чтение конфигурации;
  - **_Restore_** -  ресурс конфигурации, который описывает операцию восстановления файла с носителя резервной копии. Это обратное сохранение, за исключением того, что в большинстве случаев для восстановления обычно требуется небольшой набор файлов для восстановления, в то время как обычно для сохранения создаются резервные копии всех файлов в системе. Конечно, после сбоя диска Bacula можно вызвать для полного восстановления всех файлов, которые были в системе;
  - **_Retention Period_** - Существуют различные виды периодов хранения, которые распознает Bacula. Наиболее важными являются:
     - период хранения файлов,
     - период хранения заданий,
     - период хранения томов.

       - **_Период хранения файлов_** определяет время хранения записей о файлах в базе данных каталога. Этот период важен по двум причинам: во-первых, пока записи файлов остаются в базе данных, вы можете «просматривать» базу данных с помощью консольной программы и восстанавливать любой отдельный файл. После того как записи о файлах удалены (removed) или усечены (pruned) из базы данных, отдельные файлы задания резервного копирования больше не могут быть «просмотрены». Вторая причина, по которой следует тщательно выбирать период хранения записей файлов, заключается в том, что объем файловых записей базы данных занимает больше всего места в базе данных. Как следствие, вы должны убедиться, что регулярное «усечение» записей файла базы данных выполняется, чтобы предотвратить слишком большой рост вашей базы данных. (см. команду _prune_ в _Console_ для более подробной информации по этому вопросу).

       - **_Период хранения заданий_** - это период времени, в течение которого записи работ будут храниться в базе данных. Обратите внимание, что все записи файла связаны с заданием, которое сохранило эти файлы. Записи файла могут быть удалены, оставляя записи задания. В этом случае будет доступна информация о запущенных заданиях, но не информация о файлах, для которых было выполнено резервное копирование. Обычно при очистке записи задания все записи файла также удаляются.

       - **_Период хранения тома_** - это минимальное время хранения _тома_ до его повторного использования. Обычно _Bacula_ никогда не перезаписывает _том_, содержащий единственную резервную копию файла. В идеальных условиях в _каталоге_ будут храниться записи для всех файлов, для которых созданы резервные копии для всех текущих _томов_. После перезаписи _тома_ файлы, для которых было выполнено резервное копирование, автоматически удаляются из _каталога_. Однако, если существует очень большой _пул томов_ (pool of Volumes) или _том_ (Volume) никогда не перезаписывается, база данных _каталога_ (Catalog) может стать огромной. Чтобы сохранить _каталог_ в управляемом размере, информация о резервной копии должна быть удалена из каталога после определенного периода хранения файлов. _Bacula_ предоставляет механизмы автоматического усечения _каталога_ в соответствии с определенными периодами хранения. 
> [!IMPORTANT]
> Каждый из этих периодов хранения применяется ко времени, когда конкретные записи будут храниться в базе данных каталога. Их не следует путать со сроком действия данных, сохраненных на _томе_.
  - **_Scan_** - операция сканирования вызывает сканирование содержимого _тома_ (Volume) или серии _томов_. Информация о том, какие файлы содержат эти _тома_, восстанавливается в каталоге _Bacula_. Как только информация восстановлена в _базе данных_ (Catalog), файлы, содержащиеся в этих _томах_, могут быть легко восстановлены. Эта функция особенно полезна, если определенные _тома_ или _задания_ превысили срок хранения и были удалены или удалены из _каталога_. Сканирование данных из _томов_ в _каталог_ осуществляется с помощью программы _bscan_.

### Установка
_Bacula_, в отличие от _Bareos_ присутствует в стандартных репозиториях _Debian_, поэтому будем использовать её. Для установки воспользуемся пакетным менеджером _apt_:
```
apt install bacula
```

> [!NOTE]
> При установке метапакета _Bacula_ устанавливаются следующий перечень пакетов: 
>
> `bacula-bscan bacula-client bacula-common bacula-common-pgsql bacula-console bacula-director bacula-director-pgsql bacula-fd bacula-sd bacula-server bsd-mailx dbconfig-common dbconfig-pgsql exim4-base exim4-config exim4-daemon-light libcommon-sense-perl
> libgnutls-dane0 libjson-perl libjson-xs-perl libllvm14 liblockfile1 libpq5 libtypes-serialiser-perl libunbound8 mt-st mtx postgresql postgresql-15 postgresql-client postgresql-client-15 postgresql-client-common postgresql-common sysstat`.
>
> По крайней мере, один пакет из данного списка, а имненно _bacula-director-pgsql_, потребует интерактивной настройки.

![Настройка пакета _bacula-director-pgsql_](20240910-01.png)

Чтобы избежать такого поведения, например, при автоматизированной установке, необходимо перед запуском `apt install` задать переменную **_DEBIAN_FRONTEND=noninteractive_**. Например, таким образом: 

```
export DEBIAN_FRONTEND=noninteractive
```

Тогда установщик при настройке _Bacula Catalog_ назначит параметры по умолчанию:

```
# Generic catalog service
  Catalog {
  Name = MyCatalog
  dbname = "bacula"; DB Address = "localhost"; dbuser = "bacula"; dbpassword = "Password"
}
```
где __Password__ - произвольный пароль, созданный инсталлятором.

База данных для _Bacula_ создается автоматически во время установки:
```
root@debian12:/etc/bacula# su postgres 
postgres@debian12:/etc/bacula$ psql -d bacula
psql (15.8 (Debian 15.8-0+deb12u1))
Введите "help", чтобы получить справку.

bacula=# \dt
               Список отношений
 Схема  |      Имя       |   Тип   | Владелец 
--------+----------------+---------+----------
 public | basefiles      | таблица | bacula
 public | cdimages       | таблица | bacula
 public | client         | таблица | bacula
 public | counters       | таблица | bacula
 public | device         | таблица | bacula
 public | file           | таблица | bacula
 public | filename       | таблица | bacula
 public | fileset        | таблица | bacula
 public | job            | таблица | bacula
 public | jobhisto       | таблица | bacula
 public | jobmedia       | таблица | bacula
 public | location       | таблица | bacula
 public | locationlog    | таблица | bacula
 public | log            | таблица | bacula
 public | media          | таблица | bacula
 public | mediatype      | таблица | bacula
 public | path           | таблица | bacula
 public | pathhierarchy  | таблица | bacula
 public | pathvisibility | таблица | bacula
 public | pool           | таблица | bacula
 public | restoreobject  | таблица | bacula
 public | snapshot       | таблица | bacula
 public | status         | таблица | bacula
 public | storage        | таблица | bacula
 public | unsavedfiles   | таблица | bacula
 public | version        | таблица | bacula
(26 строк)
```

Для того, чтобы пользователь (_vagrant_ в данном случае) мог пользоваться графическим приложением _Bacula Admin Tool - bat_, добавим его в группу _bacula_. 
```
root@debian12:/etc/bacula# usermod -a -G bacula vagrant
```
#### Настройка на сервере. Director

##### Блок настроек Director
> [!NOTE]
> Director - Главный демон сервера Bacula, который планирует и направляет все операции Bacula. Сокращённое обозначение в тексте документации - DIR.

```
#
# Default Bacula Director Configuration file
#
#  The only thing that MUST be changed is to add one or more
#   file or directory names in the Include directive of the
#   FileSet resource.
#
#  For Bacula release 9.6.7 (10 December 2020) -- debian bookworm/sid
#
#  You might also want to change the default email address
#   from root to your address.  See the "mail" and "operator"
#   directives in the Messages resource.
#
# Copyright (C) 2000-2020 Kern Sibbald
# License: BSD 2-Clause; see file LICENSE-FOSS
#

Director {                            # define myself
  Name = debian12-dir
  DIRport = 9101                # where we listen for UA connections
  QueryFile = "/etc/bacula/scripts/query.sql"
  WorkingDirectory = "/var/lib/bacula"
  PidDirectory = "/run/bacula"
  Maximum Concurrent Jobs = 20
  Password = "QU3EOkh3mnNp1oe4DLbXCJ4uRLaWZMxVD"         # Console password
  Messages = Daemon
  DirAddress = 127.0.0.1
}
```
Если _bconsole_ будет использоваться не только на данной машине, то сетевой адрес, на котором доступен демон _Director_, нужно будет изменить, например, на `0.0.0.0`, остальные настройки можно оставить как есть.

##### Блок настроек Jobdefs
_Шаблоны задач_. Некоторые параметры для различных задач _Job_ могут повторяться. Поэтому их можно вынести в блоки _Jobdefs_, чтобы затем ссылаться на них из директив _Job_ для сокращения количества настроек и более простого чтения конфигурационного файла. 

По умолчанию, конфигурационный файл _bacula-dir.conf_ уже содержит минимальный набор параметров, здесь я приведу только те, которые добавил сам.

```
JobDefs {
  Name = "My-JobDef-Tpl"
  Type = Backup
  Storage = debian12-sd
  Messages = Standard
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}
```
##### Блок настроек Job

> [!NOTE]
> Job - Задание в _Bacula_ - это ресурс конфигурации, который определяет работу, которую _Bacula_ должна выполнить для резервного копирования или восстановления конкретного клиента. Он состоит из типа (резервное копирование, восстановление, проверка и т.д.), Уровня (полный, дифференциальный, инкрементный и т.д.), Набора файлов и хранилища, для которого необходимо создать резервные копии файлов (устройство хранения, пул носителей). 

Создадим две задачи - _Clnt1-fs-Job_ и _Clnt2-fs-Job_, в которых будут архивироваться каталоги, перечисленные в параметрах _FileSet = "My-fs-FS"_ и _FileSet = "My-tfs-FS"_. 
Параметры _FileSet_ могут быть общими для разных задач и клиентов.  Далее, в конфигурационном файле, мы зададим для данных параметров списки каталогов, подлежащих резервному копированию.

Также, в ресурсе задачи (_Job_) перечислены пулы - _Pools_ для каждого из трёх типов резервных копий - полная, разностная и инкрементная. Для задания пулов также можно восмользоваться функцией _переопределения_, которая доступна для ресурса _Schedule_.
> [!NOTE]
> Параметр _Pool_ для ресурса _Job_ является обязательным.
>
> Даже если вы используете _переопределения_ параметров _Pool_, _Full Backup Pool_, _Differential Backup Pool_ или _Incremental Backup Pool_, _задачи_ в ресурсе _Schedule_, параметр _Pool_ должен присутствовать в ресурсе _Job_ или _JobDefs_, на который ссылается _Job_.
>
> Также он должен быть задан в ресурсе _Job_ или _JobDefs_, на который ссылается _Job_, независимо от наличия в ресурсе задачи параметров _Full Backup Pool_, _Differential Backup Pool_ или _Incremental Backup Pool_.

В нашем случае, пулы в каждой задаче свои и будут содержать тома только одного клиента - это необязательное условие, просто здесь я выбрал такой принцип для наглядности.

_Расписание_ (_Schedule_) - также для каждого клиента своё.

_Скрипты_ (_ClientRunBeforeJob_ и _ClientRunAfterJob_), выполняющиеся до и после задачи резервного копирования запускаются на клиентской машине, т.е. на стороне _File Daemon_. Такие скрипты могут содержать Bash-команды по предварительному сжатию архивируемых файлов, копированию их в tar-архив или, например, SQL-команды, выполняющие дамп баз данных. 
Скрипты, выполняющиеся после задачи резервного копирования, могут содержать команды по удалению архивируемых файлов или ранее созданных tar-архивов или sql-файлов.

Скрипт для параметра _ClientRunBeforeJob_:
```
#!/bin/bash
tar -c -f /bacula-backup/backup.tar /etc /var
```

Скрипт для параметра _ClientRunAfterJob_:
```
#!/bin/bash
rm -rf /bacula-backup/backup.tar
```

> [!NOTE]
> Чтобы было легче ориентироваться, для задач, пулов, расписаний и клиентов я выбрал шаблонные имена, которые содержат общие части. 
> Например, для клиента _Debian12cl1-fd_ соответствует имя задачи _Clnt1-fs-Job_, пулы томов _Clnt1-fs-Full_, _Clnt1-fs-Diff_, _Clnt1-fs-Incr_ расписание - _Clnt1-fs-Sdl_.

```
Job {
  Name = "Clnt1-fs-Job"
  FileSet = "My-tfs-FS"
  Pool = Clnt1-fs-Full
  Full Backup Pool = Clnt1-fs-Full                  # write Full Backups into "Full" Pool         (#05)
  Differential Backup Pool = Clnt1-fs-Diff
  Incremental Backup Pool = Clnt1-fs-Incr           # write Incr Backups into "Incremental" Pool  (#11)
  Schedule = "Clnt1-fs-Sdl"
  JobDefs = "My-JobDef-Tpl"
  Client = "Debian12cl1-fd"
  ClientRunBeforeJob = "/etc/bacula/scripts/bacula-before-fs.sh" # скрипт выполняющийся до задачи
  ClientRunAfterJob = "/etc/bacula/scripts/bacula-after-fs.sh" # скрипт выполняющийся после задачи
}

Job {
  Name = "Clnt1-fs-Monthly-Job"
  FileSet = "My-tfs-FS"
  Pool = Clnt1-fs-Monthly
  Full Backup Pool = Clnt1-fs-Monthly               # write Full Backups into "Full" Pool         (#05)
  Schedule = "Clnt1-fs-Monthly-Sdl"
  JobDefs = "My-JobDef-Tpl"
  Client = "Debian12cl1-fd"
  ClientRunBeforeJob = "/etc/bacula/scripts/bacula-before-fs.sh" # скрипт выполняющийся до задачи
  ClientRunAfterJob = "/etc/bacula/scripts/bacula-after-fs.sh" # скрипт выполняющийся после задачи
}

Job {
  Name = "Clnt2-fs-Job"
  FileSet = "My-fs-FS"
  Pool = Clnt2-fs-Full
  Full Backup Pool = Clnt2-fs-Full                  # write Full Backups into "Full" Pool         (#05)
  Differential Backup Pool = Clnt2-fs-Diff
  Incremental Backup Pool = Clnt2-fs-Incr           # write Incr Backups into "Incremental" Pool  (#11)
  Schedule = "Clnt2-fs-Sdl"
  JobDefs = "My-JobDef-Tpl"
  Client = "Debian12cl2-fd"
}

Job {
  Name = "Clnt2-fs-Monthly-Job"
  FileSet = "My-fs-FS"
  Pool = Clnt2-fs-Monthly
  Full Backup Pool = Clnt2-fs-Monthly               # write Full Backups into "Full" Pool         (#05)
  Schedule = "Clnt2-fs-Monthly-Sdl"
  JobDefs = "My-JobDef-Tpl"
  Client = "Debian12cl2-fd"
}

Job {
  Name = "MyRestoreFiles"
  Type = Restore
  Client=debian12-fd
  Storage = debian12-sd
# The FileSet and Pool directives are not used by Restore Jobs  but must not be removed
  FileSet="Full Set"
  Pool = File
  Messages = Standard
  Where = /bacula-restores
}
```

В примере ниже описывается стандартная задача восстановления на несуществующий носитель, как говорится в описании, этого достаточно для всех наборов _Jobs_, _Clients_, _Storages_. 
Тем не менее, я создал похожую задачу, где лишь изменил каталог по умолчанию, в который будут восстанавливаться данные. Содержимое моей задачи восстановления в листинге кода выше по тексту. 
```
#
# Standard Restore template, to be changed by Console program
#  Only one such job is needed for all Jobs/Clients/Storage ...
#
Job {
  Name = "RestoreFiles"
  Type = Restore
  Client=debian12-fd
  Storage = File1
# The FileSet and Pool directives are not used by Restore Jobs
# but must not be removed
  FileSet="Full Set"
  Pool = File
  Messages = Standard
  Where = /bacula-restores
}
```
##### Наборы файлов - FileSet
> [!NOTE]
> Ресурс _FileSet_ определяет, какие файлы должны быть включены в задании резервного копирования или исключены из него. Набор файлов требуется для каждого задания резервного копирования. Он состоит из списка файлов или каталогов, которые необходимо включить, 
> списка файлов или каталогов, которые необходимо исключить, и различных параметров резервного копирования, таких как сжатие, шифрование и подписи, которые должны применяться к каждому файлу.

```
FileSet {
  Name = "My-tfs-FS"
  Enable VSS = yes
  Include {
    Options {
      Signature = SHA1
      Compression = GZIP
      No Atime = yes
      Sparse = yes
      Checkfilechanges = yes
      IgnoreCase = no
    }
    File = "/bacula-backup/backup.tar"
  }
}

FileSet {
  Name = "My-fs-FS"
  Enable VSS = yes
  Include {
    Options {
      Signature = SHA1
      Compression = LZO
      No Atime = yes
      Sparse = yes
      Checkfilechanges = yes
      IgnoreCase = no
    }
    File = "/etc"
    File = "/var"
  }
}
```
Здесь:
  - **_Name_** - имя набора файлов;
  - **_Enable VSS_** - Эта директива эффективна только для VSS включенных файловых демонов Win32. Она позволяет создавать согласованную копию открытых файлов для сотрудничающих записывающих приложений, а для приложений, которые не находятся в VSS, Bacula может по крайней мере получить доступ к открытым файлам. Значение по умолчанию — **_yes_**.
  - **_Signature_** - Подпись SHA1 будет вычисляться для каждого сохраненного файла. Предполагается, что алгоритм SHA1 несколько медленнее, чем алгоритм MD5, но в то же время он значительно лучше с криптографической точки зрения (т.е. гораздо меньше коллизий). Подпись SHA1 требует добавления 20 байтов на файл в ваш каталог.
  - **_Compression_** - Все сохраненные файлы будут программно сжаты с использованием формата сжатия GZIP. Сжатие выполняется файл за файлом файловым демоном. Для программного сжатия задания резервного копирования Bacula можно настроить на использование сжатия ZIP (уровни с 1 по 9, по умолчанию 6), LZO (уровень LZ01X) или zstd. LZO обеспечивает гораздо более высокую скорость сжатия и распаковки, но более низкую степень сжатия, чем GZIP.
  - **_No Atime_** - Если этот параметр включен и ваша операционная система поддерживает флаг открытия файла O_NOATIME, Bacula откроет все файлы для резервного копирования с этой опцией. Это позволяет читать файл без обновления atime inode (а также без обновления ctime inode, которое происходит, если вы пытаетесь установить atime обратно в его предыдущее значение). Это также предотвращает состояние гонки, когда две программы читают один и тот же файл, но только одна не хочет изменять время. Это наиболее полезно для программ резервного копирования и проверок целостности файлов (и bacula может соответствовать обеим категориям).
  - **_Sparse_** - Включает специальный код, который проверяет наличие разреженных файлов, например, созданных ndbm. Значение по умолчанию — no , поэтому для разреженных файлов проверки не производятся. Вы можете указать sparse=yes даже для файлов, которые не являются разреженными файлами. Вреда не будет, но будут небольшие дополнительные накладные расходы на проверку буферов, состоящих из одних нулей, и если есть блок размером 32 КБ, состоящий из одних нулей (см. ниже), этот блок станет дырой в файле, что может быть нежелательным, если исходный файл не был разреженным файлом.
  - **_Checkfilechanges_** - Если включено, клиент будет проверять размер и возраст каждого файла после резервного копирования, чтобы узнать, изменились ли они во время резервного копирования. Если время или размер не совпадают, возникнет ошибка. Значение по умолчанию — **_no_**.
  - **_IgnoreCase_** - Значение по умолчанию — no. В системах _Windows_ вы почти наверняка захотите установить это значение **_yes_** . Если эта директива установлена ​​в **_yes_**, регистр символов будет игнорироваться при сравнении с подстановочными знаками и регулярными выражениями. То есть заглавная буква A будет соответствовать строчной букве a.

###### Исключения из резервных копий. 

Для исключения файлов и каталогов при определении наборов файлов - _Fileset_ используется ресурс _Exclude_, например, так:
```
FileSet {
  Name = "Full Set"
  Include {
    Options {
      signature = MD5
    }
    File = /usr/sbin
  }
  Exclude {
    File = /var/lib/bacula
    File = /nonexistant/path/to/file/archive/dir
    File = /proc
    File = /tmp
    File = /sys
    File = /.journal
    File = /.fsck
  }
}
```
###### Подстановочные знаки

**wilddir=строка** - Указывает строку подстановочных знаков, которая будет применяться только к именам каталогов. Никакие имена файлов не будут сопоставлены этой директивой. Обратите внимание, если Exclude не включен, подстановочный знак выберет каталоги для включения. Если указано Exclude=yes , подстановочный знак выберет каталоги, которые следует исключить. Можно указать несколько директив подстановочных знаков, и они будут применяться по очереди, пока не будет найдена первая совпадающая. Обратите внимание, если вы исключите каталог, никакие файлы или каталоги под ним не будут сопоставлены. Рекомендуется заключить строку в двойные кавычки.

**wildfile=строка** - Указывает строку с подстановочными знаками, которая будет применяться к именам файлов, исключая каталоги. То есть ни одна запись каталога не будет сопоставлена ​​этой директивой. Однако обратите внимание, что сопоставление выполняется с полным путем и именем файла, поэтому ваша строка с подстановочными знаками должна учитывать, что именам файлов предшествует полный путь. Если Exclude не включен, подстановочный знак выберет, какие файлы должны быть включены. Если указано Exclude=yes , подстановочный знак выберет, какие файлы должны быть исключены. Можно указать несколько директив с подстановочными знаками, и они будут применяться по очереди, пока не будет найдена первая совпадающая. Рекомендуется заключить строку в двойные кавычки.

Пример:
```
FileSet {
  Name = "SRV01-DB-FS"
  Enable VSS = yes
  Include {
    Options {
      Signature = SHA1
      Compression = GZIP
      No Atime = yes
      Sparse = yes
      Checkfilechanges = yes
      Drive Type = fixed
      IgnoreCase = yes
      WildFile = "[A-Z]:/pagefile.sys"
      WildDir = "[A-Z]:/RECYCLER"
      WildDir = "[A-Z]:/$RECYCLE.BIN"
      WildDir = "[A-Z]:/System Volume Information"
      Exclude = yes
    }
    File = "F:/BackUP/"
  }
```

##### Расписания
Пример расписания:
```
Schedule {
  Enabled = yes
  Name = "Clnt1-fs-Sdl"
  Run = Level=Full Pool=Clnt1-fs-Monthly on 1 at 00:00
  Run = Level=Full at 01:00
  Run = Level=Differential at 13:00
  Run = Level=Incremental 2-12
  Run = Level=Incremental 14-23
  Run = Level=Incremental on 2-31 at 00:00
}

Schedule {
  Enabled = yes
  Name = "Clnt2-fs-Sdl"
  Run = Level=Full Pool=Clnt2-fs-Monthly on 1 at 00:00
  Run = Level=Full Pool=Clnt2-fs-Full at 01:00
  Run = Level=Differential Pool=Clnt2-fs-Diff at 13:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 02:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 03:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 04:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 05:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 06:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 07:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 08:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 09:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 10:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 11:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 12:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 14:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 15:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 16:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 17:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 18:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 19:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 20:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 21:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 22:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr at 23:00
  Run = Level=Incremental Pool=Clnt2-fs-Incr on 2-31 at 00:00
}
```
В данном примере для расписания с именем _Clnt1-fs-Sdl_ действуют следующие задания:
  - **Run = Level=Full Pool=Clnt1-fs-Monthly on 1 at 00:00** - выполняет полное резервное копировние в 00:00 каждого первого числа месяца, копия будет храниться на отдельном пуле томов в течение года;
  - **Run = Level=Full at 01:00** - ежедневно в 01:00 выполняет полное резервное копирование на пул томов со сроком хранения 92 дня;
  - **Run = Level=Differential at 13:00** - ежедневно в 13:00 выполняет разностное резервное копирование на пул томов со сроком хранения 31 день;
  - **Run = Level=Incremental 2-12** - инкрементная копия каждый час с 2 до 12 на том со сроком хранения - 7 дней;
  - **Run = Level=Incremental 14-23** - инкрементная копия каждый час с 14 до 23 на том со сроком хранения - 7 дней;
  - **Run = Level=Incremental on 2-31 at 00:00** - инкрементная копия, выполняется в полночь кроме дня (1-е число каждого месяца), когда создается полная копия со сроком хранения 1 год.

Директива _Run_ определяет, когда _задание_ должно быть выполнено, а также задаёт _переопределения_, если таковые имеются для применения. Вы можете указать несколько директив _Run_ в ресурсе _Schedule_. Если вы это сделаете, все они будут применены (т.е. несколько расписаний). Если у вас есть две директивы _Run_, которые запускаются одновременно, два задания запускаются одновременно (ну, в течение одной секунды друг от друга).

_Переопределения_ заданий (Job-overrides) позволяют переопределять спецификации Уровня (Level), Хранилища (Storage), Сообщений (Messages) и Пула (Pool), представленные в ресурсе _задание_ (Job resource). Кроме того, спецификации _FullPool_, _IncrementalPool_ и _DifferentialPool_ позволяют переопределять спецификацию пула в зависимости от того, какой уровень заданий резервного копирования действует.

Используя переопределения, вы можете настроить конкретную _задачу_. Например, вы можете указать переопределение сообщений для _инкрементных резервных копий_ (Incremental backups), которые выводят сообщения в файл журнала, но для еженедельных или ежемесячных _полных резервных копий_ вы можете отправлять выходные данные по электронной почте, используя другое переопределение сообщений.

Переопределения заданий задаются как: **keyword=value**, где ключевым словом является _Level_, _Storage_, _Messages_, _Pool_, _FullPool_, _DifferentialPool_, или _IncrementalPool_, а значение определено в соответствующих форматах директив для ресурса _Job_. Вы можете указать несколько переопределений заданий в одной директиве _Run_, разделяя их одним или несколькими пробелами или запятой. Например:

  - **Level=Full** - все файлы в FileSet независимо от того, были ли они изменены.
  - **Level=Incremental** - это все файлы, которые изменились с момента последнего резервного копирования.
  - **Pool=Weekly** - указывает на использование пула с именем Weekly.
  - **Storage=DLT\_Drive - указывает использовать DLT\_Drive** в качестве устройства хранения.
  - **Messages=Verbose** - указывает на использование ресурса подробных сообщений для задания.
  - **FullPool=Full** - указывает на использование пула с именем Full, если задание является полной резервной копией или обновлено с другого типа до полной резервной копии.
  - **DifferentialPool=Differential** - указывает на использование пула с именем Differential, если задание является дифференциальной резервной копией.
  - **IncrementalPool=Incremental** - указывает использовать пул с именем Incremental, если задание является инкрементной резервной копией.
  - **Accurate=yes|no** - говорит _Bacula_ использовать или нет точный код для конкретной работы. Это может позволить вам сэкономить память и ресурсы ЦП на сервере Catalog в некоторых случаях.
  - **SpoolData=yes|no** - говорит _Bacula_ использовать или не использовать буферизацию (spooling) для конкретной Задачи.

В расписании _Clnt2-fs-Sdl_ немного изменена форма записи для демонстрации гибкости при настройке данного ресурса.

> [!NOTE]
> Можно заметить, что пул томов, на которые производится резервное копирование, может быть задан как в ресурсе **Job** в виде:
>  - **Full Backup Pool = Pool-name-Full**, 
>  - **Differential Backup Pool = Pool-name-Diff**, 
>  - **Incremental Backup Pool = Pool-name-Incr**,
>
> а также в ресурсе **Schedule** в виде _преопределений_, о которых была сказано ранее: 
>  - **Level=Full Pool=Pool-name-Full**, 
>  - **Level=Differential Pool=Pool-name-Diff**, 
>  - **Level=Incremental Pool=Pool-name-Incr**. 

> [!IMPORTANT] 
> Эти записи при определённых условиях равнозначны, но существует особенность, из-за которой не рекомендуется использовать явное указание пула томов в задаче расписания, например, в случае использования различных пулов под разные типы резервных копий.
> Предположим, что у нас пришло время для запуска задания `Run = Level=Incremental Pool=Clnt2-fs-Incr at 14:00` из расписания `Name = "Clnt2-fs-Sdl"`, но по какой-то причине для клиента _debian12-fd_ в пулах томов отстутствуют полные и разностные копии файловых систем. Это стандартная ситуация при первоначальном запуске системы резервного копирования, когда еще нет ни одной резервной копии или при добавлении нового клиента в существующую СРК.
> В таком случае, _Bacula_ всё-равно выполнит задачу резервного копирования, но с одним условием - будет создана **полная** _Full_ копия файловых систем, указанных в параметре _FileSet_. А, так как в ресурсе расписания _"Clnt2-fs-Sdl"_ явно указан пул _Clnt2-fs-Incr_, 
> то данные, содержащиеся в этой **полной** копии запишутся в явно указанный пул - _Clnt2-fs-Incr_, предназначенный для инкрементных копий. Ничего страшного при этом не произойдёт, кроме того что нарушится система ротации резервных копий на томах, а также
> на томе из "неправильного" пула может не хватить места для полной копии.

Таким образом, окончательно, ресурс расписаний в нашем примере выглядит следующим образом:
```
# My Schedules

Schedule {
  Enabled = yes
  Name = "Clnt1-fs-Sdl"
  Run = Level=Full Pool=Clnt1-fs-Monthly on 1 at 00:00
  Run = Level=Full at 01:00
  Run = Level=Differential at 13:00
  Run = Level=Incremental 2-12
  Run = Level=Incremental 14-23
  Run = Level=Incremental on 2-31 at 00:00
}

Schedule {
  Enabled = yes
  Name = "Clnt2-fs-Sdl"
  Run = Level=Full Pool=Clnt2-fs-Monthly on 1 at 00:00
  Run = Level=Full Pool=Clnt2-fs-Full at 01:00
  Run = Level=Differential at 13:00
  Run = Level=Incremental at 02:00
  Run = Level=Incremental at 03:00
  Run = Level=Incremental at 04:00
  Run = Level=Incremental at 05:00
  Run = Level=Incremental at 06:00
  Run = Level=Incremental at 07:00
  Run = Level=Incremental at 08:00
  Run = Level=Incremental at 09:00
  Run = Level=Incremental at 10:00
  Run = Level=Incremental at 11:00
  Run = Level=Incremental at 12:00
  Run = Level=Incremental at 14:00
  Run = Level=Incremental at 15:00
  Run = Level=Incremental at 16:00
  Run = Level=Incremental at 17:00
  Run = Level=Incremental at 18:00
  Run = Level=Incremental at 19:00
  Run = Level=Incremental at 20:00
  Run = Level=Incremental at 21:00
  Run = Level=Incremental at 22:00
  Run = Level=Incremental at 23:00
  Run = Level=Incremental on 2-31 at 00:00
}
```

Дата/время запуска задания могут быть указаны в формате псевдо-BNF следующим образом: 
```
<week-keyword>         ::= 1st | 2nd | 3rd | 4th | 5th | first | second | third | fourth | fifth | last
<wday-keyword>         ::= sun | mon | tue | wed | thu | fri | sat | sunday | monday | tuesday | wednesday | thursday | friday | saturday
<week-of-year-keyword> ::= w00 | w01 | ... w52 | w53
<month-keyword>        ::= jan | feb | mar | apr | may | jun | jul | aug | sep | oct | nov | dec | january | february | ... | december
<digit>                ::= 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 0
<number>               ::= <digit> | <digit><number>
<12hour>               ::= 0 | 1 | 2 | ... 12
<hour>                 ::= 0 | 1 | 2 | ... 23
<minute>               ::= 0 | 1 | 2 | ... 59
<day>                  ::= 1 | 2 | ... 31
<time>                 ::= <hour>:<minute> | <12hour>:<minute>am | <12hour>:<minute>pm
<time-spec>            ::= at <time> | hourly
<day-range>            ::= <day>-<day>
<month-range>          ::= <month-keyword>-<month-keyword>
<wday-range>           ::= <wday-keyword>-<wday-keyword>
<range>                ::= <day-range> | <month-range> | <wday-range>
<modulo>               ::= <day>/<day> | <week-of-year-keyword>/<week-of-year-keyword>
<date>                 ::= <date-keyword> | <day> | <range>
<date-spec>            ::= <date> | <date-spec>
<day-spec>             ::= <day> | <wday-keyword> | <day> | <wday-range> | <week-keyword> <wday-keyword> | <week-keyword> <wday-range> | daily
<month-spec>           ::= <month-keyword> | <month-range> | monthly
<date-time-spec>       ::= <month-spec> <day-spec> <time-spec>
```
###### Client (FileDaemon)
> [!NOTE]
> **Client** - В терминологии _Bacula_ слово «Клиент» относится к резервируемому компьютеру и является синонимом Файловых служб или Файлового демона, и довольно часто его называют _File Daemon_. Клиент определяется в ресурсе файла конфигурации.
```
# Client (File Services) to backup
Client {
  Name = debian12-fd
  Address = localhost
  FDPort = 9102
  Catalog = MyCatalog
  Password = "LxsN1MC0RmvrRhyQ_5dZjyoVonnheWzjl"          # password for FileDaemon
  File Retention = 60 days            # 60 days
  Job Retention = 6 months            # six months
  AutoPrune = yes                     # Prune expired Jobs/Files
}

# Client (File Services) to backup
Client {
  Name = Debian12cl1-fd
  Address = 192.168.121.11
  FDPort = 9102
  Catalog = MyCatalog
  Password = "DYrPl1SQnGYgUHDy809bU6ejZyo-N97m4"          # password for FileDaemon
  File Retention = 365 days            # 60 days
  Job Retention = 12 months            # six months
  AutoPrune = yes                     # Prune expired Jobs/Files
}
 
# Client (File Services) to backup
Client {
  Name = Debian12cl2-fd
  Address = 192.168.121.12
  FDPort = 9102
  Catalog = MyCatalog
  Password = "tyWfHO1Bp3joollMSdXggFoeBoMTPZF8G"          # password for FileDaemon
  File Retention = 365 days           # 60 days
  Job Retention = 12 months           # six months
  AutoPrune = yes                     # Prune expired Jobs/Files
} 
```
Здесь:
  - **File Retention = time-period-specification** - Директива _File Retention_ определяет период времени, в течение которого _Bacula_ будет хранить записи _File_ в базе данных _Catalog_ после времени окончания задания, соответствующего записям _File_. По истечении этого периода времени, и если _AutoPrune_ установлено на yes, _Bacula_ удалит (обрежет) записи _File_, которые старше указанного периода _File Retention_. Обратите внимание, что это влияет только на записи в базе данных _Catalog_. Это не влияет на ваши резервные копии архива. Записи файлов могут фактически храниться в течение более короткого периода, чем вы указали в этой директиве, если вы укажете более короткий период хранения _Job Retention_ или более короткий период хранения _Volume Retention_ . Самый короткий период хранения из трех имеет приоритет. Время может быть выражено в секундах, минутах, часах, днях, неделях, месяцах, кварталах или годах. Значение по умолчанию — 60 дней.
  - **Job Retention = time-period-specification** - Директива _Job Retention_ определяет период времени, в течение которого _Bacula_ будет хранить записи _Job_ в базе данных _Catalog_ после времени окончания _Job_. По истечении этого периода времени, и если _AutoPrune_ установлено на yes, _Bacula_ удалит (обрежет) записи _Job_, которые старше указанного периода _File Retention_. Как и в случае с другими периодами хранения, это влияет только на записи в каталоге, а не на данные в резервной копии архива. Если для обрезки выбрана запись _Job_, все связанные записи _File_ и _JobMedia_ также будут обрезаны независимо от установленного периода хранения _File_. Как следствие, вы обычно устанавливаете период хранения _File_ меньше периода хранения _Job_. Период хранения _Job_ может быть фактически меньше указанного вами здесь значения, если вы установите директиву _Volume Retention_ в ресурсе _Pool_ на меньшую продолжительность. Это связано с тем, что период хранения _Job_ и период хранения _Volume_ применяются независимо, поэтому приоритет имеет меньший из двух. Период сохранения задания указывается в секундах, минутах, часах, днях, неделях, месяцах, кварталах или годах. Значение по умолчанию — 180 дней.
  - **AutoPrune = yes/no** - Если _AutoPrune_ установлен на **_yes_** (по умолчанию), _Bacula_ (версия 1.20 или выше) автоматически применит период хранения файлов и период хранения заданий для клиента в конце задания. Если вы установите **_AutoPrune = no_** , обрезка не будет выполняться, и ваш каталог будет увеличиваться в размере каждый раз, когда вы запускаете задание. Обрезка влияет только на информацию в каталоге, а не на данные, хранящиеся в архивах резервных копий (на томах).

>[!NOTE]
> _Bacula_ никогда не будет удалять записи из _Catalog_, даже если срок их хранения закончился, пока в соответствующих пулах есть свободные тома или на томах есть свободное место и не превышено количество заданий на них.

###### Сообщения (Messages)
Значения по умолчанию:
```
# Reasonable message delivery -- send most everything to email address
#  and to the console
Messages {
  Name = Standard
#
# NOTE! If you send to two email or more email addresses, you will need
#  to replace the %r in the from field (-f part) with a single valid
#  email address in both the mailcommand and the operatorcommand.
#  What this does is, it sets the email address that emails would display
#  in the FROM field, which is by default the same email as they're being
#  sent to.  However, if you send email to more than one address, then
#  you'll have to set the FROM address manually, to a single address.
#  for example, a 'no-reply@mydomain.com', is better since that tends to
#  tell (most) people that its coming from an automated source.

#
  mailcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula: %t %e of %c %l\" %r"
  operatorcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula: Intervention needed for %j\" %r"
  mail = root = all, !skipped
  operator = root = mount
  console = all, !skipped, !saved
#
# WARNING! the following will create a file that you must cycle from
#          time to time as it will grow indefinitely. However, it will
#          also keep all your messages if they scroll off the console.
#
  append = "/var/log/bacula/bacula.log" = all, !skipped
  catalog = all
}

 
#
# Message delivery for daemon messages (no job).
Messages {
  Name = Daemon
  mailcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula daemon message\" %r"
  mail = root = all, !skipped
  console = all, !skipped, !saved
  append = "/var/log/bacula/bacula.log" = all, !skipped
}
```
Здесь можно задать свои параметры:
  - **operator = spanish.airman@firma.com  = mount** - адрес получателя сообщений и события из команды **operatorcommand**;
  - **mail = spanish.airman@firma.com all, !skipped** - адрес получателя сообщений и события из команды **mailcommand**.

Сокращения, используемые в командах:
  - %% = %
  - %c = Имя клиента
  - %d = Имя Директора
  - %e = Код выхода из задания (_Job exit code_) (OK, Ошибка, ...)
  - %h = Адрес клиента
  - %i = Идентификатор задания (_Job Id_)
  - %j = Уникальное имя задания (_Unique Job name_)
  - %l = Уровень задания (_Job level_)
  - %n = Название задания (_Job name_)
  - %r = Получатели (_Recipients_)
  - %s = Время запуска (_Since time_)
  - %t = Тип задания (_Job type_) (например, резервное копирование, ...)
  - %v = Имя тома (_Volume name_) (Только на стороне Директора)

>[!IMPORTANT]
> Любая директива _Mail Command_ должна быть указана в ресурсе _Messages_ перед параметром _Mail, Mail On Success_ или _Mail On Error_. 

  - **Operator Command** - Этот ресурс похож на _Mail Command_, за исключением того, что он используется для сообщений _Оператора_. Замены, выполненные для _Mail Command_ также выполняются для этой команды. Обычно для этой команды устанавливается то же значение, которое указано для _Mail Command_. Директива _Operator Command_ должна появиться в ресурсе _Messages_ перед директивой _Operator_.
  - **Mail** - Отправить сообщение на адреса электронной почты, указанные в виде списка, разделенного запятыми (без пробела) в поле адреса. Сообщения электронной почты группируются во время выполнения задания, а затем отправляются как одно сообщение электронной почты после завершения задания. Преимущество этого назначения заключается в том, что мы получаем уведомление о каждом запущенном задании. Однако, если мы выполняем резервное копирование нескольких машин каждую ночь, количество сообщений электронной почты может раздражать. Некоторые пользователи используют программы-фильтры, такие как procmail, для автоматической регистрации этого сообщения электронной почты на основе кода завершения задания.
  - **Operator** - Отправлять сообщение на адреса электронной почты, указанные в виде списка, разделенного запятыми (без пробела) в поле адреса. Это похоже на директиву _Mail_ выше, за исключением того, что каждое сообщение отправляется по мере получения. Таким образом, на каждое сообщение приходится одно электронное письмо. Это наиболее полезно для сообщений о монтировании.

###### Хранилище - Storage
Параметры для подключения к службе _Storage Daemon_. Здесь также указано устройство хранения - `FileStorage`
```
Storage {
  Name = debian12-sd
# Do not use "localhost" here
  Address = 192.168.121.10                # N.B. Use a fully qualified name here
  SDPort = 9103
  Password = "gYnV07jAsfPmX-mFbmXAsE0QrAve2TK9i"
  Device = FileStorage
  Media Type = File
  Maximum Concurrent Jobs = 10        # run up to 10 jobs a the same time
}
```

###### Пулы томов - Pool

Пулы объединяют тома таким образом, чтобы резервная копия не ограничивалась длиной одного тома (ленты). Следовательно, вместо того, чтобы явно указывать тома в своих заданиях, вы указываете пул и Bacula выберет следующий добавляемый том из пула и смонтирует его. Прежде чем Bacula будет читать или записывать том, физический том должен иметь метку программного обеспечения Bacula, чтобы Bacula мог быть уверен, что установлен правильный том. В зависимости от конфигурации это может быть сделано Bacula автоматически или вручную с помощью команды label в bconsole. 

>[!NOTE]
> Том - это физическое устройство, такое как лента или файл, на которое Bacula будет записывать резервные данные. 

Настроить _Bacula_ для записи на диск, а не на ленту довольно просто. В файле конфигурации  _Storage daemon_ вы просто определяете с помощью директивы «File» в качестве устройства архивации каталог (_Storage Daemon → Device_). Здесь же настраивается каталог по умолчанию для хранения резервных копий на диске /var/lib/bacula/storage (директива _Archive Device_). 

```
# File Pool definition Clnt1-fs
Pool {
  Name = Clnt1-fs-Monthly
  Pool Type = Backup
  Recycle = yes                         # Bacula can automatically recycle Volumes
  AutoPrune = yes                       # Prune expired volumes
  Recycle Oldest Volume = yes           # Prune the oldest volume in the Pool, and if all files were pruned, recycle this volume and use it.
  Volume Retention = 365  days          # How long should the Full Backups be kept? (#06)
  Maximum Volume Bytes = 1G             # Limit Volume size to something reasonable
  Maximum Volume Jobs = 1               # One Job = One Vol
  Maximum Volumes = 12                  # Limit number of Volumes in Pool
  Label Format = "Clnt1-fs-Monthly-"    # Volumes will be labeled "Full-<volume-id>"
}
Pool {
  Name = Clnt1-fs-Full
  Pool Type = Backup
  Recycle = yes
  AutoPrune = yes
  Recycle Oldest Volume = yes
  Volume Retention = 92  days
  Maximum Volume Bytes = 1G
  Maximum Volume Jobs = 1
  Maximum Volumes = 4
  Label Format = "Clnt1-fs-Full-"
}
Pool {
  Name = Clnt1-fs-Diff
  Pool Type = Backup
  Recycle = yes
  AutoPrune = yes
  Recycle Oldest Volume = yes
  Volume Retention = 31  days
  Maximum Volume Bytes = 1G
  Maximum Volume Jobs = 31
  Maximum Volumes = 2
  Label Format = "Clnt1-fs-Diff-"
}
Pool {
  Name = Clnt1-fs-Incr
  Pool Type = Backup
  Recycle = yes
  AutoPrune = yes
  Recycle Oldest Volume = yes
  Volume Retention = 7   days
  Maximum Volume Bytes = 1G
  Maximum Volume Jobs = 22
  Maximum Volumes = 2
  Label Format = "Clnt1-fs-Incr-"
}

# File Pool definition Clnt2-fs
Pool {
  Name = Clnt2-fs-Monthly
  Pool Type = Backup
  Recycle = yes                         # Bacula can automatically recycle Volumes
  AutoPrune = yes                       # Prune expired volumes
  Recycle Oldest Volume = yes           # Prune the oldest volume in the Pool, and if all files were pruned, recycle this volume and use it.
  Volume Retention = 365  days          # How long should the Full Backups be kept? (#06)
  Maximum Volume Bytes = 1G             # Limit Volume size to something reasonable
  Maximum Volume Jobs = 1               # One Job = One Vol
  Maximum Volumes = 12                  # Limit number of Volumes in Pool
  Label Format = "Clnt2-fs-Monthly-"    # Volumes will be labeled "Full-<volume-id>"
}
Pool {
  Name = Clnt2-fs-Full
  Pool Type = Backup
  Recycle = yes
  AutoPrune = yes
  Recycle Oldest Volume = yes
  Volume Retention = 92  days
  Maximum Volume Bytes = 1G
  Maximum Volume Jobs = 1
  Maximum Volumes = 4
  Label Format = "Clnt2-fs-Full-"
}
Pool {
  Name = Clnt2-fs-Diff
  Pool Type = Backup
  Recycle = yes
  AutoPrune = yes
  Recycle Oldest Volume = yes
  Volume Retention = 31  days
  Maximum Volume Bytes = 1G
  Maximum Volume Jobs = 31
  Maximum Volumes = 2
  Label Format = "Clnt2-fs-Diff-"
}
Pool {
  Name = Clnt2-fs-Incr
  Pool Type = Backup
  Recycle = yes
  AutoPrune = yes
  Recycle Oldest Volume = yes
  Volume Retention = 7   days
  Maximum Volume Bytes = 1G
  Maximum Volume Jobs = 22
  Maximum Volumes = 2
  Label Format = "Clnt2-fs-Incr-"
}
```
Здесь:
  - **Recycle** - Эта директива определяет, могут ли быть использованы повторно очищенные тома (_Purged Volumes_). Если установлено значение **yes** (по умолчанию) и _Bacula_ нужен том, но не находится ни одного присоединяемого, она будет искать и перерабатывать (повторно использовать) (_recycle (reuse)_) очищенные тома (т. е. тома со всеми просроченными заданиями и файлами, которые, таким образом, удалены из каталога). Если том переработан, все предыдущие данные, записанные в этот том, будут перезаписаны. Если для _Recycle_ установлено значение **no**, том не будет переработан, и, следовательно, данные останутся действительными. Если вы хотите повторно использовать (перезаписать) (_reuse (re-write)_) том, а флаг переработки равен **no** (0 в базе данных _Catalog_), вы можете вручную установить флаг переработки (_recycle flag_) с помощью команды `update Volume` для тома, который будет повторно использован.
  - **AutoPrune** - Если _AutoPrune_ установлен на **yes** (по умолчанию), _Bacula_ (версия 1.20 или выше) автоматически применит _Volume Retention period_, когда потребуется новый том и в пуле не будет томов, которые можно добавить. Удаление тома приводит к удалению из каталога просроченных заданий (старше периода хранения тома) и позволяет возможно повторно использовать том.
  - **Recycle Oldest Volume** - Эта директива предписывает _Director_ искать самый старый используемый том в пуле, когда _Storage daemon_ запрашивает другой том, но ни один не доступен. Затем _Catalog_ очищается с учетом периодов хранения всех файлов и заданий, записанных в этот том. Если все задания (_Job_) очищены (т. е. том очищен), то том перерабатывается (_recycled_) и будет использоваться в качестве следующего тома для записи. Эта директива учитывает любые периоды хранения заданий, файлов или томов, которые вы могли указать, и поэтому гораздо лучше использовать эту директиву, чем _Purge Oldest Volume_.
  - **Volume Retention** - Директива _Volume Retention_ определяет период времени, в течение которого _Bacula_ будет хранить записи, связанные с томом, в базе данных _Catalog_ после времени окончания каждого задания, записанного в том. Когда этот период времени истекает, и если **_AutoPrune_** установлено на **yes**, _Bacula_ может обрезать (удалить) (_prune (remove)_) записи заданий, которые старше указанного периода хранения тома, если это необходимо для освобождения тома. Переработка не будет происходить до тех пор, пока не возникнет абсолютная необходимость освободить том (т. е. не будет другого доступного для записи тома). Все записи файлов, связанные с усечёнными заданиями, также удаляются. Время может быть указано в секундах, минутах, часах, днях, неделях, месяцах, кварталах или годах. Хранение тома применяется независимо от периодов хранения заданий и хранения файлов , определенных в ресурсе клиента. Это означает, что все периоды хранения применяются по очереди, и что более короткий период является тем, который фактически имеет приоритет. Обратите внимание, что когда период сохранения тома истекает и необходимо получить новый том, _Bacula_ удалит записи _Job_ и _File_. Это удалит также во время команды `status dir` , поскольку она использует похожие алгоритмы для поиска следующего доступного тома.
>[!NOTE]
> Важно знать, что когда период сохранения тома истекает, _Bacula_ не перерабатывает том автоматически. Она пытается сохранить данные тома нетронутыми как можно дольше, прежде чем перезаписать том.
  - **Maximum Volume Bytes** - Эта директива определяет максимальное количество байтов, которое может быть записано в том. Если указать ноль (по умолчанию), то ограничений нет, за исключением физического размера тома. В противном случае, когда количество байтов, записанных в том, равно размеру, том будет помечен как Used . Когда том помечен как **_Used_**, он больше не может использоваться для добавления заданий, как и статус **_Full_**, но его можно переработать, если включена переработка, и, таким образом, том может быть повторно использован после переработки. Это значение проверяется, и статус **_Used_** устанавливается, пока задание записывает данные в определенный том.
  - **Maximum Volume Jobs** - Эта директива определяет максимальное количество заданий, которые могут быть записаны в том. Если указать ноль (по умолчанию), ограничений нет. В противном случае, когда количество заданий, сохраненных в томе, равно положительному целому числу, том будет помечен как **_Used_**. Когда том помечен как **_Used_**, он больше не может использоваться для добавления заданий, как и статус **_Full_**, но его можно переработать, если включена переработка, и, таким образом, использовать снова. Установив **_MaximumVolumeJobs_** в единицу, вы получите тот же эффект, что и при установке **_UseVolumeOnce = yes_**.
>[!NOTE]
>Значение, определенное этой директивой в файле _bacula-dir.conf_, является значением по умолчанию, используемым при создании тома. После создания тома изменение значения в файле _bacula-dir.conf_ не изменит то, что хранится для тома. Чтобы изменить значение для существующего тома, необходимо использовать команду `update volume` в консоли.
  - **Maximum Volumes** - Эта директива определяет максимальное количество томов (лент или файлов), содержащихся в пуле. Эта директива необязательна, если ее пропустить или установить в ноль, будет разрешено любое количество томов. В целом, эта директива полезна для Autochanger, где есть фиксированное количество томов, или для хранилища файлов, где вы хотите убедиться, что резервные копии, сделанные на дисковых файлах, не станут слишком многочисленными или не займут слишком много места.

>[!IMPORTANT] 
> **Purge Oldest Volume** - Эта директива предписывает _Director_ искать самый старый используемый Том (_Volume_) в Пуле (_Pool_), когда _Storage daemon_ запрашивает другой _Volume_, но ни один не доступен. Затем _Catalog_ очищается независимо от сроков хранения всех Файлов (_Files_) и Заданий (_Jobs_), записанных в этот Том (_Volume_). Затем Том (_Volume_)перерабатывается и будет использоваться в качестве следующего _Volume_ для записи. Эта директива переопределяет любые сроки хранения Заданий, Файлов или Томов (_Jobs, Files, Volumes_), которые вы могли указать.
> 
> Эта директива может быть полезна, если у вас есть фиксированное количество томов в пуле, и вы хотите циклически проходить по ним и повторно использовать самый старый, когда все тома заполнены, но вы не хотите беспокоиться об установке правильных сроков хранения. Однако, используя эту опцию, вы рискуете потерять ценные данные.
>
> Обратите внимание, что _Purge Oldest Volume_ игнорирует все периоды хранения (_Retention periods_). Если у вас определен только один том и вы включаете эту переменную, этот том всегда будет немедленно перезаписан при заполнении! Поэтому как минимум убедитесь, что у вас есть приличное количество томов в вашем пуле, прежде чем запускать какие-либо задания. Если вы хотите, чтобы применялись периоды хранения, **не используйте эту директиву**!. Чтобы указать период хранения, используйте директиву _Volume Retention_ (см. выше).
>
> Настоятельно рекомендуется **не использовать эту директиву**, поскольку наверняка когда-нибудь _Bacula_ переработает том, содержащий текущие данные. Значение по умолчанию — **no** .

Исходя из вышеуказанного, рассмотрим наш пул _Clnt1-fs-Monthly_, предназначенный для хранения полных - _Full_ резервных копий в течение года (365 дней). Здесь, с помощью параметра `Maximum Volume Jobs = 1` мы задали хранение одного задания резервного копирования в каждом томе, максимальное количество томов в пуле задано с помощью параметра `Maximum Volumes = 12`. Срок хранения тома до того, когда он сможет быть перезаписан определяется параметром `Volume Retention = 365  days`

Примерно такого же эффекта можно достичь, изменим кол-во томов в пуле и время их хранения, например, вот так:

```
Volume Retention = 28  days            # How long should the Full Backups be kept? (#06)
Maximum Volume Bytes = 12G             # Limit Volume size to something reasonable
Maximum Volume Jobs = 12               # One Job = One Vol
Maximum Volumes = 1                    # Limit number of Volumes in Pool
```
Отличие здесь в том, что когда _Storage Daemon_ заполнит том двенадцатью заданиями, он перезапишет единственный имеющийся том и раз в год мы будем терять все архивы, записанные в течение года на данный том. Чтобы избежать подобной ситуации, кол-во томов в пуле можно увеличить до двух. Также рекомендуется увеличить значение параметра `Volume Retention` Вот так:
```
Volume Retention = 365  days           # How long should the Full Backups be kept? (#06)
Maximum Volume Bytes = 12G             # Limit Volume size to something reasonable
Maximum Volume Jobs = 12               # One Job = One Vol
Maximum Volumes = 2                    # Limit number of Volumes in Pool
```
Аналогичным образом настраиваются пулы томов _Clnt1-fs-Full_, _Clnt1-fs-Diff_, _Clnt1-fs-Incr_ - для полных, разностноых и инкрементных резервных копий.

#### Настройка на сервере. Storage Daemon

Блок общих настроек:
```
#
# Default Bacula Storage Daemon Configuration file
#
#  For Bacula release 9.6.7 (10 December 2020) -- debian bookworm/sid
#
# You may need to change the name of your tape drive
#   on the "Archive Device" directive in the Device
#   resource.  If you change the Name and/or the
#   "Media Type" in the Device resource, please ensure
#   that dird.conf has corresponding changes.
#
#
# Copyright (C) 2000-2020 Kern Sibbald
# License: BSD 2-Clause; see file LICENSE-FOSS
#

Storage {                             # definition of myself
  Name = debian12-sd
  SDPort = 9103                  # Director's port
  WorkingDirectory = "/var/lib/bacula"
  Pid Directory = "/run/bacula"
  Plugin Directory = "/usr/lib/bacula"
  Maximum Concurrent Jobs = 20
  SDAddress = 0.0.0.0
}
```
Здесь изменим только сетевой адрес, на котором _Storage Daemon_ ожидает подключения, с Localhost на 0.0.0.0. Таким образом, служба _Storage Daemon_ будет доступна для удалённых клиентов, отправляющих ей свои резервные копии.

Блок _Director_:
```
#
# List Directors who are permitted to contact Storage daemon
#
Director {
  Name = debian12-dir
  Password = "OfxoJ1unN7aMOCzwZ8ag8J5eGdN1de4vK"
}

#
# Restricted Director, used by tray-monitor to get the
#   status of the storage daemon
#
Director {
  Name = debian12-mon
  Password = "FMv1iAMurQAXi1__VQydNI3C-fyOkAyaL"
  Monitor = yes
}
```
Здесь указаны параметры подключения к демону со стороны Директора и службы _Tray-monitor_.

Блок _Device_:
```
Device {
  Name = FileStorage
  Media Type = File
  Archive Device = /var/lib/bacula/storage
  LabelMedia = yes;                   # lets Bareos label unlabeled media
  Random Access = yes;
  AutomaticMount = yes;               # when device opened, read it
  RemovableMedia = no;
  AlwaysOpen = no;
  Description = "File device. A connecting Director must have the same Name and MediaType."
}
```
Здесь:
  - **Media Type** - Тип носителя, поддерживаемый этим устройством, например, _"DLT7000"_. Имена типов носителя произвольны, поскольку вы можете задать им все, что захотите, но они должны быть известны _volume database_, чтобы отслеживать, какие из _Storage Daemons_ могут читать какие тома. В общем случае, каждый отдельный тип хранилища должен иметь уникальный тип носителя, связанный с ним. Та же строка имени должна появляться в соответствующем определении ресурса хранилища в файле конфигурации _Director_.
  - **Archive Device** - Системное имя файла устройства хранения, управляемого этим демоном хранения. Обычно это будет имя файла устройства съемного устройства хранения (ленточного накопителя), например _"/dev/nst0"_ или _"/dev/rmt/0mbn"_. Для _DVD-рекордера_ это будет, например, _/dev/hdc_. Это также может быть имя каталога, если вы архивируете на дисковом хранилище. В этом случае вы должны указать полный абсолютный путь к каталогу.
  - **LabelMedia** - Автоматическая разметка томов.
  - **Random Access** - Если указано **Yes**, предполагается, что архивное устройство является носителем с произвольным доступом, который поддерживает _lseek_ (или _lseek64_, если _Largefile_ включен во время настройки). Для всех файловых систем, таких как _DVD_, _USB_ и фиксированные файлы, следует установить значение **Yes**. Для устройств с непроизвольным доступом, таких как ленты и именованные каналы, следует установить значение **No**.
  - **AlwaysOpen** - Если установлено значение **Yes** (по умолчанию), _Bacula_ всегда будет держать устройство открытым, если оно специально не размонтировано программой _Console_. Это позволяет _Bacula_ гарантировать, что ленточный накопитель всегда доступен и правильно расположен. Если вы установите _AlwaysOpen_ на **no** , _Bacula_ будет открывать накопитель только при необходимости, а в конце задания, если никакие другие задания не используют накопитель, он будет освобожден. В следующий раз, когда _Bacula_ захочет добавить ленту на освобожденный накопитель, _Bacula_ перемотает ленту и установит ее в конец. Чтобы избежать ненужного позиционирования ленты и свести к минимуму ненужное вмешательство оператора, настоятельно рекомендуется установить _Always Open_ = **yes** . Это также гарантирует, что накопитель будет доступен, когда он понадобится _Bacula_.

### Практическая часть. Разворачивание сервера Bacula с настройками, которые были приведены в предыдущей главе

Для развёртывания стенда будем использовать наш [Vagrantfile](files/Vagrantfile). Также будут использоваться файлы с параметрами для сервера _Bacula_ - _Debian12_ и двух клиентов - _Debian12cl1_ и _Debian12cl2_:
  - [bacula-dir-clients.conf](files/srv/bacula-dir-clients.conf)
  - [bacula-dir-filesets.conf](files/srv/bacula-dir-filesets.conf)
  - [bacula-dir-jobs.conf](files/srv/bacula-dir-jobs.conf)
  - [bacula-dir-pools.conf](files/srv/bacula-dir-pools.conf)
  - [bacula-dir-schedules.conf](files/srv/bacula-dir-schedules.conf)
  - [bacula-dir-storage.conf](files/srv/bacula-dir-storage.conf)
  - [bacula-sd.conf](files/srv/bacula-sd.conf)
  - [bacula-fd.conf](files/clnt1/bacula-fd.conf)
  - [bacula-fd.conf](files/clnt2/bacula-fd.conf)

Дерево каталогов тестового стенда выглядит следующим образом:
```
tree
.
├── bacula
│   ├── clnt1
│   │   └── bacula-fd.conf
│   ├── clnt2
│   │   └── bacula-fd.conf
│   └── srv1
│       ├── bacula-dir-clients.conf
│       ├── bacula-dir.conf
│       ├── bacula-dir-filesets.conf
│       ├── bacula-dir-jobs.conf
│       ├── bacula-dir-pools.conf
│       ├── bacula-dir-schedules.conf
│       ├── bacula-dir-storage.conf
│       ├── bacula-sd.conf
└── Vagrantfile
```
Параметры, задаваемые в данных конфигурационных файлах из каталога _files/srv_, добавляются в главный конфиг _Director_ - [bacula-dir.conf](Files/bacula-dir.conf), из каталогов _clnt1_ и _clnt2_ - заменяют параметры, созданные установщиком.

C помощью прилагаемого [Vagrantfile](Vagrantfile) разворачиваем стенд.

#### Создание резервных копий
Логинимся на сервер и создаём полные резервные копии для файлов, перечисленных в параметре _FileSet_ для первого клиента - _Debian12cl1-fd_:
```
root@debian12:~# bconsole 
Connecting to Director localhost:9101
1000 OK: 103 debian12-dir Version: 9.6.7 (10 December 2020)
Enter a period to cancel a command.
*run job=
BackupCatalog   BackupClient1   Clnt1-fs-Job    Clnt2-fs-Job    MyRestoreFiles  RestoreFiles    
*run job=Clnt1-fs-Job 
Using Catalog "MyCatalog"
Run Backup job
JobName:  Clnt1-fs-Job
Level:    Incremental
Client:   Debian12cl1-fd
FileSet:  My-tfs-FS
Pool:     Clnt1-fs-Full (From Job resource)
Storage:  debian12-sd (From Job resource)
When:     2024-09-17 17:13:06
Priority: 10
OK to run? (yes/mod/no): m
Parameters to modify:
     1: Level
     2: Storage
     3: Job
     4: FileSet
     5: Client
     6: When
     7: Priority
     8: Pool
     9: Plugin Options
Select parameter to modify (1-9): 1
Levels:
     1: Full
     2: Incremental
     3: Differential
     4: Since
     5: VirtualFull
Select level (1-5): 1
Run Backup job
JobName:  Clnt1-fs-Job
Level:    Full
Client:   Debian12cl1-fd
FileSet:  My-tfs-FS
Pool:     Clnt1-fs-Full (From Job resource)
Storage:  debian12-sd (From Job resource)
When:     2024-09-17 17:13:06
Priority: 10
OK to run? (yes/mod/no): y
Job queued. JobId=1
```
Здесь мы изменили с помощью модификатора _mod_ параметры по умолчанию:
```
JobName:  Clnt1-fs-Job
Level:    Incremental 
Client:   Debian12cl1-fd
FileSet:  My-tfs-FS
Pool:     Clnt1-fs-Full (From Job resource)
Storage:  debian12-sd (From Job resource)  
When:     2024-09-17 17:13:06
Priority: 10
```
на:
```
JobName:  Clnt1-fs-Job
Level:    Full
Client:   Debian12cl1-fd
FileSet:  My-tfs-FS
Pool:     Clnt1-fs-Full (From Job resource)
Storage:  debian12-sd (From Job resource)  
When:     2024-09-17 17:13:06
Priority: 10
```
Т.е. изменили уровень резервной копии на _Full_.

Теперь тоже самое для второго клиента - _Debian12cl2-fd_:
```
*run job=Clnt2-fs-Job 
Run Backup job
JobName:  Clnt2-fs-Job
Level:    Incremental
Client:   Debian12cl2-fd
FileSet:  My-fs-FS
Pool:     Clnt2-fs-Full (From Job resource)
Storage:  debian12-sd (From Job resource)
When:     2024-09-17 17:14:07
Priority: 10
OK to run? (yes/mod/no): m
Parameters to modify:
     1: Level
     2: Storage
     3: Job
     4: FileSet
     5: Client
     6: When
     7: Priority
     8: Pool
     9: Plugin Options
Select parameter to modify (1-9): 1
Levels:
     1: Full
     2: Incremental
     3: Differential
     4: Since
     5: VirtualFull
Select level (1-5): 1
Run Backup job
JobName:  Clnt2-fs-Job
Level:    Full
Client:   Debian12cl2-fd
FileSet:  My-fs-FS
Pool:     Clnt2-fs-Full (From Job resource)
Storage:  debian12-sd (From Job resource)
When:     2024-09-17 17:14:07
Priority: 10
OK to run? (yes/mod/no): y
Job queued. JobId=2
```
Проверим статус запущенных задач через сообщения:
```
*messages 
17-сен 17:13 debian12-dir JobId 1: Start Backup JobId 1, Job=Clnt1-fs-Job.2024-09-17_17.13.54_03
17-сен 17:13 debian12-dir JobId 1: Created new Volume="Clnt1-fs-Full-0001", Pool="Clnt1-fs-Full", MediaType="File" in catalog.
17-сен 17:13 debian12-dir JobId 1: Using Device "FileStorage" to write.
17-сен 17:13 debian12-fd JobId 1: shell command: run ClientRunBeforeJob "/etc/bacula/scripts/bacula-before-fs.sh"
17-сен 17:13 debian12-fd JobId 1: ClientRunBeforeJob: tar: Удаляется начальный `/' из имен объектов
17-сен 17:13 debian12-fd JobId 1: ClientRunBeforeJob: tar: Удаляются начальные `/' из целей жестких ссылок
17-сен 17:14 debian12-sd JobId 1: Labeled new Volume "Clnt1-fs-Full-0001" on File device "FileStorage" (/var/lib/bacula/storage).
17-сен 17:14 debian12-sd JobId 1: Wrote label to prelabeled Volume "Clnt1-fs-Full-0001" on File device "FileStorage" (/var/lib/bacula/storage)
17-сен 17:14 debian12-dir JobId 1: Max Volume jobs=1 exceeded. Marking Volume "Clnt1-fs-Full-0001" as Used.
17-сен 17:14 debian12-dir JobId 2: Start Backup JobId 2, Job=Clnt2-fs-Job.2024-09-17_17.14.21_04
17-сен 17:14 debian12-fd JobId 1: shell command: run ClientAfterJob "/etc/bacula/scripts/bacula-after-fs.sh"
17-сен 17:14 debian12-sd JobId 1: Elapsed time=00:00:30, Transfer rate=6.501 M Bytes/second
17-сен 17:14 debian12-sd JobId 1: Sending spooled attrs to the Director. Despooling 224 bytes ...
17-сен 17:14 debian12-dir JobId 1: Bacula debian12-dir 9.6.7 (10Dec20):
  Build OS:               x86_64-pc-linux-gnu debian bookworm/sid
  JobId:                  1
  Job:                    Clnt1-fs-Job.2024-09-17_17.13.54_03
  Backup Level:           Full
  Client:                 "Debian12cl1-fd" 9.6.7 (10Dec20) x86_64-pc-linux-gnu,debian,bookworm/sid
  FileSet:                "My-tfs-FS" 2024-09-17 17:13:54
  Pool:                   "Clnt1-fs-Full" (From Job FullPool override)
  Catalog:                "MyCatalog" (From Client resource)
  Storage:                "debian12-sd" (From Job resource)
  Scheduled time:         17-сен-2024 17:13:06
  Start time:             17-сен-2024 17:14:00
  End time:               17-сен-2024 17:14:30
  Elapsed time:           30 secs
  Priority:               10
  FD Files Written:       1
  SD Files Written:       1
  FD Bytes Written:       195,052,043 (195.0 MB)
  SD Bytes Written:       195,052,159 (195.0 MB)
  Rate:                   6501.7 KB/s
  Software Compression:   61.5% 2.6:1
  Comm Line Compression:  None
  Snapshot/VSS:           no
  Encryption:             no
  Accurate:               no
  Volume name(s):         Clnt1-fs-Full-0001
  Volume Session Id:      1
  Volume Session Time:    1726582003
  Last Volume Bytes:      195,254,627 (195.2 MB)
  Non-fatal FD errors:    0
  SD Errors:              0
  FD termination status:  OK
  SD termination status:  OK
  Termination:            Backup OK

17-сен 17:14 debian12-dir JobId 1: Begin pruning Jobs older than 12 months .
17-сен 17:14 debian12-dir JobId 1: No Jobs found to prune.
17-сен 17:14 debian12-dir JobId 1: Begin pruning Files.
17-сен 17:14 debian12-dir JobId 1: No Files found to prune.
17-сен 17:14 debian12-dir JobId 1: End auto prune.

17-сен 17:14 debian12-dir JobId 2: Created new Volume="Clnt2-fs-Full-0002", Pool="Clnt2-fs-Full", MediaType="File" in catalog.
17-сен 17:14 debian12-dir JobId 2: Using Device "FileStorage" to write.
17-сен 17:14 debian12-sd JobId 2: Labeled new Volume "Clnt2-fs-Full-0002" on File device "FileStorage" (/var/lib/bacula/storage).
17-сен 17:14 debian12-sd JobId 2: Wrote label to prelabeled Volume "Clnt2-fs-Full-0002" on File device "FileStorage" (/var/lib/bacula/storage)
17-сен 17:14 debian12-dir JobId 2: Max Volume jobs=1 exceeded. Marking Volume "Clnt2-fs-Full-0002" as Used.
*messages
17-сен 17:15 debian12-sd JobId 2: Elapsed time=00:00:15, Transfer rate=16.78 M Bytes/second
17-сен 17:15 debian12-sd JobId 2: Sending spooled attrs to the Director. Despooling 3,355,574 bytes ...
17-сен 17:15 debian12-dir JobId 2: Bacula debian12-dir 9.6.7 (10Dec20):
  Build OS:               x86_64-pc-linux-gnu debian bookworm/sid
  JobId:                  2
  Job:                    Clnt2-fs-Job.2024-09-17_17.14.21_04
  Backup Level:           Full
  Client:                 "Debian12cl2-fd" 9.6.7 (10Dec20) x86_64-pc-linux-gnu,debian,bookworm/sid
  FileSet:                "My-fs-FS" 2024-09-17 17:14:21
  Pool:                   "Clnt2-fs-Full" (From Job FullPool override)
  Catalog:                "MyCatalog" (From Client resource)
  Storage:                "debian12-sd" (From Job resource)
  Scheduled time:         17-сен-2024 17:14:07
  Start time:             17-сен-2024 17:14:53
  End time:               17-сен-2024 17:15:09
  Elapsed time:           16 secs
  Priority:               10
  FD Files Written:       13,887
  SD Files Written:       13,887
  FD Bytes Written:       249,852,594 (249.8 MB)
  SD Bytes Written:       251,802,548 (251.8 MB)
  Rate:                   15615.8 KB/s
  Software Compression:   49.3% 2.0:1
  Comm Line Compression:  2.0% 1.0:1
  Snapshot/VSS:           no
  Encryption:             no
  Accurate:               no
  Volume name(s):         Clnt2-fs-Full-0002
  Volume Session Id:      2
  Volume Session Time:    1726582003
  Last Volume Bytes:      252,483,814 (252.4 MB)
  Non-fatal FD errors:    0
  SD Errors:              0
  FD termination status:  OK
  SD termination status:  OK
  Termination:            Backup OK

17-сен 17:15 debian12-dir JobId 2: Begin pruning Jobs older than 12 months .
17-сен 17:15 debian12-dir JobId 2: No Jobs found to prune.
17-сен 17:15 debian12-dir JobId 2: Begin pruning Files.
17-сен 17:15 debian12-dir JobId 2: No Files found to prune.
17-сен 17:15 debian12-dir JobId 2: End auto prune.
```
Напомню, что параметиры резервного копирования для _Debian12cl1-fd_ и _Debian12cl2-fd_ отличаются. В первом случае, мы с помощью скрипта, запускаемого директивой _RunBeforeJob_ создаем tar-архив без сжатия с каталогами /etc и /var внутри, затем создаем
резервную копию полученного файла, используя сжатие _GZIP_ на стороне клиента. Во втором случае, каталоги /etc и /var уже второго клиента - _Debian12cl2-fd_ непосредственно архивируются демоном _Bacula File Daemon_ с алгоритмом сжатия _LZO_.

Из листинга, полученного командой messages, видно, что коэффициент сжатия _Software Compression_ для первого клиента равен **61.5% 2.6:1**, время работы по созданию резервной копии `Elapsed time: 30 secs` составило 30 сек - сопутствующие параметры `Start time: 17-сен-2024 17:14:00` и `End time: 17-сен-2024 17:14:30`.
У второго клиента - `Elapsed time: 16 secs`, `Start time: 17-сен-2024 17:14:53`, `End time: 17-сен-2024 17:15:09`, при этом `Software Compression: 49.3% 2.0:1`. Размеры резервных копий на томе составили, соответственно, - `Last Volume Bytes: 195,254,627 (195.2 MB)` 
в первом случае и `  Last Volume Bytes:      252,483,814 (252.4 MB)` - во втором.

Создадим тестовый файл на виртуальной машине клиента _Debian12cl2-fd_ в каталоге `/var`:
```
echo 'Hello! Its Test File' > /var/testfile.txt
```

Далее проверим, что уже по расписанию создались инкрементные копии наборов файлов для обоих клиентов:
```
*messages 
18-сен 09:00 debian12-dir JobId 3: Start Backup JobId 3, Job=Clnt1-fs-Job.2024-09-18_09.00.00_06
18-сен 09:00 debian12-dir JobId 3: Created new Volume="Clnt1-fs-Incr-0003", Pool="Clnt1-fs-Incr", MediaType="File" in catalog.
18-сен 09:00 debian12-dir JobId 3: Using Device "FileStorage" to write.
18-сен 09:00 debian12-fd JobId 3: shell command: run ClientRunBeforeJob "/etc/bacula/scripts/bacula-before-fs.sh"
18-сен 09:00 debian12-fd JobId 3: ClientRunBeforeJob: tar: Удаляется начальный `/' из имен объектов
18-сен 09:00 debian12-fd JobId 3: ClientRunBeforeJob: tar: Удаляются начальные `/' из целей жестких ссылок
18-сен 09:00 debian12-sd JobId 3: Labeled new Volume "Clnt1-fs-Incr-0003" on File device "FileStorage" (/var/lib/bacula/storage).
18-сен 09:00 debian12-sd JobId 3: Wrote label to prelabeled Volume "Clnt1-fs-Incr-0003" on File device "FileStorage" (/var/lib/bacula/storage)
18-сен 09:00 debian12-dir JobId 4: Start Backup JobId 4, Job=Clnt2-fs-Job.2024-09-18_09.00.00_07
18-сен 09:00 debian12-fd JobId 3: shell command: run ClientAfterJob "/etc/bacula/scripts/bacula-after-fs.sh"
18-сен 09:00 debian12-sd JobId 3: Elapsed time=00:00:30, Transfer rate=6.515 M Bytes/second
18-сен 09:00 debian12-sd JobId 3: Sending spooled attrs to the Director. Despooling 224 bytes ...
18-сен 09:00 debian12-dir JobId 3: Bacula debian12-dir 9.6.7 (10Dec20):
  Build OS:               x86_64-pc-linux-gnu debian bookworm/sid
  JobId:                  3
  Job:                    Clnt1-fs-Job.2024-09-18_09.00.00_06
  Backup Level:           Incremental, since=2024-09-17 17:14:00
  Client:                 "Debian12cl1-fd" 9.6.7 (10Dec20) x86_64-pc-linux-gnu,debian,bookworm/sid
  FileSet:                "My-tfs-FS" 2024-09-17 17:13:54
  Pool:                   "Clnt1-fs-Incr" (From Job IncPool override)
  Catalog:                "MyCatalog" (From Client resource)
  Storage:                "debian12-sd" (From Job resource)
  Scheduled time:         18-сен-2024 09:00:00
  Start time:             18-сен-2024 09:00:03
  End time:               18-сен-2024 09:00:33
  Elapsed time:           30 secs
  Priority:               10
  FD Files Written:       1
  SD Files Written:       1
  FD Bytes Written:       195,478,009 (195.4 MB)
  SD Bytes Written:       195,478,125 (195.4 MB)
  Rate:                   6515.9 KB/s
  Software Compression:   61.6% 2.6:1
  Comm Line Compression:  None
  Snapshot/VSS:           no
  Encryption:             no
  Accurate:               no
  Volume name(s):         Clnt1-fs-Incr-0003
  Volume Session Id:      3
  Volume Session Time:    1726582003
  Last Volume Bytes:      195,681,169 (195.6 MB)
  Non-fatal FD errors:    0
  SD Errors:              0
  FD termination status:  OK
  SD termination status:  OK
  Termination:            Backup OK

18-сен 09:00 debian12-dir JobId 3: Begin pruning Jobs older than 12 months .
18-сен 09:00 debian12-dir JobId 3: No Jobs found to prune.
18-сен 09:00 debian12-dir JobId 3: Begin pruning Files.
18-сен 09:00 debian12-dir JobId 3: No Files found to prune.
18-сен 09:00 debian12-dir JobId 3: End auto prune.

18-сен 09:00 debian12-dir JobId 4: Created new Volume="Clnt2-fs-Incr-0004", Pool="Clnt2-fs-Incr", MediaType="File" in catalog.
18-сен 09:00 debian12-dir JobId 4: Using Device "FileStorage" to write.
18-сен 09:00 debian12-sd JobId 4: Labeled new Volume "Clnt2-fs-Incr-0004" on File device "FileStorage" (/var/lib/bacula/storage).
18-сен 09:00 debian12-sd JobId 4: Wrote label to prelabeled Volume "Clnt2-fs-Incr-0004" on File device "FileStorage" (/var/lib/bacula/storage)
18-сен 09:00 debian12-sd JobId 4: Elapsed time=00:00:01, Transfer rate=6.556 M Bytes/second
18-сен 09:00 debian12-sd JobId 4: Sending spooled attrs to the Director. Despooling 22,406 bytes ...
18-сен 09:00 debian12-dir JobId 4: Bacula debian12-dir 9.6.7 (10Dec20):
  Build OS:               x86_64-pc-linux-gnu debian bookworm/sid
  JobId:                  4
  Job:                    Clnt2-fs-Job.2024-09-18_09.00.00_07
  Backup Level:           Incremental, since=2024-09-17 17:14:53
  Client:                 "Debian12cl2-fd" 9.6.7 (10Dec20) x86_64-pc-linux-gnu,debian,bookworm/sid
  FileSet:                "My-fs-FS" 2024-09-17 17:14:21
  Pool:                   "Clnt2-fs-Incr" (From Job IncPool override)
  Catalog:                "MyCatalog" (From Client resource)
  Storage:                "debian12-sd" (From Job resource)
  Scheduled time:         18-сен-2024 09:00:00
  Start time:             18-сен-2024 09:00:35
  End time:               18-сен-2024 09:00:36
  Elapsed time:           1 sec
  Priority:               10
  FD Files Written:       115
  SD Files Written:       115
  FD Bytes Written:       6,544,139 (6.544 MB)
  SD Bytes Written:       6,556,501 (6.556 MB)
  Rate:                   6544.1 KB/s
  Software Compression:   68.7% 3.2:1
  Comm Line Compression:  5.4% 1.1:1
  Snapshot/VSS:           no
  Encryption:             no
  Accurate:               no
  Volume name(s):         Clnt2-fs-Incr-0004
  Volume Session Id:      4
  Volume Session Time:    1726582003
  Last Volume Bytes:      6,567,531 (6.567 MB)
  Non-fatal FD errors:    0
  SD Errors:              0
  FD termination status:  OK
  SD termination status:  OK
  Termination:            Backup OK

18-сен 09:00 debian12-dir JobId 4: Begin pruning Jobs older than 12 months .
18-сен 09:00 debian12-dir JobId 4: No Jobs found to prune.
18-сен 09:00 debian12-dir JobId 4: Begin pruning Files.
18-сен 09:00 debian12-dir JobId 4: No Files found to prune.
18-сен 09:00 debian12-dir JobId 4: End auto prune.
```
>[!TIP]
> Можно обратить внимание, что инкрементные копии были созданы уже на следующий день, т.к. ноутбук с работающим стендом на ночь отпрвлялся в режим сна.

Проверим, что в файлах _bootstrap_ на сервере резервных копий появились соответствующие записи:
```
root@debian12:~# cat /var/lib/bacula/Debian12cl1-fd.bsr 
# 17-сен-2024 17:14:30 - Clnt1-fs-Job.2024-09-17_17.13.54_03 - Full
Volume="Clnt1-fs-Full-0001"
MediaType="File"
VolSessionId=1
VolSessionTime=1726582003
VolAddr=238-195254626
FileIndex=1-1
# 18-сен-2024 09:00:33 - Clnt1-fs-Job.2024-09-18_09.00.00_06 - Incremental, since=2024-09-17 17:14:00
Volume="Clnt1-fs-Incr-0003"
MediaType="File"
VolSessionId=3
VolSessionTime=1726582003
VolAddr=238-195681168
FileIndex=1-1
root@debian12:~# cat /var/lib/bacula/Debian12cl2-fd.bsr 
# 17-сен-2024 17:15:09 - Clnt2-fs-Job.2024-09-17_17.14.21_04 - Full
Volume="Clnt2-fs-Full-0002"
MediaType="File"
VolSessionId=2
VolSessionTime=1726582003
VolAddr=238-252483813
FileIndex=1-13887
# 18-сен-2024 09:00:36 - Clnt2-fs-Job.2024-09-18_09.00.00_07 - Incremental, since=2024-09-17 17:14:53
Volume="Clnt2-fs-Incr-0004"
MediaType="File"
VolSessionId=4
VolSessionTime=1726582003
VolAddr=238-6567530
FileIndex=1-115
```
Также проверим наличие физических файлов томов:
```
root@debian12:~# ls -l /var/lib/bacula/storage/
итого 634764
-rw-r----- 1 bacula tape 195254627 сен 17 17:14 Clnt1-fs-Full-0001
-rw-r----- 1 bacula tape 195681169 сен 18 09:00 Clnt1-fs-Incr-0003
-rw-r----- 1 bacula tape 252483814 сен 17 17:15 Clnt2-fs-Full-0002
-rw-r----- 1 bacula tape   6567531 сен 18 09:00 Clnt2-fs-Incr-0004
```

#### Восстановление информации
Удалим ранее созданный тестовый файл с клиентской машины и затем восстановим его из копии:
```
max@localhost:~/vagrant/vg3> vagrant ssh Debian12cl2 
Linux debian12 6.1.0-22-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.94-1 (2024-06-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Sep 18 08:57:17 2024 from 192.168.121.1
vagrant@debian12:~$ sudo -i
root@debian12:~# ls -l /var/
итого 40
drwxr-xr-x  2 root root  4096 сен 18 08:52 backups
drwxr-xr-x 19 root root  4096 сен 18 08:52 cache
drwxr-xr-x 54 root root  4096 сен 17 17:03 lib
drwxrwsr-x  2 root staff 4096 янв 29  2024 local
lrwxrwxrwx  1 root root     9 мая 19 23:39 lock -> /run/lock
drwxr-xr-x 11 root root  4096 сен 18 08:52 log
drwxrwsr-x  2 root mail  4096 мая 19 23:39 mail
drwxr-xr-x  2 root root  4096 мая 19 23:39 opt
lrwxrwxrwx  1 root root     4 мая 19 23:39 run -> /run
drwxr-xr-x  6 root root  4096 мая 20 00:00 spool
-rw-r--r--  1 root root    21 сен 18 08:57 testfile.txt
drwxrwxrwt 11 root root  4096 сен 18 08:52 tmp
root@debian12:~# rm /var/testfile.txt 
root@debian12:~# ls -l /var/
итого 36
drwxr-xr-x  2 root root  4096 сен 18 08:52 backups
drwxr-xr-x 19 root root  4096 сен 18 08:52 cache
drwxr-xr-x 54 root root  4096 сен 17 17:03 lib
drwxrwsr-x  2 root staff 4096 янв 29  2024 local
lrwxrwxrwx  1 root root     9 мая 19 23:39 lock -> /run/lock
drwxr-xr-x 11 root root  4096 сен 18 08:52 log
drwxrwsr-x  2 root mail  4096 мая 19 23:39 mail
drwxr-xr-x  2 root root  4096 мая 19 23:39 opt
lrwxrwxrwx  1 root root     4 мая 19 23:39 run -> /run
drwxr-xr-x  6 root root  4096 мая 20 00:00 spool
drwxrwxrwt 11 root root  4096 сен 18 08:52 tmp
```

Теперь на сервере заходим в консоль _Bacula_ и запускаем процесс восстановления, с этим нам поможет команда `restore`:
```
root@debian12:~# bconsole 
Connecting to Director localhost:9101
1000 OK: 103 debian12-dir Version: 9.6.7 (10 December 2020)
Enter a period to cancel a command.
*restore 
Automatically selected Catalog: MyCatalog
Using Catalog "MyCatalog"

First you select one or more JobIds that contain files
to be restored. You will be presented several methods
of specifying the JobIds. Then you will be allowed to
select which files from those JobIds are to be restored.

To select the JobIds, you have the following choices:
     1: List last 20 Jobs run
     2: List Jobs where a given File is saved
     3: Enter list of comma separated JobIds to select
     4: Enter SQL list command
     5: Select the most recent backup for a client
     6: Select backup for a client before a specified time
     7: Enter a list of files to restore
     8: Enter a list of files to restore before a specified time
     9: Find the JobIds of the most recent backup for a client
    10: Find the JobIds for a backup for a client before a specified time
    11: Enter a list of directories to restore for found JobIds
    12: Select full restore to a specified Job date
    13: Cancel
```
Здесь нам предложили выбрать, каким образом мы будем искать нужный файл. Выберем пункт _5_:
```
Select item:  (1-13): 5
Defined Clients:
     1: Debian12cl1-fd
     2: Debian12cl2-fd
     3: debian12-fd
Select the Client (1-3): 2
Automatically selected FileSet: My-fs-FS
+-------+-------+----------+-------------+---------------------+--------------------+
| jobid | level | jobfiles | jobbytes    | starttime           | volumename         |
+-------+-------+----------+-------------+---------------------+--------------------+
|     2 | F     |   13,887 | 249,852,594 | 2024-09-17 17:14:53 | Clnt2-fs-Full-0002 |
|     4 | I     |      115 |   6,544,139 | 2024-09-18 09:00:35 | Clnt2-fs-Incr-0004 |
+-------+-------+----------+-------------+---------------------+--------------------+
You have selected the following JobIds: 2,4

Building directory tree for JobId(s) 2,4 ...  ++++++++++++++++++++++++++++++++++++++++++++++++
13,411 files inserted into the tree.

You are now entering file selection mode where you add (mark) and
remove (unmark) files to be restored. No files are initially added, unless
you used the "all" keyword on the command line.
Enter "done" to leave this mode.

cwd is: /
```
После того, как мы выбрали просмотр последней резервной копии, мы выбираем нужный _Bacula Client_ или _Bacula FD_ и попадаем в дерево каталогов, которое содержится в резервной копии.

Перейдём в нужный каталог и выберем наш тестовый файл с помощью команды `mark`:
```
$ cd /var
cwd is: /var/
$ ls
backups/
cache/
lib/
local
lock
log/
mail
opt
run
spool/
testfile.txt
tmp/
$ mark testfile.txt 
1 file marked.
$ done
Bootstrap records written to /var/lib/bacula/debian12-dir.restore.1.bsr

The Job will require the following (*=>InChanger):
   Volume(s)                 Storage(s)                SD Device(s)
===========================================================================
   
    Clnt2-fs-Incr-0004        debian12-sd               FileStorage              

Volumes marked with "*" are in the Autochanger.


1 file selected to be restored.
```
Завершается выбор командой `done`. Далее нам предлагают выбрать задачу для восстановления и проверить параметры, которые в ней заданы:
```
The defined Restore Job resources are:
     1: RestoreFiles
     2: MyRestoreFiles
Select Restore Job (1-2): 2
Using Catalog "MyCatalog"
Run Restore job
JobName:         MyRestoreFiles
Bootstrap:       /var/lib/bacula/debian12-dir.restore.1.bsr
Where:           /bacula-restores
Replace:         Always
FileSet:         Full Set
Backup Client:   Debian12cl2-fd
Restore Client:  Debian12cl2-fd
Storage:         debian12-sd
When:            2024-09-18 09:15:58
Catalog:         MyCatalog
Priority:        10
Plugin Options:  *None*
```
Изменим параметр **Where**, чтобы восстановить файл в тоже место, где он размещался изначально, и запустим задачу:
```
OK to run? (yes/mod/no): m
Parameters to modify:
     1: Level
     2: Storage
     3: Job
     4: FileSet
     5: Restore Client
     6: When
     7: Priority
     8: Bootstrap
     9: Where
    10: File Relocation
    11: Replace
    12: JobId
    13: Plugin Options
Select parameter to modify (1-13): 9
Please enter the full path prefix for restore (/ for none): /
Run Restore job
JobName:         MyRestoreFiles
Bootstrap:       /var/lib/bacula/debian12-dir.restore.1.bsr
Where:           
Replace:         Always
FileSet:         Full Set
Backup Client:   Debian12cl2-fd
Restore Client:  Debian12cl2-fd
Storage:         debian12-sd
When:            2024-09-18 09:15:58
Catalog:         MyCatalog
Priority:        10
Plugin Options:  *None*
OK to run? (yes/mod/no): yes
Job queued. JobId=5
```
Проверим её статус:
```
*messages 
18-сен 09:16 debian12-dir JobId 5: Start Restore Job MyRestoreFiles.2024-09-18_09.16.38_10
18-сен 09:16 debian12-dir JobId 5: Restoring files from JobId(s) 2,4
18-сен 09:16 debian12-dir JobId 5: Using Device "FileStorage" to read.
18-сен 09:16 debian12-sd JobId 5: Ready to read from volume "Clnt2-fs-Incr-0004" on File device "FileStorage" (/var/lib/bacula/storage).
18-сен 09:16 debian12-sd JobId 5: Forward spacing Volume "Clnt2-fs-Incr-0004" to addr=238
18-сен 09:16 debian12-sd JobId 5: Elapsed time=00:00:01, Transfer rate=149  Bytes/second
18-сен 09:16 debian12-dir JobId 5: Bacula debian12-dir 9.6.7 (10Dec20):
  Build OS:               x86_64-pc-linux-gnu debian bookworm/sid
  JobId:                  5
  Job:                    MyRestoreFiles.2024-09-18_09.16.38_10
  Restore Client:         Debian12cl2-fd
  Where:                  
  Replace:                Always
  Start time:             18-сен-2024 09:16:40
  End time:               18-сен-2024 09:16:40
  Elapsed time:           1 sec
  Files Expected:         1
  Files Restored:         1
  Bytes Restored:         21 (21 B)
  Rate:                   0.0 KB/s
  FD Errors:              0
  FD termination status:  OK
  SD termination status:  OK
  Termination:            Restore OK

18-сен 09:16 debian12-dir JobId 5: Begin pruning Jobs older than 12 months .
18-сен 09:16 debian12-dir JobId 5: No Jobs found to prune.
18-сен 09:16 debian12-dir JobId 5: Begin pruning Files.
18-сен 09:16 debian12-dir JobId 5: No Files found to prune.
18-сен 09:16 debian12-dir JobId 5: End auto prune.
```
А также наличие восстановленного файла на своём месте:
```
max@localhost:~/vagrant/vg3> vagrant ssh Debian12cl2 
Linux debian12 6.1.0-22-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.94-1 (2024-06-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Sep 18 09:09:52 2024 from 192.168.121.1
vagrant@debian12:~$ ls -l /var/
итого 40
drwxr-xr-x  2 root root  4096 сен 18 08:52 backups
drwxr-xr-x 19 root root  4096 сен 18 08:52 cache
drwxr-xr-x 54 root root  4096 сен 17 17:03 lib
drwxrwsr-x  2 root staff 4096 янв 29  2024 local
lrwxrwxrwx  1 root root     9 мая 19 23:39 lock -> /run/lock
drwxr-xr-x 11 root root  4096 сен 18 08:52 log
drwxrwsr-x  2 root mail  4096 мая 19 23:39 mail
drwxr-xr-x  2 root root  4096 мая 19 23:39 opt
lrwxrwxrwx  1 root root     4 мая 19 23:39 run -> /run
drwxr-xr-x  6 root root  4096 мая 20 00:00 spool
-rw-r--r--  1 root root    21 сен 18 08:57 testfile.txt
drwxrwxrwt 11 root root  4096 сен 18 08:52 tmp
vagrant@debian12:~$ cat /var/testfile.txt 
Hello! Its Test File
```

##### Восстановление информации в базе данных _Catalog_ в случае её утери или повреждения
Данный раздел должен начинаться с цитаты, размещённой на странице документации утилиты _bscan_: **_Если вы обнаружили, что используете эту программу, вы, вероятно, сделали что-то неправильно._**.

В любом случае, как правило, база данных _Cataliog_ размещается отдельно от службы _Storage Daemon_ и существует, хоть и небольшая, но вероятность того, что SQL-кластер повредился в результате выхода из строя дисков или серверов или
вследствие действия зловредного ПО. При этом физические тома находятся в целостности и сохранности.

Воспроизвведём такую ситуацию. Скопируем _bootstrap_-файлы и файлы томов - _Volumes_ на хостовую машину для последующего восстановления каталога с нуля. Для этого на сервере _Bacula_ скопируем нужные файлы в домашний каталог пользователя _Vagrant_ и изменим их _владельца_ и _группу_:
```
root@debian12:~# cp -R /var/lib/bacula/storage /home/vagrant/
root@debian12:~# cp /var/lib/bacula/Debian12cl* /home/vagrant/

root@debian12:~# chown -R vagrant:vagrant /home/vagrant/storage
root@debian12:~# chown vagrant:vagrant /home/vagrant/Debian12cl*
```
Теперь перейдём на хост и скопируем файлы с виртуальной машины:
```
max@localhost:~/vagrant/vg3> scp vagrant@192.168.121.10:/home/vagrant/storage/Clnt* bacula/srv1/
vagrant@192.168.121.10's password: 
Clnt1-fs-Full-0001                                                                                         100%  186MB  81.2MB/s   00:02    
Clnt1-fs-Incr-0003                                                                                         100%  187MB 104.9MB/s   00:01    
Clnt2-fs-Full-0002                                                                                         100%  241MB  89.1MB/s   00:02    
Clnt2-fs-Incr-0004                                                                                         100% 6414KB  79.5MB/s   00:00    
max@localhost:~/vagrant/vg3> scp vagrant@192.168.121.10:/home/vagrant/Debian* bacula/srv1/
vagrant@192.168.121.10's password: 
Debian12cl1-fd.bsr                                                                                         100%  420   645.2KB/s   00:00    
Debian12cl2-fd.bsr                                                                                         100%  424   682.0KB/s   00:00
```
Удалим все виртуальные машины с помощью команды `vagrant destroy -f` и создадим их заново командой `vagrant up`. Предварительно отключим наши расписания в файле **bacula-dir-schedules.conf** каталога srv1 на хостовой машине, установив значение параметра _Enabled_ в `no`:
```
Schedule {
  Enabled = no
  Name = "Clnt1-fs-Sdl"
...
Schedule {
  Enabled = no
  Name = "Clnt2-fs-Sdl"
...

```
После успешного разворачивания стенда, зайдем на сервер и узнаем, какой пароль для подключения к базе данных _Catalog_ был задан при установке _Bacula Director_. Он потребуется для восстановления _Catalog_. Для этого найдём строку:
```
dbname = "bacula"; DB Address = "localhost"; dbuser = "bacula"; dbpassword = "5vjqDSZc5cCG"
```
Загрузим наши файлы _bootstrap_ (необязательно), а также файлы томов _Volume_ с хоста на виртуальную машину в их исходное место на сервере _Bacula_, 
после чего изменим владельца и группу на исходные значения. 

Для файлов _Debian12cl1-fd.bsr_ и _Debian12cl2-fd.bsr_ это `bacula:bacula`, для томов _Clnt1-fs-Full-0001_, _Clnt1-fs-Incr-0003_, _Clnt2-fs-Full-0002_ и _Clnt2-fs-Incr-0004_ - `bacula:tape`.

Просканируем первый том, пока что без внесения изменений в _Catalog_:
```
root@debian12:~# bscan -c /etc/bacula/bacula-sd.conf -h localhost -P "5vjqDSZc5cCG" -v -V Clnt1-fs-Full-0001 /var/lib/bacula/storage
bscan: butil.c:292-0 Using device: "/var/lib/bacula/storage" for reading.
18-сен 11:01 bscan JobId 0: Ready to read from volume "Clnt1-fs-Full-0001" on File device "FileStorage" (/var/lib/bacula/storage).
bscan: bscan.c:323-0 Using Database: bacula, User: bacula
bscan: bscan.c:468-0 Pool record for Clnt1-fs-Full found in DB.
bscan: bscan.c:482-0 Pool type "Backup" is OK.
bscan: bscan.c:499-0 VOL_LABEL: Media record not found for Volume: Clnt1-fs-Full-0001
bscan: bscan.c:510-0 Media type "File" is OK.
bscan: bscan.c:519-0 VOL_LABEL: OK for Volume: Clnt1-fs-Full-0001
bscan: bscan.c:542-0 SOS_LABEL: Job record not found for JobId: 0
18-сен 11:01 bscan JobId 0: End of Volume "Clnt1-fs-Full-0001" at addr=195254627 on device "FileStorage" (/var/lib/bacula/storage).
bscan: bscan.c:669-0 End of all Volumes. VolFiles=0 VolBlocks=0 VolBytes=195,145,465
Records would have been added or updated in the catalog:
      1 Media
      1 Pool
      1 Job
      1 File
```
Видно, что _bscan_ обнаружила один файл, очевидно, это наш tar-архив, созданный скриптом _RunBeforeJob_.

Просканируем второй том, тоже без внесения изменений в _Catalog_:
```
root@debian12:~# bscan -c /etc/bacula/bacula-sd.conf -h localhost -P "5vjqDSZc5cCG" -v -V Clnt2-fs-Full-0002 /var/lib/bacula/storage
bscan: butil.c:292-0 Using device: "/var/lib/bacula/storage" for reading.
18-сен 11:02 bscan JobId 0: Ready to read from volume "Clnt2-fs-Full-0002" on File device "FileStorage" (/var/lib/bacula/storage).
bscan: bscan.c:323-0 Using Database: bacula, User: bacula
bscan: bscan.c:468-0 Pool record for Clnt2-fs-Full found in DB.
bscan: bscan.c:482-0 Pool type "Backup" is OK.
bscan: bscan.c:499-0 VOL_LABEL: Media record not found for Volume: Clnt2-fs-Full-0002
bscan: bscan.c:510-0 Media type "File" is OK.
bscan: bscan.c:519-0 VOL_LABEL: OK for Volume: Clnt2-fs-Full-0002
bscan: bscan.c:542-0 SOS_LABEL: Job record not found for JobId: 0
18-сен 11:02 bscan JobId 0: End of Volume "Clnt2-fs-Full-0002" at addr=252483814 on device "FileStorage" (/var/lib/bacula/storage).
bscan: bscan.c:669-0 End of all Volumes. VolFiles=0 VolBlocks=0 VolBytes=252,342,768
Records would have been added or updated in the catalog:
      1 Media
      1 Pool
      1 Job
  13887 File
```
Здесь уже мы видим 13887 файлов, т.е. содержимое каталогов _/etc_ и _/var_ на клиентской машине.

Попробуем восстановить зписи в базе данных, соответствующие нашему первому тому. Для этого добавим ключи _-s_ и _-m_. Информация по ключам доступна в `man bscan`:
```
root@debian12:~# bscan -c /etc/bacula/bacula-sd.conf -h localhost -P "5vjqDSZc5cCG" -s -m -v -V Clnt1-fs-Full-0001 /var/lib/bacula/storage
bscan: butil.c:292-0 Using device: "/var/lib/bacula/storage" for reading.
18-сен 11:17 bscan JobId 0: Ready to read from volume "Clnt1-fs-Full-0001" on File device "FileStorage" (/var/lib/bacula/storage).
bscan: bscan.c:323-0 Using Database: bacula, User: bacula
bscan: bscan.c:468-0 Pool record for Clnt1-fs-Full found in DB.
bscan: bscan.c:482-0 Pool type "Backup" is OK.
bscan: bscan.c:996-0 Created Media record for Volume: Clnt1-fs-Full-0001
bscan: bscan.c:510-0 Media type "File" is OK.
bscan: bscan.c:519-0 VOL_LABEL: OK for Volume: Clnt1-fs-Full-0001
bscan: bscan.c:1067-0 Created Client record for Client: Debian12cl1-fd
bscan: bscan.c:1149-0 Created new JobId=1 record for original JobId=1
bscan: bscan.c:1093-0 Created FileSet record "My-tfs-FS"
bscan: bscan.c:1214-0 Updated Job termination record for JobId=1 Level=Full TermStat=T
bscan: bscan.c:1303-0 Created JobMedia record JobId 1, MediaId 1
18-сен 11:17 bscan JobId 0: End of Volume "Clnt1-fs-Full-0001" at addr=195254627 on device "FileStorage" (/var/lib/bacula/storage).
bscan: bscan.c:1017-0 Updated Media record at end of Volume: Clnt1-fs-Full-0001
bscan: bscan.c:1017-0 Updated Media record at end of Volume: Clnt1-fs-Full-0001
bscan: bscan.c:669-0 End of all Volumes. VolFiles=0 VolBlocks=0 VolBytes=195,145,465
Records added or updated in the catalog:
      1 Media
      1 Pool
      1 Job
      1 File
```
Проверим результат:
```
root@debian12:~# bconsole 
Connecting to Director localhost:9101
1000 OK: 103 debian12-dir Version: 9.6.7 (10 December 2020)
Enter a period to cancel a command.
*list pools 
Automatically selected Catalog: MyCatalog
Using Catalog "MyCatalog"
+--------+------------------+---------+---------+----------------+--------------+---------+----------+-------------------+
| poolid | name             | numvols | maxvols | maxvolbytes    | volretention | enabled | pooltype | labelformat       |
+--------+------------------+---------+---------+----------------+--------------+---------+----------+-------------------+
|      1 | Default          |       0 |     100 | 53,687,091,200 |   31,536,000 |       1 | Backup   | *                 |
|      2 | File             |       0 |     100 | 53,687,091,200 |   31,536,000 |       1 | Backup   | Vol-              |
|      3 | Scratch          |       0 |       0 |              0 |   31,536,000 |       1 | Backup   | *                 |
|      4 | Clnt1-fs-Monthly |       0 |      12 |  1,073,741,824 |   31,536,000 |       1 | Backup   | Clnt1-fs-Monthly- |
|      5 | Clnt1-fs-Full    |       0 |       4 |  1,073,741,824 |    7,948,800 |       1 | Backup   | Clnt1-fs-Full-    |
|      6 | Clnt1-fs-Diff    |       0 |       2 |  1,073,741,824 |    2,678,400 |       1 | Backup   | Clnt1-fs-Diff-    |
|      7 | Clnt1-fs-Incr    |       0 |       2 |  1,073,741,824 |      604,800 |       1 | Backup   | Clnt1-fs-Incr-    |
|      8 | Clnt2-fs-Monthly |       0 |      12 |  1,073,741,824 |   31,536,000 |       1 | Backup   | Clnt2-fs-Monthly- |
|      9 | Clnt2-fs-Full    |       0 |       4 |  1,073,741,824 |    7,948,800 |       1 | Backup   | Clnt2-fs-Full-    |
|     10 | Clnt2-fs-Diff    |       0 |       2 |  1,073,741,824 |    2,678,400 |       1 | Backup   | Clnt2-fs-Diff-    |
|     11 | Clnt2-fs-Incr    |       0 |       2 |  1,073,741,824 |      604,800 |       1 | Backup   | Clnt2-fs-Incr-    |
+--------+------------------+---------+---------+----------------+--------------+---------+----------+-------------------+
```
Как мы можем увидеть, в столбце _numvols_ для всех пулов установлены нулеывые значения. Т.е. новый том не появился. Не паникуем, перезагружаем _Director_ и проверяем снова:
```
*reload
*list pools
+--------+------------------+---------+---------+----------------+--------------+---------+----------+-------------------+
| poolid | name             | numvols | maxvols | maxvolbytes    | volretention | enabled | pooltype | labelformat       |
+--------+------------------+---------+---------+----------------+--------------+---------+----------+-------------------+
|      1 | Default          |       0 |     100 | 53,687,091,200 |   31,536,000 |       1 | Backup   | *                 |
|      2 | File             |       0 |     100 | 53,687,091,200 |   31,536,000 |       1 | Backup   | Vol-              |
|      3 | Scratch          |       0 |       0 |              0 |   31,536,000 |       1 | Backup   | *                 |
|      4 | Clnt1-fs-Monthly |       0 |      12 |  1,073,741,824 |   31,536,000 |       1 | Backup   | Clnt1-fs-Monthly- |
|      5 | Clnt1-fs-Full    |       1 |       4 |  1,073,741,824 |    7,948,800 |       1 | Backup   | Clnt1-fs-Full-    |
|      6 | Clnt1-fs-Diff    |       0 |       2 |  1,073,741,824 |    2,678,400 |       1 | Backup   | Clnt1-fs-Diff-    |
|      7 | Clnt1-fs-Incr    |       0 |       2 |  1,073,741,824 |      604,800 |       1 | Backup   | Clnt1-fs-Incr-    |
|      8 | Clnt2-fs-Monthly |       0 |      12 |  1,073,741,824 |   31,536,000 |       1 | Backup   | Clnt2-fs-Monthly- |
|      9 | Clnt2-fs-Full    |       0 |       4 |  1,073,741,824 |    7,948,800 |       1 | Backup   | Clnt2-fs-Full-    |
|     10 | Clnt2-fs-Diff    |       0 |       2 |  1,073,741,824 |    2,678,400 |       1 | Backup   | Clnt2-fs-Diff-    |
|     11 | Clnt2-fs-Incr    |       0 |       2 |  1,073,741,824 |      604,800 |       1 | Backup   | Clnt2-fs-Incr-    |
+--------+------------------+---------+---------+----------------+--------------+---------+----------+-------------------+
*list volume pool=Clnt1-fs-Full 
+---------+--------------------+-----------+---------+-------------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+------------+
| mediaid | volumename         | volstatus | enabled | volbytes    | volfiles | volretention | recycle | slot | inchanger | mediatype | voltype | volparts | lastwritten         | expiresin  |
+---------+--------------------+-----------+---------+-------------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+------------+
|       1 | Clnt1-fs-Full-0001 | Archive   |       1 | 195,145,465 |        0 |   31,536,000 |       0 |    0 |         0 | File      |       0 |        0 | 2024-09-17 17:14:30 | 31,470,806 |
+---------+--------------------+-----------+---------+-------------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+------------+
```
Отлично! Информация о первом томе появилась. 
Теперь выполним наши действия для всех остальных томов. 

Второй том:
```
root@debian12:~# bscan -c /etc/bacula/bacula-sd.conf -h localhost -P "5vjqDSZc5cCG" -s -m -v -V Clnt2-fs-Full-0002 /var/lib/bacula/storage
bscan: butil.c:292-0 Using device: "/var/lib/bacula/storage" for reading.
18-сен 11:21 bscan JobId 0: Ready to read from volume "Clnt2-fs-Full-0002" on File device "FileStorage" (/var/lib/bacula/storage).
bscan: bscan.c:323-0 Using Database: bacula, User: bacula
bscan: bscan.c:468-0 Pool record for Clnt2-fs-Full found in DB.
bscan: bscan.c:482-0 Pool type "Backup" is OK.
bscan: bscan.c:996-0 Created Media record for Volume: Clnt2-fs-Full-0002
bscan: bscan.c:510-0 Media type "File" is OK.
bscan: bscan.c:519-0 VOL_LABEL: OK for Volume: Clnt2-fs-Full-0002
bscan: bscan.c:1067-0 Created Client record for Client: Debian12cl2-fd
bscan: bscan.c:1149-0 Created new JobId=2 record for original JobId=2
bscan: bscan.c:1093-0 Created FileSet record "My-fs-FS"
bscan: bscan.c:1214-0 Updated Job termination record for JobId=2 Level=Full TermStat=T
bscan: bscan.c:1303-0 Created JobMedia record JobId 2, MediaId 2
18-сен 11:23 bscan JobId 0: End of Volume "Clnt2-fs-Full-0002" at addr=252483814 on device "FileStorage" (/var/lib/bacula/storage).
bscan: bscan.c:1017-0 Updated Media record at end of Volume: Clnt2-fs-Full-0002
bscan: bscan.c:1017-0 Updated Media record at end of Volume: Clnt2-fs-Full-0002
bscan: bscan.c:669-0 End of all Volumes. VolFiles=0 VolBlocks=0 VolBytes=252,342,768
Records added or updated in the catalog:
      1 Media
      1 Pool
      1 Job
  13887 File

root@debian12:~# bconsole 
Connecting to Director localhost:9101
1000 OK: 103 debian12-dir Version: 9.6.7 (10 December 2020)
Enter a period to cancel a command.
*reload
*list pools
Automatically selected Catalog: MyCatalog
Using Catalog "MyCatalog"
+--------+------------------+---------+---------+----------------+--------------+---------+----------+-------------------+
| poolid | name             | numvols | maxvols | maxvolbytes    | volretention | enabled | pooltype | labelformat       |
+--------+------------------+---------+---------+----------------+--------------+---------+----------+-------------------+
|      1 | Default          |       0 |     100 | 53,687,091,200 |   31,536,000 |       1 | Backup   | *                 |
|      2 | File             |       0 |     100 | 53,687,091,200 |   31,536,000 |       1 | Backup   | Vol-              |
|      3 | Scratch          |       0 |       0 |              0 |   31,536,000 |       1 | Backup   | *                 |
|      4 | Clnt1-fs-Monthly |       0 |      12 |  1,073,741,824 |   31,536,000 |       1 | Backup   | Clnt1-fs-Monthly- |
|      5 | Clnt1-fs-Full    |       1 |       4 |  1,073,741,824 |    7,948,800 |       1 | Backup   | Clnt1-fs-Full-    |
|      6 | Clnt1-fs-Diff    |       0 |       2 |  1,073,741,824 |    2,678,400 |       1 | Backup   | Clnt1-fs-Diff-    |
|      7 | Clnt1-fs-Incr    |       0 |       2 |  1,073,741,824 |      604,800 |       1 | Backup   | Clnt1-fs-Incr-    |
|      8 | Clnt2-fs-Monthly |       0 |      12 |  1,073,741,824 |   31,536,000 |       1 | Backup   | Clnt2-fs-Monthly- |
|      9 | Clnt2-fs-Full    |       1 |       4 |  1,073,741,824 |    7,948,800 |       1 | Backup   | Clnt2-fs-Full-    |
|     10 | Clnt2-fs-Diff    |       0 |       2 |  1,073,741,824 |    2,678,400 |       1 | Backup   | Clnt2-fs-Diff-    |
|     11 | Clnt2-fs-Incr    |       0 |       2 |  1,073,741,824 |      604,800 |       1 | Backup   | Clnt2-fs-Incr-    |
+--------+------------------+---------+---------+----------------+--------------+---------+----------+-------------------+
*list volume pool=Clnt2-fs-Full 
+---------+--------------------+-----------+---------+-------------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+------------+
| mediaid | volumename         | volstatus | enabled | volbytes    | volfiles | volretention | recycle | slot | inchanger | mediatype | voltype | volparts | lastwritten         | expiresin  |
+---------+--------------------+-----------+---------+-------------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+------------+
|       2 | Clnt2-fs-Full-0002 | Archive   |       1 | 252,342,768 |        0 |   31,536,000 |       0 |    0 |         0 | File      |       0 |        0 | 2024-09-17 17:15:07 | 31,469,209 |
+---------+--------------------+-----------+---------+-------------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+------------+
```
Теперь тома с инкрементными копиями. Восстановим третий том - Clnt1-fs-Incr-0003:
```
root@debian12:~# bscan -c /etc/bacula/bacula-sd.conf -h localhost -P "5vjqDSZc5cCG" -s -m -v -V Clnt1-fs-Incr-0003 /var/lib/bacula/storage
bscan: butil.c:292-0 Using device: "/var/lib/bacula/storage" for reading.
18-сен 11:50 bscan JobId 0: Ready to read from volume "Clnt1-fs-Incr-0003" on File device "FileStorage" (/var/lib/bacula/storage).
bscan: bscan.c:323-0 Using Database: bacula, User: bacula
bscan: bscan.c:468-0 Pool record for Clnt1-fs-Incr found in DB.
bscan: bscan.c:482-0 Pool type "Backup" is OK.
bscan: bscan.c:996-0 Created Media record for Volume: Clnt1-fs-Incr-0003
bscan: bscan.c:510-0 Media type "File" is OK.
bscan: bscan.c:519-0 VOL_LABEL: OK for Volume: Clnt1-fs-Incr-0003
bscan: bscan.c:1067-0 Created Client record for Client: Debian12cl1-fd
bscan: bscan.c:1149-0 Created new JobId=3 record for original JobId=3
bscan: bscan.c:1084-0 Fileset "My-tfs-FS" already exists.
bscan: bscan.c:1214-0 Updated Job termination record for JobId=3 Level=Incremental TermStat=T
bscan: bscan.c:1303-0 Created JobMedia record JobId 3, MediaId 3
18-сен 11:50 bscan JobId 0: End of Volume "Clnt1-fs-Incr-0003" at addr=195681169 on device "FileStorage" (/var/lib/bacula/storage).
bscan: bscan.c:1017-0 Updated Media record at end of Volume: Clnt1-fs-Incr-0003
bscan: bscan.c:1017-0 Updated Media record at end of Volume: Clnt1-fs-Incr-0003
bscan: bscan.c:669-0 End of all Volumes. VolFiles=0 VolBlocks=0 VolBytes=195,571,743
Records added or updated in the catalog:
      1 Media
      1 Pool
      1 Job
      1 File
      
----- И снова проверим результат -----
      
root@debian12:~# bconsole
Connecting to Director localhost:9101
1000 OK: 103 debian12-dir Version: 9.6.7 (10 December 2020)
Enter a period to cancel a command.
*reload
*list pools
Automatically selected Catalog: MyCatalog
Using Catalog "MyCatalog"
+--------+------------------+---------+---------+----------------+--------------+---------+----------+-------------------+
| poolid | name             | numvols | maxvols | maxvolbytes    | volretention | enabled | pooltype | labelformat       |
+--------+------------------+---------+---------+----------------+--------------+---------+----------+-------------------+
|      1 | Default          |       0 |     100 | 53,687,091,200 |   31,536,000 |       1 | Backup   | *                 |
|      2 | File             |       0 |     100 | 53,687,091,200 |   31,536,000 |       1 | Backup   | Vol-              |
|      3 | Scratch          |       0 |       0 |              0 |   31,536,000 |       1 | Backup   | *                 |
|      4 | Clnt1-fs-Monthly |       0 |      12 |  1,073,741,824 |   31,536,000 |       1 | Backup   | Clnt1-fs-Monthly- |
|      5 | Clnt1-fs-Full    |       1 |       4 |  1,073,741,824 |    7,948,800 |       1 | Backup   | Clnt1-fs-Full-    |
|      6 | Clnt1-fs-Diff    |       0 |       2 |  1,073,741,824 |    2,678,400 |       1 | Backup   | Clnt1-fs-Diff-    |
|      7 | Clnt1-fs-Incr    |       1 |       2 |  1,073,741,824 |      604,800 |       1 | Backup   | Clnt1-fs-Incr-    |
|      8 | Clnt2-fs-Monthly |       0 |      12 |  1,073,741,824 |   31,536,000 |       1 | Backup   | Clnt2-fs-Monthly- |
|      9 | Clnt2-fs-Full    |       1 |       4 |  1,073,741,824 |    7,948,800 |       1 | Backup   | Clnt2-fs-Full-    |
|     10 | Clnt2-fs-Diff    |       0 |       2 |  1,073,741,824 |    2,678,400 |       1 | Backup   | Clnt2-fs-Diff-    |
|     11 | Clnt2-fs-Incr    |       0 |       2 |  1,073,741,824 |      604,800 |       1 | Backup   | Clnt2-fs-Incr-    |
+--------+------------------+---------+---------+----------------+--------------+---------+----------+-------------------+
*list volume pool=Clnt1-fs-Incr 
+---------+--------------------+-----------+---------+-------------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+------------+
| mediaid | volumename         | volstatus | enabled | volbytes    | volfiles | volretention | recycle | slot | inchanger | mediatype | voltype | volparts | lastwritten         | expiresin  |
+---------+--------------------+-----------+---------+-------------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+------------+
|       3 | Clnt1-fs-Incr-0003 | Archive   |       1 | 195,571,743 |        0 |   31,536,000 |       0 |    0 |         0 | File      |       0 |        0 | 2024-09-18 09:00:33 | 31,525,789 |
+---------+--------------------+-----------+---------+-------------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+------------+
```
Восстановим четвёртый том - Clnt2-fs-Incr-0004:
```
root@debian12:~# bscan -c /etc/bacula/bacula-sd.conf -h localhost -P "5vjqDSZc5cCG" -s -m -v -V Clnt2-fs-Incr-0004 /var/lib/bacula/storage
bscan: butil.c:292-0 Using device: "/var/lib/bacula/storage" for reading.
18-сен 11:51 bscan JobId 0: Ready to read from volume "Clnt2-fs-Incr-0004" on File device "FileStorage" (/var/lib/bacula/storage).
bscan: bscan.c:323-0 Using Database: bacula, User: bacula
bscan: bscan.c:468-0 Pool record for Clnt2-fs-Incr found in DB.
bscan: bscan.c:482-0 Pool type "Backup" is OK.
bscan: bscan.c:996-0 Created Media record for Volume: Clnt2-fs-Incr-0004
bscan: bscan.c:510-0 Media type "File" is OK.
bscan: bscan.c:519-0 VOL_LABEL: OK for Volume: Clnt2-fs-Incr-0004
bscan: bscan.c:1067-0 Created Client record for Client: Debian12cl2-fd
bscan: bscan.c:1149-0 Created new JobId=4 record for original JobId=4
bscan: bscan.c:1084-0 Fileset "My-fs-FS" already exists.
bscan: bscan.c:1214-0 Updated Job termination record for JobId=4 Level=Incremental TermStat=T
bscan: bscan.c:1303-0 Created JobMedia record JobId 4, MediaId 4
18-сен 11:51 bscan JobId 0: End of Volume "Clnt2-fs-Incr-0004" at addr=6567531 on device "FileStorage" (/var/lib/bacula/storage).
bscan: bscan.c:1017-0 Updated Media record at end of Volume: Clnt2-fs-Incr-0004
bscan: bscan.c:1017-0 Updated Media record at end of Volume: Clnt2-fs-Incr-0004
bscan: bscan.c:669-0 End of all Volumes. VolFiles=0 VolBlocks=0 VolBytes=6,563,633
Records added or updated in the catalog:
      1 Media
      1 Pool
      1 Job
    115 File
root@debian12:~# bconsole
Connecting to Director localhost:9101
1000 OK: 103 debian12-dir Version: 9.6.7 (10 December 2020)
Enter a period to cancel a command.
*reload
*list pools
Automatically selected Catalog: MyCatalog
Using Catalog "MyCatalog"
+--------+------------------+---------+---------+----------------+--------------+---------+----------+-------------------+
| poolid | name             | numvols | maxvols | maxvolbytes    | volretention | enabled | pooltype | labelformat       |
+--------+------------------+---------+---------+----------------+--------------+---------+----------+-------------------+
|      1 | Default          |       0 |     100 | 53,687,091,200 |   31,536,000 |       1 | Backup   | *                 |
|      2 | File             |       0 |     100 | 53,687,091,200 |   31,536,000 |       1 | Backup   | Vol-              |
|      3 | Scratch          |       0 |       0 |              0 |   31,536,000 |       1 | Backup   | *                 |
|      4 | Clnt1-fs-Monthly |       0 |      12 |  1,073,741,824 |   31,536,000 |       1 | Backup   | Clnt1-fs-Monthly- |
|      5 | Clnt1-fs-Full    |       1 |       4 |  1,073,741,824 |    7,948,800 |       1 | Backup   | Clnt1-fs-Full-    |
|      6 | Clnt1-fs-Diff    |       0 |       2 |  1,073,741,824 |    2,678,400 |       1 | Backup   | Clnt1-fs-Diff-    |
|      7 | Clnt1-fs-Incr    |       1 |       2 |  1,073,741,824 |      604,800 |       1 | Backup   | Clnt1-fs-Incr-    |
|      8 | Clnt2-fs-Monthly |       0 |      12 |  1,073,741,824 |   31,536,000 |       1 | Backup   | Clnt2-fs-Monthly- |
|      9 | Clnt2-fs-Full    |       1 |       4 |  1,073,741,824 |    7,948,800 |       1 | Backup   | Clnt2-fs-Full-    |
|     10 | Clnt2-fs-Diff    |       0 |       2 |  1,073,741,824 |    2,678,400 |       1 | Backup   | Clnt2-fs-Diff-    |
|     11 | Clnt2-fs-Incr    |       1 |       2 |  1,073,741,824 |      604,800 |       1 | Backup   | Clnt2-fs-Incr-    |
+--------+------------------+---------+---------+----------------+--------------+---------+----------+-------------------+
*list volume pool=Clnt2-fs-Incr
+---------+--------------------+-----------+---------+-----------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+------------+
| mediaid | volumename         | volstatus | enabled | volbytes  | volfiles | volretention | recycle | slot | inchanger | mediatype | voltype | volparts | lastwritten         | expiresin  |
+---------+--------------------+-----------+---------+-----------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+------------+
|       4 | Clnt2-fs-Incr-0004 | Archive   |       1 | 6,563,633 |        0 |   31,536,000 |       0 |    0 |         0 | File      |       0 |        0 | 2024-09-18 09:00:35 | 31,525,697 |
+---------+--------------------+-----------+---------+-----------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+------------+
```
##### Восстановление информации с резервной копии после восстановления каталога

Проверим, что на клиентской машине никаким чудным образом не появился ранее созданный и уничтоженный вместе с виртуальной машиной тестовый файл:
```
max@localhost:~/vagrant/vg3> vagrant ssh Debian12cl2
==> vagrant: A new version of Vagrant is available: 2.4.1 (installed version: 2.2.18)!
==> vagrant: To upgrade visit: https://www.vagrantup.com/downloads.html

Linux debian12 6.1.0-22-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.94-1 (2024-06-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Jul 11 09:21:01 2024 from 192.168.122.1
vagrant@debian12:~$ ls -l /var/
итого 36
drwxr-xr-x  2 root root  4096 сен 18 10:42 backups
drwxr-xr-x 18 root root  4096 июл 10 14:12 cache
drwxr-xr-x 54 root root  4096 сен 18 10:42 lib
drwxrwsr-x  2 root staff 4096 янв 29  2024 local
lrwxrwxrwx  1 root root     9 мая 19 23:39 lock -> /run/lock
drwxr-xr-x 11 root root  4096 сен 18 10:43 log
drwxrwsr-x  2 root mail  4096 мая 19 23:39 mail
drwxr-xr-x  2 root root  4096 мая 19 23:39 opt
lrwxrwxrwx  1 root root     4 мая 19 23:39 run -> /run
drwxr-xr-x  6 root root  4096 мая 20 00:00 spool
drwxrwxrwt 10 root root  4096 сен 18 10:47 tmp
```
Выполняем восстановление файла, как и в предыдущей главе:
```
max@localhost:~/vagrant/vg3> vagrant ssh Debian12
Linux debian12 6.1.0-22-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.94-1 (2024-06-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Sep 18 10:56:47 2024 from 192.168.121.1
vagrant@debian12:~$ sudo -i
root@debian12:~# bconsole 
Connecting to Director localhost:9101
1000 OK: 103 debian12-dir Version: 9.6.7 (10 December 2020)
Enter a period to cancel a command.
*restore
Automatically selected Catalog: MyCatalog
Using Catalog "MyCatalog"

First you select one or more JobIds that contain files
to be restored. You will be presented several methods
of specifying the JobIds. Then you will be allowed to
select which files from those JobIds are to be restored.

To select the JobIds, you have the following choices:
     1: List last 20 Jobs run
     2: List Jobs where a given File is saved
     3: Enter list of comma separated JobIds to select
     4: Enter SQL list command
     5: Select the most recent backup for a client
     6: Select backup for a client before a specified time
     7: Enter a list of files to restore
     8: Enter a list of files to restore before a specified time
     9: Find the JobIds of the most recent backup for a client
    10: Find the JobIds for a backup for a client before a specified time
    11: Enter a list of directories to restore for found JobIds
    12: Select full restore to a specified Job date
    13: Cancel
Select item:  (1-13): 5
Defined Clients:
     1: Debian12cl1-fd
     2: Debian12cl2-fd
     3: debian12-fd
Select the Client (1-3): 2
Automatically selected FileSet: My-fs-FS
+-------+-------+----------+-------------+---------------------+--------------------+
| jobid | level | jobfiles | jobbytes    | starttime           | volumename         |
+-------+-------+----------+-------------+---------------------+--------------------+
|     2 | F     |   13,887 | 251,802,548 | 2024-09-17 17:14:53 | Clnt2-fs-Full-0002 |
|     4 | I     |      115 |   6,556,501 | 2024-09-18 09:00:35 | Clnt2-fs-Incr-0004 |
+-------+-------+----------+-------------+---------------------+--------------------+
You have selected the following JobIds: 2,4

Building directory tree for JobId(s) 2,4 ...  ++++++++++++++++++++++++++++++++++++++++++++++++
13,411 files inserted into the tree.

You are now entering file selection mode where you add (mark) and
remove (unmark) files to be restored. No files are initially added, unless
you used the "all" keyword on the command line.
Enter "done" to leave this mode.

cwd is: /
$ cd /var
cwd is: /var/
$ ls
backups/
cache/
lib/
local
lock
log/
mail
opt
run
spool/
testfile.txt
tmp/
$ mark testfile.txt 
1 file marked.
$ done
Using Storage "debian12-sd" from MediaType "File".
Bootstrap records written to /var/lib/bacula/debian12-dir.restore.1.bsr

The Job will require the following (*=>InChanger):
   Volume(s)                 Storage(s)                SD Device(s)
===========================================================================
   
    Clnt2-fs-Incr-0004                                                           

Volumes marked with "*" are in the Autochanger.


1 file selected to be restored.

The defined Restore Job resources are:
     1: RestoreFiles
     2: MyRestoreFiles
Select Restore Job (1-2): 2
Using Catalog "MyCatalog"
Run Restore job
JobName:         MyRestoreFiles
Bootstrap:       /var/lib/bacula/debian12-dir.restore.1.bsr
Where:           /bacula-restores
Replace:         Always
FileSet:         Full Set
Backup Client:   Debian12cl2-fd
Restore Client:  Debian12cl2-fd
Storage:         debian12-sd
When:            2024-09-18 11:54:51
Catalog:         MyCatalog
Priority:        10
Plugin Options:  *None*
OK to run? (yes/mod/no): m
Parameters to modify:
     1: Level
     2: Storage
     3: Job
     4: FileSet
     5: Restore Client
     6: When
     7: Priority
     8: Bootstrap
     9: Where
    10: File Relocation
    11: Replace
    12: JobId
    13: Plugin Options
Select parameter to modify (1-13): 9
Please enter the full path prefix for restore (/ for none): /
Run Restore job
JobName:         MyRestoreFiles
Bootstrap:       /var/lib/bacula/debian12-dir.restore.1.bsr
Where:           
Replace:         Always
FileSet:         Full Set
Backup Client:   Debian12cl2-fd
Restore Client:  Debian12cl2-fd
Storage:         debian12-sd
When:            2024-09-18 11:54:51
Catalog:         MyCatalog
Priority:        10
Plugin Options:  *None*
OK to run? (yes/mod/no): y
Job queued. JobId=5
*messages
18-сен 11:55 debian12-dir JobId 5: Start Restore Job MyRestoreFiles.2024-09-18_11.55.08_07
18-сен 11:55 debian12-dir JobId 5: Restoring files from JobId(s) 2,4
18-сен 11:55 debian12-dir JobId 5: Using Device "FileStorage" to read.
18-сен 11:55 debian12-sd JobId 5: Ready to read from volume "Clnt2-fs-Incr-0004" on File device "FileStorage" (/var/lib/bacula/storage).
18-сен 11:55 debian12-sd JobId 5: Elapsed time=00:00:01, Transfer rate=149  Bytes/second
18-сен 11:55 debian12-dir JobId 5: Bacula debian12-dir 9.6.7 (10Dec20):
  Build OS:               x86_64-pc-linux-gnu debian bookworm/sid
  JobId:                  5
  Job:                    MyRestoreFiles.2024-09-18_11.55.08_07
  Restore Client:         Debian12cl2-fd
  Where:                  
  Replace:                Always
  Start time:             18-сен-2024 11:55:10
  End time:               18-сен-2024 11:55:11
  Elapsed time:           1 sec
  Files Expected:         1
  Files Restored:         1
  Bytes Restored:         21 (21 B)
  Rate:                   0.0 KB/s
  FD Errors:              0
  FD termination status:  OK
  SD termination status:  OK
  Termination:            Restore OK

18-сен 11:55 debian12-dir JobId 5: Begin pruning Jobs older than 12 months .
18-сен 11:55 debian12-dir JobId 5: No Jobs found to prune.
18-сен 11:55 debian12-dir JobId 5: Begin pruning Files.
18-сен 11:55 debian12-dir JobId 5: No Files found to prune.
18-сен 11:55 debian12-dir JobId 5: End auto prune.

*exit
```
Здесь я уже не стал комментировать все действия по задаче **Restore**, так как они подробно разобраны в [предыдущей](https://github.com/spanishairman/bacula-debian#%D0%B2%D0%BE%D1%81%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5-%D0%B8%D0%BD%D1%84%D0%BE%D1%80%D0%BC%D0%B0%D1%86%D0%B8%D0%B8) главе.

Проверим, появился ли наш файл:
```
max@localhost:~/vagrant/vg3> vagrant ssh Debian12cl2
Linux debian12 6.1.0-22-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.94-1 (2024-06-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Sep 18 11:52:49 2024 from 192.168.121.1
vagrant@debian12:~$ ls -l /var
итого 40
drwxr-xr-x  2 root root  4096 сен 18 10:42 backups
drwxr-xr-x 18 root root  4096 июл 10 14:12 cache
drwxr-xr-x 54 root root  4096 сен 18 10:42 lib
drwxrwsr-x  2 root staff 4096 янв 29  2024 local
lrwxrwxrwx  1 root root     9 мая 19 23:39 lock -> /run/lock
drwxr-xr-x 11 root root  4096 сен 18 10:43 log
drwxrwsr-x  2 root mail  4096 мая 19 23:39 mail
drwxr-xr-x  2 root root  4096 мая 19 23:39 opt
lrwxrwxrwx  1 root root     4 мая 19 23:39 run -> /run
drwxr-xr-x  6 root root  4096 мая 20 00:00 spool
-rw-r--r--  1 root root    21 сен 18 08:57 testfile.txt
drwxrwxrwt 10 root root  4096 сен 18 10:47 tmp
vagrant@debian12:~$ cat /var/testfile.txt 
Hello! Its Test File
```
Как видим, файл был успешно восстановлен и его содержимое соответствует исходному.

##### Как работают ограничения _Volume Retation_, _Job Retention_

Настроим наши расписания так, чтобы задачи _Clnt1-fs-Monthly-Job_ и _Clnt2-fs-Monthly-Job_ писали полную резервную копию на тома _Clnt1-fs-Monthly_ и _Clnt2-fs-Monthly_ каждые две минуты.
При этом для обоих пулов установим значение _Volume Retention_ равное 10  минутам. Также, для клиента _Debian12cl1-fd_ установим параметр _Job Retention_, также равным 10 минутам.

Тогда описание клиентов будет таким:
```
# Client (File Services) to backup
Client {
  Name = Debian12cl1-fd
  Address = 192.168.121.11
  FDPort = 9102
  Catalog = MyCatalog
  Password = "DYrPl1SQnGYgUHDy809bU6ejZyo-N97m4"          # password for FileDaemon
  File Retention = 10 min
  Job Retention = 10 min
  AutoPrune = yes                     # Prune expired Jobs/Files
}

# Client (File Services) to backup
Client {
  Name = Debian12cl2-fd
  Address = 192.168.121.12
  FDPort = 9102
  Catalog = MyCatalog
  Password = "tyWfHO1Bp3joollMSdXggFoeBoMTPZF8G"          # password for FileDaemon
  File Retention = 365 days           # 60 days
  Job Retention = 12 months           # six months
  AutoPrune = yes                     # Prune expired Jobs/Files
}
```

Настройки для пулов будут выглядеть так:
```
Pool {
  Name = Clnt1-fs-Monthly
  Pool Type = Backup
  Recycle = yes                         # Bacula can automatically recycle Volumes
  AutoPrune = yes                       # Prune expired volumes
  Recycle Oldest Volume = yes           # Prune the oldest volume in the Pool, and if all files were pruned, recycle this volume and use it.
  Volume Retention = 10  min            # How long should the Full Backups be kept? (#06)
  Maximum Volume Bytes = 3G             # Limit Volume size to something reasonable
  Maximum Volume Jobs = 12              # One Job = One Vol
  Maximum Volumes = 1                   # Limit number of Volumes in Pool
  Label Format = "Clnt1-fs-Monthly-"    # Volumes will be labeled "Full-<volume-id>"
}

Pool {
  Name = Clnt2-fs-Monthly
  Pool Type = Backup
  Recycle = yes                         # Bacula can automatically recycle Volumes
  AutoPrune = yes                       # Prune expired volumes
  Recycle Oldest Volume = yes           # Prune the oldest volume in the Pool, and if all files were pruned, recycle this volume and use it.
  Volume Retention = 10  min            # How long should the Full Backups be kept? (#06)
  Maximum Volume Bytes = 1G             # Limit Volume size to something reasonable
  Maximum Volume Jobs = 1               # One Job = One Vol
  Maximum Volumes = 12                  # Limit number of Volumes in Pool
  Label Format = "Clnt2-fs-Monthly-"    # Volumes will be labeled "Full-<volume-id>"
}
```
Расписания, в таком случае, примут следующий вид:
```
Schedule {
  Enabled = yes
  Name = "Clnt1-fs-Monthly-Sdl"
  # Run = Level=Full Pool=Clnt1-fs-Monthly on 1 at 00:00
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:02
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:04
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:06
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:08
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:10
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:12
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:14
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:16
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:18
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:20
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:22
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:24
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:26
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:28
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:30
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:32
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:34
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:36
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:38
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:40
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:42
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:44
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:46
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:48
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:50
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:52
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:54
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:56
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:58
  Run = Level=Full FullPool=Clnt1-fs-Monthly hourly at 0:00
}

Schedule {
  Enabled = yes
  Name = "Clnt2-fs-Monthly-Sdl"
  # Run = Level=Full Pool=Clnt2-fs-Monthly on 1 at 00:00
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:02
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:04
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:06
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:08
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:10
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:12
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:14
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:16
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:18
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:20
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:22
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:24
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:26
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:28
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:30
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:32
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:34
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:36
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:38
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:40
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:42
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:44
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:46
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:48
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:50
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:52
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:54
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:56
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:58
  Run = Level=Full Pool=Clnt2-fs-Monthly hourly at 0:00
}
```
> [!NOTE]
> Bacula хранит расписание в виде битовой маски. Для каждого расписания есть шесть масок и поле минут. Маски — это час (hour), день месяца (mday), месяц (month), день недели (wday), 
> неделя месяца (wom) и неделя года (woy). Расписание инициализируется так, чтобы биты каждой из этих масок были установлены, что означает, что в начале каждого часа 
> задание будет выполняться. Когда вы указываете в расписании конкретный  месяц, соответствующая маска будет сброшена, и будет выбран бит, соответствующий выбранному вами месяцу. 
> Если вы указываете второй месяц, то соответствующий ему бит также будет добавлен в маску _month_. Таким образом, когда _Bacula_ проверяет маски, чтобы увидеть, 
> установлены ли биты в соответствии с текущим временем, ваше задание будет выполняться только в течение двух установленных вами месяцев. 
> Аналогично, если вы устанавливаете час (hour), маска _hour_ будет очищена, и указанный вами час будет установлен в битовой маске.
> Чтобы проверить, как установлены биты нашего расписания, в консоли _Bacula_ выполним команду `show schedule=Clnt1-fs-Monthly-Sdl`.
```
*show schedule=Clnt1-fs-Monthly-Sdl 
Schedule: Name=Clnt1-fs-Monthly-Sdl Enabled=1
  --> Run Level=Full
      hour=0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 
      mday=0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 
      month=0 1 2 3 4 5 6 7 8 9 10 11 
      wday=0 1 2 3 4 5 6 
      wom=0 1 2 3 4 5 
      woy=0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 
      mins=2
  --> Run Level=Full
      hour=0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 
      mday=0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 
      month=0 1 2 3 4 5 6 7 8 9 10 11 
      wday=0 1 2 3 4 5 6 
      wom=0 1 2 3 4 5 
      woy=0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 
      mins=4
  --> Run Level=Full
      hour=0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 
      mday=0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 
      month=0 1 2 3 4 5 6 7 8 9 10 11 
      wday=0 1 2 3 4 5 6 
      wom=0 1 2 3 4 5 
      woy=0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 
      mins=6
  --> Run Level=Full
      hour=0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 
      mday=0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 
      month=0 1 2 3 4 5 6 7 8 9 10 11 
      wday=0 1 2 3 4 5 6 
      wom=0 1 2 3 4 5 
      woy=0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 
      mins=8
  --> Run Level=Full
      hour=0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 
      mday=0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 
      month=0 1 2 3 4 5 6 7 8 9 10 11 
      wday=0 1 2 3 4 5 6 
      wom=0 1 2 3 4 5 
      woy=0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 
      mins=10
...
...
```
Теперь, после всех наших манипуляций мы должны получить следующий результат:
  - на тома в пулах _Clnt1-fs-Monthly_ и _Clnt2-fs-Monthly_ каждые две минуты будут записываться полные копии каталогов, перечисленных в ресурсах _"My-tfs-FS"_ и _"My-fs-FS"_ (содержимое каталога /etc на клиентской машине);
  - в соответствии с ограничениями, в пуле _Clnt1-fs-Monthly_ должен создаться один том с меткой _Clnt1-fs-Monthly-_, содержащий 12 задач с наборами резервных копий, а в пуле _Clnt2-fs-Monthly_ - 12 томов с метками _Clnt2-fs-Monthly-_ и с одной задачей, содержащей наборы резервных копий, в каждом томе;
  - в тоже время, для томов каждого пула установлено значение _Volume Retention_, равное 10 минутам, а для задач клиента _Debian12cl1-fd_ - _Job Retention_ так же равное 10 минутам. Т.е., начиная с шестой задачи, в первом пуле будет один том с просроченным параметром _Volume Retention_, а во втором - одна задача также с просроченным значением _Job Retention_. Это позволяет предположить, что оба пула, при данных настройках, никогда не дойдут до своих ограничений в количестве томов и задач в них.

Запускаем стенд и ждем, когда будут созданы первые резервные копии, после этого проверяем содержимое пулов. Пул _Clnt1-fs-Monthly_ (один том, двенадцать задач на нём) - пока что содержит две задачи, время первой записи - `firstwritten: 2024-09-20 23:36:53`:
```
*llist volume=Clnt1-fs-Monthly-0002
          mediaid: 2
       volumename: Clnt1-fs-Monthly-0002
             slot: 0
           poolid: 4
        mediatype: File
      mediatypeid: 0
     firstwritten: 2024-09-20 23:36:53
      lastwritten: 2024-09-20 23:38:30
        labeldate: 2024-09-20 23:36:53
          voljobs: 2
         volfiles: 0
        volblocks: 6,054
         volparts: 0
    volcloudparts: 0
   cacheretention: 0
        volmounts: 2
         volbytes: 390,527,517

```
Пул _Clnt2-fs-Monthly_ (двенадцать томов по 1 задаче):
```
*list volume pool=Clnt2-fs-Monthly
+---------+-----------------------+-----------+---------+-----------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+-----------+
| mediaid | volumename            | volstatus | enabled | volbytes  | volfiles | volretention | recycle | slot | inchanger | mediatype | voltype | volparts | lastwritten         | expiresin |
+---------+-----------------------+-----------+---------+-----------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+-----------+
|       1 | Clnt2-fs-Monthly-0001 | Used      |       1 | 2,306,324 |        0 |          600 |       1 |    0 |         0 | File      |       1 |        0 | 2024-09-20 23:36:40 |       511 |
+---------+-----------------------+-----------+---------+-----------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+-----------+
```
Здесь пока один том, время записи (параметр _lastwritten_) - `2024-09-20 23:36:40`.

через несколько минут уже:
```
*list volume pool=Clnt2-fs-Monthly
+---------+-----------------------+-----------+---------+-----------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+-----------+
| mediaid | volumename            | volstatus | enabled | volbytes  | volfiles | volretention | recycle | slot | inchanger | mediatype | voltype | volparts | lastwritten         | expiresin |
+---------+-----------------------+-----------+---------+-----------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+-----------+
|       1 | Clnt2-fs-Monthly-0001 | Used      |       1 | 2,306,324 |        0 |          600 |       1 |    0 |         0 | File      |       1 |        0 | 2024-09-20 23:36:40 |         0 |
|       3 | Clnt2-fs-Monthly-0003 | Used      |       1 | 2,306,324 |        0 |          600 |       1 |    0 |         0 | File      |       1 |        0 | 2024-09-20 23:38:35 |        47 |
|       4 | Clnt2-fs-Monthly-0004 | Used      |       1 | 2,306,324 |        0 |          600 |       1 |    0 |         0 | File      |       1 |        0 | 2024-09-20 23:40:35 |       167 |
|       5 | Clnt2-fs-Monthly-0005 | Used      |       1 | 2,306,324 |        0 |          600 |       1 |    0 |         0 | File      |       1 |        0 | 2024-09-20 23:42:35 |       287 |
|       6 | Clnt2-fs-Monthly-0006 | Used      |       1 | 2,306,324 |        0 |          600 |       1 |    0 |         0 | File      |       1 |        0 | 2024-09-20 23:44:35 |       407 |
|       7 | Clnt2-fs-Monthly-0007 | Used      |       1 | 2,306,324 |        0 |          600 |       1 |    0 |         0 | File      |       1 |        0 | 2024-09-20 23:46:35 |       527 |
+---------+-----------------------+-----------+---------+-----------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+-----------+
```
Здесь мы видим, что последний том с именем _Clnt2-fs-Monthly-0007_ в пуле _Clnt2-fs-Monthly_ записан в 23:46:35, всего томов - шесть, разница во времени между первым и последним томом - 9:55. 

Проверяем спустя несколько секунд:
```
*list volume pool=Clnt2-fs-Monthly
+---------+-----------------------+-----------+---------+-----------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+-----------+
| mediaid | volumename            | volstatus | enabled | volbytes  | volfiles | volretention | recycle | slot | inchanger | mediatype | voltype | volparts | lastwritten         | expiresin |
+---------+-----------------------+-----------+---------+-----------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+-----------+
|       1 | Clnt2-fs-Monthly-0001 | Used      |       1 | 2,306,324 |        0 |          600 |       1 |    0 |         0 | File      |       1 |        0 | 2024-09-20 23:48:35 |       574 |
|       3 | Clnt2-fs-Monthly-0003 | Used      |       1 | 2,306,324 |        0 |          600 |       1 |    0 |         0 | File      |       1 |        0 | 2024-09-20 23:38:35 |         0 |
|       4 | Clnt2-fs-Monthly-0004 | Used      |       1 | 2,306,324 |        0 |          600 |       1 |    0 |         0 | File      |       1 |        0 | 2024-09-20 23:40:35 |        94 |
|       5 | Clnt2-fs-Monthly-0005 | Used      |       1 | 2,306,324 |        0 |          600 |       1 |    0 |         0 | File      |       1 |        0 | 2024-09-20 23:42:35 |       214 |
|       6 | Clnt2-fs-Monthly-0006 | Used      |       1 | 2,306,324 |        0 |          600 |       1 |    0 |         0 | File      |       1 |        0 | 2024-09-20 23:44:35 |       334 |
|       7 | Clnt2-fs-Monthly-0007 | Used      |       1 | 2,306,324 |        0 |          600 |       1 |    0 |         0 | File      |       1 |        0 | 2024-09-20 23:46:35 |       454 |
+---------+-----------------------+-----------+---------+-----------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+-----------+
```
Здесь уже наглядно видно, что последняя запись произведена в том с именем _Clnt2-fs-Monthly-0001_, т.е. первый том в данном пуле томов. Следовательно, ограничение _Volume Retention_ для пула сработало
и резервные копии записываются циклично в самый старый том а количество томов осталось прежним, равным шести не достигнув ограничения пула в 12 томов, в соответствии с заданными параметрами:
```
Recycle Oldest Volume = yes
Volume Retention = 10  min
```
В случае же с ограничением _Job Retention_, заданном в ресурсе Клиента, наборы резервных копий продолжают записываться в том до тех пор, пока не будет достигнуто ограничение в количестве задач для данного тома:
```
*llist volume=Clnt1-fs-Monthly-0002
          mediaid: 2
       volumename: Clnt1-fs-Monthly-0002
             slot: 0
           poolid: 4
        mediatype: File
      mediatypeid: 0
     firstwritten: 2024-09-20 23:36:53
      lastwritten: 2024-09-20 23:58:30
        labeldate: 2024-09-20 23:36:53
          voljobs: 12
         volfiles: 0
        volblocks: 36,324
         volparts: 0
    volcloudparts: 0
   cacheretention: 0
        volmounts: 12

```
При этом, так как для томов в пуле _Clnt2-fs-Monthly_ задан параметр _Volume Retention_ равным 10 минутам, а значения времени для этого параметра начинают отсчитываться от момента 
последней записи, мы получаем очередь из задач, ожидающих свободные тома, пока не истечёт десять минут с момента записи задачи с номером двенадцать для этого тома, так как единственный том в пуле имеет статус `volstatus: Used`.
Чтобы избежать таких очередей уменьшим параметр _Volume Retention_ до значения меньшего или равного частоте запуска задачи _Clnt1-fs-Monthly-Job_ из расписания _Clnt1-fs-Monthly-Sdl_:
```
*update volume=Clnt1-fs-Monthly-0002
Parameters to modify:
     1: Volume Status
     2: Volume Retention Period
     3: Volume Use Duration
     4: Maximum Volume Jobs
     5: Maximum Volume Files
     6: Maximum Volume Bytes
     7: Recycle Flag
     8: Slot
     9: InChanger Flag
    10: Volume Files
    11: Pool
    12: Volume from Pool
    13: All Volumes from Pool
    14: All Volumes from all Pools
    15: Enabled
    16: RecyclePool
    17: Action On Purge
    18: Cache Retention
    19: Done
Select parameter to modify (1-19): 2
Updating Volume "Clnt1-fs-Monthly-0002"
Current retention period is: 10 mins 
59
New retention period is: 59 secs
```
Дальнейший листинг показывает, как меняется содержимое и статус тома c _Append_ на _Used_ затем _Recycle_ и снова _Append_ при достижении им ограничивающего значения для параметра `Maximum Volume Jobs = 12`

```
*llist volume=Clnt1-fs-Monthly-0002
          mediaid: 2
       volumename: Clnt1-fs-Monthly-0002
             slot: 0
           poolid: 4
        mediatype: File
      mediatypeid: 0
     firstwritten: 2024-09-21 00:29:02
      lastwritten: 2024-09-21 00:46:29
        labeldate: 2024-09-21 00:24:01
          voljobs: 12
         volfiles: 0
        volblocks: 36,408
         volparts: 0
    volcloudparts: 0
   cacheretention: 0
        volmounts: 36
         volbytes: 2,348,356,412
        volabytes: 0
      volapadding: 0
     volholebytes: 0
         volholes: 0
    lastpartbytes: 0
        volerrors: 0
        volwrites: 109,143
 volcapacitybytes: 0
        volstatus: Used
          enabled: 1
          recycle: 1
     volretention: 59
   voluseduration: 0
       maxvoljobs: 12
      maxvolfiles: 0
      maxvolbytes: 3,221,225,472
      
*llist volume=Clnt1-fs-Monthly-0002
          mediaid: 2
       volumename: Clnt1-fs-Monthly-0002
             slot: 0
           poolid: 4
        mediatype: File
      mediatypeid: 0
     firstwritten: 1970-01-01 03:00:00
      lastwritten: 2024-09-21 00:46:29
        labeldate: 2024-09-21 00:24:01
          voljobs: 0
         volfiles: 0
        volblocks: 0
         volparts: 0
    volcloudparts: 0
   cacheretention: 0
        volmounts: 36
         volbytes: 1
        volabytes: 0
      volapadding: 0
     volholebytes: 0
         volholes: 0
    lastpartbytes: 0
        volerrors: 0
        volwrites: 109,143
 volcapacitybytes: 0
        volstatus: Recycle
          enabled: 1
          recycle: 1
     volretention: 59
   voluseduration: 0
       maxvoljobs: 12
      maxvolfiles: 0
      maxvolbytes: 3,221,225,472
      
*llist volume=Clnt1-fs-Monthly-0002
          mediaid: 2
       volumename: Clnt1-fs-Monthly-0002
             slot: 0
           poolid: 4
        mediatype: File
      mediatypeid: 0
     firstwritten: 2024-09-21 00:48:01
      lastwritten: 2024-09-21 00:48:29
        labeldate: 2024-09-21 00:48:01
          voljobs: 1
         volfiles: 0
        volblocks: 3,034
         volparts: 0
    volcloudparts: 0
   cacheretention: 0
        volmounts: 37
         volbytes: 195,696,224
        volabytes: 0
      volapadding: 0
     volholebytes: 0
         volholes: 0
    lastpartbytes: 0
        volerrors: 0
        volwrites: 112,178
 volcapacitybytes: 0
        volstatus: Append
          enabled: 1
          recycle: 1
     volretention: 59
   voluseduration: 0
       maxvoljobs: 12
      maxvolfiles: 0
      maxvolbytes: 3,221,225,472
```
Спасибо за прочтение! :potted_plant:
