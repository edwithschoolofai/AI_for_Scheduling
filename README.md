# Sam the Slack Scheduler Bot

### 개론

이 코드는 다음 Siraj Raval의 유튜브 영상 AI for scheduling [this](https://www.youtube.com/watch?v=nvLJq4GnCg4&feature=youtu.be) 을 위한 코드입니다.

### 목적

생산성은 빠른 속도의 작업 환경을 따라가기 힘듭니다. 수십 개의 브라우저 탭, 바탕화면 앱과 휴대폰 알람은 여러분이 집중하지 못하게 방해합니다. Google, Facebook, Amazon, IBM 과 같은 기업에서는 강력한 AI 기반의 플랫폼과 도구를 지속적으로 만들어 실제 사용자 문제에 기계학습을 적용하는 데 있던 걸림돌을 급격하게 낮추고 있습니다.

The Slack Scheduler Bot 은 크고 addressable한 시장 (미팅 일정을 잡고 Slack을 사용하는 팀) 의 업무량을 정확한 시간 내에 (새롭고 강력하며 사용하기 쉬운 인공지능 기술의 도래 + 직장에서 사용되는 소프트웨어 도구의 확산) 간소화함으로써 실제 사용자 문제를 해결합니다.


### 개론

TL;DR: The Slack Scheduler bot 은 구글 캘린더를 사용하는 다른 slack 사용자와의 리마인더 혹은 미팅을 잡는 언어 인식 인텔리젼트 slack bot입니다.

Slack Scheduler bot 은 여러 비동기 API 들로 만들어져, 다중 사용자들의 상태를 유지하는 데 매우 힘듭니다. 세 가지 중요 API는 Slack RTM, API.AI, a그리고 Google Calendar API 입니다. Slack bot 이 메세지를 보내면, 즉각적으로 API.AI 에 보내고 이를 분석하여 미리 저장된 의도를 파악한 뒤 사용자의 입력을 기반으로 반응을 어떻게 할 지 결정합니다. 예를 들어, 다음과 같이 입력했을 때 "*특정 사람과* *특정 시간에* *특정 날짜에* *미팅 주제* 를 얘기나누기 위해 *1 시간 동안* 미팅 약속을 잡아주세요". 이탤릭체에 해당하는 부분이 없으면, 특정 부분을 사용자에게 다시 물어볼 것입니다. 소요 시간은 필요하지 않지만, 입력되지 않은 경우 기본값은 30 분 입니다.
  
```Example:  
User: "Schedule a meeting with @pneelde tomorrow"  
Bot: "What Time?"  
User: "3pm"  
Bot: "What will be the topic of the meeting?"  
User: "Milestone #3"  
Bot: "Confirm meeting"  
Bot: "Meeting Confirmed"  
```
  
  
Scheduler bot 은 양 사용자의 캘린더에 일정을 추가할 권한이 있는지 확인합니다. 권한이 없다면, 사용자에게 링크를 보내 캘린더 수정 권한을 요구합니다.
  
권한을 받고 나면, 양 사용자의 캘린더를 확인하여 다른 사용자와의 선약이 있는지 확인합니다.

선약이 있다면, bot 은 답장하여 양 사용자들에게 선약이 있음을 알리고, 미팅에 참석할 모든 사용자가 가능한 10 개의 미팅 시간 후보 리스트를 보여주고 고르도록 합니다. 이는 하루 당 최대 세 개의 타임만 잡도록 제한되어있으며 이 빈 시간 이전 7 일을 확인합니다. 소요 시간이 입력된 경우, 미팅 소요 시간 동안 전부 가능한지 확인합니다. 그렇지 않은 경우, 30 분을 기본값으로 잡습니다.

사용자는 가장 적합한 시간을 고르고 bot은 미팅 시간을 잡고 각각의 사용자의 캘린더에 일정을 추가하고 메일을 보냅니다. 
  
Scheduler bot 은 can also create reminders for users. Using the same process as scheduling, the bot will set a reminder in the users calendar. These reminders are also added to an internal list of reminders that the scheduler will remind  the user of the task at midnight, and the user will see the reminder in their slack Direct Messages when they next open slack. This is done through the `script.js` file, Heroku's script scheduler, acting as a cron job to fire every night at midnight.  
  
```Example:  
User: "Remind me to turn in progress report"  
Bot: "When should I remind you?"  
User: "Tomorrow"  
Bot: "Confirm reminder"  
Bot: "Reminder Set"  
```

### Usage

#### Authenicating throught Google:  
![](https://cl.ly/1u3x1y2Y1W1u/Screen%20Recording%202017-08-01%20at%2008.12%20PM.gif)

#### Set Reminders:  
![](https://cl.ly/0T3Z3c0m2n2k/Screen%20Recording%202017-08-01%20at%2008.22%20PM.gif)

#### Schedules Meetings with other Slack Users:

##### Checks Permissions for both users:  
![](https://cl.ly/1p39180G0a0O/Screen%20Recording%202017-08-01%20at%2008.25%20PM.gif)

##### Schedules meeting on both users calendar:  
![](https://cl.ly/0g200n0I2w1M/Screen%20Recording%202017-08-01%20at%2008.32%20PM.gif)

##### Checks for time conflicts for both users:  
![](https://cl.ly/3S1N0q2a1W0b/Screen%20Recording%202017-08-01%20at%2008.40%20PM.gif)  
![](https://cl.ly/0o401z1J2k2k/Screen%20Recording%202017-08-01%20at%2008.48%20PM.gif)


## Running the tests

Since the free version of Heroku has been used, the server will automatically sleep after 12 hours. To test this application, simply deploy this project to heroku, set the Heroku address with the */slack/interactive* endpoint to the request URL for interactive messages in the Slack API application dashboard. For example, here is my interactive message endpoint:  
*https://dry-sierra-94244.herokuapp.com/slack/interactive*  
  
You will also need to set up a mongodb. Instructions on how to do this can be found [here.](http://fredrik.anderzon.se/2017/01/17/setting-up-a-free-mongodb-database-on-mlab-and-connecting-to-it-with-node-js/)  

Finally, set all of your environmental variables in heroku. You will need the following names for your environment variables:  
API_AI_TOKEN: *Your API.AI token*  
CLIENT_ID: *Your Slack credentials*  
CLIENT_SECRET: *Your Slack credentials*  
DOMAIN: *Heroku domain name*  
GOOGLE_CLIENT_ID: *Google API Credentials*  
GOOGLE_CLIENT_SECRET: *Google API Credentials*  
MONGODB_URI: *Mongo DB credentials*  
SLACK_API_TOKEN: *Bot User OAuth Access Token*  


## Built With

* [API.AI](https://api.ai/) - The ai/machine learning tool used to process natural language  
* [Slack](https://api.slack.com/) - Chat application used for RTM and interactive messages
* [Google Calendar API](https://developers.google.com/google-apps/calendar/) - Used by bot to schedule meetings and reminders  
* [Express](https://expressjs.com/) - Backend Server  


## Credits

Credits for this code go to [floresfxf](https://www.youtube.com/watch?v=nvLJq4GnCg4&feature=youtu.be). I've merely created a wrapper to get people started. 
