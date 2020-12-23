# Mysql-Handbook

Mysql优化：负载数据或者数据量大的时候可能会存在问题，但是要考虑系统的所有方面，主要讨论性能模式、sys模式的视图、语句性能分析器特性，MysqlWorkbench、Mysql监测、语句性能分析器。在调优的时候固定数据量在时间和其他非确定性因素存在的情况下，同一示例的输出将有所不同。由于锁竞争而出现的问题在21章、事务问题在22章。

性能模式：

语句事件表：
events_statements_current
select * from performance_schema.events_statements_current
当前执行的语句或空闲连接最新执行的查询。在执行存储过程的时候可能有多行。
events_statements_history
select * from performance_schema.events_statements_history
每个连接最后执行的几条语句。每个连接语句默认上限为10条（show variables like 'performance_schema%history%size’;）当连接关闭时，连接最后执行的几条语句记录会被删除。
events_statements_history_long
select * from performance_schema.events_statements_history_long
实例的最新查询（不关心是哪个连接执行的），该表还包括已经关闭的连接执行的语句。一般情况下禁用此表的使用者。行数上限为10000（show variables like 'performance_schema%history%size';）。
events_statements_summary_by_digest（最常用的表）
select * from performance_schema.events_statements_summary_by_digest
按默认模式和摘要分组的语句报表统计数据
events_statements_summary_by_account_by_event_name
select * from performance_schema.events_statements_summary_by_account_by_event_name
按账户和事件名称分组的报表统计数据，事件名称显示的是执行的那种语句（例如：statement/sql/select）。
events_statements_summary_by_host_by_event_name
select * from performance_schema.events_statements_summary_by_host_by_event_name
按账户的主机地址和事件名称分组的语句统计
events_statements_summary_by_program
select * from performance_schema.events_statements_summary_by_program
由执行语句的存储程序（事件、函数、过程、表或触发器）分组的语句的统计数据。这对与查找执行次数最多的存储程序（事件、函数、过程、表或触发器）非常有用。
events_statements_summary_by_thread_by_event_name
select * from performance_schema.events_statements_summary_by_thread_by_event_name
按线程和事件名分组的语句统计信息，只包括当前连接的线程
events_statements_summary_by_user_by_event_name
select * from performance_schema.events_statements_summary_by_user_by_event_name
按账户用户名（如：root）和事件名称分组的语句统计数据信息
events_statements_summary_global_by_event_name
select * from performance_schema.events_statements_summary_global_by_event_name
按事件名称分组的语句统计信息
events_statements_histogram_by_digest（没找到该信息）
select * from performance_schema.events_statements_histogram_by_digest
直方图统计：按默认的模式和摘要分组
events_statements_histogram_global
select * from performance_schema.events_statements_histogram_global
直方图统计：其中所有查询都聚合在一个直方图中
threads
select * from performance_schema.threads
实例中所有的线程信息，包括后台线程（background）和前台线程（foregroud）
可以使用此表而不是SHOW ProcessLIST
除了process列表信息外，还有列显示线程是否被仪器化，操作系统线程ID等
总结：除了列两个直方图和线程表外，其他表都有类似的列。最常用的表是（events_statements_summary_by_digest：本质上是从上次实例重置（通常是重启Mysql）以来在实例上执行的所有查询报表信息，按默认的模式和摘要分组）。

表19-1events_statements_summary_by_digest表中的列
栏名 说明
schema_name 执行查询时的默认模式。 如果没有模式是默认的，则值为NULL。
digest 规范化查询的摘要。 在MySQL8中，这是一个sha256 hash。
digest_text 规范化查询。
count_star 已执行查询的次数。
sum_timer_wait 执行查询所花费的总时间。 请注意，在执行时间超过30周后，该值会流过去。
min_timer_wait 查询执行得最快。
avg_timer_wait 平均执行时间。 这与sum_timer_wait/count_star相同，除非sum_timer_wait有溢出。
max_timer_wait 查询执行得最慢。
sum_lock_time 等待表锁的总时间。
sum_errors 执行查询时遇到的错误总数。
sum_warnings 执行查询时遇到的警告总数。
sum_rows_affected 查询已修改的行总数。
sum_rows_sent 已返回（发送）给客户端的行总数。
sum_rows_examined 查询已检查的行总数。
sum_created_tmp_disk_tables 磁盘内部临时表的总数由查询创建。
sum_created_tmp_tables 内部临时表的总数-是否创建在内存或磁盘上-由查询创建。
sum_select_full_join 执行完整表扫描的连接总数没有连接条件的索引，或者没有连接条件。 这与增加Select_full_join状态变量相同。
sum_select_full_range_join 使用全范围搜索的连接总数。 这就是增加Select_full_range_join状态变量的相同。
sum_select_range 查询使用范围搜索的总次数。这与增加Select_range状态变量相同。
sum_select_range_check 连接不在的查询的连接总数有一个索引，在每一行之后检查索引的使用情况。这是相同的，增加Select_range_check状态变量。
sum_select_scan 查询执行完整表的总次数扫描连接中的第一个表。 这与增加Select_scan状态变量相同。
sum_sort_merge_passes 已完成的排序合并传递的总数对查询的结果进行排序。 这是相同的增量Sort_merge_passes状态变量。
sum_sort_range 使用范围完成排序的总次数。 是增加Sort_range状态变量的相同。
sum_sort_rows 排序的行总数。 这与增加Sort_rows状态变量相同。
sum_sort_scan 通过扫描表完成排序的总次数。这与增加Sort_scan状态变量相同。
sum_no_index_used 没有索引的总次数用于执行查询。
sum_no_good_index_used 总次数未使用好指标。 这意味着EXPLAIN输出中的Extra列包括“为每个记录检查范围。”
first_seen 当查询被第一次看到的时候。 当表被截断时，第一个可见值也被重置。
last_seen 当最后一次看到查询时。
quantile_95 查询延迟的第95百分位数。 也就是说，95%的查询在给定的时间或较短的时间内完成。
quantile_99 查询延迟的第99百分位数。
quantile_999 查询延迟的第99.9百分位数。
query_sample_text 查询规范化之前的一个例子。 你可以用这个获取查询的查询执行计划。
query_sample_seen 当看到示例查询时。
query_sample_timer_wait 示例查询要执行多长时间。 

events_statements_summary_by_digest（基础统计表）有一个唯一索引(unique index)在列（schema_name, digest）用于对数据进行分组。events_statements_summary_by_digest表中最多可以有performance_schema_digests_size（show variables like 'performance_schema_digests_size’;默认10000），当插入最后一行时，schema_name和digest都设置为NULL，统计所有前面插入的信息并且Performance_schema_digest_lost递增。events_statements_summary_by_digest表中的信息也可用于events_statements_current、events_statements_history、events_statements_history_long的信息统计。

注意：由于数据是由scheme_name、digest分组的，所以当应用程序在默认模式（例如：在Mysql Shell（或等同的客户端）使用\use world或—shceme命令行选项）？？？？？？

两组列需要更多的解释（分位数quantile、查询示例query_sample）
分位数（quantile）列是根据直方图统计来确定的，基本上，在默认schema和bucket为95%的查询执行语句下使用events_statements_summary_by_digest表，那么bucket被用于确定第95位。
查询示例（query_sample）：如果满足下面三个条件之一，则替换样本查询
在默认模式下第一次遇到的digest（摘要）
与当前用作示例查询的查询相比，新的摘要和模式出现的IIMER WAIT值更高(即更慢)
如果performance_schema_max_digest_sample_age选项的值大于0，并且当前示例查询的时间大于performance_schema_max_digest_sample_age秒
performance_schema_max_digest_sample_age的值默认为60秒，如果每分钟监视events_statements_summary_by_digest表，就可以很好地工作。这样，监控代理将能够在每一分钟的间隔内选取最慢的查询，并获得最慢查询的完整历史记录。如果您的监视间隔更大，请考虑增加performance_schema_max_digest_sample_age的值

正如您可以从列列表中看到的，有足够的机会查询满足某些需求的语句。诀窍是查询重要的事情。什么才算重要取决于具体情况，因此，不可能给出适用于所有情况的具体查询。例如，如果您从监视中知道使用内存或磁盘的大量内部临时表存在问题，那么sum_created_tmp_disk_tables、sum_created_tmp_tables列都是筛选的好选择。

