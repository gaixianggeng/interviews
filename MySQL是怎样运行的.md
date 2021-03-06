> 在github上正好看到一篇读书笔记，省的自己一个一个字敲了 
https://github.com/Shellbye/Shellbye.github.io/issues/84
# MySQL
mysql采用客户端/服务器架构，用户通过客户端发送增删改查请求，服务器程序收到请求后处理，并把处理结果返回客户端
bin目录下存放可执行文件
客户端进程和服务端进程通信方式：TCP/IP 命名管道或共享内存，unix域套接字
查询请求：
* 连接管理：建立连接 认证信息
* 解析与优化：查询缓存，语法解析、查询优化等
* 存储引擎：读取or写入底层表数据

mysql支持多种存储引擎，InnoDb是服务器默认的存储引擎。

2. 装作自己是个小白——初识MySQL
    2.1 MySQL的客户端／服务器架构
        每个进程都有一个名称，这个名称是编写程序的人自己定义的，比如我们启动的MySQL服务器进程的默认名称为mysqld， 
        而我们常用的MySQL客户端进程的默认名称为mysql
    2.2 MySQL的安装
    2.3 启动MySQL服务器程序
        UNIX里启动服务器程序
            mysqld
            mysqld_safe
            mysql.server
    2.4 启动MySQL客户端程序
        mysql -h主机名  -u用户名 -p密码
    2.5 客户端与服务器连接的过程
        TCP/IP
            3306
        命名管道和共享内存
            Windows
        Unix域套接字文件
    2.6 服务器处理客户端请求
        连接管理
            MySQL服务器会为每一个连接进来的客户端分配一个线程
        解析与优化
            查询缓存
                从MySQL 5.7.20开始，不推荐使用查询缓存，并在MySQL 8.0中删除
            语法解析
            查询优化
        存储引擎
            常用存储引擎
                InnoDB
                    具备外键支持功能的事务存储引擎
                MyISAM
                    主要的非事务处理存储引擎
            关于存储引擎的一些操作
                查看当前服务器程序支持的存储引擎
                    SHOW ENGINES;
                设置表的存储引擎
                创建表时指定存储引擎

3. MySQL 的调控按钮————启动选项和系统变量
    3.1 在命令行上使用选项
        mysqld --skip-networking
            禁止各客户端使用 TCP/IP 网络进行通信
    3.2 配置文件中使用选项
        配置文件的路径
            Windows操作系统的配置文件
            类Unix操作系统中的配置文件
                /etc/my.cnf
                /etc/mysql/my.cnf
                ~/.my.cnf
                ~/.mylogin.cnf
        配置文件的内容
        配置文件的优先级
            在多个配置文件中设置了相同的启动选项，那以最后一个配置文件中的为准
            同一个配置文件中多个组的优先级
                以最后一个出现的组中的启动选项为准
    3.3 系统变量
        系统变量简介
        查看系统变量
            SHOW VARIABLES [LIKE 匹配的模式];
        设置系统变量
            通过启动选项设置
            服务器程序运行过程中设置
                对于大部分系统变量来说，它们的值可以在服务器程序运行过程中进行动态修改而无需停止并重启服务器
                设置不同作用范围的系统变量
                    GLOBAL：全局变量，影响服务器的整体操作
                    SESSION：会话变量，影响某个客户端连接的操作。（注：SESSION有个别名叫LOCAL）
                查看不同作用范围的系统变量
    3.4 启动选项和系统变量的区别
    3.5 状态变量
        SHOW [GLOBAL|SESSION] STATUS [LIKE 匹配的模式];

4. 乱码的前世今生————字符集和比较规则
    4.1 字符集和比较规则简介
        字符集简介
        比较规则简介
        一些重要的字符集
    4.2 MySQL中支持的字符集和排序规则
        MySQL中的utf8和utf8mb4
            utf8mb3：阉割过的utf8字符集，只使用1～3个字节表示字符
            utf8mb4：正宗的utf8字符集，使用1～4个字节表示字符
        字符集的查看
            SHOW (CHARACTER SET|CHARSET) [LIKE 匹配的模式];
        比较规则的查看
            SHOW COLLATION [LIKE 匹配的模式];
            比较规则的命名
                比较规则名称以与其关联的字符集的名称开头
                后边紧跟着该比较规则主要作用于哪种语言
                名称后缀意味着该比较规则是否区分语言中的重音、大小写
                    _ai	    accent insensitive	不区分重音
                    _as	    accent sensitive	区分重音
                    _ci	    case insensitive	不区分大小写
                    _cs	    case sensitive	    区分大小写
                    _bin	binary	            以二进制方式比较
    4.3 字符集和比较规则的应用
        各级别的字符集和比较规则
            服务器级别
            数据库级别
            表级别
            列级别
        仅修改字符集或仅修改比较规则
            只修改字符集，则比较规则将变为修改后的字符集默认的比较规则
            只修改比较规则，则字符集将变为修改后的比较规则对应的字符集
    4.4 客户端和服务器通信中的字符集
        编码和解码使用的字符集不一致的后果
        字符集转换的概念
        MySQL中字符集的转换
            character_set_client
            character_set_connection
            character_set_results
    4.5 比较规则的应用

5. 从一条记录说起————InnoDB 记录存储结构
    5.1 InnoDB页简介
        InnoDB采取的方式是：将数据划分为若干个页，以页作为磁盘和内存之间交互的基本单位，InnoDB中页的大小一般为 16 KB
    
    5.2 InnoDB行格式
        设计InnoDB存储引擎的大叔们到现在为止设计了4种不同类型的行格式，分别是
            Compact
                变长字段长度列表 | NULL值列表 | 记录头信息 | 各个列的值（+隐藏列 DB_ROW_ID, DB_TRX_ID, DB_ROLL_PTR）
                变长字段长度列表
                    1. 真正的数据内容
                    2. 占用的字节数
                    在Compact行格式中，把所有变长字段的真实数据占用的字节长度都存放在记录的开头部位，
                    从而形成一个变长字段长度列表，各变长字段数据占用的字节数按照列的顺序逆序存放
            Redundant
                MySQL5.0之前用的一种行格式
                字段长度偏移列表 | 记录头信息 | 各个列的值
            Dynamic
                MySQL 5.7 的默认格式
            Compressed
    5.3 总结
        1. 页是MySQL中磁盘和内存交互的基本单位，也是MySQL是管理存储空间的基本单位
        2. 一个页一般是16KB，当记录中的数据太多，当前页放不下的时候，会把多余的数据存储到其他页中，这种现象称为行溢出

    MySQL对一条记录占用的最大存储空间是有限制的，除了BLOB或者TEXT类型的列之外，
    其他所有的列（不包括隐藏列和记录头信息）占用的字节长度加起来不能超过65535个字节
    MySQL中规定一个页中至少存放两行记录

