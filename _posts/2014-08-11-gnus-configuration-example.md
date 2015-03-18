---
layout: post
title:  "Gnus configuration example"
excerpt_separator: <!--excerpt-->
date:   2014-08-11
categories: articles
tags:
- emacs
- email
---

Configure one of the most efficient and powerful email client: Gnus.
<!--excerpt-->

# About Gnus

## Quick presentation

Gnus is a message reader included in Emacs. It is obviously written in Lisp and its configuration must also be written in Lisp.

As *message*, Gnus refer to every kind of messages: *news*, *rss*, and *emails*.

It is a full featured mail user agent with some extensible capabilities that leads you to a more efficient message manager than any other existing MUA.

Of course, it is interfaced with many external Lisp and non-Lisp programs like *org-mode*, *web browsers*, *encryption tools*, ... etc.

## About credentials

The problem with configuration files is that they are readable if an access to your account is stolen or provided by error (please, lock your machine when you go away, even for a few seconds).
Even if you set some hardened permissions on the file, if someone has access to your terminal, a simple `cat` command will display your credentials.

Gnus relies on a file that list all credentials for your accounts so they are not written in your email client configuration.
But the problem still exists because that credential file is still there.

We will rely on the fact that Emacs has a powerful inteface with GPG tools and allows to create/edit encrypted files directly in Lisp. Therefore, if we store our credentials in an encrypted file, Gnus (via Emacs) will be able to decrypt that file and use the content to authenticate you on your IMAP/SMTP accounts. And that file stays secure for a (bad) human.

To do so, just create a `~/.authinfo.gpg` file from Emacs and let Emacs store your credentials here automatically (it will be asked once).

# The configuration example

## Prerequisites

In order to use Gnus, you must create a `~/.gnus` file and add all the following configuration here. Simple as that.

## A detailed configuration

### General configuration

{% highlight elisp %}
(setq-default

 ;; User identity
 user-full-name "Your name"
 user-mail-address "your@email.com"

 ;; Behaviour
 read-mail-command 'gnus
 gnus-select-method '(nnml "")
 gnus-large-newsgroup 'nil ;; No expiration
 gnus-fetch-old-headers 'some ;; Load read messages
 mm-text-html-renderer 'w3m
 gnus-inhibit-images nil
 gnus-ignored-newsgroups ""
 gnus-topic-display-empty-topics nil
 gnus-summary-thread-gathering-function 'gnus-gather-threads-by-references
 gnus-thread-sort-functions '(gnus-thread-sort-by-most-recent-date)
 gnus-parameters '(("nnimap.*"
                    (display . 300) ;; (display . all)
                    (gnus-use-scoring nil)
                    (expiry-wait . 5)))

 ;; Signature
 gnus-posting-styles '((".*" (signature "\n\nBest regards,\nYour name\n"))))
{% endhighlight %}

Some extra configuration

{% highlight elisp %}
(setq

 ;; Add date to reply & quote
 message-citation-line-function 'message-insert-formatted-citation-line
 message-citation-line-format "\nOn %a, %b %d %Y, %f wrote:\n"

 ;; Mime types
 mm-discouraged-alternatives '("text/html" "text/richtext")
 gnus-ignored-mime-types '("text/x-vcard")

 ;; TLS
 starttls-use-gnutls t
 ;; starttls-extra-arguments '("--insecure")
 starttls-gnutls-program "gnutls-cli")
{% endhighlight %}

### Formatting

Display informations as you want

{% highlight elisp %}
(setq-default

 ;; Date specific format
 gnus-user-date-format-alist '(((gnus-seconds-today) . "Today, %H:%M")
                               ((+ 86400 (gnus-seconds-today)) . "Yesterday, %H:%M")
                               (604800 . "%A %H:%M")
                               ((gnus-seconds-month) . "%A %d")
                               ((gnus-seconds-year) . "%d %B")
                               (t . "%d/%m/%Y %H:%M"))

 ;; Group line
 gnus-group-line-format " %S [%5y] | %-10s | %G\n"

 ;; Summary line
 gnus-summary-line-format " %R%U%z %4k | %(%~(pad-right 16)&user-date; | %-25,25f | %B%s%)\n")
{% endhighlight %}

### Wrap message to 72 columns

Yes, I know we are in the 21st century, but this is a habit.

