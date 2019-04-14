## Emoji

Emoji 字符的特殊之处是，在存储时，需要用到 4 个字节。而 MySQL 中常见的 utf8 字符集的 utf8_general_ci 这个 collate 最大只支持 3 个字节。所以为了能够存储 Emoji，你需要改用 utf8mb4 字符集。

在创建表时，用类似这样的语句：

CREATE TABLE `tbl` (...) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE utf8mb4_general_ci;

## MySQL 版本

对 utf8mb4 字符集的支持是 MySQL 5.5 的新功能，所以你需要确保你使用的 MySQL 版本至少是 5.5。基本上，2014 年以后的新项目都应该直接上 5.6 了。

## MySQL 备份和导入

在启用了 utf8mb4 字符集之后，备份和导入时就不能再用默认参数了。

用 mysqldump 备份时，需要加入：

mysqldump --default-character-set=utf8mb4

而在恢复备份或通过程序连接时，需要在每次连接打开之后发送下面这条 SQL 指令：

SET CHARSET utf8mb4