6. 盛放记录的大盒子————InnoDB 数据页结构
    6.1 不同类型的页简介
    6.2 数据页结构的快速浏览
        名称	            中文名	            占用空间大小	简单描述
        File Header	        文件头部	        38字节	    页的一些通用信息
        Page Header	        页面头部	        56字节	    数据页专有的一些信息
        Infimum + Supremum	最小记录和最大记录	  26字节	  两个虚拟的行记录
        User Records	    用户记录	        不确定	     实际存储的行记录内容
        Free Space	        空闲空间	        不确定	     页中尚未使用的空间
        Page Directory	    页面目录	        不确定	     页中的某些记录的相对位置
        File Trailer	    文件尾部	        8字节	     校验页是否完整
    6.3 记录在页中的存储
            在页的7个组成部分中，我们自己存储的记录会按照我们指定的行格式存储到User Records部分
        记录头信息的秘密
            delete_mask
                这个属性标记着当前记录是否被删除，占用1个二进制位
            min_rec_mask
                B+树的每层非叶子节点中的最小记录都会添加该标记
            n_owned
                表示该记录拥有多少条记录，也就是该组内共有几条记录
            heap_no
                这个属性表示当前记录在本页中的位置
            record_type
                这个属性表示当前记录的类型，一共有4种类型的记录，
                0表示普通记录，1表示B+树非叶节点记录，2表示最小记录，3表示最大记录
            next_record
                表示从当前记录的真实数据到下一条记录的真实数据的地址偏移量
            不论我们怎么对页中的记录做增删改操作，InnoDB始终会维护一条记录的单链表，
            链表中的各个节点是按照主键值由小到大的顺序连接起来的
    6.4 Page Directory（页目录）
        对于最小记录所在的分组只能有 1 条记录，最大记录所在的分组拥有的记录条数只能在 1~8 条之间，
        剩下的分组中记录的条数范围只能在是 4~8 条之间

        在一个数据页中查找指定主键值的记录的过程分为两步：
            1. 通过二分法确定该记录所在的槽，并找到该槽所在分组中主键值最小的那条记录。
            2. 通过记录的next_record属性遍历该槽所在的组中的各个记录。
    6.5 Page Header（页面头部）
    6.6 File Header（文件头部）
    6.7 File Trailer

7. 快速查询的秘籍————B+ 树索引
    7.1 没有索引的查找
        在一个页中的查找
            以主键为搜索条件
                在页目录中使用二分法快速定位到对应的槽
            以其他列作为搜索条件
                从最小记录开始依次遍历单链表中的每条记录，然后对比每条记录是不是符合搜索条件
        在很多页中查找
            1. 定位到记录所在的页
            2. 从所在的页内中查找相应的记录
            在没有索引的情况下，不论是根据主键列或者其他列的值进行查找，
            由于我们并不能快速的定位到记录所在的页，所以只能从第一个页沿着双向链表一直往下找
    7.2 索引
        一个简单的索引方案
            下一个数据页中用户记录的主键值必须大于上一个页中用户记录的主键值
                因为各个页中的记录并没有规律，我们并不知道我们的搜索条件匹配哪些页中的记录，
                    所以 不得不 依次遍历所有的数据页
                新分配的数据页编号可能并不是连续的，也就是说我们使用的这些页在存储空间里可能并不挨着
                下一个数据页中用户记录的主键值必须大于上一个页中用户记录的主键值
                在对页中的记录进行增删改操作的过程中，我们必须通过一些诸如记录移动的操作来始终保证这个状态一直成立：
                    下一个数据页中用户记录的主键值必须大于上一个页中用户记录的主键值。
                    这个过程我们也可以称为页分裂
            给所有的页建立一个目录项
        InnoDB中的索引方案
            复用了之前存储用户记录的数据页来存储目录项，为了和用户记录做一下区分，
            我们把这些用来表示目录项的记录称为目录项记录

            record_type
                0：普通的用户记录
                1：目录项记录
                2：最小记录
                3：最大记录
        实际用户记录其实都存放在B+树的最底层的节点上，这些节点也被称为叶子节点或叶节点
    7.3 聚簇索引
        具有以下两种特性的B+树称为聚簇索引
        1. ，使用记录主键值的大小进行记录和页的排序这包括三个方面的含义：
            页内的记录是按照主键的大小顺序排成一个单向链表。
            各个存放用户记录的页也是根据页中用户记录的主键大小顺序排成一个双向链表。
            存放目录项记录的页分为不同的层次，在同一层次中的页也是根据页中目录项记录的主键大小顺序排成一个双向链表。
        2. B+树的叶子节点存储的是完整的用户记录。
            所谓完整的用户记录，就是指这个记录中存储了所有列的值（包括隐藏列）
    7.4 二级索引
        如果我们想根据c2列的值查找到完整的用户记录的话，仍然需要到聚簇索引中再查一遍，
        这个过程也被称为回表。也就是根据c2列的值查询一条完整的用户记录需要使用到2棵B+树

        按照非主键列建立的B+树需要一次回表操作才可以定位到完整的用户记录，
        所以这种B+树也被称为二级索引（英文名secondary index），或者辅助索引。
    7.5 联合索引
        先把各个记录和页按照c2列进行排序
        在记录的c2列相同的情况下，采用c3列进行排序
    7.6 InnoDB的B+树索引的注意事项
        根页面万年不动窝
            一个B+树索引的根节点自诞生之日起，便不会再移动
        内节点中目录项记录的唯一性
            需要保证在B+树的同一层内节点的目录项记录除页号这个字段以外是唯一的，
            所以添加了主键
        一个页面最少存储2条记录
        MyISAM中的索引方案简单介绍
            InnoDB中索引即数据，也就是聚簇索引的那棵B+树的叶子节点中已经把所有完整的用户记录都包含了，
            而MyISAM的索引方案虽然也使用树形结构，但是却将索引和数据分开存储
        MySQL中创建和删除索引的语句

