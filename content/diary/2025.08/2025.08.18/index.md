+++
date = '2025-08-18T10:00:00+08:00'
draft = true
title = '2025.08.18'
+++
<!--more-->
工作：
- owl适配google search工具
  - 先去测下账号有什么权限


- owl并发调用
  - 

在 Python 中，有三种主流的并发模型：
- 多进程 (multiprocessing): 适合 CPU 密集型任务（如大量数学计算）。它会创建多个独立的进程，开销最大，但不共享内存（GIL不是问题）。
- 多线程 (threading): 适合 I/O 密集型任务。它在同一进程中创建多个线程，但由于全局解释器锁（GIL），在同一时刻只有一个线程能执行 Python 字节码。对于等待网络响应这类任务，GIL 会被释放，因此多线程依然有效。
- 异步 (asyncio): I/O 密集型任务的最佳选择。它在单线程内通过事件循环实现并发，开销极小。当一个任务（如 API 调用）在等待时，程序会立即切换到另一个准备好的任务，最大化利用等待时间。



```python
    @api_keys_required(
  [
    (None, 'GOOGLE_API_KEY'),
    (None, 'SEARCH_ENGINE_ID'),
  ]
)
def search_google(
        self, query: str, num_result_pages: int = 5
) -> List[Dict[str, Any]]:
  r"""Use Google search engine to search information for the given query.

  Args:
      query (str): The query to be searched.
      num_result_pages (int): The number of result pages to retrieve.

  Returns:
      List[Dict[str, Any]]: A list of dictionaries where each dictionary
      represents a website.
          Each dictionary contains the following keys:
          - 'result_id': A number in order.
          - 'title': The title of the website.
          - 'description': A brief description of the website.
          - 'long_description': More detail of the website.
          - 'url': The URL of the website.

          Example:
          {
              'result_id': 1,
              'title': 'OpenAI',
              'description': 'An organization focused on ensuring that
              artificial general intelligence benefits all of humanity.',
              'long_description': 'OpenAI is a non-profit artificial
              intelligence research company. Our goal is to advance
              digital intelligence in the way that is most likely to
              benefit humanity as a whole',
              'url': 'https://www.openai.com'
          }
      title, description, url of a website.
  """
  import requests

  # https://developers.google.com/custom-search/v1/overview
  GOOGLE_API_KEY = os.getenv("GOOGLE_API_KEY")
  # https://cse.google.com/cse/all
  SEARCH_ENGINE_ID = os.getenv("SEARCH_ENGINE_ID")

  # Using the first page
  start_page_idx = 1
  # Different language may get different result
  search_language = "en"
  # How many pages to return
  num_result_pages = num_result_pages
  # Constructing the URL
  # Doc: https://developers.google.com/custom-search/v1/using_rest
  url = (
    f"https://www.googleapis.com/customsearch/v1?"
    f"key={GOOGLE_API_KEY}&cx={SEARCH_ENGINE_ID}&q={query}&start="
    f"{start_page_idx}&lr={search_language}&num={num_result_pages}"
  )

  responses = []
  # Fetch the results given the URL
  try:
    # Make the get
    result = requests.get(url)
    data = result.json()

    # Get the result items
    if "items" in data:
      search_items = data.get("items")

      # Iterate over 10 results found
      for i, search_item in enumerate(search_items, start=1):
        # Check metatags are present
        if "pagemap" not in search_item:
          continue
        if "metatags" not in search_item["pagemap"]:
          continue
        if (
                "og:description"
                in search_item["pagemap"]["metatags"][0]
        ):
          long_description = search_item["pagemap"]["metatags"][
            0
          ]["og:description"]
        else:
          long_description = "N/A"
        # Get the page title
        title = search_item.get("title")
        # Page snippet
        snippet = search_item.get("snippet")

        # Extract the page url
        link = search_item.get("link")
        response = {
          "result_id": i,
          "title": title,
          "description": snippet,
          "long_description": long_description,
          "url": link,
        }
        # skip huggingface GAIA results
        if "huggingface" in link.lower() and "gaia" in link.lower():
          continue
        # skip GAIA paper results
        if "2311.12983" in link.lower():
          continue
        # skip GAIA benchmark results
        if "gaia" in snippet.lower() and "benchmark" in snippet.lower():
          continue
        responses.append(response)
    else:
      responses.append({"error": "google search failed."})

  except requests.RequestException:
    # Handle specific exceptions or general request exceptions
    responses.append({"error": "google search failed."})
  # If no answer found, return an empty list
  return responses


```

提升：
- 高阶05
- 功能性瑜伽
  - 老师夸我肌肉感觉很好，让我一周去练三次器械
  - 吃饭吃7分饱

其他：

