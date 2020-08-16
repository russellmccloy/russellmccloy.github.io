---
layout: post
title:  "Documentation Sucks - But it doesn't"
date:   2020-08-17 07:26:18 +1000
categories: general

---

Today I'd like to write about documentation in the context of being a developer. Over my career, whenever anyone has asked me to *"document it"* I always get a little bit of a sick feeling in my stomach. Thing is, once I start documenting, I realise it is not too bad and I actually enjoy it.

I have also written this post as, when reviewing my colleague's code, I realise that the information in this post may help them too.

Now that the tools we use to do our jobs have got to the point where we can write documentation the same way we write code, there really is no barrier to writing documentation.

## Why Doesn't Documentation Suck

**Here are my top thoughts on why documentation doesn't suck:**

- I can write documentation (in particular `README.md` files written using the Markdown syntax) within `VSCode` and check it into to source control the same way I do with code.
- When I write documentation I find it solidifies the knowledge and understanding of the code I have just written.
- Writing documentation forces me to put myself in the shoes of an audience which in turn, forces me to write and update my documentation until I feel the reader will have no issue in understanding my code very quickly.
- There are tools to help you with you documentation that automatically detect issues with the format of what you have written and help you fix it. For example, within VSCode there are `linting` extensions for markdown.

## My Tips for Writing Documentation as a Developer

- **Use the tools available to you** - use linters etc. See the Documentation Tools section below
- **Start documenting at the same time as you start writing code** - It is hard to be disciplined in this way but if you don't do this, it may be hard to remember later if you decide to only document after you have finished coding.
- **Document as you go** - keep the document in an open tab so you can update it as you go.
- **Use code snippets and pictures** - I believe pictures and samples help out the reader immensely and it also means you will need to type less information in your documents.

```python
def do_something(something):
    """
    Does something
    :param something: The something to do
    :return: the result of something
    """
    print("Something was: {}".format(something))

```

## Documentation Tools

**Some documentation tools that I use are:**

- **Markdown All in One** - This extension includes auto-complete, GitHub flavoured markdown -  <https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one>.

- **markdownlint** - It includes a library of rules to encourage standards and consistency for Markdown files. If you are writing a markdown file and you notice a markdownlint warning like the one shown in this picture:
![markdownlint](/assets/markdownlint_1.jpg)
you can immediately fix it.
These rules are described in full on this page: <https://github.com/DavidAnson/markdownlint/blob/v0.20.4/doc/Rules.md>:
![rules](/assets/rules1.jpg) <https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint>

- **Code Spell Checker** - This extension checks you spelling in code and documentation. Yes, even though you write code you still need correct spelling. How many times have you completed a Pull Request for a colleague and the code is awesome but they can't spell!? <https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker>

- **Todo Tree** - This is not directly related to documentation but is one of the most valuable tools I use. Whilst I am coding and I think of something I need to do later, I can quickly whack a `TODO:` in my code and `Todo tree` will keep track of it for me. See pic below. <https://marketplace.visualstudio.com/items?itemName=Gruntfuggly.todo-tree>![Todo Tree](/assets/doc_something.jpg)

That's all for now and thanks for reading.