8. 好东西也得先学会怎么用————B+树索引的使用
    8.1 索引的代价
        空间上的代价
            每建立一个索引都要为它建立一棵B+树，
            每一棵B+树的每一个节点都是一个数据页，
            一个页默认会占用16KB的存储空间
        时间上的代价
            增、删、改操作可能会对节点和记录的排序造成破坏，
            所以存储引擎需要额外的时间进行一些记录移位，
            页面分裂、页面回收啥的操作来维护好节点和记录的排序
    8.2 B+树索引适用的条件
        全值匹配
        匹配左边的列
        匹配列前缀
        匹配范围值
            如果对多个列同时进行范围查找的话，只有对索引最左边的那个列进行范围查找的时候才能用到B+树索引
        精确匹配某一列并范围匹配另外一列
        用于排序
            使用联合索引进行排序注意事项
                对于联合索引有个问题需要注意，ORDER BY的子句后边的列的顺序也必须按照索引列的顺序给出
            不可以使用索引进行排序的几种情况
                ASC、DESC混用
                WHERE子句中出现非排序使用到的索引列
                排序列包含非同一个索引的列
                排序列使用了复杂的表达式
        用于分组
    8.3 回表的代价
        覆盖索引
            为了彻底告别回表操作带来的性能损耗，我们建议：最好在查询列表里只包含索引列
            只需要用到索引的查询方式称为索引覆盖
    8.4 如何挑选索引
        只为用于搜索、排序或分组的列创建索引
        考虑列的基数
            最好为那些列的基数大（不重复元素多）的列建立索引，为基数太小列的建立索引效果可能不好
        索引列的类型尽量小
            数据类型越小，在查询时进行的比较操作越快
            数据类型越小，索引占用的存储空间就越少，在一个数据页内就可以放下更多的记录，
                从而减少磁盘I/O带来的性能损耗，也就意味着可以把更多的数据页缓存在内存中，从而加快读写效率
        索引字符串值的前缀
            只索引字符串值的前缀的策略是我们非常鼓励的，尤其是在字符串类型能存储的字符比较多的时候
            索引列前缀对排序的影响
        让索引列在比较表达式中单独出现
            如果索引列在比较表达式中不是以单独列的形式出现，
            而是以某个表达式，或者函数调用形式出现的话，是用不到索引的
        主键插入顺序
        冗余和重复索引

9. 数据的家————MySQL 的数据目录
    9.1 数据库和文件系统的关系
    9.2 MySQL数据目录
        数据目录和安装目录的区别
        如何确定MySQL中的数据目录
            mysql> SHOW VARIABLES LIKE 'datadir';
    9.3 数据目录的结构
        数据库在文件系统中的表示
        表在文件系统中的表示
        InnoDB是如何存储表数据的
            系统表空间（system tablespace）
            独立表空间(file-per-table tablespace)
        MyISAM是如何存储表数据的
        视图在文件系统中的表示
    9.4 文件系统对数据库的影响
        数据库名称和表名称不得超过文件系统所允许的最大长度
        特殊字符的问题
        文件长度受文件系统最大长度限制
    9.5 MySQL系统数据库简介
        mysql
            这个数据库贼核心，它存储了MySQL的用户账户和权限信息，一些存储过程、事件的定义信息，
            一些运行过程中产生的日志信息，一些帮助信息以及时区信息等。
        information_schema
            这个数据库保存着MySQL服务器维护的所有其他数据库的信息，
            比如有哪些表、哪些视图、哪些触发器、哪些列、哪些索引吧啦吧啦。
            这些信息并不是真实的用户数据，而是一些描述性信息，有时候也称之为元数据。
        performance_schema
            这个数据库里主要保存MySQL服务器运行过程中的一些状态信息，
            算是对MySQL服务器的一个性能监控。包括统计最近执行了哪些语句，
            在执行过程的每个阶段都花费了多长时间，内存的使用情况等等信息。
        sys
            这个数据库主要是通过视图的形式把information_schema和performance_schema结合起来，
            让程序员可以更方便的了解MySQL服务器的一些性能信息

10. 存放页面的大池子————InnoDB 的表空间
    10.1 独立表空间结构
        区（extent）的概念
            对于16KB的页来说，连续的64个页就是一个区，1MB
            每256个区被划分成一组
        段（segment）的概念
            叶子节点有自己独有的区，非叶子节点也有自己独有的区。
            存放叶子节点的区的集合就算是一个段（segment），
            存放非叶子节点的区的集合也算是一个段。
            也就是说一个索引会生成2个段，一个叶子节点段，一个非叶子节点段
        区的分类
            空闲的区（FREE）：现在还没有用到这个区中的任何页面
            有剩余空间的碎片区（FREE_FRAG）：表示碎片区中还有可用的页面
            没有剩余空间的碎片区（FULL_FRAG）：表示碎片区中的所有页面都被使用，没有空闲页面
            附属于某个段的区（FSEG）。每一个索引都可以分为叶子节点段和非叶子节点段，
                除此之外InnoDB还会另外定义一些特殊作用的段，在这些段中的数据量很大时将使用区来作为基本的分配单位
            
            XDES Entry的结构（全称就是Extent Descriptor Entry），每一个区都对应着一个XDES Entry结构，这个结构记录了对应的区的一些属性
                Segment ID
                List Node
                State
                Page State Bitmap
            XDES Entry链表
                提高向表插入数据的效率又不至于数据量少的表浪费空间
                向某个段中插入数据的过程
                    1. 首先会查看表空间中是否有状态为FREE_FRAG的区，
                        否则到表空间下申请一个状态为FREE的区，
                        也就是空闲的区，把该区的状态变为FREE_FRAG，
                        
                        FREE 链表
                        FREE_FRAG 链表
                        FULL_FRAG 链表
                    2. 当段中数据已经占满了32个零散的页后，就直接申请完整的区来插入数据了
                        FREE 链表
                        NOT_FULL 链表
                        FULL 链表
            链表基节点
            链表小结
        段的结构
            段其实不对应表空间中某一个连续的物理区域，而是一个逻辑上的概念，由若干个零散的页面以及一些完整的区组成
            定义了一个INODE Entry结构来记录一下段中的属性
                Segment ID
                NOT_FULL_N_USED
                3个List Base Node
                Magic Number
                Fragment Array Entry
        各类型页面详细情况
            FSP_HDR类型 
                存储了表空间的一些整体属性以及第一个组内256个区的对应的XDES Entry结构
            File Space Header部分
                存储表空间的一些整体属性的
        XDES Entry部分
            XDES类型
            IBUF_BITMAP类型
            INODE类型
        Segment Header 结构的运用
        真实表空间对应的文件大小
    10.2 系统表空间
        系统表空间的整体结构
        InnoDB数据字典
        SYS_TABLES表
        SYS_COLUMNS表
        SYS_INDEXES表
        SYS_FIELDS表
        Data Dictionary Header页面
        information_schema系统数据库

