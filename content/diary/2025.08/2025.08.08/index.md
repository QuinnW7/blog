+++
date = '2025-08-08T15:54:00+08:00'
draft = true
title = '2025.08.08'
+++

<!--more-->

工作：
- owl
  - 看上去和manus有点像，看看区别
- 云函数开发
  - 现在是可以调用了云函数了，直接配上试试

提升：
- 高阶三阶段1
  - 周末要进行选择！
- xhs fire day1发布
- 晚上游泳

其他：


```bash
python3 -m venv venv       
source venv/bin/activate      

cd ..
rm -rf package_for_scf deployment_package.zip 
mkdir package_for_scf

pip install -r requirements.txt -t package_for_scf/

cp app.py package_for_scf/
cp scf_bootstrap package_for_scf
cp config.py package_for_scf/

cd package_for_scf
zip -r ../deployment_package.zip .

```

```python

def call_hunyuan_workflow(user_content: str, workflow_id: str, api_key: str):

url = "http://hunyuanapi.woa.com/openapi/v1/workflow/chat/completions/id"

headers = {
    'Authorization': f'Bearer {api_key}',
    'Content-Type': 'application/json'
}

payload = {
    "workflow_id": workflow_id,
    "version": "1",
    "messages": [
        {
            "role": "user",
            "content": user_content
        }
    ]
}

try:
    response = requests.post(url, headers=headers, json=payload, timeout=30)
    response.raise_for_status() # 对 4xx 或 5xx 响应引发 HTTPError
    return response.json()

except requests.exceptions.HTTPError as http_err:
    print(f"HTTP 错误: {http_err} - 响应内容: {response.text if 'response' in locals() else 'N/A'}")
    return {"error": f"HTTP 错误: {http_err}", "details": response.text if 'response' in locals() else 'N/A'}
except requests.exceptions.ConnectionError as conn_err:
    print(f"连接错误: {conn_err}")
    return {"error": f"连接错误: {conn_err}"}
except requests.exceptions.Timeout as timeout_err:
    print(f"请求超时: {timeout_err}")
    return {"error": f"请求超时: {timeout_err}"}
except requests.exceptions.RequestException as req_err:
    print(f"调用 API 时发生未知错误: {req_err}")
    return {"error": f"未知请求错误: {req_err}"}
except json.JSONDecodeError as json_err:
    print(f"JSON 解析错误: {json_err} - 响应内容: {response.text if 'response' in locals() else 'N/A'}")
    return {"error": f"JSON 解析错误: {json_err}", "details": response.text if 'response' in locals() else 'N/A'}

```