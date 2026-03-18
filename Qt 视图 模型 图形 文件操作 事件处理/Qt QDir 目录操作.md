## Qt QDir 目录操作笔记：系统目录与文件实战

QDir 是 Qt 中用于目录操作的核心类，能轻松访问系统目录结构、管理文件路径、创建 / 删除目录等，结合文件读写类（QFile、QDataStream）可实现完整的文件管理功能。这篇笔记聚焦核心知识点与实战场景，帮你快速掌握 QDir 的用法。

## 一、QDir 核心定位与基础特性

### 1. 核心作用

- 访问系统目录结构（如获取 C 盘、桌面目录下的文件 / 文件夹）；
- 处理路径名（绝对路径 / 相对路径转换、路径拼接与清理）；
- 目录操作（创建、删除、重命名、切换目录）；
- 筛选文件 / 文件夹（按名称、类型、大小等过滤）。

### 2. 关键特性

- 跨平台兼容：**Qt 统一使用 “/” 作为目录分隔符，自动适配 Windows（\）和 Linux/Mac（/）**；

- 支持两种路径：

  - 绝对路径：从根目录开始（如`C:/Users/Desktop`、`/home/user`）；
  - 相对路径：相对于当前程序运行目录（如`./data`、`../config`）；

  

- 资源访问：不仅能操作本地文件系统，还能访问 Qt 的资源文件（`:res/icon.png`）。

## 二、必学核心 API（按功能分类）

### 1. 路径相关操作

|                  API 函数                   |               功能说明                |                             示例                             |
| :-----------------------------------------: | :-----------------------------------: | :----------------------------------------------------------: |
|         `QDir(const QString &path)`         |      构造函数，绑定目标目录路径       |               `QDir dir("C:/Users/Desktop");`                |
|              `absolutePath()`               |          获取目录的绝对路径           |  `qDebug() << dir.absolutePath();` → 输出`C:/Users/Desktop`  |
|              `canonicalPath()`              | 获取标准路径（去除`./`、`../`等冗余） |         处理`C:/Users/../Desktop` → 输出`C:/Desktop`         |
|      `cleanPath(const QString &path)`       |    静态函数，清理路径中的冗余字符     |          `QDir::cleanPath("a/b/../c")` → 输出`a/c`           |
|    `isAbsolutePath(const QString &path)`    |     静态函数，判断是否为绝对路径      |          `QDir::isAbsolutePath("C:/test")` → `true`          |
| `absoluteFilePath(const QString &fileName)` |    拼接目录与文件名，返回绝对路径     | `dir.absoluteFilePath("test.txt")` → `C:/Users/Desktop/test.txt` |

### 2. 目录操作（创建 / 删除 / 切换）

|                         API 函数                         |             功能说明             |                             示例                             |
| :------------------------------------------------------: | :------------------------------: | :----------------------------------------------------------: |
|             `mkdir(const QString &dirName)`              |           创建单层目录           |       `dir.mkdir("newDir")` → 在桌面创建`newDir`文件夹       |
|             `mkpath(const QString &dirPath)`             |     创建多层目录（支持嵌套）     |       `dir.mkpath("a/b/c")` → 创建`a`目录及子目录`b/c`       |
|             `rmdir(const QString &dirName)`              |  删除空目录（非空目录删除失败）  |           `dir.rmdir("newDir")` → 删除空的`newDir`           |
|                  `removeRecursively()`                   | 删除目录及其所有内容（递归删除） | `QDir("a").removeRecursively()` → 删除`a`目录及所有子文件 / 文件夹 |
|               `cd(const QString &dirName)`               |     切换到当前目录下的子目录     |     `dir.cd("newDir")` → 切换到`C:/Users/Desktop/newDir`     |
|                         `cdUp()`                         |          切换到上级目录          |            `dir.cdUp()` → 从`newDir`回到`Desktop`            |
| `rename(const QString &oldName, const QString &newName)` |        重命名目录 / 文件         |       `dir.rename("oldDir", "newDir")` → 重命名文件夹        |

### 3. 目录内容获取与筛选