11. 条条大路同罗马——单表访问方法
    11.1 访问方法（access method）的概念
        MySQL的查询的执行方式大致分为下边两种
            使用全表扫描进行查询
            使用索引进行查询
                针对主键或唯一二级索引的等值查询
                针对普通二级索引的等值查询
                针对索引列的范围查询
                直接扫描整个索引
    11.2 访问方法（access method）的概念
            const
                通过主键或者【唯一】二级索引列与常数的等值比较来定位一条记录
                对于唯一二级索引来说，查询该列为NULL值的情况不可以使用const访问方法来执行
            ref
                搜索条件为二级索引列与常数等值比较，采用二级索引来执行查询的访问方法
                ref访问方法比const差了那么一丢丢（const结果是唯一的）
                如果匹配的二级索引记录太多那么回表的成本就太大了
                1. 二级索引列值为NULL的情况
                    我们采用key IS NULL这种形式的搜索条件最多只能使用ref的访问方法，而不是const的访问方法
                2. 对于某个包含多个索引列的二级索引来说，只要是最左边的连续索引列是与常数的等值比较就可能采用ref的访问方法
            ref_or_null
                找出某个二级索引列的值等于某个常数的记录，还想把该列的值为NULL的记录也找出来
            range
            index
                【遍历】二级索引记录的执行方式
                不需要回表，但是要遍历整个索引
            all
                全表扫描
    11.3 注意事项
            二级索引+回表
                因为二级索引的节点中的记录只包含索引列和主键，
                所以在步骤1中使用idx_key1索引进行查询时只会用到与key1列有关的搜索条件，
                其余条件，比如key2 > 1000这个条件在步骤1中是用不到的，
                只有在步骤2完成回表操作后才能继续针对完整的用户记录中继续过滤

                也就是说一个使用到索引的搜索条件和没有使用该索引的搜索条件使用OR连接起来后是无法使用该索引的
            明确range访问方法使用的范围区间

            索引合并
                Intersection合并
                    MySQL在某些特定的情况下才可能会使用到Intersection索引合并
                        情况一：二级索引列是等值匹配的情况，对于联合索引来说，
                                在联合索引中的每个列都必须等值匹配，不能出现只匹配部分列的情况
                        情况二：主键列可以是范围匹配
                Union合并
                    情况一：二级索引列是等值匹配的情况，对于联合索引来说，
                        在联合索引中的每个列都必须等值匹配，不能出现只出现匹配部分列的情况
                    情况二：主键列可以是范围匹配
                    情况三：使用Intersection索引合并的搜索条件
                Sort-Union合并
                    先根据key1 < 'a'条件从idx_key1二级索引中获取记录，并按照记录的主键值进行排序
                    再根据key3 > 'z'条件从idx_key3二级索引中获取记录，并按照记录的主键值进行排序
                索引合并注意事项
                    联合索引替代Intersection索引合并

12. 两个表的亲密接触————连接的原理
    12.1 连接简介
        连接的本质
            笛卡尔积
        连接过程简介
            过滤条件
                涉及单表的条件
                涉及两表的条件
            1. 首先确定第一个需要查询的表，这个表称之为驱动表
            2. 针对上一步骤中从驱动表产生的结果集中的每一条记录，
                分别需要到t2表中查找匹配的记录，所谓匹配的记录，指的是符合过滤条件的记录
        内连接和外连接
            需求： 驱动表中的记录即使在被驱动表中没有匹配的记录，也仍然需要加入到结果集
                对于内连接的两个表，驱动表中的记录在被驱动表中找不到匹配的记录，该记录不会加入到最后的结果集
                对于外连接的两个表，驱动表中的记录即使在被驱动表中没有匹配的记录，也仍然需要加入到结果集
            WHERE子句中的过滤条件
                WHERE子句中的过滤条件就是我们平时见的那种，不论是内连接还是外连接，
                凡是不符合WHERE子句中的过滤条件的记录都不会被加入最后的结果集
            ON子句中的过滤条件
                对于外连接的驱动表的记录来说，如果无法在被驱动表中找到匹配ON子句中的过滤条件的记录，
                那么该记录仍然会被加入到结果集中，对应的被驱动表记录的各个字段使用NULL值填充
                ON子句是专门为外连接驱动表中的记录在被驱动表找不到匹配记录时应不应该把该记录加入结果集这个场景下提出的
            左（外）连接的语法
                SELECT * FROM t1 LEFT [OUTER] JOIN t2 ON 连接条件 [WHERE 普通过滤条件];
            右（外）连接的语法
                SELECT * FROM t1 RIGHT [OUTER] JOIN t2 ON 连接条件 [WHERE 普通过滤条件];
            内连接的语法
                内连接和外连接的根本区别就是在驱动表中的记录不符合ON子句中的连接条件时不会把该记录加入到最后的结果集
            小结
                对于内连接来说，驱动表和被驱动表是可以互换的，并不会影响最后的查询结果。
                但是对于外连接来说，由于驱动表中的记录即使在被驱动表中找不到符合ON子句条件的记录时也要将其加入到结果集，
                所以此时驱动表和被驱动表的关系就很重要了，也就是说左外连接和右外连接的驱动表和被驱动表不能轻易互换
    12.2 连接的原理
        嵌套循环连接（Nested-Loop Join）
            驱动表只访问一次，但被驱动表却可能被多次访问，
            访问次数取决于对驱动表执行单表查询后的结果集中的记录条数的连接执行方式称之为嵌套循环连接（Nested-Loop Join）
        使用索引加快连接速度
        基于块的嵌套循环连接（Block Nested-Loop Join）
            join buffer

