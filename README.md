# xr-environments

UKUI XR 的可独立安装环境数据。首个包为 `xr-environment-qingkong`，采用晴空最终版的六方向拼接全景；不重新生成、镜像或拉伸原资产。

## 构建与文件

```sh
dpkg-buildpackage -us -uc -b
dpkg-deb -x ../xr-environment-qingkong_0.1.0_all.deb /tmp/xr-environment-inspect
```

运行时内容安装到 `/usr/share/xr/environments/qingkong/`：

- `environment.json`：版本、方向、色彩、采样和纹理摘要。
- `qingkong-360.png`：4096×2048、sRGB、完整 equirectangular 全景。
- `ARTWORK-LICENSE.txt`：美术资产的原有授权边界。

这是 `Architecture: all` 的纯数据包，没有安装脚本、网络下载、服务、自启动、硬件访问或 runtime 默认值修改。包内容可在任何目录解包检查；Shell 实现后通过环境 ID `qingkong` 选择它。

## 渲染约定

世界坐标为 +Y 向上、-Z 正前、+X 右侧。纹理图片左上角为 UV 原点：

```text
direction = normalized world-space view ray
u = fract(atan2(direction.x, -direction.z) / (2*pi) + 0.5)
v = 0.5 - asin(clamp(direction.y, -1, 1)) / pi
```

前方位于 `(u,v)=(0.5,0.5)`，右侧 `(0.75,0.5)`，左侧 `(0.25,0.5)`，背面为 U 接缝。U repeat、V clamp；按 sRGB 解码后在线性空间采样/合成。采样方向来自世界朝向，忽略相机位置；不跟随头部旋转锁屏。每眼使用各自视线，环境本身位于无限远。

全景中画出的丝带、地面与反射没有近景深度或碰撞。原 GLB 保留在设计资产中供预览；运行包只包含完整纹理，避免在同一包中重复嵌入一份 PNG。桌面 glTF/WebGL 检查不等于 Rust Shell 或头显验收。

## 维护边界

环境 schema 是本仓的输入格式；Shell 消费受支持的版本。未知版本应明确拒绝并使用 Shell 内建恢复背景。协议变更与纯纹理替换分别管理版本。

当前为本地开发包。美术的来源与使用记录保留在原设计任务；此处没有为美术重新指定开放分发许可证。打包文件按 GPL-3.0-or-later 维护，见 `LICENSE.md`；美术说明随包保存。
