# Java File Upload

#### 内容摘要

文件上传要求

---

#### 文件上传要求

1. `<form method="post" enctype="multipart/form-data">`
   * 提交方式 **POST** 
   * 默认请求头 `content-Type : application / x-www-form-urlencoded`
     * 提交数据对于**每一项表单组件**以键值对 **Map<String, String> **的方式提交请求
     * 文件上传只能拿到文件名
   * `enctype="multipart/form-data"`
     * 提交请求对于每一项表单组件以**整体 POST 报文用分隔符分隔各个组件**，二进制流
     * 文件上传的是二进制流
2. `<input type="file" name="uploadFile">`
   * 需要 **name** 属性和 **file** 值

```html
<form method="POST" enctype="multipart/form-data" action="fileHandle">
  File to upload: <input type="file" name="upfile"><br/>
  Notes about the file: <input type="text" name="note"><br/>
  <br/>
  <input type="submit" value="Press"> to upload the file!
</form>
```

#### 服务端接收

> If the parameter data was sent in the request body, such as occurs with an HTTP POST request, then reading the body directly via `getInputStream()`or `getReader()` can interfere with the execution of this method.

* `request.getParameter(String name)` 会被 **POST** 二进制流影响
* 使用框架解析 **POST** 报文二进制流
  * `Commons-fileupload.jar`  依赖 `Commons-io`
* 流程说明
  1. 设置上传文件 base 路径，创建多级目录
  2. 初始化文件上传工厂
  3. 传入请求，拿到解析后的列表
  4. 遍历列表对简单表单数据和文件上传数据分别处理
     1. 简单表单数据
        1. `String name = item.getFieldName()`
        2. `String value = item.getString("utf-8")` // 如果有中文，要设定解码方式
     2. 上传文件
        1. 拿到文件名
        2. 拼接文件路径
        3. 写入

