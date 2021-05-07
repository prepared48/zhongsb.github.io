---
layout: post
title:  "Java读取Excel成List对象数组"
date:   2021-04-17 20:41:54
categories: JAVA JVM
tags: JVM JAVA
mathjax: true
---

* content
{:toc}

一个工具类





文件IO是任何软件进行的重要组成部分，我们在电脑上创建一个Excel文件，然后打开它修改一些东西或者删除它。Java给我们提供了操纵文件的很多工具类，本文主要是使用POI操纵Excel文件。

## 1 介绍POI包的一些概念

- Workbook: 这就是一个工作Excel文件。XSSFWorkbook 和 HSSFWorkbook classes都实现这个接口
- XSSFWorkbook: 针对 XLSX 类型文件的一个实现.
- HSSFWorkbook: 针对 XLS 类型文件的一个实现. 
- Sheet: 一个Excel的一个sheet页。XSSFSheet and HSSFSheet 都实现这个接口。
- XSSFSheet: Sheet页的一个针对 XLSX 类型文件的实现.
- HSSFSheet: Sheet页的一个针对 XLS 类型文件的实现.
- Row: 一个sheet页中的一行数据。
- Cell: 一个Excel最小框格，也就是确定行、确定列的一个框格。

## 2 具体实现

### 2.1 导入POM包

```
<!--POI 包-->
<dependency>
  <groupId>org.apache.poi</groupId>
  <artifactId>poi</artifactId>
  <version>${poi.versin}</version>
</dependency>
<dependency>
  <groupId>org.apache.poi</groupId>
  <artifactId>poi-ooxml</artifactId>
  <version>${poi.versin}</version>
</dependency>
```

### 2.2 根据文件流创建一个Workbook

可以使用`WorkbookFactory`自动根据Excel类型是XLSX还是XLS自动创建对应的Workbook

```
File file = new File(filePath + File.separator + fileName);
FileInputStream inputStream = new FileInputStream(file);
// 使用工厂模式 根据文件扩展名 创建对应的Workbook
Workbook workbook = WorkbookFactory.create(inputStream);
```

### 2.3 获取确定sheet页的数据

根据 sheet 页的名字获取 sheet 页数据

```
Sheet sheet = workbook.getSheet(sheetName);
```

### 2.4 获取总的数据行数

Sheet 类提供了获取首行行号和最后一行行号的方法，可以根据这两个方法获取 sheet 页中的数据行数。

```
int rowCount = sheet.getLastRowNum() - sheet.getFirstRowNum();
```

### 2.5 获取行列数据封装到Map中

如果只需要返回Map数据，到这里就可以返回结果

```
for (int i = 1; i < rowCount + 1; i++) {
    Row row = sheet.getRow(i);
    resultMap = new HashMap<>();
    for (int j = 0; j < row.getLastCellNum(); j++) {
        if(Objects.equals(row.getCell(j).getCellType(), CellType.STRING)) {
            resultMap.put(sheet.getRow(0).getCell(j).toString(), row.getCell(j).getStringCellValue());
          } else if(Objects.equals(row.getCell(j).getCellType(), CellType.NUMERIC)) {
                resultMap.put(sheet.getRow(0).getCell(j).toString(), row.getCell(j).getNumericCellValue());
          }else {
                resultMap.put(sheet.getRow(0).getCell(j).toString(), row.getCell(j));
                }
      }
      jsonObject = new JSONObject(resultMap);
      resultMapList.add(jsonObject.toJSONString());
}
```

### 2.6 将结果数据转成List对象

使用 `fasterxml.jackson` 将Map结果数据转成 List 对象

```
return JsonUtil.ofList(resultMapList.toString(), tClass);
```

### 代码如下

```
/**
 * 获取Excel，将数据转换成 List<T> 的形式
 * Excel 数据要求第一行为对象的属性名称
 *
 * @param filePath  文件路径
 * @param fileName  文件名称
 * @param sheetName sheet名称
 * @param tClass    要转换成的实体类
 * @param <T>
 * @return List对象数组
 * @throws IOException
 */
public static <T> List<T> readExcelOfList(String filePath, String fileName, String sheetName, Class<T> tClass) throws IOException {
    List<String> resultMapList = new ArrayList<>();
    File file = new File(filePath + File.separator + fileName);
    FileInputStream inputStream = new FileInputStream(file);
    // 使用工厂模式 根据文件扩展名 创建对应的Workbook
    Workbook workbook = WorkbookFactory.create(inputStream);
    Sheet sheet = workbook.getSheet(sheetName);
    int rowCount = sheet.getLastRowNum() - sheet.getFirstRowNum();
    JSONObject jsonObject;
    Map<String, Object> resultMap;
    for (int i = 1; i < rowCount + 1; i++) {
        Row row = sheet.getRow(i);
        resultMap = new HashMap<>();
        for (int j = 0; j < row.getLastCellNum(); j++) {
            if(Objects.equals(row.getCell(j).getCellType(), CellType.STRING)) {
                resultMap.put(sheet.getRow(0).getCell(j).toString(), row.getCell(j).getStringCellValue());
            } else if(Objects.equals(row.getCell(j).getCellType(), CellType.NUMERIC)) {
                resultMap.put(sheet.getRow(0).getCell(j).toString(), row.getCell(j).getNumericCellValue());
            }else {
                resultMap.put(sheet.getRow(0).getCell(j).toString(), row.getCell(j));
            }
        }
        jsonObject = new JSONObject(resultMap);
        resultMapList.add(jsonObject.toJSONString());
    }
    return JsonUtil.ofList(resultMapList.toString(), tClass);
}
```

源代码：https://github.com/prepared48/dataProcess-tools.git

参考链接：https://www.guru99.com/all-about-excel-in-selenium-poi-jxl.html