# thefever.xyz



## Creating a new blog post

### TL;DR

1. Run `hugo new posts/my-post.md`
2. Update metadata and write post in `posts/my-post.md`
3. Run `git add posts/my-post.md`
4. Run `git commit -m "Add my-post."`
5. Run `git push`



### 1. Add yourself as an author (only once)

First add yourself to the list of authors, this is done by creating a json file
in the [data/authors](data/authors) directory. Copy at one of the existing json
files, rename to `yourname.json` and edit the contents of the file.



### 2. Generate from template

To create a new blog post run the following command:

```shell
hugo new posts/my-post.md
```

This will create a new (empty) file in [content/posts](content/posts/), you can
open that file and start writing your post! Note that `draft: true` by default,
meaning your post will not be public until draft is set to `false`.

If you want your post to show you as the author, ensure that `author: yourname`
is added to the metadata block of your post.



### 3. Write your post

Posts are written in [Markdown](https://www.markdownguide.org/cheat-sheet/), it
is possible, through the power of our generator, to write posts in HTML or some
other formats check [here](https://gohugo.io/content-management/) for supported
formats.



### 4. Preview your post locally (optional)

To run Hugo with drafts enabled, run the following command:

```shell
hugo server -D
```

This will start instruct Hugo to generate pages from your posts, and run a http
server with the blog locally.

Visit [http://localhost:1313](http://localhost:1313) to preview your changes.

### 5. Publish

When you are satisfied with your post, deploying is done like this:

1. Ensure that `draft: false` is set in the posts metadata.
2. Commit and push your changes to Github
3. Done

A Github Action will take care of generating and actually deploying the changes
for real. Check [https://thefever.dev](http://thefever.dev) to ensure that your
post is showing up.

### 6. Celebrate

You have just made a contribution to the world's collective knowledge. Not only
is writing a good skill to hone in general, but your peers and your future self
will thank you for your input. Congratulations!

If you like you can share the link to the fruits of your labor on channels such
as Hacker News, Reddit, Linkedin, etc. and/or cross-post to your personal blog.



## Setup for local development

### 1. Clone this repository

Run the following command:

```shell
git clone git@github.com:TSting/techblog.git
```

Alternatively you can use the buttons in the Github interface.

### 2. Install Hugo

Run the following command:

```shell
# Assuming macOs
brew install hugo
```

If you're using a different platform, please refer to the official installation
instructions [here](https://gohugo.io/getting-started/installing).

Ensure that Hugo is installed using the following command:

```shell
hugo version
```

If you see output you should be fine.

### 3. Verifying

To run a local development server run the following command:

```shell
hugo server
```

If all went well, visiting [http://localhost:1313](http://localhost:1313) will
show you a local version of the blog.