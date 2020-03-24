# Project Title

Case_Study1: DNA_OPS_Psomagen

## Getting Started

These instructions will get you running on your local machine for development and testing purposes. 

## Imports
```
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from bs4 import BeautifulSoup
import urllib.request
import re
import os
import numpy as np
import pandas as pd
import time
from selenium.common.exceptions import ElementClickInterceptedException
import time
```

## Installing

A step by step series of examples that tell you how to get a development env running

Say what the step will be


### Pandas configuration
```
pd.set_option('display.max_rows', 1000) # max number of rows to display
start_time = time.time()
```
### Start the chromedriver
```
path_to_chromedriver ='/Users/yeryoungkim/Desktop/chromedriver' 

browser = webdriver.Chrome(executable_path = path_to_chromedriver)
```
### Set the url
```
url = 'http://macrogenusa.com/admin/'
browser.get(url)
```

### Find the 'user_id' element by name
```
browser.find_element_by_name('user_id')
browser.find_element_by_name('user_id').clear() # Make sure that the form is empty
browser.find_element_by_name('user_id').send_keys('yrkim') # writing ID 
browser.find_element_by_name('passwd') # find the 'passwd' element by name
browser.find_element_by_name('passwd').clear()# Make sure that the form is empty
browser.find_element_by_name('passwd').send_keys('macrogen0605')# writing password
browser.find_element_by_xpath('/html/body/div/div/form/button').click()# Searching the login button's xpath and click it.
```
### Switch to frame in menu
```
browser.switch_to_frame('menu')
browser.find_element_by_xpath('//*[@id="leftmenu"]/ul/li[4]/ul/li[15]/a').click()# click the quantify button
browser.switch_to_default_content()
browser.switch_to_frame('main')
```

## Web scraping - read the 'Quantify' page table.
```
soup = BeautifulSoup(browser.page_source)
table = soup.find_all('table')[4]
list_df = pd.read_html(str(table))
df = list_df[0]
df = df.dropna(thresh=4) #Remove missing values.
print('Web scraping finished!')
print(list_df)
print("--- %s seconds ---" % (time.time() - start_time))
```

### Get all checkboxes in the webpage
```
len_table = len(df)
list_checkboxes = []
list_checkboxes.append(browser.find_element_by_xpath('/html/body/table/tbody/tr[3]/td/table/tbody/tr[2]/td[9]/input')) # for first row
for i in range(len_table-2):
    list_checkboxes.append(browser.find_element_by_xpath('/html/body/table/tbody/tr[3]/td/table/tbody/tr[%d]/td[7]/input'%(i+3)))
 ```
### Get gugan checkbox
 
```
gugan_checkbox = browser.find_element_by_xpath('//*[@id="gugan_menu"]/table/tbody/tr[1]/td[2]/input')  
print('getting Checkbox finished!')
print("--- %s seconds ---" % (time.time() - start_time))
```
### Setting up universal primers ...
```
arr_index_EB3 = df.loc[df[4] == 'EB3', 0].to_numpy().astype(int)
arr_index_IntFw = df.loc[df[4] == 'IntFw', 0].to_numpy().astype(int)
arr_index_KanFw = df.loc[df[4] == 'KanFw', 0].to_numpy().astype(int)
arr_index_KanRv = df.loc[df[4] == 'KanRv', 0].to_numpy().astype(int)
arr_index_KanRv2 = df.loc[df[4] == 'KanRv2', 0].to_numpy().astype(int)
arr_index_IntFw2 = df.loc[df[4] == 'IntFw2', 0].to_numpy().astype(int)
arr_index_Kan_Del_Fw = df.loc[df[4] == 'Kan_Del_Fw', 0].to_numpy().astype(int)
arr_index_KanFw2 = df.loc[df[4] == 'KanFw2', 0].to_numpy().astype(int)
arr_index_EPORv = df.loc[df[4] == 'EPORv', 0].to_numpy().astype(int)
arr_index_EPOFw = df.loc[df[4] == 'EPOFw', 0].to_numpy().astype(int)
```
### Create list of universal primers name
```
list_univ = [arr_index_EB3, arr_index_EPOFw, arr_index_EPORv, 
            arr_index_IntFw, arr_index_IntFw2, arr_index_Kan_Del_Fw,
            arr_index_KanFw, arr_index_KanFw2, arr_index_KanRv,arr_index_KanRv2]
  ```

