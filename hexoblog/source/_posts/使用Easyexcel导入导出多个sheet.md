---
title: 使用Easyexcel导入导出多个sheet
img: /images/Easyexcel.jpg
categories: java
tags: Easyexcel
abbrlink: 50078
date: 2022-11-05 22:34:20
---

**EasyExcel对于导入导出的操作十分简洁**

**对于多个sheet且内容不一致的导入导出**

## 1、引入 [easyExcel](https://so.csdn.net/so/search?q=easyExcel&spm=1001.2101.3001.7020)依赖

```properties
<dependency>
<groupId>com.alibaba</groupId>
<artifactId>easyexcel</artifactId>
<version>3.0.1</version>
</dependency>
```

## 2、导出下载

提示：其中部分代码操作Dao层可以删除，可以自己创建ExportUserExcel 对象进行测试，思路数据映射到excel中。这里ExportUserExcel .class 映射的模板替换下面代码中

```java
@AllArgsConstructor
@NoArgsConstructorUser
@Builder
@HeadRowHeight(value = 20)
public class ExportUserExcel {

@ExcelProperty(value = "姓名",index = 0)
@ColumnWidth(value = 10)
private String userName;

@ExcelProperty(value = "年龄",index = 1)
@ColumnWidth(value = 20)
private String age;

@ExcelProperty(value = "性别",index = 2)
@ColumnWidth(value = 20)
private String gender;
}
```

导出下载 ExportUserExcel.class 模板

```java
public void modelExport(HttpServletResponse response, Long id) {
    response.setContentType("application/vnd.ms-excel");
    response.setCharacterEncoding("utf-8");
    // 这里URLEncoder.encode可以防止中文乱码
    try {
        String fileName = URLEncoder.encode("Physical", "UTF-8");
        response.setHeader("Content-disposition", "attachment;filename=" + fileName + ".xlsx");
        //新建ExcelWriter
        ExcelWriter excelWriter = EasyExcel.write(response.getOutputStream()).build();
        //获取sheet0对象
        WriteSheet mainSheet = EasyExcel.writerSheet(0, "模型信息").head(ExportUserExcel.class).build();
        //获取模型信息,向sheet0写入数据
        DmmPhysicalModel dmmPhysicalModel = dmmPhysicalModelMapper.selectById(id);
        List<DmmPhysicalModel> dmmPhysicalModels1 = Arrays.asList(dmmPhysicalModel);
        excelWriter.write(dmmPhysicalModels1, mainSheet);
        //获取sheet1对象
        WriteSheet detailSheet = EasyExcel.writerSheet(1, "词条信息").head(ExportUserExcel.class).build();
        HashMap entryHashMap = new HashMap<>();
        entryHashMap.put("model_id", id);
        //获取词条信息,向sheet1写入数据
        List<DmmPhysicalEntry> list = dmmPhysicalEntryMapper.selectByMap(entryHashMap);
        excelWriter.write(list, detailSheet);
        //关闭流
        excelWriter.finish();
    } catch (IOException e) {
        log.error("导出异常{}", e.getMessage());
    }
}
```

## 3、 导入excel 存入数据库

注意：一定要和excel表标题内容一致 ，
@ExcelProperty(value = “标题名称”)

```java
@Data
public class ModelImportVO{

@ExcelProperty(value = "标题名称")
private String title;

@ExcelProperty(value = "标题名称")
private String merchant_no;

@ExcelProperty(value = "标题名称")
private String terminal_id;

@ExcelProperty(value = "标题名称")
private String token;
```

一个监视器读取excel的类

```java
public class ExcelListener extends AnalysisEventListener {
    //可以通过实例获取该值
    private List<Object> datas = new ArrayList<Object>();
    public void invoke(Object o, AnalysisContext analysisContext) {
        datas.add(o);
        doSomething(o);
    }

    private void doSomething(Object object) {
    }

    public List<Object> getDatas() {
        return datas;
    }

    public void setDatas(List<Object> datas) {
        this.datas = datas;
    }

    public void doAfterAllAnalysed(AnalysisContext analysisContext) {
    }
}
```

## 4、excel导入到数据库 多个sheet

ModelImportVO.class这也是模板，自己可以定义，对应的值

```java
 @PostMapping("test/excel/import")
 public void modelImport(MultipartFile serviceFile) throws IOException {
        //输入流
        InputStream inputStream = serviceFile.getInputStream();
        //监视器
        ExcelListener listener = new ExcelListener();
        ExcelReader excelReader = EasyExcel.read(inputStream, listener).build();
        // 第一个sheet读取类型
        ReadSheet readSheet1 = EasyExcel.readSheet(0).head(ModelImportVO.class).build();
        // 第二个sheet读取类型
        ReadSheet readSheet2 = EasyExcel.readSheet(1).head(EntryImportVO.class).build();
        // 开始读取第一个sheet
        excelReader.read(readSheet1);
        //excel sheet0 信息
        List<Object> list = listener.getDatas();
      	//List<object> 转 List<实体类>
		 List<JsonRootBean> dtoList = new ArrayList<>();
        //List object for 转换 实体类
        for (Object objects : list) {
            JsonRootBean dto = (JsonRootBean) objects;
            dtoList.add(dto);
        }
        //List 转JOSN
        String json = JSON.toJSONString(dtoList);
        System.out.println("json = " + json);


        // 清空之前的数据
        listener.getDatas().clear();
        // 开始读取第二个sheet
        excelReader.read(readSheet2);
        //excel sheet1 信息
        List<Object> entry = listener.getDatas();
   		//copy上面作法
       
    }
```

补充获取sheet名称 长度等信息方法
//获取从第二个sheet读取类型 词条多个sheet
//监视器
ExcelListener listener = new ExcelListener();
ExcelReader excelReader = EasyExcel.read(inputStream, listener).build();
//弃用方法 List sheets = excelReader.getSheets();
// 新方法
List sheets = excelReader.excelExecutor().sheetList();

## 5、常用excel导入 数据转JSON 格式

```java
@PostMapping("test/excel/import")
public void modelImport(MultipartFile serviceFile) throws IOException {
    //输入流
    InputStream inputStream = serviceFile.getInputStream();
    //监视器
    ExcelListener listener = new ExcelListener();
    ExcelReader excelReader = EasyExcel.read(inputStream, listener).build();
    // 第一个sheet读取类型
    ReadSheet readSheet1 = EasyExcel.readSheet(0).head(ModelImportVO.class).build();
    // 开始读取第一个sheet
    excelReader.read(readSheet1);
    //excel sheet0 信息
    List<Object> list = listener.getDatas();
    //List<object> 转 List<实体类>
    List<ModelImportVO> dtoList = new ArrayList<>();
    //List object for 转换 实体类
    for (Object objects : list) {
        ModelImportVO dto = (ModelImportVO) objects;
        dtoList.add(dto);
    }

    List<ModelImportVO> ten = new ArrayList<>();
    //每个二十个输出一次
    for (ModelImportVO modelImportVO : dtoList) {
        if (ten.size() == 20) {
            System.out.println( JSON.toJSONString(ten));
            ten.clear();
        }
        ten.add(modelImportVO);
    }
    //最后不足二十输出一次
    System.out.println(JSON.toJSONString(ten));
}
```