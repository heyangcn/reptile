# reptile
#get sina china model  the final news 
#author emacs_young
#since Oct 26th 2016
#
#lience 1.0#
#
import requests
import re
import json
from bs4 import BeautifulSoup
from datetime import datetime

def getNewsDetail():
    result ={}    
    dict = getNewsUrl()
    j = 0
    for i  in dict :
        result[i] = {}
        result[i]['href'] = dict[i]        
        res = requests.get(result[i]['href'])
        res.encoding = 'utf-8'
        soup = BeautifulSoup(res.text,'html.parser')
        result[i]['title'] = soup.select('#artibodyTitle')[0].text
        result[i]['newssource'] = soup.select('.time-source span a')[0].text
        timesource = soup.select('.time-source')[0].contents[0].strip()
        result[i]['dt'] = datetime.strptime(timesource,'%Y年%m月%d日%H:%M')
        result[i]['article'] = '{@}'.join([p.text.strip() for p in soup.select('#artibody p')[:-1]])
        result[i]['editor'] = soup.select('.article-editor')[0].text.strip('责任编辑')
        result[i]['commentcount'] = getCommentCount(result[i]['href'])    
    return result

#get all final China news
def getNewsUrl():                                                                                       
    result = {}                                                                                         
    res =requests.get('http://news.sina.com.cn/china/')                                         
    res.encoding = 'utf-8'                                                                               
    soup = BeautifulSoup(res.text,'html.parser')
    i =0 
    for news in soup.select('.news-item'):
        if len(news.select("h2"))>0:     
            result[i] = news.select('h2 a')[0].attrs.get('href')
            i = i+1                                                                                    
    return result  
commentUrl = 'http://comment5.news.sina.com.cn/page/info?version=1&format=js&channel=gn&newsid=comos-{}&group=&compress=0&ie=utf-8&oe=utf-8&page=1&page_size=20'
def getCommentCount(newsurl):
    #newid = newsurl.split('/')[-1].restrip('.shtml').lstrip('doc-i') 
    m = re.search('doc-i(.+).shtml',newsurl)
    newsid = m.group(1)
    comments = requests.get(commentUrl.format(newsid))
    jd = json.loads(comments.text.strip('var data='))
    return jd['result']['count']['total']
print(getNewsDetail())
