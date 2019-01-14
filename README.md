## 01/02/2019 Wednesday 3:00pm

- Had to figure out that I never selected EC2 as the trusted source when originally creating my role and frigged around with users, groups and other things including installing the aws cli. I now know a little more about access and IAM roles, groups and users. 
- Now finding out I have a to have a codedeploy role and a ec2 role. OKay!
- Deployments are failing due to: The overall deployment failed because 
  - 1. too many individual instances failed deployment, 
  - 2. too few healthy instances are available for deployment,
  - or 3. some instances in your deployment group are experiencing problems.
- Makes me think that this is because I don't have a load balancer. I want to create a load balancer but have to set up a certificate for that to happen. ACM manager time. 
- Had to use a classic load balancer because I had to mess around with godaddy DNS to get a ACM certificate which is needed for a non classic load balancer.
  - Got the load balancer working.
- The code deploy is failing because of instance health. 

## 01/03/2019 Thursday 6:30pm
  - Overall goal is to have all my websites hosted on aws
    - Where we left off last night was that I finally got the roles, users, and groups set in IAM as well as in EC2 & CodeDeploy  
    - The deploy is failing because of instance health.
      - "To resolve the issue, review and correct any errors in the the health check configuration for the load balancer."
      - It seems that it is failing when it gets to the blockTraffic step, which makes me think that the load balancer is to blame
      -- **The issue is that it might be that codedeployagent is not installed on my instance, which would make sense.**
      - Time to install putty
      - I have changed the EC2 keys to ppk format to use with putty and am about to start a session
      - We are connected! Now I need to install codedeploy agent on the instance using this guide [https://docs.aws.amazon.com/codedeploy/latest/userguide/codedeploy-agent-operations-install-linux.html](https://docs.aws.amazon.com/codedeploy/latest/userguide/codedeploy-agent-operations-install-linux.html). Done!
    - The deployment got to download bundle and succeeded, but now is throwing an error about an appspec [http://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file.html](http://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file.html)
      - Created and committed an `appspec.yml` copied from here [https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-example.html](https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-example.html)
      - Thought I was smart and changed the version to 0.1 and got this error (The deployment failed because an invalid version value (0.1) was entered in the application specification file. Make sure your AppSpec file specifies "0.0" as the version, and then try again :p)
      - Atleast I get a chance to change the load balancer timeouts which currently takes... 30 seconds nvm or so I thought. Something takes 5 minutes for block traffic to resolve. I have to change the healthy threshold from 10 to 2
        == Leaving off here, having to edit my `appspec.yml` to run npm build and npm start [https://hub.packtpub.com/deploy-nodejs-apps-aws-code-deploy/](https://hub.packtpub.com/deploy-nodejs-apps-aws-code-deploy/)
  - Next goal will be to delete root access permissions and update the healfly app with s3 only ones. (Hopefully before I push this!)
    -- Worked with relative ease, added s3fullaccess to user, group, role and to a custom policy. 

## 01/04/2019 Friday 8:00pm
  - Again overall goal is to have all my websites hosted on AWS and then if I have time figure out what lambda is about
  - But first I must do my weekly timesheets!
  8:25pm 
      - About to dig into how to create an `appspec.yml`. I know that I want the server to run npm install, (install npm as well), npm build and then npm start when called. 
        - Installed node & npm v 4.4.5 on my EC2 Instance using this guide [https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html)
        - Copied the `appspec.yml` and scripts folder (but added npm build after npm install) into my app from [https://github.com/brettstack/node-starter](https://github.com/brettstack/node-starter)
          - We get to afterInstall and failed with this message  `npm: command not found.`
            - Found out that node and npm are installed at `/home/ec2-user/.nvm/versions/node/v4.4.5/bin/npm` . Using the commands found in this article to shift the link to ... Not sure where I should link it to or where I should install npm to. 
              - I didn't install it globally because I didn't use sudo, which would have installed it globally. Wouldn't have minded doing that but now I see this article and realize that it could lead to errors down the road [https://givan.se/do-not-sudo-npm/](https://givan.se/do-not-sudo-npm/)
              > Packages can run arbitrary scripts, which makes sudoing a package manager command as safe as a chainsaw haircut. – @izs, npm core
              - Now I just have to make a symbolic link to node, npm, node-waf, *(react-scripts would find out later)* in the correct directory, which to me looks like either `/usr/bin/node` or `/usr/local/bin/npm` [https://stackoverflow.com/questions/4976658/on-ec2-sudo-node-command-not-found-but-node-without-sudo-is-ok](https://stackoverflow.com/questions/4976658/on-ec2-sudo-node-command-not-found-but-node-without-sudo-is-ok)
                - Running these 4 commands (
                  - `sudo ln -s /home/ec2-user/.nvm/versions/node/v4.4.5/bin/node /usr/bin/node`
                  - `sudo ln -s /home/ec2-user/.nvm/versions/node/v4.4.5/lib/node /usr/bin/node`
                  - `sudo ln -s /home/ec2-user/.nvm/versions/node/v4.4.5/bin/npm /usr/bin/npm`
                  - `sudo ln -s /home/ec2-user/.nvm/versions/node/v4.4.5/bin/node-waf /usr/bin/node-waf`
                  - in the future I did `sudo ln -s /var/api/node_modules/.bin/react-scripts /usr/bin/react-scripts`)
                - It seems to be failing at the after install again. I'm now suspicious of the `#!/bin/bash   cd /var/api`  part if npm works... Isn;t failing on time so I won't know until next time.. Time to ponder what I could have done wrong.. 
              -- **NPM Works now!!!** But now my script timed out: (Script at specified location: scripts/npm-install.sh failed to complete in 300 seconds)
                == The logs are saying that node 4.4.5 is too old -_-
        - All these changed on my instances infrastructure makes me think about version control for it. Looking at using Ec2Audit as described in this article [http://offbytwo.com/2012/08/03/audit-ec2-infra-with-scm.html](http://offbytwo.com/2012/08/03/audit-ec2-infra-with-scm.html)

## 01/05/2019 Saturday 1:15 pm
  - Goal again for the fourth day is to get all my sites hosted on ec2 and then learn about either next JS or lambda, or about using my own hardware.
    - First thing is to update node using nvm install 8.9.0 (`--reinstall-packages-from=4.4.5` didn't work)
    - Failed deployment with `exit code 1 "react-scripts: command not found"` 
    - I also notice that it is still using node 4.4.5 and I assume that's because of the all the linking commands I ran. I think this is causing the issue.  
      - Was going to run the linking commands again but instead just ran `npm i -g npm` and `nvm install 8.9.0` in the root folder
      - Next I ran `npm i -g react-scripts` 
      - running `npm ls react` showed that I didn't have react scripts installed. 
        - The output when running `npm ls react` should be 
        > calculator-app@0.1.0 /Users/ka_stewart/devmtn:notes/React.js/calculator-app
        > ├── react@16.0.0
        > └─┬ react-scripts@1.0.14
        >   └─┬ react-dev-utils@4.1.0
        >     └─┬ react-error-overlay@2.0.2
        >       └── react@16.0.0  deduped
          - But I get empty when I run in `/home/ec2-user` and I only get this one line when I run in `var/api` (including when I run in `node_modules`) where my project is installed.
            > api@1.0.0 /var/api
            > └── react@16.7.0
          - Now pondering if I should put my project in a different folder; I put it in the base folder because I thought it would get more access to commands and such. But also when I run npm ls react in my user folder I get (empty)
      - Now it doesn't show it still but when I run `npm ls react-scripts` it shows it... and npm start works now.
    - Next error was that I didn't have a browsers list in my package.json so I added it and pushed it and deployed it. [https://github.com/browserslist/browserslist#queries](https://github.com/browserslist/browserslist#queries)
      - The list I used was
       `"browserslist": [
          "last 1 version",
          "> 1%",
          "IE 10"
        ]`
    - It keeps going back to node 4.4.5 when I run the deployment
      - aaaaaaand were doing a new instance. 
        - To see if I can do it from scratch without looking at my prior notes, here's how I think it goes:
          - 1. you link it to the code deploy in console *right*
          - 2. you link it to the load balancer (*wrong* the load balancer is added to the de)
          - 3. you get the keys for the putty connection *wrong*
          - 4. you install code deploy agent (I'm doing everything in the root folder this time, except for installing codedeploy) *right*
          - 5. You install nvm and npm *right*
          - 6. you deploy? *right*
      - Now getting the `npm: command not found error`. Trying to do `npm i -g npm` instead of doing those links.
      - Npm still not found. 
      -- **Running the link commands again but this time with `sudo ln -s /var/api/node_modules/.bin/react-scripts /usr/bin/react-scripts`**
        -- **`sudo rm -r /usr/bin/node` to remove it if you screw up the version**
      - Running the deployment, and react-scripts command works and it launched the server to localhost:3000 
        - Then it times out after 60 seconds with this error: 
          `"ScriptTimedOut: Script at specified location: scripts/npm-start.sh failed to complete in 60 seconds"` 
        - In the terminal: 
          `LifecycleEvent - ApplicationStart
          Script - scripts/npm-start.sh
          [stdout]
          [stdout]> api@1.0.0 start /var/api
          [stdout]> react-scripts start
          [stdout]
          [stdout]Starting the development server...
          [stdout]
          [stdout]Compiled successfully!
          [stdout]
          [stdout]You can now view api in the browser.
          [stdout]
          [stdout] Local: http://localhost:3000/
          [stdout] On Your Network: http://172.31.39.77:3000/
          [stdout]
          [stdout]Note that the development build is not optimized.
          [stdout]To create a production build, use npm run build.
          [stdout]`

## 1/7/2019 Monday 6:00pm
  - All I want to do tonight is have the server run to an external ip.
    - There are a few ways I can do this: 
      [https://zeit.co/examples/create-react-app/](https://zeit.co/examples/create-react-app/)
        - This option sounds like the ultimate: SSR with Serverless. Will look this up and probably will be the end point, but not now. Also where the containers at?
     [https://medium.com/@sgobinda007/setting-up-react-redux-application-for-production-and-hosting-in-aws-ec2-8bbb8bf3c643](https://medium.com/@sgobinda007/setting-up-react-redux-application-for-production-and-hosting-in-aws-ec2-8bbb8bf3c643)
        - This option uses react and redux, but doesn't seem to use react-scripts or create-react-app
      [https://medium.com/@nishankjaintdk/setting-up-a-node-js-app-on-a-linux-ami-on-an-aws-ec2-instance-with-nginx-59cbc1bcc68c](https://medium.com/@nishankjaintdk/setting-up-a-node-js-app-on-a-linux-ami-on-an-aws-ec2-instance-with-nginx-59cbc1bcc68c)
        -- **This seems to be the best option because I only have to follow steps 7-9** aaaand this is the guide I should have been following all along. Could have skipped codedeploy completely and just installed git on the server. 
      [https://www.npmjs.com/package/codedeploy-scripts](https://www.npmjs.com/package/codedeploy-scripts)
        - This is a package that replaces all scripts for lifecycle events in the appspec file with one file called deployment.js. Sounds easy but doesn't have a guide with it and I don't think it will be that common/useful for me in the future.
    - We are live after adding a new custom TCP rule with port range 3000 and accessible from everywhere (0.0.0.0/0) to the security group of the instance, so the site is now available at the publicDNS ip port 3000
      [**http://ec2-18-191-187-105.us-east-2.compute.amazonaws.com:3000/**](http://ec2-18-191-187-105.us-east-2.compute.amazonaws.com:3000/)
      And it is fast, but I forgot I was messing with the transitions on the slider when I pushed up the recent commits!
    - We are able to have it running without my terminal open by installing pm2 with `npm install pm2 -g` and then in var/api running: 
      `pm2 start npm --name reactscriptsstart -- start`
    
## 1/8/19 Tuesday 6:30 pm
  - Hungry
  - Today going to follow the rest of this guide [https://medium.com/@nishankjaintdk/setting-up-a-node-js-app-on-a-linux-ami-on-an-aws-ec2-instance-with-nginx-59cbc1bcc68c](https://medium.com/@nishankjaintdk/setting-up-a-node-js-app-on-a-linux-ami-on-an-aws-ec2-instance-with-nginx-59cbc1bcc68c)
    - Questions I have off the bat: do I have to run another instance to host another website or just run them on another port? 
      - To test this I added a rule allowing access to port 3001 in security groups for the instance
      - Now I have to code deploy and npm install my sperry farms app
        - I'm just going to install git on my server and clone it that way >:|
        - Installed git but was unable to clone inside of root folder or /var and not sure why. cloned at home/ec2-user .
          - Have to change the port that the app runs on: going to make a branch for the linux distro and pull from there
          - Made branch and changed port to 3001 as well as updated banners on the order page.
          -- **It is hosted and working on port 3001** ! Maybe I should have finished the guide and figured out how xginx works first? 
    - I ran pm2 startup which gave me this script which I ran then I ran pm2 save after. `sudo env PATH=$PATH:/home/ec2-user/.nvm/versions/node/v9.3.0/bin /home/ec2-user/.nvm/versions/node/v9.3.0/lib/node_modules/pm2/bin/pm2 startup systemd -u ec2-user --hp /home/ec2-user`
    - And now when I reboot the server the apps start automatically. 
    - I installed nginx, added port 80, and then set the location in the config file (all in the guide.) 
      - **Pairbnb now works at the public dns without port:3000 extension**
    - Now I have to figure out how to get my other website served without port 3001.
    - Port 3001 no longer working and I'm not sure why, it is running in pm2 and allowed in security groups. 
    
## 1/9/19 Wednesday 7:45 pm
  - Had a crappy day at work
  - Felt bad for myself and so I decided to relax and induldge in twitch instead of programming for 2 hours.... Also because my work is somewhat ambigious right now
  - I'll atleast define where I'm at:
    - I have to get sperryfarms.org to map to the public dns of my instance
    - I also have to get the same for pairbnb?
      - List of nice to haves: https, ssr, serverless, 1 file to load, this article looks good (https://dev.to/saigowthamr/server-side-rendering-react-apps-using-serverless-jfe)


## 1/10/19 Thursday 4:00 pm
  - Going to fix the readme's on my apps today because they are very important and then if I have time I am going to put healfly onto my aws server
  - Not sure if I should fix errors/make edits in the code as I go 
    - Just gonna finish the readmes and outline the errors/edits I want to make.
  - First up: pairBNB
    - I already want to markup my dev log with ticks and bolding now that I see the syntax to do so on github
      - Here are the most important: --- gives a line, ## = h2, *word* = italics, **word** = bold, `word` = code, ```javascript word``` = js code, [text](site) = link, ![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1") = image
    - And now I don't haha just the code and the headings.  

## 1/11/19 Friday 2:45 pm
  - Couple things that could be worked on today
    - Either I start in on the readme's (priority)
    - Or I figure out why port 3001 isn't working (not important now)
    - Also was listening to a video talking about how the projects on your portfolio might not be 100% and I agree I should definitely work on pairbnb a little more. 

## 1/12/19 Saturday 
  - An idea I have for my portfolio site is to have the screenshots turn to scrolling gifs on hover
  - Today I have been working on using Icons in my readme... sort of a distraction from the important work but when i use icons in all my readmes they should look good.
    -- **After looking for a while I found [https://konpa.github.io/devicon/](https://konpa.github.io/devicon/) which is exactly what I'm looking for: same size developer icons. They are only missing lambda and graphql.** 
    -- [https://svgporn.com/](https://svgporn.com/) also looks very good and has way more icons, only available for download however you can't embed them

## 1/13/19 Sunday 11:30am Quill Books
  - Lots of hype up to this day because the past week which was spent working at home was not good and now I am at the bookstore. 
  - Today going to work on just the readme for Node PairBNB
    - Around commit 46 I'm thinking I should use a blank repo to do the readme editing because I have to make a commit everytime I want to see a change. 
      - I can reset the commits by doing `--hard head~x` where x is the number of commits I want to roll back, so I will do that when I am done and then paste in the finished readme. 
  - Finished the readme! Actually couple things I should add: 
    - The HTML5 link to the original website
    - That I run off the main.js in the public folder
    - That I have to update the transition
    - That I'm still working on getting the fade transition
    - That I couldn't figure out the import into the app and instead just had the defered tags in index.html
    - Link to the github in the about section
  - Good Day overall! 

## 1/14/19 Monday 3:30pm Glickman Library
  - First up let's take care of the backlog from yesterday
  - After that I hope to finish the readme for sperry farms. 
    - Finished it except for the chicken page and the gif there. But good!