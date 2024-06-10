重建blog步骤:

0. cd hexo-site
1. sudo pacman -S nodejs npm --noconfirm
2. sudo npm install -g hexo-cli
3. npm i hexo-abbrlink --save
4. npm i hexo-word-counter --save
5. sudo pacman -S pandoc
6. npm un hexo-renderer-marked
7. npm i hexo-renderer-pandoc --save
8. npm i hexo-generator-searchdb --save
9. npm i hexo-generator-feed --save
