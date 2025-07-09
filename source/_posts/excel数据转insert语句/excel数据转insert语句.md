---
abbrlink: ''
categories:
- - 开发心得
date: '2025-07-08T09:47:28.056069+08:00'
description: 一个vb脚本，可以将excel数据转为insert语句
keywords: excel，sql
tags:
- excel
- sql
- 开发心得
title: 将excle数据转化为INSERT语句
updated: '2025-07-08T10:25:49.669+08:00'
---
# 将excle数据转化为INSERT语句

修改tableName为完整表名即可，支持mysql，要求第一行为表字段名称，后续列为数据

如果sheet页名字不为Sheet1,修改`Set ws = ThisWorkbook.Sheets("Sheet1")`中Sheets中参数

```basic
Sub GenerateMultiRowInsert()
    Dim ws As Worksheet
    Dim tableName As String
    Dim lastRow As Long, lastCol As Integer
    Dim headers() As String
    Dim i As Long, j As Integer
    Dim sql As String, valueStr As String
    Dim fieldList As String
    Dim outputCol As Integer
    Dim dataValue As String
  
    ' 设置MySQL表名
    tableName = "db_name.tb_name" ' 修改为你的表名
  
    ' 获取工作表
    Set ws = ThisWorkbook.Sheets("Sheet1")
  
    ' 获取数据范围
    lastRow = ws.Cells(ws.Rows.Count, 1).End(xlUp).Row
    lastCol = ws.Cells(1, ws.Columns.Count).End(xlToLeft).Column
  
    ' 确定输出列（最后一列+1）
    outputCol = lastCol + 1
  
    ' 设置输出列标题
    ws.Cells(1, outputCol).Value = "INSERT Part"
    ws.Cells(1, outputCol).EntireColumn.AutoFit
  
    ' 读取表头
    ReDim headers(1 To lastCol)
    For j = 1 To lastCol
        headers(j) = ws.Cells(1, j).Value
    Next j
  
    ' 构建字段列表
    fieldList = ""
    For j = 1 To lastCol
        fieldList = fieldList & IIf(j > 1, ", ", "") & "`" & headers(j) & "`"
    Next j
  
    ' 处理每一行数据
    For i = 2 To lastRow
        ' 构建该行数据值的字符串
        valueStr = ""
        For j = 1 To lastCol
            ' 获取单元格值
            dataValue = CStr(ws.Cells(i, j).Value)
  
            ' 处理特殊字符：移除运算符前的反引号（如果存在）
            If Left(dataValue, 1) = "`" And Len(dataValue) > 1 Then
                dataValue = Mid(dataValue, 2)
            End If
  
            ' 处理特殊值和数据类型
            If UCase(dataValue) = "CURRENT_TIMESTAMP" Then
                valueStr = valueStr & IIf(j > 1, ", ", "") & "CURRENT_TIMESTAMP"
            ElseIf IsNumeric(dataValue) And Not IsDate(dataValue) Then
                valueStr = valueStr & IIf(j > 1, ", ", "") & dataValue
            ElseIf IsDate(dataValue) Then
                valueStr = valueStr & IIf(j > 1, ", ", "") & "'" & Format(dataValue, "yyyy-mm-dd hh:nn:ss") & "'"
            ElseIf IsEmpty(dataValue) Or dataValue = "" Then
                valueStr = valueStr & IIf(j > 1, ", ", "") & "NULL"
            Else
                ' 转义单引号
                valueStr = valueStr & IIf(j > 1, ", ", "") & "'" & Replace(dataValue, "'", "''") & "'"
            End If
        Next j
  
        ' 根据行号生成不同的部分
        If i = 2 Then
            ' 第一行数据：完整的INSERT开头
            sql = "INSERT INTO " & tableName & " (" & fieldList & ") VALUES (" & valueStr & ")"
        Else
            ' 后续行：以逗号开头
            sql = ",(" & valueStr & ")"
        End If
  
        ' 如果是最后一行，添加分号
        If i = lastRow Then
            sql = sql & ";"
        End If
  
        ' 写入当前行的输出列
        ws.Cells(i, outputCol).Value = sql
    Next i
  
    ' 自动调整列宽
    ws.Columns(outputCol).AutoFit
  
    MsgBox "生成完成！" & vbCrLf & _
           "请复制第" & outputCol & "列（从第2行到第" & lastRow & "行）的内容，得到完整的INSERT语句。", vbInformation
End Sub




```
