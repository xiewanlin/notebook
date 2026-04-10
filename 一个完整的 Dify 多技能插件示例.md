一个完整的 Dify 多技能插件示例，包含所有必要的配置文件和代码实现。

## 完整项目结构

```
weather_assistant/
├── manifest.yaml                 # 插件入口
├── main.py                       # 启动入口
├── requirements.txt              # 依赖
├── provider/
│   ├── __init__.py
│   ├── weather_provider.yaml     # 提供者配置，声明所有技能
│   └── weather_provider.py       # 提供者实现
└── tools/
    ├── __init__.py
    ├── current_weather.yaml      # 技能1：当前天气
    ├── current_weather.py        # 技能1实现
    ├── forecast.yaml             # 技能2：天气预报
    ├── forecast.py               # 技能2实现
    ├── air_quality.yaml          # 技能3：空气质量
    └── air_quality.py            # 技能3实现
```

## 1. 插件入口配置

### `manifest.yaml`

```yaml
name: weather_assistant
author: demo_user
version: 0.0.1
type: plugin

plugins:
  tools:
    - provider/weather_provider.yaml

meta:
  version: 0.0.1
  arch: [amd64, arm64]
  runner:
    language: python
    version: "3.12"
    entrypoint: main

resource:
  memory: 1048576
  permission:
    tool:
      enabled: true
    model:
      enabled: true
      llm: true
```

### `main.py`

```python
from dify_plugin import Plugin, DifyPluginEnv

if __name__ == "__main__":
    plugin = Plugin(DifyPluginEnv(MAX_REQUEST_TIMEOUT=120))
    plugin.run()
```

### `requirements.txt`

```
dify_plugin==0.0.1b72
requests>=2.31.0
```

## 2. 提供者配置

### `provider/weather_provider.yaml`

```yaml
identity:
  author: demo_user
  name: weather_provider
  label:
    en_US: Weather Assistant
    zh_CN: 天气助手
  description:
    en_US: Multi-functional weather service with current, forecast and air quality
    zh_CN: 多功能天气服务，包含当前天气、预报和空气质量
  icon: icon.svg
  tags: [weather, utility, ai]

credentials_for_provider:
  api_key:
    type: secret-input
    required: true
    label:
      en_US: OpenWeather API Key
      zh_CN: OpenWeather API密钥
    placeholder:
      en_US: Enter your OpenWeatherMap API key
      zh_CN: 请输入OpenWeatherMap API密钥
    help:
      en_US: Get API key from https://openweathermap.org/api
      zh_CN: 从 https://openweathermap.org/api 获取API密钥

  default_unit:
    type: select
    required: true
    default: metric
    options:
      - value: metric
        label:
          en_US: Celsius (°C)
          zh_CN: 摄氏度 (°C)
      - value: imperial
        label:
          en_US: Fahrenheit (°F)
          zh_CN: 华氏度 (°F)
      - value: standard
        label:
          en_US: Kelvin (K)
          zh_CN: 开尔文 (K)
    label:
      en_US: Temperature Unit
      zh_CN: 温度单位

# ★★★ 三个技能在此声明 ★★★
tools:
  - tools/current_weather.yaml
  - tools/forecast.yaml
  - tools/air_quality.yaml

extra:
  python:
    source: provider/weather_provider.py
```

### `provider/weather_provider.py`

```python
from typing import Any
from dify_plugin import ToolProvider
from dify_plugin.errors.tool import ToolProviderCredentialValidationError
import requests


class WeatherProvider(ToolProvider):
    def _validate_credentials(self, credentials: dict[str, Any]) -> None:
        """验证API密钥是否有效"""
        api_key = credentials.get("api_key", "")
        if not api_key:
            raise ToolProviderCredentialValidationError("API key is required")
        
        # 可选：发送测试请求验证密钥
        try:
            test_url = f"http://api.openweathermap.org/data/2.5/weather?q=London&appid={api_key}"
            response = requests.get(test_url, timeout=10)
            if response.status_code == 401:
                raise ToolProviderCredentialValidationError("Invalid API key")
        except requests.exceptions.RequestException:
            # 网络问题不阻断，仅记录
            pass
```

## 3. 技能1：当前天气查询

### `tools/current_weather.yaml`

