# Sub-Store 节点重命名脚本

## 概述

本脚本（`rename.js`）用于 Sub-Store 订阅节点的名称标准化、国家识别、专线识别、云服务厂商识别、分组排序与序号生成。

脚本采用固定的纯净命名格式：

```text
国旗 国家 机场名/自定义名 专线 序号 云服务厂商
```

示例：

```text
🇭🇰 香港 机场A 专线 01 Azure
🇭🇰 香港 机场A 02
🇺🇸 美国 机场A 专线 01 AWS
🇯🇵 日本 机场A 01
```

倍率、节点协议、地区别名、原始线路编号、套餐信息和其他无关内容均不会保留。

## 功能特性

- **固定纯净格式**：只输出国旗、国家、机场名、专线、序号和云服务厂商。
- **智能国家识别**：内置多国家/地区映射，支持中文、英文、国家代码、城市名和常见别名。
- **国旗与多语言**：支持中文国家名、英文国家名或两位国家代码。
- **专线识别**：识别 `专线`、`DL`、`dedicated`、`dedicated line` 等关键词。
- **云厂商识别**：从原始节点名中识别云服务厂商，并统一追加到名称最后。
- **国家分组序号**：按国家分组生成 `01`、`02` 至 `99` 的两位序号。
- **节点排序**：可按国家识别顺序和线路标签对节点进行整理。
- **无效节点过滤**：过滤套餐、到期、流量、官网、客服等非节点信息。
- **未识别节点过滤**：无法识别国家或地区的节点不会保留。
- **本地执行**：脚本不请求外部接口，不上传订阅内容。

## 命名结构

| 位置 | 内容 | 来源 |
| --- | --- | --- |
| 1 | 国旗 | 原节点名中的国家或地区 |
| 2 | 国家/地区 | `countryLabelType` |
| 3 | 机场名/自定义名 | `providerLabel` |
| 4 | 专线 | 原节点名中识别到专线关键词时添加 |
| 5 | 序号 | 按国家分组自动生成 |
| 6 | 云服务厂商 | 从原节点名中自动识别 |

没有识别到的字段不会占位，也不会保留原始杂项。

## 参数说明

| 参数名 | 取值 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `countryLabelType` | `zh` / `en` / `code` | `zh` | 国家标签类型：中文、英文或国家代码 |
| `providerLabel` | 任意字符串 | 空 | 机场名或自定义名称 |
| `addFlagLabel` | `true` / `false` | `true` | 是否添加国旗 |
| `sortNodes` | `true` / `false` | `false` | 是否按国家和线路顺序整理节点 |
| `rmSingleIdx` | `true` / `false` | `false` | 单个节点国家是否移除序号 |
| `indexLabelSep` | 1-2 个符号 | 无 | 序号前后的分隔符；默认不添加任何前缀 |
| `filterInvalid` | `true` / `false` | `true` | 是否过滤套餐、官网、流量等无效信息 |
| `filterUnmatched` | `true` / `false` | `false` | 兼容旧参数；本版本始终过滤无法识别国家的节点 |
| `blockQuic` | `true` / `false` | `false` | 是否为节点添加 `block-quic=on` |

以下参数为原脚本兼容参数，本版本不会把对应内容输出到节点名称中：

| 参数名 | 兼容说明 |
| --- | --- |
| `addRateLabel` | 不输出倍率 |
| `addLineLabel` | 仅保留专线识别，不输出其他线路标签 |
| `customLabel` | 不输出自定义标签 |
| `providerLabelPos` | 机场名固定放在国家后 |
| `providerLabelSep` | 不使用旧版机场标签分隔符 |
| `attrLabelSep` | 不使用旧版属性标签分隔符 |
| `attrItemSep` | 不使用旧版属性项目分隔符 |
| `rateRange` | 不进行倍率区间筛选 |

## 使用方法

### 方式一：Sub-Store 添加本地脚本

