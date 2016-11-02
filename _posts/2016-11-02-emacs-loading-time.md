---
layout: post
title:  "Debug your Emacs loading time"
excerpt_separator: <!--excerpt-->
date:   2016-11-02
categories: articles
tags:
- emacs
- debug
---

How to debug your config file loading time with the --debug-init command line argument.
<!--excerpt-->

# The goal

According to the numerous Emacs configuration repositories on Gitlab/Github/Bitbucket (or any other VCS frontends), we are a lot to write our config this way: **an `init.el` file that sets some basic stuff and then loads some files from a given directory**

It is indeed a pretty common way to manage its configuration since these loaded files divides the configuration by "topic" or by "package".

Now let's imagine that the loading time of Emacs is quite long and you want to determine which file causes that.

# The prerequisites

Make sure you have a folder where you store your lisp code. Let's say `/home/you/src/lisp`.

```shell
total 16
-rw-r--r--  1 you you    41B  2 nov 10:48 super-feature.el
-rw-r--r--  1 you you    41B  2 nov 10:48 extra-feature.el
```

And your lisp code must use the [`provide`](https://www.gnu.org/software/emacs/manual/html_node/elisp/Named-Features.html) function to handle named features:

```emacs-lisp
;;; super-feature.el --- My super feature

;;; Commentary:

;;; Code:

(some-heavy-configuration)

(provide 'super-feature)

;;; super-feature.el ends here
```

Pretty common and nothing really fancy, here!

_Tip: do not name your files with a digit at the first position. It won't be loaded._

# The snippet

Here is the trick: we will activate some benchmarking using the `--debug-init` command line argument.

We will detect the usage of this argument tanks to the Emacs internal variable [`init-file-debug`](http://git.savannah.gnu.org/cgit/emacs.git/tree/lisp/startup.el#n1007) and then use the [`benchmark`](http://git.savannah.gnu.org/cgit/emacs.git/tree/lisp/emacs-lisp/benchmark.el) internal library.

To handle that, here is what your `init.el` must look like:

```emacs-lisp
;; Display the total loading time in the minibuffer

(defun display-startup-echo-area-message ()
  "Display startup echo area message."
  (message "Initialized in %s" (emacs-init-time)))

;; Benchmark loading time file by file and display it in the *Messages* buffer

(when init-file-debug
  (require 'benchmark))

(let ((lisp-dir "/home/you/src/lisp"))
  (add-to-list 'load-path lisp-dir)
  (mapc (lambda (fname)
          (let ((feat (intern (file-name-base fname))))
            (if init-file-debug
                (message "Feature '%s' loaded in %.2fs" feat
                         (benchmark-elapse (require feat fname)))
              (require feat fname))))
        (directory-files lisp-dir t "\\.el")))
```

Now, you will have some detailed loading time in the `*Messages*` buffer and the general loading time in your minibuffer each time you launch Emacs this way:

```shell
emacs --debug-init
```

You can leave this snippet in your Emacs configuration file since it is only activated when the command line argument is used. Therefore, it won't disturb the normal process when not using it.

# The result

Here is what is displayed in your `*Messages*` buffer:

```
Feature ’extra-feature’ loaded in 0.08s
Feature ’super-feature’ loaded in 2.27s
```

Happy debugging!
