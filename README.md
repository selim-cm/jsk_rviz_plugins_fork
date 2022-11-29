# jsk_rviz_plugins_fork
Fork of https://github.com/jsk-ros-pkg/jsk_visualization/tree/master/jsk_rviz_plugins keeping only PolygonArray.

## Intructions

Instructions are for JRL configuration (Ubuntu 18.04 with ROS melodic). 

Make sure to first install the following dependancies:

- `jsk_recognition`:
```bash
sudo apt install ros-melodic-jsk-recognition
```

- `pointcloud library` version 1.8:
```bash
# Install pcl 1.8
# More information on https://pcl.readthedocs.io/projects/tutorials/en/master/compiling_pcl_posix.html#compiling-pcl-posix
wget https://github.com/PointCloudLibrary/pcl/archive/refs/tags/pcl-1.8.1.tar.gz  # get source code
tar xvf pcl-pcl-1.8.1.tar.gz  # extract archive
cd pcl-pcl-1.8.1 && mkdir build && cd build  # create build folder
ccmake ..  # here configure install folder in /usr instead of /usr/local
sudo make -j`nproc` install  # compile and install 
```

Then install as catkin package:
```bash
cd <catkin_ws>/src
git clone https://github.com/selim-cm/jsk_rviz_plugins_fork.git
catkin build jsk_rviz_plugins_fork
```

Restart RViz to see the PolygonArray display option appear.


## License

BSD