### Create list of General primers well numbers
```
list_alphabet = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H']
list_number = ['1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12']

list_well_num = []
for number in list_number:
    for alphabet in list_alphabet:
        list_well_num.append(r'\(' + alphabet + number + r'\)')
  ```

### Setting up General primers list
 ```
general_primers = df.loc[df[4].str.contains('Sequencing'), 0].to_numpy().astype(int)

print('list_univ: ' + str(list_univ))
print('list_well_num: ' + str(list_well_num))
print('general_primers: ' + str(general_primers))
def check_univ_primers(arr_index):
```
### Click Universal primers corresponding elements...\
```
    bln_pass = False
    quotient_48 = int(len(arr_index)/48)
    quotient_32 = int(len(arr_index)/32)
    quotient_16 = int(len(arr_index)/16)
    quotient_8 = int(len(arr_index)/8)
    print(arr_index)
    if quotient_48 > 0:
        list_checkboxes[arr_index[0]-1].click()
        gugan_checkbox.click()
        list_checkboxes[arr_index[(quotient_48-1)*48+47]-1].click()
        gugan_checkbox.click()
        bln_pass = True       
        print('48 checked')
```
### At this point, there is no arr_index which has elements more than 32
```
    elif (quotient_32 > 0):
    
        list_checkboxes[arr_index[0]-1].click()
        gugan_checkbox.click()
        list_checkboxes[arr_index[(quotient_32-1)*32+31]-1].click()
        gugan_checkbox.click()
        bln_pass = True   
        print('32 checked')
```
### At this point, there is no arr_index which has elements more than 16
```
    elif (quotient_16 > 0):
        list_checkboxes[arr_index[0]-1].click()
        gugan_checkbox.click()
        list_checkboxes[arr_index[(quotient_16-1)*16+15]-1].click()
        gugan_checkbox.click()
        bln_pass = True   
        print('16 checked')
```
### At this point, there is no arr_index which has elements more than 8
```
    elif (quotient_8 > 0):
        list_checkboxes[arr_index[0]-1].click()
        gugan_checkbox.click()
        list_checkboxes[arr_index[(quotient_8-1)*8+7]-1].click()
        gugan_checkbox.click()
        bln_pass = True   
        print('8 checked')
        
    return bln_pass
```
### Click General primers corresponding elements
```
def check_general_primers(well_num):
    _indicies = df.loc[
                    df[3].str.contains(well_num), 0
                ].to_numpy().astype(int)
    indicies = [_index for _index in _indicies if (_index in general_primers)]
    print(well_num + str(indicies))

    bln_pass = False
 ```
### Click elements by 8...
    ```
    quotient_8 = int(len(indicies)/8)
    if len(indicies) >= 8:
        list_checkboxes[indicies[0]-1].click()
        gugan_checkbox.click()
        list_checkboxes[indicies[7]-1].click()
        gugan_checkbox.click()
        print('check 8')
        bln_pass = True
      ```   
### Click elements by 4...
 ``` 
    elif int(len(indicies)/4) > 0:
        list_checkboxes[indicies[0]-1].click()
        gugan_checkbox.click()
        list_checkboxes[indicies[3]-1].click()
        gugan_checkbox.click()
        print('check 4')
        bln_pass = True
  ``` 
### Click elements by 2...
 ``` 
    elif int(len(indicies)/2) > 0:
        list_checkboxes[indicies[0]-1].click()
        gugan_checkbox.click()
        list_checkboxes[indicies[1]-1].click()
        gugan_checkbox.click()
        print('check 2')
        bln_pass = True
        
    return bln_pass

        
bln_sample_to_send = True
``` 

## Functions for running the entire platting code
``` 
cnt = 0
max_loop_count = 1
while(bln_sample_to_send and cnt < max_loop_count):
    cnt = cnt + 1
    print('loop start, cnt: %d'%(cnt))
    
    
    bln_univ_pass = False
``` 
    
