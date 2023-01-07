##################
SwerveModule Class
##################

The SwerveModule class initializes the swerve module on the robot. The method 
``getState`` returns a new object of the ``SwerveModuleState`` class, allowing us to 
know the current state according to the linear speed of the throttle in meters 
per second and the degrees the encoder has rotated. The method ``setState`` gets 
the current heading of the drive and takes a ``SwerveModuleState`` object. The 
method will use a PID controller to turn the drive to the desired heading at a desired 
speed.

.. note:: 

    For API specific issues, please see the `WPILib API <https://www.youtube.com/watch?v=dQw4w9WgXcQ>`_
    , `CTRE API <https://api.ctr-electronics.com/phoenix/release/java/>`_, and the `REV Robotics API 
    <https://codedocs.revrobotics.com/java/com/revrobotics/package-summary.html>`_.

Constructor
***********

.. tabs::

    .. tab:: Talon FX & CANCoder

        .. code-block:: java
            :linenos:

            // 初始化 rotor & throttle 馬達
            private WPI_TalonFX mRotor;
            private WPI_TalonFX mThrottle;

            // 初始化 rotor encoder
            private WPI_CANCoder mRotorEncoder; 

            // 初始化 rotor PID controller
            private PIDController mRotorPID; 

            /**
             * 構建新的 SwerveModule
             * 
             * @param throttleID CAN ID of throttle 馬達
             * @param rotorID CAN ID of rotor 馬達
             * @param rotorEncoderID CAN ID of rotor encoder
             * @param rotorOffsetAngleDeg rotor encoder 偏移量
             */
            public SwerveModule(int throttleID, int rotorID, int rotorEncoderID, double rotorOffsetAngleDeg) 
            {
                // 實例化 throttle 馬達
                mThrottle = new WPI_TalonFX(throttleID);

                // 實例化 rotor 馬達 
                mRotor = new WPI_TalonFX(rotorID);

                // 實例化 rotor absolute encoder
                mRotorEncoder = new WPI_CANCoder(rotorEncoderID);

                // 重置所有配置（保險起見以免有舊的配置）
                mThrottle.configFactoryDefault();
                mRotor.configFactoryDefault();
                mRotorEncoder.configFactoryDefault();

                // 根據之前的常數配置 rotor 馬達
                mRotor.setInverted(SwerveConstants.kRotorMotorInversion); 
                mRotor.configVoltageCompSaturation(Constants.kVoltageCompensation);
                mRotor.enableVoltageCompensation(true);
                mRotor.setNeutralMode(NeutralMode.Brake);

                // 根據之前的常數配置轉向 rotor encoder
                mRotorEncoder.configAbsoluteSensorRange(AbsoluteSensorRange.Signed_PlusMinus180);
                mRotorEncoder.configMagnetOffset(rotorOffsetAngleDeg);
                mRotorEncoder.configSensorDirection(SwerveConstants.kRotorEncoderDirection); 
                mRotorEncoder.configSensorInitializationStrategy(
                    SensorInitializationStrategy.BootToAbsolutePosition
                );

                // 根據之前的常數配置 rotor 馬達的PID控制器
                mRotorPID = new PIDController(
                    SwerveConstants.kRotor_kP, 
                    SwerveConstants.kRotor_kI, 
                    SwerveConstants.kRotor_kD
                );

                // ContinuousInput 認為 min 和 max 是同一點並且自動計算到設定點的最短路線
                mRotorPID.enableContinuousInput(-180, 180);

                // 根據之前的常數配置 throttle 馬達
                mThrottle.configSelectedFeedbackSensor(FeedbackDevice.IntegratedSensor);
                mThrottle.configVoltageCompSaturation(Constants.kVoltageCompensation);
                mThrottle.enableVoltageCompensation(true);
                mThrottle.setNeutralMode(NeutralMode.Brake);
            }
        
    .. tab:: SPARK MAX & CANCoder

        .. code-block:: java
            :linenos:

            // 初始化 rotor & throttle 馬達
            private CANSparkMax mRotor;
            private CANSparkMax mThrottle;

            // 初始化 throttle encoder 
            private RelativeEncoder mThrottleEncoder;

            // 初始化 rotor encoder
            private WPI_CANCoder mRotorEncoder; 

            // 初始化 rotor PID controller
            private PIDController mRotorPID; 

            /**
             * 構建新的 SwerveModule
             * 
             * @param throttleID CAN ID of throttle 馬達
             * @param rotorID CAN ID of rotor 馬達
             * @param rotorEncoderID CAN ID of rotor encoder
             * @param rotorOffsetAngleDeg rotor encoder 偏移量
             */
            public SwerveModule(int throttleID, int rotorID, int rotorEncoderID, double rotorOffsetAngleDeg) 
            {
                // 實例化 throttle 馬達 & encoder
                mThrottle = new CANSparkMax(throttleID, MotorType.kBrushless);
                mThrottleEncoder = mThrottle.getEncoder();

                // 實例化 rotor 馬達
                mRotor = new CANSparkMax(rotorID, MotorType.kBrushless);

                // 實例化 rotor absolute encoder
                mRotorEncoder = new WPI_CANCoder(rotorEncoderID);

                // 重置所有配置（保險起見以免有舊的配置）
                mThrottle.restoreFactoryDefaults();
                mRotor.restoreFactoryDefaults();
                mRotorEncoder.configFactoryDefault();

                // 根據之前的常數配置 rotor 馬達
                mRotor.setInverted(SwerveConstants.kRotorMotorInversion); 
                mRotor.enableVoltageCompensation(Constants.kVoltageCompensation);
                mRotor.setIdleMode(IdleMode.kBrake);

                // 根據之前的常數配置轉向 rotor encoder
                mRotorEncoder.configAbsoluteSensorRange(AbsoluteSensorRange.Signed_PlusMinus180);
                mRotorEncoder.configMagnetOffset(rotorOffsetAngleDeg);
                mRotorEncoder.configSensorDirection(SwerveConstants.kRotorEncoderDirection); 
                mRotorEncoder.configSensorInitializationStrategy(
                    SensorInitializationStrategy.BootToAbsolutePosition
                );

                // 根據之前的常數配置 rotor 馬達的PID控制器
                mRotorPID = new PIDController(
                    SwerveConstants.kRotor_kP, 
                    SwerveConstants.kRotor_kI, 
                    SwerveConstants.kRotor_kD
                );

                // ContinuousInput 認為 min 和 max 是同一點並且自動計算到設定點的最短路線
                mRotorPID.enableContinuousInput(-180, 180);

                // 根據之前的常數配置 throttle 馬達
                mThrottle.enableVoltageCompensation(Constants.kVoltageCompensation);
                mThrottle.setIdleMode(IdleMode.kBrake);

                // 給與 throttle encoder 轉換係數以便它以米每秒而不是 RPM 為單位讀取速度
                mThrottleEncoder.setVelocityConversionFactor(
                    SwerveConstants.kThrottleVelocityConversionFactor
                );
            }

    .. tab:: SPARK MAX & Analog Absolute Encoder

        .. code-block:: java
            :linenos:

            // 初始化 rotor & throttle 馬達
            private CANSparkMax mRotor;
            private CANSparkMax mThrottle;

            // 初始化 throttle encoder
            private RelativeEncoder mThrottleEncoder;

            // 初始化 rotor encoder
            private AnalogPotentiometer mRotorEncoder;

            // 初始化 rotor PID controller
            private PIDController mRotorPID; 

            /**
             * 構建新的 SwerveModule
             * 
             * @param throttleID CAN ID of throttle 馬達
             * @param rotorID CAN ID of rotor 馬達
             * @param rotorEncoderID analog ID of rotor encoder
             * @param rotorOffsetAngleDeg rotor encoder 偏移量
             */
            public SwerveModule(int throttleID, int rotorID, int rotorEncoderID, double rotorOffsetAngleDeg) 
            {
                // 實例化 throttle 馬達 & encoder
                mThrottle = new CANSparkMax(throttleID, MotorType.kBrushless);
                mThrottleEncoder = mThrottle.getEncoder();

                // 實例化 rotor 馬達
                mRotor = new CANSparkMax(rotorID, MotorType.kBrushless);

                // 實例化 rotor absolute encoder
                // - 完整範圍 = 360，因為這是編碼器能返回的最大值
                mRotorEncoder = new AnalogPotentiometer(rotorEncoderID, 360, rotorOffsetAngleDeg);

                // 重置所有配置（保險起見以免有舊的配置）
                mThrottle.restoreFactoryDefaults();
                mRotor.restoreFactoryDefaults();

                // 根據之前的常數配置 rotor 馬達
                mRotor.setInverted(SwerveConstants.kRotorMotorInversion); 
                mRotor.enableVoltageCompensation(Constants.kVoltageCompensation);
                mRotor.setIdleMode(IdleMode.kBrake);

                // 根據之前的常數配置 rotor 馬達的PID控制器
                mRotorPID = new PIDController(
                    SwerveConstants.kRotor_kP, 
                    SwerveConstants.kRotor_kI, 
                    SwerveConstants.kRotor_kD
                );

                // ContinuousInput 認為 min 和 max 是同一點並且自動計算到設定點的最短路線
                mRotorPID.enableContinuousInput(-180, 180);

                // 根據之前的常數配置 throttle 馬達
                mThrottle.enableVoltageCompensation(Constants.kVoltageCompensation);
                mThrottle.setIdleMode(IdleMode.kBrake);

                // 給與 throttle encoder 轉換係數以便它以米每秒而不是 RPM 為單位讀取速度
                mThrottleEncoder.setVelocityConversionFactor(
                    SwerveConstants.kThrottleVelocityConversionFactor
                );
            }

