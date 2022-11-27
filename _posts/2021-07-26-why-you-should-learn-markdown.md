---
title: Why You Should Learn Markdown
Author: Brandon J. Kessler
tags: Software DevOps Productivity
categories: Technology
published: true
---

<h1>{{ page.title }}</h1>

## What is Markdown

Markdown was released in December of 2004 by John Gruber. His website [Daring Fireball](https://daringfireball.net/projects/markdown/) gives the following as the introduction for Markdown:

> Markdown is a text-to-HTML conversion tool for web writers. Markdown allows you to write using an easy-to-read, easy-to-write plain text format, then convert it to structurally valid XHTML (or HTML).

That sounds simple enough, and it really is. Markdown is a set of rules and syntax.
<!--more-->
## Why _should_ you learn Markdown?

Why _should_ you learn Markdown anyway? Don’t we already have a bunch of document editors that do all this formatting already? And we have HTML that does something similar, right? Even the creator is saying it’s a text-to-html conversion tool.

What makes Markdown so powerful is that it’s extremely easy to pick up. If you understand why you’d want to use a Heading 1 in a document vs a Heading 2 then you’re already well on your way to learning Markdown. The syntax is pretty straight forward. A `#` is used for headings. Specifically a `#` followed by a  (space) followed by the heading you want, i.e. `# Heading 1`. For a Heading 2 you just add another `#`, so `## Heading 2`, and for Heading 3 you’ll have 3 `#` marks. To _italicise_ something you add a `*` to front and end of the word or phrase. For **bold** you add `**` to the beginning and end. For **_bold italics_** you add `***` to beginning and end. That’s pretty straight forward! Markdown basically uses `#`, `*`, `-`, and `1.` to format most of the information you’ll ever need to intaract with. For a full list of [Syntax](https://daringfireball.net/projects/markdown/syntax) follow the link. So writing `#` is a whole heck of a lot faster than `<h1></h1>`

Now that you already learned some Markdown it’s time to figure out why you should really learn it _and_ use it. The beauty of Markdown is that you can format as you write without ever leaving the keyboard. So while yes you could use `ctrl` + `i` in most text editors to italicize the word or phrase, Markdown is also portable. We’ll talk more on that in the next paragraph. You’ll notice I have a lot of `code` blocks in my document. I’m creating those on the fly using the `` ` `` (backtick). You can also use 3 `` ` `` at the beginning and end of a code sequence to format multiple lines. There’s no handy set of hotkeys for that in most editors. Markdown allows you to write quickly with minimal movement and distractions.

Markdown is also very portable and future resistant (not proof). At it’s core it’s just a plaintext document that can be converted to HTML. If you were to view a Markdown document in Notepad (or any other plaintext editor) you’d see something like the following:

```
## What is Markdown
Markdown was released in December of 2004 by John Gruber. His website [Daring Fireball](https://daringfireball.net/projects/markdown/) gives the following as the introduction for Markdown:
> Markdown is a text-to-HTML conversion tool for web writers. Markdown allows you to write using an easy-to-read, easy-to-write plain text format, then convert it to structurally valid XHTML (or HTML).

That sounds simple enough, and it really is. Markdown is a set of rules and syntax.
```

As you can see that’s just the first paragraph of this blog post, but without any of the formatting. All the information that needs to be there is present, including the URL for the link. You can still read it even if you don’t fully understand the syntax. That means that even if there’s no longer any way to convert the Markdown to something more attractive you still have the information as long as you can still open plaintext documents. That also means that you can write and read Markdown on any OS with relative ease. You’re not vendor locked in.

There are also multiple websites and social networks that use Markdown. Google Chat uses a stripped down version, and Reddit uses markdown for it’s posts. Github uses the `readme.md` file to display at the bottom of your repository page. You can inform viewers how to use your application or script and it’s already formated for you through github.

## How do _I_ use Markdown?

I have myself just recently started using Markdown consistently and constantly, but it is an absolute game changer in job. I work as a Deskside Analyst, generally solving Tier 2 and Tier 3 IT issues with our Desktops and Laptops. It’s primarily done using Microsoft’s SCCM, lot’s of PowerShell, and lots of Googling. I have a lot of meetings in regards to SCCM, PowerShell, and the standards we’ll support. So I use Markdown to take notes in those meetings. I use Markdown in our ticketing system, which doesn’t support it, but if we migrate to a system that does my tickets are already formatted.

I use Markdown when researching a topic or when creating documentation now. I use Markdown for the `readme.md` files I’m starting to create with my scripts. I am using Markdown right now to write this blogpost. I’m also heavily using it with [Dendron](http://dendron.so/), which has helped me create my own personal Knowledge Base. I can use other applications, like Pandoc, to convert my Markdown into something more business-standard-y like Microsoft Word or Google Docs, but I still have the original that is extremely portable and can be converted another way with little to no overhead.

## Conclusion

If you’re still not sold on using Markdown, that’s still okay! I strongly suggest finding a good Markdown editor (I use Microsoft Visual Studio Code), and try it out for a week. Pick one thing that you do often, like taking meeting notes, and then replace the regular application you use with writing in Markdown. Time it. Document it. And then the next week return to how you were doing it and time it again. Document it again. And then see which one you liked better.

## References

[Daring Fireball](https://daringfireball.net/projects/markdown/)