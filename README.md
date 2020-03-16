# org-backlink
Show backlinks for orgmode entries.

When taking notes in org it is useful to add links to other notes with related content. However, if you add a link from A to B then obviously B is also related to A, so the user should see the connection without adding manually a link from B to A.

This is what this package does. First you need to run M-x org-backlink-mode-refresh-cache which scans your org files (your agenda files by default) for links.

Then you can turn on org-backlink-mode minor mode in an org buffer which automatically shows backlinks to the current entry.

Here's a screenshot using the Org Manual as an example: 

![Screenshot](https://raw.githubusercontent.com/codecoll/org-backlink/master/screenshot.png)

If you don't want to see a message if the entry has no backlinks then set org-backlink-mode-show-no-backlink-message to nil.

*This code works for me, I don't plan to develop it further. I just posted it in case others find it useful. Please don't submit pull requests.*
