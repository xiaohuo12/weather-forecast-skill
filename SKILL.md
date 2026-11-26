---
name: weather-forecast
description: "查询全球城市的当前天气与未来预报，包括气温、体感、天气状况、降雨概率、风速、湿度、穿衣与出行建议。当用户询问天气、今天/明天/周末天气、气温、温度、是否下雨、降雨概率、风大不大、湿度、要不要带伞、穿什么衣服、出行天气，或出现 weather / forecast / temperature 等关键词时使用本技能。基于 Open-Meteo 免费公共 API，无需注册、无需 API Key。"
---

# 天气查询技能

用 Open-Meteo 的免费公开 API 查询全球任意城市的当前天气与未来预报。全程无需 API Key、无需注册。

## 适用场景

- "今天天气怎么样？" / "北京现在多少度？"
- "明天会下雨吗？" / "周末要带伞吗？"
- "上海未来三天预报" / "东京气温多少？"
- "体感温度、湿度、风速是多少？"
- 任何 weather / forecast / temperature 相关查询

不适用：历史天气、台风预警、专业气象建模（请查当地气象局）。

## 查询流程

分两步：先把城市名解析成经纬度，再查天气。两步都是 GET 请求，用 `curl` 即可。

### 第 1 步：城市名 -> 经纬度

```bash
curl -s "https://geocoding-api.open-meteo.com/v1/search?name=<城市名>&count=1&language=zh&format=json"
```

- `<城市名>` 需 URL 编码：中文用原字（curl 会自动处理），英文/拼音直接写，如 `Beijing`、`New%20York`
- 也支持机场代码和大部分语言的城市名
- 从返回的 `results[0]` 取 `latitude`、`longitude`、`timezone`、`country`
- 若 `results` 为空，说明城市名没匹配到，提示用户换个写法（如用拼音或英文）

示例返回（北京）：

```json
{"results":[{"name":"北京","latitude":39.9075,"longitude":116.39723,"timezone":"Asia/Shanghai","country":"中国"}]}
```

### 第 2 步：经纬度 -> 天气

```bash
curl -s "https://api.open-meteo.com/v1/forecast?latitude=<纬度>&longitude=<经度>&current=temperature_2m,relative_humidity_2m,apparent_temperature,precipitation,weather_code,wind_speed_10m&daily=weather_code,temperature_2m_max,temperature_2m_min,precipitation_probability_max&timezone=<时区>&forecast_days=3"
```

- `<时区>` 用第 1 步返回的 `timezone` 值，中文用户常见为 `Asia/Shanghai`，URL 里斜杠写成 `%2F`
- `forecast_days` 取 1~7，按用户问的时间范围调整：问"今天"取 1，"明天"取 2，"周末/未来几天"取 3~7
- 需要逐小时数据时，把 `current` 换成 `hourly=temperature_2m,precipitation_probability,weather_code`

返回结构要点：

- `current`：当前天气。`temperature_2m` 气温、`apparent_temperature` 体感、`relative_humidity_2m` 湿度、`precipitation` 当前降水量(mm)、`weather_code` 天气代码、`wind_speed_10m` 风速(km/h)
- `daily`：逐日预报。`time[]` 日期、`weather_code[]` 天气、`temperature_2m_max[]` / `temperature_2m_min[]` 最高最低温、`precipitation_probability_max[]` 最大降雨概率(%)

## WMO 天气代码对照

接口返回的是数字代码，按此表翻译成中文再回答用户：

| 代码 | 含义 | 代码 | 含义 |
|------|------|------|------|
| 0 | 晴 | 45, 48 | 雾 |
| 1 | 大部晴朗 | 51, 53, 55 | 毛毛雨（小/中/大）|
| 2 | 局部多云 | 56, 57 | 冻毛毛雨 |
| 3 | 阴 | 61, 63, 65 | 小雨 / 中雨 / 大雨 |
| 66, 67 | 冻雨 | 71, 73, 75 | 小雪 / 中雪 / 大雪 |
| 77 | 雪粒 | 80, 81, 82 | 小阵雨 / 中阵雨 / 强阵雨 |
| 85, 86 | 阵雪 | 95 | 雷阵雨 |
| 96, 99 | 雷阵雨伴冰雹 | | |

## 回答方式

拿到数据后，不要原样输出 JSON。用中文自然语言回答，并给出实用建议：

1. 说明城市和查询时间
2. 当前天气：状况 + 气温 + 体感温度
3. 需要时补充湿度、风速、降雨概率
4. 未来预报：按天列出天气和最高/最低温
5. 给出穿衣/带伞建议：降雨概率 > 40% 建议带伞；气温 < 10℃ 建议厚外套；> 30℃ 提醒防暑补水

示例回答：

> 北京现在晴，气温 22℃，体感 21℃，湿度 45%，风速 8 km/h。未来三天以晴到多云为主，最高 24~26℃，最低 13~15℃，降雨概率低于 10%，不用带伞，早晚稍凉建议带件薄外套。

## 注意事项

- API 偶有超时，失败时重试一次；仍失败则说明服务暂时不可用
- 单位是摄氏度、km/h、毫米，除非用户要求不要换算成华氏度
- 数据来自 Open-Meteo 公开接口，仅供日常参考
- 城市名有大量重名时（如" Cambridge"），取 `results[0]` 并在回答中带上国家/地区名以消除歧义
