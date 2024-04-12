# dev4all

git config --global user.name "ccbruce"
git config --global user.email "4055588+ccbruce@users.noreply.github.com"

git rm --cached `git ls-files -i -c --exclude-from=.gitignore` 

development 
nocodb
   https://docs.nocodb.com/engineering/development-setup/

Clone the repo
git clone https://github.com/nocodb/nocodb
# change directory to the project root
cd nocodb

Install dependencies
# run from the project root
# this command will install the dependencies including sdk build
pnpm bootstrap

Start Frontend
# run from the project root
pnpm start:frontend
# runs on port 3000

Start Backend
# run from the project root
pnpm start:backend
# runs on port 8080

Any changes made to frontend and backend will be automatically reflected in the browser.