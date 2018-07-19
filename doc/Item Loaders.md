# Item Loaders
阅读文档[item loaders](https://doc.scrapy.org/en/latest/topics/loaders.html)部分

## Item Loaders用来做什么？
> Item Loaders provide a convenient mechanism for populating scraped Items
item loader主要用来更加方便的填充item，比较一下例子。
```
class StudyItem(Item):
    full_name = Field()
    age = Field()
    weight = Field()
    height = Field()
```
scrapy的Item只可以实现通过类似下面的方法
```
def parse(self, response):
    full_name = response.xpath("//div[contains(@class,'name')]/text()").extract()
    # i.e. returns ugly ['John\n', '\n\t  ', '  Snow']
    item['full_name'] = ' '.join(i.strip() for i in full_name if i.strip())
    age = response.xpath("//div[@class='age']/text()").extract_first(0)
    item['age'] = int(age) 
    weight = response.xpath("//div[@class='weight']/text()").extract_first(0)
    item['weight'] = int(age) 
    height = response.xpath("//div[@class='height']/text()").extract_first(0)
    item['height'] = int(age) 
    return item
```
对比Item Loaders 方法
```
# import的function是scrapy自带的方法，不重要
from scrapy.loader.processors import Compose, MapCompose, Join, TakeFirst
clean_text = Compose(MapCompose(lambda v: v.strip()), Join())   
to_int = Compose(TakeFirst(), int)

class MyItemLoader(ItemLoader):
    default_item_class = MyItem
    full_name_out = clean_text
    age_out = to_int
    weight_out = to_int
    height_out = to_int

# parse as many different places and times as you want  
def parse(self, response):
    loader = MyItemLoader(item=StudyItem(), response=response)
    loader.add_xpath('full_name', "//div[contains(@class,'name')]/text()")
    loader.add_xpath('age', "//div[@class='age']/text()")
    loader.add_xpath('weight', "//div[@class='weight']/text()")
    loader.add_xpath('height', "//div[@class='height']/text()")
    return loader.load_item()
```
## 实现
猜测了一下具体实现
```
class ItemLoader:
  def __init__(item: Item, response: str):
      self.item = item
      self.response = response
    
  def add_xpath(self, key, selector):
      item['key'] = response.xpath(selector)
  
  def load_item(self):
      return self.item
      
  def Join(self): 
    ......
```
TODO： 源码


## 用法
### input和output程序
input用来处理传入的数据
output输出数据

### 使用dome
#### 定义item
```
定义一个处理数据的函数
def filter_price(value):
    if value.isdigit():
        return value

#### 定义item
class Product(scrapy.Item):
    name = scrapy.Field(
        input_processor=MapCompose(remove_tags),
        output_processor=Join(),
    )
    price = scrapy.Field(
        input_processor=MapCompose(remove_tags, filter_price),
        output_processor=TakeFirst(),
    )
```
#### 通过itemloader加载item
```
il = ItemLoader(item=Product())
```
#### 解析数据
```
il.add_value('name', [u'Welcome to my', u'<strong>website</strong>'])
il.add_value('price', [u'&euro;', u'<span>1000</span>'])
il.load_item()
```
#### 结果
```
{'name': u'Welcome to my website', 'price': u'1000'}
```

## 重要api
context
key/value dict 在输入和输出程序之间


## 参考文档：
[Items vs item loaders in scrapy](https://stackoverflow.com/questions/39127256/items-vs-item-loaders-in-scrapy)  

[Item Loaders - 数据传递的另一中方式](https://zhuanlan.zhihu.com/p/28249356)

