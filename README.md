## Bacula. Установка и настройка в операционной системе Debian Bookworm
`Bacula — это набор компьютерных программ, позволяющий системному администратору управлять резервным копированием, восстановлением и проверкой компьютерных данных в сети компьютеров разных типов. Bacula также может работать полностью на одном компьютере и может выполнять резервное копирование на различные типы носителей, включая ленту и диск.`
[Источник](https://www.bacula.org/what-is-bacula/)
### Компоненты Bacula

  -  **_Bacula Director_** - центральная управляющая программа для всех остальных демонов. Он планирует и контролирует все операции резервного копирования, восстановления, проверки и архивирования. Системный администратор использует Bacula Director для планирования резервного копирования и восстановления файлов. Директор работает как демон (или служба) в фоновом режиме.
  -  **_The Bacula Console_** - это программа, которая позволяет администратору или пользователю общаться с Bacula Director. Он запускается в окне консоли (интерфейс TTY).
  -  **_Bacula File Daemon_** - это программа, которая должна быть установлена на каждом (клиентском) компьютере, для которого необходимо создать резервную копию. По запросу директора Bacula он находит файлы для резервного копирования и отправляет их (их данные) в демон хранения Bacula.
  -  **_Bacula Storage Daemon_** - По запросу от Bacula Director, Storage Daemon отвечает за прием данных от Bacula File Daemon и сохранение атрибутов и данных файла на физическом носителе или томах резервного копирования. В случае запроса на восстановление он отвечает за поиск данных и отправку их демону файлов Baacula. В вашей среде может быть несколько демонов Bacula Storage, каждый из которых контролируется одним и тем же директором Bacula. Службы хранилища работают как демон на компьютере с устройством резервного копирования (например, на ленточном накопителе).
  -  **_Catalog_** - Службы каталога состоят из программ, отвечающих за поддержание файловых индексов и баз данных томов для всех резервных копий файлов. Службы каталога позволяют системному администратору или пользователю быстро найти и восстановить любой нужный файл. Службы каталога устанавливают Bacula отдельно от простых программ резервного копирования, таких как tar и bru, поскольку в каталоге ведется запись всех используемых томов, всех выполненных заданий и всех сохраненных файлов, что обеспечивает эффективное восстановление и управление томами. В настоящее время Bacula поддерживает три разные базы данных: MySQL, PostgreSQL и SQLite, одну из которых необходимо выбрать при сборке Bacula.

![Bacula](bacula.png)

Сетевые взаимодействия между компонентами _Bacula_. 

Для корректной работы, нам потребуется открыть некоторые порты на межсетевом экране, если такой имеется в нашей сети. Следующий список поможет настроить правила файервола:
```
Console       -> Director:9101
Director      -> Storage Daemon:9103
Director      -> File Daemon:9102
File Daemon   -> Storage Daemon:9103
```

### Термины
  - **_Client_** - в терминологии _Bacula_ слово _Client_ относится к резервируемому компьютеру и является синонимом Файловых служб или Файлового демона, и довольно часто его называют File Daemon. Клиент определяется в ресурсе файла конфигурации;
  - **_Directive_** - термин _Directive_ используется для обозначения оператора или записи в ресурсе в файле конфигурации, который определяет один конкретный параметр. Например, директива Name определяет имя ресурса;
  - **_File Attributes_** -  это вся информация, необходимая для идентификации файла и все его свойства, такие как размер, дата создания, дата изменения, разрешения и т.д. Обычно атрибуты полностью обрабатываются Baculs, так что пользователю никогда не нужно беспокоиться о них. Атрибуты не содержат данные файла.
  - **_FileSet_** - ресурс, содержащийся в файле конфигурации, который определяет файлы для резервного копирования. Он состоит из списка включенных файлов или каталогов, списка исключенных файлов и способа хранения файла (сжатие, шифрование, подписи). Для получения дополнительной информации;
  - **_Differential_** - разностная резервная копия включает все файлы, измененные с момента начала последнего полного сохранения. Обратите внимание, что другие программы резервного копирования могут определять это по-другому;
  - **_Incremental_** - инкрементное резервное копирование включает все файлы, измененные с момента запуска последнего полного, разностного или инкрементного резервного копирования. Обычно он указывается в директиве Level в определении ресурса Job или в ресурсе Schedule;
  - **_Resource_** - ресурс является частью файла конфигурации, который определяет конкретную единицу информации, доступную для Bacula. Он состоит из нескольких директив (отдельных операторов конфигурации). Например, ресурс задания определяет все свойства конкретного задания: имя, расписание, пул томов, тип резервного копирования, уровень резервного копирования, и т.д.;
  - **_Job_** - задание в _Bacula_ - это ресурс конфигурации, который определяет работу, которую Bacula должна выполнить для резервного копирования или восстановления конкретного клиента. Он состоит из типа (резервное копирование, восстановление, проверка и т.д.), Уровня (полный, дифференциальный, инкрементный и т.д.), Набора файлов и хранилища, для которого необходимо создать резервные копии файлов (устройство хранения, пул носителей);
  - **_JobDefs_** - необязательный ресурс для предоставления значений по умолчанию для ресурсов Job. Т.о. - JobDefs представляет собой шаблон со значениями, которые часто используются в ресурсах Job, позволяющий сократить ваши настройки и облегчить чтение конфигурации;
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
  - **_Scan_** - операция сканирования вызывает сканирование содержимого _тома_ или серии _томов_. Информация о том, какие файлы содержат эти _тома_, восстанавливается в каталоге _Bacula_. Как только информация восстановлена в _базе данных_ (Catalog), файлы, содержащиеся в этих _томах_ (Volumes), могут быть легко восстановлены. Эта функция особенно полезна, если определенные _тома_ или _задания_ превысили срок хранения и были удалены или удалены из _каталога_. Сканирование данных из _томов_ в _каталог_ осуществляется с помощью программы _bscan_.

### Установка
_Bacula_, в отличие от _Bareos_ присутствует в стандартных репозиториях _Debian_, поэтому будем использовать её. Для установки воспользуемся пакетным менеджером _Apt_:
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
```export DEBIAN_FRONTEND=noninteractive```

Тогда установщик при настройке _Bacula Catalog_ назначит параметры по умолчанию:

```# Generic catalog service
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

#### Настройка на сервере. Director
Для того, чтобы пользователь (_vagrant_ в данном случае) мог пользоваться графическим приложением _Bacula Admin Tool - bat_, добавим его в группу _bacula_. 
```
root@debian12:/etc/bacula# usermod -a -G bacula vagrant
```
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
_Шаблоны задач_. Некоторые параметры для различных задач _Job_ могут повторяться. Поэтому их можно вынести в блоки _Jobdefs_, чтобы затем ссылаться на них из директив _Job_ для сокращения, таким образом, 
количества настроек и более простого чтения конфигурационного файла. По умолчанию, конфигурационный файл _bacula-dir.conf_ уже содержит минимальный набор параметров, здесь я привенду только те, которые добавил сам.
```
JobDefs {
  Name = "My-Full-Tpl"
  Type = Backup
  Level = Full
  Storage = debian12-sd
  Messages = Standard
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}

JobDefs {
  Name = "My-Diff-Tpl"
  Type = Backup
  Level = Differential
  Storage = debian12-sd
  Messages = Standard
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}

JobDefs {
  Name = "My-Incr-Tpl"
  Type = Backup
  Level = Incremental
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

Также, в одной из задач (_Job_) перечислены пулы для каждого из трёх типов резервных копий - полная, разностная и инкрементная. Для задания пулов во второй задаче мы восмользовались функцией _переопределения_, которая доступна для ресурса _Schedule_.
> [!NOTE]
> Параметр _Pool_ для ресурса _Job_ является обязательным.
>
> Даже если вы используете _переопределения_ параметров _Pool_, _Full Backup Pool_, _Differential Backup Pool_ или _Incremental Backup Pool_, _задачи_ в ресурсе _Schedule_, параметр _Pool_ должен присутствовать в ресурсе _Job_ или _JobDefs_, на который ссылается _Job_.
>
> Также он должен быть задан в ресурсе _Job_ или _JobDefs_, на который ссылается _Job_, независимо от наличия в ресурсе задачи параметров _Full Backup Pool_, _Differential Backup Pool_ или _Incremental Backup Pool_.

Пулы в каждой задаче свои и будут содержать тома только одного клиента - это необязательное условие, просто здесь я выбрал такой принцип для наглядности.

_Расписание_ (_Schedule_) - также для каждого клиента своё.

_Скрипты_ (_ClientRunBeforeJob_ и _ClientRunAfterJob_), выполняющиеся до и после задачи резервного копирования запускаются на клиентской машине, т.е. на стороне _File Daemon_. 
Такие скрипты могут содержать Bash-команды по предварительному сжатию архивируемых файлов, копированию их в tar-архив или, например, sql-команды, выполняющие дамп баз данных. 
Скрипты, выполняющиеся после задачи резервного копирования, могут содержать команды по удалению архивируемых файлов или ранее созданных tar-архивов или sql-файлов.
> [!NOTE]
> Чтобы было легче ориентироваться, для задач, пулов, расписаний и клиентов я выбрал шаблонные имена, которые содержат общие части. 
> Например, для клиента _Debian12cl1-fd_ соответствует имя задачи _Clnt1-fs-Job_, пул _Clnt1-fs-Full_, расписание - _Clnt1-fs-Sdl_.

```
Job {
  Name = "Clnt1-fs-Job"
  Type = Backup
  FileSet = "My-tfs-FS"
  Pool = Clnt1-fs-Full
  Full Backup Pool = Clnt1-fs-Full                  # write Full Backups into "Full" Pool         (#05)
  Differential Backup Pool = Clnt1-fs-Diff
  Incremental Backup Pool = Clnt1-fs-Incr           # write Incr Backups into "Incremental" Pool  (#11)
  Schedule = "Clnt1-fs-Sdl"
  JobDefs = "My-Full-Tpl"
  Client = "Debian12cl1-fd"
  ClientRunBeforeJob = "/etc/bacula/scripts/bacula-before-fs.sh" # скрипт выполняющийся до задачи
  ClientRunAfterJob = "/etc/bacula/scripts/bacula-after-fs.sh" # скрипт выполняющийся после задачи
}
 
Job {
  Name = "Clnt2-fs-Job"
  Type = Backup
  FileSet = "My-fs-FS"
  Pool = Clnt2-fs-Full
  Schedule = "Clnt2-fs-Sdl"
  JobDefs = "My-Full-Tpl"
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

В примере ниже описывается стандартная задача восстановления на несуществующий носитель, как говорится в описании, этого достаточно для всех наборов Jobs, Clients, Storages. Я лишь изменил каталог по умолчанию, в который будут восстанавливаться данные. 
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
> Ресурс FileSet определяет, какие файлы должны быть включены в задании резервного копирования или исключены из него. Набор файлов требуется для каждого задания резервного копирования. Он состоит из списка файлов или каталогов, которые необходимо включить, списка файлов или каталогов, которые необходимо исключить, и различных параметров резервного копирования, таких как сжатие, шифрование и подписи, которые должны применяться к каждому файлу.

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
      Compression = GZIP
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

Переопределения заданий (Job-overrides) позволяют переопределять спецификации Уровня (Level), Хранилища (Storage), Сообщений (Messages) и Пула (Pool), представленные в ресурсе _задание_ (Job resource). Кроме того, спецификации _FullPool_, _IncrementalPool_ и _DifferentialPool_ позволяют переопределять спецификацию пула в зависимости от того, какой уровень заданий резервного копирования действует.

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
> 
> Эти записи равнозначны и выбор зависит от того, хотите ли вы использовать одно общее расписание для разных задач, или каждой задаче ссответствует свое расписание.

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
> Client - В терминологии Bacula слово «Клиент» относится к резервируемому компьютеру и является синонимом Файловых служб или Файлового демона, и довольно часто его называют FD. Клиент определяется в ресурсе файла конфигурации.
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
  - %d = Имя директора
  - %e = Код выхода из задания (_Job exit code_) (OK, Ошибка, ...)
  - %h = Адрес клиента
  - %i = Идентификатор задания (_Job Id_)
  - %j = Уникальное имя задания (_Unique Job name_)
  - %l = Уровень задания (_Job level_)
  - %n = Название задания (_Job name_)
  - %r = Получатели (_Recipients_)
  - %s = Время запуска (_Since time_)
  - %t = Тип задания (_Job type_) (например, резервное копирование, ...)
  - %v = Имя тома (_Volume name_) (Только на стороне директора)

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
  Label Format = "Clnt1-fs-Monthly-"    # Volumes will be labeled "Full-<volume-id>"
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
  Label Format = "Clnt1-fs-Full-"
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
  Label Format = "Clnt1-fs-Diff-"
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
  Label Format = "Clnt1-fs-Incr-"
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

Исходя из вышеуказанного, рассмотрим наш пул _Clnt1-fs-Monthly_, предназначенный для хранения полных (Full)  резервных копий в течение года (365 дней). Здесь, с помощью параметра `Maximum Volume Jobs = 1` мы задали хранение одного задания резервного копирования в каждом томе, максимальное количество томов в пуле задано с помощью параметра `Maximum Volumes = 12`. Срок хранения тома до того, когда он сможет быть перезаписан определяется параметром `Volume Retention = 365  days`

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
Здесь поменяем только сетевой адрес, на котором _Storage Daemon_ ожидает подключения, с Localhost на 0.0.0.0. Таким образом, служба SD будет доступна для удалённых клиентов, отправляющих ей свои архивы.

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

