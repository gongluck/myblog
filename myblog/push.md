git submodule add -f https://github.com/amzrk2/hugo-theme-fuji.git themes/fuji
hugo server -t fuji --buildDrafts
hugo --theme=fuji --baseUrl="https://gongluck.github.io" --buildDrafts
cd public
git init
git add .
git commit -m ""
git remote add origin http://github.com/gongluck/gongluck.github.io.git
git push -u origin master