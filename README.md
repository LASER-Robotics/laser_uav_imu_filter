# LASER UAV IMU Filter

`laser_uav_imu_filter` filters accelerometer and gyroscope measurements before they reach the LASER UAV System (LUS) estimators. It wraps the IIR and notch filters from `laser_uav_lib` in a ROS 2 lifecycle node.

The node accepts a combined IMU stream or separate accelerometer and gyroscope streams, as used by some RealSense configurations. It filters acceleration and angular velocity; it does not estimate orientation.

## Run

For a combined IMU topic:

```bash
ros2 launch laser_uav_imu_filter imu_filter.launch.py   imu_data_united:=true   topic_imu_in:=/uav1/rgbd/imu   topic_imu_out:=/uav1/vio_imu/filtered
```

For separate streams, set `imu_data_united:=false` and pass `topic_accel_in` and `topic_gyro_in`. The launch configures and activates the node automatically.

## Interfaces

Every topic uses `sensor_msgs/msg/Imu`. The node-level names can be remapped with the launch arguments below.

| Direction | Node topic | Launch default | Use |
| --- | --- | --- | --- |
| Input | `imu_in` | `/uav1/rgbd/imu` | Combined data when `imu_data_united=true`. |
| Input | `accel_in` | `/uav1/rgbd/accel/sample` | Acceleration when `imu_data_united=false`. |
| Input | `gyro_in` | `/uav1/rgbd/gyro/sample` | Angular velocity when `imu_data_united=false`. |
| Output | `imu_out` | `/uav1/vio_imu/filtered` | Filtered measurements. |

In separate-stream mode, each gyroscope callback publishes a message containing the latest filtered acceleration. The output keeps the gyroscope timestamp and frame; the node does not interpolate the two streams to a common timestamp.

## Configuration

The supplied [params/rs_d435i.yaml](params/rs_d435i.yaml) enables first-order Butterworth IIR filters and disables notch filters. Its accelerometer coefficients were designed for 100 Hz sampling with a 10 Hz cutoff; its gyroscope coefficients use 200 Hz sampling with a 10 Hz cutoff.

For both `accelerometer` and `gyro`, the parameter groups are:

| Parameter suffix | Meaning |
| --- | --- |
| `iir_filter.enable` | Enables IIR filtering. |
| `iir_filter.a` | Feedback coefficients. |
| `iir_filter.b` | Feedforward coefficients. |
| `notch_filter.enable` | Enables notch filtering. |
| `notch_filter.sample_rate` | Sampling frequency in Hz; supplied value is `1000`. |
| `notch_filter.frequencies` | Frequencies to attenuate in Hz; supplied list is `[100, 200, 300]`. |
| `notch_filter.bandwidth` | Stopband width in Hz; supplied value is `100`. |

For example, `accelerometer.iir_filter.enable` controls the accelerometer IIR filter. Set coefficients and notch frequencies for the actual sensor rate and measured vibration spectrum.

The current node reads the gyroscope notch frequency list from `accelerometer.notch_filter.frequencies`. Setting `gyro.notch_filter.frequencies` alone does not change the gyroscope frequencies in this implementation.

### Launch arguments

| Argument | Default | Meaning |
| --- | --- | --- |
| `namespace` | `UAV_NAME`, or `uav1` | Node namespace. |
| `use_sim_time` | `true` when `REAL_UAV=false` | Uses the simulation clock. |
| `imu_filter_params_file` | Installed `params/rs_d435i.yaml` | Filter configuration file. |
| `imu_data_united` | `True` | Combined or separate input mode. Overrides the `false` value in the supplied YAML. |
| `topic_imu_in`, `topic_accel_in`, `topic_gyro_in` | Paths in the interface table | Input remappings. |
| `topic_imu_out` | `/uav1/vio_imu/filtered` | Output remapping. |

The default topic paths use `UAV_NAME`, falling back to `uav1`. When changing only `namespace`, also pass explicit topic arguments if their paths need to change.
