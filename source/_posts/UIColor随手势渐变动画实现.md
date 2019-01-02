---

title: UIColor随手势渐变动画实现

---
#### 在平常开发过程中，关于颜色的动画，只要使用UIView的animation方法就可以实现。但是有一种比较特殊的需求，就是颜色渐变效果的实现，颜色需要根据距离或者是手指拖动的范围来进行渐变。
- 在开始介绍颜色渐变动画之前，需要了解一个方法，将UIColor转换成RGBColor，因为我们如果要做渐变动画，需要使用RGB来做。
```
/** 将UIColor转换成RGBColor */
- (void)getRGBComponents:(CGFloat [3])components forColor:(UIColor *)color{
    CGColorSpaceRef rgbColorSpac = CGColorSpaceCreateDeviceRGB();
    unsigned char resultingPixel[4];
    CGContextRef context = CGBitmapContextCreate(&resultingPixel, 1, 1, 8, 4, rgbColorSpac, kCGImageAlphaNoneSkipLast);
    CGContextSetFillColorWithColor(context, [color CGColor]);
    CGContextFillRect(context, CGRectMake(0, 0, 1, 1));
    CGContextRelease(context);
    CGColorSpaceRelease(rgbColorSpac);
    for (int compont = 0; compont < 3; compont++) {
        components[compont] = resultingPixel[compont] / 255.0f;
    }
}
//具体的使用方法
CGFloat componts[3];
[self getRGBComponents:componts forColor:[UIColor blackColor]];
NSLog(@"%f %f %f", componts[0], componts[1], componts[2]);
```

- 我遇到的需求就是自定义UISegment,在左右滑动的时候，选项卡放大，同时字体颜色渐变，具体的实现方法如下，在scrollViewDidScroll方法中添加如下代码：
```
- (void)scrollViewDidScroll:(UIScrollView *)scrollView {
    CGFloat offsetX = scrollView.contentOffset.x;
    CGFloat scrollWidth = scrollView.frame.size.width;
    if (offsetX < 0) return;
    //获取当前index，和目标index
    int tempIndex = (offsetX / scrollWidth);
    if (tempIndex > _numberOfSegments - 2) return;
    //获取当前的view和目标View
    UILabel *leftView = _items[tempIndex].label;
    UILabel *rightView = _items[tempIndex + 1].label;
    //获取scale和color渐变的具体值0~1
    float leftColorValue = fmod((double)offsetX,scrollWidth) / scrollWidth;
    float leftScaleValue = _maxScale - fmod((double)offsetX,scrollWidth) / scrollWidth * (_maxScale - _minScale);
    float rightScaleValue = _minScale + fmod((double)offsetX,scrollWidth) / scrollWidth * (_maxScale - _minScale);
    //对当前view和目标view进行放大动画
    leftView.transform = CGAffineTransformMakeScale(leftScaleValue, leftScaleValue);
    rightView.transform = CGAffineTransformMakeScale(rightScaleValue, rightScaleValue);
    
    //获取当前NormalColor
    UIColor *normalColor = [self.attributesNormal objectForKey:NSForegroundColorAttributeName];
    UIColor *selectColor = [self.attributesSelected objectForKey:NSForegroundColorAttributeName];
    CGFloat normalColorComponents[3];
    CGFloat selectColorComponents[3];
    [self getRGBComponents:normalColorComponents forColor:normalColor];
    [self getRGBComponents:selectColorComponents forColor:selectColor];

    NSMutableDictionary *normalDic = [NSMutableDictionary dictionaryWithDictionary:_attributesNormal];
    NSMutableDictionary *selectDic = [NSMutableDictionary dictionaryWithDictionary:_attributesSelected];
    
    //取出变化范围
    CGFloat rDis = selectColorComponents[0] - normalColorComponents[0];
    CGFloat gDis = selectColorComponents[1] - normalColorComponents[1];
    CGFloat bDis = selectColorComponents[2] - normalColorComponents[2];
    
    normalDic[NSForegroundColorAttributeName] = [UIColor colorWithRed:selectColorComponents[0] - rDis * leftColorValue  green:selectColorComponents[1] - gDis * leftColorValue blue:selectColorComponents[2] - bDis * leftColorValue alpha:1];
    
    selectDic[NSForegroundColorAttributeName] = [UIColor colorWithRed:normalColorComponents[0] + rDis * leftColorValue  green:normalColorComponents[1] + gDis * leftColorValue blue:normalColorComponents[2] + bDis * leftColorValue alpha:1];
    
    leftView.attributedText = [[NSMutableAttributedString alloc] initWithString:leftView.attributedText.string  attributes:normalDic];
    
    rightView.attributedText = [[NSMutableAttributedString alloc] initWithString:rightView.attributedText.string  attributes:selectDic];
}
```
#### 以上就完成了整个颜色渐变的过程
