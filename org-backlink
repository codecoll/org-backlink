(defvar org-backlink-mode-files org-agenda-files
  "These files are scanned for backlinks.")


(defface org-backlink-mode-face
  '((t :background "#bbdefb"
       :foreground "black"))
  "")


(setq org-backlink-mode-show-no-backlink-message t)

(setq org-backlink-mode-overlay (make-overlay (point) (point)))
(delete-overlay org-backlink-mode-overlay) ;; hide it


(setq org-backlink-cache nil)

(setq org-backlink-current-entry-start 0)
(setq org-backlink-current-entry-end 0)

(setq org-backlink-current-entry-backlinks nil)


(defun org-backlink-mode-get-heading-path ()
  (append (org-get-outline-path)
          (list (let ((heading (org-get-heading)))
                  (set-text-properties 0 (length heading) nil heading)
                  heading))))


(defun org-backlink-mode-get-heading-path-string ()
  (concat (buffer-file-name)
          ":"
          (s-join
           "/"
           (org-backlink-mode-get-heading-path))))


(defun org-backlink-mode-refresh-cache ()
  (interactive)
  (let ((org-link-search-inhibit-query t)
        (linkcache (make-hash-table :test 'equal))
        (storefunc (lambda (linkcache source-file source-path)
                     (let* ((dest (org-backlink-mode-get-heading-path-string))
                            (entry (gethash dest linkcache))
                            (link (list 'file source-file
                                        'path source-path)))
                       (unless (member link entry)
                         (puthash dest
                                  (cons link entry)
                                  linkcache))))))
    (dolist (file org-backlink-mode-files)
      (message file)
      (with-current-buffer (find-file-noselect file)      
        (save-excursion
          (goto-char (point-min))
          (while (re-search-forward "\\[\\[\\(file:[^:]+::\\)?\\*" nil t)
            (ignore-errors
              (let* ((filelink (match-string 1))
                     (source-path (org-backlink-mode-get-heading-path))
                     (source-file (buffer-file-name)))
                (save-excursion
                  (org-open-at-point)
                  (if filelink
                      ;; in case of file links the file is opened asynchronously
                      ;; for some reason, so we have to wait for it to open
                      (run-with-idle-timer
                       0 nil storefunc linkcache source-file source-path)

                    ;; otherwise we can process it right away
                    (funcall storefunc linkcache source-file source-path)))))))))

    ;; restoring original buffer and position after idle timers, which
    ;; save-excursion cannot do
    (run-with-idle-timer
     0 nil
     (lambda (buffer pos)
       (switch-to-buffer buffer buffer)
       (goto-char pos))
     (current-buffer) (point))

    (setq org-backlink-cache linkcache)
    (message "Done.")))



(defun org-backlink-mode-show-backlinks ()
  (when (and (sit-for 0.3)
             (or org-backlink-cache
                 (progn
                   (message "Backlink cache is empty. Run org-backlink-mode-refresh-cache.")
                   nil)))
    (when (or (< (point) org-backlink-current-entry-start)
              (> (point) org-backlink-current-entry-end))
      (setq org-backlink-current-entry-start
            (or (ignore-errors (org-entry-beginning-position))
                1))
      (setq org-backlink-current-entry-end
            (org-entry-end-position))
      (setq org-backlink-current-entry-backlinks nil)
      (delete-overlay org-backlink-mode-overlay)

      (ignore-errors
        (let ((backlinks (gethash (org-backlink-mode-get-heading-path-string)
                                  org-backlink-cache)))
          (save-excursion
            (goto-char org-backlink-current-entry-start)
            (forward-line 1)            
            (move-overlay org-backlink-mode-overlay
                          (line-beginning-position)
                          (line-beginning-position)
                          (current-buffer)) 

            (overlay-put
             org-backlink-mode-overlay
             'after-string
             (concat
              (propertize
               (concat
                (if (or backlinks
                        org-backlink-mode-show-no-backlink-message)
                    "    "
                  "")

                (if backlinks
                    (substitute-command-keys
                     (format
                      "%s (press \\[org-backlink-mode-visit-backlink] to visit)"
                      (if (cdr backlinks)
                          (format "%s backlinks" (length backlinks))

                        (let ((backlink (car backlinks)))
                          (concat
                           "backlink: "
                           (if (equal (buffer-file-name)
                                      (plist-get backlink 'file))
                               ""
                             (concat (plist-get backlink 'file) ":"))
                           (s-join
                            "/"
                            (plist-get backlink 'path)))))))

                  (if org-backlink-mode-show-no-backlink-message
                      "No backlinks"))

                (if (or backlinks
                        org-backlink-mode-show-no-backlink-message)
                    "    "
                  ""))
               'face 'org-backlink-mode-face)
              "\n")))
          (setq org-backlink-current-entry-backlinks backlinks))))))



(defun org-backlink-mode-visit-backlink ()
  (interactive)
  (if org-backlink-current-entry-backlinks
      (let* ((selected
              (if (cdr org-backlink-current-entry-backlinks)
                  (let ((candidates
                         (mapcar
                          (lambda (backlink)
                            (cons (concat
                                   (if (equal (buffer-file-name)
                                              (plist-get backlink 'file))
                                       ""

                                     (concat (plist-get backlink 'file) ":"))
                                   (s-join
                                    "/"
                                    (plist-get backlink 'path)))
                                  backlink))
                          org-backlink-current-entry-backlinks)))
                    (assoc-default
                     (minibuffer-with-setup-hook
                         #'minibuffer-completion-help
                       (completing-read "Select a backlink: " candidates))                     
                     candidates))
                (car org-backlink-current-entry-backlinks))))
        (find-file (plist-get selected 'file))
        (goto-char (point-min))
        (dolist (elem (plist-get selected 'path))
          (re-search-forward (concat "^\\*+ *"(regexp-quote elem)))))

    (message "No backlink available here.")))



(define-minor-mode org-backlink-mode
  "Toggle backlink display for Org mode buffer."
  :init-value nil
  :lighter " OrgBL"
  :keymap (let ((map (make-sparse-keymap)))
            (define-key map (kbd "C-c l") 'org-backlink-mode-visit-backlink)
            map)

  (if org-backlink-mode
      (add-hook 'post-command-hook 'org-backlink-mode-show-backlinks t t)
    
    (remove-hook 'post-command-hook 'org-backlink-mode-show-backlinks t)
    (delete-overlay org-backlink-mode-overlay)))
