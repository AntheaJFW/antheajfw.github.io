---
title: "Flattening deeply nested json data to pandas dataframe with json_normalize"
collection: projects
tags:
    - python
    - notebook
--- 
It is not unusual to see json data being stored as strings in databases as it is
more efficient to store certain data in these formats than to convert them back
into tabular format especially when the data must be readily available at all
times.

As a result, sometimes we have to use scripts to flatten these data to work with
them. For deeply nested data, you may use `from pandas.io.json import
json_normalize`. 

**In [1]:**

{% highlight python %}
import json
import pandas as pd
from pandas.io.json import json_normalize
{% endhighlight %}
 
### Data sourced from [Kaggle, FakeNewsNet dataset](https://www.kaggle.com/mdepa
k/fakenewsnet#BuzzFeed_real_news_content.csv): 

**In [2]:**

{% highlight python %}
df = pd.read_csv('BuzzFeed_real_news_content.csv')
{% endhighlight %}
 
Below is the first few lines of the `meta_data` column which contains the json
data stored as a string in each row: 

**In [3]:**

{% highlight python %}
df.head()[['meta_data']]
{% endhighlight %}




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>meta_data</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>{"description": "\u201cWe believe at this poin...</td>
    </tr>
    <tr>
      <td>1</td>
      <td>{"fb_title": "Trump: Drugs a 'Very, Very Big F...</td>
    </tr>
    <tr>
      <td>2</td>
      <td>{"googlebot": "noimageindex", "og": {"site_nam...</td>
    </tr>
    <tr>
      <td>3</td>
      <td>{"description": "He sees it as zero-sum. She b...</td>
    </tr>
    <tr>
      <td>4</td>
      <td>{"fb_title": "President Obama Vetoes 9/11 Vict...</td>
    </tr>
  </tbody>
</table>
</div>



**In [4]:**

{% highlight python %}
# Converting metadata columns from string to dict
def to_dict(value):
    try:
        obj = json.loads(value)
        return obj
    except TypeError:
        return None
{% endhighlight %}

**In [5]:**

{% highlight python %}
df['meta_data'] = df['meta_data'].apply(to_dict)
{% endhighlight %}

**In [6]:**

{% highlight python %}
# Flattening out one row of metadata for visual indication of number of rows
json_normalize(df['meta_data'][0])
{% endhighlight %}




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>description</th>
      <th>robots</th>
      <th>viewport</th>
      <th>googlebot</th>
      <th>og.site_name</th>
      <th>og.description</th>
      <th>og.title</th>
      <th>og.locale</th>
      <th>og.image</th>
      <th>og.updated_time</th>
      <th>og.url</th>
      <th>og.type</th>
      <th>fb.app_id</th>
      <th>fb.pages</th>
      <th>article.section</th>
      <th>article.tag</th>
      <th>article.published_time</th>
      <th>article.modified_time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>“We believe at this point in this time this wa...</td>
      <td>noimageindex</td>
      <td>initial-scale=1,maximum-scale=1,user-scalable=no</td>
      <td>noimageindex</td>
      <td>Eagle Rising</td>
      <td>“We believe at this point in this time this wa...</td>
      <td>Another Terrorist Attack in NYC...Why Are we S...</td>
      <td>en_US</td>
      <td>http://eaglerising.com/wp-content/uploads/2016...</td>
      <td>2016-09-22T10:49:05+00:00</td>
      <td>http://eaglerising.com/36942/another-terrorist...</td>
      <td>article</td>
      <td>256195528075351</td>
      <td>135665053303678</td>
      <td>Political Correctness</td>
      <td>terrorism</td>
      <td>2016-09-22T07:10:30+00:00</td>
      <td>2016-09-22T10:49:05+00:00</td>
    </tr>
  </tbody>
</table>
</div>


 
Creating a function to normalize each row metadata to columns, and appending it
to a list to concat and concatenating them together to create a new dataframe
from the metadata: 

**In [7]:**

{% highlight python %}
new_dataframe_lists = []
def json_to_columns(row):
    if row['meta_data']:
        row_df = json_normalize(row['meta_data'])
        new_dataframe_lists.append(row_df)
    else:
        new_dataframe_lists.append(pd.DataFrame({}, columns=new_columns))
{% endhighlight %}

**In [8]:**

{% highlight python %}
# apply json_to_columns function and add the resulting columns to a new list
df.apply(json_to_columns, axis=1)
{% endhighlight %}




    0     None
    1     None
    2     None
    3     None
    4     None
          ... 
    86    None
    87    None
    88    None
    89    None
    90    None
    Length: 91, dtype: object



**In [9]:**

{% highlight python %}
# Checking that the number of rows are all accounted for
assert len(new_dataframe_lists) == len(df)
{% endhighlight %}

**In [10]:**

{% highlight python %}
metadata = pd.concat(new_dataframe_lists, sort=False)
metadata
{% endhighlight %}




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>description</th>
      <th>robots</th>
      <th>viewport</th>
      <th>googlebot</th>
      <th>og.site_name</th>
      <th>og.description</th>
      <th>og.title</th>
      <th>og.locale</th>
      <th>og.image</th>
      <th>og.updated_time</th>
      <th>...</th>
      <th>msApplication-PackageFamilyName</th>
      <th>verify-v1</th>
      <th>og.image.type</th>
      <th>publisher</th>
      <th>theme</th>
      <th>object-hash</th>
      <th>apple-itunes-app</th>
      <th>date</th>
      <th>vr.category</th>
      <th>vr.type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>“We believe at this point in this time this wa...</td>
      <td>noimageindex</td>
      <td>initial-scale=1,maximum-scale=1,user-scalable=no</td>
      <td>noimageindex</td>
      <td>Eagle Rising</td>
      <td>“We believe at this point in this time this wa...</td>
      <td>Another Terrorist Attack in NYC...Why Are we S...</td>
      <td>en_US</td>
      <td>http://eaglerising.com/wp-content/uploads/2016...</td>
      <td>2016-09-22T10:49:05+00:00</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>0</td>
      <td>The police-involved shooting of a black man ha...</td>
      <td>index, follow</td>
      <td>initial-scale=1.0, maximum-scale=1.0, user-sca...</td>
      <td>NaN</td>
      <td>ABC News</td>
      <td>The police-involved shooting of a black man ha...</td>
      <td>Trump: Drugs a 'Very, Very Big Factor' in Char...</td>
      <td>NaN</td>
      <td>http://a.abcnews.com/images/Politics/AP_donald...</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>0</td>
      <td>NaN</td>
      <td>noimageindex</td>
      <td>width=device-width, initial-scale=1</td>
      <td>noimageindex</td>
      <td>John Hawkins' Right Wing News</td>
      <td>Freedom is the bedrock that this nation was bu...</td>
      <td>Obama To UN: 'Giving Up Liberty, Enhances Secu...</td>
      <td>en_US</td>
      <td>NaN</td>
      <td>2016-09-21T17:46:06-04:00</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>0</td>
      <td>He sees it as zero-sum. She believes all boats...</td>
      <td>NaN</td>
      <td>width=device-width, initial-scale=1</td>
      <td>NaN</td>
      <td>POLITICO Magazine</td>
      <td>He sees it as zero-sum. She believes all boats...</td>
      <td>Trump vs. Clinton: A Fundamental Clash over Ho...</td>
      <td>NaN</td>
      <td>http://static.politico.com/e9/11/6144cdc24e319...</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>0</td>
      <td>President Obama has set up a showdown with Con...</td>
      <td>index, follow</td>
      <td>initial-scale=1.0, maximum-scale=1.0, user-sca...</td>
      <td>NaN</td>
      <td>ABC News</td>
      <td>President Obama has set up a showdown with Con...</td>
      <td>President Obama Vetoes 9/11 Victims Bill</td>
      <td>NaN</td>
      <td>http://a.abcnews.com/images/US/AP_Obama_BM_201...</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <td>0</td>
      <td>Now they are taking the lazy, but disgusting w...</td>
      <td>noimageindex</td>
      <td>initial-scale=1,maximum-scale=1,user-scalable=no</td>
      <td>noimageindex</td>
      <td>Eagle Rising</td>
      <td>Now they are taking the lazy, but disgusting w...</td>
      <td>It's "Trump is HITLER" Month at the Washington...</td>
      <td>en_US</td>
      <td>http://eaglerising.com/wp-content/uploads/2016...</td>
      <td>2016-09-22T12:23:39+00:00</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>0</td>
      <td>The president’s aides worry the Republican rep...</td>
      <td>NaN</td>
      <td>width=device-width, initial-scale=1</td>
      <td>NaN</td>
      <td>POLITICO</td>
      <td>The president’s aides worry the Republican rep...</td>
      <td>Obama’s team isn’t laughing at Trump anymore</td>
      <td>NaN</td>
      <td>http://v.politico.com/images/1155968404/201609...</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>0</td>
      <td>Donald Trump is up 3 points over Hillary Clint...</td>
      <td>NaN</td>
      <td>width=device-width, initial-scale=1.0, minimum...</td>
      <td>NaN</td>
      <td>CNN</td>
      <td>Donald Trump is up 3 points over Hillary Clint...</td>
      <td>Georgia poll: Donald Trump, Hillary Clinton in...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>0</td>
      <td>Doesn't get better than this...</td>
      <td>NaN</td>
      <td>width=device-width, initial-scale=1.0</td>
      <td>NaN</td>
      <td>AddictingInfo</td>
      <td>Doesn't get better than this...</td>
      <td>Chelsea Handler Gets The Last Word After RNC C...</td>
      <td>en_US</td>
      <td>http://addictinginfo.addictinginfoent.netdna-c...</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Addicting Info | The Knowledge You Crave</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>0</td>
      <td>In the most pivotal moment of 2016, these are ...</td>
      <td>NaN</td>
      <td>width=device-width, initial-scale=1</td>
      <td>NaN</td>
      <td>POLITICO</td>
      <td>In the most pivotal moment of 2016, these are ...</td>
      <td>Is Donald Trump qualified to be president?</td>
      <td>NaN</td>
      <td>http://v.politico.com/images/1155968404/201609...</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>91 rows × 95 columns</p>
</div>


 
(Optional) Joining the new metadata information with the original dataframe, so
that the metadata can be matched with each row of data 

**In [11]:**

{% highlight python %}
df.join(metadata.reset_index(drop=True), lsuffix='_data', rsuffix='_metadata')
{% endhighlight %}




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>title_data</th>
      <th>text</th>
      <th>url</th>
      <th>top_img</th>
      <th>authors</th>
      <th>source</th>
      <th>publish_date_data</th>
      <th>movies</th>
      <th>images</th>
      <th>...</th>
      <th>msApplication-PackageFamilyName</th>
      <th>verify-v1</th>
      <th>og.image.type</th>
      <th>publisher</th>
      <th>theme</th>
      <th>object-hash</th>
      <th>apple-itunes-app</th>
      <th>date</th>
      <th>vr.category</th>
      <th>vr.type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Real_1-Webpage</td>
      <td>Another Terrorist Attack in NYC…Why Are we STI...</td>
      <td>On Saturday, September 17 at 8:30 pm EST, an e...</td>
      <td>http://eaglerising.com/36942/another-terrorist...</td>
      <td>http://eaglerising.com/wp-content/uploads/2016...</td>
      <td>View All Posts,Leonora Cravotta</td>
      <td>http://eaglerising.com</td>
      <td>{'$date': 1474528230000}</td>
      <td>NaN</td>
      <td>http://constitution.com/wp-content/uploads/201...</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Real_10-Webpage</td>
      <td>Donald Trump: Drugs a 'Very, Very Big Factor' ...</td>
      <td>Less than a day after protests over the police...</td>
      <td>http://abcn.ws/2d4lNn9</td>
      <td>http://a.abcnews.com/images/Politics/AP_donald...</td>
      <td>More Candace,Adam Kelsey,Abc News,More Adam</td>
      <td>http://abcn.ws</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>http://www.googleadservices.com/pagead/convers...</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Real_11-Webpage</td>
      <td>Obama To UN: ‘Giving Up Liberty, Enhances Secu...</td>
      <td>Obama To UN: ‘Giving Up Liberty, Enhances Secu...</td>
      <td>http://rightwingnews.com/barack-obama/obama-un...</td>
      <td>http://rightwingnews.com/wp-content/uploads/20...</td>
      <td>Cassy Fiano</td>
      <td>http://rightwingnews.com</td>
      <td>{'$date': 1474476044000}</td>
      <td>https://www.youtube.com/embed/ji6pl5Vwrvk</td>
      <td>http://rightwingnews.com/wp-content/uploads/20...</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Real_12-Webpage</td>
      <td>Trump vs. Clinton: A Fundamental Clash over Ho...</td>
      <td>Getty Images Wealth Of Nations Trump vs. Clint...</td>
      <td>http://politi.co/2de2qs0</td>
      <td>http://static.politico.com/e9/11/6144cdc24e319...</td>
      <td>Jack Shafer,Erick Trickey,Zachary Karabell</td>
      <td>http://politi.co</td>
      <td>{'$date': 1474974420000}</td>
      <td>NaN</td>
      <td>https://static.politico.com/dims4/default/8a1c...</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>4</td>
      <td>Real_13-Webpage</td>
      <td>President Obama Vetoes 9/11 Victims Bill, Sett...</td>
      <td>President Obama today vetoed a bill that would...</td>
      <td>http://abcn.ws/2dh2NFs</td>
      <td>http://a.abcnews.com/images/US/AP_Obama_BM_201...</td>
      <td>John Parkinson,More John,Abc News,More Alexander</td>
      <td>http://abcn.ws</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>http://www.googleadservices.com/pagead/convers...</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <td>86</td>
      <td>Real_88-Webpage</td>
      <td>It’s “Trump is HITLER” Month at the Washington...</td>
      <td>Like much of the mainstream media, the Washing...</td>
      <td>http://eaglerising.com/36955/its-trump-is-hitl...</td>
      <td>http://eaglerising.com/wp-content/uploads/2016...</td>
      <td>View All Posts,Jeff Dunetz</td>
      <td>http://eaglerising.com</td>
      <td>{'$date': 1474526406000}</td>
      <td>NaN</td>
      <td>http://constitution.com/wp-content/uploads/201...</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>87</td>
      <td>Real_89-Webpage</td>
      <td>Obama’s team isn’t laughing at Trump anymore</td>
      <td>2016 Obama’s team isn’t laughing at Trump anym...</td>
      <td>http://politi.co/2cW2vAD</td>
      <td>http://v.politico.com/images/1155968404/201609...</td>
      <td>Edward-isaac Dovere,Eli Stokols,Politico Staff...</td>
      <td>http://politi.co</td>
      <td>{'$date': 1474866060000}</td>
      <td>NaN</td>
      <td>http://v.politico.com/images/1155968404/201609...</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>88</td>
      <td>Real_9-Webpage</td>
      <td>Georgia poll: Donald Trump, Hillary Clinton in...</td>
      <td>Story highlights Trump has 45%, Clinton 42% an...</td>
      <td>http://cnn.it/2cynaZx</td>
      <td>http://i2.cdn.cnn.com/cnnnext/dam/assets/16091...</td>
      <td>Tal Kopan</td>
      <td>http://cnn.it</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>http://i2.cdn.cnn.com/cnnnext/dam/assets/16102...</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>89</td>
      <td>Real_90-Webpage</td>
      <td>Chelsea Handler Gets The Last Word After RNC C...</td>
      <td>There may be a few women out there who enjoy a...</td>
      <td>http://www.addictinginfo.org/2016/09/19/chelse...</td>
      <td>http://addictinginfo.addictinginfoent.netdna-c...</td>
      <td>NaN</td>
      <td>http://www.addictinginfo.org</td>
      <td>{'$date': 1474243200000}</td>
      <td>NaN</td>
      <td>http://i.imgur.com/JeqZLhj.png,https://d5nxst8...</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Addicting Info | The Knowledge You Crave</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>90</td>
      <td>Real_91-Webpage</td>
      <td>Is Donald Trump qualified to be president?</td>
      <td>Off Message Is Donald Trump qualified to be pr...</td>
      <td>http://politi.co/2dkiM5e</td>
      <td>http://v.politico.com/images/1155968404/201609...</td>
      <td>Jack Shafer,Kyle Cheney,Daniel Strauss,Daniel ...</td>
      <td>http://politi.co</td>
      <td>{'$date': 1474866599000}</td>
      <td>NaN</td>
      <td>http://v.politico.com/images/1155968404/201609...</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>91 rows × 107 columns</p>
</div>