{% highlight elisp %}
(unless (boundp 'message-fill-column)
  (add-hook 'message-mode-hook
            (lambda ()
              (setq fill-column 72)
              (turn-on-auto-fill))))
{% endhighlight %}

### Auto correct

When writing a message, activate `flyspell` to detect typos

{% highlight elisp %}
(add-hook 'message-mode-hook
          (lambda ()
            (flyspell-mode 1)))
{% endhighlight %}

### Auto refresh and notifications

Set autorefresh to 1mn, which is a little too much for non-alert-message-paranoids

{% highlight elisp %}
;; Autorefresh 1 mn
(gnus-demon-add-handler 'gnus-demon-scan-news 1 nil)

;; Notifications
(require 'gnus-desktop-notify)
(if (string-equal system-type "darwin")
    (setq gnus-desktop-notify-function 'gnus-desktop-notify-exec
          gnus-desktop-notify-exec-program "growlnotify -a Emacs.app -m"))
(gnus-desktop-notify-mode)
(gnus-demon-add-scanmail)
{% endhighlight %}

### PGP

Use PGP to sign your messages. This should be done by everybody.
This snippet will add a `signature.asc` file in attachment.

{% highlight elisp %}
(require 'epg-config)

(setq
 epg-debug t ;;  *epg-debug*" buffer
 mml2015-use 'epg
 mml2015-verbose t
 mml2015-encrypt-to-self t
 mml2015-always-trust nil
 mml2015-cache-passphrase t
 mml2015-passphrase-cache-expiry '36000
 mml2015-sign-with-sender t
 gnus-message-replyencrypt t
 gnus-message-replysign t
 gnus-message-replysignencrypted t
 gnus-treat-x-pgp-sig t
 mm-verify-option 'always
 mm-decrypt-option 'always
 mm-sign-option nil ;; mm-sign-option 'guided
 gnus-buttonized-mime-types
 '("multipart/alternative" "multipart/encrypted" "multipart/signed"))

(defadvice mml2015-sign (after mml2015-sign-rename (cont) act)
  (save-excursion
    (search-backward "Content-Type: application/pgp-signature")
    (goto-char (point-at-eol))
    (insert "; name=\"signature.asc\"; description=\"Digital signature\"")))

(add-hook 'message-send-hook (lambda () (mml-secure-message-sign-pgpmime)))
{% endhighlight %}

### Search

`nnir` is a great package that has enhanced search capabilities for Gnus. To use it, just require the package.

{% highlight elisp %}
(require 'nnir)
{% endhighlight %}

### Contacts

Use the `bbdb` package to manage your Contacts

{% highlight elisp %}
(require 'bbdb)

(bbdb-initialize 'gnus 'message)

(setq
 bbdb-north-american-phone-numbers-p nil
 bbdb-quiet-about-name-mismatches t
 bbdb-offer-save 1
 bbdb-use-pop-up t
 bbdb-always-add-address t
 bbbd-message-caching-enabled t
 bbdb-elided-display t
 bbdb/mail-auto-create-p 'bbdb-ignore-some-messages-hook
 bbdb-ignore-some-messages-alist
 '(( "From" . "no.?reply\\|DAEMON\\|daemon\\|facebookmail\\|twitter")))

(add-hook 'gnus-startup-hook 'bbdb-insinuate-gnus)
{% endhighlight %}

### Windows positions

Setup a more user friendly layout for your email client

{% highlight elisp %}
(gnus-add-configuration
 '(article
   (horizontal 1.0
               (vertical 55 (group 1.0))
               (vertical 1.0
                         (summary 0.16 point)
                         (article 1.0)))))

(gnus-add-configuration
 '(summary
   (horizontal 1.0
               (vertical 55 (group 1.0))
               (vertical 1.0 (summary 1.0 point)))))
{% endhighlight %}

### Mail splitting

Splitting is the action to filter your mails according to a regexp

{% highlight elisp %}
(setq
 nnimap-split-inbox '("INBOX")
 nnimap-split-predicate "UNDELETED"
 nnimap-split-crosspost nil
 nnmail-split-methods
 '(
   ;; Filtered messages
   ("Emacs" "^.*emacs-devel@gnu.org")
   ("Github" "^From:.*notifications@github.com")
   ("Security" "^.*debian-security@lists.debian.org")
   ("Comm" "^Subject:.*\\[Comm\\]")
   ;; Others
   ("Misc" "")))
{% endhighlight %}

### Mail sources

Setup some IMAP accounts

{% highlight elisp %}
(setq gnus-secondary-select-methods
      '((nnimap "MyServer1"
                (nnimap-stream ssl)
                (nnimap-address "mail.myserver1.com")
                (nnimap-inbox "INBOX")
                (nnimap-split-methods default)
                (nnir-search-engine imap))
        (nnimap "MyServer2"
                (nnimap-stream ssl)
                (nnimap-address "imap.myserver2.com")
                (nnimap-server-port 993)
                (nnimap-inbox "INBOX")
                (nnimap-split-methods default)
                (nnir-search-engine imap))))
{% endhighlight %}

### Mail destinations

Setup some SMTP accounts

{% highlight elisp %}
(defun setMyServer1SMTP ()
  (interactive)
  (message "Select MyServer1 SMTP server")
  (setq user-mail-address "myname@myserver1.com")
  (setq message-send-mail-function 'smtpmail-send-it
        smtpmail-starttls-credentials '(("mail.myserver1.com" 25 nil nil))
        smtpmail-auth-credentials '(("mail.myserver1.com" 25 "myname@myserver1.com" nil))
        smtpmail-default-smtp-server "mail.myserver1.com"
        smtpmail-smtp-server "mail.myserver1.com"
        smtpmail-smtp-service 25
        smtpmail-local-domain "myserver1.com"))

(defun setMyServer2SMTP ()
  (interactive)
  (message "Select MyServer2 SMTP server")
  (setq user-mail-address "myname@myserver2.com")
  (setq message-send-mail-function 'smtpmail-send-it
        smtpmail-starttls-credentials '(("smtp.myserver2.com" 25 nil nil))
        smtpmail-auth-credentials '(("smtp.myserver2.com" 25 "myname@myserver2.com" nil))
        smtpmail-default-smtp-server "smtp.myserver2.com"
        smtpmail-smtp-server "smtp.myserver2.com"
        smtpmail-smtp-service 25
        smtpmail-local-domain "myserver2.com"))
{% endhighlight %}

Select the right SMTP server according to the message group we are reading/editing

{% highlight elisp %}
(add-hook 'message-mode-hook
          '(lambda ()
             (cond
              ((string-match "myserver1" gnus-newsgroup-name)
               (setMyServer1SMTP))
              (t (setMyServer2SMTP)))))
{% endhighlight %}

Here we are!

# The whole thing

{% highlight elisp %}
(setq-default

 ;; User identity
 user-full-name "Your name"
 user-mail-address "your@email.com"

 ;; Behaviour
 read-mail-command 'gnus
 gnus-select-method '(nnml "")
 gnus-large-newsgroup 'nil ;; No expiration
 gnus-fetch-old-headers 'some ;; Load read messages
 mm-text-html-renderer 'w3m
 gnus-inhibit-images nil
 gnus-ignored-newsgroups ""
 gnus-topic-display-empty-topics nil
 gnus-summary-thread-gathering-function 'gnus-gather-threads-by-references
 gnus-thread-sort-functions '(gnus-thread-sort-by-most-recent-date)
 gnus-parameters '(("nnimap.*"
                    (display . 300) ;; (display . all)
                    (gnus-use-scoring nil)
                    (expiry-wait . 5)))

 ;; Signature
 gnus-posting-styles '((".*" (signature "\n\nBest regards,\nYour name\n"))))

(setq
 ;; Add date to reply & quote
 message-citation-line-function 'message-insert-formatted-citation-line
 message-citation-line-format "\nOn %a, %b %d %Y, %f wrote:\n"

 ;; Mime types
 mm-discouraged-alternatives '("text/html" "text/richtext")
 gnus-ignored-mime-types '("text/x-vcard")

 ;; TLS
 starttls-use-gnutls t
 ;; starttls-extra-arguments '("--insecure")
 starttls-gnutls-program "gnutls-cli")

(setq-default
 ;; Date specific format
 gnus-user-date-format-alist '(((gnus-seconds-today) . "Today, %H:%M")
                               ((+ 86400 (gnus-seconds-today)) . "Yesterday, %H:%M")
                               (604800 . "%A %H:%M")
                               ((gnus-seconds-month) . "%A %d")
                               ((gnus-seconds-year) . "%d %B")
                               (t . "%d/%m/%Y %H:%M"))
 ;; Group line
 gnus-group-line-format " %S [%5y] | %-10s | %G\n"
 ;; Summary line
 gnus-summary-line-format " %R%U%z %4k | %(%~(pad-right 16)&user-date; | %-25,25f | %B%s%)\n")

(unless (boundp 'message-fill-column)
  (add-hook 'message-mode-hook
            (lambda ()
              (setq fill-column 72)
              (turn-on-auto-fill))))

(add-hook 'message-mode-hook
          (lambda ()
            (flyspell-mode 1)))

;; Autorefresh 1 mn
(gnus-demon-add-handler 'gnus-demon-scan-news 1 nil)

;; Notifications
(require 'gnus-desktop-notify)

(if (string-equal system-type "darwin")
    (setq gnus-desktop-notify-function 'gnus-desktop-notify-exec
          gnus-desktop-notify-exec-program "growlnotify -a Emacs.app -m"))
(gnus-desktop-notify-mode)
(gnus-demon-add-scanmail)

(require 'epg-config)

(setq
 epg-debug t ;;  *epg-debug*" buffer
 mml2015-use 'epg
 mml2015-verbose t
 mml2015-encrypt-to-self t
 mml2015-always-trust nil
 mml2015-cache-passphrase t
 mml2015-passphrase-cache-expiry '36000
 mml2015-sign-with-sender t
 gnus-message-replyencrypt t
 gnus-message-replysign t
 gnus-message-replysignencrypted t
 gnus-treat-x-pgp-sig t
 mm-verify-option 'always
 mm-decrypt-option 'always
 mm-sign-option nil ;; mm-sign-option 'guided
 gnus-buttonized-mime-types
 '("multipart/alternative" "multipart/encrypted" "multipart/signed"))

(defadvice mml2015-sign (after mml2015-sign-rename (cont) act)
  (save-excursion
    (search-backward "Content-Type: application/pgp-signature")
    (goto-char (point-at-eol))
    (insert "; name=\"signature.asc\"; description=\"Digital signature\"")))

(add-hook 'message-send-hook (lambda () (mml-secure-message-sign-pgpmime)))

(require 'nnir)

(require 'bbdb)
(bbdb-initialize 'gnus 'message)
(setq
 bbdb-north-american-phone-numbers-p nil
 bbdb-quiet-about-name-mismatches t
 bbdb-offer-save 1
 bbdb-use-pop-up t
 bbdb-always-add-address t
 bbbd-message-caching-enabled t
 bbdb-elided-display t
 bbdb/mail-auto-create-p 'bbdb-ignore-some-messages-hook
 bbdb-ignore-some-messages-alist
 '(( "From" . "no.?reply\\|DAEMON\\|daemon\\|facebookmail\\|twitter")))

(add-hook 'gnus-startup-hook 'bbdb-insinuate-gnus)

(gnus-add-configuration
 '(article
   (horizontal 1.0
               (vertical 55 (group 1.0))
               (vertical 1.0
                         (summary 0.16 point)
                         (article 1.0)))))
(gnus-add-configuration
 '(summary
   (horizontal 1.0
               (vertical 55 (group 1.0))
               (vertical 1.0 (summary 1.0 point)))))

(setq
 nnimap-split-inbox '("INBOX")
 nnimap-split-predicate "UNDELETED"
 nnimap-split-crosspost nil
 nnmail-split-methods
 '(
   ;; Filtered messages
   ("Emacs" "^.*emacs-devel@gnu.org")
   ("Github" "^From:.*notifications@github.com")
   ("Security" "^.*debian-security@lists.debian.org")
   ("Comm" "^Subject:.*\\[Comm\\]")
   ;; Others
   ("Misc" "")))

(setq gnus-secondary-select-methods
      '((nnimap "MyServer1"
                (nnimap-stream ssl)
                (nnimap-address "mail.myserver1.com")
                (nnimap-inbox "INBOX")
                (nnimap-split-methods default)
                (nnir-search-engine imap))
        (nnimap "MyServer2"
                (nnimap-stream ssl)
                (nnimap-address "imap.myserver2.com")
                (nnimap-server-port 993)
                (nnimap-inbox "INBOX")
                (nnimap-split-methods default)
                (nnir-search-engine imap))))

(defun setMyServer1SMTP ()
  (interactive)
  (message "Select MyServer1 SMTP server")
  (setq user-mail-address "myname@myserver1.com")
  (setq message-send-mail-function 'smtpmail-send-it
        smtpmail-starttls-credentials '(("mail.myserver1.com" 25 nil nil))
        smtpmail-auth-credentials '(("mail.myserver1.com" 25 "myname@myserver1.com" nil))
        smtpmail-default-smtp-server "mail.myserver1.com"
        smtpmail-smtp-server "mail.myserver1.com"
        smtpmail-smtp-service 25
        smtpmail-local-domain "myserver1.com"))

(defun setMyServer2SMTP ()
  (interactive)
  (message "Select MyServer2 SMTP server")
  (setq user-mail-address "myname@myserver2.com")
  (setq message-send-mail-function 'smtpmail-send-it
        smtpmail-starttls-credentials '(("smtp.myserver2.com" 25 nil nil))
        smtpmail-auth-credentials '(("smtp.myserver2.com" 25 "myname@myserver2.com" nil))
        smtpmail-default-smtp-server "smtp.myserver2.com"
        smtpmail-smtp-server "smtp.myserver2.com"
        smtpmail-smtp-service 25
        smtpmail-local-domain "myserver2.com"))

(add-hook 'message-mode-hook
          '(lambda ()
             (cond
              ((string-match "myserver1" gnus-newsgroup-name)
               (setMyServer1SMTP))
              (t (setMyServer2SMTP)))))
{% endhighlight %}