```java
// Commons-fileupload 使用方法
	private String fileBase;
    @Override
     public void init() throws ServletException {
        // 首先设置文件存储位置 fileBase
        fileBase = getServletContext().getRealPath("WEB-INF/uploadFiles");
        // 创建多级根目录
        new File(fileBase).mkdirs();
    }
	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 简单文件上传
        // 初始化文件上传工厂
        // Create a factory for disk-based file items
		DiskFileItemFactory factory = new DiskFileItemFactory();
		// Configure a repository (to ensure a secure temp location is used)
		ServletContext servletContext = this.getServletConfig().getServletContext();
		File repository = (File) servletContext.getAttribute("javax.servlet.context.tempdir");
		factory.setRepository(repository);
		// Create a new file upload handler
		ServletFileUpload upload = new ServletFileUpload(factory);
        // 接收上传文件列表
        List<FileItem> items;
        try {
            // 传入请求解析 POST 报文
            items = upload.parseRequest(request);
        } catch (FileUploadException e) {
            e.printStackTrace();
        }
  		// 处理请求参数
        Iterator<FileItem> iter = items.iterator();
        while (iter.hasNext()) {
            FileItem item = iter.next();
            // isFormField true： 简单表单组件数据，键值对
            if (item.isFormField()) {
                processFormField(item);
            } else {
                // false：上传的文件
                processUploadedFile(item);
            }
        }
    }
	private void processFormField(FileItem item) {
        // 处理简单表单组件数据，键值对
        // 返回原始文件名
        String name = item.getName();
        System.out.println("name = " + name);
        // 返回表单组件 name 名
        String fieldName = item.getFieldName();
        System.out.println("fieldName = " + fieldName);
        String string = null;
        try {
            // 拿到的 post 表单数据默认解码方式为 iso-8859-1
            // 中文会出现乱码问题，要在此处指定解码字符集
            // 返回表单组件 name 的值
            string = item.getString("utf-8");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        System.out.println("string = " + string);
    }
	private void processUploadedFile(FileItem item) {
        // 处理上传文件数据
        // 拿到文件名
        String fileName = item.getName();
        // 将文件存在新 fileBase 目录下
        File file = new File(fileBase, fileName);
        // 写入文件
        try {
            item.write(file);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

#### 文件上传工程问题

1. 上传文件名冲突
2. 搜索用户上传的文件，避免多个文件在同一文件夹
3. 文件内容相同的文件重复上传

#### 解决方案

1. **UUID**

   * `java.util.UUID`


   * `UUID.randomUUID()`  生成一个当前 JVM 中唯一的随机数，128 bit 十六进制
   * 解决文件重名问题

```Java
	private void processUploadedFile(FileItem item) {
        // 1. 避免重复文件名
        // 使用 UUID 唯一标示文件名
        // 拿到 UUID 128 bit
        UUID uuid = UUID.randomUUID();
        // 拼接到文件名前面
        String fileName = uuid + item.getName();
        // 写入文件
        File file = new File(fileBase, fileName);
        try {
            item.write(file);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

2. 将用户上传的文件上传到不同的文件夹中

   * 优势：搜索最多 8 次，目录树深度为 8，可以创建 16^8^ 个目录，方便搜索和管理

   1. 拿到 UUID 前缀的文件的的 hash 值
   2. 将该 hash 值转化为十六进制，8 bit 十六进制
   3. 根据每一个位十六进制数值，构造目录结构 `4542b6ec => /4/5/4/2/b/6/e/c` 
   4. 创建目录，写入文件

```Java
	private void processUploadedFile(FileItem item) {
        UUID uuid = UUID.randomUUID();
        String fileName = uuid + item.getName();
        // 2. 为不同文件创建不同目录
        // a. 拿到 fileName 的 hash 值
        int code = fileName.hashCode();
        // b. 将 code 转化为十六进制 8 bit
        String hexString = Integer.toHexString(code);
        // c. 根据每一位十六进制数值，构造目录结构
        String fileDir = getFileDir(hexString);
        // d. 创建目录
        File directory = new File(fileBase, fileDir);
        directory.mkdirs();
        // 比如 hexString: 4542b6ec
        // directory: /Users/clixin/Codes/IntelliJ-IDEA-workspace/JavaWeb/out/artifacts
        //              /Day46__upload/WEB-INF/uploadFiles/4/5/4/2/b/6/e/c
        // 写入文件
        File file = new File(directory, fileName);
        try {
            item.write(file);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
	private String getFileDir(String hexString) {
        StringBuilder sb = new StringBuilder();
        char[] chars = hexString.toCharArray();
        for (char c : chars) {
            sb.append('/').append(c);
        }
        return sb.toString();
    }
```

3. 文件内容验证
   * MD5 验证：散列函数，对每一个文件计算出唯一长度为 128 bit 的二进制数据，“数字指纹”
   * 文件修改一丁点，其 MD5 都会大有不同，以保证文件一致性
   * 不是加密算法，是 hash 算法

#### 文件下载

* 响应头 `content-type` 如果浏览器可以解析，就解析，如果不能解析将会直接下载
* `reponse.setHeader("Content-Disposition", "attachment;filename=test.txt")`
* `response.getOutputStream` 写回响应报文
* <_不能使用包装流？>

```Java
		String path = "/WEB-INF/uploadFiles/4/5/4/2/b/6/e/c" +
                "/77055cc0-aba7-4d95-ae6e-72129f8268e4normalize.css";
        String fileName = "normalize.css";
        // 1. 设置响应头部
        response.setHeader("Content-Disposition", "attachment;filename=" + fileName);
        // 2. 从服务器拿到该文件
        path = getServletContext().getRealPath(path);
        FileInputStream in = new FileInputStream(path);
        // 3. 拿到响应报文流
        ServletOutputStream outputStream = response.getOutputStream();
        // 4. 写入到响应报文流
        byte[] buffer = new byte[2048];
        int len;
        while ((len = in.read(buffer)) != -1) {
            outputStream.write(buffer, 0, len);
        }
        in.close();
```

#### 下载文件名中文乱码问题

1. 中文名 `String fileName = "中文名.txt" `
2. 先将中文名编码为 **UTF-8** `Bytes[] bytes = fileName.getBytes("utf-8")`
3. 再将中文名以 **ISO-8859-1** 解码 `fileName = new String(bytes, "iso-8859-1")`
4. 写回响应头 `reponse.setHeader("Content-Disposition", "attachment;filename=" + fileName)`

```Java
 		String path = "中文名.txt";
        String fileName = "中文名.txt";
        // 使用 utf-8 编码
        byte[] bytes = fileName.getBytes("utf-8");
        // 使用 iso-8859-1 解码
        fileName = new String(bytes, "iso-8859-1");
        // 1. 设置响应头部
        response.setHeader("Content-Disposition", "attachment;filename=" + fileName);
        // 2. 从服务器拿到该文件
        
        // 3. 拿到响应报文流

        // 4. 写入到响应报文流
```

#### 原理分析

* 响应报文发回浏览器时由 Tomcat 编码 ，为 `iso-8859-1`
* 浏览器拿到响应报文，按照约定解码方式（即 `iso-8859-1`），解码响应报文
* 若直接将中文名编码成 `iso-8859-1` 
  * 某些找不到的字符会被替换成 `?`
  * 浏览器再解码 `iso-8859-1`  ，会导致解码出来的字符与编码前不一致
* 若先将中文名编码成 `utf-8`，再以 `iso-8859-1` 解码
  * 中文在 `utf-8` 中有对应的字符
  * `utf-8` 中有 `iso-8859-1` 对应的字符，**不会导致字符丢失**
  * 浏览器再解码 `iso-8859-1` ，得到 `utf-8` 编码的中文，不会显示乱码