|                           API 函数                           |                       功能说明                        |                             示例                             |                                   |
| :----------------------------------------------------------: | :---------------------------------------------------: | :----------------------------------------------------------: | :-------------------------------: |
| `entryList(const QStringList &nameFilters, Filters filters, SortFlags sort)` |    获取目录下的文件 / 文件夹列表（支持筛选和排序）    |                        见下文实战示例                        |                                   |
|                     `entryInfoList(...)`                     | 获取目录下的文件 / 文件夹详细信息（大小、修改时间等） |              返回`QFileInfoList`，包含文件属性               |                                   |
|         `setNameFilters(const QStringList &filters)`         |                  设置文件名筛选规则                   | `dir.setNameFilters({"*.txt", "*.bin"})` → 只显示 txt 和 bin 文件 |                                   |
|                 `setFilter(Filters filters)`                 |       设置筛选类型（文件 / 目录 / 隐藏文件等）        |                  `dir.setFilter(QDir::Files                  |   QDir::Dirs)` → 显示文件和目录   |
|                 `setSorting(SortFlags sort)`                 |        设置排序规则（按名称 / 大小 / 时间等）         |                  `dir.setSorting(QDir::Name                  | QDir::Reversed)` → 按名称倒序排列 |

## 三、实战场景：核心功能示例

### 1. 基础操作：创建目录 + 路径处理

```cpp
#include <QDir>
#include <QDebug>

void basicDirOperation() {
    // 1. 绑定桌面目录（绝对路径）
    QDir desktopDir("C:/Users/Lenovo/Desktop");
    
    // 2. 创建单层目录
    if (desktopDir.mkdir("Qt_Dir_Test")) {
        qDebug() << "单层目录创建成功";
    } else {
        qDebug() << "单层目录创建失败（可能已存在）";
    }
    
    // 3. 创建多层目录
    if (desktopDir.mkpath("Qt_Dir_Test/a/b")) {
        qDebug() << "多层目录创建成功";
    }
    
    // 4. 切换到目标目录
    if (desktopDir.cd("Qt_Dir_Test/a/b")) {
        qDebug() << "当前目录：" << desktopDir.absolutePath(); // 输出C:/Users/Lenovo/Desktop/Qt_Dir_Test/a/b
    }
    
    // 5. 拼接绝对路径
    QString filePath = desktopDir.absoluteFilePath("test.txt");
    qDebug() << "文件绝对路径：" << filePath; // 输出C:/Users/Lenovo/Desktop/Qt_Dir_Test/a/b/test.txt
    
    // 6. 切换到上级目录
    desktopDir.cdUp();
    qDebug() << "切换上级目录后：" << desktopDir.absolutePath(); // 输出C:/Users/Lenovo/Desktop/Qt_Dir_Test/a
}
```

### 2. 高级操作：筛选并遍历目录下的文件

```cpp
void listDirContent() {
    QDir dir("C:/Users/Lenovo/Desktop");
    
    // 设置筛选规则：显示文件和目录，隐藏系统文件，只显示txt和exe类型
    dir.setFilter(QDir::Files | QDir::Dirs | QDir::NoDotAndDotDot | QDir::Hidden);
    dir.setNameFilters({"*.txt", "*.exe"});
    
    // 设置排序规则：按名称正序排列
    dir.setSorting(QDir::Name);
    
    // 获取目录内容列表
    QStringList entryList = dir.entryList();
    qDebug() << "桌面下的txt和exe文件/目录：";
    for (const QString& entry : entryList) {
        qDebug() << entry;
    }
    
    // 获取文件详细信息（大小、修改时间）
    QFileInfoList infoList = dir.entryInfoList();
    qDebug() << "\n文件详细信息：";
    for (const QFileInfo& info : infoList) {
        qDebug() << "名称：" << info.fileName()
                 << " 类型：" << (info.isFile() ? "文件" : "目录")
                 << " 大小：" << info.size() << "字节"
                 << " 修改时间：" << info.lastModified().toString("yyyy-MM-dd hh:mm:ss");
    }
}
```

### 3. 综合实战：遍历系统目录（模拟文件浏览器核心）

```cpp
#include <QDir>
#include <QDebug>

// 递归遍历目录下的所有文件和子目录
void traverseDir(const QString& path, int level = 0) {
    QDir dir(path);
    
    // 筛选：排除.和..目录，显示文件和目录
    dir.setFilter(QDir::Files | QDir::Dirs | QDir::NoDotAndDotDot);
    
    // 遍历目录内容
    for (const QFileInfo& info : dir.entryInfoList()) {
        // 用缩进表示层级（模拟文件树结构）
        QString indent = QString(level * 2, ' ');
        
        if (info.isDir()) {
            // 目录：输出名称并递归遍历子目录
            qDebug() << indent << "[目录]" << info.fileName();
            traverseDir(info.absoluteFilePath(), level + 1);
        } else {
            // 文件：输出名称和大小
            qDebug() << indent << "[文件]" << info.fileName() << "(" << info.size() << "字节)";
        }
    }
}

// 调用示例：遍历C盘Users目录
int main() {
    traverseDir("C:/Users");
    return 0;
}
```

### 补充实战：

可求出路径下的文件的大小，以及路径

