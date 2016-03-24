#iOS编码规范
iOS代码整体要符合<a href="https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html">苹果官方的代码规范</a><br>

#####1、声明方法或类时，请注意空格的使用，参数过多时可以换行保持对齐
```objective-c
- (instancetype)initWithName:(NSString*)name
                         age:(NSInteger)age
                      gender:(MQGender)gender;
```

#####方法的调用也一样<br>
```objective-c
MQPerson *person = [[MQPerson alloc] initWithName:@"小天"
                                              age:25
                                           gender:kMQGenderMale];
```

#####2、变量命名规则一般采用首字母小写的驼峰形式，定义的成员变量必须以’_’开头，常量以k开头，一般能用property的都应该用property而不使用自定义成员变量<br>
```objective-c
@private NSString *_name;

static const NSString *kMQServerHost = @"http://192.168.0.110";

@property (copy, nonatomic, readonly) NSString *name;
@property (assign, nonatomic, readonly) NSInteger age;
@property (assign, nonatomic, readonly) MQGender gender;
@property (copy, nonatomic) NSString *phoneNumber;

```
#####3、注释很重要，但除了开头的版权声明，尽可能把代码写的如同文档一样，让别人直接看代码就知道意思，写代码时别担心名字太长，相信Xcode的提示功能<br>

#####4、要保证头文件的简洁性，不公开的方法应该写在m文件中<br>

#####5、一般delegete的类型应该为weak弱引用，以避免产生循环饮用<br>
```objective-c
@property (weak, nonatomic) id<UITableViewDelegate> tableViewDelegate;
```
#####6、weak的使用要注意场合，除非是为了避免循环饮用，一般不建议用weak属性，使用strong提高程序效率<br>

#####7、关于copy 的使用，一般拥有可变类型的时候，属性申明应该为copy，比如NSString、NSDictionary、NSSet等都有可变的版本NSMutableArray、NSMutableDictionary、NSMutableSet<br>
```objective-c
@property (copy, nonatomic, readonly) NSArray<NSString*> *names;
@property (copy, nonatomic, readonly) NSDictionary<NSString*, NSString*> *namePhoneNumberPairs;
@property (copy, nonatomic, readonly) NSSet<NSString*> *uniqueNames;
```
#####要支持copy，必须要实现NSCoping协议<br>
```objective-c
@interface MQPerson:NSObject <NSCopying>

- (instancetype)copyWithZone:(NSZone *)zone {
    MQPerson *instance = [[[self class] allocWithZone:zone] initWithName:self.name
                                                                     age:self.age
                                                                  gender:self.gender];
    return instance;
}
```
#####8、如果想扩展一个类，一般使用category，扩展的文件名一般为NSString+(MQHash)类型，扩展的方法名必须要有前缀比如mq_hash，防止和其他扩展冲突<br>

#####9、左花括号不换行<br>
```objective-c
- (void)viewWillLayoutSubviews {
    [super viewWillLayoutSubviews];
    self.tableView.frame = self.view.bounds;
}
```

#iOS App客户端规范
#####Controller中代码差不多应该是这样子：<br>
```objective-c
#pragma mark life cycle
- (void)viewDidLoad {
    [super viewDidLoad];
    [self.view addSubview:self.tableView];
}

- (void)viewWillLayoutSubviews {
    [super viewWillLayoutSubviews];
    self.tableView.frame = self.view.bounds;
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}



#pragma mark UITableViewDataSource
//methods

#pragma mark UITableViewDelegate
//methods

#pragma mark CustomDelegate
//methods

#pragma mark event response
//methods

#pragma mark private methods
//methods

#pragma mark getter and setter
- (MQPerson*)person {
    if (!_person) {
        _person = [[MQPerson alloc] initWithName:@"小天"
                                             age:25
                                          gender:kMQGenderMale];
    }
    return _person;
}

- (UITableView*)tableView {
    if (!_tableView) {
        _tableView = [[UITableView alloc] init];
        _tableView.delegate = self;
        _tableView.dataSource = self;
    }
    return _tableView;
}

```
#####1、所有的属性都使用getter和setter
不要在viewDidLoad里面初始化你的view然后再add，这样代码就很难看。在viewDidload里面只做addSubview的事情，然后在viewWillLayoutSubviews里面做布局的事情，最后在viewDidAppear里面做Notification的监听之类的事情。至于属性的初始化，则交给getter去做<br>
```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    [self.view addSubview:self.tableView];
}

- (void)viewWillLayoutSubviews {
    [super viewWillLayoutSubviews];
    self.tableView.frame = self.view.bounds;
}
```
#####1、getter和setter全部都放在最后
因为一个Controller很有可能会有非常多的view，就像上面给出的代码样例一样，如果getter和setter写在前面，就会把主要逻辑扯到后面去，其他人看的时候就要先划过一长串getter和setter，这样不太好。然后要求业务工程师写代码的时候按照顺序来分配代码块的位置，先是life cycle，然后是Delegate方法实现，然后是event response，然后才是getters and setters。这样后来者阅读代码时就能省力很多<br>

