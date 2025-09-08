## Automated Performance Alerts via Slack (Python)

Real-time messages on performance connecting data from SQL database and Slack API to provide timely client service and increase internal efficiency.


## This code includes

* libraries
* connecting database and webook (Slack API chatrooms)
* selecting KPI data
* comparison metrics
* message


# Import libraries
```
import warnings
warnings.filterwarnings("ignore")
import numpy as np
import pandas as pd

#from subprocess import check_output

import datetime
from datetime import date,timedelta
now = datetime.datetime.now()
```

# Date Selectors
```
yesterday_date = date.today() - timedelta(days = 1)
yesterday = str(yesterday_date)
print(yesterday)

one_week_date = date.today() - timedelta(days = 8)
one_week = str(one_week_date)

two_week_date = date.today() - timedelta(days = 15)
two_week = str(two_week_date)

three_week_date = date.today() - timedelta(days = 22)
three_week = str(three_week_date)

four_week_date = date.today() - timedelta(days = 29)
four_week = str(four_week_date)

last_year_date = date.today() - timedelta(days = 365)
last_year = str(last_year_date)


yesterday_day = yesterday_date.strftime('%A')
print(yesterday_day)
```
```
import os
import imaplib
import base64
import csv
```
# Connect to DB
```
import urllib
import pyodbc
import sqlalchemy
from sqlalchemy import MetaData, Table, Column, Integer, String, Float, Date, create_engine, event, update, DateTime
from sqlalchemy.orm import sessionmaker
import json
params = urllib.parse.quote_plus("DRIVER={SQL Server};SERVER=XXX.database.windows.net,XXXX;DATABASE=XXXXo;UID=XXX;PWD=XXX")
engine = create_engine("mssql+pyodbc:///?odbc_connect=%s" % params)

meta=MetaData()
```
```
#Slack connections
#import slack
#import slack_sdk
from slack_sdk import WebClient
from slack_sdk.errors import SlackApiError
from slack_sdk.webhook import WebhookClient
```

# Webhooks & accounts
```
sally_p = 'https://hooks.slack.com/services/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'
chat = 'https://hooks.slack.com/services/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'

#Account name for Slack Message and SQL filter
account_name = 'XXX'
google_ads_id = 'XXXXX'
```
# Google Ads Purchases & Revenue / Goal: ''
# GA Transaction / Goal: Usually 'transactions'
```
clicks = 'clicks'
conversions = 'purchases'
conversion_value = 'revenue'
```
# Slack inputs for webhook
```
slack_channel = sally_p
webhook = WebhookClient(slack_channel)
```

```
sql_statement = """
SELECT date AS date, 
    SUM(clicks) AS clicks,
    SUM(conversions) AS purchases,
    SUM(conversions_value) AS revenue
FROM [dbo].[google_ads_ql_campaign]
WHERE date = '""" + yesterday + """'
AND account_id ='""" + google_ads_id + """'
GROUP BY date
"""
print(sql_statement)

yesterday_df = pd.read_sql(sql_statement, con = engine)
yesterday_df['CVR'] = float(yesterday_df['purchases'] / yesterday_df['clicks'])
yesterday_df

yesterday_spend_sql = """
SELECT date,
SUM(cost) AS spend
FROM google_ads_ql_campaign
WHERE date = '""" + yesterday + """'
AND account_id ='6006919429'
GROUP BY date
"""

print(yesterday_spend_sql)
yesterday_spend_df = pd.read_sql(yesterday_spend_sql, con = engine)
yesterday_spend_df

yesterday_roas_df = float(yesterday_df['revenue'] / yesterday_spend_df.iloc[0][1])
print(yesterday_roas_df) 

yesterday_spend =  '$' + '{:,}'.format(int(round(yesterday_spend_df).iloc[0][1]))
print(yesterday_spend)                   
yesterday_clicks = '{:,}'.format(int(yesterday_df['clicks']))
print(yesterday_clicks)
yesterday_purchases = '{:,}'.format(int(yesterday_df['purchases']))
print(yesterday_purchases)
yesterday_revenue = '$'+'{:,}'.format(int(yesterday_df['revenue']))
print(yesterday_revenue)

#yesterday_cvr = str(round(yesterday_df['CVR'] * 100,2))
yesterday_cvr = str(round(yesterday_df.iloc[0][4] * 100,2)) + '%'
print(yesterday_cvr)
yesterday_roas = str(round(yesterday_roas_df* 100)) + '%'
print(yesterday_roas)
```

