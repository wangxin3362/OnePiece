#!/usr/bin/env python
#_*_ coding:utf-8 _*_
#author:Jacky

from selenium.common.exceptions import TimeoutException
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait
from selenium import webdriver
from bs4 import BeautifulSoup
import lxml.html
import pymongo,re
import sys
reload(sys)
sys.setdefaultencoding('utf8')

MONGO_URL='192.168.171.19'
MONGO_DB='amazon'
MONGO_TABLE='amazon'
SERVICE_AGES=['--load-images=false','--disk-cache=true']
KEYWORD='python'        #搜索关键字
client=pymongo.MongoClient(MONGO_URL)      #这里没写端口，就用默认的27017
db=client[MONGO_DB]
browser=webdriver.PhantomJS(executable_path=r"D:\software\dev\phantomjs-2.1.1-windows\bin\phantomjs.exe")    #
#PhantomJS 是一个可编程的无头浏览器,诸如网络监测、网页截屏、无需浏览器的 Web 测试、页面访问自动化
# browser=webdriver.PhantomJS(service_args=SERVICE_AGES)
wait=WebDriverWait(browser,10)
browser.set_window_size(1400,900)

def search():
    print('searching...')
    try:
        browser.get('https://www.amazon.cn/')
        input=wait.until(
            EC.presence_of_element_located((By.CSS_SELECTOR,'#twotabsearchtextbox'))
        )
        print('input',input)
        submit=wait.until(
            EC.element_to_be_clickable((By.CSS_SELECTOR,'#nav-search > form > div.nav-right > div > input'))
        )
        print('submit',submit)
        input.send_keys(KEYWORD)
        submit.click()
        total = wait.until(
            EC.presence_of_element_located((By.CSS_SELECTOR,'#pagn > span.pagnDisabled'))
        )
        print('total========',total)
        get_products()
        print('total:'+total.text+'page')
        return total.text
    except TimeoutException:
        return search()

def next_page(number):
    '''
    获取下一页的内容
    :param number:
    :return:
    '''
    try_times = 0
    print('page turning...',number)
    try:
        #wait.until(EC.text_to_be_present_in_element((By.CSS_SELECTOR,'#pagnNextString'),'下一页'))
        wait.until(EC.text_to_be_present_in_element(
            (By.CSS_SELECTOR, '#pagnNextString'), '下一页'))
        wait.until(EC.text_to_be_present_in_element((By.CSS_SELECTOR,'#pagnNextString'),'下一页'))
        submit=wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR,'#pagnNextString')))
        submit.click()
        wait.until(EC.text_to_be_present_in_element((By.CSS_SELECTOR,'.pagnCur'),str(number)))
        get_products()
        print ('--------------')
    except TimeoutException:
        print('try_times',try_times)
        next_page(number)



def get_products():
    '''
    获取商品名称、图片、价格、出版日期
    :return:
    '''
    try:
        wait.until(EC.presence_of_element_located((By.CSS_SELECTOR,'#s-results-list-atf')))
        html = browser.page_source
        soup= BeautifulSoup(html,'lxml')
        doc=lxml.html.fromstring(html)
        date = doc.xpath('//li[@class="s-result-item s-result-card-for-container-noborder s-carded-grid celwidget  "]/div/div[3]/div[1]/span[2]/text()')
        #在使用xpath查找的时候div和span的下标从1开始
        content = soup.find_all(attrs={"id": re.compile(r'result_\d+')})
        for item,time in zip(content,date):
            product={
                'title': item.find(class_='s-access-title').get_text(),
                'image': item.find(class_='s-access-image cfMarker').get('src'),
                'price': item.find(class_='a-size-base a-color-price s-price a-text-bold').get_text(),
                'date':  time
            }
            #save_to_mongo(product)
            print product['title'],product['date']
    except Exception as e:
        print(e)

def save_to_mongo(result):
    try:
        if db[MONGO_DB].insert(result):
            print('store to mongo successfully',result)
    except Exception:
        print('failed to store to mongo',result)



def main():
    try:
        total=int(search())
        for i in range(2,total+1):
           next_page(i)
    except Exception as e:
        print('ERROR',e)
    finally:
        browser.close()


total=int(search())
for i in range(2,total+1):
    next_page(i)

if __name__=='__main__':
    main()
