# RollingGo UV 参考文件

> 本文件适用于 uv / uvx / Python 环境。
> 交互策略、语气和分支逻辑请看 `../SKILL.md`。

## 最新版执行前缀

```bash
uvx --refresh --from rollinggo@latest rollinggo
```

## API Key 配置

解析顺序：`--api-key` 参数 → `RollingGo_API_KEY` 环境变量。宿主丢失 Key 时，见 [claw-host-env.md](claw-host-env.md)。

## 这个技能最常用的 3 个命令

### 1. 搜候选酒店：`search-hotels`

```bash
uvx --refresh --from rollinggo@latest rollinggo search-hotels \
  --origin-query "杭州西湖附近值得盯价的高档酒店" \
  --place "杭州西湖" \
  --place-type "<先用 --help 查看合法值>" \
  --check-in-date 2026-05-01 \
  --stay-nights 2 \
  --adult-count 2 \
  --size 5
```

### 2. 查具体酒店：`hotel-detail`

```bash
uvx --refresh --from rollinggo@latest rollinggo hotel-detail \
  --hotel-id 123456 \
  --check-in-date 2026-05-01 \
  --check-out-date 2026-05-03 \
  --adult-count 2 \
  --room-count 1
```

### 3. 查标签：`hotel-tags`

```bash
uvx --refresh --from rollinggo@latest rollinggo hotel-tags
```

## 盯价技能里的典型使用顺序

### 已有订单用户

1. 先拿到酒店名、至少一个定位信息、日期、人数、房型
2. 如果还没有 `hotelId`，先用 `hotel-detail --name` 或 `search-hotels` 解析候选酒店
3. 用“酒店名 + 城市 / 区域 + 可拿到的地址 / 品牌信号”确认具体是哪一家；如果仍然有歧义，就让用户确认
4. 对已确认的酒店用 `hotel-detail` 查当前价格和取消规则
5. 必要时再补平台、方案名、原始下单价
6. 按 `booked_price_protection` 或用户自定义场景生成 `watch_config`
7. 如果 agent 有 `Heartbeat`、`Cron` 或其他定时任务能力，就用 `watch_config` 直接创建定时监控
8. 如果没有定时能力，再产出包含 `watch_config` 的 `监控任务摘要`

### 未下单用户

1. 先用 `hotel-tags` 处理模糊偏好
2. 用 `search-hotels` 拿 3 到 5 个候选
3. 用户点中某家后，再用 `hotel-detail` 深查
4. 如果用户命中"周末度假监控"、"出差备选监控"、"心仪酒店蹲守"等场景，先套用对应模板生成 `watch_config`
5. 如果用户想继续观察，再按监控路径处理：agent 有 `Heartbeat`、`Cron` 或其他定时任务能力，就用 `watch_config` 创建定时监控；否则转成包含 `watch_config` 的 `监控任务摘要`
6. 如果用户想现在预订，就提供结果里的预订 URL 或酒店详情页链接，并概括推荐房型、当前价格和取消规则

## 关键规则

- `uvx` 这里必须用 `--from rollinggo`
- 显式使用 `rollinggo@latest`，让 uv 解析当前最新发布版本
- `--place-type` 需要来自 `search-hotels --help`
- `hotel-detail` 不支持 `--format table`
- 不要只靠 `--name` 的模糊匹配直接比价，先确认具体酒店实体
- 没有真实历史波动数据时，不要捏造波动百分比
- 如果 agent 有 `Heartbeat` 或 `Cron`，优先用它们承接定时盯价，不要只停在一次性摘要
- 用户只说场景名时，优先套用 `weekend_getaway`、`business_backup`、`favorite_hotel`、`booked_price_protection`；说不准时才用 `custom`
- `watch_config` 只是配置；只有真实创建提醒 / 任务成功后，才能说监控已经建立

## 排查

- **终端找不到 `rollinggo`：** 直接用 `uvx --refresh --from rollinggo@latest rollinggo ...`
- **提示缺少 API Key：** 设置 `RollingGo_API_KEY`、传 `--api-key`，或者如果你跑在任何 claw 风格宿主里，就按 [claw-host-env.md](claw-host-env.md) 把同一个 key 注入到那个宿主的配置层
- **没有搜索结果：** 从星级、标签、距离、预算开始逐步放宽