## Last Four Week Comparison
```
sql_statement_weeks_comp = """
SELECT date AS date, 
    SUM(clicks) AS clicks,
    SUM(conversions) AS purchases,
    SUM(conversions_value) AS revenue
    FROM [dbo].[google_ads_ql_campaign]
WHERE (date = '""" + one_week + """' OR date = '""" + two_week + """' OR date = '""" + three_week + """' OR date = '""" + four_week + """')
AND account_id ='""" + google_ads_id + """'
GROUP BY date
"""
print(sql_statement_weeks_comp)
weeks_comp_df = pd.read_sql(sql_statement_weeks_comp, con = engine)

last_four_weeks_spend_sql = """
SELECT date,
SUM(cost) AS spend
FROM [dbo].[google_ads_ql_campaign]
WHERE (date = '""" + one_week + """' OR date = '""" + two_week + """' OR date = '""" + three_week + """' OR date = '""" + four_week + """')
AND account_id ='""" + google_ads_id + """'
GROUP BY date
"""

print(last_four_weeks_spend_sql)
last_four_weeks_spend_df = pd.read_sql(last_four_weeks_spend_sql, con = engine)

weeks_comp_df
last_four_weeks_spend_df
```

## Average Calculation 
```
four_week_averages = weeks_comp_df.mean()
four_week_averages['spend'] = round(last_four_weeks_spend_df['spend'].mean())
four_week_averages['CVR'] = four_week_averages['purchases'] / four_week_averages['clicks']
four_week_averages['ROAS'] = four_week_averages['revenue'] / four_week_averages['spend']
print(four_week_averages)
four_week_averages

four_week_average_clicks = '{:,}'.format(int(four_week_averages['clicks']))
print(four_week_average_clicks)
four_week_average_purchases = '{:,}'.format(int(four_week_averages['purchases']))
print(four_week_average_purchases)
four_week_average_revenue = '$' + '{:,}'.format(int(four_week_averages['revenue']))
print(four_week_average_revenue)                                
four_week_average_cvr = str(round(four_week_averages['CVR'] * 100,2)) + '%'

#four_week_average_cvr = str(round(four_week_averages.iloc[0][4] * 100,2)) + '%'
print(four_week_average_cvr)
four_week_average_spend = '$' + '{:,}'.format(int(round(four_week_averages['spend'])))
print(four_week_average_spend)
four_week_average_roas = str(round(four_week_averages['ROAS']*100)) +'%'
print(four_week_average_roas)
```

