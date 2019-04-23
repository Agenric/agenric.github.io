---
title: 模仿 iOS 8.0 邮件左划删除的功能
date: 2015-10-08 13:38:00
categories:
  - 技术
tags: 
  - iOS小知识
---
最近自己看了一下iOS 8.0中的邮件删除的功能，在iOS8.0中苹果给tableView新增了一个在Cell上侧滑，右边弹出按钮进行便捷操作的一个新的代理方法。但是细心的话你会发现，这个方法只提供类似QQ、微信中的那种单调的弹出菜单，并不能实现类似邮件中的从右一直往左拉到一定程度可以直接删除的快捷操作。所以自己就想试着写一个这样的小Demo，如果你不经意间看到了，欢迎指证，不喜勿喷...

其实仔细的思考一下这个东西，并不是太有技术含量（ps:好吧， 承认自己程度没那么高，所以也想通过这种方式提高自己）。我大概的把要做的工作分了以下几步：

1. 首先自定义自己的Cell，Cell要提供两个代理方法
   * 获取每一行Cell右边将要显示的按钮集合
   * 每一行Cell右边按钮被点击后的回调
2. 其次就是在Cell内部要实现touchesBegan等一系列方法来监测我们的拖拽手势
3. 在touchesMoved里面根据用户拖拽手势的点来动态改变Cell中contentView和右边按钮的frame

---

## 定义自己的Cell

下面我们首先创建自定义的一个Cell

```objc
#import <UIKit/UIKit.h>
#import "AGTableViewRowAction.h"

@protocol AGTableViewCellDelegate <NSObject>
@optional
/*!
 * @brief  获取每一行Cell对应的按钮集合
 *
 * @param tableView 父级tableView
 * @param indexPath 索引
 *
 * @return 该行Cell的按钮集合
 */
- (NSArray *)AGTableView:(UITableView *)tableView editActionsForRowAtIndexPath:(NSIndexPath *)indexPath;

/*!
 * @brief  每一行Cell的动作触发回调
 *
 * @param tableView 父级tableView
 * @param index     点击按钮集合的动作索引
 * @param indexPath 索引
 */
- (void)AGTableView:(UITableView *)tableView didSelectActionIndex:(NSInteger)index forRowAtIndexPath:(NSIndexPath *)indexPath;
@end


@interface AGTableViewCell : UITableViewCell

/*!
 * @brief  滑动过程中刷新动画的时间间隔，默认值是0.2s
 */
@property (nonatomic, assign) CGFloat dragAnimationDuration;

/*!
 * @brief  重置动画的时长，默认值是0.3s
 */
@property (nonatomic, assign) CGFloat resetAnimationDuration;

@property (nonatomic, assign) BOOL isEditing;
@property (nonatomic, weak) id<AGTableViewCellDelegate> delegate;

-(instancetype)initWithStyle:(UITableViewCellStyle)style reuseIdentifier:(NSString *)reuseIdentifier inTableView:(UITableView *)tableView;
@end
```

这个类的实例化方法除了接收tableView的Cell的样式以及复用标示之外，还把其所属的tableView作为气的一个属性传过来，目的是为了在用户拖动Cell的时候限制tableView的滚动。

头文件中的两个设置动画时长的属性以及Cell的预设值需要在初始化方法中进行如下设置

```objc
self = [super initWithStyle:style reuseIdentifier:reuseIdentifier];

if (self) {
    [self.contentView setBackgroundColor:[UIColor grayColor]];
    self.tableView = tableView;
    self.touchBeganPointX = 0.0f;
    self.dragAnimationDuration = 0.2f;
    self.resetAnimationDuration = 0.3f;
    self.isEditing = NO;
    _isMoving = NO;
    _hasMoved = NO;
}
return self;
```

这里我使用了两个临时的局部变量_isMoving _hasMoved来在Cell被拖动的时候标示其状态，使用self.isEditing来判断在当前Cell上是应该响应一般touch事件还是响应它的tableView的didSelectRowAtIndexPath事件。然后我们要在Cell内部拥有一个可变的数组来存储该行Cell右边的action集合，然后最好在layoutSubviews的时候通过代理方法拿到这个集合。

