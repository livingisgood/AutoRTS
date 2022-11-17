### Localization

* 源文本

  源文本以csv格式提供, 放在content/Resources/LocalizationSource路径下. 

  csv的字段结构为 Key, SourceString, Comment.

  Key是代码或者UI上填的文本索引, SourceString是需要翻译的源文本, Comment为注释, 可不填

  目前已添加 UICommon.csv 用来保存那些 UI界面上的常用字面文本

* 代码中使用

  ```
  	FText::FromStringTable(TEXT("UICommon"), TEXT("wonderful")); // FName FString
  	LOCTABLE("UICommon", "wonderful");
  ```

  以上两种方式都可以, 区别在于宏只接受字面量, 函数可以传变量.

* UI控件中使用![LocalTextInUI](Pics\LocalTextInUI.jpg)点点开Text右边的箭头可以用下拉框选择表名和key.  对于特别巨大的表, 也可以选择在上面的RtsTextBlock中手填Key和表名

* 添加key的规范

  请全部小写, 单词之间下划线分割, 不要使用驼峰

* 添加csv

  比较独立的功能和模块可以在添加自己的本地化csv. 添加完后需要在代码中注册![TableRegister](C:\Users\50194\Projects\ProjectDoc\AutoRTS\Pics\TableRegister.jpg)暂时没有自动化这一步的方法, 这是因为用宏的方式加载的csv才能被本地化功能采集到文本. 所以每次添加文本, 这个函数中也需要手动注册.

* 注册过本地化csv以后, ue的编辑器会监控文件变化. 编辑器打开的时候去编辑csv并保存, ue能读到新添加的条目, 不需要重启引擎.

* 预览

  在Edit-EditorPreference选项中, 有个Region&Language设置. 选择 PreviewGameLanguage 可以改变Game的预览语言![LocalizePreview](Pics\LocalizePreview.jpg)