13. 谁最便宜就选谁————MySQL 基于成本的优化
    13.1 什么是成本
        I/O成本
        CPU成本
    13.2 单表查询的成本
        基于成本的优化步骤
            1. 根据搜索条件，找出所有可能使用的索引
                一个查询中可能使用到的索引称之为possible keys
            2. 计算全表扫描的代价
                Rows
                    本选项表示表中的记录条数
                Data_length
                    本选项表示表占用的存储空间字节数
            3. 计算使用不同索引执行查询的代价
            4. 对比各种执行方案的代价，找出成本最低的那一个
        基于索引统计数据的成本计算
    13.3 连接查询的成本
        Condition filtering介绍
        两表连接的成本分析
        多表连接的成本分析
    13.4 调节成本常数
        读取一个页面花费的成本默认是1.0
        检测一条记录是否符合搜索条件的成本默认是0.2
        mysql.server_cost表
        mysql.engine_cost表

14. 兵马未动，粮草先行————InnoDB 统计数据是如何收集的
    14.1 两种不同的统计数据存储方式
        永久性的统计数据
        非永久性的统计数据
    14.2 基于磁盘的永久性统计数据
        innodb_table_stats存储了关于表的统计数据，每一条记录对应着一个表的统计数据。
        innodb_index_stats存储了关于索引的统计数据，每一条记录对应着一个索引的一个统计项的统计数据

        n_rows统计项的收集
            按照一定算法（并不是纯粹随机的）选取几个叶子节点页面，计算每个页面中主键值记录数量，
            然后计算平均一个页面中主键值的记录数量乘以全部叶子节点的数量就算是该表的n_rows值
        定期更新统计数据
    14.3 基于内存的非永久性统计数据
    14.4 innodb_stats_method的使用
        nulls_equal
            认为所有NULL值都是相等的。这个值也是innodb_stats_method的默认值
        nulls_unequal
            认为所有NULL值都是不相等的
        nulls_ignored
            直接把NULL值忽略掉

15. 不好看就要多整容————MySQL 基于规则的优化
    15.1 条件化简
        移除不必要的括号
        常量传递（constant_propagation）
        等值传递（equality_propagation）
        移除没用的条件（trivial_condition_removal）
        表达式计算
        HAVING子句和WHERE子句的合并
        常量表检测
        外连接消除
            在外连接查询中，指定的WHERE子句中包含被驱动表中的列不为NULL值的条件称之为空值拒绝（英文名：reject-NULL）。
            在被驱动表的WHERE子句符合空值拒绝的条件后，外连接和内连接可以相互转换。
            这种转换带来的好处就是查询优化器可以通过评估表的不同连接顺序的成本，选出成本最低的那种连接顺序来执行查询
    15.2 子查询优化
        子查询语法
            子查询可以在一个外层查询的各种位置出现，比如
                SELECT子句中
                FROM子句中
                WHERE或ON子句中
            按返回的结果集区分子查询
                标量子查询
                行子查询
                列子查询
                表子查询
            按与外层查询关系来区分子查询
                不相关子查询
                相关子查询
            子查询在布尔表达式中的使用
                使用=、>、<、>=、<=、<>、!=、<=>作为布尔表达式的操作符
                [NOT] IN/ANY/SOME/ALL子查询
                EXISTS子查询
            子查询语法注意事项
                子查询必须用小括号扩起来
                在SELECT子句中的子查询必须是标量子查询
                在想要得到标量子查询或者行子查询，但又不能保证子查询的结果集只有一条记录时，应该使用LIMIT 1语句来限制记录数量
                对于[NOT] IN/ANY/SOME/ALL子查询来说，子查询中不允许有LIMIT语句
        子查询在MySQL中是怎么执行的
            标量子查询、行子查询的执行方式
                对于包含不相关的标量子查询或者行子查询的查询语句来说，
                MySQL会分别独立的执行外层查询和子查询，就当作两个单表查询就好了
            IN子查询优化
                物化表的提出
                    不直接将不相关子查询的结果集当作外层查询的参数，而是将该结果集写入一个临时表里
                物化表转连接
                将子查询转换为semi-join
                    新概念 --- 半连接（英文名：semi-join）。将s1表和s2表进行半连接的意思就是：
                    对于s1表的某条记录来说，我们只关心在s2表中是否存在与之匹配的记录是否存在，
                    而不关心具体有多少条记录与之匹配，最终的结果集中只保留s1表的记录
            ANY/ALL子查询优化
            [NOT] EXISTS子查询的执行

