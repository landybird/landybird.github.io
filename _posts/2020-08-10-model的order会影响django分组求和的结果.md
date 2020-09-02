---
title: model的order会影响django分组求和的结果      
description: model的order会影响django分组求和的结果   
categories:
- python
tags:
- python   
---
    

```python

class AccountsInsightsHourly(models.Model):
    account_id = models.CharField(max_length=32, blank=True, null=True)
    spend = models.DecimalField(max_digits=12, decimal_places=2, blank=True, null=True)
    date = models.IntegerField(blank=True, null=True)
    hour = models.IntegerField(blank=True, null=True)
    created_time = models.DateTimeField(blank=True, null=True)

    class Meta:
        managed = False
        db_table = 'accounts_insights_hourly'
        unique_together = (('account_id', 'date', 'hour'),)
        ordering = ["account_id", "hour"]




id  account_id    spend    date        hour   created_time 

1     1222         200     20200820    12      ....
1     1222         300     20200820    14      ....
```

直接对 account_id 分组， 求最大值会得到不正确的结果

[default-ordering-or-order-by](https://docs.djangoproject.com/en/1.11/topics/db/aggregation/#interaction-with-default-ordering-or-order-by)
```python
# 1
base_queryset_yesterday = AccountsInsightsHourly.objects.filter(date=date_yesterday). \
            annotate(yesterday_spend=Max("spend", output_field=FloatField())). \
            values("account_id", "yesterday_spend")

# print(base_queryset_yesterday.query) 这里的group by 是 id 
<QuerySet [{'account_id': '1222', 'yesterday_spend': 200}, {'account_id': '1222', 'yesterday_s
pend': 300}]>

# 2 
base_queryset_yesterday = AccountsInsightsHourly.objects.filter(date=date_yesterday).\
            values("account_id"). \
            annotate(yesterday_spend=Max("spend", output_field=FloatField())). \
            values("account_id", "yesterday_spend").\

# print(base_queryset_yesterday.query) 这里的group by 是 account_id 和 hour
<QuerySet [{'account_id': '1222', 'yesterday_spend': 200}, {'account_id': '1222', 'yesterday_s
pend': 300}]>

# 3 需要加上 order_by()
 base_queryset_yesterday = AccountsInsightsHourly.objects.filter(date=date_yesterday).\
            values("account_id"). \
            annotate(yesterday_spend=Max("spend", output_field=FloatField())). \
            values("account_id", "yesterday_spend").\
            order_by() 

<QuerySet [{'account_id': '1222', 'yesterday_spend': 300}]>

```

|Account_id|余额|过去7天消耗合计|
|---|---|---|
|1232323|23.4|34.5|

|Account_id|余额|过去3天消耗合计|
|---|---|---|
|1232323|23.4|34.5|