有些情况是大家都感兴趣的。需要进一步调查的一些情况的例子包括：
与发送回客户端（sum_rows_sent）或已修改的行数（sum_rows_affected）相比，有大量检查的行（sum_rows_examined）。 这可能表明索引使用情况不佳
没有使用索引（sum_no_index_used）或没有使用好的索引（sum_no_good_index_used）的总和很高。这可能意味着查询可以从新索引或重写查询中获益
完全连接（full join）的数量很高。这表明要么需要一个索引，要么缺少连接条件。
范围检查（sum_select_range_check）的数量很高。这可能意味着您需要更改查询中表上的索引
如果分位数（quantile）延迟在向更高分位数逼近时表现出严重的退化，则可能表明您有时在及时解决查询时存在问题。这可能是由于实例通常被重载、锁问题、某些条件触发糟糕的查询计划或其他原因
在磁盘中创建的内部临时表（sum_created_tmp_disk_tables）的数量很高。这可能意味着您需要考虑用于排序和分组的索引、允许用于内部临时表的内存量，或者其他可能首先阻止将内部临时表写入磁盘或创建内部临时表的更改
排序合并（sum_sort_merge_passes）的数量很高。这可能意味着该查询可以从更大的排序缓冲区中获益
注意：如果你仍然在使用MYSQL 5.7，增加排序缓冲区大小的值时要小心，因为它会降低性能，即使它降低了排序合并的数量。MYSQL 8中，排序缓冲区得到了改进，大缓冲区的性能下降更小。不过，不要把尺寸增大到超出需要的程度
已执行（count_star）数很多。这并不意味着查询有任何问题，但在某些情况下，查询执行得越频繁，对查询的改进产生的影响就越大，不必要的查询执行也可能导致较高的执行计数
错误（sum_errors）或警告（sum_warnings）的数量很高。虽然这可能不会影响性能，但这表明出了问题。是否注意到有些查询总是生成一个警告，例如，EXPLAIN，因为它使用警告返回附加信息
您应该知道，仅仅因为查询满足了这些条件之一，并不意味着需要更改任何东西。例如，考虑一个从表中聚合数据的查询。该查询可能检查表的大部分，但只返回几行。在没有有意义的索引可以帮助的情况下，它甚至可能需要全表扫描。从检查的行数和发送的行数之间的比率来看，查询的性能很差，而且no索引计数器可能在增加。然而，查询很可能只完成返回所需结果所需的最少工作量。如果您确定查询是一个性能问题，您将需要找到一个不同于添加索引的解决方案;例如，您可以在非高峰期间执行查询并缓存结果，或者您可以有一个单独的实例，在其中执行这样的查询（备份实例）

清单19-1显示了一个查找默认模式和语句摘要组合的示例，这些默认模式和语句摘要组合自上次重置事件摘要表中的事件语句摘要以来被执行的次数最多。
SELECT * FROM performance_schema.events_statements_summary_by_digest ORDER BY COUNT_STAR DESC LIMIT 1



输出显示，按名称查询world模式中的city表是执行最多的查询。您应该将值计数星（count_star）与其他查询进行比较，以了解与其他查询相比该查询的执行频率。在本例中，您可以看到查询平均每次执行返回1.3行(????)，但检查4079行（检查的行数(4079)= sum_rows_examined(417481571) / sum_no_index_used(102349)）。这意味着为返回的每行检查3000多行。因为这是一个经常执行的查询，提示需要在用于过滤的Name列上建立索引。输出的底部显示了一个查询的实际示例，您可以在下一章中描述的使用EXPLAIN来分析查询执行计划

执行语句的直方图统计信息（有两个可用的直方图表）：

有两个直方图表可用的事件语句摘要直方图(events_statements_histogram_by_digest)和事件执行语句直方图全局(events_statements_histogram_global)。两者之间的区别在于前者具有按默认模式和摘要分组的直方图信息，而后者包含分组在一起的所有查询的信息。直方图信息可以用于确定查询延迟的分布，类似于在events_statements_summary_by_digest表中讨论的分位数列，但粒度更细。这些表是自动管理的。

如前所述，预处理语句不包括在语句事件表中。相反，您需要使用prepared_statements_instances表

Prepared Statements Summary(预处理语句统计)

预处理语句对于加速连接中重用的查询的执行非常有用。例如，如果您有一个应用程序一直使用相同的连接，那么您可以准备应用程序使用的语句，然后在需要时执行准备好的语句。

预处理语句使用占位符，因此在准备查询时只需要提交模板。这样，您就可以为每次执行提交不同的参数。以这种方式使用时，预处理语句充当一个语句目录，应用程序可以使用这些语句和给定执行所需的参数

清单19-2显示了在应用程序中通过SQL接口使用准备好的语句的一个简单示例，您通常将使用一个连接器以更透明的方式处理准备好的语句。例如，对于我的SQL连接器/Python，您告诉它要使用准备好的语句，连接器将在您第一次执行语句时自动为您准备语句。但基本原则是相同的。



SQL接口使用用户变量将语句和值传递给MYSQL。第一步是准备语句;然后可以根据需要多次使用它传递查询所需的参数。最后，撤销准备好的语句。

当您想要调查准备好的语句的性能时，您可以使用准备好的语句实例表。这些信息类似于事件语句摘要摘要表中的信息。清单19-3显示了清单19-2中使用的预准备语句的示例输出

与events语句表的主要区别是，没有分位数统计信息和查询示例，主键是OBJECT_INSTANCE_BEGIN(1999818114352)，即准备好的语句的内存地址，而不是默认模式和摘要上的唯一键。实际上，prepared_statements_instances表中甚至没有提到默认模式和摘要

由于主键是预备语句的内存地址，因此只在预备语句存在时维护预备语句统计信息。因此，当由于连接关闭而显式或隐式地释放语句时，统计信息将被清除

关于语句统计的讨论到此结束。还有一些更高级的统计数据，比如表I/O统计

Ttable I/O Summaries

与传统I/O输入-输出的一般概念不同。Table I/O它不涉及磁盘I/O。相反，它是对表上花费时间的一般衡量。也就是说，表的磁盘I/O越多，在表上花费的时间也就越多。

有两个性能模式表，其中包含表的延迟统计信息:
table_io_waits_summary_by_table:包含read详细信息的表的聚合信息。写、取、插入和更新I/O
table_io_waits_summary_by_index_usage:与table_io_waits_summary_by_table表相同的信息，除了统计信息是每个索引或缺少的。
这些表使您能够详细了解如何使用这些表以及在各种操作上使用了多少时间。有7组活动，它们同时存在和、最小、平均和最大的延迟和操作数量。表19-2显示了基于列名的组

Table 19-2 The groups of latencies for table and index I/O statistics
Group                  Colums                               Descriptions
整个表或索引的统计信息
所有读操作的汇总统计信息。目前只有一个读操作，fetch，所以读统计数据将与读取统计数据相同
所有写操作的汇总统计信息。写操作是插入操作。更新。和删除
获取记录的统计信息。没有调用select的原因是，获取记录的目的可能不是用于select语句
用于插入记录的统计信息
用于更新记录的统计信息
用于删除记录的统计信息

在清单19-4中可以看到关于world.city表的table_io_waits_summary_by_table_table中这些列的信息示例。
select * from performance_schema.table_io_waits_summary_by_table


在这个输出中，除了未删除的行之外，该表有广泛的用途。还可以看到，大部分时间都花在读取数据上(（sum_timer_wait）125987200409940皮秒中的(sum_timer_read)122703207563448皮秒，即97%)

清单19-5显示了同一个table_but_using_the_table_io_waits_summary_by_index使用情况表的输出。usage列与table_io_waits_summary_by_table表相同，在示例中大部分都被忽略了，以重点关注两个表之间的差异。如果前面示例中有任何额外的索引，那么将返回更多的行

