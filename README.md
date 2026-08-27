# 笔记
一名运维工程师的运维笔记。
## 目录索引
- **[操作系统](/操作系统/Linux/)**
  - [Linux系统概述](/操作系统/Linux/Linux系统概述.md)
- **[数据库](/数据库/)**
  - [关系型数据库概念](/数据库/RDB/关系型数据库概念.md)
  - [MySQL单实例安装](/数据库/RDB/MySQL/单实例安装.md)
  - [MySQL密码验证组件安装](/数据库/RDB/MySQL/安装MySQL密码验证组件.md)
  - [非关系型数据库概述](/数据库/NoSQL/非关系型数据库概述.md)


# TEST
This directory contains files that can be used to set up the menagerie
database that is used in the tutorial chapter of the MySQL Reference
Manual.

These instructions assume that your current working directory is
the directory that contains the files created by unpacking the
menagerie.zip or menagerie.tar.gz distribution. Any MySQL programs
that you use (such as mysql or mysqlimport) should be invoked from
that directory.

To invoke mysql or mysqlimport, either specify the full path name
to the program, or set your PATH environment variable to the bin
directory that contains the programs so that you can invoke them
from anywhere without specifying their full path name. (See your
operating system's help instructions for environment variables.)
The full path name to the programs depends on where MySQL is installed.

For the mysql and mysqlimport commands, supply any connection parameters
necessary (host, user, password) on the command line before the database
name.  For more information, see:

  http://dev.mysql.com/doc/mysql/en/invoking-programs.html


First, you must create the menagerie database.  Invoke the mysql program,
and then issue this statement:

  mysql> CREATE DATABASE menagerie;

If you use a database name different from menagerie in the CREATE
DATABASE statement, use that same name wherever you see menagerie
in the following instructions.

Select the menagerie database as the default database:

  mysql> USE menagerie;

Create the pet table:

  mysql> SOURCE cr_pet_tbl.sql

Load the pet table:

  mysql> LOAD DATA LOCAL INFILE 'pet.txt' INTO TABLE pet;

To add Puffball's record, use this command:

  mysql> SOURCE ins_puff_rec.sql

Create the event table:

  mysql> SOURCE cr_event_tbl.sql

Load the event table:

  mysql> LOAD DATA LOCAL INFILE 'event.txt' INTO TABLE event;


Alternate procedure:

If you want to create and load the tables from your command interpreter
(cmd.exe or command.com in Windows, or your shell in Unix), use the
following commands after using CREATE DATABASE as shown earlier to
create the menagerie database. In these instructions, "shell>"
represents your command interpreter prompt.

Create the pet table:

  shell> mysql menagerie < cr_pet_tbl.sql

To load the pet table, use either of these commands:

  shell> mysql menagerie < load_pet_tbl.sql
  shell> mysqlimport --local menagerie pet.txt

To add Puffball's record, use this command:

  shell> mysql menagerie < ins_puff_rec.sql

Create the event table:

  shell> mysql menagerie < cr_event_tbl.sql

Load the event table:

  shell> mysqlimport --local menagerie event.txt
