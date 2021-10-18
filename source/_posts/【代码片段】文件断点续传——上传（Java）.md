---
title: 【代码片段】文件断点续传——上传（Java）
date: 2021-10-09 09:30:58
tags:
    - 代码片段
    - 断点续传
---

# java版
**依赖包**
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.5.5</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-autoconfigure</artifactId>
        <version>2.5.5</version>
    </dependency>
    <dependency>
        <groupId>commons-fileupload</groupId>
        <artifactId>commons-fileupload</artifactId>
        <version>1.3.3</version>
    </dependency>
    <dependency>
        <groupId>commons-io</groupId>
        <artifactId>commons-io</artifactId>
        <version>1.3.2</version>
    </dependency>
    <dependency>
        <groupId>org.apache.httpcomponents</groupId>
        <artifactId>httpcore</artifactId>
        <version>4.4.14</version>
    </dependency>
    <dependency>
        <groupId>org.apache.httpcomponents</groupId>
        <artifactId>httpclient</artifactId>
        <version>4.5.13</version>
    </dependency>
</dependencies>
```
## UploadController
```java
import org.apache.commons.fileupload.FileItem;
import org.apache.commons.fileupload.disk.DiskFileItemFactory;
import org.apache.commons.fileupload.servlet.ServletFileUpload;
import org.apache.commons.io.FileUtils;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.*;
import java.util.List;

@Controller
public class UploadController {
    private final static String utf8 = "utf-8";

    @RequestMapping("/upload")
    @ResponseBody
    public void upload(HttpServletRequest req, HttpServletResponse res) throws Exception {

        res.setCharacterEncoding(utf8);
        Integer schunk = null;
        Integer schunks = null;
        String name = null;
        String uploadPath = "F:\\fileItem";
        BufferedOutputStream os = null;
        try {
            DiskFileItemFactory factory = new DiskFileItemFactory();
            factory.setSizeThreshold(1024);
            factory.setRepository(new File(uploadPath));
            ServletFileUpload upload = new ServletFileUpload(factory);
            upload.setFileSizeMax(51 * 10241 * 10241 * 10241);
            upload.setSizeMax(101 * 10241 * 10241 * 10241);
            List<FileItem> items = upload.parseRequest(req);

            for (FileItem item : items) {
                if (item.isFormField()) {
                    if ("chunk".equals(item.getFieldName())) {
                        schunk = Integer.parseInt(item.getString(utf8));
                    }
                    if ("chunks".equals(item.getFieldName())) {
                        schunks = Integer.parseInt(item.getString(utf8));
                    }
                    if ("name".equals(item.getFieldName())) {
                        name = item.getString(utf8);
                    }
                }
            }

            for (FileItem item : items) {
                if (!item.isFormField()) {
                    String temFileName = name;
                    if (name != null) {
                        if (schunk != null) {
                            temFileName = schunk + "_" + name;
                        }
                        File temFile = new File(uploadPath, temFileName);
                        // 断点续传
                        if (!temFile.exists()) {
                            item.write(temFile);
                        }
                    }
                }
            }
            // 文件合并
            if (schunk != null && schunk.intValue() == schunks.intValue()-1) {
                File tempFile = new File(uploadPath, name);
                os = new BufferedOutputStream(new FileOutputStream(tempFile));
                for (int i = 0; i < schunks; i++) {
                    File file = new File(uploadPath, i + "_" + name);
                    while (!file.exists()) {
                        Thread.sleep(100);
                    }
                    byte[] bytes = FileUtils.readFileToByteArray(file);
                    os.write(bytes);
                    os.flush();
                    file.delete();
                }
                os.flush();
            }
            res.getWriter().write("上传成功");
        }finally {
            try {
                if (os != null) {
                    os.close();
                }
            }catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```
