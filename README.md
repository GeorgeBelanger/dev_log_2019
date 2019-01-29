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

## 1/15/19 Tuesday 3:00pm Quill
  - Going to do the readme for healfly today.
    -  Got a decent amount done, but still have to get into the nitty gritty of authentication, search and others. But decent day. 
    -  Also fixed a broken link:
      - Had to change the default email in `app\mailers\application_mailer.rb` to healflymail@gmail.com and turn on 'allow less secure apps' at [https://www.google.com/settings/security/lesssecureapps](https://www.google.com/settings/security/lesssecureapps) and also go to [https://accounts.google.com/b/0/DisplayUnlockCaptcha](https://accounts.google.com/b/0/DisplayUnlockCaptcha) to allow unlock capchas for 10 minutes. 

## 1/16/19 Wednesday 3:00pm Quill
  - Finished the healfly readme. What a biotch lol still some glaring bugs but it's ok. 

## 1/17/19 Thursday 5:30pm home
  - Didn't do any development, just read on product hunt for 1 hour about differeneces between databases and new products. 

## 1/19/19 Saturday 11:45pm Quill
  - Going to remake my resume today.
    - Here is how I want it to look: 
      - <img src='https://lh3.googleusercontent.com/rXMDE_tUITNpcVLZdG5hJ0FHU3ZxR_iqbVYHFDtCEb8wDLAvsQmvnZOVSrtkuQrAMDAqvB0X_8TvHQyez3MU-pztTPK_PE0dUjFtMZooL3KzhHAZsXkSEmWzpzIc5SSiJ8ZjcJpQ3pe_xF9wV5B1uOgZsW90xx__o39j8IcQgvhnl5ZGsa7lv1YzD9z_aH7XTrtfLrqWT7xyNcP6aUtZwA7aKaYRDMoHtqlgDh2yeJelYpRTYKd92-gdDE9ewWFJZ9Kn_1H7YQ4YpIUnoT5rXsHw7IBIQytHRrKd5mmE-S_GtM9Tu6D-X03XxVgELxIKFHGGenM42W6rS9Po8OLn154rS_qKiwBx0JqlUjeL3O-Ckxs8-ABhKZvxX63BzP83cB-6qHTl6DEcZYcsQ1lwK5I2mMQ-w5lmbc0Ije_p-bNrm1rTECDqXLdwVta_5hWWx_TekVnv5fy6P8FCeCH786hRu-OYWTNWnqX5bXmeo3-NVbFxV737rg1rAdrsJX5TMyA3t6uTsEpCDzBx2jObjbhOOSy2tKFeEjO6zeKI2ZNE_1zXMyY-u3G0_3qszKh9zGvRf4DEmd6nVv327qmSLukZ3f7pops52InwnMSs8lBumZTyc8P5ff-xBciFyxUvz2bTI6yaQqH3fv9h_7X7ki3gtaZ2JZzziVbrM_MGoexZCHoa0QBtKNNgo9JbQPQrMlb9bl6LtnUCu6yWfoU=w950-h753-no'>

      - Not sure if the easiest way to do this is on google docs but that is where I want to do it. 
      - Looking for alternatives but all the templates they have aren't the quarter sections I am looking for, and a lot ignore the F chart of attention.
      - Another issue is that my old resume was not responsive when you viewed it on a phone. Not finding anything about making a word document or pdf responsive. 
        -- **It's all about those tables baby. I think I can get everything the way I like if I get the icons.**
      - This is what I got at the end of the first day, I like it! 
      <img src='https://lh3.googleusercontent.com/ih7YayuNgXZnH7cxi3U3EyOweYXCazQ2EkCzMhMcly2E1NQqq46iAufOt6cx1ykzWMib9R4hjT-9eEj-hPSiSvX18OTHwVejcVC9LEXE7OMlOvKJm7byi1120njgBM2ZzmkdjxgUI7O8QUKMHCctnnYJ5qY-io8_TZHhvAQsrOtyVJ0xWtKxdq4SyTDXRRV5_6lwB36A99iQAUZYL7082YfILFIjfR4PzsS_xz5HPodrdB8Teuwq93WqNrm-LVxoiXoSrbSMcALER4PuLS7KHKGntnSno0WSYSEBzjfpmYme51EWNtAXbQxu45lb-ZzKBf4-IGfrxcu8KQtJ9rZedrRqaUAev5NAIX0QhcCAwdSRBK3y7KztjKZKWKcSrUYMyaQjt2lPG153eyU5ghtxEcDAskfUGHJ9aWozplazTrLm3hvY9P23q1jYbKKiHjMFCGzYI_F5uArIIshs0T34stdkRUPVjtQ_3Mv-KuFG9yLkuXqiLMlhF_o4g_ZcFQGW8Uq0YI4Yw5jUA_bKZrJcziN7mNYo6O_80-BijYUXmvbEwg0562JAq6XaKQzjdknTkmRTR3bXpTLlli0uAlMGyMCYWH3mOQmxqI28Y5HqSO4ck76g36sD5shQxcjMe3GZJfHxSt7v-Gqe4ac0EUQr2PvhXI0t4nIGtPCKIUANjxeB5NbtD6uKjn_fu4VBooftBohlZ0Icf21gAJysK7U=w1589-h643-no'>
  - Also created .gitattribute files in my repos to change the language type of the repo. And updated sperry farms a bit. 

## 1/21/19 Monday 10:00am Quill
  - 2 Hours in and I have finished my resume and sent it to ziprecruiter, monster, indeed and linkedin. Here it is: [https://docs.google.com/document/d/1PP4NuqfVHf5CSl8WMJjxCY_E3U2iIEZ_9HPhz0O3hc8/edit?usp=sharing](https://docs.google.com/document/d/1PP4NuqfVHf5CSl8WMJjxCY_E3U2iIEZ_9HPhz0O3hc8/edit?usp=sharing)
  - Now it is time to apply to jobs. The key here is that I follow up in some fashion. That can either mean:
    - Calling in and checking that they saw my resume
    - Finding an email of someone that works there and sending them a reason why I want to work there
    - Sending someone that works there a message on linked in on what I'd be a good fit
  - It's also important that I do atleast 15 minutes of research on the company/product before I apply. 
    - I will first apply to jobs that are local because I think I have the highest chance of getting those all else equal. Should be able to get 5 today. 
      - Hall Internet marketing:
        - PHP and wordpress developer. No experience in either but there are a few elements on thier website that has the same effect as the ones on sperry farms
          - The counter
          - The typed.js
          - The mockups that I will have on my official portfolio (The focus this week(I should have an online kanban board or scrum thing.)))
        - Cool thing about these guys is that they have tools such as google analytics evaluator, website templates and dashboards. and the woo commerce accolade. 
        -- Sent an email to tims@hallme.com with my resume and a blerb. Felt kind of pushy but hey that's how it works.
      - Prosearch UX/UI Position with not much additional info
        - It said on Dice that I applied for this position on 11-27-2018 but reached out to Kathleen parker from pro serach to see if we can make something happen. 
      - Applied for a position at robert half
      - Could apply for a job at cashstar but feels bad
  - I used `sudo nano about.js` to edit the cringy hoped you liked the transition part at the beginning. It worked without having to update pm2 which is awesome.
  - Seems that I also have to use `sudo git` whenever I need to do a git command


## 1/22/19 Tuesday 5:00pm Quill
  - Going to work on PairBNB tonight because it is kind of a BS app at this point. It need to have atleast a few more pages. Only have 1 hour
    - I want to check out React spring tonight as well. It's between this, SVGator, react-transition-group, and Anime.js for my animation needs.
      - When I look at react-spring it looks moderately easy... and it's got ryan florance and dan abramov talking about it soooo
    - When I get back into the backend stuff, definitely looking at gitlab. 
    - The fade in doesn't work too well. It's not in sync and has a decent load time of 2 seconds
  - Basically just looked at react-spring examples tonight


## 1/23/19 Wednesday 5:45pm Home
  - Going to put a listing view page on pairbnb tonight and put some react-spring animation on it. 
    - Got my first spring animation on my page but,
      - Had to put everything in my app.js file
        - In order to have a global store, everyone is saying to use redux. I would use useContext instead of redux right? 
          - There are a few options I have for passing this state down:
            - Use traditional redux
              - This is going to be overkill but perhaps the most useful going forward
            - Use react hooks
              - I think that this might be the most appropriate because it is presumably the easiest, smallest and most forward moving. The only issue is that redux is alot more commercially viable/used and should be learned. 
            - Use redux-starter-kit
              - This is going to be better than the former starting out but not as easy/forward leaning as the mid.
        - I think this would be a good issue to do a version with each one to see the process and pros/cons.  
          - Going to start with redux-starter-kit first
            - Got sucked into trying to pass props down through react-router and couldn't get it to go. 
            - [https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367)
            - [https://stackoverflow.com/questions/41108701/react-router-share-state-between-routes-without-redux](https://stackoverflow.com/questions/41108701/react-router-share-state-between-routes-without-redux)
            - [https://github.com/reduxjs/redux-starter-kit](https://github.com/reduxjs/redux-starter-kit)
      - Had to revert the version to allow for 'template'
        - Nothing in the documentation about template and it seems that there is only one example using it so I just removed this example completely from my file. 
      
```
remote: This repository moved. Please use the new location:
remote:   https://github.com/GeorgeBelanger/PairBNB.git
remote:
remote: Create a pull request for 'react-spring-animations' on GitHub by visiting:
remote:      https://github.com/GeorgeBelanger/PairBNB/pull/new/react-spring-animations
remote:
```

## 1/24/19 thursday 3:00pm home
  - Going to try and set up animations with hooks before i do overkill redux
    - It was easy to set up local state using useState. I had to update the version of react and react-dom in `package.json` to "next"
  - Just kinda said F it I felt tired cuz I ate like crap today and took a nap and then never came back. 

## 1/25/19 Friday 2:40pm quill
  - There are a few things I have on the agenda for the next few days, probably saturday:
    - Get my taxes done x
    - Donate 1.8k x
    - Unsub from these spam job emails x
    - Look for rooms in portland month-to-month x
    - Create an automated investing account 
      - Applied for an account on Etrade
    - Move my 401K into an index fund
      - All I remember about the guy who managed my account is that he's from edward jones and I can't find anything in my email from edward jones
      - Sent Tom an email
    - Apply to tyler tech and send an email to HR@tylertechnologies.com
    - Get a virtual assistant to apply for junior jobs for me in austin and omaha and even seattle
      - Set up a new gmail account for the applications
      - Create a default email to send to people that he finds on hunter
      - Come up with guidelines of what jobs to apply to
      - If it asks for my SSN then stop the application
      - Put all the jobs he applied to in a spreadsheet with name of employer, date, address email was sent to
  - But today we are going to work on PairBNB listing page
    - I am first going to find a design that I like, then take steps toward implementing it and then and only then will I add additional features such as flights
    - Aaaaand now I think I am going to scrap any improvements on pairbnb.... 
      - I have to add navlink to the listings, but it breaks it when I try to wrap the article in a `<navLink>` element because of the CSS > and : and so on. But now I'm thinking if I make a navlink within the article that has padding that reaches to the edges of the article that could work. 
      - Continuted to mess around with it until I realized it wasn't going to happen: I made the link to the airbnb listing on the title a link to the landing page. 
      - Fixed the slider transition as well. 

## 1/27/19 Sunday 10:20 quill
  - Ok today I can either find out how to hire a virtual assistant or I can work on getting my landing page done.
    - I have been putting off the landing page and I know it. I have however found some awesome CSS libraries that I am exited to use. 
      -  I want to spend a bunch of time watching red Stapler videoes but I don't think that the library is the place to do it. 
      - [Awesome CSS Libraries](https://www.youtube.com/watch?v=r1ZtMCr0Pr4)
      - [Stunning CSS Effects (I really like the dematerializing one)](https://www.youtube.com/watch?v=bjUoQbSJDJs)
      - [Animate.css](https://daneden.github.io/animate.css/)
      - Next video I want to watch is on vs code extensions and I also want to try out fish shell
    - The virtual assistant I really want to get going by monday... but I think that I want to take it slow at first just until he gets the hang of it. 
      - I'm going to read in my virtual freedom book. 
    - Decided against doing this beacuse a guy said that he built a bot to apply to jobs and it got like 5 percent response rate. He advocated networking since only 6 percent of applications are referrals but 30 percent of hires are referrals. 
      - Still wonder how well it would work if he sent an email to someone from the company after each application like I will.
    - I am applying to places on my own now. Boston seems to have more positions open than in Austin and it's closer, thus I have a greater chance of hire. 
      - Applied to cyber coders Newington NH Jr. Full Stack role
      - Applied to sas SQL programmer boston imetris

## 1/28/19 Monday 4:15 Quill
  - Applying to jobs today, working on the new portfolio site tomorrow. I think I am going to leave pairbnb the way it is. 
    - Applied to SQL Programmer II at Southworth International Group on indeed
      - Sent an email to bmcnamara@southworthproducts.com
    - Applied to web developer position at redfin soultions through indeed
      - Sent a badass email to jobs@redfinsolutions.com
    - Applied for web developer at brickells through indeed
      - Sent an email to support@brickell
      - Makes me think I should use google analytics. I have vague experience with paid ads. 
    - Applied to junior web developer at iBec creative through indeed
      - Sent an email
    - Applied to compunell, alexander tech group, andiamo, through 1 click on linkedin. *blackhole*
    - Applied to CyberCoders in Portsmouth NH
      - Emailed Mike cordano about the position. 
    - Applied at workbridge for junior front end position in boston.
      - Sent an email to Lauren
    - applied for junior full stack position in portsmouth through cybercoders
  - Noticed that Mike cordaro manages a lot of positions that I could fit into.
  - The best method for applying to jobs on Monday is to filter by posted within the last week and then have the term "junior developer" for my various locations.

## 1/29/19 Tuesday 1:30pm Quill
  - Going to start in on the portfolio site today
    - Going to view that github repo that had a lot of great portfolios on it: [https://github.com/iRaul/awesome-portfolios](https://github.com/iRaul/awesome-portfolios)
      - I see from a lot of portfolios that everyone is using canvas and webGL(and three.js) to make thier websites look like GCI if you will. 
    - [http://cihadturhan.com/](http://cihadturhan.com/) might be the best portfolio I've seen. The works section is absolutely amazing I see a lot of these mouse parallax designs but his works one is the best I've seen and what I want to do with my projects.  
    - I also liked the background effect here [https://mamboleoo.be/](https://mamboleoo.be/)
    - 3D dragging is also a feature I'd like to have. Again I liked cihad's contact page
    - [http://samsy.ninja/](http://samsy.ninja/) is amazing as well. The portfolio is just ok but man the content is crazy. Using AR and webGL.
      - He linked to [https://www.stevenmengin.com/](https://www.stevenmengin.com/) where I really like a lot of things about his portfolio. 
    - [http://luruke.com/](http://luruke.com/) Is awesome and fun. Not applicable but wow
  - Need to decide what to use for the backend on my portfolio. It's already come down to either Next.JS or Gatsby.
    - I just found a nice tutorial on SSR vs CSR that uses Next.Js so I think that would be a good read, and potentially a good place to start.
    - I've also heard that gatsby is better suited for static sites and Next is overkill. 
      - gatsby has 19 job postings and next has 40. Nothing great if I was looking for a job. 
    - Still looking at netlify vs heroku and figuring out what a CMS is... 
      - Now I get it. A place where non-technicals(sorry) can change the content of the site.
  - It's looking like I'm going to use Gatsby due to the last image in this article [https://www.vojtechruzicka.com/gatsby-migration/](https://www.vojtechruzicka.com/gatsby-migration/)
    - So there are 6 jobs for netlify and netlify CMS while there are 6,000 for wordpress so not sure what benefit could outweigh that if I was going to use a CMS.
  - Starting the SSR vs CSR now.... 
