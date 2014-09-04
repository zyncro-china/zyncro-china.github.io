---
title: review confident ruby
date: 2014-09-03 03:44 UTC
tags: ruby
---

## 再读confident ruby(performing work部分)

再读confident ruby，记下以下要点。

## 每个函数应包含4部分
1. collecting input
1. performing work
1. delivering result
1. handling failures

## performing work的模式
这个部分只有一个pattern

### sending a strong message
要发消息给trused object,需要3个条件

1. 识别要完成工作所需要发送的消息
1. 识别这些消息对应的roles
1. 确保method能担任这些roles

#### 一个从文件中读取用户信息，并发邮件的例子

##### 需要做以下步骤

1. Parse the purchase records from the CSV contained in a provided IO object.
2. For each purchase record, use the record's email address to get the
associated customer record, or, if the email hasn't been seen before, create a
new customer record in our system.
3. Use the legacy record's product ID to find or create a product record in our
system.
4. Add the product to the customer record's list of purchases.
5. Notify the customer of the new location where they can download their files
and update their account info.
6. Log the successful import of the purchase record.

##### 识别message

1. \#parse\_legacy\_purchase\_records.
2. For \#each purchase record, use the record's \#email\_address to \#get\_customer.
3. Use the record's \#product\_id to \#get\_product.
4. \#add\_purchased\_product to the customer record.
5. \#notify\_of\_files\_available for the purchased product.
6. \#log\_successful\_import of the product record.

##### 识别roles

|Message|Receiver Role|
| ------------- | ------------- |
|#parse_legacy_purchase_records|legacy_data_parser|
|#each|purchase_list|
|#email_address,|#product_id|purchase_record|
|#get_customer|customer_list|
|#get_product|product_inventory|
|#add_purchased_product|customer|
|#notify_of_files_available|customer|
|#log_successful_import|data_importer|

#### 关于duck type
不要在method中再根据type写分支逻辑，而是首先找出我们需要什么样的message和role，然后确保我们在程序里只允许使用准备好的duck
