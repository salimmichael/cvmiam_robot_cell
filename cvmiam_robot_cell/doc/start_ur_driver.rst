:github_url: https://github.com/UniversalRobots/Universal_Robots_ROS2_Tutorials/blob/main/cvmiam_robot_cell/doc/start_ur_driver.rst

=========================
Start the ur_robot_driver
=========================

In the last chapter we created a custom scene description containing our robot. In order to make
that description usable to ``ros2_control`` and hence the ``ur_robot_driver``, we need to add
control information. We will also add a custom launchfile to startup our demonstrator.

We will generate a new package called `cvmiam_robot_cell_control <https://github.com/UniversalRobots/Universal_Robots_ROS2_Tutorials/tree/main/cvmiam_robot_cell/cvmiam_robot_cell_control>`_ for that purpose.


Create a description with ros2_control tag
------------------------------------------

The first step is to create a description containing the control instructions:

.. literalinclude:: ../cvmiam_robot_cell_control/urdf/cvmiam_robot_cell_controlled.urdf.xacro
    :language: xml
    :linenos:
    :caption: cvmiam_robot_cell_control/urdf/cvmiam_robot_cell_controlled.urdf.xacro

This URDF is very similar to the one we have already assembled. We simply need to include the ros2_control macro,

.. literalinclude:: ../cvmiam_robot_cell_control/urdf/cvmiam_robot_cell_controlled.urdf.xacro
    :language: xml
    :start-at:   <xacro:include filename="$(find ur_robot_driver)/urdf/ur.ros2_control.xacro"/>
    :end-at:   <xacro:include filename="$(find ur_robot_driver)/urdf/ur.ros2_control.xacro"/>
    :caption: cvmiam_robot_cell_control/urdf/cvmiam_robot_cell_controlled.urdf.xacro


define the necessary arguments that need to be passed to the macro,

.. literalinclude:: ../cvmiam_robot_cell_control/urdf/cvmiam_robot_cell_controlled.urdf.xacro
    :language: xml
    :start-at: <xacro:arg name="robot_ip" default="0.0.0.0"/>
    :end-at: <xacro:arg name="mock_sensor_commands" default="false" />
    :caption: cvmiam_robot_cell_control/urdf/cvmiam_robot_cell_controlled.urdf.xacro


and then call the macro by providing all the specified arguments.

.. literalinclude:: ../cvmiam_robot_cell_control/urdf/cvmiam_robot_cell_controlled.urdf.xacro
    :language: xml
    :start-at:    <xacro:ur_ros2_control
    :end-at:   />
    :caption: cvmiam_robot_cell_control/urdf/cvmiam_robot_cell_controlled.urdf.xacro

Extract the calibration
-----------------------

One very important step is to extract the robot's specific calibration and save it to our
workcell's startup package. For details, please refer to `our documentation on extracting the calibration information <https://docs.ros.org/en/ros2_packages/rolling/api/ur_robot_driver/installation/robot_setup.html#extract-calibration-information>`_.
For now, we just copy the default one for the ur20.

.. code-block::

   cp $(ros2 pkg prefix ur_description)/share/ur_description/config/ur20/default_kinematics.yaml \
     cvmiam_robot_cell_control/config/cvmiam_robot_calibration.yaml


Create robot_state_publisher launchfile
---------------------------------------

To use the custom controlled description, we need to generate a launchfile loading that description
(Since it contains less / potentially different) arguments than the "default" one. In that
launchfile we need to start a ``robot_state_publisher`` (RSP) node that will get the description as a
parameter and redistribute it via the ``robot_description`` topic:

.. literalinclude:: ../cvmiam_robot_cell_control/launch/rsp.launch.py
    :language: py
    :start-after: # Author: Felix Exner
    :linenos:
    :caption: cvmiam_robot_cell_control/launch/rsp.launch.py

With this we could start our workcell using

.. code-block:: bash

    ros2 launch ur_robot_driver ur_control.launch.py \
      description_launchfile:=$(ros2 pkg prefix cvmiam_robot_cell_control)/share/cvmiam_robot_cell_control/launch/rsp.launch.py \
      use_mock_hardware:=true \
      robot_ip:=123 \
      ur_type:=ur20 \
      rviz_config_file:=$(ros2 pkg prefix cvmiam_robot_cell_description)/share/cvmiam_robot_cell_description/rviz/urdf.rviz \
      tf_prefix:=ur20_

Create start_robot launchfile
-----------------------------

Since the command above is obviously not very convenient to start our robot, we wrap that into another
launchfile that includes the ``ur_control.launch.py`` launchfile with the correct description
launchfile and prefix:

.. literalinclude:: ../cvmiam_robot_cell_control/launch/start_robot.launch.py
    :language: py
    :linenos:
    :caption: cvmiam_robot_cell_control/launch/start_robot.launch.py

With that we can start the robot using

.. code-block:: bash

   ros2 launch cvmiam_robot_cell_control start_robot.launch.py use_mock_hardware:=true

Testing everything
------------------

Before we can test our code, it's essential to build and source our Colcon workspace:

.. code-block:: bash

    #cd to your colcon workspace root, e.g.
    cd ~/colcon_ws

    #source and build your workspace
    colcon build
    source install/setup.bash

We can start the system in a mocked simulation

.. code-block:: bash

    #start the driver with mocked hardware
    ros2 launch cvmiam_robot_cell_control start_robot.launch.py use_mock_hardware:=true

Or to use it with a real robot:

.. code-block:: bash

    #start the driver with real hardware
    ros2 launch cvmiam_robot_cell_control start_robot.launch.py

.. note::
   We extracted the calibration information from the robot and saved it in the
   ``cvmiam_robot_cell_control`` package. If you have a different robot, you need to extract the
   calibration information from that, and pass that to the launch file. Changing the ``ur_type``
   parameter only will lead to a wrong model, as it will still be using the example kinematics from
   a UR20. To use a UR5e, for example, you can do

   .. code-block:: bash

      ros2 launch cvmiam_robot_cell_control start_robot.launch.py ur_type:=ur5e use_mock_hardware:=true \
        kinematics_parameters_file:=$(ros2 pkg prefix ur_description)/share/ur_description/config/ur5e/default_kinematics.yaml