```yaml
identity:
  name: current_weather
  author: demo_user
  label:
    en_US: Current Weather
    zh_CN: 当前天气
  icon: icon.svg

description:
  human:
    en_US: Get real-time weather information for a specific city
    zh_CN: 获取指定城市的实时天气信息
  llm: Use this tool to get current weather conditions including temperature, humidity, wind speed and weather description for a specific city. Use natural city names like "Beijing", "London", "New York".

parameters:
  - name: city
    type: string
    required: true
    label:
      en_US: City Name
      zh_CN: 城市名称
    human_description:
      en_US: Name of the city to query
      zh_CN: 要查询的城市名称
    llm_description: The city name to get weather for. Examples: "Beijing", "Shanghai", "Tokyo", "London", "New York"
    form: llm

  - name: include_details
    type: boolean
    required: false
    default: true
    label:
      en_US: Include Details
      zh_CN: 包含详细信息
    human_description:
      en_US: Whether to include humidity, wind speed and other details
      zh_CN: 是否包含湿度、风速等详细信息
    llm_description: Set to true to get detailed weather information including humidity, wind speed, pressure, etc.
    form: llm

extra:
  python:
    source: tools/current_weather.py
```

### `tools/current_weather.py`

```python
from collections.abc import Generator
from dify_plugin import Tool
from dify_plugin.entities.tool import ToolInvokeMessage
import requests


class CurrentWeatherTool(Tool):
    def _invoke(self, tool_parameters: dict) -> Generator[ToolInvokeMessage]:
        # 安全获取参数
        city = tool_parameters.get("city", "").strip()
        include_details = tool_parameters.get("include_details", True)
        
        if not city:
            yield self.create_text_message("Error: City name is required")
            return
        
        # 获取认证信息
        api_key = self.runtime.credentials.get("api_key")
        unit = self.runtime.credentials.get("default_unit", "metric")
        
        # 单位映射
        unit_symbols = {
            "metric": "°C",
            "imperial": "°F",
            "standard": "K"
        }
        symbol = unit_symbols.get(unit, "°C")
        
        try:
            # 调用天气API
            url = "http://api.openweathermap.org/data/2.5/weather"
            params = {
                "q": city,
                "appid": api_key,
                "units": unit,
                "lang": "zh_cn"  # 返回中文描述
            }
            
            response = requests.get(url, params=params, timeout=10)
            data = response.json()
            
            if response.status_code != 200:
                error_msg = data.get("message", "Unknown error")
                yield self.create_text_message(f"Failed to get weather: {error_msg}")
                return
            
            # 解析数据
            weather_desc = data["weather"][0]["description"]
            temp = data["main"]["temp"]
            feels_like = data["main"]["feels_like"]
            
            # 构建回复
            result = f"🌤️ {city} 当前天气\n\n"
            result += f"天气状况: {weather_desc}\n"
            result += f"温度: {temp}{symbol} (体感 {feels_like}{symbol})"
            
            if include_details:
                humidity = data["main"]["humidity"]
                wind_speed = data["wind"]["speed"]
                pressure = data["main"]["pressure"]
                
                result += f"\n湿度: {humidity}%\n"
                result += f"风速: {wind_speed} m/s\n"
                result += f"气压: {pressure} hPa"
            
            # 返回文本消息
            yield self.create_text_message(result)
            
            # 同时返回结构化JSON
            yield self.create_json_message({
                "city": city,
                "weather": weather_desc,
                "temperature": temp,
                "feels_like": feels_like,
                "unit": unit,
                "humidity": data["main"].get("humidity"),
                "wind_speed": data["wind"].get("speed"),
                "timestamp": data.get("dt")
            })
            
            # 保存到变量供后续节点使用
            yield self.create_variable_message("weather_temp", temp)
            
        except requests.exceptions.Timeout:
            yield self.create_text_message("Error: Request timeout, please try again")
        except Exception as e:
            yield self.create_text_message(f"Error: {str(e)}")
```

## 4. 技能2：天气预报

### `tools/forecast.yaml`

