---
title: "Setting up this blog with Hugo and Hyde-hyde"
date: 2018-03-25T14:47:00-04:00
draft: false
tags: [ "hugo" , "git", "hyde" , "hyde-hyde"]
categories: [ "programming" ]
---

## **Hugo**

First, I follow the [quick start guide](https://gohugo.io/getting-started/quick-start/) to install and set up a hugo server locally.

## **Adding content**

Use `hugo new posts/my-first-post.md` or simply add a markdown file under the `content\posts` folder. Make sure the markdown files include the [front matter](https://gohugo.io/content-management/front-matter/), in a regular post page, it looks something like:

```
title: "Setting up this blog"
date: 2018-03-25T14:47:00-04:00
draft: false
tags: [ "hugo" , "git" ]
categories: [ "programming" ]
```

For a static page like "About" or "Contact", I only use title and draft.

## **Customize theme**

### **Picking theme**

There are hundreds of themes available on the [hugo site](https://themes.gohugo.io/). At first, I picked [hyde](https://themes.gohugo.io/hyde/) to start with, I like the simplistic two-column design. Hyde is originally a [Jekyll theme](https://github.com/poole/hyde), it was ported by the creator of Hugo [@spf13](https://github.com/spf13).  As one of the most basic and popular theme, I think it would be a good theme to begin with. I tried to add new features like social media logos, setting up Posts as list page, soon I realized it would be easier to use a theme that support these features out of the box. I found different variations of Hyde-based themes like Hyde-X, Hyde-Y, Hyde-Hyde. I picked [Hyde-hyde](https://themes.gohugo.io/hyde-hyde) at the end.

### **Changing config.toml**

First step is to modify config.toml to personalize your site. Set theme = "hyde-hyde" to use selected theme. Change the title, description, update social accounts etc. Set up the menu menu in the side bar:

```toml
## Main Menu
[[menu.main]]
    name = "Posts"
    weight = 100
    identifier = "posts"
    url = "/posts/"
[[menu.main]]
    name = "About"
    identifier = "about"
    weight = 200
    url = "/about/"
[[menu.main]]
    name = "Contact"
    identifier = "contact"
    weight = 300
    url = "/contact/"
```

### **Background color**

There are 8 default color scheme out of the box in hyde:

![Hyde theme classes](https://camo.githubusercontent.com/31722ca812424795bb0c9a6ea99ccdd5fa171c24/68747470733a2f2f662e636c6f75642e6769746875622e636f6d2f6173736574732f39383638312f313831373034342f65356230656330362d366636382d313165332d383364372d6163643139343237393761312e706e67)

If you like them, just change the `themeColor` parameter as in the [hyde docs](https://github.com/spf13/hyde#options). I don't like any of them, so I created my own theme.

1. Go to `\theme\hyde-hyde\static\css` folder

2. Copy  `hyde.css` to `\static\css` 
   Note:  `\static` should be empty. Originally I modify it from the theme folder directly, then I realized it is not a good practice, I would like to keep everything in the theme folder as is. Hugo's [look up order](https://gohugo.io/templates/lookup-order/) will look into site folder first before going into theme.

3. Near the bottom of the `hyde.css`, I copied one of theme and the added

   ```css
   /* Dark blue & dark red*/
   .theme-base-0g .sidebar {
     background-color: #212F3D;
   }
   .theme-base-0g .content a,
   .theme-base-0g .related-posts li a:hover {
     color: #943126;
   }
   ```

   You can also change the accent color on the post title. I looked up the HTML color codes from this [site](https://htmlcolorcodes.com/).

4. Back to `config.toml`, in the `[params]` section, set

   `themeColor = "theme-base-0g"`

   Save all files and refresh the page, you should see the changes instantly.

###  **Font**

We can also change the font size and family from `custom.css`. 

I modified the font-family and font-size to:

```css
html,
body
{
    font-family: "Century Gothic", CenturyGothic, AppleGothic, sans-serif;
    font-size: 16px;
    font-weight: 300;
    line-height: 1.5;
}
```

I used [cssfontstack.com](https://www.cssfontstack.com) to find a collection of web safe CSS font.

###  **Favicon**

Favicon is the website icon on the browser tab and bookmark. 

I used [realfavicongenerator](https://realfavicongenerator.net) to generate a favicon and put the extracted files in `/static/`. More info [here](http://www.enthuseandinspire.co.uk/blog/favicon/).

Next, I tried to find the corresponding place hugo specifying the favicon location. It is in the `/theme/hyde-hyde/layouts/partials/header.html`

Near the bottom, I added:

```html
    {{ "<!-- Icons -->" | safeHTML }}
    <link rel="apple-touch-icon-precomposed" sizes="144x144" href="/apple-touch-icon-144-precomposed.png">
    <link rel="shortcut icon" href="/favicon.png">
    <link rel="apple-touch-icon" sizes="76x76" href="/apple-touch-icon.png">
    <link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png">
    <link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png">
```



### **Code syntax highlighting**

I noticed that the hyde-hyde theme has its own syntax.css that is not in the hyde theme. 

By theme default, the text is in white with black background color. There were some issues for code block, I [fixed](https://stackoverflow.com/questions/38821339/hugo-pygments-how-to-change-highlighting-theme) it by adding these on the config.toml. A subset of list of Pygment style is available [here](https://help.farbox.com/pygments.html).

```toml
PygmentsCodeFences = true
PygmentsStyle = "monokai"
```

Update (04-18-2018): I switched to another highlighting style from `config.toml` `[param]` by adding `highlightjsstyle = "dark"`.

## **Hosting on Github**

Once everything look okay locally, the last step is to host it on Github pages. I follow the [guide](https://gohugo.io/hosting-and-deployment/hosting-on-github) from Hugo.

Basic steps:

1. Create 2 repo: `darwinleung.github.io` and `personal-blog`
2. Add my local blog folder to the personal-blog github repo, more info [here](https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/)
3. Remove the public folder inside the blog folder: `rm -rf public`
4. `git submodule add -b master https://github.com/darwinleung/darwinleung.github.io.git public`
5. Run `hugo` inside blog folder to generate the `public` folder
6. Run [deploy.sh](https://gohugo.io/hosting-and-deployment/hosting-on-github/#put-it-into-a-script) 


## **Workflow**

It is simple to add new posts or make any updates to my blog site.

1. Make changes within the local blog folder, you can preview the changes locally by `hugo server -D`
2. Push changes to github repo `personal-blog`
   1. git add *
   2. git commit -m "comment"
   3. git push
3. Run deploy.sh




