---
title: 游戏逆向-翻译游戏
author: Tu
date: 2022-09-07
description:
category:
layout: post
---

# 起因

某日拿从网上找到了某游戏资源。打开游戏看到了游戏的主界面，顿时眉头一皱，发现事情并不简单。游戏处在汉化了但是又没完全汉化的程度。遂打算凭借一己之力对游戏来一次逆向+汉化。

# 过程

## 初步分析游戏

查看游戏根目录，发现目录是这样的

```
│  Readme.txt.txt
│  {GameName}.exe
└─{GameName}_Data
   │  level0
   │  level1
   |  ...
   │  mainData
   │  resources.assets
   |  ...
   │  sharedassets0.assets
   |  ...
   ├─Managed
   │    Assembly-CSharp.dll
   |    ...
   ├─Mono
   │    mono.dll
   │    ...
   └─Resources
        ...
```

可以得出以下几个结论：

1. 游戏使用 Unity 开发
2. 游戏未采用 il2cpp

然后右键查看 **_{GameName}.exe_** 的版本信息，可以得知游戏使用 4.6.4.10090627 版本的 Unity 进行的开发。

## 逆向分析

虽然可以尝试使用 **_XUnity.AutoTranslator_** ，但是这里仍旧采用逆向的方法。

首先使用 **_dnSpy_** 工具查看 **_{GameName}\_Data \ Managed \ Assembly-CSharp.dll_**，这个文件里面会包含大部分的游戏脚本。由于是汉化，搜索的关键词为"UI"、"Menu"、"Button"、"Dialog"等。对应的内容为菜单上呈现的文本、按钮上的文本、部分对话等。

具体内容需要结合游戏内的情况。以本游戏为例，游戏内有一个奖杯系统（Trophy），有多段人物对话（Text/Dialog/Scence/Event），有多个菜单和按钮（UI/GUI/Menu/Button）。并且这些内容大多会有一个管理脚本（Manage/Manager/Control/Controller）。

根据关键词搜索具体代码可以发现如下片段：

```
class InqVoiceManager:

        GameObject.Find("UI Root/Mode/text").GetComponent<UILabel>().text = "{TextHere}";

        GameObject.Find("UI Root/Count/SelectButton01_/SelectButton01/Select01_font").GetComponent<UILabel>().text = "{TextHere}";

	if (this.stateInfo.IsName("Text01"))
	{
		this.WordText.text = "{TextHere}";
		if (this.TextOne)
		{
			return;
		}
		this.TextOne = true;
	}

```

```
class QualitySetting:
        bool flag=GUI.Button(position,"{TextHere}")
```

```
class SaveLoadSys
	switch (SaveLoad.Data.setting.Type)
	{
	case 0:
		this.UIPopupList.value = "{TextHere}";
		break;
	case 1:
		this.UIPopupList.value = "{TextHere}";
		break;
	default:
		this.UIPopupList.value = "{TextHere}";
		break;
	}
```

```
class SaveLoadViewer:
        GUI.Box(new Rect((float)(0 + this.ButtonOff), 0f, 180f, 700f), string.Concat(new string[]
		{
			"{TextHere}",
			SaveLoad.Data.setting.Level.ToString(),
			"\n{TextHere}",
			SaveLoad.Data.setting.Count.ToString(),
			"\n{TextHere}",
                        ...
                }
        )
```

```
class VoiceManager:
        if (this.count == 0)
	{
		this.label.text = "{TextHere}";
	}
```

这里我们会遇到过一个问题：如果我们直接在 **_dnSpy_** 软件中对源代码进行修改，那么我们需要手动编辑一个个类一个个方法，然后手动把所有修改后的文本粘贴进去。工作量实在太大。

所以我们这里使用 **_dnSpy_** 的"导出到工程"功能，把 Assembly-CSharp.dll 导出为 Visual Studio 的解决方案（导出时需要注意引用关系）。每一个在 **_Assembly-CSharp.dll_** 中的类都会被拆分成一个单独的 _.cs_ 文件。

也可以使用 **_uTinyRipper_** 提取整个游戏，之后我们就可以在 **_\Assets\Scripts\Assembly-CSharp_** 这个目录下见到提取后的脚本。每一个类都会被拆分成一个单独的 _.cs_ 脚本和一个对应的 _.meta_ 文件。

然后我们就可以尝试使用自动化脚本，根据我们之前发现的片段，编写对应的正则表达式，从而批量地将文本从 _.cs_ 文件中提取出来。

之后也可以使用一些脚本，将汉化后的文本替换回对应的 _.cs_ 脚本。

之后，我们需要把这些零碎的 _.cs_ 脚本重新打包成 _.dll_ 库。如果之前使用 **_dnSpy_** 导出为 Visual Studio 解决方案，这里就可以使用重新生成功能，生成汉化后的 _.dll_ 库。
如果你使用的是 **_uTinyRipper_** 就会更麻烦一些，可以尝试使用 **_UnityEX_** 进行资源的导入。笔者这里不再做尝试。

---

游戏中有一些内容会以贴图的形式存在。例如有些游戏中的菜单，菜单上龙飞凤舞的文字可能就是和菜单融为一体的一张图片。我们接下来就尝试对这种图片类的资源进行汉化。

这里我们要用到 **_UABE(Unity Asset Bundle Extractor)_** 、**_Asset Studio GUI_** 。

图片类的资源在 Unity 工程中，多会以 Texture2D 的格式存在，存在的可能在 **_Level0_**、**_Level1_**、**_resources.assets_** 等文件中。我们可以在 **_UABE_** 和 **_Asset Studio_** 中查看这些文件。比较尴尬的是,这两个软件的功能不完全重合，这里推荐两个软件配合使用。使用 **_Asset Studio_** 的预览功能和过滤功能找到需要修改的 Texture2D 格式的内容。使用 **_UABE_** 导出，PS ，**_UABE_** 导入，收工。

| 功能 | UABE | Asset Studio |
| :--: | :--: | :----------: |
| 预览 |      |      √       |
| 导出 |  √   |      √       |
| 导入 |  √   |              |

如果文件名不能识别，可以根据文件 ID（File ID）和路径 ID(Path ID)、文件大小（File Size）等信息寻找。

图片的修改内容这里不再过多赘述。

---

汉化的时候，我们也需要对场景中独有的文本内容或者脚本内容进行汉化。在两个软件中对 MonoBehavior、TextAsset 类型的文件进行过滤查看。

以这个游戏为例，我们发现在 **_resources.assets_** 中存在一个名为 _Event01_ 、格式为 _MonoBehaviour_ 的对话脚本。首先尝试 **_UABE_** 的导出功能，查看导出后的文件，发现文件内容根本对不上。所以这里我们需要使用 Export Raw 功能，导出 _.dat_ 格式的源文件。

之后的步骤会比较麻烦，我们需要使用十六进制编辑器对 _.dat_ 格式的源文件中的字符串进行修改。这里有两个难点：

1. 需要增加或删除字节来对齐修改后的内容
2. 修改后需要修改字符串前用于表明字符串长度的数值