.. warning:: 
    
    使用無刷馬達時，強烈建議設置寬鬆的電流限制 以防止損壞馬達。查看 `CTRE <https://
    api.ctr-electronics.com
    /phoenix/release/java/com/ctre/phoenix/motorcontrol/can/TalonFX.html#configSupplyCu
    rrentLimit(com.ctre.phoenix.motorcontrol.SupplyCurrentLimitConfiguration)>`_ 和 
    `REV <https://codedocs.revrobotics.com/java/com/revrobotics/cansparkmax#
    setSmartCurrentLimit(int)>`_ 文檔以獲取有關電流限制的更多信息。

Methods
*******

getState
--------

輸出 swerve module 的當前狀態。

.. tabs::
    
    .. tab:: Talon FX

        .. code-block:: java
            :linenos:

            public SwerveModuleState getState() {
                double throttleVelocity = 
                    mThrottle.getSelectedSensorVelocity() * SwerveConstants.kThrottleVelocityConversionFactor; 

                return new SwerveModuleState(
                    throttleVelocity, 
                    Rotation2d.fromDegrees(mRotorEncoder.getAbsolutePosition())
                );
            }

    .. tab:: SPARK MAX

        .. code-block:: java
            :linenos:

            public SwerveModuleState getState() {
                return new SwerveModuleState(
                    mThrottleEncoder.getVelocity(),
                    Rotation2d.fromDegrees(mRotorEncoder.getAbsolutePosition())
                );
            }

