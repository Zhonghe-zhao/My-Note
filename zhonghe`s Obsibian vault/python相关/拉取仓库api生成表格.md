



 
```D:\python\python.exe main.py


~~~python
import requests  
import datetime  
  
# GitHub用户名  
GITHUB_USERNAME = 'Whuichenggong'  
# GitHub个人访问令牌（可选）  
GITHUB_TOKEN = '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'  # 可替换为你的 GitHub token  
# GitHub API URL  
API_URL = f'https://api.github.com/users/{GITHUB_USERNAME}/repos'  
  
# 设置请求头，包含 token（如果需要）  
headers = {'Authorization': f'token {GITHUB_TOKEN}'} if GITHUB_TOKEN else {}  
  
# 发起请求获取仓库列表  
response = requests.get(API_URL, headers=headers)  
  
# 检查请求是否成功  
if response.status_code == 200:  
    repos = response.json()  
  
    # 输出 Markdown 表头  
    print(  
        '| ID   | REPO                                                         | START      | UPDATE     | LANGUAGE | STARS |')  
    print(  
        '| ---- | ------------------------------------------------------------ | ---------- | ---------- | -------- | ----- |')  
  
    for idx, repo in enumerate(repos, 1):  
        repo_name = repo['name']  
        repo_url = repo['html_url']  
        start_date = repo['created_at'][:10]  # 取创建日期的前10位  
        update_date = repo['updated_at'][:10]  # 取更新时间的前10位  
        language = repo.get('language', 'N/A')  # 获取语言，若没有则返回 'N/A'        stars = repo['stargazers_count']  # 获取 Stars 数量  
  
        # 输出 Markdown 格式的表格行  
        print(f'| {idx}    | [{repo_name}]({repo_url}) | {start_date} | {update_date} | {language} | {stars}  |')  
  
else:  
    print(f'Error: {response.status_code}')


