# os-learn

OS learning ...

## Usage

development:

```sh
# git clone  https://github.com/alex-shpak/hugo-book themes/book
git submodule add https://github.com/alex-shpak/hugo-book themes/book

hugo server --minify --theme book
```

deploy:

```sh
./deploy.sh
```

## Menu

By default, the [hugo-book](https://github.com/alex-shpak/hugo-book) theme will render pages from the `content/docs` section as a menu in a tree structure.
You can set `title` and `weight` in the front matter of pages to adjust the order and titles in the menu.