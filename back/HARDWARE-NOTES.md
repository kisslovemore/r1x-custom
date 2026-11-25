# R1X 硬件备忘（N305 / fnOS）

## 风扇（it8628 温控）

### 问题
内核 it87 驱动不认 it8628，需强制 force_id 加载。

### 持久化
- /etc/modprobe.d/it87-force.conf：
  options it87 force_id=0x8628 ignore_resource_conflict=1
- 开机 udev 晚于服务，r1-toolbox.service 用 ExecStartPre=/usr/local/bin/r1-wait-hwmon.sh
  等待 hwmon name=it87/it8628 出现后再启动。

### 实测
- hwmon9 = it8628；fan2_input=1480、fan3_input=1216 在转，fan4/fan5=0。
- 默认 pwm3 = CPU 风扇、pwm2 = 机箱风扇。

## RGB 灯（I2C bus1 / 0x26）

### 协议
- 控制器在 I2C bus1 / 地址 0x26；i2cdetect 对 0x26 不应答，须用 i2cget/i2cset -f -y。
- 0x90 模式：0=关 / 1=常亮 solid / 2=彩虹 rainbow / 3=呼吸 breathing。
- 0x92 红、0x93 绿、0x94 蓝（0-255）。

### 持久化
- i2c_dev 模块不常驻会导致 /dev/i2c-1 缺失、写入被吞；
  /etc/modules-load.d/i2c-dev.conf 写入 i2c_dev 常驻。
- 旧兜底服务 r1-rgb-rainbow.service 已 disable --now。

### 注意
- 灯珠硬件偏弱，纯蓝偏暗，不做混色提亮，维持纯蓝。

## 服务与编译
- 编译：Docker golang:1.19-bookworm，容器内装 libsdl2-dev/libsdl2-ttf-dev，
  GOFLAGS=-mod=vendor CGO_ENABLED=1 go build -o r1-toolbox ./cmd。
- 部署：停服务 → 替换 /usr/local/bin/r1-toolbox → 启动 r1-toolbox.service。
- 回滚备份：/usr/local/bin/r1-toolbox.bak-1125。