#####1、每一个delegate都把对应的protocol名字带上，delegate方法不要到处乱写，写到一块区域里面去
比如UITableViewDelegate的方法集就老老实实写上#pragma mark  UITableViewDelegate。这样有个好处就是，当其他人阅读一个他并不熟悉的Delegate实现方法时，他只要按住command然后去点这个protocol名字，Xcode就能够立刻跳转到对应这个Delegate的protocol定义的那部分代码去，就省得他到处找了<br>

#####3、event response专门开一个代码区域
所有button、gestureRecognizer的响应事件都放在这个区域里面，不要到处乱放<br>

#####4、关于private methods，正常情况下ViewController里面不应该写
不是delegate方法，不是event response方法，不是life cycle方法，就是private method了。对的，正常情况下ViewController里面一般是不会存在private methods的，这个private methods一般是用于日期换算、图片裁剪啥的这种小功能。这种小功能要么把它写成一个category，要么把他做成一个模块，哪怕这个模块只有一个函数也行<br>
ViewController基本上是大部分业务的载体，本身代码已经相当复杂，所以跟业务关联不大的东西能不放在ViewController里面就不要放。另外一点，这个private method的功能这时候只是你用得到，但是将来说不定别的地方也会用到，一开始就独立出来，有利于将来的代码复用<br>

#####5、关于View的布局
直接使用CGRectMake的话可读性很差，光看那几个数字，也无法知道view和view之间的位置关系。用Autolayout可读性稍微好点儿，但生成Constraint的长度实在太长，代码观感不太好。所以代码中一般不准使用CGRectMake布局，一律采用Autolayout，Autolayout可以使用Masonry，代码的可读性就能好很多<br>

#####6、关于xib，storyboard，代码的选择
1 一般不使用storyboard写界面<br>
2 对于复杂的，动态生成的界面，建议使用手工编写<br>
3 对于统一风格的按钮或者UI控件，建议使用手工代码构造，方便之后的修复和复用<br>
4 对于需要继承或者组合关系的UIView或UIViewController类，建议用手工编写<br>
5 对于简单的、静态的、非核心的界面，可以用xib<br>

#常用的第三方库
<a href="https://github.com/AFNetworking/AFNetworking">AFNetWorking－－－网络请求</a><br>
<a href="https://github.com/CoderMJLee/MJRefresh">MJRefresh－－－下拉刷新</a><br>
<a href="https://github.com/SnapKit/Masonry">Masonry－－－自动布局求</a><br>
<a href="https://github.com/jdg/MBProgressHUD">MBProgressHUD－－－各种提示框</a><br>
<a href="https://github.com/CoderMJLee/MJExtension">MJExtension－－－转换速度快、使用简单方便的字典转模型框架</a><br>
<a href="https://github.com/adad184/MMPopupView">MMPopupView－－－弹出框</a><br>
<a href="https://github.com/mutualmobile/MMDrawerController">MMDrawerController－－－抽屉布局控件</a><br>
<a href="https://github.com/mwaterfall/MWPhotoBrowser">MWPhotoBrowser－－－图片查看器</a><br>
<a href="https://github.com/chiunam/CTAssetsPickerController">CTAssetsPickerController－－－支持多选的图片选择器</a><br>
<a href="https://github.com/lukabernardi/LBBlurredImage">LBBlurredImage－－－图片模糊效果</a><br>
<a href="https://github.com/itouch2/PhotoTweaks">PhotoTweaks－－－图片裁剪</a><br>
<a href="https://github.com/honcheng/RTLabel">RTLabel－－－富文本</a><br>
<a href="https://github.com/imchenglibin/XTPageControl">XTPageControl－－－类似网易客户端的布局控件（page controllers）</a><br>
<a href="https://github.com/steipete/Aspects">Aspects－－－AOP</a><br>
<a href="https://github.com/ninjinkun/NJKWebViewProgress">NJKWebViewProgress－－－带有进度条的webView</a><br>
<a href="https://github.com/marcuswestin/WebViewJavascriptBridge">WebViewJavascriptBridge－－－JS和OC互相调用</a><br>
<a href="https://github.com/lxcid/LXReorderableCollectionViewFlowLayout">LXReorderableCollectionViewFlowLayout－－－支持拖动排序</a><br>
<a href="https://github.com/ReactiveCocoa/ReactiveCocoa">ReactiveCocoa－－－函数式编程</a><br>