考虑COUNT_STAR的三个值。如果你把它们加起来。20004+ 549000 +417489729=418058733，你得到相同的值COUNT_STAR在表I/O等待汇总表表。这个示例显示了相同的数据，但是在city表上的两个索引和NULL索引中分开显示。这意味着没有使用索引。这使得表I/O等待索引使用情况汇总表对于评估索引的有用性以及是否表扫描是否被执行非常有用

花一分钟考虑取、插入、更新和删除计数器何时增加，以及针对哪些索引增加，这是很有用的。考虑到世界。城市表，它在ID列中有一个主键，在CountryCode列上有一个二级索引。这意味着您可以根据所使用或缺少的索引推理出city表设置三种类型的过滤器：
By Primary Key: 通过主键定位行的行数（例如：where id = 130）
二级索引：使用CountryCode索引定位的行数，（例如，其中CountryCode = ‘AUS’）
没有索引：使用全表扫描定位的行数（例如：where name = 'sydeney'）
表19-3显示了使用带有SELECT、UPDATE或DELETE语句的三个示例WHERE子句以及执行INSERT语句的矩阵，TNSERT语句没有WHERE子句。所以有一点不同。为每个受影响的索引。其中列出了读取和写入的数量。Rows列显示每个语句返回或受影响的行数。

表19-3各种查询对表I/O计数器的影响
file:///Applications/Quiver.app/Contents/Resources/dist/quiver-image-url/36861780A8056208B5FFFADF390AD9B1.jpg
从表中可以得到的一个重要信息是，对于UPDATE和DELETE语句，尽管它们是写语句，但仍然有read操作。原因是，在更改行之前，仍然必须先找到行。另一个观察结果是，当使用辅助索引或no索引更新或删除行时，读取的记录多于符合条件的记录。最后，插入行算作非索引操作。

如何理解I/O延迟?

当您看到显示I/O延迟的监视图时(无论是表I/O还是文件I/O)，很容易得出存在问题的结论。在此之前，请先后退一步，并考虑一下这些数据的含义。
从性能模式度量的I/O延迟的增加既不是好事也不是坏事。这是事实。这意味着某些东西正在进行I/O操作，如果出现峰值，则意味着在此期间有比平时更多的I/O操作，但除此之外，您无法从事件本身得出结论。
使用这些数据的一种更有用的方式是在出现问题时进行报告。这可能是系统管理员报告磁盘已被100%利用，或者最终用户报告系统运行缓慢。然后，你可以去看看发生了什么。如果磁盘IO在那个时间点异常高，那么这可能是相关的，您可以从那里继续您的研究。另一方面，如果I/O正常，那么高利用率可能是由MYSQL以外的其他进程造成的，或者磁盘阵列中的磁盘正在重新构建，或类似的原因。

使用table_io_waits_summary_by_table和table_io_waits_summary_by_index_usage_tables中的信息，可以确定哪些表最适用于各种工作负载。例如，如果有一个表特别忙于写操作，您可能需要考虑将其表空间移动到一个更快的磁盘上。在做出这样的决定之前，您还应该考虑实际的文件I/O

File I/O

与刚才讨论的表I/O不同，文件I/O统计信息是针对MYSQL使用的各种文件所涉及的实际磁盘I/O的。这是对表I/O信息的很好的补充。
有三个性能模式表可以用来获取关于MySQL实例的文件IO的信息。
events_waits_summary_global_by_event_name
这是一个按照事件名称分组的汇总表。通过查询以wait/io/file/开头的事件名称，您可以获得按io类型分组的I/O统计信息，例如，读写二进制日志文件时使用单个事件(wait/io/file/sql/binlog)导致的I/O。注意，设置为wait/io/table/sql/handler的事件对应于刚才讨论过的表I/O;包含表I/O允许您轻松地比较花费在文件I/O和表I/O上的时间
file_summary_by_event_name
这类似于events_waits_summary_global_by_event_name表，但只是包含文件I/O，事件分为读、写和其他项
file_summary_by_instance
这是一个汇总表，按实际文件分组，事件分为读、写和杂项。例如，对于二进制日志，每个二进制日志文件有一行
这三个表都很有用，您需要根据要查找的信息在它们之间进行选择。例如，如果您想要文件类型的聚合，则events_waits_summary_global_by_event_name表和file_summary_by_event_name表是更好的选择，而研究单个文件的I/O，则file_summary_by_instance表更有用

file_summary_by_event_name表和file_summary_by_instance表将事件分为读、写和杂项。读和写很容易理解。杂项I/O是所有不读或写的东西。这包括但不限于创建、打开、关闭、删除、刷新和获取文件的元数据。杂项操作都不涉及数据传输，因此不存在杂项字节计数器。

清单19-6显示了事件等待汇总全局事件名称表中可用数据的示例。该查询查找花费在I/O上的总时间最多的事件。
select * from performance_schema.events_waits_summary_global_by_event_name where event_name like 'wait/io/file/%'order by sum_timer_wait desc

这表明在本例中。最活跃的事件是InnoDB重做日志文件。这是一个相当典型的结果。每一个事件都有一个相应的工具。一个特别有趣的事件是wait/io/file/innodb/innodb_data_file，它用于innodb表空间文件的I/O

events_waits_summary_global_by_event_name表的一个缺点是，花费在I/O操作上的所有时间都被汇总到一个总计数器中，而不是仅有读和写。也只有计时可用。如果您使用file_summary_by_event名称表，您可以获得更多的详细信息。



----------------------------
20. Analyzing Queries

在前一章中，您学习了如何查找需要优化的查询。现在是进行下一步的时候了——分析查询以确定为什么它们不能按预期执行。分析期间的主要工具是EXPIAIN语句，它显示优化器将使用的查询计划。与此相关的是优化器跟踪，该跟踪可用于调查优化器最终使用查询计划的原因。另一种可能性是在性能模式中使用语句和阶段信息，以查看存储过程或查询在哪里花费的时间最多。这三个主题将在本章中讨论。

关于EXPIAIN statement的讨论是本章中最大的部分，共分为四个部分：
EXPLAIN Usage: explain在语句中的基本用法
explain格式：EXPLAIN格式化查询计划可以在其中查看的每种格式特定的详细信息。这包括EXPIAIN语句显式选择的格式和MYSQL Workbench使用的可视化解释
explain output:EXPLAIN输出:对查询计划中可用信息的讨论
EXPLAIN Examples: 一些使用EXPIAIN语句的示例，讨论返回的数据
explain的基本语法

EXPLAIN语句返回MYSQL优化器将对给定查询使用的查询计划的概述。它同时非常简单，也是查询调优中比较复杂的工具之一。它很简单，因为您只需要在想要调查的查询之前添加EXP IAIN命令;它很复杂，因为理解这些信息需要一定程度上理解我的SQL及其优化器的工作方式。您既可以将EXPLAIN与您显式指定的查询一起使用，也可以与当前由另一个连接执行的查询一起使用。本节介绍EXPLAIN语句的基本用法。

用于显示查询

您可以通过在查询前添加EXPLAIN来生成查询的查询计划，还可以选择添加FORMAT选项来指定是否希望以传统表格式返回结果。使用JSON格式。或者以树格式。它支持SELECT、DELETE、INSERT、RE PLACE和UPDATE语句。查询不会被执行(但请参阅下一小节关于解释分析的异常)，因此可以安全地获取查询

如果你需要分析复合查询如存储过程和存储函数需要先执行了分割成单个查询,然后使用前女友PIA应该分析每个查询的一种方法来确定单个查询的存储程序是使用性能模式。本章后面将给出一个实现这一目标的例子

EXPIAIN最简单的用法就是用你想要分析的查询指定EXPLAIN:

在上面的示例中，<query>是您想要分析的查询。使用EXPIAIN语句而不使用FORMAT选项将返回传统表格式的结果。如果你想指定格式，你可以通过添加FORMAT= TRADITIONAL | JSON | TREE:

哪种格式是首选的取决于您的需要。当需要概述查询计划、使用的索引和关于查询计划的其他基本信息时，使用传统格式更容易。JSON格式提供了更多的细节，更易于应用程序使用。例如，MYSQL Workbench中的Visual Explain使用json格式的输出。

TREE格式是MYSOL 8.0.16及更高版本中支持的最新格式。它要求使用火山迭代器执行器执行查询，在编写时，所有查询都不支持火山迭代器执行器。树格式的一种特殊用法是用于EXPLAIN ANALYZE语句。