16. 查询优化的百科全书————Explain 详解（上）
    mysql> EXPLAIN SELECT * FROM s1;
    +----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
    | id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
    +----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
    |  1 | SIMPLE      | s1    | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 9688 |   100.00 | NULL  |
    +----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
    
    16.1 执行计划输出中各列详解
        table 
            EXPLAIN语句输出的每条记录都对应着某个单表的访问方法，该条记录的table列代表着该表的表名
        id
            查询语句中每出现一个SELECT关键字，设计MySQL的大叔就会为它分配一个唯一的id值
        select_type
            SIMPLE
                查询语句中不包含UNION或者子查询的查询都算作是SIMPLE类型
            PRIMARY
                对于包含UNION、UNION ALL或者子查询的大查询来说，它是由几个小查询组成的，
                其中最左边的那个查询的select_type值就是PRIMARY
            UNION
                对于包含UNION或者UNION ALL的大查询来说，它是由几个小查询组成的，
                其中除了最左边的那个小查询以外，其余的小查询的select_type值就是UNION
            UNION RESULT
                MySQL选择使用临时表来完成UNION查询的去重工作，针对该临时表的查询的select_type就是UNION RESULT
            SUBQUERY
                如果包含子查询的查询语句不能够转为对应的semi-join的形式，并且该子查询是不相关子查询，
                并且查询优化器决定采用将该子查询物化的方案来执行该子查询时，
                该子查询的第一个SELECT关键字代表的那个查询的select_type就是SUBQUERY
            DEPENDENT SUBQUERY
            DEPENDENT UNION
            DERIVED
                对于采用物化的方式执行的包含派生表的查询，该派生表对应的子查询的select_type就是DERIVED
            MATERIALIZED
            UNCACHEABLE SUBQUERY
            UNCACHEABLE UNION
        partitions
        type
            system
                当表中只有一条记录并且该表使用的存储引擎的统计数据是精确的，
                比如MyISAM、Memory，那么对该表的访问方法就是system
            const
                根据主键或者唯一二级索引列与常数进行等值匹配时，对单表的访问方法就是const
            eq_ref
                在连接查询时，如果被驱动表是通过主键或者唯一二级索引列等值匹配的方式进行访问的
                （如果该主键或者唯一二级索引是联合索引的话，所有的索引列都必须进行等值比较），
                则对该被驱动表的访问方法就是eq_ref
            ref
                当通过普通的二级索引列与常量进行等值匹配时来查询某个表，那么对该表的访问方法就可能是ref
            ref_or_null
                当对普通二级索引进行等值匹配查询，该索引列的值也可以是NULL值时，
                那么对该表的访问方法就可能是ref_or_null
            index_merge
            unique_subquery
            index_subquery
            range
                如果使用索引获取某些范围区间的记录，那么就可能使用到range访问方法
            index
                当我们可以使用索引覆盖，但需要扫描全部的索引记录时，该表的访问方法就是index
            ALL
        possible_keys和key
            possible_keys列表示在某个查询语句中，对某个表执行单表查询时可能用到的索引有哪些，
            key列表示实际用到的索引有哪些
            possible_keys列中的值并不是越多越好，可能使用的索引越多，
            查询优化器计算查询成本时就得花费更长时间，所以如果可以的话，尽量删除那些用不到的索引
        key_len
            key_len列表示当优化器决定使用某个索引执行查询时，该索引记录的最大长度
        ref
            当使用索引列等值匹配的条件去执行查询时，也就是在访问方法是const、eq_ref、ref、
            ref_or_null、unique_subquery、index_subquery其中之一时，
            ref列展示的就是与索引列作等值匹配的东东是个啥，比如只是一个常数或者是某个列
        rows
            需要扫描的行数
        filtered

17. 查询优化的百科全书————Explain 详解（下）
    17.1 执行计划输出中各列详解
        Extra
            No tables used
            Impossible WHERE
            No matching min/max row
            Using index
            Using index condition
            Using where
            Using join buffer (Block Nested Loop)
            Not exists
            Using intersect(...)、Using union(...)和Using sort_union(...)
            Zero limit
            Using filesort
            Using temporary
            Start temporary, End temporary
            LooseScan
            FirstMatch(tbl_name)
    17.2 Json格式的执行计划
    17.3 Extented EXPLAIN

18. 神兵利器————optimizer trace 表的神奇功效
    略 

19. 调节磁盘和CPU的矛盾————InnoDB 的 Buffer Pool
    19.1 缓存的重要性
        即使我们只需要访问一个页的一条记录，那也需要先把整个页的数据加载到内存中
    19.2 InnoDB的Buffer Pool
        啥是个Buffer Pool
            在MySQL服务器启动的时候就向操作系统申请了一片连续的内存，
            叫做Buffer Pool（中文名是缓冲池）
        Buffer Pool内部组成
            Buffer Pool中默认的缓存页大小和在磁盘上默认的页大小是一样的，都是16KB
            控制块和缓存页是一一对应的，它们都被存放到 Buffer Pool 中，
                其中控制块被存放到 Buffer Pool 的前边，缓存页被存放到 Buffer Pool 后边
            free链表的管理
                把所有空闲的缓存页对应的控制块作为一个节点放到一个链表中，
                这个链表也可以被称作free链表（或者说空闲链表）
            缓存页的哈希处理
                表空间号 + 页号作为key，缓存页作为value创建一个哈希表，
                在需要访问某个页的数据时，先从哈希表中根据表空间号 + 页号看看有没有对应的缓存页，
                如果有，直接使用该缓存页就好，如果没有，那就从free链表中选一个空闲的缓存页，
                然后把磁盘中对应的页加载到该缓存页的位置
            flush链表的管理
                如果我们修改了Buffer Pool中某个缓存页的数据，
                那它就和磁盘上的页不一致了，这样的缓存页也被称为脏页（英文名：dirty page）
                存储脏页的链表，凡是修改过的缓存页对应的控制块都会作为一个节点加入到一个链表中，
                因为这个链表节点对应的缓存页都是需要被刷新到磁盘上的，所以也叫flush链表
            LRU链表的管理
                缓存不够的窘境
                简单的LRU链表
                划分区域的LRU链表
                更进一步优化LRU链表
            其他的一些链表
            刷新脏页到磁盘
            多个Buffer Pool实例
            配置Buffer Pool时的注意事项
        Buffer Pool中存储的其它信息
            存储锁信息、自适应哈希索引等信息
        查看Buffer Pool的状态信息
            SHOW ENGINE INNODB STATUS\G

20. 从猫爷被杀说起————事务简介
    20.1 事务的起源
        原子性（Atomicity）
        隔离性（Isolation）
        一致性（Consistency）
        持久性（Durability）
    20.2 事务的概念
        把需要保证原子性、隔离性、一致性和持久性的一个或多个数据库操作称之为一个事务（英文名是：transaction）
        事务大致上划分成了这么几个状态
            活动的（active）
                事务对应的数据库操作正在执行过程中时，我们就说该事务处在活动的状态
            部分提交的（partially committed）
                当事务中的最后一个操作执行完成，但由于操作都在内存中执行，
                所造成的影响并没有刷新到磁盘时，我们就说该事务处在部分提交的状态
            失败的（failed）
                当事务处在活动的或者部分提交的状态时，
                可能遇到了某些错误（数据库自身的错误、操作系统错误或者直接断电等）而无法继续执行，
                或者人为的停止当前事务的执行，我们就说该事务处在失败的状态
            中止的（aborted）
                如果事务执行了半截而变为失败的状态，就是要撤销失败事务对当前数据库造成的影响。
                书面一点的话，我们把这个撤销的过程称之为回滚。当回滚操作执行完毕时，
                也就是数据库恢复到了执行事务之前的状态，我们就说该事务处在了中止的状态
            提交的（committed）
                当一个处在部分提交的状态的事务将修改过的数据都同步到磁盘上之后，我们就可以说该事务处在了提交的状态
    20.3 MySQL中事务的语法
        开启事务
            BEGIN [WORK];
            START TRANSACTION;
        提交事务
            COMMIT [WORK]
        手动中止事务
            ROLLBACK [WORK]
        自动提交
        隐式提交
            定义或修改数据库对象的数据定义语言（Data definition language，缩写为：DDL）
            隐式使用或修改mysql数据库中的表
            事务控制或关于锁定的语句
            加载数据的语句
            关于MySQL复制的一些语句
            其它的一些语句
        保存点
            SAVEPOINT 保存点名称;
            ROLLBACK [WORK] TO [SAVEPOINT] 保存点名称;
            RELEASE SAVEPOINT 保存点名称;