## Comparisons (YoY vs Last 4 Weeks)
```
vs_4w_clicks_int = int(round(float((yesterday_df['clicks'] - four_week_averages['clicks']) / four_week_averages['clicks']) * 100,0))
vs_4w_clicks = str(vs_4w_clicks_int)
print(vs_4w_clicks)

vs_4w_purchases_int = int(round(float((yesterday_df['purchases'] - four_week_averages['purchases']) / four_week_averages['purchases']) * 100,0))
vs_4w_purchases = str(vs_4w_purchases_int)
print(vs_4w_purchases)

vs_4w_revenue_int = int(round(float((yesterday_df['revenue'] - four_week_averages['revenue']) / four_week_averages['revenue']) * 100,0))
vs_4w_revenue = str(vs_4w_revenue_int)
print(vs_4w_revenue)

vs_4w_spend_int = int(round(float((yesterday_spend_df['spend'] - four_week_averages['spend']) / four_week_averages['spend']) * 100,0))
vs_4w_spend = str(vs_4w_spend_int)
print(vs_4w_spend)

vs_4w_cvr_int = int(round(float((yesterday_df['CVR'] - four_week_averages['CVR']) / four_week_averages['CVR']) * 100,0))
vs_4w_cvr = str(vs_4w_cvr_int)
print(vs_4w_cvr)

vs_4w_roas_int = int(round(float((yesterday_roas_df - four_week_averages['ROAS']) / four_week_averages['ROAS']) * 100,0))
vs_4w_roas = str(vs_4w_roas_int)
print(vs_4w_roas)

def plus_minus(value):
    if value >= 0:
        result = '+'
    else:
        result = ''
    return str(result)
    
plus_minus(vs_4w_clicks_int)

plus_minus_clicks = plus_minus(vs_4w_clicks_int)
plus_minus_purchases = plus_minus(vs_4w_purchases_int)
plus_minus_revenue = plus_minus(vs_4w_revenue_int)
plus_minus_spend = plus_minus(vs_4w_spend_int)
plus_minus_cvr = plus_minus(vs_4w_cvr_int)
plus_minus_roas = plus_minus(vs_4w_roas_int)
```

## Message
```
message_intro = ':canary: Yesterday Recap for *' + account_name + '* - Paid Search'

message_spend = '*Spend:* ' + yesterday_spend + ' | ' + plus_minus_spend + vs_4w_spend + '% vs Avg Last 4 ' + yesterday_day + 's (' + four_week_average_spend + ')' 

message_clicks = '*Clicks:* ' + yesterday_clicks + ' | ' + plus_minus_clicks + vs_4w_clicks + '% vs Avg Last 4 ' + yesterday_day + 's (' + four_week_average_clicks + ')'

message_purchases = '*Purchases:* ' + yesterday_purchases + ' | ' + plus_minus_purchases + vs_4w_purchases + '% vs Avg Last 4 ' + yesterday_day + 's (' + four_week_average_purchases + ')' 

message_revenue = '*Revenue:* ' + yesterday_revenue + ' | ' + plus_minus_revenue + vs_4w_revenue + '% vs Avg Last 4 ' + yesterday_day + 's (' + four_week_average_revenue + ')' 

message_roas = '*ROAS:* ' + yesterday_roas + ' | ' + plus_minus_roas + vs_4w_roas + '% vs Avg Last 4 ' + yesterday_day + 's (' + four_week_average_roas + ')' 

message_cvr = '*CVR:* ' + yesterday_cvr + ' | ' + plus_minus_cvr + vs_4w_cvr + '% vs Avg Last 4 ' + yesterday_day + 's (' + four_week_average_cvr + ')' 

#message_revenue
slack_message=[
        {
            'type': 'section',
            'text': {
                'type': 'mrkdwn',
                'text': message_intro
            }
        },
#        
#        
        {
            'type': 'section',
            
            'text': {
                'type': 'mrkdwn',
                'text': message_spend
            }
            
        },
#        
#        
        {
            'type': 'section',
            
            'text': {
                'type': 'mrkdwn',
                'text': message_clicks
            }
            
        },
#        
#        
        {
            'type': 'section',
            
            'text': {
                'type': 'mrkdwn',
                'text': message_purchases
            }
            
        },
# 
#        
        {
            'type': 'section',
            
            'text': {
                'type': 'mrkdwn',
                'text': message_revenue
            }
            
        },
#
#        
        {
            'type': 'section',
            
            'text': {
                'type': 'mrkdwn',
                'text': message_roas
            }
            
        },
# 
#       
        {
            'type': 'section',
            
            'text': {
                'type': 'mrkdwn',
                'text': message_cvr
            }
            
        },
# 
#       
]

slack_message

def main():
    response = webhook.send(blocks = slack_message)
    assert response.status_code == 200
    assert response.body == "ok"

main()
```
