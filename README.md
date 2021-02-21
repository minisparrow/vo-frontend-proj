# vo-frontend-proj
<!-- TOC depthFrom:2 orderedList:true--> 
- [vo-frontend-proj](#vo-frontend-proj)
  - [dependency](#dependency)
  - [dataset](#dataset)
  - [build and run](#build-and-run)
  - [camera model](#camera-model)
  - [Frame 类](#frame-类)
  - [MapPoint 路标点](#mappoint-路标点)
  - [Map 管理所有路标点](#map-管理所有路标点)
  - [Config 类](#config-类)

## progress
- [x] v0.1
- [ ] v0.2

## dependency 

opencv3.4

sudo apt install libvtk6-dev 

## dataset 

https://vision.in.tum.de/data/datasets/rgbd-dataset/download

fr1/xyz

## build and run 

cd 0.1 or 
cd 0.2 
cd 0.3 
cd 0.4 

mkdir build && cd build && cmake ..

make -j8

../run.sh


## camera model 

> 世界坐标到相机坐标

```
  Vector3d Camera::world2camera ( const Vector3d& p_w, const SE3& T_c_w )
  {
      return T_c_w*p_w;
  }

  Vector3d Camera::camera2world ( const Vector3d& p_c, const SE3& T_c_w )
  {
      return T_c_w.inverse() *p_c;
  }
```

> 相机坐标到像素坐标

```
  Vector2d Camera::camera2pixel ( const Vector3d& p_c )
  {
      return Vector2d (
          fx_ * p_c ( 0,0 ) / p_c ( 2,0 ) + cx_,
          fy_ * p_c ( 1,0 ) / p_c ( 2,0 ) + cy_
      );
  }

  Vector3d Camera::pixel2camera ( const Vector2d& p_p, double depth )
  {
      return Vector3d (
          ( p_p ( 0,0 )-cx_ ) *depth/fx_,
          ( p_p ( 1,0 )-cy_ ) *depth/fy_,
          depth
      );
  }
```

> 世界坐标到像素坐标

```
  Vector2d Camera::world2pixel ( const Vector3d& p_w, const SE3& T_c_w )
  {
      return camera2pixel ( world2camera ( p_w, T_c_w ) );
  }

  Vector3d Camera::pixel2world ( const Vector2d& p_p, const SE3& T_c_w, double depth )
  {
      return camera2world ( pixel2camera ( p_p, depth ), T_c_w );
  }
```
## Frame 类

## MapPoint 路标点

我们将估计它的世界坐标，并且拿当前帧提取到的特征点与地图中的路标点匹配，
来估计相机的运动，因此还需要存储对应的描述子。

还需记录一个点被观察到的次数和被匹配的次数。作为评价其好坏程度的指标。

```c++
class MapPoint
{
public:
    typedef shared_ptr<MapPoint> Ptr;
    unsigned long      id_; // ID
    Vector3d    pos_;       // Position in world
    Vector3d    norm_;      // Normal of viewing direction 
    Mat         descriptor_; // Descriptor for matching 
    int         observed_times_;    // being observed by feature matching algo.
    int         correct_times_;     // being an inliner in pose estimation
    
    MapPoint();
    MapPoint( long id, Vector3d position, Vector3d norm );
    
    // factory function
    static MapPoint::Ptr createMapPoint();
};
```
## Map 管理所有路标点

- 添加新路标
- 删除不好的路标
- VO 匹配过程只要和Map打交道即可

```c++
class Map
{
public:
    typedef shared_ptr<Map> Ptr;
    unordered_map<unsigned long, MapPoint::Ptr >  map_points_;        // all landmarks
    unordered_map<unsigned long, Frame::Ptr >     keyframes_;         // all key-frames

    Map() {}
    
    void insertKeyFrame( Frame::Ptr frame );
    void insertMapPoint( MapPoint::Ptr map_point );
};
```

## Config 类
