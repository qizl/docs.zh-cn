---
title: "如何：按扩展名对文件进行分组 (LINQ) (C#) | Microsoft 文档"
ms.custom: 
ms.date: 2015-07-20
ms.prod: .net
ms.reviewer: 
ms.suite: 
ms.technology:
- devlang-csharp
ms.topic: article
dev_langs:
- CSharp
ms.assetid: 21a98320-a5a1-4981-82d8-6a637e7d9018
caps.latest.revision: 3
author: BillWagner
ms.author: wiwagn
translation.priority.mt:
- cs-cz
- pl-pl
- pt-br
- tr-tr
translationtype: Human Translation
ms.sourcegitcommit: a06bd2a17f1d6c7308fa6337c866c1ca2e7281c0
ms.openlocfilehash: 67b3672ff7fa6c4d188c18f246f5e564ec3b0b29
ms.lasthandoff: 03/13/2017

---
# <a name="how-to-group-files-by-extension-linq-c"></a>如何：按扩展名对文件进行分组 (LINQ) (C#)
本示例演示如何使用 LINQ 来执行高级分组和对文件或文件夹列表执行排序操作。 它还演示如何使用 <xref:System.Linq.Enumerable.Skip%2A> 和 <xref:System.Linq.Enumerable.Take%2A> 方法在控制台窗口中对输出进行分页。  
  
## <a name="example"></a>示例  
 下面的查询演示如何按文件扩展名对指定的目录树的内容进行分组。  
  
```csharp  
class GroupByExtension  
{  
    // This query will sort all the files under the specified folder  
    //  and subfolder into groups keyed by the file extension.  
    private static void Main()  
    {  
        // Take a snapshot of the file system.  
        string startFolder = @"c:\program files\Microsoft Visual Studio 9.0\Common7";  
  
        // Used in WriteLine to trim output lines.  
        int trimLength = startFolder.Length;  
  
        // Take a snapshot of the file system.  
        System.IO.DirectoryInfo dir = new System.IO.DirectoryInfo(startFolder);  
  
        // This method assumes that the application has discovery permissions  
        // for all folders under the specified path.  
        IEnumerable<System.IO.FileInfo> fileList = dir.GetFiles("*.*", System.IO.SearchOption.AllDirectories);  
  
        // Create the query.  
        var queryGroupByExt =  
            from file in fileList  
            group file by file.Extension.ToLower() into fileGroup  
            orderby fileGroup.Key  
            select fileGroup;  
  
        // Display one group at a time. If the number of   
        // entries is greater than the number of lines  
        // in the console window, then page the output.  
        PageOutput(trimLength, queryGroupByExt);  
    }  
  
    // This method specifically handles group queries of FileInfo objects with string keys.  
    // It can be modified to work for any long listings of data. Note that explicit typing  
    // must be used in method signatures. The groupbyExtList parameter is a query that produces  
    // groups of FileInfo objects with string keys.  
    private static void PageOutput(int rootLength,  
                                    IEnumerable<System.Linq.IGrouping<string, System.IO.FileInfo>> groupByExtList)  
    {  
        // Flag to break out of paging loop.  
        bool goAgain = true;  
  
        // "3" = 1 line for extension + 1 for "Press any key" + 1 for input cursor.  
        int numLines = Console.WindowHeight - 3;  
  
        // Iterate through the outer collection of groups.  
        foreach (var filegroup in groupByExtList)  
        {  
            // Start a new extension at the top of a page.  
            int currentLine = 0;  
  
            // Output only as many lines of the current group as will fit in the window.  
            do  
            {  
                Console.Clear();  
                Console.WriteLine(filegroup.Key == String.Empty ? "[none]" : filegroup.Key);  
  
                // Get 'numLines' number of items starting at number 'currentLine'.  
                var resultPage = filegroup.Skip(currentLine).Take(numLines);  
  
                //Execute the resultPage query  
                foreach (var f in resultPage)  
                {  
                    Console.WriteLine("\t{0}", f.FullName.Substring(rootLength));  
                }  
  
                // Increment the line counter.  
                currentLine += numLines;  
  
                // Give the user a chance to escape.  
                Console.WriteLine("Press any key to continue or the 'End' key to break...");  
                ConsoleKey key = Console.ReadKey().Key;  
                if (key == ConsoleKey.End)  
                {  
                    goAgain = false;  
                    break;  
                }  
            } while (currentLine < filegroup.Count());  
  
            if (goAgain == false)  
                break;  
        }  
    }  
}  
```  
  
 此程序的输出可能很长，具体取决于本地文件系统的详细信息和 `startFolder` 的设置。 为了能够查看所有结果，此示例演示如何对结果进行分页。 相同的方法适用于 Windows 和 Web 应用程序。 请注意，由于代码对组中的项进行分页，因此需要使用 `foreach` 循环。 此外，还有一些其他逻辑用于计算列表中的当前位置，以及使用户能够停止分页并退出程序。 在此特定情况下，根据原始查询的缓存结果运行分页查询。 在其他上下文中，如 LINQ to SQL，则不需要此类缓存。  
  
## <a name="compiling-the-code"></a>编译代码  
 创建面向 .NET Framework 3.5 或更高版本的项目，此项目包含对 System.Core.dll 的引用和针对 System.Linq 和 System.IO 命名空间的 `using` 指令。  
  
## <a name="see-also"></a>请参阅  
 [LINQ to Objects (C#)](../../../../csharp/programming-guide/concepts/linq/linq-to-objects.md)   
 [LINQ 和文件目录 (C#)](../../../../csharp/programming-guide/concepts/linq/linq-and-file-directories.md)
