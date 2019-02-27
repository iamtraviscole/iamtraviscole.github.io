---
layout: post
title:      "Varyscary - Final project"
date:       2019-02-27 23:16:17 +0000
permalink:  varyscary_-_final_project
---


Varyscary is a single page web app built with React, Redux, and Rails that allows users to create custom monsters.

I'll briefly walk through and explain a few of the app's features starting with the monster creation page. The create page allows users to select from a bunch of pre-designed individual features to combine into a complete monster. Users can select different bodies, faces, eyes, headwear, etc. Each individual feature is just a react component that contains an SVG.

```
export const BodyCircle = (props) => {
  return (
    <svg xmlns={XMLNS} viewBox='-115.5 -115.5 375 375'>
      <circle fill={props.fillColor}
        stroke={props.strokeColor}
        strokeDasharray={props.strokeDasharray}
        strokeWidth={props.strokeWidth}
        cx='72' cy='72' r='72' />
    </svg>
  )
}
```

When a user selects a feature, I store the feature's component name in a Redux store. I use this redux store to display a preview of the monster, and, ultimately, save the monster's feature component names to the database.

```
monsterName: '',
monsterTags: [],
monsterFeatures: {
	body: {
		type: 'BodyCircle',
		hoverType: null,
		fillColor: '#000000'
	},
	face: {
		type: null,
		hoverType: null,
		fillColor: '#ffffff'
	},
	headwear: {
		type: null,
		hoverType: null,
		fillColor: '#000000'
	},
	...
}
```

There is also the option to create a randomized monster instead of selecting each feature. For the randomize function, I import all of the feature component names and make arrays for each category (bodies, faces, eyes, etc.) that contain these names as strings. When the randomize button is pressed I dispatch an action that selects a random index from each array and pushes the feature name at that index to the Redux store.

```
export const randomizeMonster = () => {
  const getRandomInt = max => Math.floor(Math.random() * max)
  const color1 = randomize.colors[getRandomInt(randomize.colors.length)]
  const color2 = randomize.colors[getRandomInt(randomize.colors.length)]
  return {
    type: actionTypes.RANDOMIZE_MONSTER,
    bodyType: randomize.bodies[getRandomInt(randomize.bodies.length)],
    bodyColor: color1,
    faceType: randomize.faces[getRandomInt(randomize.faces.length)],
    headwearType: randomize.headwear[getRandomInt(randomize.headwear.length)],
    headwearColor: color2,
	...
	}
}
```

Next up is the explore page. The explore page allows users to browse monsters. Users can search by tags and monster names as well as sort based on creation date and popularity.  The explore page initially loads 50 monsters, and an additional 50 monsters each time a 'Load more' button is pressed so that there isn't a long initial page load time. I achieved this using a limit and offset on the back end which I pass as params.

```
case params[:sort_by]
	when 'newest'
		@monsters = Monster.monsters_since(params[:since])
		.order(created_at: :desc).limit(params[:limit])
		.offset(params[:offset]).includes(:user, :liked_by, :tags)
		@monsters = sort_monsters(@monsters, params)
	when 'oldest'
		@monsters = Monster.monsters_since(params[:since])
		.order(created_at: :asc).limit(params[:limit])
		.offset(params[:offset]).includes(:user, :liked_by, :tags)
		@monsters = sort_monsters(@monsters, params)
		...
```

I also append the selected sort method and/or search term to the explore page's URL so that users can bookmark and link directly to a pre-sorted or filtered monster list.

E.g.
> /monsters?sort_by=newest&search=green


The back end is a pretty standard api-only rails app generated using the --api flag.

```
rails new my_api --api
```

For authentication, I'm using a JSON Web Token implemented via the Knock ruby gem. If the user's credentials are correct then I return a JSON Web Token via my user_token_controller that I store in the browser's local storage. I then require this JWT for authentication every time a user tries to access a protected route.




