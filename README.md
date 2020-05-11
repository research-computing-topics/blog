[![Build Status](https://travis-ci.com/research-computing-topics/research-computing-topics.github.io.svg?branch=master)](https://travis-ci.com/research-computing-topics/research-computing-topics.github.io)

# Research Computing Topics


## Requirements for local testing

- Install of `Sphinx` and `ablog`
    - e.g. an Anaconda installation, followed by `pip install -r requirements.txt` should work


## Testing

To build

```
ablog build
```

To view on a local computer

```
ablog serve
```

## Writing

Please place the source files in the right place.

## File structure

#### Blog


Place the `.rst` file containing the blog post with

```
.. posts:: 2020-05-11
   :author: author_name
   :tags: tag1, tag2
    
```
in directory `blog/`. 
Conveniently, enter the `blog/` directory and run the command `ablog post` to create a file template
as the starting point of writing a new blog.

Posts without the date are drafts, not displayed in the post archive.


#### Documents or Pages

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

