# vo-frontend-proj

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

## MapPoint 路标点

我们将估计它的世界坐标，并且拿当前帧提取到的特征点与地图中的路标点匹配，
来估计相机的运动，因此还需要存储对应的描述子。

还需记录一个点被观察到的次数和被匹配的次数。作为评价其好坏程度的指标。


