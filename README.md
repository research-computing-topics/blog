[![Build Status](https://travis-ci.com/research-computing-topics/blog.svg?branch=master)](https://travis-ci.com/research-computing-topics/blog)

# Research Computing Topics


## Requirements for local testing

- Install of `Sphinx` and `ablog`
    - e.g. an Anaconda installation, followed by `pip install -r requirements.txt` should work


## Writing

Please place the source files in the right place.

## File structure

#### Blog


Place the `.rst` file containing the blog post with the `.. posts::` directive, e.g.

```
.. posts:: 2020-05-11
   :author: author_name
   :tags: tag1, tag2
    
```
in directory `blog/`. The blog post will automatically appear after a build.
Conveniently, one can also enter the `blog/` directory and run the command `ablog post` to create a file template
as the starting point to write a new blog.

Posts without the date are drafts, not displayed in the post archive. 
A `.rst` file without the `.. posts::` directive is a regular document/page.


#### Regular (non-post) Documents or Pages

Place the `.rst` files or tree under `class` and add a `toctree` entry in the top-level `index.rst` file.

## Testing

To build

```
ablog build
```

To view on a local computer

```
ablog serve
```



## Publishing

Push the commits to the master branch on github automatically triggers a rebuild via Travis CI.
The updated contents should be viewable in a few minutes.