EXPLAIN ANALYZE：

从MYSQL 8.0.18开始，EXPLAIN ANALYZE语句是新的，它是标准EXPLAIN语句的树格式的扩展。关键的区别是，EXPLAIN ANALYZE实际执行查询，并且在执行时收集执行的统计信息。在执行语句时，查询的输出被抑制，因此只返回查询计划和统计信息。与树输出格式一样，需要使用Volcano迭代器执行器

注意在编写本文时，火山迭代器执行器（Volcano iterator executor ）上的要求将您可以使用EXPLAIN ANALYZE的查询限制在SELECT语句的一个子集内。预计支持的查询范围将随着时间的推移而增加。

EXPLAIN ANALYZE的用法与您已经看到的EXPLAIN语句非常相似：

EXPIAIN ANAIYZE的输出将在本章后面与树格式输出一起讨论。
本质上，EXPLAIN ANALYZE只对显式查询有效，因为需要从头到尾监视查询。另一方面，普通的EXPIAIN语句也可以用于正在进行的查询。

Usage for Connections（连接的使用情况）：
假设您正在调查一个性能较差的问题，并且注意到有一个查询已经运行了几个小时。您知道这不应该发生，所以您想分析为什么查询这么慢。一种选择是复制查询并为其执行EXPLAIN。但是，这可能不会提供您需要的信息，因为自慢速查询开始以来索引统计信息可能已经更改，因此现在分析查询不会显示导致慢速性能的实际查询计划。

更好的解决方案是请求用于慢速查询的实际查询计划，您可以使用EXPIAIN语句的EXPLAIN for连接变量获得慢速查询。如果您想尝试它，例如，您需要一个长时间运行的查询。

在world.city表内，大约需要420秒(每行0.1秒)。您将需要您想要调查的查询的连接id，并将其作为一个参数传递，以证明您可以从进程列表信息中获得连接id。例如，如果你使用sys.session视图中，连接id可以在conn_id列中找到:
select conn_id, current_statement, statement_latency from sys.session where command = 'Query' order by time desc limit 1

为了保持输出的简单性，对于本示例，它被限制为感兴趣的连接。查询的连接id是8。您可以使用它来获得查询的执行计划，如下所示:
Explain for connection 8;
3012 - EXPLAIN FOR CONNECTION command is supported only for SELECT/UPDATE/INSERT/DELETE/REPLACE, Time: 0.000000s

您可以选择添加您想要的格式，方法与显式指定查询时相同。如果您使用的客户端与MYSQL Shell不同，过滤后的列可能显示100.00。在讨论输出的含义之前，有必要熟悉一下输出格式。
EXPLAIN Formats（explain的输出格式）

当您需要检查查询计划（quey plans）时，您可以在几种格式之间进行选择。你选择哪一个主要取决于你的偏好。也就是说，JSON格式确实比传统格式和树格式包含更多的信息。如果您喜欢查询计划的可视化表示，那么MYSQL Workbench中的可视化解释是一个很好的选择。

本节将讨论每种格式，并显示以下查询的查询计划的输出。

该查询在欧洲面积最小的十个国家中找到五个最大的城市，并按照城市人口的降序排列。选择这个查询的原因是，它显示了各种输出格式如何表示子查询、排序和限制。在语句中由EXPLA返回的信息将不会在本节中讨论，这是在解释示例部分中讨论的。

注意EXPIAIN语句的输出依赖于优化器开关的设置、索引统计信息和mysql中的值。引擎成本和mysqlserver成本表，因此您可能不会看到与示例中相同的情况。示例输出已经使用默认值和world示例演示数据库加载分析表执行表加载完成之后,他们一直在MYSQL创建Shell警告在哪里获取自动默认(但是警告只包含在输出时讨论)。如果你不使用MYSQL Shell，你将不得不执行显示警告来检索警告。

查询计划输出非常冗长。为了便于比较输出，本节中的示例与查询结果结合成文件解释格式。在本书的Github存储库中，对于树输出格式(包括EXPIAIN ANALYZE格式)，在列名和查询计划之间添加了一个额外的新行，以使树的层次结构显示更清晰。

此惯例贯穿本章。
Traditional Format（传统格式）：

当您执行不带格式参数或将格式设置为传统的EXPLAIN命令时，输出将作为一个表返回，就像您查询了一个普通表一样。当您希望对查询计划进行概述，并且由人工数据库管理员或开发人员检查输出时，这非常有用。

提示:表输出可能非常宽，特别是如果有许多分区、可以使用的几个可能的索引或几条额外信息。当您调用mysql命令行客户端时，您可以通过使用——vertical选项请求以垂直格式得到输出，或者您可以使用\G终止查询。（page 610）

输出中有12列。如果一个字段没有任何值，则使用NULI。下一节将讨论每一列的含义。清单20-1显示了示例查询的传统输出。



清单20-1传统解释输出的示例

注意第一个表是如何被称为<derived 2>的。这是针对country表上的子查询的，数字2表示执行子查询的id列的值。额外的列包含诸如查询是否使用临时表和文件排序之类的信息。在输出的末尾是优化器重写后的查询。在许多情况下没有太多更改，但在某些情况下，优化器可能能够对查询进行重大更改。在重写的查询中，请注意注释是如何使用的，例如/ select#1 */来显示查询的这一部分使用了哪个id值。在重写的查询中可能还有其他提示来说明如何执行查询。重写的查询作为SHOW WARNINGS返回(默认情况下，由MYSQL Shell隐式执行)。

输出结果可能看起来难以应付，而且很难理解如何使用这些信息来分析查询。一旦其他输出格式化了select类型和join类型的详细信息，并且讨论了额外的信息，将会有一些示例使用EXPIAIN信息。

如果你想以编程的方式分析查询计划，你可以像处理普通SELECT查询那样处理EXPIAIN输出——或者你可以请求JSON格式的信息，其中包含一些额外的信息。

JSON Format（JSON格式）：
从MYSQL 5.6开始，可以使用JSON格式请求EXPIAIN输出。与传统选项卡相比，JSON格式的一个优势。format是使用JSON格式增加的灵活性，以一种更符合逻辑的方式对信息进行分组。

JSON输出中的基本概念是查询块（query block）。查询块定义了查询的一部分，并且可以包含json自己的查询块。这允许MYSQL将查询执行的细节指定给它们所属的查询块。这也可以从示例查询的输出中看到，如清单20-2所示




如您所见，输出非常冗长，但是这种结构使得查看哪些信息属于一起以及查询的各个部分如何相互关联变得相对容易。在本例中，有一个嵌套循环，包含两个表(co和ci)。co表本身包含一个新的查询块，这是一个使用country表的物化子查询（materialized subquery）。（page 616）

JSON格式还包括额外的信息，比如cost_info(成本信息)元素中每个部件的估计成本。成本信息可用于查看优化器认为查询中成本最高的部分在哪里。例如，如果您看到查询的某个部分的成本非常高，但是您对数据的了解意味着您知道它的成本应该很低，那么这可能表明索引统计信息不是最新的，或者需要直方图。

使用json格式输出的最大问题是有如此多的信息和如此多的输出行。解决这个问题的一种非常方便的方法是使用MYSQL Workbench中的Visual Explain特性，在讨论了树格式的输出之后，我们将介绍这个特性

Tree Format（树格式）：

虽然是从mysql 8.0开始支持，但是本地分析，可以装两个，一个mysql 5.7 一个mysql8.0，分析解决5.7的问题

树格式侧重于描述查询是如何执行的，根据查询各部分之间的关系和执行各部分的顺序，从这个意义上讲，它可能听起来类似于JSON输出;但是，树格式更易于阅读，并且没有太多的细节。树格式是在MYSQL 8.0中作为实验特性引入的。并依赖于火山迭代器执行器。从MYSQL8.0.18开始，树格式也用于EXPLAIN ANAIYZER特性

清单20-3显示了示例查询使用树形格式的输出，此输出是非分析版本。稍后将为相同的查询显示EXPLAIN ANALYZE的输出示例，因此您可以看到它们之间的区别