1. 打开 Sub-Store。
2. 进入“订阅管理”。
3. 选择单条订阅。
4. 打开“脚本操作”或“重命名”。
5. 添加 `rename.js`。
6. 设置脚本参数并运行。

推荐参数：

```text
providerLabel=机场A
countryLabelType=zh
addFlagLabel=true
sortNodes=true
rmSingleIdx=false
```

### 方式二：通过脚本 URL 使用

将脚本发布到自己的 GitHub 仓库或其他静态地址，然后把参数拼接到脚本 URL 后面：

```text
https://raw.githubusercontent.com/szt416/substore-rename/refs/heads/main/rename.js#providerLabel=%E6%9C%BA%E5%9C%BAA&countryLabelType=zh&addFlagLabel=true&sortNodes=true
```

参数之间使用 `&` 连接，参数值需要进行 URI 编码。

中文“机场A”的 URI 编码为：

```text
%E6%9C%BA%E5%9C%BAA
```

## 常用示例

### 中文国家名、国旗、机场名和排序

```text
providerLabel=机场A
countryLabelType=zh
addFlagLabel=true
sortNodes=true
```

### 英文国家名

```text
providerLabel=MyVPS
countryLabelType=en
addFlagLabel=true
sortNodes=true
```

输出示例：

```text
🇺🇸 United States MyVPS 01 AWS
```

### 使用国家代码

```text
providerLabel=机场A
countryLabelType=code
addFlagLabel=true
sortNodes=true
```

输出示例：

```text
🇭🇰 HK 机场A 专线 01
```

### 单节点国家不显示序号

```text
providerLabel=机场A
rmSingleIdx=true
```

### 自定义序号分隔符

```text
providerLabel=机场A
indexLabelSep=[]
```

输出类似：

```text
🇭🇰 香港 机场A [01] AWS
```

## 专线识别

原节点名称中出现以下关键词时，会添加 `专线`：

- `专线`
- `專線`
- `DL`
- `dedicated`
- `dedicated line`

其他线路信息，例如 `CN2`、`CMI`、`IPLC`、`IEPL`、`BGP`、`家宽`、`商宽` 等，不会输出。

## 云服务厂商识别

脚本会从原始节点名称中识别以下厂商，并统一为标准名称：

| 原名关键词 | 输出 |
| --- | --- |
| AWS、Amazon Web Services、亚马逊云 | `AWS` |
| Azure、Microsoft Azure、微软云 | `Azure` |
| GCP、Google Cloud、谷歌云 | `Google Cloud` |
| Alibaba Cloud、Aliyun、阿里云 | `阿里云` |
| Tencent Cloud、TCloud、腾讯云 | `腾讯云` |
| Huawei Cloud、HuaweiCloud、华为云 | `华为云` |
| OCI、Oracle Cloud | `Oracle Cloud` |
| Vultr | `Vultr` |
| DigitalOcean | `DigitalOcean` |
| Linode、Akamai Cloud | `Linode` |
| Hetzner | `Hetzner` |
| OVH | `OVH` |
| Bandwagon Host、BWH、搬瓦工 | `搬瓦工` |

如果原节点名中没有匹配到云厂商，名称不会添加云厂商字段。

## 节点处理流程

1. 过滤套餐、到期、流量、官网、客服等无效信息。
2. 从原始名称中识别国家或地区。
3. 无法识别国家或地区的节点直接过滤。
4. 从原始名称中识别专线和云服务厂商。
5. 按参数生成国旗和国家名称。
6. 添加机场名或自定义名称。
7. 按国家分组生成两位序号（`01` 至 `99`）。
8. 将云服务厂商追加到最终名称。
9. 清理临时字段并返回节点。

## 技术说明

- 主入口函数为 `operator(nodes)`。
- 兼容 Sub-Store 脚本运行环境。
- 不依赖第三方库。
- 不访问外部 API。
- 订阅节点内容只在本地处理。

## 文件

```text
rename.js   Sub-Store 重命名脚本
README.md   使用说明
```

## 许可证

本项目采用 MIT License。
