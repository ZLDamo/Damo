---
layout: post
title:  "Google OC代码规范"
date:   2017-09-06 10:35:21 +0800
categories: jekyll update
---

##示例
---
下面是一个示例头文件，演示了@interface声明的正确注释和间隔
```
// GOOD:

#import <Foundation/Foundation.h>

@class Bar;

/**
 * 示例类演示了良好的Objective-C的代码风格。所有的接口,
 * 类别和协议(读取:在头文件中的所有非凡的顶级声明)必须注释,
 * 注释也必须与它们所记录的对象相邻。
 */
@interface Foo : NSObject

/** The retained Bar. */
@property(nonatomic) Bar *bar;

/** The current drawing attributes. */
@property(nonatomic, copy) NSDictionary<NSString *, NSNumber *> *attributes;

/**
 * Convenience creation method.
 * See -initWithBar: for details about @c bar.
 *
 * @param bar The string for fooing.
 * @return An instance of Foo.
 */
+ (instancetype)fooWithBar:(Bar *)bar;

/**
 * Designated initializer.
 *
 * @param bar A string that represents a thing that does a thing.
 */
- (instancetype)initWithBar:(Bar *)bar;

/**
 * Does some work with @c blah.
 *
 * @param blah
 * @return YES if the work was completed; NO otherwise.
 */
- (BOOL)doWorkWithBlah:(NSString *)blah;

@end
```

一个示例源文件，演示了一个接口的@ implementation的正确的注释和间隔。

```
// GOOD:

#import "Shared/Util/Foo.h"

@implementation Foo {
  /** The string used for displaying "hi". */
  NSString *_string;
}

+ (instancetype)fooWithBar:(Bar *)bar {
  return [[self alloc] initWithBar:bar];
}

- (instancetype)init {
  // Classes with a custom designated initializer should always override
  // the superclass's designated initializer.
  return [self initWithBar:nil];
}

- (instancetype)initWithBar:(Bar *)bar {
  self = [super init];
  if (self) {
    _bar = [bar copy];
    _string = [[NSString alloc] initWithFormat:@"hi %d", 3];
    _attributes = @{
      @"color" : [UIColor blueColor],
      @"hidden" : @NO
    };
  }
  return self;
}

- (BOOL)doWorkWithBlah:(NSString *)blah {
  // Work should be done here.
  return NO;
}

@end
```

##间距和格式
---
####空格键 VS Tab 键
只使用空格，一次缩进2个空格。我们使用空格来缩进。不要在代码中使用制表符。当你点击tab键时，你应该设置你的编辑器来释放空格，并在行上减少拖尾空间
#### 行长
OC文件的最大长度为100
在Xcode中,你可以 设置Preferences > Text Editing > Page guide at column: 100来使行长错误更容易被发现
####方法声明和定义
在 - 或 + 和返回类型之间应该使用一个空格，参数列表中除了参数之外没有空格。
方法应该是这样的:
```
// GOOD:

- (void)doSomethingWithString:(NSString *)theString {
  ...
}
```
星号之前的间隔是可选的。在添加新代码时，要与周围文件的样式保持一致。
如果你有太多的参数要在一条行上，那么给一参一行是最好的选择。如果使用了多个行，则在参数之前使用冒号对齐。
```
// GOOD:

- (void)doSomethingWithFoo:(GTMFoo *)theFoo
                      rect:(NSRect)theRect
                  interval:(float)theInterval {
  ...
}
```
当第二个或后面的参数名称比第一个更长的时候，将第二个和后面的行缩进至少四个空格，保持冒号对齐
```
// GOOD:

- (void)short:(GTMFoo *)theFoo
          longKeyword:(NSRect)theRect
    evenLongerKeyword:(float)theInterval
                error:(NSError **)theError {
  ...
}
```
####条件语句
if, while, for, , switch和比较运算符后面加空格
```
// GOOD:

for (int i = 0; i < 5; ++i) {
}

while (test) {};
```
当循环体或条件语句适合于一行时，可以省略括号。
```
// GOOD:

if (hasSillyName) LaughOutLoud();

for (int i = 0; i < 10; i++) {
  BlowTheHorn();
}
```
```
// AVOID:

if (hasSillyName)
  LaughOutLoud();               // AVOID.

for (int i = 0; i < 10; i++)
  BlowTheHorn();                // AVOID.
```
如果If子句有else子句，那么这两个子句都应该使用括号。
```
// GOOD:

if (hasBaz) {
  foo();
} else {
  bar();
}
```
```
// AVOID:

if (hasBaz) foo();
else bar();        // AVOID.

if (hasBaz) {
  foo();
} else bar();      // AVOID.
```
对下一个案例的故意影响应该用注释记录下来，除非在下一次案例之前没有任何干预代码
```
// GOOD:

switch (i) {
  case 1:
    ...
    break;
  case 2:
    j++;
    // Falls through.
  case 3: {
    int k;
    ...
    break;
  }
  case 4:
  case 5:
  case 6: break;
}
```
####表达式
在二进制操作符和赋值上使用空格。为一元运算符省略空格。不要在括号内添加空格
```
// GOOD:

x = 0;
v = w * x + y / z;
v = -y * (x + z);
```
表达式中的因子可能省略空格
```
// GOOD:

v = w*x + y/z;
```
####方法调用
方法调用应该像方法声明那样格式化
当有选择格式样式时，遵循已经在给定源文件中使用的约定。调用应该在一行上有所有的参数
```
// GOOD:

[myObject doFooWith:arg1 name:arg2 error:arg3];
```
或者每一行有一个参数，冒号对齐
```
// GOOD:

[myObject doFooWith:arg1
               name:arg2
              error:arg3];
```
不要使用这些格式:
```
// AVOID:

[myObject doFooWith:arg1 name:arg2  // some lines with >1 arg
              error:arg3];

[myObject doFooWith:arg1
               name:arg2 error:arg3];

[myObject doFooWith:arg1
          name:arg2  // aligning keywords instead of colons
          error:arg3];
```
与声明和定义一样，当第一个关键字比其他关键字短时，将后面的行缩进至少4个空格，保持冒号对齐
```
// GOOD:

[myObj short:arg1
          longKeyword:arg2
    evenLongerKeyword:arg3
                error:arg4];
```
包含多个代码块的调用可能使它们的参数名在四个空间缩进中左对齐
####函数调用
函数调用应该包含尽可能多的参数，除了需要更短的行来清楚或说明参数。
函数参数的延续行可以缩进以与开括号对齐，或者可能有一个4个空格的缩进。
```
// GOOD:

CFArrayRef array = CFArrayCreate(kCFAllocatorDefault, objects, numberOfObjects,
                                 &kCFTypeArrayCallBacks);

NSString *string = NSLocalizedStringWithDefaultValue(@"FEET", @"DistanceTable",
    resourceBundle,  @"%@ feet", @"Distance for multiple feet");

UpdateTally(scores[x] * y + bases[x],  // Score heuristic.
            x, y, z);

TransformImage(image,
               x1, x2, x3,
               y1, y2, y3,
               z1, z2, z3);
```
使用带有描述性名称的局部变量来缩短函数调用和减少调用的嵌套
```
// GOOD:

double scoreHeuristic = scores[x] * y + bases[x];
UpdateTally(scoreHeuristic, x, y, z);
```
####异常

