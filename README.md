![preview image](https://i.imgur.com/IWoJtkF.png)

# simple-dev-blog-zola-starter

A simple dev-blog theme for Zola. It uses no JavaScript, prerenders links between navigation, blog posts and tags and adds common tags for SEO.

You can view it live [here](https://simple-dev-blog-zola-starter.netlify.app/).

## How to get started

To create a new Zola site, first download the CLI and install it on your system. This theme requires Zola version 0.14 or greater.


You can find installation instructions [on the Zola website](https://www.getzola.org/documentation/getting-started/installation/).

1. After you've installed the Zola CLI, run the following command to create a new site:

   ```sh
   zola init my_amazing_site
   cd my_amazing_site
   ```

2. After you've created the site, install the "Simple Dev Blog" theme like so:

   ```sh
   git clone --depth=1 \
     https://github.com/bennetthardwick/simple-dev-blog-zola-starter \
     themes/simple-dev-blog
   ```

3. Now in your `config.toml` file, choose the theme by setting `theme = "simple-dev-blog"`.

4. This theme uses the `tags` taxonomy, in your `config.toml` file set `taxonomies = [ { name = "tags" } ]`

5. This theme uses the `authors` taxonomy, in your `config.toml` file set `taxonomies = [ { name = "authors" } ]`

6. Copy across the default content from the theme by running `cp themes/simple-dev-blog/content/* ./content -r`

7. That's it! Now build your site by running the following command, and navigate to `127.0.0.1:111`:

   ```sh
   zola serve
   ```

You should now have a speedy simple dev blog up and running, have fun!

## Customisation

Look at the `config.toml` and `theme.toml` in this repo for an idea, here's a list of all the options:

### Global

The following options should be under the `[extra]` in `config.toml`

- `accent_light` - a lighter shade of your site's accent color
- `accent` - your site's accent color
- `blog_path` - the path to your blog (defaults to `blog`)
- `default_og_image` - the path default og:image for your page
- `footer_about` - the content for your footer in markdown
- `icon` - the path to the icon for your site in the content folder
  - E.g to add the file `icon.png` you should put it in `content/icon.png`
- `nav` - see `theme.toml`, the navigation links for your site
- `not_found_message` - the content for your 404 page in markdown
- `profile_large` - the path to a larger vertical version of your profile picture in the content folder
- `profile_small` - the path to a small version of your profile picture in the content folder
- `summary_truncate_length` - the length of a post's summary

### Page

The following options should be under the `[extra]` section of each page

- `thumbnail` - the path to your og:image for that page
