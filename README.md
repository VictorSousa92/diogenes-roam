# diogenes-roam

Passage notes filed by author and by work, with an index of every note on a
text — a companion to [`diogenes.el`][diogenes] and [org-roam][].

```
ref/aristotle/metaphysica/1048a-27.org
ref/aristotle/metaphysica/index.org
ref/plato/respublica/327a-5-327b-4.org
ref/plato/respublica/index.org
```

## Requires the org-integration branch of diogenes.el

**This package does nothing on its own.** It is a layer on top of
`diogenes-org.el`, which lives on the **org-integration branch** of
`diogenes.el` and is not in the released version.

Without that branch, `(featurep 'diogenes-org)` is nil, so the setup below
never runs and nothing happens at all — or, if you require it by hand,
`void-function diogenes-org--passage-parts` on the first capture. The quiet
failure is the one that wastes an afternoon, so check that first.

What comes from where:

| | |
|---|---|
| `diogenes-org.el` | the `diogenes:` link type, citations in `ROAM_REFS`, finding notes whose span covers the passage in view, capturing a note on a passage |
| this package | which directory that note goes in, what it is tagged, and an index node per work |

The hard part is `diogenes-org`'s. A note anchored to `NOTER_PAGE: 114` is
meaningless without that PDF and worthless if you change edition, whereas
`tlg:0059:030:327a.5` names the same passage in every edition there has ever
been — and comparing citations as numbers rather than strings is what lets a
note made on 327a5–327b4 be found from anywhere inside it. None of that is
here.

### Installing both

With [straight][] or [elpaca][], pointing at the branch:

```elisp
(straight-use-package
 '(diogenes :type git :host github :repo "VictorSousa92/diogenes.el"
            :branch "org-integration"))

(straight-use-package
 '(diogenes-roam :type git :host github :repo "VictorSousa92/diogenes-roam"))
```

Doom, in `packages.el`:

```elisp
(package! diogenes
  :recipe (:host github :repo "VictorSousa92/diogenes.el" :branch "org-integration"))
(package! diogenes-roam
  :recipe (:host github :repo "VictorSousa92/diogenes-roam"))
```

By hand:

```elisp
(add-to-list 'load-path "~/src/diogenes-roam")
(require 'diogenes-roam)
(require 'diogenes-roam-index)
```

## Setting up

Vanilla Emacs or anything else:

```elisp
(with-eval-after-load 'diogenes-org
  (require 'diogenes-roam)
  (require 'diogenes-roam-index)

  (diogenes-roam-mode 1)              ; file notes by author and work
  (diogenes-roam-index-global-mode 1) ; keep the indexes up to date

  (setq diogenes-roam-name-overrides
        '((("tlg" "0086") . "aristotle")
          (("tlg" "0059") . "plato")))

  (global-set-key (kbd "C-c n i") #'diogenes-roam-index-toggle)
  (global-set-key (kbd "C-c n p") #'diogenes-roam-find-passage))
```

Doom, in `config.el`:

```elisp
(use-package! diogenes-roam
  :after diogenes-org
  :demand t
  :config
  (require 'diogenes-roam-index)
  (diogenes-roam-mode 1)
  (diogenes-roam-index-global-mode 1)
  (setq diogenes-roam-name-overrides
        '((("tlg" "0086") . "aristotle")
          (("tlg" "0059") . "plato")))
  (map! :leader
        :desc "Index of this work" "n r I" #'diogenes-roam-index-toggle
        :desc "Passage notes"      "n r p" #'diogenes-roam-find-passage))
```

Note `:demand t`. `use-package!` defers by default, and a deferred
declaration of a package whose whole job is to set a variable at load time
will silently never run.

Spacemacs, in `dotspacemacs/user-config`:

```elisp
(with-eval-after-load 'diogenes-org
  (require 'diogenes-roam)
  (require 'diogenes-roam-index)
  (diogenes-roam-mode 1)
  (diogenes-roam-index-global-mode 1)
  (spacemacs/declare-prefix "od" "diogenes-roam")
  (spacemacs/set-leader-keys
    "odi" 'diogenes-roam-index-toggle
    "odp" 'diogenes-roam-find-passage))
```

## Commands

| | |
|---|---|
| `diogenes-roam-index-toggle` | open the index of the work in view, or shut it |
| `diogenes-roam-index-refresh` | rebuild the index being looked at (`g` in it) |
| `diogenes-roam-index-rebuild-all` | an index for every work there are notes on |
| `diogenes-roam-index-next` | the next note on this work, in the order of the text |
| `diogenes-roam-index-previous` | the one before |
| `diogenes-roam-find-passage` | find a passage note, any author |
| `diogenes-roam-find-by-author` | find one, an author at a time |
| `diogenes-roam-setup-latex-export` | have `diogenes:` links export as italics |

In an index buffer: `RET` opens a note, `q` shuts the index, `g` rebuilds it,
`M-n` and `M-p` walk the notes. Read-only, since it is generated; `C-x C-q` to
write prose above the block.

`diogenes-roam-index-next` and `-previous` work from a browser as well as from
the index. In a browser they take the line in view and open the first note
after it, or the last before; on a note, the one after that note. Always in
the order of the TEXT rather than the order the notes were written, which is
what a reader going through a dialogue wants.

## How the grouping works

Three groupings, answering three questions.

