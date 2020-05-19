+++
template = "page.html"
+++

## Hello, this is a simple dev blog.

This is a simple dev blog that I made [for my website](https://bennetthardwick.com).
It's quite easy to install, just go to the [project page](https://github.com/bennetthardwick/simple-dev-blog-zola-starter) and follow the prompts.

This template does some fancy stuff like [pre-rendering](https://developer.mozilla.org/en-US/docs/Web/HTML/Preloading_content) blog posts and nav links,
pre-fetching your profile image and adding a bunch of common meta tags.

### How to get started

To create a new Zola site, first download the CLI and install it on your system.
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

4. That's it! Now build your site by running the following command, and navigate to `127.0.0.1:111`:

   ```sh
   zola serve
   ```

You should now have a speedy simple dev blog up and running, have fun!

### Deployment

[Netlify](https://www.netlify.com/) is a great way to deploy your website for free to a custom domain and it's what I use [personally](https://bennetthardwick.com).
To deploy to Netlify, refer to Zola's [Netlify deployment instructions](https://www.getzola.org/documentation/deployment/netlify/).
