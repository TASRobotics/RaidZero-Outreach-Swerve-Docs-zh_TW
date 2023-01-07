############
Swerve Class
############

Swerve Class 結合了 4 個 swerve module 來控制整個 swerve。

.. note:: 

    有關 API 的特定問題，請參閱 `WPILib API <https://www.youtube.com/watch?v=dQw4w9WgXcQ>`_
    ， `CTRE API <https://api.ctr-electronics.com/phoenix/release/java/>`_，和 `REV Robotics API 
    <https://codedocs.revrobotics.com/java/com/revrobotics/package-summary.html>`_。

Constructor
***********

.. code-block:: java
    :linenos:

    // 初始化 IMU
    private final WPI_Pigeon2 mImu = new WPI_Pigeon2(SwerveConstants.kImuID);

    private final SwerveModule mLeftFrontModule, mRightFrontModule, mLeftRearModule, mRightRearModule;
    private SwerveDriveOdometry mOdometry;

    public Swerve() {
        // 實例化（instantiate）swerve module - 個代表機器上四個其一
        mLeftFrontModule = new SwerveModule(
            SwerveConstants.kLeftFrontThrottleID, 
            SwerveConstants.kLeftFrontRotorID, 
            SwerveConstants.kLeftFrontCANCoderID, 
            SwerveConstants.kLeftFrontRotorOffset
        );

        mRightFrontModule = new SwerveModule(
            SwerveConstants.kRightFrontThrottleID, 
            SwerveConstants.kRightFrontRotorID, 
            SwerveConstants.kRightFrontCANCoderID, 
            SwerveConstants.kRightFrontRotorOffset
        );

        mLeftRearModule = new SwerveModule(
            SwerveConstants.kLeftRearThrottleID, 
            SwerveConstants.kLeftRearRotorID, 
            SwerveConstants.kLeftRearCANCoderID, 
            SwerveConstants.kLeftRearRotorOffset
        );

        mRightRearModule = new SwerveModule(
            SwerveConstants.kRightRearThrottleID, 
            SwerveConstants.kRightRearRotorID, 
            SwerveConstants.kRightRearCANCoderID, 
            SwerveConstants.kRightRearRotorOffset
        );

        // 實例化（instantiate）測程法（odometry） - 用於追踪位置
        mOdometry = new SwerveDriveOdometry(SwerveConstants.kSwerveKinematics, mImu.getRotation2d());
    }

Methods
*******

使用當前 Swerve module 狀態更新測程法（odometry）。

periodic
========

.. code-block:: java
    :linenos:

    @Override
    public void periodic() {
        // 使用當前Swerve module狀態更新測程法（odometry）。
        mOdometry.update(
            mImu.getRotation2d(), 
            getModuleStates()[0], 
            getModuleStates()[1],
            getModuleStates()[2],
            getModuleStates()[3]
        );
    }

drive
=====

移動底盤到想要的方向 / 地方與轉向。

.. code-block:: java
    :linenos:

    public void drive(double xSpeed, double ySpeed, double zSpeed, boolean fieldOriented) {
        SwerveModuleState[] states = null;
        if(fieldOriented) {
            states = SwerveConstants.kSwerveKinematics.toSwerveModuleStates(
                // IMU 用於 Field Oriented Control
                ChassisSpeeds.fromFieldRelativeSpeeds(xSpeed, ySpeed, zSpeed, mImu.getRotation2d())
            );
        } else {
            states = SwerveConstants.kSwerveKinematics.toSwerveModuleStates(
                new ChassisSpeeds(xSpeed, ySpeed, zSpeed)
            );
        }
        setModuleStates(states);
    }

**Parameters:**
"""""""""""""""

    1. ``xSpeed`` - X 方向的功率百分比
    2. ``ySpeed`` - Y 方向的功率百分比
    3. ``zSpeed`` - 旋轉的功率百分比
    4. ``fieldOriented`` - 設置機器運動方式（Field or Robot Oriented）


getModuleStates
===============

輸出 4 個 Swerve Module 的當前狀態 modules。

.. code-block:: java
    :linenos:

    public SwerveModuleState[] getModuleStates() {
        return new SwerveModuleState[]{
            mLeftFrontModule.getState(), 
            mRightFrontModule.getState(), 
            mLeftRearModule.getState(), 
            mRightRearModule.getState()
        };
    }


**Return:**
"""""""""""

    返回 SwerveModuleState 數組中所有Swerve的狀態（順序：[左前、右前、左後、右後]）。

setModuleStates
===============

設置 4 個 Swerve module 的狀態。

.. code-block:: java
    :linenos:

    // Swerve module 順序：[左前、右前、左後、右後]
    public void setModuleStates(SwerveModuleState[] desiredStates) {
        SwerveDriveKinematics.desaturateWheelSpeeds(desiredStates, 1);
        mLeftFrontModule.setState(desiredStates[0]);
        mRightFrontModule.setState(desiredStates[1]);
        mLeftRearModule.setState(desiredStates[2]);
        mRightRearModule.setState(desiredStates[3]);
    }

**Parameters:**
"""""""""""""""

    1. ``desiredStates`` - 理想 `SwerveModuleState <https://first.wpi.edu/wpilib/allwpilib/docs/release/java
       /edu/wpi/first/math/kinematics/SwerveModuleState.html>`_ 數組。

getPose
=======

獲取機器人的當前位置。

.. code-block:: java
    :linenos:

    public Pose2d getPose() {
        return mOdometry.getPoseMeters();
    }

**Return:**
"""""""""""

    新的 `Pose2d <https://first.wpi.edu/wpilib/allwpilib/docs/release/java
    /edu/wpi/first/math/geometry/Pose2d.html>`_ 表示機器人在場地上的位置（以米為單位）。


setPose
=======

將測程法（odometry）位置設置為給與的 x、y、位置和角度。

.. code-block:: java
    :linenos:

    public void setPose(Pose2d pose) {
        mOdometry.resetPosition(pose, mImu.getRotation2d());
    }

**Parameters:**
"""""""""""""""

    1. ``pose`` - 具有機器人位置和角度的 `Pose2d <https://first.wpi.edu/wpilib/allwpilib/docs/release/java/edu/wpi/first/math/geometry/Pose2d.html>`_ 
       object。

.. note::
    在我們的 `Github <https://github.com/TASRobotics/RaidZero-Swerve-Template>`_
    上查看我們在這些文檔中使用的代碼！