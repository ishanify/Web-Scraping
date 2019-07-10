
# Web scraping using Python, Selenium (navigating javascript dynamic pages & links)
[Click here for Readme and read more details about the process](/README.md)
```python

from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
from selenium.common.exceptions import NoSuchElementException

from bs4 import BeautifulSoup
import re
import pandas as pd
import os
import time
```
```python
# create a new Firefox session
driver = webdriver.Firefox()
driver.implicitly_wait(30)
```

```python
# Initialize variables, counters and file names
base_url='https://www.securities-administrators.ca/nrs/nrsIndvSearchResults.aspx?mode=AS&type=I&indv=&firm=&juri=1&ctgy=1&history=0'

#  full path of folder. Multiple csv will be created. Better to give path of empty folder
__filePath = 'folder1/folder2/'
__fileName = 'scrap'
__extension = '.csv'
pageCounter = 0
fileCounter = 0

```
```python
driver.get(base_url)
time.sleep(2)
# making 100 rows on page to reduce clicks
driver.find_element_by_xpath("//select[@name='ctl00$bodyContent$list_num_per_page']/option[text()='100']").click()

```
```python
while True:
    count=0  
    df = pd.DataFrame(columns=['Name','Firm','Address'])

    #Create a new file after scraping some pages...
    if pageCounter == 3 or pageCounter==0:
        fileCounter= fileCounter + 1
        saveAs= __filePath+__fileName+str(fileCounter)+__extension
        pageCounter=0
        # DF
        #Initialize a dataframe to store each page records. Then write it to file. Repeat this for each page.
        df.to_csv(saveAs, index=False, mode='w')

    time.sleep(2)
    links = [link.get_attribute('href') for link in driver.find_elements_by_xpath("//table[@class='gridview_style']/tbody/tr/td/a")]
    
    #Loop to get all records from current page
    for link in links:
        
        # Entering into the page
        driver.get(link)
        # wait for browser to load page
        time.sleep(2)
        
        myElem = WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.ID, 'ctl00_bodyContent_divIndividualDetailsAccordion')))
        
        name = driver.find_elements_by_id('ctl00_bodyContent_lblIndvName')[0].text
        firm = driver.find_elements_by_xpath("//div[@class='current-content']/table/tbody/tr[1]/td[2]")[0].text
        addr = driver.find_elements_by_xpath("//div[@class='current-content']/table/tbody/tr[2]/td[2]")[0].text
        
        # append the record to data frame
        d = {'Name':name,'Firm':firm,'Address':addr}
        df = df.append(d, ignore_index=True)
        
    #Writing the records of page in local file
    df.to_csv(saveAs, index=False, mode='a',header=False)
    
    # Completed scraping current page and increasing page counter
    pageCounter= pageCounter + 1

    # Going back to search results
    driver.find_elements_by_xpath("//a[@id='A1']")[0].click() 
    time.sleep(2)
    
    #Clicking Next and navigating pages
    try:
        nextlink = driver.find_element_by_xpath('//*[@id="ctl00_bodyContent_lbtnNext"]').get_attribute('href')
        driver.get(nextlink)
        time.sleep(2)
    
    #stopping when there is no next page
    except NoSuchElementException:
        break
```