```yaml
identity:
  name: weather_forecast
  author: demo_user
  label:
    en_US: Weather Forecast
    zh_CN: 天气预报
  icon: icon.svg

description:
  human:
    en_US: Get 5-day weather forecast for a specific city
    zh_CN: 获取指定城市未来5天天气预报
  llm: Use this tool to get 5-day weather forecast with 3-hour intervals for a specific city. Good for planning activities or travel.

parameters:
  - name: city
    type: string
    required: true
    label:
      en_US: City Name
      zh_CN: 城市名称
    human_description:
      en_US: Name of the city to query
      zh_CN: 要查询的城市名称
    llm_description: The city name to get forecast for
    form: llm

  - name: days
    type: number
    required: false
    default: 3
    min: 1
    max: 5
    label:
      en_US: Number of Days
      zh_CN: 天数
    human_description:
      en_US: How many days of forecast to return (1-5)
      zh_CN: 返回多少天的预报（1-5天）
    llm_description: Number of days to forecast (1 to 5). Default is 3 days.
    form: llm

extra:
  python:
    source: tools/forecast.py
```

### `tools/forecast.py`

```python
from collections.abc import Generator
from dify_plugin import Tool
from dify_plugin.entities.tool import ToolInvokeMessage
import requests
from datetime import datetime


class WeatherForecastTool(Tool):
    def _invoke(self, tool_parameters: dict) -> Generator[ToolInvokeMessage]:
        city = tool_parameters.get("city", "").strip()
        days = int(tool_parameters.get("days", 3))
        
        if not city:
            yield self.create_text_message("Error: City name is required")
            return
        
        api_key = self.runtime.credentials.get("api_key")
        unit = self.runtime.credentials.get("default_unit", "metric")
        
        unit_symbols = {"metric": "°C", "imperial": "°F", "standard": "K"}
        symbol = unit_symbols.get(unit, "°C")
        
        try:
            url = "http://api.openweathermap.org/data/2.5/forecast"
            params = {
                "q": city,
                "appid": api_key,
                "units": unit,
                "lang": "zh_cn"
            }
            
            response = requests.get(url, params=params, timeout=10)
            data = response.json()
            
            if response.status_code != 200:
                yield self.create_text_message(f"Failed: {data.get('message', 'Unknown error')}")
                return
            
            # 按天分组（每3小时一个数据点，每天8个）
            daily_data = {}
            for item in data["list"]:
                date = item["dt_txt"].split(" ")[0]
                if date not in daily_data:
                    daily_data[date] = []
                daily_data[date].append(item)
            
            # 取前N天
            dates = sorted(daily_data.keys())[:days]
            
            result = f"📅 {city} {days}天天气预报\n\n"
            forecast_list = []
            
            for date in dates:
                day_items = daily_data[date]
                temps = [i["main"]["temp"] for i in day_items]
                max_temp = max(temps)
                min_temp = min(temps)
                
                # 取中午的天气描述作为当天代表
                noon_item = day_items[len(day_items)//2]
                weather = noon_item["weather"][0]["description"]
                
                weekday = datetime.strptime(date, "%Y-%m-%d").strftime("%m月%d日")
                
                result += f"{weekday}: {weather}, {min_temp:.1f}{symbol} ~ {max_temp:.1f}{symbol}\n"
                
                forecast_list.append({
                    "date": date,
                    "weather": weather,
                    "max_temp": max_temp,
                    "min_temp": min_temp,
                    "symbol": symbol
                })
            
            yield self.create_text_message(result)
            yield self.create_json_message({
                "city": city,
                "forecast": forecast_list,
                "unit": unit
            })
            
        except Exception as e:
            yield self.create_text_message(f"Error: {str(e)}")
```

## 5. 技能3：空气质量查询

### `tools/air_quality.yaml`

```yaml
identity:
  name: air_quality
  author: demo_user
  label:
    en_US: Air Quality
    zh_CN: 空气质量
  icon: icon.svg

description:
  human:
    en_US: Check air quality index (AQI) and pollutant levels
    zh_CN: 查询空气质量指数(AQI)和污染物水平
  llm: Use this tool to check air quality index, PM2.5, PM10, and other pollutant levels for a specific city. Useful for health recommendations.

parameters:
  - name: city
    type: string
    required: true
    label:
      en_US: City Name
      zh_CN: 城市名称
    human_description:
      en_US: Name of the city to query
      zh_CN: 要查询的城市名称
    llm_description: The city name to check air quality for
    form: llm

extra:
  python:
    source: tools/air_quality.py
```

