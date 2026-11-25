# Google One SKU 管理、Pro 12个月优惠测试与采集

> **🍴 Fork 变更（chengyixu/g1-acq）**：默认 20TB 目标改为 12个月免费试用 `g1.play.20tb.12months_eft`（原 `g1.20tb.3months_ip_75p.acq`）。

**v7.9.0 主模块为 G1 ACQ SKU 管理，响应运行版本为 `g1-acq-v7.9-mappings`。** 现有[本地报告](https://one.google.com/g1-sku-report)加入手动映射卡片与表单，支持新增、编辑、勾选启用、保存、删除和恢复默认；采集列表可预填源，源与目标输入支持已采集 SKU 候选。配置保存在手机，下一次 `/ai?` 响应使用新规则，历史采集与执行记录独立显示。

默认 `discount75` 保留原三条映射：20TB 12个月免费试用目标 `g1.play.20tb.12months_eft`、30TB 三个月 75p 与 2TB 目标 `g1.2tb.ai.12months_ip_90p`（无 `.acq`）；手动保存后为 `custom`。**75p = 75% off、实付 25%、2.5折**；20TB 目标为 Play 结算的 12个月免费试用 offer（首期 $0）；本版命中行 `fallback=false`。历史试用映射来自 [GitHub PR #3](https://github.com/crhywun/g1-acq/pull/3)，贡献者 [@cehomecambodia-lang](https://github.com/cehomecambodia-lang) 提供了 20TB / 30TB Ultra 1个月免费试用 SKU 与三个月 90p 备用方案，其历史含义保留。

采集只读取手机收到的原始匹配响应，**不会主动查询全量 SKU**。最多保留 300 个 ID、最近 10 条脱敏响应事件，时间使用 UTC。旧 ID 再次出现会累加次数；当前报告版本与实际记录版本分别显示，升级不重标历史。

默认 2TB 候选来自用户截图：`$1.99 × 12 months = $23.88/year` 表达连续 12 个月的月度优惠，`90p` 表示 **90% off、约一折**。截图标题 **Pro 5TB** 未改变 `g1.ios.2tb.` 默认源，也未新增 5TB 前缀；截图不证明 iOS 商品、一次年付扣款或实际 5TB 权益。商品名称、周期、初次与续费金额按 App Store 返回记录，权益按 Google One 账户实际显示记录。

## 1. 安装与升级主模块

**G1 ACQ SKU 管理 — 本地手动映射，保留 20TB 12个月试用、30TB 三个月 75p 与 2TB 目标**

```text
https://cdn.jsdelivr.net/gh/chengyixu/g1-acq@main/g1-acq-ultra-discount.sgmodule
```

| 项目 | 当前主入口 |
|---|---|
| 当前模式 | 默认 `discount75`；手动保存后 `custom` |
| 响应脚本 | [g1-acq-discount.js](../g1-acq-discount.js)，与 [g1-acq.js](../g1-acq.js) 实现相同 |
| 响应运行版本 | `g1-acq-v7.9-mappings` |
| 报告脚本 | [g1-sku-report-discount.js](../g1-sku-report-discount.js)，与 [g1-sku-report.js](../g1-sku-report.js) 实现相同 |
| 报告版本 | `v7.9.0` |
| 本地存储 | 原采集 / 执行两键 schema 1，独立映射键 `g1.sku.mappings.v1` |

1. 关闭旧 **G1 ACQ Ultra 75p**、**G1 ACQ**、**G1 ACQ Ultra 一折**、**G1 ACQ Ultra 试用**、独立 **G1 SKU Scan**、**G1 Pro Annual Test** 与其它改写副本。
2. 在小火箭模块管理中添加上方 `v7.9.0` 地址，只启用 **G1 ACQ SKU 管理**。模块包含响应与报告脚本，无需单独添加。
3. 重新连接小火箭，再打开 Google App 的方案页。
4. 打开本机报告核对 **本机脚本已接管** 与 **v7.9.0**，分别查看当前配置、实际 `scan.version/mode/updatedAt` 和 `test.version/mode/updatedAt`；历史保持到新记录成功写入。

安装入口与全部内部脚本指向 fork 的 main 分支（`chengyixu/g1-acq@main`），旧固定链接需换新地址。历史 `v7.8.0` / `v7.7.0` / `v7.6.0` / `v7.5.0` / `v7.4.0` / `v7.3.0` / `v7.2.0` / `v7.1.0` / `v7.0.0` 和 `pro-annual-test-v1` / `pro-annual-test-v2` 标签保持原样；试用模块仍缺席，旧固定试用链接仍引用原版本。升级保留两份 schema 1 数据集、累计次数与最近 10 次响应，不要求清空记录。

[g1-acq-v7.sgmodule](../g1-acq-v7.sgmodule) 保留名称 **G1 ACQ**；[g1-pro-annual-test.sgmodule](../g1-pro-annual-test.sgmodule) 保留名称 **G1 Pro Annual Test** 和 `/ai?` 响应范围，均使用同一映射配置。[g1-sku-scan.sgmodule](../g1-sku-scan.sgmodule) 仅更新 pin 与报告请求体配置，[g1-sku-scan.js](../g1-sku-scan.js) 仍为 `pro-annual-test-v2`，保持响应原文，没有模式或响应事件字段。使用主入口时兼容入口保持关闭。

四个模块的报告 `http-request` 均加入 `requires-body=1,max-size=0`。原响应匹配、`[Rule]`、`[MITM]` 保留：hostname 仅追加 `one.google.com`，全局拦截 UDP 443，也影响其它 App 的 HTTP/3、速度或连通性。

## 2. 配置 HTTPS 解密

模块要处理 HTTPS 响应，需在当前使用的配置中启用解密并信任证书。

1. 只启用第 1 节的 `v7.9.0` **G1 ACQ SKU 管理**，旧版用户先关闭旧模块并安装新地址，保持小火箭 VPN 连接。
2. 进入 **配置页 → 当前使用配置右侧的 (i) 图标 → HTTPS 解密**，打开解密开关。
3. 已有可用且已受信任的 Shadowrocket 证书时直接沿用；尚未配置证书时，在该页生成并安装证书，再前往 iOS **设置 → 通用 → VPN 与设备管理** 完成描述文件安装。
4. 首次配置证书时，前往 iOS **设置 → 通用 → 关于本机 → 证书信任设置**，开启对应 Shadowrocket 根证书的信任。
5. 在当前配置的 **HTTPS 解密 hostname 列表**中核对 `one.google.com`。模块使用 `%APPEND%` 追加该域名；此处是 HTTPS 解密列表，不是 DNS Hosts。
6. 更改模块开关、升级模块或更改解密配置后重新连接小火箭，再重新进入 Google App 的方案页面。

[配置入口截图](../docs-config-entry-example.png)与 [README 安装步骤](../README.md#安装shadowrocket--小火箭)展示当前配置右侧的 `(i)` 入口；[Ultra 方案截图](../docs-ultra-plans-example.png)展示 20TB 入口示例。两图保留脱敏后的文件名与内容。MITM hostname 覆盖 `one.google.com` 整个域名，脚本读取与改写范围由下文的 URL 匹配限定。

## 3. 浏览方案，被动采集原始 SKU

主入口处理 `one.google.com/` 响应时先采集原始正文，再检查 `/ai?` 改写范围；候选保留改写前原值，非文本只计入非文本诊断，本地报告页、末尾 `/` 和 query 均跳过。

1. 在 Safari 打开[本地报告](https://one.google.com/g1-sku-report)，记录 `scan.version/mode/updatedAt`、`scannedResponses`、`matchedResponses`、`nonTextResponses` 与最新事件时间。
2. 打开 **Google App（彩色 G 图标）→ Google One 管理**，浏览实际提供的 **2TB / Pro、20TB / 30TB、月付、年付与优惠入口**，等待页面加载，无需完成购买。
3. 返回报告刷新比较。`known-skus` 表示收到已保存 ID，次数与发现时间更新；`no-skus` 表示没有识别到候选。计数与时间均未变时，按第 7 节核对缓存、匹配范围与保存日志。
4. 查看候选的完整 ID、次数与发现时间。点采集行的源预填入口可开始添加映射，或点 **导出 JSON** 保存当前结果。

采集识别 `g1.*` 与 `com.google.one.*`，在识别副本中还原 `\u002e`、`\x2e` 与 HTML 点号实体，该还原步骤不修改响应。同一 ID 在单响应或后续响应重复出现均累加次数，次数不是购买次数。

最多保留 **300 个不同 ID**。满额后继续更新已有 ID，超出的新 ID 计入 `omittedCount`；`addedCount>0` 优先为 `new-skus`，只有 `addedCount=0` 且 `omittedCount>0` 才为 `limit-reached`。响应脚本不读取请求头或请求体，不保存完整正文，不查询 App Store，不向外发送采集数据。

## 4. 编辑规则、核对默认映射与执行

1. 在 Safari 打开[本地报告](https://one.google.com/g1-sku-report)，查看当前配置卡片。从采集列表点源预填入口，或直接使用新增规则表单；源与目标输入框会提示已采集候选，也支持手动输入完整 ID。
2. 选择 **精确 `exact`** 或 **前缀 `prefix`**，填写源和目标 ID，勾选 **启用**，点击该条的 **保存**。已有规则在各自卡片编辑；取消启用后也需保存。
3. **删除**移除该条规则；**恢复默认**恢复下表三条并回到 `discount75`，不清采集 / 执行记录。删除全部规则或全部禁用并保存后为 `custom`，响应原样通过。
4. 保存成功后页面返回固定成功提示；非法输入保留填写值并给出错误。旧页面冲突时先刷新核对新配置，再重新填写提交；保存失败时按反馈核对，未见成功提示不按新规则判断执行。
5. 退出已打开或缓存的方案，重新进入 Google App，使下一次 `/ai?` 响应使用新配置。保存只改配置，最近执行保留到新响应成功写入；比较 `mappingConfig.revision` 与 `test.mappingRevision`，同时核对版本、模式和时间。
6. 从实际方案进入订阅流程，记录 App Store 商品名称、周期、币种、初次与续费金额或实际错误提示，再导出 JSON。默认规则分别核对 2TB / Pro、20TB / 30TB；手动规则核对完整源与实际 `to`。

| 规则字段 | 填写与匹配规则 |
|---|---|
| `match` | `exact` 匹配完整源；`prefix` 匹配带非空后缀的完整 SKU |
| `from` | 精确源为完整 `g1.*` 或 `com.google.one.*` ID；前缀源以点结尾，命名空间之后仍需有内容 |
| `to` | 完整目标 ID，支持 `g1.*` / `com.google.one.*`，不以点结尾，不与源相同 |
| ID 字符 | 命名空间之后 1–160 个 `[A-Za-z0-9._-]` 字符；前缀也受同一长度限制；表单会去掉源与目标两端空白 |
| `enabled` | 勾选为启用；禁用规则保留在列表，但不参与匹配 |
| `id` | 页面生成的稳定规则标识，编辑时保留；不是待改写商品 ID |
| 上限与重复 | 最多 30 条，禁用规则也计数；相同 `match+from` 重复或规则 ID 重复均拒绝 |

**启用的精确规则优先于前缀；多个启用前缀匹配时取最长前缀。** 每个原始 ID 只替换一次，不链式替换目标；例如精确规则 `g1.example.source → g1.example.middle` 与 `g1.example.middle → g1.example.final` 同时存在时，原始 `g1.example.source` 只变成 `g1.example.middle`。累计清单与正文中是否已有目标均不参与目标选择。

主入口匹配 `^https://one\.google\.com/` 响应，先采集，再仅在 `^https://one\.google\.com/ai\?` 中替换完整明文 ID。首次使用或恢复默认时为 `discount75`，三条规则均启用：

| `id` | 匹配方式与源 | 目标 ID |
|---|---|---|
| `default-2tb` | 前缀 `g1.ios.2tb.` | `g1.2tb.ai.12months_ip_90p` |
| `default-20tb` | 精确 `g1.ios.20tb.m.v2` | `g1.play.20tb.12months_eft` |
| `default-30tb` | 精确 `g1.ios.30tb.ai.pro.m` | `g1.ai.ultra.3months_ip_75p.acq` |

默认 2TB 目标保留 90p、无 `.acq`；20TB 使用 12个月免费试用目标、30TB 保留三个月 75p。默认与手动映射均无备用，`fallback=false`；试用 / 90p / 75p 目标是否出现在原始响应中不改变规则。所有规则未命中、全部禁用或列表为空时原样返回响应；配置损坏或存储读取出错也暂停改写，不假定默认配置。

| 执行字段 | 含义 |
|---|---|
| `test.version` / `test.mode` | 执行时版本与模式；本版 `g1-acq-v7.9-mappings`，正常配置下为 `discount75` 或 `custom`，配置出错时模式为空 |
| `test.mappingRevision` / `test.mappingUpdatedAt` / `test.mappingStatus` | 执行使用的配置修订、配置保存 UTC 时间、配置状态；旧记录缺失时不填入当前配置 |
| `test.matchedFrom` | 每行保留 `{id, to, count, fallback}`，`to` 为实际目标，`count` 为该源次数；本版 `fallback=false` |
| `test.target` / `test.targetOrigin` | 执行时启用且源恰为 `g1.ios.2tb.` 的前缀目标；默认来源 `user-specified`，手动配置为 `user-configured`，缺少该前缀则均为空；精确覆盖目标看各行 `to` |
| `test.hitCount` | 全部规则的总改写数；日志 `hits=N` |
| `test.annualHitCount` | 只统计源以 `g1.ios.2tb.` 开头的改写，手动精确命中也计入；日志 `annualHits=N`，HTML 显示“其中 2TB”；字段名不证明年付扣款 |
| `test.updatedAt` | 最近一次执行时间，UTC，与配置保存时间独立 |
| `test.status=rewritten` | 至少一条规则命中 |
| `test.status=no-match` | 没有启用规则命中，响应原样通过 |
| `test.status=non-text` | 正文非文本，响应原样通过 |
| `test.status=config-error` | 配置 `invalid` 或 `unavailable`，暂停改写并保留响应原文 |

日志前缀 `[g1-acq]` 与运行版本 `g1-acq-v7.9-mappings` 用于定位本次执行，包含模式、映射状态 / 修订、`hits`、`annualHits`；逐行显示 `源 SKU → 实际目标 xN`。采集摘要保留 `status`、`addedCount`、`candidates`、`omittedCount`、`occurrences`、`saved=yes/no`，`saved=yes` 才表示该次记录成功写入。本版没有备用日志，旧 `fallback=实际完整目标编号` 与更早 `fallback=75p` 仍按历史读取。

执行键保存最近一次 `/ai?` 检查，后续零命中或非文本也会更新；编辑、删除、恢复规则和升级本身均不改写历史。旧目标推断使用固定默认 2TB 常量，不用新的 `configuredTarget` 回填。历史版本、来源与模式按下表保留：

| 历史版本 | 保留的 2TB 目标 | 目标来源 | Ultra 与模式记录 |
|---|---|---|---|
| `g1-acq-v7.8-75p` | `g1.2tb.ai.12months_ip_90p` | `user-specified` | `mode=discount75`，固定三个月 75p，`fallback=false` |
| `g1-acq-v7.7-trial` | `g1.2tb.ai.12months_ip_90p` | `user-specified` | 已保存的 `mode=trial` 保留试用历史含义；默认 1个月试用、备用 90p 的实际行保留 |
| `g1-acq-v7.7-discount` | `g1.2tb.ai.12months_ip_90p` | `user-specified` | 已保存的 `mode=discount` 保留一折历史含义；默认 90p、备用 75p 的实际行保留 |
| `g1-acq-v7.6` | `g1.2tb.ai.12months_ip_90p` | `user-specified` | 保留 1个月试用默认与 90p 备用的实际行；缺失 `mode` 保持空值 |
| `g1-acq-v7.5` | `g1.2tb.ai.12months_ip_90p` | `user-specified` | 保留当时实际行与备用状态；缺失模式保持空值 |
| v7.3 / v7.4 | `g1.2tb.ai.annual_ip_90p.acq` | `user-specified` | 保留当时实际行；缺失模式保持空值 |
| v7.2 | `g1.2tb.ai.annual.3months_ip_90p.acq` | `pattern-derived` | 保留当时实际行；缺失模式保持空值 |
| v1 / v7.1 | `g1.2tb.ai.annual` | `legacy` | 保留普通年付记录；缺失模式保持空值 |

历史 Ultra 完整目标对应如下，仅用于读取旧记录：

| 历史模式 | 20TB 默认 / 备用 | 30TB 默认 / 备用 |
|---|---|---|
| v7.7 `trial` | `g1.20tb.1month_eft.acq` / `g1.20tb.3months_ip_90p.acq` | `g1.ai.ultra.1month_eft.acq` / `g1.ai.ultra.3months_ip_90p.acq` |
| v7.7 `discount` | `g1.20tb.3months_ip_90p.acq` / `g1.20tb.3months_ip_75p.acq` | `g1.ai.ultra.3months_ip_90p.acq` / `g1.ai.ultra.3months_ip_75p.acq` |

相同目标不代表新版本已执行：v7.5 / v7.6 / v7.7 / v7.8 与本版默认 2TB 目标相同，旧 v7.7 `discount` 的 75p 备用也与当前默认 Ultra 目标相同。历史 `fallback=true` 表示当时使用备用，HTML 按实际 `to` 显示 75p、90p 或“是”；目标不足时只显示“是”。本版与旧 v7.8 固定规则的 `fallback=false` 显示“否”。

历史 `mode` 保留实际值，缺失或未知时为空，旧 `discount` 仍表示一折。旧 v1 仅 2TB 行缺少 `to` 时补为 `g1.2tb.ai.annual`，缺少 `annualHitCount` 时从旧行次数计算，旧行按未使用备用显示。判断实际执行需一起看版本、模式、时间、`to` 与 `fallback`。

`hitCount>0` 且 `annualHitCount=0` 表示有源不以 `g1.ios.2tb.` 开头的规则命中，默认配置下对应 20TB / 30TB。采集可识别转义点号，改写只处理 `/ai?` 原文中的完整明文 ID，采集到某个候选不等同于另一响应已命中。

## 5. 本地管理页、JSON、存储与清理

[本地报告](https://one.google.com/g1-sku-report)显示 **Google One SKU 管理**、**本机脚本已接管** 与 **v7.9.0**。页面由手机 Shadowrocket `http-request` 脚本生成，支持末尾 `/` 和 query，直接返回 HTML / JSON，不把请求交给 Google。HTML 仅用原生表单，无页面 JavaScript 或外部资源，也无网络上报。

| 操作 | 结果 |
|---|---|
| 普通报告 GET | 读取当前配置、累计采集与最近执行，不写配置 |
| [导出 JSON](https://one.google.com/g1-sku-report?format=json) | 只读导出当前 `mappingConfig` 与原有 `scan` / `test` 字段 |
| 页面保存 / 删除 / 恢复默认 | POST 到既有报告地址，只写映射键；成功 303 返回报告 GET，刷新不重复提交 |
| [清空采集与测试记录](https://one.google.com/g1-sku-report?clear=1) | 保留旧 GET 清理行为，只清原两键，不清映射 |

表单只接受 `Origin: https://one.google.com`，提供 `Sec-Fetch-Site` 时须为 `same-origin`；正文为 `application/x-www-form-urlencoded`，可带 `charset=UTF-8`，按 UTF-8 字节计算最大 20KB。只读取这三个头与表单正文，不读 Cookie / 认证头。重复、未知参数与非法百分号编码会被拒绝；GET 的 `action` 不改配置。成功提示区分已保存、已删除、已恢复默认。

| 提交反馈 | 处理 |
|---|---|
| 400 输入不合要求 | 保留转义后的本次填写值，修改 ID、前缀、重复规则或数量后再提交；原配置不变 |
| 403 来源不匹配 | 从同一手机 Safari 打开完整本地报告，用页面表单提交 |
| 413 正文超过 20KB | 按条填写并保存，源与目标使用完整商品编号 |
| 415 正文类型不匹配 | 使用页面原生表单提交 |
| 409 旧页面冲突 / 缺少修订 | 刷新报告核对最新配置，再填写提交，避免覆盖另一页已保存的规则 |
| 503 存储读取 / 写入 / 回读失败 | 页面显示失败而非保存成功；刷新核对实际配置，待本地存储恢复再提交 |
| `mappingConfig.status=invalid` | 存储配置损坏，暂停改写；仅显式恢复默认可重建配置 |
| `mappingConfig.status=unavailable` | 读取接口暂未就绪或读取出错，暂停改写；存储恢复后再核对配置 |

数据保存在手机 `$persistentStore`，所有时间使用 UTC；原两份数据集仍为 schema 1，映射使用独立第三键：

| 键 | 内容 |
|---|---|
| `g1.sku.scan.v1` | 累计候选、次数、时间、实际采集版本与模式、文字 / 命中 / 非文本计数、容量状态与最多 10 条事件 |
| `g1.pro.annual.test.v1` | 最近执行版本与模式、源前缀、命中行、目标与来源、次数、时间和状态，新增 `mappingRevision` / `mappingUpdatedAt` / `mappingStatus` |
| `g1.sku.mappings.v1` | `{schemaVersion:1, revision, updatedAt, customized, rules}`，最多 30 条 |

映射键缺失或为空时使用内建默认规则；默认记录 `customized=false`，恢复默认保存空 `rules` 并重新采用内建三条。手动保存为 `customized=true`，显式空列表表示关闭全部改写，仍为 `custom`。配置 JSON、schema 或规则损坏属于 `invalid`；读取出错属于 `unavailable`，两者均暂停改写。

| 当前报告 JSON 字段 | 含义 |
|---|---|
| `reportVersion` | 当前报告版本 `v7.9.0`，独立于实际采集与执行版本 |
| `mappingConfig.status` | `default` / `custom` / `invalid` / `unavailable` |
| `mappingConfig.revision` | 当前变更标识；未保存默认值为 `defaults-75p-v1`，每次成功保存生成新的 48 位小写十六进制修订；用于并发检查，不是认证令牌 |
| `mappingConfig.updatedAt` | 当前配置保存的 UTC ISO 时间；未保存默认配置时为空 |
| `mappingConfig.mode` / `customized` | 默认 `discount75` / `false`，手动 `custom` / `true`；配置出错时模式为空 |
| `mappingConfig.rules` / `limit` | 当前规则数组 `{id, match, from, to, enabled}` 与上限 `30` |
| `mappingConfig.target` / `targetOrigin` | 当前启用且源恰为 `g1.ios.2tb.` 的前缀目标；默认 `user-specified`、手动 `user-configured`，缺少此规则时均为空 |
| `mappingConfig.error` | 固定错误提示，不回显损坏存储原文 |
| `configuredMode` / `configuredTarget` | 从当前 `mappingConfig` 派生，默认分别为 `discount75` / `g1.2tb.ai.12months_ip_90p` |

`mappingConfig` 完整字段为 `status, revision, updatedAt, mode, customized, rules, target, targetOrigin, error, limit`。保存只改变配置；`scan.mode` / `test.mode` 取实际记录，缺失或未知时 JSON 为空字符串、HTML 显示未记录，不按当前配置、旧版本或目标推断。执行新增的三项配置字段见第 4 节，旧记录缺失时不补入当前值。

| 保存的模式值 | 报告名称 |
|---|---|
| `discount75` | Ultra 20TB 12个月试用 / 30TB 三个月 75p（2.5折） |
| `custom` | 手动映射 |
| `trial` | Ultra 1个月免费试用优先（历史模式） |
| `discount` | Ultra 三个月一折优先（历史模式） |
| 空字符串、缺失或未知值 | 未记录 |

采集与执行独立写入两键：`one.google.com/` 响应可更新采集，只有 `/ai?` 检查才更新执行；保存失败保留旧记录。因此当前配置、采集、执行的模式与时间分别核对。只有 `g1-acq-v7.9-mappings` 标为本版采集记录；`g1-acq-v7.8-75p`、v7.7 两入口、`g1-acq-v7.6` 和 `pro-annual-test-v2` 保留真实旧版本。未知版本或无记录显示“尚未保存本版采集响应记录”，旧记录缺少响应历史时不补造。

`scan.skus` 每项保留 `{id, count, lastSeen}`。升级继续保留累计记录与最近 10 次响应；事件本身没有模式字段，不套用当前 `scan.mode`。

| 采集字段 | 含义 |
|---|---|
| `scan.version` | 实际采集版本；本版为 `g1-acq-v7.9-mappings`，旧记录保留原值 |
| `scan.mode` | 执行当时的配置模式 `discount75` / `custom`；配置出错时为空；历史模式与缺失值保留，不按当前入口推断 |
| `scan.updatedAt` | 最近一次成功写入采集记录的 UTC 时间 |
| `scan.scannedResponses` | 累计文字响应数；非文本不计入，原有含义不变 |
| `scan.matchedResponses` | 至少识别出一个候选 ID 的累计文字响应数，重复 ID 也算命中 |
| `scan.nonTextResponses` | 累计非文本响应数，与文字响应分开 |
| `scan.recentResponses` | 最多 10 条事件；JSON 旧 → 新保存，超过 10 条移出最旧事件；HTML 新 → 旧展示 |

每条响应事件包含以下字段：

| 事件字段 | 含义 |
|---|---|
| `at` | 本次响应处理时间，UTC |
| `path` | 固定类别路径，仅取 `/`、`/ai`、`/plans`、`/_/…/data/batchexecute` 或 `[其他 one.google.com 路径]` |
| `bodyType` | 响应体类型，不包含正文内容 |
| `status` | 本次响应的采集诊断状态，见下表 |
| `candidateCount` | 本响应内识别出的不同候选 ID 数；同一 ID 多次出现只计 1 个候选 |
| `addedCount` | 本次新加入并实际保留的不同 ID 数 |
| `omittedCount` | 本次因 300 个 ID 容量限制而丢弃的不同 ID 数；同一未保留 ID 多次出现只计 1 次丢弃 |

| `status` | 含义 |
|---|---|
| `new-skus` | `addedCount>0`，本次保留新增 ID；即使 `omittedCount>0` 仍优先使用此状态 |
| `known-skus` | 本次识别出的 ID 均已保存，只更新累计次数与发现时间 |
| `no-skus` | 本次文字响应未识别到候选 ID |
| `limit-reached` | 仅在 `addedCount=0` 且 `omittedCount>0` 时使用：没有新保留 ID，存在容量丢弃 |
| `non-text` | 本次响应不是可采集文字，增加 `nonTextResponses` |

`candidateCount`、`addedCount`、`omittedCount` 按每个响应中的不同 ID 计数；`scan.skus[].count` 累计字符串出现次数，两者含义不同。已有记录缺少 `recentResponses` 时，不根据旧时间或累计计数补造历史；实际处理的新响应才产生事件。最近 10 次响应不代表全部历史。

响应路径固定脱敏：`/_/…/data/batchexecute` 中的 `…` 是固定展示文本，其它路径归入 `[其他 one.google.com 路径]`。事件与导出不保存 query、完整 URL、请求头、响应头、请求体或响应正文。用最后一次响应时间与最近历史对照本次打开方案页的时间。

**清空采集 / 测试记录不清映射，恢复默认不清历史。** `?clear=1` 只清 `g1.sku.scan.v1` 和 `g1.pro.annual.test.v1`，保留 `g1.sku.mappings.v1`；普通报告与导出不清理。需要重置历史时先导出，清理后返回不带 `clear=1` 的[本地报告](https://one.google.com/g1-sku-report)，避免刷新清理地址再次清除新记录。

## 6. 在 iOS 小火箭中采集与验证隐藏 SKU

当前项目运行在 **iPhone + Shadowrocket + Google App** 中。某个方案未显示在界面上，但其 ID 已在匹配原始正文里时，采集仍会提取；服务器未下发的 ID 不会进入清单。

1. 在 Google App 依次浏览实际月付、年付和优惠入口，等待新响应。
2. 在 Safari 查看[本地报告](https://one.google.com/g1-sku-report)中的原始候选，点采集行预填源，按第 4 节填写规则并保存；候选输入只帮助填写，不自动启用规则。
3. 重新进入对应源方案，按 `test.version`、`test.mode`、`test.mappingRevision`、`test.updatedAt` 与各行 `to` 核对下一响应。使用默认配置时，2TB 保留 90p，20TB 使用 12个月免费试用目标，30TB 保留 75p，`fallback=false`。
4. 进入原生订阅流程，记录 App Store 实际商品、周期、初次与续费金额或错误提示，再导出 JSON 对照。

原始采集与配置目标分开保存，替换生成的目标不会自动成为已采集 SKU。脚本只处理网络响应，没有直接调用 Google App 内 StoreKit；`Product.products(for:)` / `SKProductsRequest` 是由对应 App 查询已知 ID 的商品接口，不是全量商品目录。报告命中描述文本改写，实机订阅结果按实际显示填写。

## 7. 常见结果的判断

| 观察结果 | 下一步 |
|---|---|
| Google 普通页、404，缺少本机标识 / `v7.9.0` / 编辑控件 | 核对新模块、VPN、当前配置 `(i) → HTTPS 解密`、`one.google.com` hostname 与证书信任，再粘贴完整报告 URL |
| 保存后方案仍旧 | 保存仅作用于下一响应；退出缓存方案再进入，比较配置修订与 `test.mappingRevision` |
| 当前 `custom`，执行仍为 `discount75` / `trial` / `discount` | 历史未改写；按版本、模式、时间核对新响应是否成功写入 |
| 报告 `v7.9.0`，采集仍为 `g1-acq-v7.8-75p` 或更早 | 当前显示旧采集记录；关闭旧副本、重新连接并重开方案，再核对时间 |
| 采集模式已更新、执行仍旧 | 两键独立写入且范围不同，查看 `/ai?` 是否产生新执行记录 |
| `invalid` / `config-error` | 配置损坏时暂停改写；显式恢复默认后重新进入方案 |
| `unavailable` 或 503 保存失败 | 核对本地存储状态；页面未声称保存成功，刷新查看实际配置 |
| 409 冲突 | 旧页面修订过期，刷新核对最新规则后再提交 |
| 规则有误、重复或超过 30 条 | 按保留的输入修正；禁用规则也计入数量与重复检查 |
| 全部禁用或全部删除后 `no-match` | 空规则 / 禁用规则不改写；恢复默认会重建三条启用规则 |
| 只有 `known-skus`、没有新 ID | 本次是重复数据，比较次数、发现时间和最新事件 |
| 三个计数与最新时间均未变 | 先看 `saved=no` / `SKU collection unavailable`，再核对缓存、响应范围、唯一入口与解密配置 |
| 已采集 ID，但没有改写 | 核对规则启用、精确 / 前缀、完整明文 ID 与 `/ai?` 范围，采集与改写范围不同 |
| `hitCount>0`、`annualHitCount=0` | 命中源不以 `g1.ios.2tb.` 开头；查看实际行 `id` 与 `to` |
| 旧模式、旧目标或 `fallback=true` 仍显示 | 按第 4 节历史表读取，不把当前配置套入历史；目标相同不证明新版本运行 |
| `addedCount=0`、`omittedCount>0` | 达到 300 ID 上限；需要清理时先导出，仅清采集 / 执行两键，映射保留 |
| 改写命中但收银台报错、未出现或商品不同 | 分别记录实际 `to`、商品 / 金额与错误提示，文本命中不代替商店返回 |

按“先记录 → 重开实际方案 → 刷新比较”核对活动。重复安装或反复清空不会使服务器下发新 ID；没有模式或响应历史的旧记录保留空值，不补造。

## 8. 实机记录模板

```text
采集 / 测试时间（含时区）：[填写]
iOS / Google App / Shadowrocket 版本：[填写]
唯一开启的主模块：G1 ACQ SKU 管理
主安装地址：https://cdn.jsdelivr.net/gh/chengyixu/g1-acq@main/g1-acq-ultra-discount.sgmodule
实际模块地址 / 发布版本：[填写；当前 v7.9.0]
预期响应版本：g1-acq-v7.9-mappings
旧模块与独立扫描器关闭情况：[填写实际状态]
重新连接 / 重开方案时间：[填写]
报告本机标识 / reportVersion：[填写实际显示；当前 v7.9.0]
当前 configuredMode / configuredTarget：[填写实际值；默认 discount75 / g1.2tb.ai.12months_ip_90p]
当前 mappingConfig.status / customized / error：[填写]
当前 mappingConfig.revision / updatedAt（UTC）：[填写]
当前 mappingConfig.rules：[填写 {id, match, from, to, enabled}，最多30条]
本次新增 / 编辑 / 禁用 / 删除 / 恢复操作：[填写实际操作]
保存结果 / 错误提示：[填写实际显示；未执行则填未执行]
保存后重新进入方案时间：[填写]
实际 scan.version / mode / updatedAt（UTC）：[填写实际记录，不重标历史或缺失模式]
打开方案前 / 后 scannedResponses / matchedResponses / nonTextResponses：[填写]
打开方案前 / 后最新响应时间：[填写；缺失则记未记录]
scan.recentResponses：[按 JSON 旧→新填写最多10条；HTML 新→旧；不补写事件模式]
最新事件 at / path / bodyType / status：[填写 UTC 时间、脱敏路径、类型、采集状态]
最新事件 candidateCount / addedCount / omittedCount：[填写]
Google App 实际方案 / 原始完整源 SKU：[填写实际显示与采集值]
默认 2TB 目标：g1.2tb.ai.12months_ip_90p（无 .acq）
默认 20TB / 30TB 目标：g1.play.20tb.12months_eft / g1.ai.ultra.3months_ip_75p.acq
手动规则目标：[填写实际配置；未使用则填默认]
实际 test.version / mode / updatedAt（UTC）：[填写实际记录，不按当前配置补写]
实际 test.mappingRevision / mappingUpdatedAt / mappingStatus：[填写；旧记录缺失则记未记录]
当前配置与实际执行修订是否一致：[填写对照结果]
test.matchedFrom：[逐行填写 {id, to, count, fallback}；本版 false，历史按原值]
test.target / targetOrigin：[填写目标 ID 与来源；来源为 user-specified / user-configured / pattern-derived / legacy / 空值]
test.hitCount / annualHitCount / status：[填写总数 / g1.ios.2tb.源改写数 / 状态]
保留的历史版本 / 模式 / 时间 / to / fallback：[填写实际值；缺失保持空值]
日志模式 / 配置状态 / 修订 / saved / hits / annualHits：[填写实际值]
日志 fallback：[本版无备用日志；历史按实际原值]
App Store 是否拉起 / 商品 / 周期：[填写实际显示]
币种 / 初次金额 / 续费金额 / 错误提示：[填写实际显示]
是否完成购买 / 购买后账户权益：[按实际填写；未购买则记未购买]
本轮 JSON 导出文件：[填写]
```

## 9. 离线验证与生成副本

使用 Node.js 18 或更新版本，在仓库根目录执行：

```sh
node tools/build-variants.cjs --check
node --test tests/*.test.cjs
```

[lib/mapping-config.js](../lib/mapping-config.js) 提供共享配置引擎；[tools/build-variants.cjs](../tools/build-variants.cjs) 把生成区块嵌入 [g1-acq.js](../g1-acq.js) 与 [g1-sku-report.js](../g1-sku-report.js)，再生成 [g1-acq-discount.js](../g1-acq-discount.js) 与 [g1-sku-report-discount.js](../g1-sku-report-discount.js)。修改源后执行 `node tools/build-variants.cjs`，`--check` 只检查两个源内区块和两个副本，不写文件。

验证项目包括默认 2TB、20TB 12个月试用与 30TB 75p 规则、手动增删改 / 禁用 / 恢复、空列表、精确优先 / 最长前缀 / 单次替换、两个 SKU 命名空间、30 条上限、表单错误与旧页面冲突、存储失败、下一响应生效、历史独立、300 / 10 上限、JSON 与模块 pin / 请求体标记。测试结果按实际命令输出记录，实机结果按第 8 节填写。
