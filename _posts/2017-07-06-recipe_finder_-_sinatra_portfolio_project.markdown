---
layout: post
title:  "Recipe Finder - Sinatra Portfolio Project"
date:   2017-07-06 21:59:16 +0000
---


I made an app that lets a user add ingredients and recipes and then tells them what recipes they can make with those ingredients. I think the areas I struggled with the most were the object model relationships and the main recipe finder feature.

For the object model relationships, I had three models: a user model, a recipe model, and an ingredient model. I decided that a user can have many recipes, but recipes should only belong to one user. Ingredients can have many recipes, and recipes can have many ingredients. And, users can have many ingredients, and an ingredient can have many users.

I ended up using two "has_many, through" Active Record associations (for the users and ingredients) and one "has_many / belongs_to" (for the users / recipes). I needed join tables (user_ingredient & recipe_ingredient) for both the "has_many, through" associations.

I think what helped me the most was drawing out the relationships with Gliffy.

![](https://lh4.googleusercontent.com/yNT_RLh5kUNuVPC-aoraQDnDOiFDZNO2W5LeSzz-R06ijQHBFHspPbMw_RKmg8RvXKGtQI1rF9z-mBY=w2880-h1506)

The second part I struggled with was the actual recipe finder feature that tells users what recipes they can make with the ingredients they have. Basically what I ended up doing was looping through a hash containing each recipe with a key of the recipe's id, and a value of all that recipe's ingredient's ids. I then found the intersection (the recipe's ingredient's ids that match the user's ingredient's ids) of the current user's ingredients, and the recipe's ingredients.

```
user_recipe_ids_intersection = user_ingredients_ids & ingredient_ids
```

If the size of ingredients in the user_recipe_ids_intersection was equal to the size of the user's ingredients then they can make that recipe and I pushed that recipe to a hash.

```
if ingredient_ids.size == user_recipe_ids_intersection.size && ingredient_ids.size >= 1
          @you_can_make["#{recipe_id}"] = Recipe.find_by(id: recipe_id) #adds recipe to hash with recipe id as key
```

I then did the same thing, but i subtracted 1 and 2 from the user's ingredients size, to see which recipes they could make if they had an additional 1 or 2 missing ingredients and added those ingredients to a hash with a key of the recipe_id.

```
elsif ingredient_ids.size - 2 == user_recipe_ids_intersection.size || ingredient_ids.size - 1 == user_recipe_ids_intersection.size
          if ingredient_ids.size >= 1 && user_recipe_ids_intersection.size >= 1 then ingredient_ids_needed = ingredient_ids - user_recipe_ids_intersection | user_recipe_ids_intersection - ingredient_ids
            ingredient_ids_needed.each do |id| #add ingredient ids to array with key of recipe id
              if @you_can_almost_make["#{recipe_id}"]
                @you_can_almost_make["#{recipe_id}"] << Ingredient.find_by(id: id)
              else
                @you_can_almost_make["#{recipe_id}"] = Array.new
                @you_can_almost_make["#{recipe_id}"] << Ingredient.find_by(id: id)
```

This left me with two populated hashes stored in @you_can_make, and @you_can_almost_make which I looped through and displayed on the "What Can I Make?" .erb view page. I'll probably try to refactor and clean this section up a little bit, but for now it's working :).

Overall though, I feel like I learned a ton from this project and I'm super happy with how far I've come in just a few months!




