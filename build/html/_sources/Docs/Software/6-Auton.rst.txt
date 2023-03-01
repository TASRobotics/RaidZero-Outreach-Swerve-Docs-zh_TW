########
自動路徑
########

有許多不同的方法來生成路徑。在本指南中，我們將使用 `PathPlanner <https://
github.com/mjansen4857/pathplanner>`_ 來生成和跟隨路徑。

下載
****

.. note:: 

    可以在 `here <https://github.com/mjansen4857/pathplanner/wiki>`_ 找到官方文檔。

首先，使用此鏈接安裝 PathPlannerLib ：

.. code-block:: bash

    https://3015rangerrobotics.github.io/pathplannerlib/PathplannerLib.json

接下來，從 `Microsoft <https://www.microsoft.com/en-us
/p/frc-pathplanner/9nqbkb5dw909?cid=storebadge&ocid=badge&rtc=1&activetab=pivo
t:overviewtab>`_ 或 `Mac App <https://apps.apple.com/us/app/frc-pathplanner/id1593046876>`_
下載 PathPlanner。

生成路徑
********

1. 打開 PathPlanner，點擊葉面中的「切換項目（Switch Project）」，然後選擇有你的項目所有項目源的文建夾。

.. image:: ../Photos/PathPlanner1.PNG
    :scale: 50%

2. 去葉面中的「設置（settings）」，並根據您的機器人配置設定所有內容。確定也選擇「完整模式（Holonomic Mode）」。

.. image:: ../Photos/PathPlanner2.PNG
    :scale: 50%

3. 最後，繪製您希望機器人跟隨的路徑。

路徑跟隨
********

我們現在將提取在 PathPlanner 中創建的路徑並將其用於我們的機器人代碼。

在 ``RobotContainer.java`` 中，通過提取您在 PathPlanner 中創建的路徑來創
建 ``PathPlannerTrajectory`` 的實例。

.. code-block:: java
    :linenos:

    private PathPlannerTrajectory mTrajectory = PathPlanner.loadPath(
        "PATH NAME", 
        SwerveConstants.kMaxVelocityMetersPerSecond, 
        SwerveConstants.kMaxAccelerationMetersPerSecond
    );

接著，創建 3 個不同的 PID 控制器，用於控制機器人的 X 和 Y 方向和面對方向。

.. code-block:: java
    :linenos:

    // X 方向的 PID 控制器
    private PIDController mXController = new PIDController(
        SwerveConstants.kPathingX_kP, 
        SwerveConstants.kPathingX_kI, 
        SwerveConstants.kPathingX_kD
    );

    // Y 方向的 PID 控制器
    private PIDController mYController = new PIDController(
        SwerveConstants.kPathingY_kP, 
        SwerveConstants.kPathingY_kI, 
        SwerveConstants.kPathingY_kD
    );

    // 面對方向的 PID 控制器
    private PIDController mThetaController = new PIDController(
        SwerveConstants.kPathingTheta_kP, 
        SwerveConstants.kPathingTheta_kI, 
        SwerveConstants.kPathingTheta_kD
    );

.. warning::

    PID 控制器中使用的所有常數都必須根據你的機器進行調整。 X 和 Y 控制器通常
    具有相同的常數，但 面對方向（theta）控制器通常具有不同的常數。可以通過使
    機器沿著一條長的直線路徑來調整 X 和 Y 控制器常數。調整 X 和 Y 控制器後，
    通過使機器人沿著一條短並具有急轉彎的路徑來調整面對方向（theta）控制器。

最後，創建一個將用於跟隨路徑的 ``SwerveControllerCommand`` 。

.. code-block:: java
    :linenos:

    private PPSwerveControllerCommand mCommand = new PPSwerveControllerCommand(
        mTrajectory, 
        mSwerve::getPose, 
        SwerveConstants.kSwerveKinematics, 
        mXController, 
        mYController, 
        mThetaController, 
        mSwerve::setModuleStates, 
        mSwerve
    );

在 ``getAutonomousCommand()`` 中返回命令（return）。

.. code-block:: java
    :linenos:

    @Override
    public Command getAutonomousCommand() {
        // `andThen...` 用於在路徑完成後停止機器人
        return command.andThen(() -> mSwerve.drive(0, 0, 0, false));
    }

這允許機器人自主跟隨您在 PathPlanner 中創建的路徑。

.. note::

    如果跟隨的路徑不准確或不穩定，請嘗試更改 PID 控制器常數。

.. note::
    在我們的 `Github <https://github.com/TASRobotics/RaidZero-Swerve-Template>`_
    上查看我們在這些文檔中使用的代碼！