### Check for univ primers
    for arr_index in list_univ:
        _bln = check_univ_primers(arr_index)
        bln_univ_pass = _bln or bln_univ_pass
    print('univ_primers checked: ' + str(bln_univ_pass))
    
### Check for general primers
    bln_general_pass = False
    for well_num in list_well_num:
        _bln = check_general_primers(well_num)
        bln_general_pass = bln_general_pass or _bln
    print('general_primers checked: ' + str(bln_general_pass))
    
    bln_pass = bln_univ_pass or bln_general_pass
        
    if (bln_pass == False):  
        bln_sample_to_send = False
    else:
        print('Click SEND!!!!!!!!!!!!!')
        qty_btn = browser.find_element_by_xpath('/html/body/table/tbody/tr[6]/td/p/img[3]')
        qty_btn.click()
        print('\n')
        pass

# And repeat  

``` 
browser.switch_to_default_content()
browser.switch_to_frame('main')

# Web scraping - read the table.
soup = BeautifulSoup(browser.page_source)
table = soup.find_all('table')[4]
list_df = pd.read_html(str(table))
df = list_df[0]
df = df.dropna(thresh=4) #Remove missing values.
print('Web scraping finished!')
print(list_df)
print("--- %s seconds ---" % (time.time() - start_time))

# get all checkboxes in the webpage
len_table = len(df)
list_checkboxes = []
list_checkboxes.append(browser.find_element_by_xpath('/html/body/table/tbody/tr[3]/td/table/tbody/tr[2]/td[9]/input')) # for first row
for i in range(len_table-2):
    list_checkboxes.append(browser.find_element_by_xpath('/html/body/table/tbody/tr[3]/td/table/tbody/tr[%d]/td[7]/input'%(i+3)))
#get gugan checkbox
gugan_checkbox = browser.find_element_by_xpath('//*[@id="gugan_menu"]/table/tbody/tr[1]/td[2]/input')  
print('getting Checkbox finished!')
print("--- %s seconds ---" % (time.time() - start_time))
## setting up univeral primers ...
arr_index_EB3 = df.loc[df[4] == 'EB3', 0].to_numpy().astype(int)
arr_index_IntFw = df.loc[df[4] == 'IntFw', 0].to_numpy().astype(int)
arr_index_KanFw = df.loc[df[4] == 'KanFw', 0].to_numpy().astype(int)
arr_index_KanRv = df.loc[df[4] == 'KanRv', 0].to_numpy().astype(int)
arr_index_KanRv2 = df.loc[df[4] == 'KanRv2', 0].to_numpy().astype(int)
arr_index_IntFw2 = df.loc[df[4] == 'IntFw2', 0].to_numpy().astype(int)
arr_index_Kan_Del_Fw = df.loc[df[4] == 'Kan_Del_Fw', 0].to_numpy().astype(int)
arr_index_KanFw2 = df.loc[df[4] == 'KanFw2', 0].to_numpy().astype(int)
arr_index_EPORv = df.loc[df[4] == 'EPORv', 0].to_numpy().astype(int)
arr_index_EPOFw = df.loc[df[4] == 'EPOFw', 0].to_numpy().astype(int)


list_univ = [arr_index_EB3, arr_index_EPOFw, arr_index_EPORv, 
            arr_index_IntFw, arr_index_IntFw2, arr_index_Kan_Del_Fw,
            arr_index_KanFw, arr_index_KanFw2, arr_index_KanRv,arr_index_KanRv2]

# setting up General primers

list_alphabet = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H']
list_number = ['1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12']

list_well_num = []
for number in list_number:
    for alphabet in list_alphabet:
        list_well_num.append(r'\(' + alphabet + number + r'\)')

        
general_primers = df.loc[df[4].str.contains('Sequencing'), 0].to_numpy().astype(int)

print('list_univ: ' + str(list_univ))
print('list_well_num: ' + str(list_well_num))
print('general_primers: ' + str(general_primers))
def check_univ_primers(arr_index):
    # click elements by 48....\
    bln_pass = False
    quotient_48 = int(len(arr_index)/48)
    quotient_32 = int(len(arr_index)/32)
    quotient_16 = int(len(arr_index)/16)
    quotient_8 = int(len(arr_index)/8)
    print(arr_index)
    if quotient_48 > 0:
        list_checkboxes[arr_index[0]-1].click()
        gugan_checkbox.click()
        list_checkboxes[arr_index[(quotient_48-1)*48+47]-1].click()
        gugan_checkbox.click()
        bln_pass = True       
        print('48 checked')
        
    elif (quotient_32 > 0):
    # at this point, there is no arr_index which has elements more than 32
        list_checkboxes[arr_index[0]-1].click()
        gugan_checkbox.click()
        list_checkboxes[arr_index[(quotient_32-1)*32+31]-1].click()
        gugan_checkbox.click()
        bln_pass = True   
        print('32 checked')

    elif (quotient_16 > 0):
    # at this point, there is no arr_index which has elements more than 16
        list_checkboxes[arr_index[0]-1].click()
        gugan_checkbox.click()
        list_checkboxes[arr_index[(quotient_16-1)*16+15]-1].click()
        gugan_checkbox.click()
        bln_pass = True   
        print('16 checked')

    elif (quotient_8 > 0):
     # at this point, there is no arr_index which has elements more than 8
        list_checkboxes[arr_index[0]-1].click()
        gugan_checkbox.click()
        list_checkboxes[arr_index[(quotient_8-1)*8+7]-1].click()
        gugan_checkbox.click()
        bln_pass = True   
        print('8 checked')
        
    return bln_pass

def check_general_primers(well_num):
    _indicies = df.loc[
                    df[3].str.contains(well_num), 0
                ].to_numpy().astype(int)
    indicies = [_index for _index in _indicies if (_index in general_primers)]
    print(well_num + str(indicies))

    bln_pass = False
    # click elements by 8...
    quotient_8 = int(len(indicies)/8)
    if len(indicies) >= 8:
        list_checkboxes[indicies[0]-1].click()
        gugan_checkbox.click()
        list_checkboxes[indicies[7]-1].click()
        gugan_checkbox.click()
        print('check 8')
        bln_pass = True

    elif int(len(indicies)/4) > 0:
        list_checkboxes[indicies[0]-1].click()
        gugan_checkbox.click()
        list_checkboxes[indicies[3]-1].click()
        gugan_checkbox.click()
        print('check 4')
        bln_pass = True

    elif int(len(indicies)/2) > 0:
        list_checkboxes[indicies[0]-1].click()
        gugan_checkbox.click()
        list_checkboxes[indicies[1]-1].click()
        gugan_checkbox.click()
        print('check 2')
        bln_pass = True
        
    return bln_pass

        
bln_sample_to_send = True

cnt = 0
max_loop_count = 1
while(bln_sample_to_send and cnt < max_loop_count):
    cnt = cnt + 1
    print('loop start, cnt: %d'%(cnt))
    
    
    bln_univ_pass = False
    # check for univ primers
    for arr_index in list_univ:
        _bln = check_univ_primers(arr_index)
        bln_univ_pass = _bln or bln_univ_pass
    print('univ_primers checked: ' + str(bln_univ_pass))
    
    # check for general primers
    bln_general_pass = False
    for well_num in list_well_num:
        _bln = check_general_primers(well_num)
        bln_general_pass = bln_general_pass or _bln
    print('general_primers checked: ' + str(bln_general_pass))
    
    bln_pass = bln_univ_pass or bln_general_pass
        
    if (bln_pass == False):  
        bln_sample_to_send = False
    else:
        print('Click SEND!!!!!!!!!!!!!')
        qty_btn = browser.find_element_by_xpath('/html/body/table/tbody/tr[6]/td/p/img[3]')
        qty_btn.click()
        print('\n')
        pass

```

## Deployment

Add additional notes about how to deploy this on a live system

## Built With

* [Dropwizard](http://www.dropwizard.io/1.0.2/docs/) - The web framework used
* [Maven](https://maven.apache.org/) - Dependency Management
* [ROME](https://rometools.github.io/rome/) - Used to generate RSS Feeds

## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/your/project/tags). 

## Authors

* **Billie Thompson** - *Initial work* - [PurpleBooth](https://github.com/PurpleBooth)

See also the list of [contributors](https://github.com/your/project/contributors) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

* Hat tip to anyone whose code was used
* Inspiration
* etc
