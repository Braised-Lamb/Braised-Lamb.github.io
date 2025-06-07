---
layout: post
title: 
date: 2025-06-08 01:27:00 +0800
categories:
  - Qt
tags:
  - Qt
  - MySql
toc: true
---

用于windows平台下基于MSVC的Qt和MySql配置

# Qt安装

Qt安装的时候要选择source，用于编译mysql驱动


![](https://cdn.jsdelivr.net/gh/Braised-Lamb/picbed/cb5368eb6c66f5fd59f58cb680d58a20_MD5.jpeg)


![](https://cdn.jsdelivr.net/gh/Braised-Lamb/picbed/1006342a81b6be28a1334a92848abe6b_MD5.jpeg)

CDB下载：[winsdksetup.exe](https://download.microsoft.com/download/4/2/2/42245968-6A79-4DA7-A5FB-08C0AD0AE661/windowssdk/winsdksetup.exe)

***需要选择对应MSVC编译器版本***

![](https://cdn.jsdelivr.net/gh/Braised-Lamb/picbed/d79610ebcf9b0c31c50e4d929627775a_MD5.jpeg)

# 编译qmysql驱动

在Qt的source文件夹在找到mysql.pro工程，具体路径为：`${QT_DIR}\Src\qtbase\src\plugins\sqldrivers\mysql`

用Qt Creator打开mysql.pro文件，修改对应文件

mysql.pro

```c
TARGET = qsqlmysql

HEADERS += $$PWD/qsql_mysql_p.h
SOURCES += $$PWD/qsql_mysql.cpp $$PWD/main.cpp


# QMAKE_USE += mysql


OTHER_FILES += mysql.json


PLUGIN_CLASS_NAME = QMYSQLDriverPlugin
include(../qsqldriverbase.pri)
  

#这个主要是添加.h依赖文件使用
INCLUDEPATH+="C:\Program Files\MySQL\MySQL Server 8.0\include"

#添加依赖的.lib文件
LIBS+="C:\Program Files\MySQL\MySQL Server 8.0\lib\libmysql.lib"

#生成你所需要的dll存放目录
DESTDIR="C:\plugins\sqldrivers"
```

qsqldriverbase.pri 

```c
QT  = core core-private sql-private

# For QMAKE_USE in the parent projects.
# include($$shadowed($$PWD)/qtsqldrivers-config.pri)
include(./configure.pri)

PLUGIN_TYPE = sqldrivers
load(qt_plugin)
  
DEFINES += QT_NO_CAST_TO_ASCII QT_NO_CAST_FROM_ASCII
```

修改完成后，选择合适的编译套件（MSVC或者MinGW）构建工程，在输出文件夹里得到了两个dll文件，qsqlmysql.dll和qsqlmysqld.dll

# 拷贝文件

1. qsqlmysql.dll、qsqlmysqld.dll：复制到对应编译器文件夹下，例如：`${QT_DIR}\msvc2017_64\plugins\sqldrivers`
2. libmysql.dll：从MySQL的安装文件夹下复制到Qt的文件夹下，`C:\Program Files\MySQL\MySQL Server 8.0\lib`到`C:\Qt\Qt5.14.2\5.14.2\msvc2017_64\bin`
3. libcrypto-3-x64.dll、libssl-3-x64.dll：`C:\Program Files\MySQL\MySQL Server 8.0\bin`到`C:\Qt\Qt5.14.2\5.14.2\msvc2017_64\bin`

# 测试

```c++
 #include "mainwindow.h"        // *****注意****

#include<QSqlDatabase>
#include <QApplication>
#include<QMessageBox>
#include <QSqlError>
#include <QDebug>
  
int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    MainWindow w;            //*****替换一下*****
    w.show();
  
    QSqlDatabase db = QSqlDatabase::addDatabase("QMYSQL");
        db.setHostName("127.0.0.1");  //连接本地主机
        db.setPort(3306);             //端口号，默认的
        db.setDatabaseName("mysql");     //数据库名
        db.setUserName("root");         //用户
        db.setPassword("mysql");  //密码
        bool ok = db.open();
        if (ok){
            QMessageBox::information(0, "infor", "link success");
        }
        else {
            QMessageBox::information(0, "infor", "link failed");
           // qDebug()<<"error open database because"<<db.lastError().text();
        }
        QSqlError error;
        error = db.lastError();
        if(error.isValid())
        {
            switch(error.type())
            {
            case QSqlError::NoError:
                qDebug() << "no error";
                break;
            case QSqlError::ConnectionError:
                qDebug() << error.text();
                break;
            case QSqlError::StatementError:
                qDebug() << error.text();
                break;
            case QSqlError::TransactionError:
                qDebug() << error.text();
                break;
            default:
                qDebug() << error.text();
                break;
            }
        }
        return a.exec();
}
```

测试成功：

![image.png](https://cdn.jsdelivr.net/gh/Braised-Lamb/picbed/f460619642bc046b4fd2b902371970d7_MD5.jpeg)