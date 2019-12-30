# UITableView优化相关 
1. 使用`self-sizing`结合`autolayout`自动算高 或者 使用系统API `-systemLayoutSizeFittingSize:`自适应高度，注意点是自适应内容的最大宽度`` 需要提前设置好期望的一个值
```swift
 tableView.rowHeight = UITableView.automaticDimension
 //行高的预估值/近似值，虽然可以随便给，但是还是尽量给一个折中的或者出现次数较多的一个高度
 tableView.estimatedRowHeight = 100;
```
2. 提前计算好高度并缓存起来 可使用`UITableView+FDTemplateLayoutCell`[链接](https://github.com/forkingdog/UITableView-FDTemplateLayoutCell)来抹平优化难度
3. CoreText异步渲染文本内容为图片展示
4. 减少cell视图层级,尽量精简子控件数量
5. 按需加载内容，只加载出现在屏幕上的cell的图片，离开屏幕的则取消加载图片的任务

```swift
有些情况下我们可能会去快速的滑动列表，这时候其实会有大量的cell对象被创建、被重用，但其实我们可能只是去浏览列表停止的那一页的上下一定范围内的信息，前面快速划过的那些信息对我们来说都是无用的。此时我们可以通过ScrollView的代理方法
targetContentOffset 是TableView减速到停止的地方, velocity 表示速度向量。
#pragma mark - UIScrollViewDelegate  
//按需加载 - 如果目标行与当前行相差超过指定行数，只在目标滚动范围的前后指定3行加载。  
- (void)scrollViewWillEndDragging:(UIScrollView *)scrollView withVelocity:(CGPoint)velocity targetContentOffset:(inout CGPoint *)targetContentOffset {  
    NSIndexPath *targetPath = [_myTableView indexPathForRowAtPoint:CGPointMake(0, targetContentOffset->y)];  
    NSIndexPath *firstVisiblePath = [[_myTableView indexPathsForVisibleRows] firstObject];  
    NSInteger skipCount = 8;  
    if (labs(firstVisiblePath.row - targetPath.row)>  skipCount) {  
        NSArray *temp = [_myTableView indexPathsForRowsInRect:CGRectMake(0, targetContentOffset->y, _myTableView.frame.size.width, _myTableView.frame.size.height)];  
        NSMutableArray *arr = [NSMutableArray arrayWithArray:temp];  
        if (velocity.y<0) {  
            NSIndexPath *indexPath = [temp lastObject];  
            if (indexPath.row+33) {  
                [arr addObject:[NSIndexPath indexPathForRow:indexPath.row-3 inSection:0]];  
                [arr addObject:[NSIndexPath indexPathForRow:indexPath.row-2 inSection:0]];  
                [arr addObject:[NSIndexPath indexPathForRow:indexPath.row-1 inSection:0]];  
            }  
        }  
        [_dataList addObjectsFromArray:arr];  
    }  
}  
```

还有一种说法,滚动停止时才加载图片
```swift
- (void)scrollViewDidEndDragging:(UIScrollView *)scrollView willDecelerate:(BOOL)decelerate{  
    if (!decelerate){  
        [self loadImagesForOnscreenRows];  
    }  
} 
```
6. 利用重用机制,防止大量创建cell
7. 尽量使所有的view 不透明，包括cell自身,`cell.opaque = YES;`，默认为YES，不透明的视图可以极大地提高渲染的速度
8. 避免离屛渲染,圆角可用贝塞尔曲线绘制
9.  避免重复创建子视图，可使用hidden隐藏 
10. 分页预加载（用户无感知）[预加载原理](https://draveness.me/preload)
11. Cell类中应该避免请求网络加载数据
    > 如果确实有需求不可避免，可以将网络加载任务添加到Runloop中，
    > 设置DefaultRunloopModule模式。这样子可以起到延迟加载的作用。
12. 在`willDisplayCell:forRowAtIndexPath:`代理方法中绑定数据
    > 很多人都喜欢在cellForRowAtIndexPath：方法中绑定数据，然后此时的Cell其实还未显示，该方法中包含了大量的布局、绘制相关的操作。我们应该在该方法中尽量简化我们自身的逻辑操作。这时我们可以使用在‘willDisplayCell：forRowAtIndePath:’方法中绑定数据。

```swift
#pragma mark - UITableViewDataSource  
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {  
    static NSString *cellIdentifier = @"MyTableViewCell";  
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:cellIdentifier];  
    if (!cell) {  
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellIdentifier];  
    }  

    return cell;  
}  

#pragma mark - UITableViewDelegate  
- (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath {  

    NSDictionary *dict = self.dataList[indexPath];  
    [cell updateData:dict];  
} 
```
13. 啊啊啊

#### 参考博客
* [UITableView 性能优化](https://juejin.im/post/5acf34eff265da237314d6e0)
* [如何加强 iOS 里的列表滚动时的顺畅感](https://www.zhihu.com/question/20382396)
* [iOS 保持界面流畅的技巧](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)
* [UITableView性能优化 - 中级篇](https://segmentfault.com/a/1190000018161741)
* [UITableView的优化](https://xiaopengmonsters.github.io/2017/12/26/UITableView%E7%9A%84%E4%BC%98%E5%8C%96/)
* [详细整理：UITableView优化技巧](http://www.cocoachina.com/articles/11968)
* [优化UITableViewCell高度计算的那些事](http://blog.sunnyxx.com/2015/05/17/cell-height-calculation/)
