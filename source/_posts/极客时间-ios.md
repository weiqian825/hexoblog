---
title: 极客时间-ios
date: 2020-07-02 19:50:06
tags:
---

### 一、IOS 基础

1. UIView - 最基础的视图类，管理屏幕上区域内容展示

作用：
布局 （大小，位置 frame， addSubView）
使用栈管理全部的子 View（）

```c
#import "ViewController.h"
@interface TestView: UIView
@end
@implementation TestView
-(instancetype) init{
    self = [super init];
    if(self){
    }
    return self;
}
-(void)willMoveToSuperview:(nullable UIView *)newSuperview{
    [super willMoveToSuperview: newSuperview];
}
- (void)didMoveToSuperview{
    [super didMoveToSuperview];
}
- (void)willMoveToWindow:(nullable UIWindow *)newWindow{
    [super willMoveToWindow: newWindow];
}
- (void)didMoveToWindow{
    [super didMoveToWindow];
}
@end
@interface ViewController ()
@end
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor whiteColor];
    TestView *view2 = [[TestView alloc] init];
    view2.backgroundColor = [UIColor greenColor];
    view2.frame = CGRectMake(150, 150, 100, 100);
    [self.view addSubview:view2];
}
@end
```

2. UIViewController - 视图控制器，管理视图 View 层级结构

a. 管理 View 视图生命周期

b. ViewController 生命周期
init
viewDidLoad
viewWillAppear
viewDidAppear
viewWillDisappear
viewDidDisappear
Dealloc

c. UIView 负责页面内的内容呈现
使用基础的 UIViewController 管理多个 UIView
UIViewController 在管理 UIView 的同时，负责不同页面的切换

```c
#import "ViewController.h"
@interface ViewController ()
@end
@implementation ViewController
-(instancetype) init{
    self = [super init];
    if(self){}
    return self;
}
- (void)viewWillAppear:(BOOL)animated{
    [super viewWillAppear:animated];
}
- (void)viewDidAppear:(BOOL)animated{
    [super viewDidAppear:animated];
}
- (void)viewWillDisappear:(BOOL)animated{
    [super viewWillDisappear:animated];
}
- (void)viewDidDisappear:(BOOL)animated{
    [super viewDidDisappear:animated];
}
- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor whiteColor];
    UIView *view2 = [[UIView alloc] init];
    view2.backgroundColor = [UIColor greenColor];
    view2.frame = CGRectMake(150, 150, 100, 100);
    [self.view addSubview:view2];
}
@end
```

3. 快捷键

```
command+R 改完代码自动运行
shift+command+o UITabBarController 这个是 open quickly 的快捷键，可以在里面输入类和函数，快速的进行跳
```

### 二、常见 app 页面分析

1. UITabBarController = ViewControllers（ UIVIewContoller） + TabBar（UITabBar）

```
    self.window = [[UIWindow alloc] initWithWindowScene:(UIWindowScene *)scene];
    UITabBarController *tabbarController = [[UITabBarController alloc] init];

    UIViewController *controller1 = [[UIViewController alloc] init];
    controller1.view.backgroundColor = [UIColor redColor];
    controller1.tabBarItem.title = @"新闻";
    controller1.tabBarItem.image = [UIImage imageNamed:@"icon.bundle/home@2x.png"];
    controller1.tabBarItem.selectedImage = [UIImage imageNamed:@"icon.bundle/home_selected@2x.png"];

    UIViewController *controller2 = [[UIViewController alloc] init];
    controller2.view.backgroundColor = [UIColor greenColor];
    controller2.tabBarItem.title = @"视频";

    UIViewController *controller3 = [[UIViewController alloc] init];
    controller3.view.backgroundColor = [UIColor blueColor];
    controller3.tabBarItem.title = @"推荐";

    UIViewController *controller4 = [[UIViewController alloc] init];
    controller4.view.backgroundColor = [UIColor yellowColor];
    controller4.tabBarItem.title = @"我的";

    [tabbarController setViewControllers:@[controller1, controller2, controller3, controller4]];

    self.window.rootViewController = tabbarController;
    [self.window makeKeyAndVisible];
```


### 三、ios基本页面的构成和实现

1. UIWindow 
```
M+ V+ C
UIWindow （特殊形式的UIView，提供App中展示内容的基础窗口）

使用storyborad会帮我们自动创建
手动创建：创建UIWindow、设置rootViewCotroller、makeKeyAndVisiable
```

2. delegate设计模式
```
设计者  
提供一些使用者可以自定义的操作
@optional/@required注解
提供@property-delegate
在对应的时机，让delegate执行对应的方法

使用者
delegate = self按需实现方法
```
3. UITableView简介
```
系统的列表类型组件，提供默认基础的UITableViewCell样式、header footer的管理，点击等。
UITableViewCell: 系统提供复用回收池子
NSINDEXPath（UIKitAddtion）
```
基于dataSource以及delegate驱动的
4. UICollectionView  & UICollectionViewLayout
```
灵活的、可定制的列表类型组件
UITableView 特殊布局的UICollectionView
```
5. UIScrollView
UILabel
UIImage
UIImageView



