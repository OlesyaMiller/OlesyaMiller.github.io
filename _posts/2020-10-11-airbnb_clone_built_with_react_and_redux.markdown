---
layout: post
title:      "Airbnb Clone Built with React and Redux"
date:       2020-10-11 16:38:58 +0000
permalink:  airbnb_clone_built_with_react_and_redux
---


For my final project I decided to build this simple airbnb clone for Puerto Rico. I named it PRairbnb. 
It lets the user create listings and leave reviews for the listings. My project has two directories - client side and server side. The server side is responsible for handling the data and the client side handles the front-end part of the application. The technologies used for the front-end are React and Redux. Let me walk you through the builing process of this app. 

To see the details of an individual listing the user has to click the listing which takes them to that particular listing's page. I was able to enable that functionality by implementing nested routes in my app. In my ListingsContainer I mount a Route component with a path /listings/:listingId. Inside the Route component I mount a render function. The render function gives the ListingShow component, that I map to the Route, access to routerProps and allows me to pass other props to ListingShow. routerProps is a nested object that has keys of history, location and match.

```
<Route path="/listings/:listingId" render={routerProps => <ListingShow {...routerProps} listings={this.props.listings}/>} />
```

In my ListingShow component I have access to the match prop. The match prop is an object that contains a params key that has key/value pairs parsed from the url. The pair we are looking for is listingId with a value of an id. Now we can find the listing with that id!

```
const listing = listings.find(listing => listing.id === parseInt(match.params.listingId))
```

That lets us dynamically display the data of individual listings.

In ListingShow I also mount  ReviewsContainer and ListingMap and pass those components that listing we found by id as a prop. In the ReviewsContainer we need it to be able to associate reviews with a listing, and in ListingMap we need that listing to be able to access the latitude and longitude of the listing's location.

Now let me walk you through how I implemented the review feature in my app. My ReviewsContainer component is mounted in my ListingShow component where it is passed a listing prop with an id that matches the id in the url. That lets me associate the reviews with a listing they belong to. In the ReviewsContainer component I mount ReviewInput and Reviews components and pass both of them a listingId prop.

```
<ReviewInput addReview={this.props.addReview} listingId={this.props.listing.id} />
<Reviews reviews={this.props.reviews} listingId={this.props.listing.id} />
```

Then in my ReviewInput in the handleOnSubmit function I pass the addReview action an object with two key/value pairs, content and listingId. Our addReview action needs a listingId to be able to associate the review with a listing.

```
 handleOnSubmit = (event) => {
        event.preventDefault()
        this.props.addReview({content: this.state.content, listingId: this.props.listingId })
        this.setState({ content: "" }) 
    }
```

In my Reviews component I iterate over all the reviews using filter to collect all the reviews associated with the listing

```
const associatedReviews = this.props.reviews.filter(review => review.listing_id === this.props.listingId)
```

Now I can successfully display reviews under a listing they beling to.

Another feature that my application has is embedded Google Maps. To integrate Google Maps into my React application I used Google Maps API. To be able to use the API first you have to install a dependancy by running this command: 

```
npm install google-maps-react@2.0.6
```

You also have to import Map and GoogleApiWrapper components from 'google-maps-react';

Once you have those ready, here is what your ListingMap component should look like:

```
const mapStyles = {
  width: '100%',
  height: '100%'
};

export class MapContainer extends Component {
  render() {
    return (
      <Map
        google={this.props.google}
        zoom={14}
        style={mapStyles}
        initialCenter={
          {
            lat: this.props.latitude,
            lng: this.props.longitude
          }
        }
      />
    );
  }
}

export default GoogleApiWrapper({
  apiKey: 'YOUR_GOOGLE_MAPS_API_KEY_GOES_HERE'
})(MapContainer);
```

I dynamically set the value of latitude and longitude for each location by passing the props to the ListingMap component in my ListingShow component.

```
<ListingMap latitude={listing.location.latitude}
                          longitude={listing.location.longitude}
 />
```

The latitude and longitude properties come from my Location model in the backend directory. I was able to dynamically create Location instances and assign them the name, latitude and longitude properties in my seeds file by iterating over a local JSON file that contains data for all cities in Puerto Rico. I downloaded that file from simplemaps.com

```
cities = File.read('/Users/user/Development/code/portfolio-projects/react-redux-final-project/server-side/db/pr.json')

JSON.parse(cities).each do 
    |city| Location.create(name: city["city"], latitude: city["lat"], longitude: city["lng"])
end
```







