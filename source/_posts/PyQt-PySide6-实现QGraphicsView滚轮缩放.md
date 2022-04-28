---
title: PyQt(PySide6)实现QGraphicsView滚轮缩放与拖动
date: 2022-04-28 10:21:26
categories:
- 客户端
tags:
- python
- pyqt
---
使用PyQt基于`QGraphicsView`实现可用滚轮缩放、鼠标按住拖动的图片查看器，同时隐藏滚动条。网上找到的都是C++的代码，故翻译成Python代码，已经测试，记录备查。

<!-- more -->

```python
class ImageView(QGraphicsView):
    def __init__(self):
        super(ImageView, self).__init__()
        self.setHorizontalScrollBarPolicy(Qt.ScrollBarAlwaysOff) #隐藏横向滚动条
        self.setVerticalScrollBarPolicy(Qt.ScrollBarAlwaysOff)   #隐藏纵向滚动条
        self.setDragMode(QGraphicsView.ScrollHandDrag)           #启用拖动

    def wheelEvent(self, event: QWheelEvent) -> None:
        if len(self.scene().items()) == 0:
            return

        curPoint = event.position()
        scenePos = self.mapToScene(QPoint(curPoint.x(), curPoint.y()))
    
        viewWidth = self.viewport().width()
        viewHeight = self.viewport().height()
    
        hScale = curPoint.x() / viewWidth
        vScale = curPoint.y() / viewHeight
    
        wheelDeltaValue = event.angleDelta().y()
        scaleFactor = self.transform().m11()
        if (scaleFactor < 0.05 and wheelDeltaValue<0) or (scaleFactor>50 and wheelDeltaValue>0):
            return

        if wheelDeltaValue > 0:
            self.scale(1.2, 1.2)
        else:
            self.scale(1.0/1.2, 1.0/1.2)
        
        viewPoint = self.transform().map(scenePos)
        self.horizontalScrollBar().setValue(int(viewPoint.x() - viewWidth * hScale ))
        self.verticalScrollBar().setValue(int(viewPoint.y() - viewHeight * vScale ))
    
        self.update()
        # 此处不需要调用 return super().wheelEvent(event)
```