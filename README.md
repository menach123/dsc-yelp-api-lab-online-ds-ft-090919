
# Yelp API - Lab


## Introduction 

Now that we've seen how the Yelp API works and some basic Folium visualizations, it's time to put those skills to work in order to create a working map! Taking things a step further, you'll also independently explore how to perform pagination in order to retrieve a full results set from the Yelp API!

## Objectives

You will be able to: 
* Create HTTP requests to get data from Yelp API
* Parse HTTP responses and perform data analysis on the data returned
* Perform pagination to retrieve troves of data!
* Create a simple geographical system to view information about selected businesses, at a given location. 

## Problem Introduction

You've now worked with some API calls, but we have yet to see how to retrieve a more complete dataset in a programmatic manner. Returning to the Yelp API, the [documentation](https://www.yelp.com/developers/documentation/v3/business_search) also provides us details regarding the API limits. These often include details about the number of requests a user is allowed to make within a specified time limit and the maximum number of results to be returned. In this case, we are told that any request has a maximum of 50 results per request and defaults to 20. Furthermore, any search will be limited to a total of 1000 results. To retrieve all 1000 of these results, we would have to page through the results piece by piece, retrieving 50 at a time. Processes such as these are often referred to as pagination.

In this lab, you will define a search and then paginate over the results to retrieve all of the results. You'll then parse these responses as a DataFrame (for further exploration) and create a map using Folium to visualize the results geographically.

## Part I - Make the Initial Request

Start by making an initial request to the Yelp API. Your search must include at least 2 parameters: **term** and **location**. For example, you might search for pizza restaurants in NYC. The term and location is up to you but make the request below.


```python
import requests
import pandas as pd
import json

def get_keys(path):
    with open(path) as f:
        return json.load(f)

```


```python
keys = get_keys("/Users/Flatiron_User/.secret/yelp_api.json")

api_key = keys["api_key"]
```

## Pagination

Now that you have an initial response, you can examine the contents of the JSON container. For example, you might start with ```response.json().keys()```. Here, you'll see a key for `'total'`, which tells you the full number of matching results given your query parameters. Write a loop (or ideally a function) which then makes successive API calls using the offset parameter to retrieve all of the results (or 5000 for a particularly large result set) for the original query. As you do this, be mindful of how you store the data. Your final goal will be to reformat the data concerning the businesses themselves into a pandas DataFrame from the json objects.

**Note: be mindful of the API rate limits. You can only make 5000 requests per day and are also can make requests too fast. Start prototyping small before running a loop that could be faulty. You can also use time.sleep(n) to add delays. For more details see https://www.yelp.com/developers/documentation/v3/rate_limiting.**


```python
term = 'Mexican'
location = 'Knoxvile TN'
SEARCH_LIMIT = 50

url = 'https://api.yelp.com/v3/businesses/search'

headers = {
        'Authorization': 'Bearer {}'.format(api_key),
    }

url_params = {
                'term': term.replace(' ', '+'),
                'location': location.replace(' ', '+'),
                'limit': SEARCH_LIMIT,
                'offset': 0
            }
```


```python
response = requests.get(url, headers=headers, params=url_params)
```

## Exploratory Analysis

Take the restaurants from the previous question and do an initial exploratory analysis. At minimum, this should include looking at the distribution of features such as price, rating and number of reviews as well as the relations between these dimensions.


```python
len(response.json()['businesses'])
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-5-26882218917f> in <module>()
    ----> 1 len(response.json()['businesses'])
    

    NameError: name 'response' is not defined



```python
response.json()['total']
```




    205




```python
offset=0
dflist=[]
response = requests.get(url, headers=headers, params=url_params)
dflist.append(pd.DataFrame(response.json()['businesses']))
while offset < response.json()['total']+ SEARCH_LIMIT:
    
    
    offset += SEARCH_LIMIT
    url_params['offset'] = offset
    response = requests.get(url, headers=headers, params=url_params)
    dflist.append(pd.DataFrame(response.json()['businesses']))
df = pd.concat(dflist, ignore_index=True)    

```


```python
df
```




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
      <th>alias</th>
      <th>categories</th>
      <th>coordinates</th>
      <th>display_phone</th>
      <th>distance</th>
      <th>id</th>
      <th>image_url</th>
      <th>is_closed</th>
      <th>location</th>
      <th>name</th>
      <th>phone</th>
      <th>price</th>
      <th>rating</th>
      <th>review_count</th>
      <th>transactions</th>
      <th>url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>el-tipico-knoxville</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>{'latitude': 35.95718, 'longitude': -83.98297}</td>
      <td>(865) 583-4008</td>
      <td>2357.142481</td>
      <td>hl0iZaIOVXchBNxPDUdI3g</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/9atHrb...</td>
      <td>False</td>
      <td>{'address1': '4329 Lonas Dr', 'address2': '', ...</td>
      <td>El Tipico</td>
      <td>+18655834008</td>
      <td>$</td>
      <td>4.5</td>
      <td>51</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/el-tipico-knoxville?a...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>fiesta-garibaldi-mexican-grill-knoxville</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>{'latitude': 35.9213, 'longitude': -84.06384}</td>
      <td>(865) 249-7011</td>
      <td>5964.062139</td>
      <td>O-Mi4FTqw-JUWuiNAJh6qw</td>
      <td>https://s3-media3.fl.yelpcdn.com/bphoto/qgsfj1...</td>
      <td>False</td>
      <td>{'address1': '8520 Kingston Pike', 'address2':...</td>
      <td>Fiesta Garibaldi Mexican Grill</td>
      <td>+18652497011</td>
      <td>$$</td>
      <td>4.5</td>
      <td>55</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/fiesta-garibaldi-mexi...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>chez-guevara-knoxville</td>
      <td>[{'alias': 'tex-mex', 'title': 'Tex-Mex'}, {'a...</td>
      <td>{'latitude': 35.9266081460704, 'longitude': -8...</td>
      <td>(865) 690-5250</td>
      <td>4378.482994</td>
      <td>Gqdpdrf5xqEOwoY1dUMJOA</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/LRSZw4...</td>
      <td>False</td>
      <td>{'address1': '8025 Kingston Pike', 'address2':...</td>
      <td>Chez Guevara</td>
      <td>+18656905250</td>
      <td>$$</td>
      <td>4.0</td>
      <td>196</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/chez-guevara-knoxvill...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>la-esperanza-knoxville</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>{'latitude': 36.00187, 'longitude': -83.91159}</td>
      <td>(865) 637-9292</td>
      <td>10417.328638</td>
      <td>DPkDmIoGEsANReuSrZdepg</td>
      <td>https://s3-media4.fl.yelpcdn.com/bphoto/aE3aG7...</td>
      <td>False</td>
      <td>{'address1': '2412 Washington Pike', 'address2...</td>
      <td>La Esperanza</td>
      <td>+18656379292</td>
      <td>$</td>
      <td>4.5</td>
      <td>66</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/la-esperanza-knoxvill...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>la-casa-de-burro-flojo-knoxville-2</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 35.925811, 'longitude': -84.047818}</td>
      <td>(865) 247-0414</td>
      <td>4487.879282</td>
      <td>b0TId-7Av1NF54cTxUOJeA</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/zxrf6N...</td>
      <td>False</td>
      <td>{'address1': '8079 Kingston Pike', 'address2':...</td>
      <td>La Casa De Burro Flojo</td>
      <td>+18652470414</td>
      <td>$</td>
      <td>4.0</td>
      <td>73</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/la-casa-de-burro-floj...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>taqueria-la-herradura-knoxville-2</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 35.996128, 'longitude': -83.92339...</td>
      <td>(678) 815-5388</td>
      <td>9170.313198</td>
      <td>jHACXSWwZtfKlUh6-oHm7A</td>
      <td>https://s3-media3.fl.yelpcdn.com/bphoto/Ng3NyU...</td>
      <td>False</td>
      <td>{'address1': '2625 N Broadway', 'address2': No...</td>
      <td>Taqueria La Herradura</td>
      <td>+16788155388</td>
      <td>$</td>
      <td>4.5</td>
      <td>78</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/taqueria-la-herradura...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>chivo-taqueria-knoxville-2</td>
      <td>[{'alias': 'cocktailbars', 'title': 'Cocktail ...</td>
      <td>{'latitude': 35.9667821946351, 'longitude': -8...</td>
      <td>(865) 444-3161</td>
      <td>8162.133382</td>
      <td>Ii-0kABYzfmAE1y3bqTRSA</td>
      <td>https://s3-media4.fl.yelpcdn.com/bphoto/zksqms...</td>
      <td>False</td>
      <td>{'address1': '314 South Gay St', 'address2': '...</td>
      <td>Chivo Taqueria</td>
      <td>+18654443161</td>
      <td>$$</td>
      <td>4.0</td>
      <td>195</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/chivo-taqueria-knoxvi...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>porton-mexican-kitchen-knoxville</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>{'latitude': 35.9467401616264, 'longitude': -8...</td>
      <td>(865) 247-4914</td>
      <td>9598.584652</td>
      <td>T5mbTxYvpmLMGjLS5U7XBA</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/7Wz_C-...</td>
      <td>False</td>
      <td>{'address1': '9623 Countryside Center Ln', 'ad...</td>
      <td>Porton Mexican Kitchen</td>
      <td>+18652474914</td>
      <td>$</td>
      <td>5.0</td>
      <td>23</td>
      <td>[pickup, delivery]</td>
      <td>https://www.yelp.com/biz/porton-mexican-kitche...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>potrillos-taqueria-and-neveria-knoxville-2</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 35.8865762636829, 'longitude': -8...</td>
      <td>(865) 671-4763</td>
      <td>14743.142942</td>
      <td>BaSNLUuolVQjuNgeeS981Q</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/GbLhZv...</td>
      <td>False</td>
      <td>{'address1': '11145 Kingston Pike', 'address2'...</td>
      <td>Potrillos Taqueria &amp; Neveria</td>
      <td>+18656714763</td>
      <td>$</td>
      <td>4.0</td>
      <td>59</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/potrillos-taqueria-an...</td>
    </tr>
    <tr>
      <th>9</th>
      <td>habaneros-knoxville</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>{'latitude': 36.0042531133173, 'longitude': -8...</td>
      <td>(865) 247-0391</td>
      <td>14885.656379</td>
      <td>zG69Ag9Aew_tRihiPdN5cA</td>
      <td>https://s3-media3.fl.yelpcdn.com/bphoto/_HT1mK...</td>
      <td>False</td>
      <td>{'address1': '4704 Asheville Hwy', 'address2':...</td>
      <td>Habaneros</td>
      <td>+18652470391</td>
      <td>$$</td>
      <td>4.0</td>
      <td>56</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/habaneros-knoxville?a...</td>
    </tr>
    <tr>
      <th>10</th>
      <td>las-fuentes-knoxville</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>{'latitude': 35.9505372494459, 'longitude': -8...</td>
      <td>(865) 609-6866</td>
      <td>8344.972414</td>
      <td>5C8ZwMjwfaUggCHb9rWp3g</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/LldMGh...</td>
      <td>False</td>
      <td>{'address1': '2525 Chapman Hwy', 'address2': N...</td>
      <td>Las Fuentes</td>
      <td>+18656096866</td>
      <td>NaN</td>
      <td>4.5</td>
      <td>14</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/las-fuentes-knoxville...</td>
    </tr>
    <tr>
      <th>11</th>
      <td>soccer-taco-knoxville</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 35.93233, 'longitude': -84.01339}</td>
      <td>(865) 588-2020</td>
      <td>1931.357473</td>
      <td>0CZQG_4slkKM_xxCCmMoIQ</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/lo9dZj...</td>
      <td>False</td>
      <td>{'address1': '6701 Kingston Pike', 'address2':...</td>
      <td>Soccer Taco</td>
      <td>+18655882020</td>
      <td>$$</td>
      <td>3.5</td>
      <td>138</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/soccer-taco-knoxville...</td>
    </tr>
    <tr>
      <th>12</th>
      <td>mi-pueblo-knoxville</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>{'latitude': 35.9225082397461, 'longitude': -8...</td>
      <td>(865) 560-0411</td>
      <td>5010.613630</td>
      <td>44eS5EZhgDeCwzmjiPTeWQ</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/KR0Zy-...</td>
      <td>False</td>
      <td>{'address1': '1645 Downtown West Blvd', 'addre...</td>
      <td>Mi Pueblo</td>
      <td>+18655600411</td>
      <td>$</td>
      <td>4.5</td>
      <td>31</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/mi-pueblo-knoxville?a...</td>
    </tr>
    <tr>
      <th>13</th>
      <td>country-burrito-fresh-mex-knoxville</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>{'latitude': 35.95009, 'longitude': -84.15523}</td>
      <td>(865) 312-5881</td>
      <td>13335.456202</td>
      <td>Af4vAQVV8fRmSQnSgPDUOw</td>
      <td>https://s3-media3.fl.yelpcdn.com/bphoto/4sEMo6...</td>
      <td>False</td>
      <td>{'address1': '10636 Hardin Valley Rd', 'addres...</td>
      <td>Country Burrito Fresh Mex</td>
      <td>+18653125881</td>
      <td>NaN</td>
      <td>4.5</td>
      <td>37</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/country-burrito-fresh...</td>
    </tr>
    <tr>
      <th>14</th>
      <td>pelanchos-mexican-grill-knoxville</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>{'latitude': 35.91986, 'longitude': -84.05079}</td>
      <td>(865) 694-9060</td>
      <td>5078.831990</td>
      <td>iSDRACAikaqA33sEBfRC8g</td>
      <td>https://s3-media4.fl.yelpcdn.com/bphoto/01eZUJ...</td>
      <td>False</td>
      <td>{'address1': '1516 Downtown West Blvd', 'addre...</td>
      <td>Pelanchos Mexican Grill</td>
      <td>+18656949060</td>
      <td>$$</td>
      <td>3.5</td>
      <td>98</td>
      <td>[pickup, delivery]</td>
      <td>https://www.yelp.com/biz/pelanchos-mexican-gri...</td>
    </tr>
    <tr>
      <th>15</th>
      <td>mexico-lindo-knoxville-2</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>{'latitude': 35.92521, 'longitude': -84.093377}</td>
      <td>(865) 692-9515</td>
      <td>8215.245758</td>
      <td>ckw-wRRzrxhelpnMYsaOYg</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/w78CnL...</td>
      <td>False</td>
      <td>{'address1': '462 N Cedar Bluff Rd', 'address2...</td>
      <td>Mexico Lindo</td>
      <td>+18656929515</td>
      <td>$</td>
      <td>4.0</td>
      <td>46</td>
      <td>[delivery]</td>
      <td>https://www.yelp.com/biz/mexico-lindo-knoxvill...</td>
    </tr>
    <tr>
      <th>16</th>
      <td>victors-taco-shop-knoxville-2</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 35.954556, 'longitude': -83.9385}</td>
      <td>(865) 633-0330</td>
      <td>6188.516168</td>
      <td>14NO0KikSpCMm8SfcYP8ng</td>
      <td>https://s3-media3.fl.yelpcdn.com/bphoto/_75jav...</td>
      <td>False</td>
      <td>{'address1': '2121 Cumberland Ave', 'address2'...</td>
      <td>Victor's Taco Shop</td>
      <td>+18656330330</td>
      <td>$</td>
      <td>4.0</td>
      <td>96</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/victors-taco-shop-kno...</td>
    </tr>
    <tr>
      <th>17</th>
      <td>el-charro-knoxville-2</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>{'latitude': 35.94456, 'longitude': -83.98275}</td>
      <td>(865) 584-9807</td>
      <td>2212.469233</td>
      <td>liXS4syPfoKntXOfnHKtQQ</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/x-IvDS...</td>
      <td>False</td>
      <td>{'address1': '3816 Sutherland Ave', 'address2'...</td>
      <td>El Charro</td>
      <td>+18655849807</td>
      <td>$$</td>
      <td>3.0</td>
      <td>26</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/el-charro-knoxville-2...</td>
    </tr>
    <tr>
      <th>18</th>
      <td>taco-boy-knoxville</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 35.8971636680229, 'longitude': -8...</td>
      <td>(865) 288-7889</td>
      <td>16260.472195</td>
      <td>tSfSMr4GnwC6fMFGm78M_g</td>
      <td>https://s3-media4.fl.yelpcdn.com/bphoto/DTfLjQ...</td>
      <td>False</td>
      <td>{'address1': '747 N Campbell Station Rd', 'add...</td>
      <td>Taco Boy</td>
      <td>+18652887889</td>
      <td>$</td>
      <td>4.5</td>
      <td>79</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/taco-boy-knoxville?ad...</td>
    </tr>
    <tr>
      <th>19</th>
      <td>el-girasol-knoxville</td>
      <td>[{'alias': 'grocery', 'title': 'Grocery'}, {'a...</td>
      <td>{'latitude': 35.9418830871582, 'longitude': -8...</td>
      <td>(865) 558-0202</td>
      <td>2153.359752</td>
      <td>0Zv0Yt9NKE5bRdR2569IwQ</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/aXDBpY...</td>
      <td>False</td>
      <td>{'address1': '4829 Newcom Ave', 'address2': ''...</td>
      <td>El Girasol</td>
      <td>+18655580202</td>
      <td>$</td>
      <td>4.0</td>
      <td>37</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/el-girasol-knoxville?...</td>
    </tr>
    <tr>
      <th>20</th>
      <td>la-fiesta-mexican-restaurant-and-grill-knoxville</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>{'latitude': 35.9790878, 'longitude': -84.0141...</td>
      <td>(865) 588-2599</td>
      <td>3496.691088</td>
      <td>cE3QvlZ_s8UBxpNucJ3OFQ</td>
      <td>https://s3-media4.fl.yelpcdn.com/bphoto/l03DWV...</td>
      <td>False</td>
      <td>{'address1': '5707 Western Ave', 'address2': N...</td>
      <td>La Fiesta Mexican Restaurant &amp; Grill</td>
      <td>+18655882599</td>
      <td>$$</td>
      <td>4.0</td>
      <td>45</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/la-fiesta-mexican-res...</td>
    </tr>
    <tr>
      <th>21</th>
      <td>chuys-knoxville-2</td>
      <td>[{'alias': 'tex-mex', 'title': 'Tex-Mex'}, {'a...</td>
      <td>{'latitude': 35.91191, 'longitude': -84.08856}</td>
      <td>(865) 670-4141</td>
      <td>8431.015629</td>
      <td>7yNEjhUXReT6dAYluyYJDQ</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/l0vSKk...</td>
      <td>False</td>
      <td>{'address1': '9235 Kingston Pike', 'address2':...</td>
      <td>Chuy's</td>
      <td>+18656704141</td>
      <td>$$</td>
      <td>3.5</td>
      <td>197</td>
      <td>[delivery]</td>
      <td>https://www.yelp.com/biz/chuys-knoxville-2?adj...</td>
    </tr>
    <tr>
      <th>22</th>
      <td>la-palma-de-oro-knoxville</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>{'latitude': 36.006241, 'longitude': -84.020377}</td>
      <td>(865) 938-8222</td>
      <td>6478.831056</td>
      <td>SvsDhlHGnPXW8oiGP7C0-Q</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/ZUaSgq...</td>
      <td>False</td>
      <td>{'address1': '6631 Clinton Hwy', 'address2': '...</td>
      <td>La Palma De Oro</td>
      <td>+18659388222</td>
      <td>$</td>
      <td>3.5</td>
      <td>76</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/la-palma-de-oro-knoxv...</td>
    </tr>
    <tr>
      <th>23</th>
      <td>babalu-tapas-and-tacos-knoxville-2</td>
      <td>[{'alias': 'latin', 'title': 'Latin American'}...</td>
      <td>{'latitude': 35.96599, 'longitude': -83.918553}</td>
      <td>(865) 329-1002</td>
      <td>8180.054465</td>
      <td>8nfT3f1dSnCrIS8AB3eClg</td>
      <td>https://s3-media3.fl.yelpcdn.com/bphoto/euz-15...</td>
      <td>False</td>
      <td>{'address1': '412 S Gay St', 'address2': None,...</td>
      <td>Babalu Tapas &amp; Tacos</td>
      <td>+18653291002</td>
      <td>$$</td>
      <td>4.0</td>
      <td>358</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/babalu-tapas-and-taco...</td>
    </tr>
    <tr>
      <th>24</th>
      <td>abuelos-knoxville</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 35.9005440921107, 'longitude': -8...</td>
      <td>(865) 966-0075</td>
      <td>14626.982606</td>
      <td>R7kx6J1hbuQfqgMJ2qlBEg</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/5JAfDO...</td>
      <td>False</td>
      <td>{'address1': '11299 Parkside Drive', 'address2...</td>
      <td>Abuelo's</td>
      <td>+18659660075</td>
      <td>$$</td>
      <td>3.5</td>
      <td>103</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/abuelos-knoxville?adj...</td>
    </tr>
    <tr>
      <th>25</th>
      <td>el-chico-knoxville</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>{'latitude': 36.0130310058594, 'longitude': -8...</td>
      <td>(865) 687-4242</td>
      <td>7951.840376</td>
      <td>TCHUROAbGzp1rtV18LaotQ</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/uavKUK...</td>
      <td>False</td>
      <td>{'address1': '116 Cedar Ln', 'address2': '', '...</td>
      <td>El Chico</td>
      <td>+18656874242</td>
      <td>$$</td>
      <td>3.5</td>
      <td>61</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/el-chico-knoxville?ad...</td>
    </tr>
    <tr>
      <th>26</th>
      <td>o-chulos-grill-and-bar-knoxville-2</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 35.8639, 'longitude': -84.06457}</td>
      <td>(865) 500-7073</td>
      <td>10798.229535</td>
      <td>VIfEr7rGql6hmiTA5UZ-ng</td>
      <td>https://s3-media4.fl.yelpcdn.com/bphoto/p8NqCH...</td>
      <td>False</td>
      <td>{'address1': '9411 S Northshore Dr', 'address2...</td>
      <td>O Chulos Grill and Bar</td>
      <td>+18655007073</td>
      <td>$</td>
      <td>4.0</td>
      <td>33</td>
      <td>[pickup, delivery]</td>
      <td>https://www.yelp.com/biz/o-chulos-grill-and-ba...</td>
    </tr>
    <tr>
      <th>27</th>
      <td>amigos-and-beer-mexican-grill-knoxville-3</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 36.0304718017578, 'longitude': -8...</td>
      <td>(865) 444-1990</td>
      <td>14248.659170</td>
      <td>JPP6uWNoQzxBd8ZtccEwEA</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/QnfSEp...</td>
      <td>False</td>
      <td>{'address1': '5020 Washington Pike', 'address2...</td>
      <td>Amigo's &amp; Beer Mexican Grill</td>
      <td>+18654441990</td>
      <td>$</td>
      <td>4.0</td>
      <td>45</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/amigos-and-beer-mexic...</td>
    </tr>
    <tr>
      <th>28</th>
      <td>amigo-s-mexican-grill-knoxville</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 36.00078, 'longitude': -83.77449}</td>
      <td>(865) 312-9444</td>
      <td>21698.365408</td>
      <td>hE6dzYHfpXusnVQ3W4EaSA</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/SnN02h...</td>
      <td>False</td>
      <td>{'address1': '7228 Region Ln', 'address2': '',...</td>
      <td>Amigo’s Mexican Grill</td>
      <td>+18653129444</td>
      <td>NaN</td>
      <td>4.0</td>
      <td>2</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/amigo-s-mexican-grill...</td>
    </tr>
    <tr>
      <th>29</th>
      <td>la-piñata-mexican-bar-and-grill-knoxville</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>{'latitude': 35.9921824476744, 'longitude': -8...</td>
      <td>(865) 540-0800</td>
      <td>9192.718668</td>
      <td>l7Hg6d_Vb32nQtefADhLnQ</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/ADn4nR...</td>
      <td>False</td>
      <td>{'address1': '2038 N Broadway', 'address2': ''...</td>
      <td>La Piñata Mexican Bar &amp; Grill</td>
      <td>+18655400800</td>
      <td>NaN</td>
      <td>4.0</td>
      <td>28</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/la-pi%C3%B1ata-mexica...</td>
    </tr>
    <tr>
      <th>...</th>
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
      <th>175</th>
      <td>taco-bell-knoxville-24</td>
      <td>[{'alias': 'hotdogs', 'title': 'Fast Food'}, {...</td>
      <td>{'latitude': 35.88341, 'longitude': -84.155577}</td>
      <td>(865) 966-4144</td>
      <td>15238.219043</td>
      <td>8UPqIVva7ugkiraRUgbtZw</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/WsN3hU...</td>
      <td>False</td>
      <td>{'address1': '11217 Kingston Pike', 'address2'...</td>
      <td>Taco Bell</td>
      <td>+18659664144</td>
      <td>$</td>
      <td>3.0</td>
      <td>5</td>
      <td>[delivery]</td>
      <td>https://www.yelp.com/biz/taco-bell-knoxville-2...</td>
    </tr>
    <tr>
      <th>176</th>
      <td>taco-bell-knoxville-29</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 35.921053, 'longitude': -83.866328}</td>
      <td>(865) 577-5365</td>
      <td>13035.220638</td>
      <td>6tz3dCgHYyaDIkd3CSI_1g</td>
      <td>https://s3-media4.fl.yelpcdn.com/bphoto/lb3vUz...</td>
      <td>False</td>
      <td>{'address1': '6504 Chapman Hwy', 'address2': '...</td>
      <td>Taco Bell</td>
      <td>+18655775365</td>
      <td>$</td>
      <td>2.0</td>
      <td>4</td>
      <td>[delivery]</td>
      <td>https://www.yelp.com/biz/taco-bell-knoxville-2...</td>
    </tr>
    <tr>
      <th>177</th>
      <td>ihop-knoxville-4</td>
      <td>[{'alias': 'breakfast_brunch', 'title': 'Break...</td>
      <td>{'latitude': 36.01006, 'longitude': -83.97435}</td>
      <td>(865) 689-1202</td>
      <td>7403.529841</td>
      <td>weQqtZMmW5CGXfUAm2lMCw</td>
      <td>https://s3-media3.fl.yelpcdn.com/bphoto/oXzEBO...</td>
      <td>False</td>
      <td>{'address1': '5604 Merchants Center Blvd', 'ad...</td>
      <td>IHOP</td>
      <td>+18656891202</td>
      <td>$</td>
      <td>3.0</td>
      <td>37</td>
      <td>[delivery, pickup]</td>
      <td>https://www.yelp.com/biz/ihop-knoxville-4?adju...</td>
    </tr>
    <tr>
      <th>178</th>
      <td>farm-to-griddle-crepes-knoxville-2</td>
      <td>[{'alias': 'southern', 'title': 'Southern'}, {...</td>
      <td>{'latitude': 36.04514, 'longitude': -83.93742}</td>
      <td>(865) 246-9975</td>
      <td>12392.295819</td>
      <td>pJMDNpSRJZ-eGD7kNhDi5g</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/P6Wyc_...</td>
      <td>False</td>
      <td>{'address1': '2922 Edonia Dr', 'address2': Non...</td>
      <td>Farm To Griddle Crepes</td>
      <td>+18652469975</td>
      <td>$$</td>
      <td>2.0</td>
      <td>5</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/farm-to-griddle-crepe...</td>
    </tr>
    <tr>
      <th>179</th>
      <td>blaze-fast-fired-pizza-knoxville-2</td>
      <td>[{'alias': 'salad', 'title': 'Salad'}, {'alias...</td>
      <td>{'latitude': 35.9026173, 'longitude': -84.1493...</td>
      <td>(865) 225-1840</td>
      <td>13801.621399</td>
      <td>sVZUGtC46jB1FLQLDwvlGw</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/YyIv3I...</td>
      <td>False</td>
      <td>{'address1': '10978 Parkside Dr', 'address2': ...</td>
      <td>Blaze Fast-Fire'd Pizza</td>
      <td>+18652251840</td>
      <td>$</td>
      <td>4.0</td>
      <td>68</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/blaze-fast-fired-pizz...</td>
    </tr>
    <tr>
      <th>180</th>
      <td>taco-bell-maryville</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 35.75165, 'longitude': -83.99787}</td>
      <td>(865) 984-5045</td>
      <td>21951.846692</td>
      <td>8nHvPcTuqqKi_D5dnHhayA</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/uKBKuG...</td>
      <td>False</td>
      <td>{'address1': '836 Foothills Mall Dr', 'address...</td>
      <td>Taco Bell</td>
      <td>+18659845045</td>
      <td>$</td>
      <td>3.0</td>
      <td>11</td>
      <td>[delivery]</td>
      <td>https://www.yelp.com/biz/taco-bell-maryville?a...</td>
    </tr>
    <tr>
      <th>181</th>
      <td>taco-bell-knoxville-22</td>
      <td>[{'alias': 'tex-mex', 'title': 'Tex-Mex'}, {'a...</td>
      <td>{'latitude': 36.069001, 'longitude': -83.92707}</td>
      <td>(865) 922-0603</td>
      <td>15162.144148</td>
      <td>Gsc7YJbH1X72zFSGlsBaaw</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/QNpqCZ...</td>
      <td>False</td>
      <td>{'address1': '6802 Maynardville Hwy', 'address...</td>
      <td>Taco Bell</td>
      <td>+18659220603</td>
      <td>$</td>
      <td>3.0</td>
      <td>4</td>
      <td>[delivery]</td>
      <td>https://www.yelp.com/biz/taco-bell-knoxville-2...</td>
    </tr>
    <tr>
      <th>182</th>
      <td>taco-bell-knoxville-28</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 35.9239341577193, 'longitude': -8...</td>
      <td>(865) 694-3107</td>
      <td>3872.768327</td>
      <td>X7a17rQcF5r_XZI11zs98g</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/C37MtM...</td>
      <td>False</td>
      <td>{'address1': 'West Town Mall', 'address2': '76...</td>
      <td>Taco Bell</td>
      <td>+18656943107</td>
      <td>$</td>
      <td>2.5</td>
      <td>3</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/taco-bell-knoxville-2...</td>
    </tr>
    <tr>
      <th>183</th>
      <td>olive-garden-italian-restaurant-knoxville-3</td>
      <td>[{'alias': 'italian', 'title': 'Italian'}, {'a...</td>
      <td>{'latitude': 35.9034004211426, 'longitude': -8...</td>
      <td>(865) 966-4392</td>
      <td>13670.313407</td>
      <td>M757njU9z7d5hpnl3ya6GA</td>
      <td>https://s3-media3.fl.yelpcdn.com/bphoto/yDKg23...</td>
      <td>False</td>
      <td>{'address1': '10923 Parkside Dr', 'address2': ...</td>
      <td>Olive Garden Italian Restaurant</td>
      <td>+18659664392</td>
      <td>$$</td>
      <td>3.0</td>
      <td>32</td>
      <td>[delivery]</td>
      <td>https://www.yelp.com/biz/olive-garden-italian-...</td>
    </tr>
    <tr>
      <th>184</th>
      <td>taco-bell-alcoa</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 35.8165816815712, 'longitude': -8...</td>
      <td>(865) 681-8110</td>
      <td>14906.597192</td>
      <td>poQiL0MaOXlvYvbFoQnIFw</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/JC_GGJ...</td>
      <td>False</td>
      <td>{'address1': '2612 Alcoa Hwy', 'address2': '',...</td>
      <td>Taco Bell</td>
      <td>+18656818110</td>
      <td>$</td>
      <td>2.5</td>
      <td>9</td>
      <td>[delivery]</td>
      <td>https://www.yelp.com/biz/taco-bell-alcoa?adjus...</td>
    </tr>
    <tr>
      <th>185</th>
      <td>ihop-knoxville-6</td>
      <td>[{'alias': 'breakfast_brunch', 'title': 'Break...</td>
      <td>{'latitude': 36.077254, 'longitude': -83.926692}</td>
      <td>(865) 337-8128</td>
      <td>15991.300378</td>
      <td>rtKIYkTU47sI7tiIPxHAYA</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/H_H2GP...</td>
      <td>False</td>
      <td>{'address1': '7048 Maynardville Pike NE', 'add...</td>
      <td>IHOP</td>
      <td>+18653378128</td>
      <td>$</td>
      <td>2.5</td>
      <td>12</td>
      <td>[delivery, pickup]</td>
      <td>https://www.yelp.com/biz/ihop-knoxville-6?adju...</td>
    </tr>
    <tr>
      <th>186</th>
      <td>ihop-knoxville-2</td>
      <td>[{'alias': 'tradamerican', 'title': 'American ...</td>
      <td>{'latitude': 35.90648, 'longitude': -83.83625}</td>
      <td>(865) 577-0230</td>
      <td>16079.155905</td>
      <td>zES-ksmloUtaLAYn4A5cuQ</td>
      <td>https://s3-media3.fl.yelpcdn.com/bphoto/hZyjoe...</td>
      <td>False</td>
      <td>{'address1': '7609 Mountain Grove Dr', 'addres...</td>
      <td>IHOP</td>
      <td>+18655770230</td>
      <td>$$</td>
      <td>1.5</td>
      <td>18</td>
      <td>[pickup]</td>
      <td>https://www.yelp.com/biz/ihop-knoxville-2?adju...</td>
    </tr>
    <tr>
      <th>187</th>
      <td>taco-bell-alcoa-3</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 35.75974, 'longitude': -83.975523}</td>
      <td>(865) 983-5411</td>
      <td>21226.900986</td>
      <td>_-7QuliOweqDigoSW1v3Vw</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/s0pjSK...</td>
      <td>False</td>
      <td>{'address1': '297 S Calderwood St', 'address2'...</td>
      <td>Taco Bell</td>
      <td>+18659835411</td>
      <td>$</td>
      <td>2.0</td>
      <td>10</td>
      <td>[delivery]</td>
      <td>https://www.yelp.com/biz/taco-bell-alcoa-3?adj...</td>
    </tr>
    <tr>
      <th>188</th>
      <td>ihop-knoxville</td>
      <td>[{'alias': 'tradamerican', 'title': 'American ...</td>
      <td>{'latitude': 35.9305, 'longitude': -84.0265527}</td>
      <td>(865) 588-8331</td>
      <td>2704.035136</td>
      <td>yQsEIVT0g6-lr0vAEwfA5A</td>
      <td>https://s3-media3.fl.yelpcdn.com/bphoto/sOCGSN...</td>
      <td>False</td>
      <td>{'address1': '7128 Kingston Pike', 'address2':...</td>
      <td>IHOP</td>
      <td>+18655888331</td>
      <td>$$$</td>
      <td>1.5</td>
      <td>38</td>
      <td>[delivery, pickup]</td>
      <td>https://www.yelp.com/biz/ihop-knoxville?adjust...</td>
    </tr>
    <tr>
      <th>189</th>
      <td>chilis-lenoir-city-2</td>
      <td>[{'alias': 'tradamerican', 'title': 'American ...</td>
      <td>{'latitude': 35.82382, 'longitude': -84.27038}</td>
      <td>(865) 988-4061</td>
      <td>27499.407595</td>
      <td>pSXfPQDVpPP1cysc9ZAamA</td>
      <td>https://s3-media3.fl.yelpcdn.com/bphoto/AVp8ep...</td>
      <td>False</td>
      <td>{'address1': '320 Fort Loudoun Medical Center ...</td>
      <td>Chili's</td>
      <td>+18659884061</td>
      <td>$$</td>
      <td>2.5</td>
      <td>23</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/chilis-lenoir-city-2?...</td>
    </tr>
    <tr>
      <th>190</th>
      <td>taco-bell-kodak</td>
      <td>[{'alias': 'hotdogs', 'title': 'Fast Food'}, {...</td>
      <td>{'latitude': 35.9872098940696, 'longitude': -8...</td>
      <td>(865) 465-3147</td>
      <td>36346.796059</td>
      <td>eekbsUKCWXj4O8JNrAHxWw</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/UX3Dtk...</td>
      <td>False</td>
      <td>{'address1': '145 Stadium Dr', 'address2': '',...</td>
      <td>Taco Bell</td>
      <td>+18654653147</td>
      <td>$</td>
      <td>3.0</td>
      <td>2</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/taco-bell-kodak?adjus...</td>
    </tr>
    <tr>
      <th>191</th>
      <td>taco-bell-lake-city-5</td>
      <td>[{'alias': 'tex-mex', 'title': 'Tex-Mex'}, {'a...</td>
      <td>{'latitude': 36.231945, 'longitude': -84.160021}</td>
      <td>(865) 426-9616</td>
      <td>34343.892749</td>
      <td>UrdPunuOAXIKyqJYmXtOCg</td>
      <td>https://s3-media3.fl.yelpcdn.com/bphoto/sMVl2-...</td>
      <td>False</td>
      <td>{'address1': '117 Colonial Lane', 'address2': ...</td>
      <td>Taco Bell</td>
      <td>+18654269616</td>
      <td>$</td>
      <td>1.0</td>
      <td>4</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/taco-bell-lake-city-5...</td>
    </tr>
    <tr>
      <th>192</th>
      <td>ihop-knoxville-3</td>
      <td>[{'alias': 'breakfast_brunch', 'title': 'Break...</td>
      <td>{'latitude': 35.90265, 'longitude': -84.1412}</td>
      <td>(865) 671-3804</td>
      <td>13137.422567</td>
      <td>2bAD1wqu3c3yL6WmcEtcZg</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/mlDjfa...</td>
      <td>False</td>
      <td>{'address1': '313 Lovell Rd', 'address2': '', ...</td>
      <td>IHOP</td>
      <td>+18656713804</td>
      <td>$</td>
      <td>2.5</td>
      <td>51</td>
      <td>[delivery, pickup]</td>
      <td>https://www.yelp.com/biz/ihop-knoxville-3?adju...</td>
    </tr>
    <tr>
      <th>193</th>
      <td>taco-bell-knoxville-2</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 36.0357341, 'longitude': -83.8742}</td>
      <td>(865) 544-1556</td>
      <td>15200.818183</td>
      <td>Hregj16obCX_Fmo0WUS4rw</td>
      <td></td>
      <td>False</td>
      <td>{'address1': '2961 Knoxville Center Dr', 'addr...</td>
      <td>Taco Bell</td>
      <td>+18655441556</td>
      <td>$</td>
      <td>4.0</td>
      <td>1</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/taco-bell-knoxville-2...</td>
    </tr>
    <tr>
      <th>194</th>
      <td>cottage-restaurant-lake-city</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>{'latitude': 36.2294960021973, 'longitude': -8...</td>
      <td>(865) 426-4221</td>
      <td>34038.437175</td>
      <td>H33WsVUDyo2AWaQrcxkkbA</td>
      <td></td>
      <td>False</td>
      <td>{'address1': '513 N Main St', 'address2': '', ...</td>
      <td>Cottage Restaurant</td>
      <td>+18654264221</td>
      <td>$</td>
      <td>3.0</td>
      <td>1</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/cottage-restaurant-la...</td>
    </tr>
    <tr>
      <th>195</th>
      <td>taco-bell-oak-ridge-3</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 36.007265, 'longitude': -84.256274}</td>
      <td>(865) 483-7630</td>
      <td>23353.219214</td>
      <td>5-CE-PrYPQmgZY2qYESMyA</td>
      <td>https://s3-media3.fl.yelpcdn.com/bphoto/HFSJbc...</td>
      <td>False</td>
      <td>{'address1': '353 South Illinois Avenue', 'add...</td>
      <td>Taco Bell</td>
      <td>+18654837630</td>
      <td>$</td>
      <td>3.0</td>
      <td>3</td>
      <td>[delivery]</td>
      <td>https://www.yelp.com/biz/taco-bell-oak-ridge-3...</td>
    </tr>
    <tr>
      <th>196</th>
      <td>taco-bell-loudon-2</td>
      <td>[{'alias': 'hotdogs', 'title': 'Fast Food'}, {...</td>
      <td>{'latitude': 35.73117, 'longitude': -84.38877}</td>
      <td>(865) 458-5440</td>
      <td>42081.085005</td>
      <td>ORka2xl96kEOir90ANG0jw</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/T_PNHC...</td>
      <td>False</td>
      <td>{'address1': '12395 Highway 72 North', 'addres...</td>
      <td>Taco Bell</td>
      <td>+18654585440</td>
      <td>$</td>
      <td>3.5</td>
      <td>6</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/taco-bell-loudon-2?ad...</td>
    </tr>
    <tr>
      <th>197</th>
      <td>taco-bell-lenoir-city-2</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 35.797698, 'longitude': -84.256317}</td>
      <td>(865) 988-8700</td>
      <td>28064.102960</td>
      <td>yLeU247iDW27L9m1EJqiQA</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/mVcF6O...</td>
      <td>False</td>
      <td>{'address1': '716 Highway 321 North', 'address...</td>
      <td>Taco Bell</td>
      <td>+18659888700</td>
      <td>$</td>
      <td>2.0</td>
      <td>12</td>
      <td>[delivery]</td>
      <td>https://www.yelp.com/biz/taco-bell-lenoir-city...</td>
    </tr>
    <tr>
      <th>198</th>
      <td>taco-bell-maynardville</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 36.2468863007484, 'longitude': -8...</td>
      <td>(865) 745-1838</td>
      <td>37323.862522</td>
      <td>wAjC7arvj59tvErPZTD09w</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/lCtZFW...</td>
      <td>False</td>
      <td>{'address1': '3003 Maynardville Hwy', 'address...</td>
      <td>Taco Bell</td>
      <td>+18657451838</td>
      <td>$</td>
      <td>1.0</td>
      <td>2</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/taco-bell-maynardvill...</td>
    </tr>
    <tr>
      <th>199</th>
      <td>taco-bell-maryville-3</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>{'latitude': 35.717246, 'longitude': -84.0124763}</td>
      <td>(865) 379-6178</td>
      <td>25766.908519</td>
      <td>StEmvX7XSqzrzipsnhMBLA</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/CDXJxW...</td>
      <td>False</td>
      <td>{'address1': '2341 Market Place Dr', 'address2...</td>
      <td>Taco Bell</td>
      <td>+18653796178</td>
      <td>$</td>
      <td>1.5</td>
      <td>15</td>
      <td>[delivery]</td>
      <td>https://www.yelp.com/biz/taco-bell-maryville-3...</td>
    </tr>
    <tr>
      <th>200</th>
      <td>mcalisters-deli-alcoa</td>
      <td>[{'alias': 'delis', 'title': 'Delis'}, {'alias...</td>
      <td>{'latitude': 35.772139, 'longitude': -83.985135}</td>
      <td>(865) 380-0012</td>
      <td>19756.416156</td>
      <td>FgoTAt06wKRgCvxp5MgZ8A</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/hPL500...</td>
      <td>False</td>
      <td>{'address1': '465 Marilyn Ln', 'address2': Non...</td>
      <td>McAlister's Deli</td>
      <td>+18653800012</td>
      <td>$</td>
      <td>4.0</td>
      <td>21</td>
      <td>[]</td>
      <td>https://www.yelp.com/biz/mcalisters-deli-alcoa...</td>
    </tr>
    <tr>
      <th>201</th>
      <td>taco-bell-clinton-3</td>
      <td>[{'alias': 'hotdogs', 'title': 'Fast Food'}, {...</td>
      <td>{'latitude': 36.11533, 'longitude': -84.12033}</td>
      <td>(865) 457-7282</td>
      <td>21125.395756</td>
      <td>luHFRRwRMv1eb6N94wmfvg</td>
      <td>https://s3-media4.fl.yelpcdn.com/bphoto/U70iYr...</td>
      <td>False</td>
      <td>{'address1': '1114 N Charles G Seivers Blvd', ...</td>
      <td>Taco Bell</td>
      <td>+18654577282</td>
      <td>$</td>
      <td>1.0</td>
      <td>7</td>
      <td>[delivery]</td>
      <td>https://www.yelp.com/biz/taco-bell-clinton-3?a...</td>
    </tr>
    <tr>
      <th>202</th>
      <td>ihop-oak-ridge</td>
      <td>[{'alias': 'breakfast_brunch', 'title': 'Break...</td>
      <td>{'latitude': 36.007121, 'longitude': -84.25571}</td>
      <td>(865) 220-5450</td>
      <td>23300.037761</td>
      <td>0pr_hOyW0FDyhStS6MUe5A</td>
      <td>https://s3-media3.fl.yelpcdn.com/bphoto/9eKpco...</td>
      <td>False</td>
      <td>{'address1': '355 S Illinois Ave', 'address2':...</td>
      <td>IHOP</td>
      <td>+18652205450</td>
      <td>$</td>
      <td>2.5</td>
      <td>18</td>
      <td>[pickup, delivery]</td>
      <td>https://www.yelp.com/biz/ihop-oak-ridge?adjust...</td>
    </tr>
    <tr>
      <th>203</th>
      <td>taco-bell-seymour-2</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 35.87303, 'longitude': -83.76041}</td>
      <td>(865) 577-3430</td>
      <td>23753.957610</td>
      <td>mS-1HJ2M33-l00hrtB-r8A</td>
      <td>https://s3-media4.fl.yelpcdn.com/bphoto/MgLOiR...</td>
      <td>False</td>
      <td>{'address1': '11311 Chapman Highway', 'address...</td>
      <td>Taco Bell</td>
      <td>+18655773430</td>
      <td>$</td>
      <td>2.0</td>
      <td>4</td>
      <td>[delivery]</td>
      <td>https://www.yelp.com/biz/taco-bell-seymour-2?a...</td>
    </tr>
    <tr>
      <th>204</th>
      <td>ihop-maryville</td>
      <td>[{'alias': 'breakfast_brunch', 'title': 'Break...</td>
      <td>{'latitude': 35.75295, 'longitude': -83.99588}</td>
      <td>(865) 981-9677</td>
      <td>21814.867362</td>
      <td>46xENq7S6LIUCamYmvwNng</td>
      <td>https://s3-media3.fl.yelpcdn.com/bphoto/ZW7kT9...</td>
      <td>False</td>
      <td>{'address1': '906 Turner St', 'address2': '', ...</td>
      <td>IHOP</td>
      <td>+18659819677</td>
      <td>$</td>
      <td>2.0</td>
      <td>25</td>
      <td>[pickup, delivery]</td>
      <td>https://www.yelp.com/biz/ihop-maryville?adjust...</td>
    </tr>
  </tbody>
</table>
<p>205 rows × 16 columns</p>
</div>



## Mapping

Look at the initial Yelp example and try and make a map using Folium of the restaurants you retrieved. Be sure to also add popups to the markers giving some basic information such as name, rating and price.


```python
import folium