输出很好地概述了查询是如何执行的。通过从内部和外部读取输出，在一定程度上更容易理解执行。对于嵌套循环，您有两个表，第一个是co上的表扫描(缩进已经减少)。


在这里，您可以看到co表是如何成为一个物化子查询（materialize subquery）的，首先对world表进行表扫描，然后对大陆应用一个过滤器，然后基于表面积进行排序，然后将结果限制为10行。

嵌套循环的第二部分更简单，因为它只包含使用CountryCode索引对ci表(城市表)进行索引查找

当使用内连接解析嵌套循环时，结果是流的(即：非物化)排序，并返回前五行。

虽然这没有像JSON输出那样提供非常详细的图像，但它仍然包含关于查询计划的大量信息。这包括每个表的估计成本和估计行数。例如，从排序步骤在国家的表面面积。

一个好的问题是，这与查询表的实际成本有什么关系?可以使用EXPLAIN ANALYZE语句。清单20-4显示了为示例查询生成的输出示例。


这与FORMAT= tree的树输出相同，只是每一步都有关于性能的信息。如果您查看ci表的行，可以看到有两种计时方式，行数和循环数(为了提高可读性而重新格式化)。

在这里，对于预期的18行(每个循环)，估计成本为4.69。实际统计数据显示，第一行是在0.012毫秒之后读取的，所有行都是在0.013毫秒之后读取的。总共有10个循环(每个循环用于10个国家)，每个循环平均取2行，总共20行，因此，在本例中，估计不是很准确(因为查询只选择小国家)。


注意，EXPLAIN ANALYZE的行数是取整为rows=2和loops=10的每个循环的平均行数，这意味着总读取行数在15到24之间。在这个特定的示例中，使用性能模式中的table_io_waits_summary_by_table表可以看到读取了15行。

如果在我的SQL 8.0.18或更高版本中有使用散列连接的查询，您将需要使用树形格式的输出来确认何时使用散列连接算法。例如，如果city表使用hash join与country表连接。

注意，连接是内部散列连接（Inner hash join），country表上的表扫描是作为使用散列给出的。

到目前为止，所有示例都使用了基于文本的输出。特别是json格式的输出可能很难用于获得查询计划的概览。可视化解释（Visual Explain）是更好的选择

Visual Explain（可视化解释）：

可视化解释特性是MYSQL Workbench的一部分，它通过将json格式的查询计划转换为图形表示来工作。在第16章中，当您研究向sakila.file添加直方图的效果时，您已经使用了Visual Explain。

如图20-1所示，点击Icon前面的放大镜图标，你就可以得到可视化的解释图


如果查询执行时间很长或者查询修改了数据，那么这是生成查询计划的一种特别有用的方法。如果你已经执行了查询，你可以选择点击结果网格右边的执行计划图标，如图20-2所示


可视化解释图被创建为一个流程图，每个查询块和表有一个矩形。使用其他形状(如用于连接的钻石)来描述数据的处理。图20-3显示了Visual Explain中使用的每个基本形状的示例。

在图中，一个查询块是灰色的，而表的两个示例(子查询中的单行查找和全表扫描)是蓝色和红色的，表框下面的文本分别以标准文本和e显示表名或别名。灰色块也被使用，例如，在联合的情况下。索引名称以粗体显示。带有圆角的矩形描述了行上的操作，如排序、分组、不同操作等等。

左上角的数字是该表、操作或查询块的相对成本。表和连接的右上角的数字是估计要结转的行数。操作的颜色用来显示应用操作的成本有多高。表还根据表访问类型使用颜色，主要是对类似的访问类型进行分组，其次是指示访问类型的成本。使用Visual Explain估算的成本，颜色和成本之间的关系可以在图20-4中看到。


蓝色(1)是耗时最少的;绿色(2)、黄色(3)和橙色(4)表示低到中等成本:耗时最高的访问类型和操作是红色，表示高(5)到非常高(6)耗时。

颜色组之间有许多重叠的部分。每个成本估算都考虑了一个“平均”用例，因此成本估算不应该被视为绝对真理。查询优化是复杂的，对于一个特定的查询，有时一种通常比另一种方法便宜的方法可以提供更好的性能。

注意，本书的作者曾经决定改进一个查询，该查询的查询计划看起来很糟糕:内部临时表、文件排序、糟糕的访问方法，等等。在花了很长时间重写查询并验证表是否具有正确的索引之后，查询计划看起来很漂亮—但结果却是查询的性能比原始查询差。教训:始终在优化后测试查询性能，不要依赖于访问方法和操作的成本是否在纸面上有所改善。

对于表来说，耗时与访问类型相关，即传统EXPIAIN输出中的类型列和json格式输出中的access_type字段的值。图20-5显示了Visual Explain如何表示当前存在的12种访问类型。对访问类型的解释将推迟到下一节。

此外，Visual Explain有一个“未知”的访问类型，颜色为黑色，以防遇到未知的访问类型。访问类型根据它们的颜色和大致成本从左到右然后从上到下排序。

图20-6将所有这些放在一起，显示了本节中使用的示例查询的查询计划。

从图的左下角到右下角再往上读。所以图显示的子查询country表执行全表扫描,然后另一个全表扫描的物化co表的行加入ci(city)表使用nonunique索引（非唯一索引）查找最后,结果是使用临时表和文件分类排序。

如果你想要比图表所显示的更多细节，你可以将鼠标悬停在你想要了解更多信息的查询计划的部分，图20-7显示了ci表包含的细节示例。

弹出框架不仅显示了JSON输出中可用的其余细节，还提供了帮助理解数据含义的提示。所有这些都意味着可视化解释是通过查询计划开始分析查询的一种很好的方式。随着经验的积累，您可能更喜欢使用基于文本的输出，特别是如果您更喜欢使用shell。但是不要因为你认为使用基于文本的输出格式更好而放弃Visual Explain。即使对于专家来说，可视化解释也是理解查询如何执行的一个很好的工具。

希望通过对输出格式的讨论，您可以了解EXPIAIN可以提供哪些信息。然而，为了充分理解和利用它，有必要深入研究信息的意义。

EXPLAIN Output

explain output中有许多可用的信息，因此有必要深入研究这些信息的含义。本节首先概述传统输出和json格式输出中包含的字段，然后将更详细地介绍选择类型和访问类型以及其他信息

EXPLAIN Fields
在您的工作中建设性地使用EXPLAIN语句以改进查询的第一步是了解哪些信息是可用的。这些信息的范围从对查询部分的引用的id到关于可用于查询的索引的详细信息，并与使用的索引和应用的优化器特性进行了比较。

如果在第一次阅读定义后不能回忆起所有的细节，也不要担心。大多数字段都是相当自解释的，因此您可以对它们表示的数据进行适当的猜测。当您自己分析一些查询时，您也会很快熟悉这些信息。表20-1列出了传统格式中包含的所有字段以及JSON格式中一些常见字段。

Table 20-1 The EXPLAIN fields


数字标识符，显示表或子查询属于查询的哪一部分。顶级表的id为1，第一个子查询的id为2，以此类推，对于表示union结果聚合的行，id将为NULL，表值设置为<unionM, N>(也请参阅表列)

这显示了该表将如何包含在整个语句中。已知的选择类型将在后面的“选择类型”一节中讨论。对于JSON格式,JSON文档的结构选择类型所暗示的依赖（dependent）和缓存（cacheable）等。是否相关子查询,也就是说,它取决于查询的外层部分是否子查询的结果可以缓存它必须重新评估中的每一行外查询。

表或子查询的名称。如果指定了别名，则使用该别名。这确保每个表名对于id列的给定值是惟一的。特殊情况包括联合、派生表和物化子查询，其中表名分别为<unionM、N> <derivedN>和<subqueryN>，其中N和M指查询计划早期部分的id。

查询中包含的分区可以使用该分区来确定是否按预期应用分区修剪

如何访问数据。这显示了优化器如何决定限制表中要检查的行数。这些类型将在“访问类型”一节中讨论

要用于该表的候选索引的列表。使用<auto_key0>模式的键名表示自动生成的索引可用

为表选择的索引。使用<auto_key0>模式的键名意味着使用自动生成的索引