### `tools/air_quality.py`

```python
from collections.abc import Generator
from dify_plugin import Tool
from dify_plugin.entities.tool import ToolInvokeMessage
import requests


class AirQualityTool(Tool):
    def _invoke(self, tool_parameters: dict) -> Generator[ToolInvokeMessage]:
        city = tool_parameters.get("city", "").strip()
        
        if not city:
            yield self.create_text_message("Error: City name is required")
            return
        
        api_key = self.runtime.credentials.get("api_key")
        
        try:
            # 先获取城市坐标
            geo_url = "http://api.openweathermap.org/geo/1.0/direct"
            geo_params = {"q": city, "limit": 1, "appid": api_key}
            geo_response = requests.get(geo_url, params=geo_params, timeout=10)
            geo_data = geo_response.json()
            
            if not geo_data:
                yield self.create_text_message(f"City '{city}' not found")
                return
            
            lat, lon = geo_data[0]["lat"], geo_data[0]["lon"]
            
            # 获取空气质量数据
            aqi_url = "http://api.openweathermap.org/data/2.5/air_pollution"
            aqi_params = {"lat": lat, "lon": lon, "appid": api_key}
            aqi_response = requests.get(aqi_url, params=aqi_params, timeout=10)
            aqi_data = aqi_response.json()
            
            aqi = aqi_data["list"][0]["main"]["aqi"]
            components = aqi_data["list"][0]["components"]
            
            # AQI等级说明
            aqi_levels = {
                1: ("优", "🟢", "空气质量令人满意，基本无空气污染"),
                2: ("良", "🟡", "空气质量可接受，但某些污染物可能对极少数异常敏感人群健康有较弱影响"),
                3: ("轻度污染", "🟠", "易感人群症状有轻度加剧，健康人群出现刺激症状"),
                4: ("中度污染", "🔴", "进一步加剧易感人群症状，可能对健康人群心脏、呼吸系统有影响"),
                5: ("重度污染", "🟣", "心脏病和肺病患者症状显著加剧，运动耐受力降低，健康人群普遍出现症状")
            }
            
            level, emoji, advice = aqi_levels.get(aqi, ("未知", "⚪", "暂无建议"))
            
            result = f"🌫️ {city} 空气质量\n\n"
            result += f"AQI指数: {aqi} {emoji} ({level})\n"
            result += f"建议: {advice}\n\n"
            result += "主要污染物 (μg/m³):\n"
            result += f"  • PM2.5: {components.get('pm2_5', 'N/A')}\n"
            result += f"  • PM10: {components.get('pm10', 'N/A')}\n"
            result += f"  • NO₂: {components.get('no2', 'N/A')}\n"
            result += f"  • O₃: {components.get('o3', 'N/A')}"
            
            yield self.create_text_message(result)
            yield self.create_json_message({
                "city": city,
                "aqi": aqi,
                "level": level,
                "pm25": components.get("pm2_5"),
                "pm10": components.get("pm10"),
                "no2": components.get("no2"),
                "o3": components.get("o3"),
                "advice": advice
            })
            
        except Exception as e:
            yield self.create_text_message(f"Error: {str(e)}")
```

## 6. 测试与打包

### 本地测试配置 `.env`

```bash
INSTALL_METHOD=remote
REMOTE_INSTALL_URL=debug.dify.ai:5003
REMOTE_INSTALL_KEY=your-debug-key-here
```

### 打包命令

```bash
# 安装 CLI
pip install dify-plugin-daemon

# 打包
cd weather_assistant
dify-plugin plugin package

# 输出: weather_assistant.difypkg
```

## 使用示例

在 Dify 工作流中，你可以这样使用这些技能：

1. **LLM 节点** 会自动根据用户输入选择合适的技能：
   - "北京今天天气怎么样？" → 触发 `current_weather`
   - "上海未来三天天气如何？" → 触发 `weather_forecast`
   - "广州空气质量怎么样？" → 触发 `air_quality`

2. **工具节点** 可以手动选择特定技能，并通过 `{{#node_id.variable#}}` 引用其他节点的输出。

这个示例展示了如何组织多个相关技能，共享认证配置，同时保持每个技能的独立性和可维护性。