lat = 35.9606
long = -83.9207

#Create a map of the area
base_map = folium.Map([lat, long], zoom_start=9)
```


```python
for i in range(0,len(df)):
    lat= df.coordinates.iloc[i]['latitude']
    long= df.coordinates.iloc[i]['longitude']
    name= df.name.iloc[i]
    popup_text = name
    popup = folium.Popup(popup_text, parse_html=True)
    marker = folium.Marker(location=[lat, long], popup=popup)
    marker.add_to(base_map)
base_map
    
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2Nkbi5naXRoYWNrLmNvbS9weXRob24tdmlzdWFsaXphdGlvbi9mb2xpdW0vbWFzdGVyL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5jc3MiLz4KICAgIDxzdHlsZT5odG1sLCBib2R5IHt3aWR0aDogMTAwJTtoZWlnaHQ6IDEwMCU7bWFyZ2luOiAwO3BhZGRpbmc6IDA7fTwvc3R5bGU+CiAgICA8c3R5bGU+I21hcCB7cG9zaXRpb246YWJzb2x1dGU7dG9wOjA7Ym90dG9tOjA7cmlnaHQ6MDtsZWZ0OjA7fTwvc3R5bGU+CiAgICAKICAgICAgICAgICAgPG1ldGEgbmFtZT0idmlld3BvcnQiIGNvbnRlbnQ9IndpZHRoPWRldmljZS13aWR0aCwKICAgICAgICAgICAgICAgIGluaXRpYWwtc2NhbGU9MS4wLCBtYXhpbXVtLXNjYWxlPTEuMCwgdXNlci1zY2FsYWJsZT1ubyIgLz4KICAgICAgICAgICAgPHN0eWxlPgogICAgICAgICAgICAgICAgI21hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyB7CiAgICAgICAgICAgICAgICAgICAgcG9zaXRpb246IHJlbGF0aXZlOwogICAgICAgICAgICAgICAgICAgIHdpZHRoOiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgbGVmdDogMC4wJTsKICAgICAgICAgICAgICAgICAgICB0b3A6IDAuMCU7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgIDwvc3R5bGU+CiAgICAgICAgCjwvaGVhZD4KPGJvZHk+ICAgIAogICAgCiAgICAgICAgICAgIDxkaXYgY2xhc3M9ImZvbGl1bS1tYXAiIGlkPSJtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMiID48L2Rpdj4KICAgICAgICAKPC9ib2R5Pgo8c2NyaXB0PiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzID0gTC5tYXAoCiAgICAgICAgICAgICAgICAibWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzIiwKICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICBjZW50ZXI6IFszNS45NjA2LCAtODMuOTIwN10sCiAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NywKICAgICAgICAgICAgICAgICAgICB6b29tOiA5LAogICAgICAgICAgICAgICAgICAgIHpvb21Db250cm9sOiB0cnVlLAogICAgICAgICAgICAgICAgICAgIHByZWZlckNhbnZhczogZmFsc2UsCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICk7CgogICAgICAgICAgICAKCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHRpbGVfbGF5ZXJfMzVkZjUxMGU4M2QxNDg3NTg3YzkwNGJjZjBkNTExNTMgPSBMLnRpbGVMYXllcigKICAgICAgICAgICAgICAgICJodHRwczovL3tzfS50aWxlLm9wZW5zdHJlZXRtYXAub3JnL3t6fS97eH0ve3l9LnBuZyIsCiAgICAgICAgICAgICAgICB7ImF0dHJpYnV0aW9uIjogIkRhdGEgYnkgXHUwMDI2Y29weTsgXHUwMDNjYSBocmVmPVwiaHR0cDovL29wZW5zdHJlZXRtYXAub3JnXCJcdTAwM2VPcGVuU3RyZWV0TWFwXHUwMDNjL2FcdTAwM2UsIHVuZGVyIFx1MDAzY2EgaHJlZj1cImh0dHA6Ly93d3cub3BlbnN0cmVldG1hcC5vcmcvY29weXJpZ2h0XCJcdTAwM2VPRGJMXHUwMDNjL2FcdTAwM2UuIiwgImRldGVjdFJldGluYSI6IGZhbHNlLCAibWF4TmF0aXZlWm9vbSI6IDE4LCAibWF4Wm9vbSI6IDE4LCAibWluWm9vbSI6IDAsICJub1dyYXAiOiBmYWxzZSwgIm9wYWNpdHkiOiAxLCAic3ViZG9tYWlucyI6ICJhYmMiLCAidG1zIjogZmFsc2V9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2YzYWFlNzM3ZTUwZDRjZDBhNzQxZTExMzRhNjFkOWQxID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTU3MTgsIC04My45ODI5N10sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfYWYyMjZlZjNiODU5NDI0YmI3ODAxNDIyNzg4NjU3MmIgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzY5NjA1MDY0ZmExOTRkZDU5OGVmNDYzMjhiYTU1ZjcyID0gJChgPGRpdiBpZD0iaHRtbF82OTYwNTA2NGZhMTk0ZGQ1OThlZjQ2MzI4YmE1NWY3MiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RWwgVGlwaWNvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2FmMjI2ZWYzYjg1OTQyNGJiNzgwMTQyMjc4ODY1NzJiLnNldENvbnRlbnQoaHRtbF82OTYwNTA2NGZhMTk0ZGQ1OThlZjQ2MzI4YmE1NWY3Mik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9mM2FhZTczN2U1MGQ0Y2QwYTc0MWUxMTM0YTYxZDlkMS5iaW5kUG9wdXAocG9wdXBfYWYyMjZlZjNiODU5NDI0YmI3ODAxNDIyNzg4NjU3MmIpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfMDYyM2QwZmQyNjg3NGEzZWI3YzRmZDc3N2M5YTNkN2MgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45MjEzLCAtODQuMDYzODRdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzM2NDFhM2QyMzBjMDQzZTI4MzMzZjIyZDcwZTQ3YWYwID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF83ZTczYzU1ZTgwZDY0ODQ2YTI3MzE3MGIxYTEyZmRhNSA9ICQoYDxkaXYgaWQ9Imh0bWxfN2U3M2M1NWU4MGQ2NDg0NmEyNzMxNzBiMWExMmZkYTUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkZpZXN0YSBHYXJpYmFsZGkgTWV4aWNhbiBHcmlsbDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8zNjQxYTNkMjMwYzA0M2UyODMzM2YyMmQ3MGU0N2FmMC5zZXRDb250ZW50KGh0bWxfN2U3M2M1NWU4MGQ2NDg0NmEyNzMxNzBiMWExMmZkYTUpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfMDYyM2QwZmQyNjg3NGEzZWI3YzRmZDc3N2M5YTNkN2MuYmluZFBvcHVwKHBvcHVwXzM2NDFhM2QyMzBjMDQzZTI4MzMzZjIyZDcwZTQ3YWYwKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzJiZGZmMGRhYWFkNDQ3OTI5ZjI3MDRkYWNiNGJmMjU0ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTI2NjA4MTQ2MDcwNCwgLTg0LjA0NzAyMjI3MTE2MzldLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzlkMTg1MDkxOWRlNjQ5MTY5ZTQ0ZGQ5MGMwMmEyMjNjID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8wNTJkOWM5YzY3NjQ0ZTNmOTIwMGU1OWZkYjAyZWQyYyA9ICQoYDxkaXYgaWQ9Imh0bWxfMDUyZDljOWM2NzY0NGUzZjkyMDBlNTlmZGIwMmVkMmMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNoZXogR3VldmFyYTwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF85ZDE4NTA5MTlkZTY0OTE2OWU0NGRkOTBjMDJhMjIzYy5zZXRDb250ZW50KGh0bWxfMDUyZDljOWM2NzY0NGUzZjkyMDBlNTlmZGIwMmVkMmMpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfMmJkZmYwZGFhYWQ0NDc5MjlmMjcwNGRhY2I0YmYyNTQuYmluZFBvcHVwKHBvcHVwXzlkMTg1MDkxOWRlNjQ5MTY5ZTQ0ZGQ5MGMwMmEyMjNjKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzIyNGRlMTA2ODFjZjRkMmQ4NGZhYTk0OWI1N2EzZjRmID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMDAxODcsIC04My45MTE1OV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNTk3YjQ3MDEwNWZmNDNhN2I1OGE4NTE5MzAwOWVkM2IgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzg0YTE1YmFhNDgyMjQxZjlhMDBmYjJmNTc0NzdjNzZmID0gJChgPGRpdiBpZD0iaHRtbF84NGExNWJhYTQ4MjI0MWY5YTAwZmIyZjU3NDc3Yzc2ZiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TGEgRXNwZXJhbnphPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzU5N2I0NzAxMDVmZjQzYTdiNThhODUxOTMwMDllZDNiLnNldENvbnRlbnQoaHRtbF84NGExNWJhYTQ4MjI0MWY5YTAwZmIyZjU3NDc3Yzc2Zik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl8yMjRkZTEwNjgxY2Y0ZDJkODRmYWE5NDliNTdhM2Y0Zi5iaW5kUG9wdXAocG9wdXBfNTk3YjQ3MDEwNWZmNDNhN2I1OGE4NTE5MzAwOWVkM2IpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNjAxOTk3NGY2YTUwNDU2Zjg1ZmE3OTBlODU0ZTY1ZDMgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45MjU4MTEsIC04NC4wNDc4MThdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzkwOTQ5ODg1NzYyMTQ4NTk4NTg4NDcyNDY1MzU4ZTFlID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8zM2UyODBlNjUzNDk0YjFjYjI0MTIzOWMxNzE5YTU1MSA9ICQoYDxkaXYgaWQ9Imh0bWxfMzNlMjgwZTY1MzQ5NGIxY2IyNDEyMzljMTcxOWE1NTEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkxhIENhc2EgRGUgQnVycm8gRmxvam88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfOTA5NDk4ODU3NjIxNDg1OTg1ODg0NzI0NjUzNThlMWUuc2V0Q29udGVudChodG1sXzMzZTI4MGU2NTM0OTRiMWNiMjQxMjM5YzE3MTlhNTUxKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzYwMTk5NzRmNmE1MDQ1NmY4NWZhNzkwZTg1NGU2NWQzLmJpbmRQb3B1cChwb3B1cF85MDk0OTg4NTc2MjE0ODU5ODU4ODQ3MjQ2NTM1OGUxZSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl8zOGJiMjEwOWFmN2E0MWMyOGZhMDM0MTNhMTk4MmY3YiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljk5NjEyOCwgLTgzLjkyMzM5MTkxNDYwMDVdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzk0ZDE2MzQ5YzI4MDQ4NmE4NzI0NWQ2NWE1MmYxYTQ2ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9hYzU5ZTJiNDQwMjk0MTdjYjY2N2FjOWIwZmEwMTM1MCA9ICQoYDxkaXYgaWQ9Imh0bWxfYWM1OWUyYjQ0MDI5NDE3Y2I2NjdhYzliMGZhMDEzNTAiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRhcXVlcmlhIExhIEhlcnJhZHVyYTwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF85NGQxNjM0OWMyODA0ODZhODcyNDVkNjVhNTJmMWE0Ni5zZXRDb250ZW50KGh0bWxfYWM1OWUyYjQ0MDI5NDE3Y2I2NjdhYzliMGZhMDEzNTApOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfMzhiYjIxMDlhZjdhNDFjMjhmYTAzNDEzYTE5ODJmN2IuYmluZFBvcHVwKHBvcHVwXzk0ZDE2MzQ5YzI4MDQ4NmE4NzI0NWQ2NWE1MmYxYTQ2KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzgwNmM4N2Q5Y2VjZDQ3NjA4NDY0YTRkZDk1OWNhNmE5ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTY2NzgyMTk0NjM1MSwgLTgzLjkxODk5Njg2ODk4NDRdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzUzMmRmNTg4YmI3YTRmZWE4ZGU1OGFjNDEzOTg2NjA4ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF84MmE3MzU1NzdkYmI0NmRjYWUzODFhNDJhNWNiMGNmZiA9ICQoYDxkaXYgaWQ9Imh0bWxfODJhNzM1NTc3ZGJiNDZkY2FlMzgxYTQyYTVjYjBjZmYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNoaXZvIFRhcXVlcmlhPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzUzMmRmNTg4YmI3YTRmZWE4ZGU1OGFjNDEzOTg2NjA4LnNldENvbnRlbnQoaHRtbF84MmE3MzU1NzdkYmI0NmRjYWUzODFhNDJhNWNiMGNmZik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl84MDZjODdkOWNlY2Q0NzYwODQ2NGE0ZGQ5NTljYTZhOS5iaW5kUG9wdXAocG9wdXBfNTMyZGY1ODhiYjdhNGZlYThkZTU4YWM0MTM5ODY2MDgpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfODQ3Njc0NjY3NTY3NGZhZjgwYjc2OWU5YTg2YTY1YmMgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45NDY3NDAxNjE2MjY0LCAtODQuMTEzNTUzMzUyNjU0XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8yZTViZDM5OTRjNTI0NzdlOTQ5ZmY4MzFlZDg0NTliNSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMmRjNTYyOTY1N2YzNGMxYTlkZmFkODI4OTBiY2RhNzggPSAkKGA8ZGl2IGlkPSJodG1sXzJkYzU2Mjk2NTdmMzRjMWE5ZGZhZDgyODkwYmNkYTc4IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Qb3J0b24gTWV4aWNhbiBLaXRjaGVuPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzJlNWJkMzk5NGM1MjQ3N2U5NDlmZjgzMWVkODQ1OWI1LnNldENvbnRlbnQoaHRtbF8yZGM1NjI5NjU3ZjM0YzFhOWRmYWQ4Mjg5MGJjZGE3OCk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl84NDc2NzQ2Njc1Njc0ZmFmODBiNzY5ZTlhODZhNjViYy5iaW5kUG9wdXAocG9wdXBfMmU1YmQzOTk0YzUyNDc3ZTk0OWZmODMxZWQ4NDU5YjUpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfZmNhNjgzNzBiNzlhNDgyYzk3MDhiOWJmMzA4MGY5ZTcgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS44ODY1NzYyNjM2ODI5LCAtODQuMTUxNDQxNDI0ODY5NV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNjBiNGZmODQ0MmRkNGIxOGI1YjQxNjM2NGRiZWM1YTYgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2U4YzQ1NWNiNGY1MzRhMGY4NDA0ZDUxMzRiMDQyZTA4ID0gJChgPGRpdiBpZD0iaHRtbF9lOGM0NTVjYjRmNTM0YTBmODQwNGQ1MTM0YjA0MmUwOCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UG90cmlsbG9zIFRhcXVlcmlhICZhbXA7IE5ldmVyaWE8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNjBiNGZmODQ0MmRkNGIxOGI1YjQxNjM2NGRiZWM1YTYuc2V0Q29udGVudChodG1sX2U4YzQ1NWNiNGY1MzRhMGY4NDA0ZDUxMzRiMDQyZTA4KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2ZjYTY4MzcwYjc5YTQ4MmM5NzA4YjliZjMwODBmOWU3LmJpbmRQb3B1cChwb3B1cF82MGI0ZmY4NDQyZGQ0YjE4YjViNDE2MzY0ZGJlYzVhNikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl84NDMzNjViNjBjMWE0NDQzODQ1MjFkNjBlNGU1NWEwZiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjAwNDI1MzExMzMxNzMsIC04My44NTYzMjA5ODEzODkzXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF81YjJmMjg1YjYzNTA0YWZkOTBkMTMyMWQ4NzQyNTZkZSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfM2ZlMzlhN2UzN2YzNDE4ODllNzhmNGRjMzljNzc4N2YgPSAkKGA8ZGl2IGlkPSJodG1sXzNmZTM5YTdlMzdmMzQxODg5ZTc4ZjRkYzM5Yzc3ODdmIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5IYWJhbmVyb3M8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNWIyZjI4NWI2MzUwNGFmZDkwZDEzMjFkODc0MjU2ZGUuc2V0Q29udGVudChodG1sXzNmZTM5YTdlMzdmMzQxODg5ZTc4ZjRkYzM5Yzc3ODdmKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzg0MzM2NWI2MGMxYTQ0NDM4NDUyMWQ2MGU0ZTU1YTBmLmJpbmRQb3B1cChwb3B1cF81YjJmMjg1YjYzNTA0YWZkOTBkMTMyMWQ4NzQyNTZkZSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl8yZDlmMGYwY2YyYzk0NTBlYmE3NzU0NDI3Njc1MmY1MyA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljk1MDUzNzI0OTQ0NTksIC04My45MTQyNzM2MDQ3NTA2XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9lM2FjNDlmN2E4NDA0YmNhOTgwOWVkNWQ0N2VlYjc5OSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfZDcxZDQ4ZmM1ZmVjNGE4Zjk4ZTIyNWY2Zjg5MDRjYzQgPSAkKGA8ZGl2IGlkPSJodG1sX2Q3MWQ0OGZjNWZlYzRhOGY5OGUyMjVmNmY4OTA0Y2M0IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5MYXMgRnVlbnRlczwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9lM2FjNDlmN2E4NDA0YmNhOTgwOWVkNWQ0N2VlYjc5OS5zZXRDb250ZW50KGh0bWxfZDcxZDQ4ZmM1ZmVjNGE4Zjk4ZTIyNWY2Zjg5MDRjYzQpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfMmQ5ZjBmMGNmMmM5NDUwZWJhNzc1NDQyNzY3NTJmNTMuYmluZFBvcHVwKHBvcHVwX2UzYWM0OWY3YTg0MDRiY2E5ODA5ZWQ1ZDQ3ZWViNzk5KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzc2MTM5OGZmYWY2MDRmOTA5MzE1ZjU3MzZiYjhhMzMwID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTMyMzMsIC04NC4wMTMzOV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfZDk5YTIxOGYzZjk0NDBkN2FlMjY0OTMzNmFlYmY2YTAgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzgwNjE4MDM1N2NhMjQ3YjI4MDk1ZGI4NDkyODVmMTA1ID0gJChgPGRpdiBpZD0iaHRtbF84MDYxODAzNTdjYTI0N2IyODA5NWRiODQ5Mjg1ZjEwNSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U29jY2VyIFRhY288L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfZDk5YTIxOGYzZjk0NDBkN2FlMjY0OTMzNmFlYmY2YTAuc2V0Q29udGVudChodG1sXzgwNjE4MDM1N2NhMjQ3YjI4MDk1ZGI4NDkyODVmMTA1KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzc2MTM5OGZmYWY2MDRmOTA5MzE1ZjU3MzZiYjhhMzMwLmJpbmRQb3B1cChwb3B1cF9kOTlhMjE4ZjNmOTQ0MGQ3YWUyNjQ5MzM2YWViZjZhMCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl8wYjk5MmVmODU3NDU0Mjk0OWQ3OGM5ZWE5YTRjYzdmNiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1LjkyMjUwODIzOTc0NjEsIC04NC4wNTI1NjY1MjgzMjAzXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8yOTlhMWEyZWFhMmE0ZTk5ODFlOTVhOThlZDlmMjI1OSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfZWRkNDdmYzlkOTExNGViNzljY2VmNTllNDM4OTA2MjUgPSAkKGA8ZGl2IGlkPSJodG1sX2VkZDQ3ZmM5ZDkxMTRlYjc5Y2NlZjU5ZTQzODkwNjI1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5NaSBQdWVibG88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMjk5YTFhMmVhYTJhNGU5OTgxZTk1YTk4ZWQ5ZjIyNTkuc2V0Q29udGVudChodG1sX2VkZDQ3ZmM5ZDkxMTRlYjc5Y2NlZjU5ZTQzODkwNjI1KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzBiOTkyZWY4NTc0NTQyOTQ5ZDc4YzllYTlhNGNjN2Y2LmJpbmRQb3B1cChwb3B1cF8yOTlhMWEyZWFhMmE0ZTk5ODFlOTVhOThlZDlmMjI1OSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl8xNjBkNjBmMGVhODE0NDU2OTk3MThlODY2NjRjNjMwZSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljk1MDA5LCAtODQuMTU1MjNdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2FjNjFiMGYxYWU4MTRjNmJiOGFlODU2YmFmNjUxNTNlID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF83YWVkNDU3Y2Q4MGE0ZGE2YTFiZGFmZDlhMzY3YTVkMSA9ICQoYDxkaXYgaWQ9Imh0bWxfN2FlZDQ1N2NkODBhNGRhNmExYmRhZmQ5YTM2N2E1ZDEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNvdW50cnkgQnVycml0byBGcmVzaCBNZXg8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfYWM2MWIwZjFhZTgxNGM2YmI4YWU4NTZiYWY2NTE1M2Uuc2V0Q29udGVudChodG1sXzdhZWQ0NTdjZDgwYTRkYTZhMWJkYWZkOWEzNjdhNWQxKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzE2MGQ2MGYwZWE4MTQ0NTY5OTcxOGU4NjY2NGM2MzBlLmJpbmRQb3B1cChwb3B1cF9hYzYxYjBmMWFlODE0YzZiYjhhZTg1NmJhZjY1MTUzZSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl84YzczMzY5M2YwN2E0MGFjYTAyOWMyNTgzMjRiYmI1MyA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1LjkxOTg2LCAtODQuMDUwNzldLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2Q4ODQwNjM2NGEzNTQxMjA4YTU0YzcwOTgxYjg3Y2NjID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF83ZjM0ZTFjNWMyODA0ZGJkODdlOTk1ZjUwZDI0NTIwZCA9ICQoYDxkaXYgaWQ9Imh0bWxfN2YzNGUxYzVjMjgwNGRiZDg3ZTk5NWY1MGQyNDUyMGQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlBlbGFuY2hvcyBNZXhpY2FuIEdyaWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2Q4ODQwNjM2NGEzNTQxMjA4YTU0YzcwOTgxYjg3Y2NjLnNldENvbnRlbnQoaHRtbF83ZjM0ZTFjNWMyODA0ZGJkODdlOTk1ZjUwZDI0NTIwZCk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl84YzczMzY5M2YwN2E0MGFjYTAyOWMyNTgzMjRiYmI1My5iaW5kUG9wdXAocG9wdXBfZDg4NDA2MzY0YTM1NDEyMDhhNTRjNzA5ODFiODdjY2MpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfZTZlODVkM2I1NjUzNDQ4M2E3MDgxOWE0NDYxODdhNjUgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45MjUyMSwgLTg0LjA5MzM3N10sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNjk4N2ZlYzg3ODAzNDMzNGFmY2YyODdlOTk1ODZjMzEgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzdlNWRkNWY5ZDdiMzRlMzFhNzg5YWE1OGVkZmE0OWYxID0gJChgPGRpdiBpZD0iaHRtbF83ZTVkZDVmOWQ3YjM0ZTMxYTc4OWFhNThlZGZhNDlmMSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TWV4aWNvIExpbmRvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzY5ODdmZWM4NzgwMzQzMzRhZmNmMjg3ZTk5NTg2YzMxLnNldENvbnRlbnQoaHRtbF83ZTVkZDVmOWQ3YjM0ZTMxYTc4OWFhNThlZGZhNDlmMSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9lNmU4NWQzYjU2NTM0NDgzYTcwODE5YTQ0NjE4N2E2NS5iaW5kUG9wdXAocG9wdXBfNjk4N2ZlYzg3ODAzNDMzNGFmY2YyODdlOTk1ODZjMzEpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfMzQ3MmVmNDY4MzYwNDNkNTlhNDFkOGE3ZWFhNTFiMGQgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45NTQ1NTYsIC04My45Mzg1XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9iZDI3Y2Q1M2VlNDI0ODJhYWVmODkwMGFiZmFmMDdiZSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNGVmMmE4ZmE4YTVlNDIwYzkxMGU0YjhjMzEyNDUyNDEgPSAkKGA8ZGl2IGlkPSJodG1sXzRlZjJhOGZhOGE1ZTQyMGM5MTBlNGI4YzMxMjQ1MjQxIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5WaWN0b3ImIzM5O3MgVGFjbyBTaG9wPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2JkMjdjZDUzZWU0MjQ4MmFhZWY4OTAwYWJmYWYwN2JlLnNldENvbnRlbnQoaHRtbF80ZWYyYThmYThhNWU0MjBjOTEwZTRiOGMzMTI0NTI0MSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl8zNDcyZWY0NjgzNjA0M2Q1OWE0MWQ4YTdlYWE1MWIwZC5iaW5kUG9wdXAocG9wdXBfYmQyN2NkNTNlZTQyNDgyYWFlZjg5MDBhYmZhZjA3YmUpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfY2FhYmIxMjY1ZjIyNDY0NmFlOWY2ZmViMTI5NjM4YzMgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45NDQ1NiwgLTgzLjk4Mjc1XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF80NjY0MWE4MjA0NTA0ZTY1YTIzYjEzMjY2YWQwOWM3YSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMWI0NDVjNjJmY2IzNDU3YmE3YjczMTliYWQ1NTNmZDUgPSAkKGA8ZGl2IGlkPSJodG1sXzFiNDQ1YzYyZmNiMzQ1N2JhN2I3MzE5YmFkNTUzZmQ1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5FbCBDaGFycm88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNDY2NDFhODIwNDUwNGU2NWEyM2IxMzI2NmFkMDljN2Euc2V0Q29udGVudChodG1sXzFiNDQ1YzYyZmNiMzQ1N2JhN2I3MzE5YmFkNTUzZmQ1KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2NhYWJiMTI2NWYyMjQ2NDZhZTlmNmZlYjEyOTYzOGMzLmJpbmRQb3B1cChwb3B1cF80NjY0MWE4MjA0NTA0ZTY1YTIzYjEzMjY2YWQwOWM3YSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl84NGNhNjYyOTY1Zjg0MTdmOWY4NDBlZTU5ZDFjY2FlZSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljg5NzE2MzY2ODAyMjksIC04NC4xNzU4NDM5OTk5NzI2XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8wMTJjNDg5YWVlMTg0NTNjYmNhZTYzNmZmZGY1OTVjZCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMDNiZjhlNWY3NWY5NDgyYzk0OThiYjJkMGE4NjJjNmQgPSAkKGA8ZGl2IGlkPSJodG1sXzAzYmY4ZTVmNzVmOTQ4MmM5NDk4YmIyZDBhODYyYzZkIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UYWNvIEJveTwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8wMTJjNDg5YWVlMTg0NTNjYmNhZTYzNmZmZGY1OTVjZC5zZXRDb250ZW50KGh0bWxfMDNiZjhlNWY3NWY5NDgyYzk0OThiYjJkMGE4NjJjNmQpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfODRjYTY2Mjk2NWY4NDE3ZjlmODQwZWU1OWQxY2NhZWUuYmluZFBvcHVwKHBvcHVwXzAxMmM0ODlhZWUxODQ1M2NiY2FlNjM2ZmZkZjU5NWNkKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2YxNzk0OGEzZTFkYTRmMmRhODg2NjU1ZmY4YTAyMTViID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTQxODgzMDg3MTU4MiwgLTgzLjk4NDY4MDE3NTc4MTJdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2IyODg5YmQ1MWU4ZDQxZDhiZmY4ZDQ4ODg5NmU3MjgxID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF81MDNhNmU4YzJmMWM0NTc2ODYzMzliNTcxNzQ3MGUzNiA9ICQoYDxkaXYgaWQ9Imh0bWxfNTAzYTZlOGMyZjFjNDU3Njg2MzM5YjU3MTc0NzBlMzYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkVsIEdpcmFzb2w8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfYjI4ODliZDUxZThkNDFkOGJmZjhkNDg4ODk2ZTcyODEuc2V0Q29udGVudChodG1sXzUwM2E2ZThjMmYxYzQ1NzY4NjMzOWI1NzE3NDcwZTM2KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2YxNzk0OGEzZTFkYTRmMmRhODg2NjU1ZmY4YTAyMTViLmJpbmRQb3B1cChwb3B1cF9iMjg4OWJkNTFlOGQ0MWQ4YmZmOGQ0ODg4OTZlNzI4MSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl8xZTY3NDAxNDY1M2Q0YTZlOGY0NTNiZWYwM2JiMzY4NiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljk3OTA4NzgsIC04NC4wMTQxNjc4XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF81ODFmZDA2ZDdmYTc0Njc3YmFlOWY4YWVhNTM3Y2EzMSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMDVmYTQ0YWExNjJlNDc2ZmE2NmM5M2ZkM2ZiZmQ3ODYgPSAkKGA8ZGl2IGlkPSJodG1sXzA1ZmE0NGFhMTYyZTQ3NmZhNjZjOTNmZDNmYmZkNzg2IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5MYSBGaWVzdGEgTWV4aWNhbiBSZXN0YXVyYW50ICZhbXA7IEdyaWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzU4MWZkMDZkN2ZhNzQ2NzdiYWU5ZjhhZWE1MzdjYTMxLnNldENvbnRlbnQoaHRtbF8wNWZhNDRhYTE2MmU0NzZmYTY2YzkzZmQzZmJmZDc4Nik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl8xZTY3NDAxNDY1M2Q0YTZlOGY0NTNiZWYwM2JiMzY4Ni5iaW5kUG9wdXAocG9wdXBfNTgxZmQwNmQ3ZmE3NDY3N2JhZTlmOGFlYTUzN2NhMzEpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfOTMyNjQwMTdjZmFhNGIyMmFmMTYxOTU5MDY1ZDg0ZjMgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45MTE5MSwgLTg0LjA4ODU2XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF81MDM0MTc1YTFhMjA0Y2ZjOWU3MjEwZTE1ZDEzMzkyNyA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfZGYzZTk2ODBiOTYzNGE2OWEzMDlhMDM1YTljNDIwZjAgPSAkKGA8ZGl2IGlkPSJodG1sX2RmM2U5NjgwYjk2MzRhNjlhMzA5YTAzNWE5YzQyMGYwIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DaHV5JiMzOTtzPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzUwMzQxNzVhMWEyMDRjZmM5ZTcyMTBlMTVkMTMzOTI3LnNldENvbnRlbnQoaHRtbF9kZjNlOTY4MGI5NjM0YTY5YTMwOWEwMzVhOWM0MjBmMCk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl85MzI2NDAxN2NmYWE0YjIyYWYxNjE5NTkwNjVkODRmMy5iaW5kUG9wdXAocG9wdXBfNTAzNDE3NWExYTIwNGNmYzllNzIxMGUxNWQxMzM5MjcpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNzFhZTAxN2UxY2NmNDE2MDkxOTllZDk5MjE2YTQ3Y2EgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNi4wMDYyNDEsIC04NC4wMjAzNzddLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzBmNjNkZjA5MjM0NzRhMDBhODc1OTczYjBmNmE1MjllID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8wYTFlZmYyYzczZGM0NGRmYTEwY2NiMTUwMDkwOGI4OSA9ICQoYDxkaXYgaWQ9Imh0bWxfMGExZWZmMmM3M2RjNDRkZmExMGNjYjE1MDA5MDhiODkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkxhIFBhbG1hIERlIE9ybzwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8wZjYzZGYwOTIzNDc0YTAwYTg3NTk3M2IwZjZhNTI5ZS5zZXRDb250ZW50KGh0bWxfMGExZWZmMmM3M2RjNDRkZmExMGNjYjE1MDA5MDhiODkpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNzFhZTAxN2UxY2NmNDE2MDkxOTllZDk5MjE2YTQ3Y2EuYmluZFBvcHVwKHBvcHVwXzBmNjNkZjA5MjM0NzRhMDBhODc1OTczYjBmNmE1MjllKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2Q3YzFlYmU4NTg0YzQ2MzdiMTNiM2M0YjQzNWUxOWFmID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTY1OTksIC04My45MTg1NTNdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2ZlZmYwMGU3YzMwYjRkMzg4YTFhNzBkNGYxMTljYzM3ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9hY2I4ZmVjYmI2NTU0ZWJiODZlNDJhM2NjY2EyMDZhZiA9ICQoYDxkaXYgaWQ9Imh0bWxfYWNiOGZlY2JiNjU1NGViYjg2ZTQyYTNjY2NhMjA2YWYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJhYmFsdSBUYXBhcyAmYW1wOyBUYWNvczwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9mZWZmMDBlN2MzMGI0ZDM4OGExYTcwZDRmMTE5Y2MzNy5zZXRDb250ZW50KGh0bWxfYWNiOGZlY2JiNjU1NGViYjg2ZTQyYTNjY2NhMjA2YWYpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfZDdjMWViZTg1ODRjNDYzN2IxM2IzYzRiNDM1ZTE5YWYuYmluZFBvcHVwKHBvcHVwX2ZlZmYwMGU3YzMwYjRkMzg4YTFhNzBkNGYxMTljYzM3KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzc2Zjk4NDZkNmNkMTQzNDY5ODAwZTFhMGE0MjhkYmQ5ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTAwNTQ0MDkyMTEwNywgLTg0LjE1ODAxMTU1NDkwMTFdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzhjNjllODZiZjFhNDRlZDg5NmUwODkyNTJjZWI5YzM0ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9hYjU5YjQ4MDlhMTE0ZWIxOTk1NjI4YWNiZTE5NmE1MiA9ICQoYDxkaXYgaWQ9Imh0bWxfYWI1OWI0ODA5YTExNGViMTk5NTYyOGFjYmUxOTZhNTIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkFidWVsbyYjMzk7czwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF84YzY5ZTg2YmYxYTQ0ZWQ4OTZlMDg5MjUyY2ViOWMzNC5zZXRDb250ZW50KGh0bWxfYWI1OWI0ODA5YTExNGViMTk5NTYyOGFjYmUxOTZhNTIpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNzZmOTg0NmQ2Y2QxNDM0Njk4MDBlMWEwYTQyOGRiZDkuYmluZFBvcHVwKHBvcHVwXzhjNjllODZiZjFhNDRlZDg5NmUwODkyNTJjZWI5YzM0KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzkzZDM4M2JlMDhjZDQ1ZDE5ZTYxODg1MWIyYTk5NGNlID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMDEzMDMxMDA1ODU5NCwgLTgzLjk2ODEwOTEzMDg1OTRdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2NlNWZkZDNkNmNhYTQwYjNhMTg3YzQ4N2EzMWYwZDU5ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9jMGEwMmMwNTVjZTY0MWEyOTZkMTcwYzRmM2RlYThiNCA9ICQoYDxkaXYgaWQ9Imh0bWxfYzBhMDJjMDU1Y2U2NDFhMjk2ZDE3MGM0ZjNkZWE4YjQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkVsIENoaWNvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2NlNWZkZDNkNmNhYTQwYjNhMTg3YzQ4N2EzMWYwZDU5LnNldENvbnRlbnQoaHRtbF9jMGEwMmMwNTVjZTY0MWEyOTZkMTcwYzRmM2RlYThiNCk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl85M2QzODNiZTA4Y2Q0NWQxOWU2MTg4NTFiMmE5OTRjZS5iaW5kUG9wdXAocG9wdXBfY2U1ZmRkM2Q2Y2FhNDBiM2ExODdjNDg3YTMxZjBkNTkpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNzljYWRlNDUxNDI1NGI4MTk0ZjkyM2MwYzNmMzQ5ZTkgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS44NjM5LCAtODQuMDY0NTddLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzQ5NTU0NGQ1ODY3ZjQ1NmE4MDdhYmUyOGUwMmE0MzcxID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9lMWRkYjI5NWQ1NDA0Y2U2YjJjOTZmMzE4NDNkMGExMyA9ICQoYDxkaXYgaWQ9Imh0bWxfZTFkZGIyOTVkNTQwNGNlNmIyYzk2ZjMxODQzZDBhMTMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk8gQ2h1bG9zIEdyaWxsIGFuZCBCYXI8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNDk1NTQ0ZDU4NjdmNDU2YTgwN2FiZTI4ZTAyYTQzNzEuc2V0Q29udGVudChodG1sX2UxZGRiMjk1ZDU0MDRjZTZiMmM5NmYzMTg0M2QwYTEzKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzc5Y2FkZTQ1MTQyNTRiODE5NGY5MjNjMGMzZjM0OWU5LmJpbmRQb3B1cChwb3B1cF80OTU1NDRkNTg2N2Y0NTZhODA3YWJlMjhlMDJhNDM3MSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl85M2RkOWQyZjhlNGI0YjBhODJjMTU2Mjg2MGY4YTQ5MCA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjAzMDQ3MTgwMTc1NzgsIC04My44ODQ4NDE5MTg5NDUzXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF83YjZlNTM4MjhmN2E0ZmU2YWM5OTQ4MjViNzgwY2I2YyA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNGMyM2ZmYTk2ZTFiNGQwNTlhNTQzZTQ2NjM1ZTg1MDIgPSAkKGA8ZGl2IGlkPSJodG1sXzRjMjNmZmE5NmUxYjRkMDU5YTU0M2U0NjYzNWU4NTAyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5BbWlnbyYjMzk7cyAmYW1wOyBCZWVyIE1leGljYW4gR3JpbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfN2I2ZTUzODI4ZjdhNGZlNmFjOTk0ODI1Yjc4MGNiNmMuc2V0Q29udGVudChodG1sXzRjMjNmZmE5NmUxYjRkMDU5YTU0M2U0NjYzNWU4NTAyKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzkzZGQ5ZDJmOGU0YjRiMGE4MmMxNTYyODYwZjhhNDkwLmJpbmRQb3B1cChwb3B1cF83YjZlNTM4MjhmN2E0ZmU2YWM5OTQ4MjViNzgwY2I2YykKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9mNmIzMzRmZDFlYTI0MDRiYWFlNWQ4ZjM3Mzc0ZWY0ZSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjAwMDc4LCAtODMuNzc0NDldLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzFmOGJkY2MxM2U4NTRmODU5MDczNTcwNTRhMjBmOTAwID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9mMTk5N2M4ODdhZjI0ZWE4OGMyMWUwY2Y3YWE1YTk4OSA9ICQoYDxkaXYgaWQ9Imh0bWxfZjE5OTdjODg3YWYyNGVhODhjMjFlMGNmN2FhNWE5ODkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkFtaWdv4oCZcyBNZXhpY2FuIEdyaWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzFmOGJkY2MxM2U4NTRmODU5MDczNTcwNTRhMjBmOTAwLnNldENvbnRlbnQoaHRtbF9mMTk5N2M4ODdhZjI0ZWE4OGMyMWUwY2Y3YWE1YTk4OSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9mNmIzMzRmZDFlYTI0MDRiYWFlNWQ4ZjM3Mzc0ZWY0ZS5iaW5kUG9wdXAocG9wdXBfMWY4YmRjYzEzZTg1NGY4NTkwNzM1NzA1NGEyMGY5MDApCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNWI4ZDFiZDFkMjQ0NDBlMzgwNmVlZTdmODY1NzAwMTEgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45OTIxODI0NDc2NzQ0LCAtODMuOTE5OTAyODkwOTIwNl0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfYzRmN2QyYTE4NzA3NDFmZjliM2YwODRkM2U1NTdmNmUgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzBmZjVkZDdlNGJiMTRmMTdhZGJlMzg3ZWJlMGY2NDNhID0gJChgPGRpdiBpZD0iaHRtbF8wZmY1ZGQ3ZTRiYjE0ZjE3YWRiZTM4N2ViZTBmNjQzYSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TGEgUGnDsWF0YSBNZXhpY2FuIEJhciAmYW1wOyBHcmlsbDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9jNGY3ZDJhMTg3MDc0MWZmOWIzZjA4NGQzZTU1N2Y2ZS5zZXRDb250ZW50KGh0bWxfMGZmNWRkN2U0YmIxNGYxN2FkYmUzODdlYmUwZjY0M2EpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNWI4ZDFiZDFkMjQ0NDBlMzgwNmVlZTdmODY1NzAwMTEuYmluZFBvcHVwKHBvcHVwX2M0ZjdkMmExODcwNzQxZmY5YjNmMDg0ZDNlNTU3ZjZlKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2Y2NGM2Yjc4NmRmNzQ2MDY4YmUyNGEzNDMxY2I2YTM4ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMDAyMjEsIC04My45MjY1OV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMmNjNjI3OTBjNjg0NDhiNDg5MDRhYmRjNDA4Y2M1OWIgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzI0ZmYzOTIyZTMyMDRiYmVhNmJkMmJhMGE5ZTBlNWY5ID0gJChgPGRpdiBpZD0iaHRtbF8yNGZmMzkyMmUzMjA0YmJlYTZiZDJiYTBhOWUwZTVmOSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U2Vub3IgVGFjbzwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8yY2M2Mjc5MGM2ODQ0OGI0ODkwNGFiZGM0MDhjYzU5Yi5zZXRDb250ZW50KGh0bWxfMjRmZjM5MjJlMzIwNGJiZWE2YmQyYmEwYTllMGU1ZjkpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfZjY0YzZiNzg2ZGY3NDYwNjhiZTI0YTM0MzFjYjZhMzguYmluZFBvcHVwKHBvcHVwXzJjYzYyNzkwYzY4NDQ4YjQ4OTA0YWJkYzQwOGNjNTliKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzUwNmUxOThkZDBlZjRjODM4MGY2NzZkODQxM2I0MWJiID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMDEzNDMyNCwgLTg0LjI1NzEwMjFdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzBkMmQwNDNlYzVlMTQ2YWZiZjgwNWZmODcyZjBiMDU1ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF80YTUwZjc0MjJkNmE0M2Q4YmM4YWNkZjgwMDc4NmFmOSA9ICQoYDxkaXYgaWQ9Imh0bWxfNGE1MGY3NDIyZDZhNDNkOGJjOGFjZGY4MDA3ODZhZjkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkVsIENhbnRhcml0byBNZXhpY2FuIFJlc3RhdXJhbnQ8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMGQyZDA0M2VjNWUxNDZhZmJmODA1ZmY4NzJmMGIwNTUuc2V0Q29udGVudChodG1sXzRhNTBmNzQyMmQ2YTQzZDhiYzhhY2RmODAwNzg2YWY5KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzUwNmUxOThkZDBlZjRjODM4MGY2NzZkODQxM2I0MWJiLmJpbmRQb3B1cChwb3B1cF8wZDJkMDQzZWM1ZTE0NmFmYmY4MDVmZjg3MmYwYjA1NSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl82ZGI3OTcwNTY4NjY0ODQyODJhYjVkZjA3MzgwNDE3YiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljk0MTk2LCAtODMuOTgxMjQ3XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF82MWFlMDIyNmUyZDQ0ZDIzYjg1ZTRkMzVkNDRkZWMyNiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMTdkZDQyNjUzZmEwNDc5MDhhZjA0NjkyYWU0NTgyZTYgPSAkKGA8ZGl2IGlkPSJodG1sXzE3ZGQ0MjY1M2ZhMDQ3OTA4YWYwNDY5MmFlNDU4MmU2IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5FbCBNZXpjYWwgTWV4aWNhbiBSZXN0YXVyYW50PC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzYxYWUwMjI2ZTJkNDRkMjNiODVlNGQzNWQ0NGRlYzI2LnNldENvbnRlbnQoaHRtbF8xN2RkNDI2NTNmYTA0NzkwOGFmMDQ2OTJhZTQ1ODJlNik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl82ZGI3OTcwNTY4NjY0ODQyODJhYjVkZjA3MzgwNDE3Yi5iaW5kUG9wdXAocG9wdXBfNjFhZTAyMjZlMmQ0NGQyM2I4NWU0ZDM1ZDQ0ZGVjMjYpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNTVlYWIxY2JmNjNhNGQwMWI5YjgxZjM3NDhiNTQwMjkgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45MDIxNjg2LCAtODQuMDIyNjc1NF0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNWQwNzIyYmRiZjUyNGU2Mjg1YWU3ODdlMDhjZDViYTAgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzhlNjAxMDhiY2NmMjRiMjVhMDcyYTA5YTQxNTViMzNlID0gJChgPGRpdiBpZD0iaHRtbF84ZTYwMTA4YmNjZjI0YjI1YTA3MmEwOWE0MTU1YjMzZSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2FzYSBEb24gR2FsbG88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNWQwNzIyYmRiZjUyNGU2Mjg1YWU3ODdlMDhjZDViYTAuc2V0Q29udGVudChodG1sXzhlNjAxMDhiY2NmMjRiMjVhMDcyYTA5YTQxNTViMzNlKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzU1ZWFiMWNiZjYzYTRkMDFiOWI4MWYzNzQ4YjU0MDI5LmJpbmRQb3B1cChwb3B1cF81ZDA3MjJiZGJmNTI0ZTYyODVhZTc4N2UwOGNkNWJhMCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9lODhiNzQ1MzUwNDc0MDUzOTkwNjBjMWNiMWMxYWJmYiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjAyNDM4NSwgLTg0LjIzMDc4MV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMWVjNTdlMjExMmU1NDdiY2I2YTI3Y2U3MDJjMGY3MzQgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzZmMTEyNjdiMDM2MTRjNTc4MmVlZWNmOGZiNWE5ODUxID0gJChgPGRpdiBpZD0iaHRtbF82ZjExMjY3YjAzNjE0YzU3ODJlZWVjZjhmYjVhOTg1MSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RWwgSmluZXRlIE1leGljYW48L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMWVjNTdlMjExMmU1NDdiY2I2YTI3Y2U3MDJjMGY3MzQuc2V0Q29udGVudChodG1sXzZmMTEyNjdiMDM2MTRjNTc4MmVlZWNmOGZiNWE5ODUxKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2U4OGI3NDUzNTA0NzQwNTM5OTA2MGMxY2IxYzFhYmZiLmJpbmRQb3B1cChwb3B1cF8xZWM1N2UyMTEyZTU0N2JjYjZhMjdjZTcwMmMwZjczNCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl8xMTdmMGM2NDliOGI0MTM2YjI0MzdhMDJhZGM5ZjZlOSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljc5OTg0LCAtODMuOTQ1MTJdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2NmNTFiOWI0ZjFiYzQ4ZTZiZGM1NDkwNDMwMjY0ZTVmID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9mMWQ0NWJiZmRhOGY0ZWYwOTNkNWU0ZjdkYmZiMmVjOSA9ICQoYDxkaXYgaWQ9Imh0bWxfZjFkNDViYmZkYThmNGVmMDkzZDVlNGY3ZGJmYjJlYzkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNoYXB1bGluZSYjMzk7cyBTdHJlZVRhY29zPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2NmNTFiOWI0ZjFiYzQ4ZTZiZGM1NDkwNDMwMjY0ZTVmLnNldENvbnRlbnQoaHRtbF9mMWQ0NWJiZmRhOGY0ZWYwOTNkNWU0ZjdkYmZiMmVjOSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl8xMTdmMGM2NDliOGI0MTM2YjI0MzdhMDJhZGM5ZjZlOS5iaW5kUG9wdXAocG9wdXBfY2Y1MWI5YjRmMWJjNDhlNmJkYzU0OTA0MzAyNjRlNWYpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNjk0ODg5YWI5YTE4NGUzMjgwYWQwMWMwMTk0ZDU4ZmQgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45MjM2MywgLTg0LjA1MDA5XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9jNzVmNmY5Y2M4ZWI0MzFhYWUyNTZlZjFmYTA4YzRjMSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNWMyYzNlNjkzMzExNDZiOGE0ZjAxMjY0NGRlZTEzMDcgPSAkKGA8ZGl2IGlkPSJodG1sXzVjMmMzZTY5MzMxMTQ2YjhhNGYwMTI2NDRkZWUxMzA3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5MZW1waXJhIE1leGljYW4gUmVzdGF1cmFudCAmYW1wOyBMYXRpbiBDdWlzaW5lPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2M3NWY2ZjljYzhlYjQzMWFhZTI1NmVmMWZhMDhjNGMxLnNldENvbnRlbnQoaHRtbF81YzJjM2U2OTMzMTE0NmI4YTRmMDEyNjQ0ZGVlMTMwNyk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl82OTQ4ODlhYjlhMTg0ZTMyODBhZDAxYzAxOTRkNThmZC5iaW5kUG9wdXAocG9wdXBfYzc1ZjZmOWNjOGViNDMxYWFlMjU2ZWYxZmEwOGM0YzEpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfZDAwZjE1MzIxYjNjNDZhM2FmZjcwOTFkOGExNWI0MTggPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNi4wMTI0OTY5NDgyNDIyLCAtODMuOTY5OTkzNTkxMzA4Nl0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNTg0MmFlODNlZjY0NDc2MDgzNTczMDA1MDE3ODRhYzIgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2Y4ZjQ2ZmE3ODYwMzRlNDlhYTRiZmI3OGY4MzQyZDNkID0gJChgPGRpdiBpZD0iaHRtbF9mOGY0NmZhNzg2MDM0ZTQ5YWE0YmZiNzhmODM0MmQzZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TW9udGVycmV5IE1leGljYW4gUmVzdGF1cmFudDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF81ODQyYWU4M2VmNjQ0NzYwODM1NzMwMDUwMTc4NGFjMi5zZXRDb250ZW50KGh0bWxfZjhmNDZmYTc4NjAzNGU0OWFhNGJmYjc4ZjgzNDJkM2QpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfZDAwZjE1MzIxYjNjNDZhM2FmZjcwOTFkOGExNWI0MTguYmluZFBvcHVwKHBvcHVwXzU4NDJhZTgzZWY2NDQ3NjA4MzU3MzAwNTAxNzg0YWMyKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzg0MWUwOTI5NDUwYjRiZTE5MjlmZDQ2YmM2YzA3YmMyID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMDA2MjksIC04My45ODcwNl0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfZWU3ODY0Y2Q4OWM1NGY3ZmE1MDNkM2ZhYjUzN2E0YjUgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2UzOWQ1ZDFjMWJmMzQzN2NiZWFlOTc3YTEyZTAwYmU2ID0gJChgPGRpdiBpZD0iaHRtbF9lMzlkNWQxYzFiZjM0MzdjYmVhZTk3N2ExMmUwMGJlNiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGFxdWVyaWEgTGEgQ2VudHJhbDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9lZTc4NjRjZDg5YzU0ZjdmYTUwM2QzZmFiNTM3YTRiNS5zZXRDb250ZW50KGh0bWxfZTM5ZDVkMWMxYmYzNDM3Y2JlYWU5NzdhMTJlMDBiZTYpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfODQxZTA5Mjk0NTBiNGJlMTkyOWZkNDZiYzZjMDdiYzIuYmluZFBvcHVwKHBvcHVwX2VlNzg2NGNkODljNTRmN2ZhNTAzZDNmYWI1MzdhNGI1KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzI0MzA4YzgwMWU0MDQ2OGY4MWZiMTgyODFmZWNiMGE4ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTQ0NzkwOTI1Nzk5OSwgLTgzLjg5MDQ0Nzc2NzQyN10sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNzA0NDdiZjBiMmQxNDE3MTkwNTRiZGYwZDIyMjA3NTkgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzE0YTUxMzZlYzk1NjQxYzRhMDUzYjY2YTcxMzZlODFhID0gJChgPGRpdiBpZD0iaHRtbF8xNGE1MTM2ZWM5NTY0MWM0YTA1M2I2NmE3MTM2ZTgxYSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U29Lbm8gVGFjbyBDYW50aW5hPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzcwNDQ3YmYwYjJkMTQxNzE5MDU0YmRmMGQyMjIwNzU5LnNldENvbnRlbnQoaHRtbF8xNGE1MTM2ZWM5NTY0MWM0YTA1M2I2NmE3MTM2ZTgxYSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl8yNDMwOGM4MDFlNDA0NjhmODFmYjE4MjgxZmVjYjBhOC5iaW5kUG9wdXAocG9wdXBfNzA0NDdiZjBiMmQxNDE3MTkwNTRiZGYwZDIyMjA3NTkpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfOTk4Zjg4MDM5MDk1NDEwZWEzZTE2YzgzNmI2MTMxMTQgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45NzExNzk4NTExNDQ3LCAtODMuOTIwMjE3NjEyNDUwNl0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfYTMzNzNlYTdjNGUzNDUwY2JlM2VlMjI3ZmFlZGY2NGIgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2I3ZmJmZjA3OGIzMjQ2Y2VhOTU5YWYxMjRiMTIwYmZjID0gJChgPGRpdiBpZD0iaHRtbF9iN2ZiZmYwNzhiMzI0NmNlYTk1OWFmMTI0YjEyMGJmYyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGFrbyBUYWNvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2EzMzczZWE3YzRlMzQ1MGNiZTNlZTIyN2ZhZWRmNjRiLnNldENvbnRlbnQoaHRtbF9iN2ZiZmYwNzhiMzI0NmNlYTk1OWFmMTI0YjEyMGJmYyk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl85OThmODgwMzkwOTU0MTBlYTNlMTZjODM2YjYxMzExNC5iaW5kUG9wdXAocG9wdXBfYTMzNzNlYTdjNGUzNDUwY2JlM2VlMjI3ZmFlZGY2NGIpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNGRjNWY1M2JmMzZjNDFkYTg3NDAxNjM5NTlhNWUyZTMgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45NjU0NTg1LCAtODMuOTIwMDE0XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9mNzUwMTQ2MDM2M2Y0YzBkOTVjODAzMDNlYWI1MzE3YSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMDRmZmJhYzNkNzE1NDZlYjhjY2M4NTE2YjMwNGYyMDggPSAkKGA8ZGl2IGlkPSJodG1sXzA0ZmZiYWMzZDcxNTQ2ZWI4Y2NjODUxNmIzMDRmMjA4IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Tb2NjZXIgVGFjbzwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9mNzUwMTQ2MDM2M2Y0YzBkOTVjODAzMDNlYWI1MzE3YS5zZXRDb250ZW50KGh0bWxfMDRmZmJhYzNkNzE1NDZlYjhjY2M4NTE2YjMwNGYyMDgpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNGRjNWY1M2JmMzZjNDFkYTg3NDAxNjM5NTlhNWUyZTMuYmluZFBvcHVwKHBvcHVwX2Y3NTAxNDYwMzYzZjRjMGQ5NWM4MDMwM2VhYjUzMTdhKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzkyZTM4NmRmNmU3ODQ2ZmU4NzIwZTkwMmRiNGY1ZWE2ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuNzU3MzM5NSwgLTgzLjk3Mzc3NzhdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2MwMzNhNTdhMmZlZDRjMTNhOGZiN2I4Mjg3NjZiZTlmID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8wNjEwNTJlODU5YjQ0MTFlODRkYTE5Njc4NzgzMjkxZCA9ICQoYDxkaXYgaWQ9Imh0bWxfMDYxMDUyZTg1OWI0NDExZTg0ZGExOTY3ODc4MzI5MWQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkxvcyBBbWlnb3MgPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2MwMzNhNTdhMmZlZDRjMTNhOGZiN2I4Mjg3NjZiZTlmLnNldENvbnRlbnQoaHRtbF8wNjEwNTJlODU5YjQ0MTFlODRkYTE5Njc4NzgzMjkxZCk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl85MmUzODZkZjZlNzg0NmZlODcyMGU5MDJkYjRmNWVhNi5iaW5kUG9wdXAocG9wdXBfYzAzM2E1N2EyZmVkNGMxM2E4ZmI3YjgyODc2NmJlOWYpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNTA0YTFkODJhOGM1NGFiN2E0MDU4NmU2MmM3M2FjM2EgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45MDc3MTI4LCAtODMuODQxMjQ2NF0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfY2MyMTAxYTkyZTRiNDcxYzkwZDQxMmIzMWU4MjZmNGIgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2M0YzQyODQ5YzJlNjQyZmQ5OGNmZmNhNjM0YTdmMGU5ID0gJChgPGRpdiBpZD0iaHRtbF9jNGM0Mjg0OWMyZTY0MmZkOThjZmZjYTYzNGE3ZjBlOSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UGFuY2hvJiMzOTtzIE1leGljYW4gR3JpbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfY2MyMTAxYTkyZTRiNDcxYzkwZDQxMmIzMWU4MjZmNGIuc2V0Q29udGVudChodG1sX2M0YzQyODQ5YzJlNjQyZmQ5OGNmZmNhNjM0YTdmMGU5KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzUwNGExZDgyYThjNTRhYjdhNDA1ODZlNjJjNzNhYzNhLmJpbmRQb3B1cChwb3B1cF9jYzIxMDFhOTJlNGI0NzFjOTBkNDEyYjMxZTgyNmY0YikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl8xMmE3NzJlMWYzNTA0ZDNiOGMyOGI1YTc5MzkwODU4NCA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljk0OTIzNzUyMzQxOTksIC04My45MTQwODQ1MDkwMTUxXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF84YmM4ZmVjZjJjOGU0MGE4YjBlNWExOGVlZjE4OWU2OCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYzk4YzQwODE5NjQ3NGNkZTgwNWE4MWEyODFiNjQyMDcgPSAkKGA8ZGl2IGlkPSJodG1sX2M5OGM0MDgxOTY0NzRjZGU4MDVhODFhMjgxYjY0MjA3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5MYSBCYW1iYSBTZWFmb29kPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzhiYzhmZWNmMmM4ZTQwYThiMGU1YTE4ZWVmMTg5ZTY4LnNldENvbnRlbnQoaHRtbF9jOThjNDA4MTk2NDc0Y2RlODA1YTgxYTI4MWI2NDIwNyk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl8xMmE3NzJlMWYzNTA0ZDNiOGMyOGI1YTc5MzkwODU4NC5iaW5kUG9wdXAocG9wdXBfOGJjOGZlY2YyYzhlNDBhOGIwZTVhMThlZWYxODllNjgpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNWJlNGU2YmQ3YzlkNDhjOWI0NGJjYmZiNzM1ZGY2NWQgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45NDg4NTMxLCAtODMuOTM4Njk1Nl0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNmExMjE4ODkxNzBhNDBiNmI2OTNjMzEwYmRlMWUzMjIgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2ExYWFlYjRlOGFmZDRmYmU4NGNjY2NiNTQ5ODQwNWY4ID0gJChgPGRpdiBpZD0iaHRtbF9hMWFhZWI0ZThhZmQ0ZmJlODRjY2NjYjU0OTg0MDVmOCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U2lsdmlhJiMzOTtzIG1leGljYW4gcmVzdGF1cmFudDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF82YTEyMTg4OTE3MGE0MGI2YjY5M2MzMTBiZGUxZTMyMi5zZXRDb250ZW50KGh0bWxfYTFhYWViNGU4YWZkNGZiZTg0Y2NjY2I1NDk4NDA1ZjgpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNWJlNGU2YmQ3YzlkNDhjOWI0NGJjYmZiNzM1ZGY2NWQuYmluZFBvcHVwKHBvcHVwXzZhMTIxODg5MTcwYTQwYjZiNjkzYzMxMGJkZTFlMzIyKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzc5YmNkNTI4YTllYjQ2Zjk5NDIxNWY5YjNiYTM5YjA4ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTE2MTA0OSwgLTg0LjA4NzU3NF0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfZTY2MjcyMjllNzY1NGQzODgxMzVkMzI3MjljMTI4YTAgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2FiYTFjMTEyNGY5NDRiNWE4MjYxZDA0ODhlMTU3ODliID0gJChgPGRpdiBpZD0iaHRtbF9hYmExYzExMjRmOTQ0YjVhODI2MWQwNDg4ZTE1Nzg5YiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2FuY3VuIE1leGljYW4gR3JpbGwgJmFtcDsgQmFyPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2U2NjI3MjI5ZTc2NTRkMzg4MTM1ZDMyNzI5YzEyOGEwLnNldENvbnRlbnQoaHRtbF9hYmExYzExMjRmOTQ0YjVhODI2MWQwNDg4ZTE1Nzg5Yik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl83OWJjZDUyOGE5ZWI0NmY5OTQyMTVmOWIzYmEzOWIwOC5iaW5kUG9wdXAocG9wdXBfZTY2MjcyMjllNzY1NGQzODgxMzVkMzI3MjljMTI4YTApCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNDdhN2RjMjJmZGVhNDQyOGIwZDMyYzc1YTNiMTc4ZTUgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNi4wMjM1NjAzNTYxNzI2LCAtODQuMjQwODYxMzg2MDYwN10sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfYzdhNzNmYjJlNjJlNDQ4OWIzYTUzMDg0MDRjYjY0ZjcgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzAzOTIxZTRlNDM5MDQzNzg5YTVjNGJmNDAxYWEzZDA0ID0gJChgPGRpdiBpZD0iaHRtbF8wMzkyMWU0ZTQzOTA0Mzc4OWE1YzRiZjQwMWFhM2QwNCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TGEgQ2FzaXRhIENoYXJyYTwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9jN2E3M2ZiMmU2MmU0NDg5YjNhNTMwODQwNGNiNjRmNy5zZXRDb250ZW50KGh0bWxfMDM5MjFlNGU0MzkwNDM3ODlhNWM0YmY0MDFhYTNkMDQpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNDdhN2RjMjJmZGVhNDQyOGIwZDMyYzc1YTNiMTc4ZTUuYmluZFBvcHVwKHBvcHVwX2M3YTczZmIyZTYyZTQ0ODliM2E1MzA4NDA0Y2I2NGY3KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2ZlYjAzYTgxMmVjMTQ2OGI5YzU3NGRkZTIzNTg4NTI4ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTc4ODA1NTQxOTkyMiwgLTg0LjExNjQxNjkzMTE1MjNdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzQwMzhiM2Q4ZTZlZTRiNTc4OWE2NDU3YzEwOGI0NTc5ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9lMzFkYjg5NWI3MDg0MGMwYjIyODg2NTI3NDQyMTA4OCA9ICQoYDxkaXYgaWQ9Imh0bWxfZTMxZGI4OTViNzA4NDBjMGIyMjg4NjUyNzQ0MjEwODgiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlNlbm9yIENhY3R1czwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF80MDM4YjNkOGU2ZWU0YjU3ODlhNjQ1N2MxMDhiNDU3OS5zZXRDb250ZW50KGh0bWxfZTMxZGI4OTViNzA4NDBjMGIyMjg4NjUyNzQ0MjEwODgpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfZmViMDNhODEyZWMxNDY4YjljNTc0ZGRlMjM1ODg1MjguYmluZFBvcHVwKHBvcHVwXzQwMzhiM2Q4ZTZlZTRiNTc4OWE2NDU3YzEwOGI0NTc5KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzI1MzVhMjFiNDI0MjQ0MWZhOWI5NWE0OTJkYjZlNTYyID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTUxMjM4LCAtODQuMTU3MDE0XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF84ZTJlNmM2NjQ2ZmU0MTA5OWRhNWNjZDM0NTJkNWUwYSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfOTY3MDZkZjBlMTVlNDhjOGExN2I1MjE2MTBmYmMxOWIgPSAkKGA8ZGl2IGlkPSJodG1sXzk2NzA2ZGYwZTE1ZTQ4YzhhMTdiNTIxNjEwZmJjMTliIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Eb24gR2FsbG88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfOGUyZTZjNjY0NmZlNDEwOTlkYTVjY2QzNDUyZDVlMGEuc2V0Q29udGVudChodG1sXzk2NzA2ZGYwZTE1ZTQ4YzhhMTdiNTIxNjEwZmJjMTliKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzI1MzVhMjFiNDI0MjQ0MWZhOWI5NWE0OTJkYjZlNTYyLmJpbmRQb3B1cChwb3B1cF84ZTJlNmM2NjQ2ZmU0MTA5OWRhNWNjZDM0NTJkNWUwYSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9mMGM1MWFhOWFlNDk0ZGZkOGJmN2FmZDFjOTAyYWFlMSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjAzNDMzMTg0MjEwNzksIC04My44ODk0OTMzNDYyMTQzXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF84OTM1Y2ExMTQyYjU0YWQ0YjA1M2YzYWJkNDcxODY0YSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNzEwNzE3YzJiODIxNGI1NTg5YTAyNGE0YjE5YzQ3OTcgPSAkKGA8ZGl2IGlkPSJodG1sXzcxMDcxN2MyYjgyMTRiNTU4OWEwMjRhNGIxOWM0Nzk3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5IYWJhbmVyb3M8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfODkzNWNhMTE0MmI1NGFkNGIwNTNmM2FiZDQ3MTg2NGEuc2V0Q29udGVudChodG1sXzcxMDcxN2MyYjgyMTRiNTU4OWEwMjRhNGIxOWM0Nzk3KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2YwYzUxYWE5YWU0OTRkZmQ4YmY3YWZkMWM5MDJhYWUxLmJpbmRQb3B1cChwb3B1cF84OTM1Y2ExMTQyYjU0YWQ0YjA1M2YzYWJkNDcxODY0YSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9jNjYwMDFmZWFmNzQ0NjA0ODEyZjk0MGI3ZGE2Y2EyNyA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljk3OTY2NzksIC04My44MjU4MjddLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2UxMDEyZmU1ZDg3NjQ4MDRhZDU0MDc2M2QwNjI2NGVmID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF84MGM4ZmI3NTU4NTk0Nzg1YTYwMjM4OWE4NzRlY2RjNCA9ICQoYDxkaXYgaWQ9Imh0bWxfODBjOGZiNzU1ODU5NDc4NWE2MDIzODlhODc0ZWNkYzQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlJldHJvIFRhY288L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfZTEwMTJmZTVkODc2NDgwNGFkNTQwNzYzZDA2MjY0ZWYuc2V0Q29udGVudChodG1sXzgwYzhmYjc1NTg1OTQ3ODVhNjAyMzg5YTg3NGVjZGM0KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2M2NjAwMWZlYWY3NDQ2MDQ4MTJmOTQwYjdkYTZjYTI3LmJpbmRQb3B1cChwb3B1cF9lMTAxMmZlNWQ4NzY0ODA0YWQ1NDA3NjNkMDYyNjRlZikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9kYjExYTFmNjZiMTM0MDI4YjhjZmY3MDNjMjY0YTkyZSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjA4ODQzNDYwOTkzODcsIC04My45MzYwNDAxNDc3NjNdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzUwNTljNmE2ZTY3MTQ5M2Y5YTM0ODEzMGYzOTU4MWZmID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF84OGY3NTI5ZDYxMzQ0OGRkODVkMmIwNGI0NjljOTVhOSA9ICQoYDxkaXYgaWQ9Imh0bWxfODhmNzUyOWQ2MTM0NDhkZDg1ZDJiMDRiNDY5Yzk1YTkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkVsIE1ldGF0ZTwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF81MDU5YzZhNmU2NzE0OTNmOWEzNDgxMzBmMzk1ODFmZi5zZXRDb250ZW50KGh0bWxfODhmNzUyOWQ2MTM0NDhkZDg1ZDJiMDRiNDY5Yzk1YTkpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfZGIxMWExZjY2YjEzNDAyOGI4Y2ZmNzAzYzI2NGE5MmUuYmluZFBvcHVwKHBvcHVwXzUwNTljNmE2ZTY3MTQ5M2Y5YTM0ODEzMGYzOTU4MWZmKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzNiOWFiYzI4ZWQ0ODQ4OTJiOGQ4Zjc3MzFmNTgyMWYzID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMDA3NzU0OSwgLTgzLjk2NjI5NTldLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzg5MmQ3YjdmZTIwMzQzMTNhMDYzMzQyZDZiZWEzNDdiID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9iYzFhM2JhYTA0YjE0ZTYwOGZkNzg0MWE0YTYzYTM4OCA9ICQoYDxkaXYgaWQ9Imh0bWxfYmMxYTNiYWEwNGIxNGU2MDhmZDc4NDFhNGE2M2EzODgiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlNhYm9yIENhdHJhY2hvIExhdGluIEN1aXNpbmU8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfODkyZDdiN2ZlMjAzNDMxM2EwNjMzNDJkNmJlYTM0N2Iuc2V0Q29udGVudChodG1sX2JjMWEzYmFhMDRiMTRlNjA4ZmQ3ODQxYTRhNjNhMzg4KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzNiOWFiYzI4ZWQ0ODQ4OTJiOGQ4Zjc3MzFmNTgyMWYzLmJpbmRQb3B1cChwb3B1cF84OTJkN2I3ZmUyMDM0MzEzYTA2MzM0MmQ2YmVhMzQ3YikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl85ZGRiNjZkZmRmNTA0ZmY2YjIyMzEwM2Y1YzQ0NjkyZiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjAyMTIxMiwgLTg0LjA1MTA0NV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNTZhNjY5NjUyYWMyNGIwMmJhOTZjNzlhOGE2ODgxZGUgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2JmNDY2MTkyMGU0YTQ5YmQ5NzMyMTg0ZmQ4MWIzODU0ID0gJChgPGRpdiBpZD0iaHRtbF9iZjQ2NjE5MjBlNGE0OWJkOTczMjE4NGZkODFiMzg1NCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RWwgUmV5IE1leGljYW4gUmVzdGF1cmFudDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF81NmE2Njk2NTJhYzI0YjAyYmE5NmM3OWE4YTY4ODFkZS5zZXRDb250ZW50KGh0bWxfYmY0NjYxOTIwZTRhNDliZDk3MzIxODRmZDgxYjM4NTQpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfOWRkYjY2ZGZkZjUwNGZmNmIyMjMxMDNmNWM0NDY5MmYuYmluZFBvcHVwKHBvcHVwXzU2YTY2OTY1MmFjMjRiMDJiYTk2Yzc5YThhNjg4MWRlKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzAwMjFlZDU3NzYxZjQ2OGFiNWQzZjI3MzJmZGU0ZTE5ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuODE3NTA0ODgyODEyNSwgLTgzLjk3OTYzNzE0NTk5NjFdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzRiOWQ1YTYwMGVjMzQyMGE5YjEyNjMyZWM4NTQyZWJmID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9mMzU5YmIzMzAyMWM0ZTZhYmRiZmQ3NDY5OGEzZjc1MyA9ICQoYDxkaXYgaWQ9Imh0bWxfZjM1OWJiMzMwMjFjNGU2YWJkYmZkNzQ2OThhM2Y3NTMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkVsIFNhem9uIE1leGljYW5vIFJlc3RhdXJhbnQ8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNGI5ZDVhNjAwZWMzNDIwYTliMTI2MzJlYzg1NDJlYmYuc2V0Q29udGVudChodG1sX2YzNTliYjMzMDIxYzRlNmFiZGJmZDc0Njk4YTNmNzUzKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzAwMjFlZDU3NzYxZjQ2OGFiNWQzZjI3MzJmZGU0ZTE5LmJpbmRQb3B1cChwb3B1cF80YjlkNWE2MDBlYzM0MjBhOWIxMjYzMmVjODU0MmViZikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9iMzlhZmQxZjhjNzg0ZWUwOGIyZjY3ZTY5YmJlYTJkZSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1LjkzNDg2OTksIC04NC4wMDM3Mzk5XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9iNzYxNDFiNjNkZWM0ODRhOGViNjA2MmY2YWYyY2JiNiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYmQyZWY3NWQ0ZGQwNGUyYTljMTBkMTUwODBkYTQxMjIgPSAkKGA8ZGl2IGlkPSJodG1sX2JkMmVmNzVkNGRkMDRlMmE5YzEwZDE1MDgwZGE0MTIyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CYXJiZXJpdG9zPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2I3NjE0MWI2M2RlYzQ4NGE4ZWI2MDYyZjZhZjJjYmI2LnNldENvbnRlbnQoaHRtbF9iZDJlZjc1ZDRkZDA0ZTJhOWMxMGQxNTA4MGRhNDEyMik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9iMzlhZmQxZjhjNzg0ZWUwOGIyZjY3ZTY5YmJlYTJkZS5iaW5kUG9wdXAocG9wdXBfYjc2MTQxYjYzZGVjNDg0YThlYjYwNjJmNmFmMmNiYjYpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNzQxM2Q1MDZjYTQzNDJlMWIzZmI1NzFjOWIyMDM4YmIgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS43MTM0MDYyOTY5NTMxLCAtODQuMDIyNTIxNDMwOTIyNF0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMWIwNDk3OGMwNWE5NDI4MmJiYzVmM2RkNjM0ZWM1ZjAgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzk4MzA0ODkyYjY5ZjQ0ZGM5MDk5YTdlZGUxNjFjMzVjID0gJChgPGRpdiBpZD0iaHRtbF85ODMwNDg5MmI2OWY0NGRjOTA5OWE3ZWRlMTYxYzM1YyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UGFuY2hvJiMzOTtzIE1leGljYW4gUmVzdGF1cmFudDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8xYjA0OTc4YzA1YTk0MjgyYmJjNWYzZGQ2MzRlYzVmMC5zZXRDb250ZW50KGh0bWxfOTgzMDQ4OTJiNjlmNDRkYzkwOTlhN2VkZTE2MWMzNWMpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNzQxM2Q1MDZjYTQzNDJlMWIzZmI1NzFjOWIyMDM4YmIuYmluZFBvcHVwKHBvcHVwXzFiMDQ5NzhjMDVhOTQyODJiYmM1ZjNkZDYzNGVjNWYwKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2FiZDYxZmU3MzJiNTRmZTJhN2JlY2MzZGY4ZTlkN2E4ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuODk2MTExMSwgLTg0LjE3MDg5NzddLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzFmNjAzNGFhZTQ0YjQ1MTE4OWJiYTQxZjRkN2I3Y2YxID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8zZDg1YzU0YThkODg0NWEyOWJhOGVhYTRlMTllYjc4MiA9ICQoYDxkaXYgaWQ9Imh0bWxfM2Q4NWM1NGE4ZDg4NDVhMjliYThlYWE0ZTE5ZWI3ODIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkxhIFBhcnJpbGxhPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzFmNjAzNGFhZTQ0YjQ1MTE4OWJiYTQxZjRkN2I3Y2YxLnNldENvbnRlbnQoaHRtbF8zZDg1YzU0YThkODg0NWEyOWJhOGVhYTRlMTllYjc4Mik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9hYmQ2MWZlNzMyYjU0ZmUyYTdiZWNjM2RmOGU5ZDdhOC5iaW5kUG9wdXAocG9wdXBfMWY2MDM0YWFlNDRiNDUxMTg5YmJhNDFmNGQ3YjdjZjEpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfMWE0ZTBmOGZiMGE3NDc2YmIyMjA1ZjViNmJjMzEwZTcgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS44NjI0NjAyOTgzODI2LCAtODQuMTQxNjc5NTg3MzYxM10sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNzUyZGY1OGQyNGUwNGQ4NmI1NmJiZjRhYjIyNTcwMzcgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2UzZWU5ZjRmMjQzMDRkYTc5NDVkMDRiMGY2OGVhMDVhID0gJChgPGRpdiBpZD0iaHRtbF9lM2VlOWY0ZjI0MzA0ZGE3OTQ1ZDA0YjBmNjhlYTA1YSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGFjb3MgRWwgUHJpbW8gRm9vZCBUcnVjazwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF83NTJkZjU4ZDI0ZTA0ZDg2YjU2YmJmNGFiMjI1NzAzNy5zZXRDb250ZW50KGh0bWxfZTNlZTlmNGYyNDMwNGRhNzk0NWQwNGIwZjY4ZWEwNWEpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfMWE0ZTBmOGZiMGE3NDc2YmIyMjA1ZjViNmJjMzEwZTcuYmluZFBvcHVwKHBvcHVwXzc1MmRmNThkMjRlMDRkODZiNTZiYmY0YWIyMjU3MDM3KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2QwYjc4ZGU2MmNjMDQ2ZGJhODA1NGQxMWY5MWUwNGNiID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuODA0NTQ5MTc2NjA5NCwgLTg0LjI2MDI3MTE5MTU5N10sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNmI3Y2I1MWE1MDA2NDc4ODg4YTEzYzk0N2E0Y2ZiZDYgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2I5OGQ2Y2U4MjRiMDQ2OWE4YWNlZWZmMDFiYjNkNDc1ID0gJChgPGRpdiBpZD0iaHRtbF9iOThkNmNlODI0YjA0NjlhOGFjZWVmZjAxYmIzZDQ3NSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2FzYSBGaWVzdGE8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNmI3Y2I1MWE1MDA2NDc4ODg4YTEzYzk0N2E0Y2ZiZDYuc2V0Q29udGVudChodG1sX2I5OGQ2Y2U4MjRiMDQ2OWE4YWNlZWZmMDFiYjNkNDc1KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2QwYjc4ZGU2MmNjMDQ2ZGJhODA1NGQxMWY5MWUwNGNiLmJpbmRQb3B1cChwb3B1cF82YjdjYjUxYTUwMDY0Nzg4ODhhMTNjOTQ3YTRjZmJkNikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl84NGRiNjQzMDJmYzA0ZGI2YTc3YjU3ZDFiZjgyNzBkYiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjA3MjY0LCAtODMuOTI1MTZdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2I4MTE2MDg0Y2QxMDRlNzQ4OWQ3ODBlYjI5ODUwOWM2ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8zZWQzZDdkODk5Y2Y0ZjY2YmI4ZWU2M2FmMTk5OTU1YiA9ICQoYDxkaXYgaWQ9Imh0bWxfM2VkM2Q3ZDg5OWNmNGY2NmJiOGVlNjNhZjE5OTk1NWIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNhbmN1biBNZXhpY2FuIEdyaWxsICZhbXA7IENhbnRpbmE8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfYjgxMTYwODRjZDEwNGU3NDg5ZDc4MGViMjk4NTA5YzYuc2V0Q29udGVudChodG1sXzNlZDNkN2Q4OTljZjRmNjZiYjhlZTYzYWYxOTk5NTViKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzg0ZGI2NDMwMmZjMDRkYjZhNzdiNTdkMWJmODI3MGRiLmJpbmRQb3B1cChwb3B1cF9iODExNjA4NGNkMTA0ZTc0ODlkNzgwZWIyOTg1MDljNikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9iYzJmZGE3MzhkZmQ0Y2QxOWEwMDNkZWM3NTNlN2UwMSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjE3MDc2MjMxMTYyNiwgLTg0LjA3NzU0MTQ2NjY2NjZdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzFjNDNiMmRlZjY1ODRhODI4MzY5YTk0MTZiNDNlOGM0ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8yY2YwZTc3MGVhY2E0ZGQ2ODI0OGNmZDFiYzc2NTk5YSA9ICQoYDxkaXYgaWQ9Imh0bWxfMmNmMGU3NzBlYWNhNGRkNjgyNDhjZmQxYmM3NjU5OWEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkhhYmFuZXJvczwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8xYzQzYjJkZWY2NTg0YTgyODM2OWE5NDE2YjQzZThjNC5zZXRDb250ZW50KGh0bWxfMmNmMGU3NzBlYWNhNGRkNjgyNDhjZmQxYmM3NjU5OWEpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfYmMyZmRhNzM4ZGZkNGNkMTlhMDAzZGVjNzUzZTdlMDEuYmluZFBvcHVwKHBvcHVwXzFjNDNiMmRlZjY1ODRhODI4MzY5YTk0MTZiNDNlOGM0KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzkwNjFhYmExNTZhMzQ0YWRhYjYxOThjM2Q1YTNiZjAyID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTUxMDgsIC04My44OTM4OF0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfOTMyYWE4NDA0ZDMyNDMzOGJlYmZhM2Q5ZjNmNjlhOGUgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzc5NzAxMmY3MTI5NzQzOGRhMmM4MWJhODczNDgzYjBmID0gJChgPGRpdiBpZD0iaHRtbF83OTcwMTJmNzEyOTc0MzhkYTJjODFiYTg3MzQ4M2IwZiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2FwdGFpbiBNdWNoYWNob3M8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfOTMyYWE4NDA0ZDMyNDMzOGJlYmZhM2Q5ZjNmNjlhOGUuc2V0Q29udGVudChodG1sXzc5NzAxMmY3MTI5NzQzOGRhMmM4MWJhODczNDgzYjBmKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzkwNjFhYmExNTZhMzQ0YWRhYjYxOThjM2Q1YTNiZjAyLmJpbmRQb3B1cChwb3B1cF85MzJhYTg0MDRkMzI0MzM4YmViZmEzZDlmM2Y2OWE4ZSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl83YjA0YjIzMGU3NWQ0NDYyYjE0ODg1MjJkMGEwZWU3NyA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjA1MjA2NzQ0NjU5NjQsIC04My45OTIwMTcwOTc3NzEyXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF80YzU3ODRmNmFiMGU0ZjZjOWVkMzFmNjI3YTZhYjVmNyA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYjY0NGYzNGQ4OThiNDk0NjlkY2U2NjQ2NWQ1YjA0YjUgPSAkKGA8ZGl2IGlkPSJodG1sX2I2NDRmMzRkODk4YjQ5NDY5ZGNlNjY0NjVkNWIwNGI1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5FbCBNYXJpYWNoaSBMb2NvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzRjNTc4NGY2YWIwZTRmNmM5ZWQzMWY2MjdhNmFiNWY3LnNldENvbnRlbnQoaHRtbF9iNjQ0ZjM0ZDg5OGI0OTQ2OWRjZTY2NDY1ZDViMDRiNSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl83YjA0YjIzMGU3NWQ0NDYyYjE0ODg1MjJkMGEwZWU3Ny5iaW5kUG9wdXAocG9wdXBfNGM1Nzg0ZjZhYjBlNGY2YzllZDMxZjYyN2E2YWI1ZjcpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfODVmYTA0YzczODZiNGMyYWI5MzgzMjgzZjRiNjM0NmMgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNi4wMjg1Nzk3LCAtODQuMjM4NjE2OV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMDEzN2UyYTJlNzNhNDBjZmJhMDM1MzZkZTQyN2RlNWIgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2VjODcyMTYxOWQxNTQ4ZTU5NjBjODFjMmRkZjU3M2JkID0gJChgPGRpdiBpZD0iaHRtbF9lYzg3MjE2MTlkMTU0OGU1OTYwYzgxYzJkZGY1NzNiZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+R2FsbG8gTG9jbzwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8wMTM3ZTJhMmU3M2E0MGNmYmEwMzUzNmRlNDI3ZGU1Yi5zZXRDb250ZW50KGh0bWxfZWM4NzIxNjE5ZDE1NDhlNTk2MGM4MWMyZGRmNTczYmQpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfODVmYTA0YzczODZiNGMyYWI5MzgzMjgzZjRiNjM0NmMuYmluZFBvcHVwKHBvcHVwXzAxMzdlMmEyZTczYTQwY2ZiYTAzNTM2ZGU0MjdkZTViKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzY5M2MxMjdhYjBmMTQ0OGM4YzQ4Y2FmMGQzMzBlZTMxID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTMyMDg2LCAtODMuOTA1MzQ2XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8zNWEwOGNiM2Y5Zjc0Y2NiYTU2NWZjZjE5MDQ1NjMyMiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfY2RmYzYwOTk0MmNmNDJiOGFlN2U2NWQyMjhhZjI2ZTEgPSAkKGA8ZGl2IGlkPSJodG1sX2NkZmM2MDk5NDJjZjQyYjhhZTdlNjVkMjI4YWYyNmUxIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DYW5jdW4gTWV4aWNhbiBSZXN0YXVyYW50ICZhbXA7IENhbnRpbmE8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMzVhMDhjYjNmOWY3NGNjYmE1NjVmY2YxOTA0NTYzMjIuc2V0Q29udGVudChodG1sX2NkZmM2MDk5NDJjZjQyYjhhZTdlNjVkMjI4YWYyNmUxKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzY5M2MxMjdhYjBmMTQ0OGM4YzQ4Y2FmMGQzMzBlZTMxLmJpbmRQb3B1cChwb3B1cF8zNWEwOGNiM2Y5Zjc0Y2NiYTU2NWZjZjE5MDQ1NjMyMikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9kNmY2YzkxNDE4N2Q0NGU1OWI5NDIzMzJlMmVmNDNlOCA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljk3NDc4Mzg3MzQyMywgLTgzLjk3MTgwOTE0MDM2MjVdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2Q1Y2UxMjdlMTczNjQxZjc4MzE1ODY5OWE0Yjg3NDMyID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF81MjhkMDM2Mjk3OWQ0YjYzYjI0OGUxMjhkMGYyY2UwMiA9ICQoYDxkaXYgaWQ9Imh0bWxfNTI4ZDAzNjI5NzlkNGI2M2IyNDhlMTI4ZDBmMmNlMDIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPnNxdWlzaXRvcyBtZXhpY2Fub3M8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfZDVjZTEyN2UxNzM2NDFmNzgzMTU4Njk5YTRiODc0MzIuc2V0Q29udGVudChodG1sXzUyOGQwMzYyOTc5ZDRiNjNiMjQ4ZTEyOGQwZjJjZTAyKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2Q2ZjZjOTE0MTg3ZDQ0ZTU5Yjk0MjMzMmUyZWY0M2U4LmJpbmRQb3B1cChwb3B1cF9kNWNlMTI3ZTE3MzY0MWY3ODMxNTg2OTlhNGI4NzQzMikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl82MzQyYzE2ZTRiMWE0YWY2YWM1M2FiZGM2NDZmMDU4YiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljg4NzY3LCAtODQuMTQ4NjVdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzc5YTFhYzAwZjIzYjRmN2Y5MDk3NmE1ZjQ2MDliM2Y4ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8xZjE1ZTIxYjM1MDU0MmRmOGUzMjAyZTBlYTM4YjMzMyA9ICQoYDxkaXYgaWQ9Imh0bWxfMWYxNWUyMWIzNTA1NDJkZjhlMzIwMmUwZWEzOGIzMzMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkVsIE1lemNhbCBNZXhpY2FuIFJlc3RhdXJhbnQ8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNzlhMWFjMDBmMjNiNGY3ZjkwOTc2YTVmNDYwOWIzZjguc2V0Q29udGVudChodG1sXzFmMTVlMjFiMzUwNTQyZGY4ZTMyMDJlMGVhMzhiMzMzKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzYzNDJjMTZlNGIxYTRhZjZhYzUzYWJkYzY0NmYwNThiLmJpbmRQb3B1cChwb3B1cF83OWExYWMwMGYyM2I0ZjdmOTA5NzZhNWY0NjA5YjNmOCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9iODIxOTJhNjA3NzU0MGI0YmNlOWVjN2E3YjQzMDcwYyA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljg5NzA1NDYsIC04NC4xNzU1MzM3XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF85MGI1MDY5ZDZjZjk0ZWMxYmUzMjUwNmE5YzUyMTRkZiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNmFjNjg1YTEyMWRjNGUyYmE5NzI3MDY5NzNiNjY4NjIgPSAkKGA8ZGl2IGlkPSJodG1sXzZhYzY4NWExMjFkYzRlMmJhOTcyNzA2OTczYjY2ODYyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5MYSBHb3RhIEZyaWE8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfOTBiNTA2OWQ2Y2Y5NGVjMWJlMzI1MDZhOWM1MjE0ZGYuc2V0Q29udGVudChodG1sXzZhYzY4NWExMjFkYzRlMmJhOTcyNzA2OTczYjY2ODYyKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2I4MjE5MmE2MDc3NTQwYjRiY2U5ZWM3YTdiNDMwNzBjLmJpbmRQb3B1cChwb3B1cF85MGI1MDY5ZDZjZjk0ZWMxYmUzMjUwNmE5YzUyMTRkZikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl84NTk0ZDJjMjNlNGE0OGU3ODE3MjU0YThjMTE2NmI2OSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjAwNTM3ODIzNzY2MDQsIC04My45MjY5MDcyMjY5ODA4XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9kNmIyYWRiM2VlY2U0YmJiODkxMTQ1MjRlZTQ1NmRiNSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYjM3YzM5ZDI3Mzc0NDgxNGJhNjJiNzMwYzlkYWU4NjEgPSAkKGA8ZGl2IGlkPSJodG1sX2IzN2MzOWQyNzM3NDQ4MTRiYTYyYjczMGM5ZGFlODYxIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5KYWxhcGXDsW8mIzM5O3MgTWV4aWNhbiBHcmlsbDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9kNmIyYWRiM2VlY2U0YmJiODkxMTQ1MjRlZTQ1NmRiNS5zZXRDb250ZW50KGh0bWxfYjM3YzM5ZDI3Mzc0NDgxNGJhNjJiNzMwYzlkYWU4NjEpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfODU5NGQyYzIzZTRhNDhlNzgxNzI1NGE4YzExNjZiNjkuYmluZFBvcHVwKHBvcHVwX2Q2YjJhZGIzZWVjZTRiYmI4OTExNDUyNGVlNDU2ZGI1KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzJhZDg2YTExMGIzODRiNzE4YTcyYzc3OTlkNmVhNWEwID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTI0MzIsIC04NC4wNTU4Nl0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMThlY2Y1Mzg3MzliNGI0MTkwNGIyMzc4NGNjZmJlZmEgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2VmNjNjMmMwYmRmMTQwZTdhZTViYzllMTI0MjRjZjBhID0gJChgPGRpdiBpZD0iaHRtbF9lZjYzYzJjMGJkZjE0MGU3YWU1YmM5ZTEyNDI0Y2YwYSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGFxdWVyaWEgTGEgSGVycmFkdXJhPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzE4ZWNmNTM4NzM5YjRiNDE5MDRiMjM3ODRjY2ZiZWZhLnNldENvbnRlbnQoaHRtbF9lZjYzYzJjMGJkZjE0MGU3YWU1YmM5ZTEyNDI0Y2YwYSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl8yYWQ4NmExMTBiMzg0YjcxOGE3MmM3Nzk5ZDZlYTVhMC5iaW5kUG9wdXAocG9wdXBfMThlY2Y1Mzg3MzliNGI0MTkwNGIyMzc4NGNjZmJlZmEpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNTMxMjBmMzM3ODA2NGRlYmI2MWY2ZWU3ZjI0Y2ZiZGIgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNi4wODM1NjIsIC04NC4xMzEwNTI3XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9iNjJmNzRhMjU4ZDc0ODI5YWNiNmVhZTNjMWIzN2UyOSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNGE4ZmYxN2I2Mjc1NGY3ZDgyMGFmYTc1MDkwOTczYTYgPSAkKGA8ZGl2IGlkPSJodG1sXzRhOGZmMTdiNjI3NTRmN2Q4MjBhZmE3NTA5MDk3M2E2IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5FbCBUZXBhbWU8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfYjYyZjc0YTI1OGQ3NDgyOWFjYjZlYWUzYzFiMzdlMjkuc2V0Q29udGVudChodG1sXzRhOGZmMTdiNjI3NTRmN2Q4MjBhZmE3NTA5MDk3M2E2KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzUzMTIwZjMzNzgwNjRkZWJiNjFmNmVlN2YyNGNmYmRiLmJpbmRQb3B1cChwb3B1cF9iNjJmNzRhMjU4ZDc0ODI5YWNiNmVhZTNjMWIzN2UyOSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9iYzIwMjVhZDEzOWQ0NDNmOTE1YjAzM2QyOWNlY2YwOCA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjAyNzE0LCAtODMuOTI2NV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfODc3NTJiN2M4NmMwNGFlM2E2NWE3YjZhYjRiYTI4ZDQgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzkwMDMyZjk1ZTBiYTRhMDA4MDE3MWVkZTBjZjJjZDkyID0gJChgPGRpdiBpZD0iaHRtbF85MDAzMmY5NWUwYmE0YTAwODAxNzFlZGUwY2YyY2Q5MiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QW1pZ28mIzM5O3MgVGVxdWlsYSBNZXhpY2FuIEdyaWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzg3NzUyYjdjODZjMDRhZTNhNjVhN2I2YWI0YmEyOGQ0LnNldENvbnRlbnQoaHRtbF85MDAzMmY5NWUwYmE0YTAwODAxNzFlZGUwY2YyY2Q5Mik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9iYzIwMjVhZDEzOWQ0NDNmOTE1YjAzM2QyOWNlY2YwOC5iaW5kUG9wdXAocG9wdXBfODc3NTJiN2M4NmMwNGFlM2E2NWE3YjZhYjRiYTI4ZDQpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNzM1ODAzNWM5Y2E4NDllMmEwYmVlMmJkNTk2Njg2ZmQgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45MzA2MzA0MDM4Mjc5LCAtODQuMDMxMzE1MTQ3ODc2N10sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfZTk3OWRhNDBhZTEzNGRmZDgxNWU3OTM1MmNhMzk0MTEgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzU3NGVmMzA2OTA2ZTQ3NzE4MDdhNWY1NTUzMzNiNTUzID0gJChgPGRpdiBpZD0iaHRtbF81NzRlZjMwNjkwNmU0NzcxODA3YTVmNTU1MzMzYjU1MyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UURPQkEgTWV4aWNhbiBFYXRzPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2U5NzlkYTQwYWUxMzRkZmQ4MTVlNzkzNTJjYTM5NDExLnNldENvbnRlbnQoaHRtbF81NzRlZjMwNjkwNmU0NzcxODA3YTVmNTU1MzMzYjU1Myk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl83MzU4MDM1YzljYTg0OWUyYTBiZWUyYmQ1OTY2ODZmZC5iaW5kUG9wdXAocG9wdXBfZTk3OWRhNDBhZTEzNGRmZDgxNWU3OTM1MmNhMzk0MTEpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNDljMjA5NGFjZWQ3NDEzOWI1ZTdjZGRkMWUxMzQwOGQgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNi4xODcyMDgsIC04NC4wNjA3MDIzXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9kZTFlZThhODViMzc0M2NhYmVhNTg3NjI3YzFlNGQ5YiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYzUyODJjMmMzYzQxNDcxMDk4OGFkNzY4NTVjODUxODEgPSAkKGA8ZGl2IGlkPSJodG1sX2M1MjgyYzJjM2M0MTQ3MTA5ODhhZDc2ODU1Yzg1MTgxIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5MYSBTaWVycmEgTWV4aWNhbiBSZXN0YXVyYW50PC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2RlMWVlOGE4NWIzNzQzY2FiZWE1ODc2MjdjMWU0ZDliLnNldENvbnRlbnQoaHRtbF9jNTI4MmMyYzNjNDE0NzEwOTg4YWQ3Njg1NWM4NTE4MSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl80OWMyMDk0YWNlZDc0MTM5YjVlN2NkZGQxZTEzNDA4ZC5iaW5kUG9wdXAocG9wdXBfZGUxZWU4YTg1YjM3NDNjYWJlYTU4NzYyN2MxZTRkOWIpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfYmE3NGNhOTQ3NzZmNDE5ZmE5ZTQ4ODZhMmI2ZjAwNzUgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45NTIzNDQsIC04NC4xNTI3OThdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2Y0MGNlMGI0NjNkZDRlNzQ5MjQ2MDdhYWYwMGRhY2E3ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8wMzc1ZDgxMjcwODY0NWQzOTkzYTY1YTRhZjVmOWI1YiA9ICQoYDxkaXYgaWQ9Imh0bWxfMDM3NWQ4MTI3MDg2NDVkMzk5M2E2NWE0YWY1ZjliNWIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlNhbHNhcml0YSYjMzk7cyBGcmVzaCBNZXhpY2FuIEdyaWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2Y0MGNlMGI0NjNkZDRlNzQ5MjQ2MDdhYWYwMGRhY2E3LnNldENvbnRlbnQoaHRtbF8wMzc1ZDgxMjcwODY0NWQzOTkzYTY1YTRhZjVmOWI1Yik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9iYTc0Y2E5NDc3NmY0MTlmYTllNDg4NmEyYjZmMDA3NS5iaW5kUG9wdXAocG9wdXBfZjQwY2UwYjQ2M2RkNGU3NDkyNDYwN2FhZjAwZGFjYTcpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfMzFlOGMxNzQ0YjQzNDNjOTk5ZGViZGRkYTFlOWFhYWEgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS42NzkzMiwgLTgzLjczNDg4XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF82ZjNjOTFkYWQ1NGE0M2NhOTg3NTE2NzIyZmQ5OWU3NiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYTZkY2RiODJlMDJmNDY4MjgwNjdhNmNmNzY3NTI1OGQgPSAkKGA8ZGl2IGlkPSJodG1sX2E2ZGNkYjgyZTAyZjQ2ODI4MDY3YTZjZjc2NzUyNThkIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Nb250ZSBSZWFsIE1leGljYW4gUmVzdGF1cmFudDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF82ZjNjOTFkYWQ1NGE0M2NhOTg3NTE2NzIyZmQ5OWU3Ni5zZXRDb250ZW50KGh0bWxfYTZkY2RiODJlMDJmNDY4MjgwNjdhNmNmNzY3NTI1OGQpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfMzFlOGMxNzQ0YjQzNDNjOTk5ZGViZGRkYTFlOWFhYWEuYmluZFBvcHVwKHBvcHVwXzZmM2M5MWRhZDU0YTQzY2E5ODc1MTY3MjJmZDk5ZTc2KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzM3YWQzYTc4N2Y0ZjRkOTJhNjg2NmJmOWExZTNlYTkwID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMDE5Njc1MDM2NTYyMiwgLTg0LjI0ODM1MTExOTQ1ODddLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2Y0OTNlYTYyZDJkOTRhYzY4NjgyMTZhNzYyNTQ0ZjRiID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9kNzkwMzQ2ZmFhMzM0ZTVjYjQxZGM4MzgwNWY3YzRkOSA9ICQoYDxkaXYgaWQ9Imh0bWxfZDc5MDM0NmZhYTMzNGU1Y2I0MWRjODM4MDVmN2M0ZDkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkxvcyBDYWJhbGxlcm9zPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2Y0OTNlYTYyZDJkOTRhYzY4NjgyMTZhNzYyNTQ0ZjRiLnNldENvbnRlbnQoaHRtbF9kNzkwMzQ2ZmFhMzM0ZTVjYjQxZGM4MzgwNWY3YzRkOSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl8zN2FkM2E3ODdmNGY0ZDkyYTY4NjZiZjlhMWUzZWE5MC5iaW5kUG9wdXAocG9wdXBfZjQ5M2VhNjJkMmQ5NGFjNjg2ODIxNmE3NjI1NDRmNGIpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNWFmNjM0ZGVkODY3NDA0NzllYjI1YzA2ZDM2NjFlNzkgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS44NTgxMDY3LCAtODQuMDc5NjA3N10sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNmY2MWRiM2E1YTlmNDczYmFlOWM0NDQ5ZjIxOTI5ZGIgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzk3YWM2M2VlOGIwYzRlOWU4NmU0MTMwOTlhMTUzNGRkID0gJChgPGRpdiBpZD0iaHRtbF85N2FjNjNlZThiMGM0ZTllODZlNDEzMDk5YTE1MzRkZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U29jY2VyIFRhY288L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNmY2MWRiM2E1YTlmNDczYmFlOWM0NDQ5ZjIxOTI5ZGIuc2V0Q29udGVudChodG1sXzk3YWM2M2VlOGIwYzRlOWU4NmU0MTMwOTlhMTUzNGRkKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzVhZjYzNGRlZDg2NzQwNDc5ZWIyNWMwNmQzNjYxZTc5LmJpbmRQb3B1cChwb3B1cF82ZjYxZGIzYTVhOWY0NzNiYWU5YzQ0NDlmMjE5MjlkYikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl83NTJiNDYzZmM3MWY0YjkxYjRjYTU4ZmYxMjA1OTYyZSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjA4NjgwOSwgLTgzLjkyNTZdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2VkMWE1NzgzOTNhMjQ4ODg5MjNiMTY4MTc0NTJkZDc3ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8wNDQyZThhZTdiNmU0Y2Q4YmQ5N2U0MTJhZGE0NTU0NCA9ICQoYDxkaXYgaWQ9Imh0bWxfMDQ0MmU4YWU3YjZlNGNkOGJkOTdlNDEyYWRhNDU1NDQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlBsYXphIE1leGljbzwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9lZDFhNTc4MzkzYTI0ODg4OTIzYjE2ODE3NDUyZGQ3Ny5zZXRDb250ZW50KGh0bWxfMDQ0MmU4YWU3YjZlNGNkOGJkOTdlNDEyYWRhNDU1NDQpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNzUyYjQ2M2ZjNzFmNGI5MWI0Y2E1OGZmMTIwNTk2MmUuYmluZFBvcHVwKHBvcHVwX2VkMWE1NzgzOTNhMjQ4ODg5MjNiMTY4MTc0NTJkZDc3KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzU1YWQ1MGQ4NmIxZTQwYzE5ZWRkNjhjYmQwYzQyYTMyID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMTIxNTE5NjQ1MjE2MSwgLTgzLjg1NDAzMDg1NV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfN2YxYzlkNjllNzE0NDIyMmFkMzdiNWE1YjBhYmZhN2YgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2JmNDJiMjg1MmNiNDRkNjQ4ZmIxODYwOGZjMWJkNDEzID0gJChgPGRpdiBpZD0iaHRtbF9iZjQyYjI4NTJjYjQ0ZDY0OGZiMTg2MDhmYzFiZDQxMyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RG9uIEpvc2UmIzM5O3MgTWV4aWNhbiBHcmlsbDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF83ZjFjOWQ2OWU3MTQ0MjIyYWQzN2I1YTViMGFiZmE3Zi5zZXRDb250ZW50KGh0bWxfYmY0MmIyODUyY2I0NGQ2NDhmYjE4NjA4ZmMxYmQ0MTMpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNTVhZDUwZDg2YjFlNDBjMTllZGQ2OGNiZDBjNDJhMzIuYmluZFBvcHVwKHBvcHVwXzdmMWM5ZDY5ZTcxNDQyMjJhZDM3YjVhNWIwYWJmYTdmKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzA1ZTY3MmU0NTg4YzQzNmE5M2Y5N2ZkZWVmODIwNDJkID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuNzczOTAyLCAtODQuMTM4MDJdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzA4YzIyZWQ4M2U3ODQ3MWQ4ZjIyMGJmMWQ5MDVjMWY3ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8wMWFmMjUxZTM5MjI0ZjdjODNiMzhkY2VkNWVmNTQ3NiA9ICQoYDxkaXYgaWQ9Imh0bWxfMDFhZjI1MWUzOTIyNGY3YzgzYjM4ZGNlZDVlZjU0NzYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNhc2EgQW1pZ29zIEJhciAmYW1wOyBHcmlsbDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8wOGMyMmVkODNlNzg0NzFkOGYyMjBiZjFkOTA1YzFmNy5zZXRDb250ZW50KGh0bWxfMDFhZjI1MWUzOTIyNGY3YzgzYjM4ZGNlZDVlZjU0NzYpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfMDVlNjcyZTQ1ODhjNDM2YTkzZjk3ZmRlZWY4MjA0MmQuYmluZFBvcHVwKHBvcHVwXzA4YzIyZWQ4M2U3ODQ3MWQ4ZjIyMGJmMWQ5MDVjMWY3KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzZhMzlkNDI4M2NiNTQyMGY4ZWE1ZGRlODViOTcyYzQ5ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuODE4NzUwMDE0OTQ0MiwgLTg0LjI2NDk5OTI1NTUzOF0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNWU5MWYwNDc2M2MwNDhkNGE4N2I5N2NiMWM5YmFiMWUgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2M4ZTI0MDU3MzY3ZjQ1MTlhNjc1ZDE4ZTY0MWRiMDYyID0gJChgPGRpdiBpZD0iaHRtbF9jOGUyNDA1NzM2N2Y0NTE5YTY3NWQxOGU2NDFkYjA2MiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RWwgU2Vub3IgUmFuY2hvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzVlOTFmMDQ3NjNjMDQ4ZDRhODdiOTdjYjFjOWJhYjFlLnNldENvbnRlbnQoaHRtbF9jOGUyNDA1NzM2N2Y0NTE5YTY3NWQxOGU2NDFkYjA2Mik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl82YTM5ZDQyODNjYjU0MjBmOGVhNWRkZTg1Yjk3MmM0OS5iaW5kUG9wdXAocG9wdXBfNWU5MWYwNDc2M2MwNDhkNGE4N2I5N2NiMWM5YmFiMWUpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfYmJhNWRhYzY4NDE0NDViN2I1MDdkYTRiMTFiYmYzMWUgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNi4wMDMwMywgLTgzLjk3OTU0XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF82NzA2NzRhYTI2ZGU0YjY0ODQ0ZGI0ZDBhM2JhZTE2NiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfZjI4MzE4ZTA2NjU1NDMwMDhmNDY3N2IwNjNkNDFjZTIgPSAkKGA8ZGl2IGlkPSJodG1sX2YyODMxOGUwNjY1NTQzMDA4ZjQ2NzdiMDYzZDQxY2UyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5BenRlY2EgTWFya2V0PC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzY3MDY3NGFhMjZkZTRiNjQ4NDRkYjRkMGEzYmFlMTY2LnNldENvbnRlbnQoaHRtbF9mMjgzMThlMDY2NTU0MzAwOGY0Njc3YjA2M2Q0MWNlMik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9iYmE1ZGFjNjg0MTQ0NWI3YjUwN2RhNGIxMWJiZjMxZS5iaW5kUG9wdXAocG9wdXBfNjcwNjc0YWEyNmRlNGI2NDg0NGRiNGQwYTNiYWUxNjYpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfZDVmZjRhNmI2ZWUyNDhmYmIzOGZlYTM3YmM0MjE1ZTYgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS44MjY5MjYsIC04NC4yNzg4NTYxXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF84YzQ1MDQxZDZlNGM0NDNkYjA5NmNiOThlNWVhYjhkMiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNTEwMDdjNDczYTEyNDAyN2E1N2I5ODM1Yzk0MmRhMWMgPSAkKGA8ZGl2IGlkPSJodG1sXzUxMDA3YzQ3M2ExMjQwMjdhNTdiOTgzNWM5NDJkYTFjIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DaW5jbyBBbWlnb3MgTWV4aWNhbiBSZXN0YXVyYW50IC0gTGVub2lyIENpdHk8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfOGM0NTA0MWQ2ZTRjNDQzZGIwOTZjYjk4ZTVlYWI4ZDIuc2V0Q29udGVudChodG1sXzUxMDA3YzQ3M2ExMjQwMjdhNTdiOTgzNWM5NDJkYTFjKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2Q1ZmY0YTZiNmVlMjQ4ZmJiMzhmZWEzN2JjNDIxNWU2LmJpbmRQb3B1cChwb3B1cF84YzQ1MDQxZDZlNGM0NDNkYjA5NmNiOThlNWVhYjhkMikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl80NzBjNGU0ZmVkNjY0NmU2OTJiZWU4NzJjNmVkMGViMyA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1LjkwMjYsIC04NC4xNDcyOTk0XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8xM2Y4NzFlMjliZmI0YjZhYmJhMjU2NTRkODc1YjFiMSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYjliYTY0Y2VkY2ZmNGExNzg3NmY1YTM2ZDE0MjViODggPSAkKGA8ZGl2IGlkPSJodG1sX2I5YmE2NGNlZGNmZjRhMTc4NzZmNWEzNmQxNDI1Yjg4IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5TYWxzYXJpdGEmIzM5O3MgRnJlc2ggTWV4aWNhbiBHcmlsbDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8xM2Y4NzFlMjliZmI0YjZhYmJhMjU2NTRkODc1YjFiMS5zZXRDb250ZW50KGh0bWxfYjliYTY0Y2VkY2ZmNGExNzg3NmY1YTM2ZDE0MjViODgpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNDcwYzRlNGZlZDY2NDZlNjkyYmVlODcyYzZlZDBlYjMuYmluZFBvcHVwKHBvcHVwXzEzZjg3MWUyOWJmYjRiNmFiYmEyNTY1NGQ4NzViMWIxKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzIwODg2ZTJjNjQ2ZjQxZTU5M2RiMjg5ZTk4YWIzM2U4ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuNzMxNjI0NjAzMjcxNSwgLTg0LjM0ODU1NjUxODU1NDddLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2MzNGVkNDNmOTFmMTQwNGFiNzQyYWFjZTZhMDc5YzM3ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8zNDE4Mjg1ODA0ODQ0YjBkYTJjMDU5YTliYTNmNGRkNCA9ICQoYDxkaXYgaWQ9Imh0bWxfMzQxODI4NTgwNDg0NGIwZGEyYzA1OWE5YmEzZjRkZDQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNpbmNvIEFtaWdvczwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9jMzRlZDQzZjkxZjE0MDRhYjc0MmFhY2U2YTA3OWMzNy5zZXRDb250ZW50KGh0bWxfMzQxODI4NTgwNDg0NGIwZGEyYzA1OWE5YmEzZjRkZDQpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfMjA4ODZlMmM2NDZmNDFlNTkzZGIyODllOThhYjMzZTguYmluZFBvcHVwKHBvcHVwX2MzNGVkNDNmOTFmMTQwNGFiNzQyYWFjZTZhMDc5YzM3KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2UzZmJjMDQ3MTBkODQwMTY4NTE3YjNjY2FkOTdkZGY3ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuNzgwNDYyLCAtODMuOTUxMzc0XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8xNWZhODQ5MGVkZDU0YWM2YjkyYjRlMDFlOGMwOTFiOCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNDIyOGYxNjhjYWM2NDhjN2I5MjIzYjg3OTI3ZmFhNGMgPSAkKGA8ZGl2IGlkPSJodG1sXzQyMjhmMTY4Y2FjNjQ4YzdiOTIyM2I4NzkyN2ZhYTRjIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5NYXJnYXJpdGEmIzM5O3MgTWV4aWNhbiBSZXN0YXVyYW50PC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzE1ZmE4NDkwZWRkNTRhYzZiOTJiNGUwMWU4YzA5MWI4LnNldENvbnRlbnQoaHRtbF80MjI4ZjE2OGNhYzY0OGM3YjkyMjNiODc5MjdmYWE0Yyk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9lM2ZiYzA0NzEwZDg0MDE2ODUxN2IzY2NhZDk3ZGRmNy5iaW5kUG9wdXAocG9wdXBfMTVmYTg0OTBlZGQ1NGFjNmI5MmI0ZTAxZThjMDkxYjgpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfOTk2YTRiZGVlYTBkNGM3Zjk4NGM1ZThmMGEwMmU0YmYgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNi4yMzA3OSwgLTg0LjE1ODE5OTldLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzUwOTYyNjZiMGI1NDRhMmY5NTk1MTE3ODNmMTcxNDMzID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8zMDAxNzk3MDBjMDA0NWUwOWIwOTM2MzJjMjhkZGRjMCA9ICQoYDxkaXYgaWQ9Imh0bWxfMzAwMTc5NzAwYzAwNDVlMDliMDkzNjMyYzI4ZGRkYzAiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkxhIEZpZXN0YSBNZXhpY2FuIFJlc3RhdXJhbnQ8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNTA5NjI2NmIwYjU0NGEyZjk1OTUxMTc4M2YxNzE0MzMuc2V0Q29udGVudChodG1sXzMwMDE3OTcwMGMwMDQ1ZTA5YjA5MzYzMmMyOGRkZGMwKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzk5NmE0YmRlZWEwZDRjN2Y5ODRjNWU4ZjBhMDJlNGJmLmJpbmRQb3B1cChwb3B1cF81MDk2MjY2YjBiNTQ0YTJmOTU5NTExNzgzZjE3MTQzMykKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9lMzBlZmY1ODA5ZTA0OWNkYjVmZjdjZTllNWQwYTM0MiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljg4MjI4OTksIC04My43MzkxODkxXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8xYzQ4NzA4ZjNhMDU0NTQ1OGZlZWYxMTU5ODg4MjQ1NSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMWJjNDYzMGFhNGM0NDNhNGEwMTEwODhhNjA4NzMzODMgPSAkKGA8ZGl2IGlkPSJodG1sXzFiYzQ2MzBhYTRjNDQzYTRhMDExMDg4YTYwODczMzgzIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5LaWtvJiMzOTtzPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzFjNDg3MDhmM2EwNTQ1NDU4ZmVlZjExNTk4ODgyNDU1LnNldENvbnRlbnQoaHRtbF8xYmM0NjMwYWE0YzQ0M2E0YTAxMTA4OGE2MDg3MzM4Myk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9lMzBlZmY1ODA5ZTA0OWNkYjVmZjdjZTllNWQwYTM0Mi5iaW5kUG9wdXAocG9wdXBfMWM0ODcwOGYzYTA1NDU0NThmZWVmMTE1OTg4ODI0NTUpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNGQ4NGRjMmY5ZWVmNDg4YTk1Y2Q5ZWJlM2ZmNmJhNzIgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45NjUxMywgLTgzLjkxOTg4XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF84YzgyZThjZTgwMzM0ZWMyYTc5ZGZiYzc0M2I5ODUxZCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMzI0M2ViNzFlMmI4NDE5OTg1ZDVkYzUxZjVlY2QzZTEgPSAkKGA8ZGl2IGlkPSJodG1sXzMyNDNlYjcxZTJiODQxOTk4NWQ1ZGM1MWY1ZWNkM2UxIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Ob3QgV2F0c29uJiMzOTtzIGtpdGNoZW4gKyBCYXI8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfOGM4MmU4Y2U4MDMzNGVjMmE3OWRmYmM3NDNiOTg1MWQuc2V0Q29udGVudChodG1sXzMyNDNlYjcxZTJiODQxOTk4NWQ1ZGM1MWY1ZWNkM2UxKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzRkODRkYzJmOWVlZjQ4OGE5NWNkOWViZTNmZjZiYTcyLmJpbmRQb3B1cChwb3B1cF84YzgyZThjZTgwMzM0ZWMyYTc5ZGZiYzc0M2I5ODUxZCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl8wMjdkNzA4M2I0Mjg0YzY5OWNiMDAyMTU5YmU4ZmM1ZiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjAyMDQxMzk4ODc3NTcsIC04NC4zMTA0NjcwNTY5MzAxXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9iYWZkZDk2YWNiMTg0YTFjODg5NzdkNmZmYTIyNDc2ZSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfZjcxODNiZTExY2I1NGUyZDljZGUyYjYyNjAxMGQwOGIgPSAkKGA8ZGl2IGlkPSJodG1sX2Y3MTgzYmUxMWNiNTRlMmQ5Y2RlMmI2MjYwMTBkMDhiIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Mb3MgVHJpb3MgTWV4aWNhbjwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9iYWZkZDk2YWNiMTg0YTFjODg5NzdkNmZmYTIyNDc2ZS5zZXRDb250ZW50KGh0bWxfZjcxODNiZTExY2I1NGUyZDljZGUyYjYyNjAxMGQwOGIpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfMDI3ZDcwODNiNDI4NGM2OTljYjAwMjE1OWJlOGZjNWYuYmluZFBvcHVwKHBvcHVwX2JhZmRkOTZhY2IxODRhMWM4ODk3N2Q2ZmZhMjI0NzZlKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzg4MmEwZmMzZTZlMDQxMjc4YWRiMDk2ZDMwNGJiZWVjID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuODc3NTY0NjYxODQ4LCAtODMuNzc2MDEyNzcwODMxNl0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfZjE4ZTU2NmQxNjVhNGM2N2JiYmM4MjQ2NzBlNmVkMmYgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzNjZDkwY2E2YzJjOTRkZWJhNGYzMDE2MDM1ZDk2OWRlID0gJChgPGRpdiBpZD0iaHRtbF8zY2Q5MGNhNmMyYzk0ZGViYTRmMzAxNjAzNWQ5NjlkZSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UGVsYW5jaG8mIzM5O3MgTWV4aWNhbiBHcmlsbDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9mMThlNTY2ZDE2NWE0YzY3YmJiYzgyNDY3MGU2ZWQyZi5zZXRDb250ZW50KGh0bWxfM2NkOTBjYTZjMmM5NGRlYmE0ZjMwMTYwMzVkOTY5ZGUpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfODgyYTBmYzNlNmUwNDEyNzhhZGIwOTZkMzA0YmJlZWMuYmluZFBvcHVwKHBvcHVwX2YxOGU1NjZkMTY1YTRjNjdiYmJjODI0NjcwZTZlZDJmKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzQyMmJkMGE5ZjQ0YTRhMDQ4NGJlMWIyZWQ0YTJiZThmID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuNzU2MjExLCAtODMuOTQ4ODU1XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF83OWMwNDAyMTBjN2U0YzgwODU1ZTUxYTE0NTJmMGVjOSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNjIxYjNmNmQ4NjMyNDZmZjk0MzJiYWI4NTQ2NWEwOGEgPSAkKGA8ZGl2IGlkPSJodG1sXzYyMWIzZjZkODYzMjQ2ZmY5NDMyYmFiODU0NjVhMDhhIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5FbCBKaW1hZG9yIE1leGljYW4gR3JpbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNzljMDQwMjEwYzdlNGM4MDg1NWU1MWExNDUyZjBlYzkuc2V0Q29udGVudChodG1sXzYyMWIzZjZkODYzMjQ2ZmY5NDMyYmFiODU0NjVhMDhhKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzQyMmJkMGE5ZjQ0YTRhMDQ4NGJlMWIyZWQ0YTJiZThmLmJpbmRQb3B1cChwb3B1cF83OWMwNDAyMTBjN2U0YzgwODU1ZTUxYTE0NTJmMGVjOSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl85OTk2OTlkZDgzMWI0ZGJlYTUzZDU5OTU3YWUxZDBhNCA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1LjkxMDI3LCAtODQuMDkxMzldLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzc4MTM3MmU5ZGQzMzQ0MmVhYjc4NDA2ODFkZTY3MGQ0ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF84ODkxMzlkMGY4NWI0NzgwOGM2Y2ViOTkyNGQ5Mjk0MyA9ICQoYDxkaXYgaWQ9Imh0bWxfODg5MTM5ZDBmODViNDc4MDhjNmNlYjk5MjRkOTI5NDMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlNhbHNhcml0YSYjMzk7cyBGcmVzaCBNZXhpY2FuIEdyaWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzc4MTM3MmU5ZGQzMzQ0MmVhYjc4NDA2ODFkZTY3MGQ0LnNldENvbnRlbnQoaHRtbF84ODkxMzlkMGY4NWI0NzgwOGM2Y2ViOTkyNGQ5Mjk0Myk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl85OTk2OTlkZDgzMWI0ZGJlYTUzZDU5OTU3YWUxZDBhNC5iaW5kUG9wdXAocG9wdXBfNzgxMzcyZTlkZDMzNDQyZWFiNzg0MDY4MWRlNjcwZDQpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfOWJlNjYzN2E1ZjExNDkxN2FmOWZhY2U5Zjc5ZmE1M2EgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45MjM3OTYyNDYzODM1LCAtODQuMDU2Nzg0NDQ4MzcwNF0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfYmY4Y2JkYjI0OTIxNDU1YzgzNGQ0NjJmMjk3ZWI2OTQgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2Q2ZWM5OGI4NmY5NzQ1ZjdhOGY0MGY1YjUyNWJmOWMwID0gJChgPGRpdiBpZD0iaHRtbF9kNmVjOThiODZmOTc0NWY3YThmNDBmNWI1MjViZjljMCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGFjb21leCBUYWNvIFRydWNrPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2JmOGNiZGIyNDkyMTQ1NWM4MzRkNDYyZjI5N2ViNjk0LnNldENvbnRlbnQoaHRtbF9kNmVjOThiODZmOTc0NWY3YThmNDBmNWI1MjViZjljMCk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl85YmU2NjM3YTVmMTE0OTE3YWY5ZmFjZTlmNzlmYTUzYS5iaW5kUG9wdXAocG9wdXBfYmY4Y2JkYjI0OTIxNDU1YzgzNGQ0NjJmMjk3ZWI2OTQpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfODI2ZDJiMzQ3YTJjNDhjMjk1YTZiYzMwMDY4NjQ4OGEgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNi4xMzIwMDEyLCAtODMuNzE0MzU0N10sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfYTBmOGYyZDVjMGRhNDZhODgxMWEyNzU5YWZiZTE2MjcgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2IyNTM1NTFhMDM2NjQyMWU4MmJmMDU1N2U4MGEzNWY2ID0gJChgPGRpdiBpZD0iaHRtbF9iMjUzNTUxYTAzNjY0MjFlODJiZjA1NTdlODBhMzVmNiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RWwgUGFpc2FqZSBNZXhpY2FuIEdyaWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2EwZjhmMmQ1YzBkYTQ2YTg4MTFhMjc1OWFmYmUxNjI3LnNldENvbnRlbnQoaHRtbF9iMjUzNTUxYTAzNjY0MjFlODJiZjA1NTdlODBhMzVmNik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl84MjZkMmIzNDdhMmM0OGMyOTVhNmJjMzAwNjg2NDg4YS5iaW5kUG9wdXAocG9wdXBfYTBmOGYyZDVjMGRhNDZhODgxMWEyNzU5YWZiZTE2MjcpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfMGJmNTEwNjU5YzVjNDU1NGE5Yjc4YzgwMmFkNGE4Y2EgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS44Mjk4Njg1MzU1MDYxLCAtODQuMTY3MDg2NTk0ODc5N10sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfYjc3YTYwMTg1NDRiNDcyN2IzZGQyZWIxYzZhMDgyNzMgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2IwNGI2ODZhZGExZjRlNDJhZTcxZGFjYzBhMWVlMDNjID0gJChgPGRpdiBpZD0iaHRtbF9iMDRiNjg2YWRhMWY0ZTQyYWU3MWRhY2MwYTFlZTAzYyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RG9uIEdhbGxvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2I3N2E2MDE4NTQ0YjQ3MjdiM2RkMmViMWM2YTA4MjczLnNldENvbnRlbnQoaHRtbF9iMDRiNjg2YWRhMWY0ZTQyYWU3MWRhY2MwYTFlZTAzYyk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl8wYmY1MTA2NTljNWM0NTU0YTliNzhjODAyYWQ0YThjYS5iaW5kUG9wdXAocG9wdXBfYjc3YTYwMTg1NDRiNDcyN2IzZGQyZWIxYzZhMDgyNzMpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNThhOGU3NzA3YjFmNGVhNjg0ZWQxOTNjMzc5NTM2YTEgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNi4xMjEyMDQ2NzQ1MjI1LCAtODQuMTE0NTQ1MDk5NDM3Ml0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNDM2MWM5OWQ5MGRkNDBkMTliM2Y3MzhiY2E2ZjExMDQgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzY3ZGNmODJiZTljZDQwMzg5MTAxNDUwNmUyZjNlNWM3ID0gJChgPGRpdiBpZD0iaHRtbF82N2RjZjgyYmU5Y2Q0MDM4OTEwMTQ1MDZlMmYzZTVjNyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2FzYSBkb24gUGVkcm88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNDM2MWM5OWQ5MGRkNDBkMTliM2Y3MzhiY2E2ZjExMDQuc2V0Q29udGVudChodG1sXzY3ZGNmODJiZTljZDQwMzg5MTAxNDUwNmUyZjNlNWM3KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzU4YThlNzcwN2IxZjRlYTY4NGVkMTkzYzM3OTUzNmExLmJpbmRQb3B1cChwb3B1cF80MzYxYzk5ZDkwZGQ0MGQxOWIzZjczOGJjYTZmMTEwNCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl80MzU0MjdhNjg5OTQ0MzgyYjYzNzM5ZmE4NzJiMTRhMyA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljg4MjUyMzgwNDkwMywgLTg0LjE1NzU1MTMwMzUwNTldLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2Q0MTFkNWFjMTgxZjQyYzVhOWYzZGMxNjczMDQ4NDEzID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9mZGU0MTFhZTEwZmE0ZTk4OTkxNjI5MWI3MGEwZDc5MCA9ICQoYDxkaXYgaWQ9Imh0bWxfZmRlNDExYWUxMGZhNGU5ODk5MTYyOTFiNzBhMGQ3OTAiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkF6dWwgVGVxdWlsYTwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9kNDExZDVhYzE4MWY0MmM1YTlmM2RjMTY3MzA0ODQxMy5zZXRDb250ZW50KGh0bWxfZmRlNDExYWUxMGZhNGU5ODk5MTYyOTFiNzBhMGQ3OTApOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNDM1NDI3YTY4OTk0NDM4MmI2MzczOWZhODcyYjE0YTMuYmluZFBvcHVwKHBvcHVwX2Q0MTFkNWFjMTgxZjQyYzVhOWYzZGMxNjczMDQ4NDEzKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzgwYjc1MjQ0NTU0MjRkOTk4YTBkMGQxODlmYTczMzVhID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuNzE3OTMzODc0NTE5LCAtODQuMDEwMzExMTA0MzU3Ml0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfOWVmYmJmYWVjYzAxNGE2MGE5OWQ3NjU5ZDRlYzI2N2YgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2M4NDBiODUwOGI1ZjRlNzk4ZjA3ZGVkMzI3ODRjZTc0ID0gJChgPGRpdiBpZD0iaHRtbF9jODQwYjg1MDhiNWY0ZTc5OGYwN2RlZDMyNzg0Y2U3NCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RWwgQmFycmlsIE1leGljYW4gR3JpbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfOWVmYmJmYWVjYzAxNGE2MGE5OWQ3NjU5ZDRlYzI2N2Yuc2V0Q29udGVudChodG1sX2M4NDBiODUwOGI1ZjRlNzk4ZjA3ZGVkMzI3ODRjZTc0KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzgwYjc1MjQ0NTU0MjRkOTk4YTBkMGQxODlmYTczMzVhLmJpbmRQb3B1cChwb3B1cF85ZWZiYmZhZWNjMDE0YTYwYTk5ZDc2NTlkNGVjMjY3ZikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl85MWQ2YjVjYjAyMjI0ZTJkOTgyNzUxNzFmOGY3MDk1YyA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1LjcxNTI4LCAtODQuMDE4OTRdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzNmYjIwOTk1ODdmNjQ0ZGY4NWMzYmQ1YTA0ZTEwODM3ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9jZjFjYTQzZTkzZGQ0MDE2OWI1OWJiOGQ3NTVlNGMwNiA9ICQoYDxkaXYgaWQ9Imh0bWxfY2YxY2E0M2U5M2RkNDAxNjliNTliYjhkNzU1ZTRjMDYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkxhcyBNYXJnYXJpdGFzIE1leGljYW4gR3JpbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfM2ZiMjA5OTU4N2Y2NDRkZjg1YzNiZDVhMDRlMTA4Mzcuc2V0Q29udGVudChodG1sX2NmMWNhNDNlOTNkZDQwMTY5YjU5YmI4ZDc1NWU0YzA2KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzkxZDZiNWNiMDIyMjRlMmQ5ODI3NTE3MWY4ZjcwOTVjLmJpbmRQb3B1cChwb3B1cF8zZmIyMDk5NTg3ZjY0NGRmODVjM2JkNWEwNGUxMDgzNykKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl81ZjZhMjFiYWE0OTM0MGVkYmI5MWFhZGVhNDVkMGI1ZCA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljc5Mjc5NzQyMTgxNjgsIC04NC4yNjE1Mjg2Nzk1NDA3XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF83ZGRkYzU2ZjRmMWQ0ZmE0YWNhN2VmZTdhNTY0OWZhZSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNDkyMzVmNDU1OWM0NDE4NzhiNmE1YWRiODM1Njg1YTMgPSAkKGA8ZGl2IGlkPSJodG1sXzQ5MjM1ZjQ1NTljNDQxODc4YjZhNWFkYjgzNTY4NWEzIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5QYW5hZGVyaWEgWSBQYXN0ZWxlcmlhIEFndWlsYTwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF83ZGRkYzU2ZjRmMWQ0ZmE0YWNhN2VmZTdhNTY0OWZhZS5zZXRDb250ZW50KGh0bWxfNDkyMzVmNDU1OWM0NDE4NzhiNmE1YWRiODM1Njg1YTMpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNWY2YTIxYmFhNDkzNDBlZGJiOTFhYWRlYTQ1ZDBiNWQuYmluZFBvcHVwKHBvcHVwXzdkZGRjNTZmNGYxZDRmYTRhY2E3ZWZlN2E1NjQ5ZmFlKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzNmYmU5NGZlNDU0MzRjZGM4YTZjMTEyMTA5NTBkMDRmID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuNjgzNjIsIC04NC4yNjc3NTZdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzMzMWRiOGYwOWMyODQzMDE4OTQwYTNiZDUyOThlOGRlID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF83NDU4ZDZhMjlhOWQ0Zjc3ODJiMDM3Y2ZjMTc2OTI0OSA9ICQoYDxkaXYgaWQ9Imh0bWxfNzQ1OGQ2YTI5YTlkNGY3NzgyYjAzN2NmYzE3NjkyNDkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkxvcmVuem8mIzM5O3MgTWV4aWNhbiBHcmlsbGU8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMzMxZGI4ZjA5YzI4NDMwMTg5NDBhM2JkNTI5OGU4ZGUuc2V0Q29udGVudChodG1sXzc0NThkNmEyOWE5ZDRmNzc4MmIwMzdjZmMxNzY5MjQ5KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzNmYmU5NGZlNDU0MzRjZGM4YTZjMTEyMTA5NTBkMDRmLmJpbmRQb3B1cChwb3B1cF8zMzFkYjhmMDljMjg0MzAxODk0MGEzYmQ1Mjk4ZThkZSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9jODU5ODRkYTUzYzU0ZGJlOWM2OGQxOGYxZjg3ZDI3NyA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljk1NjYyMDA3MTkxOTEsIC04My45MzI3NjMyNjYzNjQyXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8wY2Y4NDJjYzQ2N2Y0NDhlODc1YTUwOTZiODAxMWMxNSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfZDg0NWVlODhkN2MxNGQyNDhhM2M0OTA5NzQxODEwMjAgPSAkKGA8ZGl2IGlkPSJodG1sX2Q4NDVlZTg4ZDdjMTRkMjQ4YTNjNDkwOTc0MTgxMDIwIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DaGlwb3RsZSBNZXhpY2FuIEdyaWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzBjZjg0MmNjNDY3ZjQ0OGU4NzVhNTA5NmI4MDExYzE1LnNldENvbnRlbnQoaHRtbF9kODQ1ZWU4OGQ3YzE0ZDI0OGEzYzQ5MDk3NDE4MTAyMCk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9jODU5ODRkYTUzYzU0ZGJlOWM2OGQxOGYxZjg3ZDI3Ny5iaW5kUG9wdXAocG9wdXBfMGNmODQyY2M0NjdmNDQ4ZTg3NWE1MDk2YjgwMTFjMTUpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfMjc4ODBiZDIxOTgxNDA1ODkzMWQyZjhmNzAwNTVhNjUgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS43ODc0NzAzNjA3OTk1LCAtODQuMjcwODk1NTEyOTY5NV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMjE5MWFmMzAwYzNhNGYxYjhlZmNhZTgxMDg0YzU4ZWUgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2ZlZWQ5NzEyY2NkOTQ3MDU4ZjQxZmI1ZjRmMjBhZThmID0gJChgPGRpdiBpZD0iaHRtbF9mZWVkOTcxMmNjZDk0NzA1OGY0MWZiNWY0ZjIwYWU4ZiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TGEgTHVwaXRhIC0gTGVub2lyIENpdHk8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMjE5MWFmMzAwYzNhNGYxYjhlZmNhZTgxMDg0YzU4ZWUuc2V0Q29udGVudChodG1sX2ZlZWQ5NzEyY2NkOTQ3MDU4ZjQxZmI1ZjRmMjBhZThmKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzI3ODgwYmQyMTk4MTQwNTg5MzFkMmY4ZjcwMDU1YTY1LmJpbmRQb3B1cChwb3B1cF8yMTkxYWYzMDBjM2E0ZjFiOGVmY2FlODEwODRjNThlZSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9jMmQxZjFjNGJjOWY0YWVjYmZjZWExMTZhMjk3NWI5ZSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljk1MzY2MywgLTgzLjkzOTEzXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF82MWRkMTA4Y2UxNjk0ODM2YTA4YzlhOTY3YWRkMmM5OSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNGY3NzYxMTY0YmRmNDU2Mzk5ZTJjNWI2ZDgxZTNjYTEgPSAkKGA8ZGl2IGlkPSJodG1sXzRmNzc2MTE2NGJkZjQ1NjM5OWUyYzViNmQ4MWUzY2ExIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5TdW5zcG90PC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzYxZGQxMDhjZTE2OTQ4MzZhMDhjOWE5NjdhZGQyYzk5LnNldENvbnRlbnQoaHRtbF80Zjc3NjExNjRiZGY0NTYzOTllMmM1YjZkODFlM2NhMSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9jMmQxZjFjNGJjOWY0YWVjYmZjZWExMTZhMjk3NWI5ZS5iaW5kUG9wdXAocG9wdXBfNjFkZDEwOGNlMTY5NDgzNmEwOGM5YTk2N2FkZDJjOTkpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfMWIxNGUzYmZhMjJjNGMyMjhlOGE0ODhjOWQ0N2I0MWMgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45Mjk4MSwgLTg0LjAzMTI3XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9hNDU1YjI4OWRkNjY0NzUyYTUzNTBlYWZmOGMwODFiYSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYTM3YTcyODM4ZWM0NGYwODgxOGQwMTU2OWIxNGNiYWMgPSAkKGA8ZGl2IGlkPSJodG1sX2EzN2E3MjgzOGVjNDRmMDg4MThkMDE1NjliMTRjYmFjIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DaGlsaSYjMzk7czwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9hNDU1YjI4OWRkNjY0NzUyYTUzNTBlYWZmOGMwODFiYS5zZXRDb250ZW50KGh0bWxfYTM3YTcyODM4ZWM0NGYwODgxOGQwMTU2OWIxNGNiYWMpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfMWIxNGUzYmZhMjJjNGMyMjhlOGE0ODhjOWQ0N2I0MWMuYmluZFBvcHVwKHBvcHVwX2E0NTViMjg5ZGQ2NjQ3NTJhNTM1MGVhZmY4YzA4MWJhKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzUyOGFjYTE4YmYyYTRiYzg5YjZlNmRiZTEwYjQ2NWIxID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMDA2MzA2MzIwOTQzMywgLTg0LjAyMTE0ODY0NTc5MTZdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzEyNmNlNzBjZGVjNzRkMmZiNWNjNjg2ODFkOTdkOThhID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9mOWY1YjVhODhkZGQ0OGM1YmRiODdhOWZhNTRlMmM4YyA9ICQoYDxkaXYgaWQ9Imh0bWxfZjlmNWI1YTg4ZGRkNDhjNWJkYjg3YTlmYTU0ZTJjOGMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNoaWxpJiMzOTtzPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzEyNmNlNzBjZGVjNzRkMmZiNWNjNjg2ODFkOTdkOThhLnNldENvbnRlbnQoaHRtbF9mOWY1YjVhODhkZGQ0OGM1YmRiODdhOWZhNTRlMmM4Yyk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl81MjhhY2ExOGJmMmE0YmM4OWI2ZTZkYmUxMGI0NjViMS5iaW5kUG9wdXAocG9wdXBfMTI2Y2U3MGNkZWM3NGQyZmI1Y2M2ODY4MWQ5N2Q5OGEpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNGUyODBlOGMyYzhkNDkyNGJkOTQxZGQ4ZDhlMzQyNTQgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45NTUwNCwgLTgzLjkzNTczXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF80OWQ0YzAwZmM0OWI0Y2Y5ODU3MzYzZmZhMzE2ZTE0OCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYTIxYzYzNTE3ZjBjNDRiMzk5ZjAxOGRjZTcyM2U0ZDQgPSAkKGA8ZGl2IGlkPSJodG1sX2EyMWM2MzUxN2YwYzQ0YjM5OWYwMThkY2U3MjNlNGQ0IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5IYXBweSBUYWNvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzQ5ZDRjMDBmYzQ5YjRjZjk4NTczNjNmZmEzMTZlMTQ4LnNldENvbnRlbnQoaHRtbF9hMjFjNjM1MTdmMGM0NGIzOTlmMDE4ZGNlNzIzZTRkNCk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl80ZTI4MGU4YzJjOGQ0OTI0YmQ5NDFkZDhkOGUzNDI1NC5iaW5kUG9wdXAocG9wdXBfNDlkNGMwMGZjNDliNGNmOTg1NzM2M2ZmYTMxNmUxNDgpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfYzhiNzM5Yjg5MGVhNDNhNTllZmNhNDFiYTY4MjVhMWIgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNi4wMDY0NzY1NDYzNzExLCAtODQuMjU1MDMwMzQ4NTI3OF0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfZjA0ODU1YTllMDhjNDNjMmEzZDg1NDk5ZWMyZDg4OTcgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2M1ZWE3MTkxNWZkNDQyMzc5ZjJjYWQxNzAwOTUzZDFiID0gJChgPGRpdiBpZD0iaHRtbF9jNWVhNzE5MTVmZDQ0MjM3OWYyY2FkMTcwMDk1M2QxYiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U2Fsc2FyaXRhJiMzOTtzIEZyZXNoIE1leGljYW4gR3JpbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfZjA0ODU1YTllMDhjNDNjMmEzZDg1NDk5ZWMyZDg4OTcuc2V0Q29udGVudChodG1sX2M1ZWE3MTkxNWZkNDQyMzc5ZjJjYWQxNzAwOTUzZDFiKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2M4YjczOWI4OTBlYTQzYTU5ZWZjYTQxYmE2ODI1YTFiLmJpbmRQb3B1cChwb3B1cF9mMDQ4NTVhOWUwOGM0M2MyYTNkODU0OTllYzJkODg5NykKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl85ZDlhYzFhODA1NGQ0YzIyOTBkNWI5YmRlY2Q1ODgyNyA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1LjkyNDE2NCwgLTg0LjAzODI5MV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMmEzM2Q1MzgxODBiNDdmZWEwNWIyMzQ3YWViNWFjODIgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzRhMDc2YzJmZmI0ZjQzN2Y5MGQ0OWU5ZTY2MjczZjc2ID0gJChgPGRpdiBpZD0iaHRtbF80YTA3NmMyZmZiNGY0MzdmOTBkNDllOWU2NjI3M2Y3NiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UGV0cm8mIzM5O3MgQ2hpbGkgJmFtcDsgQ2hpcHMtV2VzdCBUb3duIE1hbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMmEzM2Q1MzgxODBiNDdmZWEwNWIyMzQ3YWViNWFjODIuc2V0Q29udGVudChodG1sXzRhMDc2YzJmZmI0ZjQzN2Y5MGQ0OWU5ZTY2MjczZjc2KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzlkOWFjMWE4MDU0ZDRjMjI5MGQ1YjliZGVjZDU4ODI3LmJpbmRQb3B1cChwb3B1cF8yYTMzZDUzODE4MGI0N2ZlYTA1YjIzNDdhZWI1YWM4MikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9iZjIwZGU4YTg1ZGQ0ODdlYWRiMTAwZjY3YjAxOTU0MyA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjAxMjAxMSwgLTgzLjkyMjY5XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF83MTcxZTU5OTlhMTY0Y2JlOTZlYTQyMDdhMDk4NmM3YyA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYjZkNTRjOTFhNzg2NGY1NjgzNzk4OWU4MGNkZWZiNjUgPSAkKGA8ZGl2IGlkPSJodG1sX2I2ZDU0YzkxYTc4NjRmNTY4Mzc5ODllODBjZGVmYjY1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UaW8gQ29uZWpvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzcxNzFlNTk5OWExNjRjYmU5NmVhNDIwN2EwOTg2YzdjLnNldENvbnRlbnQoaHRtbF9iNmQ1NGM5MWE3ODY0ZjU2ODM3OTg5ZTgwY2RlZmI2NSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9iZjIwZGU4YTg1ZGQ0ODdlYWRiMTAwZjY3YjAxOTU0My5iaW5kUG9wdXAocG9wdXBfNzE3MWU1OTk5YTE2NGNiZTk2ZWE0MjA3YTA5ODZjN2MpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfYTZmM2ZlZjA0OTIyNDc1ZTkzMjNmN2M0ZDM1Y2M5YzYgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45MzIxODA3LCAtODQuMDIyNzU1MV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfYTMwZWEzYjgxY2JkNDNlMjgwZWZhYTcyMzA5MzVhZjEgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2E1YWZkODg5ZWRmNDQ2N2Y5YmExZTk4Y2U1NmFmZmUyID0gJChgPGRpdiBpZD0iaHRtbF9hNWFmZDg4OWVkZjQ0NjdmOWJhMWU5OGNlNTZhZmZlMiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UGV0cm8mIzM5O3MgQ2hpbGkgJmFtcDsgQ2hpcHMtV2VzdCBIaWxsczwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9hMzBlYTNiODFjYmQ0M2UyODBlZmFhNzIzMDkzNWFmMS5zZXRDb250ZW50KGh0bWxfYTVhZmQ4ODllZGY0NDY3ZjliYTFlOThjZTU2YWZmZTIpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfYTZmM2ZlZjA0OTIyNDc1ZTkzMjNmN2M0ZDM1Y2M5YzYuYmluZFBvcHVwKHBvcHVwX2EzMGVhM2I4MWNiZDQzZTI4MGVmYWE3MjMwOTM1YWYxKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzBkNjhiN2MzZTEzZDQ5ZTU4MzQ1OTMxZTQzNzQ3YTBhID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMDA2MzU4LCAtODQuMDIxOTU1XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9lNzY2OWQzN2QzMmU0YzBmOGYzYWE5NTE5OGI3ZDdjMyA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfOGViNWJhYzM3ZDg3NDM5M2I3YzM5YmE5MzM2Nzk1OTggPSAkKGA8ZGl2IGlkPSJodG1sXzhlYjViYWMzN2Q4NzQzOTNiN2MzOWJhOTMzNjc5NTk4IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5TYWxzYXJpdGEmIzM5O3MgRnJlc2ggTWV4aWNhbiBHcmlsbDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9lNzY2OWQzN2QzMmU0YzBmOGYzYWE5NTE5OGI3ZDdjMy5zZXRDb250ZW50KGh0bWxfOGViNWJhYzM3ZDg3NDM5M2I3YzM5YmE5MzM2Nzk1OTgpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfMGQ2OGI3YzNlMTNkNDllNTgzNDU5MzFlNDM3NDdhMGEuYmluZFBvcHVwKHBvcHVwX2U3NjY5ZDM3ZDMyZTRjMGY4ZjNhYTk1MTk4YjdkN2MzKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzMxNTQ2NTYwMThhZTQ3ZTBhMjg2Y2NjNmVkZDRjZjFkID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuODIwNjA3MDIxNDUxLCAtODQuMjY4OTgyMzM1OTI1MV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfZjFiOTFlZTA4ZmIwNGJjOTk2ZDhhODNmOGVhODUzMjUgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzlkYzVkYjBhMTEwZDQ3MDFhMmE1OGRmMzdhOWI2ODE1ID0gJChgPGRpdiBpZD0iaHRtbF85ZGM1ZGIwYTExMGQ0NzAxYTJhNThkZjM3YTliNjgxNSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+SGFjaWVuZGEgQXlhbGEmIzM5O3M8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfZjFiOTFlZTA4ZmIwNGJjOTk2ZDhhODNmOGVhODUzMjUuc2V0Q29udGVudChodG1sXzlkYzVkYjBhMTEwZDQ3MDFhMmE1OGRmMzdhOWI2ODE1KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzMxNTQ2NTYwMThhZTQ3ZTBhMjg2Y2NjNmVkZDRjZjFkLmJpbmRQb3B1cChwb3B1cF9mMWI5MWVlMDhmYjA0YmM5OTZkOGE4M2Y4ZWE4NTMyNSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl8wZDBlMGZhNDBmY2I0YWJiYTFiMjZhYjQ3YzkyZjY1ZiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjAwNDUwNjUsIC04NC4yNDg0OTgxN10sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfOWU2ZTE2N2E4YTgyNGVkZDhiNDBkMjE2NWFmMGI3MGUgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzJiMTg5ZGMzYWVkMTRmNGNiYjQ3YjdlNzNkNTdhZDgzID0gJChgPGRpdiBpZD0iaHRtbF8yYjE4OWRjM2FlZDE0ZjRjYmI0N2I3ZTczZDU3YWQ4MyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RG9zQnJvcyBGcmVzaCBNZXhpY2FuIEdyaWxsIC0gT2FrIFJpZGdlPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzllNmUxNjdhOGE4MjRlZGQ4YjQwZDIxNjVhZjBiNzBlLnNldENvbnRlbnQoaHRtbF8yYjE4OWRjM2FlZDE0ZjRjYmI0N2I3ZTczZDU3YWQ4Myk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl8wZDBlMGZhNDBmY2I0YWJiYTFiMjZhYjQ3YzkyZjY1Zi5iaW5kUG9wdXAocG9wdXBfOWU2ZTE2N2E4YTgyNGVkZDhiNDBkMjE2NWFmMGI3MGUpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfZTgzZTAwMWY4NTgxNDI0NjhjMDI3ZWFmZWVhODM4MzYgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45MjY5OSwgLTg0LjA0MzNdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzNkOGRiYmVhZGJlNDRlZDdhMGI1MmRhMmE3ZTg0NDA2ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8xYWI1NmI2OTVhZWE0MzJmOTI4N2VlMGZiYjljOTM4ZiA9ICQoYDxkaXYgaWQ9Imh0bWxfMWFiNTZiNjk1YWVhNDMyZjkyODdlZTBmYmI5YzkzOGYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlNhbHNhcml0YSYjMzk7cyBGcmVzaCBNZXhpY2FuIEdyaWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzNkOGRiYmVhZGJlNDRlZDdhMGI1MmRhMmE3ZTg0NDA2LnNldENvbnRlbnQoaHRtbF8xYWI1NmI2OTVhZWE0MzJmOTI4N2VlMGZiYjljOTM4Zik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9lODNlMDAxZjg1ODE0MjQ2OGMwMjdlYWZlZWE4MzgzNi5iaW5kUG9wdXAocG9wdXBfM2Q4ZGJiZWFkYmU0NGVkN2EwYjUyZGEyYTdlODQ0MDYpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfM2E0MjkyMGM0NWQ2NDlkZGFkYWIzNjE4ZjA1YmJmNjQgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNi4yNDAyOTE2LCAtODMuODE4MDkyM10sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfODk5ZTM0ZDhhYmNlNGI4MDljNTE0NDAxNzU2YTA5OGYgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2VmNmZjOTQyMzhlMjQyMGY5ZTgzOWMxNjFjOTk3ZTFlID0gJChgPGRpdiBpZD0iaHRtbF9lZjZmYzk0MjM4ZTI0MjBmOWU4MzljMTYxYzk5N2UxZSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RWwgTWFyaWFjaGkgTWV4aWNhbiBSZXN0YXVyYW50PC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzg5OWUzNGQ4YWJjZTRiODA5YzUxNDQwMTc1NmEwOThmLnNldENvbnRlbnQoaHRtbF9lZjZmYzk0MjM4ZTI0MjBmOWU4MzljMTYxYzk5N2UxZSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl8zYTQyOTIwYzQ1ZDY0OWRkYWRhYjM2MThmMDViYmY2NC5iaW5kUG9wdXAocG9wdXBfODk5ZTM0ZDhhYmNlNGI4MDljNTE0NDAxNzU2YTA5OGYpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfMWUxY2ZjOTljNThmNGFlYmFjNThjNzNlMWNmNjdiZTEgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS44NjIwMzI4NjY3MjAzLCAtODQuMDY2MTI3MTI5OTA3NV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfYzgzY2QyMzIyODcwNDM4MzlhMmRhYWZiN2RjMzFjMzMgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzk5NWZkN2RjMGNkZjQxYzA5MjhjNmI0YmVlMTdmYTIxID0gJChgPGRpdiBpZD0iaHRtbF85OTVmZDdkYzBjZGY0MWMwOTI4YzZiNGJlZTE3ZmEyMSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TW9lJiMzOTtzIFNvdXRod2VzdCBHcmlsbDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9jODNjZDIzMjI4NzA0MzgzOWEyZGFhZmI3ZGMzMWMzMy5zZXRDb250ZW50KGh0bWxfOTk1ZmQ3ZGMwY2RmNDFjMDkyOGM2YjRiZWUxN2ZhMjEpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfMWUxY2ZjOTljNThmNGFlYmFjNThjNzNlMWNmNjdiZTEuYmluZFBvcHVwKHBvcHVwX2M4M2NkMjMyMjg3MDQzODM5YTJkYWFmYjdkYzMxYzMzKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzA3NzdhZDJmMzU1MzRiMGQ5YWRiMGJmZmE4N2VmYjhmID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTY5NjY2LCAtODMuOTE4MzI3XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9iMjJiMjkxODE4NGI0NDQzOGQ4YTMxYmY1ODVjNmFlYiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfZjU0Y2JkY2RmNTljNDY5N2I2YWQ5NTE2OTMzM2ZiMTggPSAkKGA8ZGl2IGlkPSJodG1sX2Y1NGNiZGNkZjU5YzQ2OTdiNmFkOTUxNjkzMzNmYjE4IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5PbGlCZWE8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfYjIyYjI5MTgxODRiNDQ0MzhkOGEzMWJmNTg1YzZhZWIuc2V0Q29udGVudChodG1sX2Y1NGNiZGNkZjU5YzQ2OTdiNmFkOTUxNjkzMzNmYjE4KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzA3NzdhZDJmMzU1MzRiMGQ5YWRiMGJmZmE4N2VmYjhmLmJpbmRQb3B1cChwb3B1cF9iMjJiMjkxODE4NGI0NDQzOGQ4YTMxYmY1ODVjNmFlYikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl8zNDYyNDQ2MTk4ZTE0MDkzOWEzYmY4OGVkY2IxYjk1ZiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljk0MjA5LCAtODMuOTg0NDFdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2IxYTA3ODEyOWZiMTRhM2VhZjA2YTI0NjY3OGFiNTk1ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8zMGU5N2UxMzRjZGQ0NDk3YmI4NGY5ZjUwZDQzZDg0NSA9ICQoYDxkaXYgaWQ9Imh0bWxfMzBlOTdlMTM0Y2RkNDQ5N2JiODRmOWY1MGQ0M2Q4NDUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNvbG9uZWwmIzM5O3MgQ2FmZTwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9iMWEwNzgxMjlmYjE0YTNlYWYwNmEyNDY2NzhhYjU5NS5zZXRDb250ZW50KGh0bWxfMzBlOTdlMTM0Y2RkNDQ5N2JiODRmOWY1MGQ0M2Q4NDUpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfMzQ2MjQ0NjE5OGUxNDA5MzlhM2JmODhlZGNiMWI5NWYuYmluZFBvcHVwKHBvcHVwX2IxYTA3ODEyOWZiMTRhM2VhZjA2YTI0NjY3OGFiNTk1KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2ZmN2VhYmU5MWM3YjRjYTY4OWU1NzY5MTJjMmYxNmVmID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTI1NTA5LCAtODQuMDM3ODE4XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF81OWYzNjUzZjQzZmU0Mzc0YmU4NDA5MmYwYjYxZTllNSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfM2JlNGQ2MzQ3ZDM3NGJmMWI0ZTg0NDk3NWRjNGUwM2IgPSAkKGA8ZGl2IGlkPSJodG1sXzNiZTRkNjM0N2QzNzRiZjFiNGU4NDQ5NzVkYzRlMDNiIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DaGlwb3RsZSBNZXhpY2FuIEdyaWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzU5ZjM2NTNmNDNmZTQzNzRiZTg0MDkyZjBiNjFlOWU1LnNldENvbnRlbnQoaHRtbF8zYmU0ZDYzNDdkMzc0YmYxYjRlODQ0OTc1ZGM0ZTAzYik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9mZjdlYWJlOTFjN2I0Y2E2ODllNTc2OTEyYzJmMTZlZi5iaW5kUG9wdXAocG9wdXBfNTlmMzY1M2Y0M2ZlNDM3NGJlODQwOTJmMGI2MWU5ZTUpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNWMwZWI0NTllMWYyNDcyNjg1MzgwNWYxYWU0Y2FkYWMgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNi4wMzM2MjY2LCAtODMuOTMwODkyOV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfYWE1ZWNlOTFiMmMxNDA1N2IzNDJiZmUzYjZmM2Y2ZDggPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzY1OGI4N2FjODI1YzQ5ZGE4YTU2Yjc5MDI3ZThhOGRlID0gJChgPGRpdiBpZD0iaHRtbF82NThiODdhYzgyNWM0OWRhOGE1NmI3OTAyN2U4YThkZSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U2Fsc2FyaXRhcyBGcmVzaCBNZXhpY2FuIEdyaWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2FhNWVjZTkxYjJjMTQwNTdiMzQyYmZlM2I2ZjNmNmQ4LnNldENvbnRlbnQoaHRtbF82NThiODdhYzgyNWM0OWRhOGE1NmI3OTAyN2U4YThkZSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl81YzBlYjQ1OWUxZjI0NzI2ODUzODA1ZjFhZTRjYWRhYy5iaW5kUG9wdXAocG9wdXBfYWE1ZWNlOTFiMmMxNDA1N2IzNDJiZmUzYjZmM2Y2ZDgpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfMWM5YTJlZTkwYTY3NGU4ZGFiYWU1ZGJlYWUyNjdlMmQgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45NjQxMjI3NzIyMTY4LCAtODMuOTIwNTg1NjMyMzI0Ml0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfOTEyMmE2YzdkNDJkNGFjZDgzMDMwMTA5M2NkYmE2NzkgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzg2OTdiMmI4MjY4NjRhOTVhZTJmOTA2NzcwMjJkNDhmID0gJChgPGRpdiBpZD0iaHRtbF84Njk3YjJiODI2ODY0YTk1YWUyZjkwNjc3MDIyZDQ4ZiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2hlZiBBYXJvbiYjMzk7cyBDcmVhdGlvbnM8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfOTEyMmE2YzdkNDJkNGFjZDgzMDMwMTA5M2NkYmE2Nzkuc2V0Q29udGVudChodG1sXzg2OTdiMmI4MjY4NjRhOTVhZTJmOTA2NzcwMjJkNDhmKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzFjOWEyZWU5MGE2NzRlOGRhYmFlNWRiZWFlMjY3ZTJkLmJpbmRQb3B1cChwb3B1cF85MTIyYTZjN2Q0MmQ0YWNkODMwMzAxMDkzY2RiYTY3OSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9iOWY4NmI3M2ZlZGM0NmJiYWE0NjExMDhmODc5YjE3NSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1LjkxMzAwODQ1NzM0MSwgLTgzLjk1NTcyMzE2MTQ3MV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNTkzODZiMTE0YmJhNDJhM2JlZDllZDMyN2ZiNTI2YWIgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2I3MDkxMGFiZWYzODRiZGU5ZGZkOTJhMjFkN2EwMTBjID0gJChgPGRpdiBpZD0iaHRtbF9iNzA5MTBhYmVmMzg0YmRlOWRmZDkyYTIxZDdhMDEwYyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RWwgUHVscG8gTG9jbyBLbm94PC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzU5Mzg2YjExNGJiYTQyYTNiZWQ5ZWQzMjdmYjUyNmFiLnNldENvbnRlbnQoaHRtbF9iNzA5MTBhYmVmMzg0YmRlOWRmZDkyYTIxZDdhMDEwYyk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9iOWY4NmI3M2ZlZGM0NmJiYWE0NjExMDhmODc5YjE3NS5iaW5kUG9wdXAocG9wdXBfNTkzODZiMTE0YmJhNDJhM2JlZDllZDMyN2ZiNTI2YWIpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfYzVmNDQyMTk1ZGQ4NDQwMmI1Njg3MTM5YmJkNmMxNmEgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45NTU2Mzg5LCAtODMuOTMzOTgyOF0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMjgxYmY1ZGQxNGJhNDU4YWFhMzUxYzljNWM0NjRiNzMgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzE5MzhiNTA3ZGY1ZjQxZGY5NWM5NjU2NDg1NmE4MWQ0ID0gJChgPGRpdiBpZD0iaHRtbF8xOTM4YjUwN2RmNWY0MWRmOTVjOTY1NjQ4NTZhODFkNCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TW9lJiMzOTtzIFNvdXRod2VzdCBHcmlsbDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8yODFiZjVkZDE0YmE0NThhYWEzNTFjOWM1YzQ2NGI3My5zZXRDb250ZW50KGh0bWxfMTkzOGI1MDdkZjVmNDFkZjk1Yzk2NTY0ODU2YTgxZDQpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfYzVmNDQyMTk1ZGQ4NDQwMmI1Njg3MTM5YmJkNmMxNmEuYmluZFBvcHVwKHBvcHVwXzI4MWJmNWRkMTRiYTQ1OGFhYTM1MWM5YzVjNDY0YjczKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2ZmMGM3NDU0NWRkYTRmNmRhMDkyZGZhZmY4YTBjZjYzID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTE1MzgsIC04NC4wOTExXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF80MjU2ZmFmNDk3NWU0MTdjYmE2N2M0NjdiMGNhMTIxOCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfZGE5Nzg4MWJiZGQzNGQ1M2EwMmYyODAwZDY5NTI4OTcgPSAkKGA8ZGl2IGlkPSJodG1sX2RhOTc4ODFiYmRkMzRkNTNhMDJmMjgwMGQ2OTUyODk3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DSiYjMzk7cyBUYWNvczwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF80MjU2ZmFmNDk3NWU0MTdjYmE2N2M0NjdiMGNhMTIxOC5zZXRDb250ZW50KGh0bWxfZGE5Nzg4MWJiZGQzNGQ1M2EwMmYyODAwZDY5NTI4OTcpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfZmYwYzc0NTQ1ZGRhNGY2ZGEwOTJkZmFmZjhhMGNmNjMuYmluZFBvcHVwKHBvcHVwXzQyNTZmYWY0OTc1ZTQxN2NiYTY3YzQ2N2IwY2ExMjE4KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2M2ZDBlYjQyMTMzYTQ2ZjNhYTA4MjQ0OWU0NDhhODI0ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTY1NTksIC04My45MjAyODFdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2EzNWQ0NDQ5NmY4MTQxY2ViMWJhZWQ3YmUwNzA3OTNhID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9kZTcyZmRlNGZkYWU0ODk5YWY3ZmE2YWI1NDk5MzQ1MiA9ICQoYDxkaXYgaWQ9Imh0bWxfZGU3MmZkZTRmZGFlNDg5OWFmN2ZhNmFiNTQ5OTM0NTIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlN0b2NrICZhbXA7IEJhcnJlbDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9hMzVkNDQ0OTZmODE0MWNlYjFiYWVkN2JlMDcwNzkzYS5zZXRDb250ZW50KGh0bWxfZGU3MmZkZTRmZGFlNDg5OWFmN2ZhNmFiNTQ5OTM0NTIpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfYzZkMGViNDIxMzNhNDZmM2FhMDgyNDQ5ZTQ0OGE4MjQuYmluZFBvcHVwKHBvcHVwX2EzNWQ0NDQ5NmY4MTQxY2ViMWJhZWQ3YmUwNzA3OTNhKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2U1ZmM5MjYwMzZiMTRjOGNiMzYzZWRlYWMxZTg1OWIwID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTY1OTAzNywgLTgzLjkxODQ5NzNdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzM0ZWQ1ZGZkM2E5NTQ4NDE4MDY5ZDlkNTJlYzFjZTQ3ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF80NDcwODc3OTMxODM0ZjIxOGU4MjYxNjg3ZDU2OTM0OCA9ICQoYDxkaXYgaWQ9Imh0bWxfNDQ3MDg3NzkzMTgzNGYyMThlODI2MTY4N2Q1NjkzNDgiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk1hcGxlIEhhbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMzRlZDVkZmQzYTk1NDg0MTgwNjlkOWQ1MmVjMWNlNDcuc2V0Q29udGVudChodG1sXzQ0NzA4Nzc5MzE4MzRmMjE4ZTgyNjE2ODdkNTY5MzQ4KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2U1ZmM5MjYwMzZiMTRjOGNiMzYzZWRlYWMxZTg1OWIwLmJpbmRQb3B1cChwb3B1cF8zNGVkNWRmZDNhOTU0ODQxODA2OWQ5ZDUyZWMxY2U0NykKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9mMjljNWQxZTg3MTA0ZTNiYTg2YmFhM2UwM2YwZmVkNyA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljk0MDUwNjQ1ODI4MjUsIC04My45Nzk2MjI5NDUxODk1XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9jMzI4NjA0YjcxNjQ0Yzk2ODUzYWRlMWY1ZTRjYTZjNCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfZmU1YTIyNmYxZDdhNDYwYjkxZDY4MGEwZGUyYjJkMzkgPSAkKGA8ZGl2IGlkPSJodG1sX2ZlNWEyMjZmMWQ3YTQ2MGI5MWQ2ODBhMGRlMmIyZDM5IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CZWFyZGVuIEJlZXIgTWFya2V0PC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2MzMjg2MDRiNzE2NDRjOTY4NTNhZGUxZjVlNGNhNmM0LnNldENvbnRlbnQoaHRtbF9mZTVhMjI2ZjFkN2E0NjBiOTFkNjgwYTBkZTJiMmQzOSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9mMjljNWQxZTg3MTA0ZTNiYTg2YmFhM2UwM2YwZmVkNy5iaW5kUG9wdXAocG9wdXBfYzMyODYwNGI3MTY0NGM5Njg1M2FkZTFmNWU0Y2E2YzQpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfOTBmNGVmZDBiMzMwNDBlYjlkNGRlZGIzMTJiMjFhNGEgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45MzQ4Njk5LCAtODQuMDAzNzM5OV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMzVhNzcxOWZhNmQyNDJiZDhmNDY1ZGI5NDAyMmZhMGQgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzFjODA0MGZlOTczNzQ0Yjg4MmVlMDg0MDQyZjcwYzQ3ID0gJChgPGRpdiBpZD0iaHRtbF8xYzgwNDBmZTk3Mzc0NGI4ODJlZTA4NDA0MmY3MGM0NyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QmlzdHJvIEJ5IFRoZSBUcmFja3M8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMzVhNzcxOWZhNmQyNDJiZDhmNDY1ZGI5NDAyMmZhMGQuc2V0Q29udGVudChodG1sXzFjODA0MGZlOTczNzQ0Yjg4MmVlMDg0MDQyZjcwYzQ3KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzkwZjRlZmQwYjMzMDQwZWI5ZDRkZWRiMzEyYjIxYTRhLmJpbmRQb3B1cChwb3B1cF8zNWE3NzE5ZmE2ZDI0MmJkOGY0NjVkYjk0MDIyZmEwZCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9jYjA2OTJhZWVkOWE0NzcwYWUxODUxZWJhNTA5YmI2MSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljg5NjY3ODA3NjEyMjgsIC04NC4xNzA5MzAyNDY2NjcyXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF81Njg1MzI4NTZlN2U0MmM1YTI5ZTBlODkzZjNmM2IyNSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMTY3MzI5ODUwMGQ3NGNmNmIyYjI1ZGViNTU3MzkwNTYgPSAkKGA8ZGl2IGlkPSJodG1sXzE2NzMyOTg1MDBkNzRjZjZiMmIyNWRlYjU1NzM5MDU2IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5TZWFzb25zIElubm92YXRpdmUgQmFyICZhbXA7IEdyaWxsZTwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF81Njg1MzI4NTZlN2U0MmM1YTI5ZTBlODkzZjNmM2IyNS5zZXRDb250ZW50KGh0bWxfMTY3MzI5ODUwMGQ3NGNmNmIyYjI1ZGViNTU3MzkwNTYpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfY2IwNjkyYWVlZDlhNDc3MGFlMTg1MWViYTUwOWJiNjEuYmluZFBvcHVwKHBvcHVwXzU2ODUzMjg1NmU3ZTQyYzVhMjllMGU4OTNmM2YzYjI1KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzVjMzk0YzU4M2Q4NzQ0OTM4YTIzMTJjYzhiY2Y1NDc2ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTM5NSwgLTgzLjk4ODc0Ml0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMGMyZWMxMGYwYTQ1NGU0Yjg1YTMyODVmNTZiMWM2ZjggPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzVkZjYxYmNiYmNkOTRmMDY5MGUyOGNlYjM0ZjExNzBkID0gJChgPGRpdiBpZD0iaHRtbF81ZGY2MWJjYmJjZDk0ZjA2OTBlMjhjZWIzNGYxMTcwZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+SG9sbHkmIzM5O3MgR291cm1ldCYjMzk7cyBNYXJrZXQgJmFtcDsgQ2FmZTwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8wYzJlYzEwZjBhNDU0ZTRiODVhMzI4NWY1NmIxYzZmOC5zZXRDb250ZW50KGh0bWxfNWRmNjFiY2JiY2Q5NGYwNjkwZTI4Y2ViMzRmMTE3MGQpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNWMzOTRjNTgzZDg3NDQ5MzhhMjMxMmNjOGJjZjU0NzYuYmluZFBvcHVwKHBvcHVwXzBjMmVjMTBmMGE0NTRlNGI4NWEzMjg1ZjU2YjFjNmY4KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzFiMWE4MjMyOWE4YzRkM2U4OTgxOTgwOWVhZDNkMmQyID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTA0MjksIC04NC4xNDI5NV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNTIwZjNlZjYwZjEyNDViZTg4NjRiY2FjNmEzZjc1MTMgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzA0NGUxNGMxODMwNDQ4N2JhZDdmODA0NDUwMjAzN2JiID0gJChgPGRpdiBpZD0iaHRtbF8wNDRlMTRjMTgzMDQ0ODdiYWQ3ZjgwNDQ1MDIwMzdiYiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGFjbyBCZWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzUyMGYzZWY2MGYxMjQ1YmU4ODY0YmNhYzZhM2Y3NTEzLnNldENvbnRlbnQoaHRtbF8wNDRlMTRjMTgzMDQ0ODdiYWQ3ZjgwNDQ1MDIwMzdiYik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl8xYjFhODIzMjlhOGM0ZDNlODk4MTk4MDllYWQzZDJkMi5iaW5kUG9wdXAocG9wdXBfNTIwZjNlZjYwZjEyNDViZTg4NjRiY2FjNmEzZjc1MTMpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNWQ1YTJjYmRmZmY1NGZlMTg3ZTU1MjUyYmI0MGIxNTEgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS43NTgxNywgLTgzLjk2NTYxXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8xNDJhNjk2Mzk1OGM0NDM3ODJhZmM3NmYxMjcxZTkxNiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNjhjMGI1YjkxZGE1NDIxYmJiZDkzYTExNWIwMGM2ODcgPSAkKGA8ZGl2IGlkPSJodG1sXzY4YzBiNWI5MWRhNTQyMWJiYmQ5M2ExMTViMDBjNjg3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Gb290aGlsbHMgTWlsbGluZzwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8xNDJhNjk2Mzk1OGM0NDM3ODJhZmM3NmYxMjcxZTkxNi5zZXRDb250ZW50KGh0bWxfNjhjMGI1YjkxZGE1NDIxYmJiZDkzYTExNWIwMGM2ODcpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNWQ1YTJjYmRmZmY1NGZlMTg3ZTU1MjUyYmI0MGIxNTEuYmluZFBvcHVwKHBvcHVwXzE0MmE2OTYzOTU4YzQ0Mzc4MmFmYzc2ZjEyNzFlOTE2KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2EzZDU2OGQxZjIzOTQ4MjY4YTY4M2NiMzdiNTdlMWNiID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMDAzNzE1LCAtODQuMDIwNjgxXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9jYWEwMDhhNDg3YTc0NmFlOWIxMmIyNDU1YTNjNGU0NCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfOGJlMDY3OWM3NDExNDgxNmFiOGRjN2YxNjBiZWI2ZDEgPSAkKGA8ZGl2IGlkPSJodG1sXzhiZTA2NzljNzQxMTQ4MTZhYjhkYzdmMTYwYmViNmQxIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5NY0FsaXN0ZXImIzM5O3MgRGVsaTwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9jYWEwMDhhNDg3YTc0NmFlOWIxMmIyNDU1YTNjNGU0NC5zZXRDb250ZW50KGh0bWxfOGJlMDY3OWM3NDExNDgxNmFiOGRjN2YxNjBiZWI2ZDEpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfYTNkNTY4ZDFmMjM5NDgyNjhhNjgzY2IzN2I1N2UxY2IuYmluZFBvcHVwKHBvcHVwX2NhYTAwOGE0ODdhNzQ2YWU5YjEyYjI0NTVhM2M0ZTQ0KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2YyYzY2OWU0NTg2MjRjMjhiNjNhZjlhOTA1ZmYxZjBiID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMDI2MjQyOTY5OTMwOCwgLTgzLjkyNzgzNzY1NTUyMjVdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzUxZGUzNzZhNGZmYjRjZjU4YWM1YzMzZTI3ODNiNmE3ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF84MjcyNTY5MzE3ZDI0NmI1YTgwMGNlZmNkNzk0ZjE1MSA9ICQoYDxkaXYgaWQ9Imh0bWxfODI3MjU2OTMxN2QyNDZiNWE4MDBjZWZjZDc5NGYxNTEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNoaXBvdGxlIE1leGljYW4gR3JpbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNTFkZTM3NmE0ZmZiNGNmNThhYzVjMzNlMjc4M2I2YTcuc2V0Q29udGVudChodG1sXzgyNzI1NjkzMTdkMjQ2YjVhODAwY2VmY2Q3OTRmMTUxKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2YyYzY2OWU0NTg2MjRjMjhiNjNhZjlhOTA1ZmYxZjBiLmJpbmRQb3B1cChwb3B1cF81MWRlMzc2YTRmZmI0Y2Y1OGFjNWMzM2UyNzgzYjZhNykKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl83Njk1ZDU4OWZmYTg0YjJhYTZkYmE0NjQ2ZTk4MTk3NSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljk3MDU5MDM5MzQzMjgsIC04My45MTg0MDQ4NzcxODU4XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF82NWMyMDZmMTk1ZTY0NzRmODk0ZTM0MWQ1NTVhN2ExZCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfZDU1ZThjYTEzYWMwNDgzN2JhMDU4ZjM1MzNiNDViOGUgPSAkKGA8ZGl2IGlkPSJodG1sX2Q1NWU4Y2ExM2FjMDQ4MzdiYTA1OGYzNTMzYjQ1YjhlIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Mb25lc29tZSBEb3ZlIFdlc3Rlcm4gQmlzdHJvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzY1YzIwNmYxOTVlNjQ3NGY4OTRlMzQxZDU1NWE3YTFkLnNldENvbnRlbnQoaHRtbF9kNTVlOGNhMTNhYzA0ODM3YmEwNThmMzUzM2I0NWI4ZSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl83Njk1ZDU4OWZmYTg0YjJhYTZkYmE0NjQ2ZTk4MTk3NS5iaW5kUG9wdXAocG9wdXBfNjVjMjA2ZjE5NWU2NDc0Zjg5NGUzNDFkNTU1YTdhMWQpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfYTMxZDM3NDViYzVkNGEwNThiNzhlMzNmMjY1NmQzYzQgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45NjM2MjEzLCAtODMuOTE4OTcyNl0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMDIwNTBmMTgyYjVlNGUyOWE4OGE5ZWQ5MTY2MDQ1NjQgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzc0MDliZmI5ZTJhZTQzMmI4MzVhNzVlYjc4ZDJhMzU3ID0gJChgPGRpdiBpZD0iaHRtbF83NDA5YmZiOWUyYWU0MzJiODM1YTc1ZWI3OGQyYTM1NyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGhlIEZyZW5jaCBNYXJrZXQgQ3JlcGVyaWU8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMDIwNTBmMTgyYjVlNGUyOWE4OGE5ZWQ5MTY2MDQ1NjQuc2V0Q29udGVudChodG1sXzc0MDliZmI5ZTJhZTQzMmI4MzVhNzVlYjc4ZDJhMzU3KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2EzMWQzNzQ1YmM1ZDRhMDU4Yjc4ZTMzZjI2NTZkM2M0LmJpbmRQb3B1cChwb3B1cF8wMjA1MGYxODJiNWU0ZTI5YTg4YTllZDkxNjYwNDU2NCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9lNGM5MzE1NTg4YzI0ODQ3YTFlYmUzYzg5YTkwOTM4ZiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1LjkyNTc5NSwgLTg0LjAzMzE5NV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMGJhMjg2YjdjZjBiNDI2YjlkNWE4ODIwOTUzZmNlZmUgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzFmZWY0YzRmZTRmZjQ2MjI4NGVhMzU0YzdmZjI4NzUwID0gJChgPGRpdiBpZD0iaHRtbF8xZmVmNGM0ZmU0ZmY0NjIyODRlYTM1NGM3ZmYyODc1MCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TWNBbGlzdGVyJiMzOTtzIERlbGk8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMGJhMjg2YjdjZjBiNDI2YjlkNWE4ODIwOTUzZmNlZmUuc2V0Q29udGVudChodG1sXzFmZWY0YzRmZTRmZjQ2MjI4NGVhMzU0YzdmZjI4NzUwKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2U0YzkzMTU1ODhjMjQ4NDdhMWViZTNjODlhOTA5MzhmLmJpbmRQb3B1cChwb3B1cF8wYmEyODZiN2NmMGI0MjZiOWQ1YTg4MjA5NTNmY2VmZSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl8xNjBiNDc5N2VmYzQ0YjBmOTk0ZDdhMzIzNDExYTY1YSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjA1MDQxNDg5NDYxODYsIC04My45OTI5NzU2NDI4MzcyXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8zYTIwMWVhZGVhMDQ0NzM3ODY3OWE5YTg0ZTVkZWZkZSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYjhhZjNkNmQzZGZmNDE0MTgzYzYwZjkxZjlkMDk0NDkgPSAkKGA8ZGl2IGlkPSJodG1sX2I4YWYzZDZkM2RmZjQxNDE4M2M2MGY5MWY5ZDA5NDQ5IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5TYWxzYXJpdGEmIzM5O3MgRnJlc2ggQ2FudGluYTwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8zYTIwMWVhZGVhMDQ0NzM3ODY3OWE5YTg0ZTVkZWZkZS5zZXRDb250ZW50KGh0bWxfYjhhZjNkNmQzZGZmNDE0MTgzYzYwZjkxZjlkMDk0NDkpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfMTYwYjQ3OTdlZmM0NGIwZjk5NGQ3YTMyMzQxMWE2NWEuYmluZFBvcHVwKHBvcHVwXzNhMjAxZWFkZWEwNDQ3Mzc4Njc5YTlhODRlNWRlZmRlKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2JkMTdjYmZhMzdjOTQ0YTE5YTQ5ZjUyZmFlMTM0YzliID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuODk5MzcxLCAtODQuMTYwNjcxXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8xMmY3YTc1NWQyMGE0NDRmODUyOTNmY2VmNjE5YTM1OSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMDcxYmU1Yjc4ZjlhNDUyNDhmODA2ZjE5ZjUyYTQ4M2UgPSAkKGA8ZGl2IGlkPSJodG1sXzA3MWJlNWI3OGY5YTQ1MjQ4ZjgwNmYxOWY1MmE0ODNlIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DaGlwb3RsZSBNZXhpY2FuIEdyaWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzEyZjdhNzU1ZDIwYTQ0NGY4NTI5M2ZjZWY2MTlhMzU5LnNldENvbnRlbnQoaHRtbF8wNzFiZTViNzhmOWE0NTI0OGY4MDZmMTlmNTJhNDgzZSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9iZDE3Y2JmYTM3Yzk0NGExOWE0OWY1MmZhZTEzNGM5Yi5iaW5kUG9wdXAocG9wdXBfMTJmN2E3NTVkMjBhNDQ0Zjg1MjkzZmNlZjYxOWEzNTkpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfZWRjYWM2ZjhhZGQ0NDcxM2IzYWNjMjg0YWQ4NTQwMDQgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45NTkwMzAxNzk0NDE2LCAtODMuOTA1MTM3MjczMzI4NF0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfOTM0NWQ0NWVmNWNiNGVhM2FhMjc4ZTBjYjY4YTMyM2EgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzE3ZGQ5Y2UxZmIzMzRmZTc5NDQ5YWM2NGQyYzhiMTAyID0gJChgPGRpdiBpZD0iaHRtbF8xN2RkOWNlMWZiMzM0ZmU3OTQ0OWFjNjRkMmM4YjEwMiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U291dGhzaWRlIEdhcmFnZSBGb29kIFRydWNrIFBhcms8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfOTM0NWQ0NWVmNWNiNGVhM2FhMjc4ZTBjYjY4YTMyM2Euc2V0Q29udGVudChodG1sXzE3ZGQ5Y2UxZmIzMzRmZTc5NDQ5YWM2NGQyYzhiMTAyKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2VkY2FjNmY4YWRkNDQ3MTNiM2FjYzI4NGFkODU0MDA0LmJpbmRQb3B1cChwb3B1cF85MzQ1ZDQ1ZWY1Y2I0ZWEzYWEyNzhlMGNiNjhhMzIzYSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl8zNjRmNWJhZjVkYmU0OGZiYWM1NmIwYjhjMDRkMDI4YSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljg5ODMzLCAtODQuMTY1NDFdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2UzMGM3NzEyN2I4MDRlMjk5YWU5MGY3MzQyNTAwMzFmID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8yMzljNTc4ODA1NTQ0NGQ3ODI5MjA1YjUyMTQzODYxZSA9ICQoYDxkaXYgaWQ9Imh0bWxfMjM5YzU3ODgwNTU0NDRkNzgyOTIwNWI1MjE0Mzg2MWUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNoaWxpJiMzOTtzPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2UzMGM3NzEyN2I4MDRlMjk5YWU5MGY3MzQyNTAwMzFmLnNldENvbnRlbnQoaHRtbF8yMzljNTc4ODA1NTQ0NGQ3ODI5MjA1YjUyMTQzODYxZSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl8zNjRmNWJhZjVkYmU0OGZiYWM1NmIwYjhjMDRkMDI4YS5iaW5kUG9wdXAocG9wdXBfZTMwYzc3MTI3YjgwNGUyOTlhZTkwZjczNDI1MDAzMWYpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfZTEyZTIyNDc3ZTNmNGUxYTgzYzY0ZDRiOWM1NjZlNWEgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45MTU4MTgsIC04NC4wODk0NDQ2NDU5ODA2XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8wODI3MDE2NGY3MmU0NzBjYjBiMmIyM2RjMzExNTFhZSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfOTYyMGZlMmZhZmUyNDM3OGIxYjk0YzE3NTM0MDRmNDMgPSAkKGA8ZGl2IGlkPSJodG1sXzk2MjBmZTJmYWZlMjQzNzhiMWI5NGMxNzUzNDA0ZjQzIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DaGlwb3RsZSBNZXhpY2FuIEdyaWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzA4MjcwMTY0ZjcyZTQ3MGNiMGIyYjIzZGMzMTE1MWFlLnNldENvbnRlbnQoaHRtbF85NjIwZmUyZmFmZTI0Mzc4YjFiOTRjMTc1MzQwNGY0Myk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9lMTJlMjI0NzdlM2Y0ZTFhODNjNjRkNGI5YzU2NmU1YS5iaW5kUG9wdXAocG9wdXBfMDgyNzAxNjRmNzJlNDcwY2IwYjJiMjNkYzMxMTUxYWUpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfZGUwNTRlNjllOTQ1NGUyN2JkMWYzODBlOThlM2IxYjggPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45NjUyOCwgLTgzLjkxOTE0XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF85YWRkZWU2NjkyZGY0ZTA1ODYwZmVjN2JkMmNlNDJjNSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMjYwMWVlMmQxNGQ0NGE3MWI0YzBjMDA4NDNlMDMwZjEgPSAkKGA8ZGl2IGlkPSJodG1sXzI2MDFlZTJkMTRkNDRhNzFiNGMwYzAwODQzZTAzMGYxIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UaGUgVG9tYXRvIEhlYWQ8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfOWFkZGVlNjY5MmRmNGUwNTg2MGZlYzdiZDJjZTQyYzUuc2V0Q29udGVudChodG1sXzI2MDFlZTJkMTRkNDRhNzFiNGMwYzAwODQzZTAzMGYxKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2RlMDU0ZTY5ZTk0NTRlMjdiZDFmMzgwZTk4ZTNiMWI4LmJpbmRQb3B1cChwb3B1cF85YWRkZWU2NjkyZGY0ZTA1ODYwZmVjN2JkMmNlNDJjNSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl85M2IyYzEyOWNiZDY0MzEyYTk5NzE0MjhhZDU5NmM4MCA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1LjkwMDQzOCwgLTg0LjE1OTE1XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9jYzkwM2YzYTAzOTg0NzllOTJhNzA0MWUyMTkwMzNkMSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNzZmM2E1NGE2M2M2NGJmNDk4MjZiODk0YjExYTdjMDAgPSAkKGA8ZGl2IGlkPSJodG1sXzc2ZjNhNTRhNjNjNjRiZjQ5ODI2Yjg5NGIxMWE3YzAwIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Nb2UmIzM5O3MgU291dGh3ZXN0IEdyaWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2NjOTAzZjNhMDM5ODQ3OWU5MmE3MDQxZTIxOTAzM2QxLnNldENvbnRlbnQoaHRtbF83NmYzYTU0YTYzYzY0YmY0OTgyNmI4OTRiMTFhN2MwMCk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl85M2IyYzEyOWNiZDY0MzEyYTk5NzE0MjhhZDU5NmM4MC5iaW5kUG9wdXAocG9wdXBfY2M5MDNmM2EwMzk4NDc5ZTkyYTcwNDFlMjE5MDMzZDEpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfYzE3MWRiNDFjMzlmNGQ4OGIzYmM2OTc3ZDRhM2M2ZDAgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS44MTMzNTE3ODg1NTg1LCAtODQuMjY0NDQ5MDA4ODYyOF0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNDViZGQ1NGEzODJhNDhkZWEwZmZlZjc1MmU1NDQ4MTkgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2RjOWVkMDlkNDQ4NzQwNDA5MzlkZjBhZjcxZjlkY2I0ID0gJChgPGRpdiBpZD0iaHRtbF9kYzllZDA5ZDQ0ODc0MDQwOTM5ZGYwYWY3MWY5ZGNiNCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UGV0cm8mIzM5O3MgQ2hpbGkgJmFtcDsgQ2hpcHMgLSBMZW5vaXIgQ2l0eTwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF80NWJkZDU0YTM4MmE0OGRlYTBmZmVmNzUyZTU0NDgxOS5zZXRDb250ZW50KGh0bWxfZGM5ZWQwOWQ0NDg3NDA0MDkzOWRmMGFmNzFmOWRjYjQpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfYzE3MWRiNDFjMzlmNGQ4OGIzYmM2OTc3ZDRhM2M2ZDAuYmluZFBvcHVwKHBvcHVwXzQ1YmRkNTRhMzgyYTQ4ZGVhMGZmZWY3NTJlNTQ0ODE5KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzNhNzNlOWNiZWM2NTQ4ZWFhZmFhYmJmMWZkMjgwYjVjID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTE4MzA0NDQzMzU5NCwgLTg0LjAwMzYwMTA3NDIxODhdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzRmNDBmN2UyMDA4MjRlZWE5NDRiZjEwMmEzYjY5N2UyID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9iNGE5NjBiMjkzMjU0MzcyYWE4YzQyNGY0NDMwYjYzMSA9ICQoYDxkaXYgaWQ9Imh0bWxfYjRhOTYwYjI5MzI1NDM3MmFhOGM0MjRmNDQzMGI2MzEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRvb3RzaWUgVHJ1Y2s8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNGY0MGY3ZTIwMDgyNGVlYTk0NGJmMTAyYTNiNjk3ZTIuc2V0Q29udGVudChodG1sX2I0YTk2MGIyOTMyNTQzNzJhYThjNDI0ZjQ0MzBiNjMxKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzNhNzNlOWNiZWM2NTQ4ZWFhZmFhYmJmMWZkMjgwYjVjLmJpbmRQb3B1cChwb3B1cF80ZjQwZjdlMjAwODI0ZWVhOTQ0YmYxMDJhM2I2OTdlMikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl8zNDBiNzM4ZjgwODI0MjVjODkwYTQ2NWZlMDA4ZjQyMiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljk1Mjk2NDkxODMyNzMsIC04My45MzIyNTgwNjg1NTg5XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8xZTc0ZTFlMDgyMTQ0OWJjOTYwYTQwNDYzNGQyMjJlNiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfOGNhYjllNGJkMzZmNGU1MGI2YmE5ZGMxYmJlNmFhNGYgPSAkKGA8ZGl2IGlkPSJodG1sXzhjYWI5ZTRiZDM2ZjRlNTBiNmJhOWRjMWJiZTZhYTRmIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Ud2lzdGVkIFRhY288L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMWU3NGUxZTA4MjE0NDliYzk2MGE0MDQ2MzRkMjIyZTYuc2V0Q29udGVudChodG1sXzhjYWI5ZTRiZDM2ZjRlNTBiNmJhOWRjMWJiZTZhYTRmKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzM0MGI3MzhmODA4MjQyNWM4OTBhNDY1ZmUwMDhmNDIyLmJpbmRQb3B1cChwb3B1cF8xZTc0ZTFlMDgyMTQ0OWJjOTYwYTQwNDYzNGQyMjJlNikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9jZWJhZGNhYzM2MjI0NmQ3ODllYmE0NjFjZTQ0Y2ExYSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljc0NTY5MTU2MTk2MDMsIC04My45OTYwMDcyMTg5NTY5XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF82OTA1MzEwZTA4MWQ0MGU1OTY1MzdiOGNjZjFmZjE5YSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNjY5YjcyZDA1MDc1NGM4MjhlM2E0ZTA5YTBhOTA1ZWIgPSAkKGA8ZGl2IGlkPSJodG1sXzY2OWI3MmQwNTA3NTRjODI4ZTNhNGUwOWEwYTkwNWViIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DYW5jdW4gTWV4aWNhbiBSZXN0YXVyYW50PC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzY5MDUzMTBlMDgxZDQwZTU5NjUzN2I4Y2NmMWZmMTlhLnNldENvbnRlbnQoaHRtbF82NjliNzJkMDUwNzU0YzgyOGUzYTRlMDlhMGE5MDVlYik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9jZWJhZGNhYzM2MjI0NmQ3ODllYmE0NjFjZTQ0Y2ExYS5iaW5kUG9wdXAocG9wdXBfNjkwNTMxMGUwODFkNDBlNTk2NTM3YjhjY2YxZmYxOWEpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNmE5NGE3ZDA1ZGMyNDVlYmEyOTMxOGVjYjQxZTM5Y2YgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45MzYxNzYzMDQ0OTYxLCAtODQuMDA0NjUzMDQyODA5N10sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfZjE3OWVhY2U0MmZmNGNkYzllMjlkNDRlNzU2ZDY3YTIgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzJjYzgyZThjMjAzNTRmNGZiMmY3MWIyYjZiOGI3MzNiID0gJChgPGRpdiBpZD0iaHRtbF8yY2M4MmU4YzIwMzU0ZjRmYjJmNzFiMmI2YjhiNzMzYiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGhlIENhc3VhbCBQaW50IC0gQmVhcmRlbjwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9mMTc5ZWFjZTQyZmY0Y2RjOWUyOWQ0NGU3NTZkNjdhMi5zZXRDb250ZW50KGh0bWxfMmNjODJlOGMyMDM1NGY0ZmIyZjcxYjJiNmI4YjczM2IpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNmE5NGE3ZDA1ZGMyNDVlYmEyOTMxOGVjYjQxZTM5Y2YuYmluZFBvcHVwKHBvcHVwX2YxNzllYWNlNDJmZjRjZGM5ZTI5ZDQ0ZTc1NmQ2N2EyKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzZiMWRlNjkzNjdjYjQ1MWM4NDk5ZDVlY2M4YzBlOWI3ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTcyNTQzLCAtODMuOTg2MTk4XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8zYTlhMGIyMGM0ZGQ0ZmZlYTIxZGQyNzY0ODA1ODI2MiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfM2VkNDg3ZjA5MjQxNDlhZmJkNTAwMjA1ZTRjOTZhZWUgPSAkKGA8ZGl2IGlkPSJodG1sXzNlZDQ4N2YwOTI0MTQ5YWZiZDUwMDIwNWU0Yzk2YWVlIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UYWNvIEJlbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfM2E5YTBiMjBjNGRkNGZmZWEyMWRkMjc2NDgwNTgyNjIuc2V0Q29udGVudChodG1sXzNlZDQ4N2YwOTI0MTQ5YWZiZDUwMDIwNWU0Yzk2YWVlKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzZiMWRlNjkzNjdjYjQ1MWM4NDk5ZDVlY2M4YzBlOWI3LmJpbmRQb3B1cChwb3B1cF8zYTlhMGIyMGM0ZGQ0ZmZlYTIxZGQyNzY0ODA1ODI2MikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl8zYzEwMTkwNTdmNTk0MTcwOTYwYWI3YzhiYzdhMDk0NCA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljk3MDQzMjI4MTQ5NDEsIC04My45MTc3MDkzNTA1ODU5XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8wZDgxN2E0NmYyZDc0ZDAxYTc5YjlhMzhiYzM0NzEyZiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMWRkMDBiYTc4OGM4NDVlMGJlYjIwMTlmY2ZmMmQyNjEgPSAkKGA8ZGl2IGlkPSJodG1sXzFkZDAwYmE3ODhjODQ1ZTBiZWIyMDE5ZmNmZjJkMjYxIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Tb3V0aGJvdW5kIEJhciAmYW1wOyBHcmlsbDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8wZDgxN2E0NmYyZDc0ZDAxYTc5YjlhMzhiYzM0NzEyZi5zZXRDb250ZW50KGh0bWxfMWRkMDBiYTc4OGM4NDVlMGJlYjIwMTlmY2ZmMmQyNjEpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfM2MxMDE5MDU3ZjU5NDE3MDk2MGFiN2M4YmM3YTA5NDQuYmluZFBvcHVwKHBvcHVwXzBkODE3YTQ2ZjJkNzRkMDFhNzliOWEzOGJjMzQ3MTJmKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzA3NWNkNDVkOWJlYTRkYzlhOWExYjI5ZTcxOTdkYjdjID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTI5NzI5LCAtODQuMDMxOTU4XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF84MDYwNzE5ZTk0ZmM0YTU4ODgwYjU1NmU1ZDhhMTQ1ZSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfZjg0NzcwODNiZGJmNGYwODhmNjJiY2VmYjU1M2E5MDMgPSAkKGA8ZGl2IGlkPSJodG1sX2Y4NDc3MDgzYmRiZjRmMDg4ZjYyYmNlZmI1NTNhOTAzIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UYWNvIEJlbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfODA2MDcxOWU5NGZjNGE1ODg4MGI1NTZlNWQ4YTE0NWUuc2V0Q29udGVudChodG1sX2Y4NDc3MDgzYmRiZjRmMDg4ZjYyYmNlZmI1NTNhOTAzKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzA3NWNkNDVkOWJlYTRkYzlhOWExYjI5ZTcxOTdkYjdjLmJpbmRQb3B1cChwb3B1cF84MDYwNzE5ZTk0ZmM0YTU4ODgwYjU1NmU1ZDhhMTQ1ZSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9hMmIyZmUzNmFjN2I0NjE5YWIyYmYwZTkzOTUyNDZjNSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljk1NDgyNiwgLTgzLjkzNjA1OF0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNDM0YWRlNzA4Y2RjNDQ3NjgwZWQxZGFkMWU3ZGFmNmEgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzlhOTk3NTk1OTFiOTRlOWQ5ODBmOWNhYWEyZTNhNTgzID0gJChgPGRpdiBpZD0iaHRtbF85YTk5NzU5NTkxYjk0ZTlkOTgwZjljYWFhMmUzYTU4MyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGFjbyBCZWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzQzNGFkZTcwOGNkYzQ0NzY4MGVkMWRhZDFlN2RhZjZhLnNldENvbnRlbnQoaHRtbF85YTk5NzU5NTkxYjk0ZTlkOTgwZjljYWFhMmUzYTU4Myk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9hMmIyZmUzNmFjN2I0NjE5YWIyYmYwZTkzOTUyNDZjNS5iaW5kUG9wdXAocG9wdXBfNDM0YWRlNzA4Y2RjNDQ3NjgwZWQxZGFkMWU3ZGFmNmEpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfMWU5ODkxZjZkNGJiNGRmYWI5YzdmNmRkNTg3Y2M5MDEgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45OTE5NSwgLTgzLjkyMDg4XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF85MjIwM2YxZDZkNTI0MThiYTBhNDhkMDBhMWE1MmFiZiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNjI1NTkyZjQ5OWMwNGZlYjhhOTA0MjBjODI3NDM3MmMgPSAkKGA8ZGl2IGlkPSJodG1sXzYyNTU5MmY0OTljMDRmZWI4YTkwNDIwYzgyNzQzNzJjIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UYWNvIEJlbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfOTIyMDNmMWQ2ZDUyNDE4YmEwYTQ4ZDAwYTFhNTJhYmYuc2V0Q29udGVudChodG1sXzYyNTU5MmY0OTljMDRmZWI4YTkwNDIwYzgyNzQzNzJjKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzFlOTg5MWY2ZDRiYjRkZmFiOWM3ZjZkZDU4N2NjOTAxLmJpbmRQb3B1cChwb3B1cF85MjIwM2YxZDZkNTI0MThiYTBhNDhkMDBhMWE1MmFiZikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl82OTViMTNiYzM0NDM0ZWM1OWE1MWY5NGZlOTVmMWU1YiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljk1NjQ5NDkyMjgwNTQsIC04My45MzM3MzczODk3NDMzXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF85NTAxYTJiNzZiM2I0OTFhOTYyM2ZhNGU3YzZhNGRlZCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfZjBjNzc5ZTJlMmRiNGFiNDkyODhmNGZiMGYxZTgwMjQgPSAkKGA8ZGl2IGlkPSJodG1sX2YwYzc3OWUyZTJkYjRhYjQ5Mjg4ZjRmYjBmMWU4MDI0IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5BbGFkZGluIEdyaWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzk1MDFhMmI3NmIzYjQ5MWE5NjIzZmE0ZTdjNmE0ZGVkLnNldENvbnRlbnQoaHRtbF9mMGM3NzllMmUyZGI0YWI0OTI4OGY0ZmIwZjFlODAyNCk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl82OTViMTNiYzM0NDM0ZWM1OWE1MWY5NGZlOTVmMWU1Yi5iaW5kUG9wdXAocG9wdXBfOTUwMWEyYjc2YjNiNDkxYTk2MjNmYTRlN2M2YTRkZWQpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNTNjZDE0NmJmYWU2NDg2NGI1MDE1OTliMDAyMWJmZTIgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS43NjYxODkwNjEyOTU4LCAtODMuOTg2MDgxMzY5MjIxMl0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMWM0Njc0Nzc5MzgwNGZhNGIyNDJlM2JjMjI3MzQxYWYgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzRmMmM0MWIwMTU3YTRlNDg4YTE2ODA4MzgwZjgzZDFiID0gJChgPGRpdiBpZD0iaHRtbF80ZjJjNDFiMDE1N2E0ZTQ4OGExNjgwODM4MGY4M2QxYiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TW9lJiMzOTtzIFNvdXRod2VzdCBHcmlsbDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8xYzQ2NzQ3NzkzODA0ZmE0YjI0MmUzYmMyMjczNDFhZi5zZXRDb250ZW50KGh0bWxfNGYyYzQxYjAxNTdhNGU0ODhhMTY4MDgzODBmODNkMWIpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNTNjZDE0NmJmYWU2NDg2NGI1MDE1OTliMDAyMWJmZTIuYmluZFBvcHVwKHBvcHVwXzFjNDY3NDc3OTM4MDRmYTRiMjQyZTNiYzIyNzM0MWFmKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzExNDk3MGQ1NWM2ODQyYTI5YjNmOWFkZDE3ZDg2MGI1ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMDA4NDUsIC04My45NzU4OF0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfZDkyNDc3MDdkYTFmNDkzZmI1MmY4ODAzZTA3YTZjMjkgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzdmNzk3NzVkYTg1NzQwZGY5ZTkyOGRjZWM2NTIxODM0ID0gJChgPGRpdiBpZD0iaHRtbF83Zjc5Nzc1ZGE4NTc0MGRmOWU5MjhkY2VjNjUyMTgzNCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGFjbyBCZWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2Q5MjQ3NzA3ZGExZjQ5M2ZiNTJmODgwM2UwN2E2YzI5LnNldENvbnRlbnQoaHRtbF83Zjc5Nzc1ZGE4NTc0MGRmOWU5MjhkY2VjNjUyMTgzNCk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl8xMTQ5NzBkNTVjNjg0MmEyOWIzZjlhZGQxN2Q4NjBiNS5iaW5kUG9wdXAocG9wdXBfZDkyNDc3MDdkYTFmNDkzZmI1MmY4ODAzZTA3YTZjMjkpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfOTQ0MGUwNzEzOTdmNGMxMGFjYWM4MjllN2M4MjlhYTQgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS44OTI3OTA1NDE1MTIzLCAtODQuMTcxMDA2OTQwMzA1Ml0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfYzdlN2M1MWFiNDM0NDJkY2E4MmZjNzNlOTFjNGIyNzAgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzNjNzNiYWU3ODllMjRhZGY4MDdkZjJhMWE5NjE1OGYzID0gJChgPGRpdiBpZD0iaHRtbF8zYzczYmFlNzg5ZTI0YWRmODA3ZGYyYTFhOTYxNThmMyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Rmlyc3QgV2F0Y2g8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfYzdlN2M1MWFiNDM0NDJkY2E4MmZjNzNlOTFjNGIyNzAuc2V0Q29udGVudChodG1sXzNjNzNiYWU3ODllMjRhZGY4MDdkZjJhMWE5NjE1OGYzKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzk0NDBlMDcxMzk3ZjRjMTBhY2FjODI5ZTdjODI5YWE0LmJpbmRQb3B1cChwb3B1cF9jN2U3YzUxYWI0MzQ0MmRjYTgyZmM3M2U5MWM0YjI3MCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9mODBlZTI4YjUxNjc0ZmMwYWJiODI1MWFiY2QwMjJhYSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjAyNzQ3MjM5MTcyNDYsIC04My45MjY3MTAzMzc0MDA0XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8yNTEzYmM2ZjhjOGQ0MDZmOWRiNjQwMWM3Y2Y0MjkwOCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYWZkOWY1NmY1MDljNDZjZDk2YWZiYTgwNjJhNTQzMWEgPSAkKGA8ZGl2IGlkPSJodG1sX2FmZDlmNTZmNTA5YzQ2Y2Q5NmFmYmE4MDYyYTU0MzFhIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UaGUgQ2hvcCBIb3VzZTwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8yNTEzYmM2ZjhjOGQ0MDZmOWRiNjQwMWM3Y2Y0MjkwOC5zZXRDb250ZW50KGh0bWxfYWZkOWY1NmY1MDljNDZjZDk2YWZiYTgwNjJhNTQzMWEpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfZjgwZWUyOGI1MTY3NGZjMGFiYjgyNTFhYmNkMDIyYWEuYmluZFBvcHVwKHBvcHVwXzI1MTNiYzZmOGM4ZDQwNmY5ZGI2NDAxYzdjZjQyOTA4KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzJiYTY5OWFjNWFlZjRiZmJiMDc1MDI0NzUyYzA5MWVkID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMDMzNDY1LCAtODMuODY5NDU5XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8xZDllMTExOGViMWI0YjAzODFlNDllNDVlMjQwZDMwMyA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYjdhZWJlODc1ZmM3NGI4OWI4YzA0ZDQ0NmI0ZGNkMDIgPSAkKGA8ZGl2IGlkPSJodG1sX2I3YWViZTg3NWZjNzRiODliOGMwNGQ0NDZiNGRjZDAyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UYWNvIEJlbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMWQ5ZTExMThlYjFiNGIwMzgxZTQ5ZTQ1ZTI0MGQzMDMuc2V0Q29udGVudChodG1sX2I3YWViZTg3NWZjNzRiODliOGMwNGQ0NDZiNGRjZDAyKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzJiYTY5OWFjNWFlZjRiZmJiMDc1MDI0NzUyYzA5MWVkLmJpbmRQb3B1cChwb3B1cF8xZDllMTExOGViMWI0YjAzODFlNDllNDVlMjQwZDMwMykKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl8xNmNlZTFmYTY2NmY0YjJkODU4OWJhMjgyNjI2NjIyYSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljk1NTcwMjQ1MjU2NzMsIC04My45MzIwMDkwNjg0MTMzXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8zYWI0ODFkZmY2MTU0YWViOTkxYWFhMjg1OTdjODdmOSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMjZkOWJiNDJjYjEzNDcxNGJlMGNmM2Y0ODhkOTM2MjMgPSAkKGA8ZGl2IGlkPSJodG1sXzI2ZDliYjQyY2IxMzQ3MTRiZTBjZjNmNDg4ZDkzNjIzIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UaGUgR29sZGVuIFJvYXN0IENvZmZlZSBSb2FzdGVyczwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8zYWI0ODFkZmY2MTU0YWViOTkxYWFhMjg1OTdjODdmOS5zZXRDb250ZW50KGh0bWxfMjZkOWJiNDJjYjEzNDcxNGJlMGNmM2Y0ODhkOTM2MjMpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfMTZjZWUxZmE2NjZmNGIyZDg1ODliYTI4MjYyNjYyMmEuYmluZFBvcHVwKHBvcHVwXzNhYjQ4MWRmZjYxNTRhZWI5OTFhYWEyODU5N2M4N2Y5KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzEyMDJjZWVkMmRhOTQwYmM5MTliMjYyMGMyNGNkY2E1ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuNzcwNDgsIC04My45ODc0Ml0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNDM2YzJiNGUxZDBlNDZhMWJjOTNiZjRjZTIyZmU2MGQgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzA5MTliOTNlMDZjOTQ3OWRiMWIxMDE1NjZkMDcxOGMxID0gJChgPGRpdiBpZD0iaHRtbF8wOTE5YjkzZTA2Yzk0NzlkYjFiMTAxNTY2ZDA3MThjMSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2hpbGkmIzM5O3M8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNDM2YzJiNGUxZDBlNDZhMWJjOTNiZjRjZTIyZmU2MGQuc2V0Q29udGVudChodG1sXzA5MTliOTNlMDZjOTQ3OWRiMWIxMDE1NjZkMDcxOGMxKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzEyMDJjZWVkMmRhOTQwYmM5MTliMjYyMGMyNGNkY2E1LmJpbmRQb3B1cChwb3B1cF80MzZjMmI0ZTFkMGU0NmExYmM5M2JmNGNlMjJmZTYwZCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9kMWYyZGM5MGZhYjI0OTZlODRhYjRlNWVjOWIxNTRkZCA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjAyNjY0NiwgLTgzLjkyNzUxM10sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfYmI3YmRiYTM4MDdlNDRlYWI4MDExY2Q2MTBjODM4YmIgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzIzZTc1NjkzYjQ2ZjRlOWJiOTUyZWY2MWU1MWU0ODFiID0gJChgPGRpdiBpZD0iaHRtbF8yM2U3NTY5M2I0NmY0ZTliYjk1MmVmNjFlNTFlNDgxYiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGFjbyBCZWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2JiN2JkYmEzODA3ZTQ0ZWFiODAxMWNkNjEwYzgzOGJiLnNldENvbnRlbnQoaHRtbF8yM2U3NTY5M2I0NmY0ZTliYjk1MmVmNjFlNTFlNDgxYik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9kMWYyZGM5MGZhYjI0OTZlODRhYjRlNWVjOWIxNTRkZC5iaW5kUG9wdXAocG9wdXBfYmI3YmRiYTM4MDdlNDRlYWI4MDExY2Q2MTBjODM4YmIpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfZjc5ZjQ1OWJlZGMwNGM2NjhhZmM4ZjVjNmMwMGVmNzkgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45MjQyMjMsIC04NC4wOTQxMjJdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzRkODhhYjYyZjRhYTRkODk5NWM3OTJjMGRiNGIzNjlhID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF82ZDFmMDFjYzg5ZDE0YjY5YTUyOTVmNTJmY2Y2ZTBlYyA9ICQoYDxkaXYgaWQ9Imh0bWxfNmQxZjAxY2M4OWQxNGI2OWE1Mjk1ZjUyZmNmNmUwZWMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRhY28gQmVsbDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF80ZDg4YWI2MmY0YWE0ZDg5OTVjNzkyYzBkYjRiMzY5YS5zZXRDb250ZW50KGh0bWxfNmQxZjAxY2M4OWQxNGI2OWE1Mjk1ZjUyZmNmNmUwZWMpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfZjc5ZjQ1OWJlZGMwNGM2NjhhZmM4ZjVjNmMwMGVmNzkuYmluZFBvcHVwKHBvcHVwXzRkODhhYjYyZjRhYTRkODk5NWM3OTJjMGRiNGIzNjlhKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2E4ZDJjZWZjZTcyMTQwN2Q4NmEyNzAxZTIzZmY1Nzg2ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTUzODExNjQ1NTA3OCwgLTgzLjkwMzc1NTE4Nzk4ODNdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzQyNTBmOTE0ZThkNDQwNTA5MzM5YzI4OTU2YjYzMTRhID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8xNjc2ODNkY2FlMmM0NmVjOGM0N2U4MDY0ZmE4NGYxMiA9ICQoYDxkaXYgaWQ9Imh0bWxfMTY3NjgzZGNhZTJjNDZlYzhjNDdlODA2NGZhODRmMTIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlNhdm9yeSAmYW1wOyBTd2VldCBUcnVjazwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF80MjUwZjkxNGU4ZDQ0MDUwOTMzOWMyODk1NmI2MzE0YS5zZXRDb250ZW50KGh0bWxfMTY3NjgzZGNhZTJjNDZlYzhjNDdlODA2NGZhODRmMTIpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfYThkMmNlZmNlNzIxNDA3ZDg2YTI3MDFlMjNmZjU3ODYuYmluZFBvcHVwKHBvcHVwXzQyNTBmOTE0ZThkNDQwNTA5MzM5YzI4OTU2YjYzMTRhKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzE2Y2FkNzk0OTMyYzRhYWI5MTE4ODA3ZGFhM2Q3MGMzID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuNzcxNTE4LCAtODMuOTg2NTgzOV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMTRhMzQ0Y2E3MTAyNDhkZjgxMTFlZTFlZmUzOWM3OGUgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzg0MGViMDJhMmJiZjRkYjE4M2JmZWMwNzdiYjdkYWE2ID0gJChgPGRpdiBpZD0iaHRtbF84NDBlYjAyYTJiYmY0ZGIxODNiZmVjMDc3YmI3ZGFhNiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U2Fsc2FyaXRhJiMzOTtzIEZyZXNoIE1leGljYW4gR3JpbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMTRhMzQ0Y2E3MTAyNDhkZjgxMTFlZTFlZmUzOWM3OGUuc2V0Q29udGVudChodG1sXzg0MGViMDJhMmJiZjRkYjE4M2JmZWMwNzdiYjdkYWE2KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzE2Y2FkNzk0OTMyYzRhYWI5MTE4ODA3ZGFhM2Q3MGMzLmJpbmRQb3B1cChwb3B1cF8xNGEzNDRjYTcxMDI0OGRmODExMWVlMWVmZTM5Yzc4ZSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl84ODY0ZDhlMGVmMTE0NDAyYWNmYWUwNzMzNjYxYjcxZSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjAwMDUsIC04My43Nzg2M10sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMzllN2QwOTg3ZjcxNDdmZmE3MWM2ZGY3YjlhMDAzMDEgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzBiMDk1OTRhZmNlMzRhMTU5OWYzNDU2MjVhOTgyODdlID0gJChgPGRpdiBpZD0iaHRtbF8wYjA5NTk0YWZjZTM0YTE1OTlmMzQ1NjI1YTk4Mjg3ZSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGFjbyBCZWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzM5ZTdkMDk4N2Y3MTQ3ZmZhNzFjNmRmN2I5YTAwMzAxLnNldENvbnRlbnQoaHRtbF8wYjA5NTk0YWZjZTM0YTE1OTlmMzQ1NjI1YTk4Mjg3ZSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl84ODY0ZDhlMGVmMTE0NDAyYWNmYWUwNzMzNjYxYjcxZS5iaW5kUG9wdXAocG9wdXBfMzllN2QwOTg3ZjcxNDdmZmE3MWM2ZGY3YjlhMDAzMDEpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfYWU4OTc4NjI5ZDg5NDNjNGEzN2YxOWUxOGMwZjNkYTUgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45MTUyMiwgLTg0LjA4Nzc5OV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfYTczOWE5MjlkOTIzNGI4MWFkY2U0YmYxZDdjNmYzN2UgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2IxMDU4ZGQ4YTI3ODQ0OTdhZjRhYWY0MmRkMGNiMjU1ID0gJChgPGRpdiBpZD0iaHRtbF9iMTA1OGRkOGEyNzg0NDk3YWY0YWFmNDJkZDBjYjI1NSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QXBwbGViZWUmIzM5O3MgR3JpbGwgKyBCYXI8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfYTczOWE5MjlkOTIzNGI4MWFkY2U0YmYxZDdjNmYzN2Uuc2V0Q29udGVudChodG1sX2IxMDU4ZGQ4YTI3ODQ0OTdhZjRhYWY0MmRkMGNiMjU1KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2FlODk3ODYyOWQ4OTQzYzRhMzdmMTllMThjMGYzZGE1LmJpbmRQb3B1cChwb3B1cF9hNzM5YTkyOWQ5MjM0YjgxYWRjZTRiZjFkN2M2ZjM3ZSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl83ZjQxYzM1ZDk5NmI0MzEwYTg3ZWY4NzUwNzVlZjY1YiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjAwNjQ2MjEsIC04NC4yNTQxMDQ2XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF85YmUzNjQ4OTRjZjA0Y2U5YTVkNDc4ZjYxMzcxM2U2NCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfY2Q2MThiMGVhZDk5NDFiZTkwYWFlNDIwNzAyZTIzM2QgPSAkKGA8ZGl2IGlkPSJodG1sX2NkNjE4YjBlYWQ5OTQxYmU5MGFhZTQyMDcwMmUyMzNkIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5SZWQgTG9ic3RlcjwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF85YmUzNjQ4OTRjZjA0Y2U5YTVkNDc4ZjYxMzcxM2U2NC5zZXRDb250ZW50KGh0bWxfY2Q2MThiMGVhZDk5NDFiZTkwYWFlNDIwNzAyZTIzM2QpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfN2Y0MWMzNWQ5OTZiNDMxMGE4N2VmODc1MDc1ZWY2NWIuYmluZFBvcHVwKHBvcHVwXzliZTM2NDg5NGNmMDRjZTlhNWQ0NzhmNjEzNzEzZTY0KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzAzMTcxYTNkMGI4MjQ3ZmViMTY2ZGVhMGEyODNkMjU4ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMDA2OTI0LCAtODQuMDI2NTQ5XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF82YzhiNGM5OTE0YWU0Njk4ODhiOGJlYjFjYjRmZWJiNCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMzU4Njc4ZGE5YWM2NDVjMmFjNjhlZjZhMzczZWUzY2YgPSAkKGA8ZGl2IGlkPSJodG1sXzM1ODY3OGRhOWFjNjQ1YzJhYzY4ZWY2YTM3M2VlM2NmIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UYWNvIEJlbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNmM4YjRjOTkxNGFlNDY5ODg4YjhiZWIxY2I0ZmViYjQuc2V0Q29udGVudChodG1sXzM1ODY3OGRhOWFjNjQ1YzJhYzY4ZWY2YTM3M2VlM2NmKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzAzMTcxYTNkMGI4MjQ3ZmViMTY2ZGVhMGEyODNkMjU4LmJpbmRQb3B1cChwb3B1cF82YzhiNGM5OTE0YWU0Njk4ODhiOGJlYjFjYjRmZWJiNCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9kNjJkZDBkNGVhMGQ0ZWM0OWNjZTExMWNkOTQwNjZkNCA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljg4MzQxLCAtODQuMTU1NTc3XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF80N2EyZGI1YjM2OWM0NDk4OWQxNGZkMjY0NjgwZWExMSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfZmRkYTYxMDY1MGZjNGRjMThmNGE3MjZjZGVmMTBmYzIgPSAkKGA8ZGl2IGlkPSJodG1sX2ZkZGE2MTA2NTBmYzRkYzE4ZjRhNzI2Y2RlZjEwZmMyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UYWNvIEJlbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNDdhMmRiNWIzNjljNDQ5ODlkMTRmZDI2NDY4MGVhMTEuc2V0Q29udGVudChodG1sX2ZkZGE2MTA2NTBmYzRkYzE4ZjRhNzI2Y2RlZjEwZmMyKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2Q2MmRkMGQ0ZWEwZDRlYzQ5Y2NlMTExY2Q5NDA2NmQ0LmJpbmRQb3B1cChwb3B1cF80N2EyZGI1YjM2OWM0NDk4OWQxNGZkMjY0NjgwZWExMSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl82ZjM0M2ZmMDY3MWU0NzlmOGYyODJiODhkN2FjYTI5NiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1LjkyMTA1MywgLTgzLjg2NjMyOF0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNzYwZDQzY2ZjOTk0NDk1N2IyZjY4ZTQ0ZDVlYTkxNWUgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzczODExMmQ5OGNmMjQ3ODI4ZTA2ODc1MTlhNTc2OGEwID0gJChgPGRpdiBpZD0iaHRtbF83MzgxMTJkOThjZjI0NzgyOGUwNjg3NTE5YTU3NjhhMCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGFjbyBCZWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzc2MGQ0M2NmYzk5NDQ5NTdiMmY2OGU0NGQ1ZWE5MTVlLnNldENvbnRlbnQoaHRtbF83MzgxMTJkOThjZjI0NzgyOGUwNjg3NTE5YTU3NjhhMCk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl82ZjM0M2ZmMDY3MWU0NzlmOGYyODJiODhkN2FjYTI5Ni5iaW5kUG9wdXAocG9wdXBfNzYwZDQzY2ZjOTk0NDk1N2IyZjY4ZTQ0ZDVlYTkxNWUpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfN2E5YmI0NDlhZjA1NDI2NTg4NTk1NTk4YjFkMDU1MWIgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNi4wMTAwNiwgLTgzLjk3NDM1XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9kMTYzM2M4MjJmMjc0MjY0OGNiOGFiMTY4MGRiYmU0YiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNGRiNTdhMDljODQyNDY5ZWIxNTA5YTM5ZTVjMWRlZTUgPSAkKGA8ZGl2IGlkPSJodG1sXzRkYjU3YTA5Yzg0MjQ2OWViMTUwOWEzOWU1YzFkZWU1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5JSE9QPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2QxNjMzYzgyMmYyNzQyNjQ4Y2I4YWIxNjgwZGJiZTRiLnNldENvbnRlbnQoaHRtbF80ZGI1N2EwOWM4NDI0NjllYjE1MDlhMzllNWMxZGVlNSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl83YTliYjQ0OWFmMDU0MjY1ODg1OTU1OThiMWQwNTUxYi5iaW5kUG9wdXAocG9wdXBfZDE2MzNjODIyZjI3NDI2NDhjYjhhYjE2ODBkYmJlNGIpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfZWRhNTdiYTEyNTkzNDlhOWJkMjc5YzVjNTE3ZmE5NTMgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNi4wNDUxNCwgLTgzLjkzNzQyXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF82ODI0Njg4MmM4YmY0ZjRiODAyMjhkN2VkOWI3NTExMiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfOTdhN2Q0MTEwZGRmNDhkZWFjMWM0ZmI0ZTFhMzlmODAgPSAkKGA8ZGl2IGlkPSJodG1sXzk3YTdkNDExMGRkZjQ4ZGVhYzFjNGZiNGUxYTM5ZjgwIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5GYXJtIFRvIEdyaWRkbGUgQ3JlcGVzPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzY4MjQ2ODgyYzhiZjRmNGI4MDIyOGQ3ZWQ5Yjc1MTEyLnNldENvbnRlbnQoaHRtbF85N2E3ZDQxMTBkZGY0OGRlYWMxYzRmYjRlMWEzOWY4MCk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9lZGE1N2JhMTI1OTM0OWE5YmQyNzljNWM1MTdmYTk1My5iaW5kUG9wdXAocG9wdXBfNjgyNDY4ODJjOGJmNGY0YjgwMjI4ZDdlZDliNzUxMTIpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNzdjNjE4ODJhMGY2NGJmYWIyNjA3YzY5MzY0ZjFlNDkgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS45MDI2MTczLCAtODQuMTQ5MzQwNF0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfY2VkOWUzY2YwZjUxNDZiODgxNzI0YmYxYjhmZDJlMjYgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzNiMjg3MWY1M2QxMjRmY2NhODU1MDg5NDhjZjE4NDlmID0gJChgPGRpdiBpZD0iaHRtbF8zYjI4NzFmNTNkMTI0ZmNjYTg1NTA4OTQ4Y2YxODQ5ZiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QmxhemUgRmFzdC1GaXJlJiMzOTtkIFBpenphPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2NlZDllM2NmMGY1MTQ2Yjg4MTcyNGJmMWI4ZmQyZTI2LnNldENvbnRlbnQoaHRtbF8zYjI4NzFmNTNkMTI0ZmNjYTg1NTA4OTQ4Y2YxODQ5Zik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl83N2M2MTg4MmEwZjY0YmZhYjI2MDdjNjkzNjRmMWU0OS5iaW5kUG9wdXAocG9wdXBfY2VkOWUzY2YwZjUxNDZiODgxNzI0YmYxYjhmZDJlMjYpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNzNhMDdlNjBkMGZjNDI1M2IzZDVkYWNmM2JhZTljOGMgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS43NTE2NSwgLTgzLjk5Nzg3XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8yNzI4MGQ1YmEyZmI0MTQzODNjZTA4MDc1NGExZTUwNyA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMmEyODdhMDMzZGFmNGE3N2E3MjBmOTE3NzczYjI4NDkgPSAkKGA8ZGl2IGlkPSJodG1sXzJhMjg3YTAzM2RhZjRhNzdhNzIwZjkxNzc3M2IyODQ5IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UYWNvIEJlbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMjcyODBkNWJhMmZiNDE0MzgzY2UwODA3NTRhMWU1MDcuc2V0Q29udGVudChodG1sXzJhMjg3YTAzM2RhZjRhNzdhNzIwZjkxNzc3M2IyODQ5KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzczYTA3ZTYwZDBmYzQyNTNiM2Q1ZGFjZjNiYWU5YzhjLmJpbmRQb3B1cChwb3B1cF8yNzI4MGQ1YmEyZmI0MTQzODNjZTA4MDc1NGExZTUwNykKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl8wMWIwZWUwOTYyN2I0MTQwYWNkYzk0NjBhMDc2Zjg4YyA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjA2OTAwMSwgLTgzLjkyNzA3XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9hZTFiOTYxYjJhNTE0MmU2YjJkYzQxNjZiN2NkM2M5YyA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfZWMyZjBiNTUzNDhlNDMyMWJjNjMwMTdhMDMzNWRjOTEgPSAkKGA8ZGl2IGlkPSJodG1sX2VjMmYwYjU1MzQ4ZTQzMjFiYzYzMDE3YTAzMzVkYzkxIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UYWNvIEJlbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfYWUxYjk2MWIyYTUxNDJlNmIyZGM0MTY2YjdjZDNjOWMuc2V0Q29udGVudChodG1sX2VjMmYwYjU1MzQ4ZTQzMjFiYzYzMDE3YTAzMzVkYzkxKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzAxYjBlZTA5NjI3YjQxNDBhY2RjOTQ2MGEwNzZmODhjLmJpbmRQb3B1cChwb3B1cF9hZTFiOTYxYjJhNTE0MmU2YjJkYzQxNjZiN2NkM2M5YykKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9jYzdmMzM3ZTdmMTM0NjNkOTBmOGY3ZDFkMTE4ODcyNiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1LjkyMzkzNDE1NzcxOTMsIC04NC4wMzY5MTI2MTQ2NzAzXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF80MzQ4MTU0NzU4Mzk0ZDQ3ODYxNTg5YWZjMzU2ZDE2ZCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNjNkOWMxYThmNDM0NGE1ZDhmNmJkZTNjOTAzY2IzYmQgPSAkKGA8ZGl2IGlkPSJodG1sXzYzZDljMWE4ZjQzNDRhNWQ4ZjZiZGUzYzkwM2NiM2JkIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UYWNvIEJlbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNDM0ODE1NDc1ODM5NGQ0Nzg2MTU4OWFmYzM1NmQxNmQuc2V0Q29udGVudChodG1sXzYzZDljMWE4ZjQzNDRhNWQ4ZjZiZGUzYzkwM2NiM2JkKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2NjN2YzMzdlN2YxMzQ2M2Q5MGY4ZjdkMWQxMTg4NzI2LmJpbmRQb3B1cChwb3B1cF80MzQ4MTU0NzU4Mzk0ZDQ3ODYxNTg5YWZjMzU2ZDE2ZCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9kMTMxMzI2MzljMGI0MDI0ODVmZmU0Y2EzZGE3NmEwYiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1LjkwMzQwMDQyMTE0MjYsIC04NC4xNDgyNjIwMjM5MjU4XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF81ODY0ZmEzODlmYzI0ZjFhYmJjYjlhYTNhMjk3N2IzOCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNTQ5MTI1NTAxZjQ2NGFlNmJkNDY2ZTdmZTViMzFiZjggPSAkKGA8ZGl2IGlkPSJodG1sXzU0OTEyNTUwMWY0NjRhZTZiZDQ2NmU3ZmU1YjMxYmY4IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5PbGl2ZSBHYXJkZW4gSXRhbGlhbiBSZXN0YXVyYW50PC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzU4NjRmYTM4OWZjMjRmMWFiYmNiOWFhM2EyOTc3YjM4LnNldENvbnRlbnQoaHRtbF81NDkxMjU1MDFmNDY0YWU2YmQ0NjZlN2ZlNWIzMWJmOCk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9kMTMxMzI2MzljMGI0MDI0ODVmZmU0Y2EzZGE3NmEwYi5iaW5kUG9wdXAocG9wdXBfNTg2NGZhMzg5ZmMyNGYxYWJiY2I5YWEzYTI5NzdiMzgpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfMDYxZTMwMWFmNGM4NDI3M2IzNmJmNDZhYzU3ZDkwNGYgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS44MTY1ODE2ODE1NzEyLCAtODMuOTgwNjE3NTUxODUwNl0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMmUzMjBlZWFjMDI2NGEyZjlhNDlkMzA2ZGRkZDkxZjQgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2M3Y2FkYTM2ODI5NDQ2N2E4MTI4ZTk0NmJmMjQ4Nzc0ID0gJChgPGRpdiBpZD0iaHRtbF9jN2NhZGEzNjgyOTQ0NjdhODEyOGU5NDZiZjI0ODc3NCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGFjbyBCZWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzJlMzIwZWVhYzAyNjRhMmY5YTQ5ZDMwNmRkZGQ5MWY0LnNldENvbnRlbnQoaHRtbF9jN2NhZGEzNjgyOTQ0NjdhODEyOGU5NDZiZjI0ODc3NCk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl8wNjFlMzAxYWY0Yzg0MjczYjM2YmY0NmFjNTdkOTA0Zi5iaW5kUG9wdXAocG9wdXBfMmUzMjBlZWFjMDI2NGEyZjlhNDlkMzA2ZGRkZDkxZjQpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfM2E3ZjI5NGNjMzMyNGU2YWFkZjZhMTk2N2Q4MzAwMDIgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNi4wNzcyNTQsIC04My45MjY2OTJdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzMwNmRiNmQ3M2Y2NTRjYTE4NTE4MmM5YjVjNGVhZTkzID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9lMTFkODQyNTU5MTk0ZmJjYmEzYTEyMDY4N2QwYjcyNiA9ICQoYDxkaXYgaWQ9Imh0bWxfZTExZDg0MjU1OTE5NGZiY2JhM2ExMjA2ODdkMGI3MjYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPklIT1A8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMzA2ZGI2ZDczZjY1NGNhMTg1MTgyYzliNWM0ZWFlOTMuc2V0Q29udGVudChodG1sX2UxMWQ4NDI1NTkxOTRmYmNiYTNhMTIwNjg3ZDBiNzI2KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzNhN2YyOTRjYzMzMjRlNmFhZGY2YTE5NjdkODMwMDAyLmJpbmRQb3B1cChwb3B1cF8zMDZkYjZkNzNmNjU0Y2ExODUxODJjOWI1YzRlYWU5MykKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9mMmRjMDYwMjcxZDg0ZjI5ODdlMWE1ZTk5MjUyMmY4YiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1LjkwNjQ4LCAtODMuODM2MjVdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzQwMDRjYWMxODAxOTQ5YmNiOTU4OWQxYmU2YzdmNjM4ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9kOWI2ZTBkNTZkODU0ZWUyYjY3M2M4NDU4ZWIxNzczOCA9ICQoYDxkaXYgaWQ9Imh0bWxfZDliNmUwZDU2ZDg1NGVlMmI2NzNjODQ1OGViMTc3MzgiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPklIT1A8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNDAwNGNhYzE4MDE5NDliY2I5NTg5ZDFiZTZjN2Y2Mzguc2V0Q29udGVudChodG1sX2Q5YjZlMGQ1NmQ4NTRlZTJiNjczYzg0NThlYjE3NzM4KTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2YyZGMwNjAyNzFkODRmMjk4N2UxYTVlOTkyNTIyZjhiLmJpbmRQb3B1cChwb3B1cF80MDA0Y2FjMTgwMTk0OWJjYjk1ODlkMWJlNmM3ZjYzOCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9lZDY1ODQxODBiMGQ0NTYzYWYxOGJkMjkzMDhhYjM2NiA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1Ljc1OTc0LCAtODMuOTc1NTIzXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9mMTViMDQ3ZjQ4NTk0MmRlYjkyYTczNTcwOTliZGNlYSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNjU3MjE4Yjk4YjhiNGZmNzk1ZWQ3MDJiMWU3MDdiYmEgPSAkKGA8ZGl2IGlkPSJodG1sXzY1NzIxOGI5OGI4YjRmZjc5NWVkNzAyYjFlNzA3YmJhIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UYWNvIEJlbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfZjE1YjA0N2Y0ODU5NDJkZWI5MmE3MzU3MDk5YmRjZWEuc2V0Q29udGVudChodG1sXzY1NzIxOGI5OGI4YjRmZjc5NWVkNzAyYjFlNzA3YmJhKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2VkNjU4NDE4MGIwZDQ1NjNhZjE4YmQyOTMwOGFiMzY2LmJpbmRQb3B1cChwb3B1cF9mMTViMDQ3ZjQ4NTk0MmRlYjkyYTczNTcwOTliZGNlYSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl8zMGEzYzIyOGVhODU0NDYyOGNhYmI4YzkzYzUzMDRhMCA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1LjkzMDUsIC04NC4wMjY1NTI3XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF82MDg4N2YzZTU5ZmI0MWM4OGNjOWI2ZTMyMGYxNGE1NCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMTZmNGE3ZTBlODIxNGU2MzlmODg1ZGUxOGQ1NmQ4OTUgPSAkKGA8ZGl2IGlkPSJodG1sXzE2ZjRhN2UwZTgyMTRlNjM5Zjg4NWRlMThkNTZkODk1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5JSE9QPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzYwODg3ZjNlNTlmYjQxYzg4Y2M5YjZlMzIwZjE0YTU0LnNldENvbnRlbnQoaHRtbF8xNmY0YTdlMGU4MjE0ZTYzOWY4ODVkZTE4ZDU2ZDg5NSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl8zMGEzYzIyOGVhODU0NDYyOGNhYmI4YzkzYzUzMDRhMC5iaW5kUG9wdXAocG9wdXBfNjA4ODdmM2U1OWZiNDFjODhjYzliNmUzMjBmMTRhNTQpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfZTliNjUxZjk0ZjY2NGJhYjg0M2Y2ZDU3MWNiOWUzMGIgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS44MjM4MiwgLTg0LjI3MDM4XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF80ZTQxZmUxNDU0N2M0N2JmYmMzMGRiZmYzY2RiZWQ3NSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNzM1NDYzYjc2NDlkNDRiMjkxNWU1NjY3YWE4NTZmMGEgPSAkKGA8ZGl2IGlkPSJodG1sXzczNTQ2M2I3NjQ5ZDQ0YjI5MTVlNTY2N2FhODU2ZjBhIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DaGlsaSYjMzk7czwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF80ZTQxZmUxNDU0N2M0N2JmYmMzMGRiZmYzY2RiZWQ3NS5zZXRDb250ZW50KGh0bWxfNzM1NDYzYjc2NDlkNDRiMjkxNWU1NjY3YWE4NTZmMGEpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfZTliNjUxZjk0ZjY2NGJhYjg0M2Y2ZDU3MWNiOWUzMGIuYmluZFBvcHVwKHBvcHVwXzRlNDFmZTE0NTQ3YzQ3YmZiYzMwZGJmZjNjZGJlZDc1KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2QwNGVkMjk2ODc3MjQ5YmU5Y2I1NTlmN2I2ZWE2YWJjID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuOTg3MjA5ODk0MDY5NiwgLTgzLjYwNTg2MDQ3MTcyNTVdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2UyYWE0M2EzY2M3YTRhMjc4NjEzOTI2MWJhNjM5Y2ZjID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9lY2ZlNTdkMjFkNzg0MDc0YTA4ZTcyOWFlMzcwNmUzOSA9ICQoYDxkaXYgaWQ9Imh0bWxfZWNmZTU3ZDIxZDc4NDA3NGEwOGU3MjlhZTM3MDZlMzkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRhY28gQmVsbDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9lMmFhNDNhM2NjN2E0YTI3ODYxMzkyNjFiYTYzOWNmYy5zZXRDb250ZW50KGh0bWxfZWNmZTU3ZDIxZDc4NDA3NGEwOGU3MjlhZTM3MDZlMzkpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfZDA0ZWQyOTY4NzcyNDliZTljYjU1OWY3YjZlYTZhYmMuYmluZFBvcHVwKHBvcHVwX2UyYWE0M2EzY2M3YTRhMjc4NjEzOTI2MWJhNjM5Y2ZjKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzRlMTk4YzUyMTJmNTQ2MGI5MmNiOGM1ODNiN2M1Y2EzID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMjMxOTQ1LCAtODQuMTYwMDIxXSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9hOTk1YWFjZDJjYzc0YjMwOTNkYWE0YTY3MDY4NmIyMCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfZTg1MDU2NTliY2IxNDA3ODk4MTliZDAwZTkxZTNhNGMgPSAkKGA8ZGl2IGlkPSJodG1sX2U4NTA1NjU5YmNiMTQwNzg5ODE5YmQwMGU5MWUzYTRjIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UYWNvIEJlbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfYTk5NWFhY2QyY2M3NGIzMDkzZGFhNGE2NzA2ODZiMjAuc2V0Q29udGVudChodG1sX2U4NTA1NjU5YmNiMTQwNzg5ODE5YmQwMGU5MWUzYTRjKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzRlMTk4YzUyMTJmNTQ2MGI5MmNiOGM1ODNiN2M1Y2EzLmJpbmRQb3B1cChwb3B1cF9hOTk1YWFjZDJjYzc0YjMwOTNkYWE0YTY3MDY4NmIyMCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl82NDU1NDAzNGE3N2M0MWEyODM4YzNhNzI3ODQwNjg1OSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1LjkwMjY1LCAtODQuMTQxMl0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfOTJmZjY0Y2Q5Y2FhNGQxZTllYTY1Yzk2Y2NmZDAzYTUgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzVkYjgyMDZiN2IyMzRhY2ZiMWNhODI3NDdhNTAwNTA0ID0gJChgPGRpdiBpZD0iaHRtbF81ZGI4MjA2YjdiMjM0YWNmYjFjYTgyNzQ3YTUwMDUwNCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+SUhPUDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF85MmZmNjRjZDljYWE0ZDFlOWVhNjVjOTZjY2ZkMDNhNS5zZXRDb250ZW50KGh0bWxfNWRiODIwNmI3YjIzNGFjZmIxY2E4Mjc0N2E1MDA1MDQpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNjQ1NTQwMzRhNzdjNDFhMjgzOGMzYTcyNzg0MDY4NTkuYmluZFBvcHVwKHBvcHVwXzkyZmY2NGNkOWNhYTRkMWU5ZWE2NWM5NmNjZmQwM2E1KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2EzMjI0YTM1Y2I0MTRjMzc4Mzc5ODVhMTIyNGI3ZTQxID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMDM1NzM0MSwgLTgzLjg3NDJdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzdjZTlhMDgzOTdlMTQxNjlhNWFmMmZhMjY1YTQ5OTBiID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8zM2JhMzIxMzZjMjc0OWZhOWM3MDU0NWVhNDg2MjVjOCA9ICQoYDxkaXYgaWQ9Imh0bWxfMzNiYTMyMTM2YzI3NDlmYTljNzA1NDVlYTQ4NjI1YzgiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRhY28gQmVsbDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF83Y2U5YTA4Mzk3ZTE0MTY5YTVhZjJmYTI2NWE0OTkwYi5zZXRDb250ZW50KGh0bWxfMzNiYTMyMTM2YzI3NDlmYTljNzA1NDVlYTQ4NjI1YzgpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfYTMyMjRhMzVjYjQxNGMzNzgzNzk4NWExMjI0YjdlNDEuYmluZFBvcHVwKHBvcHVwXzdjZTlhMDgzOTdlMTQxNjlhNWFmMmZhMjY1YTQ5OTBiKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzRhNzQ3OTI5ZmIyYzQ5MzhhZTdiNDQwNzExMzBhYWZhID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMjI5NDk2MDAyMTk3MywgLTg0LjE1ODQzOTYzNjIzMDVdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2JmODVjYWQwNDVkNTQ0ZmNiM2MzNzAzNGVjYmJiZTU4ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9lYWEzZGEzMzgxY2I0YThhOTE3YTBmYmRmODY3YWJlMyA9ICQoYDxkaXYgaWQ9Imh0bWxfZWFhM2RhMzM4MWNiNGE4YTkxN2EwZmJkZjg2N2FiZTMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNvdHRhZ2UgUmVzdGF1cmFudDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9iZjg1Y2FkMDQ1ZDU0NGZjYjNjMzcwMzRlY2JiYmU1OC5zZXRDb250ZW50KGh0bWxfZWFhM2RhMzM4MWNiNGE4YTkxN2EwZmJkZjg2N2FiZTMpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNGE3NDc5MjlmYjJjNDkzOGFlN2I0NDA3MTEzMGFhZmEuYmluZFBvcHVwKHBvcHVwX2JmODVjYWQwNDVkNTQ0ZmNiM2MzNzAzNGVjYmJiZTU4KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzNlNjQzZjI2NzU0MTQ4OTY5ZWExZWExZDVlZjc3NzEzID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMDA3MjY1LCAtODQuMjU2Mjc0XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9hNDI5NDY3OTk1ODI0MTBiODQxNmUzODAwMWQzOTQ0MSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMTdkMGNkZWU0ZGU1NDMxNWFhMGJjNGU2OWZiMjdkOWUgPSAkKGA8ZGl2IGlkPSJodG1sXzE3ZDBjZGVlNGRlNTQzMTVhYTBiYzRlNjlmYjI3ZDllIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UYWNvIEJlbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfYTQyOTQ2Nzk5NTgyNDEwYjg0MTZlMzgwMDFkMzk0NDEuc2V0Q29udGVudChodG1sXzE3ZDBjZGVlNGRlNTQzMTVhYTBiYzRlNjlmYjI3ZDllKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzNlNjQzZjI2NzU0MTQ4OTY5ZWExZWExZDVlZjc3NzEzLmJpbmRQb3B1cChwb3B1cF9hNDI5NDY3OTk1ODI0MTBiODQxNmUzODAwMWQzOTQ0MSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl81YjAxOGNkZTc1OTc0NTNjYTJkYjQ1OGE5YmZhYmVhYSA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1LjczMTE3LCAtODQuMzg4NzddLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2UzNjhmYmVjMWVkYzQyYWFhZDdhNmJiNjk3ODU2ZDNhID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9kZmYxOGZhODYxYzE0NDU1OTUwNmEyMjQwZTczNDA1YyA9ICQoYDxkaXYgaWQ9Imh0bWxfZGZmMThmYTg2MWMxNDQ1NTk1MDZhMjI0MGU3MzQwNWMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRhY28gQmVsbDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9lMzY4ZmJlYzFlZGM0MmFhYWQ3YTZiYjY5Nzg1NmQzYS5zZXRDb250ZW50KGh0bWxfZGZmMThmYTg2MWMxNDQ1NTk1MDZhMjI0MGU3MzQwNWMpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNWIwMThjZGU3NTk3NDUzY2EyZGI0NThhOWJmYWJlYWEuYmluZFBvcHVwKHBvcHVwX2UzNjhmYmVjMWVkYzQyYWFhZDdhNmJiNjk3ODU2ZDNhKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzZhZTkyOGI3NjVhZDQ5ODA5NmMwOTZlMzIwZjgwNjdmID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuNzk3Njk4LCAtODQuMjU2MzE3XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9jMjQ5MGM5YjJlODE0YWU5YjhjNzkxZDk4ZjJkZWM1MCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfOTM1MjRmODhmMGFlNDkzNGE2NGRkYjcyZTc4MjA2OGQgPSAkKGA8ZGl2IGlkPSJodG1sXzkzNTI0Zjg4ZjBhZTQ5MzRhNjRkZGI3MmU3ODIwNjhkIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UYWNvIEJlbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfYzI0OTBjOWIyZTgxNGFlOWI4Yzc5MWQ5OGYyZGVjNTAuc2V0Q29udGVudChodG1sXzkzNTI0Zjg4ZjBhZTQ5MzRhNjRkZGI3MmU3ODIwNjhkKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyXzZhZTkyOGI3NjVhZDQ5ODA5NmMwOTZlMzIwZjgwNjdmLmJpbmRQb3B1cChwb3B1cF9jMjQ5MGM5YjJlODE0YWU5YjhjNzkxZDk4ZjJkZWM1MCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl9hMWE3YTczZjhjNjY0ODE4YjE0YmMyZTliZWY5YjFjYyA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjI0Njg4NjMwMDc0ODQsIC04My44MTU2NjMxOTYxNDY1XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF81ZTdhOThkNDgwYmQ0MGY2YTI5ZTc3YTE5OWYzNzZkNiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNTJiZDBjYjU4OWY2NDlmMmIxNzViN2UwMTE2YzllYjEgPSAkKGA8ZGl2IGlkPSJodG1sXzUyYmQwY2I1ODlmNjQ5ZjJiMTc1YjdlMDExNmM5ZWIxIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UYWNvIEJlbGw8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNWU3YTk4ZDQ4MGJkNDBmNmEyOWU3N2ExOTlmMzc2ZDYuc2V0Q29udGVudChodG1sXzUyYmQwY2I1ODlmNjQ5ZjJiMTc1YjdlMDExNmM5ZWIxKTsKICAgICAgICAKCiAgICAgICAgbWFya2VyX2ExYTdhNzNmOGM2NjQ4MThiMTRiYzJlOWJlZjliMWNjLmJpbmRQb3B1cChwb3B1cF81ZTdhOThkNDgwYmQ0MGY2YTI5ZTc3YTE5OWYzNzZkNikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcmtlcl80ZmI5MDViNWJjOTU0YmJmYTUzNWY4MmJjMzAwMDM3NCA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzM1LjcxNzI0NiwgLTg0LjAxMjQ3NjNdLAogICAgICAgICAgICAgICAge30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfZmIwNGFlZTEyOTY3NGQ2YWFjZWJhZGYzYTlmNGNhYjMpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzIzZjg3NzQxNTY4MjQ0YmQ5YmRiMWNjOTBjZGY5OWU1ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8wNzY1NTQxZjMzOWI0Zjg1OTFhZDk3MTRlZDFmMDM2MCA9ICQoYDxkaXYgaWQ9Imh0bWxfMDc2NTU0MWYzMzliNGY4NTkxYWQ5NzE0ZWQxZjAzNjAiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRhY28gQmVsbDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8yM2Y4Nzc0MTU2ODI0NGJkOWJkYjFjYzkwY2RmOTllNS5zZXRDb250ZW50KGh0bWxfMDc2NTU0MWYzMzliNGY4NTkxYWQ5NzE0ZWQxZjAzNjApOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfNGZiOTA1YjViYzk1NGJiZmE1MzVmODJiYzMwMDAzNzQuYmluZFBvcHVwKHBvcHVwXzIzZjg3NzQxNTY4MjQ0YmQ5YmRiMWNjOTBjZGY5OWU1KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzAwZTJjOTYyZTM0MDQ2NTBiOGU1ZjcyNDc2OTE3OGIxID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuNzcyMTM5LCAtODMuOTg1MTM1XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF85NjM3YjQ3M2RkZjc0MTkwODFmNWE4NjY1NDc0MmY3NiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfODhmZDZiMmVkNmVlNDkwZDg5YzUzYTM1YjZhMTMxMGUgPSAkKGA8ZGl2IGlkPSJodG1sXzg4ZmQ2YjJlZDZlZTQ5MGQ4OWM1M2EzNWI2YTEzMTBlIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5NY0FsaXN0ZXImIzM5O3MgRGVsaTwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF85NjM3YjQ3M2RkZjc0MTkwODFmNWE4NjY1NDc0MmY3Ni5zZXRDb250ZW50KGh0bWxfODhmZDZiMmVkNmVlNDkwZDg5YzUzYTM1YjZhMTMxMGUpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfMDBlMmM5NjJlMzQwNDY1MGI4ZTVmNzI0NzY5MTc4YjEuYmluZFBvcHVwKHBvcHVwXzk2MzdiNDczZGRmNzQxOTA4MWY1YTg2NjU0NzQyZjc2KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2U4MmE0YzBkOTI1NzRhNzI4Yzk4ZWEyNGZjNDBlYWU4ID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzYuMTE1MzMsIC04NC4xMjAzM10sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfOWIwZDc5OWVhZjk5NGM2M2IxMzg4Y2Y5MDM4MmJmMGEgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2UyYjRlNjcwYWFlMzQ5ZjI5MjgwZjJkZWQ4YzM5Y2VlID0gJChgPGRpdiBpZD0iaHRtbF9lMmI0ZTY3MGFhZTM0OWYyOTI4MGYyZGVkOGMzOWNlZSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGFjbyBCZWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzliMGQ3OTllYWY5OTRjNjNiMTM4OGNmOTAzODJiZjBhLnNldENvbnRlbnQoaHRtbF9lMmI0ZTY3MGFhZTM0OWYyOTI4MGYyZGVkOGMzOWNlZSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl9lODJhNGMwZDkyNTc0YTcyOGM5OGVhMjRmYzQwZWFlOC5iaW5kUG9wdXAocG9wdXBfOWIwZDc5OWVhZjk5NGM2M2IxMzg4Y2Y5MDM4MmJmMGEpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfYzZiYjFjNDFlZGI0NDZlMDg3ZTQ0YTk0YjQ1MmRkZjMgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNi4wMDcxMjEsIC04NC4yNTU3MV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMTJkZDU1ZWQ1ZjZkNDJhNjgwZTM5ODUzYWE3ZWU2Y2UgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzUwYmNkZWRiMDA1NTRmZjZhY2IyZGU2Y2U5ODIzOTYzID0gJChgPGRpdiBpZD0iaHRtbF81MGJjZGVkYjAwNTU0ZmY2YWNiMmRlNmNlOTgyMzk2MyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+SUhPUDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8xMmRkNTVlZDVmNmQ0MmE2ODBlMzk4NTNhYTdlZTZjZS5zZXRDb250ZW50KGh0bWxfNTBiY2RlZGIwMDU1NGZmNmFjYjJkZTZjZTk4MjM5NjMpOwogICAgICAgIAoKICAgICAgICBtYXJrZXJfYzZiYjFjNDFlZGI0NDZlMDg3ZTQ0YTk0YjQ1MmRkZjMuYmluZFBvcHVwKHBvcHVwXzEyZGQ1NWVkNWY2ZDQyYTY4MGUzOTg1M2FhN2VlNmNlKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyXzYwNzYyOWJhY2RlZjQwMWM4ZWM3YWMwZmVjNmJiYTBkID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbMzUuODczMDMsIC04My43NjA0MV0sCiAgICAgICAgICAgICAgICB7fQogICAgICAgICAgICApLmFkZFRvKG1hcF9mYjA0YWVlMTI5Njc0ZDZhYWNlYmFkZjNhOWY0Y2FiMyk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNDYzY2MxMGQzN2QyNGE0MTk3ZWUxZjZkMjkwZjRlM2QgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzE1Nzg2YjBjMjhmNDRmNDBhNDhlNjRmZjA2NjI3YjVmID0gJChgPGRpdiBpZD0iaHRtbF8xNTc4NmIwYzI4ZjQ0ZjQwYTQ4ZTY0ZmYwNjYyN2I1ZiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGFjbyBCZWxsPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzQ2M2NjMTBkMzdkMjRhNDE5N2VlMWY2ZDI5MGY0ZTNkLnNldENvbnRlbnQoaHRtbF8xNTc4NmIwYzI4ZjQ0ZjQwYTQ4ZTY0ZmYwNjYyN2I1Zik7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl82MDc2MjliYWNkZWY0MDFjOGVjN2FjMGZlYzZiYmEwZC5iaW5kUG9wdXAocG9wdXBfNDYzY2MxMGQzN2QyNGE0MTk3ZWUxZjZkMjkwZjRlM2QpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBtYXJrZXJfNjZhZjMxZDM2OTNhNDQ3NTgzNjgwOWVhODJhNGQxOTMgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFszNS43NTI5NSwgLTgzLjk5NTg4XSwKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2ZiMDRhZWUxMjk2NzRkNmFhY2ViYWRmM2E5ZjRjYWIzKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8xOWQ3NWE4MWMyMWY0OTExYmNlNjQ5ZmQzNDA4OTgwZCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfN2Y3NDM3OWMzNTE0NDY3YmExYzljMjFhNGEzNDk2ZDkgPSAkKGA8ZGl2IGlkPSJodG1sXzdmNzQzNzljMzUxNDQ2N2JhMWM5YzIxYTRhMzQ5NmQ5IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5JSE9QPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzE5ZDc1YTgxYzIxZjQ5MTFiY2U2NDlmZDM0MDg5ODBkLnNldENvbnRlbnQoaHRtbF83Zjc0Mzc5YzM1MTQ0NjdiYTFjOWMyMWE0YTM0OTZkOSk7CiAgICAgICAgCgogICAgICAgIG1hcmtlcl82NmFmMzFkMzY5M2E0NDc1ODM2ODA5ZWE4MmE0ZDE5My5iaW5kUG9wdXAocG9wdXBfMTlkNzVhODFjMjFmNDkxMWJjZTY0OWZkMzQwODk4MGQpCiAgICAgICAgOwoKICAgICAgICAKICAgIAo8L3NjcmlwdD4=" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




```python

```

## Summary

Nice work! In this lab, you synthesized your skills for the day, making multiple API calls to Yelp in order to paginate through a results set, performing some basic exploratory analysis and then creating a nice map visual to display the results! Well done!
