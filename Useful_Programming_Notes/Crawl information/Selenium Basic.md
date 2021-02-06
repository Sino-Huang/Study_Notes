# Selenium Basic 

```python
from selenium import webdriver
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By

# open browser
browser = webdriver.Firefox()
# get url into the browser
browser.get(level_video_url)

# you can find element by using XPATH, noted that the page dynamically loads, thus if you do not wait for it,
# you may not get full contents
video_as = browser.find_elements(By.XPATH, f'''//a[contains(@class, 'video')]''')

video_count = len(video_as)

print(f'Video Count is {video_count}')

for video_a in video_as:
    # execute click action
    browser.execute_script("arguments[0].click();", video_a)
    try:
        # variable is Boolean type when triggered
        # this is to let Webdriver to wait until the condition become true
        WebDriverWait(browser, 20, 0.5).until(lambda broswer: len(browser.find_elements(By.XPATH, f"//iframe[contains(@src, 'embed')]")) == video_count)
        youtube_url = browser.find_elements(By.XPATH, f'''//iframe[contains(@src, 'embed')]''')
        youtube_url = [x.get_attribute('src') for x in youtube_url]
        level_video_url = youtube_url
        print(f'TOP A VIDEO Url is {level_video_url}, page url is {response.url}')
        finally:
            pass
```

