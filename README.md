# Loconotion
**Loconotion** is a Python script that parses a [Notion.so](https://notion.so) public page (alongside with all of its subpages) and generates a static site out of it.

## But Why?
[Notion](https://notion.so) is a web app where you can create your own workspace / perosnal wiki out of content blocks. It feels good to use, and the results look very pretty - the developers did a great job. Given that it also offers the possibility of making a page (and its sub-page) public on the web, several people choose to use Notion to manage their personal blog, portfolio, or some kind of simple website. Sadly Notion does not support custom domains: your public pages are stuck in the `notion.so` domain, under long computer generated URLs.

Some services like Super, HostingPotion, HostNotion and Fruition try to work around this issue by relying on a [clever hack](https://gist.github.com/mayneyao/b9fefc9625b76f70488e5d8c2a99315d) using CloudFlare workers. This solution, however, has some disadvantages:
- **Not free** - Super, HostingPotion and HostNotion all take a monthly fee since they manage all the "hacky bits" for you; Fruition is open-source but any domain with a decent amount of daily visit will soon clash against CloudFlare's free tier limitations, and force you to upgrade to the 5$ or more plan (plus you need to setup Cloudflare yourself)
- **Slow-ish** - As the page is still hosted on Notion, it comes bundled with all their analytics, editing / collaboration javascript, vendors css, and more bloat which causes the page to load at speeds that are not exactly appropriate for a simple blog / website. Running [this](https://www.notion.so/The-perfect-It-s-Always-Sunny-in-Philadelphia-episode-d08aaec2b24946408e8be0e9f2ae857e) example page on Google's [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/) scores a measly **24 - 66** on mobile / desktop.
- **Ugly URLs** - While the services above enable the use of custom domains, the URLs for individual pages are stuck with the long, ugly, original Notion URL (apart from Fruition - they got custom URLs figured out, altough you will always see the original URL flashing for an instant when the page is loaded).
- **Notion Free Account Limitations** - Recently Notion introduced a change to its pricing model where public pages can't be set to be indexed by search engines on a free account (but they also removed the blocks count limitations, which is a good trade-off if you ask me)

Loconotion approaches this a bit differently. It lets Notion render the page, then scrapes it and saves a static version of the page to disk. This offers the following benefits:
- Strips out all the unnecessary bloat, like Notion's analytics, vendors scripts / styles, and javascript left in to enable collaboration.
- Caches all images / assets / fonts (hashing filenames), while keeping links intact.
- Cleans up the pages urls, letting you use custom slugs if desired
- Full meta tags controls, for the whole site or individual pages
- Granular custom Goggle Fonts control on headings, navbar, body and code blocks 
- Lets you inject any custom style or script, from custom analytics or real-time chat support to hidden crypto miners (please don't do that)
- Outputs static files ready to be deployed on Netlify, GitHub Pages, Vercel, your Raspberry PI, that cheap second-hand Thinkpad you're using as a random server - you name it.

The result? A faster, self-contained version of the page that keeps all of Notion's nice layouts and eye candies. For comparison, the same example page parsed with Loconotion and deployed on Netflify's free tier achieves a PageSpeed Insight score of **96 - 100**!

Bear in mind that as we are effectively parsing a static version of the page, there are some limitations compared to Notion's live public pages:
- All pages will open in their own page and not modals (depending on how you look at it this could be a plus)
- Databases will be presented in their initial view - for example, no switching views from table to gallery and such
- All editing features will be disabled - no ticking checkboxes or dragging kanban boards cards around. Usually not an issue since a public page to serve as a website would have changes locked.
- Dynamic elements won't update automatically - for example, the calendar will not highlight the current date.

Everything else should be fine. Loconotion rebuilds the logic for toggle boxes and embeds so they still work; plus it defines some additional CSS rules to enable mobile responsiveness across the whole site (in some cases looking even better than Notion's defaults - wasn't exactly thought for mobile).

### But Notion already had an html export function?
It does, but I wasn't really happy with the styling - the pages looked a bit uglier than what they look like on a live Notion page. Plus, it doesn't support all the cool customization features outlined above!

## Installation & Requirements
`pip install -r requirements.txt`

This script uses [ChromeDriver](chromedriver.chromium.org) to automate the Google Chrome browser - therefore Google Chrome needs to be installed in order to work.

The script comes bundled with the default windows chromedriver executable. On Max / Linux, download the right distribution for you from https://chromedriver.chromium.org/downloads and place the executable in this folder. Alternatively, use the `--chromedriver` argument to specify its path at runtime

## Simple Usage
`python loconotion.py https://www.notion.so/The-perfect-It-s-Always-Sunny-in-Philadelphia-episode-d08aaec2b24946408e8be0e9f2ae857e`

In its simplest form, the script takes the URL of a public Notion.so page, and generates the site inside the `dist` folder, based on the page's title (the above example will generate the site inside `dist\The-perfect-It-s-Always-Sunny-in-Philadelphia\`).

## Advanced Usage
You can fully configure Loconotion to your needs by passing a [.toml](https://github.com/toml-lang/toml) configuration file to the script instead:
`python loconotion.py example\example_site.toml`

Here's what a full configuration would look like, alongside with explanations for each parameter.
```toml
## Loconotion Site Configuration File ##
# full .toml  configuration example file to showcase all of Loconotion's available settings
# check out https://github.com/toml-lang/toml for more info on the toml format

# name of the folder that the site will be generated in
name = "Notion Test Site"
# the notion.so page to being parsing from. This page will become the index.html
# of the generated site, and loconotation will parse all sub-pages present on the page.
page = "https://www.notion.so/A-Notion-Page-03c403f4fdc94cc1b315b9469a8950ef"

## Global Site Settings ##
# this [site] table defines override settings for the whole site
# later on we will see how to define settings for a single page
[site]
  ## Custom Meta Tags ##
  # defined as an array of tables (double square brackets)
  # each key in the table maps to an atttribute in the tag
  # the following adds the tag <meta name="title" content="Loconotion Test Site"/>
  [[site.meta]]
  name = "title"
  content = "Loconotion Test Site"
  [[site.meta]]
  name = "description"
  content = "A static site generated from a Notion.so page using Loconotion"

  ## Custom Fonts ##
  # you can specify the name of a google font to use on the site - use the font embed name
  # if in doubt select a style on fonts.google.com and navigate to the "embed" tag to check the name under CSS rules
  # the table keys controls the font of the following elements:
    # site: changes the font for the whole page (apart from code blocks) but the following settings override it
    # navbar: site breadcrumbs on the top-left of the page
    # title: page title (under the icon)
    # h1: heading blocks, and inline databases' titles
    # h2: sub-heading blocks
    # h3: sub-sub-heading blocks
    # body: non-heading text on the page
    # code: text inside code blocks
  [site.fonts]
  site = 'Lato'
  navbar = ''
  title = 'Montserrat'
  h1 = 'Montserrat'
  h2 = 'Montserrat'
  h3 = 'Montserrat'
  body = ''
  code = ''

  ## Custom Element Injection ##
  # defined as an array of tables [[site.inject]], followed by 'head' or 'body' to set where the injection point,
  # followed by name of the tag to inject. Each key in the table maps to an atttribute in the tag
  # the following injects <link href="favicon-16x16.png" rel="icon" sizes="16x16" type="image/png"/> in the <head>
  [[site.inject.head.link]]
  rel="icon" 
  sizes="16x16"
  type="image/png"
  href="/example/favicon-16x16.png"
  
  # the following injects <script src="custom-script.js" type="text/javascript"></script> in the <body>
  [[site.inject.body.script]]
  type="text/javascript"
  src="/example/custom-script.js"

## Individual Page Settings ##
# the [pages] table defines override settings for individual pages, by defining a sub-table named after the page url 
# (or part of the url, but careful into not use a string that appears in multiple page urls)
[pages]
  # the following settings will only apply to this page: https://www.notion.so/d2fa06f244e64f66880bb0491f58223d
  [pages.d2fa06f244e64f66880bb0491f58223d]
    ## custom slugs ##
    # inside page settings, you can change the url that page will map to with the 'slug' key
    # e.g. page "/d2fa06f244e64f66880bb0491f58223d" will now map to "/list"
    slug = "list"

    # change the description meta tag for this page only
    [[pages.d2fa06f244e64f66880bb0491f58223d.meta]] 
    name = "description"
    content = "A fullscreen list database page, now with a pretty slug"

    # change the title font for this page only
    [pages.d2fa06f244e64f66880bb0491f58223d.fonts]
    title = 'Nunito' 

  # for smaller sets of settings you can use inline notation
  # 2483a3a5c3fd445980c1adc8e550b552.slug = "gallery"
  # 2604ce45890645c79f67d92833083fee.slug = "table"
  # a28dba2e7a67448da52f2cd2c641407b.slug = "board"
```

On top of this, the script can take this optional arguments:
```
  --clean        Delete all previously cached files for the site before generating it
  -v, --verbose  Shows way more exciting facts in the output
  --single-page  Don't parse sub-pages
```

## Roadmap / Features wishlist
- [ ] Dark / custom themes
- [ ] Automated Netlify / GitHub pages / Vercel deployements
- [ ] Injectable custom HTML
- [ ] Html / css / js minification & images optimization
- [ ] GUI / sites manager, potentially as a paid add-on to rack up some $ - daddy needs money to get more second-hand thinkpads to use as random servers

## Who uses this?
If you used Loconotion to build a cool site, shoot me a mail! I'd love to feature it in some sort of showcase.

## Support
If you found this useful, and / or it saved you some money, consider using part of that saved money to buy me a coffee and mantain the balance in the universe.

<a href="https://www.buymeacoffee.com/leoncvlt" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-blue.png" alt="Buy Me A Coffee" style="height: 51px !important;width: 217px !important;" ></a>