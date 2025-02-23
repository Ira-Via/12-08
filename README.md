# 12-08
12-08 «Резервное копирование баз данных» Васяева Ирина
# Задание 1
Вопрос резервного копирования данных, особенно в финансовой компании, имеет критическое значение, так как речь идет о защите жизненно важных данных клиентов и обеспечении непрерывности бизнеса. Для обеспечения высокой надежности работы баз данных существует несколько подходов к резервному копированию, и каждый из них может быть использован в зависимости от конкретных потребностей.  
Для сценария, когда необходимо восстанавливать данные в полном объёме за предыдущий день, подойдёт метод полного резервного копирования. Этот метод заключается в создании полной снимка базы данных на конец дня. Такой подход позволяет быстро и полностью восстановить данные до состояния на конец предыдущего дня. Важно также учитывать, что при использовании полного резервного копирования потребуется значительное место для хранения резервных копий. Можно рассмотреть применение ротационного хранения копий, чтобы не переполнять хранилище.  
Когда необходимо восстанавливать данные за час до предполагаемой поломки, разумнее использовать метод инкрементного резервного копирования. Этот метод предполагает создание резервных копий только тех данных, которые изменились с момента последнего полного или инкрементного резервного копирования. Таким образом, если поломка произошла, например, в 15:30, а последнее полное резервное копирование было сделано в 14:00, пользователь может восстановить данные сначала из полноценных копий, а затем загрузить инкрементальные копии, чтобы минимизировать потерю данных и восстановить систему до состояния на 14:59.  
Что касается момента переключения на работающую или починенную базу данных при поломке, существует технология аварийного переключения, которая служит для обеспечения высокой доступности системы. В этом случае важнейшую роль играют проекты по репликации баз данных или использование кластеров. Основная идея заключается в том, что используется активная-активная или активная-резервная схема работы баз данных, где основная база данных (активная) и её резервная копия (резервная) находятся в синхронизации. В случае сбоя активной базы, система автоматически переключается на резервную базу, что позволяет минимизировать время простоя и продолжать работу без значительных перерывов.  
Кроме того, некоторые современные решения по резервированию данных и восстановления после сбоев позволяют использовать механизмы, такие как "горячее резервирование", что дает возможность вести восстановление в реальном времени. Это может быть особенно полезно в условиях, когда актуальность данных критически важна, как в финансовых сервисах.  
Таким образом, для эффективного резервного копирования и восстановления данных в финансовой компании желательно комбинировать различные подходы и технологии, чтобы обеспечить надежность и скорость восстановления данных в зависимости от условий и потребностей бизнеса.
# Задание 2
В PostgreSQL для резервного копирования и восстановления данных используются утилиты `pg_dump` и `pg_restore`. Эти инструменты позволяют создавать резервные копии баз данных и восстанавливать их в случае необходимости. Рассмотрим их использование подробнее.  
Чтобы выполнить резервное копирование базы данных с помощью pg_dump, можно использовать команду следующего вида:
```
pg_dump -U username -h hostname -F c -b -v -f backup_file.backup dbname
```
В этой команде:  
* -U username — имя пользователя, который имеет доступ к базе данных.
* -h hostname — адрес хоста, на котором расположена база данных. Если вы работаете локально, этот параметр можно опустить.
* -F c — формат резервной копии, в данном случае — пользовательский (custom), который позволяет использовать pg_restore для гибкого восстановления.
* -b — включает в резервное копирование большие объекты (large objects).
* -v — включает детальный вывод сообщений (verbose).
* -f backup_file.backup — имя файла, в который будет сохранена резервная копия.
* dbname — имя базы данных, которую вы хотите сохранить.  
Для восстановления базы данных из резервной копии используется команда pg_restore. Например:
```
pg_restore -U username -h hostname -d dbname -v backup_file.backup
```
В этой команде:  
* -d dbname — имя базы данных, в которую вы хотите восстановить данные.  
Остальные параметры аналогичны тем, что мы использовали для резервного копирования.  
Теперь о возможности автоматизации этого процесса. Да, резервное копирование и восстановление данных можно автоматизировать с помощью различных инструментов и скриптов. Один из самых популярных способов автоматизации — использование cron на UNIX-подобных системах. С помощью cron можно настроить периодические задачи для выполнения команд pg_dump. Например, чтобы создать резервную копию базы данных каждый день в полночь, можно добавить следующую строку в файл crontab:  
```
0 0 * * * pg_dump -U username -h hostname -F c -b -f /path/to/backup/backup_$(date +\%Y-\%m-\%d).backup dbname
```
Этот пример создаст файл резервной копии с текущей датой в названии, что поможет организовать хранение резервных копий.  
Также для автоматизации процесса можно использовать различные языки программирования (например, Python, Bash) для написания более сложных сценариев, которые могут включать проверку состояния резервного копирования, отправку уведомлений и даже управление хранением старых резервных копий.
# Задание 3
Для инкрементного резервного копирования в MySQL можно воспользоваться бинарными логами, которые позволяют записывать все изменения, происходящие в базе данных. Это удобно для создания резервных копий, так как только изменения с момента последнего полного резервного копирования нужно сохранять после первого создания полной резервной копии.  
Пример команды для создания инкрементного резервного копирования может выглядеть так:  
1. Сначала создается полное резервное копирование базы данных:
```
mysqldump -u username -p --all-databases --routines --triggers > full_backup.sql
```
2. Затем, после включения бинарного логирования на сервере MySQL, можно делать инкрементальное резервное копирование путём копирования бинарных логов. Предположим, что вы сохранили бинарный лог с именем mysql-bin.000001, команда для получения его может выглядеть следующим образом:
```
mysqlbinlog mysql-bin.000001 > incremental_backup.sql
```
Это создаст файл incremental_backup.sql, который будет содержать все изменения, записанные в бинарном логе после создания полного резервного копирования.  
Теперь, что касается использования реплики по сравнению с обычным резервным копированием. Репликация имеет ряд преимуществ, особенно в условиях высокой нагрузки и необходимости обеспечения доступности данных. К основным из них можно отнести следующее:  
Во-первых, реплики позволяют уменьшить время простоя, поскольку при сбое основной базы данных можно быстро переключиться на реплику. Это обеспечивает непрерывность бизнеса и минимизирует потери данных.  
Во-вторых, реплика позволяет распределять нагрузку на чтение. При большом количестве запросов на чтение, реплики можно использовать для обработки запросов, что снижает нагрузку на основную базу данных и улучшает общую производительность системы.  
Третьим преимуществом является то, что репликация позволяет обеспечить более быстрые и частые резервные копии данных. В отличие от традиционного резервного копирования, которое может занимать много времени и приводить к задержкам, реплики могут обновляться в реальном времени и обеспечить более актуальные данные.  
Таким образом, использование реплик дает значительные преимущества в срочности восстановления, производительности и доступности данных по сравнению с традиционными методами резервного копирования.