21. 说过的话就一定要办到————redo日志（上）
    21.1 redo日志是个啥
        如何保证这个持久性呢？一个很简单的做法就是在事务提交完成之前把该事务所修改的所有页面都刷新到磁盘，
        但是这个简单粗暴的做法有些问题：
            1. 刷新一个完整的数据页太浪费了
            2. 随机IO刷起来比较慢
        这样我们在事务提交时，把上述内容刷新到磁盘中，即使之后系统崩溃了，
        重启之后只要按照上述内容所记录的步骤重新更新一下数据页，
        那么该事务对数据库中所做的修改又可以被恢复出来，也就意味着满足持久性的要求。
        因为在系统崩溃重启时需要按照上述内容所记录的步骤重新更新数据页，
        所以上述内容也被称之为重做日志

        与在事务提交时将所有修改过的内存中的页面刷新到磁盘中相比，
        只将该事务执行过程中产生的redo日志刷新到磁盘的好处如下：
            1. redo日志占用的空间非常小
            2. redo日志是顺序写入磁盘的
    21.2 redo日志格式
        type | space id | page number | data |
        简单的redo日志类型
            redo日志中只需要记录一下在某个页面的某个偏移量处修改了几个字节的值，
            具体被修改的内容是啥就好了，设计InnoDB的大叔把这种极其简单的redo日志称之为物理日志
        复杂一些的redo日志类型
            
        redo日志格式小结
            redo日志会把事务在执行过程中对数据库所做的所有修改都记录下来，
            在之后系统崩溃重启后可以把事务所做的任何修改都恢复出来
    21.3 Mini-Transaction
        以组的形式写入redo日志
        Mini-Transaction的概念
            对底层页面中的一次原子访问的过程称之为一个Mini-Transaction
    21.4 redo日志的写入过程
        redo log block
        redo日志缓冲区
        redo日志写入log buffer
    
22. 说过的话就一定要办到————redo日志（下）
    22.1 redo日志文件
        redo日志刷盘时机
            log buffer空间不足时
            事务提交时
            后台线程不停的刷刷刷
            正常关闭服务器时
            做所谓的checkpoint时
            其他
        redo日志文件组
        redo日志文件格式
            将log buffer中的redo日志刷新到磁盘的本质就是把block的镜像写入日志文件中
    22.2 Log Sequeue Number
        flushed_to_disk_lsn
        lsn值和redo日志文件偏移量的对应关系
        flush链表中的LSN
    22.3 checkpoint
        批量从flush链表中刷出脏页
        查看系统中的各种LSN值
    22.4 innodb_flush_log_at_trx_commit的用法
        0   当该系统变量值为0时，表示在事务提交时不立即向磁盘中同步redo日志，这个任务是交给后台线程做的
        1   当该系统变量值为1时，表示在事务提交时需要将redo日志同步到磁盘，可以保证事务的持久性。
            1也是innodb_flush_log_at_trx_commit的默认值
        2   当该系统变量值为2时，表示在事务提交时需要将redo日志写到操作系统的缓冲区中，但并不需要保证将日志真正的刷新到磁盘。
            这种情况下如果数据库挂了，操作系统没挂的话，事务的持久性还是可以保证的，但是操作系统也挂了的话，那就不能保证持久性了
    22.5 崩溃恢复
        确定恢复的起点
            最近发生的那次checkpoint的信息
        确定恢复的终点
            普通block的log block header部分有一个称之为LOG_BLOCK_HDR_DATA_LEN的属性，
            该属性值记录了当前block里使用了多少字节的空间。对于被填满的block来说，该值永远为512。
            如果该属性的值不为512，那么就是它了，它就是此次崩溃恢复中需要扫描的最后一个block
        怎么恢复
            使用哈希表
            跳过已经刷新到磁盘的页面

23. 后悔了怎么办————undo日志（上）
    23.1 事务回滚的需求
    23.2 事务id
        给事务分配id的时机
            1. 对于只读事务来说，只有在它第一次对某个用户创建的临时表执行增、删、改操作时才会为这个事务分配一个事务id，
                否则的话是不分配事务id的
            2. 对于读写事务来说，只有在它第一次对某个表（包括用户创建的临时表）执行增、删、改操作时才会为这个事务分配一个事务id，
                否则的话也是不分配事务id的
        事务id是怎么生成的
            全局变量自增
        trx_id隐藏列
    23.3 undo日志的格式
        INSERT操作对应的undo日志
            roll_pointer隐藏列的含义
                一个指向记录对应的undo日志的一个指针
        DELETE操作对应的undo日志
            删除的过程需要经历两个阶段
                阶段一：仅仅将记录的delete_mask标识位设置为1，
                    其他的不做修改（其实会修改记录的trx_id、roll_pointer这些隐藏列的值）。
                    这个阶段称为 delete mark
                    为了实现MVCC的功能
                阶段二：当该删除语句所在的事务提交之后，会有专门的线程后来真正的把记录删除掉
                    这个阶段称之为purge
        UPDATE操作对应的undo日志
            不更新主键的情况
                就地更新（in-place update）
                    更新后的列和更新前的列占用的存储空间都一样大
                先删除掉旧记录，再插入新记录
            更新主键的情况
                在聚簇索引中分了两步处理：
                    将旧记录进行delete mark操作
                    根据更新后各列的值创建一条新记录，并将其插入到聚簇索引中（需重新定位插入的位置）

