### selenium

- 声明浏览器对象

  browser = webdriver.Chrome()

- 关闭浏览器

​       browser.close()

- 获取cookie

  browser.get_cookies()

- 获取页面内容

  browser.page_source

- 查找元素

  browser.find_element_by_id('q')

  browser.find_element_by_css_selector('#q')

  browser.find_element_by_xpath('//*[@id="q"]')

- 获取元素属性

  from selenium import webdriver

  from selenium.webdriver import ActionChains

  browser = webdriver.Chrome()

  url = ''

  browser.get(url)

  logo = browser.find_element_by_id('zh-top-link-logo')

  print(logo)

  pritnt''



https://bj.zu.anjuke.com/fangyuan/1174272197?from=Filter_19&hfilter=filterlist



https://bj.zu.anjuke.com/v3/ajax/getBrokerPhone/?broker_id=5181158&token=886494b441bd12fa28275af04ef467b0&prop_id=1174272197&prop_city_id=14&house_type=1&captcha=



https://bj.zu.anjuke.com/v3/ajax/getBrokerPhone/?broker_id=1004169&token=861e32f80b804c8e8ad6c06f7f3c74c2&prop_id=1171825906&prop_city_id=14&house_type=1&captcha=