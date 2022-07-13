# 设计模式学习笔记

## 课程目标：
- 理解松耦合设计思想
- 掌握面向对象设计原则
- 掌握重构技法改善设计
- 掌握GOF核心设计模式

## 什么是设计模式：
“每一个模式描述了一个在我们周围不断重复发生的问题以及该问题的解决方案的核心。这样，你就能一次又一次地使用该方案而不必做重复劳动” 
                          - Christopher Alexand

## Gang of Four 设计模式
- 《设计模式：**可复用面向对象**软件的基础》
    - **可复用**是设计模式的目标
    - **面向对象**是设计模式的手法
    - 这本书定义了23种面向对象的设计模式

## 从面向对象谈起
### 面向对象包含两个思维模型：
- 程序员与计算机之间的沟通称之为**底层思维**：向下，如何把握机器底层从微观理解对象构造，帮助程序员建立机器模型
    - 语言构造
    - 编译转换
    - 内存模式
    - 运行时机制
- 抽象思维：向上，如何将我们的周围世界抽象为程序代码
    - 面向对象
    - 组件封装
    - 设计模式
    - 架构模式

## 深入理解面向对象
- 向下：深入理解三大面向对象机制
    - 封装，隐藏内部实现
    - 继承，复用现有代码
    - 多态，改写对象行为
- 向上：深刻把握面向对象机制所带来的抽象意义，理解如何使用这些机制来表达现实世界，掌握什么是“好的面向对象设计”。评判这个设计的标准要靠抽象思维。

## 软件设计固有复杂性
建筑商从来不会去想给一栋已经建好的100层高的楼房底下再新修一个小地下室--这样做花费极大而且注定要失败。而且令人惊奇的是，软件系统的用户在要求作出类似改变时却不会仔细考虑，而且他们认为这只是需要简单编程的事。 -- Grady Booch

## 软件设计复杂的根本原因
- 客户需求的变化
- 技术平台的变化
- 开发团队的变化
- 市场环境的变化

## 如何解决复杂性
人类解决问题一般有两种思维模型
1. **分解**：
    - 人们面对复杂性有一个常见的做法：分而治之，大问题分解为多个小问题，复杂的分解为多个简单的。
2. **抽象**：
    - 解决更高层次的来讲，人们处理复杂性有一种通用的技术叫抽象。由于不能掌握全部的复杂对象，人们选择忽视它的非本质细节，而去处理泛化和理想化后的对象模型。
    
    ```cpp
    Shape.h
    class Shape{
    public:
        // 纯虚函数/抽象函数
        virtual void Draw(const Graphic& g) = 0;
        // 虚的析构函数。子类以后通过多态释放的时候，子类的析构函数才会被正确的调用到
        virtual ~Shape(){}
    }
    class Point{ //辅助类型
    public:
        int x;
        int y;
    }
    class Line: public Shape{ //Line继承Shape。C加加中绝大多数的继承都用public
    public:
        Point start;
        Point end;
        //构造器
        Line(const Point& start, const Point& end){
            this->start = start;
            this->end = end;
        }
        //Overwrite父类的虚函数，实现自己的Draw，负责画自己。
        virtual void Draw(const Graphics& g){
            g.DrawLine(Pens.Red, start.x, start.y, end.x, end.y);
        }
    }
    class Rect: public Shape{
    public:
        Point leftUp;
        int width;
        int height;
        //构造器
        Rect(const Point& leftUp, int width, int height){
            this->leftUp = leftUp;
            this->width = width;
            this->height = height;
        }
        //实现自己的Draw，来画自己
        virtual void Draw(const Graphics& g){
            g.DrawRectangle(Pens.Red, leftUp, width, height);
            }
    }
    ```
    
    ```cpp
    MainForm.cpp
    class MainForm: public Form{
    private:
        Point p1;
        Point p2;
        //针对所有形态，可以插入Line，Rect类型。为了支持多态性所以需要用指针
        vector<Shape*> shapeVector;
    public:
        MainForm(){
            //...
        }
    protected:
        virtual void OnMouseDown(const MouseEventArgs& e);
        virtual void OnMouseUp(const MouseEventArgs& e);
        virtual void OnPaint(const MouseEventArgs& e);
    }
    void MainForm::OnMouseDown(const MouseEventArgs& e){
        p1.x = e.X;
        p1.y = e.Y;
        if(rdoLine.Checked){
            shapeVector.push_back(new Line(p1,p2));//因为shapeVector是指针，所以要new一个堆对象指针。之后一定要Delet
        }
        else if (rdoRect.Checked){
            int width = abs(p2.x - p1.x);
            int height = abs(p2.y - p1.y);
            shapeVector.push_back(new Rect(p1, width, height));
        }

        //...
        this->Refresh();
        Form::OnMouseUp(e);
    }
    void MainForm::OnPain(const PaintEventArgs& e){
        //针对所有形状
        for(int i=0; i<shapeVector.size(); i++){
            //直接对shape的所有形状调用Draw。Draw是虚函数，所以是多态调用，各负其责。例：如果存储的Line的话，找Line的Draw调用DrawLine
            shapeVector[i]->Draw(e.Graphics);//
        }
        Form::OnPaint(e);
    }
    ```

    ## 软件设计的目标
    什么是好的软件设计？软件设计的金科玉律：**复用**。所有设计模式的目标就是复用性。