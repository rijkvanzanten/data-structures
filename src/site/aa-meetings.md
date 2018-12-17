---
layout: base.ejs
title: AA Meetings | Data Structures
---

# AA Meetings Map

_[GitHub](https://github.com/rijkvanzanten/ds-fa-1) • [Live Demo](https://ds-fa-1.rijks.website)_

## Brief

You will scrape all ten zones of the [New York's AA Meeting List](https://parsons.nyc/aa/m01.html) to capture, clean, and store all meetings in Manhattan. The meetings data should be stored in PostgreSQL and queried using Node.

## Extra challenge: Natural Language Processing

The user most likely wants to search for specific locations, times, or days. When you live in the Lower East Side and you have a fulltime job, you're probably not interested in Monday morning meetings in West Harlem. Initially I was initially thinking to include a search widget that had a variety of filter options. I figured it could be made even easier by using Natural Language Processing (NLP). This allows the user to type in what they were looking for, instead of having to go through the whole form in order to search and filter.

## Data Structure

We were warned at the beginning of the course that the data that we had to parse was going to be very _very_ nasty. And yeah, yeah, I do agree with that. My god it was annoying to extract the data. Not even the coding bit, even trying to read the page as a human was not the easiest thing in the world. I eventually extracted the following entities from the pages:

* Locations  
  The physical locations the meetings are held
* Zone  
  A location is in a particular zone. While these zones aren't necessary for the end user (I think), the "client" probably had them for a reason
* Neighborhoods  
  Each zone contains multiple neighborhoods
* Zipcodes  
  Each neighborhoods covers a few zipcodes
* Meetings  
  Not too sure if this is the right name, but these are the individual "groups" of events that happen at a single location
* Meeting Hours  
  The actual day and time a certain "meeting" group came together
* Meeting Types  
  Each meeting can be of a single type (Closed, Discussion, Book)

I figured it would be wise to separate and normalize the data as much as possible, to be able to search and query in creative ways later down the line.

I came up with the following data structure:

![ERD](/aa-erd.png)

## Design Inspiration

I think the general layout of Apple Maps on iOS works pretty well. It has a huge emphasis on the map itself, but manages to display the search form on top in a way that feels very "integrated", by using translucency and the famous Apple "frosted glass" effect.

![Inspiration: Apple Maps](/aa-inspiration.png)

## Map Library Research

The final assignment brief mentioned using [Leaflet](https://leafletjs.com) as a map library (using OpenStreetMap as a map provider). While the library is very easy to use, I had one gripe with it: it just looks kinda... ugly. 

![Leaflet](/aa-leaflet.png)

I figured it would be a good idea to see what other options where available:

#### Google Maps

Google Maps is one of the most used map providers out there. While the map SDK itself is pretty easy to use, getting a token in Google's console is anything but. Changing the styling of the thing is more difficult than it has to be too. I haven't been able to tweak the styling of the map in a way I'm happy with. 

![Google Maps](/aa-google-maps.png)

#### Mapbox

Mapbox is "the location data platform for mobile and web applications". They handle maps, datasets, POIs and a lot more. What's really nice about Mapbox is the fact that their maps are built up from vector layers that are 100% customizable. If you wanted, you could make the oceans pink, remove all forests and plop 3D buildings on there.

![Mapbox](/aa-mapbox.png)

#### MapKit JS

MapKit is Apple's offering in the web-map space. I do like the way Apple's maps look in their own software, so I was enthusiastic to give it a spin. [Their examples](https://developer.apple.com/maps/mapkitjs/) look surprisingly easy and their pricing is very competitive; giving a quarter million views a day for free. When I was trying to create an access token I realized a very big downside: you have to be a registered Apple Developer to use it. To become an Apple Developer, you have to shell out a 100 bucks. Parsons is already expensive enough as is, so that was a hard pass.

![Apple MapKit JS](/aa-apple-maps.png)

### Conclusion

| Provider    | Ease of Use | Ease of Setup | Pricing | Customizability | Markers / Popups |
|-------------|-------------|---------------|---------|-----------------|------------------|
| Leaflet     | +++         | +++           | +++     | ---             | -                |
| Google Maps | ++          | ---           | -       | ---             | --               |
| Mapbox      | +++         | +++           | -/+     | +++             | +++              |
| Apple Maps  | ++          | ++            | ---     | ?               | +++              |

I decided to go with Mapbox

## Natural Language Processing

I wanted to provide the user with the ability to not just search, but filter using a single search field. This means that I had to have some sort of machine learning algorithm that would learn to extract keywords and meanings from full sentences.

For example "Closed Discussion meetings in Chelsea on Friday Afternoon" should search for meetings of the Closed Discussion type that are in or in a radius of Chelsea (lat/long) on friday (day) in the afternoon (after 12pm).

I am by no means a scientist or a mathematician, and there's only so much time I can spend teaching a computer to speak English, so I opted to use an existing NLP service. I chose to go with [Wit.ai](https://wit.ai) primarily because I had played with it before and was somewhat familiar with how it works.

It works rather simple, you feed it a sentence and tell the machine what the different parts mean. This is what it looks like for the above example:

![Wit AI](/aa-wit.png)

After you've told it for the 999th time that Chelsea is in fact a neighborhood in NYC, and _not_ [a city in Massachusetts](https://en.wikipedia.org/wiki/Chelsea,_Massachusetts), it will start returning what it _thinks_ is being said in the sentence, providing a "certainty" score:

```json
{
  "_text": "Closed Discussion meetings in Chelsea on Friday Afternoon",
  "entities": {
    "meeting_type": [
      {
        "confidence": 0.94730779491866,
        "value": "Closed Discussion",
        "type": "value"
      }
    ],
    "neighborhood": [
      {
        "metadata": "40.7465,-74.0014",
        "confidence": 0.9927278731258,
        "value": "Chelsea",
        "type": "value"
      }
    ],
    "datetime": [
      {
        "confidence": 0.93921333333333,
        "to": {
          "value": "2018-12-21T19:00:00.000-05:00",
          "grain": "hour"
        },
        "from": {
          "value": "2018-12-21T12:00:00.000-05:00",
          "grain": "hour"
        }
      }
    ]
  },
  "msg_id": "10CDoSYME5upk8fiI"
}
```

While it was more effort than I care to admit to provide it with enough training data to start returning accurate predictions, I'm very happy with the result. You can now search on the map for location, day, and time. I had some trouble with the meeting type (it started recognizing the letter `b` as `book meeting` because I fed it corrupted training data) so that didn't make the cut.

## Data fetching

I wanted to show a full overview of the whole island of Manhattan with all the locations plotted on it (clustered). To achieve this, the server has to retrieve all locations on page load. I'm only fetching the bare minimum of data in order to be able to show the dots on the map (`id`, `lat`, and `long`). When you click on one of the dots, the app sends a new request to the server to fetch all the information for that particular location.

When you search for a particular set, the server only returns the `id` fields of the locations that are still relevant after which the app will reset the dataset to only use those ids. It will maintain a copy of the original set, so it doesn't have to refetch all the ideas when the user clears the search query.

I'm using the GeoJSON data format in the initial load to render the map markers more efficiently using Mapbox. The clusters are handled by mapbox as well.

## Result

![AA Map Default State](/aa-1.png)

![AA Map w/ search query](/aa-2.png)

[Checkout the live demo!](https://ds-fa-1.rijks.website)

## Conclusions / final thoughts

* I had never worked with PostgreSQL before, so that was a bit scary at the beginning. I'm happy to report that the learning curve from MySQL to PostgreSQL is very low.
* I'm really pleased with Mapbox as a map library. The current tilted opening screen is gorgeous.
* I was initially a bit sceptical about using NLP as the only way of filtering and searching, but I gotta admit that it works pretty well! I'm wondering if enough users expect it to work that way, but that's something that more user testing has to prove.

### Future improvements

* I only use the locations, meetings, and meeting_hours tables for my final assignment. The complexity of the database can be reduced by quite a lot. 
* The time it takes before the individual.
* Add support for searching for meeting types.
* I think it would be nice to add a small preview of the search results below the search form.
* Filter out the popup content based on the search filters. Right now it still shows all the meetings for a location even if only 1 of the meetings fits the description.
* Styling of the popups: they feel a bit unpolished compared to the map itself
