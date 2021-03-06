本工程为基于高德地图iOS SDK进行封装，实现了点击选中overlay的功能。
## 前述 ##
- [高德官网申请Key](http://lbs.amap.com/dev/#/).
- 阅读[开发指南](http://lbs.amap.com/api/ios-sdk/summary/).
- 工程基于iOS 3D地图SDK实现.

## 功能描述 ##
基于3D地图SDK=进行封装，实现了点击选中overlay的功能。

## 核心类/接口 ##
| 类    | 接口  | 说明   | 版本  |
| -----|:-----:|:-----:|:-----:|
| SelectableOverlay	| --- | 实现<MAOverlay>协议，自定义的可选中Overlay | --- |
| ---	| BOOL isOverlayWithLineWidthContainsPoint(id<MAOverlay> overlay, double mapPointDistance, MAMapPoint mapPoint); | 判断点point是否在overlay图形内 | --- |

## 核心难点 ##

### 使用教程

- ClickOverlay文件夹下的代码可以支持实现MAOverlay的点击，包括MAPolygon、MAPolyline、MACircle。
- Utility提供如下方法判断点point是否在overlay图形内。

```
BOOL isOverlayWithLineWidthContainsPoint(id<MAOverlay> overlay, double mapPointDistance, MAMapPoint mapPoint)
```
- 其中参数mapPointDistance提供了overlay的线宽（需换算到MAMapPoint坐标系）。对非封闭图形MAPolyline，若点距MAPolyline的距离小于距离门限，则认为点在图形内。距离门限设置为4倍mapPointDistance。
- 举个例子,判断点touchLocation是否在selectableOverlay内。

```
// 把屏幕坐标转换为MAMapPoint坐标.
MAMapPoint mapPoint = MAMapPointForCoordinate([self.mapView convertPoint:touchLocation toCoordinateFromView:self.mapView]);
// overlay的线宽换算到MAMapPoint坐标系的宽度.
double mapPointDistance = [self mapPointsPerPointInViewAtCurrentZoomLevel] * View.lineWidth;
              
//判断是否选中了overlay.
if (isOverlayWithLineWidthContainsPoint(selectableOverlay.overlay, mapPointDistance, mapPoint) )
{
    // ... 
}
```

###核心代码
`Objective-C`
```
- (void)mapView:(MAMapView *)mapView didSingleTappedAtCoordinate:(CLLocationCoordinate2D)coordinate
{
    /* 逆序遍历overlay判断单击点是否在overlay响应区域内. */
    [self.mapView.overlays enumerateObjectsWithOptions:NSEnumerationReverse usingBlock:^(id<MAOverlay> overlay, NSUInteger idx, BOOL *stop)
    {
        if ([overlay isKindOfClass:[SelectableOverlay class]])
        {
            SelectableOverlay *selectableOverlay = overlay;

            /* 获取overlay对应的renderer. */
            MAOverlayPathRenderer * renderer = (MAOverlayPathRenderer *)[self.mapView rendererForOverlay:selectableOverlay];

            /* 把屏幕坐标转换为MAMapPoint坐标. */
            MAMapPoint mapPoint = MAMapPointForCoordinate(coordinate);
            /* overlay的线宽换算到MAMapPoint坐标系的宽度. */
            double mapPointDistance = [self mapPointsPerPointInViewAtCurrentZoomLevel] * renderer.lineWidth;

            /* 判断是否选中了overlay. */
            if (isOverlayWithLineWidthContainsPoint(selectableOverlay.overlay, mapPointDistance, mapPoint) )
            {
                /* 设置选中状态. */
                selectableOverlay.selected = !selectableOverlay.isSelected;

                /* 修改view选中颜色. */
                renderer.fillColor   = selectableOverlay.isSelected? selectableOverlay.selectedColor:selectableOverlay.regularColor;
                renderer.strokeColor = selectableOverlay.isSelected? selectableOverlay.selectedColor:selectableOverlay.regularColor;

                /* 修改overlay覆盖的顺序. */
                [self.mapView exchangeOverlayAtIndex:idx withOverlayAtIndex:self.mapView.overlays.count - 1];

                [renderer glRender];

                *stop = YES;
            }
        }
    }];
}

```

`Swift`
```
func mapView(_ mapView: MAMapView!, didSingleTappedAt coordinate: CLLocationCoordinate2D) {

    for (index, overlay) in mapView.overlays.enumerated().reversed() {
        if (overlay as AnyObject).isKind(of: SelectableOverlay.self) {
            let selectableOverlay: SelectableOverlay = overlay as! SelectableOverlay;

            let renderer: MAOverlayPathRenderer = self.mapView.renderer(for: selectableOverlay) as! MAOverlayPathRenderer
            let mapPoint = MAMapPointForCoordinate(coordinate)
            let distance = self.mapPointsPerPointInViewAtCurrentZoomLevel()

            // 判断是否选中了overlay
            if isOverlayWithLineWidthContainsPoint(selectableOverlay.overlay, distance, mapPoint) {
                // selected
                selectableOverlay.isSelected = !selectableOverlay.isSelected

                // change color
                renderer.strokeColor = selectableOverlay.isSelected ? selectableOverlay.selectedColor : selectableOverlay.regularColor
                renderer.fillColor = selectableOverlay.isSelected ? selectableOverlay.selectedColor : selectableOverlay.regularColor

                // 修改顺序
                self.mapView.exchangeOverlay(at: UInt(index), withOverlayAt: UInt(self.mapView.overlays.count - 1))
            }
        }
    }
}

```

详见工程Demo文件夹。

### 架构

##### Controllers
- `<UIViewController>`
  * `ClickOverlayViewController` 点击选中overlay


##### Models

* `Conform to <MAOverlay>`
  - `SelectableOverlay` 自定义可选中的overlay(记录overlay选中状态,颜色属性)

##### Utility

* `Utility` 数学运算(计算点是否包含在overlay响应区域内)

### 截图效果

![selectCircle](https://raw.githubusercontent.com/amap-demo/iOS-selectable-overlay/master/iOS_Selectable_Overlay/Resources/selectCircle.png)
![unselectCircle](https://raw.githubusercontent.com/amap-demo/iOS-selectable-overlay/master/iOS_Selectable_Overlay/Resources/unselectCircle.png)

