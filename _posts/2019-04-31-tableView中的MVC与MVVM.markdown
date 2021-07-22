---
layout: post
title: tableView中的MVC与MVVM
date: 2019-04-31 12:39:57 +0300
image: 3681626924053.jpg
tags:
---

这是我写的第一篇博客😄 ，谢谢支持～！
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

##  iOS tableView中的MVC和MVVM
> 开头：最近在利用有道的api尝试做一个翻译的应用，其中用到了tableview。有一段时间没有接触这个常用UI，发现该忘的都忘了哈哈。


#### 本文不着重讲述tableView的各种基本使用了，而打算通过下面几个方面来进行叙述思考。在复习tableView的同时，想思考一下代码的规范问题。
- 1.tableView下中MVC思考
- 2.tableView与自定义cell
- 3.一个tableView，多种类型cell(MVVM模式)

--------

## 1.tableView中MVC思考

先贴上一张MVC的一张大图（给自己看就好）


<!--![mvc.jpeg](tableView中的MVC与MVVM/13160879-7738983f8f5790d8.jpeg)-->

![This is an example image](/images/05_post/13160879-7738983f8f5790d8.jpg)

controller 相当于媒介，帮助model和View建立其联系。道理我都懂，但是以往在coding的时候，往往会出现以下的情况(代码不看)：
```

-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {

TeacherHomeworkListCell *cell = [tableView dequeueReusableCellWithIdentifier:@"TeacherHomeworkListCell" forIndexPath:indexPath];
cell.delegate = self;
NSArray *arr = homeworkListModelTDArray[indexPath.section];
cell.model = arr[indexPath.row];
if ([cell.model.requirements isEqualToString:@"0"]) {
cell.operateView.hidden = YES;
cell.correctHomeworkButton.hidden = YES;
cell.operateViewNR.hidden = NO;
} else {
cell.operateView.hidden = NO;

.......
```
贴上这么多以前的代码主要的，是可以看出在以往coding过程中，我又容易忽视了mvc的设计思想。这可能会导致：
- 1.将view（cell）和model（数据）在controller中建立起了直接的联系。
- 2.又会让tableView显得庞然大物。

因此在这次coding 的过程中，我时不时有意地注意到了这个问题，将所有的与UI展示有关的数据交由View下来处理，而避免在controller中直接的设置。所以这次我的设置数据源的方法图片如下：

<!--![image](/13160879-1705a70268be71ab)-->

![This is an example image](/images/05_post/13160879-1705a70268be71ab.jpg)

以往在设计一个tableView中有两种以上不同cell时候，我容易在cellForRowAtIndexPath：这个方法中作出许许多多的if-else的判断，而现在我将这个判断交由cell自己来做，controller自己不需要知道cell是要什么类型，而只要将得到的cell展示即可。

**我们需要做的，就是传入需要的数据，可以indexPath也可以是model等。** 这可能也很好的贯彻了“依赖注入”的设计原则。

同时，这可能对于后面的关于：自定义行高，一个tableView中有两个cell等提供有益的帮助。

不过白猫黑猫，能抓到老鼠的的都是好猫哈哈哈。

#### 补充：注意一定要传入tableView，因为cell的复用问题。
---------
## 2.tableView与自定义cell
> 这部分写给自己看哈哈哈，主要是遇到了几个忘记的知识点。
> 

- 1.如果事先给tableView注册了cell类型，则不需要进行cell为空的处理，缓存池中就会有足够的cell等待复用。
- 2.在通过[_tableView registerClass...]注册了cell之后，在进行cell复用时，底层是调用了
`- (instancetype)initWithStyle:(UITableViewCellStyle)style reuseIdentifier:(NSString *)reuseIdentifier`的方法，因此如果需要返回xib自定义的cell，则需要对这个方法override。
- 3.在通过xib自定义时，需要通过tableViewDelegate设置行高（代理里的默认行高为44），否则可能无法完全展示xib的cell。
- 4.通过xib加载cell时候，底层会调用awakeFromNib的方法（这个也忘了😂）

-------
## 3.一个tableView，多种类型cell
>在现如今的应用程序当中，一个tableView中含有多种类型的cell，这已是一个普遍的UI需求。

参考了网上各种方法之后，自我总结了一下。
我想到了如下两种：
- 1.将相关参数传入view，通过view来返回需要指定类型的cell。
- 2.根据两种不同的标识符，获取不同的cell。

