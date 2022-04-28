---
title: 基于bitblt函数实现高帧率不闪的MFC示波器图表绘制（附使用例）
date: 2022-04-18 10:34:23
categories:
- 客户端
tags:
- C++
- MFC
---
使用bitblt函数，用剪贴图像代替全图重绘，在MFC程序中实现不闪的示波器图表绘制。

<!-- more -->

## 分析
自控实验要求用MFC做上位机画图，但是老师给的代码帧率一高就闪。经研究发现是因为它每一次画图就把整个图像涂黑，再重新绘制所有网格和曲线，而人眼能捕捉到涂黑的一瞬间，造成闪动。

我的思路是，在每一次画图的时候，将旧图像整体向左平移`DELTAX`像素，用背景色填充右边`DELTAX`宽的区域，最后连接上一次画图的最后一点和这一次要绘制的点。由于不用重新绘制整张图片，不仅不会闪，而且绘制速度更快，还将需要记住的点从整张图变成了最后一点从而大大节省了储存空间。

## 代码
Graph.h
```C++
#pragma once
#include "framework.h"
#include <vector>

class GraphControl
{
private:
    //绘图句柄
    CDC* pDC;
    //绘图区域矩形
    CRect rectPicture;
    //存放各个曲线画笔
    std::vector<CPen*> pens;
    //背景画笔
    CPen bgPen;
    //网格画笔
    CPen gridPen;
    //旧画笔暂存指针
    CPen* oldPen;
    //背景画刷
    CBrush bgBrush;
    //旧画刷暂存指针
    CBrush* oldBrush;
    //图能容纳的数据值数量
    int SIZE;
    //每次左移距离（像素）
    double DELTAX;
    //每个曲线的最小值、最大值、最后一个数据
    std::vector<double> valMin;
    std::vector<double> valMax;
    std::vector<double> last;
    //将数据值转化成画图坐标
    int doubleToCoord(double val, double min, double max);
public:
    GraphControl();
    ~GraphControl();
    void init(CStatic* graph);
    //添加曲线，设置画笔颜色、最小值、最大值
    void addItem(COLORREF color, double min, double max);
    //清空全图
    void clear();
    //画下一个数据，使用了变长参数
    void updateGraph(int argLength, ...);
    //获取图能容纳的数据值数量
    int getSize();
};
```
Graph.cpp
```C++
#include "pch.h"
#include "Graph.h"
#include <stdarg.h>
#pragma warning(disable:4244)

GraphControl::GraphControl():
    SIZE(0),
    DELTAX(0),
    pDC(nullptr),
    oldPen(nullptr),
    oldBrush(nullptr)
{
    bgPen.CreatePen(PS_SOLID, 4, RGB(30, 30, 30));
    bgBrush.CreateSolidBrush(RGB(30, 30, 30));
    gridPen.CreatePen(PS_SOLID, 1, RGB(100, 100, 100));
}

void GraphControl::init(CStatic* graph)
{
    pDC = graph->GetDC();
    graph->GetClientRect(&rectPicture);
    DELTAX = 2;
    SIZE = rectPicture.Width() / DELTAX;
}

void GraphControl::addItem(COLORREF color, double valMin, double valMax)
{
    CPen* pen = new CPen(PS_SOLID, 4, color);
    pens.push_back(pen);
    this->valMin.push_back(valMin);
    this->valMax.push_back(valMax);
    this->last.push_back(-1);
}

void GraphControl::clear()
{
    oldPen = pDC->SelectObject(&bgPen);
    oldBrush = pDC->SelectObject(&bgBrush);
    pDC->Rectangle(rectPicture);
    for (size_t i = 0; i < last.size(); i++)last[i] = 0;
    pDC->SelectObject(&gridPen);
    for (size_t i = 10; i < rectPicture.Height(); i += 20)
    {
        pDC->MoveTo(0, i);
        pDC->LineTo(rectPicture.Width(), i);
    }
    pDC->SelectObject(oldPen);
    pDC->SelectObject(oldBrush);
}

GraphControl::~GraphControl()
{
    for (size_t i = 0; i < pens.size(); i++)
    {
        pens[i]->DeleteObject();
        delete pens[i];
    }
    bgPen.DeleteObject();
    gridPen.DeleteObject();
    bgBrush.DeleteObject();
}

int GraphControl::doubleToCoord(double val, double min, double max)
{
    return rectPicture.Height() - ((double)rectPicture.Height() * (val - min) / (max - min));
}

void GraphControl::updateGraph(int argLength, ...)
{
    //初始化参数列表
    va_list args = NULL;
    va_start(args, argLength);

    //剪切旧图像
    int w1 = rectPicture.right - rectPicture.left - DELTAX;
    int w2 = rectPicture.Width() - DELTAX;
    pDC->BitBlt(0, 0, rectPicture.Width(), rectPicture.Height(), pDC, DELTAX, 0, SRCCOPY);

    //填充背景
    oldPen = pDC->SelectObject(&bgPen);
    oldBrush = pDC->SelectObject(&bgBrush);
    pDC->Rectangle(rectPicture.right - DELTAX + 1, 0, rectPicture.right, rectPicture.bottom + 1);
    pDC->SelectObject(&gridPen);
    for (size_t i = 10; i < rectPicture.Height(); i += 20)
    {
        pDC->MoveTo(rectPicture.right - DELTAX-1, i);
        pDC->LineTo(rectPicture.Width(), i);
    }
    pDC->SelectObject(oldPen);
    pDC->SelectObject(oldBrush);

    //绘制曲线
    for (size_t i = 0; i < pens.size(); i++)
    {
        oldPen = pDC->SelectObject(pens[i]);
        double val = va_arg(args, double);
        pDC->MoveTo(rectPicture.right, doubleToCoord(val, valMin[i], valMax[i]));
        if (last[i] != -1)
        {
            int ypos = doubleToCoord(last[i], valMin[i], valMax[i]);
            pDC->LineTo(rectPicture.right - DELTAX, ypos);
        }
        last[i] = val;
        pDC->SelectObject(oldPen);
    }
}

int GraphControl::getSize()
{
    return this->SIZE;
}
```

## 使用示例
在对话框头文件中，引用此头文件，并声明一个控制器对象。
```c++
#pragma once

//其他引用文件……
#include "Graph.h"
//其他引用文件……

// CMainDlg 对话框
class CMainDlg : public CDialogEx
{
//无关代码……
public:
    GraphControl graphControl; //控制器对象
    CStatic graph;             //被控制的Picture Control
    //无关代码……
};

```

在对话框的`OnInitDialog()`函数中，初始化控制器，设置每条曲线的颜色和最小最大值。
```c++
BOOL CMainDlg::OnInitDialog()
{
    //无关代码……

    //初始化画图
    graphControl.init(&graph);
    graphControl.addItem(RGB(78, 201, 176), -10, 260);
    graphControl.addItem(RGB(220, 220, 170), -10, 260);

    //无关代码……
}
```

画图操作的延迟会造成控制器不稳定，所以必须使用单独的线程画图。注意线程主函数必须声明为全局函数而不是某个类的成员函数。
```c++
bool uiEnable = false;
void uiThread()
{
    dlg->graphControl.clear();
    while (uiEnable)
    {
        dlg->graphControl.updateGraph(2, rod.expectVal, rod.readVal);
    }
}
void uiStart()
{
    uiEnable = true;
    std::thread th(uiThread);
    th.detach();
}
void uiEnd()
{
    uiEnable = false;
}
```