---
layout: post
title:      "Nested Layouts in Sinatra with ERB"
date:       2018-03-14 21:47:59 +0000
permalink:  nested_layouts_in_sinatra_with_erb
---


<img src="https://s7d1.scene7.com/is/image/BedBathandBeyond/10356416309656p?$478$" alt="Nested bowls" style="width: 200px;"/>

When working on my Sinatra Porfotilio project [Snippet Manager](http://https://github.com/allysonw/snippet-manager), I found myself repeating the same chunk of code across several of my views. My application manages code snippets, and one of the features is the ability to add a label to each code snippet. When displaying all of their snippets, the user sees a label navigation pane on the left of the page that lets them filter their snippets by label. When I went to update the styling for the label navigation pane, I realized I'd need to update the same code in 3 places. I realized the whole setup violated DRY, and that there had to be a better way.

I had already learned to use a _layout.erb_ file to contain basic HTML elements that I wanted to appear on every page of my application, and was using this technique to render my header, navbar, and footer. The body of the rest of my views was then yielded to the _layout.erb_ template with the  `yield` keyword. This saved me from having to code, for example, my persistent navbar in every view. I wondered if there was some way I could similarly abstract out the code for my labels navigator and then `yield` my page-specific layouts to that common template, all within my larger template contained in _layout.erb_. I knew that the filename _layout.erb_ was a convention recognized by Sinatra as a default main template view, but I wasn't sure how to create other templates and what was possible in terms of nesting them.

### Templates
First, I had to learn a little more about using templates in Sinatra. I was using ERB and was familiar with how to pass in template content to a view:
`erb :index` loads the HTML defined in my _views/index.erb_ file (you can tell Sinatra where your views are, or it by default assumes they are in a folder called _views_).

What I didn't know was that you can also pass additional arguments to `erb`, the rendering method for ERB. The second argument is called the options hash, and it specifies which layout to embed your rendered view in. For example,
```Ruby
erb :snippets, :layout => :main_layout
```
will embed the _snippets.erb_ view within a view called _main\_layout.erb_. (Basically, what is happening when you don't include the options hash is something like `erb :snippets, :layout => :layout` which just renders your view embedded in _layout.erb_ - the default behavior).

So I figured I could create a layout containing just my labels navigator, place a `yield` statement in that layout to load in my other views within it, and let Sinatra know to do so using something like
 ```Ruby
 erb :snippets, :layout => :labels_layout
```
 Now I just had to find a way to also embed all of that into my over-arching site layout.

### Nested Templates
It turns out that you can also pass blocks to rendering methods such as `erb`.
```Ruby
erb :snippets, :layout => :labels_layout
```
is basically equivalent to
```Ruby
erb :labels_layout, :layout => :false do
  erb :snippets
end
```
Don't ask me why the syntax is this way, but I'm sure there's some good Ruby reason for it. This block will, as above, render the _snippets_ view embedded in the _labels\_layout_ view.

In order to actually nest layouts, one needs only to nest additional blocks. So to solve my problem, I could do the following:
```Ruby
erb :main_layout, :layout => :false do
  erb :labels_layout do
    erb :snippets
  end
end
```
This has the desired effect. I nest the view containing my snippets data within the view containing my labels navigator (which I need to use across several pages), and then nest that within my main site view (which should show on on _all_ of my pages). Now, a user can view various pages and see the labels navigator on the left and the navbar at the top. I only need to write the labels navigator code in one file, _labels\_layout.erb_, and I keep all of my main site HTML in _main_layout.erb_. I can change the styling of my site-wide elements and labels navigator in just one place each and I have solved my DRY problem.

#### Shorthand
The above can actually be written with fewer lines of code as:
```Ruby
erb :labels_layout, :layout => :main-layout do
    erb :snippets
end
```
I guess the way one can read this is that you render your "middle" layout within your "outer" layout, then pass in the "innermost" layout as a block. I think it's slightly more confusing this way, but it is more concise.

### Yield
Now all that's left to do is use `yield` appropriately in my main layout and labels navigator layout. I accomplished this as follows in my main layout view:
```HTML
<!-- main_layout.erb -->
<body>
  <div>
    <div>
      <!-- navbar code -->
    </div>

    <%= yield %>

  </div>
</body>
```
My labels layout code loads at the `yield` in this file. Then, in my labels layout view:
```HTML
<!-- labels_layout.erb -->
<div>
  <h2>Labels</h2>
    <!-- code to display the various labels -->
  </div>

  <div>
    <%= yield %>
  </div>
</div>
```
All page content for my snippets pages then loads within the `<div>` that calls yield. Problem solved!