```cpp
#include <QCoreApplication>

#include <QDir>
#include <QStringList>
#include <QtDebug>


// 自定义函数实现获取目录下大小
qint64 GetDirFileInfoSizeFunc(const QString &qpath)
{
    // QDir类专门用来操作路径名称或底层文件系统，可以使用相对或绝对路径来指向一个文件/目录
    QDir qdirs(qpath);

    qint64 qsize=0; // 存放目录占据空间

    // QFileInfor类提供有关文件系统当中文件名称、路径（位置）、访问权限、文件类型等等信息
    // entryInfoList函数获取过滤之后获得的文件名夹下面的文件信息列表
    foreach(QFileInfo finfo,qdirs.entryInfoList(QDir::Files))
    {
        qsize=qsize+finfo.size();
    }

    // QDir::Dirs列出目录。QDir::separator()不列出文件系统当中的特殊文件
    foreach(QString sDir,qdirs.entryList(QDir::Dirs|QDir::NoDotAndDotDot))
    {
        qsize=qsize+GetDirFileInfoSizeFunc(qpath+QDir::separator()+sDir);
    }

    
    //计算容量大小
    char uint='B';

    qint64 currentdirsize=qsize;

    if(currentdirsize>1024)
    {
        currentdirsize=currentdirsize/1024;
        uint='K';

        if(currentdirsize>1024)
        {
            currentdirsize=currentdirsize/1024;
            uint='M';
            if(currentdirsize>1024)
            {
                currentdirsize=currentdirsize/1024;
                uint='G';
                if(currentdirsize>1024)
                {
                    currentdirsize=currentdirsize/1024;
                    uint='T';
                }
            }
        }
    }
    qDebug()<<"目录占据空间为："<<currentdirsize<<"\t"<<qPrintable(qpath);
    return qsize;
}



int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 该字符串存储路径
    QString strPath;

    strPath=QDir::currentPath(); // 获取当前目录
    qDebug()<<"当前目录为："<<strPath<<endl;

    // 调用此函数求目录占据空间的大小
    GetDirFileInfoSizeFunc(strPath);

    return a.exec();
}
```



## 四、关键避坑点

1. **路径分隔符**：始终使用 “/” 作为分隔符，Qt 会自动转换为系统兼容格式，避免手动写 “\”（需转义为 “\”，容易出错）；
2. **目录权限**：创建 / 删除目录时需确保程序有对应路径的权限（如 Windows 系统 C 盘根目录可能无写入权限，优先使用桌面或程序目录）；
3. **`mkdir`与`mkpath`区别**：`mkdir`只能创建单层目录（父目录不存在则失败），`mkpath`可创建多层嵌套目录；
4. **`rmdir`限制**：`rmdir`只能删除空目录，删除非空目录需用`removeRecursively()`（谨慎使用，会彻底删除所有内容）；
5. **相对路径基准**：相对路径是相对于程序运行目录（如 Qt Creator 默认是 build 文件夹），而非源码所在目录，建议优先使用绝对路径或拼接路径。

## 五、QDir 与 QFile/QFileInfo 的配合使用

QDir 常与其他文件类配合，实现完整的文件管理流程：

- **QDir + QFile**：用 QDir 处理路径，QFile 负责文件读写；
- **QDir + QFileInfo**：用 QFileInfo 获取文件详细属性（大小、修改时间、是否可读可写等）。

示例：检查文件是否存在并获取大小

```cpp
#include <QDir>
#include <QFile>
#include <QFileInfo>

void checkFile() {
    QDir desktopDir("C:/Users/Lenovo/Desktop");
    QString filePath = desktopDir.absoluteFilePath("test.txt");
    
    // 检查文件是否存在
    if (QFile::exists(filePath)) {
        QFileInfo info(filePath);
        qDebug() << "文件存在：" << filePath;
        qDebug() << "文件大小：" << info.size() << "字节";
        qDebug() << "是否可写：" << info.isWritable();
    } else {
        qDebug() << "文件不存在，创建文件...";
        QFile file(filePath);
        file.open(QIODevice::WriteOnly);
        file.close();
    }
}
```



## 总结

QDir 的核心是 “路径处理 + 目录管理”，掌握以下几点即可应对绝大多数场景：

1. 路径操作：用`absolutePath()`、`absoluteFilePath()`确保路径正确性；
2. 目录操作：`mkdir`/`mkpath`创建目录，`rmdir`/`removeRecursively()`删除目录；
3. 内容筛选：用`setFilter()`、`setNameFilters()`精准获取目标文件 / 目录；
4. 配合使用：与 QFile、QFileInfo 结合，实现 “目录管理 + 文件读写 + 属性查询” 的完整流程。