hugo server -t diary --buildDrafts
hugo --theme=diary --baseUrl="https://gongluck.github.io" --buildDrafts
cd public
git init
git add .
git commit -m ""
git remote add origin http://github.com/gongluck/gongluck.github.io.git
git push -u origin master