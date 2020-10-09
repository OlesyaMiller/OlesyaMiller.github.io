---
layout: post
title:      "JS and Rails SPA Trip Finder "
date:       2020-08-06 10:39:06 -0400
permalink:  js_and_rails_spa_trip_finder
---


Welcome to my JS and Rails SPA Trip Finder app! My app lets you look for trips by type and area and to create new ones. It was a very fun project to build because I had to create my own API with Ruby and Rails and use JavaScript to display the API data. All interactions between the client and the server are handled asynchronously and use JSON as the communication format.

Here is how I was able to implement this functionality.
Since Rails serves just as backend in this project it is considered good practice to add  - - api flag to your new app generator. That lets you configure your application with a more limited middleware than normal and skip generating views, helpers, and assets when you generate a new resource. This is how the file structure looks like:

-	JS-and-Rails-SPA
    - Backend
	    - App
	    - Config
	    - db
	    - other folders and files
    - Frontend
	    - models
	    - index.html
	    - index.js

My app has five models – Trip, Area, Type, Hotel and Attraction. Trip has many Attractions and Hotels and belongs to Type and Area. Attraction and Hotel belong to Trip. Area and Type have many Trips. 
My HTML file has two forms – one for searching for a trip and one for creating one. Both forms have two dropdowns – one for area and one for type which are models in my project. I render types by making a fetch request to the Type index action what lets me grab all of the Type instances and dynamically display them.  This is how I do it:

  1.	In my type.js I create a method called “template”
  2.	The search form has a select tag with an id of #types and the create form has a select tag with an id of #type. In the          #template method I grab both select tags with JavaScript querySelector and assign them to variables
  3.	Then I create an option tag and dynamically assign its value attribute the type id 
  4.	Then I append the option tag to the select tags of the both forms using insertAdjasentHTML method 
  5.	I call the #template method on each instance of the type in my renderTypes function


The code snippet below demonstrated the steps described above:

//index.js

```
const getTypes = () => {
    fetch(`${BASE_URL}/types`)
    .then(resp => resp.json())
    .then(types => {
        types.forEach(type => new Type(type))
        renderTypes()
    })
}

const renderTypes = () => {
    Type.all.forEach(type => type.template())
}
```

//type.js 

```
template() {
        let typesSelect = document.querySelector('#types')
        let typeSelect = document.querySelector('#type')
        let option = `<option value=${this.id}>${this.name}</option>`
        typeSelect.insertAdjacentHTML('beforeend', option)
        typesSelect.insertAdjacentHTML('beforeend', option)
}
```

I use the exact same principle to render areas in both forms.

I’m able to fetch trips based on the type and area criteria by sending a fetch GET request to the URL that contains area id and type id which are values from the dropdown select tags

```
const typeId = document.querySelector("#types").value 
const areaId = document.querySelector("#areas").value

fetch(`${TRIPS_URL}/${areaId}/${typeId}`)
```

To be able to do that I created a custom route in my config/routes.rb 

```
get '/trips/:area_id/:type_id', to: 'trips#index'
```

That lets me send the params hash with a type_id and an area_id to the Trips controller and find the trip I’m looking for

```
trips = Trip.where(area: params[:area_id], type: params[:type_id])
```

My create form, besides the area and type dropdowns that I talked about above, also has fields for city, country, attractions and hotels. City and country are attributes in my Trip model. To be able to let the user to enter as many attractions and hotels as they want, attraction and hotel are models in my app. That lets me iterate over the arrays of attractions and hotels in the Trip controller and create a new instance for each element. Under attractions and hotels fields there is a button that generates a new field on click. I enable that functionality by grabbing the div that contains the input and inserting a new input underneath the last input using insertAdjacentHTML

```
const addAttraction = (e) => {
    e.preventDefault()
    const attractions = document.querySelector("#attractions")
    let input = `<input type="text" name="attraction" id="attraction" class="form-control attraction"><br>`
    attractions.insertAdjacentHTML('beforeend', input)
}
```

Once all the fields in the create form are filled out and the user clicks “Create New Trip”, the fetch POST request is triggered and the params are sent to the create action in the Trips controller

```
def create 
        area = Area.find_by(id: params[:area_id])
        type = Type.find_by(id: params[:type_id])
        trip = Trip.new(city: params[:city], country: params[:country])
        trip.area = area  
        trip.type = type 
        if trip.valid? && params[:attractions] != [""] && params[:hotels] != [""]
            attractions = params[:attractions].select {|attr| attr != ""}
            attractions.each do |attr|
                trip.attractions.build(name: attr)
            end
            hotels = params[:hotels].select {|hotel| hotel != ""}
            hotels.each do |hotel|
                trip.hotels.build(name: hotel)
            end
            trip.save 
            render json: trip.to_json(include: [:attractions, :hotels, :type, :area])
        else
            flash.alert = "Fields cannot be empty"
        end
end
```

I am able to send attractions and hotels as arrays in the params hash by assigning all the attractions input fields the same class and all the hotels input fields the same class, grabbing all the elements by class using querySelectorAll and afterwards iterating over the arrays of elements and pushing the value of each element to a new array

```
const attractionsElements = document.querySelectorAll(".attraction")
let attractionsValues = [ ]
attractionsElements.forEach(element => { attractionsValues.push(element.value) })

const hotelsElements = document.querySelectorAll(".hotel")
let hotelsValues = [ ]
hotelsElements.forEach(element => { hotelsValues.push(element.value) })
```

Then I simply pass the attractions and hotels arrays to the body of the post request.

I hoped you enjoyed reading my blog post! Happy coding!