索引使用的字节数。对于由多个列组成的索引，优化器可能只能使用列的一个子集。在这种情况下，键长度可以用来确定有多少索引对这个查询有用。如果索引中的列支持NULI值，与NOT NULL列的情况相比，将在长度上增加1个字节。

索引中使用筛选所针对的列。例如，它可以是一个常量这样的条件，<table>.<column> = ' abc '，或在连接情况下的另一个表中的列的名称。

rows_examined_per_scan：包含该表所产生的行数的估计。对于已联接到以前的表的表。它是每次连接估计能找到的行数。一种特殊情况是，引用是表上的主键或唯一键，在这种情况下，行估计值正好是1。
rows_produced_per_join：连接产生的估计行数。实际上是预期循环行数乘以rows_examined_per_scan和过滤的行数(filtered rows)的百分比
filtered：这是将包含多少已检查行的估计值。该值的单位是percent，因此如果值为100.00，将返回所有检查过的行。值为100.00是最优的，最差的值为0。注意:传统格式的值的四舍五入取决于您使用的客户机。例如，MYSQL Shell将返回100，而MYSQL命令行客户端返回100.00
cost_info：一个JSON列表，包含查询部分的耗时信息

Extra：关于优化器决策的其他信息。这可以包括有关使用的排序算法、是否使用了覆盖索引等信息。最常见的支持值将在“额外信息”一节中讨论。
message：传统输出的额外列中，在JSON输出中没有专用字段的信息。例如“Impossible WHERE”
using_filesort：是否使用文件排序
using_index：是否使用覆盖索引
using_temporary_table：子查询或排序之类的操作是否需要一个内部临时表
attached_condition：与查询部分关联的WHERE子句
used_columns：表中所需的列。这是有用的，看看你是否接近能够使用覆盖索引
有些信息最初在JSON格式中缺失，因为字段只对传统格式存在。事实并非如此:相反，信息可以通过其他方式获得，例如，Extra中的一些消息在SON格式中有它们自己的字段。其他Extra（额外信息）使用message字段。在本节后面的额外列中讨论信息时，将提到JSON输出表中未包含的一些字段。

通常，如果值为false, json格式输出中的布尔字段将被省略;一个例外是可缓存的（cacheable），因为不可缓存的子查询或联合表明与可缓存的情况相比耗时成本更高。

对于JSON输出（JSON Output），还有一些字段用于对操作的信息进行分组。这些操作的范围从访问一个表到对多个操作进行分组的复杂操作。一些常见的操作以及触发它们的例子：
table：访问一个表。这是操作的最低级别
query_block：最高级的概念，一个查询块对应一个传统格式的id。所有查询都至少有一个查询块
nested_block：一个join操作
grouping operation: 例如，由GROUP BY子句产生的操作
ordering operation: 例如，产生一个ORDER BY子句
duplicates removal: 重复项删除:例如，使用DISTINCT关键字时产生的操作
windowing: 窗口操作:使用窗口函数所产生的操作
materialized from subquery: 附加子查询:附加到查询其余部分的子查询。例如，对于IN子句中子查询的IN(SELECT..)之类的子句，就会发生这种情况
union result: 联合结果:对于使用联合来组合两个或多个查询的结果的查询。在union结果块中，有一个查询规范块，其中包含union中每个查询的定义
表20-1中的字段和复杂操作列表对于JSON格式来说并不全面，但它应该能让您对可用信息有一个很好的了解。一般来说。字段名本身携带信息，与它们出现的上下文相结合通常足以理解字段的含义。但是，一些字段的值值得更多的关注—从select类型（select types）开始

Select Types

select类型显示查询的每个部分属于哪种查询块。在此上下文中，查询的一部分可以包含几个表。例如，如果有一个简单的查询连接了一列表，但没有使用子查询等构造，那么所有表都将位于查询的相同(且仅位于)部分。查询的每个部分获得各自的id(在JSON输出中选择id)。

有几种选择类型。对于它们中的大多数，在JSON输出中没有直接字段;但是，可以从结构和一些其他字段派生select类型。表20-2显示了当前存在的select类型以及如何从JSON输出中派生类型的提示。在表中，Select类型列的值是传统输出格式中用于Select类型列的值。

simple：对于SEIECIT查询不使用派生表、子查询、联合或类似的
primary：对于使用子查询或联合的查询，主要部分是最外层的部分。
insert：与insert语句有关
delete：与delete语句有关
update：与update语句有关
replase：与replace语句有关
union：与union语句有关，第二个或以后的SELECT语句。
对于联合语句，第二个或以后的SELECT语句，它依赖于外部查询。
UNION RESUIT union result：查询中聚合union SELECT语句的结果的部分
subquery：用于子查询中的SELECI语句
DEPENDENT SUBQUERY dependent=true：关于依赖子查询，第一个SELECT语句
DERIVED：派生表——通过查询创建的表，但在其他方面表现类似于普通表
DEPENDENT DERIVED dependent=true：依赖于另一个表的派生表
MATERIALIZED materialized_from_subquery：物化子查询
UNCACHEABIE SUBQUERY cacheable=false：一个子查询，其中的结果必须求值，或外部查询中的每一行。
UNCACHEABLE UNION cacheable=false：关于Union语句。第二个或以后的SEIECT语句，它是一个uncacheable子查询的一部分。
可以将某些select类型作为信息，以便更容易理解正在查看的查询的哪个部分。例如，这包括PRIMARY和UNION。但是，一些select类型表明它是查询中开销较大的部分。这尤其适用于不受约束的类型。依赖类型还意味着优化器在决定将表添加到执行计划中的何处时灵活性较低。如果您的查询速度较慢，并且您看到了可删除的或依赖的部分，那么您可以考虑是否可以重写这些部分或将查询分割成两个部分

另一个重要的信息是如何访问表的：
Access Types（访问类型）

在讨论Visual Explain时已经遇到了表访问类型。它们显示查询是否使用索引、扫描等方式访问表。由于与每种访问类型相关的成本差异很大，因此它也是需要在EXPIAIN输出中寻找的重要值之一，以确定处理查询的哪些部分以提高性能

本小节的其余部分总结了MYSQL中的访问类型。标题是在传统格式的type列中使用的值。对于每种访问类型，都有一个使用该访问类型的示例。

