git clone https://github.com/vimux/mainroad
hugo --theme=mainroad --baseUrl="https://gongluck.github.io" --buildDrafts
cd public
git init
git add .
git commit -m ""
git remote add origin http://github.com/gongluck/gongluck.github.io.git
git push -u origin master