# Javaweb文件上传至服务器/从服务器下载

### 思路图

![image-20230412143522611](https://s2.loli.net/2023/04/12/QqPvS1aIoLiDY2N.png)

![image-20230412143543516](https://s2.loli.net/2023/04/12/BCkeUZtPqOgVL1w.png)

### 文件上传思路:

**也可以直接看代码**

1.  判断是不是文件表单(判断form的enctype是不是="multipart/form-data")，因为只有文件表单才能上传文件
2. 创建 DiskFileItemFactory 对象, 用于构建一个解析上传数据的工具对象
3. 创建一个解析上传数据的工具对象servletFileUpload
4. 关键的地方, servletFileUpload 对象可以把表单提交的数据 text / 文件，将其封装到为 FileItem 类型的List中，然后就可以遍历，并分别处理表单中的每一个提交的数据
5. 判断数据是一个text还是文件然后分别进行处理
6. 如果是一个文件，就把这个上传到 服务器的 temp 下的文件**保存到你指定的目录**
7. 指定一个在我们网站工作目录下的目录，（相对于当前 Web 应用程序的虚拟路径）
8.  获取到指定的虚拟目录的物理实际路径，因为我们上传文件不可能是在一个虚拟的地方
9. 创建这个指定的目录
10. 将文件拷贝到该目录

以下是完整代码：放在servlet的dopost/get函数里，为了美观已省略try-catch，为了保证代码美观最好请用电脑全屏观看

```java
//1. 判断是不是文件表单(判断form的enctype是不是="multipart/form-data")，因为只有文件表单才能上传文件
if (ServletFileUpload.isMultipartContent(request)) {
            //2. 创建 DiskFileItemFactory 对象, 用于构建一个解析上传数据的工具对象
            DiskFileItemFactory diskFileItemFactory = new DiskFileItemFactory();
            //3. 创建一个解析上传数据的工具对象
            ServletFileUpload servletFileUpload =
                    new ServletFileUpload(diskFileItemFactory);
            //解决接收到文件名是中文乱码问题
            servletFileUpload.setHeaderEncoding("utf-8");
            //4. 关键的地方, servletFileUpload 对象可以把表单提交的数据text / 文件,将其封装到 FileItem 文件项中
                List<FileItem> list = servletFileUpload.parseRequest(request);
                //遍历，并分别处理
                for (FileItem fileItem : list) {
                    //判断是不是一个文件
                    if (fileItem.isFormField()) {					//如果是true就是文本 input text
                        String name = fileItem.getString("utf-8");    //想要获取input text要通过getString方法获取到
                        System.out.println("家具名=" + name);
                    } else {				//是一个文件
                        //获取上传的文件的名字
                        String name = fileItem.getName();
                        System.out.println("上传的文件名=" + name);

                        //把这个上传到 服务器的 temp下 的文件保存到你指定的目录

                        //1.指定一个目录 , 但是这个目录是相对于这个web应用的路径，相当于以xxx_war_exploded这个目录为参照
                        //写一个相对路径表示你要创建的目录
                        String filePath = "/upload/";       //最后一个斜杠表示这个路径指向一个目录，而不是一个文件
                        //2. 获取到指定的虚拟目录的物理路径，request.getServletContext()获取到一个ServletContext，然后它可以调用					                         getRealPath(filePath),将filePath转换为物理的实际路径
                        //因为我们知道我们的web应用其实一直是一个放在设定好的application context目录下的一个虚拟路径里。
                        //所以我们要调用getRealPath()将其转换成一个物理上的真实路径，这样才能真实的保存一个文件。
                        //一句话概括这个过程发生了什么呢，其实就是以xxx_war_exploded这个web应用目录为基础，在上一级目录下建一个文件夹叫upload
                        
                        String fileRealPath = request.getServletContext().getRealPath(filePath);
					  // fileRealPath=E:fileupdown\out\artifacts\fileupdown_war_exploded\xupload\

                        //3. 创建fileRealPath这个已指定好的用于上传的实际目录
                        File fileRealPathDirectory = new File(fileRealPath);
                        // fileRealPathDirectory=E:fileupdown\out\artifacts\fileupdown_war_exploded\xupload\2023\4\12
                        if (!fileRealPathDirectory.exists()) {		//不存在，就创建
                            fileRealPathDirectory.mkdirs();			//创建
                        }
                        
                        //4. 将文件拷贝到fileRealPathDirectory目录
                        //   构建一个上传文件的完整路径 ：目录+文件名
                        //   对上传的文件名进行处理, 前面增加一个前缀，保证是唯一即可
                        name = UUID.randomUUID().toString() + "_" +System.currentTimeMillis() + "_" + name;
                        String fileFullPath = fileRealPathDirectory + "/" +name;
 //fileFullPath=E:\JavaProject\hspedu_javaweb\fileupdown\out\artifacts\fileupdown_war_exploded\upload\yyyy\mm\dd/name
                        fileItem.write(new File(fileFullPath));
                        
                        //5. 提示信息
                        response.setContentType("text/html;charset=utf-8");
                        response.getWriter().write("上传成功~");
                    }
                }
        } else {
            System.out.println("不是文件表单...");
        }
```

### 文件下载思路：

1. 先准备要下载的文件[假定这些文件是公共的资源]，

   重要: 保证当我们的tomcat启动后，在工作目录out下有download文件夹, 并且有可供下载的文件!!

2. 获取到要下载的文件的名字

3. 给http响应设置响应头文件格式 Content-Type，就是文件的MIME

4. 给http响应设置响应头 Content-Disposition来指定下载的数据的展示形式，attachment 则使用文件下载方式，复制以下代码即可

   ```java
   		//(1)如果是Firefox 则中文编码需要 base64
           //(2)Content-Disposition 是指定下载的数据的展示形式 , 如果attachment 则使用文件下载方式
           //(3)如果是其他(主流ie/chrome) 中文编码使用URL编码
           if (request.getHeader("User-Agent").contains("Firefox")) {
               // 火狐 Base64编码
               response.setHeader("Content-Disposition", "attachment; filename==?UTF-8?B?" +
                       new BASE64Encoder().encode(downLoadFileName.getBytes("UTF-8")) + "?=");
           } else {
               // 其他(主流ie/chrome)使用URL编码操作
               response.setHeader("Content-Disposition", "attachment; filename=" +
                       URLEncoder.encode(downLoadFileName, "UTF-8"));
           }
   ```

5. 读取下载的文件数据，返回给客户端/浏览器 

接下来走一下代码：

```java
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1. 先准备要下载的文件[假定这些文件是公共的资源]
        //   重要: 保证当我们的tomcat启动后，在工作目录out下有download文件夹, 并且有可供下载的文件!!
        //   如果你没有看到你创建的download在工作目录out下 rebuild project -> restart, 就OK

        //2. 获取到要下载的文件的名字
        request.setCharacterEncoding("utf-8");
        String downLoadFileName = request.getParameter("name");
        //System.out.println("downLoadFileName= " + downLoadFileName);

        //3. 给http响应，设置响应头文件格式 Content-Type , 就是文件的MIME
        //   通过servletContext 来获取
        ServletContext servletContext = request.getServletContext();
        String downLoadPath = "/download/"; //下载目录从 web工程根目录计算
        String downLoadFileFullPath = downLoadPath + downLoadFileName;   //  /download/1.jpg
        String mimeType = servletContext.getMimeType(downLoadFileFullPath);
        System.out.println("mimeType= " + mimeType);
        response.setContentType(mimeType);

        //4. 给http响应，设置响应头 Content-Disposition
        //(1)如果是Firefox 则中文编码需要 base64
        //(2)Content-Disposition 是指定下载的数据的展示形式 , 如果attachment 则使用文件下载方式
        //(3)如果是其他(主流ie/chrome) 中文编码使用URL编码
        if (request.getHeader("User-Agent").contains("Firefox")) {
            // 火狐 Base64编码
            response.setHeader("Content-Disposition", "attachment; filename==?UTF-8?B?" +
                    new BASE64Encoder().encode(downLoadFileName.getBytes("UTF-8")) + "?=");
        } else {
            // 其他(主流ie/chrome)使用URL编码操作
            response.setHeader("Content-Disposition", "attachment; filename=" +
                    URLEncoder.encode(downLoadFileName, "UTF-8"));
        }

        //5. 读取下载的文件数据，返回给客户端/浏览器
        //(1) 创建一个和要下载的文件，关联的输入流
        InputStream resourceAsStream = servletContext.getResourceAsStream(downLoadFileFullPath);
        //(2) 得到返回数据的输出流 [因为返回文件大多数是二进制(字节), IO java基础]
        ServletOutputStream outputStream = response.getOutputStream();
        //(3) 使用工具类，将输入流关联的文件，对拷到输出流，并返回给客户端/浏览器
        IOUtils.copy(resourceAsStream, outputStream);
    }
```

