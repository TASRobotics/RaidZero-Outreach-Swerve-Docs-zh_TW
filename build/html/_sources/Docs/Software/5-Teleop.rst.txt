########
手動操控
########

本頁將解釋持續 2 分 15 秒的手動機器人控制（Teleop）。本文檔將參考與提到 Manual 
Drive Class 和 RobotContainer。可以在 `here <https://docs.wpilib.org/en/
stable/docs/software/commandbased/structuring-command-based-project.html#robotcontainer>`_ 
找到更多有關於 RobotContainer 的信息。

操縱
====

機器有兩種主要的移動類型：Field Oriented Driving 和 Robot Oriented Driving。

Field Oriented Driving的移動放式可以使機器在面向場地任何一邊時平移向各個方向。
例如，當機器人面朝東方時將操縱桿向前推機器人會向北移動。

與Field Oriented Driving相比，Robot Oriented Driving 的操縱桿的移動將與機器
相關，而不是與場地相關。例如，當機器人面朝東方時將操縱桿向前堆，機器人就會向東移動。

.. figure:: ../Photos/Software/Field-robot-oriented.png
    :scale: 50%
    :alt: Field/robot-oriented driving comparison


.. list-table:: **Field vs robot-oriented driving**
   :widths: 25 50 50
   :header-rows: 1

   * - 
     - Field-oriented
     - Robot-oriented
   * - 需要 IMU
     - 要（會有誤差積累*）
     - 不
   * - 操縱桿的移動
     - 相對於場地
     - 相對於機器


\* IMU 誤差積累又可能會造成 Field Orientation Driving 有些微的不準確


以下是 field-oriented control 的示範：

.. raw:: html

    <iframe width="560" height="315" src="https://www.youtube.com/embed/50ZRrYFWPIc?start=21" title="Field oriented control" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


驅動程式
========

因為我們使用 `command based programming <https://docs.wpilib.org/en
/latest/docs/software/commandbased/index.html>`_，所以需要驅動程式來控制機器。

Constructor
***********

.. code-block:: java
    :linenos:

    private final Swerve mSwerve;
    private final XboxController mController;
    
    public ManualDrive(Swerve drive, XboxController controller) {
        mSwerve = drive;
        mController = controller;

        // 加入 swerve 為這條命令的必要條件
        addRequirements(mSwerve);
    }

**Parameters:**
"""""""""""""""

1. ``drive`` - a swerve object for the drive
2. ``controller`` - a XboxController method for the controller

必須創建驅動命令才能手動操控機器人。調用構造函數將創建驅動命令，然後在 RobotContainer 中使用它。

execute
*******

當 drivecommand 被叫到時，execute method 會一直執行，所以操控程式要寫在這裡。

.. code-block:: java
    :linenos:

    @Override 
    public void execute() {
        // Drives with XSpeed, YSpeed, zSpeed
        // True/false for field-oriented driving
        mSwerve.drive(mController.getLeftY(), mController.getLeftX(), mController.getRightX(), true);
    }

執行驅動程式
===========

Drive command 需要再 RobotContainer 裡被叫到。

.. code-block:: java
    :linenos:

    // 創造一個新的 ManualDrive instance, 給他 Swerve 跟 Controller 當 parameters
    private final ManualDrive mManualDriveCommand = new ManualDrive(mSwerve, mController);

    public RobotContainer() {
        // 配置按鈕綁定
        configureButtonBindings();

        // 設置 ManuelDrive 在手動操控中執行
        mSwerve.setDefaultCommand(mManualDriveCommand);
    }

創建了一個新的 ManualDrive instance，它也創建了 drive command。為了要在 
RobotContainer 中使用 drive command，Swerve object 的原始設置為手動驅動。
當 Teleop 啟動時，ManualDrive 命令將自動執行。