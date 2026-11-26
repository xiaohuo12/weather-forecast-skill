# weather-forecast Skill

一个给 AI 助手用的天气查询技能（WorkBuddy / Claude 等支持 Skill 的客户端通用）。

基于 [Open-Meteo](https://open-meteo.com/) 免费公共 API：**无需注册、无需 API Key、无调用费用**。支持全球城市，中文名、拼音、英文名均可查询。

## 它做什么

当用户问"今天天气""明天会下雨吗""北京气温多少""周末要带伞吗"这类问题时，AI 会自动加载这个技能，按里面的流程：

1. 用 Open-Meteo Geocoding API 把城市名解析成经纬度
2. 用 Forecast API 拉取当前天气和未来预报
3. 把 WMO 天气代码翻译成中文，给出穿衣/带伞建议

## 安装

把整个目录放进你的 skills 目录：

```bash
# WorkBuddy 用户级
git clone https://github.com/xiaohuo12/weather-forecast-skill.git ~/.workbuddy/skills/weather-forecast

# WorkBuddy 项目级
git clone https://github.com/xiaohuo12/weather-forecast-skill.git .workbuddy/skills/weather-forecast
```

Windows PowerShell：

```powershell
git clone https://github.com/xiaohuo12/weather-forecast-skill.git "$env:USERPROFILE\.workbuddy\skills\weather-forecast"
```

安装后重启客户端，让技能列表刷新。

## 文件结构

```
weather-forecast-skill/
├── SKILL.md      # 技能本体：触发条件 + 查询流程 + 天气代码对照表
├── README.md     # 本说明
└── LICENSE       # MIT
```

## 直接试用接口

```bash
# 1. 城市 -> 经纬度
curl -s "https://geocoding-api.open-meteo.com/v1/search?name=北京&count=1&language=zh&format=json"

# 2. 经纬度 -> 天气（把 lat/lon 换成上一步的值）
curl -s "https://api.open-meteo.com/v1/forecast?latitude=39.9075&longitude=116.39723&current=temperature_2m,apparent_temperature,relative_humidity_2m,weather_code,wind_speed_10m&daily=weather_code,temperature_2m_max,temperature_2m_min,precipitation_probability_max&timezone=Asia%2FShanghai&forecast_days=3"
```

## 为什么做成 Skill 而不是 MCP

Skill 是一份"给 AI 看的操作说明书"，不需要运行服务进程、不需要装依赖，拷进目录就能用；天气查询只是两条 HTTP 请求，做成 Skill 最轻量。

## 数据来源

[Open-Meteo](https://open-meteo.com/) 开放天气 API，CC BY 4.0 数据许可。仅供日常参考，极端天气请以当地气象部门为准。

## 许可证

MIT — 见 [LICENSE](LICENSE)。
