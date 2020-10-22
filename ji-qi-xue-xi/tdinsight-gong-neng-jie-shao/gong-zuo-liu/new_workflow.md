# 新建工作流

## 1. 工作流页面

在工程页点击某个工程下拉框中的“+”，输入工作流名称，就可新建一个工作流，进入工作流页面，是用户的主要工作页面。 工作流页面最左侧是组件算法区， 中间空白区域是画布，画布的上方是工作流的运行快捷键。单击画布上的任一个节点将显示右侧的节点参数配置区。

 ![](../../../.gitbook/assets/newflow1.png) 左侧组件算法区上方有搜索框，如果你想找到想要的算法，可以直接搜索。 画布左侧有画布的缩放，拖拽等按钮 工具栏左侧的下拉框中可以切换本工程内的任意一个工作流。

在最左侧的算法区中选择所需要的算法，拖拽或双击该算法，就能新增节点，系统会自动进行连线，可右击连线，删除连线。连线表示节点之前的依赖关系，同时对于数据，模型节点，还代表这数据的流向。 例如图中第一个节点是数据集，如果第二个节点是算法节点（图中是组件节点，需要在程序参数中指定输入输出），那么这个算法节点的输入就会自动匹配为上一个节点的输出。

## 2. 工作流状态

在工程页面，会列出其下的所有工作流，工作流有五种常见状态，分别是**就绪、运行中、成功、失败和被终止**，每个状态对应不同的颜色和图标。状态和流的最近一个节点的状态一致。

 ![](../../../.gitbook/assets/pic2_1.png) ![](../../../.gitbook/assets/pic2_2.jpg) ![](../../../.gitbook/assets/pic2_3.jpg) ![](../../../.gitbook/assets/pic2_4.jpg) ![](../../../.gitbook/assets/pic2_5.jpg)

### 3 工作流快捷操作

对于工作流有几种操作：**更名、运行、删除、复制、关注**，在上图工作流卡片的下方有快捷键，方便用户进行简单的操作。用法如下：

1. **更名**：更改工作流名称；
2. **运行**：运行工作流；
3. **删除**：删除工作流；
4. **复制**：复制工作流到其他工程；
5. **关注**：关注了这个工作流后，将根据用户设置的告警规则，达到触发条件后，发送告警