```objc
- (void)getActionsArray {
    self.indexPath = [self.tableView indexPathForCell:self];

    if ([self.delegate respondsToSelector:@selector(AGTableView:editActionsForRowAtIndexPath:)]) {
        self.actionButtons = [[self.delegate AGTableView:self.tableView editActionsForRowAtIndexPath:self.indexPath] mutableCopy];
        CGFloat buttonWidth = (SCREENWIDTH / 2.0f) / self.actionButtons.count;
        self.buttonWidth = buttonWidth;
        for (AGTableViewRowAction *action in self.actionButtons) {
            action.frame = CGRectMake(SCREENWIDTH, 0.0f, buttonWidth, self.height);
            [action addTarget:self action:@selector(rightActionDidSelected:) forControlEvents:UIControlEventTouchUpInside];
            [self addSubview:action];
        }
    }
}
```

注意最后[self addSubview:action]，通常我们习惯在self.contentView上添加子View，但是记得不要把右边的action添加到contentView上去。这样的话在移动contentView的时候会带着右边的action一同移动。

## 监听手势

这些工作做完接下来就是核心的处理逻辑，监听用户触摸的手势：
  **首先**要touchesBegan方法内判断touches.count是否为1，若不为1则调用父类的方法 [super touchesBegan:touches withEvent:event] ，若为1则记录用户开始接触到屏幕是的点的水平(x)轴的坐标，以备使用。
  **然后**要在touchesMoved方法内针对用户触摸的点与开始触摸屏幕是的点进行对比并对当期Cell的所有子View的frame进行更改。
  **最后**在touchesEnded方法中获取到用户最后触摸的点，然后来判定Cell当前该执行哪一种操作
    1. 删除自己
    2. 恢复到展示右边actions的状态
    3. 恢复到隐藏右边actions的最初始的状态

下面是touchesMoved中动态改变Cell内部子View的一段代码

```objc
CGFloat currentLocationX = [touch locationInView:self.tableView].x;
CGFloat distance = (self.touchBeganPointX - currentLocationX) * 1.1;

if (distance > 0) { // 向左拉
    CGFloat button_addWidth = (distance - (SCREENWIDTH / 2.0)) / self.actionButtons.count;
    [UIView animateWithDuration:self.dragAnimationDuration animations:^{
        self.contentView.left = -distance;
        CGFloat t_dis = distance;

        for (AGTableViewRowAction *action in self.actionButtons) {
            if (distance > SCREENWIDTH / 2.0f) {
                if (currentLocationX < 50) {
                    action.left = SCREENWIDTH - distance;
                    action.width = distance;
                } else {
                    action.left = SCREENWIDTH - t_dis;
                    action.width = self.buttonWidth + button_addWidth;
                }
            } else  {
                action.left = SCREENWIDTH - t_dis;
            }
            t_dis = t_dis - distance / self.actionButtons.count;
        }
    }];
} else { // 向右拉
    // doSomething
}
```

在这里我把distance作为用户在Cell上拖动的长度，之所以在后边乘以一个1.1，这其实是模拟一个弹性系数，让Cell的移动距离稍稍的大于用户手指拖动的距离，而1.1的系数在真实显示的情况下并不是很明显，你可以适当的修改这个数值达到自己理想的效果，从而获得更好的用户体验度。

我们用beganPointX减去currentPointX这样得到的值若是大于0则证明用户是在向左滑动，这样从开始滑动就要开始设置contentView的origin.x=-distance。然后遍历右边的actions数组，由于右边所有按钮总的初始宽度我预设的是当前屏幕的一把，这样也就是说，当用户滑动的距离超过屏幕宽度一般的时候就要开始修改每一个action的width。

那么在这里我设置的是当用户手指拉倒距离屏幕左边距离小于50，并且总的拖动长度大于屏幕的一半时，瞬间修改最右方的action的origin.x为用户手指当前位置的x轴上的位置，宽度增长到当前拖动的长度，然后当用户手指离开屏幕的时候，根据contentView的origin.x的位置来选择该如何重置或是删除当前Cell。

## 处理手势

最后，关于在touchesEnded中如何做处理，以及其他的一些细节就不在此赘述了。如果你想看到更详细的内容可以去下载该项目的demo源码。

---
>注：此文章首发在[简书](http://www.jianshu.com)转载请说明出处。
如果你想看到完整的代码，可以去[这里](https://github.com/Agenric/AGTableViewCell)。