使用**@catch**和**@finally**标签在同一行上的异常格式。在@和打开括号( **{** )之间添加空格，以及@catch和捕获对象声明之间的空格。如果您必须使用OC异常，请按照以下方式格式化它们。但是，请注意避免抛出异常，原因是您不应该使用异常。
```
// GOOD:

@try {
  foo();
} @catch (NSException *ex) {
  bar(ex);
} @finally {
  baz();
}
```
####函数的长度

倾向于定义小而集中的功能。

长函数和方法有时是适当的，因此没有严格限制在函数长度上。如果一个函数超过40行，考虑它是否可以在不损害程序结构的情况下被分解。

即使你的长函数现在运行得很好，几个月后有人修改它可能会增加新的行为。这可能导致难以找到的bug。使您的函数保持简短和简单，使其他人更容易阅读和修改您的代码。

在更新遗留代码时，考虑将长时间的函数分解成更小、更易于管理的部分。
####垂直空格

有节制的使用垂直空格。为了让更多的代码更容易在屏幕上显示，避免在函数的括号内放置空白行。将空行限制在函数和逻辑组之间的一个或两个之间。

##命名
---
在合理的范围内，名字应该尽可能的描述。遵循标准 [Objective-C naming rules](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html).。避免非标准的缩写。不要担心节省空间，因为让你的代码立即被一个新读者理解是非常重要的。例如:
```
// GOOD:

// Good names.
int numberOfErrors = 0;
int completedConnectionsCount = 0;
tickets = [[NSMutableArray alloc] init];
userInfo = [someObject object];
port = [network port];
NSDate *gAppLaunchDate;
```
```
// AVOID:

// Names to avoid.
int w;
int nerr;
int nCompConns;
tix = [[NSMutableArray alloc] init];
obj = [someObject object];
p = [network port];
```
任何类、类别、方法、函数或变量名都应该在名称中使用所有大写字母和 [缩写词](https://en.wikipedia.org/wiki/Initialism) 。这遵循了苹果的标准，在一个名称中使用所有大写字母，如URL、ID、TIFF和EXIF。
C函数和typedef的名称应该大写，并使用驼峰大小写来处理周围的代码。

####文件名
文件名应该反映它们所包含的类实现的名称，包括案例。遵循您的项目使用的约定。文件扩展应该如下:

| Extension |	 Type |
|:------------|------- |
| .h | C/C++/Objective-C header file | 	
|.m|	Objective-C implementation file|
|.mm|	Objective-C++ implementation file|
|.cc|	Pure C++ implementation file|
|.c|	C implementation file|

包含跨项目或大型项目中共享的代码的文件应该具有一个明确的惟一名称，通常包括项目或类前缀。
类的文件名应该包括被扩展的类的名称，比如GTMNSString+Utils.h或NSTextView + GTMAutocomplete.h

####类名
类名(以及类别和协议名称)应该以大写形式开始，并使用混合大小写来限制单词。
在设计跨多个应用程序共享的代码时，可以接受和推荐前缀(例如GTMSendMessage)。对于依赖于外部库的大型应用程序的类，还建议使用前缀。

####分类名
类别名应该从一个3个字符的前缀开始，将类别标识为项目的一部分，或者用于一般用途。

类别名应该包含它所扩展的类的名称。例如，如果我们想要在NSString上创建一个类别来进行解析，我们将把这个类别放入一个名为NSString+GTM.h解析的文件中,类别本身会被命名为GTMNSStringParsingAdditions,文件名称和类别可能不匹配，因为这个文件可能有许多与解析相关的单独类别。方法类别应该共享前缀(gtm_myCategoryMethodOnAString:)为了防止碰撞在Objective - C的全局名称空间。

在类名和类的开括号之间应该有一个空格
```
// GOOD:

// Using a category to extend a Foundation class.
@interface NSString (GTMNSStringParsingAdditions)
- (NSString *)gtm_parsedString;
@end
```

####OC方法名
方法和参数名通常以小写形式开始，然后使用混合大小写。

适当的大写应该受到尊重，包括名字的开头。
```
// GOOD:

+ (NSURL *)URLWithString;
```
如果可能的话，方法名应该像一个句子一样读，这意味着您应该选择使用方法名流的参数名。OC的方法名称往往很长，但是这有一个好处，那就是代码块几乎可以像散文一样读懂，因此很多实现注释都是不必要的。

使用介词和连接词，如“and”、“from”和“to”，在第二个和后面的参数名中，只有在必要的时候才可以澄清方法的含义或行为
```
// GOOD:

- (void)addTarget:(id)target action:(SEL)action;                          // GOOD; no conjunction needed
- (CGPoint)convertPoint:(CGPoint)point fromView:(UIView *)view;           // GOOD; conjunction clarifies parameter
- (void)replaceCharactersInRange:(NSRange)aRange
            withAttributedString:(NSAttributedString *)attributedString;  // GOOD.
```
返回对象的方法应该有一个名称，开头是一个名词，标识返回的对象:
```
// GOOD:

- (Sandwich *)sandwich;      // GOOD.
```
```
// AVOID:

- (Sandwich *)makeSandwich;  // AVOID.
```
访问器方法应该与得到的对象相同，但是不应该使用get来前缀。例如
```
// GOOD:

- (id)delegate;     // GOOD.
```

```
// AVOID:

- (id)getDelegate;  // AVOID.
```

返回布尔形容词值的访问器有方法名，但这些方法的属性名省略了is。
点表示法只用于属性名，而不使用方法名
```
// GOOD:

@property(nonatomic, getter=isGlorious) BOOL glorious;
- (BOOL)isGlorious;

BOOL isGood = object.glorious;      // GOOD.
BOOL isGood = [object isGlorious];  // GOOD.
```
```
// AVOID:

BOOL isGood = object.isGlorious;    // AVOID.
```
```
// GOOD:

NSArray<Frog *> *frogs = [NSArray<Frog *> arrayWithObject:frog];
NSEnumerator *enumerator = [frogs reverseObjectEnumerator];  // GOOD.
```

```
// AVOID:

NSEnumerator *enumerator = frogs.reverseObjectEnumerator;    // AVOID.
```
请参阅 [Apple's Guide to Naming Methods](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingMethods.html#//apple_ref/doc/uid/20001282-BCIGIJJF)，以获得更多关于Objective-C命名的详细信息。
这些指导原则仅适用于OC方法。C++方法名继续遵循C++风格指南中设置的规则。

####函数名
正则函数有混合的情况。

通常，函数应该以大写字母开头，并为每个新单词都有大写字母(也就是"[Camel Case](https://en.wikipedia.org/wiki/Camel_case)" ”或“Pascal案例”)
```
// GOOD:

static void AddTableEntry(NSString *tableEntry);
static BOOL DeleteFile(char *filename);
```
因为OC不提供命名空间，非静态函数应该有一个前缀，这样可以减少名称冲突的几率。
```
// GOOD:

extern NSTimeZone *GTMGetDefaultTimeZone();
extern NSString *GTMGetURLScheme(NSURL *URL);
```
####变量名
变量名通常以小写字母开头，并使用混合大小写来限制单词。

实例变量有突出的下划线。文件范围或全局变量有一个前缀g，例如:mylocal变量、_myInstanceVariable、gmyglobal变量。

######常见的变量名
读者应该能够从名称中推断变量类型，但是不要使用系统符号来表示语法属性，例如变量的静态类型(int或指针)。

在方法或函数范围之外声明的文件范围或全局变量(而不是常量)应该很少见，并且应该具有前缀g。
```
// GOOD:

static int gGlobalCounter;
```

######实例变量
实例变量名是混合用例，应该使用下划线进行前缀，比如_usernameTextField。

注意:谷歌之前对OC的惯例是一个拖尾下划线。现有的项目可能会选择在新代码中继续使用拖尾下划线，以保持项目代码库中的一致性。前缀或后缀的一致性应该在每个类中保持。

######常量
常量符号(const的全局变量和静态变量和用定义创建的常量)应该使用混合大小写来限制单词。

常量应该有一个适当的前缀
```
// GOOD:

extern NSString *const GTLServiceErrorDomain;

typedef NS_ENUM(NSInteger, GTLServiceError) {
  GTLServiceErrorQueryResultMissing = -3000,
  GTLServiceErrorWaitTimedOut       = -3001,
};
```
因为OC不提供命名空间，所以使用外部链接的常量应该有一个前缀，它可以最小化名称冲突的机会，通常就像ClassNameConstantName或ClassNameEnumName一样。

对于与Swift代码的互操作性，枚举值应该具有扩展typedef名称的名称:
```
// GOOD:

typedef NS_ENUM(NSInteger, DisplayTinge) {
  DisplayTingeGreen = 1,
  DisplayTingeBlue = 2,
};
```
常量可以在适当的时候使用小写的k前缀:
```
// GOOD:

static const int kFileCount = 12;
static NSString *const kUserKey = @"kUserKey";
```

##类型和声明
----

####局部变量
在最窄的实际范围内声明变量，并接近它们的使用。在声明中初始化变量。
```
// GOOD:

CLLocation *location = [self lastKnownLocation];
for (int meters = 1; meters < 10; meters++) {
  reportFrogsWithinRadius(location, meters);
}
```

有时候，效率会使在其使用范围之外声明一个变量更加合适。这个示例声明了不同的初始化，并且每次通过循环都不必要地发送lastlocation消息:
```
// AVOID:

int meters;                                         // AVOID.
for (meters = 1; meters < 10; meters++) {
  CLLocation *location = [self lastKnownLocation];  // AVOID.
  reportFrogsWithinRadius(location, meters);
}
```

在自动引用计数中，指向OC对象的指针默认初始化为nil，因此不需要显式地初始化到nil

####Unsigned Integers
避免无符号整数，除非系统接口使用的类型匹配。

当使用无符号整数时，会出现一些微妙的错误。除了在系统接口中匹配NSUInteger时，只需要在数学表达式中使用已签名的整数

```
// GOOD:

NSUInteger numberOfObjects = array.count;
for (NSInteger counter = numberOfObjects - 1; counter > 0; --counter)
```
```
// AVOID:

for (NSUInteger counter = numberOfObjects - 1; counter > 0; --counter)  // AVOID.
```

无符号整数可用于标记和位掩码，不过通常NS_OPTIONS或NS_ENUM会更合适

####类型与大小不一致
由于在32位和64位构建中存在不同的大小，所以除了匹配系统接口之外，避免类型long、NSInteger、NSUInteger和CGFloat。

类型long、NSInteger、NSUInteger和CGFloat在32-64位构建之间的大小有所不同。在处理由系统接口公开的值时，使用这些类型是合适的，但是在大多数其他计算中应该避免使用这些类型

```
// GOOD:

int32_t scalar1 = proto.intValue;

int64_t scalar2 = proto.longValue;

NSUInteger numberOfObjects = array.count;

CGFloat offset = view.bounds.origin.x;
```
```
// AVOID:

NSInteger scalar2 = proto.longValue;  // AVOID.
```
文件和缓冲区大小通常超过32位限制，因此应该使用int64t声明它们，而不是使用long、NSInteger或NSUInteger。

##注释
---
注释对于保持我们的代码可读性是至关重要的。下面的规则描述了您应该如何评论和在哪里。但是请记住:虽然注释很重要，但最好的代码是自文档化。给类型和变量提供合理的名称要比使用模糊的名称好得多，然后试图通过注释来解释它们。

注意标点、拼写和语法;读写得好的评论比写得不好的评论要容易得多。

注释应该像叙述性文本一样可读，有适当的大小写和标点符号。在很多情况下，完整的句子比句子片段更容易读懂。较短的注释，例如代码行末尾的注释，有时可能不那么正式，但是使用一致的样式。当你写你的评论时，为你的读者写:下一个需要理解你的代码的贡献者。下一个可能是你！

####文档注释
文件可以从对其内容的描述开始。每个文件都可能包含以下内容:

- 许可样板如果必要的话。为项目使用的许可选择适当的样板文件。
- 必要时对文件内容的基本描述。
如果您对带有作者行的文件进行了重大更改，请考虑删除作者行，因为修订历史已经提供了更详细、更准确的作者身份记录。

####声明注释
每一个非凡的界面，公共的和私有的，都应该有一个附带的评论来描述它的目的以及它是如何与更大的场景相适应的。

注释应该用于文档类、属性、ivars、函数、类别、协议声明和枚举。
```
// GOOD:

/**
 * A delegate for NSApplication to handle notifications about app
 * launch and shutdown. Owned by the main app controller.
 */
@interface MyAppDelegate : NSObject {
  /**
   * The background task in progress, if any. This is initialized
   * to the value UIBackgroundTaskInvalid.
   */
  UIBackgroundTaskIdentifier _backgroundTaskID;
}

/** The factory that creates and manages fetchers for the app. */
@property(nonatomic) GTMSessionFetcherService *fetcherService;

@end
```

在Xcode解析这些接口以显示格式化的文档时，会鼓励使用doxygen风格的注释。有各种各样的Doxygen命令;在项目中始终如一地使用它们。

如果您已经在文件顶部的注释中详细描述了一个接口，那么可以简单地声明“在文件的顶部看到注释”，但是一定要有一些注释。

另外，每个方法都应该有一个注释来解释它的函数、参数、返回值、线程或队列假设以及任何副作用。文档注释应该放在公共方法的标题中，或者在方法之前，用于非平凡的私有方法。

使用描述性形式(“打开文件”)而不是命令形式(“打开文件”)来获取方法和函数注释。注释描述了这个函数;它没有告诉函数该做什么。

如果有的话，记录线程的使用假设，即类、属性或方法。如果一个类的实例可以被多个线程访问，那么就需要格外注意来记录多线程使用的规则和不变量。

任何属性和ivars的值，例如NULL或-1，都应该在注释中记录。

声明注释解释了如何使用方法或函数。注释解释方法或函数是如何实现的，应该与实现而不是声明一起使用。

####实现注释
提供注释解释复杂的、微妙的或复杂的代码段
```
// GOOD:

// Set the property to nil before invoking the completion handler to
// avoid the risk of reentrancy leading to the callback being
// invoked again.
CompletionHandler handler = self.completionHandler;
self.completionHandler = nil;
handler();
```

当有用的时候，也提供关于被考虑或放弃的实现方法的注释。

结束行注释应该与代码至少分隔两个空格。如果您对后续行有几条注释，则可以更容易地将它们排好

```
// GOOD:

[self doSomethingWithALongName];  // Two spaces before the comment.
[self doSomethingShort];          // More spacing to align the comment.
```

####有歧义的符号
在需要避免歧义的地方，使用反引号或竖条在注释中引用变量名和符号，以便使用引号或将符号命名为内联。

在Doxygen- Style的注释中，更喜欢使用单空间文本命令来分隔符号，比如@c。

当一个符号是一个常见的词，可能使这个句子读起来像它的结构很糟糕时，划分有助于提供清晰的信息。一个常见的例子是符号计数:

>// GOOD:
// Sometimes `count` will be less than zero.

或者引用已经包含引号的内容
>// GOOD:
// Remember to call `StringWithoutSpaces("foo bar baz")`

当一个符号是自显的时候，不需要反勾或竖条
>// GOOD:
// This class serves as a delegate to GTMDepthCharge.

Doxygen格式也适用于标识符号。
>// GOOD:
/** @param maximum The highest value for @c count. */

####目标拥有者
对于没有使用ARC管理的对象，当它超出了最常见的OC-用法惯例时，使指针明确的持有所有权模型。

######手动引用计数
NSObject派生对象的实例变量被假定为保留;如果它们没有被保留，它们应该被认为是弱的，或者是用__ weak限定符声明的。

在Mac软件中，一个例外是被标记为@IBOutlet的变量，这些变量被认为是不被保留的。

实例变量是指向Core Foundation、C++和其他非Objective-C对象的指针，它们应该总是用强而弱的注释声明，以指示哪些指针是不被保留的。Core Foundation和其他非Objective-C对象指针需要显式的内存管理，即使在构建自动引用计数时也是。

强而弱的声明的例子:
```
// GOOD:

@interface MyDelegate : NSObject

@property(nonatomic) NSString *doohickey;
@property(nonatomic, weak) NSString *parent;

@end


@implementation MyDelegate {
  IBOutlet NSButton *_okButton;  // Normal NSControl; implicitly weak on Mac only

  AnObjcObject *_doohickey;  // My doohickey
  __weak MyObjcParent *_parent;  // To send messages back (owns this instance)

  // non-NSObject pointers...
  CWackyCPPClass *_wacky;  // Strong, some cross-platform object
  CFDictionaryRef *_dict;  // Strong
}
@end
```

######自动引用计数
当使用ARC时，对象的所有权和生命周期是明确的，因此自动保留的对象不需要额外的注释。

##C语言特性
---
####宏命令
避免使用宏，特别是在const变量、枚举、XCode代码片段或C函数的情况下。

宏使您看到的代码与编译器看到的代码有所不同。现代C将传统的宏用于常量和不必要的实用函数。只有当没有其他可用的解决方案时，才应该使用宏。

在需要宏的地方，使用一个惟一的名称来避免编译单元中符号冲突的风险。如果实际操作，则在使用后不定义宏的范围限制范围。

宏名应该使用大写字母来和下划线表示。类似函数的宏可以使用C函数命名方法。不要定义看起来是C或objective-C关键字的宏
```
// GOOD:

#define GTM_EXPERIMENTAL_BUILD ...      // GOOD

// Assert unless X > Y
#define GTM_ASSERT_GT(X, Y) ...         // GOOD, macro style.

// Assert unless X > Y
#define GTMAssertGreaterThan(X, Y) ...  // GOOD, function style.
```

```
// AVOID:

#define kIsExperimentalBuild ...        // AVOID

#define unless(X) if(!(X))              // AVOID
```

避免那些扩展到不平衡C或objective-C结构的宏。避免引入范围的宏，或者可能模糊块中值的捕获。

避免在header中生成类、属性或方法定义的宏，以作为公共API使用。这些代码只会让代码难以理解，而且语言已经有了更好的方法来实现这一点。

避免产生方法实现的宏，或者生成稍后在宏之外使用的变量的声明。宏不应该通过隐藏变量的位置和方式来让代码难以理解。
```
// AVOID:

#define ARRAY_ADDER(CLASS) \
  -(void)add ## CLASS ## :(CLASS *)obj toArray:(NSMutableArray *)array

ARRAY_ADDER(NSString) {
  if (array.count > 5) {              // AVOID -- where is 'array' defined?
    ...
  }
}
```

可以接受的宏使用的例子包括断言和调试日志宏，这些宏是根据构建设置有条件地编译的，这些宏通常都没有编译成版本。

####非标准扩展
除非另有说明，否则不能使用C/OC的非标准扩展。

编译器支持不属于标准c语言的各种扩展，包括复合语句表达式(例如`foo = ({ int x; Bar(&x); x }))`)和变长数组。

`__attribute__`是一个被批准的异常，因为它在OC API规范中被使用。

条件运算符的二进制形式，`A ?: B`，是一个批准的例外。

##Cocoa and Objective-C特征
---
####确定指定初始值
清楚地标识您的指定初始化器。

对于那些可能会子类化你的类的人来说，很重要的一点是，指定的初始化器被清楚地识别出来。这样，它们只需要覆盖一个单一的初始化器(可能是几个)，以保证其子类的初始化器被调用。它还可以帮助那些在将来调试类的人理解初始化代码的流程，如果他们需要的话。使用注释或`NS_DESIGNATED_INITIALIZER `初始化器宏来识别指定的初始化器。如果你使用`NS_DESIGNATED_INITIALIZER`初始化器，用`NS_UNAVAILABLE`标记不支持的初始化器。

####重写指定初始值
当编写一个需要init的子类时。方法，确保覆盖超类的指定初始化器。

如果您不能覆盖超类的指定初始化器，那么您的初始化器可能不会在所有的情况下被调用，这会导致难以找到bug的微妙和困难。

####重写NSObject方法
提供一个NSObject的重写方法在@implementation前
这通常适用于(但不限于)`init`,`copyWithZone:`和`dealloc`方法。`init`方法应该组合在一起，然后是其他典型的NSObject方法，比如`description`、`isEqual:`和`hash`。
便利类工厂方法可能在NSObject方法之前。

####初始化
不要在init方法中初始化实例变量为0或nil;这样做是多余的。

新分配的对象的所有实例变量都被初始化为0(除了isa)，所以不要通过重新初始化变量为0或nil来搞乱init方法。

####header中的实例变量应该是@protected或@private
实例变量通常应该在实现文件中声明，或者由属性自动合成。当在头文件中声明ivars时，应该将其标记为@protected或@private。
```
// GOOD:

@interface MyClass : NSObject {
 @protected
  id _myInstanceVariable;
}
@end
```

####Avoid +new
不要调用NSObject类方法，也不要在子类中重写它。相反，使用alloc和init方法来实例化保留对象。

现在OC代码显式地调用alloc和init方法来创建和保留对象。由于新的类方法很少被使用，所以它使得对正确的内存管理的代码检查变得更加困难。

####保持公共API的简单性
保持简单的类;避免“kitchen-sink”API。如果一个方法不需要公开，就把它放在公共接口之外。

与C++不同，OC不区分公共和私有方法;任何消息都可以发送到一个对象。因此，避免将方法放置在公共API中，除非它们实际被用于类的使用者使用。这有助于减少当你不期望时被调用的可能性。这包括从父类中覆盖的方法。

由于内部方法不是真正的私有的，所以很容易意外地覆盖超类的“私有”方法，从而使一个非常困难的bug发现。一般来说，私有方法应该有一个相当独特的名称，它可以防止子类无意地覆盖它们。

#### #import and #include
#import OC和Objective-C++头文件，并#include c/c++头文件。

在#import和#include之间进行选择，这是基于您所包含的消息头的语言。

当包含使用Objective-C++或OC的头时，使用#import。当包含标准C或C++头时，请使用#include。标题应该提供自己的定义保护。

####包括顺序
头包的标准顺序是相关的头、操作系统头、语言库头，以及其他依赖项的头组。

相关的头在其他的前面，以确保它没有隐藏的依赖项。对于实现文件，相关的头文件是头文件。对于测试文件，相关的头是包含测试接口的头。

空白行可以分隔包含标题的逻辑上不同的组。

```
// GOOD:

#import "ProjectX/BazViewController.h"

#import <Foundation/Foundation.h>

#include <unistd.h>
#include <vector>

#include "base/basictypes.h"
#include "base/integral_types.h"
#include "util/math/mathutil.h"

#import "ProjectX/BazModel.h"
#import "Shared/Util/Foo.h"
```

####为系统框架使用伞头
为系统框架和系统库导入伞头文件，而不是包含单独的文件。

虽然从诸如Cocoa或Foundation之类的框架中包含单独的系统头可能看起来很诱人，但实际上，如果您包含顶级根框架，那么它在编译器上的工作就会更少。根框架通常是预编译的，并且可以更快地加载。此外，请记住使用@import或import而不是OC frameworks。
```
// GOOD:

@import UIKit;     // GOOD.
#import <Foundation/Foundation.h>     // GOOD.
```
```
// AVOID:

#import <Foundation/NSArray.h>        // AVOID.
#import <Foundation/NSString.h>
...
```

####在init和dealloc中避免访问器
在init和dealloc方法执行过程中，实例的子类可能处于不一致的状态，所以这些方法中的代码应该避免调用self的访问方法。

当init和dealloc方法执行时，子类还没有被初始化或者已经进行了处理，使得访问器方法可能不可靠。在实际操作中，在这些方法中直接分配和释放ivars，而不是依赖访问器。
```
// GOOD:

- (instancetype)init {
  self = [super init];
  if (self) {
    _bar = 23;  // GOOD.
  }
  return self;
}
```
```
// AVOID:

- (instancetype)init {
  self = [super init];
  if (self) {
    self.bar = 23;  // AVOID.
  }
  return self;
}
```

```
// GOOD:

- (void)dealloc {
  [_notifier removeObserver:self];  // GOOD.
}
```

```
// AVOID:

- (void)dealloc {
  [self removeNotifications];  // AVOID.
}
```
####setter copy NSString
使用NSString的setter应该总是复制它所接受的字符串。这通常也适用于NSArray和NSDictionary之类的集合。

不要仅仅保留字符串，因为它可能是一个NSMutableString。这就避免了调用者在您不知情的情况下更改它。

接收和保存集合对象的代码也应该考虑到传递的集合可能是可变的，因此集合可以更安全地作为原始副本的副本或可变副本被保存。
```
// GOOD:

@property(nonatomic, copy) NSString *name;

- (void)setZigfoos:(NSArray<Zigfoo *> *)zigfoos {
  // Ensure that we're holding an immutable collection.
  _zigfoos = [zigfoos copy];
}
```

####使用轻量级泛型来记录包含的类型
所有在Xcode 7或更新版本上编译的项目都应该使用OC轻量级泛型表示法来输入包含的对象。

每个NSArray、NSDictionary或NSSet引用都应该使用轻量级泛型进行声明，以改进类型安全性，并显式地记录使用情况。
```
// GOOD:

@property(nonatomic, copy) NSArray<Location *> *locations;
@property(nonatomic, copy, readonly) NSSet<NSString *> *identifiers;

NSMutableArray<MyLocation *> *mutableLocations = [otherObject.locations mutableCopy];
```
如果完全注释的类型变得复杂，请考虑使用typedef来保持可读性
```
// GOOD:

typedef NSSet<NSDictionary<NSString *, NSDate *> *> TimeZoneMappingSet;
TimeZoneMappingSet *timeZoneMappings = [TimeZoneMappingSet setWithObjects:...];
```
使用最具描述性的公共超类或协议。在最常见的情况下，当没有其他已知的情况时，将该集合声明为使用id显式的异构性
```
// GOOD:

@property(nonatomic, copy) NSArray<id> *unknowns;
```

####避免抛出异常
不要`@throw` OC异常，但是您应该准备好从第三方或OS调用中捕获它们。

在此之前，在[Apple's Introduction to Exception Programming Topics for Cocoa](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Exceptions/Exceptions.html).中，建议使用错误对象来进行错误交付。

我们使用`-fobjc-exception`来编译(主要是我们得到`@synchronized`)，但是我们没有`@throw`。当需要使用第三方代码或库时，可以使用`@try`、`@catch`和`@finally`。如果您确实使用了它们，请准确地记录您希望抛出的方法。
####nil Checks
只对逻辑流使用`nil`检查。

使用`nil`指针检查应用程序的逻辑流，而不是在发送消息时防止崩溃。将消息发送给`nil` [reliably returns](http://www.sealiesoftware.com/blog/archive/2012/2/29/objc_explain_return_value_of_message_to_nil.html) nil，作为一个整数或浮点值，0作为一个值，它被初始化为`0`，而`_Complex`的值等于`{0,0}`。

注意，这适用于`nil`作为消息目标，而不是作为参数值。个别方法可能或不能安全地处理nil参数值。

请注意，这与检查C/C++指针和针对`NULL`的块指针是不同的，这是运行时不处理的，会导致应用程序崩溃。您仍然需要确保您不会取消一个空指针。
####BOOL陷阱
将一般值转换为`BOOL`时要小心。避免直接与`YES`进行比较。

在OS X和32位的iOS版本中，`BOOL`定义为一个已签名的字符，因此它可能具有除`YES(1)`和`NO(0)`之外的值，不要将一般值直接转换为`BOOL`。

常见的错误包括转换或转换数组的大小、指针值或对`BOOL`的逐位逻辑操作的结果，这取决于整数值的最后一个字节的值，仍然会导致没有值。当将一个一般值转换为`BOOL`时，使用三元运算符返回一个`YES`或`NO`值。

您可以安全地交换和转换`BOOL`、`_Bool`和`bool`;
(参见C++Std 4.7.4、4.12和C99 Std 6.3.1.2)。在Objective-C方法签名中使用BOOL。

使用BOOL使用逻辑运算符 (`&&`, `||` and`!`)也是有效的，并且将返回可以安全地转换为BOOL的值，而不需要使用三元运算符。

```
// AVOID:

- (BOOL)isBold {
  return [self fontTraits] & NSFontBoldTrait;  // AVOID.
}
- (BOOL)isValid {
  return [self stringValue];  // AVOID.
}
```

```
// GOOD:

- (BOOL)isBold {
  return ([self fontTraits] & NSFontBoldTrait) ? YES : NO;
}
- (BOOL)isValid {
  return [self stringValue] != nil;
}
- (BOOL)isEnabled {
  return [self isValid] && [self isBold];
}
```

另外，不要直接将`BOOL`变量直接与`YES`进行比较。对于那些精通C语言的人来说，阅读不仅困难，而且上面的第一点表明，返回值可能并不总是您所期望的

```
// AVOID:

BOOL great = [foo isGreat];
if (great == YES) {  // AVOID.
  // ...be great!
}
```

```
// GOOD:

BOOL great = [foo isGreat];
if (great) {         // GOOD.
  // ...be great!
}
```

####没有实例变量的接口
忽略不声明任何实例变量的接口上的空括号
```
// GOOD:

@interface MyClass : NSObject
// Does a lot of stuff.
- (void)fooBarBam;
@end
```

```
// AVOID:

@interface MyClass : NSObject {
}
// Does a lot of stuff.
- (void)fooBarBam;
@end
```

##Cocoa 模式
---
####代理模式
在这样做的时候，委托、目标对象和块指针不应该被保留，这样就可以创建一个保留循环。

为了避免引起一个保留循环，当一个委托或目标指针被清除时，就应该立即释放，这样就不再需要对对象进行消息传递了。

如果没有明确的时间，委托或目标指针不再需要，指针只应该被保留。

块指针不能被保留。为了避免在客户机代码中造成保留周期，块指针应该被用于回调，只有在它们被调用或者不再需要它们之后才可以显式地释放它们。否则，回调应该通过弱委托或目标指针来完成

##Objective-C++
----
####语言匹配风格
风格匹配语言

在一个Objective-C++源文件中，遵循您正在实现的函数或方法的语言的风格。为了减少在混合使用Cocoa/Objective-C++和C++时不同命名风格之间的冲突，请遵循所实现的方法的风格。

对于@实现块中的代码，使用OC命名规则。对于C++类方法中的代码，使用C++命名规则。
```
// GOOD:

// file: cross_platform_header.h

class CrossPlatformAPI {
 public:
  ...
  int DoSomethingPlatformSpecific();  // impl on each platform
 private:
  int an_instance_var_;
};

// file: mac_implementation.mm
#include "cross_platform_header.h"

// A typical Objective-C class, using Objective-C naming.
@interface MyDelegate : NSObject {
 @private
  int _instanceVar;
  CrossPlatformAPI* _backEndObject;
}

- (void)respondToSomething:(id)something;

@end

@implementation MyDelegate

- (void)respondToSomething:(id)something {
  // bridge from Cocoa through our C++ backend
  _instanceVar = _backEndObject->DoSomethingPlatformSpecific();
  NSString* tempString = [NSString stringWithFormat:@"%d", _instanceVar];
  NSLog(@"%@", tempString);
}

@end

// The platform-specific implementation of the C++ class, using
// C++ naming.
int CrossPlatformAPI::DoSomethingPlatformSpecific() {
  NSString* temp_string = [NSString stringWithFormat:@"%d", an_instance_var_];
  NSLog(@"%@", temp_string);
  return [temp_string intValue];
}
```
项目可能选择使用80列的行长度限制来与Google的C++风格指南保持一致
##OC风格的异常
----
####显示风格异常
没有期望遵循这些样式推荐的代码行，在行末尾的行末尾或` // NOLINTNEXTLINE`的后面，需要`// NOLINT`。有时，OC代码部分必须忽略这些样式的建议(例如，代码可能是机器生成的，或者代码结构是不可能正确的风格)。

在那一行上的`// NOLINT`或前一行的`// NOLINTNEXTLINE`评论可以用来指示读者，代码是故意忽略样式的指导方针的。此外，这些注释还可以通过诸如linters和正确处理代码之类的自动化工具来获得。注意，在`//`和`NOLINT`之间有一个空格。