system：系统访问类型用于只有一行的表。这意味着该表可以被视为常量。Visual Explain 成本，信息和颜色如下
cost：非常低
message：单行（系统常量）
color：blue
例子：SELECT * FROM (SELECT 1) my_table
系统访问类型是const访问类型的一种特殊情况
cons:例如，当主键的单个值或惟一索引上有一个筛选器时，最多匹配表的一行。Visual Explain 成本，信息和颜色如下
cost：非常低
message：单行（系统常量）
color：blue
例句：SELECT * FROM world.city WHERE ID=130
eq_ref：表是join中的右表，表上的条件位于主键或非空唯一索引上。Visual Explain 成本，信息和颜色如下
cost：低
message：唯一的键查找
color：green
例句：SE工ECT * FROM world.city STRAIGHT_JOIN world.country ON CountryCode = Code;
eq_ret访问类型是ref访问类型的一种特殊情况，在这种情况下，每次查找只能返回一行
ref：表由一个非唯一的二级索引进行筛选。Visual Explain 成本，信息和颜色如下
cost：低到中等
message：非唯一键查找
color：green
例句：SE工ECT * EROM world.city WHERE CountryCode=‘AUS’;
ref_or_null：与ref相同，但过滤后的列也可以为NULL。Visual Explain 成本，信息和颜色如下
cost：低到中等
message：键查找+获NULL值
color：green
例句：SEIECT * FROM sakila.payment WHERE renta_id=1 OR rental_id IS NULL
index_merge：优化器选择两个或多个索引的组合，以解析包含不同索引中的OR或AND之间列的筛选器。Visual Explain 成本，信息和颜色如下
cost：中等
message：合并索引
color：green
例句：SEIECT * FROM sakila.payment WHERE rental_id =1 OR customer_id =5;
中等耗时。最常见的严重性能问题之一是查询通常使用单个索引或执行全表扫描，索引统计信息变得不准确，因此优化器选择索引合并。如果使用索引合并的查询性能很差，请尝试告诉优化器忽略索引合并优化或使用的索引，并查看这是否有助于或分析表以更新索引统计信息。或者，可以将查询重写为两个查询的联合，每个查询使用过滤器的一部分。这方面的一个例子将在第24章中展示
fulltext：优化器选择一个全文索引来过滤表。Visual Explain 成本，信息和颜色如下
cost：低
message：全文索引搜索（Fulltext index search）
color：yellow（黄色）
例句：SELECT * ROM sakila.film_text WHERE MATCH(title, description) AGAINST (‘Circus' IN BOOLEAN MODE)
unique_subquery：关于IN操作符内的子查询，其中子查询返回主键或惟一索引的值。在我的MYSQL8中，这些查询通常由优化器重写，因此唯一的子查询需要禁用物化和半连接优化器开关。Visual Explain 成本，信息和颜色如下
cost：低
message：在表中查找唯一关键字的子查询
color：orange
例句：
SET optimizer switch = 'materialization=off, semijoin=off'
SELECT * FROM world city WHERE CountryCode IN( SELECT Code FROM world.country WHERE Continent ='Oceania’);
SET optimizer switch = 'materialization=on, semijoin=on'
unique_subquery访问方法是index_subquery访问方法的一种特殊情况，用于使用主索引或惟一索引的情况
index_subquery：
对于IN操作符中的子查询，在MYSQL 8中，子查询返回次要非唯一索引的值，这些查询通常由优化器重写，所以unique_subquery需要禁用物化(materialization)和半连接(semijoin)优化器开关。Visual Explain 成本，信息和颜色如下
cost：低
message：查找表中的非唯一键的子查询
color：orange
SET optimizer switch = 'materialization=off, semijoin=off’
SEIECT * EROM world.country WHERE Code IN SELECT CountryCode FROM world.city WHERE Name=‘Sydney');
SET optimizer switch = 'materialization=on, semijoin=on’
range：
当索引用于按顺序或按组查找多个值时，使用范围访问类型。IN子句等价于ID BETWEEN 1 AND 10，也用于同一列上的几个条件用OR分隔的情况。Visual Explain 成本，信息和颜色如下
cost：中
message：索引范围扫描
color：orange
例句：SE工ECT * FROM world.city where id in (130,3805);
使用范围访问的成本很大程度上取决于范围中包含多少行。在一种极端情况下，范围扫描只匹配使用主键的单个行，因此成本非常低。在另一种极端情况下，范围扫描使用辅助索引包含表的很大一部分，在这种情况下，执行全表扫描的成本可能会更低
范围访问类型与索引访问类型相关，区别在于需要部分扫描还是完整扫描
index:
优化器选择执行全索引扫描。这可以结合使用覆盖索引来选择。Visual Explain 成本，信息和颜色如下
cost：高
message：全索引扫描
color：red
例句：SELECT ID, CountryCode FROM world.city;
由于索引扫描需要使用主键进行第二次查找，除非索引是查询的覆盖索引，否则执行全表扫描的成本会非常低
ALL:
最基本的访问类型是扫描表的所有行。它也是最昂贵的访问类型，由于该类型是全大写的，Visual Explain 成本，信息和颜色如下
cost：非常高
message：全表扫描（Full table scan）
color：red
例句：SELECT * FROM world.city;
如果看到第一个表之外的表使用了全表扫描，通常是一个危险信号，表明该表上缺少条件，或者没有可以使用的索引。对于第一个表来说，所有这些都是合理的访问类型，这取决于查询需要多少表;需要的表部分越大，进行全表扫描就越合理
注意:虽然全表扫描被认为是最昂贵的访问类型，但它与主键查找一起使用是每行最便宜的访问类型。因此，如果您确实需要访问表的大部分或全部，那么全表扫描是读取行最有效的方法
至此，我们结束了对访问类型的讨论。访问类型将在本章后面的示例中再次被引用，在本书后面的示例中也将在查询优化中再次被引用，例如，在第24章中。与此同时，让我们看看额外一栏中的信息

Extra Information

传统输出格式中的Extra列是一个捕获所有不具有自己列的信息的bin。当引入JSON格式时，没有理由保留它，因为很容易引入额外的字段，也没有必要为每个输出包含所有字段。因此，JSON格式没有Extra字段，而是有一系列字段。为通用message字段保留了一些剩余的消息。

注意，Extra列中可用的信息在某些情况下依赖于存储引擎，或仅在极少数情况下使用。对于参考MYSOL参考手册的完整消息列表，本讨论将只讨论最常见的消息。https://dev.mysql.com/doc/refman/en/explain-output.html%252523explain-extra-information

一些更常见的message包括：
Using_index: 
当覆盖索引被使用。对于JSON格式，using_index字段被设置为true
Using_index_condition: 
使用索引条件:使用索引来测试是否有必要读取整行。例如，当JSON格式的索引列上有范围条件时，就会使用这种方法。index_condition字段是用过滤条件设置的
Using_where: 当WHERE子句不使用索引应用于表时。这可能表明表上的索引在J SON格式中不是最优的，attached_condition字段与筛选条件一起设置。
Using_index_for_group_by: 当使用松散索引扫描(loose index scan)来解析GROUP BY或DISTINCT时。在JSON格式。using_index_for_group_by字段被设置为true
Using join buffer (Block Nested Loop)(使用连接缓冲区(块嵌套循环)): 使用连接缓冲区(块嵌套循环):这意味着在没有索引可以使用的地方进行连接，因此使用连接缓冲区代替具有此消息的表是添加索引的候选表。JSON格式,using_join_buffer_field将块嵌套循环一件事需要注意的是,当使用一个散列连接,那么传统和JSON格式的输出仍然显示一块嵌套循环是否使用一个实际的块嵌套循环联接或散列连接,您需要使用树状产出
Using join buffer(Batched Key Access): 使用连接缓冲区(批密钥访问):这意味着连接正在使用批密钥访问(BKA)优化。要启用批密钥访问优化，您必须启用mrr(默认为on)和batch_key_access(默认为off)，并禁用mrr_cost_based(默认为on)优化器开关。优化需要连接的索引，因此与对块嵌套循环使用连接缓冲区不同，使用批密钥访问算法并不意味着对表的昂贵访问。对于JSON格式，using_join_buffer_field设置为批密钥访问(Batched_Key_Acess)
Using_MRR: 使用多范围读(MRR)优化。这有时用于减少范围条件在需要整行的二级索引上的随机I/O数量。对于JSON格式，using_MRR字段设置为true，优化由mrr和mrr_cost 优化器开关(两者都是默认启用的)控制
Using filesort: MYSQL使用一个额外的传递来确定如何以正确的顺序检索行。例如，这是通过二级索引进行排序;索引不是覆盖指数。对于JSON格式，using filesort_field被设置为true
Using temporary: 使用临时:内部临时表用于存储子查询的结果,进行排序或分组,排序并分组；内部临时表的使用有时可以避免通过添加一个索引或重写ng JSON格式的查询,using_temporary_table字段被设置为true
sort_union(…)，Using union(…)，Using intersect(…):这三个消息与索引合并一起使用，说明如何执行索引合并。对于任何一条消息，关于索引合并中涉及的索引的信息都包含在括号中。对于JSON格式，key字段指定使用的方法和索引
Recursive: 该表是递归公共表表达式(CTE)的一部分。对于JSON格式，递归字段被设置为true
Range checked for each record(index map: 0x1): 为每个记录检查的范围（索引映射：0x1）：当您有一个JOIN时，第二个表的索引列上存在一个条件，该条件取决于第一个表中的列的值，例如，带有一个 t2.val2上的索引：SELECT * FROM t1 INNER JOIN t2 WHERE t2.val2 < t1.val1; 这就是触发“性能模式”语句事件表中的NO_GOOD_INDEX_USED计数器递增的原因。 索引图是一个位掩码(bitmask)，指示哪些索引是范围检查的候选。 索引号从1开始，如SHOW INDEXES所示。 当写出位掩码(bitmask)时，设置了位的索引号是JSON格式的候选项。 range_checked_for_each_record字段设置为索引映射（index map）
Impossible WHERE（不可能的where条件）：当存在一个不可能为真的过滤器（filter）时，例如，' WHERE 1=0 '。如果筛选器中的值超出了数据类型支持的范围，例如，tinyint数据类型的' WHERE ID =300 '也适用于此。对于JSON格式，消息被添加到message字段
Impossible WHERE noticed after reading const tables：与“Impossible WHERE”相同，除了它应用于使用system或const访问方法解析表之后。例如，' SELECT * FROM (SELECT 1 AS ID)a INNER JOIN citye USING(ID) WHERE a.id = 130 '对于JSON格式，消息被添加到message字段
Impossible HAVING：不可能的拥有:与“Impossible WHERE”相同，除非它应用于HAVING从句。对于JSON格式，消息被添加到message字段
Using index for skip scan：当优化器选择使用类似于松散索引扫描的多个范围扫描时。例如，它可以用于索引的第一列不用于筛选条件的覆盖索引。这个方法在MYSQL 8.0.13和更高版本中可用。对于JSON格式，using_index_for_skip_scan字段设置为true
Select tables optimized away：这条消息意味着MYSQL能够从查询中删除表，因为只会产生一行，并且该行可以从一组确定的行生成。当表中只需要索引的最小值和/或最大值时，通常会发生这种情况。对于JSON格式，消息被添加到message字段
No tables used: 不使用表:对于不涉及任何表的子查询，例如，SELECT 1 dual; 对于JSON格式，消息被添加到message字段
no matching row in const table：对于可以使用system或const访问类型但没有匹配条件的行的表。获取JSON格式。将消息添加到message字段
提示在编写sql语句时，您需要使用树形格式的输出来查看不使用索引的JOIN是否使用散列连接算法（hash join algorithm）。

关于EXPLAIN语句输出的意义的讨论到此结束。剩下的就是开始使用它来检查查询计划（query plans）。

EXPLAIN Examples

为了结束对查询计划的讨论，有必要通过一些示例来更好地了解如何将所有查询计划组合在一起。这里的例子只是作为介绍。更多的例子将出现在本书的后面部分，特别是第24章

Single Table, Table Scan
作为第一个示例，考虑对world数据库中的city表进行查询，条件是对非索引列名。由于没有可以使用的索引，它将需要一个全表扫描来评估查询。匹配这些需求的查询示例如下

SELECT * FROM world.city WHERE Name = ‘London’;

输出的访问类型设置为ALL，这也是我们所期望的，因为在带索引的列上没有条件。据估计，将检查4188行(实际数量是4079)，对于每一行，将应用WHERE子句的条件。预计检查的行中有10%将匹配WHERE子句(注意，根据所使用的客户机，过滤列的输出可能是10或10.00)。回想一下在第17章的优化器讨论中，优化器使用默认值来估计各种条件的过滤效果，因此您不能直接使用WHERE值来估计索引是否有用
对应的Visual Explain图见图20-8

一个红色的全表扫描框显示全表扫描（Full Table SCAN）。可以看出，成本估计为425.05
这个查询只返回两行(表中有英国的伦敦和加拿大的安大略)。如果一个国家的所有城市都被请求，会发生什么?
Single Table, Index Access

第二个示例与第一个示例类似，只是过滤器条件更改为使用具有第二个非惟一索引的CountryCode列。这将降低访问匹配行的成本。对于本例，将检索所有德国城市。
SELECT * FROM world.city WHERE Countrycode ='DEU';

这一次，possible_keys列显示可以使用CountryCode索引进行查询，而key列显示使用了的索引。访问类型是ref，表示使用非唯一索引进行表访问。估计将访问93行。这是精确的，因为优化器会询问INNODB有多少行匹配。过滤后的列显示索引对表进行了完美的过滤。相应的视觉解释图如图20-9所示。（page 645）

尽管返回的行数是第一个示例的45倍多，但估计成本仅为28.05，不到全表扫描成本的十分之一
如果只使用ID和CountryCode列会发生什么?
Two Tables and a Covering Index

如果有一个索引包含表中所需的所有列。然后它被称为覆盖索引（covering index）。MYSQL将使用这个来避免检索整个行，因为city表的CountryCode索引是一个非唯一索引（Normal），它还包括ID列，因为它是主键。为了使查询更真实一些，查询还将包括country表，并根据continent过滤所包含的country。这样的查询的一个例子是
例句：SELECT ci.ID FROM world.country co INNER JOIN world.city ci ON ci.CountryCode=co.Code WHERE co.Continent = 'Asia’;

查询计划表明，优化器选择从co(country)表上的全表扫描开始，并对ci(city)表上的连接使用CountryCode索引。这里的特殊之处在于，Extra列包括使用索引。因此，没有必要读取city表的整行。还请注意，键长度(key_len)为3(字节)，这是CountryCode列的宽度。相应的可视化解释图如图20-10所示

key_len字段不包括索引的主键部分，即使它被使用。然而，看看多列索引被使用了多少是有用的
Multicolumn Index

countrylanguage表有一个主键(primary key)，包括国家代码(CountryCode)和语言(Language)列。假设你想找到一个国家的所有语言;在这种情况下，您需要过滤(WHERE)国家代码(CountryCode)，而不是语言(Language)。(CountryCode)索引仍然可以用于执行过滤(WHERE)，并且您可以使用EXPIAIN输出的键len字段来查看索引使用了多少。一个查询可以用来查找在中国使用的所有语言是
SE工ECT * FROM world.countrylanguage WHERE CountryCode= 'CHN’;

主键(primary)的总宽度是CountryLanguage列的3个字节，来自Ianguage列的30个字节。因为key_len列显示,只有3个字节的关键是使用,它可以得出只有CountryLanguage索引的一部分用于过滤(使用的部分索引总是最左边的部分)需要悬停在Visual Explain表中问题得到如图20-11扩展的信息

从图中我们可看出,在Key/Index：PRIMARY下的Used Key Parts：CountryCode，这直接表明只有CountryCode列的索引被使用。
作为最后一个示例，让我们返回到在使用EXPLAIN格式时用作示例的查询

Two Tables with Subquery and Sorting

本章前面广泛使用的示例查询将用于完成关于EXPIAIN的讨论。查询混合使用了各种特性，因此它触发了前面讨论过的信息的几个部分。它也是一个带有多个查询块的查询示例。清单20-9中重复了传统EXPIAIN格式的输出
EXPLAIN
       SELECT ci.ID, ci.Name, ci.District,
              co.Name AS Country, ci.Population
         FROM city ci
              INNER JOIN
                (SELECT Code, Name
                   FROM country
                  WHERE continent = 'Europe'
                  ORDER BY SurfaceArea
                  LIMIT 10
                ) co ON co.Code = ci.CountryCode
        ORDER BY ci.Population DESC
        LIMIT 5;


查询的可视化解释图在图20-12中重复。在继续阅读输出分析之前，建议您自己先研究一下

查询计划(query plan)从子查询(subquery)开始，子查询使用国家(country)表按区域查找十个最小的国家。子查询(subquery)是给定表标签 <derived2>，因此您需要查找使用'where id =2'的行(对于其他查询可能是几行)，在本例中这是第3行。第3行将select类型设置为DERIVED，因此它是一个派生表(derived table);这是一个通过查询创建的表，但在其他方面表现得像一个普通表。派生表是使用全表扫描(type = ALL)生成的，每一行都有一个WHERE子句，后面跟着一个文件排序(filesort)。生成的派生表被物化(materialized 或翻译成具体化、实例化)(从可视解释可见)，并称为co

一旦派生表已经构造好。它被用作与ci(city)表JOIN的第一个表。您可以从<derived2>的第1行和第2行ci行排序中看到这一点。对于派生表中的每一行，估计ci表中将使用CountryCode索引检查18行。国家代码索引（CountryCode）是Normal（非唯一索引）,可以看到从Visual Explain表箱的标签,和类型列参考价值。据估计,join将返回180行,来自派生表中的10行乘以来自ci表中每个索引查询的18行

最后，使用内部临时表（temp table）和文件排序(file sort)结果，估计查询的总成本为247.32

到目前为止，讨论的焦点一直是查询计划最终是什么。如果您想知道优化器是如何到达那里的，您将需要检查优化器跟踪。
