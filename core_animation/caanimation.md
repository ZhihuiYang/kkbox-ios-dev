CAAnimation
-----------

要讓一個 CALayer 動起來，除了透過改變 layer 的 animanatable 的屬性外，
就是建立 CAAnimation 物件，然後對 layer 呼叫 `-addAnimation:forKey:`。
我們通常會選擇一種 CALayer 的 subclass，製作我們想要的動畫效果。

### CATransition

我們先從 Core Animation 內建的轉場效果 CATransition 講起（請不要跟
CATransaction 搞混），因為 CATransition 大概是最容易上手，而且可以馬上
有成就感的動畫。CATransition 通常用在兩個 view 之間的切換；UIView 本身
就有定義一些跟轉場有幾個以 transition 開頭的 class method，像是

- `+ transitionWithView:duration:options:animations:completion:`
- `+ transitionFromView:toView:duration:options:completion:`

而這些 method 可以設定的 tansition option 包括：

- `UIViewAnimationOptionTransitionNone`
- `UIViewAnimationOptionTransitionFlipFromLeft`
- `UIViewAnimationOptionTransitionFlipFromRight`
- `UIViewAnimationOptionTransitionCurlUp`
- `UIViewAnimationOptionTransitionCurlDown`
- `UIViewAnimationOptionTransitionCrossDissolve`
- `UIViewAnimationOptionTransitionFlipFromTop`
- `UIViewAnimationOptionTransitionFlipFromBottom`

也就是說，UIView 本身就已經提供了四種方向的翻轉（上下左右）、淡入淡出
與翻頁幾種轉場效果。其實這些效果都是用 Core Animation 實作的，而
CATransition 所提供的效果也遠比這些多，只要我們知道怎樣使用
CATransition，就可以使用更多的轉場效果。

我們來寫一個簡單的 view controller：這個 view controller 的 view 上面
我們額外建立一個 CALayer，然後加了一個按鈕，這個按鈕按下去之後的
action 是讓這個 layer 在紅色與綠色之間切換。

ViewController.h

``` objc
@import UIKit;
@import QuartzCore;

@interface ViewController : UIViewController
- (IBAction)toggleColor:(id)sender;
@end
```

ViewController.m

``` objc
#import "ViewController.h"

@interface ViewController ()
@property (strong, nonatomic) CALayer *aLayer;
@property (assign, nonatomic) BOOL isGreen;
@end

@implementation ViewController
- (void)viewDidLoad
{
	[super viewDidLoad];
	self.aLayer = [[CALayer alloc] init];
	self.aLayer.frame = CGRectMake(50, 50, 100, 100);
	self.aLayer.backgroundColor = [UIColor redColor].CGColor;
	[self.view.layer addSublayer:self.aLayer];
}

- (IBAction)toggleColor:(id)sender
{
	self.isGreen = !self.isGreen;
	self.aLayer.backgroundColor = self.isGreen ? [UIColor greenColor].CGColor : [UIColor redColor].CGColor;
}
@end
```

<iframe width="350" height="621"
src="https://www.youtube.com/embed/h5PfrQ3OevA?rel=0&amp;showinfo=0"
frameborder="0" allowfullscreen></iframe>

原本這個版本只會產生 0.25 秒的屬性變化，使用的是淡入淡出效果。我們來加
入 `kCATransitionMoveIn` 這個轉場效果看看，就會變成新的畫面從上方推入。

``` objc
- (IBAction)toggleColor:(id)sender
{
	self.isGreen = !self.isGreen;
	self.aLayer.backgroundColor = self.isGreen ? [UIColor greenColor].CGColor : [UIColor redColor].CGColor;
	CATransition *transition = [[CATransition alloc] init];
	transition.type = kCATransitionMoveIn;
	transition.subtype = kCATransitionFromRight;
	[self.aLayer addAnimation:transition forKey:@"tansition"];
}
```

<iframe width="350" height="621"
src="https://www.youtube.com/embed/EtrvXUZJhjY?rel=0&amp;showinfo=0"
frameborder="0" allowfullscreen></iframe>

CATransition 可以應用的地方不只是切換背景顏色而已，無論是改變這個
layer 的任何 animatable 的屬性，像是改變 contents，或是在這個 layer 中
增加或移除 sublayer，用 `addAnimation:forKey:` 加入了 CATransition 物
件之後，都可以產生轉場效果。

如果我們想要像 UIView 的
`+transitionWithView:duration:options:animations:completion:` 那樣，是
在某個 view 上面增加或移除 subview 的時候產生轉場效果，用 CATransition
的作法也差不多，我們先寫一好要某個 view A 新增或移除 subview，然後把
CATransition 加到 view A 的 layer 上即可。

蘋果在 iOS 上的 CATransition 的 public header 中定義了幾個 type：

- `kCATransitionFade`：淡入淡出動畫
- `kCATransitionMoveIn`：新的畫面從上方移入的效果
- `kCATransitionPush`：推擠效果，很像 navigation controller 的換頁
- `kCATransitionReveal`：原本的畫面從上方移出的效果

不過…其實蘋果有不少 private API 可以用。比方說，我們可以把 type 指定成
立體塊狀翻轉效果。

``` objc
transition.type = @"cube";
```

<iframe width="350" height="621"
src="https://www.youtube.com/embed/YI1wTK1xbok?rel=0&amp;showinfo=0"
frameborder="0" allowfullscreen></iframe>

在 http://iphonedevwiki.net/index.php/CATransition 這一頁上可以看到整
理好的 private API 可以呼叫。雖然蘋果的政策是禁止在上架的 app 中呼叫
private API，如果用了可能會被 reject…不過印象中其實有很多 app 都用到這
些 private API。

如果對內建的這些效果，甚至 private 的效果都不滿意的話，iOS 5 之後，
CATransition 還有一個叫做 filter 的屬性，我們可以在這個屬性上加上
CIFilter 物件，客製更多的轉場效果。

### CAPropertyAnimation

### CAKeyframeAnimation

### CAAnimationGroup