**Directories** — for browsing. `ref/plato/respublica/` in dired shows you
that you have forty notes on the *Republic* and three on the *Theaetetus*.
org-roam is indifferent to where a file sits, so this costs nothing and
breaks nothing.

**Tags** — for searching. Each note carries
`:passage:plato:respublica:`, so typing `plato justice` in
`org-roam-node-find` narrows as you would want. Set
`org-roam-node-display-template` to include `${tags}` or this does nothing
visible:

```elisp
(setq org-roam-node-display-template
      (concat "${title:70} " (propertize "${tags:40}" 'face 'org-tag)))
```

**`ROAM_REFS`** — for the passage. Not a grouping at all: it is what
`diogenes-org-notes` uses to find the notes bearing on the lines in front of
you, by arithmetic on the citation. Untouched by this package.

### Corpus names

The corpora name an author at length. Author 0086 of the TLG is `Aristoteles
Phil. et Corpus Aristotelicum, Aristotle (0086)` — the Latin name, a genre
marker, the English name, and the number. `diogenes-roam` takes the first
word of the first name, giving `aristoteles`.

Where that is not what you would type, `diogenes-roam-name-overrides` says
otherwise. Worth setting for the authors you read often; the rest can keep
the corpus's own name.

The LSJ abbreviations are the fallback, and are a poor one for directories:
Euripides is `E.` and Sophocles `S.`, which slugify to `e/` and `s/`. They
are only reached when the corpus is not installed.

## The index

An org-roam node like any other, with an ID, so you can link to it. Only the
dynamic block is regenerated — the header and anything you write above the
block survive, and the ID survives, which is the point: a rewritten file
would lose its ID and with it every link made to the index.

```org
#+title: Aristotle, Metaphysica — notes

#+BEGIN: dio-index :corpus "tlg" :author "0086" :work "025"
- [[id:f24c8d58…][1048a.27]] — Beere (2009, p. 170, 170n2) translates "καὶ γὰρ…
- [[id:…][1048b.5–1048b.9]] — …

/2 notes · 2026-09-12 14:22/
#+END:
```

Sorted by citation arithmetic, so `1048a.27` precedes `1048b.5` rather than
sorting alphabetically after it.

It rebuilds itself after every passage capture and whenever a passage note is
saved -- the index shows each note's first line, so an edit to that line would
otherwise leave it wrong. `g` rebuilds it by hand.

### The sidebar

`diogenes-roam-index-toggle` opens it in a side window beside the browser.
Side windows are not counted by `delete-other-windows`, so shutting the index
leaves the browser filling the frame with nothing to tidy away.

```elisp
(setq diogenes-roam-index-side 'right    ; or 'left, 'top, 'bottom
      diogenes-roam-index-size 0.35      ; fraction of frame; integer = columns
      diogenes-roam-index-sidebar t      ; nil for an ordinary window
      diogenes-roam-index-select t       ; nil to keep the cursor in the text
      diogenes-roam-index-open-in 'default) ; 'window or 'frame to force
```

A note followed from the index never opens *in* the index. `RET` is bound to
`diogenes-roam-index-open-at-point`, which shadows `org-link-frame-setup` for
the length of the call, because `find-file` would otherwise take over the
window it was called from. `default` leaves the window-or-frame choice to
`pop-up-frames`, so a reader who has set that for Diogenes' sake gets the
same behaviour here.

## Options

| | |
|---|---|
| `diogenes-roam-subdirectory` | where notes live under `org-roam-directory`, default `"ref"` |
| `diogenes-roam-tag` | the tag every passage note carries, default `"passage"` |
| `diogenes-roam-name-overrides` | what to call an author |
| `diogenes-roam-file-name` | `slug`, `citation`, or `timestamped` |
| `diogenes-roam-install-capture-template` | nil to write your own template |
| `diogenes-roam-index-name` | default `"index.org"` |
| `diogenes-roam-index-snippet-width` | how much of a note's first line to show |

`diogenes-roam-file-name` is worth a thought. `slug` is the note's title,
which is what org-roam would have chosen; but the title is built from the
citation, so inside `ref/plato/respublica/` it repeats what the path already
says. `citation` gives `327a-5-327b-4.org`, which sorts in the text's order.
`timestamped` cannot collide, at the cost of being unreadable.

## Caveats

Filenames can collide under `citation`, and under `slug` too, since both
derive from the same citation — two notes on exactly the same span want the
same name. `timestamped` is the answer if that happens to you.

`diogenes-roam` reaches into a few of `diogenes-org`'s private functions
(`diogenes-org--passage-parts`, `--levels`, `--where`, `--same-work-p`) and
one of `diogenes.el`'s (`diogenes--get-author-list`). Those may change. The
calls are wrapped where a failure is survivable, so a break should mean a
directory named `tlg0086` rather than a backtrace, but it is worth knowing.

The index is a node that links to every passage note, so each note gains a
backlink from its work's index. That is useful here — it tells you which text
a note belongs to. It would be noise if you did the same for concept notes.

## Licence

GPL-3 or later, matching `diogenes.el`.

[diogenes]: https://github.com/VictorSousa92/diogenes.el
[org-roam]: https://www.orgroam.com/
[straight]: https://github.com/radian-software/straight.el
[elpaca]: https://github.com/progfolio/elpaca
