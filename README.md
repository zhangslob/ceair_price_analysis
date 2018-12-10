# ceair_price_analysis
simple analysis before to crawler ceair flight price

# 分析网站

- 该网站存在PC站与M站，分别是：`http://www.ceair.com`和`https://m.ceair.com`，接口分别是[PC站](http://www.ceair.com/otabooking/flight-search!doFlightSearch.shtml?searchCond={%22adtCount%22:1,%22chdCount%22:0,%22infCount%22:0,%22currency%22:%22CNY%22,%22tripType%22:%22OW%22,%22recommend%22:false,%22reselect%22:%22%22,%22page%22:%220%22,%22sortType%22:%22a%22,%22sortExec%22:%22a%22,%22segmentList%22:[{%22deptCd%22:%22SHA%22,%22arrCd%22:%22PEK%22,%22deptDt%22:%222018-12-11%22,%22deptAirport%22:%22%22,%22arrAirport%22:%22%22,%22deptCdTxt%22:%22%E4%B8%8A%E6%B5%B7%22,%22arrCdTxt%22:%22%E5%8C%97%E4%BA%AC%22,%22deptCityCode%22:%22SHA%22,%22arrCityCode%22:%22BJS%22}],%22version%22:%22A.1.0%22})和[M站](https://m.ceair.com/mobile/book/flight-search!flight.shtml?tripTypeFlag=0&tripType=OW&deptCode=SHA&arrvCode=BJS&deptDate=2018-12-11&backDate=&payType=RMB&goCodeType=1&backCodeType=1&channel=M)，对比之后决定采集M站
- 从该链接的Query string可以看到主要参数如下：
```json
{
tripTypeFlag: 0
tripType: OW
deptCode: SHA
arrvCode: BJS
deptDate: 2018-12-11
backDate: 
payType: RMB
goCodeType: 1
backCodeType: 1
channel: M
}
```
其中重要的有`deptCode`、`arrvCode`、`deptDate`、`backDate`，根据查询字段判断分别是：出发城市、到大城市、出发日期、返程日期。
- 搜索可得城市代码，例如：
```json
{
        code: 'CAN',
        name: 'G广州',
        city: '广州',
        city_code: '1',
        type: '331',
        regexInfo: 'CAN,CAN,广州,GuangZhou,GZ,GuangZhou,白云机场,BaiYunJiChang,BYJC,Guangzhou'
    }, {
        code: 'SZX',
        name: 'S深圳',
        city: '深圳',
        city_code: '1',
        type: '331',
        regexInfo: 'SZX,SZX,深圳,ShenZhen,SZ,SHENZHEN,宝安机场,BaoAnJiChang,BAJC,Shenzhen'
    }, {
        code: 'SIA',
        name: 'X西安',
        city: '西安',
        city_code: '1',
        type: '331',
        regexInfo: 'XIY,SIA,西安,XiAn,XA,XIAN,咸阳机场,XianYangJiChang,XYJC,Xi an Xianyang Apt'
    }
```

- 抓取日期最大为一年。比如今天是`2018-12-10`，可以查询的最远日期是`2019-12-10`
 
# 抓取方法。
1. 爬虫一。抓取所有城市代码入库，设置唯一索引：`code`，此爬虫可以天更。
2. 爬虫二。抓取所有城市航班价格.具体逻辑是：如果要抓取所有航班，则事先需要建立哈希表储存航班对应关系，再加上日期信息。开始任务前，将任务储存到redis，去redis中取任务完成分布式。根据任务数量判断更新周期。
3. App分析，和M站类似


# 重要资源

1. `https://m.ceair.com/resources/unpack/pages/booking/flight.res/flight.js`
2. `https://m.ceair.com/c/city/airports.js`
3. 开源项目：`https://github.com/SlothSimon/AirSpider/blob/master/Spider/ceair_spider.py`细节处理不好，`https://github.com/yuchu2016/flightSpider/blob/master/flightSpider/flightSpider/spiders/DongFang.py`航空网站很多

# 反爬分析

1. 频繁查询中出现`极验`活动验证码，处理方法：可以利用`selenium`模拟拖动，但方法极慢
2. 还会出现`查询频繁请稍候再试`，验证发现`cookie`储存当前用户信息，必须同意该网站`cookie和隐私政策`，可以尝试更换代理解决
3. 最好在挂代理情况下，测试出该网站对单一IP请求多少次才会出现验证码。出现验证码之后拉黑此IP更换可用IP，并重试此任务
