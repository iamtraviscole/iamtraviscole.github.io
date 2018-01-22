---
layout: post
title:      "Schemer Rails App - Now with JSON and jQuery!"
date:       2018-01-22 02:42:12 +0000
permalink:  schemer_rails_app_-_now_with_json_and_jquery
---


The goal of this project was to add some JSON and jQuery to the front end of our previous Rails project that loaded content without a page refresh. 

We had to render at least one index page via an AJAX request. To accomplish this, I rendered my index.html.erb partial as JSON in the controller,

```
partial = render_to_string partial: 'color_schemes/index', locals: { user: @user, color_schemes: @color_schemes }
respond_to do |format|
    format.html { render :index }
    format.json { render json: {index_partial: partial} }
end
```

then made a JSON request to that controller action and appended the partial's html to the DOM when the "Your Color Schemes" link was clicked:

```
function loadColorSchemes(clickData) {
  $.ajax({
    url: clickData.href,
    dataType: 'json',
    success: function(resp) {
      let response = new Response(resp)
      $('#content').html(response.yourColorSchemes())
      }
  })
}
```

I followed the same pattern to render a random color scheme when the "Random" link was clicked. I got a random scheme with the following code, 

```
color_scheme_ids = ColorScheme.ids
random_id = color_scheme_ids.sample
@random_scheme =  ColorScheme.find(random_id)
```

and then sent that @random_scheme as a local to a random.html.erb partial, which I rendered as JSON and appended to the DOM.

Another requirement was to create a resource (a color scheme in my case) using an AJAX post request and then append that resource to the page without a page refresh. My POST request looked like this:

```
$.ajax({
    method: "POST",
    url: $('#newcolorschemesubmit')[0].dataset.url,
    data: colorScheme,
    dataType: 'json',
    success: function(resp) {
      let response = new Response(resp)
      $('#content').html(response.yourColorSchemes())
      $(".alertcontainer").html(`<div class='alert alert-success'>New color scheme created!</div>`)
      }
    })
```

I used jQuery's .serialize() method to serialize the form data:

```
const colorScheme = $("form#new_color_scheme").serialize()
```

The URL came from a data-attribute I set on the New Color Scheme form's submit button:

```
<input type="submit" ... id="newcolorschemesubmit" data-url="/users/1/color_schemes">
```

The Response prototype/class you see in the ajax success functions ( let response = new Response(resp) ) just takes the ajax responses and concatenates some HTML to them.  It looks like this: 

```
class Response {
  constructor(response) {
    this.response = response
  }

  yourColorSchemes() {
    return `<div class='container'><h1>Your Color Schemes</h1>${this.response.index_partial}</div>`
  }

  randomColorScheme() {
    return `<div class='container'>${this.response.random_partial}</div>`
  }
}
```