24. 后悔了怎么办————undo日志（下）
    24.1 通用链表结构
    24.2 FIL_PAGE_UNDO_LOG页面
    24.3 Undo页面链表
        单个事务中的Undo页面链表
        多个事务中的Undo页面链表
    24.4 undo日志具体写入过程
        段（Segment）的概念
        Undo Log Segment Header
        Undo Log Header
    24.5 重用Undo页面
    24.6 回滚段
        回滚段的概念
        从回滚段中申请Undo页面链表
        多个回滚段
        回滚段的分类
        为事务分配Undo页面链表详细过程
    24.7 回滚段相关配置
        配置回滚段数量
        配置undo表空间

25. 一条记录的多幅面孔————事务的隔离级别与MVCC
    25.1 事务隔离级别
        事务并发执行遇到的问题
            脏写（Dirty Write）
                一个事务修改了另一个未提交事务修改过的数据
            脏读（Dirty Read）
                一个事务读到了另一个未提交事务修改过的数据
            不可重复读（Non-Repeatable Read）
                一个事务只能读到另一个已经提交的事务修改过的数据，
                并且其他事务每对该数据进行一次修改并提交后，
                该事务都能查询得到最新值
            幻读（Phantom）
                一个事务先根据某些条件查询出一些记录，
                之后另一个事务又向表中插入了符合这些条件的记录，
                原先的事务再次按照该条件查询时，
                能把另一个事务插入的记录也读出来
        SQL标准中的四种隔离级别
            隔离级别	        脏读	        不可重复读	    幻读
            READ UNCOMMITTED	Possible	Possible	    Possible
            READ COMMITTED	Not Possible	Possible	    Possible
            REPEATABLE READ	Not Possible	Not Possible	Possible
            SERIALIZABLE	Not Possible	Not Possible	Not Possible
        MySQL中支持的四种隔离级别
            MySQL在REPEATABLE READ隔离级别下，是可以禁止幻读问题的发生的
            MySQL的默认隔离级别为REPEATABLE READ
    25.2 MVCC原理
        版本链
            对该记录每次更新后，都会将旧值放到一条undo日志中，
            就算是该记录的一个旧版本，随着更新次数的增多，
            所有的版本都会被roll_pointer属性连接成一个链表，
            我们把这个链表称之为版本链，版本链的头节点就是当前记录最新的值
        ReadView
            ReadView中主要包含4个比较重要的内容
                1. m_ids：表示在生成ReadView时当前系统中活跃的读写事务的事务id列表
                2. min_trx_id：表示在生成ReadView时当前系统中活跃的读写事务中最小的事务id，
                    也就是m_ids中的最小值
                3. max_trx_id：表示生成ReadView时系统中应该分配给下一个事务的id值
                4. creator_trx_id：表示生成该ReadView的事务的事务id
            对于使用READ UNCOMMITTED隔离级别的事务来说，
            由于可以读到未提交事务修改过的记录，所以直接读取记录的最新版本就好了

            对于使用SERIALIZABLE隔离级别的事务来说，使用加锁的方式来访问记录

            对于使用READ COMMITTED和REPEATABLE READ隔离级别的事务来说，
            都必须保证读到已经提交了的事务修改过的记录，
            也就是说假如另一个事务已经修改了记录但是尚未提交，
            是不能直接读取最新版本的记录的，
            核心问题就是：需要判断一下版本链中的哪个版本是当前事务可见的

            READ COMMITTED —— 每次读取数据前都生成一个ReadView
            REPEATABLE READ —— 在第一次读取数据时生成一个ReadView
        MVCC小结
            所谓的MVCC（Multi-Version Concurrency Control ，多版本并发控制）
            指的就是在使用READ COMMITTD、REPEATABLE READ这两种隔离级别的事务在执行
            普通的SELECT操作时访问记录的版本链的过程
    25.3 关于purge

26. 工作面试老大难————锁
    26.1 解决并发事务带来问题的两种基本方式
        锁结构
            当一个事务想对这条记录做改动时，首先会看看内存中有没有与这条记录关联的锁结构，
            当没有的时候就会在内存中生成一个锁结构与之关联
                trx信息：代表这个锁结构是哪个事务生成的。
                is_waiting：代表当前事务是否在等待。
        方案一：读操作利用多版本并发控制（MVCC），写操作进行加锁
        方案二：读、写操作都采用加锁的方式

        一致性读（Consistent Reads）
            事务利用MVCC进行的读取操作称之为一致性读，或者一致性无锁读，有的地方也称之为快照读
        锁定读（Locking Reads）
            共享锁和独占锁
            锁定读的语句
                对读取的记录加S锁：
                    SELECT ... LOCK IN SHARE MODE;
                对读取的记录加X锁：
                    SELECT ... FOR UPDATE;
    26.2 多粒度锁
        IS、IX锁是表级锁，它们的提出仅仅为了在之后加表级别的S锁和X锁时可以快速判断表中的记录是否被上锁，
        以避免用遍历的方式来查看表中有没有上锁的记录，也就是说其实IS锁和IX锁是兼容的，IX锁和IX锁是兼容的
    26.3 MySQL中的行锁和表锁
        其他存储引擎中的锁
        InnoDB存储引擎中的锁
            InnoDB中的表级锁
                表级别的S锁、X锁
                表级别的IS锁、IX锁
                表级别的AUTO-INC锁
            InnoDB中的行级锁
                    行锁，也称为记录锁，顾名思义就是在记录上加的锁。
                Record Locks
                    官方的类型名称为：LOCK_REC_NOT_GAP
                    仅仅把一条记录锁上
                Gap Locks
                    官方的类型名称为：LOCK_GAP
                    MySQL在REPEATABLE READ隔离级别下解决幻读问题的两种方案
                        1. MVCC
                        2. 加锁
                    gap锁的提出仅仅是为了防止插入幻影记录而提出的
                Next-Key Locks
                    官方的类型名称为：LOCK_ORDINARY
                    Record Locks + Gap Locks
                Insert Intention Locks
                    插入意向锁
                隐式锁
        InnoDB锁的内存结构
        语句加锁分析
