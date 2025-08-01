+++
title = "Introduction to Emacs"
author = ["Abhimanyu G"]
description = "Introduction to the Emacs editor"
date = 2025-07-17T00:00:00+05:30
tags = ["software", "emacs"]
draft = false
+++

## Brief {#brief}

Emacs is an awesome _text editor_, _Integrated development environment (IDE)_, _Documentation utility_, _Got-to-do (GTD)_, _Recipe book_, _habit tracker_ and heck a _Music player_ too! Basically, anything you imagine, there is a good chance Emacs community has figured out how its done.
In this document, I will attempt to address few of the awesomeness of this wonderful software


### What's in the box ? {#what-s-in-the-box}

As mentioned above, Emacs is a one stop solution for all your text editing needs. Out of the box, Vanilla Emacs looks pretty boring. You get basic editing facility. But what makes it great is the extensibility.

<a id="figure--GNU Emacs"></a>

{{< figure src="/images/vanilla-emacs.png" caption="<span class=\"figure-number\">Figure 1: </span>Vanilla (a.k.a GNU) Emacs. Source [gnu.org](https://www.gnu.org/software/emacs/)" >}}

You are probably disheartened comparing this to the latest text editors/IDEs like VSCode, Cursor and plethora of others. I remember I was!

> What is all the hype about ?

you may ask. Rightfully so.

Well, under this seemingly simple piece of software lies the ability to configure pretty much anything possible


### What needs work ? {#what-needs-work}

> With great power, comes great responsibility

Emacs is a powerhouse that has to be wielded to our will. Each of our usage would be different. An academician, might use it for documentation, a software programmer might use it as an IDE, You might like Light theme, I might like dark one. You might like full screen view, I might like a different setup... you get the idea.

The configurations are done with a scripting language known as _lisp_ or a Emacs variant of it known as [elisp](https://www.gnu.org/software/emacs/manual/html_node/eintr/)


## How do I get started? {#how-do-i-get-started}

Instead of getting lost in the ocean of configurations. I suggest having a basic requirements and trying to achieve those first. Here is a reference table that I might have used if I were to start all over again.
Here is a possible road map, that gives you a good start

{{< figure src="/ox-hugo/org-getting-started-roadmap.svg" >}}


#### Installing Emacs {#installing-emacs}

Emacs can be installed in popular distros with respective package managers. To get the latest and greatest version of Emacs, I suggest installing emacs from source (That's right, Emacs is open source, in case you did not now).
Follow installation instructions [here](https://www.gnu.org/software/emacs/download.html)


#### Basics {#basics}

Emacs could feel strange, uncomfortable or non-intuitive at first. This is your training phase. Think of standing on your legs from crawling as a baby! it is supposed to be that way. Emacs is a **keyboard dominant** software. It would be a big boost knowing your way around the software. Moving up/down/right/left, jumping, marking, copying, pasting, undo, redo, etc... GNU has a good primer with their [guided tour](https://www.gnu.org/software/emacs/tour/)


#### Org mode {#org-mode}

Arguably, one of the best structural documentation mode that exists. It lets you do much more with basic text manipulation.

-   Add metadata to a document
-   Add formatted text such as prose, quote, comment
-   Add and execute code from within the document
-   Export to webpage (you are reading one such export!), pdf, markdown and other formats

Your love affair 💖 with org-mode should begin [here](https://orgmode.org/) and [here](https://orgmode.org/quickstart.html)

<!--list-separator-->

-  Org-babel

    This is what brings the ability to include code blocks in your org-document and execute them in place. It is called [literate-programming](https://en.wikipedia.org/wiki/Literate_programming).
    Information of how a code block can be included in the org-document and executed, exported can be found at [orgmode's userguide](https://orgmode.org/org.html#Working-with-Source-Code-1) .

    Another awesomeness of _org-babel_ is the ability to tangle code blocks. Imagine you write a document filled with code blocks each doing a small job. For example, consider below code-blocks

    ```C
    void welcomeGuest(const char *guestName){
      if(guestName){
        printf("Welcome %s\n", guestName);
      }
    }
    ```

    ```C
    int main (void){
      welcomeGuest("John Doe");
      return 0;
    }
    ```

    Let's say, you would want all of these individual blocks to be written to a file called `main.c`. Instead of copying all the code blocks by hand. Org babel supports a functionality called _tangle_. It picks up all the code blocks of a type (in this case 'C' type) and `T-a-n-g-l-e-s` them into a source file. This way, you could have self documenting code blocks!! **How awesome is that ?!**


#### Init configuration {#init-configuration}

`init.el` is the way you tell Emacs how to configure itself (think themes, font, font sizes, 3rd party packages, window appearances etc..).

I have intentionally put init configurations after learning a bit about basic motions and org mode (especially org-babel and org-tangle). This is because, I strongly recommend having your configurations well documented in `init.org` and tangle all the _elsip_ code to `init.el` such that you have a well documented configurations.
Checkout my fragment of gist configuration.
[view it raw](https://gist.githubusercontent.com/abhimanyu-g/44bf0f1b6a3ad191bf2ab2ed662e18bc/raw/c9c709c48af43822d6c50d7ab7ac236e5d0d34ac/init-fragment.org)


### Closing notes {#closing-notes}

Working with Emacs is a lifelong adventure. You learn-to-learn with endless discovery. It can be tailored please and support you with your specific workflow. Often, when you ask the question _Can I do this with Emacs?_, most likely the answer is a bold **YES**

Remember to not get too deep the rabbit hole, and do the configurations based on your specific requirements

If you need any help in the configurations, do not be shy to reach out to me


### Helpful links {#helpful-links}

-   <https://systemcrafters.net/emacs-essentials/absolute-beginners-guide-to-emacs/>
-   <https://karthinks.com/software/emacs-window-management-almanac/>