### 对于第一种方法:
我在实现的过程中，将view与xib绑定在一起，根据传入的indexPath或者数据，返回出指定类型的的cell。以往我会在xib文件中只放一个cell，而此次我则不再是通过[array lastObject]取出唯一的cell，而是根据指定的index从多个cell中取出指定的cell。

![image](http://upload-images.jianshu.io/upload_images/13160879-da328ce1dbb87e33?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

附上伪码:

```
@implementation CustomCell
- (instancetype)cellWithIndexPath:(NSIndexPath *)indexPath {
CustomCell *cell = nil;
NSArray *cellArr = [[NSBundle mainBundle] loadNibWith...];
if (...){
cell = [cellArr objectAtIndex:index];
}else if (...) {
...
}else {
cell = [cellArr lastObject];
}
return cell;
}
@end
```
但是这种方法的缺点也很明显，与tag的使用大同小异。当一个tableView需要许多种类型的cell来丰富内容的时候，采用这种方法在开发过程中会带来一定的混乱。

**可以适当的采用enum枚举，但同时也要注意xib中左面板中视图的上下位置。**

>例如上图中，NormalTyepCell 的index=1，MeTypeCell为2，而当两者调换了位置之后，索引也会发生改变。

### 对于第二种方法：
> 个人也比较倾向于第二种方法，对于不同的类型的cell，应有不同的Identifier进行绑定，这样子比较科学哈哈,同时采用了较为高大上的MVVM，。

我们大可以像第一种方式一样，从xib中取出所需要的cell，但是这也许就没有多个Identifier存在的必要性。而针对不同标识符取出对应的cell，更多的是依赖:

`[tableView dequeueReusableCellWithIdentifier:Identifier];`
的调用。
##### 如何获得对应cell的Identifier：

同样是可以采用前面的方法来进行开发，但是细想一下，如果cell的类型过多，这样子会导致view中代码成本过高，相比与第一种方法，这种方法则更加麻烦。

如何解决？那就着手已产生的麻烦---胖View。
#### MVVM
> 开始写文章的时候并没有考虑，但是写着写着就觉得也许可以应用进来哈哈。9012希望能多写文章，记录笔记！

先贴几张MVVM的大图：

[图片上传失败...(image-bdeca-1549460154646)]

[图片上传失败...(image-36b52f-1549460154646)]

![image](http://upload-images.jianshu.io/upload_images/13160879-250d8135a7e4f290?imageMogr2/auto-orient/strip)

对于新面孔ViewModel：从MVC的controller中抽取出来的展示逻辑，负责从model中获取view所需的数据，转换成View可以展示的数据，并暴露公开的属性和命令供view进行绑定。

#### 回到第二种方法的实现上来。
MVVM的采用可以很好的为view以及controller制定了瘦身的计划，我们将“获取指定cell的Identifier”这一逻辑交由ViewModel来实现，然后通过controller告诉view需要哪个cell。

附上viewModel和tableView设置数据源的伪码：

```
@implementation ViewModel 
- (NSString *)identifierWith:(Model *)model {
if (model.type == ?){
return firstCellID;
}else if (model ...){
return secondCellID;
}else {
return ...
}
}
@end

@implementation controller
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
NSString *identifier = [_viewModel identifierWith:model];
return [tableView dequeReusa....:identifier];
}
@end
````
在设置数据源方法中，我没有进行cell为空的操作处理，因为需要在这之前，对tableView进行所有类型cell的注册，个人也比较喜欢先注册cell的方式，这样子可以让代码看起来更加直观。

-----
> ## 结尾：
> 回顾前面所讲的，我的出发点可以理解为始终是一个---为controller制定瘦身计划，同时使代码更加容易维护。
之前只是一直想着coding，把功能实现就好，而从来没有想过将代码写在哪里会更加合理。
> 当然上面的方法并不是最佳😂，自我总结，仅供参考，大神路过有意见，还望指点下。

本来还想通过尝试tableView自动算高来巩固复习tableView，然而发现这里是一块肥肉，考虑篇幅，还得重新开一篇笔记细细评味😂。




### 参考
[ReactiveCocoa and MVVM, an Introduction](http://www.sprynthesis.com/2014/12/06/reactivecocoa-mvvm-introduction/)
