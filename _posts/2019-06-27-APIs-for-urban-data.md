---
layout: post
title: "APIs for urban data"
author: "Sokratis Papadopoulos"
categories: journal
tags: [documentation,sample]
image: APIpost/api_icon.png
---


The phrase "API calls" often sounds intimidating to people with little experience in data analytics. But, honestly, it doesn't take more than 5 lines of code to do an API request.

In this post I am showing how to access urban data through APIs. Specifically, we will get information about New York City's restaurants through the Yelp API, and find the neighborhoods and cuisines with the highest ratings. 

## Define neighborhood

First we need to define NYC neighborhoods. An approach would be to split the city by the zip codes, but this can be hard to interpret. I find Airbnb's neighborhood definition very intuitive and you can access the neighborhood boundary _geojson_ file for NYC  [here](http://data.insideairbnb.com/united-states/ny/new-york-city/2019-06-02/visualisations/neighbourhoods.geojson).

I use [Folium]([https://python-visualization.github.io/folium/](https://python-visualization.github.io/folium/)) to plot the map of NYC neighborhoods and their centroids.

![](/assets/img/APIpost/nyc_neighborhoods.png  "NYC neighborhoods")

## Call Yelp API

After you get your key [here](https://www.yelp.com/developers/v3/manage_app), here is what you need to do to interact with the API.

```python
def getYelpData(apiKey, businessCat, location):
    
    """Calls Yelp API and returns a list with business information in the given location.
    
    Args:
        api_key (str): Yelp API key.
        business_cat (str): Business category for request.
        location (str): Location of interest (e.g. East Village, NY).
    
    Returns:
        df : Dataframe including all available business information (e.g. name, rating, address, etc.)
        
    """
    
    # authorize API call
    headers = {'Authorization': 'Bearer %s' % apiKey}
    
    # API url
    url = 'https://api.yelp.com/v3/businesses/search'
    
    # set request parameters
    params = {'term': businessCat, 'location': location, 'limit': 50}
    
    # make the request
    req = requests.get(url, params=params, headers=headers)
    
    # read request as json
    j = req.json()
    
    # convert to dataframe
    df = pd.DataFrame(j['businesses'])
    
    return df
```

The function arguments are the business category, which in this context is restaurants, and location, which is extracted from the geojson we processed earlier. 

It's easy to see that the actual request is essentially one line of code! Then, I am reading the request as json and convert it to a pandas dataframe for easier data manipulation later on.

There are also Python packages like [this]([https://github.com/Yelp/yelp-python](https://github.com/Yelp/yelp-python)) and [this]([https://github.com/gfairchild/yelpapi](https://github.com/gfairchild/yelpapi)) that simplify the requests of Yelp API, but I personally prefer writing it from scratch to better understand the ins and outs of the process. 

## Insights

Once we query Yelp for each neighborhood we can aggregate the data and get meaningful insights. 

### Highest-rated neighborhoods

We see that Williamsburg, Greenpoint, Lower East Side, and Astoria have among the highest ratings, but if you are really a foodie Sunnyside is your place (red marked area). 


![](/assets/img/APIpost/choropleth.png  "Neighborhood rating choropleth")


### Highest-rated cuisines

What about the debate of which cuisine is best? Well, data can answer that. 

<p align="center">
<img src="/assets/img/APIpost/cuisineYelpRatings.png" width="450" height="600" title="Cuisine ratings">
</p>
Wine bars are leading the chart, followed by French. Korean, Mediterranean, and Greek complete the top 10 (way to go Greece!). 

## Final thoughts

Here we show how to request data from the Yelp API in few easy steps. There is no shortage of similar APIs to play with (Foursquare, Google Places, Zomato to name a few). Such data can be used in all sorts of urban analyses from measuring neighborhood quality to identifying new hot spots. 
