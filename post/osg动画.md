# 1.动画沿着直线走

```cpp
#include <osg/AnimationPath>
#include <osg/MatrixTransform>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

int main() {
    osg::AnimationPath* path = new osg::AnimationPath;
    osg::AnimationPath::TimeControlPointMap tcp;
    tcp[0.0] = osg::AnimationPath::ControlPoint(osg::Vec3d(-4, 0, 0), osg::Quat(1, 0, 0, 0));
    tcp[4.0] = osg::AnimationPath::ControlPoint(osg::Vec3d(4, 0, 0), osg::Quat(1, 0, 0, 0));
    path->setTimeControlPointMap(tcp);

    osg::ref_ptr<osg::MatrixTransform> mt = new osg::MatrixTransform;
    mt->setUpdateCallback(new osg::AnimationPathCallback(path));
    mt->addChild(osgDB::readNodeFile("glider.osg"));

    osgViewer::Viewer view;
    view.setSceneData(mt);
    view.setUpViewInWindow(200, 100, 800, 600);
    return view.run();
}
```

这个代码会让物体沿着固定的轨迹移动

其中

```cpp
tcp[0.0] = osg::AnimationPath::ControlPoint(osg::Vec3d(-4, 0, 0), osg::Quat(1, 0, 0, 0));
tcp[4.0] = osg::AnimationPath::ControlPoint(osg::Vec3d(4, 0, 0), osg::Quat(1, 0, 0, 0));
```

这两行代码是在创建动画路径的控制点。每个控制点都由一个时间、位置和旋转组成，这些控制点定义了动画路径。

第一行代码创建了一个控制点，该控制点在时间为0.0时，位置为(-4, 0, 0)，旋转为(1, 0, 0, 0)。这里的旋转是一个四元数，表示物体在3D空间中的旋转。四元数(1, 0, 0, 0)表示没有旋转。

第二行代码创建了另一个控制点，该控制点在时间为4.0时，位置为(4, 0, 0)，旋转为(1, 0, 0, 0)。这意味着在动画的第4秒，物体将移动到位置(4, 0, 0)，并保持原来的旋转。

这两个控制点一起定义了一个从(-4, 0, 0)到(4, 0, 0)的直线路径，物体将在4秒内沿着这条路径移动

# 2.动画沿着圆形走

上一个例子是沿着直线走，那如果想沿着圆的轨迹走呢

那就得在动画路径中添加更多的控制点来定义这个圆形轨迹，下面是创建了一个半径为4的圆形轨迹

```cpp
#include <osg/AnimationPath>
#include <osg/MatrixTransform>
#include <osgDB/ReadFile>
#include <osgViewer/Viewer>

int main() {
    osg::AnimationPath* path = new osg::AnimationPath;
    osg::AnimationPath::TimeControlPointMap tcp;

    // 定义圆的半径和圆心
    double radius = 4.0;
    osg::Vec3d center(0.0, 0.0, 0.0);

    // 定义圆上的点的数量
    int numPoints = 100;

    // 在圆上添加控制点
    for (int i = 0; i < numPoints; ++i) {
        double angle = ((double)i / (double)numPoints) * 2.0 * osg::PI;
        osg::Vec3d position(center.x() + radius * cos(angle), center.y() + radius * sin(angle), center.z());
        tcp[(double)i] = osg::AnimationPath::ControlPoint(position, osg::Quat(1, 0, 0, 0));
    }

    path->setTimeControlPointMap(tcp);

    osg::ref_ptr<osg::MatrixTransform> mt = new osg::MatrixTransform;
    mt->setUpdateCallback(new osg::AnimationPathCallback(path));
    mt->addChild(osgDB::readNodeFile("glider.osg"));

    osgViewer::Viewer view;
    view.setSceneData(mt);
    view.setUpViewInWindow(200, 100, 800, 600);
    return view.run();
}
```

如果觉得这个动画移动的太慢了怎么办呢

那就把里面的tcp[(double)i] 替换为下面的

```cpp
//tcp[(double)i] 
tcp[(double)i / 2.0] 
```

这样每个控制点之间的时间间隔就减半了，从而使动画的速度加快了一倍