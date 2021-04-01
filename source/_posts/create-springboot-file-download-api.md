---
title: SpringBoot 文件处理接口
copyright: true
date: 2019-05-24 15:48:57
tags:
categories:
---

# Spingboot 下载文件

## 下载文件

有时候会遇到一些需求，需要给前端提供下载文件的接口。

```java
    @GetMapping("/export")
    @ResponseBody
    public ResponseEntity<Resource> export() {
        HSSFWorkbook workbook = new HSSFWorkbook();
        ByteArrayOutputStream outputStream = new ByteArrayOutputStream()
        workbook.write(outputStream);
        Resource file = new ByteArrayResource(outputStream.toByteArray());
        String filename="app.txt";
        return ResponseEntity.ok().header(HttpHeaders.CONTENT_DISPOSITION,
                "attachment; filename="+filename).body(file);
    }
//
```

<!--more-->

可以看到我创建了一个 Resource 然后返回了这个 resource。感觉是不是超级简单。确实比较简单，不过如果这个接口在外层还有一个应用参数拦截层（输入参数和返回参数都json打印）的话，需要注意将参数序列化成json数据的时候，这里是会出异常。

上面的代码中就是流的转换了，让我想起了java中的io流装饰器模式。

在上面的代码中主要注意在返回的head里面需要加入`header(HttpHeaders.CONTENT_DISPOSITION,"attachment; filename="+filename)`;这里的filename就是下载后的文件名。


## 上传文件

```java

    @PostMapping("/file")
    @ResponseBody
    public String handleFileUpload(@RequestParam("file") MultipartFile file) {

        String originalFilename = file.getOriginalFilename();
        System.out.println(originalFilename);
        System.out.println(file.getName());
        try {
            String string = Base64.getEncoder().encodeToString(file.getBytes());
            return string;
        } catch (IOException e) {
            e.printStackTrace();
        }

        return "emmmmmmm. error";
    }
```

有下载当然有上传了， 上面就是将文件上传的操作。