# R1X 定制变更说明（2026-11-25）

基于上游 mingyangbest/r1-frontscreen v1.0.2 (42bd4ce)，本仓库定制改动见 commit 95da17e。

## 1. 修复：点亮屏幕立刻被关
- 现象：自动息屏后在 Web 面板点【点亮屏幕】，同一秒又被“超时自动息屏”关掉。
- 根因：主循环超时判定用 lastTouchTime，该时间戳只被触摸刷新；
  Web /api/screen/on → screen.Wake() 不刷新，点亮瞬间 time.Since(lastTouchTime) 已超阈值。
- 修复（cmd/main.go）：主循环记录 prevOff，检测到从“关”翻到“开”的唤醒边沿时，
  把 lastTouchTime 重置为当前帧，空闲计时从点亮重新算起。任何来源唤醒（Web/触摸/按钮）都覆盖。

## 2. RGB 总线参数化（internal/hardware/rgb.go + cmd/main.go）
- NewRGBController() 改为 NewRGBController(bus string)；bus 为空回落 "1"。
- main.go 传入 cfg.RGB.I2CBus（config.yaml: rgb.i2c_bus）。
- 不再硬编码 bus0；本设备控制器在 bus1/0x26。

## 3. 文案改 R1x
- 删除 Web 灯效页“R1 前屏硬件没有系统级灯效控制通路…”提示。
- 标题：屏幕控制台 · R1x；Bark 测试通知标题 R1x 测试。
- 同时改源码 web-ui（App.vue / index.html）与编译产物 dist，便于无 node 环境直接生效。

## 构建与部署
- 编译：Docker golang:1.19-bookworm，容器内装 libsdl2-dev/libsdl2-ttf-dev，
  GOFLAGS=-mod=vendor CGO_ENABLED=1 go build -o r1-toolbox ./cmd
- 部署：停服务 → 替换 /usr/local/bin/r1-toolbox → 启动 r1-toolbox.service。
- 回滚备份：/usr/local/bin/r1-toolbox.bak-1125

## 验证
- 点亮：自动息屏后点点亮，仅出【唤醒】，不再紧跟超时关屏。
- RGB：面板切模式/颜色，灯正常响应。
