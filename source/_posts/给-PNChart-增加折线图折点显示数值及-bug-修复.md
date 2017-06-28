---
title: 给 PNChart 增加折线图折点显示数值及 bug 修复
tags:
  - Blog
  - iOS
permalink: pnchart-ylabelinline
id: 9
updated: '2016-08-26 12:14:05'
date: 2016-05-29 00:32:04
---

### 效果

先来看看原来的效果图

![](http://7xlykq.com1.z0.glb.clouddn.com/github/pnchart_line.png)

然后我现在要给他增加功能，变成这样

![](http://7xlykq.com1.z0.glb.clouddn.com/QQ20160528-1@2x.png)

### 代码改动

改动文件：`PNLineChart.swift`

- 增加属性 `public var yLabelInLine: Bool = false`，
默认为 `false` 以第一个图的方式显示。
- 修改方法

``` Objective-C
    // 修改这里的实现代码
    public var yLabels: NSArray = []{
        didSet{
        	// 省略...
        	
            for value in yLabels
            {
                let label: PNChartLabel!
                if yLabelInLine {
                    
                    let data = yLabelPositionBy(yValue: value)
                    label = PNChartLabel(frame: CGRect(x: data.0, y: data.1, width: CGFloat(chartMargin + 5.0), height: CGFloat(yLabelHeight) ) )
                    label.text = NSString(format:yLabelFormat, data.2) as String
                }else {
                    
                    let labelY = chartCavanHeight - (index * yStepHeight)
                    label = PNChartLabel(frame: CGRect(x: 0.0, y: CGFloat(labelY), width: CGFloat(chartMargin + 5.0), height: CGFloat(yLabelHeight) ) )
                    label.text = NSString(format:yLabelFormat, Double(yValueMin + (yStep * index))) as String
                }
                label.textAlignment = NSTextAlignment.Right
                ++index
                yPNLabels.append(label)
                addSubview(label)
            }
        }
    }
```

- 添加方法 `yLabelPositionBy(yValue:)` 这个方法作用是计算出当前 `yLabel` 的位置和返回准确的数值(默认为保留两位小数的 float 类型)

```c
    private var tempIndex:[Int] = []
    private func yLabelPositionBy(yValue yValue: AnyObject) -> (CGFloat, CGFloat, CGFloat) {
        for d in 0 ..< chartData.count {
            let obj = chartData[d] as! PNLineChartData
            for i in 0 ..< obj.itemCount {
                if !tempIndex.contains(d * 10 + i) && NSString(format: "%2f", obj.getData(i).y).isEqual(yValue) {
                    
                    let yValue = CGFloat(obj.getData(i).y)
                    let innerGrade = (yValue - yValueMin) / (yValueMax - yValueMin)
                    
                    let x: CGFloat = 2.0 * chartMargin +  (CGFloat(i) * xLabelWidth) - (xLabelWidth / 2.0)
                    let y: CGFloat = chartCavanHeight! - (innerGrade * chartCavanHeight!) + (yLabelHeight / 2.0) + 5
                    
                    tempIndex.append(d * 10 + i)
                    return (x, y, yValue)
                }
            }
        }
        return (0.0, 0.0, 0.0)
    }
```

### bug 修复

在使用过程中遇到一个 bug，要是在同一个 `PNLineChart` 中多次绘制图表时，`xLabel` 和 `yLabel` 会重复创建，造成这种效果

![](http://7xlykq.com1.z0.glb.clouddn.com/image/jpg/QQ20160529-0@2x.png)

所以我添加两个属性 

``` c
private var xPNLabels: [UILabel] = [] // 存放 xLabels
private var yPNLabels: [UILabel] = [] // 存放 yLabels
```

在每次创建 label 之前先把原先的 label 移除

``` c
// xLabel didSet 中添加
for label in xPNLabels { label.removeFromSuperview() }

// yLabel didSet 中添加
for label in yPNLabels { label.removeFromSuperview() }

```

### GitHub

[PNChart](https://github.com/laichanwai/PNChart-Swift)