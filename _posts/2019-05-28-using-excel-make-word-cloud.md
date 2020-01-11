---
layout: post
title: 利用 Excel 制作词云
categories: Data-Visualization
keywords: Excel, Word Cloud
---

通过将出现频率高的「关键词」即高频关键词突出呈现，过滤掉大量出现频率低或无实意的词语，可以很直观的将主旨呈现给读者。既可以通过对文本进行分词后统计词频，也可以直接使用一些规整的词直接统计 ~ 如使用论文关键词做词云。此处使用 Excel 对中国知网中「熊猫」相关主题近十年来的论文关键词进行统计，再使用 [Word Art](https://wordart.com/) 进行词云制作。

## 1 关键词下载

通过在 [中国知网](https://www.cnki.net/) 检索以「熊猫」为主题的文献，在检索结果页面中勾选文献，再通过知网提供的「导出/参考文献」功能导出包含文献「关键词」的元数据。

**Tips**: 知网中一次最多可导出的数据为 500 条；导出数据时需要通过「自定义」勾选「Keyword-关键词」及其它需要的数据项。

导出时，为后续处理方便，直接选择 **xls** 格式。

## 2 词频统计

导出文献关键词后：

- 首先对数据进行编号，然后新建一个工作表将「编号」及「Keyword-关键词」列复制工作表中。

### 2.1 探索式词频统计

- 使用 `;` 对「Keyword-关键词」列进行分列，将不同的关键词拆分到不同单元格。
- 然后整体观察数据时发现，知网中导出的关键词存在以 `[",", "，", "；"]` 为分隔符的情况：
  - 一种解决方法是在关键词列前插入一列 ~ 筛选列，通过 `IFERROR(FIND(",",C2),0)` 函数找出以 `,` 为分隔符的数据行，通过**过滤器**将「筛选列」中不为零的值筛选出来，将这些行在新表中以 `,` 进行分类，最后再合并到同一个表中即可。其他分隔符的处理方法类似。

此时，各关键词零散的散布在表中的单元格中，难以进行数据统计，我们借助于一个简单的 VBA 程序将关键词合并到同一列：

```vb
Sub illustratekw()

    Dim i As Integer, j As Integer, k As Integer, kw As Variant
    k = 2

    ' 演示时仅以导出的前 500 条数据为例，将需要处理的数据放置在 Sheet2 的 A, B 列
    ' 处理后的结果放置在 Sheet3 的 A, B 列
    For i = 2 To 501
        For j = 2 To 50
            ' 去除关键词前后空格及统一英文字母的格式
            kw = WorksheetFunction.Proper(Trim(Sheet2.Cells(i, j)))
            If kw <> "" Then
                Sheet3.Cells(k, 1) = Sheet2.Cells(i, 1)
                Sheet3.Cells(k, 2) = kw
                k = k + 1
            End If
        Next j
    Next i

End Sub
```

最后，对于已经归并在同一列中的关键词，使用 「插入—>数据透视表」功能即可方便的统计关键词词频。

### 2.2 纯 VBA 方式

当然，上述词频统计的过程我们可以直接在一个 VBA 程序中直接完成：

```vb
Sub illustratekwall()

    Dim i As Integer, j As Integer, keyword As Variant, delimiter As Variant, tmpstr As String
    Dim keywords() As String, delimiters() As String
    Dim dict As New Scripting.Dictionary

    j = 2
    delimiters = Split("； ， , 《 》")

    For i = 2 To 501
        tmpstr = Sheet2.Cells(i, 2)

        ' 使用 ";" 替换其它分隔符
        For Each delimiter In delimiters
            tmpstr = Replace(tmpstr, delimiter, ";")
        Next delimiter

        ' 使用 ";" 拆分关键词
        keywords = Split(tmpstr, ";")

        ' 将关键词及其词频放入到字典中
        For Each keyword In keywords
            ' 关键词首字母大写（英文）并去除关键词首尾空格
            keyword = WorksheetFunction.Proper(Trim(keyword))
            If keyword = "" Then
                'pass
            Else
                If dict.Exists(keyword) Then
                    dict(keyword) = dict(keyword) + 1
                Else
                    dict.Add Key:=keyword, Item:=1
                End If
            End If
        Next keyword
    Next i

    ' 将统计好词频的字典输出到 Sheet3
    For i = 0 To dict.Count - 1
        Sheet3.Cells(j, 1) = j - 1
        Sheet3.Cells(j, 2) = dict.Keys(i)
        Sheet3.Cells(j, 3) = dict.Items(i)
        j = j + 1
    Next i

End Sub
```

**Tips**:

- 宏（VBA 程序）需要在「启用宏」的 Excel 文件中使用 `*.xlsm`；
- VBA 程序需要在「Visual Basic 编辑器」中撰写，通过按 <kbd>Alt</kbd>+<kbd>F11</kbd> 进入；
- 此外，为了更好地呈现效果，可能需要在最后人工合并「意义相同」的关键词；
- 为使用 `Scripting.Dictionary`，需要「Visual Basic 编辑器」中从「工具—>宏」中勾选 `Microsoft Scripting Runtime` 库。

## 3 词云制作

通过[词频统计](#词频统计)我们得到了关键词及其词频，此时我们需要借助于第三方的词云进行词云制作：

- WordArt: <https://wordart.com/>.
- Word Cloud Generator: <https://www.jasondavies.com/>.
- WordClouds: <https://www.wordclouds.com/>.

**具体的制作步骤为**：

1. 对于已统计好词频的关键词，按关键词词频「降序排列」，选取前 200 个左右关键词；
2. 勾选「CSV format」后将选取的关键词导入 [WordArt](https://wordart.com/create)；
3. 从「SHAPES」中选取自己喜欢的图案或自行上传图片；
4. 挑选字体 ~ 对于中文而言，可能需要添加本地字体；
5. 对布局及样式进行调整并点击「Visualize」按钮即可生成相应词云。

此时，可爱的「熊猫词云」已经出炉啦，快来看看吧：

![pandas word cloud](/assets/images/posts/data-visualization/pandas.jpeg)

## 4 参考资料

1. [Excel Easy](https://www.excel-easy.com/), Niels Weterings, [VBA](https://www.excel-easy.com/vba.html), 2019/05/28.
2. [EXCEL MACRO MASTERY](https://excelmacromastery.com/), PAUL KELLY, [Excel VBA Dictionary – A Complete Guide](https://excelmacromastery.com/vba-dictionary/), 2019/05/28.
3. [Microsoft](https://www.microsoft.com/), olprod, [Excel VBA 参考](https://docs.microsoft.com/zh-cn/office/vba/api/overview/excel), 2019/05/28.