**Return:**
"""""""""""

    新的 `SwerveModuleState <https://first.wpi.edu/wpilib/allwpilib/docs/release/java
    /edu/wpi/first/math/kinematics/SwerveModuleState.html>`_ 代表著目前的前進速度和面相角度。

setState
--------

設置 swerve module 的狀態。

.. code-block:: java
    :linenos:

    public void setState(SwerveModuleState state) {
        // 優化狀態，使轉向馬達不必旋轉超過 90 度來獲得目標的角度
        SwerveModuleState optimizedState = SwerveModuleState.optimize(state, getState().angle);

        // 通過比較目前角度與目標角度來用 PID 控制器計算轉向馬達所需的輸出
        double rotorOutput = mRotorPID.calculate(
            getState().angle.getDegrees(), 
            optimizedState.angle.getDegrees()
        );

        mRotor.set(rotorOutput);
        mThrottle.set(optimizedState.speedMetersPerSecond);
    }

Parameters:
"""""""""""

1. ``state`` - 理想的 `SwerveModuleState <https://first.wpi.edu/wpilib/allwpilib/docs/release/java
   /edu/wpi/first/math/kinematics/SwerveModuleState.html>`_ (angle & speed) of SwerveModule

.. note::
    在我們的 `Github <https://github.com/TASRobotics/RaidZero-Swerve-Template>`_
    上查看我們在這些文檔中使用的代碼！