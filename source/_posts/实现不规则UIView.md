---
title: UIView形状
date: 2017-11-01 23:20:13
tags:
---
```Objective-c
#import <UIKit/UIKit.h>
#import <QuartzCore/QuartzCore.h>

@interface UIView (Shape)

- (void)setShape:(CGPathRef)shape;
@end

#import "UIView+Shape.h"

@implementation UIView (Shape)

- (void)setShape:(CGPathRef)shape
{
    if (shape == nil) {
        self.layer.mask = nil;
    }

    CAShapeLayer* maskLayer = [CAShapeLayer layer];
    maskLayer.path = shape;
    self.layer.mask = maskLayer;
}

@end
```

Shape相关
```Objective-C
@interface UIBezierPath (BasicShape)

+ (UIBezierPath *)cutCorner:(CGRect)originalFrame length:(CGFloat)length;
@end

#import "UIBezierPath+BasicShape.h"

@implementation UIBezierPath (BasicShape)

+ (UIBezierPath *)cutCorner:(CGRect)originalFrame length:(CGFloat)length
{
    CGRect rect = originalFrame;
    UIBezierPath *bezierPath = [UIBezierPath bezierPath];

    [bezierPath moveToPoint:CGPointMake(0, length)];
    [bezierPath addLineToPoint:CGPointMake(length, 0)];
    [bezierPath addLineToPoint:CGPointMake(rect.size.width - length, 0)];
    [bezierPath addLineToPoint:CGPointMake(rect.size.width, length)];
    [bezierPath addLineToPoint:CGPointMake(rect.size.width, rect.size.height - length)];
    [bezierPath addLineToPoint:CGPointMake(rect.size.width - length, rect.size.height)];
    [bezierPath addLineToPoint:CGPointMake(length, rect.size.height)];
    [bezierPath addLineToPoint:CGPointMake(0, rect.size.height - length)];
    [bezierPath closePath];
    return bezierPath;
}

@end
```

让自定义 Button 响应自定义 Shape 内的点击事件
```Objective-C
#import <UIKit/UIKit.h>
#import "UIView+Shape.h"
#import "UIBezierPath+BasicShape.h"

@interface RFButton : UIButton{
    CGPathRef path;
}

@end

#import "RFButton.h"

@implementation RFButton

- (id)initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:frame];
    if (self) {
        // Initialization code
    }
    return self;
}

- (void)setShape:(CGPathRef)shape
{
    [super setShape:shape];
    path = shape;
}

- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event
{
    if (CGPathIsEmpty(path)) {
        return YES;
    }
    //判断触发点是否在规定的 Shape 内
    if (CGPathContainsPoint(path, nil, point, nil)) {
        return YES;
    }
    return NO;
}
@end
```
