# Java File

#### 内容摘要

File，API，应用，拿到系统属性 `System.getProperties().list(System.out);`

---

#### File

* 操作系统中所有的信息都是以**文件**的形式表示

* File 类：**以抽象的方式代表文件名和目录路径名**

  * **抽象表示：**为路径名创建 File 实例，不会直接检查该路径名是否存在该文件

* 构造方法

  * `File (String pathName)`
    * 传入路径名字符串，转化为抽象路径名，创建一个新的 File 实例
    * Windows 下绝对路径：`d:\\fileTest.txt`
    * MacOS 下绝对路径： `/User/fileTest.txt`
  * `File (String parent, String child)`
    * parent 目录名， child 文件或目录名
  * `File (File parent, String child)`
    * 父路径（绝对路径） + 子文件名

* 绝对路径：只需该路径就能找到文件，需要前缀来标示

  * win：`d:\\`  盘符+冒号+反斜杠
  * mac：`/` 斜杠

* 相对路径：没有前缀，在定位文件时根据绝对路径来定位

  * 默认情况会根据，当前用户目录 `usr.dir` 属性
  * 即，当前 Project 目录下
  * `relative pathnames against the current user directory`

* **拿到系统属性**：

  ```Java
  System.getProperties().list(System.out);
  ```

> An *abstract pathname* has two components:
>
> 1. An optional system-dependent *prefix* string, such as a disk-drive specifier, `"/"` for the UNIX root directory, or `"\\\"` for a Microsoft Windows UNC pathname, and
> 2. A sequence of zero or more string *names*.
>
> * For UNIX platforms, the prefix of an absolute pathname is always `"/"`. Relative pathnames have no prefix. The abstract pathname denoting the root directory has the prefix `"/"` and an empty name sequence.
> * For Microsoft Windows platforms, the prefix of a pathname that contains a drive specifier consists of the drive letter followed by `":"` and possibly followed by `"\\"` if the pathname is absolute. The prefix of a UNC pathname is `"\\\"`; the hostname and the share name are the first two names in the name sequence. A relative pathname that does not specify a drive has no prefix.
> * By default the classes in the `java.io`package always resolve relative pathnames against the current user directory. This directory is named by the system property `user.dir`, and is typically the directory in which the Java virtual machine was invoked

#### File API

* 详见 Java doc

* 创建

  * `public boolean createNewFile()`
    * 创建文件，物理创建
    * 若文件不存在，则创建返回 true
    * 若文件存在，则创建失败返回 false
  * `public boolean mkdir()`  创建目录
    * 父目录路径不存在的时候，不会创建成功
  * `public boolean mkdirs()`  创建目录
    * 父目录路径不存在，会创建父目录以及当前目录

* 删除

  * `public boolean delete()`

    * 当要删除的文件不存在的时候，不会抛出异常
    * 目录必须为空才能删除

    ```Java
    	// 递归删除一个非空目录
    	public static void deleteFolder(File folder) {
        	File[] files = folder.listFiles();
            if(files!=null) { 
                for(File f: files) {
                    if(f.isDirectory()) {
                        deleteFolder(f);
                    } else {
                        f.delete();
                    }
                }
            }
            folder.delete();
    	}
    ```

* 重命名

  * `public boolean renameTo(File file)`
    * 可以重命名
    * 重命名基础上，还可以移动，类似剪切操作

* 文件大小

  * `public long length()`
    * 文件长度：**字节**
  * `public long getTotalSpace()`
    * 文件所在分区大小：字节
  * `public long getFreeSpace()`
    * 文件所在分区空闲空间大小
  * `public long getUsableSpace()`
    * 文件所在分区可用空间大小

* 列表

  * `public String[] lists()`
    * 返回 **目录** 中的文件和目录 字符串数组
    * 如果是文件，返回 null
  * `public File[] listFiles()`
    * 返回 **目录** 中的文件和目录 文件数组
    * 如果是文件，返回 null
  * `public File[] listFiles(FileFilter filter)`
    * 文件过滤条件
    * 当前正在判断的文件对象与过滤条件比较
  * `public File[] listFiles(FilenameFilter filter)`
    * 文件名过滤器
    * 自定义过滤条件

* 过滤器

  * `FileFilter` 接口
    * `accpet(File pathname)` 指明过滤条件
  * `FilenameFilter` 接口
    * `accept(File dir, String name)` 过滤条件
    * 指明当前文件名 name 是否 属于 dir 某个文件目录中

  ```Java
  // 判断文件类型
  		File[] files = file.listFiles(new FileFilter() {
              @Override
              public boolean accept(File pathname) {
                  return !pathname.isDirectory() && 															pathname.getName().endsWith(".java");
              }
          });
  ```

#### 应用

1. 批处理，重命名，移动文件
2. 打印目录结构
3. 自动化

```Java
// 文件及目录 广度优先遍历
 	public static void removeDirectory(File file) {
        File[] files = file.listFiles();
        // 广度优先遍历
        for (File f : files) {
            if (f.isDirectory()) {
                // 如果是目录则进入目录删除
                removeDirectory(f);
            }
            f.delete();
        }
        // 此处可以决定是否删除当前目录
        file.delete();
    }

// 文件及目录 深度优先遍历
	public static void removeDirectory2(File file) {
        // 如果是目录则进入目录删除
        if(file.isDirectory()) {
            removeDirectory(file);
        } else {
            File[] files = file.listFiles();
            for (File f : files) {
                f.delete();
            }
        }
    }
```

