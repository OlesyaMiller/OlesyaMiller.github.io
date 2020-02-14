---
layout: post
title:      "Quick Dinner Recipes CLI App "
date:       2020-02-14 02:48:12 -0500
permalink:  quick_dinner_recipes_cli_app
---


I am very excited to present you my Quick Dinner Recipes CLI App!
I decided to build a recipes app because I love cooking and think that an app like that would come in handy for a cooking aficionado.

The purpose of this app is to let user find a recipe that suits their time frame and diet. 

I scraped simplyrecipes.com website for my project because it has all the necessary data for creating class instance attributes and filtering the results by those attributes. 
My app has a return-to-the-main-menu feature that lets the user keep interacting with the app by typing ‘start over’ until they exit the program by typing ‘exit’. My project goes three levels deep.

In my Scraper .scrape_page I make one Nokogiri request to scrape the data

```
doc = Nokogiri::HTML(open("https://www.simplyrecipes.com/recipes/type/quick/dinner/"))
```

I then access the css for each recipe and assign it to a variable

```
details = doc.css("div.grd-tile-detail")
```

I iterate over the return value of that variable to create a hash for every recipe and I push those hashes into an empty array. Since not every recipe has a diet type listed, I check if a recipe has a food_type attribute before adding a  food_type key-value pair to each recipe hash.

I pass the array of hashes returned by the Scraper .scrape_page as an argument to my Recipe .create_from_collection to create an instance for every recipe by mass-assigning key-value pairs to each recipe using meta-programming in my Recipe #initialize. 

```
def initialize(recipe_hash)
        recipe_hash.each do |key, value|
            self.send "#{key}=", value 
        end
        @@all << self
    end

    def self.create_from_collection(recipes_array)
        recipes_array.each do |recipe|
            self.new(recipe)
        end
    end
```

Upon launching the app the user is greeted with a message with options to display all recipes, to browse recipes by diet or to browse recipes by prep time. 

The ‘search by diet’ input triggers #list_recipes_by_diet where the user is prompted to enter a name of a diet from the list. In order to display the matching recipes in a numbered list, I first iterate over Recipe.all to select recipes whose food_type attribute includes the entered diet name, save the returned array in @selected_recipes (the reason that variable is a class variable is because I use it in another method to display recipes by number) and only afterwards I use each.with_index( )  to iterate over that array to print out each recipe’s title in a numbered list

```
selected_recipes = Recipe.all.select do |recipe|
       recipe.food_type != nil && recipe.food_type.downcase.include?(input.downcase)
end

selected_recipes.each.with_index(1) do |selected_recipe, index|
       puts "" 
       puts "#{index}. #{selected_recipe.title}"
end
```

The user is able to browse recipes by diet by entering different diet names until they find a recipe they like. Once a recipe number is entered the user is taken to the next level of the program where again they can look up recipes information by entering different recipes numbers from the last list displayed by the previous level. 

Here is how I enable this functionality: 
In my #recipe_by_diet_info (which is triggered after the matching recipes are displayed), I give the user two options - to keep looking up recipes by number or keep browsing recipes by diet 

```
puts "To keep browsing, enter a different diet name”
puts "To see recipe info, enter recipe number"
```

That means that my #recipe_by_diet_info has two conditions - one that checks if the input is a diet name and the other that checks if the input is a number within a range of the matching recipes

```
if (1..@selected_recipes.length).include?(input.to_i)
      . . . . . . . . . .
elsif input.downcase == "healthy" || input.downcase == "gluten-free" || input.downcase == "low carb" || input.downcase == "paleo" || input.downcase == “vegetarian"
      . . . . . . . . .. . . 
```

If the user enters another diet name they are shown a list of matching recipes again (the same functionality can also be executed by using a while loop).  The user stays on that level of the program until they enter a number of a recipe they want to look up. Once the user enters a recipe number they are shown the recipes detailed info and right away are taken to the last level of the program by #last_level_method_for_diet. This method gives the user only one browsing option

```
puts "To look up another recipe, enter recipe number"
```

On this last level of the app the user can look up any recipe from the last list of recipes displayed without relaunching the app until they enter ‘start over’ or ‘exit’.

The reason I created a separate method for looking up recipes by the number is to not let the user keep browsing the recipes by diet name once they have entered a recipe number.
My #last_level_method_for_diet works like a loop by triggering itself after displaying a recipe info or in case of invalid input. It works just like #recipe_by_diet_info except it doesn’t handle displaying recipes by diet any more.

My #list_recipes_by_prep_time works the exact same way as #list_recipes_by_diet. The only challenge I had was to somehow make the program differentiate between recipe numbers and recipe prep times inputs (because both are numbers and a number like 10 can meet two conditions at the same time). I handled that by asking the user to type a number for time in the following format - 15 min - a number followed by the word ‘min’ separated by space. I parse that input afterwards to make my program work the way I want it to

```
if input.split(" ")[0].to_i >= @prep_times.min && input.include?("min")
```

Another thing I had to take care of was avoiding ‘magic numbers’ in my #list_recipes_by_prep_time. Here’s how I handled that

```
@prep_times = Recipe.all.map { |recipe| recipe.prep_time.split(" ")[0].to_i }
puts "All the recipes take under #{prep_times.max} minutes to cook. The quickest recipe is #{prep_times.min} minutes. How much time do you have?"
input = gets.chomp 
if input.to_i >= prep_times.min
. . . . . . .  
end
```
















