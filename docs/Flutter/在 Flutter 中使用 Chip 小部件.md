Flutter 中使用Chip 小部件



本文是关于 Flutter 中的 Chip 小部件。我们将大致了解小部件的基本原理，然后通过代码来实现它。事不宜迟，让我们开始吧。

> 作者：坚果
>
> 公众号："[大前端之旅](https://mp.weixin.qq.com/s/aJvihD4dzEJyOV3q6_Zeng)"
>
> 华为云享专家，InfoQ签约作者，阿里云专家博主，51CTO博客首席体验官，[开源项目GVA成员之一](https://www.gin-vue-admin.com/)，专注于大前端技术的分享，包括Flutter,小程序,安卓，VUE，JavaScript。



## 概述

典型的chip是一个圆角的小盒子。它有一个文本标签，并以一种有意义且紧凑的方式显示信息。chip可以在同一区域同时显示多个交互元素。一些流行的chip用例是：

- 发布标签（您可以在许多 WordPress ，VuePress，知乎，掘金，公众号或 GitHub等大型平台上看到它们）。
- 可删除的内容列表（一系列电子邮件联系人、最喜欢的音乐类型列表等）。

![img](https://luckly007.oss-cn-beijing.aliyuncs.com/image/Screen-Shot-2022-01-24-at-14.35.05.jpg)

在 Flutter 中，您可以使用以下构造函数来实现 Chip 小部件：

```dart
Chip({
  Key? key, 
  Widget? avatar, 
  required Widget label, 
  TextStyle? labelStyle, 
  EdgeInsetsGeometry? labelPadding, 
  Widget? deleteIcon, 
  VoidCallback? onDeleted, 
  Color? deleteIconColor, 
  bool useDeleteButtonTooltip = true, 
  String? deleteButtonTooltipMessage, 
  BorderSide? side, 
  OutlinedBorder? shape, 
  Clip clipBehavior = Clip.none, 
  FocusNode? focusNode, 
  bool autofocus = false, 
  Color? backgroundColor, 
  EdgeInsetsGeometry? padding, 
  VisualDensity? visualDensity, 
  MaterialTapTargetSize? materialTapTargetSize, 
  double? elevation, 
  Color? shadowColor
})
```

只有**label**属性是必需的，其他是可选的。一些常用的有：

- **avatar**：在标签前显示一个图标或小图像。
- **backgroundColor** : chip的背景颜色。
- **padding**：chip内容周围的填充。
- **deleteIcon**：让用户删除chip的小部件。
- **onDeleted**：点击**deleteIcon**时调用的函数。

[您可以在官方文档](https://api.flutter.dev/flutter/material/Chip-class.html)中找到有关其他属性的更多详细信息。但是，对于大多数应用程序，我们不需要超过一半。

## 简单示例

这个小例子向您展示了一种同时显示多个chip的简单使用的方法。我们将使用**Wrap**小部件作为chip列表的父级。当当前行的可用空间用完时，筹码会自动下行。由于Wrap 小部件的**间距**属性，我们还可以方便地设置chip之间的距离。

**截屏：**

![image-20220125100331474](https://luckly007.oss-cn-beijing.aliyuncs.com/image/image-20220125100331474.png)

**代码：**

```dart
Scaffold(
      appBar: AppBar(
        title: const Text('大前端之旅'),
      ),
      body: Padding(
        padding: const EdgeInsets.symmetric(vertical: 30, horizontal: 10),
        child: Wrap(
            // space between chips
            spacing: 10,
            // list of chips
            children: const [
              Chip(
                label: Text('Working'),
                avatar: Icon(
                  Icons.work,
                  color: Colors.red,
                ),
                backgroundColor: Colors.amberAccent,
                padding: EdgeInsets.symmetric(vertical: 5, horizontal: 10),
              ),
              Chip(
                label: Text('Music'),
                avatar: Icon(Icons.headphones),
                backgroundColor: Colors.lightBlueAccent,
                padding: EdgeInsets.symmetric(vertical: 5, horizontal: 10),
              ),
              Chip(
                label: Text('Gaming'),
                avatar: Icon(
                  Icons.gamepad,
                  color: Colors.white,
                ),
                backgroundColor: Colors.pinkAccent,
                padding: EdgeInsets.symmetric(vertical: 5, horizontal: 10),
              ),
              Chip(
                label: Text('Cooking & Eating'),
                avatar: Icon(
                  Icons.restaurant,
                  color: Colors.pink,
                ),
                backgroundColor: Colors.greenAccent,
                padding: EdgeInsets.symmetric(vertical: 5, horizontal: 10),
              )
            ]),
      ),
    );
```

在这个例子中，chip只呈现信息。在下一个示例中，chip是可交互的。

## 复杂示例：动态添加和移除筹码

### 应用预览

![chip](https://luckly007.oss-cn-beijing.aliyuncs.com/image/chip.gif)

我们要构建的应用程序包含一个浮动操作按钮。按下此按钮时，将显示一个对话框，让我们添加一个新chip。可以通过点击与其关联的删除图标来删除每个chip。

以下是应用程序的工作方式：

### 完整代码

**main.dart**中的最终代码和解释：

```
// main.dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: '大前端之旅',
      theme: ThemeData(
        primarySwatch: Colors.green,
      ),
      home: const HomePage(),
    );
  }
}

// Data model for a chip
class ChipData {
  // an id is useful when deleting chip
  final String id;
  final String name;
  ChipData({required this.id, required this.name});
}

class HomePage extends StatefulWidget {
  const HomePage({Key? key}) : super(key: key);

  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  // list of chips
  final List<ChipData> _allChips = [];

  // Text controller (that will be used for the TextField shown in the dialog)
  final TextEditingController _textController = TextEditingController();
  // This function will be triggered when the floating actiong button gets pressed
  void _addNewChip() async {
    await showDialog(
        context: context,
        builder: (_) {
          return AlertDialog(
            title: const Text('添加'),
            content: TextField(
              controller: _textController,
            ),
            actions: [
              ElevatedButton(
                  onPressed: () {
                    setState(() {
                      _allChips.add(ChipData(
                          id: DateTime.now().toString(),
                          name: _textController.text));
                    });

                    // reset the TextField
                    _textController.text = '';

                    // Close the dialog
                    Navigator.of(context).pop();
                  },
                  child: const Text('提交'))
            ],
          );
        });
  }

  // This function will be called when a delete icon associated with a chip is tapped
  void _deleteChip(String id) {
    setState(() {
      _allChips.removeWhere((element) => element.id == id);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('大前端之旅'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(15),
        child: Wrap(
          spacing: 10,
          children: _allChips
              .map((chip) => Chip(
                    key: ValueKey(chip.id),
                    label: Text(chip.name),
                    backgroundColor: Colors.amber.shade200,
                    padding:
                        const EdgeInsets.symmetric(vertical: 7, horizontal: 10),
                    deleteIconColor: Colors.red,
                    onDeleted: () => _deleteChip(chip.id),
                  ))
              .toList(),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _addNewChip,
        child: const Icon(Icons.add),
      ),
    );
  }
}

```

## 结论

我们已经探索了 Chip 小部件的许多方面，并经历了不止一个使用该小部件的示例。

大家喜欢的话，点赞支持一下坚果

