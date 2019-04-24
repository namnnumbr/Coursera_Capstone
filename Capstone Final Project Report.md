# Capstone for Coursera's IBM professional certificate in data science
## Final Project - Location Similarities
*If I like where I'm living now, but need to move, where should I look in the new city?*  

### Part I: Introduction  

Having to move is a huge undertaking.  Before you can find a new apartment/home and plan the logistics of the move, first you must identify the neighborhood in which to search for your home!  In this project, we identify characteristics that define your current location/neighborhood (or your ideal one) and find the most similar neighborhoods in your destination location.

For example, I'm currently living in a neighborhood outside of Philadelphia.  I really like it.  What if I want to buy a house, and I'm looking for similar neighborhoods locally in which to look.  This tool will help me identify other neighborhoods like my current one.  
This tool could be extended to help find similar neighborhoods in other cities, as well.

### Part II: Data  

Attributions:
Zip code geolocation coordinates were provided via Zillow's 'GetRegionChildren' API. Assuming we know the county (or counties) in which we're interested in living, the Zillow API allows us to collect all zip codes from within these counties. 

[![Zillow](https://www.zillowstatic.com/vstatic/7de9b24/static/logos/Zillow_Logo_HoodsProvided_RightAligned.gif "Zillow")](https://www.zillow.com/howto/api/neighborhood-boundaries.htm)

Feature data for a one mile radius of the center of each zip code of interest was provided via FourSquare's API.  In particular venues names, categories, and coordinates are used to identify the characteristics of each given location.

[![FourSquare](https://upload.wikimedia.org/wikipedia/commons/thumb/d/dc/Foursquare_logo.svg/320px-Foursquare_logo.svg.png "FourSquare")](https://developer.foursquare.com/docs/terms-of-use/overview)  

### Part III: Methodology  

In order to understand the similarity between areas, we required several distinct types of data: 
1. A way to define an area's location geographically.
2. A way to define the characteristics of a location.
3. A 'source' location used as an example to search against.
4. A pool of 'destination' locations used to find the final subset of top destination areas.

The aforementioned APIs provided the data.  In our example, I use my current zip code as the 'source' location, and get the feature data for my current location from Foursquare.  Then, I use Zillow's API to get all of the zip codes in nearby counties - I want to move out of my apartment, but stay in the area.  Using that list of zip codes, we again query FourSquare to get the feature data for each of those zip codes.

Before comparing locations based on features, we have to do some data transformations.  I want to define each location by not only what venues are available within a 1 mile (1600m) radius, but also by how far they are.  FourSquare's API provides a 'distance' metric, and this is defined as distance in meters from the central point of the zip code.  This means that the higher the number, the further away these venues are.  However, what if there is no venue of a particular type within the 1 mile radius?  If I set distance to 0, it implies the venue would be at the exact center of the area.  If I set the distance arbitrarily high, it will throw off our algorithm.  Therefore, I invert the distance metric by subtracting the distance from the edge of the 1600m radius.  This inverse distance means that the *lower* the number, the further away these venues are.  And if the venue does not exist in the area (or if it's further away than 1 mile), then its inverse distance is 0.  If there are several different venues of the same category, I simply average their inverse distances.

After wrangling, each zip code is defined by a list of venue categories, with a number between 0 and 1600 representing the distance to the 'average location' of each venue category.  We have a numeric vector representation of each zip code.

An algorithm called k-nearest-neighbors is designed to find some number (k) of the most similar points to an example.  This is exactly what we are trying to do.  The sklearn implementation is very quick, and I simply tell it I'm looking for the 5 most similar zip code 'neighbors' to my source zip code, where similarity (or 'nearness') is defined by cosine distance.  I use cosine distance because I'm don't necessarily care that all of the venues and categories are in specific locations in the destination location, but that they're oriented similarly - that there are grocery stores close by, a bar or two, etc.

### Part IV: Results  

When choosing the 10 most similar zip codes to my current one, the algorithm performs as expected.  I am familiar with the local area, and all of the areas pinpointed on my map demonstrate a similar distribution of restaurants, shops, and services.  In fact, one of the major determining factors seems to be shopping centers - there are two within my current zip code detection radius, each of the similar zip codes also has a shopping center or shopping strip.

### Part V: Discussion  

As noted, the driving features that define similarity between my current zip code and others seem to be a tight clustering of shops, such as would occur in a shopping center.  This is indeed a defining characteristic of my location, and indicates that the k-nearest-neighbors algorithm is learning the relationships between local venues properly.  
Looking forward, given that the intent behind this project is to help find a new place to live, it would be very interesting to include cost-of-living information, as well as housing pricing and demographics information.  These additional data would make the comparison between zip codes more directly useful in terms of cost of living decisions.  Currently, the prediction is simply based on ensuring the new destination location has similar amenties.

### Part VI: Conclusion

As I'm currently looking for a new place to live, this project was very timely!  I learned an incredible amount about geographic mapping, hitting APIs to request data, and applying clustering techniques to understand similarities between groups.  
If you're interested in a similar activity, please feel free to use and share my code!  You'll need your own credentials for the Zillow and FourSquare APIs, though.
