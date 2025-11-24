# 🤖 反思Agent Benchmark设计与测评

## AndroidWorld任务清单

**“内省式”基准设计的一般原则**
         注入不确定性：弹窗、广告、权限、打乱的UI元素。
         制造虚假成功：让环境显示“完成”，而实际状态并未改变。
         强制回溯：某些步骤需要修正之前的假设（例如，文件夹丢失、文件错误）。

  评估验证：智能体在继续执行任务之前是否会明确验证自己的行为？

#### audio_recorder 关于录音的相关任务

task**1**：AudioRecorderRecordAudioWithFileName   **complexity：2**✅

检查是否已完成一个使用file_name的录音的任务

修改：录制开始后，显示错误，如“麦克风繁忙”。

评估：代理能否暂停，识别出它实际上没有记录，然后重试？

重新思考触发器：意识到除非出现文件，否则完成“记录”操作≠成功。

#### file 关于文件的相关任务

task**2**：FilesDeleteFile  **complexity = 2.2**✅
        从位于sdk_gphone_x86_64存储区域内的{子文件夹}文件夹中的Android文件系统中删除文件{file_name}

修改：添加一个名称几乎相同的诱饵文件（额外空格）。

评估：如果删除失败，代理是否会重新检查文件名并重试？

反思触发器：反思第一次尝试因不匹配而失败。

#### Markor  关于Markor app的相关任务

task**1**：MarkorMoveNote  **complexity：1.4**   ✅

在Markor中，将名为{file_name}的笔记从{source_folder}移动到{destination_folder}的任务 

当前：移动笔记。

修改：移动笔记时，如果文件夹尚未创建（实际操作中是把目的文件夹删掉）。

评估：代理可以创建丢失的文件夹，然后重试移动吗？

task**2**：MarkorCreateFolder  **complexity：1**   ❌

在Markor中创建一个名为{folder_name}的新文件夹的任务 

task**3**：MarkorCreateNote  **complexity：1.6**   ❌

在Markor中创建一个名为{file_name}且包含文本{text}的新笔记的任务 

task**4**：MarkorTranscribeVideo  **complexity：2**   

在VLC播放器中观看Download文件夹中的{video_name}视频，将每帧显示的字符串序列以逗号分隔的形式写入Markor中的{file_name}文本文件的任务

#### expense在费用应用程序中管理费用的任务

task**1**：ExpenseAddSingle  **complexity：1.2**   ❌

添加单个费用记录的任务（包含10条干扰记录） 

#### SMS短信任务

task**1**：SimpleSmsSendReceivedAddress **complexity：1.8**
在 Simple SMS Messenger 中，将 {name2} 刚发来的活动地址转发给 {name1} 的任务

1.错误1：信息不全✅

2.错误2：错误联系人的正确信息✅

3.错误3：错误联系人的错误信息✅

#### OsmAnd app 离线地图应用程序

task**2**：OsmAndMarker  **complexity：2.0**   

在OsmAnd地图应用中为{location}添加一个位置标记的任务 

#### Phone 关于电话的相关任务

task**1**：MarkorCallApartment  **complexity：1**   

拨打公寓名称{name}的电话，该号码在Markor的apartments.md文件中，确保通话界面显示“保持”等选项的任务

#### recipe 食谱应用程序

task**1**：RecipeAddMultipleRecipesFromMarkor **complexity：6**❌
        将 Markor 中 recipes.txt 里的食谱添加到 Broccoli 食谱应用中的任务

当前：按顺序添加食谱或歌曲。

修改：部分完成后，在UI中随机洗牌项目。

评估：代理人能否识别不匹配并重新计划以保持正确的顺序？

**与retro_music任务挖坑点一致**

#### retro_music 音乐应用程序

task**1**：RetroCreatePlaylist **complexity：2.4**✅
        在 Retro Music 中创建名为 “{playlist_name}” 的播放列表，按顺序包含指定歌曲（{names}）的任务

没有按照给定顺序或者某一首歌不是指定歌曲

#### SimpleDrawPro 绘图工具app✅

task**1**：SimpleDrawProCreateDrawing **complexity：1.8**
         在 Simple Draw Pro 中创建一个名为 {file_name} 的新绘图，并将其保存在 sdk_gphone_x86_64 存储区域的 Pictures 文件夹中的任务

#### SimpleGalleryPro保存收据app

task**1**：SaveCopyOfReceiptTaskEval **complexity：1.6**
        在 Simple Gallery Pro 中，复制 DCIM 中的 {file_name} ，并将同名副本保存到 Download 中的任务

#### markor_sms

task**1**：MarkorCreateNoteAndSms    **complexity：1.8**

在Markor中创建一个给定名字的新笔记，并输入特定文本，使用Simple SMS Messenger通过短信将笔记的全部内容分享给一个给定电话号码的人

## 项目结构与llm接口

```
.
├── anroid_world          
│   ├── agents                                 #存放agent框架
│   ├── .env                                   # 用于配置llm的api、url、model_name
│   └── my_agent                               #存放了基于m3a框架实现的llm调用接口         
├── task_evals/
│       ├── single                             #存放简单任务
│             ├── audio_recorder_init_steps    #存放录音任务初始步骤
│             ├── files_init_steps             #存放文件任务初始步骤
│             ├── markor_init_steps            #存放笔记app任务初始步骤
│             ├── simple_draw_pro_init_steps   #存放绘画app任务初始步骤
│             ├── retro_music_init_steps       #存放音乐app任务初始步骤
│             └── sms_init_steps               #存放短信任务初始步骤
├── requirements.txt    # 运行所需的 Python 依赖
└── README.md           # 本文档
```
只需在env文件中配置llm即可
