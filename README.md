![Henlo Starter](https://repository-images.githubusercontent.com/270961687/4085d990-9083-451d-b39b-5316579adf09)

# Gatsby Starter Henlo (v1.0.0)

[![Netlify Status](https://api.netlify.com/api/v1/badges/43532afb-3488-432b-8185-a745645a90d8/deploy-status)](https://app.netlify.com/sites/henlo/deploys)

[Official Website / Demo](http://henlo.cleancommit.io)

Gatsby Starter Henlo is the most advanced Netlify CMS starter for Gatsby.js. We built it with Page Builder setup in mind. All pages are created out of programmable blocks, aiming to provide the best DX & admin UX possible.

This repo contains an example website that is built with [Gatsby](https://www.gatsbyjs.com/docs/), and [Netlify CMS](https://www.netlifycms.org).

It follows the [JAMstack architecture](https://jamstack.org) by using Git as a single source of truth, and [Netlify](https://www.netlify.com) for continuous deployment, and CDN distribution.

## Features

- Battle-tested starting point for small & large web projects
- Example pages, collections, CMS configuration with Netlify CMS & hooks
- Easy Netlify CMS configuration using [Manual Initialization](https://www.netlifycms.org/docs/beta-features/#manual-initialization)
- Form Builder that enables Admins to create multiple forms with ease & Netlify Forms integration.
- TailwindCSS support with PostCSS processing & PurgeCSS
- Support for Gatsby API functions
- Sitemaps using `gatsby-plugin-sitemap`
- `gatsby-plugin-image`
- Netlify deploy configuration
- Complete SEO configuration with graphql fragment and reusable components
- Perfect score on Lighthouse for SEO, Accessibility and Performance
- Readme template for custom projects
- ..and more

## Prerequisites

- Node 16.13
- [Gatsby CLI](https://www.gatsbyjs.org/docs/)
- [Netlify CLI](https://github.com/netlify/cli)

## Getting Started (Recommended)

Netlify CMS can run in any frontend web environment, but the quickest way to try it out is by running it on a pre-configured starter site with Netlify. Use the button below to build and deploy your own copy of the repository:

<a href="https://app.netlify.com/start/deploy?repository=https://github.com/clean-commit/gatsby-starter-henlo"><img src="https://www.netlify.com/img/deploy/button.svg" alt="Deploy to Netlify"></a>

After clicking that button, you’ll authenticate with GitHub and choose a repository name. Netlify will then automatically create a repository in your GitHub account with a copy of the files from the template. Next, it will build and deploy the new site on Netlify, bringing you to the site dashboard when the build is complete. Next, you’ll need to set up Netlify’s Identity service to authorize users to log in to the CMS.

## Getting Started (Without Netlify)

```
$ gatsby new [SITE_DIRECTORY_NAME] https://github.com/clean-commit/gatsby-starter-henlo
$ cd [SITE_DIRECTORY_NAME]
$ yarn start
```

### Access Locally

Pulldown a local copy of the Github repository Netlify created for you, with the name you specified in the previous step

```
$ git clone https://github.com/[GITHUB_USERNAME]/[REPO_NAME].git
$ cd [REPO_NAME]
$ yarn && yarn start
```

To test the CMS locally, you'll need run a production build of the site & [run local instance of Netlify CMS](https://www.netlifycms.org/docs/beta-features/#working-with-a-local-git-repository)

```
$ yarn start
$ npx netlify-cms-proxy-server
```

Your admin configuration will be available at http://localhost:8000/admin

### Deployment

We've added additional commands for quick deployments with Netlify CLI. To deploy the website to netlify cms simply run.

```
$ yarn deploy:prod
```

The website will build locally and then deploy to production.

### Folder structure

```
├── cms                      # Netlify CMS configuration
│   ├── blocks
│   ├── collections
│   ├── fields
│   ├── previews
│   └── cms.js
├── content                  # Your content lives here
│   ├── authors
│   ├── blog
│   ├── forms
│   ├── pages
│   └── dont-remove.md
├── public
├── src
│   ├── api                  # Gatsby functions should be placed here
│   ├── blocks               # Blocks that create sections
│   ├── components           # Reusable components
│   ├── hooks                # Hooks used in the project
│   ├── lib                  # misc
│   ├── pages
│   ├── resolvers
│   │   ├── Image.js         # Required for previews
│   │   └── Link.js          # Resolves links to gatsby and outside links
│   ├── settings             # Place for theme settings
│   ├── styles
│   └── templates            # Templates used to render page types
│       ├── page-builder.js
│       └── post.js
├── static
├── _headers
├── gatsby-config.js         # Config files for gatsby
├── gatsby-node.js           # Page generation setup
└── tailwind.config.js       # Tailwind configuration
```

### Setting up the CMS

Follow the [Netlify CMS Quick Start Guide](https://www.netlifycms.org/docs/quick-start/#authentication) to set up authentication, and hosting.

CMS configuration was placed within `cms` directory in the root of the project. This allows us to work efficiently on fields and collections without mixing CMS config with Gatsby code.

Henlo uses Manual Initialization to take advantage of componetized approach to managing configuration for Netlify CMS. Thanks to that you don't have to control the CMS from centralized YAML file.

To ensure best experience we use 2 custom widgets that are maintained by us -> [ID Widget](https://github.com/clean-commit/netlify-cms-widget-id) that provides unmutable IDs for content items and [Permalink Widget](https://github.com/clean-commit/netlify-cms-widget-permalink) that enables you to create custom permalinks with ease.

```javascript
import CMS from 'netlify-cms-app';
import { Widget as UuidWidget } from 'netlify-cms-widget-id';
import { Widget as PermalinkWidget } from 'netlify-cms-widget-permalink';

import pages from './collections/pages';
import posts from './collections/posts';
import authors from './collections/authors';
import settings from './collections/settings';
import PagePreview from './previews/Page'; // Preview for all PageBuilder based pages

window.CMS_MANUAL_INIT = true;

const config = {
  config: {
    load_config_file: false,
    display_url: process.env.GATSBY_APP_URL, // Enables urls based on env variable
    local_backend: true,
    backend: {
      name: 'git-gateway',
    },
    slug: {
      encoding: 'ascii',
      clean_accents: true,
    },
    media_folder: '/static/img',
    public_folder: '/img',
    collections: [pages, posts, authors, settings],
  },
};

CMS.registerPreviewStyle('../commons.css');
CMS.registerPreviewTemplate('pages', PagePreview);

CMS.registerWidget(UuidWidget);
CMS.registerWidget(PermalinkWidget);

CMS.init(config);
```

#### Adding blocks

Blocks are defined in `cms/blocks/index.js` file. We're leveraging use of exported functions and variables from other fields to avoid repetition within the code.

This is extremely important due to the way GraphQL works with Markdown based files. Each field will have to be present on all page queries -> we can't differetiate between different sections.

That's why it's important to reuse names of fields, hence usage of imports.

```javascript
import { Buttons, Title, Content, VariantField, ImageField } from '../fields';

const Config = {
  label: 'Blocks',
  name: 'blocks',
  widget: 'list',
  types: [
    {
      label: 'Hero',
      name: 'hero',
      widget: 'object',
      fields: [
        Title,
        Content,
        Buttons,
        VariantField('default', ['default', 'centered', 'full']),
      ],
    },
...
```

To add new block you have tp add new `Type` to `cms/blocks/index.js` file, and modify `Blocks` fragment located in `src/components/PageBuilder.js`

As you can see below we're adding all fields used by all sections. This causes issues with Gatsby's [Schema Inference](https://www.gatsbyjs.com/docs/schema-inference/). Gatsby needs an example of the field to know what type of field it is.

That's why we rely on `dont-remove.md` this file contains all possible fields used in the starter, so you're never encounter a problem with types inference!

```
export const query = graphql`
  fragment Blocks on MarkdownRemarkFrontmatter {
    blocks {
      type
      title
      content
      columns {
        title
        content
      }
      photo {
        image {
          childImageSharp {
            gatsbyImageData(
              width: 800
              quality: 72
              placeholder: DOMINANT_COLOR
              formats: [AUTO, WEBP, AVIF]
            )
          }
        }
        alt
      }
      variant
      buttons {
        button {
          content
          url
          variant
        }
      }
    }
  }
`
```

### Adding Favicons

Favicons can be generated using this [Favicon Generator](https://www.favicon-generator.org/) After generating the icons, drop the contents of downloaded file into `static/img/favicons` directory

### Preloading fonts

Since 0.4.0 Henlo supports [`gatsby-plugin-preload-fonts`](https://www.gatsbyjs.com/plugins/gatsby-plugin-preload-fonts/) plugin out of the box. To create the preload cache you need to start development server and then run `preload-fonts` command. This will generate the `font-preload-cache.json` file in the root of your project. When your projects builds fonts will be added automatically to head of the document.

```
yarn start
yarn preload-fonts
```

## Browser support

Gatsby tends to add a lot of polyfills to support older browser versions. In package.json file you can adjust which sites your project should support. As default Henlo will use `defaults` setting. If you want to learn more about the browser support visit official [Gatsby How-To Guide on this subject](https://www.gatsbyjs.com/docs/how-to/custom-configuration/browser-support/)

## PurgeCSS

This plugin uses [gatsby-plugin-purgecss](https://www.gatsbyjs.org/packages/gatsby-plugin-purgecss/) and [Tailwind](https://tailwindcss.com/). The builds are usually ~780K but reduced 98% by purgecss. Normally our websites won't cross 30Kb of CSS.

Since Henlo v0.5.0 we use PurgeCSS v4, the default configuration is already included, to learn more [check purgecss docs](https://purgecss.com/configuration.html) or [check gatsby-plugin-purgecss documentation](https://www.gatsbyjs.com/plugins/gatsby-plugin-purgecss/)

```javascript
{
  resolve: `gatsby-plugin-purgecss`,
  options: {
    printRejected: true,
    develop: false,
    tailwind: true,
    purgeCSSOptions: {
      safelist: {
        standard: [],
        deep: [],
      },
    },
  },
},
```

# CONTRIBUTING

Contributions are always welcome, no matter how large or small. Before contributing,
please read the [code of conduct](CODE_OF_CONDUCT.md).

# Additional guides

Here's a list of helpful articles that will help you with your first steps using Henlo!

- [Efficient Netlify CMS config with Manual Initialization](https://mrkaluzny.com/blog/dry-netlify-cms-config-with-manual-initialization?utm_source=GitHub&utm_medium=henlo-gatsby)
- [How to optimize SEO with Gatsby & Netlify](https://mrkaluzny.com/blog/how-to-optimize-seo-with-gatsby-netlify?utm_source=GitHub&utm_medium=henlo-gatsby)
- [https://mrkaluzny.com/blog/full-text-search-with-gatsby-and-netlify-cms/](https://mrkaluzny.com/blog/full-text-search-with-gatsby-and-netlify-cms?utm_source=GitHub&utm_medium=henlo-gatsby)
