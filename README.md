This file describes the work done and steps to perform test on the laser_assembler package with crystal release.

"https://github.com/vandanamandlik/laser_assembler/tree/ros2-crystal"


ROS2 Migration changes

	The basic concept and design are same as ROS1.
	Work Done by referring ROS1 hydro-devel branch of laser_assembler package.
	All changes for migration have been done as per Migration guide.
		- Migrated all header and source files into ROS2 style
		- Migrated yaml files into ROS2 style
		- Migrated .launch file into launch.py in ROS2 style. (referred link https://index.ros.org/doc/ros2/Tutorials/Launch-system/)
		- Migrated CMakeLists.txt and package.xml in ROS2 style
		- Added point_cloud_conversion.h and point_field_conversion.h files.(commit id 654d367560f4710b9f368deb5a6709024c4b50d4 )
			- These files were not there in sensor_msgs package of ros2 so just copied it from ROS1 and ported to ROS2 here.
			- laser assembler uses convertPointCloudToPointCloud2() function from point_cloud_conversion.h file.
			- point_cloud_conversion.h uses point_field_conversion.h file
			- Added issue here
				https://answers.ros.org/question/309685/ros2-could-not-find-alternative-option-for-point_cloud_conversionh-in-sensor_msgs-package/
		- Added custom service code which generates structure for request and response part of the communication (commit id d1ae9deb83f1afbcc06e31786b0b66c0e73f23f0)
			- replaced time datatype with int64 in AssembleScans.srv file.
			- As per ROS2 migration guide types duration and time which were builtin types in ROS 1 have been replaced with normal message definitions and must be used from the builtin_interfaces package.

Dependencies

	filters - https://github.com/swatifulzele/filters/tree/ros2_devel
	laser_geometry
	message_filters
	launch

Build packages

	mkdir test_laser_assembler

	Build filters package
		Go to test_laser_assembler directory
		mkdir filters_ws
		mkdir filters_ws/src
		cd filters_ws/src
		git clone https://github.com/swatifulzele/filters.git -b ros2_devel
		Go to filters_ws directory.
		source /opt/ros/crystal/setup.bash
		colcon build

	Build laser_geometry package (laser_geometry package was not having PointCloud1 support so I have added it here.)
		Go to test_laser_assembler directory
		mkdir laser_geometry_ws
		mkdir laser_geometry_ws/src
		cd laser_geometry_ws/src
		git clone https://github.com/vandanamandlik/laser_geometry.git -b ros2-devel
		Go to laser_geometry_ws directory.
		colcon build

	Build laser_assembler
		Go to test_laser_assembler directory
		mkdir laser_assembler_ws
		mkdir laser_assembler_ws/src
		git clone https://github.com/vandanamandlik/laser_assembler.git -b ros2-crystal
		remove laser_assembler_srv_gen folder from laser_assembler and paste it in src folder (so your src folder should contain laser_assembler and laser_assembler_srv_gen directories.)
		Go to laser_assembler_ws
		source <filters's setup.bash file path> (eg. source filters_ws/install/setup.sh)
		source <laser_geometry's setup.bash file path>
		colcon build

Do the test

	Here launch files are used independently.
	Following are the steps to run the test cases independently:

	1.  Set the path
		Go to laser_assembler_ws
		source /opt/ros/crystal/setup.bash
		source ./install/setup.bash
 
	2. To run non_zero_point_cloud_test
		ros2 launch laser_assembler test_laser_assembler.launch.py

Limitations

	colcon test does not work as launch.py files can not be executed/added with CMakeLists.txt as of now.(https://index.ros.org/doc/ros2/Tutorials/Launch-system/)

Future work

	- test_laser_assembler.launch.py files should be executed using "colcon test" automatically.
	- service should part of the laser_assembler package, it means it should generate code from the laser_assembler package cmake and package.xml itself, right now there is separte cmake and package.xml to generate custom service code. I was facing some issues to generate it from laser_assembler cmake.(Build was successfull but was facing some issues while importing files generated by custom service.)
	- transformPointCloud() method is not available in TF2_ROS package so use it when it will be available in ROS2. Right now another slower method is used from laser_geometry package.

Known issue

	- In above build steps we are building laser_geometry package But you will see that crystal release also have same package.
	So though we source laser_geometry package path that we have build separately and then do colcon build, colcon build refers to installed package path in crystal.
	So in order to use the path that we have build in separate workspace, we have to rename /opt/ros/crystal/include/laser_geometry with some another name then only colcon build gets successfull.
	created issue here: https://answers.ros.org/question/312165/local-package-path-is-not-prioritized-over-installed-package-path/
	
	 