# Linq笔记

* [语言集成查询 (LINQ)](https://learn.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/)
* [LINQ入门示例及新手常犯的错误 -- B站视频](https://www.bilibili.com/video/BV1vP411f7XF/?spm_id_from=333.1007.top_right_bar_window_history.content.click)

## 示例

使用LinqPad执行:

```C#
var numbers = new List<int>{1,2,3,4,5,6};
//numbers.Where(x=> x%2==0).Dump();
//numbers.Select(x=>x*x).Dump();

var people = new List<string>{"Alice","Bob","Charlie"};
var peopleAge = new List<int>{25,30,20};
//people.SelectMany(r=>peopleAge,(l,r)=>new {Name=l,Age=r}).Dump();
people.Zip(peopleAge).Select(d=>new{Name=d.Item1 ,Age=d.Item2}).Dump();
```
