---
title: "Emacs 29でTree-sitterでtsxの設定をする"
emoji: "🌲"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["emacs", "treesitter", "TypeScript"]
published: true
---
Emacsでtsxを編集するときのいい設定がないなあないなあと思って、長年いろいろ模索 (typescript-mode に rjsx-minor-mode を併用して `(define-derived-mode typescript-tsx-mode typescript-mode "TSX")` して `(typescript-tsx-mode-map  ("<" . rjsx-electric-lt) (">" . rjsx-electric-gt))` したり)やっていたんですが、どうもインデントとかがおかしくなりがちでした。

ここにきて **Emacs 29からならtree-sitterで設定できそう** だとわかったのでメモします。


## 手順

まずmacOSでHomebrewを使っている場合は tree-sitter を次でインストールしておきます。

```sh
brew install tree-sitter
```

そして、Emacsをビルドします。

そして、Emacsのソースの `admin/notes/tree-sitter/build-module` というところに、なんと `./batch.sh` というスクリプトがあるので実行します。

```sh
cd (emacsのソースの)emacs/admin/notes/tree-sitter/build-module
./batch.sh
```

そして、できたものを `~/.emacs.d/tree-sitter/` に放り込みます。

```sh
mkdir ~/.emacs.d/tree-sitter
mv できたもの ~/.emacs.d/tree-sitter/
```

`できたもの` は macOS では `*.dylib` でいいです。


あとはEmacsの設定です。Leafを使った例です。

tree-sitterでtsxが使えるようになっているので、拡張子が tsx のときに tsx-ts-mode を使うようにしているところがポイントです。

```emacs-lisp
(leaf tree-sitter
  :ensure (t tree-sitter-langs)
  :hook
  (tree-sitter-after-on-hook . tree-sitter-hl-mode)
  :config
  (global-tree-sitter-mode)

  (tree-sitter-require 'tsx)
  (add-to-list 'tree-sitter-major-mode-language-alist '(typescript-tsx-mode . tsx)))

(leaf tree-sitter-langs
  :ensure t
  :after tree-sitter
  :config
  (tree-sitter-require 'tsx)
  (add-to-list 'tree-sitter-major-mode-language-alist '(typescript-tsx-mode . tsx)))
  
(leaf typescript-mode
  :ensure t
  :mode 
  (("\\.ts\\'" . typescript-mode)
   ("\\.tsx\\'" . tsx-ts-mode))
  :custom
  (typescript-indent-level . 2)
  :setq 
  (typescript-tsx-indent-offset . 2)
  :hook
  (typescript-mode-hook . subword-mode)
  :config)

(leaf tide
  :ensure t
  :config
  (add-hook 'typescript-mode-hook
            (lambda ()
              (tide-setup)
              (flycheck-mode t)
              (setq flycheck-check-syntax-automatically '(save mode-enabled))
              (eldoc-mode t)
              (company-mode-on))))

```

## 謝辞 in advance

おかしいところや改善点がありましたらどうかお教え下さい。ありがとうございます in advance。


## 追記: tree-sitter-module の入れ方

tree-sitter-module を入れるときは、わざわざEmacsのソースを持って来なくても https://github.com/casouri/tree-sitter-module/releases から取ってきたり https://github.com/casouri/tree-sitter-module/ を取ってきて自分で `./batch.sh` でビルドしてもいいようです。

参考文献: https://git.savannah.gnu.org/cgit/emacs.git/tree/admin/notes/tree-sitter/starter-guide
